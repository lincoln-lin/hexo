---
title: go-micro 实现服务熔断与服务降级
date: 2018-01-28 16:08:48
tags:
   - go-micro
   - hystrix
   - 服务熔断
---
### 服务熔断与服务降级

魅族服务的下单服务在上游，下单过程中需要调用flyme的部分服务，此时flyme的服务不可用或接口错误，
导致每个请求都是timeout才返回，原本2ms就能完成的业务，现在要3s才能完成（假设timeout时间是3s），
为了保护我们自己的魅族服务，采用熔断，并降级。

go-micro 为我们准备了 hystrix 的熔断器，直接开箱可用。并且WrapClient 可以在调用添加任意的中间件，我们可
以利用这点，把熔断器放在并且WrapClient 里面。

复制以下文件到我们的项目目录，以方便我们修改：
```
github.com/micro/go-plugins/wrapper/breaker/hystrix/hystrix.go

```
复制到 meizuservice/hystrix 下面，打开hystrix.go,添加一个方法：

```
func Configure(names []string) {
	// overwrite default configure
	config := hystrix.CommandConfig{
		Timeout:               3000, //毫秒
		MaxConcurrentRequests: 100, // 最大并发请求数
		ErrorPercentThreshold: 50, // 错误率，0 - 100
		SleepWindow: 5000 , //sleep 多久重新检测
		RequestVolumeThreshold: 1, //错误率计算最少数量
	}
	configs := make(map[string]hystrix.CommandConfig)
	for _, name := range names {
		configs[name] = config
	}
	hystrix.Configure(configs)
}
```
同时修改 func (c *clientWrapper) Call 方法，添加服务降级：
```
func (c *clientWrapper) Call(ctx context.Context, req client.Request, rsp interface{}, opts ...client.CallOption) error {
	return hystrix.Do(req.Service()+"."+req.Method(), func() error {
		return c.Client.Call(ctx, req, rsp, opts...)
	}, func(e error) error {
	    //以下是服务降级
		fmt.Println(e)
		return nil
	})
}
```
因为熔断是在客户端做的，那么我们在调用下游服务之前，要先添加这个熔断：
```
//配置你要调用的服务，go.micro.srv.example.Example.Call，名字要精确到方法名
hystrix.Configure([]string{"go.micro.srv.example.Example.Call"}) //配置熔断服务名称

	service := micro.NewService(
		micro.Name("go.micro.srv.example"),
		micro.Version("latest"),
		micro.Registry(reg),
		micro.WrapClient(hystrix.NewClientWrapper()), //添加熔断
	)
	service.Init()
```

已经可以了，测试如下：
我们故意在 go.micro.srv.example.Example.Call 这个方法里 sleep 2秒，模拟服务器繁忙，故意让他timeout
客户端如果不使用熔断降级，则会等2秒，然后输出结果，如果使用熔断之后，则立刻返回降级的内容，不会等待