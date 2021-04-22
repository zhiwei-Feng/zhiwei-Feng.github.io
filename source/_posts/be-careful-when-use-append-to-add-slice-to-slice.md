---
title: slice使用append的小坑
date: 2021-04-22 21:36:36
categories:
- Golang
tags:
- go  
- 指针
toc: true
---

今天在刷一道算法题的时候遇到了一个关于slice在使用append的小细节问题。这个算法题可以参考[路径总和II](https://leetcode-cn.com/problems/path-sum-ii/)。  
题意很简单，就是从一个二叉树中所有由根到叶子节点的路径中找到所有的满足路径和等于target的路径。
<!-- more -->

## 问题
在参考官方题解的golang版本的时候，发现了有一段代码是我一开始没弄明白的。完整代码可参考
```golang
func pathSum(root *TreeNode, targetSum int) (ans [][]int) {
    path := []int{}
    var dfs func(*TreeNode, int)
    dfs = func(root *TreeNode, left int) {
        if root == nil {
            return
        }
        left -= root.Val
        path = append(path, root.Val)
        defer func() { path = path[:len(path)-1] }()
        if left == 0 && root.Left == nil && root.Right == nil {
            ans = append(ans, append([]int{}, path...))
            return
        }
        dfs(root.Left, left)
        dfs(root.Right, left)
    }
    dfs(root, targetSum)
    return
}
```
其中的`ans = append(ans, append([]int{}, path...))`则是我关注的地方，一开始我疑惑为什么不直接`ans = append(ans, path))`，但是实验证明结果会非常奇怪，比如对于这样一个例子  
![image.png](https://i.loli.net/2021/04/22/jxz98OSKfoTXsmY.png)
我们想要的是`[[5,4,11,2],[5,8,4,5]]`，但是实际上是`[[5,8,4,1],[5,8,4,1]]`.  

## 解析
这里其实是我忘了golang中的slice底层实现如下图所示
```golang
type slice struct {
    array unsafe.Pointer
    len   int
    cap   int
}
```
因此如果像`ans = append(ans, path))`这样做，其实在ans存放的是path这个slice，在后续代码运行中的改变会导致ans的结果改变，从而产生一个错误的结果。  
而`ans = append(ans, append([]int{}, path...))`则是在ans中存放了一个path的副本，且该副本不会被其他代码修改到。因此结果才是正确的。

> 一行代码就揭示golang中slice的本质~，真实妙啊。

## REFERENCE
[1] [leetcode 113题](https://leetcode-cn.com/problems/path-sum-ii/)  
[2] [深入解析 Go 中 Slice 底层实现](https://halfrost.com/go_slice/)


