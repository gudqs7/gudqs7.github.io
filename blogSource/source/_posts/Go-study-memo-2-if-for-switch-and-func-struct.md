---
title: 'Go学习笔记 (二) : 流程控制(if,for,switch) 与 函数 与 struct (匿名字段)'
date: 2020-05-20 09:21:55
tags:
  - learn-go
categories: 
  - go
---

# 为了扩展的扩展
## 流程控制

### if
```go
if condition {
	// do something
} else if condition {

}
```
if 后接条件语句(表达式) , 无括号 

```go
if 9>8 {
	//do some...
} else if 8>8 {

} else{

}
```
### for

```go
for expr1; expr2 ;expr3 {
	// some code
}
```
>expr1 为初试化变量语句, 仅执行一次
>expr2 为循环条件语句, 每次循环前都会判断其值
>expr3 是改变循环变量的地方, 每次循环后执行
>其中expr1,expr2,expr3均可省略

```go
for ; ; {
	// some code 
}
```
>另一种是仅留下 循环条件语句 如:

```go
for ; 9>8 ; {
	// some code 
}
```
此时 2个分号 也可省略, 即:

```go
for 9>8 {
	//some code
}
```
啊, 和java的while 多像

```go
me := map[string]string{"name":"wq","age":"19","long":"200"}
for k,v := range me {
	fmt.Println(k,v)
}
```
>嗯, 是的foreach(for in) 就是这个feel
>尤其提示, 如果不使用 k , 请用_ 替代, 避免编译错误

### switch

```go
switch exp {
	case 0:
		//do some ...
	case 1,2,3:
		//do some..
	default:
		//do some..
}
```
>对于exp不需要括号
>每个case后, 都默认带有break, 不会往下执行
>如需执行后面的case,可使用fallthrough 关键字
>可以用 , 号分隔代表多个case
>default就不说了, 类似与if的else

### 最后写个冒泡排序法复习巩固下

```go
package main

import "fmt"

func main(){ 
    slice := []int{3,4,2,9,1,0,88,44,20}
    maopao(slice)
    fmt.Println(slice)
}

func swap(slice []int, index1 int,index2 int) {
    temp := slice[index1]
    slice[index1]=slice[index2]
    slice[index2]=temp
}

func maopao(slice []int) {
    for i := 0; i< len(slice)-1;i++ {
        for j:=0; j< len(slice)-1-i; j++ {
            if slice[j]>slice[j+1] {
                swap(slice,j,j+1)
            }       
        }       
    }
}

```
99 乘法
```go
package main

import "fmt"

func main(){ 
    num, num2 := 1, 1 
    for num<=9 {
        num2 = 1
        for num2<=num {
            fmt.Printf("%d * %d = %d\t",num2,num,num*num2)
            num2++  
        }       
        fmt.Println()
        num++   
    }
}

```

>使用了for的完整版 , if 判断
>slice指针传递 可变性

## 函数 
### func语法

```go
func funcName (params...) (returns...) {
	//some code
	return ...
}
```
如,无参无返回值

```go
func test(){
	//some code
}
```
单参数,单返回值

```go
func test(id int) int {
	//some code
	return 0
}
```
多参数, 多返回值

```go
func test (id int, skills []string) (int , map[string]int) {
	//some code
	numbers := map[string]int{"one":1,"two":2,"three":3}
	return 0 , numbers
}
```
>大概就是这些, 写个阶乘玩玩

```go
func jiechen(n int) int {
	if n==1 {
		return 1
	}
	return n * jiechen(n-1)
}
func main(){
	sum := jiechen(4) //24
}
```
### defer延迟

```go
func test(){
	fmt.Println("func code run")
	for i:=0; i<8; i++ {
		defer fmt.Println("defer",i)
	}
	fmt.Println("func code last")
}
```
>后进先出的队列机制, 逆序输出了 i
>延迟执行, 在return之前

### import

```go
import (
	. "fmt"
	_ "fmt"
	f "fmt"
)
```
>用 . 代表无需前缀, 直接调用fmt的方法
>用_ 代表不使用fmt的函数, 仅加载
>其他名称则为别名 , 如 f.Println()
>

### a func is a type or a value ??

```go
package main

import "fmt"

func main(){
    slice := []int{1,2,3,4,5,6,7,8,9}
    fmt.Println("slice:",slice)
    odd := filter(slice,isOdd)
    even:= filter(slice,isEven)
    fmt.Println(odd,even)
}

type funcType func(int) bool

func isOdd(i int) bool {
    if i%2 ==0 {
        return false
    }
    return true;
}

func isEven(i int) bool {
    if i%2 ==0 {
        return true
    }
    return false
}

func filter(slice []int,f funcType) []int {
    var result []int
    for _, value := range slice {
        if f(value){
            result = append(result, value)
        }
    }
    return result
}

```
>将func作为一个类型, 置于参数中, 由于go也是强类型, 所以需要添加自定义类型
>将func作为实参(值)传递给另一个函数, 然后被调用


## struct
### 语法定义

```go
type person struct {
	name string
	age int
}
```
### 使用

```go
 var p person
 p.name="wq"
 p.age = 79
 
 p1 := person{"wq",89}
 p2 := person{name:"wq",age:99}
 
```
### 匿名字段

```go
type humen struct {
	name string
	age int
	long int
}

type Skills []string

type person struct {
	humen
	int
	Skills
	name string
	phone string
}

func main(){
	p := person{humen{"wq",39,99},666,[]int{"c","java","c#"},"pwq","110"}
	fmt.Println(p,p.name,p.age,p.long,p.phone,p.Skills,p.int)
	//可直接访问匿名字段的 字段, 但可能会覆盖匿名字段的 字段
}
```
>通过2个struct 实现了类似 继承
>通过不给字段名,仅给字段类型定义一个匿名字段
>可直接访问匿名字段, 但可能覆盖匿名字段中的字段
>匿名字段 还可以为 自定义类型, 基础类型 , slice, map, array等