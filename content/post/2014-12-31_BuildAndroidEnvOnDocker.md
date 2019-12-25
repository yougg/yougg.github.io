---
Categories: ["Document"]
Description: ""
Tags: ["Golang","Docker","Android"]
title: "通过golang官方的docker镜像构建历史, 查看Android开发环境的搭建流程"
date: 2014-12-31T00:00:00+08:00
draft: false
---

环境Ubuntu 14.04

```bash
# 安装docker
sudo apt-get install docker.io
# 获取golang的Android开发环境镜像
sudo docker pull golang/mobile
# 查看下载完成的镜像
sudo docker images
# REPOSITORY    TAG     IMAGE ID       CREATED        VIRTUAL SIZE
# golang/mobile latest  c0c94ec4d34c   32 hours ago   3.013 GB
# 查看镜像的构建历史
sudo docker history golang/mobile
# IMAGE         CREATED       CREATED BY                                      SIZE
# c0c94ec4d34c  32 hours ago  /bin/sh -c #(nop) WORKDIR //src/golang.org/x/   0 B
# 93037fa1b597  32 hours ago  /bin/sh -c curl -L https://github.com/golang/   259.6 MB
# d3fed9f193ad  32 hours ago  /bin/sh -c #(nop) ENV PATH=/usr/local/sbin:/u   0 B
# 8d8c5b65f466  32 hours ago  /bin/sh -c #(nop) ENV GOPATH=/                  0 B
# 75829fddcc8b  32 hours ago  /bin/sh -c #(nop) ENV GOROOT=/go                0 B
# 9e1c49be1897  32 hours ago  /bin/sh -c #(nop) ENV PATH=/usr/local/sbin:/u   0 B
# 330740a9a042  32 hours ago  /bin/sh -c #(nop) ENV PATH=/usr/local/sbin:/u   0 B
# 99f28ebd5d62  32 hours ago  /bin/sh -c #(nop) ENV PATH=/usr/local/sbin:/u   0 B
# c8fa9da6de19  32 hours ago  /bin/sh -c #(nop) ENV PATH=/usr/local/sbin:/u   0 B
# 0aada3038943  32 hours ago  /bin/sh -c #(nop) ENV PATH=/usr/local/sbin:/u   0 B
# 6b0bfb5796dc  32 hours ago  /bin/sh -c #(nop) ENV GRADLE_HOME=/usr/local/   0 B
# 9b9022338587  32 hours ago  /bin/sh -c curl -L http://services.gradle.org   107.5 MB
# 2bcf53b685f3  32 hours ago  /bin/sh -c $NDK_ROOT/build/tools/make-standal   188.8 MB
# f98fe0ace224  32 hours ago  /bin/sh -c #(nop) ENV NDK_ROOT=/usr/local/and   0 B
# 96ab7a337f22  32 hours ago  /bin/sh -c curl -L http://dl.google.com/andro   1.406 GB
# 8c68e736ac03  32 hours ago  /bin/sh -c echo y | $ANDROID_HOME/tools/andro   163.4 MB
# d9e2e589091b  32 hours ago  /bin/sh -c #(nop) ENV ANDROID_HOME=/usr/local   0 B
# 2b4ebd5efc54  32 hours ago  /bin/sh -c curl -L http://dl.google.com/andro   172.5 MB
# 381c941bf47d  32 hours ago  /bin/sh -c #(nop) ENV ANT_HOME=/usr/local/apa   0 B
# 59e7896200ed  32 hours ago  /bin/sh -c curl -L http://archive.apache.org/   37.33 MB
# 5da1211d89a9  32 hours ago  /bin/sh -c apt-get update && apt-get -y inst    547 MB
# f8c83e954cfd  32 hours ago  /bin/sh -c echo "debconf shared/accepted-orac   910.8 kB
# 773df40de9cf  32 hours ago  /bin/sh -c #(nop) ENV DEBIAN_FRONTEND=noninte   0 B
# 28a945b4333c  4 days ago    /bin/sh -c #(nop) CMD [/bin/bash]               0 B
# 14500324c585  4 days ago    /bin/sh -c sed -i 's/^#\s*\(deb.*universe\)$/   1.911 kB
# db7c9bd0e843  4 days ago    /bin/sh -c echo '#!/bin/sh' > /usr/sbin/polic   156.2 kB
# a0ab785b06af  4 days ago    /bin/sh -c #(nop) ADD file:f1e0583674d1ae07ea   130 MB
# 511136ea3c5a  18 months ago                                                 0 B
```

