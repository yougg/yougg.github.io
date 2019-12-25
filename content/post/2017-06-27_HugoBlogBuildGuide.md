---
Categories: ["Document"]
Description: ""
Tags: ["Blog","Hugo","GitHub"]
title: "我的静态博客搭建流程"
date: 2017-06-27T14:36:16+08:00
draft: false
---

1. 根据GitHub静态博客向导: https://pages.github.com/  
   在GitHub创建名为 __username__.github.io 的空仓库  
   __username__ 为自己的GitHub账号，此处我的账号为`yougg`  
   如果之前已经创建过个人博客仓库，先备份一下：

	```bash
	git clone https://github.com/yougg/yougg.github.io.git
	cd yougg.github.io
	git branch -m master old_master
	git push origin :master old_master
	```

2. 下载安装静态博客工具[hugo](https://gohugo.io/)

	```bash
	# 创建hugo工具目录
	mkdir -p ~/Downloads/hugo/
	cd $_
	# 获取Hugo Linux 64bit最新版本安装包的URL, Windows需要更改包名Windows-64bit.zip
	hgpkg=$(curl -ks -L'#' https://github.com/gohugoio/hugo/releases/latest | awk -F '"' '/Linux-64bit.tar.gz/{print "https://github.com"$2;exit}')
	# 使用curl命令下载最新版本安装包到本地
	curl -k -o $(basename ${hgpkg}) -L'#' ${hgpkg}
	# 或者使用wget命令下载
	wget --no-check-certificate ${hgpkg}
	```
	解压安装到本地，配置执行命令到PATH中
	```bash
	tar xzf $(basename ${hgpkg})
	# 建立执行文件软链接到PATH的一个路径中
	sudo unlink /usr/local/bin/hugo &> /dev/null
	sudo ln -s ~/Downloads/hugo/hugo /usr/local/bin/hugo
	```
	执行hugo命令查看版本，确认是否安装成功
	```bash
	hugo version
	```
	Debian/Ubuntu官方仓库已包含hugo(但是版本较旧)  
	使用下面一行命令安装hugo替代上面的流程
	```bash
	sudo apt install hugo
	```

3. 创建与配置博客工程

	创建博客站点，我的本地博客目录存放在`~/Documents/myblog/`
	```bash
	cd ~/Documents/
	hugo new site myblog
	cd myblog

	# 添加博客主题
	git clone https://github.com/kakawait/hugo-tranquilpeak-theme.git themes/hugo-tranquilpeak-theme

	# 复制主题中的配置模板文件
	cp -f themes/hugo-tranquilpeak-theme/exampleSite/config.toml config.toml

	mkdir -p static/css static/js
	# 导出一份语法高亮主题，参考：https://help.farbox.com/pygments.html
	hugo gen chromastyles --style=fruity > static/css/fruity.css
	sed -i 's/; background-color: #0f140f;//g' static/css/fruity.css
	```
	使用文本编辑器打开编辑配置文件
	```bash
	vi config.toml
	```
	设置站点url根路径
	```ini
	baseurl = "yougg.github.io"
	```
	修改博客默认语言
	```ini
	languageCode = "zh-cn"
	defaultContentLanguage = "zh-cn"
	```
	添加设置文章内容包含汉字
	```ini
	hasCJKLanguage = true
	```
	设置博客标题
	```ini
	title = "Yougg Blog"
	```
	设置博客主题风格
	```ini
	theme = "hugo-tranquilpeak-theme"
	```
	设置disqus评论系统用户名
	```ini
	disqusShortname = "yougg"
	```
	设置文章分页数量
	```ini
	paginate = 10
	```
	启用emoji:smile:表情设置
	```ini
	enableEmoji = true
	```
	设置语法高亮
	```ini
	pygmentsCodeFences = true
	pygmentsCodeFencesGuessSyntax = true
	pygmentsUseClasses = true
	pygmentsStyle = "fruity"
	```
	设置博客文章链接格式
	```ini
	[permalinks]
	  post = "/:year/:month/:day/:slug/"
	# 比默认的多添加了一个:day
	```
	设置作者相关信息
	```ini
	[author]
	  name = "yougg"

	  bio = "和咸鱼没有什么区别的偶尔写~~_不想写_~~博客的码农"

	  job = "搬砖"

	  location = "中国 深圳"

	  gravatarEmail = "joe2qq@gmail.com"

	  picture = "https://cdn1.iconfinder.com/data/icons/ninja-things-1/1772/ninja-simple-512.png"

	  # twitter = ""
	```
	修改日期格式
	```ini
	[params]
	  dateFormat = "2006年1月2日 15时04分05秒"
	# 必须满足Golang的日期格式：零纳秒 一月二日三点四分五秒六年七区八星九零
	```
	修改侧边导航栏默认行为
	```ini
	sidebarBehavior = 2
	```
	<!--添加github的语法高亮主题  
	下载[github.css](https://raw.githubusercontent.com/isagalaev/highlight.js/master/src/styles/github.css)、[highlight.min.js](http://cdnjs.cloudflare.com/ajax/libs/highlight.js/9.12.0/highlight.min.js)和[go.min.js](http://cdnjs.cloudflare.com/ajax/libs/highlight.js/9.4.0/languages/go.min.js)分别存放到工程的`static/css/`和`static/js/`目录中，然后将路径配置到自定义的选项中-->
	引用语法高亮主题CSS文件
	```ini
	customCSS = ["css/fruity.css"]
	```
	以上配置项修改完成保存后退出

	<!--
	```bash
	# 修改博客的默认设置：根路径、语言、标题
	cat themes/hugo-tranquilpeak-theme/exampleSite/config.toml > config.toml
	sed -i 's/baseurl = ""/baseurl = "yougg.github.io"/;
	s/en-us/zh-cn/g;
	s/title = "Hugo tranquilpeak theme"/title = "Yougg Blog"/;
	s/disqusShortname = "hugo-tranquilpeak-theme"/disqusShortname = "yougg"/;
	s/post = "\/:year\/:month\/:slug\/"/post = "\/:year\/:month\/:day\/:slug\/"/;
	s/name = "Firstname Lastname"/name = "yougg"/;
	s/job = "Your job title"/job = "Software Developer"/;
	s/location = "France"/location = "China"/;
	s/gravatarEmail = "thibaud.lepretre@gmail.com"/gravatarEmail = "joe2qq@gmail.com"/;
	s/url = "https:\/\/github.com\/kakawait"/url = "https:\/\/github.com\/yougg"/;
	s/sidebarBehavior = 1/sidebarBehavior = 2/' config.toml
	```
	-->

4. 编写和配置文章
	- 编辑默认的文档模板`archetypes/default.md`

		<pre>
		<code class="language-markdown codeblock hljs" data-lang="markdown"><span class="header">---</span>
		Categories: []
		Description: ""
		Tags: []
		title: "{{ replace .TranslationBaseName "-" " " | title }}"
		date: {{ .Date }}
		draft: false
		<span class="header">---</span>

		# Title

		Content

		<span class="header">---</span>

		_**参考链接：**_

		<span class="deletion">- [name](link)</span>
		</code>
		</pre>

	- 新建一篇文章，并添加一些内容到文章

		```bash
		hugo new post/first.md
		echo "This is my first post" >> content/post/first.md
		```

	- 启动hugo服务

		```bash
		# 本地启动服务：指定监听地址、端口，访问根路径和主题，包含草稿内容
		#hugo server --bind 127.0.0.1 -p 1313 -b http://127.0.0.1:1313/ --buildDrafts -v
		hugo server --buildDrafts
		```

	- 通过浏览器中访问主页 [http://127.0.0.1:1313/](http://127.0.0.1:1313/)  
	  访问 http://127.0.0.1:1313/post/first 查看文章内容  
	  按Ctrl+C停止hugo服务  
	  根据浏览器中查看的效果，按自己喜好修改博客的默认字体

		```bash
		sed -i 's/Merriweather,serif/youyuan,Merriweather,serif/g;
		s/"Open Sans",sans-serif/youyuan,"Open Sans",sans-serif/g;
		s/font-size:3.7rem/font-size:3.7rem;text-align:center;/g;
		s/.main-content-wrap{display:block;max-width:750px;/.main-content-wrap{display:block;max-width:1000px;/g' themes/hugo-tranquilpeak-theme/static/css/*.css
		```

5. 发布静态博客到GitHub Pages

	```bash
	# 初始化本地git仓库
	cd ~/Documents/myblog/
	git init
	echo "/public/" >> .gitignore
	echo "/themes/" >> .gitignore
	git add --all
	git commit -m "Initial commit"
	git branch -m master source

	# 本地仓库关联GitHub上面的个人博客仓库
	git remote add origin https://github.com/yougg/yougg.github.io.git
	# 将本地博客的源码推送到GitHub仓库的source分支
	git push -u origin source
	git push origin refs/heads/source:source

	# 更新草稿状态
	hugo undraft content/post/first.md
	# 生成静态网站内容
	hugo

	# 将生成的静态页面推送到master分支
	cd public
	git init
	git remote add origin https://github.com/yougg/yougg.github.io.git
	git add --all
	git commit -m "first post added"
	git push -f origin master
	```

---

_**参考链接：**_  

- [使用Hugo搭建静态站点](http://tonybai.com/2015/09/23/intro-of-gohugo/)
- [Tranquilpeak User documentation](https://github.com/kakawait/hugo-tranquilpeak-theme/blob/master/docs/user.md)
- [Hugo document](https://gohugo.io/getting-started/)
