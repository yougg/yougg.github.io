---
Categories: ["Document"]
Description: ""
Tags: ["Eclipse", "Explorer"]
title: "DIY Eclipse打开文件所在位置文件夹"
date: 2010-11-19T00:00:00+08:00
draft: false
---

MyEclipse里面带了类似的一个插件，点一下就可以打开当前编辑文件所在的文件夹，这样就可以直接到文件夹里面去做操作了。

MyEclipse 里面的插件名叫：
`Desktop toolbar`


没有找到,不过可以自己动手DIY一个,呵呵:  
`Run`-->`External Tools`-->`Open External Tools Dialog...`  
`new` 一个 `program`  
`location` 里面填 ：`C:\WINDOWS\explorer.exe`  
`Arguments` 里面填: `${container_loc}`
