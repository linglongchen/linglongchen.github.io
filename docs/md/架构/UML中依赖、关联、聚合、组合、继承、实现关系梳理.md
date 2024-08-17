## 背景

最近看源码过程中需要梳理一些类之间的关系，在画图过程中梳理了关系缺不知道怎么表示，理不清到底聚合、组合、关联等这些关系的表示，特深入学习一波，理清各关系的特点，用于后续的复习。

  


### 依赖

依赖关系在实际开发过程中经常使用的一种关系，往往会存在我们使用的client类需要引用supplier类中的某些方法，因此我们需要依赖supplier类，引用它的方法实现client类中的功能。并且如果更改suipplier类可能还需要更改client，两者存在supplier影响clinet的情况。

通过UML图表示如下：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/567b5e3b85ae41b59a4abbb5834af5b5~tplv-k3u1fbpfcp-zoom-1.image)

从上图中可以看到依赖的关系是虚线+开口箭头表示的。其实也好理解，依赖并不是一种很强的关系，因为更改了supplier类只是可能会影响client类，而不是一定会影响client类，所以用虚线也好理解。

依赖关系对于client类来说需要实现以下的一项功能：

-   将supplier类作为全局变量进行使用
-   将supplier类作为局部变量进行使用
-   将supplier类作为client类中某个方法的参数使用
-

在Java的具体实现中如下：

```
/**
 * @author chenlingl
 * @version 1.0
 * @description
 * @date 2023/2/19 11:42
 */
public class Client {
    //将Supplier作为全局变量使用
    private Supplier supplier;

    public void setSupplier(Supplier supplier) {
        this.supplier = supplier;
    }

    public Supplier getSupplier() {
        return supplier;
    }
}


/**
 * @author chenlingl
 * @version 1.0
 * @description 
 * @date 2023/2/19 11:42
 */
public class Supplier {
}
```

  


  


```
/**
* @author chenlingl
* @version 1.0
* @description
* @date 2023/2/19 11:42
*/
public class Client {

    public void execute(Supplier supplier) {
        //将Supplier作为入参使用
    }
}


/**
 * @author chenlingl
 * @version 1.0
 * @description
 * @date 2023/2/19 11:42
 */
public class Supplier {
    private String key;
    private String value;
}
```

  


  


```
/**
* @author chenlingl
* @version 1.0
* @description
* @date 2023/2/19 11:42
*/
public class Client {

    public void execute() {
        //将Supplier作为局部变量使用
        final Supplier supplier;
    }
}


/**
 * @author chenlingl
 * @version 1.0
 * @description
 * @date 2023/2/19 11:42
 */
public class Supplier {
    private String key;
    private String value;
}
```

### 关联

关联关系是一种体现两个类之间是否互相知道的一种关系，即是否可以通过一个类知道另一个类，比如老师和学生，老师可以拥有多个学生，每个学生也可以拥有多个老师，因此老师和学生的关联关系就是多对多的关系，并且知道一个老师就可以知道这个老师下面所有的学生，知道一个学生就可以知道这个学生的所有老师。

再比如丈夫和妻子，两个合法关系下一个丈夫只能拥有一个妻子，一个妻子也只能拥有一个丈夫，因此知道丈夫的情况下就可以知道该丈夫的妻子，知道妻子就可以知道该妻子的丈夫，这是双向的一种关联。

那么还存在一种关联就是单向的，即只能由A类知道关联的B类，而不能通过B类知道A类。

在UML中表示如下：

双向关联

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a73e208f7d9d4b71b8650e51cbefe1d5~tplv-k3u1fbpfcp-zoom-1.image)

单向关联

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/adadc253c77546d5bcb1dd2842839fbb~tplv-k3u1fbpfcp-zoom-1.image)

关联关系在实际使用中需要实现任何一项功能：

-   两个类之间可以通过属性的方式实现

在Java中的具体实现：

```
/**
* @author chenlingl
* @version 1.0
* @description
* @date 2023/2/19 13:42
*/
public class Association {
    //关联client类，将client类作为连接使用
    private Client connect;
    //关联自己
    private Association next;
}
```

以上的例子中可以通过Association知道关联的Client类，但是通过Client类无法知道该类关联了啥。因此以上的例子是一种单向的关联关系。

  


### 聚合

聚合关系顾名思义就是类聚合形成另一个类。例如汽车作为一个类，该类由轮胎、框架、汽车玻璃、座椅等聚合形成的汽车。因此聚合的意义就在于通过各独立的类聚合形成另一个类，这个类具备完整的功能。

聚合关系中强调的是聚合和独立，聚合比较好理解就是多个类聚合形成另外一个类，那么独立怎么理解？

独立的意义就在多个类是可以独立存在的，比如乐高的积木，我们可以通过乐高砖块搭建一个城堡，那么一个个的乐高砖块就是独立的类，这些独立存在的类可以搭建我们想要的城堡，还可以搭建桥梁、动物等等。即使这些乐高砖块脱离了城堡也可以随意的去搭建其它的模型。

在UML中表示如下：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/813ba4e7928448d894e663ef31c9a098~tplv-k3u1fbpfcp-zoom-1.image)

  


在Java中体现如下：

```
/**
* @author chenlingl
* @version 1.0
* @description
* @date 2023/2/19 14:11
*/
public class Castle {

    private Brick brick;

    public Castle(Brick brick) {
        this.brick = brick;
    }
}


/**
 * @author chenlingl
 * @version 1.0
 * @description
 * @date 2023/2/19 14:12
 */
public class Brick {
}
```

