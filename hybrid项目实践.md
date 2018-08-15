# Hybrid项目实践

#### 出现的意义
项目在对成本、效率上有高追求而产生的技术，也就是hybrid的优点：

- 跨平台（android和Ios同用一套前端）
- 低成本
- 没有版本问题，有BUG能及时修复
- 一些快速迭代的应用，迫切的解决方案
- 开发效率高（前端开发）


#### 理解定义
hybrid可以是**一种交互协议**，完善前端和native沟通，可以理解为以前只知道native可以调用js，js也可以调用native，我们在这个基础上定义协议，实现我要你做什么，你做了再返回我，反之亦然。有点像http协议，native可以当作是前端的后台。这样可以达到前端接管native绝大部分功能实现，IOS和android共用一套前端和协议，利用native和h5各有的优势，优点也就体现出来。

总结：基于webView前端和native的一种交互协议，

#### 基本内容实现
##### 协议原理
IOS 采用自定义schema 协议，android 采用js注入方式。

（1）native/js 通信机制
对于 Android调用JS代码 的方法有2种：

- 通过WebView的loadUrl（）4.4前  页面会刷新
- 通过WebView的evaluateJavascript（）4.4后 不会使页面刷新

（2）JS调用Android代码 的方法有3种：

- 通过WebView的addJavascriptInterface（）注册java对象，
- 通过 WebViewClient 的shouldOverrideUrlLoading ()方法回调拦截 url
- 通过 WebChromeClient 的onJsAlert()、onJsConfirm()、onJsPrompt（）方法回调拦截JS对话框alert()、confirm()、prompt（） 消息

**注意**：addJavascriptInterface会有安全隐患，在Android4.2js可以通过反射得到Runtime对象,执行命令。 Runtime.getRuntime().exec("exefile")。  但是实际4.2版本的手机很少了可以忽略。

##### 通信协议
1、native/js 通信协议
（1）H5传递给Native参数格式

```
{
    module: moduleName,
    method: methodName,
    apiParams: {},
    // 回调的cbId, 用于和回调Function匹配
    cbId: cbId 
}
```
（2）Native返回给H5的参数格式

```
{
    errorCode: 0,
    data: {},
    // 回调的cbId，用于和回调function匹配
    cbId: 1, 
    emitMessageName: 'networkStateChanged'
}

```
native通过JSBridge.nativeCallback(param)调用JS

2、native调用JS分两种情况
（1）native主动发消息给js

```
param = {
    emitMessageName: "", //消息名称
    data:{}
}

```
（2）native回调js

```
param = {
    errorMsg:"", //调用出错时的错误信息。成功返回空
    errorCode:"", //错误码，具体错误码待native定义
    cbId:"", //js调用native时传入的回调函数guid
    data:{}
}

```
一些Code参数定义,errorCode： 

- 0 代表调用成功
- 102	参数错误或者关键参数缺少
- 104	无权限操作或者未登录
- 101	未知或其它错误
- 107	本地无法完成业务
- 103	本地没有该方法或者方法过期
- 105	用户取消操作
- 108	用户未开启相应权限
- 106	网络错误

3、native H5 跳转
native页面跳转由h5接管，具体通过拦截shouldOverrideUrlLoading方法，解析链接
{schema}://wireless?protocol={$protocol}&url={$url}&target={$target}&showloading={$showloading}&navBarHidden={$hidden}

- schema：app中hybrid协议头， 需要验证
- protocol：file & web & native， 打开线上链接还是本地文件或者native页面
- url: 链接地址，做base64,跳转native页面时，
- showloading: 1 & 0 ，view中是否有native的loading，1 有loading，0没有loading，默认值为0。
- navBarHidden：1 & 0 ，1隐藏navBar,0显示navBar,默认值为0。
- target：new & current ， 新页面还是当前页面, 浏览器唤醒native或hybrid不需要传此参数。
- 新开／回退webview，native支持动画效果。

4、Header 组件
header左侧与右侧可配置，显示为文字或者图标（这里要求header实现主流图标，并且也可由业务控制图标），并需要控制其点击回调（native提供API，如果未注册callback，或者点击回调callback无返回则执行默认方法）
header的title可设置为单标题或者主标题、子标题类型，并且可配置lefticon与righticon（icon居中）

5、具体的整套详细native和h5通信API文档～～（百度云）

#### 项目中的实际使用：

1. webView封装支持：
第一层封装，满足常规webView的使用，如：topBar  progressBar  statusView  ~~
第二层封装，加入hybrid，注册，移除注册，状态同步

2. 单向请求转换双向请求响应
native作为h5的响应方，需要响应h5，这里通过标示ID来识别一次完整的请求响应。
3. 接受消息调用对应的模块的方法
路由表方案： 路由表为配置文件，记录所有提供给H5基础的模块实现类和方法，当接受到请求时，到路由表里找到对应类和方法，通过反射请求。 （扩展路由表）
4. 回退栈管理
通过回退栈管理（每开一个页面 都是一个Activity包含Webview）
Application注册registerActivityLifecycleCallbacks监听activity的生命周期，同时将activity入栈，出栈，在打开新的webview页面时，h5会制定这个页面的name，这里可以做个关联。  等到需要回退的时候匹配到对应的name就行。
5. 缓存优化


#### 参考
[https://www.jianshu.com/p/b9164500d3fb](https://www.jianshu.com/p/b9164500d3fb)
[https://www.cnblogs.com/yexiaochai/p/4921635.html](https://www.cnblogs.com/yexiaochai/p/4921635.html) 