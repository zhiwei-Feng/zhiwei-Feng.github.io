---
title: 从6.824的kv.go理解tcp协议
date: 2021-04-11 14:08:55
tags:
- network
- socket
- tcp
categories:
- socket program
toc: true
---

本文是在学习6.824课程时，对Lec 2的kv.go产生的困惑的解释和总结。
<!-- more -->

## kv.go
首先对kv.go的情况进行解释。这里只会取一些重点片段进行解释，需要了解完整代码细节的，读者可自行选择文章最后的参考条目进行详细了解。

### 主逻辑
```go
func main() {
    //启动rpc服务，一个kv存储
    server() 

    //写入一个kv键值对
    put("subject", "6.824")
    fmt.Printf("Put(subject, 6.824) done\n")
    //读出刚才存储的key的value
    fmt.Printf("get(subject) -> %s\n", get("subject"))
}
```
主逻辑非常简单，启动一个kv存储的rpc服务，然后运行简单的读写功能测试。

### server()
```go
func server() {
    kv := new(KV)
    kv.data = map[string]string{}
    rpcs := rpc.NewServer()
    rpcs.Register(kv)
    l, e := net.Listen("tcp", ":1234")
    if e != nil {
        log.Fatal("listen error:", e)
    }
    go func() {
        for {
            conn, err := l.Accept()
            if err == nil {
                go rpcs.ServeConn(conn)
            } else {
                break
            }
        }
        l.Close()
    }()
}
```
启动服务，监听1234端口，并开始接受客户端的连接

### client
```go
func get(key string) string {
    client := connect()
    args := GetArgs{key}
    reply := GetReply{}
    err := client.Call("KV.Get", &args, &reply)
    if err != nil {
        log.Fatal("error:", err)
    }
    client.Close()
    return reply.Value
}

func put(key string, val string) {
    client := connect()
    args := PutArgs{key, val}
    reply := PutReply{}
    err := client.Call("KV.Put", &args, &reply)
    if err != nil {
        log.Fatal("error:", err)
    }
    client.Close()
}
```
客户端方法总体逻辑是一致，首先进行tcp连接的建立，然后进行rpc方法调用，最后关闭连接

## 解析过程
如果你运行该程序，会发现打印如下：
```
$ go run kv.go
Put(subject, 6.824) done
get(subject) -> 6.824

$
```
> 为什么程序最后会退出，而不是因为server在监听阻塞住。我自己的解释是：首先是因为server()中的无限接收请求的循环是运行在goroutine当中，所以之后的客户端请求可以并发的运行；其次因为主线程没有其他方法的阻塞，所以会在所有逻辑结束后退出，也意味该进程退出了，这样其中的所有线程都会结束，包括其中创建的goroutine。
> 
> 如果读者有更好的解释欢迎留言讨论。

在该程序的主逻辑中，实际发生了两次tcp连接，我们知道一次tcp协议的运行过程有三个阶段：连接创建、数据传送和连接终止。server()下开启的rpc服务监听着1234端口，而get和put()中的connect()则完成一次tcp连接创建的三次握手，如图下

![Connection_TCP.png](https://i.loli.net/2021/04/11/qBcVlTjmay8PdsX.png)

此时通过netstat命令查看会出现有两个连接以及一个监听  

![image.png](https://i.loli.net/2021/04/11/a7CvRZsMGj9dTbq.png)

当数据传送完毕，get和put()调用Close()方法进入连接终止阶段（四次握手）后，客户端状态则会进入TIME_WAIT状态

![Deconnection_TCP.png](https://i.loli.net/2021/04/11/5b6XhAoYGQkCwDZ.png)

![image.png](https://i.loli.net/2021/04/11/LqDOcPjM27Q6eXC.png)

等待2MSL后，客户端Close。

> 如果希望对TIME_WAIT的设计想要更深入了解，可以参考[为什么 TCP 协议有 TIME_WAIT 状态](https://draveness.me/whys-the-design-tcp-time-wait/)

## REFERENCE
[1] [6.824 kv.go](http://nil.csail.mit.edu/6.824/2020/notes/kv.go)  
[2] [维基百科](https://zh.wikipedia.org/wiki/%E4%BC%A0%E8%BE%93%E6%8E%A7%E5%88%B6%E5%8D%8F%E8%AE%AE)  
