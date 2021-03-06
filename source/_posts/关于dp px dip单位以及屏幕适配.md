---

layout: post
title: 关于dp px dip单位以及屏幕适配
category: 技术
tags: Android
description:
date: 2017-02-11
---

>关于android中常见的一些单位的说明
 
此文是根据慕课网的android视频自主学习整理的，[视频地址](http://www.imooc.com/learn/484)。

#### 一.有关屏幕的重要概念

#####1.什么是屏幕尺寸、屏幕密度、屏幕像素密度？

######a. 

屏幕尺寸就是手机屏幕的对角线长度。
单位是英寸，1英寸 = 2.54厘米

######b. 

屏幕分辨率就是手机屏幕在横纵方向上的像素点数。
单位是px，1px = 1个像素点
一般以纵向像素*横向像素，如1080 * 720

######c.

屏幕像素密度就是指每英寸上的像素点数。
单位是dpi，即“dot per inch”的缩写
屏幕像素密度是由屏幕的尺寸和屏幕的分辨率来决定的
计算方法：拿华为荣耀3c来计算， 屏幕尺寸是5英寸，分辨率是1280*720
它的像素密度 = (1280^2 + 720^2)^(1/2) / 5 = 293.7

#####2.什么是dp、dip、sp、px？他们之间有什么关系？

a. dp和dpi是一回事，名字不一样而已，为了和sp统一，现在多用sp。dpi上面已经解释过了，就不用多解释了。
b. px是构成图像的最小单位
c. sp是字体大小的单位，与缩放无关的抽象像素。sp和dp很相似，单位一的区别就是，android系统允许用户自定义文字的大小（小，正常，打，超大等等），当文字尺寸是正常的时候，1sp=1dp=0.00626英寸，而当文字尺寸是大或者超大的时候，1sp>1dp=0.00625英寸。

#####3.什么是mdpi、hdpi、xdpi、xxdpi？如何计算和区分？

它们都是表示像素密度。
| 名称 | 像素密度范围 |
| :--------------: | :-------------------------: |
| mdpi | 120-160dpi |
| hdpi | 160-240dpi |
| xhdpi | 240-320dpi |
| xxdpi | 320-480dpi |
| xxxdpi | 480-640dpi |

#### 二.怎么适配屏幕

#####1.支持各种屏幕尺寸的方法

a.使用wrap_content、match_parent、weight 
warp_content:就是适配内容的大小
match_parent:就是充满父控件
weight:这个属性有点麻烦，比较难理解，我们举个例子看一下。

```xml

<!-- 我简单表示一下 就不写全了 -->
<Linearlayout>

<Button 
    android:id="@+id/button1"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:layout_weight="1"
    />
    
<Button 
    android:id="@+id/button2"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:layout_weight="2"
    />

</Linearlayout>

```

假如有以上布局，实际长度 = layout设置的长度+剩余的长度*weight权重
屏幕长度为L，则button1和button2的layout设置值都是L，button1的剩余长度就是总长度L-button1的长度-button2的长度，即为L-2L
L1 = L + (L - 2L) * 1/3 = 2/3L
L2 = L + (L - 2L) * 2/3 = 1/3L
这样算出来发现和我们所设置的权重值是相反的，所以我们一般都是设置为0dp
这样算的话是这样的
L1 = L * 1/3 = 1/3L
L2 = L * 2/3 = 2/3L

b.使用相符布局，禁用绝对布局

c.使用限定符-large

就是同一个布局文件同时适配不同大小的屏幕尺寸。
主要是来适配平板的。

d.使用自动拉伸位图

#####2.支持各种屏幕密度

#####3.实施自适应用户界面流程





