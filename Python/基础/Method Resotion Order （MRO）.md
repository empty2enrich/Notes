# Method Resotion Order （MRO）

Method Resotion Order （MRO）：多重继承方法解析顺序。

* 经典 MRO
* python2 新式 MRO
* C3

# 经典 MRO

使用从左往右深度优先算法，有重复的保留最后出现的。

# python2 新式 MRO

使用从左往右广度优先算法，但存在“二义性”， 可能出现子类、父类 MRO 不一致情况（相同部分）。

# C3

C3 与 `python2 新式 MRO` 使用相同的从左往右广度优先算法，但加上“二义性”校验，不允许定义
存在二义性的继承。