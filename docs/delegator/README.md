### 关于这篇文章的背景
之前了解到的事件代理不多，就像是一个dom将事件委托给另一个dom，又叫事件委托。后来做了个题目，要实现一个类似jquery的事件委托方法，然后认真的了解了一下。然后专注于实现，其实并没有去看jquery的源码，hhh。

发布订阅模式大概是目前前端框架使用的一种最常见的设计模式了，而我目前也只对发布订阅模式有一定了解，其他的设计模式待后续学习整理。

### 关于事件委托实现
在jquery中，实现事件委托只需要下面这一行代码就可以搞定
```javascript
$("#container").on('click', 'item', fn)
```
之前会觉得这么用理所当然，所以并没有想到哪一天我会亲手去实现一个这样的功能，现在机会来了。

题目给的很简单，就是将子节点的事件绑到其父节点上，比如下面这段dom，就是将`<li>`的事件绑定到`<ul>`上，实现点击`<li>`时触发`<ul>`上的事件。
```html
<div  id="container">
    <li class="item" id="item1">
        <span id="btn1">hello</span>
    </li>
    <li class="item">
        <span>world</span>
    </li>
</div>
```

题目的js部分是这样的
```javascript
!function(root, doc){
    class Delegator {
        constructor (selector/* root选择器 */) {
        // TODO
        }

        on (event/* 绑定事件 */, selector/* 触发事件节点对应选择器 */, fn/* 触发函数 */) {
          // TODO
        }

        destroy () {
        // TODO
        }
    }
}(window,document)
```
然后实现一个功能类似上面jquery的事件委托，就像下面这样的
```javascript
var delegator = new Delegator('#container');
delegator.on('click', 'li.item', fnli)
```

忘记一开始想的是什么方法了，反正写到一半的时候突然想起了订阅发布模式（因为刚好那几天在看发布订阅模式），然后开始撸代码。

分解上面的方法，其实可以看作是一个监听器
```javascript
delegator.addEventListener('click', delegatorFn)
```
只不过这个`delegatorFn`有点特殊，它需要遍历这个所有委托‘click’事件给`delegator`的子节点，并执行他们的委托`fnli`

首先，我们把所有的委托当作是一个订阅事件，只不过这个事件里包含了委托者。在`Delegator`对象的原型里增加一个属性`eventObj`，里面存放订阅‘click’事件的所有的委托者和委托事件，结构差不多是这样的
```javascript
this.eventObj.click=[{
    selecter:'li.item',
    callback:fnli
}]
```
那么整个on的方法其实就是委托者`selector`订阅事件并委托给调on的被委托者
```javascript
on (event/* 绑定事件 */, selector/* 触发事件节点对应选择器 */, fn/* 触发函数 */) {
      // TODO
    
      if(!this.eventsObj[event]){
        this.eventsObj[event] = [];
      }
      this.eventsObj[event].push({
          selector,
          callback: fn
      })
      //这里委托事件给this.root，当被委托者坚挺到event事件时，会触发委托函数delegatorFn
      this.root.addEventListener(event, delegatorFn);
      //因为on可以链式调用，所以这里需要返回
      return this;
    }
```
现在需要实现`delegatorFn`方法了，其实也很简单，主要是遍历事件经过的所有dom中哪个`selector`订阅了它，它就执行对应节点携带的`fn`。
```javascript
delegatorFn = (e) => {
    let target = e.target;//触发事件的selector
    let currentTarget = e.currentTarget; //被委托者
        //判断触发事件的节点及它冒泡经过的节点是否是被委托者,若是，则表示不再有委托者，无需遍历
    while(target !== currentTarget){
        this.eventsObj[e.type].forEach(item => {
            //查找订阅事件者
            if(target.matches(item.selector)){
                //执行订阅事件者携带的函数
                item.callback.call(target,e);
            }
        });
        //往上冒泡
        target = target.parentNode;
    }
}
```
以上，就是通过发布订阅实现的事件委托的核心部分，主要涉及的还有事件的冒泡。

### 总结
主要涉及知识点：
* 链式调用：在on方法里返回this
* 事件冒泡：对`e.target`进行往上查找并执行触发事件的回调
* 事件订阅







