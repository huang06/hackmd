---
tags: Paper, Deep Learning, CTR, Factorization Machine, Criteo
---

# DeepFM: A Factorization-Machine based Neural Network for CTR Prediction

---

<https://www.ijcai.org/Proceedings/2017/0239.pdf>

2017

---

[TOC]

## 重點注意

1. DeepFM內雖然提到了Feild,但是並沒用到FFM的作法,仍是用FM作法!!

   參考這個issue <https://github.com/ChenglongChen/tensorflow-DeepFM/issues/24>  

   > The filed is referred to feature field, it's not "field-aware".

2. 論文內field(feature field)指的是age,gender,category;feature指的是age=18,gender=male,category=A

## 問題整理

1. 什麼是 end-to-end

   資料的輸入至輸出過程中不需要人為介入,對比wide&deep需要對資料進行手動的特徵工程,deepfm可以直接輸入原始資料

2. 什麼是 field, 和 feature 的差異在哪?

   參考FFM對field的定義.經過one-hot encoding後,gender feature會被拆分為"gender=male", "gender=female",簡單作法我們可以將這二個feature視為同一個field.

## 文章整理

### 1 Introduction

不容易用人為方式捕捉 feature interaction, 只能依賴ML.

> However, most other feature interactions are hidden in data and difficult to identify a priori (for instance, the classic association rule “diaper and beer” is mined from data, instead of discovering by experts), which can only be captured automatically by machine learning. Even for easy-to-understand interactions, it seems unlikely for experts to model them exhaustively, especially when the number of features is large.

**_FTRL_**, 難以捕捉 high-order feature interactions.

> However, a linear model lacks the ability to learn feature interactions, and a common practice is
to manually include pairwise feature interactions in its feature vector. Such a method is hard to generalize to model
high-order feature interactions or those never or rarely appear in the training data [Rendle, 2010]

**_Factorization Machines_**, FM可以對high-order feature interactions建模, 但因為複雜度太高, 實際只用到order-2.

> While in principle FM can model high-order feature interaction, in practice usually only order-2 feature interactions are considered due to high complexity.

**_Wide & Deep_**, 依賴專門的特徵工程.

> the input of “wide part” still relies on expertise feature engineering.

**_DeepFM_** 特點,

- 能對 high-order 和 low-order feature interactions 建模, 且不需要做特徵工程!
- 分享相同的 input 和 embedding vector, 比Wide & Deep有效率

### 2 Our Approach

直接看簡報!

## 源碼/部落格文章參考

### Tensorflow實現

<https://github.com/ChenglongChen/tensorflow-DeepFM>

本來要看DeepCTR的源碼實現,但是作者為了算法能通用在其他模型上,所以不容易讀懂.所以改看另個作者只針對deepfm的算法實現.

### criteo 資料集

下載資料集

```bash
wget https://s3-eu-west-1.amazonaws.com/kaggle-display-advertising-challenge-dataset/dac.tar.gz
```

讀取資料集

```python
# 讀取資料集方法
dfgen = pandas.read_csv('train.txt',
                        chunksize=100,
                        delimiter='\t',
                        header=None,
                        index_col=None)
```
