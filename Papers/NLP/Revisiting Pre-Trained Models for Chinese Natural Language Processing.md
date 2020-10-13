# Revisiting Pre-Trained Models for Chinese Natural Language Processing

概述：`回顾`、`分析`了目前所有的预训练语言模型（如：BERT 及其变种等）; 
提出了新的 BERT 模型 `MacBERT` (Mac: MLM as correction)

## 1、回顾预训练模型

### 1.1、BERT

BERT consists of two pre-training
tasks: Masked Language Model (MLM) and Next
Sentence Prediction (NSP). 

### 1.2、ERNIE

对 BERT 的 mask 策略进行了修改，改为：mask 实体、短语。

### 1.3、XLNet

主要针对 BERT mask 策略导致的预训练、微调阶段的数据分布不一致做出修改（训练有 MASK, 
微调的时候没有）

XLNet mainly
modifies in two ways. The first is to maximize the
expected likelihood over all permutations of the
factorization order of the input, where they called
the Permutation Language Model (PLM). Another
is to change the autoencoding language model into
an autoregressive one, which is similar to the traditional
statistical language models.2

### 1.4、RoBERTa (Robustly Optimized BERT Pretraining Approach)

（大力出奇迹系列）

1、比 BERT 更大的 batchsize ，更长的 sentence len ，更多的数据

2、removing the next sentence prediction and using dynamic masking.

## 2、中文预训练模型

### 2.1、BERT-wwm & RoBERTa-wwm

修改了 MASK 方法，对中文不再是单字 MASK, 而是进行分词后对分词后的结果进行 MASK

### 2.2、MacBERT

修改了 MASK 方法，


