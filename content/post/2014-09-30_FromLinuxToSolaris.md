---
Categories: []
Description: ""
Tags: []
title: "从Linux到Solaris"
date: 2014-09-30T00:00:00+08:00
draft: false
---

# 第一章：差别概览

## 1. 默认shell

|Linux|Solaris|
|-----|-------|
|/bin/sh -> /bin/bash|/bin/sh|

    两个操作系统的默认shell虽然都是/bin/sh，
    但linux默认shell是bash，/bin/sh仅是一个指向到/bin/bash的符号链接。
    而solaris的默认shell是Bourne shell，名为/bin/sh。

## 2. 文件系统

|Linux|Solaris|
|-----|-------|
|ext2<BR>ext3<BR>ext4<BR>reiser<BR>JFS<BR>XFS|UFS<BR>VxFS<BR>QFS<BR>ZFS|


    标准的solaris文件系统格式是UFS,还可以使用VxFS,QFS,从Solaris 10 u2版开始，还可以使用ZFS。
    Linux通常使用ext4,ext3,ext2,reiser,JFS,XFS其中一种。

## 3. 目录布局

    最值得注意的是/proc目录。
    Linux的/proc目录存放与系统配置以及进程有关的信息，可以修改这些文件以更新内核变量和进程信息。
    而Solaris的/proc目录仅包含进程信息，不能从/proc目录获取系统信息或调整内核变量，但Solaris使用/platform目录，这个目录包含平台特定的信息和应用，Linux没有与Solaris的/platform对应的目录。

## 4. 命令的位置

    为了保持对System V,BSD,GNU软件的兼容性，除了常规的/bin(/usr/bin)和/sbin(/usr/sbin)目录外，Solaris还使用了一些扩展的命令目录，这些目录如下所示：
>    /usr/openwin
>    /bin/usr/dt/bin
>    /usr/sfw/bin
>    /opt/sfw/bin
>    /usr/xpg4/bin
>    /usr/ccs/bin
>    /usr/ucb

    其中：
    /usr/bin        标准的System V命令
    /usr/ucb        传统的BSD命令
    有些命令在这两个目录中都有，但用法可能不同，比如
|&nbsp;||||||
|-------|---|---|---|---|---|
|basename|df|du|echo|expr|fastboot|
|fasthalt|file|from|groups|install|ld|
|lint|ln|lpc|lpq|lpr|lprm|
|lptest|ls|mkstr|printenv|ps|rusage|
|sed|shutdown|stty|sum|test|touch|
|tr|tset|users|vipw|whereis|whoami|


    免费软件的部署Linux和Solaris相同，这些GNU的命令在Solaris中通常都以g字母开头，比如gtar。

    System V和BSD中有两个目录包含免费软件：/usr/sfw/bin和/opt/sfw/bin。前者包含从安装介质中安装的免费软件，后者则是从配套CD中安装的软件。

    随着Solaris版本的更新，有可能会把配套CD上的软件放入Solaris安装介质中，因此需要注意在旧版本上的/opt/sfw/bin目录中的命令可能会被移植到/usr/sfw/bin中。凡是在/usr/sfw/bin中的软件，表示能够通过Sun的标准支持通道获得完全技术支持，而/opt/sfw/bin中的软件则通常是由开源软件组织获得技术支持。

## 5. 网络配置文件

|Linux|Solaris|
|-----|-------|
|/etc/ntp.conf|/etc/inet/ntp.conf|
|/etc/[x]inetd.conf|/etc/inet/inetd.conf|
|/etc/sysconfig/network-scripts/ifcfg-{interface}|/etc/hostname.{interface}and/etc/inet/netmasks|
|/etc/sysconfig/network|/etc/nodenameand/etc/defaultrouter|
|/etc/networks|/etc/networks->/etc/inet/networks|

## 6. 文件系统配置

|Linux|Solaris|
|-----|-------|
|/etc/fstab|/etc/vfstab|
|/etc/exports|/etc/dfs/dfstab|
|/etc/auto.master|/etc/auto_master|
|/etc/auto.home|/etc/auto_home|

## 7. 邮件

|Linux|Solaris|
|-----|-------|
|/etc/aliases|/etc/mail/aliases|
|/etc/mail.rc|/etc/mail/Mail.rc<br>/etc/mail/mailx.rc|

## 8. 日志文件

|Linux|Solaris|
|-----|-------|
|/var/log|/var/log<BR>/var/adm|


    在Linux系统中，日志文件的主目录为/var/log，各种系统守护进程的日志文件均存在此处。

    Solaris稍有不同，/var/log目录存放syslog和authlog的日志文件，而/var/adm目录则存放消息日志文件，

    在缺省配置时，solaris的/var/adm/messages文件（redhat对应的文件为/var/log/messages）包含所有的日志记录（可通过修改syslog.conf文件为不同的日志指定不同的消息记录文件）。

## 9. 脚本移植

    如果要把脚本从Linux移植到Solaris，需要注意以下几点：
    · 首先确定脚本中所使用的所有文件和路径在Solaris中均有效
    · 确定所有的选项和参数是否有变化
    · 命令的执行输出是否有区别

## 10. 帮助文件

    1、共同点：
    都可以查看whatis数据库中的关键字,比如uname命令：# apropos uname or man -k uname
    都可以直接在man命令中指定搜索路径：# man -M /opt/man command

    2、man的差异
    Linux的man
    # whatis printf
    printf             (1)  - format and print data
    printf             (3)  - formatted output conversion
    printf [builtins]  (1)  - bash built-in commands, see bash(1)
    # man 3 printf

    Solaris的man
    $ whatis printf
    printf          printf (1)      - write formatted output
    printf          printf (3c)     - print formatted output
    printf          printf (3ucb)   - formatted output conversion
    $ man -s 3c printf

    添加新搜索路径
    linux把新搜索路径加入/etc/man.conf文件，比如“MANPATH /opt/man”。
    然后运行makewhatis，可更新whatis数据库
    solaris可在/etc/profile文件中加入：
    MANPATH=$MANPATH:/opt/man
    export MANPATH

    3、Linux特有的帮助：
    Linux还可以使用info查看帮助，info中带有简单的菜单式链接。按回车进入菜单所链接的章节，按q退出
    最后Linux在/usr/share/doc/目录中还提供了一些其他格式（pdf、html等）的帮助资源。每个子目录对应一个应用，存放和应用相关的配置、设置等帮助资料。比如/usr/share/doc/bind*,存放和DNS服务器应用软件bind有关的帮助信息。


# 第二章：命令差别

- 绝大多数linux命令都有两种类型的选项：简洁式（short form,比如-v）和长格式（long form,比如--version）。
- 而Solaris的命令，除非是GNU版本的以外，通常都没有长格式。
- Linux命令可使用"--help"查看简要帮助，Solaris部分新命令可使用"-?"达到同样的效果，比如pkginfo -?

    如果从linux移植脚本到solaris，就必须注意这点区别，用GNU版本的命令来替代或把这些Solaris没有的选项替换成对应的简洁式选项。Linux和Solaris之间相互匹配的命令清单，可到网站查看：http://bhami.com/rosetta.html。本文仅列出一些常见命令的区别点：

|&nbsp;||||
|---|---|---|---|
|awk|basename|cat|chown|
|df|du|ps|setfacl|
|getfacl|tar|uesradd|groupadd|

## 1. awk

    Linux使用的是GNU版本的awk命令。
    Solaris默认使用System V版本的awk，GNU的awk有一些System V不具备的扩展功能。但Solaris也提供了其他几种版本的awk命令，分别放置在以下几个目录中：

|Linux|Solaris|说明|
|---|---|---|
||/usr/bin/awk|标准的SystemV版本的awk|
||/usr/bin/nawk|新版的SystemVawk,比前者多了许多扩展特性|
||/usr/xpg4/bin/awk|XPG4的awk.当从Linux移植脚本到Solaris时，可使用XPG4 awk。|
|/usr/bin/awk -> /bin/gawk|/opt/sfw/bin/gawk|GNUawk.配套CD上的awk.和其他版本相比，gawk和Linux的awk的兼容性最高。Solaris10配套CD中的GNU awk版本为3.0.6|

## 2. basename

|Linux|Solaris|
|-----|-------|
|/usr/bin/basename|/usr/ucb/bin/basename|
||/usr/bin/basename|


    Linux和Solaris上的basename命令的基本功能都相同。
    Solaris提供两个basename命令：
    /usr/ucb/bin/basename和Linux版本的basename命令相同
    /usr/bin/basename的功能更强，可以通过表达式模型匹配后缀（可参见http://bhami.com/rosetta.html）

## 3. cat

    Linux和Solaris的cat命令非常类似，有些选项有细微差别，如下所示：

|Linux|Solaris|
|-----|-------|
|-a,--show-all|-vet|
|--number-nonblank|-b|
|-e|-ve|
|-E,--show-ends|-ve|
|-s,--squeeze-blank|无|
|-t|-vt|
|-T,--show-tabs|-vt|
|-u(ignored)|-u(输出不使用缓冲,默认为缓冲输出)|
|--show-nonprinting|-v|
|--help|无|
|--version|无|

## 4. chown

    两个操作系统都支持-f和-R这两个基本选项。若指定的文件是指向到目录的符号链接均需使用扩展选项
    · -f（强制执行,不报告错误）
    · -R（递归,遇到符号链接仅改变链接的目标，符号链接自身不会改变，不会遍历符号链接的目标目录）

    Solaris提供了两个版本的chown命令，/usr/bin/chown和/usr/ucb/chown.
    · /usr/ucb/chown只支持两个选项：-f和-R
    · /usr/bin/chown除了-f和-R外，还支持-h,-H,-L,-P这些扩展选项（要和基本选项-f,-R一起使用）.
    除非使用-h,-P选项，否则符号链接自身的拥有者不会改变
    Solaris的/usr/bin/chown的-h,-H,-L,-P
    Solaris的-h等同于Linux的-h or --no-dereference.比如lncht是指向到cht目录的符号链接
    # chown -R solaris lncht     仅改变目标目录cht的所有者，符号链接自身不会改变，不会遍历符号链接的目标目录
    # chown -Rh solaris lncht   与仅使用-R相比，-Rh把符号链接lncht自身的所有者也改了，其他相同

    -H 如果是指向到目录的符号连接，则目录和其下的文件的所有者发生改变。但若目录下的文件也是个符号链接，则目标文件的所有者被改变，但不会再次进行递归操作。
    -L 和-H类似,但更彻底，在遍历时遇到指向到目录的符号链接，不仅改变目标目录的所有者，同时还会继续遍历目标目录进行改变操作。
    -P 指定的文件或在遍历各级目录时遇到的文件是符号链接，则改变符号链接的所有者。-P不会对符号链接的目标目录进行遍历。-P类似于--no-dereference
    -P>-L>-H
    Linux的chown命令的扩展选项：
    -c or --changes 类似于详细模式，但仅报告被改变的部分
    --dereference 命令对符号链接起效，这是solaris的默认行为。移植脚本时需注意此项
    --from= 仅改变符合指定的属主/属组的文件的所有者。solaris没有对应的选项，可用find命令的-ower or -group选项把查找结果传递给chown。

