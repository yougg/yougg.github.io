---
Categories: ["Document"]
Description: ""
Tags: ["Emacs"]
title: "Emacs快捷键"
date: 2010-11-24T12:25:55+08:00
draft: false
---

#### 文件操作

`C-x C-f` 打开文件,出现提示时输入/username@host:filepath可编辑FTP文件  
`C-x C-v` 打开一个文件，取代当前缓冲区  
`C-x C-s` 保存文件  
`C-x C-w` 存为新文件  
`C-x i` 插入文件  
`C-x C-q` 切换为只读或者读写模式  
`C-x C-c` 退出Emacs

#### 编辑操作

`C-f` 前进一个字符  
`C-b` 后退一个字符  
`M-f` 前进一个字  
`M-b` 后退一个字  
`C-a` 移到行首  
`C-e` 移到行尾  
`M-a` 移到句首  
`M-e` 移到句尾  
`C-p` 后退一行  
`C-n` 前进一行  
`M-x` goto-line 跳到指定行  
`C-v` 向下翻页  
`M-v` 向上翻页  
`M-<` 缓冲区头部  
`M->` 缓冲区尾部

`C-M-f` 向前匹配括号  
`C-M-b` 向后匹配括号  

`C-l` 当前行居中  

`M-n` or `C-u n` 重复操作随后的命令n次  
`C-u` 重复操作随后的命令4次  
`C-u C-u` 重复操作随后的命令8次  
`C-x ESC ESC` 执行历史命令记录，`M-p`选择上一条命令，`M-n`选择下一条命令

`C-d` 删除一个字符  
`M-d` 删除一个字  
`C-k` 删除一行  
`M-k` 删除一句  
`C-w` 删除标记区域

`C-y` 粘贴删除的内容

注意：`C-y`可以粘贴连续`C-k`删除的内容；先按`C-y`，然后按`M-y`可以选择粘贴被删除的内容

`C-@` 标记开始区域  
`C-x h` 标记所有文字  
`C-x C-x` 交换光标位置和区域标记区开头  
`M-w` 复制标记区域

`C-_` or `C-x u` 撤消操作

#### 执行SHELL命令

`M-x` shell 打开SHELL  
`M-!` 执行SHELL命令 (shell-command)  
`M-1 M-!` 执行SHELL命令,命令输出插入光标位置,不打开新输出窗口  
`M-|` 针对某一特定区域执行命令(shell-command-on-region), 比如 C-x h M-|uuencode

#### 窗口操作

`C-x 0` 关闭本窗口  
`C-x 1` 只留下一个窗口  
`C-x 2` 垂直均分窗口  
`C-x 3` 水平均分窗口  
`C-x o` 切换到别的窗口  
`C-x s` 保存所有窗口的缓冲  
`C-x b` 选择当前窗口的缓冲区  
`C-x ^` 纵向扩大窗口  
`C-x }` 横向扩大窗口

#### 缓冲区列表操作

`C-x C-b` 打开缓冲区列表  
`d or k` 标记为删除  
`~` 标记为未修改状态  
`%` 标记为只读  
`s` 保存缓冲  
`u` 取消标记  
`x` 执行标记的操作

`f` 在当前窗口打开该缓冲区  
`o` 在其他窗口打开该缓冲区

#### 目录操作

`C-x d` 打开目录模式  
`s` 按日期/文件名排序显示  
`v` 阅读光标所在的文件  
`q` 退出阅读的文件  
`d` 标记为删除  
`x` 执行标记  
`D` 马上删除当前文件  
`C` 拷贝当前文件  
`R` 重名名当前文件  
`+` 新建文件夹  
`Z` 压缩文件  
`!` 对光标所在的文件执行SHELL命令  
`g` 刷新显示  
`i` 在当前缓冲区的末尾插入子目录的内容

`[n]m` 标记光标所在的文件，如果指定n，则从光标所在的文件起后n个文件被标记  
`[n]u` 取消当前光标标记的文件，n的含义同上  
`t` 反向标记文件  
`%-m` 正则标记

`q` 退出目录模式

说明：在目录模式中，如果输入`!`，在命令行中包含`*`或者`?`，有特殊的含义。  
`*`匹配当前光标所在的文件和所有标记的文件，  
`?`分别在每一个标记的文件上 执行该命令。

#### 程序编译

`M-x compile` 执行编译操作  
`M-x gdb` GDB排错  
`M-x dbx` DBX排错  
`M-x xdb` XDB排错  
`M-x sdb` SDB排错

#### 搜索模式

`C-s key` 向前搜索  
`C-s` 查找下一个  
`ENTER` 停止搜索  
`C-r key` 反向搜索  
`C-s C-w` 以光标所在位置的字为关键字搜索  
`C-s C-s` 重复上次搜索  
`C-r C-r` 重复上次反向搜索  
`C-s ENTER C-w` 进入单词搜索模式  
`C-r ENTER C-w` 进入反向单词搜索模式  
`M-x replace-string ENTER search-string ENTER` 替换  
`M-% search-string ENTER replace-string ENTER` 交互替换  
`C-r` 在进入单词搜索(查找/替换)模式后，该命令进入迭代编辑模式  
`C-M-x` 退出迭代编辑模式，返回到查找/替换模式  
`C-M- s` 向前正则搜索  
`C-M-r` 向后正则搜索  
`C-M-%` 正则交互替换

#### SHELL模式

`C-c C-c` 相当于Bash下的`C-c`  
`C-c C-z` 相当于Bash下的`C-z`  
`C-c C-d` 相当于Bash下的`C-d`  
`M-p` 执行前一条命令  
`C-n` 执行下一条命令  
`C-c C-o` 删除最后一条命令产生的输出  
`C-c C-r` 屏幕滚动到最后一条命令输出的开头  
`C-c C-e` 屏幕滚动到最后一套命令输出的结尾  
`C-c C-p` 查看前一条命令的输出  
`C-c C-n` 查看后一条命令的输出

#### 打印资料

`M-x print-buffer` 先使用pr,然后使用lpr  
`M-x lpr-buffer` 直接使用lpr  
`M-x print-region`  
`M-x lpr-region`

#### 收发邮件

`M-x mail` 发送邮件, C-c C-s 发送,C-c C-c 发送并退出  
`M-x rmail` 接受邮件

---

_**参考链接：**_  

- [Emacs百科](https://baike.baidu.com/history/emacs/6249437)
