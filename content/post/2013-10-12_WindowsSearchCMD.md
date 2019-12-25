---
Categories: ["Document"]
Description: ""
Tags: ["CMD"]
title: "Windows系统中查找命令的路径"
date: 2013-10-12T00:00:00+08:00
draft: false
---

实现类似Linux中的which/type命令的效果

In CMD:

```cmd
where cmd
for %x in (powershell.exe) do @echo %~$PATH:x
```

In PowerShell:

```powershell
Get-Command where
```
