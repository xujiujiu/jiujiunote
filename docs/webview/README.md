###  移动应用开发模式
现在的移动应用开发模式有3种：
* `Native App`： 本地应用程序（原生App）
* `Web App`：网页应用程序（移动web）
* `Hybrid App`：混合应用程序（混合App）

现在越来越多的app采用混合模式开发（`Hybrid App`），既有原生app的优良用户体验，又有web app的跨平台优点。
而其核心是使用 `WebView` 控件实现加载url，接下来我总结了关于 `WebView` 的介绍和使用。

### webview的概念

看过一些从app开发者的角度写的关于 `Hybrid App`，但却还没有从前端的角度去理解 `Hybrid App`。

> `webview` 用来展示网页的 `view` 组件，该组件是你运行自己的浏览器或者在你的线程中展示线上内容的基础。使用 `webkit` 渲染引擎来展示，并且支持前进后退等基于浏览历史，放大缩小，等更多功能。
>
> 简单来说 `WebView` 是手机中内置了一款高性能 `webkit` 内核浏览器，在 `SDK` 中封装的一个组件。不过没有提供地址栏和导航栏，只是单纯的展示一个网页界面。

上面这段话是一个app开发者说的，而站在一个前端开发者的角度，使用过后的感受就是：
> `webview` 可以简单理解为页面里的 `iframe` 。原生app与 `webview` 的交互可以简单看作是页面与页面内 `iframe ` 页面进行的交互。就如页面与页面内的 `iframe` 共用一个 `window` 一样，原生与 `webview` 也共用了一套原生的方法。

### 原生app与webview的交互
在项目中，可能会存在原生和 `webview` 属于两个不同的组开发的情况，比如一个app将业务分开，原生部分业务由原生的app开发者开发， `webview` 内业务由前端开发者开发，此时需要原生开发者与前端开发者统一一下交互方式，前端需要告知app哪些方法可以调用，而app需要告诉前端app提供了哪些方法和数据给前端。

原生app分 `android` 和 `ios` 两种应用，原生的写法，这两种应用的开发语言和框架都不同，所以他们与 `webview` 的交互也不同。

#### android 与 webview 的事件交互 
`android` 和 `webview` 的事件交互其实也就是 `android`  和 `webview` 内部的js事件交互，他们的事件交互又分为 `android` 调用js事件和js调用 `android` 事件。

[![](https://user-gold-cdn.xitu.io/2019/2/14/168eab96f406a29e?w=1240&h=378&f=png&s=102289)](https://blog.csdn.net/carson_ho/article/details/64904691/)
[博文--Android：你要的WebView与 JS 交互方式 都在这里了](https://blog.csdn.net/carson_ho/article/details/64904691/)已经将相关 `android` 方法和js使用的方法写的非常浅显易懂了。

我接触到的项目中使用了图片中的两个**方法1**，作为一个前端开发者，在js上只需要调用 `android` 提供的映射对象中的方法和声明一个方法给app调用就ok，可以说是非常的简洁了。

#### ios 与 webview 的事件交互
ios的webview有两种：`UIWebView` 和 [WKWebView](https://nshipster.cn/wkwebkit/)，不同类型的 `webview`
与ios的交互方法也不同：
> 1.拦截url（适用于 `UIWebView` 和 `WKWebView`，只适合简单的调用，如果要传递参数，虽然也可以拼接在url上，如 `jxaction://scan?method=aaa`，但是需要我们自行对字符串进行分割解析信息）
>
> 2.[JavaScriptCore](http://www.cnblogs.com/markstray/p/5757255.html)（只适用于 `UIWebView`，iOS7+）
>
> 3.[WKScriptMessageHandler](https://www.cnblogs.com/markstray/p/5757264.html)（只适用于 `WKWebView`，iOS8+）
>
> 4.[WebViewJavascriptBridge](https://github.com/marcuswestin/WebViewJavascriptBridge)（适用于 `UIWebView` 和 `WKWebView`，属于第三方框架）

这四种方法都有对应的比较好的demo，可以根据需要选择。

接触的项目中使用了 `javaScriptCore`，大概是因为使用这个方法在js中写方法和调用方法都比较简洁和方便吧。

#### 原生与webview的数据交互

上面是事件的交互，那数据的交互呢？

在使用过程中，数据的交互有两种方式：
> 1. `webview` 传入的 `url` 中携带数据
> 2. 在 `webview` 中调用原生提供的映射对象中指定的方法获取。 `webview`  传数据给原生也可以使用该方法，将数据作为参数传递给原生app。

> 注：ios的第一种方法不支持传参给原生

### webview 的属性
就像 `html` 标签都有各自的属性一样， `webview` 也有自己的属性。

android 和 ios 的 webview 有所不同。ios 官方提供了有 [UIwebview](https://developer.apple.com/documentation/uikit/uiwebview?language=objc)（貌似将要被弃用了）和 [WKwebview](https://developer.apple.com/documentation/webkit/wkwebview?language=objc) 的属性。android文档中也提供了 `webview` 相关的[接口](http://www.android-doc.com/reference/android/webkit/WebView.html)。

接触了一下RN的 `webview` 组件，官方提供了丰富的[属性和方法](https://reactnative.cn/docs/webview/)。既然将 `webview` 比作 `iframe`，那就举个例子：像 `iframe` 的 `src` 在 `webview` 中就有对应的 `source` 与其相对应，当然，RN上 `webview` 的属性要丰富的多。

`weex`中也有 `webview`，但它并不是作为一个组件使用，而是作为一个模块控制 `<web />` 组件一起使用。
官方的[demo](http://dotwe.org/vue/a350b7412c5cf140b6ea2a6285d4d11d)感觉很良心了。

最后，上个RN官方的 `webview` 的超简例子

```javascript
import React, { Component } from 'react';
import { WebView } from 'react-native';

class MyWeb extends Component {
  render() {
    return (
      <WebView
        source={{uri: 'https://github.com/facebook/react-native'}}
        style={{marginTop: 20}}
      />
    );
  }
}
```

了解了一波webview，感觉作为一个前端也可以自己写一个app了，用h5写:yum:






