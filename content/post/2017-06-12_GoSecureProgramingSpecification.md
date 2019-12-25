---
Categories: ["Document"]
Description: ""
Tags: ["Golang","Secure"]
title: "《Go语言安全编程规范》"
date: 2017-06-12T00:00:00+08:00
draft: false
---

## 0 前言

### 背景

《Go语言安全编程规范》针对Go语言编程中的整数安全、输入校验、内存管理、异常行为、敏感数据、I/O操作、序列化等方面，描述可能导致安全漏洞或风险的常见编码错误。该规范基于业界最佳实践，参考业界安全编码规范相关著作，并总结了公司内部的编程实践。该规范旨在防止内存溢出、减少SQL/命令注入、防止敏感信息泄露、目录遍历等安全问题的发生。

### 使用对象

本规范的读者及使用对象主要为使用Go语言的研发人员和测试人员等。

### 适用范围

该规范适用于基于Go语言的产品开发。 除非有特别说明，所有的代码示例均适用于Go1.4+版本。

### 术语定义

【原则】：编程时必须遵守的指导思想  
【规则】：编程时必须遵守的约定  
【建议】：编程时加以考虑的约定  
【说明】：对原则/规则/建议进行的必要解释  
【错误示例】：对此原则/规则/建议从反面给出例子  
【推荐做法】：对此原则/规则/建议从正面给出例子  
【例外情况】：相应原则/规则/建议不适用的场景  

## 1 整数安全

### 规则 1.1 确保无符号整数运算时不会反转

- **说明**

    **反转** 是指无法用无符号整数表示的运算结果，这个结果将会根据该类型可以表示的最大值加1执行求模操作。  
    无符号整数的反转会发生在如下操作中：
    `+`、`-`、`*`、`/`、`%`、`++`、`--`、`=`、`+=`、`-=`、`*=`、`/=`、`%=`、`<<=`、`<<` 和 `–`  
    > 最后的`-`表示一元否定（unary negation）  

    将运算结果用于以下之一的用途，应防止反转：
    - 作为数组索引
    - 作为对象的长度或者大小
    - 作为数组的边界（如作为循环计数器）

- **错误示例**

    下面代码中a和b两者相加时会存在内存数量不足，导致产生无符号整数反转现象。

    ```go
    func main() {
        var a, b uint64 = math.MaxUint64, 1
        sum(a, b)
    }

    func sum(a, b uint64) (s uint64) {
        s = a + b
    }
    ```

- **推荐做法**

    在运算之前进行校验，确保无符号整数运算时不会出现反转。

    ```go
    func foo(a, b uint64) {
        var c uint64
        if math.MaxUint64-a < b {
            // error
        } else {
            c = a + b
        }
    }
    ```

### 规则 1.2 确保有符号整数运算时不会出现溢出

- **说明**

    **整数溢出**是一种未定义的行为，意味着编译器在处理有符号整数溢出时具有很多选择。  
    有符号整数的溢出会发生在如下操作中：  
    `+`、`-`、`*`、`/`、`%`、`++`、`--`、`=`、`+=`、`-=`、`*=`、`/=`、`%=`、`<<=`、`<<` 和 `–`

    > 【注】：最后的`-`表示一元否定（unary negation）

    将运算结果用于以下之一的用途，应防止溢出：
    - 作为数组索引
    - 作为对象的长度或者大小
    - 作为数组的边界（如作为循环计数器）

- **错误示例**

    下面代码中a和b两者相加时可能会产生有符号溢出。

    ```go
    func main() {
        var a, b int32 = math.MaxInt32, 1
        foo(a, b)
    }

    func foo(a, b int32) {
        c := a + b
        fmt.Println(c)    // output: -2147483648
    }
    ```

- **推荐做法**

    在有符号整数运算之前进行校验，确保不会产生溢出。

    ```go
    func foo(a, b int32) {
        var c int32
        if (a > 0 && b > (math.MaxInt32-a)) || (b < 0 && a < (math.MinInt32-b)) {
            // error
        } else {
            c = a + b
        }
    }
    ```

### 规则 1.3 确保整型转换时不会出现截断错误

- **说明**

    将一个较大整型转换为较小整型，并且该数的原值超出较小整型的表示范围，就会发生截断错误，原值的低位被保留而高位被丢弃。截断错误会引起数据丢失，甚至可能引发安全问题。特别是将运算结果用于以下用途：

    - 作为数组索引
    - 作为对象的长度或者大小
    - 作为数组的边界（如作为循环计数器）

- **错误示例**

    下面代码把它a强制转换为16位有符号整数时，会导致数据被截断。

    ```go
    func foo() {
        var a int32 = math.MaxInt32
        b := int16(a)
        fmt.Println(b)    // output: -1
    }
    ```

- **推荐做法**

    当不同数据类型强制转化时需要校验数据的范围，以确定是否会发生数据的丢失。

    ```go
    func foo() {
        var a int32 = math.MaxInt32
        var b int16

        if a < math.MinInt16 || a > math.MaxInt16 {
            // error
        } else {
            b = int16(a)
        }
    }
    ```

### 规则 1.4 确保整型转换时不会出现符号错误

- **说明**

    有时从带符号整型向无符号整型转换时，最高位会丧失作为符号位的功能，即产生符号丢失但数据不丢失的问题，从而数据失去原来的含义。

    - 带符号整型数的值非负时，它向无符号整型转换后，值不变；
    - 带符号整型数的值为负时，它向无符号整型转换后，结果通常是一个非常大的正数；

- **错误示例**

    下面代码假设a为32位有符号的最小整数，把它转换为32位无符号整数时，就会发生符号丢失现象。

    ```go
    func foo() {
        var a int32 = math.MinInt32
        b := uint32(a) // 【错误】产生符号丢失
        fmt.Println("a =", a,", b =", b)
    }
    // 输出：a = -2147483648, b = 2147483648
    ```

- **推荐做法**

    在将有符号数向无符号数转换前，进行数据校验。

    ```go
    func foo() {
        var a int32 = math.MinInt32
        var b uint32

        // 【修改】添加校验以确保不会发生符号错误
        if a < 0 {
            // 错误处理
        } else {
            b = uint32(a)
            fmt.Println("a=", a, ",b=", b)
        }
    }
    ```

## 2 输入校验

### 规则 2.1 确保对不可信的输入数据进行校验

- **说明**

    软件中最为普遍的缺陷就是对来自客户端或者外部环境的输入数据没有进行正确的合法性校验，这种缺陷可以导致几乎所有的程序弱点，例如SQL注入、命令注入等，因此不可信的数据都必须在使用前进行校验，这些不可信数据可能来自：

    - 用户输入
    - 外部调用的参数
    - 进程间的通信数据
    - 网络连接，甚至是一个安全的链接

    输入校验可能包括但不局限于如下内容：

    - 校验数据长度：数据长度校验会增加攻击者实施攻击的难度；
    - 校验数据范围：可以是数值范围也可以是集合范围。如果输入数据是数值，必须校验数值的范围是否正确，如年龄应该为0~150之间的正整数；如果输入数据是某个特定集合，必须校验实际输入是否在集合内，如性别应该是男、女；
    - 校验数据类型和格式：包括数据类型（整型、浮点型、字符型等）、也包括复杂的构造数据类型（Email的地址、IP、日期）或者自定义的数据格式等。对于数据类型和格式的校验，本文推荐使用正则表达式；
    - 输入校验建议采用 **白名单** 形式，尤其要注意一些特殊情况下的特殊字符；

- **推荐做法**

    下面代码假设获取客户端输入的Email地址，在使用之前对Email进行数据格式的合法性校验。

    ```go
    package main

    import (
        "flag"
        "regexp"
    )

    var (
        // 【注】：这里只是演示输入校验，所以Email的正则表达式可能会不太严谨
        pattern = `^\w+([-+.]\w+)*@\w+([-.]\w+)*\.\w+([-.]\w+)*$`
        reg     = regexp.MustCompile(pattern)
    )

    // 采用正则表达式的方式验证客户端输入Email参数是否合法
    func validate(email string) bool {
        return reg.MatchString(email)
    }

    func main() {
        var email string
        flag.StringVar(&email, "email", "", "accept email's value")
        flag.Parse()

        // 验证客户端输入Email是否合法
        if !validate(email) {
            return
        }
    }
    ```

