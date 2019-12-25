---
Categories: ["Document"]
Description: ""
Tags: ["Golang","Algorithm","Sort"]
title: "Go语言排序算法笔记"
date: 2018-07-06T14:25:22Z
draft: false
---

# Install Flutter on Ubuntu Desktop

1. set apt proxy and http(s) proxy

    ```
    echo 'Acquire::http::Proxy "http://127.0.0.1:8080"' >> /etc/apt/apt.conf
    export http_proxy='http://127.0.0.1:8080'
    export {https,ftp,rsync,all}_proxy=$http_proxy
    ```

2. install git

    ```
    sudo apt install git
    git config --global http.sslVerify false
    ```

3. clone flutter

    ```
    mkdir -p ~/Flutter
    cd ~/Flutter
    git clone -b master https://github.com/flutter/flutter.git
    ./flutter/bin/flutter --version
    
    export PATH=${PATH}:~/Flutter/flutter/bin
    echo 'export PATH=${PATH}:~/Flutter/flutter/bin' >> ~/.profile
    
    flutter doctor
    
    sudo apt install lib32stdc++6
    ```

# set proxy forward for virtual device

1. download latest frp release package  
	https://github.com/fatedier/frp/releases

2. create a frpc_adb.ini file in host dev machine

	```ini
	[common]
	server_addr = 127.0.0.1
	server_port = 7000

	[secret_tcp]
	type = tcp
	local_ip = 127.0.0.1
	local_port = 8080
	remote_port = 8080
	```

3. create frps.ini file for emulator

	```ini
	[common]
	bind_port = 7000
	```

4. start Android Virtual Device with writeable file system

	```bash
	export LD_LIBRARY_PATH=~/Android/Sdk/emulator/lib64:~/Android/Sdk/emulator/lib64/qt/lib:~/Android/Sdk/emulator/lib64/gles_swiftshader
	~/Android/Sdk/emulator/qemu/linux-x86_64/qemu-system-x86_64 -netdelay none -netspeed full -studio-params ~/temp/emu2317465865946674366.tmp -avd Pixel_XL_API_28 -writable-system &
	```

5. copy frp files from host to emulator

	```bash
	adb shell mkdir -p /mnt/sdcard/frp
	adb push frps.ini /mnt/sdcard/frp/
	adb push frps /mnt/sdcard/frp/
	```

6. set port forward from host to emulator

	```bash
	adb forward tcp:7000 tcp:7000
	```

7. remount emulator partitions read-write

	```bash
	adb remount
	```

8. go into emulator

	```bash
	adb shell
	```

9. mount root partition writeable in emulator

	```bash
	mount -o remount,rw /
	```

10. run frps service in emulator

	```bash
	mv /mnt/sdcard/frp /data/local/tmp/
	/data/local/tmp/frp/frps -c /data/local/tmp/frp/frps.ini
	```

11. set http proxy in emulator WiFi connection

	```
	http_proxy: http://127.0.0.1:8080
	```

_**参考链接：**_  

- [非root设备如何通过WiFi连接adb?](https://www.zhihu.com/question/33436407)
- [How can I connect to Android with ADB over TCP?](https://stackoverflow.com/questions/2604727/how-can-i-connect-to-android-with-adb-over-tcp)