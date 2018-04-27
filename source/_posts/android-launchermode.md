---

layout: post
title: 定义Activity启动模式的两种方式
category: 技术
tags: android
description: 关于Activity启动模式定义的两种方法
date: 2016-10-09
---

> 关于activity启动模式定义的两种方式

关于activity的生命周期和启动模式可以查看博客: [activity生命周期和启动模式深入理解](http://bornbeauty.github.io/2015/11/11/androidLifecycleAndlauncherMode.html)

## 一 定义启动模式的方式

有两种方式：

1、通过manifest文件定义
2、在Intent中通过设置flag的方式

如果一个activity启动模式通过这两个方式都有定义，以Intent设置flag的方式为准，也就是说方式2的优先级高于方式1.

## 二 使用manifest文件

通过设置activity的launchMode来设置启动模式：

1. standard-标准启动模式

这是默认的启动模式。无论当前任务栈中是否已经存在改activity的实例，都将会重新new一个activity，并将改activity放置到当前任务栈的栈顶。

2. singleTop-栈顶复用模式

如果任务栈中已经存在改activity的实例，并且该实例位于栈顶，那么该activity将被复用，不会重新启动一个activity，而是调用activity的onNewIntent().

3. singleTask-栈内复用模式

一个任务栈中只会存在一个改启动模式的activity。当activity已经存在的时候，会直接复用，调用它的onNewIntent()方法。同时，singleTask具有clearTop的效果，位于它上方的activity都会被出栈。

4. singleInstance-单例模式

一个栈内只存在一个改启动模式的activity。


## 三 使用Intent的flag

FLAG_ACTIVITY_NEW_TASK

设置了这个flag，新启动Activity就会被放置到一个新的任务当中(与"singleTask"有点类似，但不完全一样)，当然这里讨论的仍然还是启动其它程序中的Activity。这个flag的作用通常是模拟一种Launcher的行为，即列出一推可以启动的东西，但启动的每一个Activity都是在运行在自己独立的任务当中的。

FLAG_ACTIVITY_SINGLE_TOP

设置了这个flag，如果要启动的Activity在当前任务中已经存在了，并且还处于栈顶的位置，那么就不会再次创建这个Activity的实例，而是直接调用它的onNewIntent()方法。这种flag和在launchMode中指定"singleTop"模式所实现的效果是一样的。

FLAG_ACTIVITY_CLEAR_TOP

设置了这个flag，如果要启动的Activity在当前任务中已经存在了，就不会再次创建这个Activity的实例，而是会把这个Activity之上的所有Activity全部关闭掉。比如说，一个任务当中有A、B、C、D四个Activity，然后D调用了startActivity()方法来启动B，并将flag指定成FLAG_ACTIVITY_CLEAR_TOP，那么此时C和D就会被关闭掉，现在返回栈中就只剩下A和B了。
那么此时Activity B会接收到这个启动它的Intent，你可以决定是让Activity B调用onNewIntent()方法(不会创建新的实例)，还是将ActivityB销毁掉并重新创建实例。如果ActivityB没有在manifest中指定任何启动模式(也就是"standard"模式)，并且Intent中也没有加入一个FLAG_ACTIVITY_SINGLE_TOP flag，那么此时ActivityB就会销毁掉，然后重新创建实例。而如果Activity B在manifest中指定了任何一种启动模式，或者是在Intent中加入了一个FLAG_ACTIVITY_SINGLE_TOPflag，那么就会调用ActivityB的onNewIntent()方法。
FLAG_ACTIVITY_CLEAR_TOP和FLAG_ACTIVITY_NEW_TASK结合在一起使用也会有比较好的效果，比如可以将一个后台运行的任务切换到前台，并把目标Activity之上的其它Activity全部关闭掉。这个功能在某些情况下非常有用，比如说从通知栏启动Activity的时候。

处理affinity

affinity可以用于指定一个Activity更加愿意依附于哪一个任务，在默认情况下，同一个应用程序中的所有Activity都具有相同的affinity，所以，这些Activity都更加倾向于运行在相同的任务当中。当然了，你也可以去改变每个Activity的affinity值，通过<activity>元素的taskAffinity属性就可以实现了。
taskAffinity属性接收一个字符串参数，你可以指定成任意的值(经我测试字符串中至少要包含一个.)，但必须不能和应用程序的包名相同，因为系统会使用包名来作为默认的affinity值。

affinity主要有以下两种应用场景：
当调用startActivity()方法来启动一个Activity时，默认是将它放入到当前的任务当中。但是，如果在Intent中加入了一个FLAG_ACTIVITY_NEW_TASK flag的话(或者该Activity在manifest文件中声明的启动模式是"singleTask")，系统就会尝试为这个Activity单独创建一个任务。但是规则并不是只有这么简单，系统会去检测要启动的这个Activity的affinity和当前任务的affinity是否相同，如果相同的话就会把它放入到现有任务当中，如果不同则会去创建一个新的任务。而同一个程序中所有Activity的affinity默认都是相同的，这也是前面为什么说，同一个应用程序中即使声明成"singleTask"，也不会为这个Activity再去创建一个新的任务了。
当把Activity的allowTaskReparenting属性设置成true时，Activity就拥有了一个转移所在任务的能力。具体点来说，就是一个Activity现在是处于某个任务当中的，但是它与另外一个任务具有相同的affinity值，那么当另外这个任务切换到前台的时候，该Activity就可以转移到现在的这个任务当中。
那还是举一个形象点的例子吧，比如有一个天气预报程序，它有一个Activity是专门用于显示天气信息的，这个Activity和该天气预报程序的所有其它Activity具体相同的affinity值，并且还将allowTaskReparenting属性设置成true了。这个时候，你自己的应用程序通过Intent去启动了这个用于显示天气信息的Activity，那么此时这个Activity应该是和你的应用程序是在同一个任务当中的。但是当把天气预报程序切换到前台的时候，这个Activity又会被转移到天气预报程序的任务当中，并显示出来，因为它们拥有相同的affinity值，并且将allowTaskReparenting属性设置成了true。

清空返回栈

如何用户将任务切换到后台之后过了很长一段时间，系统会将这个任务中除了最底层的那个Activity之外的其它所有Activity全部清除掉。当用户重新回到这个任务的时候，最底层的那个Activity将得到恢复。这个是系统默认的行为，因为既然过了这么长的一段时间，用户很有可能早就忘记了当时正在做什么，那么重新回到这个任务的时候，基本上应该是要去做点新的事情了。
当然，既然说是默认的行为，那就说明我们肯定是有办法来改变的，在<activity>元素中设置以下几种属性就可以改变系统这一默认行为：
alwaysRetainTaskState
如果将最底层的那个Activity的这个属性设置为true，那么上面所描述的默认行为就将不会发生，任务中所有的Activity即使过了很长一段时间之后仍然会被继续保留。

clearTaskOnLaunch

如果将最底层的那个Activity的这个属性设置为true，那么只要用户离开了当前任务，再次返回的时候就会将最底层Activity之上的所有其它Activity全部清除掉。简单来讲，就是一种和alwaysRetainTaskState完全相反的工作模式，它保证每次返回任务的时候都会是一种初始化状态，即使用户仅仅离开了很短的一段时间。

finishOnTaskLaunch

这个属性和clearTaskOnLaunch是比较类似的，不过它不是作用于整个任务上的，而是作用于单个Activity上。如果某个Activity将这个属性设置成true，那么用户一旦离开了当前任务，再次返回时这个Activity就会被清除掉。