### 规则 2.2 禁止直接使用不可信数据拼接SQL语句

- **说明**

    **SQL注入**是指原始SQL查询状态更改成一个与程序预期完全不同的查询，执行这样一个更改后的查询可能会导致信息被泄露或者数据被篡改。防止SQL注入的方式主要可以分为两类：

    - 使用预处理参数化查询
    - 对不可信数据进行校验

    预处理参数化查询是一种简单有效地防止SQL注入的措施，应该被优先考虑使用。另外，参数化查询还能提升数据库的访问性能，像SQL Server数据库和Oracle数据库会为其缓存一个查询计划，以便在多次重复执行相同的查询语句时重复使用。

- **错误示例**

    下面代码通过Id和密码去数据库中查询用户信息。  
    由于直接拼接SQL语句，给攻击者提供SQL注入途径。  
    当攻击者输入`"aaa"`, `"xxx' or '1'='1"`，SQL注入便发生了。

    ```go
    package main

    import (
        "database/sql"
        "fmt"
        _ "github.com/go-sql-driver/mysql"
    )

    func queryById(db *sql.DB, id, pwd string) string {
        // 【错误】直接拼接SQL语句
        var sqlStr = "SELECT Name, Dept FROM USER WHERE Id = '" + id + "' AND Pwd = '" + pwd + "'"
        rows, err := db.Query(sqlStr)
        if err != nil {
            fmt.Println("Query USER table error.", err)
            return ""
        }

        var result string
        for rows.Next() {
            var name, dept string
            rows.Scan(&name, &dept)
            result += name + " : " + dept + "\r\n"
        }
        return result
    }

    func main() {
        // 【注】：在真正版本中数据库用户名和密码应该从配置中读取，不能硬编码
        db, err := sql.Open("mysql", "root:root@tcp(localhost:3306)/demo?charset=utf8") 
        if err != nil {
            fmt.Println("Connect DB error.")
            return
        }
        defer db.Close()

        var result string = queryById(db, "aaa", "xxx' or '1'='1")
        fmt.Println(result)
    }
    ```

- **推荐做法**

    使用参数化查询，通过占位符表示需在运行时确定的参数值，这使得SQL查询的语义逻辑被预先定义，而实际的查询参数值则等到程序运行时再确定。参数化查询使得数据库能够区分SQL语句中语义逻辑和数据参数，以确保用户输入无法改变预期的SQL查询语义逻辑。

    ```go
    package main

    import (
        "database/sql"
        "fmt"
        _ "github.com/go-sql-driver/mysql"
    )

    func queryById(db *sql.DB, id, pwd string) string {
        // 【修改】采用参数化查询
        var sqlStr = "SELECT Name, Dept FROM USER WHERE Id = ? AND Pwd = ?"
        rows, err := db.Query(sqlStr, id, pwd)
        if err != nil {
            fmt.Println("Query USER table error.", err)
            return ""
        }
    }
    ```

### 规则 2.3 禁止调用OS命令解析器或运行程序防止命令注入

- **说明**

    使用未经校验的不可信输入作为系统命令的参数或命令的一部分，可能会导致命令注入漏洞。对于命令注入漏洞，命令将会以与Go应用程序相同的特权级别执行，它向攻击者提供了类似系统shell的功能。

    在Go中，os/exec经常被用来调用一个新的进程，如果被执行的命令来自于外部不可信输入，则可能会产生命令和参数注入。执行命令的时候，需要注意以下几点：

    - 命令执行的字符串不要去拼接输入的参数，如果必须拼接时，则需要对输入参数进行白名单过滤；
    - 对传入的参数要做类型校验。例如整数数据，需要对数据进行整数强制转换；
    - 保证格式化字符串的正确性。例如int类型参数的拼接，对于参数要用%d，不能用%s；

- **错误示例**

    下面代码通过Go语言的exec.Command()来执行Linux命令，exec.Command()函数本身不会对参数进行校验，程序员也没有对参数做合法性校验，再加上Linux的&&符号可以使多个命令连续执行，所以当输入`rm -f /opt/pwm/limit.txt && touch /opt/pwm/attack.txt`，便发生了命令注入。

    ```go
    package main

    import (
        "fmt"
        "os/exec"
    )

    // 删除指定的文件
    func delFile(param string) {
        // 【错误】允许调用OS命令解析器，也没有对入参做合法性校验
        cmd := exec.Command("/bin/sh", "-c", param)
        err := cmd.Run()
        if err != nil {
            fmt.Println("delete file error.", err)
        }
    }

    func main() {
        delFile("rm -f /opt/pwm/limit.txt && touch /opt/pwm/attack.txt")
    }
    ```

- **推荐做法 一**

    规范建议禁止调用OS命令解析器，推荐使用其它标准API替代，从根本上消除发生命令注入和参数注入的可能。针对本例而言，它是用来删除指定文件的，所以可使用Go语言的`os.Remove()`来替代。

    ```go
    package main

    import (
        "fmt"
        "os"
    )

    // 删除指定的文件
    func delFile(param string) {
        err := os.Remove(param) //【修改】使用Go的os.Remove()删除文件
        if err != nil {
            fmt.Println("delete file error.", err)
        }
    }
    ```

- **推荐做法 二**

    出于特殊原因非得使用OS命令解析器，则需要使`exec.Command()`功能单一，同时对输入数据进行检查和净化。

    ```go
    package main

    import (
         "fmt"
         "os"
         "os/exec"
    )

    func delFile(param string) {
        // 【修改】对输入参数进行合法性校验
        _, err := os.Stat(param)
        if err != nil || os.IsNotExist(err) {
            fmt.Println("The file not exist or format error.")
            return
        }

        // 【修改】使exec.Command()功能单一
        cmd := exec.Command("rm", "-f", param)
        delErr := cmd.Run()
        if delErr != nil {
            fmt.Println("delete file error.")
        }
    }
    ```

### 建议 2.4 慎用slice作为函数入参

- **说明**

    函数入参默认是值传递，一般对入参的修改处理不会影响到调用者，但是slice是引用类型，它在作为函数入参时采用的是地址传递，考虑到这种特殊性，建议慎用slice作为函数的入参。

- **错误示例**

    下面代码直接对函数的slice类型入参进行处理，调用者期望入参是不受任何影响的，但实际上经过处理后，slice的值已发生变化。

    ```go
    package main

    import (
        "fmt"
    )

    // slice作为函数入参时是地址传递
    func modify(array []int) {
        array[0] = 10 // 【错误】对入参slice的元素修改会影响到调用者
    }

    func main() {
        array := []int{1, 2, 3, 4, 5}

        modify(array)
        fmt.Println(array) //【输出】：[10 2 3 4 5]
    }
    ```

- **推荐做法**

    如果要实现数组的值传递，需要将函数入参明确为数组类型，而非slice类型。

    ```go
    package main

    import (
        "fmt"
    )

    // 【修改】： 数组作为函数入参时是值传递 
    func modify(array [5]int) {
        array[0] = 10
    }

    func main() {
        // 【修改】传入数组，注意数组与slice的区别
        array := [5]int{1, 2, 3, 4, 5}

        modify(array)
        fmt.Println(array)
    }
    ```

## 3 内存管理

### 规则3.1 确保对slice/map的大小进行合法性校验

- **说明**

    使用make声明slice或者map时，如果指定slice或者map大小的整数值是由外部输入的，则必须对输入的整数值进行合法性校验，以防止输入的是负值或者超出整数最大值，这样可能会造成系统退出或者系统崩溃。

- **错误示例**

    下面代码中slice的大小是由入参决定的，程序中也没有对该入参进行合法性校验，存在系统退出的可能。

    ```go
    func foo(size int) error {
        var s = make([]int, size) // 【错误】size值可能为负数或超过整数最大值，存在程序异常退出可能
    }
    ```

- **推荐做法**

    对外部输入的整数值进行合法性校验。

    ```go
    func foo(size int) error{
        if size < 0 || size > math.MaxInt32 { // 【修改】对入参做合法性校验
            return errors.New("size out of range")
        }
        var s = make([]int, size)
    }
    ```

### 规则3.2 禁止SetFinalizer和指针循环引用同时使用防止内存泄露

