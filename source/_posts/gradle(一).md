---
layout: post
title: gradle在Android中的使用(一)
category: 技术
tags: Android
description: gradle的使用
date: 2017-07-13
---

> google发布了Android Studio，并且一同发布了编译的新的方法-gradle，今天看一下这个怎么使用

# 1 gradle文件中参数解析

我们以project当中的app Moudle中的gradle文件为列

```
// 声明是Android程序
apply plugin: 'com.android.application'

android {
    // 编译SDK的版本
    compileSdkVersion 21
    // build tools的版本
    buildToolsVersion "21.1.1"

    defaultConfig {
    	// 应用的包名
        applicationId "me.storm.ninegag"
        //最低兼容版本
        minSdkVersion 14
        //编译版本
        targetSdkVersion 21
        //应用的版本号
        versionCode 1
        //版本号
        versionName "1.0.0"
    }

    // java版本
    compileOptions {
        sourceCompatibility JavaVersion.VERSION_1_7
        targetCompatibility JavaVersion.VERSION_1_7
    }

    buildTypes {
        debug {
            // debug模式
            // 这样就可以在手机上安装一个正式版 一个debug版本
            applicationIdSuffix ".debug"
        }

        release {
            // 是否进行混淆
            minifyEnabled false
            // 混淆文件的位置
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.txt'
        }
    }

    // 移除lint检查的error
    lintOptions {
      abortOnError false
    }
}

dependencies {
    // 编译libs目录下的所有jar包
    compile fileTree(dir: 'libs', include: ['*.jar'])
    compile 'com.android.support:support-v4:21.0.2'
    compile 'com.etsy.android.grid:library:1.0.5'
    compile 'com.alexvasilkov:foldable-layout:1.0.1'
    // 编译extras目录下的ShimmerAndroid模块
    compile project(':extras:ShimmerAndroid')
}

```

以上就是一个gradle文件中大体的参数

# 2 使用gradle编译apk文件

我们一般会去github浏览一些开源的项目。看这些项目一般都是看源码和看运行的demo。有的项目提供了demo的apk我们可以直接下载安装，有的没有提供，就不得不自己编译apk了。当然最简单的方法就是把源代码导入Android Studio中编译，但是AS非常重，速度慢，所以我们可以直接使用gradle编译出apk就好了。

下载好一个项目

1进入到根目录，执行下面代码

```
gradlew -v
```

第一次他会先下载gradle，不翻墙速度很慢。
等下载好会出现以下的界面：
![](http://7xjtan.com1.z0.glb.clouddn.com/2016-04-09_144635.png)

2紧接着执行下面的代码

```
gradlew clean
```
![](http://7xjtan.com1.z0.glb.clouddn.com/2016-04-09_151920.png)

3最后执行下面的代码

```
gradle build
```

这样在app Moudle下的build文件夹中的output文件下面就有了三个apk文件了

# 3 常用的gradle编译命令详解

Android plugin使用相同的约定以兼容其他插件，并且附加了自己的标识性task，包括：

    * assemble：这个task用于组合项目中的所有输出。
    * check：这个task用于执行所有检查。
    * connectedCheck：这个task将会在一个指定的设备或者模拟器上执行检查，它们可以同时在所有连接的设备上执行。
    * deviceCheck：通过APIs连接远程设备来执行检查，这是在CL服务器上使用的。
    * build：这个task执行assemble和check的所有工作。
    * clean：这个task清空项目的所有输出。

这些新的标识性task是必须的，以保证能够在没有设备连接的情况下执行定期检查。
注意build task不依赖于deviceCheck或者connectedCheck。

一个Android项目至少拥有两个输出：debug APK（调试版APK)和release APK（发布版APK）。每一个输出都拥有自己的标识性task以便能够单独构建它们。

    * assemble：
        * assembleDebug
        * assembleRelease
它们都依赖于其它一些tasks以完成构建一个APK需要多个步骤。其中assemble task依赖于这两个task，所以执行assemble将会同时构建出两个APK。

小提示：gradle在命令行终端上支持骆驼命名法的task简称，例如，执行gradle aR命令等同于执行gradle assembleRelease。

check task也拥有自己的依赖：

    * check：
        * lint
    * connectedCheck：
        * connectedAndroidTest
        * connectedUiAutomatorTest(目前还没有应用到）
    * deviceCheck: 这个test依赖于test创建时，其它实现测试扩展点的插件。

最后，只要task能够被安装（那些要求签名的task），android plugin就会为所有构建类型（debug，release，test）安装或者卸载。

# 4 使用gradle多渠道打包

[参考](http://stormzhang.com/devtools/2015/01/15/android-studio-tutorial6/)

[使用Gradle构建Android项目](http://blog.isming.me/2014/05/20/android4gradle/)

> 这里只是简单的了解一下gradle 逛博客的时候看到了邓凡平老师的深入理解Android系列的gradle课程，受益颇多，在学习之后再总结一下写另一篇博客
