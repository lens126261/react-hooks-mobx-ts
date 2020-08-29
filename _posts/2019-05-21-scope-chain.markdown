---
title: "深入学习javascript之作用域链和作用域"
layout: post
date: 2019-05-21 09:44
image: /assets/images/markdown.jpg
headerImage: false
tag:
    - markdown
    - elements
star: false
category: blog
author: Lens
description: js基础之作用域
---

## 作用域(scope)

#### 1、什么是作用域？

可以把作用域看作一个地盘，在这个地盘内声明的变量或方法，在作用域外是无法访问的。作用域起到了一个很好的保护和隔离的作用。

```
function fun (){
    var name = "lp"
}
console.log(name) // 因为name是在fn中，具有私有的作用域，所以在函数外部无法访问
```

#### 2、全局作用域

-   全局作用域中声明的变量或方法，在任何地方都可以被访问到
-   window 对象的内置属性都拥有全局作用域

#### 3、函数作用域

-   声明在函数内部的变量或方法，只在函数作用域中可以访问到

```
var a = 1;
var x = function () {
  console.log(a); // 函数x是在函数f的外部声明的，所以它的作用域绑定外层，内部变量a不会到函数f体内取值，所以输出1，而不是2
};

function f() {
  var a = 2;
  x();
}

f() // 1
```

-   创建函数的时候，就定义了函数的作用域

```
var a = 1;
var x = function () {
  console.log(a); // 函数x是在函数f的外部声明的，所以它的作用域绑定外层，内部变量a不会到函数f体内取值，所以输出1，而不是2
};

function f() {
  var a = 2;
  x();
}

f() // 1
```

-   内层作用域的可以访问外层作用域中的变量，反之则不行

```
function fun1 (){
    var a = 0;
    function fun2(){
        var b = 1;
        console.log(a)   // 0
    }
    console.log(b) // 无法访问到b
}
fun1()
```

-   函数内部未用 var 声明的变量，由于变量提升的缘故，相当于在全局作用域中声明了全局变量

```
function fun (){
    a = 1
}
fun()
console.log(a) // 1
```

#### 4、块级作用域

-   在 ES6 之前，并不存在块级作用域

```
if (true) {
    var name = 'lp'; // name 依然在全局作用域中
}
console.log(name); // "lp"
```

-   ES6 新增 const、let，const/let 声明并不会被提升到当前代码块的顶部

```
if(true){
    const name = "lp"
}
console.log(name) // undefined
```

-   const/let 禁止在同一作用域内重复声明，在不同作用域内可以

```
同一作用域
var name = "lp";
let name = "hhh"; // Uncaught SyntaxError: Identifier 'name' has already been declared

不同作用域
var name = "lp";
if(true){
    let name = "hhh"
}
```

## 作用域链(scopeChain)

JavaScript 上每一个函数执行时，会先在自己创建的 AO（活动对象）上找对应属性值。若找不到则往父函数的 AO 上找，再找不到则往上一层的 AO 找,直到找到全局作用域（window）而这一条形成的“AO 链” 就是 JavaScript 中的作用域链。

> AO 概念参考：https://juejin.im/post/6844904182521020423

```
var a = 1;
function fn1() {
    var b = 2;
    function fn2() {
        var c = 3
        console.log(a)
        console.log(b)
        console.log(c)
    }
    F2()
}
F1()
```

![](https://user-gold-cdn.xitu.io/2020/6/7/1728e84f3fa7b374?w=1220&h=814&f=png&s=71200)

参考：

《JavaScript 高级程序设计》

https://juejin.im/post/6844903875456008199#heading-17
