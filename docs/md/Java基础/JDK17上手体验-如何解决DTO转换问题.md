


##  JDK17新特性
JDK 17 引入了一系列新特性和改进。以下是一些主要的新特性：

1. **封装 JDK 核心类库的 API**：
    
    - JDK 17 继续引入了对 JDK 核心类库的封装，尤其是对 `java.base` 模块的 API 进行保护，限制对内部 API 的使用。
2. **Pattern Matching for `instanceof`**：
    
    - 通过 `instanceof` 操作符，可以直接将对象进行类型检查和转换，这样可以简化代码。

   `if (obj instanceof String s) {       // 这里可以直接使用 s   }`

3. **Sealed Classes**：
    - 允许类控制哪些其他类可以继承它，从而实现更严格的类型安全。

   `public sealed class Shape permits Circle, Square { }`

4. **新增强的 `switch` 表达式**：
    - 提供了更灵活的 `switch` 表达式，支持多规则匹配。

   `switch (day) {       case MONDAY, FRIDAY -> System.out.println("Happy Day!");       case TUESDAY -> System.out.println("Second day of the week");   }`

5. **Foreign Function & Memory API (预览)**：
    
    - 允许 Java 程序调用本地代码和操作本地内存，提供了一种与非 Java 代码交互的新方式。
6. **新语言功能**：
    
    - 引入了 `text blocks` 的许多改进，使多行字符串更易于编写和阅读。
7. **新标准库功能**：
    
    - 新增如 `java.nio.file` 中的 API，改善了对文件系统的处理。
8. **Deprecation**：
    
    - JDK 17 标记了一些不推荐使用的特性，例如 Applet API，表明它们将在未来版本中被移除。
9. **更新的垃圾回收器**：
    
    - 对 ZGC 和其他垃圾回收器进行了性能优化，提高了内存管理效率。
10. **JEP 411: Deprecate the Security Manager for Removal**：
    
    - 计划在将来的版本中去掉安全管理器。


## 对象转换实战

```java
@Data  
public class CheckInRecordDTO {  
    private Long userId;  
    private Long planId;  
    private Date checkInTime;  
    private String checkInData;  
}

public void test() {
	CheckInRecord newObject = BeanCopierUtils.toNewObject(recordDTO, CheckInRecord.class);
	
}
```
![[Pasted image 20240926125052.png]]
![[Pasted image 20240926125123.png]]


可以看到由于JDK17中引入模块系统，由于模块的封闭性，反射访问 `ClassLoader` 的 `defineClass` 方法访问被拒绝。


提供了两种方案解决，如果是老项目升级JDK17，那么Java项目启动时需要配置：
```shell
   java --add-opens java.base/java.lang=ALL-UNNAMED -jar your-application.jar
```

如果是新项目则直接替换掉BeanCopier即可，实测可以使用BeanUtils.

```java
import org.springframework.beans.BeanUtils;

public void test() {
	BeanUtils.copyProperties(recordDTO, checkInRecord);
}

```