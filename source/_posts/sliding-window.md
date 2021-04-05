---
title: 面试16种代码模式总结(1) - 滑动窗口
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

> 相关练习: [[1]](https://leetcode-cn.com/problems/permutation-in-string/)[[2]](https://leetcode-cn.com/problems/find-all-anagrams-in-a-string/)[[3]](https://leetcode-cn.com/problems/substring-with-concatenation-of-all-words/)

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

> 相关练习题: [[1]](https://leetcode-cn.com/problems/minimum-size-subarray-sum/)[[2]](https://leetcode-cn.com/problems/longest-substring-with-at-most-k-distinct-characters/)[[3]](https://leetcode-cn.com/problems/fruit-into-baskets/)[[4]](https://leetcode-cn.com/problems/longest-substring-without-repeating-characters/)[[5]](https://leetcode-cn.com/problems/longest-repeating-character-replacement/)[[6]](https://leetcode-cn.com/problems/max-consecutive-ones-iii/)[[7]](https://leetcode-cn.com/problems/minimum-window-substring/)

## 特殊例子
### 最小覆盖子串
原题见[leetcode](https://leetcode-cn.com/problems/minimum-window-substring/)

```golang
func findSubstring(str, pattern string) string {
	// 边界 
	if len(pattern)>len(str){
		return ""
	}
	var (
		windowStart = 0
		matched     = 0
		minLength   = len(str) + 1
		subStrStart = 0
		patternMap  = make(map[byte]int)
	)
	for i := 0; i < len(pattern); i++ {
		patternMap[pattern[i]]++
	}

	for windowEnd := 0; windowEnd < len(str); windowEnd++ {
		wright := str[windowEnd]
		if _, ok := patternMap[wright]; ok {
			patternMap[wright]-- 
			// 关键点1, 不再是==0，而是>=0，因为包含所有的字符
			if patternMap[wright] >= 0 {
				matched++
			}
		}
		// 如果当前window包含pattern所有的字符，则从左收缩窗口至不完全包含状态 
		// 注意这里等号右侧不是len(patternMap)，因为是要匹配所有字符
		for matched == len(pattern) { 
			if minLength > windowEnd-windowStart+1 {
				minLength = windowEnd - windowStart + 1
				subStrStart = windowStart
			}

			wleft := str[windowStart]
			if _, ok := patternMap[wleft]; ok {
				// 关键点2，只要有一个匹配的字符移除了窗口，则停止收缩
				if patternMap[wleft] == 0 {
					matched--
				}
				patternMap[wleft]++
			}
			windowStart++
		}
	}

	if minLength > len(str) {
		return ""
	} else {
		return str[subStrStart : subStrStart+minLength]
	}
}
```
__FAQ: 这里对上述代码做一定的解释，主要困惑点在于两个关键点__  
Q: 为什么关键点1处是>=0？  
A: 因为是要匹配所有的字符，因为每遇到一个需要匹配的字符，我们都需要对patternMap中对应值做减法并matched++, 直到我们匹配完了pattern所有该字符，此时对于多余该字符我们只需要更新patternMap，但是不必再更新matched。

Q: 为什么关键点2处是==0？  
A: 这里注意，如果窗口同一个字符没有冗余，那么移除了一个该字符则意味着匹配不完全，但是如果窗口内该字符有冗余，比如pattern是`aab`，而窗口内有3个`a`，此时`a`有冗余，如果收缩窗口只移出了一个`a`，那么此时窗口依然是满足匹配完全条件的，反映到代码上，收缩前`patternMap['a'] = -1` (在2的基础-3)，收缩后`patternMap['a']=0`，因此如果收缩掉下一个`a`，则需要matched--了。

### 串联所有单词的子串
原题见[leetcode](https://leetcode-cn.com/problems/substring-with-concatenation-of-all-words/)  
这个题目主要要对题意理解清楚，首先目标子串需要满足几个条件：
1. 长度等于单词列表中的每个单词长度*单词个数
2. 子串不可以出现不在列表上的其他单词

```go
func findWordConcatenation(str string, words []string) []int {
    if len(words)==0||len(str)==0{
        return []int{}
    }
	var (
		wordFreqMap = make(map[string]int)
		wordsCount  = len(words)
		wordLength  = len(words[0])
		// 存储满足条件的index
		startIndex  = make([]int, 0, 10) 
	)

	// 记录每个单词出现的频率
	for _, v := range words {
		wordFreqMap[v]++
	}

	// 注意这个不是常规的sliding window pattern 
	// i表示的是一个子串的起始位置，从条件可知每个子串是固定长度的 
	// 所以i最大为len(str)-wordLength*wordsCount
	for i := 0; i <= len(str)-wordLength*wordsCount; i++ {
		wordsSeenMap := make(map[string]int) // key1: 保存看过的word的数量

		// 对从当前index i开始的长度为wordLength*wordsCount的string进行判定
		for j := 0; j < wordsCount; j++ {
			nextWordIndex := i + j*wordLength // 当前word的开始index
			word := str[nextWordIndex : nextWordIndex+wordLength]
			// 如果出现不在words数组中的word，则直接跳出
			if _, ok := wordFreqMap[word]; !ok {
				break
			}

			wordsSeenMap[word]++ 
			// 如果words数组中某个word数量出现次数过多意味着一定会有其他单词不在，则也直接跳出
			if wordsSeenMap[word] > wordFreqMap[word] {
				break
			}

			// 如果遍历到该string的最后一个字符都没跳出循环，意味该string是满足条件的
			if j == wordsCount-1 {
				startIndex = append(startIndex, i)
			}
		}
		// ===end===
	}

	return startIndex
}
```


## REFERENCE
[1] https://www.educative.io/courses/grokking-the-coding-interview  
[2] https://github.com/zhiwei-Feng/Golang-Grokking-the-Coding-Interview-Patterns-for-Coding-Questions






