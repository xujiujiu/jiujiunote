# 前言
不知道看了多少篇关于this的文章，很难理解，每次看到的文基本都是说明了`this`的几个不同环境的指向。然后像记住历史一样，记在了脑子里，但这种记忆随着时间的变化会越来越分辨不清。

主要么，也是我没有真正深入的去了解`this`。真的去深入了，才发现`this`涉及的内容真的好多。

# `this`原理

在《高程3》中的p182的第9行写到：*在全局函数中，`this`等于`window`；当函数作为某个对象的方法调用时，`this`等于那个对象；匿名函数的执行环境具有全局性，所以`this`等于`window`。*

这个概念，应该是一个前端必备的知识点吧？那么问题来了，为什么会有这样一个判定呢？

先了解一个概念：

**执行上下文**：也叫一个执行环境，有 **全局执行环境** 和 **函数执行环境** 和 **`eval`**。

每个执行环境中包含这三部分：**变量对象/活动对象，作用域链，`this`的值**


下面这段代码大家肯定都不陌生
```javascript
var obj = {
    name: 'a',
    say: function(){
        console.log(this.name)
    }
}

var say = obj.say;
obj.say();  //行1 a
say();  //行2 undefined
```

以上那几行代码，其实包含了两个我曾经一度没有深入理解的内容：
>1. `this`是在函数运行时基于的函数的执行上下文绑定的。这个时候需要了解一下 [执行上下文的创建过程](https://www.cnblogs.com/Ry-yuan/p/7868029.html)

>2. 对象的值都是存入内存的，而`this`跟内存里面的数据结构有关系，了解一下阮大大的 [javascript 的 this 原理](http://www.ruanyifeng.com/blog/2018/06/javascript-this.html)

#### 对象的内存结构

我这里直接进入对象中的函数的声明与调用，以上文中的`obj`为例。

在创建执行上下文中，变量的声明的步骤依次为：**创建、初始化、赋值**。对象是一种特殊的变量，所以它也存在这3个步骤。

在初始化之后赋值之前，`obj`的值为`undefined`，在赋值操作时，`javascript`引擎会在内存里创建一个下面的对象，然后将这个对象的内存地址赋值给 `obj` 这个变量。

```javascript
{
    name: 'a',
    say: function(){
        console.log(this.name)
    }
}
```
若要读取 `obj.name` ，引擎会先拿到`obj`的内存地址，然后到这个内存地址中找到`name`这个属性的`value`属性。

当给对象赋值的是一个函数时，如上面的`obj.say`的地址，引擎会把函数单独保存在内存中，然后再将函数的地址赋值给`say`属性的`value`属性。

```javascript
{
  name: {
    [[value]]: 'a'
    [[writable]]: true
    [[enumerable]]: true
    [[configurable]]: true
  },
  say: {
    [[value]]: 函数的地址
    [[writable]]: true
    [[enumerable]]: true
    [[configurable]]: true
  }
}
```
大概的内存结构是这样的

![](https://user-gold-cdn.xitu.io/2018/11/10/166fdabe1e24bdb7?w=1520&h=492&f=png&s=38349)

这张图就可以很清晰的看出来了，因为函数是一个独立的地址，那么执行`var say = obj.say;`时，其实是把函数的地址赋值给了`say`，那么`say`就可以独立运行了，当执行`say`时，引擎可以直接拿到函数的地址而不再通过`obj`的内存了。

大概的结构是这样的

![](https://user-gold-cdn.xitu.io/2018/11/10/166fdb8235d9b230?w=1818&h=610&f=png&s=58155)

#### 环境变量
js允许在函数体内部，引用当前环境的其他变量。

比如下面的代码，x代表着当前环境的变量。
```javascript
var f = function(){
    console.log(x);
}
```
那么问题来了，函数可以在不同的环境里运行，那么怎么获取到它的当前运行环境呢？所以，this就出现了，它的设计目的就是在函数体内部，指代函数当前的运行环境。
```javascript
var x = 'a';

var f = function(){
    console.log(this.x);
}

var obj = {
    f:f,
    x: 'b'
}

f();  //a 在全局环境下单独运行，this指代全局环境，全局的x为a
obj.f(); //b  在obj中运行，this指代obj环境，obj的x 为b

```

### this的绑定
常见的有两种，一种是直接在全局运行，函数内部的this直接绑定到全局，一个是通过对象调用，函数内部的this绑定到调用他的对象上。

除了常见的两种，还有两种：
>1.是从一个环境传到另一个环境中，需要使用`call`和`apply`，俗称改变this的指向。

>2.不绑定。箭头函数独有，此时的this继承于作用域链的上一层，这里不免要聊一下作用域链上的this了。

#### call and apply

还是用上面的粒子
```javascript
var x = 'a';

var f = function(){
    console.log(this.x);
}

var obj = {
    f:f,
    x: 'b'
}

f();  //a 在全局环境下单独运行，this指代全局环境，全局的x为a
f.call(obj, x); //b  
f.apply(obj, [x]); //b
```
`call`和`apply`都是将this绑定到它第一个参数上。若其第一个参数不是对象，则会使用js的类型强制将第一个参数转为对象。

#### bind 
`bind`和`call`、`apply`不一样的是，它会生成一个新的函数，且这个新生成的函数的`this`将永久地被绑定到了`bind`的第一个参数。具体原因还不清楚，若有知悉的望告知。

继续上面的那个粒子
```javascript
var g = f.bind(obj);
var obj2 = {
    x: 'c'
}
var h = g.bind(obj2);
g(); //b
h(); //b g的this指向不会被bind改变
```

#### 箭头函数不绑定this
箭头函数不绑定this，此时的this继承于作用域链的上一层
```javascript
var x = 1;
function a(){  //函数作用域上没有x，向上找，最后找到window
    setTimeout(() => {
        console.log(this.x);
    })
}
function c(){  //函数作用域上没有x，向上找，最后找到window
    setTimeout(() => {
        console.log(this.x);
    })
}
var obj = {
    x: 2, //对象的原型链上有x
    c: c
}
var obj1 = {//对象的原型链上没有x
    c: c
}
a(); //1 执行环境是全局
obj.c(); //2 执行环境是对象
obj1.c(); //undefined 执行环境是对象
```
#### 箭头函数上的call和apply，bind
由于箭头函数没有自己的this指针，通过 call() 或 apply()，bind()方法调用一个函数时，只能传递参数，不能绑定this，他们的第一个参数会被忽略。

```javascript
var x = 1;
var a = (b) =>{  //函数作用域上没有x，向上找，最后找到window
    console.log(this.x + b);
}
var obj = {
    b: 2,
    x: 3
}
a.call(obj, obj.b); //3
```

### 文末
因为写到这道题，发现自己并不清楚为什么结果是window，然后翻了下this的原理，算是做一份总结吧。
回顾一下上面的内容，因为js引擎在生成对象的内存时，obj这个变量还未被赋值，所以它的值为undefined，在es5中以明确，当bind第一个参数为undefined时，在浏览器上将执行window。
```javascript
var obj = {
        say: function () {
          function _say() {
            console.log(this)
          }
          return _say.bind(obj)
        }()
      }
 obj.say() //window
```

























