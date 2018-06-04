---
title: PyTorch Load Pretrained Model
date: 2018-06-04 17:00:00
categories:
- DeepLearning
tags:
- PyTorch
- DeepLearning
mathjax: true
---

<center>加载（整个/部分）预训练模型。</center>

<!-- more -->

当前代码在PyTorch 0.4.0版本下编写。

## 加载整个模型

### 方法1

```python
# 保存
torch.save(model, PATH)
# 加载
model = torch.load(PATH)
```

### 方法2

```python
# 保存
torch.save(model.state_dict(), PATH)
# 加载
model = ModelClass() # 初始化
model.load_state_dict(torch.load(PATH))
```

## 加载部分模型

```python
# 预训练模型
pretrained_dict = torch.load('pretrained_state_dict')
# 或者 pretrained_dict = torch.load('pretrained_modek').state_dict()
# 待加载模型
model_dict = model.state_dict() 
# 从预训练模型的state_dict中筛选出所需层和参数的key-value对
pretrained_dict = {k: v for k, v in X2dict.items() if ('$筛选条件')}
model_dict.update(pretrained_dict)
model.load_state_dict(model_dict)
```

### 加载DataParallel后的模型

有时会遇到这样的情况：

训练模型用的是双卡：

```python
model = torch.nn.DataParallel(self.model, device_ids=[0, 1])
torch.save(model.state_dict(), PATH)
```

加载模型的时候用的是单卡，就会出现无法load的情况。解决方案：

```python
state_dict = torch.load(PATH) # state_dict path
# create new OrderedDict that does not contain `module.`
new_state_dict = OrderedDict()
for k, v in state_dict.items():
    namekey = k[7:]  # remove `module.`
    new_state_dict[namekey] = v
model.load_state_dict(new_state_dict)
```

或者<a href="https://izhaolei.github.io/bug%E9%9B%86%E5%90%88/2018/01/21/linux-%E9%97%AE%E9%A2%98%E9%9B%86%E5%90%88/#">iZhaolei</a>说在多卡训练时不`model = torch.nn.DataParallel(model, device_ids=[0, 1])`，

而`output = torch.nn.parallel.data_parallel(self.model, input)`

