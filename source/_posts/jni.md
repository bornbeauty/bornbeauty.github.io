---
layout: post
title: jni使用详解
category: 技术
tags: Android
description: 开发中难免要使用jni，现在系统的学一下
date: 2017-07-25
---

> 开发中难免要使用jni，现在系统的学习总结一下使用的方法

##### 1 jni概述

  jni是Java Native Interface的缩写，中文译为“java本地方法接口”。通俗的说，jni是一种技术，通过jni你可以：

- java程序中的函数可以调用Native语言写的函数，Native一般是指C/C++编写的代码。
- Native程序中的函数可以调用Java层的函数，也就是说在C/C++程序中可以调用java的函数。

##### 2 加载jni库以及注册jni函数

  加载jni库非常简单，只需要在调用Native函数之前使用`System.loadLibrary("your_libray_name")`即可。我们的通常做法是在class中的静态块中加载，比如：

```java
package com.jimbo.jni;

public class JNIInterface {

  static {
    System.loadLibrary("your_libray_name");
  }

  //这里可以定义你的Native函数
  public static native final void native_say_hello();

  int a;
  int b;
  public int calc() {
    return a+b;
  }
}
```

知道了`java`代码编写的方法，那么问题来了，Native代码怎么编写呢？java函数怎么找到对应的Native函数呢？all right,让我们来看一下注册jni的两种方法。

- 静态方法

借助java的工具程序`javah`来实现这一过程。答题流程是这样的：

1. 编写java代码，然后编译生成`.class`文件。
2. 使用javah，例如`javah -o output packname.classname`来生成一个叫做`output.h`的头文件。
3. 在jni层实现这些函数。

上面的`JNIInterface`类经过上述操作后会得到这样的头文件：

```c
#include <jni.h>
//...省略
//注：如果java的函数中已经有了“_”，则"_"将会被替换成"_l"
JNIEXPORT void JNICALL Java_com_jimbo_jni_JNIInterface_native_lsay_lhello(JNIEnv *, jclass);

//...省略
```

可以看出，这个Native函数的名字就是包名+函数名，只是因为“.”在c中有特殊的含义，所以被替换成了"_"。这个过程是这样的：

> 当java层调用native_say_hello()函数时，他会从对应的JNI库中寻找Java_com_jimbo_jni_JNIInterface_native_lsay_lhello()函数，如果没有就会报错。如果找的到，则会为这个native_say_hello()和Java_com_jimbo_jni_JNIInterface_native_lsay_lhello()建立一个关联关系，其实就是保存jni层函数的函数指针。以后再调用native_say_hello()函数时，直接使用这个函数指针就可以了。当然这个过程是虚拟机来完成的，不需要我们操作。

- 动态注册

  既然java函数和native函数时一一对应的，那么是不是有一种结构来保存这些数据信息呢？答案是肯定的。在jni技术中，用一个`JNINativeMethod`来保存，结构定义如下：

```c
typedef struct {
  //java函数名 不用携带包名，待会会有其他方式提供包名，
  //这样查找起来效率就会更高
  const char *name;
  //函数签名信息，包括函数的参数以及函数的返回值等信息
  const char *signature;
  //函数指针，类型为void*
  void* fnptr;
} JNINativeMethod;
```

那么，这些对应数据什么时候会被加载出来了呢？其实在调用`System.loadLibrary("your_libray_name");`之后，紧接着会查看该库中一个叫做`JNI_OnLoad()`的函数，如果有就会调用它，动态注册就需要在这里完成。那么究竟如何实现这一个过程呢？需要调用两个函数就可以了：

```c
jclass clazz = (*env) -> FindClass(env, className);

(*env) -> RegisterNatives(env, clazz, gMethods, numMethods);
```

具体注册过程可以这样写：

