---
title: go https 示例分析
date: 2018-03-17 00:00:00
tags: ["Go"]
abbrlink: go-https-example
img: ""
comments: false
---

本文首先大概介绍一下https的协议原理（其实是TLS协议原理），然后给出构建https的go代码例子。

## 1.HTTPS协议
网上已经有很多资料介绍HTTPS协议了，我归纳一下，加深自己的记忆。
HTTPS是HTTP通过SSL/TLS加密的方式进行通讯，TLS是SSL的改进版本。TLS协议可以分为握手阶段和对话阶段，握手阶段采用非对称加密方式（密钥由第三方机构提供），对话阶段采用对称加密方式（密钥由握手阶段协商得出）。
握手阶段主要分为4个步骤（细分来说，可以有多个步骤，具体查看RFC文档）：

### 1.1 Client Hello
客户端向服务器端发起hello请求，同时带入以下信息：

客户端支持的加密方式
客户端支持的SSL/TLS协议版本号
客户端生成的随机数1，该随机数在握手阶段无用，但是将用来生成对话阶段中对称加密的密钥。

### 1.2 Server Hello
客户端向服务器端发起hello请求，同时带入以下信息：

服务器从客户端支持的加密方式中挑出一个进行确认
确认SSL/TLS协议版本号
服务器生成的随机数2，同样将用来对话阶段生成密钥
服务器的证书（HTTPS由第三方机构颁发，当然可以调用openssl命令生成，然后私下发给客户端，客户端添加信任）
请求客户端证书（可选。通常来说，客户端认证服务器的证书就够了，但有些情况也需要认证客户端的证书，比如希望只对部分用户开放服务，该步骤在以下省略，如果有还会有2次报文交互才会进入1.3）



### 1.3 编码变更请求
客户端对服务器的证书进行认证，如果证书不是信任的第三方机构颁布或者证书过期，则向用户反馈是否继续。若用户确认，则从证书中拿出公钥，同时发送如下信息：

客户端生成随机数3，该随机数将用之前证书中的公钥加密。
变更编码通知：请求生成对称密钥
### 1.4 编码变更确认
服务器执行以下操作：

对编码进行确认：采用私钥解密随机数3，同时采用固定算法生成对话阶段密钥（输入为随机数1-3），之后通讯将采用该密钥进行对称加密
结束握手阶段
## 2.代码示例
网上有一些例子，但是解释的很少，有些根本是乱讲，生成了一堆证书，分别是什么也没有讲清楚，此处我结合协议给出代码示例，方便理解。其实概念很简单：*.key是私钥，*.crt是包含公钥的证书。
假设服务端不需要认证客户端代码，但是客户端需要认证服务器，则通过openssl命令生成服务端证书server.crt，并私下传递给客户端。
注意：服务端的证书和客户端的证书在哪里生成都可以，只要保证两侧一致即可（scp拷贝）。比如，可以在服务侧生成client.crt和client.key。还有openssl生成时候需要填入以下选项，其中Common Name需要填入服务提供方的domain（域名），我试了一下，不支持通配（比如*.baidu.com），不知道是不是我操作问题，如果有谁知道请告知一下。此处CN选择localhost的话只有本机访问：
```
Country Name (2 letter code) [AU]:CN  
State or Province Name (full name) [Some-State]:Zhejiang  
Locality Name (eg, city) []:Hangzhou  
Organization Name (eg, company) [Internet Widgits Pty Ltd]:My CA  
Organizational Unit Name (eg, section) []:  
Common Name (e.g. server FQDN or YOUR name) []:localhost  
Email Address []:  
```
生成server的私钥和证书，最后的CN即Common Name。这个命令也可以生成client.key和client.crt，只要把名字改一下就可以，当然还有CN。
```
openssl req \  
    -x509 \
    -nodes \
    -newkey rsa:2048 \
    -keyout server.key \
    -out server.crt \
    -days 3650 \
    -subj "/C=GB/ST=London/L=London/O=Global Security/OU=IT Department/CN=*"
```

