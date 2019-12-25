---
Categories: ["Programing"]
Description: ""
Tags: ["Golang","MySQL"]
title: "用Go语言写一个简单的MySQL客户端"
date: 2017-08-24T09:30:43+08:00
draft: false
---

项目需要验证一下MySQL服务启用安全通信证书双向认证在Go中是否可行，以便后续将服务代码快捷的切换到SSL安全通道上。服务功能来做验证对环境依赖比较麻烦，那么就写一个简单的客户端先验证一下吧。

```go
//go:generate GOOS=windows go build -ldflags '-s -w' -o gosql.exe sql.go
//go:generate GOOS=linux go build -ldflags '-s -w' -o gosql sql.go

package main

import (
	"crypto/tls"
	"crypto/x509"
	"database/sql"
	"flag"
	"fmt"
	"io/ioutil"
	"log"
	"os"
	"reflect"
	"regexp"
	"strconv"
	"strings"
	"time"
	"unicode/utf8"

	"github.com/go-sql-driver/mysql"
)

var (
	host = flag.String("h", "127.0.0.1", "")
	port = flag.Int("P", 3306, "")
	user = flag.String("u", "myuser", "")
	pwd  = flag.String("p", "mypwd", "")
	dbn  = flag.String("D", "myuser", "")
	exec = flag.String("e", "", "")

	sslMode = flag.String("ssl-mode", "DISABLED", "")
	sslCA   = flag.String("ssl-ca", "", "")
	sslCert = flag.String("ssl-cert", "", "")
	sslKey  = flag.String("ssl-key", "", "")
)

func init() {
	// TODO read app.conf for default value
	flag.StringVar(host, "host", "127.0.0.1", "")
	flag.IntVar(port, "port", 3306, "")
	flag.StringVar(user, "user", "myuser", "")
	flag.StringVar(pwd, "password", "mypwd", "")
	flag.StringVar(exec, "execute", "", "")
	flag.StringVar(dbn, "database", "myuser", "")

	flag.Usage = func() {
		fmt.Fprintln(os.Stderr, `gosql Ver 1.0, A mysql command line client write in Golang for DataSight MRS
Copyright (c) 2017, yougg. All rights reserved.

Usage: gosql [OPTIONS] [database]
All followed flags are compatible with mysql command
  -h, --host      Connect to host. (default "127.0.0.1")
  -P, --port      Port number to use for connection or 0 for default to. (default 3306)
  -u, --user      User for login if not current user. (default "myuser")
  -p, --password  Password to use when connecting to server. (default "mypwd")
  -D, --database  Database to use. (default "myuser")
      --ssl-mode  SSL connection mode. (default "DISABLED")
                  PREFERRED:       Use ssl connection if server supports encrypted connections, else connect without ssl.
                  REQUIRED:        Use ssl connection if server supports encrypted connections, else connection attempt fails.
                  VERIFY_CA:       As REQUIRED, additionally verify server certificate against the configured CA certificates.
                  VERIFY_IDENTITY: As VERIFY_CA, additionally verify server certificate matches the host.
                  DISABLED:        Establish an unencrypted connection.
      --ssl-ca    CA file in PEM format.
      --ssl-cert  X509 cert in PEM format.
      --ssl-key   X509 key in PEM format.
  -e, --execute   Execute command and quit.

CAUTION: No interactive mode support.

