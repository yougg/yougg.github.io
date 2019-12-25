---
Categories: []
Description: ""
Tags: []
title: "Ubuntu在内网通过配置代理更新源"
date: 2012-10-27T00:00:00+08:00
draft: false
---

在终端首先配置代理:

```bash
export http_proxy="http://用户名:密码@代理IP:代理端口"
```

然后在ubuntu添加apt-get的中国(搜狐)的源:

```bash
sudo echo 'deb http://cn.archive.ubuntu.com/ubuntu/ quantal main multiverse restricted universe' >> /etc/apt/source.list
```

附加: 添加Debian源到Ubuntu中

```bash
sudo echo 'deb http://ftp.cn.debian.org/debian/ stable main contrib non-free' >> /etc/apt/source.list

#执行更新

apt-get update

#报如下错误:
#W: GPG error: http://ftp.cn.debian.org stable Release: The following signatures couldn't be verified because the public key is not available: NO_PUBKEY AED4B06F473041FA NO_PUBKEY 64481591B98321F9 
#这是因为此源不受信任, 下面手动将此源的PUBKEY添加到信任中

#添加的PUBKEY就是上面报错信息中的NO_PUBKEY的后8位
apt-key adv --recv-keys --keyserver keyserver.Ubuntu.com 473041FA 
apt-key adv --recv-keys --keyserver keyserver.Ubuntu.com B98321F9 
```

现在就可以在内网服务器通过统一的代理访问公共源进行 更新与安装软件了.
