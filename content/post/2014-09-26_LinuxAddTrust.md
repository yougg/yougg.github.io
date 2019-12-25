---
Categories: []
Description: ""
Tags: []
title: "Linux/Unix添加信任关系脚本"
date: 2014-09-26T00:00:00+08:00
draft: false
---

```bash
#!/bin/bash

ip=$1
# usr=$2
pswd=$2

if [ -z "$ip" -o -z "$pswd" ]; then
	echo "Usage: `basename $0` <ip> <password>"
	exit 1
fi
[ -z "$usr" ] && usr=root


grep "$ip" known_ips && {
	echo "$ip is already added trust."
	exit 0
}

ping -c 1 -W 3 $ip >& /dev/null || {
	echo "can not connect to $ip ."
	exit 1
}

ssh $ip -o PreferredAuthentications=publickey -o StrictHostKeyChecking=no ":" && {
	echo "$ip is already added trust."
	exit 0
}

cd ~/.ssh/
[ -f "id_rsa.pub" ] || {
	echo -e '\n'
	echo -e '\n'
	echo -e '\n'
} | ssh-keygen -t rsa


expect -c "
set timeout 2;
spawn scp ${HOME}/.ssh/id_rsa.pub ${usr}@${ip}:~/.ssh/id_rsa.my
expect {
\"*yes/no*\" {send yes\n;}
\"*assword*\" {send $pswd\n;}
}
expect eof;"
[ $? -ne 0 ] && {
	echo "scp id_rsa.pub to $ip failed."
	exit 1
}

expect -c "
set timeout 2;
spawn ssh ${usr}@${ip} \"cd ~/.ssh/; cat id_rsa.my >> authorized_keys; rm id_rsa.my\"
expect {
\"*yes/no*\" {send yes\n;}
\"*assword*\" {send $pswd\n;}
}
expect eof;"
[ $? -ne 0 ] && {
	echo "add pub key into authorized_keys failed."
	exit 1
}

echo "$ip" >> known_ips

#echo "add trust for $ip success."
echo -e "\e[32;1;4;5madd trust for $ip success.\e[0m" 
```

