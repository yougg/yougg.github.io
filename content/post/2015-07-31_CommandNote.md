---
Categories: ["Document"]
Description: ""
Tags: ["CMD","Shell"]
title: "Command Note"
date: 2015-07-31T00:00:00+08:00
draft: false
---

- 查看进程环境变量

	- Linux

		```bash
		tr \\0 \\n < /proc/${PID}/environ
		
		ps eww -p ${PID}
		```

	- Solaris

		```bash
		pargs -e ${PID}
		```

	- Windows `PowerShell`

		```ps
		(Get-Process -id %PID%).StartInfo.EnvironmentVariables
		```

- 挂载/卸载分区

	- Windows

		```cmd
		:: 查看分区信息
		mountvol /?
		:: 卸载分区
		mountvol D: /D
		:: 挂载分区
		mountvol D: \\?\Volume{e316bd82-2441-11e5-9656-d6d12ffbbef0}
		```

- 修改系统代理

	- Linux

		```bash
		export http_proxy='http://x.proxy.com:8080'
		export https_proxy=${http_proxy}
		```

	- Windows

		```ps1
		Set-ItemProperty -Path 'HKCU:\Software\Microsoft\Windows\CurrentVersion\Internet Settings' -Name ProxyEnable -Value 1
		Set-ItemProperty -Path 'HKCU:\Software\Microsoft\Windows\CurrentVersion\Internet Settings' -Name ProxyServer -Value 'x.proxy.com:8080'
		Get-ItemProperty -Path 'HKCU:\Software\Microsoft\Windows\CurrentVersion\Internet Settings' -Name *Proxy*
		```

