---
title: Etcd启用https服务
date: 2018-03-28 00:00:00
tags: ["HTTPS"]
abbrlink: etcd-use-https-services
img: ""
comments: false
---

kubernetes依靠etcd来存储docker集群的配置信息，进一步说etcd作为一个受到ZooKeeper与doozer启发而催生的项目，除了拥有与之类似的功能外，更专注于以下四点：

- 简单：基于HTTP+JSON的API让你用curl就可以轻松使用
- 安全：可选SSL客户认证机制
- 快速：每个实例每秒支持一千次写操作
- 可信：使用Raft算法充分实现了分布式
由于在分布式环境中，安全性已经成为互联网技术团队非常看重的部分，所以本文主要介绍如何搭建一个支持证书的https Etcd集群。



## 生成证书
下载 cfssl
```
$ curl -s -L -o /usr/bin/cfssl https://pkg.cfssl.org/R1.2/cfssl_linux-amd64
$ curl -s -L -o /usr/bin/cfssljson https://pkg.cfssl.org/R1.2/cfssljson_linux-amd64

$ chmod +x /usr/bin/{cfssl,cfssljson}
```
## 生成key
先编写相关配置
```
$ vim ca-config.json

{
  "signing": {
    "default": {
      "expiry": "100000h"
    },
    "profiles": {
      "server": {
        "usages": ["signing", "key encipherment", "server auth", "client auth"],
        "expiry": "100000h"
      },
      "client": {
        "usages": ["signing", "key encipherment", "server auth", "client auth"],
        "expiry": "8760h"
      }
    }
  }
}
```

```
$ vim ca-csr.json

{
  "CN": "Etcd",
  "key": {
    "algo": "rsa",
    "size": 4096
  },
  "names": [
    {
      "C": "CN",
      "L": "Guangzhou",
      "O": "Gopher",
      "OU": "PaaS",
      "ST": "Guangzhou"
    }
  ]
}
```
编写完上面的配置文件后，我们可以开始使用cfssl来生成ca证书
```
$ cfssl gencert -initca ca-csr.json | cfssljson -bare ca
```
执行后，会生成三个文件：

- ca-key.pem - private key for the CA
- ca.pem - certificate for the CA
- ca.csr - certificate signing request for the CA

## 生成服务端证书
编写配置文件
```
$ vim server-csr.json

{
  "CN": "etcd-server",
  "hosts": [
    "localhost",
    "0.0.0.0",
    "127.0.0.1",
    "192.168.139.1",
    "192.168.139.134"
  ],
  "key": {
    "algo": "rsa",
    "size": 4096
  },
  "names": [
    {
      "C": "CN",
      "L": "Guangzhou",
      "O": "Gopher",
      "OU": "PaaS",
      "ST": "Guangzhou"
    }
  ]
}
```
导出证书
```
cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=server server-csr.json | cfssljson -bare server
```
注意：hosts需要包括允许访问ETCD Cluster的IP或者FQDN

生成三个文件：
- server.pem
- server-key.pem
- server.csr

产生客户端证书
```
$ vim client-csr.json

{
  "CN": "etcd-client",
  "hosts": [
    ""
  ],
  "key": {
    "algo": "rsa",
    "size": 4096
  },
  "names": [
    {
      "C": "CN",
      "L": "Guangzhou",
      "ST": "Guangzhou",
      "O": "Gopher",
      "OU": "PaaS"
    }
  ]
}
```
执行命令生成证书
```
cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=client client-csr.json | cfssljson -bare client
```
生成三个文件：
- client.pem
- client-key.pem
- client.csr

## 配置etcd 集群
下面会以一个五个节点的集群为例

