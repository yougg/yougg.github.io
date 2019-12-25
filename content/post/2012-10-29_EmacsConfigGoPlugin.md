---
Categories: []
Description: ""
Tags: []
title: "Ubuntu 下配置Emacs下的Go语言插件"
date: 2012-10-29T00:00:00+08:00
draft: false
---

首先安装Emacs

```bash
sudo apt-get install emacs
```

然后进入Emacs的插件目录,配置go语言语法插件

```bash
cd /usr/share/emacs/site-lisp
ln -s $GOROOT/misc/emacs/go-mode.el go-mode.el
ln -s $GOROOT/misc/emacs/go-mode-load.el go-mode-load.el
```

安装Emacs的自动补全插件

```bash
sudo apt-get install auto-complete-el
```

再在用户HOME目录下配置.emacs,加入以下内容

```perl
;set for syntax highlight
(add-to-list 'load-path "PATH CONTAINING go-mode-load.el" t)
(require 'go-mode-load)

;set for auto complete
(require 'go-autocomplete)
(require 'auto-complete-config)
```

现在使用emacs打开go源文件就可以自动高亮语法显示和自动补全代码了.

