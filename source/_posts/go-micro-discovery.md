---
title: go-micro 使用 consul 、 etcd 、k8s 等作为服务发现
tags:
      - go-micro
      - consul
      - etcd
      - 服务发现
      - Go微服务
date: 2018-01-27 15:40:34
---

### go-micro 使用 consul 作服务发现

go-micro 默认使用 consul 作服务发现,所以不用修改代码,直接在服务启动时添加consul服务器即可 :
```

go run main.go --registry_address 172.16.196.132:8500

```

当然前提是consul允许公网访问:

```
consul agent -dev  -http-port 8500 -client 0.0.0.0

```
### go-micro 使用 etcd 作服务发现

使用 etcdv3 作服务发现, 安装: mac :brew install etcd, //centos : yum install etcd
然后直接执行 : etcd 即可启动.
导入 etcdv3需要安装库 : go get go.etcd.io/etcd/clientv3

```
import (
    "github.com/micro/go-plugins/registry/etcdv3" 
)

func main() {
	// New Service
	
	reg := etcdv3.NewRegistry(func(op *registry.Options){ 
		op.Addrs = []string{
			"http://172.16.196.132:2379",
			"http://172.16.196.133:2379",
			"http://172.16.196.134:2379",  //三台etcd
		}

	})

	service := micro.NewService(
		micro.Name("go.micro.srv.example"),
		micro.Version("latest"),
		micro.Registry(reg), // 服务发现
	)
```

### go-micro 使用 k8s 作服务发现

还支持k8s作为服务发现,以上代码改为:

```
reg := Kubernetes.NewRegistry()
micro.Registry(reg), // 服务发现
```
