---
title: go-micro 实现微服务限流器功能
date: 2018-11-28 15:57:02
tags:
    - go-micro
    - 限流器
    - Go微服务
    - juju/ratelimit
  
---
### go-micro 与 juju/ratelimit 实现微服务限流器功能
```
先安装 juju/ratelimit：
go get github.com/juju/ratelimi
```
在 go-micro 里面实现限流器功能非常简单,既可以在客户端也可以在服务端实现，这里我们选择在服务端实现：
```
import (
	"github.com/micro/go-plugins/wrapper/ratelimiter/ratelimit"
    ratelimit2 "github.com/juju/ratelimit"
)
func main() {
    	b := ratelimit2.NewBucketWithRate(1,1) //配置qps 和 容量
        service := micro.NewService(
		    micro.Name("go.micro.srv.example"),
		    micro.Version("latest"),
		    micro.Registry(reg),
		    micro.WrapClient(ratelimit.NewClientWrapper(b, false)),//加入限流器
	    )
}

```



