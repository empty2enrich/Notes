# Pytorch 多进程

概述：torch cpu 类型的变量直接使用多进程运行正常，但 cuda 不正常。

使用 torch.multiprocessing 可以使用 cuda 变量。

## torch.multiprocessing 不能在主进程创建 cuda 变量传入多个子进程

[CUDA IPC and the caching allocator] 多个 tensor 会存储在一个 cudaMalloc 
allocation 中，进程间通信会传递整个 cudaMalloc allocation，
但一个 cudaMalloc allocation 每个设备上一次只能在一个进程、一个上下文中打开。
(However, cudaIpcMemHandles from each device in a given process may 
only be opened by one context per device per other process.) 

问题原因：下面这段代码应该是多个进程同时访问一个 cudaMalloc allocation 导致异常。

`queue.put(tensor)` 注释掉代码就没有问题，说明是 queue 访问数据导致出错了。
但在 `queue.put(tensor)` 之前使用 `torch.tensor(tensor)` （复制原 tensor）就不会报错了。

torch.multiprocessing 在子进程中使用 `tensor` 计算运行正常，但没有返回值，
需要提供 `ctc.Queue()` 把返回值传回主进程

将传入子进程的 `tensor` deepcopy 下再使用 `queue.put(tensor)` 运行正常，
但主进程 `queue.get()` 的时候报错。 
`THCudaCheck FAIL file=/pytorch/aten/src/THC/THCCachingAllocator.cpp line=625 error=30 : unknown error`

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
