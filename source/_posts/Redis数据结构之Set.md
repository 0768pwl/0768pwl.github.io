---
title: Redis数据结构之Set
date: 2018-09-11 15:52:22
tags: Redis
categories: Redis
---

​	集合（set）类型也是用来保存多个的字符串元素，但和列表类型不一样的是，集合中不允许有重复元素，并且集合中的元素是无序的，不能通过索引下标获取元素。Redis除了支持集合内的增删改查，同时还支持多个集合取交集、并集、差集，合理地使用好集合类型，能在实际开发中解决很多实际问题。

## 常用命令

###  集合内操作

#### 添加元素

```
指令： sadd key element [element ...]
示例： sadd myset java python；//向集合中添加元素java，python，返回结果为添加成功的元素个数
```
#### 删除元素

```
指令： srem key element [element ...]
示例： srem myset java；//删除集合中的元素java，返回结果为成功删除元素个数
```
#### 计算元素个数

```
指令： scard key
示例： scard myset；//返回集合myset中的元素个数
说明： scard的时间复杂度为O（1），它不会遍历集合所有元素，而是直接用Redis内部的变量
```
#### 判断元素是否在集合中

```
指令： sismember key element
示例： sismember myset python；//如果存在集合内返回1，反之返回0
```
#### 随机从集合返回指定个数元素

```
指令： srandmember key [count]
示例： srandmember myset 2；//从集合myset中随机返回两个元素
说明： [count]是可选参数，如果不写默认为1
```
#### 从集合随机弹出元素

```
指令： spop key
示例： spop myset；//从集合随机选择一个元素进行删除，返回结果为删除的元素
说明： srandmember和spop都是随机从集合选出元素，两者不同的是spop命令执行后，元素会从集合中删除，而srandmember不会
```
#### 获取所有元素

```
指令： smembers key
示例： smembers myset；//返回java，python，返回结果是无序的
说明： smembers属于比较重的命令，如果元素过多存在阻塞Redis的可能性，这时候可以使用sscan来完成
```
###  集合间操作

#### 多个集合的交集

```
指令： sinter key [key ...]
示例： sinter myset1 myset2；//求集合myset1与myset2的交集
```
#### 多个集合的并集

```
指令： suinon key [key ...]
示例： suinon myset1 myset2；//求集合myset1与myset2的并集
```
#### 多个集合的差集

```
指令： sdiff key [key ...]
示例： sdiff myset1 myset2；//返回一个集合的全部成员，该集合是第一个Key对应的集合和后面key对应的集合的差集
```
#### 将交集、并集、差集的结果保存

```
sinterstore destination key [key ...]
suionstore destination key [key ...]
sdiffstore destination key [key ...]
示例： sinterstore myset1_2 myset1 myset2；//将myset1和myset2两个集合的交集结果保存在myset1_2中，myset1_2本身也是集合类
说明：集合间的运算在元素较多的情况下会比较耗时，所以Redis提供了上面三个命令（原命令+store）将集合间交集、并集、差集的结果保存在destination key中
```
## 内部编码

​	集合类型的内部编码有两种，intset（整数集合）和 hashtable（哈希表）

​	intset（整数集合）：当集合中的元素都是整数且元素个数小于set-maxintset-entries配置（默认512个时，Redis会选用intset来作为集合的内部实现，从而减少内存的使用。

​	hashtable（哈希表）：当集合类型无法满足intset的条件时，Redis会使用hashtable作为集合的内部实现。
## 使用场景

### 社交共同好友

​	比如在微博应用中，可以将一个用户所有的关注人存在一个集合中，将其所有粉丝存在一个集合，利用 Redis 为集合提供的求交集、并集、差集等操作，那么就可以非常方便的实现如共同关注、共同喜好、二度好友等功能。