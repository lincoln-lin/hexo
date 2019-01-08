---
title: Redis 与 Mysql 保持一至性
date: 2018-11-27  16:30:43
tags:
      - Redis
      - Mysql
      - 一至性
description: Redis 与 Mysql 保持一至性的一般做法
---

### Redis 与 Mysql 保持一至性的一般做法

最近在魅族服务中发现部分redis与mysql 出现不一至的问题，排查到发现是网络抖动所致（可见阿里云的坑还是挺多的）。
以下是我们项目的伪代码：
```
 func Doupdate(){
    if mysqlUpdate() != false {
        for i:=0; i< 3; i++ {
            if CleanCache(key) {
                break
            }
        }
    }
 }
```
即先操作数据库，成功后更新缓存。如果更新缓存失败则会重试3次，值得更新成功为止。但当网络抖动时间较长时，3次都会失败，这样就导致了数据不致。

#### 解决方法： 使用 Kafka 队列
```
 func Doupdate(){
    if mysqlUpdate() != false {
        cleanCache := false
        for i:=0; i< 3; i++ {
            if CleanCache(key) {
                cleanCache = true
                break
            }
        }
        if !cleanCache {
            addMq(key,"CleanCache")  //更新3次都失败时增加一个消息，异步处理更新缓存
        }
    }
 }
 ```



    
  