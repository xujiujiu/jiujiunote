类即对象

Class基本使用
=============

`es6`
代码调试最简单的方法：直接在**node控制台运行**js，完全不需要考虑浏览器兼容性而去安装babel等编译器的去编译后再调试

先举个例子：

```javascript
class a{
    
  constructor(){

    this.aa = false;
    this.ab = [];

  }

  start(){

    this.aa = true;

  }

}
```

一个类就这么产生了，只要 **`new`** 一下，这个类就可以被用了

```javascript
let b = new a();
```

可能这个类将会有很多地方要使用，那么将它模块化就行了

```javascript
export default a;
```

怎么用这个模块呢？es6的初学知识，**`import`**
就能搞定，路径是以上模块文件的路径

```javascript
import a from '../a.js';
```

模块已经被导入了，那么怎么用这个模块里的类呢？

```javascript
let b = new a();
```

emmmm，看着很眼熟啊，是不是跟一个类的引用一样？就是一样。接下来就可以用这个b干任何你想干的事了。

那么以上，一个类从出生到开始生产就介绍完了。

介绍
----

接下来可以详细介绍一下js的新星：**`Class 类`**

`Class 类`是ES6引入的一个概念，用来作为对象的模板，其实Class的绝大部分功能ES5都有，只不过Class让对象原型的写法更加清晰了。将上面的例子与下面的ES5的写法对比一下便可以看出。

下面我们将上面那个Class a 转成ES5的写法：

```javascript
function a(){
  this.aa = false;
  this.ab = [];

}

a.prototype.start = function(){
  this.aa = true;
}
```

这里可以看出来，`start`这个自定义属性，在ES5中需要通过`prototype`去实现。

`Class`的用法跟ES5上相比没有较大差别，比如都是用 **`new`** 去实例化。

![](https://user-gold-cdn.xitu.io/2018/5/20/1637d880bc03d011?w=376&h=97&f=png&s=2486)
分别运行`Class a `和ES5下的`function
a`，运行上面的代码，发现它们的构造函数都是本身，也就是说，Class本身就指向构造函数，function
本身同理。

使用
----
 
### prototype

继续看上面的例子，a有一个属性是`start`，比较Class和ES5的写法，可以看出，类上面的`start`是定义在类的`prototype`上的。这也是`Class`的一个特性，即类上的所有方法都是定义在对象的prototype上的，但是不需要在代码上标明。

但若需要重写`start`，仍需要通过`prototype`进行覆盖



仍使用最上面的这个例子，虽然Class的属性是定义在prototype上的，但是却是**不可枚举**的。

**`Object.keys(obj)`**，返回一个数组，数组里是该obj可被枚举的所有属性。
**`Object.getOwnPropertyNames(obj)`**，返回一个数组，数组里是该obj上所有的实例属性。

es6

![](https://user-gold-cdn.xitu.io/2018/5/20/1637d885b31024c1?w=512&h=109&f=png&s=3112)

es5


![](https://user-gold-cdn.xitu.io/2018/5/20/1637d88860634007?w=485&h=109&f=png&s=4124)

### constructor方法

**`constructor`方法**是类的构造函数是默认方法，通过**`new`**命令生成对象实例时，自动调用该方法。一个类必须有`constructor`方法，如果没有显式定义，一个默认的`constructor`方法会被添加。

类的构造函数，不使用**`new`**是无法调用的。但ES5是可以的，不妨回忆一下function的调用

与ES5相同，类的所有实例共享一个原型对象


![](https://user-gold-cdn.xitu.io/2018/5/20/1637d88a7ba0ee11?w=530&h=184&f=png&s=3017)

### 不存在变量提升

总所周知，ES5的变量提升是非常常见的，因此，在ES5中，并不需要关注function的定义位置，但是类**不存在变量提升**，必须先定义才能使用，**new必须在Class之后。**

### 私有方法

在类里声明的类似上例中的condtructor和start，都是共有属性，外部都是可以调用的，若需要私有方法，可将方法移出模块定义，如下面的aa()就是私有方法。

```javascript
class a{

  constructor(){

    this.aa = false;
    this.ab = [];

 }

  start(){

    this.aa = true;

  }

  stop(){

    aa();

  }

}

function aa(){

  console.log('private')

}
```

在类中，无法像ES5一样能够直接通过`var` 或者`function`
声明私有方法，但也可以通过上述操作模拟实现私有方法。

Class静态方法
=============

类相当于实例的原型，所有类中定义的方法，都会被实例继承。如果在一个方法前，加上**`static`**关键词，则表示该方法不会被实例继承，而是直接通过类来调用，这称为**“静态方法”**。

```javascript
class a{

  constructor(){

    this.aa = false;

    this.ab = [];

  }

  static start(){

    this.aa = true;

  }

}
```

这个时候生成一个实例b，则b无法使用`start()`


![](https://user-gold-cdn.xitu.io/2018/5/20/1637d88e898d7575?w=587&h=206&f=png&s=7055)

看到这里，是不是觉得好像可以用作上面的私有方法？但是，可以在原型a中使用啊。。。


![](https://user-gold-cdn.xitu.io/2018/5/20/1637d895a50038d0?w=433&h=61&f=png&s=799)

Class 静态属性
==============

看了上面的静态方法，是不是会想到一下静态变量？比如这样：

```javascript
class a{

  constructor(){
    this.aa = false;

    this.ab = [];

  }

  static aaaaa = "";

  static start(){

    this.aa = true;

  }

}
```
但是，ES6目前只存在静态方法，不存在静态属性，ES7上有静态属性的相关提案，写法就如想象中的aaaaa。
以上，属于个人学习es6的class的之后的简单理解，若有错误请指出。
