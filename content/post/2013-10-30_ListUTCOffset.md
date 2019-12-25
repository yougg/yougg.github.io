---
Categories: []
Description: ""
Tags: []
title: "获取当前操作系统所有时区对应UTC偏移量"
date: 2013-10-30T00:00:00+08:00
draft: false
---

```bash
#!/bin/bash

for tz in `awk '$1 !~/#/{print $3}' /usr/share/zoneinfo/zone.tab`; do
    export TZ="$tz"
    echo "$tz  `date +%:z`" >> tmptz
done

unset TZ

sort -k 1 tmptz > tzlist

rm -f tmptz
```
