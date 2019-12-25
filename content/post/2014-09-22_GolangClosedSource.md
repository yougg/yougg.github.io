---
Categories: []
Description: ""
Tags: []
title: "Golang 发布闭源包方案"
date: 2014-09-22T00:00:00+08:00
draft: false
---

- **发布方**

例如有如下say包要发布：

    $cd $GOPATH/src/say
    $cat say.go

```go
// say something package
package say
import "fmt"

// private function
func say(str string){
	fmt.Println(str)
}

// Say hi
func Hi(){
	say("Hi......")
}

// Say hello to someone
func Hello(me string){
	say("Hello" + me)
}
```

1. 编译生成say包的.a文件
   (如果要发布到多种系统架构,需要修改编译参数交叉编译出多种发布文件)

        $go install
        $ls $GOPATH/pkg/$GOOS_$GOARCH/say.a

2. 修改发布包对应的源文件(两种方式任选)

- 最简单的修改方式
  只保留包声明的空源文件

            $echo 'package say' > say.go

- 保留源文件中导出的接口与API注释供调用者查看使用

            $cat > say.go <<EOF
            // say something package
            package say
            
            // Say hi
            func Hi()
            
            // Say hello to someone
            func Hello(me string)
            EOF

最后发布方提供两份文件:
编译生成say包的.a文件和修改之后的源文件
`say.a`,`say.go`

- **使用方**

获取闭源包发布方提供的文件
复制`say.a`文件到`$GOROOT/pkg/$GOOS_$GOARCH/`目录中
复制`say.go`文件到`$GOROOT/src/pkg/say/`目录中

    $cp say.a $GOROOT/pkg/$GOOS_$GOARCH/
    $mkdir -p $GOROOT/src/pkg/say/
    $cp say.go $GOROOT/src/pkg/say/

然后就可以在自己的代码中像使用官方标准库一样使用第三方闭源包了.

```go
package main

import "say"

func main(){
    say.Hi()
    say.Hello(" world.")
}
```

如果闭源包保留了接口与API注释, 还可以直接使用`godoc`命令查看, 或者godoc http服务中直接查看.

    $godoc say
    PACKAGE DOCUMENTATION
    
    package say
        import "say"
    
    	say something package
    
    FUNCTIONS
    
    func Hi()
        Say hi
    
    func Hello(me string)
        Say hello to someone

- 备注
如果发布的闭源包引用了其他闭源包或第三方开源包,需要使用本文方式一起发布引用的闭源包, 在发布文档中注明使用者需要导入哪些闭源包或开源包.
