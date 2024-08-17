# 处理生产环境的MySQL死锁问题

作者： 陈汤姆
<br/>博客：[https://github.com/linglongchen/linglongchen.github.io](https://github.com/linglongchen/linglongchen.github.io)

>自我学习的积累！😄


## 1. 前言

在生产环境中发现我们数据库出现了一个异常，异常堆栈信息如下：

```sql
Error updating database. 
 Cause: com.mysql.jdbc.exceptions.jdbc4.MySQLTransactionRollbackException: 
Deadlock found when trying to get lock; try restarting transaction\n### The error may involve 
xxxMapper.updateByExampleSelective-Inline\n### The error occurred while setting parameters\n### 
SQL: UPDATE xxx  SET to_recipient_id = ?,logistics_order_id = ?,delivery_type = ?,delivery_no = ?,
delivery_company_code = ?,delivery_big_pen = ?,delivery_package_site_name = ?,delivery_extended_attribute = ?,
delivery_label_url = ?,customs_channel_code = ?,declare_no = ?,create_time = ?,update_time = ?,delete_flag = ?,
channel_hawb_code = ?,system_order_code = ?,change_sign = ?,delivery_extended_no = ?,delivery_child_no = ? 
WHERE (       (  logistics_order_id = ? ) )
```

从堆栈信息可以很容易知道死锁问题。但是这个更新语句为什么会出现死锁呢？

## 2. 寻找问题原因

死锁产生的原因有四个分别是：

-   互斥
-   循环等待
-   不可剥夺
-   请求与保持

只要产生死锁以上四个条件比然满足，因此考虑这个SQL语句是否产生了这四个死锁条件。

分析：

由于我们使用的是云数据库，因此可以通过云数据库控制台查看锁分析，分析结果如下：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/475c1058c3ab4d6bb5ef8618fdbdaa5d~tplv-k3u1fbpfcp-zoom-1.image)

可以看到死锁的产生是由于两个事务互相竞争导致的，那么两个事务如何产生死锁呢？

两个事务产生死锁的条件如下：

事务1: lock A, then B\
事务2: lock B, then A

翻译一下就是：

事务1

update table 1 set name = 1 where id = 1;

update table2 set age = 2 where id = 3;

事务2

update table2 set age = 2 where id = 3;

update table 1 set name = 1 where id = 1;

即两个事务中，T1 锁定了A，要去获取B的资源锁，但是T2已经锁定了资源B，T2要去获取A的锁，两个都不释放，从而导致死锁。

根据这种场景分析生产执行SQL找到了对应的SQL问题，问题的原因也是前面描述的一样，两个事务互相竞争等待导致的。

## 3. 解决方案

那么针对这种情况如何解决呢？

### 方案1：两个事务的执行SQL改成一样

事务1

update table 1 set name = 1 where id = 1;

update table2 set age = 2 where id = 3;

事务2

update table 1 set name = 1 where id = 1;

update table2 set age = 2 where id = 3;

按照相同的顺序执行SQL，即使出现并发情况，那么行锁也会等待而不会死锁。

### 方案2：抽取公共sql进行复用

将两个更新sql抽取成共用的方法，同时修改两个调用方，保证两个事务执行的是相同的事务。

update table 1 set name = 1 where id = 1;

update table2 set age = 2 where id = 3;