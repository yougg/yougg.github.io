---
Categories: ["Document"]
Description: ""
Tags: ["OpenSSL","Certificate","JavaKeyStore"]
title: "Public Key Infrastructure Certificate Apply Guide"
date: 2017-08-28T15:50:10+08:00
draft: false
---

1. 生成XXX服务组件(`service` `DB`)的私钥和证书签发请求

	- ~~service服务证书签发请求和RSA私钥(PKCS#1)~~`已废弃`

		```bash
		# 生成service明文RSA私钥
		openssl genrsa -out service.key 4096
		# 使用service.key生成service的服务证书签发请求
		openssl req -new -newhdr -days 3650 -subj '/C=CN/ST=Guangdong/L=Shenzhen/O=XXOO/OU=OOuu/CN=service.xxx.com' -key service.key -out service.csr
		```

	- 生成service/DB的服务证书签发请求和RSA私钥(PKCS#8)

		```bash
		# service/DB证书请求和RSA私钥
		openssl req -new -newhdr -nodes -sha512 -days 3650 -subj '/C=CN/O=XXOO/CN=svc.xxx.com' -newkey rsa:4096 -keyout service.pem -out service.csr
		openssl req -new -newhdr -nodes -sha512 -days 3650 -subj '/C=CN/O=XXOO/CN=db.xxx.com' -newkey rsa:4096 -keyout svcdb.pem -out svcdb.csr

		# 生成几个随机密码替换为下面几个服务私钥的密码
		for i in {1..2}; do
			tr -cd 'a-z0-9' < /dev/urandom | head -c32; echo
		done

		# 转换私钥为PKCS#8格式并加密 (Go代码依赖的pkcs8包解析密钥密码时，硬编码了hash函数为sha1，所以此处转换私钥时强制使用hmacWithSHA1)
		openssl pkcs8 -in service.pem -out service.key -topk8 -passout pass:aaaaaaaa -v2 aes-256-cbc -iter 65536 -v2prf hmacWithSHA1
		openssl pkcs8 -in svcdb.pem -out svcdb.key -topk8 -passout pass:bbbbbbbb -v2 aes-256-cbc -iter 65536 -v2prf hmacWithSHA1

		# 验证生成的证书私钥文件
		openssl rsa -check -noout -passin pass:aaaaaaaa -in service.key
		openssl rsa -check -noout -passin pass:bbbbbbbb -in svcdb.key

		# 验证生成的证书签发请求文件
		openssl req -verify -noout -in service.csr
		openssl req -verify -noout -in svcdb.csr
		```

	- 使用自签发的CA根证书对证书签发请求进行签发测试

		```bash
		# 生成自签发根证书和私钥
		openssl req -new -x509 -sha512 -newkey rsa:4096 -keyout test_ca.key -days 3650 -out test_ca.crt -subj '/C=CN/ST=Guangdong/L=Shenzhen/O=XXOO/OU=OOuu/CN=*.xxx.com' -passout pass:tttttttt
		# 使用根证书与证书请求文件签发服务证书
		openssl x509 -req -CA test_ca.crt -CAkey test_ca.key -set_serial 7 -passin pass:tttttttt -in svcdb.csr -out test_svcdb.crt
		# 认证签发的服务证书
		openssl verify -CAfile test_svcca.crt test_svcdb.crt

		# 测试验证通过后删除临时文件
		rm -f test_*
		```

2. 到XXX机构申请签发服务证书

	- `证书申请` -> `SSL和TLS证书` -> 填写申请信息：

		|          |                     |                |                   |
		|----------|---------------------|----------------|-------------------|
		|申请人    |                     |申请时间        |2017-09-12 19:21:05|
		|申请人部门|XXOO产品部           |                |                   |
		|所属人    |                     |                |                   |
		|部门信息  |XXOO产品部           |                |                   |
		|产品线    |OOuu                 |                |                   |
		|产品分类  |XXOO Service         |                |                   |
		|证书模板  |SSL Server 10-year   |                |                   |
		|密钥算法  |RSA                  |哈希算法        |SHA256             |
		|有效期(天)|3650                 |到期时间（预期）|2027-09-10 19:56:30|
		|申请理由  |                     |                |                   |

		`添加CSR`上传上一步生成的所有`.csr`文件，提交申请

	- 下载签发成功的service服务证书到本地，查看证书信息

		```bash
		openssl x509 -noout -text -in service.crt
		openssl x509 -noout -text -in svcdb.crt
		```

	- 下载服务根证书进行验证

		一级根证书、二级根证书，将两级CA根证书合并为一个文件`svcca.crt`

		```bash
		awk '{print $0}' ca0.crt ca1.crt > svcca.crt
		```

		<!--
		```
		-----BEGIN CERTIFICATE-----
		......
		-----END CERTIFICATE-----

		-----BEGIN CERTIFICATE-----
		......
		-----END CERTIFICATE-----
		```
		-->

		使用根证书根证书认证签发下来的服务证书

		```bash
		# 使用下面3种的任意一种方式进行认证
		openssl verify -CAfile svcca.crt *.crt
		openssl verify -trusted svcca.crt *.crt
		openssl verify -CAfile ca1.crt -untrusted ca0.crt *.crt
		```

3. 替换旧的服务证书

	- 使用`certtool`工具导入service服务的私钥和签发的证书到JavaKeyStore中

		```bash
		certtool -import -jks -certin service.crt -keyin service.key -passin aaaaaaaa -jksout service.jks -trust svcca.crt
		```

	- 将CA根证书加入到service/DB服务的证书链中

		```bash
		(echo; cat svcca.crt) >> service.crt
		(echo; cat svcca.crt) >> svcdb.crt
		```

	- 更新Knox服务CA根证书

		```bash
		# openssl req -new -key knoxca.key -passin pass:ffffffff -out newknox.csr -subj '/C=CN/ST=Guangdong/L=Shenzhen/O=XXOO/OU=OOuu/CN=knox.xxx.com'
		# openssl x509 -req -days 3650 -in newknox.csr -signkey knoxca.key -passin pass:ffffffff -out newknoxca.crt
		openssl req -new -x509 -days 3650 -sha512 -key knoxca.key -passin pass:ffffffff -out newknoxca.crt -subj '/C=CN/ST=Guangdong/L=Shenzhen/O=XXOO/OU=OOuu/CN=knox.xxx.com'
		rm -f newknox.csr
		mv knox_root_ca.key knoxca.key
		mv newknoxca.crt knoxca.crt
		```

4. 服务证书相关问题答疑

	- <font style="color: #FF5A79;font-size: 1.5em;">:question:</font> 未申请PKI签发Knox服务的证书

		> 　　在创建集群时需要有service服务动态的为每个集群签发新服务证书，然后下发到执行层提供给Knox服务。XXX机构并不支持XXOO服务在不同环境部署时的动态签发证书申请，所以Knox服务证书的签发仍然按原来的流程执行：创建集群时由service服务使用自签发的`knoxca.crt`CA根证书根据集群信息动态签发Knox服务证书。

	- <font style="color: rgb(250, 125, 0);font-size: 1.5em;">:warning:</font> 双向证书认证

		> 　　管理面现在使用IP地址直连通信，启用双向证书认证后对服务的域名校验存在问题，需要使用DNS服务注册管理面各服务的域名，证书使用域名签发。  
	  如果不使用域名签发，则需要把服务器的IP地址列表添加到证书的IP SANs列表中。

---

_**参考链接：**_  

- [OpenSSL命令参考手册](https://www.openssl.org/docs/man1.1.0/apps/)  
- [Certification authority root certificate expiry and renewal](https://serverfault.com/questions/306345/certification-authority-root-certificate-expiry-and-renewal)
