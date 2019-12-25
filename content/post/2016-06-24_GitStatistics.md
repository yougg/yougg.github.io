---
Categories: ["Programing"]
Description: ""
Tags: ["Shell","Git"]
title: "Git项目代码提交记录统计工具"
date: 2016-06-24T15:02:12+08:00
draft: false
---

### 使用说明

1. 本机下载安装Git版本管理工具
2. Clone项目代码仓库到本地
3. 下载Git项目代码提交记录统计工具保存到本地
4. 打开Git Bash, 执行gitstatistics.sh脚本(详细参数 -h 查看脚本帮助信息), 导出结果到 csv文件中
5. 使用Excel打开导出的csv文件,选中所有内容, 点击标签  `插入` ->  `柱形图` -> (二维柱形图)`簇状柱形图`
6. 查看柱形图右边显示标签为 提交数(Commits), 文件数(Files), 增加行(Insertions), 删除行(Deletions), 下方标签为人名
    如果右方和下方标签互换了, 需要选中柱形图, 点击标签  设计 -> 切换行/列
7. 选中柱形图, 点击标签  布局 -> 数据标签 -> 数据标签外
8. 拖动缩放柱形图, 选中复制即可粘贴到需要此图形的地方使用.

### 改进效果与推广价值

- 项目组使用此工具快速的生成开发周期性的数据图表，  
- 在项目进展周报等地方直观的展示开发人员的的工作量与效率，  
- 给管理人员提供项目计划的数据参考与反馈。

