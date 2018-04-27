---
title: Android打包那些事儿
date: 2017-03-04 17:39:13
tags: 
    - Android
---

[TOC]

#### 打包流程

##### 前言

我们每一个产品中一般都是由一位同事来负责打包工作的，其他同学一般是不需要关心具体的流程的。然而掌握打包的知识对我们每一个人都是必要的，以备不时之需。

一般打包有两种方式：

- 通过开发工具提供的`build`入口

![Android Studio的编译按钮](http://upload-images.jianshu.io/upload_images/711974-a28778095ec44547.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 使用命令行打包 `gradle tasksName`

不论哪种方式，背后执行的都是一整套Google提供的构建系统。下面我们就来一起看一下这一具体流程。

##### 构建流程

![构建流程简略图](http://upload-images.jianshu.io/upload_images/711974-d18d06104c496365.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

上面这个图是Android官方提供的打包简略流程图。清晰地展示了一个Android Project经过编译和打包后生成apk文件，然后再经过签名，就可以安装到设备上。

具体步骤如下：

1. 编译器将您的源代码转换成DEX（Dalvik Executable) 文件，将资源文件转换成已编译资源。
2. APK打包器将DEX文件和已编译资源合并成单个APK。不过，必须先将APK签名，才能将应用安装并部署到Android设备上。
3. APK打包器使用密钥签署APK：
      a. 如果构建的APK是debug版本，那么将使用调试密钥签名，Android会默认提供一个debug的密钥。
      b. 如果构建的是release版本，会使用发布版本的密钥签名。要生成自己的签名文件发布应用可以参考[Android的官方文档](https://developer.android.com/studio/publish/app-signing.html?hl=zh-cn#studio)。
4. 在生成最终的APK文件之前还会使用`zipalign`工具来优化文件。

生成的APK文件本质还是一个zip文件，只不过被Google强行修改了一下后缀名称而已。所以我们将APK的后缀修改成`.zip`就可以查看其包含的内容了。如图所示：

![APK包含的文件](http://upload-images.jianshu.io/upload_images/711974-7f8219ced286c65b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- AndroidManifest.xml
- assets文件夹：这里面的资源是不经过编译原样打包进来的
- classes.dex：这个是Java的字节码文件，Android会将所有的`class文件`全部放到这一个文件里。
- META-INFO：关于签名的信息存放，应用安装验证签名的时候会验证该文件里面的信息
-res：资源文件，是被编译过的。`raw和图片`是保持原样的，但是其他的文件会被编译成二进制文件。
- resources.arsc：保存资源文件的索引，由aapt生成

##### 详细流程

![详细流程](http://upload-images.jianshu.io/upload_images/711974-8f7242d157d8a4cb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

###### 1. 使用aapt进行资源预编译。

打包的第一步就是将资源进行预编译，资源文件包括res文件夹下面的所有文件。在编译的这一步，使用的是aapt工具进行的，编译后将会生成我们常见的R文件。R文件中是资源对应的ID。每一个ID都是根据特殊规则来生成的，由4个字节组成，用十六进制表示：0x PPTTEEEE。其中PP（Package ID）代表的是资源命名空间，TT（ Type ID）是资源类型标识，资源类型有animator、anim、color、drawable、layout、menu、raw、string和xml等等若干种。EEEE（Entry ID）则代表的是每一个资源所出现的顺序，呈现自增长态。

![R文件](http://upload-images.jianshu.io/upload_images/711974-f1fe92daf99a726d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

###### 2. AIDL文件编译

AIDL 全称Android Interface definition language。在设计进程之间通信的时候需要用定义aidl文件，统一通信接口。Android打包时会将aidl文件转化为一个同名的java接口。

###### 3. Java文件编译

这一步就是标准的Java编译的过程，将.java编译成.class文件。

###### 4. 生成.dex文件

将所有的.class文件（包括第三方库的.class文件）转化成Dalvik文件，即.dex文件。

> 虽然Android使用Java语言编程。但是Android并不使用标准的Java虚拟机执行java code。Android有自己的虚拟机 Dalvik以及5.0以上的ART。 可以大致了解一下Dalvik虚拟机和传统虚拟机的区别：
> - Dalvik虚拟机占用内存更少
> - JVM是基于Stack的(Stack based)，DVM是基于Register的(Register based). 更进一步说，JVM将局部变量存在stack中，而DVM将变量保存在register中。因此标准java虚拟机需要更多的指令集，而DVM的指令集更少，另一方面DVM需要用寄存器编码指令的source 和 destination ，因此Dalvik的一条指令更长。
> - 在JVM运行时，会根据需要去load每个class文件。而Dalvik中所有的class都在一个dex文件中。

![.class文件转化成.dex文件](http://upload-images.jianshu.io/upload_images/711974-45b799ebfc83ca77.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> 这里有一个潜在的危险，由于dex设计上的原因，单个dex内最多能引用的方法数上限为65536，这个数字对于当今很多APP来说都已经不够用了。不过Google也早早就放出了解决方案。这里就不详细讨论了，有兴趣了解可以戳[这里](http://developer.android.com/intl/zh-cn/tools/building/multidex.html)。

###### 5. 打包APK

这一步就是讲所有的资源文件，dex文件一起打包，生成APK文件。

###### 6. 签名

第五步生成的APK文件是不能直接安装到Android手机上面的，必须经过签名后才行。

```
$ keytool -genkey -v -keystore my-release-key.keystore-alias alias_name -keyalg RSA -keysize 2048 -validity 10000
```

生成keytool生成一个`keyStore`，有效期是10000天。

```
$ jarsigner -verbose -sigalg SHA1withRSA -digestalg SHA1-keystore my-release-key.keystore my_application.apk alias_name
```

通过jarsinger来对APK签名

###### 7. 对齐

这是打包的最后一步。对apk包进行对齐处理，工具为zipalign。对齐处理即使得所有资源文件距离文件起始偏移为4字节的整数倍，这样通过内存映射访问apk文件时处理速度更快。

```
zipalign -c -v 4 application.apk
```

以上就是Android打包的完成过程。

#### 多渠道打包

##### 配置gradle实现多渠道打包

每当应用发布一个新的版本的时候，我们会分发到每一个应用市场中去，比如，360手机助手，小米应用市场，华为应用市场等。为了能够统计每个应用市场的下载量，活跃量我们必须用一个标记来区分这些不同市场分发下去的应用，渠道号也就应运而生。随着渠道的不断增加，需要生成的渠道包也就越来越多。
在打包的过程中，我们一般都是使用gradle来进行的。gradle为我们的打包提高了很多的便利，多渠道打包也可以轻松实现。

1. 首先在`AndroidManifest.xml`文件中定义一个`meta-data`

```
<meta-data
    android:name="CHANNEL"
    android:value="${CHANNEL_VALUE}" />
```

2. 然后在gradle文件中设置一下productFlavors

```
android {  
    productFlavors {
        xiaomi {
            manifestPlaceholders = [CHANNEL_VALUE: "xiaomi"]
        }
        _360 {
            manifestPlaceholders = [CHANNEL_VALUE: "_360"]
        }
        baidu {
            manifestPlaceholders = [CHANNEL_VALUE: "baidu"]
        }
        wandoujia {
            manifestPlaceholders = [CHANNEL_VALUE: "wandoujia"]
        }
    }  
}
```

`productFlavors`是为了在同一个项目中创建应用的不同版本。具体的配置信息可以看[官方说明](https://developer.android.com/studio/build/build-variants.html?hl=zh-cn#product-flavors)。

3. 执行`gradle aS`就可以将所有的渠道包输出了。

##### gradle实现多渠道打包的缺点

虽然gradle配置多渠道打包很简单，也很方便，但是这种方式存在一个致命的缺陷，那就是费时间。因为`AndroidManifest.xml`文件被修改过了，所以所有的包都必须重新编译签名。一般来说100个渠道包就要至少一个小时的时间，这一个小时5杯咖啡都不够等的。更要命的是万一哪里需要微调一下代码或者文案，那么不好意思，一切又得重头来。这就很麻烦了，所以有没有什么方法可以快速完成打包呢？我们继续往下看。

#### 多渠道快速打包

##### 快速打包方案Version_1.0

如上所说，我们去到信息只是修改了一下manifest文件里面的一个meta-data的值而已，有没有什么办法可以不需要重新构建代码呢？答案是肯定的。我们可以使用`apktool`，反编译我们的APK文件。

```
apktool d yourApkName build
```

经过解码后，我们会得到如下文件：

![解码后的APK文件](http://upload-images.jianshu.io/upload_images/711974-a72e8977fc819400.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我们发现我们需要修改的manifest文件就在里面，所以通过命令可以修改下他的内容，然后重新打包，就可以生成一个全新的渠道包了，省去了重新编译构建代码的过程。使用一下Python脚本，将manifest文件里面channel信息进行替换。

```
import re

def replace_channel(channel, manifest):
    pattern = r'(<meta-data\s+android:name="channel"\s+android:value=")(\S+)("\s+/>)'
    replacement = r"\g<1>{channel}\g<3>".format(channel=channel)
    return re.sub(pattern, replacement, manifest)
```

然后通过`apktool`重新将文件夹打包生成APK。

```
apktool b build your_unsigned_apk
```

最后，使用jarsigner重新签名apk：

```
jarsigner -sigalg MD5withRSA -digestalg SHA1 -keystore your_keystore_path -storepass your_storepass -signedjar your_signed_apk, your_unsigned_apk, your_alias
```

通过这上面的一系列过程，我们可以实现不重新编译构建项目就生成不同的渠道包。这会节省很多的时间。但是随着渠道包增加，重新签名也会占用很大一部分时间，那能不能不重新签名呢？

分析签名的算法后发现，在打包过程后的META-INF文件夹下面添加空白文件是不会对签名的结果产生影响的。


![APK解压文件](http://upload-images.jianshu.io/upload_images/711974-b94e2d3bf3cbf7aa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

所以我们只要像META_INF文件夹里面写入空白的文件来标识渠道号就可以了。

通过Python脚本像APK文件中写入渠道：

```
import zipfile
zipped = zipfile.ZipFile(your_apk, 'a', zipfile.ZIP_DEFLATED) 
empty_channel_file = "META-INF/mtchannel_{channel}".format(channel=your_channel)
zipped.write(your_empty_file, empty_channel_file)
```

执行后会在META-INF文件夹下面生成一个空白文件：

![生成空白文件](http://upload-images.jianshu.io/upload_images/711974-13e587003f6b2f01.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

然后我们在项目中去读取这个空白文件：

```
public static String getChannel(Context context) {
        ApplicationInfo appinfo = context.getApplicationInfo();
        String sourceDir = appinfo.sourceDir;
        String ret = "";
        ZipFile zipfile = null;
        try {
            zipfile = new ZipFile(sourceDir);
            Enumeration<?> entries = zipfile.entries();
            while (entries.hasMoreElements()) {
                ZipEntry entry = ((ZipEntry) entries.nextElement());
                String entryName = entry.getName();
                if (entryName.startsWith("mtchannel")) {
                    ret = entryName;
                    break;
                }
            }
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if (zipfile != null) {
                try {
                    zipfile.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }

        String[] split = ret.split("_");
        if (split != null && split.length >= 2) {
            return ret.substring(split[0].length() + 1);

        } else {
            return "";
        }
    }
```

这样每生成一个渠道包只需要复制一下APK，然后添加一个空态的文件到META-INF下面就可以了，这样100个渠道包一分钟之内就可以搞定了。

具体的原因可以看下[这里](http://blog.csdn.net/jiangwei0910410003/article/details/50402000)。

##### 快速打包方案Version_2.0

上面的方案基本上已经比较完美的解决我们打包的问题了，然而好景不长，Google在Android 7.0中更新了应用的签名算法-[APK Signature Scheme v2](https://source.android.com/security/apksigning/v2)，它是一个对全文件进行签名的方案，能提供更快的应用安装时间、对未授权APK文件的更改提供更多保护，在默认情况下，Android Gradle 2.2.0插件会使用APK Signature Scheme v2和传统签名方案来签署你的应用。因为是对全文件进行签名的，所以之前的添加空白文件的方案就没有作用了。

不过目前这个方案还不是强制性的，我们可以选择在gradle配置文件中将其关闭：

```
android {
    defaultConfig { ... }
    signingConfigs {
      release {
        storeFile file("myreleasekey.keystore")
        storePassword "password"
        keyAlias "MyReleaseKey"
        keyPassword "password"
        v2SigningEnabled false
      }
    }
  }
```

那么新的签名方案对已有的渠道生成方案有什么影响呢？下图是新的应用签名方案和旧的签名方案的一个对比：


![v1和v2版本签名对比](http://upload-images.jianshu.io/upload_images/711974-cb293afcbda08054.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

新的签名方案会在ZIP文件格式的 Central Directory 区块所在文件位置的前面添加一个APK Signing Block区块，下面按照ZIP文件的格式来分析新应用签名方案签名后的APK包。
整个APK（ZIP文件格式）会被分为以下四个区块：

1. Contents of ZIP entries（from offset 0 until the start of APK Signing Block）
2. APK Signing Block
3. ZIP Central Directory
4. ZIP End of Central Directory

新应用签名方案的签名信息会被保存在区块2（APK Signing Block）中， 而区块1（Contents of ZIP entries）、区块3（ZIP Central Directory）、区块4（ZIP End of Central Directory）是受保护的，在签名后任何对区块1、3、4的修改都逃不过新的应用签名方案的检查。

之前的渠道包生成方案是通过在META-INF目录下添加空文件，用空文件的名称来作为渠道的唯一标识，之前在META-INF下添加文件是不需要重新签名应用的，这样会节省不少打包的时间，从而提高打渠道包的速度。但在新的应用签名方案下META-INF已经被列入了保护区了，向META-INF添加空文件的方案会对区块1、3、4都会有影响，新应用签名方案签署的应用经过我们旧的生成渠道包方案处理后，在安装时会报以下错误：

```
Failure [INSTALL_PARSE_FAILED_NO_CERTIFICATES: 
Failed to collect certificates from base.apk: META-INF/CERT.SF indicates base.apk is signed using APK Signature Scheme v2, 
but no such signature was found. Signature stripped?]
```

区块1，3，4是受保护的，任何的修改都会引起签名的不一致，但是区块2是不受保护的，所以能不能在区块2上面找到解决办法呢？首先看一下区块2的文件结构：


![区块2文件结构](http://upload-images.jianshu.io/upload_images/711974-cb09a72d6a9309da.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

区块2中APK Signing Block是由这几部分组成：2个用来标示这个区块长度的8字节 ＋ 这个区块的魔数（APK Sig Block 42）+ 这个区块所承载的数据（ID-value）。

我们重点来看一下这个ID-value，它由一个8字节的长度标示＋4字节的ID＋它的负载组成。V2的签名信息是以ID（0x7109871a）的ID-value来保存在这个区块中，不知大家有没有注意这是一组ID-value，也就是说它是可以有若干个这样的ID-value来组成，那我们是不是可以在这里做一些文章呢？

对于签名的认证过程是这样的：

1. 寻找APK Signing Block，如果能够找到，则进行验证，验证成功则继续进行安装，如果失败了则终止安装
2. 如果未找到APK Signing Block，则执行原来的签名验证机制，也是验证成功则继续进行安装，如果失败了则终止安装

![签名认证过程](http://upload-images.jianshu.io/upload_images/711974-8399151824ebe185.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在校验的时候，检验代码如下：

```
 public static ByteBuffer findApkSignatureSchemeV2Block(
            ByteBuffer apkSigningBlock,
            Result result) throws SignatureNotFoundException {
        checkByteOrderLittleEndian(apkSigningBlock);
        // FORMAT:
        // OFFSET       DATA TYPE  DESCRIPTION
        // * @+0  bytes uint64:    size in bytes (excluding this field)
        // * @+8  bytes pairs
        // * @-24 bytes uint64:    size in bytes (same as the one above)
        // * @-16 bytes uint128:   magic
        ByteBuffer pairs = sliceFromTo(apkSigningBlock, 8, apkSigningBlock.capacity() - 24);

        int entryCount = 0;
        while (pairs.hasRemaining()) {
            entryCount++;
            if (pairs.remaining() < 8) {
                throw new SignatureNotFoundException(
                        "Insufficient data to read size of APK Signing Block entry #" + entryCount);
            }
            long lenLong = pairs.getLong();
            if ((lenLong < 4) || (lenLong > Integer.MAX_VALUE)) {
                throw new SignatureNotFoundException(
                        "APK Signing Block entry #" + entryCount
                                + " size out of range: " + lenLong);
            }
            int len = (int) lenLong;
            int nextEntryPos = pairs.position() + len;
            if (len > pairs.remaining()) {
                throw new SignatureNotFoundException(
                        "APK Signing Block entry #" + entryCount + " size out of range: " + len
                                + ", available: " + pairs.remaining());
            }
            int id = pairs.getInt();
            if (id == APK_SIGNATURE_SCHEME_V2_BLOCK_ID) {
                return getByteBuffer(pairs, len - 4);
            }
            result.addWarning(Issue.APK_SIG_BLOCK_UNKNOWN_ENTRY_ID, id);
            pairs.position(nextEntryPos);
        }

        throw new SignatureNotFoundException(
                "No APK Signature Scheme v2 block in APK Signing Block");
    }
```

我们可以发现，述代码中关键的一个位置是 if (id == APK_SIGNATURE_SCHEME_V2_BLOCK_ID) {return getByteBuffer(pairs, len - 4);}，通过源代码可以看出Android是通过查找ID为 APK_SIGNATURE_SCHEME_V2_BLOCK_ID = 0x7109871a 的ID-value，来获取APK Signature Scheme v2 Block，对这个区块中其他的ID-value选择了忽略。也就是说，在[APK Signature Scheme v2](https://source.android.com/security/apksigning/v2.html)中没有看到对无法识别的ID，有相关处理的介绍。所以我们可以通过写入自定义的ID-Value来自定义渠道。

所以整理一下思路应该是这样的：

1. 对新的应用签名方案生成的APK包中的ID-value进行扩展，提供自定义ID－value（渠道信息），并保存在APK中
2. 在App运行阶段，可以通过ZIP的EOCD（End of central directory）、Central directory等结构中的信息找到我们自己添加的ID-value，从而实现获取渠道信息的功能

通过这个方案我们同样可以实现不重新构建不重新签名就能生成渠道包。美团提供了walle供我们使用，详细使用方法[点这里](https://github.com/Meituan-Dianping/walle)。