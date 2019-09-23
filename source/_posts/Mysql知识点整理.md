---
title: Mysql知识点整理
date: 2019-09-10 17:27:45
tags:
- 面试
- mysql
- java
---
# MySQL
主要整理一下在面试中mysql常被问到到基础性知识

<!--more-->

## 存储引擎
不同的存储内容的技术就是存储引擎

### InnoDB
- 适合处理多重并发更新请求
- 支持事务
- 自动灾难恢复
- 外键约束
- AUTO_INCREMENT等功能
- 适用于写密集表

### MyISAM
- 不支持事务
- 不支持外键（父子表关联）
- 适用于读密集表

### MEMORY
- 使用系统内存存储，崩溃后会丢失
- 不可用可变数据类型
- 适用于小数据、临时数据、可丢失数据

### InnoDB和MyISAM区别
- InnoDB支持事务
- InnoDB支持在线热备份；MyISAM崩溃后损坏概率比InnoDB高，恢复速度也慢
- MyISAM支持表级锁，不支持行级锁
- MyISAM支持全文索引、地理空间索引
- MYISAM不支持外键

## 索引
### 问题
MySQL的基本存储结构是页，各个数据页组成一个双向链表，数据页中的记录组成单向链表；检索需要遍历双向链表

#### 提高检索速度
无序数据变有序：使用BTree索引或Hash索引
维持平衡树需要额外开销，所以降低了增删改的效率

### 哈希索引与BTree索引的区别
- 哈希索引不能利用索引排序
- 哈希索引不支持最左匹配原则
- 有大量重复键值情况下哈希索引效率很低
- 不支持范围查询

### 聚集索引与非聚集索引
聚集索引是以主键创建的索引，非聚集索引是以非主键创建的索引
- 聚集索引在叶子节点存储表中数据，非聚集索引存储的是主键和索引列（即非聚集索引需要表明是索引的哪一列）
- 非聚集索引查到数据，再去主键查找数据

### 最左匹配原则
索引从左到右依次匹配直到遇到范围查询或某个匹配没有
比如(a,b,c,d)，查询a=1 and b=2 and c>3 and d=4,则只能匹配a、b、c，因为c是范围查询，不能匹配d

### = in自动优化顺序
不需要考虑=和in的顺序，mysql自动优化
如(a,b,c,d)索引，查询c=3,b=2,a=1,d=0依然可以使用索引

### 索引的使用
```sql
//1、创建数据表时使用主键
CREATE TABLE table(
	id INT NOT NULL AUTO_INCREMENT
);
//2、ALTER TABLE
//其中column_list指出对哪些列进行索引
ALTER TABLE table ADD INDEX index_name (column_list)
//用于全文索引
ALTER TABLE table_name ADD FULLTEXT index_name(olumu_name)
//例子，添加对商品分类的索引
ALTER TABLE table ADD INDEX classify_index(Classify_Description)‘
//3、CREATE INDEX
CREATE INDEX index_name ON table (column_list)
```

```sql
//删除索引
DROP INDEX [indexName] ON table
ALTER TABLE [table_name] drop index [index_name]
```

```sql
SHOW INDEX FROM [table_name]
```
### 索引分类
**普通索引INDEX**
```sql
//VARCHAR等需要指定索引长度
INDEX index_name(VARCHAR(20))
```
**唯一索引UNIQUE：必须唯一，允许为空**
**主键索引：必须唯一，不允许为空**
**全文索引：仅可用于MyISAM**
```sql
//对content这一列进行全文索引
SELECT * FROM article WHERE MATCH( content) AGAINST('想查询的字符串')
```
### 总结
- 最左匹配原则：一直向右匹配直到范围查询
- 选择区分度高的作为索引
- 索引列不能参与计算
- 尽可能扩展索引而不是新建索引
- 索引选择限制最严格的一个匹配

