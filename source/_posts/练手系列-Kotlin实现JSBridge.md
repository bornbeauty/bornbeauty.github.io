---
title: 练手系列-Kotlin实现JSBridge
date: 2017-10-14 20:33:53
tags:
    - Android
    - kotlin
---

[TOC]

随着H5的不断普及与发展，为了项目的灵活性，客户端的许多功能被放置在H5页面去完成。所以移动客户端经常需要与H5的页面交互。Android的WebView本身提供了一些接口来实现native与web的交互，例如`shouldOverrideUrlLoading()`、`addJavascriptInterface()`等方法。然而随着业务发展，代码可能会变得越来越臃肿，Android与IOS的接口都是不统一的，难以维护。另外，Android本身提供的接口，可能在低版本上存在漏洞。所以可以采用类似IOS的`JSBridge`方式。这篇文章就试着分析一下它的原理与实现方式。另外，Kotlin已经到来了，还不行动起来？我将会试着用Kotlin重新实现一下JSBridge，试一下Kotlin到底好用不好用，Let's Go!

### 1. 什么是JSBridge？

顾名思义，JSBridge就是在native与web之间建立一座桥梁，方便native调用web端代码，同时也方便web调用native代码。当然，需要我们前后端事先定义通信协议，具体怎么定义我们会在后面提及到。总之，我们想做到的就是通过一行代码实现native与web的通信。

### 2. 原生交互方式的简单总结

Android调用JS的代码的方法：

1. 通过`webview.loadUrl("JavaScript:function()")`实现
2. 通过`webview.evaluateJavascript()`方法实现

JS调用Android方法

1. 通过`webview.addJavaScriptInterface()`来实现
2. 通过`webViewClient`的`shouldOverrideUrlLoading`方法拦截URL实现
3. 通过`webChromeClient`的`onJsAlert()、onJsConfirm()和onJsPrompt()`方法回调拦截JS对话框`onAlert()、onJsConfirm()和OnJsPrompt()`的消息

上面的方法都是开发中经常使用到的方法，还不熟悉的话可以自己在去熟悉一下。需要注意的是，在使用这些方法的时候记得为WebView添加相应的设置，因为为了安全起见，Google默认情况下都是禁止这些方法的。

```

WebSettings webSettings = mWebView.getSettings();

// 设置与js交互的权限
webSettings.setJavaScriptEnable(true);

// 设置允许js弹框
webSettings.setJavaScriptCanOpenWindowsAutomatically(true);

```

### 3. JSBridge的实现原理

有了上面这些基础的知识，原理就很清楚了。我们还是利用原生的方法，只不过通过JSBridge来分装一套标准化的交互接口。通过我们自己定义通信协议来更加规范的实现交互。至于怎么制定我们的交互协议，下面会详细讲到。

#### 3.1 native调用web接口

对于Android调用JS接口，我们可以选用WebView的loadUrl()方法。

#### 3.2 web调用native接口

对于JS调用Android方法，我们可以选用WebViewClient的shouldOverrideUrlLoading回调方法。

#### 3.3 交互协议

要想进行正常的交互，交互协议是必不可少的。首先，我们必须告诉js我们需要调用他的哪个方法，其次要把调用的参数传递过去，一般情况下我们还需要知道js方法执行之后的结果，而且，方法有可能是异步执行的，所以我们还必须将回调的信息也告诉js，方便回调。反过来，js调用我们同样需要告诉我们这些信息。为了方便，我们可以将这些数据分装成Json数据。定义一个实体类`Message`:

```
data class Message(var callbackId: String,
                   var responseId: String,
                   var responseData: String,
                   var data: String,
                   var handlerName: String?) {

    constructor (): this("", "", "", "", "") {}

    fun toJsonString(): String {
        var jsonObject = JSONObject()
        jsonObject.put(STRING_CALLBACK_ID, callbackId)
        jsonObject.put(STRING_RESPONSE_ID, responseId)
        jsonObject.put(STRING_RESPONSE_DATA, responseData)
        jsonObject.put(STRING_DATA, data)
        jsonObject.put(STRING_HANDLER_NAME, handlerName)
        return jsonObject.toString()
    }

    companion object {
        val STRING_CALLBACK_ID = "string_callback_id"
        val STRING_RESPONSE_ID = "string_response_id"
        val STRING_RESPONSE_DATA = "string_response_data"
        val STRING_DATA = "string_message"
        val STRING_HANDLER_NAME = "string_handler_name"
        fun toMessageObject(jsonString: String): Message {
            val jsonObject = JSONObject(jsonString)
            return Message(
                    jsonObject.optString(STRING_CALLBACK_ID),
                    jsonObject.optString(STRING_RESPONSE_ID),
                    jsonObject.optString(STRING_RESPONSE_DATA),
                    jsonObject.optString(STRING_DATA),
                    jsonObject.optString(STRING_HANDLER_NAME)
            )
        }
}
```

在我们调用js接口的时候，就可以这样来：

```
JSBridge://{message}
```

js在收到数据后，将message中的数据解析出来，根据提供的数据，调用相应的方法即可。

### 4. 工作流程

#### 4.1 初始化

首先，在WebView加载完页面后要将`WebViewJavaScriptBridge.js`文件注入到页面中去，并通知WebView进行初始化操作。

![](http://upload-images.jianshu.io/upload_images/711974-01d12a530293e2af.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 4.2 Android调用JS

1. 首先调用JSBridge提供的统一接口`callHandler()`
2. JSBridge进行一系列封装转换，然后通过`loadUrl()`方法调用JS方法
3. JS方法执行，并且将结果通过`shouldOverrideUrlLoading()`方法返回

具体执行流程看下图：

![](http://upload-images.jianshu.io/upload_images/711974-3806ffd4e087d75b.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 4.3 JS调用Android


1. 首先调用JSBridge提供的统一接口`callHandler()`
2. JSBridge进行一系列封装转换，通过`shouldOverrideUrlLoading()`方法来通知Android有新消息需要处理
3. Android通过`_fetchMessageQueue()`方法来获取消息
4. 解析处理响应的消息
5. 将结果返回给JS

具体执行流程看下图：

![](http://upload-images.jianshu.io/upload_images/711974-b51436171ce31ebd.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 5. JSBridge的实现代码简要介绍

代码实现其实并不是很复杂，关键就是定义好相关的交互协议即可。

![](http://upload-images.jianshu.io/upload_images/711974-f91f7cb4acf425fe.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

具体代码查看 [GitHub-KJSBridge](https://github.com/bornbeauty/KJSBridge)（公司网络无法上传GitHub，改天上传= =），项目使用kotlin写的，说一下使用kotlin的感受：

1. 最深的感受就是对空指针的严格控制，如果一个变量或者属性可以为空，则必须声明为`type?`的形式，否则以后赋值为空的时候编译是不通过的，这就几乎可以排除空指针问题，虽然一开始使用起来非常麻烦不方便，但是好处是不言而喻的，而且，这样就会强制我们一开始就对变量的空属性进行思考，让代码更健壮。

2. 代码简洁 再也不用`findViewById`了，支持lambda表达式，支持函数扩展等等，这些都用起来超级方便，熟悉了就爱不释手

3.自建数据类

4.支持高阶函数

5.......

等等等等好多好多好处，而且kotlin完全兼容Java，可以并行存在，最最最关键的是Google官方支持。所以，貌似没有什么理由不用起来了！另外提一点，Android Studio3.0编译速度不是快了一点点，快升级吧~