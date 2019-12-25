---
Categories: ["Document"]
Description: ""
Tags: ["rkt","Container","Docker"]
title: "rkt容器简单试用"
date: 2018-07-27T16:41:47Z
draft: false
---

1. 设置代理

	```bash
	export http_proxy='http://127.0.0.1:8080'
	export {https,ftp,rsync,all}_proxy=$http_proxy
	```

2. 下载安装rkt

	```bash
	cd ~/Download/
	wget --no-check-certificate https://github.com/rkt/rkt/releases/download/v1.30.0/rkt-1.30.0-1.x86_64.rpm
	sudo rpm -ivh rkt-1.30.0-1.x86_64.rpm
	```

3. 下载安装镜像

	- 切换到root重新, 重新设置一遍http proxy, 然后再下载镜像

	    ```bash
	    sudo su -
		export http_proxy='http://127.0.0.1:8080'
		export {https,ftp,rsync,all}_proxy=$http_proxy
	    #sudo rkt --insecure-options=image --debug fetch docker://charliev5/alpine-desktop
		rkt fetch quay.io/coreos/alpine-sh
		```

	- 查看镜像、启动容器

		```bash
		sudo rkt image list
		sudo rkt run --interactive quay.io/coreos/alpine-sh -- /bin/sh
		```

	- 容器安装软件启动ssh服务

		```bash
		# https://wiki.alpinelinux.org/wiki/Setting_up_a_ssh-server
		apk update
		apk add openssh
		apk add vim
		rc-update add sshd
		rc-status
		/etc/init.d/sshd start
		```

4. 启动容器

> TODO expose port and mount volume


_**参考链接：**_  

- [CoreOS 容器 Rkt 简单介绍](https://cloud.tencent.com/developer/article/1047251)
- [rkt初探](http://licyhust.com/%E5%AE%B9%E5%99%A8%E6%8A%80%E6%9C%AF/2016/11/28/rkt/)
- [Apline Linux 操作系统](https://yeasy.gitbooks.io/docker_practice/cases/os/alpine.html)
- [rkt Documentation](https://github.com/rkt/rkt/tree/master/Documentation)
- [三年后，我们从 Docker 转到了 RKT](http://dockone.io/article/1698)