# Gibbs Sampling

    概述：一种随机采样的方法。生成特定概率分布 p(x) 的样本。
    原理细节后续再看看（https://cosx.org/2013/01/lda-math-mcmc-and-gibbs-sampling）。
    
* Box-Muller 变换 ：采样方法，Box-Muller变换是通过服从均匀分布的随机变量，来构建服从高斯分
布的随机变量的一种方法。

  通过两个随机变量构造正太分布（https://zhuanlan.zhihu.com/p/38638710）。

* 马氏链及其平稳分布：假设状态转移的概率只依赖于前一个状态。

  转移概率确定时，经过有限步之后，分布概率会趋于稳定。
