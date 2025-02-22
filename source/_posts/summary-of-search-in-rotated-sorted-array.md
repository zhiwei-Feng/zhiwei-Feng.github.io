---
title: Leetcode 旋转排序数组搜索问题总结
date: 2022-04-14 19:07:58
categories:
- 算法
tags:
- leetcode
- algorithm
- binary search
---

本篇文章主要对leetcode上关于旋转排序数组搜索问题的二分解法一个总结, 具体题目集有以下几个题目
1. [33. 搜索旋转排序数组](https://leetcode-cn.com/problems/search-in-rotated-sorted-array/)
2. [81. 搜索旋转排序数组 II](https://leetcode-cn.com/problems/search-in-rotated-sorted-array-ii/)
3. [153. 寻找旋转排序数组中的最小值](https://leetcode-cn.com/problems/find-minimum-in-rotated-sorted-array/)
4. [154. 寻找旋转排序数组中的最小值 II](https://leetcode-cn.com/problems/find-minimum-in-rotated-sorted-array-ii/)
<!-- more -->

我们可以根据排序数组是否允许重复元素将其分为元素不可重复和可重复两类.

## 元素不可重复

### 找目标值
对于每个mid位置,我们都根据他左侧有序还是右侧有序来考虑是否缩减左边界还是右边界,比如说对于mid的左侧有序(满足`nums[l]<=nums[mid]`)则判断target是否落在这个有序区间中,如果是则收缩右边界,否则收缩左边界,其他情况同理.
```go
// [33. 搜索旋转排序数组](https://leetcode-cn.com/problems/search-in-rotated-sorted-array/)

func search(nums []int, target int) int {
	l, r := 0, len(nums)-1
	for l < r {
		mid := l + (r-l)/2
		if nums[mid]==target {
			return mid
		}
		if nums[l]<=nums[mid] {
			if target<nums[mid]&&target>=nums[l] {
				r = mid-1
			}else {
				l = mid+1
			}
		}else {
			if nums[mid]<target&&target<=nums[r]{
				l = mid+1
			}else {
				r = mid-1
			}
		}
	}
	if target!=nums[l] {
		return -1
	}
	return l
}
```

### 找最小值
与找目标值不同,找最小值我们应该将数组看作为山,沿着下坡的方向进行移动,对于一个mid,如果左侧有序,左侧和右侧都可能是最小值存在的区间,因此我们应该通过判断右侧是否有序,如果有序,则当前mid及左侧为最小值存在的区间,因此收缩右边界即可.反之当前mid及左侧必定不可能为最小值存在的区间.
```go
// [153. 寻找旋转排序数组中的最小值](https://leetcode-cn.com/problems/find-minimum-in-rotated-sorted-array/)

func findMin(nums []int) int {
    l, r := 0, len(nums)-1
    for l < r {
        mid := l + (r-l)/2
        if nums[mid]<nums[r] {
            r = mid
        } else {
            l = mid+1
        }
    }
    return nums[l]
}
```

## 元素可重复

### 找目标值
在不可重复的基础上只需要多判断一下`mid, l, r`三者位置的值是不是一样, 当一样的时候无法判断收缩左侧还是右侧, 但是可以确定`l, r`两个位置肯定不是target, 因此直接左移l和右移r即可
```go
// [81. 搜索旋转排序数组 II](https://leetcode-cn.com/problems/search-in-rotated-sorted-array-ii/)

func search(nums []int, target int) bool {
    l, r := 0, len(nums)-1
    for l < r {
        mid := l + (r-l)/2
        if nums[mid]==target {
            return true
        }

        if nums[mid]==nums[l]&&nums[mid]==nums[r] {
            l++
            r--
        }else if nums[l]<=nums[mid] {
            if nums[l]<=target && target < nums[mid] {
                r = mid-1
            }else{
                l = mid+1
            }
        }else {
            if nums[mid]<target&&target<=nums[r] {
                l = mid+1
            }else {
                r = mid-1
            }
        }
    }
    return nums[l]==target
}
```

### 找最小值
和找目标值类似,在不可重复的基础,加一个`mid, l, r`三者的判断即可.
```go
// [154. 寻找旋转排序数组中的最小值 II](https://leetcode-cn.com/problems/find-minimum-in-rotated-sorted-array-ii/)

func findMin(nums []int) int {
    l, r := 0, len(nums)-1
    for l < r {
        mid := l + (r-l)/2
        if nums[l]==nums[mid]&&nums[r]==nums[mid]{
            l++
            r--
        }else if nums[mid]<=nums[r]{
            r = mid
        }else {
            l = mid+1
        }
    }
    return nums[l]
}
```