## B树与B+树
- 根节点孩子数>=2
- 每个中间节点有k-1个元素和k个孩子（ceil(m/2)=<k<=m）
- 每个叶子节点有k-1个元素
- 叶子节点位于同一层
![](https://img-blog.csdn.net/2018032420113990?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pfcnlhbg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
B+树
中间元素只做索引，叶子节点存储数据（有序链表）

![](https://img-blog.csdn.net/20180325001555181?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pfcnlhbg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

## 锁
### 事务的四大特性

- 原子性：事务所有操作，要么全部完成，要么全部不完成
- 一致性：事务执行前执行后都必须处于一致性状态，比如A给B转账，总数必须不变
- 隔离性：数据库允许多个并发事务同时进行读写和修改的能力
- 持久化：事务处理结束后，数据的修改是永久的

### 表锁

分为表读锁和表写锁，读读不阻塞

写锁优先于读锁

### 行锁

分为共享锁和排他锁

- 共享锁（读）：允许一个事务读一行，阻止其他事务获得相同数据的排他锁
- 排他锁（写）：允许排他锁的事务更新数据，阻止其他事务获得共享锁和排他锁

为了允许行锁表锁并存，InnoDB存在两种意向锁：

- 意向共享锁：事务加行共享锁之前先获得意向共享锁
- 意向排他锁：事务加行排他锁之前先获得意向排他锁

### 两者区别

- 表锁开销小，加锁快，不会出现死锁，锁定粒度大，发生冲突概率高，并发度低
- 行锁开销大，加锁慢，会出现死锁，锁定粒度小，发生冲突概率低，并发度高

### MVCC和事务隔离级别

InnoDB基于行锁，实现了mVCC多版本并发控制，在多种隔离级别下实现读写不阻塞

脏读：读取到另一个事务未提交的数据；把读放在事务提交之后即可避免

不可重复读：多次读结果不同，数量相同，但被修改了

幻读：多次读结果数量不同，读的多了，读的少了

事务隔离级别：

- Read uncommited：出现脏读、不可重复读、幻读
- Read commited：会出现不可重复读、幻读
- Repeatable read：出现幻读
- Serializable：都不会出现

## 分库分表

分表：将一张表分厂多个小表，用于不同的表名

分区：一个表分成多个区块，表只有一个

使用分库分表：数据库中表的数据量越来越大，读写开销大

### 垂直切分（切分表项）

按表功能模块、关系密切成都划分部署到不同库上

### 水平切分（切分列项）

表中数据按某种规则存储到多个结构相同的表中，如按id散列、性别进行划分

表太多且项目逻辑清晰进行垂直切分，单表数据量大进行水平切分

### 存在问题

- 事务问题，数据存储到不同库上，事务管理出现困难
- 跨库跨表连接问题，表的连接操作受限，无法连接不同分库的表，原本只需要一次查询完成的业务需要多次

## 表的优化

- 限定数据范围，比如查询历史订单限制一个月
- 读写分离：主库负责写，从库负责读
- 垂直分区：根据数据表相关性拆分，使得行数据减小，但是主键冗余、需要管理冗余列，事务变得复杂
- 水平分区：每部分数据拆分成多个表，带来逻辑、运维的麻烦

## 数据三大范式

- 关系模式R的所有属性不能分解为更基本的数据单位（确保每一列的原子性，合并属性相同的列，避免产生冗余）
- 关系模式满足第一范式，且R所有非主属性都完全依赖于R的每一个候选关键属性（一行数据只做一件事，如果数据重复将表拆开）
- 关系模式满足第一范式，X是R任意属性集，X非传递依赖于R的任意一个候选关键词（数据不能存在传递关系，而是直接关系，如所在院校决定了院校电话，那么学号和所在院校建立表，所在院校和院校电话建立表）

## 慢查询

开启慢查询后，MySQL会记录下查询超过指定时间的SQL语句，方便调优。

对应的配置项为:slow_query_log

## 批处理

使用Batch进行批处理，同时执行多条同样的SQL操作

```java
PreparedStatement statement=Connection.prepareStatement("sql语句");
statement.setInt(1,int_value);
statement.addBatch();
cnt++;
if(cnt%1000==0){
	statement.executeBatch()
	statment.clearBatch();
}
statment.close();
```