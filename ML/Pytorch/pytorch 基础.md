# Pytorch 基础

## 目录
[1、基本操作](#基本操作)

[2、常用函数](#常用函数)


## <h2 id="基本操作">1、基本操作</h2>

[1.1、乘法](#1.1乘法)

[1.2、变量赋值](#1.2变量赋值)

[1.3、查找 tensor 中某个值的 indice](#1.3查找tensor中某个值的indice)

[1.4、比较 tensor 元素的值](#1.4比较tensor元素的值)

[1.5、tensor 与 tensor 逐值比较](#1.5tensor与tensor逐值比较)

### <h2 od='1.1乘法'>1.1、乘法</h2>
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

### <h2 id='1.2变量赋值'>1.2、变量赋值</h2>

`tensor` 对象赋值执行的深拷贝。

```
def test():
  a = torch.Tensor([1,2,3]).to("cuda")
  res = []
  for i in range(3):
    a = a - 1
    res.append(a)
  print(res)
# print: [tensor([0., 1., 2.], device='cuda:0'), tensor([-1.,  0.,  1.], device='cuda:0'), tensor([-2., -1.,  0.], device='cuda:0')]
```

### <h2 id='1.3查找tensor中某个值的indice'>1.3、查找tensor中某个值的indice</h2>

```
# 可以通过对值的进行操作，最后使用 nonzero 变相达到目的 （eg: torch.gt 等）
# nonzero 有 as_tuple 参数， True，把结果按维度，每个维度 indice 分别返回， 
# False，一个元素是值的完整位置
torch.nonzero(tensor)

# 查找 tensor 维度 dim 上最大值的 indice
tensor.max(dim).indices
```

eg:

```
a = torch.rand(2,3)
print(a)
print(torch.nonzero(torch.gt(a, 0)))
print(torch.nonzero(torch.gt(a, 0), as_tuple=True))

<!--输出-->
<!--tensor([[0.8651, 0.8329, 0.6063],-->
        <!--[0.8405, 0.3706, 0.7069]])-->
<!--tensor([[0, 0],-->
        <!--[0, 1],-->
        <!--[0, 2],-->
        <!--[1, 0],-->
        <!--[1, 1],-->
        <!--[1, 2]])-->
<!--(tensor([0, 0, 0, 1, 1, 1]), tensor([0, 1, 2, 0, 1, 2]))-->


```

### <h2 id ='1.4比较tensor元素的值'>1.4、比较tensor元素的值</h2>

同 1.5

```

```


### <h2 id ='1.5tensor与tensor逐值比较'>1.5、tensor与tensor逐值比较</h2>

`注：` another tensor 可以直接是一个值，不用是 tensor 类型

```
# 大于
torch.gt(tensor, another_tensor)
# 大于等于
torch.ge(tensor, another_tensor)
# 等于
torch.equal(tensor, another_tensor)
```

## <h2 id="常用函数">2、常用函数</h2>

[2.1、torch.gather](#gather)

[2.2、tensor.max(dim) or torch.max(tensor, dim)](#max)

### <h2 id="gather">2.1、torch.gather</h2>

torch.gather(input, dim, index, *, sparse_grad=False, out=None)

概述：在指定的维度 dim 上按 index 把 input 对应位置的值拿出来。

```
t = torch.tensor([[1, 2], [3, 4]])
torch.gather(t, 1, 
    torch.tensor([[0, 0], [1, 0]])
<!--输出：-->
<!--[[1, 1], [4, 3]]-->
<!--dim = 1 表示按列上取值，index 的第一个数组 [0, 0]中 0 表示拿第一行的 第一个值，-->
<!--[1, 0] 表示拿第二行的第 2 个值、第一个值-->
```


### <h2 id="max">tensor.max(dim) or torch.max(tensor, dim)</h2>

概述：返回指定维度上的最大值和indice。 torch.max 与 tensor.max 作用一样

```
t = torch.rand(5,4,3)
t_max_value, t_max_indice = t.max(1)
# t_max_value, t_max_indice = torch.max(t, 1)
print(t)
print(t_max_value)
print(t_max_indice)
<!--输出-->
<!--tensor([[[0.9781, 0.3100],-->
         <!--[0.2625, 0.7258],-->
         <!--[0.9424, 0.6402]],-->

        <!--[[0.2065, 0.2170],-->
         <!--[0.1077, 0.5661],-->
         <!--[0.9323, 0.5006]],-->

        <!--[[0.1270, 0.3339],-->
         <!--[0.6908, 0.7654],-->
         <!--[0.5647, 0.4904]],-->

        <!--[[0.6268, 0.9114],-->
         <!--[0.5455, 0.6367],-->
         <!--[0.6343, 0.0648]]])-->
<!--tensor([[0.9781, 0.7258],-->
        <!--[0.9323, 0.5661],-->
        <!--[0.6908, 0.7654],-->
        <!--[0.6343, 0.9114]])-->
<!--tensor([[0, 1],-->
        <!--[2, 1],-->
        <!--[1, 1],-->
        <!--[2, 0]])-->

```