- **说明**

    `runtime.SetFinalizer()`用于把对象从内存移除时执行的一些特殊操作。当一个对象从被GC选中到移除内存之前，`runtime.SetFinalizer()`都不会执行，即使程序正常结束或者发生错误。由指针构成的“循环引用”虽然能被GC正确处理，但由于无法确定Finalizer依赖顺序，从而无法调用`runtime.SetFinalizer()`，导致目标对象无法变成可达状态，造成内存无法被回收。

- **错误示例**

    ```go
    package main

    import (
        "fmt"
        "runtime"
        "time"
    )

    type Data struct {
        d [1024 * 100]byte
        o *Data
    }

    // 【错误】循环指针引用和SetFinalizer()同时使用
    func foo() {
        var a, b Data
        a.o = &b
        b.o = &a

        runtime.SetFinalizer(&a, func(d *Data) {
            fmt.Printf("a %p final.\n", d)
        })
        runtime.SetFinalizer(&b, func(d *Data) {
            fmt.Printf("b %p final.\n", d)
        })
    }

    func main() {
        for {
            foo()
            time.Sleep(time.Millisecond)
        }
    }
    ```

    通过跟踪GC的处理过程，可以看到如上代码内存在不断的泄露

    `go build -gcflags "-N -l" && GODEBUG="gctrace=1" ./foo`
    > gc11(1): 2+0+0 ms, 104 -> 104 MB **1127 -> 1127** (1180-53) objects  
    gc12(1): 4+0+0 ms, 208 -> 208 MB **2151 -> 2151** (2226-75) objects  
    gc13(1): 8+0+1 ms, 416 -> 416 MB **4198 -> 4198** (4307-109) objects

    上面结果粗体数字表示对象数量。我们在代码中申请的对象都是局部变量，在正常处理过程中GC会持续地回收局部变量占用的内存，但是在当前的处理过程中，内存无法被回收，目标对象变成不可达状态。

- **推荐做法**

    需要避免`SetFinalizer()`和指针循环引用的同时使用，以防止内存泄露。

### 规则3.3 禁止重复释放channel

- **说明**

    重复释放一般存在于异常流程判断中，如果恶意攻击者构造出异常条件使程序重复释放channel，则会触发运行时恐慌，从而造成DoS攻击。

- **错误示例**

    下面代码中多次关掉channel会触发运行时错误。

    ```go
    func foo(c chan int) {
        defer close(c)
        err := processBusiness()
        if err != nil {
            c <- 0
            close(c) // 【错误】重复释放channel
            return
        }
        c <- 1
    }
    ```

- **推荐做法**

    使用defer延迟关闭channel，并且确保channel只释放一次。

    ```go
    func foo(c chan int) {
        defer close(c) // 【修改】使用defer延迟关闭channel
        err := processBusiness()
        if err != nil {
            c <- 0
            return
        }
        c <- 1
    }
    ```

### 规则3.4 确保每个协程都能退出

- **说明**

    协程`Goroutine`是Go语言并行设计的核心，启动一个协程就会做一个入栈操作，在系统不退出的情况下，协程也没有设置退出条件，则相当于协程失去了控制，它占用的资源无法回收，可能会导致内存泄露。

- **错误示例**

    下面代码启动了两个协程，每个协程都是循环向屏幕上打印信息，在main()不退出的情况，且协程也没有设置退出条件，则导致协程所占用的资源以及启动协程的栈信息无法得到释放。

    ```go
    package main

    import (
        "fmt"
        "time"
    )

    // 【错误】协程没有设置退出条件
    func doWaiter(name string, second int) {
        for {
            time.Sleep(time.Duration(second) * time.Second)
            fmt.Println(name, " is ready!")
        }
    }

    func main() {
        go doWaiter("Tea", 2)
        go doWaiter("Coffee", 1)

        fmt.Println("main() is waiting....")
        time.Sleep(5 * time.Second)
    }
    ```