- 生成随机复杂密码

	```bash
	# 16位密码:
	< /dev/urandom tr -dc '`~!@#$%^&*()-_=+[]{}\|;:\,./<>?"1-9a-zA-Z' | head -c${1:-16}; echo
	```

- 查看进程网络连接

	```bash
	lsof -i 4 -a -p ${PID}
	lsof -i :${Port}
	netstat -anp | grep ${PID}
	```

- 生成数字序列

	```bash
	seq -f "%03.f" 9 12
	# 009
	# 010
	# 011
	# 012
	```

- Linux强制关机

	```bash
	echo 1 > /proc/sys/kernel/sysrq
	echo b > /proc/sysrq-trigger
	```

- Linux重设登陆失败重试次数
	```bash
	sudo pam_tally2 -u root --reset=0
	```

- Linux设置普通用户定时任务

	```bash
	# 添加username编辑crontab权限
	sudo echo username >> /etc/cron.allow
	# 添加定时任务到username用户的任务列表
	sudo crontab -u username -e
	# 加入定时任务, 例如:
	# * * * * * /path/to/cmd

	# 查看保存在username用户下的任务
	sudo crontab -u username -l

	# 重启定时任务服务加载新的任务运行
	sudo service cron restart

	# 查看定时任务日志
	sudo vi /etc/cron.log
	```

- Linux查看网线连接状态

	```bash
	sudo ethtool eth0
	sudo mii-tool eth0
	```

- SUSE安装git

	```bash
	zypper addrepo http://download.opensuse.org/repositories/devel:/tools:/scm/SLE_11_SP3/devel:tools:scm.repo
	zypper install git-core
	```

- 禁止ssh连接超时

	- 客户端修改`/etc/ssh/ssh_config`或者`~/.ssh/config`

		```bash
		ServerAliveInterval 100
		```

	- 服务端修改`/etc/ssh/sshd_config`

		```bash
		ClientAliveInterval 30
		TCPKeepAlive yes
		ClientAliveCountMax 99999
		```
	  重启ssh服务

		```bash
		sudo service ssh restart
		systemctl restart ssh
		```

- SSH通过http proxy连接

	```bash
	ssh username@host_or_IP -o "ProxyCommand=ncat --proxy ${IP}:${Port} %h %p" -i private_key
	ssh username@host_or_IP -o "ProxyCommand=nc -X connect -x ${IP}:${Port} %h %p"
	```

- ssh远程端口转发

	```bash
	# REMOTE_IP
	# REMOTE_PORT
	# LOCAL_SERVER
	# SERVER_PORT
	# USER
	# HOST
	ssh -R ${REMOTE_IP}:${REMOTE_PORT}:${LOCAL_SERVER}:${SERVER_PORT} ${USER}@${HOST}
	```

- 重置只读变量

	```bash
	readonly HISTSIZE=0

	cat << EOF | sudo gdb
	attach $$
	call unbind_variable("HISTSIZE")
	detach
	EOF
	```

- 编译Webkit版本的LietIDE

	```bash
	sudo apt install qt4-qmake libqt4-dev libqtwebkit-dev
	go get github.com/visualfc/gotools
	cd ${SRC}/liteide/build/
	chmod +x build_linux_qt4_webkit.sh
	./build_linux_qt4_webkit.sh
	cp -rf liteide ~/
	```

- git

	- 切换远程仓库路径

		```bash
		git remote set-url origin https://github.com/user/FORK.git
		```

	- 添加子模块

		```bash
		cd parent_project
		git submodule add https://github.com/username/subproject.git external/src
		git submodule update --init --recursive external/src
		cd external/src
		git checkout -b develop origin/develop
		cd ..
		git add .
		git commit -m 'Add submodule'
		git push
		```

	- 同步子模块

		```bash
		git submodule update --init --force --recursive
		```

	- 删除子模块
		1. Delete the relevant section from the `.gitmodules` file.

			```bash
			git config -f .gitmodules --remove-section submodule.src
			```
		   Stage the `.gitmodules` changes git add `.gitmodules`
		2. Delete the relevant section from `.git/config`.

			```bash
			git config --remove-section submodule.src
			```
		3. Run `git rm --cached path_to_submodule` (no trailing slash).
		   Run `rm -rf .git/modules/path_to_submodule`
		   Commit `git commit -m "Removed submodule <name>"`
		4. Delete the now untracked submodule files

			```bash
			rm -rf path_to_submodule
			```

	- git克隆保存用户/密码

		```bash
		git clone http://username:password@github.com/user/project.git
		```

	- git指定提交时的作者和邮箱

		```bash
		GIT_AUTHOR_NAME="some one" GIT_AUTHOR_EMAIL="someone@email.com" GIT_COMMITTER_NAME="some one" GIT_COMMITTER_EMAIL="someone@email.com" git commit -m 'message'
		GIT_AUTHOR_NAME="yougg" GIT_AUTHOR_EMAIL="joe2qq@gmail.com" GIT_COMMITTER_NAME="yougg" GIT_COMMITTER_EMAIL="joe2qq@gmail.com" git commit -m ''
		git push -f --set-upstream origin master
		```

	- 配置git diff使用vimdiff

		```bash
		git config --global diff.tool vimdiff
		git config --global difftool.prompt false
		git config --global alias.d difftool
		```
	  使用方式

		```bash
		git difftool
		git d
		```
	  如果执行报错`Can't locate Error.pm in @INC ......`  
	  则执行如下方式后再操作

		```bash
		cpan Error.pm
		```
	  或者

		```bash
		sudo perl -MCPAN -e 'install Error'
		```

	- 修改仓库中文件权限

		```bash
		git update-index --chmod=+x /path/to/file
		git ls-tree HEAD /path/to/file
		git add /path/to/file
		git commit -m 'commit message'
		```

	- 一键设置git用户名为域帐户和工号

		```bash
		gituser="$(chcp.com 65001 &> /dev/null;net user //domain $(id -un) | awk -F '[ ][(]' '/Full Name/{print $1}' | awk -F 'Full Name[ ]+' '{print $2}') $(id -un | cut -c 2-)"
		git config --global user.name ${gituser}
		```

- openssl转换密钥格式pkcs8为pkcs1

	```bash
	openssl pkcs8 -in ca0.key -passin pass:xxxx -outform PEM -out ca1.key
	openssl rsa -in ca1.key -out ca2.key -aes256 -passout pass:xxxx
	```

