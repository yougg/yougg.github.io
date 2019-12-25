---
Categories: ["Document"]
Description: ""
Tags: ["Golang","UML","GoLand"]
title: "GoLand中使用PlantUML生成Go UML图"
date: 2017-11-25T10:48:48+08:00
draft: false
---

1. 确保如下软件已安装并添加到PATH中可执行

	- [Java](http://www.oracle.com/technetwork/java/javase/downloads/index.html)
	- [Golang](https://golang.org/dl/)
	- [GoLand](https://www.jetbrains.com/go/download)
	- [Graphviz](http://www.graphviz.org/Download.php)

2. GoLand中安装PlantUML插件

	1. 点击菜单 `File` -> `Settings` -> `Plugins` -> `Browse repositories...`
	2. 搜索框输入`PlantUML`，选择点击`Install`，安装重启后生效

3. 安装go-package-plantuml工具

	```bash
	go get git.oschina.net/jscode/go-package-plantuml
	```
	命令执行完成后会在`$GOPATH/bin`目录生成go-package-plantuml可执行文件

4. GoLand中添加`External Tools`

	- `Program:` 选择`gotouml`执行文件路径
	- `Parameters:` 设置为`---gopath $GOPATH$ --codedir $FileDir$ --outputfile $FileDir$.puml`

5. GoLand中右键点击源码所在的目录选择`External Tools`执行，再打开生成的uml文件，点击`PlantUML`即可查看UML图


- Package dependence

	```bash
	go get github.com/google/godepq
	godepq -from github.com/yougg/pool -ignore golang.org -o dot | dot -Tsvg -o /d/workspace/private/doc/design/sc/sc5.svg
	```

- Call graph

	```bash
	go get -u github.com/TrueFurby/go-callvis
	go-callvis -nostd -group type -focus="" -ignore github.com/astaxie/beego -limit github.com/yougg github.com/yougg/pool | dot -Tsvg -o pool.svg
	```
	```bash
	go get github.com/kisielk/godepgraph
    godepgraph -s -novendor github.com/yougg/pool
	```


---

_**参考链接：**_  

- [开源工具，使用简单的文字描述画UML图](http://plantuml.com/)
- [Visualize call graph of Go libs](https://github.com/TrueFurby/go-callvis/issues/7)
- [GNU Plot](http://www.gnuplot.info/)
- [使用 pprof 和火焰图调试 golang 应用](http://cizixs.com/2017/09/11/profiling-golang-program)
