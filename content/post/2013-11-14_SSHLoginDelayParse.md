---
Categories: ["Document"]
Description: ""
Tags: ["ssh"]
title: "ssh登录慢的原因分析及解决方法"
date: 2013-11-14T00:00:00+08:00
draft: false
---

DNS解析问题导致的登录时明显的停顿，有两个阶段：

1. 输入用户名敲回车之后，提示输入密码前；
    - 原因sshd默认UseDNS选项是打开的，失败的DNS解析超时导致。
    - 解决方案:  
      (在 `/etc/ssh/sshd_config` 添加一行 `UseDNS no` 可以关闭sshd的域名解析。)

2. 输入密码敲回车之后，shell提示符回显之前；
    - 原因：在登陆shell bash通过网络启动时，会试图作域名解析(opennet->getaddrinfo)，  
      如果配置了nameserver且不可用的话就会因DNS解析等待超时，使用户感觉到明显停顿。
    - 解决方案:  
      (把 `/etc/resolve.conf` 里面的错误的nameserver注释掉 或者 直接把该文件清空就行)
