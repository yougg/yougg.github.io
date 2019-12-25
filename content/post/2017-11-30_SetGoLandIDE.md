---
Categories: ["Document"]
Description: ""
Tags: ["Golang","GoLand"]
title: "配置GoLand集成开发环境"
date: 2017-11-30T16:38:52+08:00
draft: false
---

### 安装必备工具

> 下载安装以下软件，确保执行命令的路径都已添加到系统环境变量`PATH`中

#### [Golang](https://golang.org/dl/)

- 安装向导
	1. 使用默认安装路径

#### [Notepad++](https://notepad-plus-plus.org/download/)

- 安装向导
	1. 使用默认安装路径  
	1. 勾选默认安装组件  
	1. 勾选创建桌面快捷方式

#### [Graphviz](http://www.jetbrains.com/go/download/)

- 安装向导
	1. 使用默认安装路径  
	1. 安装后需要手工添加`C:\Program Files (x86)\Graphviz2.38\bin`到环境变量`PATH`中  
	1. 添加环境变量后，打开CMD执行`dot -V`查看版本

#### [Git](https://git-scm.com/downloads)

- 安装向导
	1. 使用默认安装路径  
	1. 勾选所有安装组件  
	1. 使用`Vim`作为Git默认编辑器  
	1. 在Windows命令中使用Git和可选的Unix工具  
	1. 使用OpenSSL库  
	1. <input type="radio" checked="checked"/>Checkout as-is, commit Unix-style line endings  
	1. 使用MinTTY终端模拟器  
	1. 启用文件系统缓存,启用Git凭证管理,启用符号链接

#### GoLand [EAP版本](http://www.jetbrains.com/go/nextversion/) [正式版](http://www.jetbrains.com/go/download/)

- 安装向导
	1. 自定义安装路径  
	1. 勾选创建桌面快捷方式

