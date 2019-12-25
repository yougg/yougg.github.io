---
Categories: ["Document"]
Description: ""
Tags: ["Golang","Cassandra"]
title: "使用Cassandra"
date: 2017-12-11T20:28:59+08:00
draft: false
---


- Golang访问Cassandra的客户端[gocql](https://github.com/gocql/gocql)

- 事务支持

	```sql
	INSERT INTO tablex (column_a, column_b, column_c) VALUES ('a','b','c') IF NOT EXISTS;
	UPDATE tablex SET column_a = 'x', column_b = 'y', column_c = 'z' WHERE column_a = 'a' IF column_a = 'a';
	```

- SQL语句转义

	```go
	var replacer = strings.NewReplacer(
		`\`, `\\`,
		`'`, `\'`,
		`\0`, `\\0`,
		`"`, `\"`,
		"\n", `\n`,
		"\r", `\r`,
		"\x1a", `\Z`,
	)

	func EscapeSql(s string) (res string) {
		res = replacer.Replace(s)
		return
	}
	```

- Golang

	```go
	func getStructFieldNames(v interface{}) (names []string) {
		var t reflect.Type
		vv := reflect.ValueOf(v)
		if reflect.Ptr == vv.Kind() {
			t = reflect.TypeOf(v).Elem()
		} else if reflect.Struct == vv.Kind() {
			t = reflect.TypeOf(v)
		}
		num := t.NumField()
		if num > 0 {
			for i := 0; i < num; i++ {
				names = append(names, t.Field(i).Name)
			}
		}
		return
	}

	func toSnakeCase(v string) (x string) {
		var words []string
		l := 0
		for s := v; s != ``; s = s[l:] {
			l = strings.IndexFunc(s[1:], unicode.IsUpper) + 1
	
			if l <= 0 {
				l = len(s)
			}
			words = append(words, s[:l])
		}
		x = strings.ToLower(strings.Join(words, `_`))
		return
	}

	func toCamelCase(v string) (x string) {
		x = strings.Replace(strings.Title(strings.Replace(v, `_`, ` `, -1)), ` `, ``, -1)
		return
	}
	```