```bash
#!/bin/bash
# Statistics the code for all users
# Author: yougg

checkArgs() 
{
	if [ -z "$1" ] || `echo "$1" | grep -E '^-{1,2}(h|H|help)$' > /dev/null 2>&1`; then
		echo "Statistics commits,changed files,insertions,deletions (within date) for all users in git project"
		echo "Usage:"
		echo "./`basename $0` [all|<begin date> <end date>] </path/to/project>... [> /path/to/output.csv]"
		echo -e "\nEnvironment variables:"
		echo -e "\e[34;47mMaxChangeFiles\e[0m  # Max changed files in one commit, ignore the commit which changed files more than \${MaxChangeFiles}"
		echo -e "\e[34;47mMaxInsLines\e[0m     # Max insertion lines in one commit, ignore the commit which insertion lines more than \${MaxInsLines}"
		echo -e "\e[34;47mMaxDelLines\e[0m     # Max deletion lines in one commit, ignore the commit which deletion lines more than \${MaxDelLines}"
		echo -e "\nThe default env variables were set: MaxChangeFiles=50  MaxInsLines=500  MaxDelLines=500"
		echo "If you want to change the default value, execute following command before run script:"
		echo "Set MaxChangeFiles=0 or MaxInsLines=0 or MaxDelLines=0 to count all datas"
		echo -e "  \e[34;47mexport MaxChangeFiles=20 MaxInsLines=200 MaxDelLines=200\e[0m"
		echo -e "\nExample:"
		echo "Statistics commits in this week (default):"
		echo -e "  \e[34;47m./`basename $0` /path/to/project > project.csv\e[0m"
		echo -e "\nStatistics commits for multi projects:"
		echo -e "  \e[34;47m./`basename $0` /path/to/projectA /path/to/projectB /path/to/projectC\e[0m"
		echo -e "\nStatistics all commits history:"
		echo -e "  \e[34;47m./`basename $0` all /path/to/project\e[0m"
		echo -e "\nStatistics commits from begin date to end date:"
		echo -e "  \e[34;47m./`basename $0` 2016-04-11 2016-04-16 /path/to/project\e[0m"
		exit 1
	fi

	if [ "all" = "$1" ]; then
		since=""
		before=""
		shift
	else
		if [ -n "$1" ] && `echo "$1" | grep -E '^201[0-9]-(0[1-9]|1[0-2])-(0[1-9]|[1-2][0-9]|3[0-1])$' > /dev/null 2>&1`; then
			since="--since=${1}T00:00:00"
			shift
		fi
		if [ -n "$2" ] && `echo "$2" | grep -E '^201[0-9]-(0[1-9]|1[0-2])-(0[1-9]|[1-2][0-9]|3[0-1])$' > /dev/null 2>&1`; then
			before="--before=${2}T23:59:59"
			shift
		fi

		if [ -z "${since}" -a -z "${before}" ]; then
			since="--since=`date -d 'last-sunday 1 day' +%F`T00:00:00"
			before="--before=`date -d sunday +%F`T23:59:59"
		fi
	fi

	gitdir="$*"
	gitdir=(${gitdir})

	if ! `echo "${MaxChangeFiles}" | grep -E '^[0-9]+$' > /dev/null 2>&1`; then
		MaxChangeFiles=50
	fi
	if ! `echo "${MaxInsLines}" | grep -E '^[0-9]+$' > /dev/null 2>&1`; then
		MaxInsLines=500
	fi
	if ! `echo "${MaxDelLines}" | grep -E '^[0-9]+$' > /dev/null 2>&1`; then
		MaxDelLines=500
	fi
	if [ ${MaxChangeFiles} -eq 0 -o ${MaxInsLines} -eq 0 -o ${MaxDelLines} -eq 0 ]; then
		MaxChangeFiles=99999999
		MaxInsLines=${MaxChangeFiles}
		MaxDelLines=${MaxChangeFiles}
	fi
}

showlogs()
{
	local WD=${PWD}
	for d in ${gitdir[@]}; do
		cd ${WD}
		if [ ! -d "${d}/.git" ]; then
			continue
		fi
		cd ${d}
		# :(exclude)path/to/ignore
		# :!path/to/ignore
		git log --all --no-merges --shortstat --cherry-pick --pretty=format:"%aE" ${since} ${before}
	done
}

filteredlogs()
{
	showlogs | sed ':a;N;$!ba;s/\n / /g;s/\n\n/\n/g'
}

replacedlogs()
{
	chcp.com 65001 > /dev/null 2>&1
	filteredlogs | while read line; do
		e=$(echo ${line} | awk '{print $1}')
		u=$(echo ${e} | awk -F '@' '{print $1}')
		if echo "${u}" | grep -E '^[a-zA-Z]{1,3}[0-9]+$' > /dev/null 2>&1; then
			nu=$(net user ${u} //domain 2> /dev/null | awk '/Full Name/{print $3}')
			if [ -n "${nu}" ]; then
				echo ${line} | sed 's/'${e}'/'${nu}'/g'
			else
				echo ${line} | sed 's/'${e}'/'${u}'/g'
			fi
		else
			echo ${line} | sed 's/'${e}'/'${u//[0-9]/}'/g'
		fi
	done
}

main()
{
	checkArgs $*
	replacedlogs | awk -v maxfs=${MaxChangeFiles} -v maxins=${MaxInsLines} -v maxdels=${MaxDelLines} '{
		if ($2 <= maxfs && $5 <= maxins && $7 <= maxdels) {
			cs[$1]=cs[$1]+1;
			fs[$1]=(fs[$1]+$2);
			if (NF==8) {
				ins[$1]=(ins[$1]+$5);
				dels[$1]=(dels[$1]+$7);
			} else if ($6~/insertion/) {
				ins[$1]=(ins[$1]+$5);
			} else if ($6~/deletion/) {
				dels[$1]=(dels[$1]+$5);
			}
		}
	} END {
		if (length(cs) > 0) {
			if (ENVIRON["LANG"]~/zh/) {
				printf "%c%c%c,%c%c%c,%c%c%c,%c%c%c,%c%c%c\n",0xFEFF,0x7528,0x6237,0x63d0,0x4ea4,0x6570,0x6587,0x4ef6,0x6570,0x589e,0x52a0,0x884c,0x5220,0x9664,0x884c;
			} else {
				print "User,Commits,Files,Insertions,Deletions";
			}
			for (k in cs) {
				print k","cs[k]+0","fs[k]+0","ins[k]+0","dels[k]+0;
			}
		} else {
			print "No commit log found.";
		}
	}'
}

main $*
```

---

_**参考链接：**_  

- [Git代码提交记录统计](http://code.huawei.com/snippets/518)
