---
title: "浏览器底层机制：堆栈内存和VO、GO"
layout: post
date: 2019-05-28 15:30
image: /assets/images/markdown.jpg
headerImage: false
tag:
    - markdown
    - elements
star: false
category: blog
author: Lens
description: 浏览器底层机制
---

## 一、栈、堆以及 GO、VO 的概念

### 1、执行环境栈（Execution Context Stack）

-   浏览器会从计算机内存中分配一块内存，专门用来供代码执行
-   javascript 是单线程的，这意味着他只有一个调用栈
-   基础数据类型存储在栈中

### 2、堆内存（Heap）

-   存放属性方法
-   任何开辟的堆内存都有一个 16 进制的内存地址，存储在栈中，供变量调用
-   引用数据类型存储会开辟一个堆内存，把内容存进去，然后把地址放到栈中供变量关联使用

### 3、全局对象（Global Object）

-   他是一个堆内存，存储的是浏览器内置的属性和方法
-   浏览器中 window 指向全局对象

### 4、VO（Varibale Object）

-   变量对象，在当前的上下文中，用来存放创建的变量和值的地方
-   每一个执行上下文中都有一个自己的变量对象
-   函数私有上下文中叫做 AO（Activation Object）活动对象

```
function test(){
    function a(){};
    var b;
}
test();

该函数的活动对象：
testAO: {
    arguments: { ... };
    a:function(){};
    b:undefined
}
```

### 5、执行上下文（Execution Context）

-   全局执行上下文：当 JavaScript 引擎第一次执行你的脚本时，它会创建一个全局的执行上下文并且压入当前执行栈
-   函数执行上下文：函数调用时会创建一个私有上下文，并压入栈的顶部，javascript 引擎会执行那些执行上下文位于栈顶的函数。当该函数执行结束时，执行上下文从栈中弹出，控制流程到达当前栈中的下一个上下文。
    ![](https://user-gold-cdn.xitu.io/2020/6/6/172896492a4fa926?w=390&h=259&f=gif&s=11698)

## 二、代码执行的过程

```
var a = {n: 10}
var b = a;
b.n = 11;
console.log(a.n)

function fun (){
    var x = "123"
    console.log(x)
}
fun()
```

![](https://user-gold-cdn.xitu.io/2020/6/6/172899a7f6a48f6d?w=1410&h=1056&f=png&s=104841)
1、创建一个全局执行上下文，并将它压入执行环境栈

2、全局执行上下文中代码执行

```
var a = {n: 10}
开辟一块堆内存，存放n: 10，生成一个16进制的地址AAAFFF000，存储在栈中供变量调用
创建变量a
将变量a和AAAFFF000关联在一起

var b = a;
创建变量b，并将b和同一个堆内存地址AAAFFF000关联起来

b.n = 11;
更改堆中的值
由于b和a指向的是同一个堆内存，所以a和b的值都变成了{n: 11}

声明函数，将函数中的代码块存储在堆中
```

3、函数执行，创建新的私有执行上下文，并压入栈的顶部执行，把全局上下文压到栈的底部。函数执行过程：

-   构建活动对象
-   初始化作用域链（scopeChain）
-   初始化 this 指向
-   初始化实参集合
-   行参赋值
-   变量提升
-   代码执行

4、函数执行完弹出私有上下文

4、弹出全局执行上下文会在页面关闭时弹出销毁

参考：

[函数执行过程](https://juejin.im/post/6844903636569440270#heading-4)