## 5. df

    Solaris支持df命令的多种实现方式，比如
    /usr/ucb/df  可使用-v选项，-v选项除了大小以（每个文件系统所支持的最小的）块的为单位显示外其他都和df -k相同。
    /usr/xpg4/bin/df  可使用附加的－P标记，大小以512字节为单位，其他与-k选项相同。

## 6. du

    Solaris提供了多个du命令，其中-H选项的含义和Linux的du命令不同.
    Linux的du的-H和-si选项作用相同，表示以1000为单位而不是1024，
    Solaris的-H选项这是表示处理符号链接所指向的实际目标文件，类似于Linux du的-L.
    其他的差别如下所示(减号"-"表示无对应选项)：

|Linux|Solaris|Solaris|Solaris|
|-----|-------|-------|-------|
|/usr/bin/du|/usr/bin/du|/usr/xpg4/bin/du|/usr/ucb/du|
|-a,--all|-a|-a-|-a|
|--block-size=SIZE|-|-|-|
|-b,--bytes|-|-|-|
|-c,--total|-|-|-|
|-D,--dereference-args|-L|-L|-L|
|-h,--human-readable|-h|-h|-h|
|-H,-si|-|-|-|
|-k,--kilobytes|-k|-k|-k|
|-l,--count-links|-|-|-|
|-L,--dereference|-L|-L|-L|
|-m,-megabytes|-|-|-|
|-S,-separate-dires|-o|-|-|
|-s,-summarize|-s|-s|-s|
|-x,-one-file-system|-d|-x|-d|
|-XFILE,-exclude-from=FILE|-|-|-|
|--exclude=PAT|-|-|-|
|--max-depth=N|-|-|-|
|--help|-|-|-|
|--version|-|-|-|


    部分选项简要说明：
    -b,-block-size=SIZE 以字节为单位显示大小
    -c produce a grand total
    -l 统计链接
    -L  处理符号链接实际指向的目标
    -m 以MB为单位显示

## 7. ps

    Solaris的/usr/ucb/ps是BSD风格的命令，和Linux的ps命令相当，但命令输出可能会有所不同

## 8. setfacl

    /usr/bin/setfacl用来管理文件的访问控制列表。两个系统上的这个命令的语法和选项不同。
    Solaris的setfacl语法：
    setfacl [-r] -s acl_entries file
    setfacl [-r] -md acl_entries file
    setfacl [-r] -f acl_file file
    -s 设置ACL，旧的ACL会被新指定的清空或覆盖
    -m用于添加ACL或修改现有的ACL，也会覆盖现有的ACL条目.
    -d 删除ACL条目
    -r 重新计算ACL的mask
    两个系统的ACL条目的格式很相似，Linux支持Solaris的ACL条目格式，但Solaris不支持某些Linux特有的ACL条目格式。
    Linux接受在other和mask关键词之后使用一个附加的冒号":",但Solaris不行。
    权限由rwx或数字组成,比如"r-x"表示读和执行权限.Linux支持缩写成"rx",而Solaris不行
    Solaris的setfacl能够使用另外一个文件的ACL来设置指定文件的ACL(使用-f选项,Linux版本的setfacl没有这个选项)
    下表列出了Linux的setfacl命令独有的一些选项：

|setfacl选项|解释|
|----------|----|
|-b,--remove-all|删除所有的ACL|
|-k,--remove-default|删除默认的ACL|
|-n,--no-mask|不重新计算有效权限掩码，等于solaris中不使用-r选项的setfacl命令|
|--mask|重新计算有效权限掩码，等于solaris的-r选项|
|-d,--default|所有的操作作用于默认的ACL|
|--restore=file|从"getfacl-R"创建的权限备份中恢复权限|
|--test|测试模式|
|-R,--recursive|递归模式，应用到所有的文件和目录|
|-L,--logical|与-P现对，对符号链接有效（followthesymboliclinks）|
|-P,--physical|忽略所有的符号链接|
|--version|查看命令版本|
|--help|查看命令的简要使用帮助|
|--|命令行结束|
|-|如果文件名参数是一个单破折号"-"，表示从标准输入读取文件列表|

## 9. getfacl

    /usr/bin/getfacl用来查看文件的访问控制列表。虽然Linux和Solaris的getfacl命令的功能相同且输出格式也很类似，但它们的可用选项有区别。
    solaris的getfacl命令不支持长格式选项。
    Linux的getfacl支持，而Solaris的getfacl不支持的简洁式选项：-R,-L,-P,以及-。

## 10. tar

    Linux的tar命令是GNU tar,Solaris的tar是System V版本.
    这两种版本的tar命令有很大的不同，最好参考它们各自的命令帮助手册.
    solaris的tar不支持使用外部压缩程序，因此没有-Z,-z,-j这些压缩选项
    Solaris的GNU tar命令为/usr/sfw/bin/gtar，安装包为SUNWgtar
    如果脚本中有使用tar命令，在移植时有两种方法
    重新编辑tar命令的用法，采用等价的tar选项，或者使用管道符把结果传递给压缩或解压程序。
    如果solaris安装了SUNWgtar，则可以在脚本中使用gtar来代替tar.

## 11. useradd

    两个系统的useradd命令非常相似，很多选项的作用几乎完全一致，主要区别如下：
    Solaris版的useradd有额外的选项以支持RBAC。
    另外一个明显的区别是-p选项。
    linux中，这个选项用来指定账号的密码，这是一种不安全的做法。
    Solaris中-p选项用来指定账号所属的项目(project),指定账号所开启的所有进程将都属于这个项目.
    另外一些具体的区别如下表所示：

|Linux|Solaris|命令解释|
|-----|-------|--------|
|-e expire_date|-e expire_date|指定过期时间，日期的格式有所不同|
|-f inactive|-f inactive|Linux中，指过期后多久变成永久禁用Solaris中，指到账号无效为止的最大天数|
|-M||不创建用户的家目录|
||-m|如果家目录不存在，就创建|
|-n||创建一个和账号同名的组|
|-o|-o|允许使用非唯一性uid|
|-p passwd||指定加密后的密码串（用crypt加密）|
||-p profile|指定用户属于哪个项目|
|-r||创建一个系统账号（默认使用一个比UID_MIN小的uid，UID_MIN在/etc/login.defs中定义）。这个是redhat专用选项|
||-K key=value|（RBAC使用）设置key/value对|
||-A authorization|（RBAC使用）设置授权|
||-P profile[,profiles...]|（RBAC使用）设置权力配置|
|-D -e default_expire_date|-D -e default_expire_date|设置账号的默认过期时间，日期格式有所不同|

## 12. groupadd

    两个系统的groupadd命令也极为相似，绝大多数时候两个命令的选项(-g指定gid，-o允许gid重复)和操作都完全一致。
    Linux的groupadd有两个专用选项（这两个选项由红帽子“redhat linux”加入）;
    -r 添加一个系统账号，若未指定gid，默认采用小于499的第一个数字
    -f 强制选项
    有关账号管理的差异请参见《从Linux到Solaris：系统管理》：http://bbs.chinaunix.net/thread-1003928-1-1.html


# 第三章：系统安装

## 1. 使用介质安装

    最常见的安装方法都是从介质进行安装（都可从官方网站下载ISO映像或是实体的CD/DVD安装盘）

    Linux的典型安装步骤包括：
        从介质启动（CD/DVD）
        硬盘分区
        选择软件包
        输入配置参数
    Solaris的典型安装步骤包括
        从介质启动
        输入系统配置参数
        选择软件包
        硬盘分区

## 2. 网络安装

    Linux从光盘引导后，要输入安装文件所在的服务器的URL，然后安装程序会下载所需的安装包并进行安装

    Solaris的网络安装程序叫做JumpStart，需要搭建JumpStart服务器，在JumpStart服务器上配置客户机的MAC地址，以及指定与之对应的IP地址，提供客户机的启动引导服务，并通过NFS共享提供安装介质，然后实现网络安装。Solaris支持跨网段进行网络安装，但需要提供一台dhcp服务器，并做适当配置

    具体可参见susbin的精华帖：
