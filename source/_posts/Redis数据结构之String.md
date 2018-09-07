---
title: Redis数据结构之String
date: 2018-09-07 14:39:42
tags: Redis
categories: Redis
---

​	字符串类型是Redis最基础的数据结构。在Redis中，所有的键都是字符串类型，而且其它几种数据结构都是在字符串的基础上构建的，字符串类型的值实际可以是字符串（简单字符串、复杂字符串（例如JSON、XML））、数字（整数、浮点数），甚至是二进制（图片、音频），但是要注意的是**值最大不能超过512M**。

## 常用命令

#### 1. 设置值 	
```
指令： set key value [ex seconds] [px milliseconds] [nx|xx]
举例： set hello world；//设置键为hello，值为world的键值对，返回ok代表设置成功
其中set指令有几个选项，下面分别进行说明：
1. ex seconds：为键设置秒级过期时间
2. px milliseconds：为键设置毫秒级过期时间
3. nx：键必须不存在，才可以设置成功，用于添加
4. xx：与nx相反，键必须存在，才可以设置成功，用于更新
```

 除了set命令，Redis还为我们提供了setex和setnx命令，其作用是与上面的ex跟nx一样，命令格式如下：

```
setex key seconds value
setnx key value
```
#### 2. 获取值

```
指令： get key
举例： get hello；//获取key为hello的值，若key存在，返回相应的值，否则返回nil
```
#### 3. 批量设置值

```
指令： mset key value [key value ...]
举例： mset name1 boy name2 girl name3 unknow；//批量设置key-value，返回ok代表设置成功
```
#### 4. 批量获取值

```
指令： mget key [key ...]
举例： mget name1 name2 name4；//批量获取key对应的值，如果当前key不存在，返回nil，结果按传入键顺序返回
```

Redis提供了批量操作命令可以有效提高开发效率，加入没有mget命令，获取n个key的值需要执行n次get命令，其中需要n次网络传输时间 + n次get命令执行时间，而有了mget命令，只需要1次网络传输时间 + n次get命令。但是要注意的是**每次批量获取的key数要适当**，不然由于Redis的单线程架构，键数太多容易造成Redis阻塞。
#### 5. 计数

```
指令： incr key
举例： incr num；//对值做自增操作,返回自增后的值
其中incr命令分为以下几种情形：
1. 值不是整数，返回错误
2. 值是整数，返回自增后的结果
3. 键不存在，按照值为0自增，返回结果为1
```

除了incr命令，Redis还提供了decr（自减）、incrby（自增指定数字）、decrby（自减指定数字）、incrbyfloat（自增浮点数），命令格式如下：

```
decr key
incrby key increment
decrby key decrement
incrbyfloat key increment
```
## 内部编码

​	Redis字符串类型的内部编码有3种，分别是int、embstr、raw，Redis会根据当前值的类型和长度决定使用哪种内部编码实现，可以使用`object encoding key`指令进行验证，这几种内部编码的使用情况分别如下：

	1. int：8个字节的长整型
	2. embstr：小于等于39个字节的字符串
	3. raw：大于39个字节的字符串
## 使用场景

#### 1.缓存信息载体

​	上面介绍时说过，Redis字符串的值可以为简单字符串，也可以为复杂字符串（JSON，XML），甚至是Binary，这也意味着我们可以将Java对象序列为Json或Binary后进行保存。

​	在这里，了解过Redis的hash数据结构的童鞋应该知道，hash也常被用于缓存信息载体，那么在这里到底是要采用string还是hash进行保存信息载体呢？

个人认为这取决于你对数据访问的访问方式，分为以下两种情况进行讨论：
1.string：
（1）在多数情况下，你需要用到的对象字段较多，可以考虑采用string；
2.hash：
（1）在多数情况下，你需要用到的对象字段较少，可以考虑采用hash；
（2）在多数情况下，总是知道哪些字段是要用的，可以考虑采用hash；

#### 2.用于计数器

​	如常见的视频网站播放量，论坛帖子的点击量，点赞人数等等。

#### 3.共享session

​	在网站的集群加负载均衡场景下，可用于做session服务器，统一管理session。

#### 4.用于操作限制

​	用于对网站接口的访问次数限制，如网站的短信接口不被频繁访问，会限制用户每分钟获取验证码的频率，例如一分钟不能超过5次。