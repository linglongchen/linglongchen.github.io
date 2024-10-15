## 解释JIT编译技术



## 1. 概念

Java代码执行过程：
	1. 编译：将.java源代码编程成字节码.class，在这个过程实现词法分析、语法分析、语义分析等
	2. 加载：通过ClassLoader将字节码加载到JVM中
	3. 验证：对加载进JVM的字节码做验证，魔数等
	4. 准备：对静态变量分配内存以及设置一些默认值
	5. 解析：将类、接口、字段等符号引用转为直接引用
	6. 初始化：对一些代码块或者方法进行初始化
	7. 执行：JVM使用解释器或即时编译器（JIT）执行字节码。解释器逐行解释字节码，而JIT会将热点代码编译为机器码（计算机能够直接理解和执行的二进制指令集合），提高运行效率

![2-Page-24.jpg](https://s2.loli.net/2024/10/15/u8BHmvZpXfS2yG5.jpg)
JIT（Just In Time）编译器：虚拟机发现某个方法或者代码块运行特别频繁时，会把这类代码认定为“热点代码”，为了提高热点代码的执行效率，在运行期间，虚拟机会把代码编译诚本地平台相关的机器码



## 2. 目标

- 提高代码执行效率

## 3. 分类
C1编译器（Client Compiler）：该编译器启动速度快，但是性能比较Server Compiler来说会差一些，主要的功能：
	- 局部简单可靠的优化，比如字节码上进行的一些基础优化，方法内联、常量传播等，放弃许多耗时较长的全局优化
	- 将字节码构造成高级中间表示（High-level Intermediate Representation，以下称为HIR），HIR与平台无关，通常采用图结构，更适合JVM对程序进行优化
	- - 最后将HIR转换成低级中间表示（Low-level Intermediate Representation，以下称为LIR），在LIR的基础上会进行寄存器分配、窥孔优化（局部的优化方式，编译器在一个基本块或者多个基本块中，针对已经生成的代码，结合CPU自己指令的特点，通过一些认为可能带来性能提升的转换规则或者通过整体的分析，进行指令转换，来提升代码性能）等操作，最终生成机器码。
C2编译器（Server Complier）：主要关注一些编译耗时较长的全局优化，编译器的启动时间长，适用于长时间运行的后台程序，它的性能通常比Client Compiler高30%以上。Hotspot虚拟机中使用的Server Compiler有两种：C2和Graal

## 4. 触发条件
- 被多次调用的方法：C1编译器中被调用超过1500次则会被优化，在C2编译器中被调用超过10000次则会被优化
- 被多次执行的循环体：同上

## 5. JIT编译优化技术


### 5.1.方法内联
简单解释：方法A调用方法B，并不会直接调用方法B而是将方法B的逻辑复制到A中执行。
```java
public void methodA() { 
	int i = 1; i = methodB(i); 
} 
public int methodB(int param) { 
	return param + 3; 
}
```
在该方法栈中执行逻辑，执行完成methodB之后再回到methodA的方法栈，整个过程需要记录帧栈的创建以及线程的切换，JVM为了避免这种多层次的调用通过方法内联进行优化。

#### 方法内联条件
- 热点方法：在Java中一些方法会被频繁的调用，当达到一定的调用次数JVM会认为该方法为热点方法。可以通过 - XX:CompileThreshold 参数来调整热点方法阈值。该参数控制的是JVM中即时编译器JIT中编译方法的次数，JIT中存在两种编译器客户端C1和服务端C2，分别对应的热点阈值为1500和10000.（但是不是绝对达到阈值就会进行方法内联，前提是方法体不能过大）。
- 小方法体：方法内联的前提就是方法体不宜过大，过大的方法体复制到另外方法中也是耗时耗性能的，因此小的方法体更容易执行内联。在JVM中对于热点方法体默认方法体大小小于325字节时会进行方法内联，可以通过 -XX:MaxFreqInlineSize=N 设置该值，非热点代码默认方法体大小小于35字节会进行内联，可以通过 -XX:MaxInlineSize=N 。
- 特殊修饰符方法体：private、static、final修饰的方法体更容易执行内联，主要考虑是public和protect修饰的方法会需要进行集成或者重写的校验。

### 5.2.逃逸分析
要了解逃逸分析首先知道什么是方法逃逸。

方法逃逸：一个对象在方法中被定义之后，如果被外部方法所引用（作为参数传递到其它方法中赋值），那么这种实现就是方法逃逸。
```java
class Test {
	private void methodA() {
		User user = new User();
		this.setUserParams(user);
	}

	//将User作为参数传递到该方法中
	private void setUserParams(User user) {
		//方法中对user对象设置参数
		user.setUserName("方法逃逸");
		user.setAge(12);
	}
}
```



因此基于方法逃逸就可以知道逃逸分析其实就是即时编译器对方法内的对象做分析，判断方法中定义的对象是否会从当前方法或者线程中逃逸。

#### 逃逸分析的原因
那么知道逃逸分析的概念，那么即时编译器为什么要做逃逸分析？

本质还是对象的优化，我们知道对象的创建都是在堆上创建的，对象通过栈上的引用获取在堆上创建的对象，对象的创建和销毁都需要通过JVM的垃圾收集器来实现，但是如果对象直接在栈上分配是否能够去掉垃圾收集的步骤呢？答案是肯定的。


#### 逃逸分析的作用
##### 栈上分配对象

因此逃逸分析就是对方法中定义的对象做分析，如果不会逃逸出当前方法，那么可以对象的创建就可以直接在栈上实现，而不需要在堆中。不过Hotspot虚拟机，并没有进行实际的栈上分配，而是使用了标量替换这一技术。所谓的标量，就是仅能存储一个值的变量，比如Java代码中的基本类型。聚合对象则可能同时存储多个值，其中一个典型的例子便是Java的对象。编译器会在方法内将未逃逸的聚合量分解成多个标量，以此来减少堆上分配


##### 锁消除

锁消除是synchronized关键字的一个特性，是并发编程中的优化项。

锁消除发生在即时编译器（JIT）阶段，当JVM检测到某些代码块或方法尽管被同步代码包围，但实际上并不涉及共享数据的竞争，或者锁对象根本没有逃逸出该线程的上下文时，JIT编译器就会执行锁消除。



### 5.3.公共子表达式消除

如果一个表达式E已经计算过了，并且从先前的计算到现在表达式E中所有的变量都没有发生变化，那么这个表达式E就会成为公共子表达式。

```java

int d = (c*b) * 12 + a + (a+b*c);
```

以上的一个表达式中，"c*b"和"b*c"是一样的表达式，并且期间没有发生值的变化，那么编译器就会优化公共子表达式"b*c"

```java
int d = E * 12 + a + (a+E);
```

通过这种表达式的转换，优化本地代码的执行效率

### 5.4.循环展开
循环展开是一种循环转换技术，它试图以牺牲程序二进制码大小为代价来优化程序的执行速度，是一种用空间换时间的优化手段。

循环展开通过减少或消除控制程序循环的指令，来减少计算开销，这种开销包括增加指向数组中下一个索引或者指令的指针算数等。如果编译器可以提前计算这些索引，并且构建到机器代码指令中，那么程序运行时就可以不必进行这种计算。也就是说有些循环可以写成一些重复独立的代码。比如下面这个循环：

```Java
public void loopRolling(){
  for(int i = 0;i<200;i++){
    delete(i);  
  }
}
```
循环展开后
```Java
public void loopRolling(){
  for(int i = 0;i<200;i+=5){
    delete(i);
    delete(i+1);
    delete(i+2);
    delete(i+3);
    delete(i+4);
  }
}
```


这样展开就可以减少循环的次数，每次循环内的计算也可以利用CPU的流水线提升效率

## 6. 实践

```Java
public class JitExample {  
    public static void main(String[] args) throws InterruptedException {  
        long start = System.currentTimeMillis();  
          
        // 热身阶段，让JIT编译器识别热点方法  
        for (int i = 0; i < 10000000; i++) {  
            compute(i);  
        }  
        // 计算实际执行时间  
        long warmupTime = System.currentTimeMillis() - start;  
        System.out.println("Warmup time: " + warmupTime + " ms");  
  
        // 再次执行，此时JIT编译已经生效  
        start = System.currentTimeMillis();  
        for (int i = 0; i < 10000000; i++) {  
            compute(i);  
        }  
        long jitTime = System.currentTimeMillis() - start;  
        System.out.println("JIT time: " + jitTime + " ms");  
    }  
  
    private static int compute(int n) {  
        return n * n + 2 * n + 1 + n * 2;  
    }  
}
```
![Dingtalk_20241015223306.jpg](https://s2.loli.net/2024/10/15/1b4dZuxVjCt5lAh.jpg)
通过以上代码可以看到JIT编译器在运行期间进行了一些优化。


## 7. 引用
- 《深入理解Java虚拟机》
- 美团技术团队：https://tech.meituan.com/2020/10/22/java-jit-practice-in-meituan.html