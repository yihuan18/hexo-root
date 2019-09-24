---
title: 谈谈Mybatis缓存
typora-root-url: ../../source/
date: 2019-09-24 11:18:36
tags:
- java
- 面试
- mybatis
---

Mybatis用了那么久，知道Mytabis里面有缓存吗

<!--more-->

# Mybatis缓存

Mybatis 中有一级缓存和二级缓存，缓存机制如下图。

这里有一篇美团技术团队关于[mytabis缓存机制](https://tech.meituan.com/2018/01/19/mybatis-cache.html)的介绍

![image-20190924125451387](/imgs/image-20190924125451387.png)

# 一级缓存

## 一级缓存介绍

一级缓存是指 SqlSession 级别的缓存，当在同一个 SqlSession 中进行相同的 SQL 语句查询时，第二次以
后的查询不会从数据库查询，而是直接从缓存中获取，一级缓存最多缓存 1024 条 SQL。

第一次发出一个查询 sql，sql 查询结果写入 sqlsession 的一级缓存中，缓存使用的数据结构是一 个 map。
```bash
key:MapperID + offset + limit + Sql + 所有的入参 
value:用户信息 
```
同一个 sqlsession 再次发出相同的 sql，就从缓存中取出数据。如果两次中间出现 commit 操作 (修改、添加、删除)，本 sqlsession 中的一级缓存区域全部清空，下次再去缓存中查询不到所 以要从数据库查询，从数据库查询到再写入缓存。 

![img](/imgs/6e38df6a.jpg)

## 一级缓存配置

我们来看看如何使用MyBatis一级缓存。开发者只需在MyBatis的配置文件中，添加如下语句，就可以使用一级缓存。共有两个选项，`SESSION`或者`STATEMENT`，默认是`SESSION`级别，即在一个MyBatis会话中执行的所有语句，都会共享这一个缓存。一种是`STATEMENT`级别，可以理解为缓存只对当前执行的这一个`Statement`有效。

```xml
<setting name="localCacheScope" value="SESSION"/>
```

## 一级缓存总结

1. MyBatis一级缓存的生命周期和SqlSession一致。
2. MyBatis一级缓存内部设计简单，只是一个没有容量限定的HashMap，在缓存的功能性上有所欠缺。
3. MyBatis的一级缓存最大范围是SqlSession内部，有多个SqlSession或者分布式的环境下，数据库写操作会引起脏数据，建议设定缓存级别为Statement。

# 二级缓存

## 二级缓存介绍

二级缓存的范围是 mapper 级别(mapper 同一个命名空间)，mapper 以命名空间为单位创建缓存数据结构，结构是 map。mybatis 的二级缓存是通过 CacheExecutor 实现的。CacheExecutor 其实是 Executor 的代理对象。所有的查询操作，在 CacheExecutor 中都会先匹配缓存中是否存 在，不存在则查询数据库。

```bash
key:Statement Id + offset + limit + Sql + 所有的入参 
```
![img](/imgs/28399eba.png)

## 二级缓存配置

具体使用需要配置: 

1. Mybatis 全局配置中启用二级缓存配置
```xml
<setting name="cacheEnabled" value="true"/>
```
2. 在对应的 Mapper.xml 中配置 cache 节点 
cache标签用于声明这个namespace使用二级缓存，并且可以自定义配置。
```xml
<cache/>
```
>type：cache使用的类型，默认是PerpetualCache，这在一级缓存中提到过。
>eviction： 定义回收的策略，常见的有FIFO，LRU。
>flushInterval： 配置一定时间自动刷新缓存，单位是毫秒。
>size： 最多缓存对象的个数。
>readOnly： 是否只读，若配置可读写，则需要对应的实体类能够序列化。
>blocking： 若缓存中找不到对应的key，是否会一直blocking，直到有对应的数据进入缓存。

cache-ref代表引用别的命名空间的Cache配置，两个命名空间的操作使用的是同一个Cache。
```xml
<cache-ref namespace="mapper.StudentMapper"/>
```
3. 在对应的 select 查询节点中添加 useCache=true 

## 二级缓存总结

1. MyBatis的二级缓存相对于一级缓存来说，实现了`SqlSession`之间缓存数据的共享，同时粒度更加的细，能够到`namespace`级别，通过Cache接口实现类不同的组合，对Cache的可控性也更强。
2. MyBatis在多表查询时，极大可能会出现脏数据，有设计上的缺陷，安全使用二级缓存的条件比较苛刻。
3. 在分布式环境下，由于默认的MyBatis Cache实现都是基于本地的，分布式环境下必然会出现读取到脏数据，需要使用集中式缓存将MyBatis的Cache接口实现，有一定的开发成本，直接使用Redis、Memcached等分布式缓存可能成本更低，安全性也更高。