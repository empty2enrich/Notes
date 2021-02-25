# 让研究人员绞尽脑汁的Transformer位置编码（苏剑林）

* 绝对位置编码
* 相对位置编码

## 绝对位置编码

* 绝对位置编码对输出长度有限制（输入最大长度训练是多少，使用时就是多少）

没有外推性，但：通过层次分解的方式，可以使得位置编码能外推到足够长的范围（参考苏剑林写的《
层次位置编码》
https://mp.weixin.qq.com/s?__biz=MzIwMTc4ODE0Mw==&mid=2247515573&idx=1&sn=2d719108244ada7db3a535a435631210&chksm=96ea6235a19deb23babde5eaac484d69e4c2f53bab72d2e350f75bed18323eea3cf9be30615b&scene=21#wechat_redirect）


