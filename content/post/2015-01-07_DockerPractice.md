---
Categories: ["Document"]
Description: ""
Tags: ["Docker"]
title: "基于Docker的轻量级XXOO系统环境搭建实践"
date: 2015-01-07T00:00:00+08:00
draft: false
---

<!-- <center>作者 [cxy](mailto:joecc@126.com)　　发布于 2015年1月</center> -->
<!-- toc -->
-----

# Docker简介

> 　　[Docker](https://www.docker.com) 是一个开源的应用容器引擎，让开发者可以打包他们的应用以及依赖包到一个可移植的容器中，然后发布到任何流行的 Linux 机器上，也可以实现虚拟化。容器是完全使用沙箱机制，相互之间不会有任何接口。几乎没有性能开销,可以很容易地在机器和数据中心中运行。最重要的是,他们不依赖于任何语言、框架或包括系统。

> 　　Docker 源于[dotcloud](https://www.dotcloud.com)公司的一个开源项目；随着编写者不断挖掘它的潜力，它迅速变成了一个炙手可热的项目。它由[Go](http://golang.org/)语言编写的，并且只支持 Linux。它基于Linux容器（LxC）来创建一个虚拟环境。  
     注意：虚拟环境 VE ( Virtual Environment ) 和 虚拟机（ VM ）很不一样。  
  　　虚拟机是由虚拟工具或者模拟器（ HyperV 、 VMWare 等）创建的，是一个全镜像的主机源，其中包括操作系统、硬盘调整、网络和虚拟进程。过于臃肿的结构吃掉了大量的硬盘空间同时拖慢了运行和开机速度。  
  　　Docker 则通过自己与操作系统内核相分离的特点很好的解决了这个问题。它会首先创建一个与机器内核相分离的 “等级0” 包裹。任何之后的改变都被保存为 namespace 的快照，并且像虚拟系统那样在单独的容器内运行。“快照”只捕捉“基本模版”的改动，而不是整个机器上面的设置（不像是VM的快照功能）

-----

#### 本文目标

> 现状：  
  XXOO的N节点方案在开发测试中需要使用大量的系统环境进程测试与验证，在当前的的硬件资源中通过Vmware搭建的虚拟机数量非常有限。
项目小组内能够分配使用的系统环境也就十几二十套左右，很难协调到大量的环境进行大型并发功能性能测试验证，采购的服务器硬件资源远远达不到开发与测试的需求。在现有少量节点的分布式系统中很难暴露出一些功能与性能问题。  
  　  
  方案：  
  采用轻量级Linux容器引擎Docker实现虚拟化方案，高效利用硬件资源，轻松解决系统环境紧缺的问题。
  当前使用Vmware搭建的虚拟机，CPU: 8核 2GHz，Mem: 16GB，单机轻松运行100虚拟容器节点，资源消耗还没达到50%  

  本文介绍通过Docker容器引擎实现以下目标：

- 简化系统环境配置
- 快速搭建测试验证环境
- 快速批量部署多套应用
- 轻松解决系统环境紧缺的问题
- 共享基础容器、数据容器、开发工具容器、不同环境的测试容器、构建容器、安装容器等使开发测试生产化

-----

<!--

#### 向导目录

- [主机系统：Ubuntu Server 14.04.1 LTS](#installhost)
- [容器引擎：Docker 1.4.1](#installdocker)
- [虚拟系统：SUSE Linux Enterprise Server 11 SP3](#installvm)
- [搭建XXOO网管系统环境](#installoss)
- [Docker维护配置](#dockermaintenance)
- [延伸阅读](#others)

-----
-->

##### 提示
_将全文先通读一两遍，完全了解整个操作流程后，再按照向导一步一步操作，能购避免很多疏忽的问题，尽可能保证一次性完成。_

_Docker支持直接从官方或第三方在线镜像仓库中直接拉取/推送镜像，但是由于内网代理限制本文将不涉及此部分内容，详细内容可以查看`docker <pull|push>`命令帮助。_

-----

#### <span id="installhost">安装主机系统</span>

> Docker官方支持的系统平台包括：
  [Mac OS X](http://docs.docker.com/installation/mac/),
  [Ubuntu](http://docs.docker.com/installation/ubuntulinux/),
  [Red Hat Enterprise Linux](http://docs.docker.com/installation/rhel/),
  [Oracle Linux](http://docs.docker.com/installation/oracle/),
  [CentOS](http://docs.docker.com/installation/centos/),
  [Debian](http://docs.docker.com/installation/debian/),
  [Gentoo](http://docs.docker.com/installation/gentoolinux/),
  [Google Cloud Platform](http://docs.docker.com/installation/google/),
  [Rackspace Cloud](http://docs.docker.com/installation/rackspace/),
  [Amazon EC2](http://docs.docker.com/installation/amazon/),
  [IBM Softlayer](http://docs.docker.com/installation/softlayer/),
  [Arch Linux](http://docs.docker.com/installation/archlinux/),
  [FrugalWare](http://docs.docker.com/installation/frugalware/),
  [Fedora](http://docs.docker.com/installation/fedora/),
  [CRUX Linux](http://docs.docker.com/installation/cruxlinux/),
  [SUSE](http://docs.docker.com/installation/SUSE/)  
  支持SUSE Linux Enterprise 12及以上版本，需要从[OBS](https://build.opensuse.org/)的[虚拟化仓库](https://build.opensuse.org/project/show/Virtualization)安装docker  
  Docker是基于Ubuntu开发的，本文以官方推荐使用Ubuntu Server操作系统。

1. 准备物理环境, 可以使用真实物理主机, 也可以使用Vmware架设的虚拟主机

2. 获取主机系统安装镜像  
   [[Download] 本地下载](http://localhost:8000/docker/files/ubuntu-14.04.1-server-amd64.iso)  
   [[Download] Ubuntu Server 14.04.1 Release](http://releases.ubuntu.com/14.04.1/ubuntu-14.04.1-server-amd64.iso)  
   [ ](http://cdimage.ubuntu.com/ubuntu-server/trusty/daily/current/trusty-server-amd64.iso)

3. 进入主机控制器(USM、vCenter、实体机等)，挂载镜像文件，启动系统安装

4. 安装流程向导  
   [[中文向导]](http://localhost:8000/docker/static/UbuntuServer14.04LTSGuide_zh.html)  
   [[English Guide]](http://localhost:8000/docker/static/UbuntuServer14.04LTSInstallGuide.html)

5. 安装完成后从光驱中移除挂载的镜像后再重启, 使用安装时设置的用户登录系统

6. 初始化root密码

	```bash
	sudo passwd root
	
	# 然后切换到root用户
	su - root
	```

7. 配置网络

	1. 配置root用户ssh登录：

		> 修改/etc/ssh/sshd_config文件的如下选项

		```
		PermitRootLogin yes
		```
	
		> 修改保存后重启sshd服务  
	
		```bash
		service ssh restart
		```

	2. 编辑 /etc/network/interfaces 文件, 修改eth0配置如下：

		```
		auto eth0
		iface eth0 inet static
		address 10.1.2.3
		netmask 255.255.254.0
		gateway 10.1.2.1
		```

		> 配置完成后重启网络
	
		```bash
		service networking restart
		# 或者
		/etc/init.d/networking restart
		# 或者
		ifdown eth0
		ifup eth0
		# 如果网络重启失败,则直接重启系统
		shutdown -r now
		```

	3. 配置默认路由：

		```bash
		# 首先ping自己的IP,确认当前网络是否连通
		ping 10.1.2.2

		# 如果不能连通,查看当前路由与其他同网段系统路由是否相同
		route -n

		# 如果显示路由不同,则先删除错误的路由
		route del default gw 10.1.2.1
		route del -net 10.1.2.0 netmask 255.255.255.0

		# 再添加正确的默认路由
		route add -net 10.1.2.0 netmask 255.255.254.0 dev eth0
		route add default gw 10.1.2.1
		# 再次ping自己的IP,确认连通后可以远程终端工具登录
		```

-----

#### <span id="installdocker">安装Docker</span>

1. 通过Putty、XShell、SecureCRT等终端工具登录到主机系统的root用户

2. 安装docker软件
	- 使用docker官方最新版本 `docker 1.4.1`
		1. 获取安装包  
		   [[Download] docker1.4.1](http://localhost:8000/docker/files/docker1.4.1.tar.gz)  
		   [[Download] docker-latest](https://get.docker.com/builds/Linux/x86_64/docker-latest)
		2. 下载解压安装

			```bash
			wget http://localhost:8000/docker/files/docker1.4.1.tar.gz
			tar xzf docker1.4.1.tar.gz
			cd docker1.4.1/
			./install.sh
			```
	- 使用Ubuntu官方源安装 ~~`docker 1.0.1`~~
		1. 配置主机系统安装源

			```bash
			# 备份当前安装源文件
			cd /etc/apt
			cp sources.list sources.list_bak

			# 添加Ubuntu中国源
			cat > sources.list <<-EOF
			deb http://cn.archive.ubuntu.com/ubuntu/ trusty main restricted universe multiverse
			deb http://cn.archive.ubuntu.com/ubuntu/ trusty-security main restricted universe multiverse
			deb http://cn.archive.ubuntu.com/ubuntu/ trusty-updates main restricted universe multiverse
			deb http://cn.archive.ubuntu.com/ubuntu/ trusty-proposed main restricted universe multiverse
			deb http://cn.archive.ubuntu.com/ubuntu/ trusty-backports main restricted universe multiverse
			EOF
			```
		2. 设置主机系统内网代理

			```bash
			# 将对应的用户名与密码替换为自己的代理账号和密码
			# 因为没有配置DNS，所以不能直接解析域名的IP地址
			export http_proxy='http://用户名:密码@12.34.56.78:8080'
			export https_proxy='http://用户名:密码@12.34.56.78:8080'
			```
		3. 安装docker

			```bash
			# 更新本地软件库
			apt-get update && apt-get upgrade -y
			
			# 从官方源安装docker
			apt-get install -y docker.io
			```

3. 启动docker服务

	```bash
	# 使用docker官方版本安装服务名称是 docker
	# 使用Ubuntu官方源安装服务名称是 docker.io
	service docker start

	# 执行docker命令, 显示docker帮助信息确认安装成功
	docker
	```

-----

#### <span id="installvm">安装配置虚拟系统</span>

1. 获取虚拟系统需要的镜像文件到主机系统的目录中
	- 使用`rtools`仓库中的Suse原厂完整系统镜像

		> 原厂镜像分为两碟,碟1为主系统安装包, 大小: 3.13GB, 碟2是扩展软件包, 大小: 4.94GB  
		[[Download] SLES-11-SP3-DVD-x86_64-GM-DVD1](SLES-11-SP3-DVD-x86_64-GM-DVD1.iso)  
		[[Download] SLES-11-SP3-DVD-x86_64-GM-DVD2](SLES-11-SP3-DVD-x86_64-GM-DVD2.iso)
	- 使用基于OSS发布的安装盘简化版镜像

		> 此镜像是基于Suse11SP3 x86_64标准版本制作, 大小: 541MB  
		  裁剪了X Window图形界面以及部分无用软件包  
		  默认root密码为：`XXOO`  
		  [[Download] suse11sp3_simplify_oss](suse11sp3_simplify_oss.tar.gz)
	- 使用docker官方仓库提供的精简版镜像
	
		> 此镜像只提供了最基础的系统环境, 大小: 58MB  
		[[Download] sles11sp3_min](sles11sp3_min.tar.gz)

2. 导入镜像到docker本地仓库中
	- 导入基于容器`container`导出的镜像或者原厂光盘ISO镜像

		```bash
		# 本文使用OSS简化版镜像, 其他镜像操作方式类似
		wget http://localhost:8000/docker/files/suse11sp3_simplify_oss.tar.gz

		cat suse11sp3_simplify_oss.tar.gz | docker import - suse11sp3:simplify_oss
		```
	- 载入基于docker仓库`image`保存的镜像

		```bash
		docker load -i xxx.tar
		```

3. 查看docker本地仓库中的镜像

	```bash
	docker images
	#REPOSITORY  TAG           IMAGE ID      CREATED     VIRTUAL SIZE
	#suse11sp3   simplify_oss  14017be7faaa  1 days ago  1.621 GB

	# 查看所有镜像
	docker images -a
	```

4. 使用导入的镜像启动测试一个docker容器

	```bash
	# 查看docker启动容器帮助命令
	docker help run
	
	# 启动一个测试容器
	docker run -it suse11sp3:simplify_oss /bin/bash
	```

	> 此时启动的容器就是一个完整的suse11sp3系统环境  
	  可以像普通系统一样执行网管安装等操作  
	  容器默认的网络配置可以访问主机相同的网络  
	  但是外部不能连接容器的网络, 此时只有主机可以连接容器

	```bash
	# 退出容器的虚拟系统, docker将会自动停止此容器
	exit
	```

5. 查看/操作容器状态

	```bash
	# 查看所有容器
	docker ps -a
	#CONTAINER ID  IMAGE                   COMMAND      CREATED     STATUS                 PORTS  NAMES
	#704aa21b7879  suse11sp3:simplify_oss  "/bin/bash"  1 days ago  Exited (0) 1 days ago         asdfa

	# 可以看到刚才退出的容器状态是Exited
	# 再次启动容器
	docker start 704aa21b7879

	# 再连接到容器的标准IO
	docker attach 704aa21b7879

	# 或者启动容器时直接连接到容器的标准IO
	docker start -ai 704aa21b7879

	# 暂停容器中所有进程
	docker pause 704aa21b7879

	# 还原暂停的容器
	docker unpause 704aa21b7879

	# 重新启动容器
	docker restart 704aa21b7879

	# 停止容器内进程并等待退出
	docker stop 704aa21b7879

	# 直接结束容器内进程退出
	docker kill 704aa21b7879

	# 删除容器
	docker rm 704aa21b7879
	```

-----

#### <span id="installoss">搭建XXOO工程监控网管环境</span>

> 说明：  
  第1,2步配置管理/业务节点模板，主要是为了建立业务节点对管理节点的root用户ssh信任关系模板，以及数据库初始化对共享内存的依赖。如果不需要ssh信任则可以直接使用基础镜像启动容器使用。  
  1,2两步主要作为自定义镜像模板操作步骤的参考。

1. 配置管理节点镜像模板

	1. 基于基础镜像启动一个带有`sshd`服务的后台运行的容器

		```bash
		# 启动一个后台运行的容器,打印出容器的ID
		docker run -d --privileged suse11sp3:simplify_oss /usr/sbin/sshd -D
		```

	2. 登录容器环境

		```bash
		# 首先获取运行容器的IP地址
		docker inspect -f '{{.NetworkSettings.IPAddress}}' ${容器ID}
		# 172.17.0.3

		# ssh登录容器, 默认root密码： XXOO
		ssh root@172.17.0.3
		```

	3. 创建管理节点root用户的ssh密钥

		```bash
		# 创建密钥, 使用默认路径与空密码
		ssh-keygen -q -t rsa -P '' -f /root/.ssh/id_rsa
		
		# 添加信任key
		cd /root/.ssh/
		cat id_rsa.pub > authorized_keys
		> known_hosts
		```

	4. 设置共享内存

		```bash
		# 默认共享内存大小是32MB, 会导致GaussDB等数据库初始化失败, 现在修改为实际物理内存的90%
		free -bo | awk '/Mem:/{printf("\nkernel.shmmax = %d\n", $2 * 0.9)}' >> /etc/sysctl.conf
		sysctl -p
		```

	5. 将容器的修改内容保存到镜像仓库中作为管理模板使用

		```bash
		# 提示：先从容器中退出，回到主机进行操作
		exit

		# -a Author, -m Commit message
		docker commit -a yougg -m 'init manager node template' ${容器ID} suse11sp3:mgrnode
		```

2. 配置业务节点镜像模板

	1. 再次ssh登录上一步启动的容器环境

		```bash
		ssh root@172.17.0.3
		```

	2. 创建业务节点的动态生成ssh密钥及启动sshd服务的脚本

		```bash
		# 先删除之前的管理模板的ssh密钥
		cd /root/.ssh/
		rm -f id_rsa*

		# 创建脚本
		cat > init.sh <<-EOF
		#!/bin/bash
		ssh-keygen -q -t rsa -P '' -f /root/.ssh/id_rsa
		/usr/sbin/sshd -D
		EOF

		# 添加脚本执行权限
		chmod +x init.sh

		# 然后退出ssh连接
		exit
		```

	3. 将容器的修改内容保存到镜像仓库中作为业务模板使用

		```bash
		docker commit -a yougg -m 'init app node template' ${容器ID} suse11sp3:appnode
		```

3. 基于镜像模板启动容器

	1. 清理无用容器释放资源

		```bash
		# 删除所有容器
		docker rm -f `docker ps -qa`
		```

		> 注意：  
		  如果有其他需要继续使用的容器运行, 请勿使用此方式  
		  请自行过滤可删除的容器执行操作

	2. 重启docker服务，重新分配IP地址

		```bash
		# 重启docker服务后，基于docker0网桥的内网IP会从头开始重新分配
		service docker restart
		```

	2. 查看当前已有的本地镜像仓库

		```bash
		docker images
		# REPOSITORY  TAG           IMAGE ID      CREATED        VIRTUAL SIZE
		# suse11sp3   appnode       d821b2e59bd0  1 minutes ago  1.621 GB
		# suse11sp3   mgrnode       5494a75f213c  2 minutes ago  1.621 GB
		# suse11sp3   simplify_oss  14017be7faaa  1 days ago     1.621 GB
		```

	3. 启动管理节点容器

		```bash
		docker run -d --privileged -h node0 --name node0 -p={22000:22,23456:23456} suse11sp3:mgrnode /usr/sbin/sshd -D
		```

		> 参数说明：  
		  后台运行，特权模式  
		  容器系统主机名: node0，容器名: node0  
		  映射容器sshd服务端口22到主机22000，23456到23456，外部网络可以通过主机映射端口访问容器服务  
		  使用 ip:port:port 方式映射容器端口到主机指定IP的端口，默认无IP就等于 0.0.0.0:port:port

	4. 批量启动业务节点容器  
	   需要根据主机系统的资源性能配置, 逐步增加容器的数量  
	   本文举例启动10容器虚拟业务节点

		```bash
		# 容器虚拟节点名从node01到node10
		# 映射各业务节点sshd服务的22端口到主机系统的 22001~22010
		# 如果节点上需要映射多个端口, 可参考上一步
		# 如果不需要外部对业务节点的ssh连接，可以去除-p端口映射选项
		for i in {101..110}; do
			n="node${i:1}"
			docker run -d --privileged -h ${n} --name ${n} -p 220${i:1}:22 suse11sp3:appnode /root/.ssh/init.sh
		done

		# 已启动的容器系统环境列表
		docker ps
		# CONTAINER ID  IMAGE              COMMAND               CREATED        STATUS        PORTS                 NAMES
		# 36261553c6d4  suse11sp3:appnode  "/root/.ssh/init.sh"  4 seconds ago  Up 4 seconds  0.0.0.0:22010->22/tcp  node10
		# 5467d591761f  suse11sp3:appnode  "/root/.ssh/init.sh"  5 seconds ago  Up 4 seconds  0.0.0.0:22009->22/tcp  node09
		# 76ff92d33f69  suse11sp3:appnode  "/root/.ssh/init.sh"  5 seconds ago  Up 5 seconds  0.0.0.0:22008->22/tcp  node08
		# 6418274f2996  suse11sp3:appnode  "/root/.ssh/init.sh"  5 seconds ago  Up 5 seconds  0.0.0.0:22007->22/tcp  node07
		# 5777164bba26  suse11sp3:appnode  "/root/.ssh/init.sh"  5 seconds ago  Up 5 seconds  0.0.0.0:22006->22/tcp  node06
		# ad8297f926e4  suse11sp3:appnode  "/root/.ssh/init.sh"  6 seconds ago  Up 5 seconds  0.0.0.0:22005->22/tcp  node05
		# 633276f35435  suse11sp3:appnode  "/root/.ssh/init.sh"  6 seconds ago  Up 6 seconds  0.0.0.0:22004->22/tcp  node04
		# 383ef72d65e3  suse11sp3:appnode  "/root/.ssh/init.sh"  6 seconds ago  Up 6 seconds  0.0.0.0:22003->22/tcp  node03
		# 85f519094b9c  suse11sp3:appnode  "/root/.ssh/init.sh"  6 seconds ago  Up 6 seconds  0.0.0.0:22002->22/tcp  node02
		# 4ad1fd1fd69e  suse11sp3:appnode  "/root/.ssh/init.sh"  7 seconds ago  Up 6 seconds  0.0.0.0:22001->22/tcp  node01
		# 6bb88cbfcd4d  suse11sp3:mgrnode  "/usr/sbin/sshd -D"   1 minutes ago  Up 1 minutes  0.0.0.0:22000->22/tcp, 0.0.0.0:23456->23456/tcp  node0
		```

4. 基于容器搭建XXOO工程监控网管环境

	1. 获取所有容器的主机名与IP地址

		```bash
		docker inspect -f '{{.Config.Hostname}} {{.NetworkSettings.IPAddress}}' `docker ps -qa` | sort
		# node0  172.17.0.13
		# node01 172.17.0.14
		# node02 172.17.0.15
		# node03 172.17.0.16
		# node04 172.17.0.17
		# node05 172.17.0.18
		# node06 172.17.0.19
		# node07 172.17.0.20
		# node08 172.17.0.21
		# node09 172.17.0.22
		# node10 172.17.0.23
		```

	2. 通过Putty、XShell、SecureCRT等终端工具登录到管理容器虚拟节点

		```bash
		# 使用主机系统IP以及映射到管理容器sshd服务的端口
		ssh -p 22000 root@10.10.11.23
		# 注意: 不同终端工具ssh命令登录参数格式可能不一致,请参考具体的命令或终端工具界面输入
		# 如XShell:
		ssh root@10.10.11.23 22000
		```

	3. 按照普通系统方式安装网管环境

		```bash
		# 创建并进入 /opt/software 目录
		mkdir -p /opt/software
		cd /opt/software
		# 获取业务安装包
		wget -crL -np -nH http://localhost:8000/2015-1-15/
		# 解压执行安装等操作本文省略...
		```

	4. 使用主机IP地址及映射的管理面端口登录

		> 客户系统使用浏览器访问：https://10.10.11.23:23456/  
		  登录系统后使用上面第一步的节点名称和IP地址添加节点安装应用

	5. **主机**中查看容器资源的使用

		```bash
		# 容器中进程信息
		docker top ${容器ID}

		# 主机中docker容器的进程树
		pstree `pidof docker`

		# 查看资源占用实时监控
		top
		```

	6. 资源性能极限尝试  
	   本文测试使用的主机环境是使用Vmware搭建的虚拟机  
	   CPU: 8核 2GHz，Mem: 16GB  
	   单机运行100虚拟容器节点 **get√**  
	   ![100nodes](http://localhost:8000/docker/static/100nodes.png)

		> `top`性能监控：  
		top - 18:54:27 up 11 days,  1:22,  4 users,  load average: 9.20, 15.15, 18.42  
		Tasks: 780 total,   3 running, 777 sleeping,   0 stopped,   0 zombie  
		%Cpu(s): 55.4 us, 20.4 sy,  0.0 ni, 22.7 id,  0.2 wa,  1.3 hi,  0.0 si,  0.0 st  
		KiB Mem:  16433824 total, 16187924 used,   245900 free,    55844 buffers  
		KiB Swap: 16775164 total,   671504 used, 16103660 free.  2388892 cached Mem  
		　  
		  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND  
		 7638 dbuser      20   0 2611928 723504 249904 S  91.2  4.4  11:40.05 Postmaster  
		 7816 ossadm      20   0  936952 394532   6216 S  87.3  2.4  12:55.25 java  
		 8205 ossadm      20   0   90496  30580   2088 R  25.6  0.2   2:19.73 nginx  
		 8047 ossadm      20   0  953840 160512  46164 S  19.7  1.0   9:35.73 python  
		 8206 ossadm      20   0   90260  33828   2080 R  18.0  0.2   2:04.38 nginx  
		 ......  
		 ......

-----

#### <span id="dockermaintenance">维护配置</span>

- 查看docker命令列表及命令帮助

	```bash
	# 列出docker的所有子命令及功能说明
	docker [help]

	# 查看子命令的帮助信息
	docker help <cmd>
	# 例如:
	docker help run
	```

	> `docker run`选项参数说明:
	<table>
		<tr style="text-align:center; font-weight:bold;">
			<td>短格式</td><td>长格式</td><td>功能</td><td>描述</td>
		</tr>
		<tr>
			<td>-d</td><td>--detach=false</td><td>后台运行容器</td><td>默认无此选项时前台运行</td>
		</tr>
		<tr>
			<td></td><td>--privileged=false</td><td>特权模式</td><td>容器访问所有主机设备的权限</td>
		</tr>
		<tr>
			<td>-h</td><td>--hostname=""</td><td>容器系统主机名</td><td>自动设置容器内系统的主机名</td>
		</tr>
		<tr>
			<td></td><td>--name=""</td><td>容器名</td><td>不设置时docker会随机生成容器名</td>
		</tr>
		<tr>
			<td>-p</td><td>--publish=[ ]</td><td>映射端口</td><td>将容器内的端口映射到主机系统中从而使外部网络能够访问</td>
		</tr>
		<tr>
			<td>-i</td><td>--interactive=false</td><td>开启标准输入</td><td>保持标准输入打开即使没有连接</td>
		</tr>
		<tr>
			<td>-t</td><td>--tty=false</td><td>分配虚拟终端</td><td>与-i一起使用连接到容器的虚拟终端</td>
		</tr>
	</table>

- 从主机中查看启动的容器

	```bash
	# 查看运行中的容器，已停止/退出的容器不会显示
	docker ps
	
	# 查看所有的容器
	docker ps -a
	```

- 删除容器

	```bash
	# 删除单个容器, -f Force即使容器在运行也强制删除
	docker rm -f ${容器ID}
	
	# 批量删除所有容器
	docker rm -f `docker ps -qa`
	# 或者循环确认删除
	for ID in `docker ps -qa`; do
		echo -n "delete container: "
		docker rm -f $ID
	done
	```

- 删除镜像

	```bash
	# 删除单个镜像, -f Force强制删除
	docker rmi -f ${镜像ID}
	
	# 批量删除所有镜像
	docker rmi -f `docker images -qa`
	# 或者循环确认删除
	for ID in `docker images -qa`; do
		echo -n "delete image: "
		docker rmi -f $ID
	done
	```

- 从主机获取镜像/容器的信息

	```bash
	# 查看完整的镜像/容器配置
	docker inspect ${镜像/容器ID}
	# 实际上是读取下面两个配置文件：
	# /var/lib/docker/graph/${镜像ID}/json
	# /var/lib/docker/containers/${容器ID}/config.json
	
	# inspect -f 读取配置信息中指定的属性值	
	# 查看容器启动初始化的IP地址
	docker inspect -f '{{.NetworkSettings.IPAddress}}' ${容器ID}
	```

- 提交保存容器中的修改内容到镜像仓库中

	```bash
	docker commit -a yougg -m 'simplify the base oss image' ${容器ID} suse11sp3:simplify_oss
	```

- 保存/导出镜像
	- 基于容器导出镜像
	
		```bash
		docker export ${容器ID} > xxx.tar

		# 导出时压缩镜像包
		docker export ${容器ID} | gzip > xxx.tar.gz
		```

	- 基于镜像仓库保存镜像
	
		```bash
		docker save -o xxx.tar ${镜像ID}

		# 保存时压缩镜像包
		docker save ${镜像ID} | gzip > xxx.tar.gz
		```

- docker服务 启动/停止/状态/重启  
  docker服务重启后，原有运行的容器不会自动拉起，需要手工启动`docker start ${容器ID}`  
  新启动容器的IP地址会从头开始分配

	```bash
	service docker <start|stop|status|restart>
	```

- **主机**中安装桌面环境及远程管理

	```bash
	apt-get install -y xorg lxde tightvncserver

	# 开启vnc服务端口, 第一次启动时会要求设置当前用户的vnc登录密码
	# 建议使用普通系统用户启用vnc服务
	vncserver :1 -geometry 1440x1000
	# 结束vnc服务端口
	vncserver -kill :1
	```

	> 客户系统使用vnc客户端(VNC Viewer，tightvnc等)连接主机系统的远程桌面

- 安装Chrome浏览器  
  [[Download] Google Chrome for Ubuntu](http://localhost:8000/docker/files/google-chrome-stable_current_amd64.deb)

	```bash
	# 首先安装Chrome依赖库
	apt-get install -y libnspr4 libnss3 libpango1.0-0 libxss1 libappindicator1 xdg-utils libdbusmenu-glib4 libdbusmenu-gtk4 libindicator7 libnss3-nssdb libpangox-1.0-0
	
	# 然后使用Chrome安装包安装
	dpkg -i google-chrome-stable_current_amd64.deb
	```

<!--
- 设置容器公开网络

	```bash
	# 安装brctl网桥管理工具
	apt-get install -y bridge-utils
	# 查看当前系统中的网桥
	brctl show
	# 添加一个新网桥
	brctl addbr br0
	# 为新网桥配置一个未被使用的IP地址
	ip addr add 10.1.2.181/24 dev br0
	# 启用网桥网络
	ip link set dev br0 up
	# 查看当前系统的所有网络设备信息
	ifconfig -a
	```
-->

- Docker容器网络拓展
	- Weave

		> [Weave](https://github.com/zettio/weave)是由Zett.io公司开发的，它能够创建一个虚拟网络来连接部署在多台主机上的Docker容器。  
		  通过Weave所有的容器就像被接入了同一个网络交换机，那些使用网络的应用程序不必去配置端口映射和链接等信息。外部设备能够访问Weave网络上的应用程序容器所提供的服务，同时已有的内部系统也能够暴露到应用程序容器上。  
		  Weave能够穿透防火墙并运行在部分连接的网络上。  
		  另外，Weave的通信支持加密，所以用户可以从一个不受信任的网络连接到主机。

	- Pipework

		> [Pipework](https://github.com/jpetazzo/pipework)是由Docker的工程师Jérôme Petazzoni开发的一个Docker网络配置工具，由200多行shell实现，方便易用。
		  通过封装Linux上的ip、brctl等命令，简化了在复杂场景下对容器连接的操作命令，为复杂的网络拓扑配置提供了一个强有力的工具。

	- [Docker网络详解及pipework源码解读与实践](http://www.infoq.com/cn/articles/docker-network-and-pipework-open-source-explanation-practice)

-----

#### <span id="others">延伸阅读</span>

- [《Docker 技术入门与实战》](http://localhost:8000/docker/files/docker_practice.pdf)

	> 此书既适用于具备基础 Linux 知识的 Docker 初学者，也希望可供理解原理和实现的高级用户参考。同时，书中给出的实践案例，可供在进行实际部署时借鉴。前六章为基础内容，供用户理解 Docker 的基本概念和操作；7 ~ 9 章介绍一些高级操作；第 10 章给出典型的应用场景和实践案例；11 ~ 13 章介绍关于 Docker 实现的相关技术。14 ~ 章介绍相关的一些开源项目。

- Docker 资料/社区

	> [CSDN Docker 资料集萃](http://special.csdncms.csdn.net/BeDocker/)  
	[深入浅出Docker](http://www.infoq.com/cn/dockers)  
	[docker中文教程](http://www.widuu.com/docker/index.html)  
	[docker.cn](https://docker.cn/)    
	[Dockerpool](http://dockerpool.com/)

- Docker Web 管理工具
	- Shipyard

		> [Shipyard](https://github.com/shipyard/shipyard) 是一个基于 Web 的 Docker 管理工具，支持多 host，可以把多个 Docker host 上的 containers 统一管理；可以查看 images，甚至 build images；并提供 RESTful API 等等。 Shipyard 要管理和控制 Docker host 的话需要先修改 Docker host 上的默认配置使其支持远程管理。

	- DockerUI

		> [DockerUI](https://github.com/crosbymichael/dockerui) 是一个Web界面用于与 Remote API 相交。其目的是提供一个纯客户端实现，让你能够很轻松的连接和管理docker。

- CoreOS & Rocket

	> 　　[CoreOS](https://coreos.com)是一个基于Linux 内核的轻量级操作系统，为了计算机集群的基础设施建设而生，专注于自动化，轻松部署，安全，可靠，规模化。作为一个操作系统，CoreOS 提供了在应用容器内部署应用所需要的基础功能环境以及一系列用于服务发现和配置共享的内建工具。  
	> 　　[Rocket](https://github.com/coreos/rocket) 是一款容器引擎，和 Docker 类似，帮助开发者打包应用和依赖包到可移植容器中，简化搭环境等部署工作。CoreOS 的 CEO Alex Polvi 在官方博文里介绍道，Rocket 和 Docker 不同的地方在于，Rocket 没有 Docker 那些为企业用户提供的“友好功能”，比如云服务加速工具、集群系统等。反过来说，Rocket 想做的，是一个更纯粹的业界标准。

- Windows的容器引擎
	- Spoon

		> [官方文档](https://spoon.net)，[简单使用向导](http://www.widuu.com/archives/12/1136.html)  
		　　Spoon是windows上的容器解决方案，和docker一样，有spoon.net/hub共享镜像和容器，只支持windows系统，它是自建的虚拟化技术，并不依靠windows的内部虚拟化技术，对操作系统没有依赖性，到目前为止在win xp、7、8和Win 10预览版上测试都可以正常运行，让windows平台上也有这种容器的体验。Spoon能对容器进行颗粒级别的隔离。对比Docker来看，Spoon默认向网络开放容器，这样可以更容易对容器封仓，然后选择性的对网络再开放。开发者声称这样可以允许桌面应用默认运行。

	- Boot2Docker

		> [官方文档](https://github.com/boot2docker/boot2docker#installation)，[简单使用向导](http://www.widuu.com/archives/07/1068.html)  
		　　Boot2Docker是一个专为Docker设计的轻量级Linux发行版，解决Windows或者OS X用户不能安装Docker的问题。  
		Boot2Docker完全运行于内存中，24M大小，启动仅5-6秒。  
		Boot2Docker需要运行在VirtualBox中，具体的安装可以参考官方文档

