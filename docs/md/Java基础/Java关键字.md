

## final
- 修饰类表示该类不能被继承
- 修饰方法表示该方法不能被重写
- 修饰基本类型变量表示该变量的值不能被修改
- 修饰引用类型变量则**引用地址**不会被改变，始终指向一个对象
```Java
public class TestFinal {  
  
    public static final String TEST_FINAL = "testFinal";  
  
  
	
    public void test() {  
        TEST_FINAL = "UPDATE";  
    }  
}
```
![[Pasted image 20240816214702.png]]
## static
修饰变量，则变量是类级别，不属于任何实例。静态变量在编译时就会初始化

```java
class MyClass {
    public static test = 100;
}


class MyClass2 {
    int mapping = Myclass.test;
}
```
## volatile
如果一个变量被声明为volatile，则Java线程内存模型确保所有线程看到这个变量的值是一致的

### 特性
- 可见性
	- 修饰变量，保证变量的可见性
		- 若一个线程修改了volatile修饰的变量，那么会将改制写入系统内存中，同时使其它线程缓存的该变量的内存地址设置为无效
		- 每个处理器通过**嗅探**在总线上传播的数据来检查自己缓存的数据是不是过期了
		- 实现原理：**Lock前缀指令**会引起处理器缓存回写到内存；一个处理器的缓存回写到内存会导致其它处理器的缓存无效
- 禁止重排序
	- volatile修饰的变量在多线程环境下不会被**指令重排序**；Java中通过**内存屏障**来保障被volatile修饰的变量的重排序

```java
public class AdapterFactory {  
  
    /**  
     * 适配map  
     */    private static volatile Map<Integer,AdapterRouterService> adapterRouterServiceMap = new ConcurrentHashMap<>();  
  
    /**  
     * 创建适配实现类  
     * @param taskCallback  
     * @return  
     */  
    public AdapterRouterService createAdapter(TaskCallback taskCallback) {  
        if(CollectionUtils.isEmpty(adapterRouterServiceMap)) {  
            Map<Integer,AdapterRouterService> tmpMap = new ConcurrentHashMap<>();  
            Map<String,AdapterRouterService> beanMap = SpringContextUtils.getContext().getBeansOfType(AdapterRouterService.class);  
            for (AdapterRouterService value : beanMap.values()) {  
                Integer accDataType = value.adapterType();  
                if(null != accDataType && accDataType != 0){  
                    tmpMap.put(accDataType,value);  
                }  
            }  
            adapterRouterServiceMap = tmpMap;  
        }  
        if (Objects.nonNull(taskCallback) && StringUtils.isNotBlank(taskCallback.getWaveCode())) {  
            return adapterRouterServiceMap.get(AdapterTypeEnum.TASK_ADAPTER.getVal());  
        }  
        return adapterRouterServiceMap.get(AdapterTypeEnum.AUTOMATIC_ADAPTER.getVal());  
    }  
}
```
## synchronized

### 特性
- 线程安全
- 原子性
- 可见性
- 可重入性
- 锁升级
	- 偏向锁：线程访问同步块获取锁时会在对象头和帧栈中的锁记录中存储偏向锁的线程ID，以后该线程进入和退出同步代码块时不需要进行CAS操作来加锁和释放锁，只要对比对象头中是否指向的是当前线程ID；如果出现其它线程竞争同步代码块的对象，则偏向锁会撤销，进而升级为轻量级锁
	- 轻量级锁：轻量级锁通过CAS的方式让线程自旋获取对象的锁，在自旋范围内获得了锁，则不会升级为重量级锁；通过-XX:PreBlockSpin控制自旋次数，一般默认**10**次
	- 重量级锁
- 锁粗化
	- 发生在Java虚拟机（JVM）的即时编译器（JIT）层面。目的是减少不必要的锁操作，提升程序的性能。
	- 假设有一个循环，循环体内有一段需要同步的代码，如果不进行锁粗化，每次循环迭代都会获取和释放锁。锁粗化之后，整个循环只会在循环开始时获取一次锁，在循环结束时释放锁，使循环体内的代码被执行了多次。
- 锁消除
	- 发生在即时编译器（JIT）阶段，当JVM检测到某些代码块或方法尽管被同步代码包围，但实际上并不涉及共享数据的竞争，或者锁对象根本没有逃逸出该线程的上下文时，JIT编译器就会执行锁消除。
	- 原理
		- **逃逸分析**：通过逃逸分析，JVM可以确定对象是否有可能被其他线程访问。如果一个对象的引用没有逃逸出线程，即该对象只在单一线程内访问，那么对该对象上的同步操作就是不必要的，因为不存在数据竞争的风险。
		- 无数据竞争：当JVM确定一个同步块或方法中的数据不会被其他线程访问或修改时，就可以安全地消除这些同步操作。
		- 局部变量和私有对象：尤其适用于那些只在方法内部创建、使用并且没有发布到其他线程的锁对象，比如一个局部变量作为锁，或者一个线程私有的对象作为锁

### 锁的形式
- 锁方法：锁的是**当前实例**的对象
- 静态方法：锁的是当前类的对象
- 同步方法块：锁的是代码块中的对象

### 实现原理
- 本质是对一个对象的**监视器**进行获取，这个获取过程是**排他**的，同一时刻只能有一个线程获取到synchronized所保护对象的监视器
- 对于**同步代码块**使用monitorenter和monitorexit指令控制是否可以获取锁
- 对于**方法**则依靠方法修饰符上的**ACC_SYNCHRONIZED**来完成

## finalize
- 方法的主要目的是在对象被垃圾回收器（Garbage Collector, GC）从内存中清除之前，给予对象最后一次机会去执行一些必要的清理工作，比如关闭文件、释放系统资源等。
- 如果需要在对象被回收前执行特定操作，可以重写 finalize() 方法。在重写时，通常会调用超类的 finalize() 方法以确保基类也有机会执行清理工作
- 从Java 9开始，finalize() 方法已被标记为不推荐使用，并在Java 17中被正式弃用