---
title: Redis 分布式锁
date: 2018-11-26 12:15:35
tags:
      - Redis
      - 分布式锁    
---

### Redis 分布式锁的实现 （2.6.12 版本之前）

在魅族服务很多地方都需要用到分布式锁，我们一般使用redis来实现，在 2.61版本之前，我们一般使用SETNX、GETSET、毫秒时间戳 来实现，
用 Golang 代码实现如下：
```
    key := "MyCacheKey"
    for {
        timestamp := time.Now().UnixNano() / 1e6 //毫秒
        lock := rds.SETNX(key,timestamp + 10) //setnx不存在key时返回1,否则设置后10毫秒的时间
        if lock == 1 || ( timestamp > rds.GET(key) &&  timestamp > rds.GETSET(key,timestamp)  ){
            break
        }else{
            time.Sleep(10 ms) //10毫秒后再次获取锁
        }
    }
    //获得锁后再更新DB
    updateDB() 
    
    //数据库执行完后，要解开锁资源
    timestamp = time.Now().UnixNano() / 1e6 //毫秒
    if timestamp < rds.GET(key) {
        rds.DEL(key)
    }
    
```
### Redis 分布式锁的实现 （2.6.12 版本之后）
    
升级Redis到最新，此时Redis增加了Set命令的可选参数，直接用set命令即可完成加锁了

    key := "MyCacheKey"
    value := "value"
    for {
        //设置key,value,过期时间为10毫秒，且当key不存在时才会设置，这是一个原子操作
        lock := rds.SET(key,value,"PX",10,"NX")     
        if lock == 1 {
            break
        }else {
            time.Sleep (10 ms) //10毫秒后再次获取锁
        }
    }
    updateDB() 
    // 10毫秒后自动会解锁