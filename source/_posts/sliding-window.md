---
title: 面试16种代码模式总结(1) - 互动窗口
date: 2021-04-04 22:37:30
tags:
- algorithm
- sliding window
- go
categories:
- [面试, 16种代码模式]
toc: true
---

本系列文章是对[Grokking the Coding Interview: Patterns for Coding Questions](https://www.educative.io/courses/grokking-the-coding-interview)课程的总结，编程语言使用Go。读者如果想要更细致的了解，请自行购买课程学习。

## 问题背景
在处理数组和链表的时候，我们经常会需要找出满足某些条件的连续子列, 比如子列的和最大等，这个时候就可以使用滑动窗口的思想来进行解答。这里注意的是子列可以是固定大小也可以是可变大小，两种情况有相应的处理方式。

<!-- more -->

下面先给出一个例子，比如在给定一个数组，找出其中每个size为K的连续子列的平均值。  
```text
Array: [1, 3, 2, 6, -1, 4, 1, 8, 2], K=5

Output: [2.2, 2.8, 2.4, 3.6, 2.8]
```

滑动窗口的解决方案如下:
```go
func findAveragesOfSubArraysBySlidingWindow(K int, arr []int) []float64 {
	results := make([]float64, len(arr)-K+1)
	windowSum := 0
	windowStart := 0
	for windowEnd := 0; windowEnd < len(arr); windowEnd++ {
		// 把下个元素加上
		windowSum += arr[windowEnd]
		// 滑动窗口，特别地，如果没有达到K个则不滑动
		if windowEnd >= K-1 {
			// 当前windowSum计算了整个窗口的和
			results[windowStart] = float64(windowSum) / float64(K)
			// 当前窗口计算完后，移动窗口需要先减去原来窗口头部的元素
			windowSum -= arr[windowStart]
			windowStart++
		}
	}
	return results
}
```
time: O(n), extra space: O(1)

## 解法总结
根据窗口大小是否固定，分为固定大小和可变大小

### 固定大小
这里拿 找出size=k的最大连续子序列和 作为例子
```text
Given an array of positive numbers and a positive number k,
find the maximum sum of any contiguous subarray of size k.

Input: [2, 1, 5, 1, 3, 2], k=3
Output: 9
Explanation: Subarray with maximum sum is [5, 1, 3].

Input: [2, 3, 4, 1, 5], k=2
Output: 7
Explanation: Subarray with maximum sum is [3, 4].
```

解决方案
```go
func max(x, y int) int {
	if x > y {
		return x
	} else {
		return y
	}
}

func solution(arr []int, k int) int {
    var (
        windowStart = 0
        windowSum = 0 // 用于记录当前窗口的状态
        maxSum = 0 // 用于记录最好的状态
    )

    for windowEnd:=0;windowEnd<len(arr);windowEnd++{
        // 这一步是窗口的右侧进行一格扩张，同时更新当前窗口的状态
        windowSum += arr[windowEnd]
        // 判断当前窗口是否满足相应条件
        if windowEnd >= k-1{
            // 更新最好的状态
            maxSum = max(maxSum, windowSum)
            // 收缩窗口左侧
            windowSum -= arr[windowStart]
            windowStart--
        }
    }

    return maxSum
}
```
time: O(n), extra space: O(1)  

### 可变大小
这里通过[长度最小的子数组](https://leetcode-cn.com/problems/minimum-size-subarray-sum/)来说明

```go
func min(i, j int) int {
    if i<j {
        return i
    }else{
        return j
    }
}

func minSubArrayLen(target int, nums []int) int {
    var(
        windowStart = 0
        windowSum = 0
        minSize = -1
    )

    for windowEnd:=0;windowEnd<len(nums);windowEnd++{
        // 窗口右侧扩张，这个部分和固定大小是一样的
        windowSum+=nums[windowEnd]
        // 这个部分是与固定大小类型最大的区别
        // 不同于固定大小每次的右侧扩张和左侧收缩是同步的
        // 可变大小的左侧收缩需要循环判断窗口是否满足条件
        // 可能右侧扩张了很多次，但是左侧才收缩一回
        // 也可能右侧扩张一次，左侧需要收缩几回
        for windowSum>=target{
            if minSize==-1{
                minSize = windowEnd-windowStart+1
            }else{
                minSize = min(minSize, windowEnd-windowStart+1)
            } 
            windowSum-=nums[windowStart]
            windowStart++
        }

        // 对于一些情况条件需要进行后处理，比如当收缩窗口直到窗口内单一字符数量<k
        // 然后再比较当前窗口的大小
    }
    
    if minSize==-1{
        return 0
    }
    return minSize
}
```
time: O(n), extra space: O(1)  

> 相关练习题: [[1]](https://leetcode-cn.com/problems/minimum-size-subarray-sum/)[[2]](https://leetcode-cn.com/problems/longest-substring-with-at-most-k-distinct-characters/)[[3]](https://leetcode-cn.com/problems/fruit-into-baskets/)[[4]](https://leetcode-cn.com/problems/longest-substring-without-repeating-characters/)

## REFERENCE
[1] https://www.educative.io/courses/grokking-the-coding-interview  
[2] https://github.com/zhiwei-Feng/Golang-Grokking-the-Coding-Interview-Patterns-for-Coding-Questions






