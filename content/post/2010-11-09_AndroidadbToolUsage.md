---
Categories: ["Document"]
Description: ""
Tags: ["Android", "ADB"]
title: "adb工具使用"
date: 2010-11-09T11:38:39+08:00
draft: false
---


`adb`(Android Debug Bridge)是Android 提供的一个通用的调试工具，可以管理设备或手机 模拟器 的状态 。还可以进行以下的操作：

1. 快速更新设备或手机模拟器中的代码，如应用或Android系统升级；
2. 在设备上运行shell命令；
3. 管理设备或手机模拟器上的预定端口；
4. 在设备或手机模拟器上复制或粘贴文件；

以下为一些常用的操作：

- 安装 应用到模拟器：

	```bash
	adb install 
	```
	比较遗憾的是，Android并没有提供一个卸载 应用的命令，只能自己手动删除 ：

	```bash
	adb shell
	cd /data/app
	rm app.apk
	```

- 进入设备或模拟器的shell： 

	```bash
	adb shell
	```
	通过上面的命令，就可以进入设备或模拟器的shell环境中，在这个Linux Shell中，你可以执行各种Linux 的命令，另外如果只想执行一条shell命令，可以采用以下的方式：

	```bash
	adb shell [command]
	```
	如：
	
	```bash
	adb shell dmesg
	# 会打印出内核的调试信息。
	```

- 发布端口：   
  可以设置任意的端口号，做为主机 向模拟器或设备的请求端口。如：

	```bash
	adb forward tcp:5555 tcp:8000
	```

- 复制文件 ：  
  可向一个设备或从一个设备中复制文件，  
  复制一个文件或目录到设备或模拟器上：

	```bash
	adb push 
	```
	如：
	
	```bash
	adb push test.txt /tmp/test.txt
	```
	从设备或模拟器上复制一个文件或目录：

	```bash
	adb pull
	```
	如：

	```bash
	adb pull /addroid/lib/libwebcore.so .
	```

- 搜索模拟器/设备的实例：   
  取得当前运行的模拟器/设备的实例的列表及每个实例的状态：

	```bash
	adb devices
	```

- 查看bug报告： 

	```bash
	adb bugreport
	```

- 记录无线通讯日志： 
一般来说，无线通讯的日志非常多，在运行时没必要去记录，但我们还是可以通过命令，设置记录：

	```bash
	adb shell
	logcat -b radio
	```

- 获取设备的ID和序列号： 

	```bash
	adb get-product
	adb get-serialno
	```

- 访问数据库SQLite3 

	```bash
	adb shell
	sqlite3
	```

- 通过gsm call命令可以像Android 模拟器打电话 ，除了在EclipseADT 的DDMS中通过按钮Dial外，还可以通过DDMS外壳调用gsm call命令直接拨打，首先需要启动AndroidEmulator，然后在cmd环境下执行`telnet localhost 5554` 下面就可以向Android模拟器 拨号，参数为`gsmcall < phoneNum>` ，比如给10086打电话 为`gsm call +10086`

---

_**转载链接：**_  

- [安装APK文件到Android模拟器和Android sdcard的使用](http://java-admin.iteye.com/blog/359573)