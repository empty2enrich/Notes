# signal

子进程对特定信号的处理，都是在主进程调用该处理代码。

Python 信号处理程序总是在主 Python 线程中执行，
即使信号是在另一个线程中接收的。这意味着信号不能用作线程间通信的手段。 你可以使用 threading 模块中的同步原函数。

## 一些应用

* 可以使用信号控制方法超时时间

* signal.pthread_kill(thread_id, signalnum) 

杀进程；检查进程是否存活；Unix  系统；