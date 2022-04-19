---
title: Leetcode 组合总和问题总结
date: 2022-04-18 00:07:19
categories:
- 算法
tags:
- leetcode
- algorithm
- backtrace
cover: https://s2.loli.net/2022/04/19/CgdTPa8ncFltV53.png
---

组合总和系列题总结.
1. [39. 组合总和](https://leetcode-cn.com/problems/combination-sum/)
2. [40. 组合总和II](https://leetcode-cn.com/problems/combination-sum-ii/)
3. [216. 组合总和III](https://leetcode-cn.com/problems/combination-sum-iii/)
4. [377. 组合总和IV](https://leetcode-cn.com/problems/combination-sum-iv/)
<!-- more -->

## 组合总和
本题注意candidates是无重复元素,且元素都大于0, 利用backtrace的思想求解即可
```go
func combinationSum(candidates []int, target int) [][]int {
    ans := [][]int{}
    path := []int{}
    pathSum := 0
    var backtrace func(idx int)
    backtrace = func(idx int) {
        if idx == len(candidates) || pathSum >= target {
            if pathSum == target {
                tmp := make([]int, len(path))
                copy(tmp, path)
                ans = append(ans, tmp)
            }
            return
        }
        backtrace(idx+1)
        for i:=1;i<=target/candidates[idx];i++{
            if candidates[idx]*i+pathSum<=target {
                pathSum+=candidates[idx]*i
                for k:=0;k<i;k++{
                    path = append(path, candidates[idx])
                }
                backtrace(idx+1)
                pathSum-=candidates[idx]*i
                path = path[:len(path)-i]
            }else {
                break
            }
        }
    }
    backtrace(0)
    return ans
}
```

## 组合总和II
本题的关键是如何去重, 思路依旧是backtrace. 具体去重的思想如下:
1. 对于candidates和target以及当前索引idx, 我们保证`candidates[idx]`只能被选取一次来扣减target, 从而保证当前情况下没有重复, 并将问题转化为`candidates[idx+1:]`凑成`target-candicates[idx]`的子问题.
2. 为了能够方便的避免重复candidates的选取, 我们将candidates首先进行排序

```go
func combinationSum2(candidates []int, target int) [][]int {
    var (
        ans = make([][]int, 0)
        path = make([]int, 0)
        backtrace func(int, int)
        n = len(candidates)

    )
    sort.Ints(candidates)
    backtrace = func(begin int, remain int) {
        if remain==0 {
            ans = append(ans, append([]int{}, path...))
            return
        }

        for i:=begin;i<n;i++{
            if candidates[i]>remain{
                // 当前候选值太大，剪枝
                break
            }
            if i>begin&&candidates[i]==candidates[i-1]{
                // 因为candidates[i-1]存在被选取的情况, 为避免重复跳过后续的重复元素
                continue
            }
            path = append(path, candidates[i])
            backtrace(i+1, remain-candidates[i])
            path = path[:len(path)-1]
        }
    }

    backtrace(0, target)
    return ans
}
```

## 组合总和III
注意条件中需要组合列表个数等于k, 因此回溯的返回条件因加入判断.
```go
func combinationSum3(k int, n int) (ans [][]int) {
    path := []int{}
    var backtrace func(idx, remain int)
    backtrace = func(idx, remain int) {
        if idx > 9 || remain == 0 || len(path) == k {
            // idx表示当前遍历到的数字, 大于9则过界
            // 同时如果remain等于0, 或者path的长度达到k了都属于到达边界的情况
            if remain == 0 && len(path) == k {
                ans = append(ans, append([]int{}, path...))
            }
            return
        }

        for i:=idx;i<=9;i++{
            if i>remain {
                break
            }
            path = append(path, i)
            backtrace(i+1, remain-i)
            path = path[:len(path)-1]
        }
    }
    backtrace(1, n)
    return
}
```

## 组合总和IV(*)
本题使用backtrace会超时, 因此需要使用dp来解, 本题的dp解法类似于完全背包问题, 但需要注意的是对于`target=7, nums=[1,2,3]`的输入, 完全背包认为`[1,2,3], [1,3,2], [2,1,3], [2,3,1], [3,1,2], [3,2,1]`六种情况是一种方案, 而本题认为是六种方案, 这一点是本题解法的关键.
```go
func combinationSum4(nums []int, target int) int {
    n := target
    // dp的含义, dp[i][j]对于长度i的组合, 总和为j的方案数, 本题由于nums全部为正数, 因此i最大为target
    dp := make([][]int, n+1)
    for i:=range dp {
        dp[i] = make([]int, target+1)
    }
    dp[0][0]=1
    ans := 0
    for i:=1;i<=n;i++{
        for j:=0;j<=target;j++{
            for _, u := range nums {
                if j>=u {
                    dp[i][j]+=dp[i-1][j-u]
                }
            }
        }
        // 对于不同长度的组合, 总和为target的方案, 我们累加
        ans += dp[i][target]
    }

    return ans
}
```

数组降维, 减少空间复杂度
```go
func combinationSum4(nums []int, target int) int {
    n := target
    dp := make([]int, target+1)
    dp[0]=1
    ans := 0
    for i:=1;i<=n;i++{
        for j:=target;j>=0;j--{
            dp[j] = 0
            for _, u := range nums {
                if j>=u {
                    dp[j]+=dp[j-u]
                }
            }
        }
        ans += dp[target]
    }

    return ans
}
```

进一步减少时间复杂度
```go
func combinationSum4(nums []int, target int) int {
    // 这个解法类似于爬楼梯, 对于每个总和j而言, nums为每次攀爬的楼梯数
    // 所以 dp[j] = sum of { dp[j-nums[i]] for each i belong to 0..len(nums) }
    dp := make([]int, target+1)
    dp[0]=1
    for j:=1;j<=target;j++{
        for _, u := range nums {
            if j>=u {
                dp[j]+=dp[j-u]
            }
        }
    }

    return dp[target]
}
```
