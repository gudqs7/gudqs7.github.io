---
title: 'Go学习笔记 (三) : method 和 interface'
date: 2020-05-20 09:24:19
tags:
  - learn-go
categories: 
  - go
---

# 为了面向对象(首先你得有个对象)

## method

### 定义一个method

```go
type Person struct {
	name string
	age int
}

func (p *Person) growup() {  //想想去掉 * 会怎样?
	p.age += 1
}

func (p Person) getName() string {
	return p.name
}

func (p *Person) setName(name string) {
	p.name = name
}

func main(){
	p := Person{"wq",10}
	println(p, p.name, p.age)
	p.growup()
	println(p.age)
	p.setName("gudqs")
	println(p.name)
}
```
>使用 func ( 方法属于者 方法属于者类型) 方法名 (方法参数列表) (返回参数列表) {..}定义一个方法
>传递 指针类型 使结构体值改变
>所有自定义类型, 和一些内置类型均可作为方法属于者,即可拥有方法

### 方法继承

```go
type Person struct {
	name string
	age int
}

type Student struct {
	Person
	no int
	phone string
}

func (p Person) sayhi() {
	println(p.name,p.age)
}

func main(){
	s := Student{Person{"wq",20},1,"110"}
	s.sayhi()
}
```

> 利用匿名字段可继承改字段类型的方法 同样的可以直接调用


### 方法重写
>在之前代码 后添加一个 Student 的sayhi方法 , 实现重写

```go
func (s Student) sayhi() {
	println(s.name,s.no,s.phone,s.age)
}

```
>查看执行结果, 如果学过java ,你一定已经露出了纯洁的笑容       : ) 

### TIP
>在方法定义时 , 方法属于者类型 为一个指针时, 方法种操作 指针不需要 加 *也可, go会自动加
>不能为 int , []int 等类型 添加方法, 但通过自定义类型 ,如 type Int int , 则又可以添加方法 ,因此自定义类型具有高扩展性啊


## interface (接口)
### 定义接口

```go
type UserDao interface {
	add()
	update()
	remove()
}
```
### 使用接口

```go
package main

import f "fmt"

type UserDao interface {
	add(u User)
	update(u User)
	remove(u User)
	findAll() []User
}

type User struct {
	name string
	id int
}	

type UserDaoMysqlImpl struct {
	
}

type UserDaoOracleImpl struct {

}

func (dao UserDaoMysqlImpl) add(u User) {
	f.Println("add user[mysql] :",u)
}

func (dao UserDaoMysqlImpl) update(u User) {
	f.Println("update user[mysql] :",u)
}

func (dao UserDaoMysqlImpl) remove(u User) {
	f.Println("remove user[mysql] :",u.id)
}

func (dao UserDaoOracleImpl) add(u User) {
	f.Println("add user[oralce] :",u)
}

func (dao UserDaoOracleImpl) update(u User) {
	f.Println("update user[oracle] :",u)
}

func (dao UserDaoOracleImpl) remove(u User) {
	f.Println("remove user[oracle] :",u.id)
}

func (dao UserDaoMysqlImpl) findAll() []User {
	users := make([]User,3)
	users[0], users[1], users[2] = User{"wq",1}, User{"aa",2}, User{"gudqs",3}
	return users;
}

func (dao UserDaoOracleImpl) findAll() []User {
	users := make([]User,3)
	users[0] = User{"gg",007}
	return users
}

func main(){
	var dao UserDao
	u := User{"qq",88}
	dao =UserDaoOracleImpl{}
	dao.add(u)
	dao.update(u)
	dao.remove(u)
	for i,u := range dao.findAll() {
		f.Println(i,u)
	}
	dao =UserDaoMysqlImpl{}
	dao.add(u)
	dao.update(u)
	dao.remove(u)
	for j,u2 := range dao.findAll() {
		f.Println("mysql:",j,u2)
	}
}

```

> 一个东东 具有 某个接口的所有方法, 那么这个东东 可以赋值给这个接口类型的变量
> 这个特性说白了就是类似多态

### 接口类型 作为函数(方法) 参数
在上面代码基础上添加 如下代码:

```go
type UserServiceImpl struct {}

func (service UserServiceImpl) add(u User, dao UserDao) {
	print("service do :")
	dao.add(u)
}

func main(){
	service := UserServiceImpl{}
	u := User{"wq",20}
	dao := UserDaoMysqlImpl{}
	service.add(u,dao)
	dao =UserDaoOracleImpl{}
	service.add(u,dao)
}
```
添加类型UserServiceImpl和对应的add方法, 使用UserDao作为参数, 然后修改main()

>  方法参数为接口类型, 好处是 可以接受更多的类型 , 只要那种类型有 这个接口的所有方法

### 空interface

```go
type a interface {}

func main(){
	var i int = 99
	var str string = "wq"
	a = i
	println(a)
	a = str
	println(str)
}
```

> 这样 a类型 就可以接受任何类型了, 就像 java 的Object类型一样 !


### 嵌套 interface

```go
type a interface {
	swap()
	len()
}

type b interface {
	a
	value()
}
```
> 然后b就有了 3个方法 , 对的, 想匿名字段继承一样


### Comma-ok断言

```go
type a interface {}

type b struct {
	name string
}

func main(){
	list := make([]a,3)
	a[0] = 1
	a[1] = "wq"
	a[2] = b{"wq"}
	for _,element := range list {
		ok,val := element.(b)
		if ok {
			println(" b is a a")
		}
		ok,val = element.(int)
		if ok {
			println("int is a a")
		}
	}
	//solution 2
	for _,ele := range list {
		switch value :=ele.(type) {
			case int :
				println("ele is an int",)
			case b :
				println("ele is a b")
			default :
				println("ele is default type")
		}
	}
}
```

> 通过 变量.(类型) 获得返回值 ok 判断 该变量存的是否是 该类型
> 变量.(type) 仅在switch中 可以使用