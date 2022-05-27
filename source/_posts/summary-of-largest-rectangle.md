---
title: Leetcode 最大矩形问题总结
date: 2022-05-27 08:34:26
categories:
- 算法
tags:
- leetcode
- algorithm
- 单调栈
---

## 概述
该节设计的Leetcode题目为:
1. [84. 柱状图中最大的矩形](https://leetcode.cn/problems/largest-rectangle-in-histogram/)
2. [85. 最大矩形](https://leetcode.cn/problems/maximal-rectangle/)

实际上, 85题可以看作是基于84题来拓展了一维的问题.首先来看84题,如下图,可以看到题目的意思就是给定一组宽度为1的矩形块,求出其中可以构成的最大矩形的面积.
![example](https://assets.leetcode.com/uploads/2021/01/04/histogram.jpg)

## 84.柱状图中最大的矩形
不难看出,这里面有个隐含的提示,及对于某个矩形块(高度为x)而言,要想以它作为最大矩形的高度,那么就必须保证构成矩阵的相邻高度都大于等于x.

比如说上图中,我们如果要以矩形高度为5的块作为最大矩形的高,则左侧已经没有块高度大于等于5,所以左侧无法拓展,而右侧因为有高度6的块,所以可以拓展一次,最终构造出来的矩形面积为2*5=10.

因此解法即为遍历每一个块高度,并以它为中心向两侧拓展并计算其拓展后的矩形面积,取其中最大值.但是显然这种做法时间复杂度较高,因为它重复遍历两侧的元素.

上述这种解法换个角度看可以认为是找到某个高度为x的矩形块左侧第一个高度低于x的块`i`和右侧第一个高度低于x块`j`,则构成的矩形面积为`area = x*(j-i-1)`.启发自*单调栈*的思想,我们可以维护一个严格递增的栈,该栈里面存储矩形块的索引index,栈中元素满足从底到顶的矩形块高度严格递增.

> 单调栈如何实现上述解法?
> 1. 保证栈单调的同时入栈当前的矩阵块索引i, 这样可以保证左侧第一个高度低于`heights[i]`的矩形块索引就是底下的那个元素
> 2. 另一方面,单调栈入栈前需要弹出其中高度大于等于当前块i高度的索引,这一步实际上对于每个弹出的矩阵块k是找到了右侧第一个高度低于`heights[k]`的矩形块索引,即i.

因此,解法就很直接,维护单调栈的过程中,每弹出一个矩形块索引i,则以其高度`heights[i]`为矩形高度,宽度为`(i-peek()-1)`来计算当前面积,如此其中的最大值即为答案.具体代码过程如下:
```Java
public int largestRectangleArea(int[] heights) {
    Deque<Integer> stk = new ArrayDeque<>();
    var ans = 0;
    for (int i = 0; i < heights.length; i++) {
        while (!stk.isEmpty() && heights[stk.peek()] >= heights[i]) {
            var curIdx = stk.pop();
            var leftIdx = stk.isEmpty() ? -1 : stk.peek();
            ans = Math.max(ans, heights[curIdx] * (i - leftIdx - 1));
        }
        stk.push(i);
    }
    var rightIdx = heights.length;
    while (!stk.isEmpty()) {
        var curIdx = stk.pop();
        var leftIdx = stk.isEmpty() ? -1 : stk.peek();
        ans = Math.max(ans, heights[curIdx] * (rightIdx - leftIdx - 1));
    }
    return ans;
}
```
> 稍微注意一下边界问题, 当弹出栈顶的时候, 如果左侧没有元素,即栈空的情况下,左侧索引取-1. 而在遍历完成后,栈不为空,则说明栈中每个元素的右侧索引为数组长度n(`n=heights.length`).

## 85.最大矩形
这一题我们将其每一行看作一个84题中的heights数组,因此只需要遍历每一行的字符,如果为'1'则更新heights中对应索引的高度(+1),如果为'0',则置0.然后应用84的算法计算该行的最大矩形面积.最终取每一行计算的面积最大值为答案. 代码如下:

```java
public int maximalRectangle(char[][] matrix) {
    if (matrix.length == 0) {
        return 0;
    }
    var heights = new int[matrix[0].length];
    var ans = 0;
    for (int i = 0; i < matrix.length; i++) {
        for (int j = 0; j < matrix[i].length; j++) {
            if (matrix[i][j] == '1') {
                heights[j]++;
            } else {
                heights[j] = 0;
            }
        }
        ans = Math.max(ans, maxRec(heights));
    }
    return ans;
}

// 84.
int maxRec(int[] heights) {
    var ans = 0;
    Deque<Integer> stk = new ArrayDeque<>();
    for (int i = 0; i < heights.length; i++) {
        while (!stk.isEmpty() && heights[stk.peek()] >= heights[i]) {
            var curIdx = stk.pop();
            var leftIdx = stk.isEmpty() ? -1 : stk.peek();
            ans = Math.max(ans, heights[curIdx] * (i - leftIdx - 1));
        }
        stk.push(i);
    }
    var rightIdx = heights.length;
    while (!stk.isEmpty()) {
        var curIdx = stk.pop();
        var leftIdx = stk.isEmpty() ? -1 : stk.peek();
        ans = Math.max(ans, heights[curIdx] * (rightIdx - leftIdx - 1));
    }
    return ans;
}
```