Example:
    gosql -h 12.34.56.78 -P 3306 -u myuser -pmypwd -D myuser -e "show variables like 'ssl%'"
    gosql -h 12.34.56.78 -P 3306 -u myuser --password="mypwd" -D myuser --ssl-mode REQUIRED --ssl-ca ./ca.crt --ssl-cert ./client.crt --ssl-key ./client.key -e "CREATE TABLE test (v INTEGER)"`)
	}

	// compatible with mysql when flag contact with value without = or space
	for i, a := range os.Args {
		// go regexp unsupported Perl syntax: `(?!`
		if regexp.MustCompile(`^-[hPupDe].+`).MatchString(a) &&
			!regexp.MustCompile(`[=]`).MatchString(a) {
			os.Args[i] = a[:2] + `=` + a[2:]
		}
	}

	flag.Parse()
	if flag.NFlag() == 0 || `` == strings.TrimSpace(*exec) {
		flag.Usage()
		os.Exit(1)
	}
}

func main() {
	// TODO interactive mode support

	var tlsKey string
	switch *sslMode {
	case "PREFERRED":
		tlsKey = "true"
		// TODO reconnect without ssl
	case "REQUIRED":
		tlsKey = "skip-verify"
	case "VERIFY_CA":
		tlsKey = "custom"
		loadCert()
		// TODO reconnect without match Common Name of server certificate
	case "VERIFY_IDENTITY":
		tlsKey = "custom"
		loadCert()
	case "DISABLED":
		tlsKey = "false"
	}

	cfg := mysql.Config{
		User:            *user,
		Passwd:          *pwd,
		Net:             "tcp",
		Addr:            fmt.Sprintf("%s:%d", *host, *port),
		DBName:          *dbn,
		Loc:             time.Local,
		TLSConfig:       tlsKey,
		AllowAllFiles:   true,
		MultiStatements: true,
	}
	//fmt.Println(cfg.FormatDSN())
	//src := fmt.Sprintf("%s:%s@tcp(%s:%d)/%s?tls=custom", *user, *pwd, *host, *port, *dbn)
	db, err := sql.Open("mysql", cfg.FormatDSN())
	if nil != err || nil == db {
		log.Fatalln(err)
	}

	var key string
	statement := strings.Fields(strings.TrimSpace(*exec))
	if len(statement) > 0 {
		key = strings.ToUpper(statement[0])
	}
	switch key {
	case "INSERT", "UPDATE", "DELETE", "CREATE", "DROP", "SET", "BEGIN", "TRUNCATE", "LOAD":
		execute(db)
	case "SELECT", "SHOW", "DESC", "EXPLAIN":
		query(db)
	default:
		log.Fatalln("Execute command or sql statement was not support:", *exec)
	}
}

func loadCert() {
	rootCertPool := x509.NewCertPool()
	ca, err := ioutil.ReadFile(*sslCA)
	if err != nil {
		log.Fatalln(err, *sslCA)
	}
	if ok := rootCertPool.AppendCertsFromPEM(ca); !ok {
		log.Fatalln("Failed to append CA certificate PEM data.")
	}
	clientCert := make([]tls.Certificate, 0, 1)
	certs, err := tls.LoadX509KeyPair(*sslCert, *sslKey)
	if err != nil {
		log.Fatalln(err, *sslCert, *sslKey)
	}
	clientCert = append(clientCert, certs)
	tlsKey := "custom"
	mysql.RegisterTLSConfig(tlsKey, &tls.Config{
		RootCAs:      rootCertPool,
		Certificates: clientCert,
	})
}

// CASE INSERT/UPDATE/DELETE/CREATE/DROP/SET/BEGIN/TRUNCATE/LOAD statement
//  Ex: "update tableX set cloumnX = 'x' where cloumnY = 'y'"
//      "delete from tableX"
func execute(db *sql.DB) {
	res, err := db.Exec(*exec)
	if nil != err {
		log.Fatalln(err)
	}
	v, err := res.RowsAffected()
	if nil != err {
		log.Fatalln(err)
	}
	log.Printf("Processed %d row(s)\n", v)
	if strings.HasPrefix(strings.ToUpper(strings.TrimSpace(*exec)), "INSERT") {
		v, err = res.LastInsertId()
		if nil != err {
			log.Fatalln(err)
		}
		log.Println("Last insert ID:", v)
	}
}

// CASE QUERY/SHOW/DESC/EXPLAIN statement
//  Ex: "select * from tableX limit 1"
//      "show tables"
//      "show variables like 'ssl%'"
//      "desc tableX"
func query(db *sql.DB) {
	rows, err := db.Query(*exec)
	if nil != err {
		log.Fatalln(err)
	}
	var showCloumnName bool
	var data [][]string
	for rows.Next() {
		columns, err := rows.Columns()
		if err != nil {
			log.Fatalln(err)
		}
		if !showCloumnName {
			showCloumnName = true
			data = append(data, columns)
		}

		refs := make([]interface{}, 0, len(columns))
		for range columns {
			refs = append(refs, new(interface{}))
		}

		if err := rows.Scan(refs...); err != nil {
			log.Fatalln(err)
		}
		line := make([]string, 0, len(columns))
		for _, v := range refs {
			if reflect.TypeOf(v).Kind() == reflect.Ptr {
				i := reflect.ValueOf(v).Elem().Interface()
				switch i.(type) {
				case int, int8, int16, int32, int64,
					uint, uint8, uint16, uint32, uint64,
					float32, float64:
					line = append(line, fmt.Sprintf("%v", i))
				case []byte:
					line = append(line, string(i.([]byte)))
				case string:
					line = append(line, i.(string))
				default:
					line = append(line, fmt.Sprintf("%v", i))
				}
			}
		}
		data = append(data, line)
	}
	formatPrint(data)
}

// TODO output column mode
func formatPrint(data [][]string) {
	if len(data) <= 1 {
		// first row is column name
		fmt.Println("No row found")
		return
	}
	columnName := data[0]
	columnWidth := make([]int, len(columnName))
	for _, line := range data {
		for i, c := range line {
			if l := utf8.RuneCountInString(c) + countFullWidthChars(c); l > columnWidth[i] {
				columnWidth[i] = l
			}
		}
	}
	splitLine := make([]string, 0, len(columnName))
	for i, v := range columnName {
		columnName[i] = fmt.Sprintf("%-"+strconv.Itoa(columnWidth[i])+"s", v)
		splitLine = append(splitLine, strings.Repeat("-", columnWidth[i]))
	}
	spl := "+-" + strings.Join(splitLine, "-+-") + "-+"
	fmt.Println(spl)
	fmt.Println("| " + strings.Join(columnName, " | ") + " |")
	fmt.Println(spl)
	for _, line := range data[1:] {
		fmt.Print("| ")
		for i, c := range line {
			fmt.Printf("%-"+strconv.Itoa(columnWidth[i]-countFullWidthChars(c))+"s | ", c)
		}
		fmt.Println()
	}
	fmt.Println(spl)
}

func countFullWidthChars(s string) (count int) {
	for _, c := range s {
		// Unicode range of Chinese/Japanese/Korean characters
		// https://zh.wikipedia.org/wiki/%E4%B8%AD%E6%97%A5%E9%9F%93%E7%B5%B1%E4%B8%80%E8%A1%A8%E6%84%8F%E6%96%87%E5%AD%97
		if c >= 0x4E00 && c <= 0x9FFF || // 中日韩统一表意文字
			c >= 0x3400 && c <= 0x4DFF || // 中日韩统一表意文字扩展区A
			c >= 0x20000 && c <= 0x2EBEF { // 扩展区B (20000~2A6DF), 扩展区C (2A700~2B73F), 扩展区D (2B740~2B81F), 扩展区E (2B820~2CEAF), 扩展区F (2CEB0~2EBEF)
			count++
		} else {
			// Unicode range of full width symbols
			// https://zh.wikipedia.org/wiki/%E5%85%A8%E5%BD%A2%E5%92%8C%E5%8D%8A%E5%BD%A2
			switch c {
			case 0x2190, 0x2191, 0x2192, 0x2193, 0x2502, 0x25A0, 0x25CB, 0x3000, 0x3001, 0x3002, 0x300C,
				0x300D, 0x309B, 0x309C, 0x30A1, 0x30A2, 0x30A3, 0x30A4, 0x30A5, 0x30A6, 0x30A7, 0x30A8,
				0x30A9, 0x30AA, 0x30AB, 0x30AD, 0x30AF, 0x30B1, 0x30B3, 0x30B5, 0x30B7, 0x30B9, 0x30BB,
				0x30BD, 0x30BF, 0x30C1, 0x30C3, 0x30C4, 0x30C6, 0x30C8, 0x30CA, 0x30CB, 0x30CC, 0x30CD,
				0x30CE, 0x30CF, 0x30D2, 0x30D5, 0x30D8, 0x30DB, 0x30DE, 0x30DF, 0x30E0, 0x30E1, 0x30E2,
				0x30E3, 0x30E4, 0x30E5, 0x30E6, 0x30E7, 0x30E8, 0x30E9, 0x30EA, 0x30EB, 0x30EC, 0x30ED,
				0x30EF, 0x30F2, 0x30F3, 0x30FB, 0x30FC, 0x3131, 0x3132, 0x3133, 0x3134, 0x3135, 0x3136,
				0x3137, 0x3138, 0x3139, 0x313A, 0x313B, 0x313C, 0x313D, 0x313E, 0x313F, 0x3140, 0x3141,
				0x3142, 0x3143, 0x3144, 0x3145, 0x3146, 0x3147, 0x3148, 0x3149, 0x314A, 0x314B, 0x314C,
				0x314D, 0x314E, 0x314F, 0x3150, 0x3151, 0x3152, 0x3153, 0x3154, 0x3155, 0x3156, 0x3157,
				0x3158, 0x3159, 0x315A, 0x315B, 0x315C, 0x315D, 0x315E, 0x315F, 0x3160, 0x3161, 0x3162,
				0x3163, 0x3164, 0xFF01, 0xFF02, 0xFF03, 0xFF04, 0xFF05, 0xFF06, 0xFF07, 0xFF08, 0xFF09,
				0xFF0A, 0xFF0B, 0xFF0C, 0xFF0D, 0xFF0E, 0xFF0F, 0xFF10, 0xFF11, 0xFF12, 0xFF13, 0xFF14,
				0xFF15, 0xFF16, 0xFF17, 0xFF18, 0xFF19, 0xFF1A, 0xFF1B, 0xFF1C, 0xFF1D, 0xFF1E, 0xFF1F,
				0xFF20, 0xFF21, 0xFF22, 0xFF23, 0xFF24, 0xFF25, 0xFF26, 0xFF27, 0xFF28, 0xFF29, 0xFF2A,
				0xFF2B, 0xFF2C, 0xFF2D, 0xFF2E, 0xFF2F, 0xFF30, 0xFF31, 0xFF32, 0xFF33, 0xFF34, 0xFF35,
				0xFF36, 0xFF37, 0xFF38, 0xFF39, 0xFF3A, 0xFF3B, 0xFF3C, 0xFF3D, 0xFF3E, 0xFF3F, 0xFF40,
				0xFF41, 0xFF42, 0xFF43, 0xFF44, 0xFF45, 0xFF46, 0xFF47, 0xFF48, 0xFF49, 0xFF4A, 0xFF4B,
				0xFF4C, 0xFF4D, 0xFF4E, 0xFF4F, 0xFF50, 0xFF51, 0xFF52, 0xFF53, 0xFF54, 0xFF55, 0xFF56,
				0xFF57, 0xFF58, 0xFF59, 0xFF5A, 0xFF5B, 0xFF5C, 0xFF5D, 0xFF5E, 0xFF5F, 0xFF60, 0xFFE0,
				0xFFE1, 0xFFE2, 0xFFE3, 0xFFE4, 0xFFE5, 0xFFE6:
				count++
			}
		}
	}
	return
}
```

---

_**参考链接：**_  

- [Unicode range of Chinese/Japanese/Korean characters](https://zh.wikipedia.org/wiki/%E4%B8%AD%E6%97%A5%E9%9F%93%E7%B5%B1%E4%B8%80%E8%A1%A8%E6%84%8F%E6%96%87%E5%AD%97)
- [Unicode range of full width symbols](https://zh.wikipedia.org/wiki/%E5%85%A8%E5%BD%A2%E5%92%8C%E5%8D%8A%E5%BD%A2)
