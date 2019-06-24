
含义
====

异步编程的一种解决方案。是一个构造对象，代表一个异步操作。有3种状态：

pending（进行中）、fulfilled（已成功）、rejected（已失败）。

状态变更： pending -> fulfilled、pending -> rejected。

状态变更后就**永久**保持这个结果。

基本用法
--------

```javascript
const p = new Promise((resolved, rejected) => {
    if(/*异步操作成功*/) {
        resolved(value);
    }else{
        rejected(error);
    }
});
p.then((value) => console.log('异步执行成功：' + value),  //异步调用回调写法1
    (error) => console.log('异步执行失败：' + error))

p.then((value) => console.log('异步执行成功：' + value))  //异步调用回调写法2
    .catch((error) => console.log('异步执行失败：' + error))
```

pending -> fulfilled执行的结果是resolved，

pending -> rejected执行的结果是rejected。

用法
====

then() catch()
--------------

基本用法中的写法1和写法2的效果一样，主要是因为promise.prototype.then()
有两个参数，第一个参数是 resolved 的回调，第二个参数（可选）是
rejected 的回调。

而promise.prototype.catch()主要用来捕获异步操作抛出的错误，与try{}catch(){}中的catch功能一样。promise.prototype.catch()除了能够捕获异步操作中的错误外，还能捕获.then()回调中的错误。

```javascript
const p = new Promise((resolved, rejected)=>resolved('resolved')).then((value) => {
    throw new Error(value);
}, (error) => {})
.catch((error) => console.log(error))
```

