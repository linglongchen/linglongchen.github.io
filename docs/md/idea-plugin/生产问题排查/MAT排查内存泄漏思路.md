# MAT排查内存泄漏思路





## 现象

应用部署配置：
4C8G-60G存储
堆内存：2G（50%）
MetaspaceSize：512M
MaxDirectMemorySize：1G
垃圾收集器：CMS
部署机器：16台



![Dingtalk_20240822211923.jpg](https://s2.loli.net/2024/08/22/zM6oAChf8RXkLt2.jpg)
728我们系统在凌晨出现FGC导致出现告警，我们的FGC一天只会出现几次，并且不会连续的产生，因此这次FGC需要我们额外排查，以下是通过MAT排查的OOM的思路，仅供参考。

## 排查思路


### dump文件
dump文件记得把当前的问题机器的流量停止，避免dump文件影响正常的业务处理。

- 手动dump：jmap -dump:format=b,file=java.hprof 'java进程PID'

- 自动dump：发生OOM时，自动dump当时的堆栈文件，-XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/home/admin/logs/java.hprof（文件地址可能不同）

我们应用接入阿里云因此会自动dump。


### Leak Suspects查看内存泄漏报告
将dump文件导入MAT可以看到概览图，可以点击Leak suspects查看内存泄漏的报告

![image _3_.png](https://s2.loli.net/2024/08/22/l5s1nFU4Ix7kcRq.png)

### Domainator Tree查看占用内存的大对象
具体的排查需要根据内存泄漏的对象排查，因此点击Domainator Tree查看内存占用情况。

![image _4_.png](https://s2.loli.net/2024/08/22/tFKC6y94YGhH25N.png)

### 定位大对象数据
打开内存分布图可以看到内存的87%都被一个对象占用，点开查看具体的对象，

![image _5_.png](https://s2.loli.net/2024/08/22/7KEsTncxGfjYedW.png)
![image _6_.png](https://s2.loli.net/2024/08/22/XAPeH58QGyLgIi9.png)
可以看到87%的内存被占用在三个大对象中，一个jackson文本、一个char类型的大对象、一个数据列表
![image _7_.png](https://s2.loli.net/2024/08/22/T9mvz4jyY2hCUsW.png)
继续点开大文件查看，看到整个json文本由1645个对象组成，共占用内存30%


### 定位具体数据
![image _8_.png](https://s2.loli.net/2024/08/22/g8zXiMcENlS295F.png)

将引发OOM的数据通过copy出来解析，查看具体的异常数据

### 查看OOM的异常堆栈（发生时刻的堆栈不一定是导致OOM的堆栈）

![Dingtalk_20240822212736.jpg](https://s2.loli.net/2024/08/22/Zc1YHVetD8hEfxp.jpg)

### 定位异常

通过异常数据定位具体的代码，伪代码如下：
```Java
public class OOM {  
  
    public String buildResponse(BatchInfo batchInfo) {  
        //todo 第三部分的内存占用，整个list没有被回收因此占用了30%内存  
        List<SubContext> list = new ArrayList<>();  
        for (String containerNo : batchInfo.getContainerNo()) {  
            SubContext subContext = new SubContext();  
            subContext.setBatchNo(batchInfo.batchNo);  
            //将整个对象塞到了上下文中，并且整个批次中包裹1645个容器  
            subContext.setContainer(new Container());  
            //将参数的整个对象  
            list.add(subContext);  
        }  
        //todo 第一部分和第二部分的内存占用，由于整个大对象需要做json序列化占用30%内存；并且序列化之后产生一个字符串数组，占用27%内存  
        return JSONUtils.toJSONString(list);  
    }  
  
  
  
    @Data  
    class BatchInfo {  
        private String batchNo;  
        private List<String> containerNo;  
    }  
  
  
  
    @Data  
    class SubContext {  
        private String batchNo;  
        private Container container;  
    }  
  
  
    @Data  
    class Container {  
        private String containerNo;  
        private String containerExtension;  
    }  
}
```

根据以上的内存占用写了一个仿异常的代码，87%内存占用分三部分：
- JSONUtils.toJSONString(list)：将大对象做序列化占用了30%内存
- 序列化之后产生字符串，形成大对象的字符串数组占用内存27%
- List列表内存没有被回收，形成大对象列表，占用内存30%


## 总结

以上就是使用MAT分析内存泄漏的思路，欢迎评论交流。