[JumpStart 安装 Solaris10 --- 用CD images设置安装服务器及一些新功能的应用](http://bbs.chinaunix.net/viewthread.php?tid=895305)

    详见docs.sun.com的文档：
    Solaris 10 安装指南：基于网络的安装.pdf
    Solaris 10 安装指南：自定义 JumpStart 和高级安装.pdf

## 3. Flash Archive

    Solaris还提供了一种克隆安装机制，把现有模版系统制作成归档文件（叫做 Flash Archive），然后可在JumpStart或标准安装程序中使用归档文件，达到克隆安装的目的，但源系统和目标系统的硬件架构必须一致，比如你不能用一台sun4u架构的solaris系统制作好Flash Archive，然后在sun4m架构的机器上安装solaris时使用这个来自sum4u的flash归档文件。

    详见docs.sun.com的文档：
    Solaris 10 安装指南：Solaris Flash 归档文件（创建和安装）.pdf

## 4. Live upgrade

    Live upgrade可以创建一个现有工作环境的备用版，然后在备用版中进行更新、升级等操作，而不会影响现有的工作环境。等升级操作完成后，再重新启动，使用备用版作为新的工作环境。

    使用Live upgrade可减少由于升级而导致的应用停止或宕机时间，而且一旦发现升级导致故障也可以很容易就回退到升级前的工作环境。

    详见docs.sun.com的文档：
    Solaris 10 安装指南：Solaris Live upgrade and upgrade planning.pdf

## 5. 查看系统版本

    1、查看内核64位还是32位
    solaris#isvinfo -b        ----------- or isainfo -kv   显示的信息更多些
        64

    redhat#getconf WORD_BIT
        32
    2、查看操作系统发布号

    通用：uname -a
    solaris#cat /etc/release
                               Solaris 10 11/06 s10s_u3wos_10 SPARC
               Copyright 2006 Sun Microsystems, Inc.  All Rights Reserved.
                            Use is subject to license terms.
                               Assembled 14 November 2006
    【注】11/06是发布的日/月，s10s_u3wos_10的"u3"是版本号，即solaris 10的update 3版本

    redhat#cat /etc/redhat_release      -------------- or /etc/issue    or  /proc/version
    Red Hat Enterprise Linux AS release 4 (Nahant Update 2)                  -------------AS4 update 2
    Red Hat Enterprise Linux Server release 5 (Tikanga)                  ------------------ES 5,没显示update即为初始版本


# 第四章：软件管理

## 1. 软件包管理

    redhat or suse linux使用Redhat Package Management(RPM)管理软件包，
    rpm -i安装，一般用rpm -ivh
    rpm -e卸载，
    rpm -qa 查看有哪些包
    rpm -qi 查看包的详细信息
    rpm -ql 查看包安装了哪些具体文件

    solaris使用System V软件包，添加为pkgadd命令，删除为pkgrm命令，查看用pkginfo，用pkgchk校验包。但solaris也可以支持rpm命令。
    pkgchk -p /etc/shadow 查看shadow文件自安装起的变化，如修改时间，文件大小，checksum等
    pkgchk -l -p /usr/bin/showrev  查看showrev属于哪个软件包
    pkgchk -v 查看包安装了哪些具体文件

## 2. 补丁包管理

    Linux系统不存在Solaris的补丁概念。Linux能够把指定的包（RPM）升级到下一个版本，它不象solaris那样有内建的机制能够在应用补丁后再进行回退（取消打补丁的操作）
    Solaris使用patchadd添加补丁，patchrm卸除补丁。补丁可从http://sunsolve.sun.colm获得，有些补丁需要有sun的技术支持服务才能下载，关键性的补丁通常都是免费提供。每个补丁包中都包含有应用此补丁以及回退的时需要做的变动的内容，因此可以回退。

## 3. 更新

    redhat系统可使用up2date命令管理所有的软件包升级所发生的版本变化。

    solaris有两个独立的命令管理升级操作
    GUI工具updatemanager
    命令行工具smpatch
    两个命令都可以实现对升级的管理


# 第五章：系统管理

    Linux和Solaris系统的管理任务非常相似，很多时候连使用的命令都是一样的。无论是Linux还是Solaris管理员，在转向另外一个系统管理时，原先所获得的那些管理经验绝大部分情况下依然可用。

## 1. 启动、关闭和运行级别

    运行级别的区别
    直到Solaris 9为止，solaris的启动步骤和Linux几乎没什么区别。二者都提供了运行级别（run level）的概念，每个运行级别都定义了哪些服务被启动和停止。都使用init命令在不同的运行级别间进行切换。

|运行级别|Linux|Solaris|Solaris10的里程碑|
|-------|-----|-------|--------------|
|0|halt（系统停止）|halt||
|1|单用户|s,S单用户|single-user|
|2|多用户（无网络）|多用户（无网络服务）|multi-user|
|3|多用户（文本）|多用户（带网络服务，默认级别）|multi-user-server|
|4|有些版本保留，有些同5|保留，未使用||
|5|多用户（图形，默认级别）|关闭系统(若机器支持还会自动关闭电源)||
|6|重启|重启||


    Solaris 10引入了SMF功能，运行级别被里程碑（milestone）所代替
    sysconfig
    devices
    single-user
    network
    name-services
    multi-user
    multi-user-server
    不过solaris10依旧保留了运行级别的概念以便和先前的Solaris版本兼容

    solaris还提供了另外两个命令，可改变系统的当前运行状态：
    reboot重启系统
    halt停止系统的运行
    但需注意的是，这些命令执行时，系统不会执行正常的关闭操作，不会停止服务，仅对进程进行简单的杀掉操作，卸除文件系统然后重启或停机。建议使用init命令或shutdown命令（两个系统的shutdown命令用法略有区别），而不是reboot or halt。

    shutdown命令的区别

    两个系统的shutdown命令默认都是进入单用户维护模式(init 1)
    Linux的shutdown选项：-r重启（init 6），-h停止系统(init 0)，-F重启后执行fsck操作，-f重启后不执行任何fsck操作（快速启动）
        shutdown now
        shutdown -h 10 "system will be shutdown to halt in10 minutes"
        shutdown -rF 5  "reboot and fsck"                重启后强制执行fsck操作
    Solaris的shutdown命令：shutdown  [-y]   [-i init_level]   [-g minutes]  [messages]
        shutdown -y -g 10 -i 5              10分钟后执行关机操作（init 5）
        solaris的shutdown命令可以切换到任一运行级别
        shutdown会在执行shutdown前 7200, 3600, 1800, 1200, 600, 300, 120, 60,30秒时重复发送消息给所有登进系统的用户

## 2. 系统服务

    服务的起停
    Linux系统中，除非服务是从inittab中以respawn属性运行，否则系统服务一旦被杀掉或非正常终止，就不会重新生成
    solaris 10中，由于SMF的存在，那些由SMF自动启动的系统服务，简单的kill操作对其无效。必须使用svcadm命令来禁用或启用这些服务。
    Solaris 10用SMF管理服务。
    如果要修改那些受inetd管理的服务，需要编辑/etc/inet/inetd.conf，
    然后执行inetconv命令在SMF中创建相应的服务条目，从而把这些服务转换成接受SMF管理。
    在solaris 10中，这类服务可以通过svcadm or inetadm命令进行管理
    Linux系统中，
    由xinetd守护进程控制服务。通常在以下几个位置存放服务的配置
        /etc/inittab 由init控制
        /etc/rc*.d 各个运行级别的专用脚本用以启动各种系统服务
        /etc/(x)inetd.conf 由inetd控制
        /etc/init.d、/etc/rc*d实际上都是链接到/etc/rc.d目录中的各同名子目录
    几种起停方式
        GUI的"系统设置"-->"服务器设置" ，CLI的# ntsysv
        # service service-name stop|restart
        # /sbin/chkconfig--level 345 service-name on|off
        常见的服务名：network,iptables,httpd,vsftpd...
    solaris中服务配置的位置
        /etc/inittab 由init控制，但solaris 10不推荐使用
        /etc/rc?.d,/etc/init.d
        /etc/inetd.conf 由inetd控制，Solaris 10中使用inetadm or SMF进行管理
        SMF 仅Solaris 10使用

## 3. 用户/组管理

    默认属性
    linux新增账号的默认属性存放在/etc/default/useradd文件中，修改这个文件立即生效
        # cat /etc/default/useradd
        # useradd defaults file
        GROUP=100
        HOME=/home
        INACTIVE=-1
        EXPIRE=
        SHELL=/bin/bash
        SKEL=/etc/skel
    Solaris没有默认的账号属性配置文件（但一些默认属性还是存在的），可以使用useradd -D命令生成。当第一次运行useradd -D时，会生成一个，/usr/sadm/defadduser文件。所有的缺省参数均保存在这个文件中。以后修改useradd命令的缺省参数只要修改这个文件即可。
        # useradd -D
        group=other,1   project=default,3  basedir=/home
        skel=/etc/skel  shell=/bin/sh      inactive=0
        expire=   auths=   profiles=   roles=   limitpriv=
        defaultpriv=  lock_after_retries=
    添加新账号
    linux系统添加新账号
    默认会根据缺省的basedir创建与账号同名的家目录
    同时还会创建和账号同名的组（-n选项可关闭这个功能），这样做的好处是如果某个账号A需要和其他账号共享自己的文件，只需把其他账号放置到与账号A同名的组内即可。
    如果同名组已存在，添加新账号时必须使用-g groupname选项指定或者用-n关闭自动建同名组的功能，否则会报错
    solaris不会自动创建用户家目录，必须在命令行中指定家目录并使用-m选项才会自动创建
    useradd -d home_directory -m username
    组管理
    Linux提供了组管理命令gpasswd，用法为gpasswd  [option]  group_name
    系统管理员可以用gpasswd -A指定组的管理员，-M指定组的成员，-r删除组密码，-R禁止通过newgrp命令切换成这个组身份
    组管理员使用gpasswd -a | -d添加和删除组成员。
    如果组没有设置密码，只有组成员可以用newgrp命令切换成这个组的成员身份
    如果组设置了密码，非组成员也可以使用这个组身份，但必须提供密码。

    此外
    Solaris扩展了useradd,groupadd命令，可配置和RBAC有关的属性，可参见《差异概述(click)》
    Solaris还提供了smuser,smgroup命令，可对名称服务器（比如NIS）上的账号和组进行管理。这两个命令是SMC(Solaris管理控制台)的一部分。SMC是solaris提供的图形化管理控制台，用来处理各种系统管理操作。运行smc命令将提供一个图形化的控制台界面，能够管理用户账号和组。


## 4. 打印管理

    绝大多数Linux系统都提供CUPS来处理它们的打印任务以及打印机管理。
    Solaris 10也包括了CUPS，能够和Linux系统兼容

    Solaris 10之前的版本采用的是System V的打印服务。

    system V的打印系统使用以下相关命令进行打印作业的管理
    lpadmin   修改打印系统参数
    lpsched   启动打印服务器(/usr/lib/lp/lpsched)
    lpshut      停止打印服务器
    cancel     取消打印作业
    lpmove   把打印作业转移到另外一个打印机
    lp           提交一个打印作业
    lpstart    查看打印机或打印作业的状态
    打印系统的配置保存在以下几个位置
    /etc/printers.conf文件
    NIS配置数据库库中的打印机地图文件
    $HOME/.printers
    $PRINTER and $LPDEST 环境变量
    Solaris 10中可运行printmgr命令进入图形化的打印机配置界面
    printmgr命令位于/usr/sadm/admin/bin/printmgr,/usr/sbin/printmgr仅是一个符号链接
    这个GUI界面类似于GNOME的打印机管理命令gnome-cups-manager（绝大部分Linux系统中都有这个命令）

## 5. 文件系统管理

    1.创建文件系统
    Linux使用mke2fs创建ext2文件系统,使用mkfs  -t  fs_type可创建ext2,ext3,xfs等文件系统
    mke2fs -j /dev/sdb2               在第三个SCSI硬盘的第二个主分区创建ext3文件系统
    mke2fs  /dev/sdb3                 在第三个SCSI硬盘的第三个主分区创建ext2文件系统
    mkfs -t ext2  /dev/sdb1          同上
    mkfs  -t  ext2 –j /dev/hda5  加上"-j"选项则创建ext3文件系统.
    Solaris使用newfs or mkfs -F fs_type创建文件系统，默认为ufs.
    newfs /dev/rdsk/c0t2d0s3           在0号控制器的第三个SCSI硬盘的第3个分片创建ufs文件系统
    mkfs -F ufs /dev/rdsk/c0t2d0s3   含义同上
    2.挂接文件系统(mount)
    Linux的Mount命令位于/bin目录，使用-t vfstype来指定文件系统类型.eg.   mount -t type ...
    # mount /dev/sdb1 /mnt/data -o ro       只读挂接
    # mount /mnt/data -o remount,rw         通过remount选项把挂接改为“可读写”
    solaris的Mount位于/usr/sbin目录，使用-F FSType来指定文件系统类型.eg. mount -F type ...
    # mount -o ro /dev/rdsk/c0t1d0s0 /mnt/data
    # mount -o remount,rw /mnt/data
    3.查看挂接列表
    Linux和solaris都可以运行不带选项的mount命令查看.还可以通过以下方式查看
    Linux# cat  /etc/mtab     or    cat /proc/mounts
    Solar# cat /etc/mnttab
    4.Linux支持的文件系统
    ext2,ext3,ext4
    ext2文件系统没有日志记录能力，且inode数量是固定的,mke2fs /dev/sdb1
    目前绝大部分Linux系统安装时默认使用ext3文件系统（但mke2fs,mkfs的缺省类型是ext2）,mke2fs -j /dev/sdb1
    data=writeback（禁用日志记录）
    data=orderd（缺省值，将元数据日志记录和数据与元数据一起写到磁盘）
    data=journal（用于数据和元数据完整性的完全数据日志记录，写性能降低一半）
    相关命令debugfs,tune2fs,chattr
    与性能有关的挂接选项：noatime,nodiratim
    ext4正在发展中
    reiserfs
    mkreiserfs /dev/sdb1
    与性能有关的挂接选项：noatime,nodiratime,notail
    xfs
    由SGI移植到Linux的企业级日志文件系统，http://oss.sgi.com/projects/xfs
    jfs
    由IBM移植到Linux的高性能日志文件系统
    vfat
    与DOS兼容的文件系统驱动程序，允许挂接基于DOS和Windows FAT的文件系统，并进行读写

    5.Solaris能够支持许多种的文件系统类型。
    能支持绝大多数存储介质比如CD,DVD,硬盘，软盘，U盘以及基于网络的文件系统协议。
    Solairs还为不同的文件系统提供接口功能，把一些内核信息输出成文件以便用户查看，比如/etc/mnttab.
    除了自身提供的文件系统支持外，还支持第三方软件厂商的文件系统，比如Veritas的vxfs文件系统。
    Solaris支持的文件系统列表：
        autofs
        cachefs
        ctfs
        devfs
        fd
        hsfs
        lofs
        mntfs
        nfs
        objfs
        pcfs
        proc
        qfs
        sam-fs
        tmpfs
        udfs
        ufs
        volfs
        xmemfs

## 6. 环回设备

    环回设备提供了一种机制，能够把磁盘映像挂接成文件系统。solaris的zone亦使用环回设备处理环回文件系统的挂接。
    Linux可以直接把映像文件挂接到指定的挂载点，挂接环回设备的命令大致如下,：
    mount -o loop /path/to/disk/image /mountpoint


    Solaris不能直接把映像文件直接挂接，需要使用lofiadm创建一个回环设备，然后再进行挂接。
    比如以下命令将创建一个回环设备/dev/lofi/X：
    lofiadm -a /path/to/disk/image

    然后把新创建的回环设备挂接：
    mount -F FSType /dev/lofi/X /mountpoint

    文件系统的类型必须根据映像文件的类型指定，比如如果映像文件是CD的ISO映像，则文件系统类型为hsfs

## 7. 文件系统配额

    常见命令
    Solaris独有的命令quot命令，查看solaris系统中每个用户的配额使用情况.
    其他命令如edquota,quota,quotaon,quotaoff,quotacheck,repquota则是两个系统共有，但选项和行为稍有不同。
    配置步骤
    系统启动时自动开启文件系统配额功能
    Redhat 9 Linux在/etc/fstab中类似条目（关键是红字的usrquota,grpquota表示挂接时开启这个文件系统的配额支持）
    /dev/sdb1               /mnt/sdb1               ext3    default,usrquota,grpquota               1 1
    # mount /mnt/sdb1
    Solaris在/etc/vfstab（关键是红字的rq表示开启配额）
    /dev/dsk/c1t0d0s3    /dev/rdsk/c1t0d0s3   /mnt/udata     ufs     2     yes      rq
    # mount /mnt/udata
    创建配额控制文件
    Redhat 9 Linux
    # touch /mnt/sdb1/quota.user
    # touch /mnt/sdb1/quota.group
    # quotacheck -mfugv
    a — 检查所有启用了配额的在本地挂载的文件系统
    v — 在检查配额过程中显示详细的状态信息
    u — 检查用户磁盘配额信息
    g — 检查组群磁盘配额信息
    c — 指定每个启用了配额的文件系统都应该创建配额文件
    f
    — 强制对已使用了配额功能的文件系统进行检查（不推荐）
    m—quotacheck在开始扫描前会尝试以只读方式remount文件系统，扫描结束后在remount成可读写方式，-m选项可让quotacheck不进行remount操作,
    M
    —类似于m选项，强行以读写方式进行扫描
    # convertquota -u /mnt/sdb1          转化用户配额控制文件quota.user的格式
    # convertquota -g /mnt/sdb1          转化组配额控制文件quota.group的格式
    Solaris
    # touch /mnt/udata/quotas
    # chmod 600 /mnt/udata/quotas
    开启配额功能(都使用quotaon命令,关闭都使用quotaoff命令)
    Redhat 9 Linux# quotaon -avug          为所有(-a)的已配置支持配额的文件系统开启用户(-u)和组(-g)的配额功能
    Solaris# quotaon /mnt/udata
    设置用户的配额
    Redhat 9 Linux和Solaris都使用edquota命令
        [root@redhat root]# edquota -u usertest
        Disk quotas for user usertest(uid 500):
        Filesystem   blocks       soft       hard     inodes     soft     hard
        /dev/sdb1         0      10000      12000          0    10000    12000
    设置文件系统配额的期限控制
    Redhat 9 Linux和Solaris都使用edquota -t命令
        [root@redhat root]# edquota -t
        Filesystem             Block grace period     Inode grace period
        /dev/hdc1                     7days                  7days
    查看用户的配额使用情况
    Redhat 9 Linux和Solaris都使用quota和repquota命令
        [root@redhat root]# repquota -a
        *** Report for user quotas on device /dev/sdb1
        Block grace time: 7days; Inode grace time: 7days
        Block limits                File limits
        User            used    soft    hard  grace    used  soft  hard  grace
        ----------------------------------------------------------------------
        root      --      20       0       0              5     0     0

        [root@redhat root]# quota -vu usertest
        Disk quotas for user usertest(uid 500):
        Filesystem  blocks   quota   limit   grace   files   quota   limit   grace
        /dev/sdb1        0   10000   12000               0   10000   12000

## 8. 磁盘和卷管理

    当新硬盘插入机器时
    如果可以识别，Linux会自动识别并使用新硬盘。
    而solaris即使可识别新硬盘，也必须运行devfsadm命令才可以使用新硬盘
    详见《从Linux到Solaris：设备管理》：http://bbs.chinaunix.net/thread-1004816-1-1.html


    磁盘管理
    Linux使用fdisk命令管理磁盘分区
    fdisk /dev/sda
    Linux fdisk常用指令：m帮助，p显示分区,n创建新分区，w保存
    注意：Ubuntu如果删除分区的话，分区号会前移，比如删除/dev/sda8分区，则后面的分区号会前移，比如/dev/sda9会变成/dev/sda8，需要修改相应的mount配置（估计其他的linux也是如此）
    solaris中与磁盘管理有关的命令主要有format和fdisk。fdisk（x86版的solaris才有）用来创建磁盘分区。

    format>fdisk  or fdisk /dev/rdsk/c0t0d0s2
    solaris x86 fdisk的指令使用和windows or dos的fdisk完全一样


    Linux文件系统的分区类型ID为0x83,而Linux的SWAP分区的类型ID为0x82
    Solaris分区的类型为0x82，和Linux的SWAP分区类型相同。
    如果x86的机器安装了双系统（solaris & Linux），这个关键点可能会导致系统启动故障

    Linux系统可支持3个主Linux分区，一个扩展分区，然后在扩展分区中创建多个Linux逻辑分区。这点和windows几乎完全一样.
    Solaris 仅使用单个分区(solaris 10 6/06之前的版本分区类型仅能为0x82)，然后在分区内通过Sun磁盘标签（disk label）把分区进一步划分成分片，分片操作由format命令的partition指令完成。

    自Solaris 10 6/06发布版开始，不再仅仅支持类型0x82,而是使用了一种新的类型0xbf(Solaris2 type)，但依然可以识别旧的0x82（Solaris type）,但默认采用solaris2类型（x86版本可通过fdisk工具把solaris2改成solaris分区）。一些旧的非Solaris的分区软件可能还无法识别这种新的分区0xbf.

    format命令用来把solaris的fdisk分区（x86系统）或者整个磁盘（sparc系统）划分成片(slice)。执行format命令时，solaris系统能认到的硬盘都会被列出，然后选定一个，使用partition指令进行划片操作


    逻辑卷管理
    Linux的卷管理使用pv*,vg*和lv*命令

    1、创建pv、vg、lv的简要步骤
        # fdisk /dev/sdb                          ----------LVM 2可直接使用linux分区，无需创建lvm分区
        # pvcreate /dev/sdb1                      ----------创建PV
        # vgcreate vgnru01 /dev/sdb1              ----------创建VG
        # vgchange -a y vgnru01                   ----------激活VG
        # vgdisplay                               ----------查看VG中的PE数量
        # lvcreate -l 174871 -n lvnru01 vgnru01   ----------把PE分配给lv，-L指定大小，-l指定PE数量
        # mkfs.ext3 /dev/vgnru01/lvnru01          ----------创建文件系统
        # vi /etc/fstab                           ----------启动时自动挂接文件系统
        /dev/vgnru01/lvnru01    /video    ext3    defaults    1 2
    2、查询用pvscan,vgscan,lvscan
    3、显示信息:pvdisplay,vgdisplay,lvdisplay
    4、逻辑卷的备份信息(backuped infomation):/etc/lvm/archive/ or /etc/lvm/backup/，可用来恢复逻辑卷

    Solaris使用元设备（metadevice）的概念进行卷管理，管理软件叫做SVM(solaris volume manager)，所有的相关管理命令为meta*, 可搜索论坛精华贴.

## 9. 网络管理

    ifconfig命令的区别
    Redhat 9 Linux新增网卡不用运行ifconfig plumb命令可直接配置
    solaris新增的网卡需先ifconfig interface plumb
    通用网络配置命令
    ifconfig  配置网络接口（配置ip地址，掩码等，Solaris需装配/卸除（plumb/unplumb）网卡才能执行其他操作，相当于redhat的ifup,ifdown命令）
    route     配置路由条目
    netstat   查看网络配置和网络连接状态信息
    ndd       查看或设置核心驱动的配置参数
    Linux的网络相关配置文件
    /etc/sysconfig/network-script/ifcfg-{interface_name}    网卡参数配置文件，包括IP地址，掩码，广播地址
    /etc/sysconfig/network          Redhat 9的主机名和配置缺省网关，对应solaris的/etc/nodename和/etc/defaultrouter
    /etc/hosts                          solaris的/etc/inet/ipnodes,/etc/hosts -> /etc/inet/hosts（链接到这个文件）
    /etc/networks                       solaris的/etc/networks -> /etc/inet/networks（链接到这个文件）
    /etc/netmasks                        Redhat 9同solaris
    /etc/nsswitch.conf                  Redhat 9同solaris
    /etc/resolv.conf                      Redhat 9同solaris

    Solaris的网络相关配置文件
    /etc/hostname.[interface_name]  比如hostname.hme0,hostname.e1000g0，每个网卡一个配置文件，仅用于配置网卡的IP地址
    /etc/nodename                      系统的主机名
    /etc/defaultrouter                   默认路由器（缺省网关）的地址
    /etc/hosts                              IP地址和主机名的对应表
    /etc/netmasks                        配置各种网络使用的默认掩码
    /etc/networks                        为网络配置易记忆的名字（可视为网络和网络名的对应表）
    /etc/dhcp.[interface_name]    网卡使用DHCP时，存放所获得的DHCP参数的配置文件
    /etc/resolv.conf                     DNS客户端的配置文件，配置DNS服务的IP地址，本机所属域，搜索域等
    /etc/nsswitch.conf                 名称服务选择文件，指定系统对各种不同的网络信息进行查询时分别使用哪些名称服务以及优先顺序。


    当修改完网卡配置文件和网关配置后，可用以下命令让其立即生效：

    solaris# svcadm restart network/physical
    redhat# service network restart

## 10. 远程管理

    Redhat 9采用xinetd
    telnet默认处于禁用状态。要开启它需要修改/etc/xinetd.d/telnet文件
    启用telnet：把最后一行改成：disable         = no
    启用tcp_wrapper：xinetd.conf自带tcp_wrapper功能，在telnet文件 or /etc/xinetd.conf中的相关配置（红字为新增）
    xinetd.conf：log_on_success        = HOST PID
    xinetd.conf：log_on_failure          = HOST
    telnet文件：log_on_failure  += USERID ATTEMPT
    然后重启xinetd进程
    # ps -ef | grep inetd         记下当前在运行的inetd进程及参数
    # kill inetd-PID
    # xinetd -stayalive -reuse -pidfile /var/run/xinetd.pid                      根据记录下来的命令行重启xinetd
    redhat的专用重启指令：service xinetd restart   类似于solaris 10的：svcadm restart inetd
    查看登录日志记录：
    # tail /var/log/secure
    Oct 26 16:20:01 redhat9 xinetd[2427]: START: telnet pid=2779 from=172.16.1.201
    Oct 26 16:20:48 redhat9 login: FAILED LOGIN 1 FROM 172.16.1.201 FOR root, Authentication failure

    新版的redhat enterprise版本可能使用kerberos5认证,要使用telnet可参考：
    http://linux.chinaunix.net/bbs/viewthread.php?tid=905516


    Solaris 9使用传统的inetd
    所有Inetd管理的服务均在/etc/inetd.conf文件中配置
    要禁用只需在配置条目前加#号注释掉即可
    要启用tcp_wrapper需要去http://www.sunfreeware.com下载软件包安装，然后修改/etc/inetd.conf文件的配置条目
    ftp     stream  tcp     nowait  root    /usr/sbin/tcpd  in.ftpd -l -a
    telnet  stream  tcp     nowait  root    /usr/sbin/tcpd  in.telnetd

    Solaris 10使用SMF

    内置telnet和ssh，默认处于启用状态。
    它保持了和传统的inetd运行机制的兼容性，但推荐采用以下操作：
    可通过SMF的svcadm or inetadm命令禁用telnet，ssh等服务
    但solaris默認禁止root賬號使用ssh遠程登錄，要允許的話，需修改/etc/ssh/sshd_config中添加PermitRootLogin yes
    自带tcp_wrapper功能，用inetadm or svccfg开启tcp_wrapper

    此外还可以使用GUI界面的smc来管理远程Solaris主机


    关于tcp_wrapper
    所有的支持tcp_wrapper的系统都可以支持/etc/hosts.allow和hosts.deny文件
    语法：daemon_list : client_list [ : shell_command ]
    hosts.deny的配置示例：in.telnetd: ALL EXCEPT LOCAL
    Solaris的相关日志/var/adm/messages
    Redhat的相关日志/var/log/secure

## 11. 内核管理

    修改核心参数
    两个操作系统对核心参数进行修改的操作有较大的区别。
    Linux系统中
    可能需要修改源，修改/proc中条目的运行时间，使用sysctl或加载内核模块
    Solaris系统中，
    可能需要修改/etc/system文件，加载内核模块，运行各种实用工具诸如ndd,DTrace or adb之类
    /etc/system是核心参数配置文件，在系统启动时由内核加载，这个文件中的设置会影响内核的行为，包括：
    分页(paging)
    交换(swapping)
    进程大小(process sizing)
    文件系统刷新(file system flushing)
    核心内存分配
    作业调度
    TCP/IP参数
    等等....
    也可以强制系统在启动时加载指定的模块或设备驱动程序，传递给设备驱动程序模块的参数也可以在/etc/system文件中设定。由于/etc/system文件仅在启动时读取一次，因此对这个文件所作的任何修改都必须重启后才能生效。

    加载核心模块

    内核模块（kernel module）是包含核心代码的二进制文件。内容通常包括：设备驱动程序、文件系统、系统调用，一些核心层的其他功能。

    这些模块有可能在OS运行期间进行加载和卸除操作。通过加载模块可修改核心功能。比如可通过加载一个（实现新文件系统）的核心模块来支持一种新的文件系统。

    Solaris中与核心模块操作有关的命令
    modload 加载核心模块的命令。
    modunload从当前正在运行的内核移除一个核心模块
    modinfo 查看系统当前加载了哪些模块
    Linux中与核心模块操作有关的命令
    modprobe
    insmod
    rmmod
    lsmod
    内核调整命令

    有些参数可通过命令进行调整，比如ndd，可修改网络接口的行为。使用ndd可以改变网络接口的配置，比如全双工或半双工；还可以改变Solaris实现TCP/IP协议栈的方式。
    adb和dtrace工具可以在系统运行时直接修改内核参数（不像/etc/system需要重启才能生效），这些修改将立即生效。因此使用时要非常小心，一旦出错会使内核产生致命的错误，导致系统崩溃。


    详见docs.sun.com：
    Solaris 可调参数参考手册.pdf

## 12. SWAP管理

    Linux的SWAP
    fdisk创建一个分区，然后用t指令把分区类型调整为Linux SWAP（0x82）
    格式化SWAP分区并检测坏块
    mkswap -c /dev/sdb2

    启用新的SWAP分区，从而扩大SWAP空间
    swapon /dev/sdb2

    查看SWAP分区
    cat /proc/swaps

    查看内存信息
    cat /proc/meminfo

    在自动挂接文件中指定使用新SWAP分区，/etc/fstab文件中的相关内容（红字为新增）：
    /dev/sda3               swap                    swap    defaults        0 0
    /dev/sdb2               swap                    swap    defaults        0 0
    删除SWAP分区
    swapoff    /dev/sdb2

    使用交换文件
    [root@redhat root]# dd if=/dev/zero of=/swapfile bs=1024 count=8192
    读入8192+0个块&cedil;每块大小为1024字节
    输出8192+0个块&cedil;
    [root@redhat root]# mkswap /swapfile 8192
    Setting up swapspace version 1, size = 8384 kB
    [root@redhat root]# sync
    [root@redhat root]# swapon /swapfile
    [root@redhat root]# cat /proc/swaps
    Filename                        Type            Size    Used    Priority
    /dev/sda3                       partition       1044216 0       -3
    /dev/sdb2                       partition       506036  0        -4
    /swapfile                         file               8184    0         -5


    Solaris的SWAP空间管理
    使用分片作为SWAP空间
    用format创建SWAP分片，然后编辑/etc/vfstab文件，加入以下条目
    # vi /etc/vfstab
    /dev/dsk/c1t0d0s1  -    -    swap    -    no    -
    除了编辑/etc/vfstab文件外，也可用swap命令临时在线添加一个分片作为交换空间使用。
    # swap –a /dev/dsk/c1t0d0s3
    使用swap files
        1.制造一个100m的文件
        # mkfile 100m/export/home/swapfile100m
        2.把上面新建的文件作为交换空间的一部分
        # swap -a/export/home/swap*
        3.验证新的交换空间情况
        # swap -s
        总数：分配了 226672k 字节 + 保留 32512k = 已使用 259184k，590284k 可用
        # swap -l
        4.保存结果
        编辑/etc/vfstab文件，加入以下条目，每次重启系统都会自动把交换文件作为交换空间使用
        /export/home/swapfile100m   －  －  swap  －  no  －
    删除swap空间
    # swap -d/dev/dsk/c1t1d0s1
    # swap -d/export/home/swap*
    # rm/export/home/swapfile100m

    注：必须注意swap -s和-l这两个选项的区别。-s列出所有的虚拟交换空间（包括物理内存部分），-l仅列出物理交换设备（交换分片或交换文件）

## 13. 网络共享NFS

    Linux使用传统的exportfs机制
    配置共享目录：输出目录配置在/etc/exports文件中，no_root_squash表示不把root账号映射成匿名账号（注2）

    /home     172.16.1.0/24(rw,sync,no_root_squash)    *(ro,sync)

    输出共享目录：运行exportfs命令把/etc/exports文件中的配置输出成NFS共享目录
    启动NFS服务：/etc/init.d/portmap start 和/etc/init.d/nfs start,如果修改了/etc/exports配置需重启nfs：service nfs restart
    查看共享目录：showmount -e     or     exportfs
    查看共享属性：cat /var/lib/nfs/etab

    Solaris采用较新的分布式文件系统机制
    输出目录配置在/etc/dfs/dfstab文件中,或者直接运行share命令，配置内容格式如下，anon=0表示不进行匿名账号映射（注2）：

    share -F nfs -o rw,anon=0 -d "home dirs" /export/home

    share -o ro/usr/share/man

    输出共享目录：share,shareall
    启动NFS服务：solaris9：/etc/init.d/nfs.server start        solaris10：svcadm enable svc:/network/nfs/server
    查看共享目录：showmount -e        or     dfshares
    查看共享属性：cat /etc/dfs/sharetab
    其他命令：dfmounts

    在不同系统间使用NFS时需要注意版本问题，比如Solaris10,redhat AS5默认使用NFS v4协议。
    Solaris 10的NFS客户端在尝试挂接NFS资源时，会先使用客户端能支持的最高版本，然后依次降低版本直至和服务器取得一致。比如Solaris 10的NFS客户机会先使用NFSv4，不成功就v3，然后v2
    从solaris挂接来自redhat的共享资源命令示例：

    Solaris 10# mount nfs://redhat/home /mnt/redhat


    注1：nfs客户端对共享资源的读写权限取决于三点：服务器端目录的权限、服务器端的共享权限，客户端挂接时采用的ro or rw选项
    注2：nfs的默认共享选项比较适合ro共享，要rw共享，建议采用uid,gid的一一映射而不是映射成匿名用户的uid和gid


# 第六章：设备管理

    随着USB设备、外置硬盘、数码相机以及其他移动设备的使用，对机器执行添加和删除设备的操作变得越来越频繁。本章主要讨论Linux和Solaris关于设备管理的区别点

## 1. 设备命名和访问

    两个系统的磁盘和TTY设备的名字有轻微差别。

    Linux系统中所有的设备文件都存放在/dev目录中.
    /dev目录是平面形，所有的设备节点都放置在这个目录下（不分级）
    TTY设备的名字为/dev/pty*.
    SCSI硬盘的设备文件名为/dev/sd[a-z],第一块SCSI硬盘为sda.IDE为/dev/hd[a-z]
    第一块IDE硬盘为hda
    硬盘分区的设备文件为/dev/sd[a-z]N,比如第一块SCSI硬盘的第一个分区为：/dev/sda0
    虽然Linux有一个专门的设备文件用来表示整个磁盘，但使用时通常把硬盘分区，然后对分区进行格式化（即创建文件系统），挂接等操作
    Solaris的/dev目录并不存放实际设备文件，它的/dev/目录中的设备文件仅是到/devices目录的符号链接
    solaris的/dev目录是分层的，按照设备的类型分成许多子目录，比如dsk,rdsk,pts,cua,rmt等
    solaris的TTY设备文件的名字为/dev/pts/*的格式，比如/dev/pts/0,/dev/pts/1
    solaris采用controller,target,device,slice来定位磁盘上的分区,比如/dev/dsk/cAtBdCsD,A是控制器编号,B是SCSI目标ID,C是LUN,D是分片号
    如果是IDE硬盘，表示为cxdxsx的形式.
    solaris通过分片来使用磁盘，最多可使用0-7个分片（0-7）。0表示第一个分片，1表示第2个分片，slice 2分片表示磁盘中所有的空间，3表示第3个分片，依次类推
    solaris不象Linux有一个相对独立的名字（比如hda,sda等）来专门表示整个磁盘。solaris中所有的磁盘设备都是指向磁盘的一个分片。分片2是一个特殊的例子，它与其他所有的分区重叠，它的空间从0号磁柱开始覆盖了整个磁盘，代表着整个磁盘的容量。

## 2. 添加/删除设备

    Linux用modload和modunload命令添加或删除设备。
    设备驱动必须已经被编译进集成的内核中，并且在启动时初始化这个静态内核。

    Solaris 8以及之前的版本用adddrv命令添加和删除设备。
    solaris 9开始，使用devfsadm命令。devfsadm -C可清除/dev中已经无效的条目。
    /devices目录树能够展示机器启动时在OBP状态所看到的设备树。

## 3. 可移除设备

    Linux对可移动介质的管理

    solaris通过卷管理器vold管理可移除设备，比如CD,DVD,软盘等。可用/etc/init.d/volmgt启动或停止vold守护进程
    当软盘插入软驱时，vold会自动把软盘挂接到/floppy目录，并创建两个设备：块设备/vol/dev/diskette0和裸设备/vol/dev/rdiskette0
    CD和DVD的处理与之类似，被自动挂接到/cdrom目录，并创建两个设备节点/vol/dev/dsk和/vol/dev/rdsk，分别用于提供块设备访问和字符设备访问

## 4. 磁带设备

    Solaris中，SCSI磁带以设备文件的形式存放在/dev/rmt目录中。
    设备文件的名字为/dev/rmt/N[lmhuc][bn]
    N表示设备编号，0表示第一个磁带
    lmhuc为磁带密度，分别是低/中/高/超高/压缩
    b表示支持BSD风格的行为。比如
    fsb：若在mt命令中指定fsb，表示将把磁带定位到前一个文件的结束点或当前文件的开始点
    fsf：表示定位到当前文件的结束点或下一个文件的开始点
    n表示不倒带

## 5. 终端/modem和串行端口

    Linux中管理连接在串行端口（serial port）上的终端或modem的典型命令有：
    minicom和seyon 管理端口
    setserial管理串行端口
    Solaris对串口的管理
    主要使用tip来管理串口连接，配置文件为/etc/remote和$HOME/.tiprc。
    串口的速度,奇偶校验(parity)和握手则通过eeprom命令设置

# 第七章：安全与加固

    安全啊安全，兄弟们，任何一台挂在网络上的系统都需要进行安全加固滴，否则后果自负！

    系统的安全性涉及许多方面，从初步加固到对补丁和漏洞进行审查等等，但根本目的都是为了保证数据的三性：完整性(一致性)、保密性、可用性。关于安全方面的内容你可能需要到专门的安全论坛获取专业资料，本文仅讨论一些管理员所必须知道的概念性的常识。

    内容包括：
    加固工具
    最低权限
    审计工具
    加固和删除服务
    内核安全性优化
    本文中涉及到安全加固的思想/思路同样也适用于其他的任何一种已联网的操作系统(虽然具体操作不同)，包括windows,linux,aix,hp-ux.....

    详见http://docs.sun.com的文档：《系统管理指南：安全**》

## 1. 加固工具

    通用工具
    TCP Wrappers
    著名的安全访问控制工具，可通过/etc/hosts.allow,hosts.deny对允许受访的服务以及来访的源进行控制，并记录源IP，账号名等登录成功/失败信息。Solaris 9 &10和Linux均已内置这个功能，Solaris中对应的软件包名为SUNWtcpd
    参见：从Linux到Solaris(5)：系统管理

    防火墙
    Linux常用iptable
    Solaris常用ipfilter，Solaris 10已内置这个软件包
    Solaris常见的一些加固工具
    Solaris Security Toolkit
    SST(Solaris安全性工具箱)的前身为JASS(Jumpstart Architecture and Security Scripts)。SST能够以一种自动的，可扩展的，可伸缩的机制维护一个安全的Solaris系统。JASS脚本可以审计并加固solaris.
    详见：http://www.sun.com/software/security/jass/
    Role Based Access Control
    RBAC实用工具可让管理员通过分配角色和特权给系统中的普通用户达到管理任务分散的目地，同时这些管理任务不再需要使用root账号从而降低了安全风险。
    详见：http://www.sun.com/bigadmin/feat ... east_privilege.html
    Basic Auditing and Reporting Tool
    BART(基本审计和报告工具)的设计目的是为了监视文件系统并查看文件的内容变换情况。BART能够查看系统中的文件并记录它们的信息从而得知文件是否有变化。
    BART的功能类似于商业版本的Tripwire软件（著名的文件完整性检查监控软件）的简化版
    详见bart的manpage.

## 2. 最低权限

    Unix系统中通常有大量进程需要以完全root权限运行，这些进程需要访问设备，修改进行，处理受限的文件，以及访问其他受限的资源等等
    如果所有的这些进程均以完全root权限运行会导致潜在的安全威胁。这些进程的任一个进程如果被攻击者控制，攻击者将获得对系统的完全root权限。
    Solaris 10设计了最低权限运行的策略。最低权限特性能够让管理员以满足一个应用程序正常运作的最低权限运行该应用，并把其他无需的特权全部删除。一个进程可设置的特权大约有50种。
    关于Solaris 10的最低权限功能及可用的特权说明，详见：
    http://www.sun.com/bigadmin/features/articles/least_privilege.html

## 3. 审计工具

    可以在Linux上使用的审计工具大部分也都有for Solaris的版本。比如
    tripwire 主要用于监视重要文件的变化
    http://www.sun.com/software/security/tripwire/
    crack and John the Ripper 主要用于检查弱密码问题（复杂性不够，长度不够等等）
    ftp://ftp.cert.dfn.de/pub/tools/password/Crack/
    http://www.openwall.com/john/

## 4. 加固和删除服务

    加固Linux或Solaris机器的步骤基本相同
    如果不想启动哪项服务，就把在/etc/rc*.d/目录下对应的文件删除（当然建议你先备份）
    如果是受inetd控制的服务，可把相关配置从Inetd的配置中删除或注释
    solaris 9的inetd配置文件为/etc/inetd.conf
    solaris10可直接用inetadm -d service_name or svcadm disable service_name命令禁用相应的服务

## 5. 内核安全性优化

    solaris有两种方法可防止堆栈缓冲溢出
    全局性设置：在/etc/system中设置noexec_user_stack.所有的应用将均采用这个选项的设置运行
    个别性设置：对单独某个应用，可将其和/usr/lib/ld/map.noexstk映射文件建立关联。
    关于/etc/system文件的noexec_user_stack选项的设置，详见：《Solaris 可调参数参考手册》
    http://docs.sun.com/app/docs/doc/817-0404/6mg74vs9h?a=view

# 第八章：性能监控

    性能监控对服务器管理员来讲至关重要，通常我们需要监控机器的CPU,内存,磁盘,以及网络流量等。主要包括：
    处理器
    内存
    网络
    磁盘,卷和文件系统
    系统和用户进程
    输入输出(I/O,input/output)

    如果需要更深入的了解瓶颈所在，可使用dtrace工具。OpenSolaris DTrace Community提供了一些专注于此方面的dtrace脚本，可在执行性能调优时使用。

## 1. 处理器监控

    1、solaris

    查看处理器状态，psrinfo显示处理器每个核(core)的状态信息

    -bash-3.00$ /usr/sbin/psrinfo
    0       on-line   since 10/13/2007 02:30:32
    1       on-line   since 10/13/2007 02:30:33
    2       on-line   since 10/13/2007 02:30:33
    ....
    查看处理器的详细信息

    -bash-3.00$ /usr/sbin/psrinfo -v
    Status of virtual processor 0 as of: 12/27/2007 09:39:37
      on-line since 10/13/2007 02:30:32.
      The sparcv9 processor operates at 1000 MHz,
            and has a sparcv9 floating point processor.
    Status of virtual processor 1 as of: 12/27/2007 09:39:37
      on-line since 10/13/2007 02:30:33.
      The sparcv9 processor operates at 1000 MHz,
            and has a sparcv9 floating point processor.
    ....

    来个中文的：
    bash-3.00$ /usr/sbin/psrinfo -v
    虚拟处理器 0 在下列时间的状态：12/27/2007 13:14:53
      自 12/27/2007 12:59:29 开始已在运行。
      i386 处理器以 1333 MHz 运行,
            而且有 i387 compatible 浮点数处理器

    查看每个处理器（同样以核为单位）的统计信息，用mpstat命令

    bash-3.00$ mpstat 2 5
    CPU minf mjf xcal  intr ithr  csw icsw migr smtx  srw syscl  usr sys  wt idl
      0 1058  21    0   309  224 1029   65    0    5    0  2825   16  35   0  49
      0    8   0    0   345  245  217    2    0    1    0    62    0   2   0  98
      0   10   0    0   343  244  209    1    0    1    0    64    0   2   0  98
      0    0   0    0   478  378  498   68    0   67    0    54    0  11   0  89
      0    0   0    0   346  245  208    1    0    1    0    60    0   1   0  99

    输出中，一般看最后四个字段：usr,sys,wt,idl。idl不低于30基本没事，sys如果常高于15需引起注意,wt是历史遗留字段对于solaris10而言总是0。其他几个常见字段：
    xcal   多个处理器间交叉调用的次数
    csw   处理器执行上下文交换的次数
    syscl 本处理器执行系统调用的次数

    注意：mpstat之类的按时间与次数的采样工具的输出第一行是自系统启动以来的汇总平均值统计

    此外，kstat命令也可以用来收集处理器的信息：

    bash-3.00$ kstat -m cpu
    module: cpu                             instance: 0
    name:   intrstat                        class:    misc
            crtime                          29.699815013
            level-1-count                   65517
            level-1-time                    991179530
            level-10-count                  163269
            level-10-time                   97182752330
            level-11-count                  0
            level-11-time                   0
            level-12-count                  2
            level-12-time                   238486
            ......

    2、redhat

    查看处理器状态，用dmesg从启动信息中查看处理器每个核(core)的状态信息

    [root@es4u5 ~]# dmesg | grep -i cpu
    Initializing CPU#0
    CPU: L1 I Cache: 64K (64 bytes/line), D cache 64K (64 bytes/line)
    CPU: L2 Cache: 512K (64 bytes/line)
    CPU: AMD Athlon(tm) 64 Processor 3200+ stepping 02
    ACPI: Processor [CPU0] (supports C1, 8 throttling states)
    Losing some ticks... checking if CPU frequency changed.[/fiont]

    查看处理器的详细信息
    [root@es4u5 ~]# cat /proc/cpuinfo
    processor       : 0
    vendor_id       : AuthenticAMD
    cpu family      : 15
    model           : 47
    model name      : AMD Athlon(tm) 64 Processor 3200+
    stepping        : 2
    cpu MHz         : 1329.309
    cache size      : 512 KB
    fpu             : yes
    fpu_exception   : yes
    cpuid level     : 1
    wp              : yes
    flags           : fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush mmx fxsr sse sse2 syscall nx mmxext lm 3dnowext 3dnow pni
    bogomips        : 2671.67
    TLB size        : 1088 4K pages
    clflush size    : 64
    cache_alignment : 64
    address sizes   : 36 bits physical, 48 bits virtual
    power management: ts fid vid ttp [4] [5]

## 2. 内存

    1、solaris

    solaris通常使用vmstat命令来查看系统的虚拟内存子系统的状态信息。vmstat可显示swap,物理内存,分页错误,磁盘信息统计和错误等信息.

    bash-3.00$ vmstat 2 3
    kthr      memory            page            disk          faults      cpu
    r b w   swap  free  re  mf pi po fr de sr f0 s0 s1 s2   in   sy   cs us sy id
    0 0 0 730888 228436 43 196 62  1  1  0 38  0  6  0  0  313  557  343  3 10 87
    0 0 0 731096 221556  3  25  0  0  0  0  0  0  0  0  0  341  111  208  0  3 97
    0 0 0 731092 221548  0   0  0  0  0  0  0  0  0  0  0  334   90  205  8  3 89
    【注意】
    swap为空余的swap空间(此处的swap为总的swap空间而不仅仅指swap分片的空间),free为空余的可用物理内存

    列出用于交换空间的硬盘分片或文件的使用情况：

    bash-3.00# swap -l
    交换文件             dev  swaplo blocks   free
    /dev/dsk/c1t0d0s3   54,3       8 1048568 1048568

    列出交换空间的总体使用情况：

    bash-3.00# swap -s
    总数：分配了 113024k 字节 + 保留 15340k = 已使用 128364k，730172k 可用

    查看内存分页的汇总情况

    bash-3.00# echo ::memstat | mdb -k
    Page Summary                Pages                MB  %Tot
    ------------     ----------------  ----------------  ----
    Kernel                      27734               108   22%
    Anon                        29908               116   23%
    Exec and libs                5862                22    5%
    Page cache                  12306                48   10%
    Free (cachelist)            18780                73   15%
    Free (freelist)             34320               134   27%

    Total                      128910               503

    此外还可以使用kstat查看内存的详细信息（以每个内存模块为单位）

    $ kstat -m vmem | more
    module: vmem                            instance: 1
    name:   heap                            class:    vmem
            alloc                           6254
            contains                        0
            contains_search                 0
            crtime                          0
            fail                            0
            free                            1200
            lookup                          113
            mem_import                      0
            mem_inuse                       86376448
            mem_total                       1646524366848
            populate_fail                   0
            populate_wait                   0
            search                          4381
            snaptime                        2441.858424006
            vmem_source                     0
            wait                            0
            ......

    2、redhat

    [root@es4u5 ~]# vmstat 2 3
    procs -----------memory---------- ---swap-- -----io---- --system-- ----cpu----
    r  b   swpd   free   buff  cache   si   so    bi    bo   in    cs us sy id wa
    0  0      0  16068  44764  99016    0    0    19     4 1015    31  0  3 96  0
    0  0      0  16068  44764  99016    0    0     0     0 1011    16  0  1 100  0
    0  0      0  16068  44764  99016    0    0     0     0 1012    19  0  1 99  0


    # free                                --------------列出内存的使用汇总情况
                 total       used       free     shared    buffers     cached
    Mem:        251016     235008      16008          0      44792      99032
    -/+ buffers/cache:      91184     159832
    Swap:       786424          0     786424

    # cat /proc/meminfo        --------------列出内存的详细状态信息

    MemTotal:       251016 kB
    MemFree:        134464 kB
    Buffers:         10988 kB
    Cached:          54648 kB
    SwapCached:          0 kB
    Active:          51288 kB
    Inactive:        35008 kB
    HighTotal:           0 kB
    HighFree:            0 kB
    LowTotal:       251016 kB
    LowFree:        134464 kB
    SwapTotal:      786424 kB
    SwapFree:       786424 kB
    Dirty:              44 kB
    Writeback:           0 kB
    Mapped:          32428 kB
    Slab:            19236 kB
    CommitLimit:    911932 kB
    Committed_AS:    90048 kB
    PageTables:       3704 kB
    VmallocTotal: 536870911 kB
    VmallocUsed:      1804 kB
    VmallocChunk: 536868343 kB
    HugePages_Total:     0
    HugePages_Free:      0
    Hugepagesize:     2048 kB

## 3. 网络负荷监控

    1、solaris

    最常见的都是用netstat命令，且redhat和solaris的常见使用方法基本一致。netstat命令可用来查看路由表、当前活跃的网络连接、各种网络数据结构、流内存统计，接口状态、DHCP等信息。常见的使用方式有：
    netstat -rn   看路由
    netstat -in   看流量统计
    netstat -an  看连接信息
    netstat -pn  看ARP解析表（MAC-IP映射表）


    此外，solaris可以使用kstat命令查看网络信息

    bash-3.00$ kstat -m e1000g | more                             模块(-m)可填网卡驱动类型，比如e1000g,e1000g0,bge,hme...
    module: e1000g                          instance: 0
    name:   e1000g0                         class:    net
            brdcstrcv                       0
            brdcstxmt                       0
            collisions                      0
            crtime                          43.023212228
            ierrors                         0
            ifspeed                         1000000000
            ipackets                        3421
            ipackets64                      3421
            ......

    2、redhat

    使用netstat命令，基本同solaris。区别在于-p选项
    redhat的netstat -pn:显示每个socket所属的程序名和进程ID

## 4. 磁盘,卷和文件系统

    1、solaris

    查看文件系统空间

    #df -h
    文件系统               大小   用了   可用 容量      挂接在
    /dev/dsk/c1t0d0s0      480M   278M   154M    65%    /
    /devices                 0K     0K     0K     0%    /devices
    ctfs                     0K     0K     0K     0%    /system/contract
    proc                     0K     0K     0K     0%    /proc
    ......

    查看文件系统类型

    # fstyp /dev/rdsk/c1t0d0s1
    ufs


    2、redhat

    redhat的df命令有个-T选项，可方便的查看文件系统类型

    # df -hT
    Filesystem    Type    Size  Used Avail Use% Mounted on
    /dev/mapper/VolGroup00-LogVol00
                  ext3   1008M  179M  779M  19% /
    /dev/sda1     ext3     99M   11M   84M  11% /boot
    none         tmpfs    123M     0  123M   0% /dev/shm
    /dev/mapper/VolGroup00-LogVol04
                  ext3    1.1G   34M 1013M   4% /home
    /dev/mapper/VolGroup00-LogVol02
                  ext3    4.0G  2.1G  1.8G  55% /usr
    /dev/mapper/VolGroup00-LogVol03
                  ext3   1008M   90M  868M  10% /var

    # fdisk -l                                --------------会列出系统认到的所有硬盘和U盘的分区信息

    Disk /dev/cciss/c0d0: 146.7 GB, 146778685440 bytes
    255 heads, 63 sectors/track, 17844 cylinders
    Units = cylinders of 16065 * 512 = 8225280 bytes

               Device Boot      Start         End      Blocks   Id  System
    /dev/cciss/c0d0p1   *           1         261     2096451   83  Linux
    /dev/cciss/c0d0p2             262        8094    62918572+  83  Linux
    /dev/cciss/c0d0p3            8095       12271    33551752+  82  Linux swap
    /dev/cciss/c0d0p4           12272       17844    44765122+   5  Extended
    /dev/cciss/c0d0p5           12272       14360    16779861   83  Linux
    /dev/cciss/c0d0p6           14361       16449    16779861   83  Linux
    /dev/cciss/c0d0p7           16450       16971     4192933+  83  Linux
    /dev/cciss/c0d0p8           16972       17493     4192933+  83  Linux

    solaris中类似的命令为prtvtoc，可列出指定磁盘的分区信息：

    # prtvtoc /dev/rdsk/c1t0d0s2
    * /dev/rdsk/c1t0d0s2 partition map
    *
    * Dimensions:
    *     512 bytes/sector
    *      32 sectors/track
    *     128 tracks/cylinder
    *    4096 sectors/cylinder
    *    4094 cylinders
    *    4092 accessible cylinders
    *
    * Flags:
    *   1: unmountable
    *  10: read-only
    *
    *                          First     Sector    Last
    * Partition  Tag  Flags    Sector     Count    Sector  Mount Directory
           0      2    00    1052672   1048576   2101247   /
           1      7    00    2101248   1273856   3375103   /var
           2      5    00          0  16760832  16760831
           3      3    01       4096   1048576   1052671
           5      0    00    3375104   1048576   4423679   /opt
           6      4    00    4423680  11288576  15712255   /usr
           7      8    00   15712256   1048576  16760831   /export/home
           8      1    01          0      4096      4095

## 5. 系统和用户进程

    1、solaris

    # prstat

    2、redhat

    # top

    Solaris没有自带top工具，如果要用top命令，需要到http://www.sunfreeware.com/ 下载对应版本的top工具包安装。

## 6. 输入输出（I/O）监控

    1、solaris

    # iostat 30 5             隔30秒采集一次,共采集5次

    输出的一行，照例还是汇总平均值

    2、redhat

    # vmstat 30 5

# 第九章：备份恢复

## 1. 转储
    linux使用dump和restore命令执行文件系统的备份和恢复作业，在solaris中则使用ufsdump和ufsrestore.
    linux
        #/sbin/dump -0u -f /dev/st0 /home
        # cd /home
        # restore rf /dev/st0
    solaris
        # ufsdump 0uf /dev/rmt/0 /export/home
        # cd /export/home
        # ufsrestore rf /dev/rmt/0

## 2. 快照

    solaris 10的zfs则使用快照功能：
    zfs snapshot tank/home@ss_monday

    上述命令将生成文件系统tank/home的快照ss_monday，然后通过快照进行备份：
    zfs send tank/home@ss_monday > /dev/rmt/0

    上述命令将把快照数据备份到磁带上。
    ufs也可使用快照功能
    # fssnap -F ufs -o raw,bs=/tmp/snapshot.0 /export/home
    # mount /dev/fssnap/0 /mnt
    # ufsdump -0uf /dev/rmt/0 /mnt

## 3. 打包

    linux和solaris还经常通过tar和cpio命令对文件系统进行备份
    此外还有许多第三方的备份工具，在两种操作系统上都可以使用，比如开源软件Amanda。比如著名的备份软件NBU(veritas netbackup)。
    solaris也发布了Sun StorageTek Enterprise Backup Software和其他一些与存储管理相关的工具。Sun同时也转售Legato Networker和IBM Tivoli。

    压缩工具

    常见的压缩工具有bzip,gzip,zip.

# 第十章：故障诊断

    正所谓天有不测风云，即使你按照各种技术指导文档努力的非常严谨且认真的按部就班的进行操作，依然会不时的遇到各种意外现象。本文将尽力帮助各位在遇到各种故障时进行分析与诊断。内容包括：
    安装
    系统启动
    Core Files
    内核崩溃转储
    日志
    命令丢失
    root密码恢复
    网络
    NFS共享
    诊断和调试工具

## 1. 安装

    从USB-CDROM安装
    即使x86机器支持从USB-CDROM启动，从USB-CDROM安装solaris 10 x86也并不见得总是能够顺利完成。有时候在安装时会出现系统找不到USB-CDROM的情况。解决方法如下：
    1、安装时选择交互式文本安装类型"Solaris Interactive Text (Console session)"
         按安装步骤操作直到安装程序出现以下错误提示，并跳到shell提示符：
                 ERROR:The disc you inserted is not a Solaris OS CD/DVD


    2、查看并记录/dev/dsk目录下可用的设备
    3、拔出USB-CDROM，等待数秒再次插入
    4、查看新认知的设备，比如c1t0d0XXXX.
    5、挂接USB-CDROM
             mount -F hsfs /dev/dsk/c1t0d0p0 /cdrom，注意设备名最后必须是p0(分区0)
    6、运行/sbin/install-solaris继续开始安装

## 2. 系统启动

    如果遇到和系统启动有关的问题，可以通过以下方法采集所需的信息：
    1、svcs -x FMRI
    solaris 10系统可通过svcs命令查看任何(未启动的)服务实例(solaris 10称之为FMRI,故障管理资源标识)的详细参考信息

    2、查看/var/adm/messages文件，通常情况下这个文件包含了所有的系统日志消息记录。
    3、确定/etc/rc*.d目录中的脚本没有错误

    修复引导文件
    如果在启动时遇到"panic:cannot mount boot archive"消息，说明可能丢失了引导文件（boot_archive）。在同时打多个需要重新引导文件的补丁时经常会出现这种故障。
    要修复这个问题，需要启动系统进入"failsafe"模式。通常情况下，failsafe模式会提示你更新引导文件。如果确实是这样的话，更新后重启系统即可恢复。

    还有另外一种常见的方法：
    把跟分区挂接到某目录，比如/a：mount -F ufs /dev/dsk/c1d0t0s0 /a
    然后用以下命令重建引导文件：/a/boot/solaris/bin/create_ramdisk -R /a
    重启系统

## 3. Core Files

    当系统异常退出时会创建Core file（核心文件）。Core File提供的信息能够帮助查找导致应用中断的原因。可通过coreadm工具来定义solaris所产生的core file的位置，名字，以及内容。
    有几个常用的应用可用来从core file中提取信息：
    pflags 查看"tracing flags"
    pcred 查看credential(证书)
    pldd 查看链接到进程的动态库
    pstack 查看每个进程的LWP的十六进制的"symbolic stack trace"
    "tracing flags","stack trace"是开发类术语，大家可google一下.

## 4. 内核崩溃转储

    如果Linux机器发生内核死机(kernel panic)，会生成一个oops消息，通过这个消息可判断导致故障的原因，linux内核并不会产生实际的崩溃转储文件(crash-dump-file)
    【注】redhat enterprise linux 5安装时可选择是否启动生成crash-dump-file.
    如果solaris内核死机，将会在/var/crash/hostname目录下生成一个崩溃转储文件。技术人员将通过这个文件分析系统崩溃的原因。

## 5. 日志

    当遇到问题时，首先查看各种相关日志文件是一个好习惯。有几个重要的系统日志文件如下：
    /var/adm/messages 主要的系统日志
    /var/log/syslog sendmail日志及其他
    /var/cron/log 自动作业(cron tab) 的日志信息
    /var/lp/logs/lpsched 打印机服务器日志
    此外，有些应用有专用的日志文件：
    /var/samba/log
    /var/apache/log
    /var/apache2/log

## 6. 命令丢失

    命令可能会在多个目录中存放，有些命令可能需要单独安装，甚至如果命令不可用可能需要重新移植（编译）
    首先要检查当前shell的路径变量$PATH。命令可能存放的目录请参见前文。
    如果命令还未安装，可通过配套CD,sunfreeware.com,blastwave.org下载提供该命令的软件包的开源版本，或用源码进行编译安装。

    如果该命令没有solaris版本，可能你需要进行移植或编写solaris版本

## 7. root密码恢复

    如果忘记了root密码或接手一台没人知道root密码的机器时，可从solaris的安装盘启动，选择进入单用户模式，然后把根分区挂接到/a目录，修改/a/etc/shadow文件，把root账号的密码字段清空，然后重启系统即可

## 8. 网络

    如果遇到网络连接问题，可使用以下常用工具进行调试诊断工作：
    traceroute 查看包在网络中的路由情况可或者那个跳点（路由器）出了问题
    /usr/sbin/ping 检测远程机器是否可达
    netstat 查看网络状态信息
    snoop 报文捕捉工具，类似于tcpdump
    tcpdump 有适用于各种操作系统的版本，solaris的配套CD上有提供

## 9. NFS共享问题

    有时候，在solaris 10上挂接从linux系统共享出来的文件系统时，可能会出现以下错误信息：
    $ mount linxu-nfs-server:/export /mnt
    nfs mount: mount: /mnt: not owner


    这是由于solaris 10NFS客户端默认使用NFS v4协议来挂接共享文件系统，而Linux的NFS不支持NFSv4
    要解决这个问题，可修改solaris 10的NFS 客户端默认使用的NFS协议：
    编辑/etc/default/nfs加入一行：
    NFS_CLIENT_VERSMAX=3

    另外，如果你不希望solaris nfs服务器使用NFSv4，可以修改/etc/default/nfs文件如下：
    NFS_SERVER_VERSMAX=3

    这样solaris系统的NFS服务器将最高只能支持到NFS协议的v3版本

## 10. 诊断和调试工具

    DTrace是solairs 10中新增的一个非常有力的工具，dtrace使进行系统故障诊断的过程变得更加容易。dtrace能够探测系统的各个方面，包括网络、IO、函数调用（function call），以及应用在CPU中的启动和停止.有许多地方可获取dtrace脚本，最常见的是openSolaris dtrace community:http://opensolaris.org/os/community/dtrace/ 。其中"dtrace toolkit"提供了许多专注于故障诊断的脚本。

    另外两个有用的工具是
    truss,用于追踪应用的系统调用情况
    apptrace，用户追踪应用的函数调用(function call)情况