### 组合

组合关系和聚合关系存在一定的迷惑性，傻傻分不清，其实直接咬文嚼字也可以理解，网上有一句话比较有意思，聚是一坨翔，散是满天星，那么是不是可以理解聚合就是多个独立的各自形成一个大的个体，既然可以聚那么就可以散。

而组合就不一样了，比如人是由四肢、大脑、内部器官组成，既然用了组成这个词那就说明如果人没了，那么器官当然也不能生存了，即四肢、大脑、内部器官是不能脱离人这个主体存在的。

以上的例子就说明了组合这种关系是依赖主体存在的，如果脱离主体则不能独立。

从实际开发中举例来说，我所接触的业务是物流行业，那么在物流行业中我们会实例化一个物订单的主体，这个主体由包裹、收发件人等信息组成，通常我们都会使用一个状态机实体来表示订单的物流状态，比如揽收、分拨、干线、派送等，那么如果物流订单丢失了，那么这个订单的状态机实体也就无意义了，因此这个状态机的主体数据已经丢失了，这个状态机也没有可以表示的意义了。

在UML中体现如下：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c327c50418594d3bbfcfc7a0db580022~tplv-k3u1fbpfcp-zoom-1.image)

在Java中体现如下：

```
/**
* @author chenlingl
* @version 1.0
* @description
* @date 2023/2/19 14:31
*/
public class LogisticsOrder {
    /**
* 订单状态机
*/
    private OrderStatusMachine orderStatusMachine;
}
```

```


/**
* @author chenlingl
* @version 1.0
* @description
* @date 2023/2/19 14:31
*/
public class OrderStatusMachine {
    /**
* 前一个状态
*/
    private Integer preStatus;
    /**
* 当前状态
*/
    private Integer curStatus;
    /**
* 下一个状态
*/
    private Integer nextStatus;
    /**
* 是否允许变更状态
*/
    private Boolean isExecute;
}
```

### 继承（泛化）

继承关系相信学习面向对象开发的都不陌生，在面向对象设计中三大特性就是封装、继承、多态。

继承在各语言中都会有不同的实现，在Java中就是通过extends关键字实现的。

继承的定义：子类可以拥有父类所有的功能，这样在不重复编码的情况下子类拥有父类的功能。

这也是平时经常使用的一种关系，比如开发过程中我们会抽象一些基础属性出来，将基础属性作为基类，所有需要这些基础属性的类可以通过继承的方式获得这些基础属性，从而避免了在每个子类中都重复写相同的代码。

  


但是在Java开发过程中使用继承也有一定的局限性，因为只能单继承，因此如果继承了一个类不能再继承其它类，因此对于基类的抽象要更加具体并复用性更高。

在UML中体现关系如下：

  


![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b99716fde16b4e24a0a697418d987231~tplv-k3u1fbpfcp-zoom-1.image)

继承关系中child可以继承parent类中所有的功能。

在Java中实现如下：

```java
/**
* @author chenlingl
* @version 1.0
* @description 产品基类，定义产品code和版本id
* @date 2023/2/19 17:32
*/
public class BaseProduct {
    private String productCode;
    private Long versionId;
}


/**
 * @author chenlingl
 * @version 1.0
 * @description 标准产品
 * @date 2023/2/19 17:33
 */
public class StandardProduct extends BaseProduct {
    private String channelCode;
}


/**
 * @author chenlingl
 * @version 1.0
 * @description 销售产品
 * @date 2023/2/19 17:34
 */
public class SaleProduct extends BaseProduct{
    private Integer price;
}


/**
 * @author chenlingl
 * @version 1.0
 * @description 运营产品
 * @date 2023/2/19 17:34
 */
public class OperateProduct extends BaseProduct{
    private String operateName;
}
```

以上的例子中将产品的产品code和版本抽象为基类，所有具象化的产品则继承该基类得到所有的属性。

### 实现

实现这种关系也是比较好理解以及平时经常使用的一种关系，实现关系中需要存在接口，接口是定义标准的一方，而接口的实现类则是实现定义的标准方法。

实现关系在面向对象中可以被多实现，即一个接口可以存在多个实现类，都可以实现相同的接口，并且重写接口中的方法。

在Java语言中实现关系是通过implements关键字实现的。

在UML中体现关系如下：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ec56a136a9634c1e84c3b90826629134~tplv-k3u1fbpfcp-zoom-1.image)

实现的关系需要存在interface，即提供实现方法的接口，interface提供方法，classImpl和classImpl2实现Interface中的process方法。

在Java中体现如下：

```
/**
* @author chenlingl
* @version 1.0
* @description 标准接口
* @date 2023/2/19 17:56
*/
public interface StandardService {

    void create(Object obj);

    void commit(Object obj);

    void destroy(Object obj);
}



/**
 * @author chenlingl
 * @version 1.0
 * @description 标准接口的实现类
 * @date 2023/2/19 17:57
 */
public class StandardServiceImpl implements StandardService{
    @Override
    public void create(Object obj) {

    }

    @Override
    public void commit(Object obj) {

    }

    @Override
    public void destroy(Object obj) {

    }
}
```

## 总结

以上就是关于自己对UML中各关系的梳理和总结，掌握了这些关系，平时在建模时搞得懂各模型之间的关系，以及在系统设计过程中，对于讲清楚细节上的各种关系是比较重要的。以上只是自己学习上的梳理和总结，如有错误欢迎指正。



## 引用
- [IBM](https://www.ibm.com/docs/zh/rsm/7.5.0?topic=diagrams-composition-association-relationships)
