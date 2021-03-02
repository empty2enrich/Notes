# TensorFlow 常见异常

## 多线程异常

异常描述：（tf 1.13.2 gpu）

```
在主进程初始化 model，然后把 model 传到子进程中去调用，抛出异常。

error log:

```

解决：在主进程初始化 model 后调用一次 model，然后传入子进程，就正常了。