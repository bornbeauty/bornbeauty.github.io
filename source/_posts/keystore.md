---
layout: post
title: 关于keystore的作用以及生成
category: 技术
tags: Android
description: 关于Android的作用以及怎么产生一个自己的keystore
date: 2017-06-03
---

> 对于混乱的Android应用市场 使用统一的keystore签名自己的应用，保护自己的产权是十分重要的，所以我们今天就看一下有关keystore的内容

##### 1. keystore的作用

为了保证每个应用程序开发商合法ID，防止部分开放商可能通过使用相同的Package Name来混淆替换已经安装的程序，我们需要对我们发布的APK文件进行唯一签名，保证我们每次发布的版本的一致性(如自动更新不会因为版本不一致而无法安装)。

#### 2. 对apk签名

1. 生成一个属于你的keystore文件
2. 使用apktool和生成的keystore文件为apk文件签名

#### 3. 生成keystore文件

使用jdk安装路径/jre/bin/keytool.exe来生成签名文件
使用jdk安装路径/bin/jarsinger.exe来签名

```
D:\>keytool -genkey -alias demo.keystore -keyalg RSA -validity 40000 -keystore demo.keystore
-genkey: 产生密钥
-alias: 签名文件的文件名
-keyalg: RSA 使用RSA算法对签名加密
-validity: 有效期

D:\>jarsigner -verbose -keystore demo.keystore -signedjar demo_signed.apk demo.apk demo.keystore
-verbose 输出签名的详细信息
-keystore  demo.keystore 密钥库位置
-signedjar demor_signed.apk demo.apk demo.keystore 正式签名，三个参数中依次为签名后产生的文件demo_signed，要签名的文件demo.apk和密钥库demo.keystore.
```

#### 4. 签名之后，用zipalign(压缩对齐)优化你的APK文件。

未签名的apk不能使用，也不能优化。签名之后的apk谷歌推荐使用zipalign.exe(位于android-sdk-windows\tools目录下)工具对其优化

#### 5. 签名对你的App的影响

  你不可能只做一个APP，你可能有一个宏伟的战略工程，想要在生活，服务，游戏，系统各个领域都想插足的话，你不可能只做一个APP，谷歌建议你把你所有的APP都使用同一个签名证书。

  使用你自己的同一个签名证书，就没有人能够覆盖你的应用程序，即使包名相同，所以影响有：

1. App升级。 使用相同签名的升级软件可以正常覆盖老版本的软件，否则系统比较发现新版本的签名证书和老版本的签名证书不一致，不会允许新版本安装成功的。
2. App模块化。android系统允许具有相同的App运行在同一个进程中，如果运行在同一个进程中，则他们相当于同一个App，但是你可以单独对他们升级更新，这是一种App级别的模块化思路。
3. 允许代码和数据共享。android中提供了一个基于签名的Permission标签。通过允许的设置，我们可以实现对不同App之间的访问和共享，如下：

`AndroidManifest.xml：<permission android:protectionLevel="normal" />`

其中protectionLevel标签有4种值：normal(缺省值),dangerous,signature,signatureOrSystem。简单来说，normal是低风险的，所有的App不能访问和共享此App。dangerous是高风险的，所有的App都能访问和共享此App。signature是指具有相同签名的App可以访问和共享此App。signatureOrSystem是指系统image中App和具有相同签名的App可以访问和共享此App，谷歌建议不要使用这个选项，因为签名就足够了，一般这个许可会被用在在一个image中需要共享一些特定的功能的情况下。

最后，请一定要记得保管好你的签名证书的两个密码，两个密码都不要告诉任何人，也不要把你的密钥库拷贝给别人，包括我！
