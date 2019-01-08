---
title: go-micro 与 kafka 实现消息队列
date: 2018-11-29 15:48:40
tags:
    - go-micro
    - kafka
    - Go微服务
---

### go-micro 与 kafka 实现消息队列
go-micro 作为一个微服务框架，很容易结合 kafka 实现消息队列。

#### 1，安装并配置zookeeper
```
brew install zookeeper
zkServer start
```
#### 2， 安装并配置kafka
```
brew install kafka
vim /usr/local/etc/kafka/server.properties
修改 listeners=PLAINTEXT://127.0.0.1:9092
然后启动 brew service start kafka

```

#### 3, 测试生产者与消费者

```
cd /usr/local/Cellar/kafka/2.0.0/bin
起动生产者：
./kafka-console-producer –broker-list 127.0.0.1:9092 –topic test
输入消息 ： aaa
启动消费者：
kafka-console-consumer –bootstrap-server 127.0.0.1:9092 –topic test –from-beginning

此时消费者收到消息 aaa，继续在生产者的命令行输入新命令，此时消费者会自动收到

说明kafka可以用了
```


#### 4，在go-micro框架使用kafka

```
import(
“github.com/micro/go-micro/broker”
“github.com/micro/go-plugins/broker/kafka”
)
micro.Broker(kafka.NewBroker(func(options *broker.Options) {
options.Addrs = []string{
“127.0.0.1:9092”,
}
})),
service.Init()
//生产者在Client端：
p := micro.NewPublisher(“test”, service.Client())
p.Publish(context.Background(), &example.Message{Say:”ss”})

//消费者：
import (
“lincolnrpc/example/subscriber”
)
micro.RegisterSubscriber(“test”, service.Server(), new(subscriber.Example))
处理逻辑在 subscriber中

收到的 msg 大概如下：

{“Header”:{“Content-Type”:”application/octet-stream”,”X-Micro-From-Service”:”go.micro.srv.example”},”Body”:”CgJzcw==”}

body 的内容 就是 Say:”ss” 里面的内容，使用 base64加密
```