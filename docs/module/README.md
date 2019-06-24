### 前言
一直感觉模块化是一个很神秘的概念，因而也很感兴趣，有幸了解到了模块化的历史，尝试着了解了一下模块化的实现，发现了一个很有意思的东西，不知道为什么，会觉得很有成就感，记录一下。

### 模块化的实现
简单模拟一下CMD的module实现，在module.js中做以下处理
```javascript
//module.js
exports.word = 'hello'
module.exports = function () {
  console.log(exports.word)
}
```
然后在test.js界面引入该模块
```javascript
//test.js
const file = require('./module.js')
file() // hello
console.log(file.word) //undefined
```
不考虑为什么会有这样的结果，上面的这部分是通过require引用模块的常见写法。下面的超简化版本代码实现了超简化的require方法（下面的runner方法）
```javascript
//require.js
const fs = require('fs')
const path = require('path')

function runner(file) {
  const code = fs.readFileSync(path.join(__dirname, file), 'utf-8')
  const module = { exports: {} }
  const fn = new Function('module', 'exports', code)
  fn(module, module.exports)
  return module.exports
}
```
这里的fn整理出来就是下面这一段代码
```javascript
function _fn(module, exports) {
  exports.word = 'hello'
  module.exports = function() {
    console.log(exports.word)
  }
}
```
所以在执行`fn(module, module.exports)`时，就是对上面声明的`const module = { exports: {} }`进行赋值。当执行runner方法时，其实就是获取`module.exports`的值。而runner方法做的事，就是获取文件中的内容，识别`module.exports`，并把该值抛出来。转到require，其实主要做的也就是这部分工作。module和 export也就成了关键词，用来在读取模块的时候识别的标识。

runner方法已经实现了，现在来看一下为什么运行后是这样的结果。

#### export 与 module.export
通过上面`runner`的实现，也可以看出，其实`exports`和`module.exports`最后被引用后其实是一个。最后暴露出来的都是`module`对象，而如果一个页面中同时存在`exports`和`module.exports`，最后一个引用都会覆盖掉掉前面所有的`exports`引用，所以这也是为什么上面的`file()`能执行成功，而`file.word`为`undefined`，因为前者覆盖了后者。`file`已经被替换成了下面这个方法
```javascript
function() {
    console.log(exports.word)
}
```
#### 模块化
那么什么叫模块化呢？我的理解有3点：
1. 引用的时候能按需调用
2. 外部调用无法修改内部参数
3. 没有全局变量的污染，方法名不冲突

因为看了模块化的历史，了解到模块化最最原始是从匿名闭包衍生出来的，再看看现在的module的实现，其实也是一个闭包。虽然之前看了很多闭包的概念，但因为使用上的局限性，一直把闭包误解为只有return出去的才是闭包。比如下面这段代码
```javascript
function fo(){
    var a = 'aaa';
    return function(){
        console.log(a)
    }
}
var bar = fo();
```
这次看到这个是真的有刷新我的认知，原来下面这样的也是闭包
```javascript
var b = {
    a: {}
}
function fo(obj){
    let a = 'aaa';
    obj.a = function(){
        console.log(a)
    }
}
fo(b.a);
```
把上面这个再扩展一下就成了下面这段代码，一段类似_fn的代码
```javascript
var b = {
    a: {}
}
function fo(b, a){
    a = '111';
    b.a = function(){
        console.log(a)
    }
}
fo(b, b.a);
```
重复一波闭包的概念：**能够读取其他函数内部变量的函数**。

所以这也是为什么上面的`file()`的结果是`hello`,这也是requireJS的模块化里面我暂时接触到的最有意思的地方。下次瞅一瞅es6的模块化


### 相关知识
模块化的历史：http://huangxuan.me/js-module-7day/#/


