---
title: "[6.824系列] Outline of lab2A solution"
date: 2022-04-26 00:11:29
categories:
- 6.824
tags: 
- go
- 分布式
---

本文主要总结了如何实现[6.824 lab2A](http://nil.csail.mit.edu/6.824/2020/labs/lab-raft.html).
<!-- more -->
主要任务如下:
1. 实现Raft Leader Election
2. 实现leader和follower之间的心跳机制(不带log)
这个lab关键是要处理好多个goroutine对Raft状态机的并发写.
> 处理不好, 容易造成死锁/有限时间内选不出leader/选出多个leader等多种问题.

主要实现参考论文中的Figure 2
![image.png](https://s2.loli.net/2022/04/26/H89GzoYJsuiNwSF.png)

## Raft状态机
我们首先需要定义出 `leader election` 和 `heartbeat` 两个过程中Raft需要维护的数据, 这一点在Raft论文中的Figure 2已经说明了, 我们在它的基础上加上一些运行时数据.具体如下:
```go
type Raft struct {
    ... // other

    currentTerm     int             // latest term server has seen
    votedFor        int             // 选举过程投给谁一票, 初始化或reset后为-1表示未投票

    roleState       int             // 表示server当前处于三角色之一: leader, candidate, follower
    electionTimeout time.Duration   // 当前term所设置的选举超时, 同时也是心跳超时
    lastTickTime    time.Time       // 上一次刷新心跳的时间
}
```

## RPC调用
### AppendEntries
在Lab2A中, AppendEntries主要用于心跳机制, 因此主要用于通知follower当前的term是什么, 以便于follower进行term的更新和timeout的重置.  
具体来说, AppendEntries调用处理的过程分为下面几个部分:
1. 上锁, 因为后续需要读写raft状态和数据
2. 判断心跳传递过来的term是否比当前节点的term新, 如果不新的话, 表示该心跳无效, 返回false
3. 有效则刷新 `lastTickTime`
4. 满足以下三种情况则重置节点状态
    - [leader with older term]
    - [candidate with older or same term]
    - [follower with older term]

> 重置节点状态实际上就是
> 1. 更新term到最新
> 2. votedFor=-1
> 3. roleState=Follower

### RequestVote
RequestVote用于当节点提升为candidate的时候, 向其他节点索要投票时调用. 具体来说, candidate需要将自己的term和id告知其他节点, 而其他节点收到RequestVote后, 需要对term的合法性(newer)进行验证, 如果term合法且节点未投票给其他节点或者已经投票给candidate, 则投票给该candidate. 大致处理过程有以下几个部分:
1. 加锁
2. 如果发送过来的term更早, 说明该选举过程已过时, 直接返回false
3. 如果term更新, 说明是新一轮的选举, 此时节点需要重置状态
4. 判断是否未投过票或者该请求的candidateId是已选过的候选者, 则给该候选者投票
    - 刷新 `lastTickTime`


## 心跳机制
每个raft节点如果是leader状态的话,会周期性地向其他节点发送心跳. 在Lab2A中, 心跳机制在发送方其实不需要做什么事情, 唯一需要做的是在心跳ack的term比自己的term大的情况下, 需要重置自己的状态, 这可能是因为网络等原因导致其他节点超时收不到心跳而发起了新的选举.
> 各个节点维护一个共同的peers数组, 表示集群所有节点的网络访问地址

## 选举
每个raft节点会在不是leader的状态且选举超时的情况下发起选举, 我们可以将这个过程转变为一个周期性过程: 即每隔 `election timeout` 就尝试发起新的选举, 如果已经是leader状态(上一次选举成功)或者选举还未超时(其他leader刷新了节点的lastTickTime)则sleep, 否则发起选举. 具体选举过程如下:
1. 向其他节点索要投票
2. 检查返回结果和当前状态
    - 如果不是candidate(说明有新的leader重置了当前节点的状态)或者当前的term已经被其他选举更新过, 则直接退出选举
    - 返回的term更新, 则说明当前candidate状态无效, 重置当前状态并退出选举
3. 返回结果为true, 则记一票, 超过半数的票的情况, 当前节点当选leader并立刻发起心跳给其他节点同步状态.

## 注意点
1. 任何需要查询和修改raft状态的操作都需要加锁
2. 避免在持有锁的过程中请求其他的锁, 极易发生死锁
3. 避免在持有锁的过程中进行RPC调用, 应在rpc调用之前释放锁. 接收返回结果后在加锁处理后续逻辑

> 这是因为在这个作业中, 被调方和调用方实际上共享锁, 这会导致其中一个协程A持有锁进行RPC调用而被调用方处于协程B由于锁被占用而阻塞.
