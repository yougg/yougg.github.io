---
Categories: []
Description: ""
Tags: []
title: "Ubuntu 重启X服务的几种方法"
date: 2013-02-12T00:00:00+08:00
draft: false
---

快捷键: `Ctrl + Alt + Backspace`

如果12.04以后的版本,快捷键默认是没有开启的, 需要在设置中默认开启才能生效.

`System Setting` -> `Keyboard Layout` -> `Options expand "Key Sequence to Kill X Server"` 勾选快捷键.


快捷键: `Alt+PrintScreen+K` 快速重启

使用命令重启X服务(Unity):

```bash
sudo restart lightdm
sudo service lightdm restart
```

使用命令重启X服务(Gnome):

```bash
sudo restart gdm
sudo service gdm restart
```

所有版本通用:

```bash
sudo pkill X
```