```c
#include <jni.h>
#include <stdlib.h>  
#include <string.h>  
#include <stdio.h>  
#include <assert.h>

//这个函数是对应java函数的
jstring native_say_hello(JNIEnv *env, jobject thiz) {
  return (*env) -> NewStringUTF(env, "hello, i am from jni~");
}

//这个函数提供方法的对应信息，通过创建JNINativeMethod结构体来实现
//至于那么参数什么意思 待会具体说
static JNINativeMethod gMethods[] = {
  {"native_say_hello", "()Ljava/lang/String", (void)*native_say_hello},
}

//为类的某一个方法注册
static int registerNativeMethod(JNIEnv *env, const char* className, JNINativeMethod *gMethods, int numbers) {
  jclass  clazz = (*env) -> FindClass(env, className);
  if (null == clazz) {
    return JNI_FALSE;
  }
  if ((*env)->RegisterNatives(env, clazz, gMethods, numMethods) < 0) {  
        return JNI_FALSE;  
  }
  return JNI_TURE;
}

//为所有类的方法注册
static int registerNatives(JNIEnv* env) {  
    const char* kClassName = "com/jimbo/jni/JNIInterface";//指定要注册的类  
    return registerNativeMethods(env, kClassName, gMethods,  
            sizeof(gMethods) / sizeof(gMethods[0]));
}

//如果成功返回JNI版本, 失败返回-1
JNIEXPORT jint JNICALL JNI_OnLoad(JavaVM* vm, void* reserved) {  
    JNIEnv* env = NULL;  

    if ((*vm)->GetEnv(vm, (void**) &env, JNI_VERSION_1_4) != JNI_OK) {  
        return -1;  
    }  
    assert(env != NULL);  

    if (!registerNatives(env)) {//注册  
        return -1;  
    }  
    //成功  
    return JNI_VERSION_1_4;  
}  
```

通过上面的方法我们就可以将jni函数和java的函数注册在一起了。但是上面代码似乎还是有点麻烦的，
其实jni的AndroidRunTime类提供了一个`registerNativeMethods()`方法，可以更加简单的实现这
一过程。

```c
#include <jni.h>
#include <stdlib.h>  
#include <string.h>  
#include <stdio.h>  
#include <assert.h>

jstring native_say_hello(JNIEnv *env, jobject thiz) {
  return (*env) -> NewStringUTF(env, "hello, i am from jni~");
}

static JNINativeMethod gMethods[] = {
  {"native_say_hello", "()Ljava/lang/String;", (void)*native_say_hello},
}
//以上代码和前面是一样的

//如果成功返回JNI版本, 失败返回-1
JNIEXPORT jint JNICALL JNI_OnLoad(JavaVM* vm, void* reserved) {  
    JNIEnv* env = NULL;   

    if ((*vm)->GetEnv(vm, (void**) &env, JNI_VERSION_1_4) != JNI_OK) {  
        return -1;  
    }  
    assert(env != NULL);  

    if (AndroidRunTime::registerNativeMethods(env, "com/jimbo/jni/JNIInterface", gMethods, sizeof(gMethods) / sizeof(gMethods[0])) < 0) {//注册  
        return -1;  
    }  
    //成功  
    return JNI_VERSION_1_4;  
}
```

代码比较好理解，但是JNINativeMethod中的signature可能会存在疑问。例如：

```c
"()V"
"()I"
"(II)Ljava/lang/String"
```

实际上这些是与函数参数以及返回值一一对对应的。`()`里面表示参数，`()`外表示的函数的返回值。
比如`(II)Ljava/lang/String`就对应这个函数`jstring functionName(int ,int)`。

具体的对应关系如下:

