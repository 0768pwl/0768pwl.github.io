---
title: Redis数据结构之Hash
date: 2018-09-11 11:29:57
tags: Redis
categories: Redis
---

​	哈希类型（hash）是指键值本身又是一个键值对结构，在哈希类型中，映射关系叫作field-value，注意这里的value是指field对应的值，不是键对应的值。

## 常用命令

###  设置值

``` 
指令： hset key field value
示例： hset user:1 name tony；//将哈希表Key为user:1中的域name的值设为tony
说明： 
1.将哈希表Key中的域field的值设为value 
2.key不存在，一个新的Hash表被创建
3.field已经存在，旧的值被覆盖

Redis还提供了hsetnx命令，，它们的关系就像set和setnx命令一样，只不过作用域由键变为field
```
###  获取值

``` 
指令： hget key field
示例： hget user:1 name；//返回tony
说明：
1.返回哈希表key中给定域field的值
2.如果键或field不存在，会返回nil
```
###  删除field

```
指令： hdel key field [field ...]
示例： hdel user:1 name；//删除key为user:1 filed为name的值
说明： hdel会删除一个或多个field，返回结果为成功删除field的个数
```
###  计算field个数

```
指令： hlen key
示例： hlen user:1；//返回1
```
###  批量设置或获取field-value

```
//批量设置field-value
指令： hmset key field value [field value ...]
示例： hmset user:1 name tony age 28

//批量获取filed-value
指令： hmget key field [field ...]
示例： hmget user:1 name age；//返回tony，28
```
###  判断field是否存在

```
指令： hexists key field
示例： hexists user:1 name；//如果field存在，返回结果为1，否则返回结果0
```
###  获取所有field

```
指令： hkeys key
示例： hkeys user:1；//返回name，age
```
###  获取所有value

```
指令： hvals key
示例： hvals user:1；//返回tony，28
```
###  获取所有的field-value

```
指令： hgetall key
示例： hgetall user:1；//返回name tony age 28
说明： 在使用hgetall命令时，如果当前hash集合的个数比较多，有可能会造成Redis阻塞，如果只是获取部分filed-value，建议使用hmget，如果一定要获取全部field-value，建议使用hscan命令，该命令会渐进式遍历哈希类型。
```
### 计数

```
指令： hincrby key field increment
示例： hincrby user:1 age 8；//为哈希表user:1中的域age加上增量8

指令： hincrbyfloat key field increment
示例： hincrbyfloat user:1 age 10.8；//为哈希表user:1中的域age加上增量10.8
```
## 内部编码

​	哈希类型的内部编码有两种，分别是ziplist（压缩列表）和 hashtable（哈希表）。

​	ziplist（压缩列表）：当哈希类型元素个数小于hash-max-ziplist-entries配置（默认512个）、同时所有值都小于hash-max-ziplist-value配置（默认64字节）时，Redis会使用ziplist作为哈希的内部实现，ziplist使用更加紧凑的结构实现多个元素的连续存储，所以在节省内存方面比hashtable更加优秀。

​	hashtable（哈希表）：当哈希类型无法满足ziplist的条件时，Redis会使用hashtable作为哈希的内部实现，因为此时ziplist的读写效率会下降，而hashtable的读写时间复杂度为O（1）。

## 使用场景

### 缓存信息载体

​	Redis的哈希类型也常被用与缓存对象信息，相比使用字符串序列化缓存用户信息，哈希类型更直观，且在更新操作上会更加方便（省去序列化与反序列化造成的开销）。