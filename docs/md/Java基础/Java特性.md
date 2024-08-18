作者： 陈汤姆
<br/>博客：[https://github.com/linglongchen/linglongchen.github.io](https://github.com/linglongchen/linglongchen.github.io)

>自我学习的积累！😄


## 基本数据类型
### 数字类型
- byte：占用1个字节
- shot：占用2个字节
- int：占用4个字节
- long：占用8个字节
- floag：占用4个字节
- double：占用8个字节

### 字符类型
- char：占用2个字节


## 布尔类型
- boolean：占用1个字节


## 面向对象概念
### 封装
将属性和行为结合在一起，一般通过private、public、protected

### 继承
子类继承父类，拥有父类的访问权限，从而实现代码的复用和二次扩展。Java中通过extends实现

### 多态
#### 重载
方法名相同但是方法参数个数和类型不同；编译时多态

#### 重写
一般通过重写接口的方法实现；运行时多态



## 接口和抽象类
### 接口
通过interface关键字实现。实现类通过implement实现接口
```java
/**  
 * @Author: CLL02  
 * @Date: 2024/8/16 21:23  
 * @Version: v1.0  
 * @Description: 测试接口  
 **/  
public interface TestInterface {  
    /**  
     * 接口的方法  
     * @return  
     */  
    Object test();  

}



/**  
 * @Author: CLL02  
 * @Date: 2024/8/16 21:24  
 * @Version: v1.0  
 * @Description: 接口的实现类  
 **/  
public class TestInterfaceImpl implements TestInterface {  
  
    /**  
     * 重写接口的方法  
     * @return  
     */  
    @Override  
    public Object test() {  
        return null;  
    }  
}
```

### 抽象类
通过abstract关键字实现
```java
/**  
 * @Author: CLL02  
 * @Date: 2024/8/16 21:26  
 * @Version: v1.0  
 * @Description: 抽象类  
 **/  
public abstract class AbstractTestClass {  
  
  
    public Object testAbstract(){  
        return null;  
    }  
  
    protected Object testAbstract(Object o) {  
        return null;  
    }  
}
```

## 泛型
Java泛型是通过类型擦除来实现的。这意味着在编译之后，所有具体的类型参数信息都会被擦除，替换为它们的上限类型（通常是Object），并且生成的字节码中不会保留泛型信息。

### 类型擦除
- 编译器类型检查
- 运行期类型擦除
- 类型参数的替代（在编译后的字节码中，泛型类型参数被替换为限定类型，而原始类型（如 `List<String>`）被替换为 List）

由于类型擦除，泛型类型参数在运行时无法通过反射获得。

## SPI
SPI（Service Provider Interface）是一种机制，允许开发者为某些操作实现一个或多个服务提供者，然后可以在运行时从多个提供者中选择一个合适的服务来执行操作。SPI是一**种动态加载**机制，它允许框架和库在**运行时动态地**查找和加载服务提供者

### SPI优势

- 解耦
- 可拔插
- 扩展性
#### 示例
META-INF/services实现接口
```java
ServiceLoader<PaymentService> serviceLoader = ServiceLoader.load(PaymentService.class);    
for (PaymentService service : serviceLoader) {    
service.pay(100.0); 
// 使用服务提供者执行支付操作  
}
```
通过ServiceLoader实现加载和调用
## 序列化
用于网络传输、数据存储、进程通信，将对象以字节的形式进行传输和存储

#### 序列化
将对象转为字节流

#### 反序列化
将字节流转为对象

## 语法糖
### 自动拆箱装箱
```java
Integer integer = 10; // 装箱
int num = integer; // 拆箱
```

### 泛型

```java
List<String> list = new ArrayList<>(); // 泛型
```
### 三目表达式
```java
int max = (a > b) ? a : b;
```

## foreach循环
```java
for (String str : strings) {
    System.out.println(str);
}
```


### 方法引用
```java
List<String> list = files.stream().filter(Files::isRegularFile);
```

### 变长参数
```java
public void printAll(Object... args) {
    for (Object obj : args) {
        System.out.println(obj);
    }
}
```

### 内部类
```java
public class OuterClass {
    public class InnerClass {
        // 内部类的内容
    }
}
```

### 注解
添加到类、方法、变量、参数上，用于提供程序元数据（程序运行期间可以使用的数据）
注解是Java语言的一个强大特性，它允许开发者以声明式的方式为代码添加元数据，这些元数据可以被**编译期或运行时**框架所使用，从而提高代码的可读性和可维护性。

### 类型推断
允许编辑器通过上下文推断出变量类型
```java
// 推断出map的类型为Map<String, List<String>>
Map<String, List<String>> map = new HashMap<>();
```

### null合并运算符
```java
String nonNullValue = possiblyNullValue ?? "default";
```


## 创建对象的方式

### new关键字
```java
public class TestInterfaceImpl implements TestInterface {  
    TestInterfaceImpl anInterface = new TestInterfaceImpl();   
}
```

### 反射-newInstance
```java
Class<?> clazz = Class.forName("包名.MyClass");
MyClass obj = (MyClass) clazz.newInstance(); 
// 或者 clazz.getDeclaredConstructor().newInstance();
```

### 克隆-实现Cloneable接口
```java
MyClass original = new MyClass();
MyClass clone = original.clone();
```

### 通过类加载器
```java
ClassLoader classLoader = ClassLoader.getSystemClassLoader();
Class<?> clazz = classLoader.loadClass("包名.MyClass");
MyClass obj = (MyClass) clazz.newInstance();
```

### 使用Spring容器
```java
// 假设使用Spring框架
MyClass obj = context.getBean(MyClass.class);
```


