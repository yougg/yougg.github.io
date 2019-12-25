---
Categories: ["Document"]
Description: ""
Tags: ["RIDE","Robot Framework","Autotest"]
title: "Ubuntu18.04安装使用RIDE"
date: 2018-07-30T19:19:52Z
draft: false
---

# Ubuntu 18.04 install Robot Framework RIDE

## set pip install source

- Linux/Unix `~/.pip/pip.conf`
- Windows `C:\Users\用户名\pip\pip.ini`

	```ini
	[global]
	trusted-host=rnd-mirrors.huawei.com
	index-url=http://rnd-mirrors.huawei.com/pypi/simple/
	```

## latest version

- install python pip

	```bash
	apt install -y python3-pip
	```

- install wxPython (for linux)

	```bash
	pip install -U -f https://extras.wxpython.org/wxPython4/extras/linux/gtk3/ubuntu-18.04 wxPython
	```

- install RIDE (for windows, only this step need)

	```bash
	pip install -U -r https://raw.githubusercontent.com/HelioGuilherme66/RIDE/v1.7.2/requirements.txt
	pip install -U https://github.com/HelioGuilherme66/RIDE/archive/v1.7.2.tar.gz
	```

## old version

- install python pip

	```bash
	apt install -y python-pip
	```

- install wxPython

	```bash
	echo "deb http://old-releases.ubuntu.com/ubuntu wily main universe" | tee /etc/apt/sources.list.d/wily-copies.list
	apt update
	apt install -y python-wxgtk2.8
	```

- install RIDE

	```bash
	pip install robotframework-ride
	pip install pygments
	```

- start RIDE

	```bash
	ride.py &
	```