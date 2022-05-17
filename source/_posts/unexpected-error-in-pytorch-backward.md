---
title: unexpected error in pytorch backward
date: 2022-05-18 01:56:43
categories:
- pytorch
tags:
- python
- pytorch
---

在写模型训练的时候遇到奇怪的报错:
```
RuntimeError: Trying to backward through the graph a second time, but the saved intermediate results have already been freed. Specify retain_graph=True when calling backward the first time.
```
参考一些网上的[解答](https://discuss.pytorch.org/t/runtimeerror-trying-to-backward-through-the-graph-a-second-time-but-the-buffers-have-already-been-freed-specify-retain-graph-true-when-calling-backward-the-first-time/6795)和报错提示,意思就是执行backward的时候,中间结果已经被释放了.一开始比较纳闷在于,我仅仅增加了一些卷积层和操作在train方法里面,怎么突然就会进行该报错呢,并且中间结果都是在train方法中,不会出现backward的时候已经被释放.

经过仔细审查,有个地方存在比较大的问题, 在我的代码出有这样一个逻辑
```python
for j, batch in enumerate(loader, 1):
    self.optimizer.zero_grad()
    # pre-handle some thing
    for i in range(1, len(images)):
        s = self.model(images[i])
        y = label[i]
        acc = self.compute_accuracy(s.detach(), y)
        loss = self.compute_loss(s, y)
        loss.backward()
    self.optimizer.step()
```
上面这段代码看起来没啥问题, 但是注意如果pre-handle部分存在有计算图的梯度计算, 这一部分中间结果在loss.backward的时候就已经free掉了,由此会导致一开始的那个报错,而我正好在这一部分作了一个类似下面的操作:
```python
for j, batch in enumerate(loader, 1):
    self.optimizer.zero_grad()
    # pre-handle some thing
    # warning: 注意这一步
    self.prev_ft = self.feature_extractor(images[0])
    for i in range(1, len(images)):
        s = self.model(images[i])
        y = label[i]
        acc = self.compute_accuracy(s.detach(), y)
        loss = self.compute_loss(s, y)
        loss.backward()
    self.optimizer.step()
```
上述使用`feature_extractor`抽取`images[0]`的特征, 因为不是处于torch.no_grad()的wrap环境下,因此会进行梯度计算. 解决方案也很简单, 将pre-handle的部分用torch.no_grad()包括起来即可.
```python
for j, batch in enumerate(loader, 1):
    self.optimizer.zero_grad()
    # pre-handle some thing
    with torch.no_grad():
        self.prev_ft = self.feature_extractor(images[0])

    for i in range(1, len(images)):
        s = self.model(images[i])
        y = label[i]
        acc = self.compute_accuracy(s.detach(), y)
        loss = self.compute_loss(s, y)
        loss.backward()
    self.optimizer.step()
```
