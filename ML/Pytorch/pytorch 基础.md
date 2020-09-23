# Pytorch 基础

## 目录
[1、基本操作](#基本操作)


## <h2 id="基本操作">1、基本操作</h2>

### 1.1、乘法
* torch.mul (点积)
```
# 两矩阵对应位相乘， size 必须一致， example:
import torch
a = torch.randn(2,3)
b = torch.randn(2,3)
tor.mul(a,b) # 结果 size: 2,3
```
* torch.mm torch.matmul (矩阵乘法)

<font color=FF0000>注：torch.mm 只能接受二维矩阵, torch.matmul 可接受多维数组</font>

```
import torch
a = torch.randn(2,3)
b = torch.randn(3,2)
tor.mm(a,b) # 结果 size: 2,2
```


