---
title: JavaScript变量提升与函数提升
tags: ['JavaScript']
categories : JavaScript
---

在遇到这个问题之前，一直感觉自己的基础挺扎实的，可惜在今年开始差不多荒废了大半年的时间。归根结底还是不够自律，公司项目做的差不多了，人员配备的又过剩，所以有很多时间用来做自己的事情。而我的自己的事情肯定是跟学习某个框架或者打牢技术基础没有半毛钱关系的。[惭愧脸~]

### 闲话不多说，先抛出问题

判断下列式子的执行顺序？

````js
  alert(a)
  a();
  var a=3;
  function a(){
    alert(10)
  }  
  alert(a)
  a=6;
  a();
 
````

 <!-- more -->

 看起来很简单的问题，手残粘到控制台执行看了一下结果，WHAT？ 竟然跟我想象的不一样，突然后背发凉，这是荒废了多久啊！基础知识都忘完了！是准备回去搬砖的节奏吗？于是乎，查了各种基础知识，连那本沉睡很久的红宝书都拿出来翻了翻 ~

### 下面是解决问题的思路。[前方高能，慎入~~]

#### JavaScript中的函数提升

**在ES6之前，JavaScript没有块级作用域(一对花括号{}即为一个块级作用域)，只有全局作用域和函数作用域。变量提升即将变量声明提升到它所在作用域的最开始的部分。**

> 创建函数有两种形式，一种是函数声明，另外一种是函数字面量，只有函数声明才有变量提升

```js
  console.log(a)  // f a() { console.log(a) }
  console.log(b) //undefined
      
  function a() {
          console.log(a) 
  }

  var b = function(){
          console.log(b)
  }

```
上式结果可以看出，使用函数字面量方式声明的函数无法提升，也就是说上式相当于 -->

```js
  var a = function() {}
  var b 
  console.log(a)
  console.log(b)

```

#### JavaScript中的变量提升

```js

  console.log(c);   //undefined
  var c = "xxxx";
  console.log(c);   // xxxx
  function fn(){
    console.log(d); //undefined
    var d = '和前面的一样啊';
    console.log(d); //和前面的一样啊
  }

  fn();

```
由结果可以转化一下思路

```js
  var c ;
  console.log(c)
  c = " xxxx "
  console.log(c)

```

### 函数提升与变量提升的优先级

上面的讨论验证了函数及变量提升的过程。那么，当函数和变量同时存在的时候，提升的优先级是什么呢？这里回到了最初的问题。

```js
  alert(a)  // function a() {alert(10)}
  a();  // 10
  var a = 3;
  function a(){
    alert(10)
  }  
  alert(a) // 3
  a = 6;
  a(); // a is not a function

```
执行这段代码验证发现，在变量和声明式函数同时存在的时候，函数的提升优先级是高于变量的，且不会被变量声明覆盖，但是会被变量赋值之后覆盖。也就是相当于

```js
 var a = function() { alert(10) }
 alert(a)  // function a() {alert(10)}
 a()  // 10
 var a = 3
 alert(a) // 3
 a = 6 
 a()  // a is not a function

```

