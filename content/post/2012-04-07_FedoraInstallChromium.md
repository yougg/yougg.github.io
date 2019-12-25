---
Categories: []
Description: ""
Tags: []
title: "Fedora17 安装Chromium"
date: 2012-04-07T00:00:00+08:00
draft: false
---

Fedora官方源中没有包含Chromium中,需要先添加安装源.

- Stable repo(稳定版本):

    > http://repos.fedorapeople.org/repos/spot/chromium-stable/  
      http://repos.fedorapeople.org/repos/spot/chromium-stable/fedora-chromium-stable.repo 

- Unstable repo(非稳定版本,开发测试中的构建版本,是最新版本):

    > http://repos.fedorapeople.org/repos/spot/chromium-unstable/  
      http://repos.fedorapeople.org/repos/spot/chromium-unstable/fedora-chromium.repo 

选择添加上面其中一个源,使用wget命令下载源库文件到 `/etc/yum.repos.d/` 目录下  
如果wget命令不存在(未安装),则需要先安装wget(官方源中就有)

```bash
#安装wget
yum install wget

cd /etc/yum.repos.d

#获取非稳定库
wget http://repos.fedorapeople.org/repos/spot/chromium-unstable/fedora-chromium.repo

#安装Chromium
yum install chromium
```
