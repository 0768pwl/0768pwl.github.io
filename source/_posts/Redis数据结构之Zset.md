---
title: Redis数据结构之Zset
date: 2018-09-12 10:44:16
tags: Redis
categories: Redis
---

​	有序集合（zset）与集合一样，元素都不能重复，但与集合不同的是，有序集合中的元素是有顺序的。与列表使用下标作为排序依据不同，有序集合为每个元素设置一个分数（score）作为排序的依据，在有序集合中，score是可以重复的。
## 常用命令

###  集合内操作

#### 添加元素

```
指令： zadd key score member [score member ...]
示例： zadd myzset 90 tony；//向有序集合myzset添加用户tony和他的分数90
说明： Redis在3.2版本为zadd命令增加了x、xx、ch、incr四个选项
1. nx：member必须不存在，才可以设置成功，用于添加
2. xx：member必须存在，才可以设置成功，用于更新
3. ch：返回此次操作后，有序集合元素和分数发生变化的个数
4. incr：对score做增加
```
#### 计算元素个数

```
指令： zcard key
示例： zcard myzset；//返回元素个数为1，与scard命令一样，zcard的时间复杂度为O（1）
```
#### 计算某个成员的分数

```
指令： zscore key member
示例： zscore myzset tony；//返回tony的分数90，如果成员不存在，返回nil
```
#### 计算成员的排名

```
指令：
zrank key member
zrevrank key member
说明：zrank是从分数从低到高返回排名，zrevrank反之，排名从0开始计算
```
#### 删除成员

```
指令： zrem key member [member ...]
示例： zrem myzset tony；//删除集合成员tony，返回删除成功个数
```
#### 增加成员的分数

```
指令： zincrby key increment member
示例： zincrby myzset 10 tony；//给集合成员的分数增加10分，变成100
```
#### 返回指定排名范围的成员

```
指令：
zrange key start end [withscores]
zrevrange key start end [withscores]
说明：有序集合是按照分值排名的，zrange是从低到高返回，zrevrange反之。果加上withscores选项，同时会返回成员的分数
```
#### 返回指定分数范围的成员

```
指令：
zrangebyscore key min max [withscores] [limit offset count]
zrevrangebyscore key max min [withscores] [limit offset count]
说明：zrangebyscore按照分数从低到高返回，zrevrangebyscore反之。withscores选项会同时返回每个成员的分数。[limit offset count]选项可以限制输出的起始位置和个数。
```
#### 返回指定分数范围成员个数

```
指令： zcount key min max
示例： zcount myzset 100 150；//返回100到150分的成员的个数
```
#### 删除指定排名内的升序元素

```
指令： zremrangebyrank key start end
示例： zremrangebyrank myzset 0 2；//删除myset第0到2排名的成员
```
#### 删除指定分数范围的成员

```
指令： zremrangebyscore key min max
示例： zremrangebyscore myzset (250 +inf；//将250分以上的成员全部删除
说明： min和max还支持开区间（小括号）和闭区间（中括号），-inf和+inf分别代表无限小和无限大
```
###  集合间操作
#### 多个集合的交集

```
zinterstore destination numkeys key [key ...] [weights weight [weight ...]] [aggregate sum|min|max]
```
#### 多个集合的并集

```
zunionstore destination numkeys key [key ...] [weights weight [weight ...]] [aggregate sum|min|max]
```
## 内部编码

​	有序集合类型的内部编码有两种，ziplist（压缩列表）和 skiplist（跳跃表）

​	ziplist（压缩列表）：当有序集合的元素个数小于zset-max-ziplistentries配置（默认128个），同时每个元素的值都小于zset-max-ziplist-value配置（默认64字节）时，Redis会用ziplist来作为有序集合的内部实现，ziplist可以有效减少内存的使用。

​	skiplist（跳跃表）：当ziplist条件不满足时，有序集合会使用skiplist作为内部实现，因为此时ziplist的读写效率会下降。
## 使用场景
#### 排行榜系统

​	视频网站需要对用户上传的视频做排行榜，榜单的维度可能是多个方面的：按照时间、按照播放数量、按照获得的赞数。