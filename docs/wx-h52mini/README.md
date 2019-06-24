## 关于背景
项目是一个涉及跨移动平台的前端项目，技术使用的是vue，因为项目不是很大，所以没有用到状态管理。

主要是用vue将h5项目编译成静态页面放到服务器的制定地址上，然后在小程序测使用`<web-view>`，将服务器上的地址传给这个标签。

## 关于遇到的坑

### 值传输的坑

因为使用的是`<web-view>`，根据我的理解，这个组件类似于html的`<iframe>`,但又没有`<iframe>`成熟，很多属性都无法使用，基本的比如宽高。下面是`<web-view>`的属性。
属性名 |	类型 |默认值|说明
-------- | ------ | ------ | ------
src	|String		| |webview 指向网页的链接。可打开关联的公众号的文章，其它网页需登录小程序管理后台配置业务域名。 **类似iframe的src**。
bindmessage|	EventHandler		| |网页向小程序 postMessage 时，**会在特定时机（小程序后退、组件销毁、分享）触发并收到消息**。e.detail = { data }
bindload|	EventHandler		| |网页加载成功时候触发此事件。e.detail = { src }
binderror|	EventHandler	| |	网页加载失败的时候触发此事件。e.detail = { src }

上面主要有两个属性可以传值：
>1. `src` 通过`url`的属性值传值。

>2. `bindmessage` 通过`h5`页面向小程序 `postMessage` 传值。

**第一个属性遇到的坑**：
总所周知，url的参数值里不能存在=、?等奇怪的符号，不然会出现很奇怪的参数。比如 `axxx.com?url=bxx.com?a=123`最后拿到的url是什么？肯定不是`bxx.com?a=123`

**解决办法**
>对要传入`<web-view>`的`src`中参数的值要进行`encodeURIComponent()`，然后在小程序中拿值是进行`decodeURIComponent()`，当然，这个需要小程序和H5双方同步（如果小程序和H5是两个项目组时）

**第二个属性遇到的坑**：
这个坑属于无意识的，没有细看这个属性的说明导致，所以看到的人请注意上面表格中加粗的位置，这个属性看起来好像很方便，但小程序这边并**不是同步接收**的，只有在**小程序后退、组件销毁、分享时触发并收到消息**

**解决办法**
>如果是实时同步获取数据的情况，这个属性并没有什么卵用。


### 布局的坑
这个也是为什么我会觉得`<web-view>`类似`<iframe>`但没有`<iframe>`成熟的原因了，`<web-view>`是全屏幕的，整个小程序页面如果要用h5，那这个`wxml`就无法放置更多的组件了，除非用`wx:if`。其实`wx:if`只是为了控制这个页面到底用h5还是原生而已。

因为`<web-view>`占满了页面，所以底部导航最后也只能在h5页面中写了，定位使用fixed。

**遇到的坑：1、fixed布局在小程序中的坑**

