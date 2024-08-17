
> SPI全称Service Provider Interface，是Java提供的一套用来被第三方实现或者扩展的API，它可以用来对框架的接口进行扩展。俗称面向服务编程
> 与之相对的就是API，API俗称面向接口编程，两者在实现维度上不同，但是核心思想是相同的，都是提高可扩展性。


对SPI网上说的最多的一个例子就是数据库的驱动接口java.sql.Driver，每个数据库厂商可以基于此接口实现自己的驱动程序，比如MySQL的实现类是com.mysql.jdbc.Driver。

所以理解下来就是SPI接口定义标准，实现具体功能时只要按照这个标准实现并按照流程注入即可。

但是网上并没有给出很好的例子来理解Driver的注册和最终调用的流程，都只是在main方法中实现一层服务加载器加载SPI实现类。

以下我就根据自己的理解，从0到1实现从**定义SPI、定义SPI实现类、实现类服务加载、实现类服务注册、实现类服务调用**梳理SPI的使用，加深对SPI的理解。


### 定义SPI标准接口
```
/**
 * @author chenlingl
 * @version 1.0
 * @description spi的标准接口
 * @date 2023/2/25 14:56
 */
public interface AnimalSpi {
    /**
     * 执行流程
     * @param source
     * @return
     */
    String process(String source);

    /**
     * 初始化实现类
     * @return
     */
    AnimalSpi newInstance();
}
```

### 定义SPI的实现类
```
/**
 * @author chenlingl
 * @version 1.0
 * @description
 * @date 2023/2/25 15:18
 */
@Service
public class CatSpiService implements AnimalSpi {
    @Override
    public String process(String source) {
        System.out.println("执行SPI的实现类!"+source);
        return source;
    }

    @Override
    public AnimalSpi newInstance() {
        return this;
    }
}
```
```
/**
 * @author chenlingl
 * @version 1.0
 * @description
 * @date 2023/2/25 14:57
 */
@Service
public class DogSpiService implements AnimalSpi {
    @Override
    public String process(String source) {
        System.out.println("执行SPI的实现类!"+source);
        return source;
    }

    @Override
    public AnimalSpi newInstance() {
        return this;
    }
}
```


### SPI实现类注册
```
/**
 * @author chenlingl
 * @version 1.0
 * @description
 * @date 2023/2/25 14:59
 */
@Component
public class SpiClassLoader {

    @PostConstruct
    public void loadInit() {
        ServiceLoader<AnimalSpi> serviceServiceLoader = ServiceLoader.load(AnimalSpi.class);
        for (AnimalSpi spiService : serviceServiceLoader) {
            //首先通过实例化的方法获取实现类实例
            AnimalSpi animalSpi = spiService.newInstance();
            //将实例注册到工厂类的容器中
            SpiFactory.register(animalSpi.getClass().getName(), animalSpi);
        }
    }
}
```


### SPI实现类的工厂模式
```
/**
 * @author chenlingl
 * @version 1.0
 * @description
 * @date 2023/2/25 15:14
 */
public class SpiFactory {
    //这里默认使用了防止并发的map
    private static Map<String, AnimalSpi> animalMap = new ConcurrentHashMap<>();

    //获取SPI的实现类实例
    public static AnimalSpi getInstance(String key) {
        return animalMap.get(key);
    }

    //注册SPI的实现类：key是spi实现类的全限定类名
    public static void register(String key, AnimalSpi animalSpi) {
        animalMap.put(key, animalSpi);
    }
}
```


### 定义SPI的实现路径

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c9e90cc90a844c3995d1c758b9575841~tplv-k3u1fbpfcp-watermark.image?)



### SPI实现类调用
```
/**
 * @author chenlingl
 * @version 1.0
 * @description
 * @date 2023/2/25 15:09
 */
@RestController
public class SpiController {

    @GetMapping("/executeSpi")
    public String executeSpi(String source) {
        AnimalSpi spiService = SpiFactory.getInstance(source);
        return spiService.process(source);
    }
}
```

### 测试结果
> 测试请求链接：http://localhost:8080/executeSpi?source=com.modules.boot.spi.service.DogSpiService


![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f11f7483bf114d5493d9e58d14e80053~tplv-k3u1fbpfcp-watermark.image?)
