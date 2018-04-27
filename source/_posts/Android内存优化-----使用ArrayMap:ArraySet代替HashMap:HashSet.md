---
title: Android内存优化-----使用ArrayMap:ArraySet代替HashMap:HashSet
date: 2018-01-04 21:12:12
tags: 
    - Android
---

[TOC]

### 1. 为什么要用ArrayMap/ArraySet

在Android开发中，经常会用到`HashMap/HashSet`等集合类，但是Java在设计集合类的时候并没有考虑到内存宝贵场景下优化。而对Android系统来说内存是非常宝贵的资源，所以Google针对Android系统的特性提供了 `HashMap/HashSet`的代替品，即`ArrayMap/ArraySet`。这几个类位于`android.util.*`包下面。

### 2. 简单回忆下HashMap<K, V>的实现

#### 2.1 数据接口

在Java中，HashMap是通过数组加引用组合实现的数据结构，结构图如下：

![注：图片来自网络](http://upload-images.jianshu.io/upload_images/711974-88f9cd66dfea5eb8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240))

从图中我们看到，数组的每个元素之后会跟着一条链表。我们应该很容易想到这是为了解决HashCode冲突而设计的，使用链地址法解决冲突。

> 顺便提一下解决冲突的常用四个方法：
> 1. 链地址法
> 2. 再Hash法
> 3. 开放地址发
> 4. 建立公共溢出区

#### 2.2 HashMap的数据结构

HashMap有三个构造方法：

```
HaspMap()
HaspMap(int initialCapacity)
HashMap(int initialCapacity, float loadFactor)
```

- 其中 initialCapacity是一个初始化容量大小，指的是table数组的初始大小。
- loadFactor是负载因子，当table数组的实际容量超过（initialCapacity*loadFactor）的时候，table数组就会扩容。
- initialCapacity默认值是15 而loadFactor的默认值是0.75 他俩都是影响HashMap性能的重要因素。

首先看下构造函数的源码：

```
public HashMap(int initialCapacity, float loadFactor) {
        // 首先检查各个参数的合法性
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal initial capacity: "
                    + initialCapacity);
        if (initialCapacity > MAXIMUM_CAPACITY)
            initialCapacity = MAXIMUM_CAPACITY;
        if (loadFactor <= 0 || Float.isNaN(loadFactor))
            throw new IllegalArgumentException("Illegal load factor: "
                    + loadFactor);

        // 计算出大于 initialCapacity 的最小的 2 的 n 次方值。
        // HashMap的实际数组大小都是2^n，是为了优化而设计
        int capacity = 1;
        while (capacity < initialCapacity)
            capacity <<= 1;
       
        this.loadFactor = loadFactor;
        // 计算极限容量 超过后将会进行扩容
        threshold = (int) (capacity * loadFactor);
        //初始化table数组
        table = new Entry[capacity];
        init();
    }
```

其中有一个`Entry`是`HashMap`的内部类：

```
Entry() {
     K key;                           
     V value;
     Entry<K, V> next;
     int hash;
}
```

它是实际储存Key和Value的实体。next是指向下一个元素的引用，hash是key的hash值。

#### 2.3 数据的存放

再来看下`put`方法的源码：

```
public V put(K key, V value) {
        // 如果key的值是null，直接将其放在null位置上，也就是第一个位置上
        if (key == null)
            return putForNullKey(value);
        //计算key的hash值
        int hash = hash(key.hashCode());
        //计算key hash 值在 table 数组中的位置
        int i = indexFor(hash, table.length);
        //从i出开始迭代 e,找到 key 保存的位置
        for (Entry<K, V> e = table[i]; e != null; e = e.next) {
            Object k;
            //判断该条链上是否有hash值相同的(key相同)
            //若存在相同，则直接覆盖value，返回旧value
            if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
                V oldValue = e.value;    //旧值 = 新值
                e.value = value;
                e.recordAccess(this);
                return oldValue;     //返回旧值
            }
        }
        //修改次数增加1
        modCount++;
        //将key、value添加至i位置处
        addEntry(hash, key, value, i);
        return null;
    }
```

- 首先判断key值是否为null，如果是null，则将hash值当成0处理 否则
- 根据key对象的`hashCode()`方法计算出hash值，然后再将hash值转化成在table中的索引 然后
- 查找该索引处的链表中是否存在key相同的对象，如果有则将其覆盖，并且返回旧值 否则
- 将key、value存放于该数组索引处链表的第一个位置上，其中这一步是在`addEntry()`方法中完成的

```
void addEntry(int hash, K key, V value, int bucketIndex) {
        //获取bucketIndex处的Entry
        Entry<K, V> e = table[bucketIndex];
        //将新创建的 Entry 放入 bucketIndex 索引处，并让新的 Entry 指向原来的 Entry 
        table[bucketIndex] = new Entry<K, V>(hash, key, value, e);
        //若HashMap中元素的个数超过极限了，则容量扩大两倍
        if (size++ >= threshold)
            resize(2 * table.length);
    }
```

#### 2.4 数据的取出

对于HashMap，取出值是相对简单的：

```
public V get(Object key) {
        // 若为null，调用getForNullKey方法返回相对应的value
        if (key == null)
            return getForNullKey();
        // 根据该 key 的 hashCode 值计算它的 hash 码  
        int hash = hash(key.hashCode());
        // 取出 table 数组中指定索引处的值
        for (Entry<K, V> e = table[indexFor(hash, table.length)]; e != null; e = e.next) {
            Object k;
            //若搜索的key与查找的key相同，则返回相对应的value
            if (e.hash == hash && ((k = e.key) == key || key.equals(k)))
                return e.value;
        }
        return null;
    }
```

#### 2.5 HashMap的小结

从上面的分析中可以看出，HashMap的存取速度都是相对比较快的，在一般情况下都能实现O(1)的速度，但是从初始容量和负载因子都可以看出，这种快速的读取都是通过内存来换取，而对于移动设备来讲，内存又是很重要的，所以，Google为我们提供了Arraymap来代替它。

### 3. ArrayMap的简单分析

在ArrayMap的内部，使用一个hash数组加一个kay，value数组来储存。

![Google官方图](http://upload-images.jianshu.io/upload_images/711974-66d02bf2c8b5c024.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> 当你想获取某个value的时候，ArrayMap会计算输入key转换过后的hash值，然后对hash数组使用二分查找法寻找到对应的index，然后我们可以通过这个index在另外一个数组中直接访问到需要的键值对。如果在第二个数组键值对中的key和前面输入的查询key不一致，那么就认为是发生了碰撞冲突。为了解决这个问题，我们会以该key为中心点，分别上下展开，逐个去对比查找，直到找到匹配的值（开放地址法）。如下图所示：

![Google官网图](http://upload-images.jianshu.io/upload_images/711974-9ade6b011f0367b4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

可以看出，ArrayMap将内存的使用率提高了很多，但是读取的复杂度却是O(lgN)（因为二分法查找）。所以在数据量不是很大（千级以内）的时候，我们使用ArrapMap可以优化内存，并且存取速度几乎是不受什么影响的。 

其实这里还有一点，就是自动装箱问题，假设我们把一个key是int类型的数据存储时，HashMap会将int值自动装箱成Integer对象，一个对象和一个基本类型所占的内存大小是差别很大的，所以为了避免这样情况，Google提供了`SparseIntArray `,`SparseBoolArray`等一系列大礼包供我们使用。

### 4. 总结

总结一句话，在数据量千级以内，使用`ArrayMap ArraySet`或者`SparseIntMap等`来代替`HashMap`，以此节约宝贵的内存资源。

















