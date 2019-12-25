---
Categories: ["Document"]
Description: ""
Tags: ["Golang","Algorithm","Sort"]
title: "Go语言排序算法笔记"
date: 2018-02-13T10:50:47Z
draft: false
---

### 冒泡排序

- 排序实现

	给定一个N个元素的数组
	1. 比较相邻的一对元素(a,b)
	2. 如果元素乱序，交换元素(当前情况是a>b)
	3. 重复步骤1和2，直到到达数组的末尾(使用基于0的索引时，最后一对元素是第(N-2)和第(N-1)项)
	4. 至此，最大的元素将处于最后的位置
	5. 然后将N减1，并重复以上步骤直到N=1

- 复杂度计算

	比较和交换需要一个常数时间，称之为`c`  
	标准冒泡排序中有两个嵌套循环  
	外层循环运行N次迭代  
	内层循环运行迭代次数逐渐变少：  
		当i=0，迭代(N-1)次(比较和可能交换)，  
		当i=1，迭代(N-2)次，  
		...  
		当i=(N-2)时，迭代1次，  
		当i=(N-1)，迭代0次。  
	`总迭代次数` = (N-1)+(N-2)+...+1+0 = N*(N-1)/2  
	`总时间` = c*N*(N-1)/2 = O(N^2)

- 代码

	```go
	package main

	import (
		"math/rand"
		"testing"
		"time"
	)
	
	func initArray(a interface{}) (array []int) {
		var (
			n int
			b *testing.B
		)
		switch a.(type) {
		case int:
			n = a.(int)
		case *testing.B:
			b = a.(*testing.B)
			n = b.N
		}
		if n <= 0 {
			return
		}
		array = make([]int, n)
		for i := range array {
			array[n-i-1] = rand.Intn(n)
		}
		if nil != b {
			b.ResetTimer()
		}
		return
	}
	
	func bubbleSort(array []int) {
		lena := len(array)
		for i := 0; i < lena; i++ {
			for j := 0; j < lena-i-1; j++ {
				if array[j] > array[j+1] {
					array[j], array[j+1] = array[j+1], array[j]
				}
			}
		}
	}
	
	func bubbleSort1(array []int) {
		la := len(array)
		for swap := true; swap; {
			swap = false
			for j := 0; j < la-1; j++ {
				if array[j] > array[j+1] {
					array[j], array[j+1] = array[j+1], array[j]
					swap = true
				}
			}
		}
	}
	
	func bubbleSort2(array []int) {
		lena := len(array)
		for i := lena - 1; i >= 0; i-- {
			for j := 0; j < i; j++ {
				if array[j] > array[i] {
					array[j], array[i] = array[i], array[j]
				}
			}
		}
	}

	func bubbleSort3(array []int) {
		lena := len(array)
		for i := lena - 1; i >= 0; i-- {
			var head = lena - i - 1
			if head >= i {
				break
			}
			for j := head; j < i; j++ {
				if array[j] < array[head] {
					array[head], array[j] = array[j], array[head]
				}
				if array[j] > array[i] {
					array[i], array[j] = array[j], array[i]
				}
			}
		}
	}
	
	func TestBubbleSort(t *testing.T) {
		var length = 20
		array := initArray(length)
		bubbleSort(array)
		t.Log("bubble sort0:", array)
	
		init := time.Now().UnixNano()
		length = 20000
		array = initArray(length)
		before := time.Now().UnixNano()
		t.Log("bubble sort0 init:", before-init)
		bubbleSort(array)
		after := time.Now().UnixNano()
		t.Log("bubble sort0 time:", after-before)
	}
	
	func TestBubbleSort1(t *testing.T) {
		var length = 20
		array := initArray(length)
		bubbleSort1(array)
		t.Log("bubble sort1:", array)
	
		init := time.Now().UnixNano()
		length = 20000
		array = initArray(length)
		before := time.Now().UnixNano()
		t.Log("bubble sort1 init:", before-init)
		bubbleSort1(array)
		after := time.Now().UnixNano()
		t.Log("bubble sort1 time:", after-before)
	}
	
	func TestBubbleSort2(t *testing.T) {
		var length = 20
		array := initArray(length)
		bubbleSort2(array)
		t.Log("bubble sort2:", array)
	
		init := time.Now().UnixNano()
		length = 20000
		array = initArray(length)
		before := time.Now().UnixNano()
		t.Log("bubble sort2 init:", before-init)
		bubbleSort2(array)
		after := time.Now().UnixNano()
		t.Log("bubble sort2 time:", after-before)
	}

	func TestBubbleSort3(t *testing.T) {
		var length = 20
		array := initArray(length)
		t.Log("bubble sort3:", array)
		bubbleSort3(array)
		t.Log("bubble sort3:", array)
	
		init := time.Now().UnixNano()
		length = 20000
		array = initArray(length)
		before := time.Now().UnixNano()
		t.Log("bubble sort3 init:", before-init)
		bubbleSort3(array)
		after := time.Now().UnixNano()
		t.Log("bubble sort3 time:", after-before)
	}
	
	func BenchmarkBubbleSort(b *testing.B) {
		if b.N <= 1 {
			return
		}
		bubbleSort(initArray(b))
	}
	
	func BenchmarkBubbleSort1(b *testing.B) {
		if b.N <= 1 {
			return
		}
		bubbleSort1(initArray(b))
	}
	
	func BenchmarkBubbleSort2(b *testing.B) {
		if b.N <= 1 {
			return
		}
		bubbleSort1(initArray(b))
	}

	func BenchmarkBubbleSort3(b *testing.B) {
		if b.N <= 1 {
			return
		}
		bubbleSort3(initArray(b))
	}
	```

- 测试

	```bash
	# 单独执行Test
	go test -v bubblesort_test.go
	# 执行Test和Benchmark
	go test -v -bench . bubblesort_test.go
	# 单独执行Benchmark
	go test -v -run ^$ -bench . bubblesort_test.go
	```

---

_**参考链接：**_  

- [大O符号](https://zh.wikipedia.org/wiki/%E5%A4%A7O%E7%AC%A6%E5%8F%B7)
- [【数学基础】如何看懂算法中的各种求和公式的含义？](https://www.guokr.com/blog/473832/)
- [visualising data structures and algorithms through animation](https://visualgo.net)
- [常见排序算法总结](https://royalbluewa3.cc/posts/67d771c4)
