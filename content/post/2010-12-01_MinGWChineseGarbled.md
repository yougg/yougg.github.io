---
Categories: ["Document"]
Description: ""
Tags: ["Mingw"]
title: "MinGW 中文显示乱码 的解决办法"
date: 2010-12-01T11:50:40+08:00
draft: false
---

MinGW msys  Cygwin的精简功能版  
在MinGW中想切换到中文目录,却提示不支持.中文全部显示?????  
这个是解决办法:  
给ls命令加上参数 `--show-control-chars`

```bash
ls --show-control-chars
```

如此则可以正常显示中文

再加上参数 `--color=tty`

```bash
ls --color=tty
```
此参数能使显示的内容按文件类型分类高亮

安装比Windows命令行更好用的终端:  

```bash
mingw-get install msys-mintty-bin  
mingw-get install msys-rxvt-bin  
```

创建终端快捷方式:

```cmd
C:\MinGW\msys\1.0\bin\mintty.exe /bin/bash -l
C:\MinGW\msys\1.0\bin\rxvt.exe /bin/bash -l
```

---

_**参考链接：**_  

- [Windows下的新玩具 MSYS](http://zealotds.iteye.com/blog/1052310)
