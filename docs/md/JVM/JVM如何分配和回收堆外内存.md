作者： 陈汤姆
<br/>博客：[https://github.com/linglongchen/linglongchen.github.io](https://github.com/linglongchen/linglongchen.github.io)

>自我学习的积累！😄


# JVM如何分配和回收堆外内存

作者： 陈汤姆
<br/>博客：[https://github.com/linglongchen/linglongchen.github.io](https://github.com/linglongchen/linglongchen.github.io)

>自我学习的积累！😄

## 1. JVM内存模型

在JVM中内存被分成两大块，分别是堆内存和堆外内存，堆内存就是JVM使用的内存，而堆外内存就是非JVM使用的内存，一般是分配给机器使用的内存。

那么整个内存模型如下：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cb4012ebb59d436a96136c56bc0a0960~tplv-k3u1fbpfcp-zoom-1.image)

因此在JVM中正常只能分配之际独有的内存即堆内存，而我们知道JVM并不建议开发者直接操作堆外内存的，因此容易造成内存泄漏，并且难以排查，但是在JVM中是可以操作堆外内存的并且也可以回收堆外内存，但是是一种不建议的方式。

## 2. 如何分配堆外内存

那么在堆内存中如何分配堆外内存呢？

在Java中存在两种方式分配堆外内存，分别是ByteBuffer#allocateDirect和Unsafe#allocateMemory。

可能第一个会经常使用到，这是Java NIO提供的一个分配内存的类，在做网络开发时会经常使用该方式进行分配内存，而第二种方式是Unsafe的方式，我们知道Unsafe是一种不安全的类，该类是提供给开发者操作最底层数据的类，类似C或者C++直接操作内存的方式，因此该类并不建议使用，如果使用该类分配内存但是没有及时回收容易造成内存泄漏。

#### 第一种方式：ByteBuffer#allocateDirect

该类分配内存的实现方式如下：


```java
public class Test {
    private static Unsafe unsafe = null;

    public static void main(String[] args) {
        //分配10M的内存
        ByteBuffer byteBuffer = ByteBuffer.allocateDirect(10 * 1024 * 1024);
    }
}
```

通过该方式分配堆外内存其实最底层还是使用的是Unsafe#allocateMemory进行分配内存，ByteBuffer只是对Unsafe做了一层封装。

#### 第二种方式：Unsafe#allocateMemory

```java
public class Test {
    private static Unsafe unsafe = null;

    public static void main(String[] args) throws NoSuchFieldException, IllegalAccessException {
        //分配10M的内存
        Field getUnsafe = Unsafe.class.getDeclaredField("theUnsafe");
        getUnsafe.setAccessible(true);
        unsafe = (Unsafe)getUnsafe.get(null);
        //分配完内存返回内存的地址
        long address = unsafe.allocateMemory(10 * 1024 * 1024);
    }
}
```

该方式中Unsafe类并不能直接被使用，但是可以通过反射的方式使用该类，该类分配内存后需要手动回收，不然被分配的内存不会被释放。

## 3. 如何回收堆外内存

说完了如何分配内存，那么继续了解如何回收堆外内存。

#### 第一种方式：Unsafe#freeMemory

分配堆外内存的两种方式中，第二种Unsafe的方式其实提供了一个释放堆外内存的实现，实现如下：

```java
public class Test {
    private static Unsafe unsafe = null;

    public static void main(String[] args) throws NoSuchFieldException, IllegalAccessException {
        //分配10M的内存
        Field getUnsafe = Unsafe.class.getDeclaredField("theUnsafe");
        getUnsafe.setAccessible(true);
        unsafe = (Unsafe)getUnsafe.get(null);
        //分配完内存返回内存的地址
        long address = unsafe.allocateMemory(10 * 1024 * 1024);
        //回收分配的堆外内存
        unsafe.freeMemory(address);
    }
}
```

在Unsafe中提供了freeMemory的实现进行回收堆外内存，但是前提是需要知道被分配的堆外内存地址才可以实现对应的内存回收。

这种回收堆外内存的方式其实是开发者自己手动回收，并不是由JVM引起的内存回收，那么JVM如何回收堆外内存呢？

#### 第二种方式：JVM回收堆外内存

通过ByteBuffer#allocateDirect分配的堆外内存在JVM中其实也是存在一定的内存占用的，具体关联关系如下：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/92a25685d3e34ae08f1956041d37b31c~tplv-k3u1fbpfcp-zoom-1.image)

当通过ByteBuffer#allocateDirect分配堆外内存后，会将堆外内存的地址、大小等信息通过DirectByteBuffer进行关联，那么堆内存中就可以关联到堆外内存。

那么Cleaner又是什么东西呢？

了解Cleaner需要知道JVM中四种引用方式：强引用、弱引用、软引用、虚引用，Cleaner就是虚引用的实现，上图中的ReferenceQueue就是一个引用队列，将需要回收的Cleaner放入到该队列中，实现逻辑如下：

1.  JVM执行Full GC时会将DirectByteBuffer进行回收，回收之后Clearner就不存在引用关系
1.  再下一次发生GC时会将Cleaner对象放入ReferenceQueue中，同时将Cleaner从链表中移除
1.  最后调用unsafe#freeMemory清除堆外内存

那么可能会存在疑问，为什么DirectByteBuffer 会被回收呢？

首先DirectByteBuffer 是存在堆内存中的对象，那么既然存在堆内存中就会发生GC晋级，即晋升到老年代中，在老年代中就会发生Full GC或者Old GC。

## 4. 注意点

#### 注意点1：

在实际使用DirectByteBuffer 时要避免把内存使用完，但是在实际操作中我们可能不知道堆外内存还剩余多少，因此我们可以在JVM中通过参数控制，通过JVM参数 -XX:MaxDirectMemorySize 指定堆外内存的上限大小，当超过指定的内存上限大小时，会主动触发一次Full GC进行回收内存。

#### 注意点2：

通过DirectByteBuffer 分配内存时，可能会出现分配内存不够的情况，因此JVM如果发现堆外内存分配不足时，也会主动发起一次GC，只不过这次GC是通过System.gc() 实现的强制GC，但是在实际生产环境中我们都是通过JVM参数 -XX:+DisableExplicitGC，禁止使用System.gc()的，因此在实际使用过程中一定要注意分配内存的情况，避免出现内存泄漏。



## 5. 引用
-  Netty 核心原理剖析与 RPC 实践