- 查看终端颜色

	```bash
	for i in {0..8}; do
		for j in {0..8}; do
			echo -en "\e[3${i};4${j}m3${i} 4${j}\e[0m"
		done
		echo
	done
	```

- Linux升级/安装openssl编译依赖库

	```bash
	cd /usr/local/src/
	tar xzf openssl-1.1.0f.tar.gz
	cd /usr/local/src/openssl-1.1.0f/

	./config --prefix=/usr/local --openssldir=/usr/local/openssl
	make
	make test
	make install

	export LD_LIBRARY_PATH=/usr/local/lib64
	echo "export LD_LIBRARY_PATH=/usr/local/lib64" > /etc/profile.d/ld_library_path.sh
	chmod a+x /etc/profile.d/ld_library_path.sh

	openssl version

	#reboot

	sudo apt install libssl-dev
	sudo yum install openssl-devel
	```

- Ubuntu systemd-shim error after upgrading fixed

	```bash
	# dpkg-maintscript-helper: error: old-conffile 'debian/systemd-shim/etc/dbus-1/system.d/org.freedesktop.systemd1.conf' is not an absolute path
	mv /var/lib/dpkg/info/systemd-shim.* /tmp/
	dpkg --remove --force-remove-reinstreq systemd-shim
	apt purge systemd-shim
	apt clean; apt autoclean; apt autoremove; apt update; apt upgrade -y; apt dist-upgrade -y
	apt -f install systemd
	reboot -f
	```