按照顺序整理完成如下, golang的Android开发环境搭建流程:

```bash
# 最开始两步是对docker镜像环境的初始化与设置, 正式系统环境中不需要,省略掉

sed -i 's/^#\s*\(deb.*universe\)$/\1/g' /etc/apt/sources.list

export DEBIAN_FRONTEND=noninteractive

echo "debconf shared/accepted-oracle-license-v1-1 select true" | debconf-set-selections
echo "debconf shared/accepted-oracle-license-v1-1 seen true" | debconf-set-selections

# 安装环境依赖工具包 547 MB
apt-get update
apt-get -y install build-essential python-software-properties bzip2 unzip curl git subversion mercurial bzr libncurses5:i386 libstdc++6:i386 zlib1g:i386
add-apt-repository ppa:webupd8team/java
apt-get update
apt-get -y install oracle-java6-installer

# 安装ant 37.33 MB
curl -L http://archive.apache.org/dist/ant/binaries/apache-ant-1.9.2-bin.tar.gz | tar xz -C /usr/local
export ANT_HOME=/usr/local/apache-ant-1.9.2

# 安装Android SDK 172.5 MB
curl -L http://dl.google.com/android/android-sdk_r23.0.2-linux.tgz | tar xz -C /usr/local
export ANDROID_HOME=/usr/local/android-sdk-linux

# 安装Android SDK工具 163.4 MB
echo y | $ANDROID_HOME/tools/android update sdk --no-ui --all --filter build-tools-19.1.0
echo y | $ANDROID_HOME/tools/android update sdk --no-ui --all --filter platform-tools
echo y | $ANDROID_HOME/tools/android update sdk --no-ui --all --filter android-19

# 安装Android NDK 1.406 GB
curl -L http://dl.google.com/android/ndk/android-ndk-r9d-linux-x86_64.tar.bz2 | tar xj -C /usr/local
export NDK_ROOT=/usr/local/android-ndk-r9d

# 安装Android NDK编译平台环境 188.8 MB
$NDK_ROOT/build/tools/make-standalone-toolchain.sh --platform=android-9 --install-dir=$NDK_ROOT --system=linux-x86_64

# 安装Gradle 107.5 MB
curl -L http://services.gradle.org/distributions/gradle-2.1-all.zip -o /tmp/gradle-2.1-all.zip
unzip /tmp/gradle-2.1-all.zip -d /usr/local
rm /tmp/gradle-2.1-all.zip
export GRADLE_HOME=/usr/local/gradle-2.1

export GOPATH=/  
export GOROOT=/go
export PATH=${PATH}:${ANDROID_HOME}/tools:${ANDROID_HOME}/platform-tools:${NDK_ROOT}:${ANT_HOME}/bin:${GRADLE_HOME}/bin:${GOROOT}/bin

# 安装Go开发版 259.6 MB
curl -L https://github.com/golang/go/archive/master.zip -o /tmp/go.zip
unzip /tmp/go.zip
rm /tmp/go.zip
mv /go-master $GOROOT
echo devel > $GOROOT/VERSION
cd $GOROOT/src
./all.bash
CC_FOR_TARGET=$NDK_ROOT/bin/arm-linux-androideabi-gcc GOOS=android GOARCH=arm GOARM=7 ./make.bash

```

---

_**参考链接：**_  

- [golang/mobile](https://hub.docker.com/r/golang/mobile/)
