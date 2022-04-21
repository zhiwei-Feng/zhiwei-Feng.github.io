---
title: bugfix of pytorch out of memory
date: 2022-04-21 18:42:22
categories:
- pytorch
tags:
- pytorch
- gpu
- oom
cover: https://s2.loli.net/2022/04/21/yreukaM2XIs8A4f.png
---

记录一次模型推理过程中显存不断增加直至Out-of-Memory的解决过程.
<!-- more -->

## 场景
当我在算法中加入了一个运算量不大的操作后, 模型推理过程直接从原来的显存占用不到11G直接爆了3090的显存.

## 显存增加定位
首先需要判断是什么地方造成了显存的增加. 这里推荐一个库[Pytorch-Memory-Utils](https://github.com/Oldpan/Pytorch-Memory-Utils).该库可以通过侵入式的track方法记录两个track方法之间的显存变化. 其记录的格式如下

```text
At main.py line 18: <module>  Total Tensor Used Memory:466.4  Mb Total Allocated Memory:466.4  Mb
+ | 1 * Size:(120, 3, 512, 512)   | Memory: 360.0 M | <class 'torch.Tensor'> | torch.float32
+ | 1 * Size:(80, 3, 512, 512)    | Memory: 240.0 M | <class 'torch.Tensor'> | torch.float32
At main.py line 23: <module>  Total Tensor Used Memory:1066.4 Mb Total Allocated Memory:1066.4 Mb
```

最终是发现在模型decoder部分导致了显存的激增, 并且随着推理的进行,不断累加. 

## 解决
解决这里参考了[Pytorch模型测试时显存一直上升导致爆显存](https://blog.csdn.net/dong_liuqi/article/details/119239671), 原因应该是decoder中的梯度在一直计算并累积导致显存的不合理累加. 因此解决方案也比较直接:
```python
with torch.no_grad():
    y = torch.sigmoid(self.refiner(s, features, im_size))
```
将decoder的过程用`torch.no_grad()` wrap起来. bug就解决好了.
