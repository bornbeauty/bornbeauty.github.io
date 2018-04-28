---
layout: post
title: 解决频繁切换Activity带来的视觉冲击问题
category: 技术
tags: Android
description: 
date: 2017-03-13
---

> 如何解决切换activity带来的闪动问题

今天在做项目的时候，有一个需求是使用微信登录，但是在用户授权后activity要频繁切换，使得app看上去有闪动的样子。但是其中的微信的回调页面是
一个不需要显示的界面。所以我将他的布局文件设置成`LoginActivity`文件，这样可能就不会闪动了，但是由于授权是切换到微信去做的，所以这样还会有
app之间切换的动画。那我们干脆就直接让微信的回调页面透明好了。

首先在color文件中定义一个颜色资源文件：

```
<?xml version="1.0" encoding="utf-8"?>
<resources>
  <color name="activity_transparent">#0000</color>
</resources>
```

然后在`style`文件中添加一个`activity`主题样式

```
<resources>
    <!-- 让activity透明的样式 -->
    <style name="ActivityTransparent">
        <item name="android:windowBackground">@color/activity_transparent</item>
        <item name="android:windowIsTranslucent">true</item>
        <item name="android:windowAnimationStyle">@android:style/Animation.Translucent</item>
    </style>
</resources>
```

然后在mainfest文件中给activity设置主题就好了。测试结果还不错~
