# Pytorch 多进程

概述：torch cpu 类型的变量直接使用多进程运行正常，但 cuda 不正常。

使用 torch.multiprocessing 可以使用 cuda 变量。

## torch.multiprocessing 在主进程创建 cuda 变量传入多个子进程报错

[CUDA IPC and the caching allocator] 多个 tensor 会存储在一个 cudaMalloc 
allocation 中，进程间通信会传递整个 cudaMalloc allocation，
但一个 cudaMalloc allocation 每个设备上一次只能在一个进程、一个上下文中打开。
(However, cudaIpcMemHandles from each device in a given process may 
only be opened by one context per device per other process.) 

问题原因：下面这段代码应该是多个进程同时访问一个 cudaMalloc allocation 导致异常。

`解决`：在 `queue.put(tensor)` 之前使用 `torch.tensor(tensor)`

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

## torch.multiprocessing 使用 model

概述：子进程与主进程直接通信还是用 queue， model 传入子进程之前需要 
`model.share_memory()` （否则程序报错），子进程 model 的输出在返回前需要使用 
`torch.tensor(tensor)` 深拷贝下，直接返回程序会卡死。

`注：` `queue.get()` 方法需要在 `process.start()` 后就调用。

```
def test_model_mul_pro(bert_model, queue, event):
  text = "微分非官方vert"
  embedding = bert_model.encode([text], batch_size=1)
  embedding = torch.tensor(embedding)
  queue.put(embedding)
  # print(embedding)
  event.wait()
  

def test_model_init_extractor(samples_in_db, file_contents, template_config, static_sample_file,
                   bert_path, bert_vocab_path, device_ids, elmo_data_pth,
                   ner_model_path, use_cuda, label_mode, label_file_path,
                   batch_size, max_str_len, bert_embedding, rnn_hidden,
                   rnn_layer, dropout_ratio, dropout1, jieba_dict_path,
                   encoder_batch_size, updated_busi=[]):
  ctx = mp.get_context("spawn")
  extractor_factory = {}
  bert_encoder, elmo, ner_model = init_model(bert_path, bert_vocab_path, device_ids, elmo_data_pth,
                   ner_model_path, use_cuda, label_mode, label_file_path,
                   batch_size, max_str_len, bert_embedding, rnn_hidden,
                   rnn_layer, dropout_ratio, dropout1)
  busi_samples = init_samples_from_db(samples_in_db, file_contents, template_config)
  extract_feature_flag = True if updated_busi else False
  reference_all = init_samples(busi_samples, static_sample_file, ner_model,
                               bert_encoder, elmo, extract_feature_flag, encoder_batch_size)
  # templates = generate_template({k:v for k,v in busi_samples.items() if k in updated_busi})


  # 多进程
  bert_encoder.share_memory()
  # elmo.share_memory()
  # ner_model.share_memory()
  queue_len = 2
  queues = [ctx.Queue() for i in range(queue_len)]
  events = [ctx.Event() for _ in range(queue_len)]
  processs = []

  for i in range(queue_len):
    p = ctx.Process(target=test_model_mul_pro, args=(bert_encoder,  queues[i], events[i]))
    processs.append(p)

  [p.start() for p in processs]

  res = [q.get() for q in queues]

  print("result:  =========================== ")
  print(res)

  [e.set() for e in events]

  [p.join() for p in processs]

  return extractor_factory
```