- 配置向导

	- 启动GoLand，选择不导入配置，同意用户协议  
	- 点击右下角`Configure` -> `Settings`进入设置  
	- 选择`Go` -> `GOROOT`，下拉选择Go SDK  
	- 选择`Go` -> `GOPATH`，去除勾选的两项：

		> <input type="checkbox" disabled="disabled"/> Use GOPATH that's defined in system environment  
		  <input type="checkbox" disabled="disabled"/> Index entrie GOPATH  
	- 选择`Go` -> `Imports`，排序类型下拉选择goimports，并勾选下面3个分组选项  

		> <input type="checkbox" checked="checked" disabled="disabled"/> Group stdlib imports  
		  　<input type="checkbox" checked="checked" disabled="disabled"/> Move all stdlib imports in a single group  
		  <input type="checkbox" checked="checked" disabled="disabled"/> Move all imports in a single declaration  
	- 选择`Appearance & Behavior` -> `Appearance`  
		- Theme下拉选择Darcula  
		- <input type="checkbox" checked="checked" disabled="disabled"/>Override default fonts by(not recommenended):

			> Name: 推荐使用[`Source Code Pro`](https://github.com/adobe-fonts/source-code-pro/releases)，Size: 推荐14号
	- 选择`Appearance & Behavior` -> `System Settings` -> `HTTP Proxy`设置自己的代理  
	- 选择`Keymap`，点击方案列表下拉框右边设置选择`Duplicate...`然后重命名  
		- 点击展开`Main menu` -> `Code` -> `Completion`  
			- 右键点击Basic，移除`Ctrl+空格`  
			- 右键点击SmartType，移除`Ctrl+Shift+空格`  
			- 右键点击Cyclic Expand Word，移除`Alt+/`  
			- 右键点击Cyclic Expand Word (Backward)，移除`Alt+Shift+/`  
			- 右键点击Basic，点击Add Keyboard Shortcut，按快捷键`Alt+/`然后点击OK  
			- 右键点击SmartType，点击Add Keyboard Shortcut，按快捷键`Alt+Shift+/`然后点击OK  
			- ~~右键点击Cyclic Expand Word，点击Add Keyboard Shortcut，按快捷键`Ctrl+空格`然后点击OK~~  
			- ~~右键点击Cyclic Expand Word (Backward)，点击Add Keyboard Shortcut，按快捷键`Ctrl+Shift+空格`然后点击OK~~  
	- 选择`Editor` -> `General` -> `Code Folding`，去除勾选`Imports`  

		> <input type="checkbox" disabled="disabled"/> Imports  
	- 选择`Plugins`

		- 勾选与勾除下列默认的插件  

			|Plugin                         |Checked|
			|-------------------------------|:-----:|
			|Copyright                      |<input type="checkbox" disabled="disabled"/>|
			|CSS Support                    |<input type="checkbox" disabled="disabled"/>|
			|Database Tools and SQL         | <input type="checkbox" checked="checked" disabled="disabled"/>   |
			|EditorConfig                   |<input type="checkbox" disabled="disabled"/>|
			|File Watchers                  | <input type="checkbox" checked="checked" disabled="disabled"/>   |
			|Git Integration                | <input type="checkbox" checked="checked" disabled="disabled"/>   |
			|GitHub                         |<input type="checkbox" disabled="disabled"/>|
			|Go                             | <input type="checkbox" checked="checked" disabled="disabled"/>   |
			|Go IDE                         | <input type="checkbox" checked="checked" disabled="disabled"/>   |
			|HTML Tools                     |<input type="checkbox" disabled="disabled"/>|
			|IntelliLang                    | <input type="checkbox" checked="checked" disabled="disabled"/>   |
			|JavaScript Debugger            |<input type="checkbox" disabled="disabled"/>|
			|JavaScript Intention Power Pack|<input type="checkbox" disabled="disabled"/>|
			|JavaScript Support             |<input type="checkbox" disabled="disabled"/>|
			|Mercurial Integration          |<input type="checkbox" disabled="disabled"/>|
			|QuirksMode                     |<input type="checkbox" disabled="disabled"/>|
			|Refactor-X                     |<input type="checkbox" disabled="disabled"/>|
			|REST Client                    | <input type="checkbox" checked="checked" disabled="disabled"/>   |
			|Terminal                       | <input type="checkbox" checked="checked" disabled="disabled"/>   |
			|TextMate bundles support       |<input type="checkbox" disabled="disabled"/>|
			|tslint                         |<input type="checkbox" disabled="disabled"/>|
			|XPathView + XSLT Support       |<input type="checkbox" disabled="disabled"/>|
			|YAML                           | <input type="checkbox" checked="checked" disabled="disabled"/>   |
		- 点击`Browse repositories...`，搜索并安装以下插件  
			- .ignore  
			- BashSupport  
			- Markdown support  
			- PlantUML  
			- IDEA Mind Map  
			- Ini4Idea  
			- Key Promoter X  
	- 选择`Tools` -> `Terminal`，在Shell path中填入  

		> `"C:\Program Files\Git\bin\bash.exe" --login -i`  

	- 点击OK，然后点击Restart重启  
		- 点击`New Project`  
			- Location选择新建一个目录，如：`D:\Workspace\20180208`  
			- Sdk点击下拉选择  
			- 点击Create创建工程
		- 点击`Open Project`
			- 选择目录，如：`D:\workspace\Proj`
	- 按快捷键`Ctrl+Shift+A` -> 输入<u>Toolbar</u>，点击OFF变为ON
	- 按快捷键`Ctrl+Shift+A` -> 输入<u>Navigation Bar</u>，点击ON变为OFF
	- 按快捷键`Ctrl+Shift+A` -> 输入<u>Tool Buttons</u>，点击OFF变为ON
	- 按快捷键`Ctrl+Shift+A` -> 输入<u>Autoscroll from Source</u>，点击OFF变为ON
	- 按快捷键`Ctrl+Shift+A` -> 输入<u>Autoscroll to Source</u>，点击OFF变为ON
	- 按快捷键`Ctrl+Shift+A` -> 输入<u>File Types</u>回车

		> 在Recognized File Types中选择`Ini`  
		  在下方Registered Patterns点击`+`  
		  输入`*.properties`点击OK  
		  再点击`+`，输入`*.conf`点击OK

	- 点击`File`菜单，选择`Settings` -> `Other Settings` -> `PlantUML`

		> Graphviz dot executable 点击Browse...，选择：  
		  `C:/Program Files (x86)/Graphviz2.38/bin/dot.exe`

### GoLand配置一个Project中支持多个Module

- 使用GoLand分别打开多个项目，`File` -> `Open...`选择项目目录
- 关闭打开的项目，保留需要支持多个Module的项目
- 用文本编辑器打开各项目根目录下的Module配置文件 `.idea/modules.xml`
- 复制module标签内容到保留的Module的配置文件中

	```xml
	<module fileurl="file://$PROJECT_DIR$/.idea/proj.iml" filepath="$PROJECT_DIR$/.idea/proj.iml" />
	```

- 修改复制的`module`标签中文件路径，使用相对路径或者绝对路径

	```xml
	<?xml version="1.0" encoding="UTF-8"?>
	<project version="4">
		<component name="ProjectModuleManager">
			<modules>
				<module fileurl="file://$PROJECT_DIR$/.idea/proj.iml" filepath="$PROJECT_DIR$/.idea/proj.iml" />
				<module fileurl="file://$PROJECT_DIR$/../module0/.idea/module0.iml" filepath="$PROJECT_DIR$/../module0/.idea/module0.iml" />
				<module fileurl="file://$PROJECT_DIR$/../module1/.idea/module1.iml" filepath="$PROJECT_DIR$/../module1/.idea/module1.iml" />
			</modules>
		</component>
	</project>
	```

- 在GoLand中保留的项目里面刷新即可显示多个Module

---

_**参考链接：**_  


