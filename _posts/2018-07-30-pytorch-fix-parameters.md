---
title: To Fix Some Layers' Parameters
date: 2018-07-30 15:23:00
categories:
- DeepLearning
- PyTorch
tags:
- PyTorch
mathjax: true
---

<center>固定模型的一部分参数而训练另一部分。</center>

<!-- more -->

### PyTorch

PyTorch模型的参数保存在`state_dict`内，换言之，以键值对的形式保存。因此在定义模型的时候，每一层的名字都应该合理选取，在这样的前提下，可以根据键来对模型参数赋`required_grad`属性。

```python
def fix(model):
    for key, params in model.state_dict.items():
        if 'conv1' in key: # 固定conv1的所有参数
            params.requires_grad = False
        else:
            params.requires_grad = True
```

只将`requires_grad=True`的参数传入`optimizer`，例如：

```python
oprimizer = torch.optim.SGD(filter(lamda p : p.requires_grad, model.parameters()), lr=1e-3, momentum=0.9)
```

