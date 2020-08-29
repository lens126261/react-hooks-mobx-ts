---
title: "面向对象及原型和原型链"
layout: post
date: 2019-07-21 18:20
image: /assets/images/markdown.jpg
headerImage: false
tag:
    - markdown
    - elements
star: false
category: blog
author: Lens
description: js基础之面向对象
---

## 面向对象编程

何为对象？万事万物我们都可以把它看作对象，对象就是对实物的抽象。

面向对象编程就是将我们的需求抽象成一个对象，对这个对象进行分析，在这个对象上添加属性和方法，来模拟事物之间的关系。这个对象我们一般把它叫做`类`。

## 类和实例

JavaScript 中每一个数据类型都有自己对应的类别：Object、Array、Number、String、Boolean 等

```
const arr = [1,2]
```

arr 即为 Array 这个类的一个实例

**但在开发中，JavaScript 内置的类是远远不够的，如何自定义类呢？**

```
const Animal = function(){
    this.age = 0;
}
```

Animal 即为创建的一个动物类，为了区分普通函数，类名首字母一般大写。在这个类上通过 this 定义了一个 age 属性。

构造函数和普通函数执行有什么区别呢？

```
const a = new Animal()
console.log(a,window.age)  // Animal {age: 0}  undefined

const b = Animal()
console.log(b,window.age)  // undefined  0
```

构造函数执行：new Animal()

-   首先默认创建一个对象（这个对象就是当前类的实例）
-   让上下文中的`this`指向这个对象
-   在没有 return 的情况下返回实例对象：Animal {age: 0}
-   return 的值为基本类型则还是返回实例对象
-   return 的值为引用类型则返回该引用类型值

普通函数执行：Animal()

-   this 指向 window
-   在没有 return 的情况下返回 undefined，有 return 的情况返回 return 的值

## 原型和原型链

在上文中我们创建了类和实例，如果我们想给多个实例对象同时添加某个效果相同的方法呢？

比如给 Animal 的两个实例对象 dog 和 pig 添加一个 say 方法，在没有 prototype 的情况下我们可以这样写：

```
const dog = new Animal()
const pig = new Animal()

dog.say=function(){
    console.log("say hello")
}
pig.say=function(){
    console.log("say hello")
}
```

两个 say 方法虽然名字相同，实现的效果也相同，但是各自分别占有一块堆内存。
试想以下，如果我们要给很多个实例对象添加 say 方法呢？内存岂不是要爆掉。

别着急，js 为我们提供了 prototype 方法，我们可以这样写：

```
Animal.prototype.say=function(){
    console.log("say hello")
}
```

prototype 作为一个对象，存放公有的属性和方法，并将其放在实例对象的构造函数上

```
console.log(Animal.prototype)
```

![](https://user-gold-cdn.xitu.io/2020/7/27/1738eb7e0813f2dc?w=342&h=130&f=png&s=7194)
在实例对象中我们可以通过**proto**获取所属类的 prototype 上公有的的属性和方法：

```
Animal.prototype===dog.__proto__
console.log(dog)
```

![](https://user-gold-cdn.xitu.io/2020/7/27/1738eba8e9cfe6cf?w=301&h=152&f=png&s=6681)
可以看到实例对象上有一个 constructor 属性，它指向 Animal 类。

```
dog.__proto__.constructor===Animal
Animal.prototype.constructor===Animal
```

总结一下：

-   每一个函数（不包含箭头函数）都有一个内置属性 prototype（原型属性），属性值是一个对象，存放公有的属性和方法。所以说 prototype 就是为当前类所属实例提供公共属性和方法的地方。
-   每一个对象（普通对象、实例对象、prototype、函数等）都有一个**proto**属性（原型链属性），属性值是当前实例所属类的原型(类的 prototype 属性值)。
-   每个对象上都有一个 constructor 属性，指向类本身
-   原型链机制：调用当前实例对象的某个属性，先看自己私有属性中是否存在，存在就调用自己私有的，不存在则默认按照**proto**找所属类的 prototype（原型属性）上的共有属性和方法，如果还没有，则基于 prototype 上的**proto**属性继续向上查找，直到找到 Object.prototype 为止

## 在内置类的原型上扩展方法

上文中我们了解了原型 prototype 其实就是一个对象，这个对象里有内置的一些属性和方法，那我们是不是可以往 prototype 上添加一些扩展方法呢？答案当然是可以的

举个例子：

数组去重我们在开发中经常用到，我们可以在 Array 的原型上扩展一个去重方法：

```
Array.prototype.myRemoveRepeat = function removeRepeat() {
    // this指向调用该方法的实例对象
    var obj = {};
    for (var i = 0; i < this.length; i++) {
        var item = this[i];
        if (typeof obj[item] !== 'undefined') {
            this[i] = this[this.length - 1];
            this.length--;
            i--;
            continue;
        }
        obj[item] = item;
    }
    obj = null;
    return this // 实现链式写法，返回数组可以调用数组类上的方法
};
var arr = [1,3,5,2,1,3];
arr.myRemoveRepeat()
```

扩展的方法名字最好加上前缀，如：myxxx，防止自己的扩展方法替换了内置方法。

> **注意**：内置类原型上的属性是不可枚举的，但是在内置类原型上扩展的方法是可以枚举的

对于一个对象来说，它的属性方法存在枚举的特点：在`for in` 循环的时候能否遍历到，能遍历到的是可枚举的，不能遍历到的是不可枚举的

```
let obj = {
	name: "lp",
    age: 12
}
for (let key in obj){
	console.log(key) // name age 是可枚举的，但Object原型上的方法不可枚举
}

// 在内置类原型上添加方法
Object.prototype.a = function a(){};
for (let key in obj){
	console.log(key) // name age a    a也成为可枚举的
}
```

所以一般在用`for in`遍历对象的时候，要用`hasOwnProperty`来检测是否是自己私有的属性

```
let obj = {
	name: "lp",
    age: 12
}

Object.prototype.a = function a(){};

for (let key in obj){
	if(obj.hasOwnProperty(key)){
    	console.log(key) // name age    此时a被过滤掉了
    }
}
```

## new 的过程中发生了什么？

1、创建一个实例对象

2、让实例对象的`__proto__`指向构造函数的`prototype`

2、把类当作普通函数执行，同时 this 指向实例对象

3、看一下函数执行是否存在返回值，不存在或者返回的是基本数据类型，则默认返回实例对象，如果返回的是引用类型则返回该引用类型值

**了解了 new 的过程中发生了什么，下面我们来模拟实现一下：**

```
function _new(Func, ...args) {
  // 创建一个实例对象
  let obj = {};
  // 让实例对象的__proto__指向构造函数的prototype
  obj.__proto__ = Func.prototype;
  // 把类当作普通函数执行，同时将构造函数中this指向实例对象
  let result = Func.call(obj,...args);
  // result是引用类型则返回当前result，否则返回实例对象
  return typeof result === 'object' || typeof result === 'function' ? result : obj
}
```

实现完成，我们来验证一下：

```
const Animal = function(){
    this.age = 0;
}

let dog = _new(Animal)
console.log(dog.age)
```
