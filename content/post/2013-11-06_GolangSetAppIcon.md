---
Categories: []
Description: ""
Tags: []
title: "Golang 设置应用程序图标"
date: 2013-11-06T00:00:00+08:00
draft: false
---

所有操作在Windows中的MingW环境中执行

```bash
# Go源文件
hello.go

# 图标源文件
hello.ico

# 创建rc文件
echo 'IDI_ICON1 ICON "hello.ico"' > hello.rc

# 生成资源目标文件
windres -o hello.syso hello.rc
# windres 帮助说明:  http://baike.baidu.com/view/10556594.htm

# 最后编译Go源码, 输出目标文件
# go build -ldflags '-s -w'
go tool 8g -o hello.8 hello.go
go tool pack grcP . hello.a hello.8 hello.syso
go tool 8l -o hello.exe -L . hello.a
```
