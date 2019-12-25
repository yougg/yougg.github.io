---
Categories: []
Description: ""
Tags: []
title: "Eclipse保存源码添加自动动作选项"
date: 2012-09-16T00:00:00+08:00
draft: false
---

Eclipse提供自动源码格式选项，并且组织输入（删除未使用的代码）。可以使用下面的这些快捷键进行操作。

`Ctrl + Shift + F`——源代码格式

`Ctrl + Shift + O`——组织输入并删除未使用的代码

代替手动调用这两个函数，只需根据Eclipse自动格式和自动组织选项，可以随时保存文件。

操作步骤，在Eclipse中进入`Window` -> `Preferences` -> `Java` -> `Editor` -> `Save Actions`，然后以选定的方式保存，最后检查Format source code + Organize imports。

> Additional actions:  
Remove 'this' qualifier for non static field accesses  
Remove 'this' qualifier for non static method accesses  
Convert control statement bodies to block  
Add final modifier to private fields  
Add final modifier to method parameters  
Add final modifier to local variables  
Remove unused imports  
Remove unused private methods  
Remove unused private constructors  
Remove unused private types  
Remove unused private fields  
Add missing '@Override' annotations  
Add missing '@Override' annotations to implementations of interface methods  
Add missing '@Deprecated' annotations  
Remove unnecessary casts  
Remove trailing white spaces on all lines