输出结果为
![](https://user-gold-cdn.xitu.io/2018/7/22/164c2b675ce2c88f?w=420&h=37&f=png&s=1838)


另外，**当Promise状态已变成resolved之后再抛出错误，抛出的错误是无效的**，同样以上面的例子为例。

```javascript
const p = new Promise((resolved, rejected) => {
    resolved('resolved');
    throw new Error('hhh');
}).then((value) => {
    throw new Error(value);
}, (error) => {})
.catch((error) =>console.log(error))
```

输出结果

![](https://user-gold-cdn.xitu.io/2018/7/22/164c2b6b3280a2e3?w=470&h=43&f=png&s=1979)

与上面的结果一致。原因:
Promise的状态变更后就**永久**保持这个结果，不会再变化了。
.catch()返回的仍是一个Promise对象，在执行.catch之后，仍可以继续调用。

```javascript
const p = new Promise((resolved, rejected) => {
    resolved('hhh');
}).then((value) => { 
    throw new Error(value);
})
.catch((error) =>console.log(error))
.then(() => console.log('hhhhh'))
```

输出结果：
![](https://user-gold-cdn.xitu.io/2018/7/22/164c2b6d10e9637c?w=222&h=54&f=png&s=1751)

错误捕获
--------

.catch()具有冒泡的性质，会一直向后传递，直到被捕获为止，也就是说错误总会被下一个catch捕获。.catch()也能捕获前一个.catch()抛出的错误。

```javascript
const p = new Promise((resolved, rejected) => {
    resolved('hhh');
}).then((value) => { 
    throw new Error(value);
})
.catch((error) =>{
    console.log(r)})
    throw new Error('hhhh')
.catch((error) => console.log(error))
```

输出结果：

![](https://user-gold-cdn.xitu.io/2018/7/22/164c2b6fc689adc2?w=297&h=76&f=png&s=2215)

**注意：一般不要使用.then()的第二个参数**

举例：
```javascript
p.then((value) => { //写法1
    throw new Error(value);
}).catch((error) =>console.log(error))

p.then((value) => { //写法2
    throw new Error(value);
}, (error) => console.log(error))
```

写法1能够捕获前面then()中抛出的错误，但写法2不行

由于.catch()的冒泡性质，当一个promise函数体未对错误进行捕获并进行处理，运行后不会有任何输出，但是浏览器会打印报错。

![](https://user-gold-cdn.xitu.io/2018/7/22/164c2b7217afc2f1?w=541&h=167&f=png&s=13158)

Node有一个事件**unhandledRejection**，专门用来监听未捕获的reject错误。
```javascript
process.on('unhandledRejection', (err, p) => {})
```

**unhandledRejection**有两个参数：

**err**:错误对象

**p**:报错的Promise实例

Promise.all([])
-----------------

Prmise.all()用于将多个Promise实例包装成一个新的promise实例。

**语法： var p = Promise.all([p1,p2,p3])**

p1,p2,p3必须都是Promise实例。若其参数不是Promise实例，则会通过[*Promise.resolve()*](#_Promise.resolve())将其转为Promise实例再传给Promise.all()

p的状态由p1，p2，p3决定：

1、p1，p2，p3状态都变为Fulfilled，p的状态才变为Fulfilled。

2、p1，p2，p3中有任一个状态变为Rejected，p的状态就变为Rejected。

**注意：若作为参数的Promise实例自身定义了catch，则它被rejected并不会触发Promise.all()的catch()。**
```javascript
const p1 = new Promise((resolved) => {
    resolved('hhh');
}).then((value) => value)
.catch((error) => error);

const p2 = new Promise(() => {
    throw new Error('hhhhh');
}).then((value) => value)
.catch((error) => error);

Promise.all([p1, p2]).then((result) => console.log(result))
.catch((error) => console.log(error));
```
![](https://user-gold-cdn.xitu.io/2018/7/22/164c2b7aae364b3d?w=724&h=78&f=png&s=11020)
输出：走到了Promise的then()，正常输出

Promise.race()
--------------

跟上面的Promise.all()一样，都是将多个Promise实例包装成一个新的Promise实例
[*Promise.resolve()*](#_Promise.resolve())

**语法： var p = Promise.race([p1,p2,p3])**

p的状态由p1，p2，p3中最先改变状态的实例决定。

Promise.resolve()
-----------------

**作用：** 将先有的对象转为Promise对象。
```javascript
let foo = () => console.log('foo')
Promise.resolve(foo)
```

等价于
```javascript
new Promise((resolve) => resolve(foo));
```
**参数：**

1、Promise实例：不做任何修改，直接返回该实例

2、带有then()的对象：将这个对象转为Promise对象并立即执行then方法。
```javascript
let foo ={
    then : (resolve) => resolve('foo')
}
let p1 = Promise.resolve(foo)
p1.then((value) => console.log(value))
```

这里会输出'foo'，因为p1会立即执行foo的then方法，执行后p1的状态才变为resolved。

这个用法比较常用，当异步调用多个接口改变数据，但后面需要同时处理改变后的这些数据时，可结合Promise.all和Promise.resolve()来使用。

3、参数不是对象

Promise.resolve('foo');

该用法在Promise实例生成时状态就为Resolved，所有回调函数会立即执行，并将参数传给Promise的回调函数。

4、不带参数：返回一个空Promise实例

**注意：**Promise.resolve()是在**本轮"事件循环"结束时**执行

![](https://user-gold-cdn.xitu.io/2018/7/22/164c2b7eab318f83?w=495&h=124&f=png&s=6655)

因为setTimeout是在下轮“事件循环”开始时执行，console.log(3)立即执行，因此有了这样的输出结果。

Promise.reject()
----------------

返回一个新的Promise实例，状态为rejected。

该方法的参数会被原封不动的传给Promise.catch()作为参数。不区分该参数类型。

.done()
-------

处于回调链的低端，保证能够接收抛出的任何可能的错误。

.finally()
---------

用于指定不管Promise对象最后状态如何都会执行的操作。它接收的回调函数作为参数。

比如：调用接口时会对接口做处理，不论接口调用成功还是失败，都要做出提示，那么就可以用这个方法用来做提示的回调。

.try()
------

实际中会遇到不知道或者不想区分回调函数是同步的还是异步的，但是想用Promise

处理，因为这样就可以**不管回调函数是同步还是异步，都能使用then()指定下一步操作，用catch捕获错误。**

Promise.try()的作用就是解决上面的问题。

以上，属于个人学习promise之后对promise的简单理解。
