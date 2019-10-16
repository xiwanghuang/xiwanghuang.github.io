---
title: NS2学习笔记（一）
date: 2019-05-29 15:58:38
tags:
---
# TCL简介
## 变量和变量赋值
无需事先声明，使用set指令对变量进行首次赋值，使用变量时前面要加$符号。
```tcl
set a "hello world"
puts "NS2 say $"
```
## 字符串
使用双引号将其括起来为单词赋值。
## 表达式
tcl包含很多表达式，使用expr指令求数学表达式的值或关系表达式的真假。
```tcl
set value [expr 2+3]
```
## 指令替代
使用中括号进行指令替代。指令替代可以将“tcl脚本的执行结果”取代“tcl脚本”。
```tcl
puts "I am [expr 10*2] years old"
```
## 流程控制
* 选择结构
  ```tcl
  if{$a=="hello"}{

  }
  else if{}{

  }
  else{

  }

  switch $a{
      2 {puts $a}
      3 {}
      default {}
  }
  ```
* 循环结构
  ```tcl
  for {set i 0}{$i<5>}{incr i 1}{

  }

  while( $i<5 ){
      incr i 1
  }

  foreach vowel {a e i o u}{
      puts "$vowel is a vowel"
  }
  ```
## 过程
与c++函数类似
```tcl
proc sum_proc {a b}{
    return [expr $a + $b]
}
puts "sum is [sum_proc 1 2]"
```
## 数组
无需像C语言一样需要事先声明数组大小
```tcl
set a(0) "Zero"
puts $a(0)
```
## 文件
```tcl
set f [open "filename" "w/r"]
puts $f "1"
close $f
```
## eval命令
语法：`eval arg1.arg2.arg3,...`  
含义：将所有的参数连起来作为命令语句执行
```tcl
file.txt中存放
1 + 2
4 + 5

set infile [open "/file.txt" "r"]
while {[gets $infile Op]<=0}{
    set Operation "expr $Op"
    set results [eval $Operation]
    puts stdout "$Op = $Results\n"
}
```
动态构造Tcl命令，然后解析并执行。
## error和catch命令
相当于throw expection.
```tcl
proc div {a b}{
    if{$b==0}{
        error "divided by zero"
    }else{
        return [expr $a/$b]
    }
}
div 1 0
```
## upvar和uplevel命令
* upvar可以使用户在过程中对全局变量或者其他过程中的局部变量进行访问
  ```tcl
  proc decer {n m}{
    upvar $n upa
    set upa [expr $upa-$m]
  }
  set nb 12
  decer nb 3
  puts $nb
  ```
* uplevel可以改变上一级栈中的变量值
  ```tcl
  proc ff {}{
      set a ff #设置了局部的a
  }
  set a global
  ff
  puts $a
  输出结果： global
  对比：
  proc ff{}{
      uplevel set a ff # 改变了上一级栈中的a
  }
  set a global
  ff
  puts $a
  输出:ff
  ```

# OTcl简介
## OTcl和C++的区别
* C++在一对{}中完成对一个类的定义，而OTcl则是写成多个分开的部分，每一个方法给类增加一个成员函数，OTcl中成员变量是在成员函数中定义。
* C++中的构造函数和析构函数对应OTcl语言中的init函数和destroy函数。OTcl中必须用next显式地调用父类的构造函数。
* OTcl函数必须通过对象来调用。变量$self和C++中的this相似，表示对象自身。OTcl函数都是虚函数。
* OTcl同样可以继承，，使用next关键字调用父类函数
## 基本语法
使用`instproc`定义类的方法，使用`proc`定义对象的方法，`proc`定义的方法只能用于该对象
### 类的定义和生成
```Otcl
% Class Box //使用关键字Class定义一个类Box
% Box box1  //产生类的对象
% box1 info Class   //使用info命令查看类和对象的关系
% Box info instances
```
产生对象之后对，对对象进行变量定义，也可通过info命令来查看对象的变量
```OTcl
% box1 set length 10
% box1 info vars
length
```
### init函数和destroy函数
```
% Box instproc init {args}{
    $self set length 10
    eval $self next $args
}
% Box instproc destroy {args}{
    puts "The instance is destroyed!"
    $self next
}
```
### 继承
OTcl支持类的多继承，唯一的要求是继承关系满足有向无环图。superclass用于声明继承。
```c++
Class Box
Box instproc init{}{    #定义类的初始函数
    $self set count0
    $self next
}
Box instproc Count{}{   #定义类的成员函数Count
    $self instvar count #重新声明变量count
    incr count
    $self next
}
Safety instproc Count{}{
    $self instvar count
    if{$count == 0}{
        return {zero}
    }
    else{ $count }
    $self next
}

Class redBox -superclass Box    #redBox继承于类Box
redBox redBox1

redBox instproc init {args}{
    $self set toppings{}
    eval $self next $args       #调用父类Box的初始化函数
}

% redBox1 set count
0
% redBox1 Count                 #调用父类Count函数
% redBox1 set count
1
% redBox1 get
1
```