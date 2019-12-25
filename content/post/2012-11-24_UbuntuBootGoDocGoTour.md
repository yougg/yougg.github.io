---
Categories: []
Description: ""
Tags: []
title: "Ubuntu下设置godoc/gotour开机自启动"
date: 2012-11-24T00:00:00+08:00
draft: false
---

编写godoc启动脚本 /etc/init.d/godoc , 内容如下:

```bash
#!/bin/bash
set -e

### BEGIN INIT INFO
# Provides:		godoc
# Required-Start:	$local_fs $remote_fs $network $time
# Required-Stop:	$local_fs $remote_fs $network $time
# Should-Start:		$syslog
# Should-Stop:		$syslog
# Default-Start:	2 3 4 5
# Default-Stop:		0 1 6
# Short-Description:	go online documention server
### END INIT INFO

# Setting environment variables for the postmaster here does not work; please
# set them in /etc/postgresql/<version>/<cluster>/environment instead.

[ -f /usr/local/go/bin/godoc ] || exit 0


case "$1" in
    start)
	nohup godoc -index -http=:6060 > /dev/null 2>&1 &
        ;;
    stop)
	pkill -9 godoc 2> /dev/null
        ;;
    status)
	ps aux | head -1
	ps aux | grep [6]060
	;;
    *)
        echo "Usage: $0 {start|stop|status}"
        exit 1
        ;;
esac

exit 0
```

设置脚本开机启动:

```bash
# 设置godoc在2,3,4,5系统启动等级,顺序77启动.  在0,1,6系统启动登记,顺序55停止.
update-rc.d godoc start 77 2 3 4 5 . stop 55 0 1 6 .
```

编写gotour启动脚本 /etc/init.d/gotour , 内容如下:

```bash
#!/bin/bash
set -e

### BEGIN INIT INFO
# Provides:		gotour
# Required-Start:	$local_fs $remote_fs $network $time
# Required-Stop:	$local_fs $remote_fs $network $time
# Should-Start:		$syslog
# Should-Stop:		$syslog
# Default-Start:	2 3 4 5
# Default-Stop:		0 1 6
# Short-Description:	go online tour server
### END INIT INFO

# Setting environment variables for the postmaster here does not work; please
# set them in /etc/postgresql/<version>/<cluster>/environment instead.

[ -f /usr/local/go/bin/gotour ] || exit 0


case "$1" in
    start)
	nohup gotour > /dev/null 2>&1 &
        ;;
    stop)
	pkill -9 gotour 2> /dev/null
        ;;
    status)
	ps aux | head -1
	ps aux | grep [t]our
	;;
    *)
        echo "Usage: $0 {start|stop|status}"
        exit 1
        ;;
esac

exit 0
```

设置脚本开机启动:

```bash
# 设置gotour在2,3,4,5系统启动等级,顺序77启动.  在0,1,6系统启动登记,顺序55停止.
update-rc.d gotour start 77 2 3 4 5 . stop 55 0 1 6 .
```

以上设置完成后, godoc与gotour将会在系统启动时自动启动,系统停止时自动杀死.

如果需要移除命令系统自启动.  
执行如下命令即可

```bash
update-rc.d -f godoc remove && rm -f /etc/init.d/godoc
update-rc.d -f gotour remove && rm -f /etc/init.d/gotour
```
