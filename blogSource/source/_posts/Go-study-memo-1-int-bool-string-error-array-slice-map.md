---
title: 'Go 学习笔记(一) : 3种方式得变量 / 常量与iota / 数据类型(int,bool,string,error,array,slice,map)'
date: 2020-05-20 09:20:11
tags:
  - learn-go
categories: 
  - go
---



# 为了基础的基础
## package , import 

```go
package main

import "fmt"

func main(){
	fmt.Println("Hello World !")
}
```
>包 , 与python类似, 与java不同. 用于模块化. 通过声明包, 和导入包可以实现程序的相互调用. 如 导入 fmt , 使用fmt的函数Println() 
>main.main()是 程序的运行入口, 无参数, 无返回值

## 变量声明

```go
package main
import "fmt"

func main(){
	var complete string = "I 'm complete !"
	var autoType = "I'm auto "
	var simple := "I'm simple "
	complete, autoType, simple = "complete again", "auto again", "simple again"
	fmt.Println(complete,autoType,simple)
}
```
如上所示, 获得一个变量有3种方式
>完整的声明和赋值 (也可只声明, 后赋值)
>声明时不写类型 , 必须同时赋值自动推导类型
>省略var , 使用 := 声明+赋值, 仅用于声明
>###常量

```go
const SEX_MALE int = 0
const SEX_FEMALE = 1
const (
	MALE = iota  //0
	FEMAILE  //1
	WHAT //2
)
```
>使用const关键字可声明常量, 使用()可一次声明多个 使用iota可便捷声, 规则为:
>遇到const重置为0
>同一行值相同
>上一行为iota, 则本行不赋值时为 上一行iota值 + 1, 以此类推

## 数据类型小计
### 整形

>int : int8 , int16, int32, int 64
>uint : uint8, uint16, uint32, uint64
>rune, int, byte

其中 int 与 uint 的区别是 有无符号 , rune 是 int32的别称, byte是uint8的别称, int虽然是32位但与int32不可互用

### Boolean
默认为false, 与java类似, 数0,空等不代表 false

```go
var ok bool 
fmt.Println(ok) //false
var gameover = true
isover := false
```

### 字符串
与java类似, 双引号包围

```go
var str = "I'm a string!"
```
当多行时 使用 反引号 

```go
var multistr = `This is 
a 
multiline
`
```
使用 ` 时, 不转义\n,\t等
>使用string类型时, 注意string不可变, 类似java . 但可强转为byte[] 类型后操作

```go
var str="hello"
str[0] = 'c' //wrong!!!
var c = (byte[])str
c[0]='c'
```

### error
Go内置有一个error类型，专门用来处理错误信息，Go的package里面还专门有一个包errors来处理错误：

```go
err := errors.New("I am error message!")
if err != nil {
    fmt.Print(err)
}
```

### array
在go中, array是值类型, 而slice(切片),map 才是引用类型

```go
arr := [3]int{1,2,3}
array:=[...]int{3,2,1,0}
arr2 := [10]int{1,2,3} //后7个元素则为零值 
var arr3 [10]int
arr3[0]=9
arr3[1]=10
```
>关于数组的赋值,取值与c和java类似
>对于数组的初始化,分完整,部分(不写长度, 不写所有元素)
>注意array是值类型, 互相传递赋值时, 不会相互影响

### slice

```go
 array := [10]int{1,2,3,4,5,6,7,8,9,10}
 aslice := array[:]
 bslice := aslice[2:]
 bslice[3] = 99
 array[0] = -99
 fmt.Println(aslice,bslice)
```
>切片 , 可通过数组[n:m] 获取
>切片是引用类型, 改变bslice , 同时aslice也会改变
>改变arr不会改变aslice , 因为array是值类型

### map

```go
package main

import "fmt"

func main(){ 
    var numbers map[string]int =map[string]int{"all":9999}
    others := make(map[string]int)
    others["name"]=007
    numbers["one"] = 1
    numbers["ten"] = 10
    numbers["three"] = 3
    delete(numbers,"one")
    val , ok := numbers["not ex"]
    fmt.Println(numbers,numbers["one"],others,len(numbers),val,ok)
}
```
>map类似与python种的字典, 操作与slice类似
>元素赋值, 取值使用[]
>map初始化使用 make 或者 map[string]int{}
>使用delete删除key
>使用多返回值检测是否存在key
>使用len查看元素个数