### 2.1 互相不认证
Server侧代码：
```
package main

import (  
    "crypto/tls"
    "log"
    "net/http"
)

func main() {  
    cfg := &tls.Config{
        InsecureSkipVerify: true, //忽略对客户端的认证
    }
    srv := &http.Server{
        Addr:      ":11443",
        Handler:   &handler{},
        TLSConfig: cfg,
    }
    log.Fatal(srv.ListenAndServeTLS("server.crt", "server.key"))
}

type handler struct{}

func (h *handler) ServeHTTP(w http.ResponseWriter, req *http.Request) {  
    w.Write([]byte("PONG\n"))
}
```
Client侧代码：
```
package main

import (  
    "crypto/tls"
    "fmt"
    "io/ioutil"
    "log"
    "net/http"
)

func main() {  
    client := &http.Client{
        Transport: &http.Transport{
            TLSClientConfig: &tls.Config{
                InsecureSkipVerify: true, //忽略对服务器的认证
            },
        },
    }

    resp, err := client.Get("https://xxx.com:11443")
    if err != nil {
        log.Println(err)
        return
    }

    htmlData, err := ioutil.ReadAll(resp.Body)
    if err != nil {
        fmt.Println(err)
        return
    }
    defer resp.Body.Close()
    fmt.Printf("%v\n", resp.Status)
    fmt.Printf(string(htmlData))
}
```

### 2.2 服务器不认证客户端，客户端认证服务器
Server侧代码如上不变。
Client代码添加服务端证书server.crt：
```
package main

import (  
    "crypto/tls"
    "crypto/x509"
    "fmt"
    "io/ioutil"
    "log"
    "net/http"
)

func main() {  
    caCert, err := ioutil.ReadFile("server.crt") //添加服务端的证书
    if err != nil {
        log.Fatal(err)
    }
    caCertPool := x509.NewCertPool()
    caCertPool.AppendCertsFromPEM(caCert)

    client := &http.Client{
        Transport: &http.Transport{
            TLSClientConfig: &tls.Config{
                RootCAs:      caCertPool, //添加认证
            },
        },
    }

    resp, err := client.Get("https://xxx.com:11443")
    if err != nil {
        log.Println(err)
        return
    }

    htmlData, err := ioutil.ReadAll(resp.Body)
    if err != nil {
        fmt.Println(err)
        return
    }
    defer resp.Body.Close()
    fmt.Printf("%v\n", resp.Status)
    fmt.Printf(string(htmlData))
}
```

### 2.3 服务器认证客户端，客户端也认证服务器
Server侧代码添加client.crt:
```
package main

import (  
    "crypto/tls"
    "crypto/x509"
    "io/ioutil"
    "log"
    "net/http"
)

func main() {  
    caCert, err := ioutil.ReadFile("client.crt")
    if err != nil {
        log.Fatal(err)
    }
    caCertPool := x509.NewCertPool()
    caCertPool.AppendCertsFromPEM(caCert)

    cfg := &tls.Config{
        ClientAuth: tls.RequireAndVerifyClientCert,  //添加对客户端的认证
        //InsecureSkipVerify: true,
        ClientCAs:  caCertPool,
    }
    srv := &http.Server{
        Addr:      ":11443",
        Handler:   &handler{},
        TLSConfig: cfg,
    }
    log.Fatal(srv.ListenAndServeTLS("server.crt", "server.key"))
}

type handler struct{}

func (h *handler) ServeHTTP(w http.ResponseWriter, req *http.Request) {  
    w.Write([]byte("PONG\n")
}
```
Client侧代码添加server.crt:
```
package main

import (  
    "crypto/tls"
    "crypto/x509"
    "fmt"
    "io/ioutil"
    "log"
    "net/http"
)

func main() {  
    caCert, err := ioutil.ReadFile("server.crt")
    if err != nil {
        log.Fatal(err)
    }
    caCertPool := x509.NewCertPool()
    caCertPool.AppendCertsFromPEM(caCert)

    cert, err := tls.LoadX509KeyPair("client.crt", "client.key")
    if err != nil {
        log.Fatal(err)
    }


    client := &http.Client{
        Transport: &http.Transport{
            TLSClientConfig: &tls.Config{
                RootCAs:      caCertPool,
                Certificates: []tls.Certificate{cert},
                //InsecureSkipVerify: true,
            },
        },
    }

    resp, err := client.Get("https://xxx.com:11443")
    if err != nil {
        log.Println(err)
        return
    }

    htmlData, err := ioutil.ReadAll(resp.Body)
    if err != nil {
        fmt.Println(err)
        return
    }
    defer resp.Body.Close()
    fmt.Printf("%v\n", resp.Status)
    fmt.Printf(string(htmlData))
}
```
说明：
装载请注明链接：http://vinllen.com/go-https-example/

参考：
https://tools.ietf.org/html/rfc5246#section-7.4.1.2 
http://www.barretlee.com/blog/2015/10/05/how-to-build-a-https-server/ 
http://colobu.com/2016/06/07/simple-golang-tls-examples/ 
https://github.com/jcbsmpsn/golang-https-example
