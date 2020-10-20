# Pytorch 多进程

概述：torch cpu 类型的变量直接使用多进程运行正常，但 cuda 不正常。

使用 torch.multiprocessing 可以使用 cuda 变量。

## torch.multiprocessing 不能在主进程创建 cuda 变量传入多个子进程

[CUDA IPC and the caching allocator] 多个 tensor 会存储在一个 cudaMalloc 
allocation 中，进程间通信会传递整个 cudaMalloc allocation，
但一个 cudaMalloc allocation 每个设备上一次只能在一个进程、一个上下文中打开。
(However, cudaIpcMemHandles from each device in a given process may 
only be opened by one context per device per other process.) 

下面这段代码应该是多个进程同时访问一个 cudaMalloc allocation 导致异常。

```

def print_cuda(queue, event, tensor=None):
  if tensor is None:
    # tensor = torch.cuda.LongTensor(2,3)
    tensor = torch.Tensor(2,3).to("cuda")
  # print(tensor)
  queue.put(tensor)
  event.wait()


def test():
  """
  # 报错 ：RuntimeError: invalid device pointer: 0x7f236be00000 at /pytorch/aten/src/THC/THCCachingAllocator.cpp:301
  """
  import multiprocessing.context.Process

  ctx = mp.get_context("spawn")
  processs = []
  q = [ctx.Queue() for i in range(10)]
  
  e = [ctx.Event() for i in range(10)]
  a = torch.Tensor(2,3).to("cuda")
  a.share_memory_()
  for i in range(10):
    p = ctx.Process(target=print_cuda, args=(q[i], e[i], torch.Tensor(2,3).to("cuda").share_memory_()))
    processs.append(p)
  [p.start() for p in processs]
  res = [i.get() for i in q]

  [i.set() for i in e]
  for p in processs:
    p.join()
``` 
