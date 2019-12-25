---
Categories: []
Description: ""
Tags: []
title: "Fedora修改默认系统启动等级"
date: 2012-04-07T00:00:00+08:00
draft: false
---

在之前是直接将 `/etc/inittab` 文件中`id:5:initdefault` 数字修改为 `id:3:initdefault` 就可以了.

现在再进入这个文件提示不再使用了.

需要修改 `/etc/systemd/system/default.target` 这个软连接文件.

这个软连接默认指向第5运行等级(GUI):

```bash
/etc/systemd/system/default.target -> /lib/systemd/system/runlevel5.target
```

现在只要将这个软连接指向需要的运行等级就行了. 比如命令行界面多用户模式:

```bash
cd /etc/systemd/system
mv default.target runlevel5.target
ln -s /lib/systemd/system/runlevel3.target default.target
```

这样使默认启动等级的target指向等级为3(CLI)的运行等级target.

```bash
/etc/systemd/system/default.target -> /lib/systemd/system/runlevel3.target
```

然后重启系统就默认进入命令行多用户模式了,不会再启动GDM了.

如果想从GUI模式快速重新启动到命令行模式,可以直接使用命令 `init 3`

下次如果想要改回去,只要把`default.target`改下名称,把`runlevel5.target`再改回`default.target`就行啦.


```bash
mv default.target runlevel3.target
mv runlevel5.target default.target
```
