---
layout: post
title: web知识总结
category: 生活
tags: life
description: 
date: 2016-06-13
---

#### 一.web概述

1. 什么是[web](http://baike.baidu.com/link?url=PX7jKRPjjXP9rsRgWU5mGAQuKAv7X8gGSESqUt7CAsw3jkYIs3YmSp_M8Du7ZOkc8Ql-JnMrYdsCBzWYLGIjBVcOGPIqJPu__GSNkRBSOPG)？

2. B/S,C/S,集中式区别

3. HTTP协议(请求，响应包)
	请求（Request）：客户端向服务器发送的http请求
	响应（Require）：服务器针对客户端发送的请求做的http响应
4. B/S框架

#### 二.web基础html

##### 1. 网页实现登录框，登录密码的html代码

```html

<html>
	<head>
		<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
		<!-- 这段代码可以解决中文乱码问题 -->
	</head>
	<body>
		<form method="post" action="http://127.0.0.1:8008/login.php">
        用户名：
        <input type="text" name="username"/>
        </br></br> <!-- 换行 一个br标签代表一个换行 -->
        密　码：
        <input type="password" name="password">
        </br></br>
        <button type="submit">登录</button>
		</form>
	</body>
</html>
```

效果预览：

![](http://7xjtan.com1.z0.glb.clouddn.com/QQ%E5%9B%BE%E7%89%8720160630175509.png)

form，就是html表单，用于收集用户的各种输入，包括文本，单选框，复选框等等都可以通过form表单来实现。
- form中的method属性：表示向服务器提交输入信息的方式，一般为post或者get方式。区别：post提交的时候输入的内容会被写在http请求的body体中，get提交的时候输入的内容会被写在提交的url中。例如上面的代码当以get方式提交的时候，url会被拼接成"http://127.0.0.1:8008/login.php?name=lyr&password=hellolyr"
- form中的action属性：表示提交的地址

input type的种类：
- text：文本域
- password：密码域
- 复选框：checkbox
- 单选按钮：radio
- 按钮：button


扩展：下拉框

```html
<html>
    <body>
        <form>
            <select name="cars">
                <option value="volvo">Volvo</option>
                <option value="saab">Saab</option>
                <option value="fiat">Fiat</option>
                <option value="audi">Audi</option>
            </select>
        </form>
    </body>
</html>
```

[一个完整安全可靠地登录表单,可不看](http://www.discuz.net/thread-888170-1-1.html)

##### 2. 常见标签

1. <meta />meta标签是对网站发展非常重要的标签，它可以用于鉴别作者，设定页面格式，标注内容提要和关键字，以及刷新页面等等.
    > meta标签分两大部分：HTTP-EQUIV和NAME变量。
    详见[这个页面一楼的回答](http://bbs.csdn.net/topics/50313415)
2. link:链接一个外部样式表.
3. p:标识一个段落
4. a:链接
5. img:图片
```
<!-- 插入一张图片 -->
<img src="图片的地址"/>
```

```
<!--（2）的实例代码： -->
<html>
    <head>
        <link rel="stylesheet" type="text/css" href="../html/csstest1.css" >
    </head>
    <body>
        <h1>我通过外部样式表进行格式化。</h1>
        <p>我也一样！</p>
        </body>
</html>
```

##### 3.跑马灯效果代码

```html
<marquee behavior=scroll>hello lyr</marquee>
```
标签的属性有：
![](http://7xjtan.com1.z0.glb.clouddn.com/paomadeng.png)

#### 三 CSS

```
<html>
    <head>
        <style type="text/css">
            body {background-color: yellow}
            h1 {background-color: #00ff00}
            h2 {background-color: transparent}
            p {background-color: rgb(250,0,255)}
            p.no2 {background-color: gray; padding: 20px;}
        </style>
    </head>
    <body>
        <h1>这是标题 1</h1>
        <h2>这是标题 2</h2>
        <p>这是段落</p>
        <p class="no2">这个段落设置了内边距。</p>
    </body>
</html>
```

效果图：
![](http://7xjtan.com1.z0.glb.clouddn.com/cs.png)

[css常用属性](http://www.cnblogs.com/gaoweipeng/archive/2009/07/02/1515549.html)