| 字符 | java类型 | c类型 |
| :-------: | :-------: | :-------: |
| V | void | void |
| Z | boolean | jboolean |
| I | int | jint |
| J | long | jlong |
| D | double | jdouble |
| F | float | jfloat |
| B | byte | jbyte |
| C | char | jchar |
| S | short | jshort |
| [I | int[] | jintarray |
| [F | float[] | jshortarray |
| [B | byte[] | jbytearray |
| [C | char[] | jchararray |
| [S | short[] | jshortarray |
| [D | double[] | jdoublearray |
| [j | long[] | jlongarray |
| [z | boolean[] | jbooleanarray |

以上是关于基本类型和基本类型的数组，那么类是如何表示的呢？

> 如果Java函数的参数是class，则以"L"开头，以";"结尾中间是用"/" 隔开的包及类名。而其对应的C函数名的参数则为jobject. 一个例外是String类，其对应的类为jstring
Ljava/lang/String; String jstring
Ljava/net/Socket; Socket jobject

##### 3 JNIEnv介绍

JNIEnv，即JNIEnvironment，字面意思就是jni环境。其实他就是一个与线程相关的jni环境结构体。
JNIEnv提供了一些jni系统函数，通过这些函数我们可以做：

- 调用java函数
- 操作jobject对象

##### 4 通过JNIEnv操作jobject

我们都知道，类都是由方法和成员变量组成的，在jni的规则中，使用jfirldID和jMethod来表示java的
成员变量和方法，可通过jni下面的两个函数得到：

```c
jfieldId GetFieldID(jclass clazz, const char *name, const char *sig);
jMethod GetMethodID(jclass clazz, const char *name, const char *sig);
```
- jclass代表的java中的类，对应`java.lang.Class`。
- 第二个参数就是类的名称
- 第三个参数是函数签名，和前面介绍的一样

得到jfieldId和jMethod后依然无法调用java函数。那到底该怎么做呢？
不着急，我们看下面的代码：

```c
jint native_calc(JNIEnv *env, jobject thiz) {

  jclass clazz = env -> FindClass("com/jimbo/jni/JNIInterface");
  jmethodID java_calc_id = env -> GetMethodID(clazz, "native_calc", "()I");
  jfieldId a_id = env -> GetFieldID(clazz, "a", "I");
  jfieldId b_id = env -> GetFieldID(clazz, "b", "I");
  jint a = env -> GetIntField(clazz, thiz, a_id);
  jint b = env -> GetIntField(clazz, thiz, b_id);
  return env -> CallIntMethod(env, thiz, java_calc_id, a, b);
}
```

通过这段代码我们知道jni是通过`CallIntMethod()`函数来调用了java的函数。

实际上，jni有一系列类似的函数，形式如下：

```c
//调用函数
//最后参数是调用函数的参数
NativeType Call<Type>Method(JNIEnv *env, jobject thiz, jmethodID methodID, ...);

//获取成员变量的值
NativeType Get<Type>Field(JNIEnv *env, jobject thiz, jfieldId fieldID);
//或者是
void Set<Typr>FieldID(JNIEnv *env, jobject thiz, jfieldId fieldID, NativeType value);
```

常用的还用如下函数:

GetObjectField(),GetIntField(),GetShortField(),GetCharField()等等。

##### 5 jni的垃圾回收以及异常处理

在jni中，有三种类型的引用，包括：

- Local Reference：包括函数调用是传入的参数，在函数内创建的jobject。Local Reference
最大的特点就是，一旦jni函数结束，就会被回收。
- Global Reference：全局引用，这种对象不主动释放永远都不会被回收。
- Weak Reference：弱全局引用，在运行过程中可能被回收。所以在使用前要调用IsSameObject()
来判断他是否已经被回收了。

我们在使用完变量后也可以通过`env -> Delete<ReferenceType>Ref`来主动释放内存。例如DeleteLocalRef();

在jni中，提供了三个函数来截获和处理异常：

1. ExceptionOccured(),用来判断时候发生了异常。
2. ExceptionClear(),用来清理jni层发生的异常。
3. ThrowNew(),用来向java层抛出异常。

本文参考了[邓凡平的深入理解Android 卷1](https://book.douban.com/subject/6802440/)以及[chenfeng0104的专栏-动态注册JNI](http://blog.csdn.net/chenfeng0104/article/details/7088600)。
