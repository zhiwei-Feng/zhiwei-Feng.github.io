---
title: 两种翻转slice的方式对比
date: 2021-04-16 23:30:42
tags:
- go
categories:
- Golang
toc: true
---

刷题时，遇到一个需求如下：对一个slice进行翻转。  
在实现的过程使用了两种方式：
- 每次插入新元素时，使用append左插入的方式
- 先正常append右插入，最后再对这个slice进行翻转

<!-- more -->

## 两种方式的实现及时间和空间消耗的对比

### 实现
下面两个方法实现的功能是一致的

- 方法1  
使用左插入法来实现翻转，如下所示
```golang
func method1() {
    result := make([][]int, 0)
    for i := 0; i < 100; i++ {
        item := make([]int, 0)
        for j := 0; j < 10; j++ {
            item = append(item, i*j)
        }
        // 左插入
        join := [][]int{item}
        result = append(join, result...)
    }
}
```

- 方法2  
先正常append，再翻转
```golang
func method2() {
    result := make([][]int, 0)
    for i := 0; i < 100; i++ {
        item := make([]int, 0)
        for j := 0; j < 10; j++ {
            item = append(item, i*j)
        }
        result = append(result, item)
    }
    tmp := result
    result = make([][]int, 0)
    for i := len(tmp) - 1; i >= 0; i-- {
        result = append(result, tmp[i])
    }
}
```

### 性能对比

- 时间
```text
goos: darwin
goarch: amd64
BenchmarkMethod1-4         17006             69558 ns/op
BenchmarkMethod2-4         43002             27851 ns/op
```
时间上，方法2要好于方法1

- 空间
  
![image.png](https://i.loli.net/2021/04/17/h3uNKYGrRn82zfd.png)
  
空间上，方法2也要优于方法1

### 分析
通过pprof工具对其源码进行分析如下
![image.png](https://i.loli.net/2021/04/17/1e9W4roOmxgXEf3.png)

从上图我们发现，`result = append(join, result...)`语句的内存消耗非常严重，
同时这种方法进行append，会使得地址重新分配（因为首地址改变了）致使多余内存和时间的消耗。

#### 补充
方法2的翻转可以有两种方法实现
- 如上面所示，通过slice反向遍历插入完成翻转
- 还可以通过双指针法来翻转

这里通过一个简单例子比较下双方的性能
```golang
func method1() {
    var input = make([]int, 0, 100)
    for i := 0; i < len(input); i++ {
        input = append(input, i)
    }
    // method1
    for i := 0; i < len(input)/2; i++ {
        j := len(input) - 1 - i
        input[i], input[j] = input[j], input[i]
    }
}

func method2() {
    var input = make([]int, 0, 100)
    for i := 0; i < len(input); i++ {
        input = append(input, i)
    }
    // method2
    tmp := input
    input = make([]int, 0, 100)
    for i := len(tmp) - 1; i >= 0; i-- {
        input = append(input, tmp[i])
    }
}
```
结果表示，双指针法会更好一些，理由很简单，因为双指针是O(N/2)的
```text
$ go test -run=. -bench=. -benchmem
goos: darwin
goarch: amd64
BenchmarkMethod1-4      71840024                16.9 ns/op             0 B/op          0 allocs/op
BenchmarkMethod2-4      26362216                39.7 ns/op             0 B/op          0 allocs/op
PASS
```


## 结论
这种情况下推荐使用方法2，同时对于方法1(左插入)的使用要格外谨慎，不当的使用会使得程序的时/空间消耗加剧。
> 如果有更好的实现方式，或者对上述程序有更好的建议，欢迎下方评论~




