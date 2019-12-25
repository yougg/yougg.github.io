---
Categories: ["Document"]
Description: ""
Tags: ["Golang","Interview"]
title: "Go语言面试题汇集"
date: 2017-12-19T10:59:00+08:00
draft: false
---


- goroutine 的调度顺序是随机的

	```go
	package main

	import (
		"fmt"
		"sync"
	)

	func main() {
		//runtime.GOMAXPROCS(1)
		//runtime.GOMAXPROCS(runtime.NumCPU())

		n := 10
		wg := new(sync.WaitGroup)
		wg.Add(n)
	
		for i := 0; i < n; i++ {
			i := i
			go func() {
				fmt.Printf("%d ", i)
				wg.Done()
			}()
		}

		wg.Wait()
		fmt.Println()
	}
	```

	以下4种执行方式的输出差别
	```bash
	GOMAXPROCS=1 go run main.go
	GOMAXPROCS=1 go run -race main.go
	GOMAXPROCS=10 go run main.go
	GOMAXPROCS=10 go run -race main.go
	```

- 这个代码的goroutine为什么会死锁？

	```go
	package main
	
	import (
		"fmt"
		"runtime"
		"time"
	)
	
	func f1() {
		for {
			//fmt.Println("11111")
			//time.Sleep(time.Second)
		}
	}
	
	func f2() {
		for {
			fmt.Println("22222")
			time.Sleep(time.Second)
		}
	}
	
	func main() {
		runtime.GOMAXPROCS(runtime.NumCPU())
		go f1()
		go f2()
	
		for {
			fmt.Println("main")
			time.Sleep(time.Second)
		}
	}
	```
	
	首先不是死锁，是goroutine没办法被调度出去goroutine是协作式调度，f1没办法把goroutine yield出去，f2是可以的fmt.Println和time.Sleep都会触发yield。sysmon虽然后台扫描长时间不交换出去的goroutine，标记为抢占调度。但是由于f1没有其他call frame存在，所以也没机会但是f1和f2不一定谁先执行，如果是f1，那么f2永远没机会了，但是如果是f2先执行，被调度出去以后f1执行，这时候定时器f2返回，没有空闲的pthread会创建一个busy loop f1，那么f2还有机会的。
	
	1. 当前运行的goroutine在达到调度点（系统调用、网络IO、等待channel等）的时候，pthread会挂起当前运行的goroutine，从runqueue中pop一个goroutine，重新设置当前pthread的执行上下文继续执行。
	2. 4个cpu，并不是一开始就启动多个pthread，而且sysmon定期检查进入系统调用的pthread，只有进入时间过长才会新建一个pthread。
	so，程序开始，f1和f2如果在同一个pthread的local runqueue中，f1先执行，没有调度点，也没有进行系统调用，触发不了调度和创建pthread，所以就卡住了。
	所以不应该在goroutine中使用没有function call的纯Infinite-loop，即使是纯数学运算也应该加入runtime.Gosched()来让当前goroutine放弃对CPU的控制权


1. Data Race问题怎么解决？能不能不加锁解决这个问题？
2. 使用goroutine以及channel设计TCP链接的消息收发，以及消息处理。
3. 使用go语言，编写并行计算的快速排序算法。

- 初级：一些语言语法，常用package的熟悉程度，数据类型slice，map相关知识，目的是考察下是不是熟悉这门语言，做过什么东西！能写代码就行！
- 中级：一些特性的底层原理，如goroutine是怎么一回事，调度器是什么样子，channel的机制是什么等等。遇到过什么坑，是否关注过GC，怎样解决的。
- 高级：什么场景让你选择Go作为开发语言，如和构建一个工程级别的Go项目，如何保证Go项目质量，目前有哪些你觉得Go解决不了的或者不适合的问题和场景，如何解决和避免，怎样看待以后Go的发展

---

_**参考链接：**_  

- [Go面试题答案与解析](https://yushuangqi.com/blog/2017/golang-mian-shi-ti-da-an-yujie-xi.html)
- [你遇到过哪些高质量的 Go 语言面试题？](https://www.zhihu.com/question/60952598)
- [如果你是一个Golang面试官，你会问哪些问题？](https://www.zhihu.com/question/67846139)
- [Go的50度灰：Golang新开发者要注意的陷阱和常见错误](http://colobu.com/2015/09/07/gotchas-and-common-mistakes-in-go-golang/)
- [Golang精编100题](https://www.jianshu.com/p/f690203ff168)
- [Go语言经典笔试题](https://goquiz.github.io/)
