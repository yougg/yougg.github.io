---
Categories: ["Code"]
Description: ""
Tags: ["Java"]
title: "交换两个变量的值的几种方式"
date: 2011-07-12T00:00:00+08:00
draft: false
---

```java
//精简,一行代码搞定
int x = 3, y = 7;
System.out.printf("x = %d, y = %d\n", x, y);
x = y + 0 * (y = x);
System.out.printf("x = %d, y = %d\n", x, y);

//兼容,支持Integer.MAXVALUE的+操作
int a = 5, b = 2;
System.out.printf("a = %d, b = %d\n", a, b);
a ^= b;
b ^= a;
a ^= b;
System.out.printf("a = %d, b = %d\n", a, b);

//明了
int p = 4, q = 9;
System.out.printf("p = %d, q = %d\n", p, q);
p = p + q;
q = p - q;
p = p - q;
System.out.printf("p = %d, q = %d\n", p, q);

//⊙﹏⊙b汗,产生中间变量浪费空间,但是大部分人是用这个吧...
int u = 8, v = 6;
System.out.printf("u = %d, v = %d\n", u, v);
int temp = u;
u = v;
v = temp;
System.out.printf("u = %d, v = %d\n", u, v);
```