忘记是`ios`还是`app`中，当小程序中下拉和上划时，整个页面会看到小程序的容器。这里了解一下小程序容器的结构。
![](https://user-gold-cdn.xitu.io/2018/11/12/1670762d02a395b5?w=448&h=752&f=png&s=83853)
而`<web-view>`属于`page`内容，所以当在小程序中下拉时，会出现`backgroud`部分覆盖`<web-view>`中定位为`fixed`的部分，如下图，红色文字区域部分被覆盖了。

![](https://user-gold-cdn.xitu.io/2018/11/13/1670c7de6bd6991c?w=176&h=305&f=png&s=5172)

**解决方法：**
> 定位方式改为`absolute`，这个可以解决覆盖的问题，但是那一块`div`会在`page`被上划或者下拉时跟随`page`移动，如果在`<web-view>`中使用小程序的导航，那么整个导航条也会跟随`page`移动。看使用场景，如果不介意这种情况的可使用这个解决，但介意的话，我这暂时也没有更好的解决方法。如果有更好的解决方案，还望告知。

**遇到的坑：2、内容高度未达到满屏时无法显示底部的fixed布局的div**

在ios上，不论是h5嵌入小程序还是app，都会出现这样的问题。当内容高度不够时，无法显示底层的`div`

**解决方法：**
> 页面设置高度，使用`css3`的`vh`特性


### jssdk的坑--重点关于支付
如果你和我一样，也是用的h5嵌入小程序，那么可以跳过在h5页面上使用jssdk进行支付操作了，亲测验证不可行。一直坚信，既然h5能调`jssdk`接口，那么一定能够调用支付接口。总在想，h5页面直接调用支付，总是比h5与小程序交互去实现支付要简单和方便的多。然而现实告诉我，不可行。原因是因为`signature`一直报错，后来查了之后，貌似是里面生成`signature`的有个参数与小程序的不一样。具体我也不是很清楚，如果有了解的还望告知一下。因为该方案并没有成功实现微信支付的功能，所以暂不介绍使用方式。
还有一个可能的原因是因为`web-view`组件中的h5页面不支持支付接口，`web-view`网页中仅支持以下JSSDK接口：

| 接口模块 |	接口说明 | 具体接口|
|-------- | ------ | --------|
|判断客户端是否支持js | |		checkJSApi|
图像接口|	拍照或上传|	chooseImage
||预览图片|	previewImage
||上传图片|	uploadImage
||下载图片|	downloadImage
||获取本地图片|	getLocalImgData
音频接口|	开始录音|	startRecord
||停止录音|	stopRecord
||监听录音自动停止|	onVoiceRecordEnd
|| 播放语音	|playVoice
||暂停播放|	pauseVoice
||停止播放|	stopVoice
||监听语音播放完毕	|onVoicePlayEnd
||上传接口|	uploadVoice
||下载接口|	downloadVoice
智能接口|	识别音频|	translateVoice
设备信息|	获取网络状态|	getNetworkType
地理位置|	使用内置地图|	getLocation
||获取地理位置|	openLocation
摇一摇周边|	开启ibeacon	|startSearchBeacons
||关闭ibeacon|	stopSearchBeacons
||监听ibeacon|	onSearchBeacons
微信扫一扫|	调起微信扫一扫|	scanQRCode
微信卡券|	拉取使用卡券列表|	chooseCard
||批量添加卡券接口|	addCard
||查看微信卡包的卡券|	openCard
长按识别|	小程序圆形码|	无

以上并没有可以用的支付接口。若以后`web-view`除了支付接口，则大概可以直接在h5页面上进行支付操作而不用到小程序上操作了。

**解决方法:**

>1.通过h5调用服务器接口获取到支付所需要的参数（这些参数通过服务器去对接支付中心获取到的）

这里需要注意的是服务器获取数据需要openid，而openid需要我们传下去，也就是在调用服务器接口前需要拿到openid。这里的处理是在`<web-view>`的url中将openid作为参数传下去

>2.将这些参数通过url跳转到小程序，将参数传到小程序页面（需要新建一个小程序页面）

跳转使用的接口是`wx.miniProgram.navigateTo`，因为当支付失败时需要返回原来的h5页面，返回用的接口是`wx.navigateBack`,因为返回是在小程序中操作的。
举个粒子，小程序中的页面链接为`/page/pay/pay`,那么h5中进行跳转的操作为
```javascript
const successUrl =encodeURIComponent('https://abc.com/successPage'); //支付成功页
const path = `/page/pay/pay?timeStamp=${timeStamp}&nonceStr=${nonceStr}&package=${package}&signType=${signType}&paySign=${paySign}&successpage=${successUrl}`； //？后面的参数都是支付需要的参数
wx.miniProgram.navigateTo({
   url: path
})
```
>3.小程序页面接受参数并执行支付操作。

小程序中的`/page/pay/pay`页面中进行支付操作,该页面有一个`<web-view>`组件，用来跳转支付成功后的页面也就是参数中的`successpage`
```html
<web-view src='${url}' />
```

```javascript
onLoad: function (options) {
    console.log(options)
    // 获取网页传过来的值
    this.timeStamp = options.timeStamp
    this.nonceStr = options.nonceStr
    this.package = options.package
    this.signType = options.signType
    this.paySign = options.paySign
    this.successUrl = decodeURIComponent(options.successUrl)
    ...
  },
onReady: function(){
  wx.requestPayment({
    timeStamp: this.timeStamp,
    nonceStr: this.nonceStr,
    package: this.package,
    signType: this.signType,
    paySign: this.paySign,
    success (res) { 
        this.url = this.successUrl; //支付成功后会跳转到h5的登陆成功页
    },
    fail (res) { 
        wx.navigateBack();
    }
  })
}
```
关于`wx.requestPayment`，参见[wx.requestPayment](https://developers.weixin.qq.com/miniprogram/dev/api/open-api/payment/wx.requestPayment.html)

这样实现有一个缺点，就是当跳转到小程序页面进行支付时，页面已经换掉了，不再是原来的h5支付页面了，解决也好解决，就是在小程序这边实现一个一摸一样的h5页面视觉，但仍会出现页面刷新跳转的动作。

## to be continued
以上，便是目前记得的所有的坑，后期如果再遇到别的坑，再补充


## 关于文档

[小程序开发文档](https://developers.weixin.qq.com/miniprogram/dev/api/)

[jssdk文档](https://mp.weixin.qq.com/wiki?t=resource/res_main&id=mp1445241432)
