# 为什么要用last\_insert\_id来生成id序列号？

## 直接这生成序列号：

```
update batch_job_seq set id = id + 1;
...
select id from batch_job_seq;
```

## **那么问题来了：**

并发环境下，update后，取id还要用select查一下数据库，此时id可能已经变了，自己生成的id没取到，取了别人生成的id.

## **怎么解决问题：**

last\_insert\_id是被设计成基于connection存储的，不同connection是隔离的，互不影响，在update和select之间如果有其connection修改了last\_insert\_id，select last\_insert\_id\(\)还是会取到自己生成的id，不会取到其connection生成的id.

```
update batch_job_seq set id = last_insert_id( id + 1 );
select last_insert_id() from batch_job_seq;
```

## 更进一步解决问题：

但是如果同一个connection下分出多个线程同时使用update+select也会导致本线程id没取到，取了其他线程生成的id。所以需要给update+select**放在一个方法中用synchronize屏蔽多线程同时用一个connection导致的问题**,参考代码：

### MySQLMaxValueIncrementer.java

```java
 

```