- **推荐做法**

    通过channel机制对每个协程都设置退出条件，从而达到回收资源的目的，其中channel是一个消息队列通道。

    ```go
    package main

    import (
        "fmt"
        "time"
    )

    // 【修改】为每个协程增加一个channel，用来控制退出
    func doWaiter(name string, second int, signal chan int) {
        for {
            select {
            case <-time.Tick(time.Duration(second) * time.Second):
                 fmt.Println(name, " is ready!")
            case <-signal:
                 fmt.Println(name, " close goroutine.")
                 return
            }
        }
    }

    func main() {
        var signal1 = make(chan int) // 【修改】增加两个channel
        var signal2 = make(chan int)

        // 【修改】关闭channel
        defer close(signal1)
        defer close(signal2)

        go doWaiter("Tea", 2, signal1)
        go doWaiter("Coffee", 1, signal2)

        fmt.Println("main() is waiting....")
        time.Sleep(4 * time.Second)

        // 【修改】设置退出条件
        signal1 <- 1
        signal2 <- 1
        time.Sleep(time.Second)
    }
    ```

    > 上面代码在某些时候可能无法达到预期的执行效果，比如在main()退出的情况，协程所占用的资源以及启动协程的栈信息一起释放，这时协程还未全部执行完成，输出非预期结果。修改等待时间为：`2*time.Second`、`time.Second`或`0`，已经入栈的协程部分执行或者没有执行。利用休眠函数延长程序运行时间的做法，在不同运行环境和系统时间下，可能输出非预期结果，推荐示例见[《规则 5.2 确保并发安全》](#_规则_5.2_确保并发安全)。

## 4 异常行为

### 规则 4.1 禁止在异常中泄露敏感信息

- **说明**

    敏感信息的范围应该基于应用场景、产品威胁分析的结果来确定。典型的敏感数据包括密码、银行账号、个人信息、通讯记录、密钥等。如果在传递异常的时候没有对敏感信息进行过滤，则有可能会导致敏感信息泄露，这有可能帮助攻击者尝试发起进一步的攻击，攻击者可以通过构造恶意的输入参数来发掘应用的内部结构和机制。

    不管是异常中的文本消息，还是异常本身的类型都可能泄露敏感信息。例如error中的异常消息会透露文件系统的结构信息，而通过异常本身的类型，可以得知所请求的文件不存在。因此，当异常会被传递到信任边界以外时，必须同时对敏感的异常消息和敏感的异常类型进行过滤。

- **错误示例**

    下面代码尝试打开一个指定的文件，当文件不存在时，返回的异常中则说明文件不存在，反之，不抛异常说明文件是存在的，这样攻击者可以尝试多次传入所有可能的文件名，来发现系统中的有效文件，从而发动攻击。

    ```go
    package main

    import (
        "fmt"
        "os"
    )

    func main() {
        _, err := os.Open(os.Args[1])
        if err != nil {
            // 【错误】异常没有封装，把所有异常信息都抛出
            fmt.Println(err.Error())
            return
        }
    }
    ```

- **推荐做法**

    下面代码不仅封装了错误信息，而且对要打开的文件也进行了初步的路径校验。这样攻击者无法发现指定目录以外的任何文件，同时从错误信息中也无法获取有助于攻击的信息。

    ```go
    package main

    import (
        "fmt"
        "os"
        "path/filepath"
        "regexp"
    )

    // 【修改】对文件路径标准化后判断是否在安全目录下
    func validate(path string) bool {
        relpath, err := filepath.Abs(path)
        if err != nil {
            fmt.Println("It's error when converted to an absolute path.")
            return false
        }

        pattern := `^/opt/pwm`
        reg := regexp.MustCompile(pattern)
        return reg.MatchString(relpath)
    }

    func main() {
        defer func() {
            if err := recover(); err != nil {
                fmt.Println(err) // 【修改】把错误信息抛出来 
            }
        }()

        var path = os.Args[1]
        if !validate(path) {
            panic("Invalid File")
            return
        }

        _, err := os.Open(path)
        if err != nil {
            panic("Invalid File") // 【修改】封装错误使之不包括敏感信息
            return
        }
    }
    ```

    【注】：该示例假定/opt/pwm为安全目录，关于安全目录可参见[*《规则 8.4 避免在共享目录操作文件》*](#_规则_8.4_避免在共享目录操作文件)。

### 规则 4.2 确保方法异常时对象能恢复到之前的状态

- **说明**

    当方法发生异常时，对象一般需要恢复到原来状态，关键的安全对象则必须恢复到原来状态。保持对象状态一致性的常用手段包括：

    - 输入校验，如校验方法的调用参数；
    - 调整逻辑顺序，使可能发生异常的代码在对象被修改之前执行；
    - 当业务操作失败时进行回滚；
    - 对临时副本对象进行操作，直到操作完成后，才把更新提交到原始对象上；
    - 避免去修改对象状态；

- **错误示例**

    下面代码在没有异常发生的情况下能够计算出正确的体积值，一旦出现异常则回滚代码不会被执行，cube对象值无法恢复到原始状态，如果攻击者恶意调用increase()，使得cube的长度超出最大值，那么程序的正常使用者永远无法获取到正确值。

    ```go
    package main

    import (
        "fmt"
    )

    const MAX_LEN = 10
    type cube struct {
        length int
        width  int
        height int
    }

    // 只计算长度小于10的体积
    func (c *cube) getVolume() int {
        defer func() {
            if err := recover(); err != nil {
                fmt.Println(err)
            }
        }()

        // 如果长度小于MAX_LEN直接计算体积，否则长度回滚到原来长度再计算体积
        if c.isLessMaxLen() { // 【错误】抛出异常导致后面的代码无法执行，边长无法恢复
            return c.height * c.width * c.length
        } else {
            c.reduce(MAX_LEN)
            return c.height * c.width * c.length
        }
    }

    func (c *cube) increase(num int) {
        c.height += num
        c.width  += num
        c.length += num
    }

    func (c *cube) reduce(num int) {
        c.height -= num
        c.width  -= num
        c.length -= num
    }

    func (c *cube) isLessMaxLen() bool {
        if c.length > MAX_LEN {
            panic("The length more than max width.")
        }
        return true
    }

    func main() {
        dim := &cube{1, 2, 3}
        dim.increase(MAX_LEN)
        fmt.Println("The extended volume is:", dim.getVolume())
        fmt.Println("The second volume is:", dim.getVolume())// 依旧获取不到正确的值
    }
    ```

- **推荐做法 一**

    在defer语句中执行回滚操作，保障不管是否发生异常都进行回滚。

    ```go
    // 前面代码略……
    func (c *cube) getVolume() int {
        defer func() {
            if err := recover(); err != nil {
                fmt.Println(err)
            }
            c.reduce(MAX_LEN) // 【修改】放在defer语句中执行回滚
        }()

        // 如果长度小于MAX_LEN直接计算体积，否则长度回滚到原来长度再计算体积
        if c.isLessMaxLen() {
            return c.height * c.width * c.length
        } else {
            c.reduce(MAX_LEN) // 【修改】删除此处的回滚
            return c.height * c.width * c.length
        }
    }
    ```

- **推荐做法 二**

    使校验方法isLessMaxLen()职责单一化，对长度进行合法性校验。

    ```go
    func (c *cube) isLessMaxLen() bool {
        return c.length < MAX_LEN // 【修改】使isLessMaxLen()功能单一
    }

    // 只计算长度小于10的体积
    func (c *cube) getVolume() int {
        if c.isLessMaxLen() {
            return c.height * c.width * c.length
        } else {
            c.reduce(MAX_LEN) 
            return c.height * c.width * c.length
        }
    }
    ```

    当然最理想的情况是不修改对象，使其状态总保持一致性。

### 规则 4.3 确保异常情况下能释放文件句柄、DB连接和内存资源等

- **说明**

    当异常发生时若申请的资源（如文件句柄、数据库连接、内存资源等）未得到释放，会引起资源异常占用，可能会导致系统崩溃或者不可用。

- **推荐做法**

    下面代码以数据库为例，在数据入库成功之后，通过延迟defer语句把stmt关闭掉，从而防止资源占用。

    ```go
    func insert(db *sql.DB) {
        stmt, err := db.Prepare("insert into UserTbl(user,password) values(?,?)")
        // 【修改】通过defer语句，确保stmt在函数返回前被关闭
        defer stmt.Close() 

        if err != nil {
            fmt.Printf("stmt err:%s\n", err)
            return
        }
        stmt.Exec("user1", "user1")
    }
    ```

## 5 数据竞争访问

### 规则 5.1 禁止在闭包中直接调用循环变量

- **说明**

    Go语言的特性决定了它会出现其它语言不存在的一些问题，比如在循环中启动协程，当协程中使用到了循环的索引值，往往会出现意想不到的问题，通常需要程序员显式地进行变量调用。

    ```go
    package main

    import (
        "fmt"
    )

    func main() {
        for i := 0; i < limit; i++ {
            go func() {
                fmt.Println("example one:", i)
            }() // 【注】：错误做法 
            go func(i int) {
                fmt.Println("Ex. two:", i)
            }(i) // 【注】：正确做法 
        }
    }
    ```

- **错误示例**

    下面代码输出结果为“5 5 5 5 5”，由于多个协程同时使用变量i产生了数据竞争，这个结果并不是我们所期望的。

    ```go
    package main

    import (
        "fmt"
        "runtime"
        "sync"
    )

    func main() {
        runtime.GOMAXPROCS(runtime.NumCPU())
        var group sync.WaitGroup

        for i := 0; i < 5; i++ {
            group.Add(1)
            go func() {
                defer group.Done()
                fmt.Printf("%-2d", i) //【错误】这里打印的i不是所期望的
            }()
        }
        group.Wait()
    }
    ```

- **推荐做法**

    对循环语句的协程需要进行显式地索引变量调用，这样才能得到类似“0 1 2 3 4”期望结果。

    ```go
    package main

    import (
        "fmt"
        "runtime"
        "sync"
    )

    func main() {
        runtime.GOMAXPROCS(runtime.NumCPU())
        var group sync.WaitGroup

        for i := 0; i < 5; i++ {
            group.Add(1)
            go func(j int) {
                defer group.Done()
                fmt.Printf("%-2d", j) // 【修改】闭包内部使用局部变量
            }(i)  // 【修改】把循环变量显式地传给协程
        }
        group.Wait()
    }
    ```

### 规则 5.2 确保并发安全

- **说明**

    两种最常见的并发安全通信模型：共享数据和消息。共享数据：指多个并发单元分别保持对同一个数据的引用，实现对该数据的共享。被共享的数据可能有多种形式，比如内存数据块、磁盘文件、网络数据等。比如常见的通过同步锁共享内存实现如下。

    ```go
    package main

    import (
        "fmt"
    )

    var count int
    func Count(lock *sync.Mutex) {
        lock.Lock()// 加写锁
        count++
        fmt.Println(count)
        lock.Unlock()// 解写锁，任何一个Lock()或RLock()均需要保证对应有Unlock()或RUnlock()
    }

    func main() {
        lock := &sync.Mutex{}
        for i := 0; i < 10; i++ {
            go Count(lock) //传递指针是为了防止 函数内的锁和 调用锁不一致
        }
        for {
            lock.Lock()
            c := count
            lock.Unlock()
            runtime.Gosched()//交出时间片给协程
            if c > 10 {
                break
            }
        }
    }
    ```

    消息：提供的消息通信机制被称为channel，Go官方社区提到 **不要通过共享内存来通信，而应该通过通信来共享内存。** 强调了channel的重要性，不带缓冲channel使用参考规则3.4，这里介绍一下带缓冲的channel读写。

    给channel带上缓冲，从而达到消息队列的效果，创建一个带缓冲的channel：

    ```go
    c := make(chan int, 1024)
    ```

    在调用make()时将缓冲区大小作为第二个参数传入即可，比如上面这个例子就创建了一个大小为1024的int类型channel，即使没有读取方，写入方也可以一直往channel里写入，在缓冲区被填完之前都不会阻塞。

    从带缓冲的channel中读取数据可以使用与常规非缓冲channel完全一致的方法，但我们也可以使用range关键来实现更为简便的循环读取：

    ```go
    for i := range c {
        fmt.Println("Received:", i)
    }
    ```

    Go语言提供其他原子操作API，可参考golang官网https://golang.org/pkg/sync/atomic/

## 6 敏感数据安全

### 规则 6.1 禁止将敏感信息硬编码在程序中

- **说明**

    如果将敏感信息（如密码、密钥等）硬编码在程序中，可能会将敏感信息暴露给攻击者，任何能够访问到二进制文件的人都可以反汇编二进制文件并发现这些敏感信息。因此，不能将信息硬编码在程序中。

    同时，硬编码敏感信息会增加代码管理和维护的难度。例如，在一个已经部署的程序中修改一个硬编码的密码需要发布一个补丁才能实现。

- **错误示例**

    下面代码假设密码是敏感数据，且把密码明文硬编码在代码中，恶意用户可以反汇编可执行性文件，发现该密码。

    ```go
    var password = "asdf1234" // 【错误】把密码明文硬编码在代码中
    ```

- **推荐做法**

    首先把包含用户密码信息的文件放在安全目录下，然后程序读取安全目录下的该文件内容，并且使用完之后立即从内存中清除，这样即使攻击者成功反汇编了可执行性文件，也无法获取用户密码。

    ```go
    package main

    import (
        "fmt"
        "io/ioutil"
        "os"
    )

    var passwrod string

    func main() {
        // 【修改】读取安全目录下的带用户密码信息的文件内容
        f, err := os.Open("/opt/pwm/etc.info")
        if err != nil {
            fmt.Println("Invaild file.")
            return
        }
        defer f.Close()

        pwd, err := ioutil.ReadAll(f)
        if err != nil {
            fmt.Println("Invaild file content.")
            return
        }
        // 【注】若文件中的密码没有加密，则读取到内存时进行加密 
        password = encrypt(pwd, salt, iterNum, sha256.New)

        // 加密之后清理内存明文
        for i := range pwd {
            pwd[i] = 0
        }
    }
    ```

### 规则 6.2 禁止将敏感信息保存在日志中

- **说明**

    在日志中不能保存密码（包括明文密码和密文密码）、密钥和其它敏感信息。对于敏感信息建议采取以下方法：

    - 不在日志中打印敏感信息；
    - 若因为特殊原因必须要打印日志，则用固定长度的星号`*`代替输出的敏感信息；

## 7 加解密

### 规则 7.1 使用crypto/rand包生成安全随机数

- **说明**

    在安全敏感的代码块（如挑战算法中的随机数生成、SessionID的生成等场景），必须采用安全随机数。

    `cryto/rand`包中提供了密码学安全伪随机数生成器，它提供了Reader变量

    - 在Unix系统中Reader读取`/dev/urandom`生成随机数、
    - 在Linux系统中Reader使用`getrandom`生成随机数、
    - 在Windows系统中Reader使用`CryptGenRandom`API生成随机数。

- **推荐做法**

    ```go
    package main

    import (
        "crypto/rand"
        "fmt"
    )

    func main() {
        k := make([]byte, 32)
        // 使用rand.Read()生成安全随机数 
        if _, err := rand.Read(k); err != nil {
            fmt.Printf("rand.Read() error : %v n", err)
        }
        fmt.Printf("rand.Read(): %v n", k)
    }
    ```

### 规则 7.2 禁止使用私有或者弱加密算法

- **说明**

    禁止使用私有算法或者弱加密算法（如DES、MD5、SHA1等），应该使用经过验证的、安全的、公开的加密算法。

    加密算法分为对称加密算法和非对称加密算法：

    - 推荐使用的对称加密算法有：AES：128位
    - 推荐使用的非对称加密算法有：RSA：2048位
    - 推荐使用的数字签名算法有：DSA、ECDSA：2048位

    除了以上提到的几种算法之外，还经常使用安全哈希算法（如SHA256等）来验证消息或数据的完整性。

    有关加密算法的选择和使用的详细规定，请参考公司发布的《密码算法应用规范》。

### 规则 7.3 基于哈希算法的密码安全存储必须加入盐值

- **说明**

    单向哈希是在一个方向上工作的哈希函数，从预映射的值很容易计算其哈希值，但是要根据特定哈希值产生一个预映射的值却是非常困难的。单向哈希主要应用于加密、消息完整性校验、冗余校验等场景。

    - 没有加入盐值的哈希加密原理：

        `密文 = 哈希算法（明文）`

        > 此时若攻击者获取到密文，同时知道哈希算法，就可以通过字典攻击来探测和获取密码。

    - 加入盐值的哈希加密原理：

        `密文 = 哈希算法（明文 + 盐值）`

        > 由于盐值可以随机设置，这样即使相同的密码，但由于盐值不同，密文也不同，从而增加了破解难度。

    密码单向Hash场景下可以使用[PBKDF2](https://github.com/golang/crypto/tree/master/pbkdf2)算法，它是一个密钥导出算法，既可用于导出密钥，也可以用于密码保存，并且已在RFC 2898标准中给予了定义。它目前被最为广泛、被大多数算法库所支持。

    关于如何使用带盐值的哈希算法来安全的存储密码，请参考公司发布的《密码算法应用规范》。

## 8 I/O安全

### 规则 8.1 确保临时文件使用完毕后及时删除

- **说明**

    程序员经常会在全局可写的目录中创建临时文件，例如Windows系统的C:\\Windows\\Temp、POSIX系统的/tmp与/var/tmp，这类目录中的文件可能会被定期清理，然而，如果文件未被完全地创建或者用完后还是可访问的，那么具备本地文件系统访问权限的攻击者便可以利用共享目录中的文件进行攻击。

    删除已经不再需要的临时文件有助于对文件或者其它资源（如二级存储）进行回收利用，每个程序在正常运行过程中都有责任确保删除已使用完毕的临时文件。

- **错误示例**

    下面代码虽然在程序运行结束时把临时文件关闭了，但由于没有将临时文件删除，给攻击者留下可乘之机。

    ```go
    package main

    import (
        "fmt"
        "io/ioutil"
    )

    func main() {
        f, err := ioutil.TempFile("/opt/pwm", "prefix")
        if err != nil {
            fmt.Println("Create temp file failure.")
            return
        }
        defer f.Close() // 【错误】临时文件虽然被关闭，但未被删除

        f.Write([]byte("This is a example."))
    }
    ```

- **备注**

    该错误示例假设把临时文件创建到安全目录中，并且指定了合适的访问权限。

    关于安全目录可参见[《规则 8.4 避免在共享目录操作文件》](#_规则_8.4_避免在共享目录操作文件)；

    关于文件访问权限可参见[《规则 8.2 在多用户系统中创建文件时必须指定合适的访问许可》](#_规则_8.2_在多用户系统中创建文件时必须指定合适的访问权限)；

- **推荐做法**

    在临时文件使用完毕之后，程序终止之前，调用os的Remove()函数显式地对文件进行删除。

    ```go
    package main
    import (
        "fmt"
        "io/ioutil"
        "os"
    )

    func closeAndRemove(f *os.File) {
        f.Close()
        os.Remove(f.Name()) // 【修改】显式地调用os.Remove()删除临时文件
    }

    func main() {
        f, err := ioutil.TempFile("/opt/pwm", "prefix")
        if err != nil {
             fmt.Println("Create temp file failure.")
             return
        }
        defer closeAndRemove(f) // 【修改】调用关闭并删除函数

        f.Write([]byte("This is a example."))
    }
    ```

### 规则 8.2 确保在多用户系统中创建文件时指定合适的访问许可

- **说明**

    多用户的文件系统是通过权限和许可模型来保护文件访问的，多用户系统中的文件通常归属于某个特定的用户，该特定用户可以指定系统中的其他用户访问此文件内容。当一个文件被创建时，文件访问许可规定了哪些用户可以访问或者操作这个文件。

    若文件创建时没有对文件的访问许可做足够的限制，攻击者便有机会对文件进行读取或者修改，因此一定要在创建文件时就为其指定访问许可，以防止未授权的文件访问。

- **错误示例**

    下面代码在创建文件时没有显式指定要创建的文件权限，Go语言缺省采用0666模式创建一个任何人都可读写，不可执行的名为unlimit.txt文件，这样无法防止未授权的访问。

    ```go
    package main

    import (
        "fmt"
        "os"
    )

    func main() {
        // 【错误】没有显式指定文件权限
        f, err := os.Create("/opt/pwm/unlimit.txt") 
        if err != nil {
             fmt.Println("Create unlimit.txt file failure.")
             return
        }

        defer f.Close()
        f.Write([]byte("This is a example."))
    }
    ```

- **例外情况**

    如果文件是创建到安全目录中，而且该目录对于非受信用户是不可读的，那么允许以默认许可创建文件。例如，如果整个文件系统是可信的或者只有可信用户可以访问，就属于这种情况。关于安全目录可参见[《规则 8.4 避免在共享目录操作文件》](#_规则_8.4_避免在共享目录操作文件)。

- **推荐做法 一**

    通过`Chmod()`函数设置文件的访问许可，这样可以防止未经授权的用户越权访问。另外在必要的情况下也可以使用`Chown()`函数更改文件的拥有者。

    ```go
    package main

    import (
        "fmt"
        "os"
    )

    func main() {
        f, err := os.Create("/opt/pwm/limit.txt") // 默认文件权限为0666
        if err != nil {
             fmt.Println("Create limit.txt file failure.")
             return
        }

        defer f.Close()
        f.Chmod(0600) // 【修改】使用Chmod()函数显式更改文件权限

        f.Write([]byte("This is a example."))
    }
    ```

- **推荐做法 二**

    Go语言提供了`OpenFile()`函数，它要求使用者指定选项（如O_RDONLY）、指定模式（如0660）地去打开文件。若使用者指定选项为O_CREATE，则OpenFile()发现文件不存时会主动创建文件。

    ```go
    package main

    import (
        "fmt"
        "os"
    )

    func main() {
         // 【修改】使用OpenFile()指定文件访问许可
         f, err := os.OpenFile("/opt/pwm/limit.txt", os.O_CREATE|os.O_WRONLY|os.O_TRUNC, 0600) // O_CREATE表示文件不存在时自动创建
         if err != nil {
              fmt.Println("Create limit.txt file failure.")
              return
         }
         defer f.Close()

        f.Write([]byte("This is a example."))
    }
    ```

### 规则 8.3 确保文件路径验证前对其进行标准化

- **说明**

    绝对路径名或者相对路径名中可能会包含文件链接（例如：软链接、硬链接、快捷方式、影子文件、别名等），或者包含特殊字符（例如：.与..），这使得验证文件路径变得困难；同时还有很多操作系统和文件系统相关的命名约定，也增加了验证文件路径的困难。

    若不对文件路径进行验证，攻击者便可以访问任意文件，或者利用目录遍历、等价路径等方式，读取/修改系统重要数据文件，对系统进行攻击。

    这就意味着当文件路径来自非信任域时，在文件操作之前必须对文件路径进行验证，而对文件路径标准化使得验证文件路径简单起来。

- **错误示例**

    当攻击者执行`go run main.go –filepath="/opt/pwm/src/../../../etc/passwd"`时，下面代码没有对攻击者输入的filepath进行标准化验证，攻击者于是能绕过对文件路径的校验。

    ```go
    package main

    import (
        "flag"
        "fmt"
        "io/ioutil"
        "os"
        "regexp"
    )

    // 验证文件路径是否在安全目录/opt/pwm下
    func validate(path string) bool {
        // 【错误】未把文件路径标准化，当path为/opt/pwm/src/../../../etc/passwd时，攻击者可以绕过该验证
        pattern := `^/opt/pwm`
        reg := regexp.MustCompile(pattern)
        return reg.MatchString(path)
    }

    func main() {
        var path string
        flag.StringVar(&path, "filepath", "", "Get user input file path")
        flag.Parse()

        if !validate(path) {
            fmt.Println("file not in security directory.")
            return
        }
    }
    ```

- **备注**

    该错误示例假设/opt/pwm为安全目录，关于安全目录可参见[《规则 8.4 避免在共享目录操作文件》](#_规则_8.4_避免在共享目录操作文件)。

- **推荐做法**

    `filepath.Abs()`函数返回绝对路径，删除了所有符号链接。同时对文件路径是否在安全目录的验证也是至关重要的，可以防止黑客通过构造文件路径指向系统的关键文件。

    ```go
    package main

    import (
        "flag"
        "fmt"
        "io/ioutil"
        "os"
        "regexp"
        "path/filepath"
    )

    // 验证文件路径是否在安全目录/opt/pwm下
    func validate(path string) bool {
        relpath, err := filepath.Abs(path) // 【修改】对文件路径进行标准化
        if err != nil {
            fmt.Println("It's error when converted to an absolute path.")
            return false
        }

        pattern := `^/opt/pwm`
        reg := regexp.MustCompile(pattern)
        //【修改】对标准化后的路径进行正则匹配
        return reg.MatchString(relpath)
    }

    func main() {
        var path string
        flag.StringVar(&path, "filepath", "", "Get user input file path")
        flag.Parse()

        if !validate(path) {
            fmt.Println("file not in security directory.")
            return
        }

        // 后面省略对该路径的文件操作……
    }
    ```

### 规则 8.4 避免在共享目录操作文件

- **说明**

    多用户系统允许多个具有不同权限的用户共享一个文件系统。攻击者可以利用许多文件系统的特性和功能，包括文件链接、设备文件和共享文件访问，对文件进行越权访问，特别是在多个用户可以创建、移动、删除文件的共享目录中操作文件时。当程序以较高权限运行时，可能被攻击者利用提升自己的权限。

    为了防止漏洞，程序必须仅在安全目录中操作文件。如果对于某个特定用户，只有该用户与系统管理员可以在其中创建、移动、删除文件，那么这个目录就是安全目录。

- **错误示例**

    下面代码有两个假设前提：
    1. 文件myfifo在共享目录/opt/pwm下；
    2. 攻击者成功越权后篡改了文件myfifo内容。

    当执行`go run main.go –filepath="/opt/pwm/myfifo"`时，程序会读取文件myfifo的内容（攻击者篡改后的），便可能发生不可预测的灾难。更甚者，若该文件是一个先进先出（FIFO）的文件或者锁定设备，攻击者对文件myfifo操作时文件会挂起，合法程序永远无法操作该文件。

    ```go
    package main

    import (
        "flag"
        "fmt"
        "os"
    )

    func main() {
        var path string
        flag.StringVar(&path, "filepath", "", "Get file which in secure directory")
        flag.Parse()

        // 【错误】读取共享目录/opt/pwm/myfifo文件，由于该文件可能会被篡改，可能会发生不可预期的灾难
        f, err := os.Open(path)
        if err != nil {
            fmt.Println("Invalid file.")
            return
        }
        defer f.Close()

        // 后面代码省略……
    }
    ```

- **推荐做法**

    考虑到共享目录固有的可访问性以及潜在的条件竞争，必须限定在安全目录中对文件进行操作，同时给予文件必要的合法性校验。

    ```go
    package main

    import (
        "flag"
        "fmt"
        "os"
    )

    func main() {
        var path string
        flag.StringVar(&path, "filepath", "", "Get file which in secure directory")
        flag.Parse()

        if !isInSecureDir(path) { //【修改】增加判断文件是否在安全目录中
            fmt.Println("File not in secure directory")
            return
        }

        // 判断文件是否为符号链接
        f, err := os.Lstat(path) //【修改】判断文件的合法性
        if err != nil {
        fmt.Println("Read file error", err)
        return
        }
        if f.Mode()&os.ModeSymlink != 0 {
            fmt.Println("File is soft link")
            return
        }

        // 进行文件读写操作
        fi, err := os.Open(path)
        if err != nil {
            fmt.Println("Invalid file.")
            return
        }
        defer fi.Close()

        // 后面代码省略……
    }
    ```

- **备注**

    这里需要重点强调：

    - 该安全目录函数`isInSecureDir()`用于确保传入的文件及其所有上层目录是归当前用户或者管理员所有，任何其他的用户不能对其进行写操作，并且其所有上层目录也不能被任何其他用户删除或者改名（除了系统管理员）。
    - 该安全目录函数`isInSecureDir()`需要开发人员自己编写。在对目录进行检查时，非常重要的一点是必须从根目录往叶节点目录遍历，这样做可以防止危险的条件竞争，以防止具有其中至少一个目录权限的攻击者对目录进行重命名或者重新创建目录的操作，而这个操作发生在被篡改目录的权限检查之前，但在其子目录的权限检查之后。
    - 如果路径中包含任何符号链接，这个方法会递归调用链接的目标目录以确保它也是安全的。对于一个符号链接目录，只有当源目录与目标目录都是安全的时候才是安全的。这个方法会检查路径中的每一个目录，以确保所有目录都是属于当前用户或者管理员的，其他任何用户都不能在其中创建、删除、或者重命名文件。

- **例外情况**

    对于运行在单用户系统或者没有共享目录的系统，或者不会发生文件系统漏洞的系统上的程序，则无需在操作文件之前确保其在安全目录之中。

### 规则 8.5 确保安全地从压缩包中提取文件

- **说明**

    解压文件时有两个问题需要避免：

    - 提取出的文件标准路径是否落在解压目标目录之外？
    - 提取出的文件是否消耗过多的系统资源？

    对于第1种情况：攻击者可以从zip文件中往用户可访问的任何目录写入任意的数据；  
    对于第2种情况：当资源使用远远大于输入数据所使用的资源时，可能会发生DoS攻击。  
    就ZIP算法的本质而言就可能导致ZIP炸弹的出现，因为极高的压缩率，即便在解压小文件时（如ZIP、GIF、以及gzip编码的HTTP内容），也可能会导致过度的资源消耗。

    ZIP算法能够产生非常高的压缩比率，例如一个文件由多行a字符和多行b字符交替出现构成，对于这样的一个文件可达到200：1以上的压缩比率。

    任何被提取条目的目标目录不在程序预期计划的目录之内时，要么拒绝将其提取出来，要么将其提取到一个安全的位置。ZIP中任何被提取条目，若解压之后的文件大小超过一定的限制时，必须拒绝将期解压，具体限制多少，由平台和业务的具体情况所决定。

- **错误示例**

    下面代码未对解压的文件名做校验，直接将文件路径传递给os.MkdirAll()函数，同时也未检查解压文件的资源消耗情况，它可能会导致程序运行到操作完成或者本地资源被耗尽。

    ```go
    package main

    import (
        "archive/zip"
        "fmt"
        "io"
        "os"
        "path/filepath"
    )

    // 解压文件操作
    // @zipfile 要解压的压缩包
    // @destpath 解压到的目的路径
    func unzipfile(zipfile, destpath string) bool {
        // 打开压缩包
        rc, err := zip.OpenReader(zipfile)
        if err != nil {
            fmt.Println("Open the zipfile error.")
            return false
        }
        defer rc.Close()

        // 遍历压缩包内的目录和文件
        for _, file := range rc.File {
            irc, err := file.Open()
            if err != nil {
            fmt.Println("open file which in zip archive error.")
            break
        }
        defer irc.Close()

        // 拼装指定的解压路径
        var targetpath = filepath.Join(destpath, file.Name)

        // 如果是目录则创建；如果是文件则把文件复制到指定的目的地
        if file.FileInfo().IsDir() {
            os.MkdirAll(targetpath, file.Mode()) // 【错误】直接将文件路径传给MkdirAll()
        } else {
            f, err := os.OpenFile(targetpath, os.O_WRONLY|os.O_CREATE|os.O_TRUNC, file.Mode())
            if err != nil {
            fmt.Println("open the dest file error.")
            break
        }
        defer f.Close()

        // 从压缩流中把内容复制到目的文件中
        wt, err := io.Copy(f, irc) // 【错误】未检查解压文件消耗情况 
        if err != nil {
            fmt.Println("copy file content error.")
            break
        }
        fmt.Println("wt:", wt)
        }
        }
        return true
    }
    // 后面代码略……
    ```

- **推荐做法**

    下面代码在每解压一个条目之前都对文件路径进行校验，若校验不通过则跳过该条目，继续后面的解压，除了校验文件路径之外，还会检查每一个条目的大小，如果条目太大（例如是100M）则会跳过该条目；最后代码会计算压缩包中的总条目数量（例如超过1024个）则解压失败。

    ```go
    package main

    import (
        "archive/zip"
        "errors"
        "fmt"
        "io"
        "os"
        "path/filepath"
    )

    const TOO_MANY_FILE int = 1024
    // max size of unzipped data, 100MB
    const TOOBIG = 0x6400000 
    const BUFSIZE = 1024

    // 解压文件操作
    // @zipfile 要解压的压缩包
    // @destpath 解压到的目的路径
    func unzipfile(zipfile, destpath string) bool {
    // 打开压缩包
    rc, err := zip.OpenReader(zipfile)
    if err != nil {
        fmt.Println("Open the zipfile error.")
        return false
    }
    defer rc.Close()

    //【修改】解压文件的数量超过1024限制
    if len(rc.File) > TOO_MANY_FILE { 
        fmt.Println("Too many file will be unzip.")
        return false
    }
    destpath = filepath.Clean(destpath) //【修改】将目的路径标准化

    // 遍历压缩包内的目录和文件
    for _, file := range rc.File {
        irc, err := file.Open()
        if err != nil {
            fmt.Println("open file which in zip archive error.")
            break
        }
        defer irc.Close()
    
        // 拼装指定的解压路径
        var targetpath = filepath.Join(destpath, file.Name)
    
        // 如果是目录则创建；如果是文件则把文件复制到指定的目的地
        if file.FileInfo().IsDir() {
            os.MkdirAll(targetpath, file.Mode()) 
        } else {
            f, err := os.OpenFile(targetpath, os.O_WRONLY|os.O_CREATE|os.O_TRUNC, file.Mode())
            if err != nil {
            fmt.Println("open the dest file error.")
            break
        }
        defer f.Close()
    
        // 从压缩流中把内容复制到目的文件中
        wt, err := copyBuffer(f, irc) //【修改】检查解压文件消耗情况
        if err != nil {
            fmt.Println("copy file content error.")
            break
        }
        fmt.Println("wt:", wt)
    }
    }
    return true
    }
    // 后面代码略……
    ```

    其中copyBuffer()函数是自定义的，代码如下：

    ```go
    func copyBuffer(dst io.Writer, src io.Reader) (written int64, err error) {
        buf := make([]byte, BUFSIZE)
        for {
            nr, er := src.Read(buf)
            if nr > 0 {
                //判断文件大小是否超出限制
                if written > TOOBIG {
                    err = errors.New("The file too big!")
                    break
                }
                nw, ew := dst.Write(buf[0:nr])
                if nw > 0 {
                    written += int64(nw)
                }
                if ew != nil {
                    err = ew
                    break
                }
                if nr != nw {
                    err = io.ErrShortWrite
                    break
                }
            }
            if er == io.EOF {
                break
            }
            if er != nil {
                err = er
                break
            }
        }
        return written, err
    }
    ```

## 9 序列化和反序列化

### 规则 9.1 禁止序列化未加密的敏感数据

- **说明**

    虽然序列化可以将对象的状态保存为一个字节序列，之后通过反序列化该字节序列又能重新构造出原来的对象，但是它并没有提供一种机制来保证序列化数据的安全性。可访问序列化数据的攻击者可以借此获取敏感信息并确定对象的实现细节。攻击者也可恶意修改其中的数据，试图在其被反序列化之后对系统造成危害。敏感数据序列化之后是潜在对外暴露着的，因此序列化信息中不应该包括：密钥、数字证书、以及那些在序列化时引用敏感数据的类。此条规则的意义在于防止敏感数据被无意识的序列化导致敏感信息泄露。

- **错误示例**

    下面代码中密码是敏感信息，那么将其序列化到数据流中使之面临敏感信息泄露和被恶意篡改的风险。注意：这里以Json为例，但xml、csv、binary等也存在类似问题。

    ```go
    package main
    import (
        "encoding/json"
        "fmt"
    )

    type CreditCard struct {
        Id     int    `json:"card_id"`
        Bank   string `json:"bank_name"`
        Owner  string `json:"owner_id"`
        Passwd string `json:"password"` // 敏感信息
    }

    func main() {
        var card = CreditCard{
            Id:     1320044730,
            Bank:   "China bank",
            Owner:  "golang",
            Passwd: "Tell4U", // 密码不能硬编码在代码中，这里只是为了程序演示
        }

        // 序列化
        serialStr, err := json.Marshal(card) // 【错误】直接将敏感信息序列化
        if err != nil {
        fmt.Println("Serialization error.")
        return
        }
        fmt.Println(string(serialStr), err)
        // 发送给接收方
        // 后面代码略……
    }
    ```

- **推荐做法**

    在将某个包含敏感数据的结构体序列化时，程序必须确保敏感数据不被序列化。这包括阻止包含敏感信息的数据成员被序列化，以及不可序列化或者敏感对象的引用被序列化。该示例针对Json将相应的字段声明为`-`， 从而使它们不包括在依照默认的序列化机制应该被序列化的字段列表中。这样既避免了错误的序列化，又防止了敏感数据被意外序列化。

    ```go
    package main

    import (
        "encoding/json"
        "fmt"
    )

    type CreditCard struct {
        Id     int    `json:"card_id"`
        Bank   string `json:"bank_name"`
        Owner  string `json:"owner_id"`
        Passwd string `json:"-"` // 【修改】声明Passwd敏感信息不被序列化
    }

    func main() {
        var card = CreditCard{
            Id:     1320044730,
            Bank:   "China bank",
            Owner:  "golang",
            Passwd: "Tell4U", // 密码不能硬编码在代码中，这里只是为了程序演示
        }

        // 序列化
        serialStr, err := json.Marshal(card) 
        if err != nil {
        fmt.Println("Serialization error.")
        return
        }
        fmt.Println(string(serialStr), err)
        // 发送给接收方
        // 后面代码略……
    }
    ```

- **备注**

    - 上例中的`-`阻止包含敏感数据的数据成员被序列化，仅对Json、Xml有效；其它格式的根据语言特点来阻止，比如gob可以通过数据成员首字母小写来阻止；
    - 密码不能硬编码在代码中，这里只是为了程序演示；

- **例外情况**

    可以序列化已正确加密的敏感数据。

### 规则 9.2 确保将敏感对象发送出信任区域前必须先签名后加密

- **说明**

    敏感数据传输过程中要防止窃取和恶意篡改。使用安全的加密算法加密传输对象可以保护数据，防止对象被非法篡改，保持其完整性。  
    在以下场景中，需要对对象密封和数字签名来保证数据安全：

    - 序列化或者传输敏感数据
    - 没有使用类似SSL传输通道
    - 敏感数据需要长久保存，比如在硬盘驱动器上

    关于加密算法的规定请参考公司规范库发布的《密码算法应用规范》

- **错误示例**

    下面代码中假设密码是敏感信息，程序没有采取任何措施抵御数据传输过程中可能遭遇到的攻击，因此任何人都可以对序列化的数据流实施反序列化从而获取到数据信息。

    ```go
    package main
    import (
        "bytes"
        "encoding/gob"
        "fmt"
    )

    type CreditCard struct {
        Id     int
        Bank   string
        Owner  string
        Passwd string
    }

    func main() {
        var card = CreditCard{
            Id:     1320044730,
            Bank:   "China bank",
            Owner:  "golang",
            Passwd: "Tell4U", // 密码不能硬编码在代码中，这里只是为了程序演示
        }

        // 序列化
        cache := new(bytes.Buffer)
        encoder := gob.NewEncoder(cache)
        err := encoder.Encode(card) // 【错误】直接将敏感信息序列化
        if err != nil {
            fmt.Println("Serialization error.")
            return
        }
        // 发送给接收方
        // 后面代码略……
    }
    ```

- **推荐做法**

    对敏感信息先签名后加密再后进行序列化。

    ```go
    package main

    import (
        "bytes"
        "crypto"
        "crypto/aes"
        "crypto/cipher"
        "crypto/rand"
        "crypto/rsa"
        "crypto/sha256"
        "crypto/x509"
        "encoding/gob"
        "encoding/pem"
        "errors"
        "fmt"
        "io"
        "io/ioutil"
    )

    type CreditCard struct {
        Id     int
        Bank   string
        Owner  string
        Passwd string
    }

    func main() {
        //【修改】对密码先签名
        //【注】：密码不能硬编码在代码中,这里只是为了演示签名
        signature, err := signSensitiveData("Tell4U") 
        if err != nil {
            fmt.Println("sign sensitive data error.", err)
            return
        }

        //【修改】对密码签名后再加密
        encrypt, err := encryptSensitiveData(signature)
        if err != nil {
            fmt.Println("encrypt sensitive data error.", err)
            return
        }

        // 序列化
        var card = CreditCard{
            Id:     1320044730,
            Bank:   "China bank",
            Owner:  "golang",
            Passwd: encrypt,
        }

        cache := new(bytes.Buffer)
        encoder := gob.NewEncoder(cache)
        err = encoder.Encode(card)
        if err != nil {
            fmt.Println("Serialization error.")
            return
        }
        // 发送给接收方
        // 后面代码略……
    }
    ```

    其中签名函数如下：

    ```go
    // 签名敏感数据
    func signSensitiveData(sensitive string) (string, error) {
        // 读取私钥文件内容
        privateKey, err := ioutil.ReadFile("/opt/pwm/privatekey.pem")
        if err != nil {
            return sensitive, errors.New("Read private key content error.")
        }

        // 对私钥文件内容进行解码
        privateContent, _ := pem.Decode(privateKey)
        if privateContent == nil {
            return sensitive, errors.New("PrivateKey is not PEM formate.")
        }

        // 解析私钥文件
        content, err := x509.ParsePKCS8PrivateKey(privateContent.Bytes)
        if err != nil {
            return sensitive, errors.New("Parse private key content error.)
        }

        // 对敏感数据进行签名
        hashed := sha256.Sum256([]byte(sensitive))
        signature, err := rsa.SignPKCS1v15(rand.Reader, content.(*rsa.PrivateKey), crypto.SHA256, hashed[:])
        if err != nil {
            return sensitive, errors.New("Use private key singe sensitive date error.")
        }
        return string(signature), nil
    }
    ```

    其中加密函数如下：

    ```go
    // 加密敏感数据
    func encryptSensitiveData(sensitive string) (string, error) {
        // 生成随机向量
        encryptText := make([]byte, aes.BlockSize+len(sensitive))
        iv := encryptText[:aes.BlockSize]
        if _, err := io.ReadFull(rand.Reader, iv); err != nil {
            return sensitive, errors.New("Generate random vector error")
        }

        // 使用密钥加密，其中密钥长为32时表示选择AES-256
        const key string = "YnVluD8fNo1mzid6xNrCGX7tkwjydmN7"
        block, err := aes.NewCipher([]byte(key))
        if err != nil {
            return sensitive, errors.New("Generate dictionary block error")
        }

        // 设置加密工作模式
        mode := cipher.NewCBCDecrypter(block, iv)
        mode.CryptBlocks(encryptText, []byte(sensitive))
        return string(encryptText), nil
    }
    ```

- **例外情况**

    - 为已加密对象签名在特定场景下是合理的，比如验证从其它地方接收的加密对象的真实性，这是对于被机密对象本身非其内容的保证；
    - 签名和加密仅仅对于必须跨过信任边界的对象是必需的，始终位于信任边界内的对象不需要签名和加密。例如，某网络全部位于信任边界内，则始终处于该网络上的对象无需签名和加密；

## 10 其它

### 规则 10.1 确保对channel是否关闭做检查

- **说明**

    在调用方法时不能想当然地认为它们都会执行成功，当错误发生时往往会出现意想不到的行为，因此必须严格校验并合适处理函数的返回值。例如：channel在关闭后仍然支持读操作，如果channel中的数据已经被读取，再次读取时会立即返回0值与一个channel关闭指示。如果不对channel关闭指示进行判断，可能会误认为收到一个合法的值。因此在使用channel时，需要判断channel是否已经关闭。

- **错误示例**

    下面代码中若cc已被关闭，如果不能cc是否关闭做检查，则会产生死循环造成DoS攻击。

    ```go
    package main
    import (
        "errors"
        "fmt"
        "time"
    )

    func main() {
        var cc = make(chan int)
        go client(cc)

        for {
            select {
                case <-cc: //【错误】当channel cc被关闭后如果不做检查则造成死循环
                    fmt.Println("continue")
                case <-time.After(5 * time.Second):
                    fmt.Println("timeout")
            }
        }
    }

    func client(c chan int) {
        defer close(c)
        for {
            err := processBusiness()
            if err != nil {
                c <- 0
                return
            }
            c <- 1
        }
    }

    func processBusiness() error {
        return errors.New("domo")
    }
    ```

- **推荐做法**

    对通道增加关闭判断。

    ```go
    for {
        select {
        case _, ok := <-cc:
            // 【修改】增加对chnnel关闭的判断，防止死循环
            if !ok {
                fmt.Println("channel closed")
                return
            }
            fmt.Println("continue")
        case <-time.After(5 * time.Second):
            fmt.Println("timeout")
        }
    }
    ```

---

_**参考链接：**_  

- [Go Secure Coding Practices](https://checkmarx.gitbooks.io/go-scp/)