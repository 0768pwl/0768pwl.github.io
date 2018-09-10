---
title: Redis数据结构之List
date: 2018-09-10 16:26:32
tags: Redis
categories: Redis
---

​	列表（list）类型是用来存储多个有序的字符串，列表中的每个字符串称为元素（element），一个列表最多可以存储232-1个元素。在Redis中，列表类型有两个特点，第一为列表中的元素是有序的，第二是列表中的元素是可以重复的，基于以上两个特点，我们可以对列表两端进行插入（push）和弹出（pop），还可以获取指定范围的元素列表、获取指定索引下标的元素等。总而言之，列表是一种比较灵活的数据结构，它可以充当栈和队列的角色，在实际开发上有很多应用场景。

## 常用命令

#### 1. 添加元素

```
//从右边插入元素
指令： rpush key value [value...]
举例： rpush names tony mike jaxi；//从右边插入key为names，value为tony mike jaxi的元素

//从左边插入元素
指令： lpush key value [value...]
举例： lpush names tony mike jaxi；//从左边插入key为names，value为tony mike jaxi的元素

//向某个元素前或者后插入元素
指令： linsert key before|after pivot value
举例： linsert names before mike nanxi；//从列表中找到value为mike的值，在前面插入value为nanxi的元素
```

#### 2. 查找元素

```
//获取指定范围内的元素列表
指令： lrange key start end
举例： lrange names 1 3；// lrange的end下标是有包含了自身，如该示例是获取第2到第4个元素
关于lrange指令的几点操作说明：
1. lrange的end下标是有包含了自身元素
2. 索引下标从左到右分别是0到N-1，从右到左分别是-1到-N

//获取列表指定索引下标的元素
指令： lindex key index
举例： lindex names -1；//获取当前列表最后一个元素为‘tony’

//获取列表长度
指令： llen key
举例： llen names；//返回key为names的列表长度为4
```
#### 3. 删除元素

```
//从列表左侧删除元素
指令： lpop key
举例： lpop names；//从左边删除列表names，如上示例删除的元素为‘jaxi’

//从列表右侧删除元素
指令： rpop key
举例： rpop names；//从右边删除列表names，如上示例删除的元素为‘tony’

//删除指定元素
指令： lrem key count value
举例： lrem names 1 nanxi；//删除元素为‘nanxi’
lrem命令会从列表中找到等于value的元素进行删除，根据count的不同分为三种情况：
1. count>0，从左到右，删除最多count个元素
2. count<0，从右到左，删除最多count绝对值个元素
3. count=0，删除所有元素

//按照索引范围修剪列表
指令： ltrim key start end
举例： ltrim names 1 3；//保留names列表的第二到第四个元素
```
#### 4. 修改元素

```
//修改指定索引下标的元素
指令： lset key index newValue
举例： lset names 0 houyi；//将names列表的第一个元素修改为houyi，下标规则同上
```
#### 5. 阻塞式命令

```
Redis为list列表类型提供了brpop与blpop两个阻塞式命令，blpop和brpop是lpop和rpop的阻塞版本，它们除了弹出方向不同，使用方法基本相同。

指令：blpop key [key ...] timeout 或 brpop key [key ...] timeout
其中key[key...]为多个列表的键，timeout为阻塞时间（单位：秒）

关于blpop和brpop的使用说明，这里分为两种情况：
第一，列表为空：如果timeout=3，那么客户端要等到3秒后返回，如果timeout=0，那么客户端一直阻塞等下去；
第二，列表不为空：客户端会立即返回

在使用brpop时，有两点要注意：
第一点，如果是多个键，那么brpop会从左至右遍历键，一旦有一个键能弹出元素，客户端立即返回；
第二点，如果多个客户端对同一个键执行brpop，那么最先执行brpop命令的客户端可以获取到弹出的值；
```
## 内部编码

​	Redis列表类型的内部编码有两种：
​	1.ziplist（压缩列表）：当列表的元素个数小于list-max-ziplist-entries配置（默认512个），同时列表中每个元素的值都小于list-max-ziplist-value配置时（默认64字节），Redis会选用ziplist来作为列表的内部实现来减少内存的使用；
	2.linkedlist（链表）：当列表类型无法满足ziplist的条件时，Redis会使用linkedlist作为列表的内部实现；
​	3.Redis3.2版本提供了quicklist内部编码，简单地说它是以一个ziplist为节点的linkedlist，它结合了ziplist和linkedlist两者的优势，为列表类型提供了一种更为优秀的内部编码实现；
## 使用场景

#### 1.消息队列

​	Redis提供的lpush+brpop或rpush+blpop可以实现延迟队列。

​	开发口诀：

​	lpush+lpop=Stack（栈）
​	lpush+rpop=Queue（队列）
​	lpsh+ltrim=Capped Collection（有限集合）
​	lpush+brpop=Message Queue（消息队列）



