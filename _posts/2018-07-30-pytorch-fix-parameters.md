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

```python
def fix(model):
    for index, params in enumerate(model.parameters()):
        if index in [$layers you want to fix]:
            params.requires_grad = False
```

```python
def fix(model):
    print(model._modules.keys())
    # 以resnet18为例，输出为：
    # odict_keys(['conv1', 'bn1', 'relu', 'maxpool', 'layer1', 'layer2', 'layer3', 'layer4', 'avgpool', 'fc'])
    # 若要fix住conv1的参数：
    for params in model._modules['conv1'].parameters():
        params.requires_grad = False
    # 若fix住bn1的参数
    for params in model._modules['bn1'].parameters():
        params.requires_grad = False
        model._modules['bn1'].eval()1
```



只将`requires_grad=True`的参数传入`optimizer`，例如：

```python
oprimizer = torch.optim.SGD(filter(lamda p : p.requires_grad, model.parameters()), lr=1e-3, momentum=0.9)
```