- Linux非root用户启动服务绑定到小于1024的端口[参考](http://rk700.github.io/2016/10/26/linux-capabilities/)

    ```bash
    # 使用root权限执行下面命令
    setcap cap_net_bind_service=+eip /path/to/file
    getcap /path/to/file
    ```

- MingW环境变量与快捷命令别名设置 `/etc/profile`

	```bash
	export PS1='\e[35;40m[\w]\e[0m\$ '

	alias clear=clsb
	alias reset=clsb
	alias ls='/bin/ls --color=auto'
	alias l='ls -AlF'
	alias vi=vim
	```

- Windows从命令行下载文件[参考](https://superuser.com/questions/59465/is-it-possible-to-download-using-the-windows-command-line)

- 使用inode删除特殊名称的文件，如：`''$'\270\016'` `'x'$'\016'` [参考](http://blog.51cto.com/lovvvve/871747) [find](https://stackoverflow.com/questions/3925337/find-without-recursion)

	```bash
	ls -i .
	find . -maxdepth 1 -inum 2988 -exec rm -i {} \;
	```

- SLES 11 SP4设置自启动服务

	```bash
	cat /etc/init.d/rc.local
	```

	```
	#! /bin/sh
	### BEGIN INIT INFO
	# Provides:          rc.local
	# Required-Start:    $all
	# Required-Stop:
	# Default-Start:     2 3 4 5
	# Default-Stop:
	# Short-Description: Run /etc/rc.local if it exist
	### END INIT INFO

	[ -z "`pidof vatch`" ] && (cd /usr/local/bin; nohup vatch -C 1 &> /dev/null &)
	[ -z "`pidof gohttpserver`" ] && (gohttpserver -r /data/data12/file -a ':80' --theme green --upload --mkdir --cors --title 'Huawei cloud version service' &> /dev/null &)

	exit 0
	```

	```bash
	cat /usr/local/bin/1
	```

	```json
	{
		"Debug": false,
		"Retries": 10,
		"ServeNodes": [
			":8080"
		],
		"ChainNodes": [
			"http://proxy.aaaaa.com:8080",
			"http://proxy.bbbbb.com:8080"
		]
	}
	```

- 查询域帐号信息

	```bash
	curl -ks 'http://athena.huawei.com/rd-search/api/neo4j/queryEntityData?orderIndex=0&start=1&size=10&key=x00123456' -H 'Accept-Language: zh-CN,zh;q=0.9'
	curl -ks 'http://10.21.247.117:12355/Employee/GetEmplouees?text=00332211'
	```

- Golang

	- 单元测试

		- 覆盖率报告

			```bash
			go test -cover -coverprofile=cover.out -covermode=count
			go tool cover -html cover.out -o cover.html
			```

		- 测试包

			```bash
			# 测试当前包
			go test .
			# go test会在GOROOT和GOPATH下查找这个包并执行包的测试
			go test fmt
			# 测试整个包
			go test github.com/yougg/pool/...
			```

	- 时间解析

		```go
		t, err := time.ParseInLocation("2006-01-02 15:04:05.000000000", "2016-01-08 09:10:01.440114278", time.Local)
		t, err = time.Parse("2006-01-02 15:04:05.000000000 -0700", "2016-01-08 09:10:01.440114278 +0800")
		t, err = time.Parse("2006-01-02 15:04:05.000000000 Z0700", "2016-01-08 09:10:01.440114278 +0800")
		```

	- 一行`go generate main.go`命令静态编译Go，假设编译在`github.com/user/project/`目录中文件

		```go
		// 将bash/sh的执行路径加入PATH环境变量中
		// 将下面命令加入main.go的头部
		//go:generate sh -c "GOPATH=`cd ../../../..;pwd` CGO_ENABLED=0 go build -installsuffix netgo -tags netgo -ldflags \"-s -w -extldflags '-static'\" -o $DOLLAR(basename `pwd`)`go env GOEXE` ${GOFILE}"
		```

	- 静态链接  
	  引用了`net`包的程序, 编译时默认会动态链接

		```bash
		CGO_ENABLED=0 go build -installsuffix cgo yourcode.go
		# 首先需要静态链接标准库中的net包
		go install -tags netgo -a -v std
		# 然后再编译自己的程序
		CGO_ENABLED=0 go build yourcode.go
		```
		  使用`gccgo`静态编译引用了`cgo`代码的程序
		```bash
		# static cgo
		go build --ldflags '-extldflags "-static"' yourcode.go
		# gccgo
		go build -compiler gccgo --gccgoflags "-static" yourcode.go
		```

	- 反射访问私有成员

		```go
		package main
	
		import (
			"fmt"
			"reflect"
			"unsafe"
		)
		
		type Tab struct {
			A string
			b int
			c *bool
		}
		
		func (t *Tab) Name(x string) string {
			return x+`{A:"`+t.A+`",b:`+fmt.Sprintf("%d",t.b)+`,c:`+fmt.Sprintf("%t",*t.c)+`}`
		}
		
		func main() {
			var t = &Tab{c:new(bool)}
			fmt.Printf("%#v\n",t)
			val := reflect.ValueOf(t)
			vv := reflect.Indirect(val)
			vv.FieldByName("A").SetString("XXXXXX")
			// panic: reflect: reflect.Value.SetInt using value obtained using unexported field
			// vv.FieldByName("b").SetInt(222)
			b := (*int)(unsafe.Pointer(vv.FieldByName("b").Addr().Pointer()))
			*b = 33333
			// panic: reflect: reflect.Value.SetBool using value obtained using unexported field
			// vv.FieldByName("c").SetBool(true)
			c := (*bool)(unsafe.Pointer(vv.FieldByName("c").Pointer()))
			*c = true
			res := val.MethodByName("Name").Call([]reflect.Value{reflect.ValueOf("&main.Tab")})
			for _, v := range res {
				fmt.Println(v)
			}
		}
		```

- MySQL

	- 连接数据库

		```bash
		mysql -h ${IP} -P ${Port} -D ${database} -u ${user} -p${pwd}
		```

	- 启用用户远程连接权限

		```sql
		grant all privileges on *.* to ${username}@${IP} identified by '${pwd}' with grant option;
		flush privileges;
		```

	- 超级权限

		```sql
		grant super on *.* to 'my_user'@'localhost';
		revoke super on *.* from 'my_user'@'localhost';
		```

- 压缩包加密

	```bash
	zip -rP ${pwd} filename.zip filename
	unzip -P ${pwd} filename.zip
	```

- TC

	- 关闭全屏模式

		```js
		// Ctrl + Shift + k
		window.open('about:addons')
		// 禁用扩展 "R-kisok"
		```

	- 进入自主维护主机命令行

		1. 登陆进入自助维护界面，打开Chrome
		2. Chrome地址栏输入：`file:///C:/temp/`回车
		3. 点击一个`.csv`文件下载
		4. 点击下载完成的csv文件，如果自动从Excel中打开则继续，否则退出重新第一步往下操作，直到看到Excel打开csv文件
		5. Excel中点击`开发工具`->`Visual Basic`进入VB工程界面
		6. 右键点击VBAProject列表中的`ThisWorkbook`->`插入`->`模块`
		7. 在模块编辑中填入下面代码：

			```vbs
			Sub x()
			Shell ("cmd /c start powershell")
			End Sub
			```

		8. 按`F5`运行代码，等待PowerShell命令窗口弹出即可

			```cmd
			mstsc
			taskmgr
			notepad++
			```

- HTTPS SSL TLS

	> [TLS with Go](https://ericchiang.github.io/tls/go/https/2015/06/21/go-tls.html)  
	  [https详解](http://luojinping.com/2016/04/17/https%E8%AF%A6%E8%A7%A3/)  
	  [详解https是如何确保安全的](http://www.wxtlife.com/2016/03/27/%E8%AF%A6%E8%A7%A3https%E6%98%AF%E5%A6%82%E4%BD%95%E7%A1%AE%E4%BF%9D%E5%AE%89%E5%85%A8%E7%9A%84%EF%BC%9F/)

- Textual Encodings of PKIX, PKCS, and CMS Structures  
  PEM file format  
  https://tools.ietf.org/html/rfc7468

	|Type         |Label                     |
	|-------------|--------------------------|
	|CMS          | "CMS"                    |
	|DHPARAMS     | "DH PARAMETERS"          |
	|DSA          | "DSA PRIVATE KEY"        |
	|DSAPARAMS    | "DSA PARAMETERS"         |
	|DSA_PUBLIC   | "DSA PUBLIC KEY"         |
	|ECDSA_PUBLIC | "ECDSA PUBLIC KEY"       |
	|ECPARAMETERS | "EC PARAMETERS"          |
	|ECPRIVATEKEY | "EC PRIVATE KEY"         |
	|EVP_PKEY     | "ANY PRIVATE KEY"        |
	|PARAMETERS   | "PARAMETERS"             |
	|PKCS7        | "PKCS7"                  |
	|PKCS7_SIGNED | "PKCS #7 SIGNED DATA"    |
	|PKCS8        | "ENCRYPTED PRIVATE KEY"  |
	|PKCS8INF     | "PRIVATE KEY"            |
	|PUBLIC       | "PUBLIC KEY"             |
	|RSA          | "RSA PRIVATE KEY"        |
	|RSA_PUBLIC   | "RSA PUBLIC KEY"         |
	|SSL_SESSION  | "SSL SESSION PARAMETERS" |
	|X509         | "CERTIFICATE"            |
	|X509_CRL     | "X509 CRL"               |
	|X509_OLD     | "X509 CERTIFICATE"       |
	|X509_PAIR    | "CERTIFICATE PAIR"       |
	|X509_REQ     | "CERTIFICATE REQUEST"    |
	|X509_REQ_OLD | "NEW CERTIFICATE REQUEST"|
	|X509_TRUSTED | "TRUSTED CERTIFICATE"    |

- GitHub Huawei Organization

	- [Huawei Openlab](https://github.com/huawei-openlab)
	- [Huawei Open Source](https://github.com/openhuawei)
	- [Huawei-PaaS](https://github.com/Huawei-PaaS)
	- [Huawei-OSG](https://github.com/Huawei-OSG)
	- [huawei-cloudfederation](https://github.com/huawei-cloudfederation)
