# 生产使用Sentinel限流

作者： 陈汤姆
<br/>博客：[https://github.com/linglongchen/linglongchen.github.io](https://github.com/linglongchen/linglongchen.github.io)

>自我学习的积累！😄


## 背景

目前接触的项目中存在一些大报文请求，比如一个批次挂载5k+大包数量，系统在处理大包列表时需要做解析，对单机会造成FGC，如果大批量的大报文请求打到系统会导致应用全部FGC，因此需要对这类大报文请求做限流处理，如果超过限流将大报文请求写入调度任务表挂起，隔一段时间重新唤起。

基于以上背景使用Sentinel实现了限流，目前生产环境限流的控制达到预期。



## 实现方案

对于限流本次我们使用了sentinel中的热点参数限流的方案，即使用sentinel-parameter-flow-control实现，以下是实现方案。


### pom依赖
```java
<?xml version="1.0" encoding="UTF-8"?>  
<project xmlns="http://maven.apache.org/POM/4.0.0"  
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">  
    <modelVersion>4.0.0</modelVersion>  
	<parent>  
	    <groupId>org.springframework.boot</groupId>  
	    <artifactId>spring-boot-starter-parent</artifactId>  
	    <version>2.2.7.RELEASE</version>  
	    <relativePath/> <!-- lookup parent from repository -->  
	</parent>
    <artifactId>sentinel</artifactId>  
  
    <properties>  
        <maven.compiler.source>8</maven.compiler.source>  
        <maven.compiler.target>8</maven.compiler.target>  
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>  
    </properties>  
  
  
    <dependencies>  
        <dependency>  
            <groupId>com.alibaba.csp</groupId>  
            <artifactId>sentinel-parameter-flow-control</artifactId>  
            <version>1.8.2</version>  
        </dependency>  
        <dependency>  
            <groupId>org.springframework.cloud</groupId>  
            <artifactId>spring-cloud-context</artifactId>  
            <version>2.1.2.RELEASE</version>  
            <scope>compile</scope>  
        </dependency>  
        <dependency>  
            <groupId>org.springframework.boot</groupId>  
            <artifactId>spring-boot-starter-web</artifactId>  
        </dependency>  
  
    </dependencies>  
  
</project>
```

### 定义限流规则
```java
  
import com.alibaba.csp.sentinel.slots.block.RuleConstant;  
import com.alibaba.csp.sentinel.slots.block.flow.param.ParamFlowItem;  
import com.alibaba.csp.sentinel.slots.block.flow.param.ParamFlowRule;  
import lombok.Data;  
  
import java.util.HashMap;  
import java.util.List;  
import java.util.Map;  
import java.util.Objects;  
import java.util.stream.Collectors;  
  
  
/**  
 * @author CLL02  
 * @description 限流规则  
 */  
@Data  
public class HotMsgLimitRule {  
    //默认限流数量
    private static final Integer DEFAULT_LIMIT_COUNT = 10;  
    //限流配置map
    private Map<String, Integer> msgTypePermitsMap;  
    //限流数量
    private Long limitDurationInSec;  
    //资源名称
    private String resourceName;  
  
    public HotMsgLimitRule(String resourceName, Map<String, Integer> msgTypePermitsMap, Long limitDurationInSec) {  
        //重新初始化对象，避免出现配置刷新的时候，属性同样被更新，无法判断配置是否有变更  
        this.msgTypePermitsMap = new HashMap<>(msgTypePermitsMap);  
        this.limitDurationInSec = limitDurationInSec;  
        this.resourceName = resourceName;  
    }  
  
    public ParamFlowRule toParamFlowRule() {  
        //固定资源名称  
        ParamFlowRule rule = new ParamFlowRule(resourceName);  
        //设置限流的配置  
        List<ParamFlowItem> flowItems = msgTypePermitsMap.keySet().stream().map(key -> {  
            ParamFlowItem flowItem = new ParamFlowItem();  
            flowItem.setObject(key);  
            flowItem.setCount(msgTypePermitsMap.getOrDefault(key, DEFAULT_LIMIT_COUNT));  
            flowItem.setClassType(String.class.getName());  
            return flowItem;  
        }).collect(Collectors.toList());  
        rule.setParamFlowItemList(flowItems);  
        //设置为不等待  
        rule.setMaxQueueingTimeMs(0);  
        rule.setParamIdx(0);  
        rule.setDurationInSec(limitDurationInSec);  
        rule.setGrade(RuleConstant.FLOW_GRADE_QPS);  
        return rule;  
    }  
  
    @Override  
    public boolean equals(Object o) {  
        if (this == o) {  
            return true;  
        }  
        if (!(o instanceof HotMsgLimitRule)) {  
            return false;  
        }  
        HotMsgLimitRule that = (HotMsgLimitRule) o;  
        return Objects.equals(msgTypePermitsMap, that.msgTypePermitsMap) && Objects.equals(limitDurationInSec, that.limitDurationInSec);  
    }  
  
    @Override  
    public int hashCode() {  
        return Objects.hash(msgTypePermitsMap, limitDurationInSec);  
    }  
}
```


### 限流实现
```java
import com.alibaba.csp.sentinel.Entry;  
import com.alibaba.csp.sentinel.EntryType;  
import com.alibaba.csp.sentinel.SphU;  
import com.alibaba.csp.sentinel.slots.block.BlockException;  
import lombok.extern.slf4j.Slf4j;  
import org.springframework.stereotype.Component;  
  
import javax.annotation.Resource;  
  
/**  
 * @Author: CLL02  
 * @Date: 2024/9/15 20:41  
 * @Version: v1.0  
 * @Description: 限流服务实现  
 **/  
@Component  
@Slf4j  
public class LimitService {  
  
  
    public static final String MSG_LIMIT_RESOURCE_NAME = "msgLimiter";  
  
    @Resource  
    private LimitConfig limitConfig;  
  
  
    /**  
     * 是否限流  
     * @param apiPath  
     * @return  
     */  
    public boolean isLimit(String apiPath) {  
        //判断是否对api做限流  
        if (!isLimitApi(apiPath)) {  
            return false;  
        }  
        Entry entry = null;  
        try {  
            entry = SphU.entry(MSG_LIMIT_RESOURCE_NAME, EntryType.IN, 1, apiPath);  
            return  true;  
        } catch (BlockException e) {  
            log.warn("===========请求被限流==========");  
            return  false;  
        } finally {  
            if (entry != null) {  
                entry.exit();  
            }  
        }  
    }  
  
  
    private boolean isLimitApi(String apiPath) {  
        if (limitConfig.getLimitCountMap().containsKey(apiPath)) {  
            return true;  
        }  
        return false;  
    }  
  
  
}
```

### 限流配置加载

```java
  
import com.alibaba.csp.sentinel.slots.block.flow.param.ParamFlowRule;  
import com.alibaba.csp.sentinel.slots.block.flow.param.ParamFlowRuleManager;  
import com.alibaba.fastjson.JSON;  
import lombok.extern.slf4j.Slf4j;  
import org.springframework.stereotype.Component;  
import org.springframework.util.CollectionUtils;  
  
import javax.annotation.Resource;  
import java.util.List;  
import java.util.Map;  
import java.util.Objects;  
  
import static com.example.sentinel.limit.service.LimitService.MSG_LIMIT_RESOURCE_NAME;  
  
/**  
 * @Author: CLL02  
 * @Date: 2024/9/15 22:40  
 * @Version: v1.0  
 * @Description: TODO  
 **/  
@Component  
@Slf4j  
public class LimitManager {  
  
    private HotMsgLimitRule historyHotMsgLimitRule = null;  
  
    @Resource  
    private LimitConfig limitConfig;  
  
  
    /**  
     * 加载限流规则  
     */  
    public synchronized void reloadParamFlowRule() {  
        HotMsgLimitRule  hotMsgLimitRule = reloadParamFlowRuleInternal(MSG_LIMIT_RESOURCE_NAME,historyHotMsgLimitRule,limitConfig.getLimitCountMap(),limitConfig.getLimitDurationInSec());  
        if(hotMsgLimitRule != null){  
            historyHotMsgLimitRule = hotMsgLimitRule;  
        }  
    }  
  
  
  
  
    public  HotMsgLimitRule reloadParamFlowRuleInternal(String resourceName, HotMsgLimitRule historyHotMsgLimitRule, Map<String, Integer> msgTypePermitsMap, Long limitDurationInSec) {  
        List<ParamFlowRule> rules = ParamFlowRuleManager.getRules();  
  
        if (CollectionUtils.isEmpty(msgTypePermitsMap)) {  
            rules.removeIf(next -> Objects.equals(next.getResource(), resourceName));  
            ParamFlowRuleManager.loadRules(rules);  
            return null;  
        }  
  
        HotMsgLimitRule hotMsgLimitRule = new HotMsgLimitRule(resourceName,msgTypePermitsMap, limitDurationInSec);  
  
        if (hotMsgLimitRule.equals(historyHotMsgLimitRule)) {  
            return null;  
        }  
  
        rules.removeIf(next -> Objects.equals(next.getResource(), resourceName));  
        //组装限流规则  
        ParamFlowRule rule = hotMsgLimitRule.toParamFlowRule();  
        log.info("load Hot MsgType limiter rule:{}", JSON.toJSONString(rule));  
        rules.add(rule);  
        ParamFlowRuleManager.loadRules(rules);  
        return hotMsgLimitRule;  
    }  
}
```

```java
import lombok.Getter;  
import lombok.Setter;  
import org.springframework.boot.context.properties.ConfigurationProperties;  
import org.springframework.context.annotation.Configuration;  
  
import java.util.HashMap;  
import java.util.Map;  
import java.util.concurrent.TimeUnit;  
  
/**  
 * @Author: CLL02  
 * @Date: 2024/9/15 21:01  
 * @Version: v1.0  
 * @Description: TODO  
 **/  
@Configuration  
@Getter  
@Setter  
@ConfigurationProperties(prefix = "limit")  
public class LimitConfig {  
  
    //限流配置：key-限流的api或者其它事件  
    private Map<String,Integer> limitCountMap = new HashMap<>();  
  
    /**  
     * 限流的时间控制：默认一秒钟  
     */  
    private Long limitDurationInSec = TimeUnit.MINUTES.toSeconds(1);  
}
```


### 限流配置自动加载
```java
import lombok.extern.slf4j.Slf4j;  
import org.springframework.cloud.context.environment.EnvironmentChangeEvent;  
import org.springframework.context.ApplicationListener;  
  
import javax.annotation.Resource;  
  
/**  
 * @Author: CLL02  
 * @Date: 2024/9/15 21:22  
 * @Version: v1.0  
 * @Description: TODO  
 **/  
@Slf4j  
public class NacosListener implements ApplicationListener<EnvironmentChangeEvent> {  
  
  
    @Resource  
    private LimitManager limitManager;  
  
    public void init() {  
        loadParamFlowRule();  
    }  
  
    private void loadParamFlowRule() {  
        try {  
            limitManager.reloadParamFlowRule();  
        } catch (Exception e) {  
            log.error("重新加载限流规则配置失败", e);  
        }  
    }  
  
    @Override  
    public void onApplicationEvent(EnvironmentChangeEvent environmentChangeEvent) {  
        limitManager.reloadParamFlowRule();  
    }  
}
```

```java
import org.springframework.context.annotation.Bean;  
import org.springframework.context.annotation.Configuration;  
  
/**  
 * @Author: CLL02  
 * @Date: 2024/9/15 21:44  
 * @Version: v1.0  
 * @Description: TODO  
 **/  
@Configuration  
public class BeanConfig {  
    @Bean(initMethod = "init")  
    public NacosListener nacosPropertyChangeListener() {  
        return new NacosListener();  
    }  
}
```


### 测试功能

```java
import com.example.sentinel.limit.service.LimitService;  
import org.springframework.web.bind.annotation.PostMapping;  
import org.springframework.web.bind.annotation.RequestParam;  
import org.springframework.web.bind.annotation.RestController;  
  
import javax.annotation.Resource;  
  
/**  
 * @Author: CLL02  
 * @Date: 2024/9/15 21:27  
 * @Version: v1.0  
 * @Description: TODO  
 **/  
@RestController  
public class TestController {  
  
    @Resource  
    private LimitService limitService;  
  
  
    @PostMapping("/limit")  
    public void test(@RequestParam("param") String param) {  
        if (!limitService.isLimit(param)) {  
            System.out.println("=======================请求被限流=======================");  
            return;  
        }  
  
        System.out.println("=======================执行业务逻辑=======================");  
    }  
}
```


### 基础配置
```yaml
server:  
  port: 8083  
  
# 限流配置  
limit:  
  limit-count-map:  
    test: 2  
  limit-duration-in-sec: 1
```


### 效果展示
![Dingtalk_20240915225409.jpg](https://s2.loli.net/2024/09/15/NyD4HisxXl8opVW.jpg)

