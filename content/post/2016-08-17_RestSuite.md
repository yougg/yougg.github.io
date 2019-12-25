---
Categories: ["Programing"]
Description: ""
Tags: ["Golang","REST"]
title: "REST工具"
date: 2017-08-17T15:56:31+08:00
draft: false
---

```go
package main

import (
	"crypto/tls"
	"encoding/json"
	"flag"
	"fmt"
	"io/ioutil"
	"log"
	"net"
	"net/http"
	"os"
	"time"
)

type content struct {
	Title   string
	Desc    string
	Scheme  string
	Host    string
	Port    int
	Request struct {
		IP     string
		Url    string
		Method string
		Cert   struct {
			RootCAs      []string
			Certificates []string
		}
		Header map[string]string
		Body   string
	}
	Response struct {
		Status int
		Cert   map[string]string
		Header map[string]string
		Body   string
	}
}

type handler struct{}

// tcpKeepAliveListener sets TCP keep-alive timeouts on accepted connections.
// It's used by ListenAndServe and ListenAndServeTLS so dead TCP connections
// (e.g. closing laptop mid-download) eventually go away.
type tcpKeepAliveListener struct {
	*net.TCPListener
}

const (
	Http  = "HTTP"
	Https = "HTTPS"
)

var (
	file   = flag.String("f", "", "json file file")
	client = flag.Bool("c", false, "Client mode")
	server = flag.Bool("s", false, "Server mode, if no mode was set, this will be default")
	maxConn= flag.Int("m", 1000, "Max connections")
)

var (
	contents []content
	handle   handler
)

func init() {
	flag.StringVar(file, "file", "", "json file file")
	flag.BoolVar(client, "client", false, "Client mode")
	flag.BoolVar(server, "server", false, "Server mode")
	flag.IntVar(maxConn, "maxConn", 1000, "Max connections")
	flag.Parse()

	if *client && *server {
		// set server mode for default, if both client and server flag were set
		*client = false
		*server = true
	}
	if !*client && !*server {
		// set server mode for default, if both client and server flag were not set
		*server = true
	}
}

func main() {
	parseData()
	fmt.Println(*client, *server, *file)
}

func parseData() {
	fi, err := os.Open(*file)
	if os.IsNotExist(err) {
		log.Fatalln(err)
	}
	data, err := ioutil.ReadAll(fi)
	if nil != err {
		log.Fatalln(err)
	}
	err = json.Unmarshal(data, &contents)
	if nil != err {
		log.Fatalln(err)
	}

	// Test marshal back
	data, err = json.MarshalIndent(contents, "", "\t")
	if nil != err {
		log.Fatalln(err)
	}
	fmt.Println(string(data))
}

func custom() {

}

func serve() {
	listeners := make(map[int]http.Server, len(contents))
	for _, c := range contents {
		//if Https != c.Scheme {
		//	continue
		//}
		if svr, ok := listeners[c.Port]; ok {
			if Https == c.Scheme && nil == svr.TLSConfig {

			}
		} else {
			var config *tls.Config
			if Https == c.Scheme && len(c.Response.Cert) == 2 {
				cert, err := tls.LoadX509KeyPair(c.Response.Cert["chains"], c.Response.Cert["key"])
				if nil == err {
					config = &tls.Config{
						Certificates: []tls.Certificate{cert},
						NextProtos:   []string{"http/1.1"},
					}
				}
			}
			svr = http.Server{
				Addr:      fmt.Sprintf("%s:%d", c.Request.IP, c.Port),
				Handler:   handle,
				TLSConfig: config,
			}
			listeners[c.Port] = svr
			//svr.ListenAndServe()
			ln, err := net.Listen("tcp", fmt.Sprintf("%s:%d", c.Request.IP, c.Port))
			if err != nil {
				log.Fatalln(err)
			}
			tlsListener := tls.NewListener(tcpKeepAliveListener{ln.(*net.TCPListener)}, config)
			svr.Serve(tlsListener)
		}
	}
	for _, svr := range listeners {
		if nil == svr.TLSConfig {
			err = svr.ListenAndServe()
			svr.ListenAndServeTLS()
		}
	}
}

func (handler) ServeHTTP(w http.ResponseWriter, r *http.Request) {

}

func (ln tcpKeepAliveListener) Accept() (c net.Conn, err error) {
	tc, err := ln.AcceptTCP()
	if err != nil {
		return
	}
	tc.SetKeepAlive(true)
	tc.SetKeepAlivePeriod(3 * time.Minute)
	return tc, nil
}
```


```json
[
	{
		"title": "a HTTP test suite",
		"desc": "description for HTTP test suite",
		"scheme": "HTTP",
		"host": "a.b.c.com",
		"port": 20008,
		"request": {
			"ip": "127.0.0.1",
			"url": "/a/b/c",
			"method": "POST",
			"header": {
				"X-Auth-Token": "asdf"
			},
			"body": "this is body content"
		},
		"response": {
			"status": 200,
			"header": {
				"X-Subject-Token": "ASDFASFD"
			},
			"body": "server response body"
		}
	},
	{"port":20003,"request":{"ip": "127.0.0.1","url":"/a/b/c"},"response":{"status":200}},
	{"port":20004,"request":{"ip": "127.0.0.1","url":"/a/b/d"},"response":{"status":200}},
	{"port":20008,"request":{"ip": "127.0.0.1","url":"/a/b/e"},"response":{"status":200}},
	{"port":20010,"request":{"ip": "127.0.0.1","url":"/a/b/f"},"response":{"status":200}},
	{
		"title": "a HTTPS test suite",
		"desc": "description for HTTPS test suite",
		"scheme": "HTTPS",
		"port": 20028,
		"request": {
			"url": "/v2/service_instance/",
			"method": "POST",
			"cert": {
				"RootCAs": [
					"/path/to/roo_ca.crt"
				],
				"Certificates": [
					"/path/to/client.crt"
				]
			},
			"header": {
				"X-Auth-Token": "asdf"
			},
			"body": "this is body content"
		},
		"response": {
			"status": 200,
			"cert": {
				"key": "/path/to/service.key",
				"chains": "/path/to/service.crt"
			},
			"header": {
				"X-Subject-Token": "ASDFASFD"
			},
			"body": "server response body"
		}
	}
]
```
