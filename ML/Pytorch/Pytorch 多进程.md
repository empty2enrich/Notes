# Pytorch 多进程

概述：torch cpu 类型的变量直接使用多进程运行正常，但 cuda 不正常。

使用 torch.multiprocessing 可以使用 cuda 变量。

`解决问题`：
    
* 1、queue 返回 tensor 报错，执行下 `tensor=tensor`， torch 的 tensor 作为
基础数据类型，执行赋值操作后会生成一个新的变量，对原 tensor 操作会导致报错，
进行赋值操作后就正常了

目录：

[1、torch.multiprocessing 在主进程创建 cuda 变量传入多个子进程报错](#tensor进程间通信)

[2、torch.multiprocessing 使用 torch model 对象](#model)

[3、torch.multiprocessing 使用 hdf5](#hdf5)

## <h2 id="tensor进程间通信">torch.multiprocessing 在主进程创建 cuda 变量传入多个子进程报错</h2>

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
# 通信报错信息：
# 处理 ：RuntimeError: invalid device pointer: 0x7f236be00000 at /pytorch/aten/src/THC/THCCachingAllocator.cpp:301
```

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

## <h2 id="model">torch.multiprocessing 使用 model</h2>

概述：子进程与主进程直接通信还是用 queue， model 传入子进程之前需要 
`model.share_memory()` （否则程序报错），子进程 model 的输出在返回前需要: 
`torch.tensor(tensor)` 深拷贝下，直接返回程序会卡死； 
embedding 不拷贝，放到一个字典中也行; `embedding=embedding` 也行。

`注：` `queue.get()` 方法需要在 `process.start()` 后就调用。

```
def test_model_mul_pro(bert_model, queue, event):
  text = "微分非官方vert"
  embedding = bert_model.encode([text], batch_size=1)
  
  # 方式一
  # embedding = torch.tensor(embedding)
  # queue.put(embedding)
  
  # 方式二
  # queue.put({"emb": embedding})
  
  #方式三
  embedding = embedding
  queue.put(embedding)
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

## <h2 id="hdf5">在子进程中打开 hdf5 文件报错</h2>

### 使用中碰到的问题

#### TypeError: __cinit__() takes exactly 1 positional argument (0 given)

解决：

    * 1、把 h5py 改为 h5pickle
    
之前的问题解决了，但 h5group 获取不到 name, 导致报错。

好像是包的问题，修改下源码看看， 结果： 解决问题，源码修改在下面

```
def get_h5data():
  import h5pickle as h5py
  model_dir = "/home/data/nlp_model/zhs.model"
  with h5py.File(os.path.join(model_dir, 'dic.hdf5.bk'), 'a') as f_emb:
    dic_hdf5 = None
    if list(f_emb.keys()) == []:
      dic_hdf5 = f_emb.create_group('word_emb')
    else:
      dic_hdf5 = f_emb['word_emb']
    return f_emb, dic_hdf5

def process_h5(data, queue, event):
  print("Finish")

def test_hdf5_file():
  ctx = mp.get_context("spawn")
  data = get_h5data()
  queue_len = 1
  queues = [ctx.Queue() for i in range(queue_len)]
  events = [ctx.Event() for _ in range(queue_len)]
  processs = []

  for i in range(queue_len):
    p = ctx.Process(target=process_h5,
                    args=(data, queues[i], events[i]))
    processs.append(p)

  [p.start() for p in processs]

  res = [q.get() for q in queues]

  print("result:  =========================== ")
  print(res)

  [e.set() for e in events]

  [p.join() for p in processs]

  return ""
```


修改 h5pickel 的源码, 解决问题。(`注：这里改了不报错了，但后面引起了其他问题，不能这么改`)

```
# h5pickle.__init__.py  line 69,修改了 line 69~73

 59 class PickleAbleH5PyObject(object):
 60     """Save state required to pickle and unpickle h5py objects and groups.
 61     This class should not be used directly, but is here as a base for inheritance
 62     for Group and Dataset"""
 63     def __getstate__(self):
 64         """Save the current name and a reference to the root file object."""
 65         return {'name': self.name, 'file': self.file_info}
 66 
 67     def __setstate__(self, state):
 68         """File is reopened by pickle. Create a dataset and steal its identity"""
 69         if state['name']:
 70           self.__init__(state['file'][state['name']].id)
 71         else:
 72           name = state['file'].name
 73           self.__init__(state['file'][name].id)
 74         self.file_info = state['file']
 75 
 76     def __getnewargs__(self):
 77         """Override the h5py getnewargs to skip its error message"""
 78         return ()

```

#### 又出现 `RuntimeError: invalid device pointer`

```
/home/miniconda3/lib/python3.6/site-packages/field_extraction/pipeline/field_extraction.py:61: ResourceWarning: unclosed file <_io.BufferedReader name='/home/data/nlp_model/data/dict.txt'>
  jieba.load_userdict(jieba_dict_path)
Traceback (most recent call last):
  File "/home/miniconda3/lib/python3.6/multiprocessing/queues.py", line 234, in _feed
    obj = _ForkingPickler.dumps(obj)
  File "/home/miniconda3/lib/python3.6/multiprocessing/reduction.py", line 51, in dumps
    cls(buf, protocol).dump(obj)
  File "/home/miniconda3/lib/python3.6/site-packages/torch/multiprocessing/reductions.py", line 213, in reduce_tensor
    (device, handle, storage_size_bytes, storage_offset_bytes) = storage._share_cuda_()
RuntimeError: invalid device pointer: 0x7fec72000000 at /pytorch/aten/src/THC/THCCachingAllocator.cpp:301
```

概述：这个错误与 CUDA 类型的 tensor 使用 queue 传递有关, 

结论：`经调查，是把 model 传到子进程再传回来报错的`, 返回 model 之前 `copy.deepcopy()`
下，解决这个问题。

* 可能是 jieba 的问题，测试结果：不是
* 可能是 reference 的问题，测试结果：不是
* 可能是 BM25 的问题，测试结果：不是
* 最后通过注释查到是 NER 模型的问题(不光是 ner 的问题，除了 ner 剩下的也有些有问题,
如下的，可能都有问题) 结果：都不是
    * _ner_model
    * _lang_model
    * _elmo
    * bm25_model
    * _JIEBA_DEFAULT_DICT
    * _para_extractor
    * _wrd_extractor
    * _sen_extractor
    * _entity_extrator
    * _doc_extract
    
#### 出现 `ValueError: Not a location`

```
2020-10-28 11:22:15,187 - nlp_extract_service - ERROR - start_queue - Extract consumer error: Traceback (most recent call last):
  File "/home/zbd/ll/ffes/python/nlp_extract_service/nlp_extract_server.py", line 397, in start_queue
    extractor_factory = init_extractor_update_template()
  File "/home/zbd/ll/ffes/python/nlp_extract_service/init_extractors.py", line 280, in init_extractor_update_template
    encoder_batch_size, updated_busi)
  File "/home/zbd/ll/ffes/python/nlp_extract_service/init_extractors.py", line 212, in init_extractor
    res = [queue.get() for queue in queues]
  File "/home/zbd/ll/ffes/python/nlp_extract_service/init_extractors.py", line 212, in <listcomp>
    res = [queue.get() for queue in queues]
  File "/home/zbd/miniconda3/lib/python3.6/multiprocessing/queues.py", line 113, in get
    return _ForkingPickler.loads(res)
  File "/home/zbd/miniconda3/lib/python3.6/site-packages/h5pickle/__init__.py", line 70, in __setstate__
    self.__init__(state['file'][state['name']].id)
  File "/home/zbd/miniconda3/lib/python3.6/site-packages/h5pickle/__init__.py", line 136, in __getitem__
    obj = h5py_wrap_type(h5py.Group.__getitem__(self, name))
  File "h5py/_objects.pyx", line 54, in h5py._objects.with_phil.wrapper
  File "h5py/_objects.pyx", line 55, in h5py._objects.with_phil.wrapper
  File "/home/zbd/miniconda3/lib/python3.6/site-packages/h5py/_hl/group.py", line 262, in __getitem__
    oid = h5o.open(self.id, self._e(name), lapl=self._lapl)
  File "h5py/_objects.pyx", line 54, in h5py._objects.with_phil.wrapper
  File "h5py/_objects.pyx", line 55, in h5py._objects.with_phil.wrapper
  File "h5py/h5o.pyx", line 190, in h5py.h5o.open
ValueError: Not a location (invalid object ID)

```

概述：感觉是 h5py 对象深拷贝后导致的，测试结果：好像不是，可能是 h5py 对象出 bug 
改源码后不对的，把源码先改回去。 h5py torch 多进程有问题。h5pickle 传 h5py 对象没问题，
但传 `.f_emb.create_group('word_emb')` 的对象报错，把这个 group 对象从成员变量
里删掉就可以了。

* 情形一、hdf5 文件存在，打开文件访问某个属性抛出此问题

通过调查，发现在一个进程多处打开同一个 hdf5 文件，刚开始是正常的，到了后面会打不开，访问文件属性，抛出此问题。