## 节点启动脚本
```
./etcd --name cn0 \
--initial-advertise-peer-urls https://127.0.0.1:1080 \
--listen-peer-urls https://127.0.0.1:1080 \
--listen-client-urls https://$NODE:1079,https://127.0.0.1:1079 \
--advertise-client-urls https://$NODE:1079,https://127.0.0.1:1079 \
--initial-cluster-token etcd-cluster-token \
--initial-cluster cn0=https://127.0.0.1:1080,cn1=https://127.0.0.1:1180,cn2=https://127.0.0.1:1280,cn3=https://127.0.0.1:1380,cn4=https://127.0.0.1:1480 \
--initial-cluster-state new \
--cert-file=./server.pem \
--key-file=./server-key.pem \
--peer-cert-file=./server.pem \
--peer-key-file=./server-key.pem \
--trusted-ca-file=./ca.pem \
--peer-trusted-ca-file=./ca.pem 
--data-dir=./cn0 \
--peer-client-cert-auth=true \
--client-cert-auth=true

./etcd --name cn1 \
--initial-advertise-peer-urls https://127.0.0.1:1180 \
--listen-peer-urls https://127.0.0.1:1180 \
--listen-client-urls https://$NODE:1179,https://127.0.0.1:1179 \
--advertise-client-urls https://$NODE:1179,https://127.0.0.1:1179 \
--initial-cluster-token etcd-cluster-token \
--initial-cluster cn0=https://127.0.0.1:1080,cn1=https://127.0.0.1:1180,cn2=https://127.0.0.1:1280,cn3=https://127.0.0.1:1380,cn4=https://127.0.0.1:1480 \
--initial-cluster-state new \
--cert-file=./server.pem \
--key-file=./server-key.pem \
--peer-cert-file=./server.pem \
--peer-key-file=./server-key.pem \
--trusted-ca-file=./ca.pem \
--peer-trusted-ca-file=./ca.pem 
--data-dir=./cn1 \
--peer-client-cert-auth=true \
--client-cert-auth=true

./etcd --name cn2 \
--initial-advertise-peer-urls https://127.0.0.1:1280 \
--listen-peer-urls https://127.0.0.1:1280 \
--listen-client-urls https://$NODE:1279,https://127.0.0.1:1279 \
--advertise-client-urls https://$NODE:1279,https://127.0.0.1:1279 \
--initial-cluster-token etcd-cluster-token \
--initial-cluster cn0=https://127.0.0.1:1080,cn1=https://127.0.0.1:1180,cn2=https://127.0.0.1:1280,cn3=https://127.0.0.1:1380,cn4=https://127.0.0.1:1480 \
--initial-cluster-state new \
--cert-file=./server.pem \
--key-file=./server-key.pem \
--peer-cert-file=./server.pem \
--peer-key-file=./server-key.pem \
--trusted-ca-file=./ca.pem \
--peer-trusted-ca-file=./ca.pem  \
--data-dir=./cn2 \
--peer-client-cert-auth=true \
--client-cert-auth=true

./etcd --name cn3 \
--initial-advertise-peer-urls https://127.0.0.1:1380 \
--listen-peer-urls https://127.0.0.1:1380 \
--listen-client-urls https://$NODE:1379,https://127.0.0.1:1379 \
--advertise-client-urls https://$NODE:1379,https://127.0.0.1:1379 \
--initial-cluster-token etcd-cluster-token \
--initial-cluster cn0=https://127.0.0.1:1080,cn1=https://127.0.0.1:1180,cn2=https://127.0.0.1:1280,cn3=https://127.0.0.1:1380,cn4=https://127.0.0.1:1480 \
--initial-cluster-state new \
--cert-file=./server.pem \
--key-file=./server-key.pem \
--peer-cert-file=./server.pem \
--peer-key-file=./server-key.pem \
--trusted-ca-file=./ca.pem \
--peer-trusted-ca-file=./ca.pem  \
--data-dir=./cn3 \
--peer-client-cert-auth=true \
--client-cert-auth=true

./etcd --name cn4 \
--initial-advertise-peer-urls https://127.0.0.1:1480 \
--listen-peer-urls https://127.0.0.1:1480 \
--listen-client-urls https://$NODE:1479,https://127.0.0.1:1479 \
--advertise-client-urls https://$NODE:1479,https://127.0.0.1:1479 \
--initial-cluster-token etcd-cluster-token \
--initial-cluster cn0=https://127.0.0.1:1080,cn1=https://127.0.0.1:1180,cn2=https://127.0.0.1:1280,cn3=https://127.0.0.1:1380,cn4=https://127.0.0.1:1480 \
--initial-cluster-state new \
--cert-file=./server.pem \
--key-file=./server-key.pem \
--peer-cert-file=./server.pem \
--peer-key-file=./server-key.pem \
--trusted-ca-file=./ca.pem \
--peer-trusted-ca-file=./ca.pem  \
--data-dir=./cn4 \
--peer-client-cert-auth=true \
--client-cert-auth=true
```
## 验证集群的健康情况
```
./etcdctl --cert-file=./client.pem  --key-file=./client-key.pem --ca-file=./ca.pem --endpoints=https://$NODE:1179,https://$NODE:1279,https://$NODE:1379,https://$NODE:1479 cluster-health

./etcdctl --cert-file=./client.pem  --key-file=./client-key.pem --ca-file=./ca.pem --endpoints=https://$NODE:1179,https://$NODE:1279,https://$NODE:1379,https://$NODE:1479 member list
```
