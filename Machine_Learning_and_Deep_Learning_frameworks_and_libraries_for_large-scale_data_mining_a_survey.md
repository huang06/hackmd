---
tags: Paper, Deep Learning, Machine Learning, Spark
---

# Machine Learning and Deep Learning frameworks and libraries for large-scale data mining: a survey

---

<https://doi.org/10.1007/s10462-018-09679-z>

Published online: 19 January 2019

Twitter/KirkDBorne 推薦

有助於了解目前ML/DL的框架進展,優缺點.

---

[TOC]

## 1. Introduction

Data Mining 的定義

> Data mining (DM) is the core stage of the knowledge discovery process that aims to extract interesting and potentially useful information from data (Goodfellow et al. 2016; Mierswa 2017).

AI 的定義

> Artificial Intelligence (AI) is any technique that aims to enable computers to mimic human behaviour, including machine learning, natural language processing (NLP), language synthesis, computer vision, robotics, sensor analysis, optimization and simulation.

Machine Learning 的定義

> Machine Learning (ML) is a subset of AI techniques that enables computer systems to learn from previous experience (i.e. data observations) and improve their behaviour for a given task. ML techniques include Support Vector Machines (SVM), decision trees, Bayes learning, k-means clustering, association rule learning, regression, neural networks, and many more.

Neural Networks 的定義

> Neural Networks (NNs) or artificial NNs are a subset of ML techniques, loosely inspired by biological neural networks. They are usually described as a collection of connected units, called artificial neurons, organized in layers.

Deep Learning 的定義

> Deep Learning (DL) is a subset of NNs that makes the computational multi-layer feasible. Typical DL architectures are deep neural networks (DNNs), convolutional neural networks (CNNs), recurrent neural networks (RNNs), generative adversarial networks (GAN), and many more.

## 2. Machine Learning and Deep Learning

## 2.1 Machine Learning process

Data Mining 經典流程 - CRISP-DM

![](https://i.imgur.com/bulXi3x.png)

1. business understanding
2. data understanding
3. data preparation
4. modelling phase
5. evaluation phase
6. deployment phase

## 2.2 Neural Networks and Deep Learning

介紹經典架構

## 3. Accelerated computing

硬體加速包含 GPU, FPGA, TPU

新的schema有利於減少記憶體用量,加速運算:

1. Sparse computation

    > Sparse computation, which imposes the use of sparse representations along the neural network. Benefits include lower memory requirements and faster computation.

2. Low precision data types

    > Low precision data types (Konsor 2012), smaller than 32-bits (e.g. half-precision or integer) with experimentation even with 1-bit computation (Courbariaux et al. 2016). Again this speeds up algebra calculation as well as greatly decreasing memory consumption at the cost of a slightly less accurate model (Iandola et al. 2016; Markidis et al. 2018). In these recent years, most DL frameworks are starting to support 16-bit and 8-bit computation (Harris 2016; Andres et al. 2018).

---

作者提到發展大規模的DL仍受限於GPU記憶體大小(NVIDIA Volta 上限是32GB).

解決方法: Multi-GPU, distributed-GPU, 進而有了 data parallelism, model parallelism 延伸研究.

---

相關的加速套件(accelerated libraries)

> Manufacturers often offer the possibility to
enhance hardware configuration with many-core accelerators to improve machine/cluster
performance as well as accelerated libraries, which provide highly optimized primitives,
algorithms and functions to access the massively parallel power of GPUs.

- NVIDIA CUDA
- NVIDIA cuDNN
- Intel MKL
- OpenCL
- AMD ROCm
- OpenMP
- Open MPI

同時結合 OpenMP 和 OpenMPI 的應用

> OpenMP is used for parallelism within a (multi-core) node while MPI is used for parallelism between nodes.

## 4. Machine Learning frameworks and libraries

作者列出許多ML/DL開源套件.

## 4.2 Deep Learning frameworks and libraries

## 4.2.12 Deep Learning wrapper libraries

大公司其實都有提出轉換prototype至production的方案.

1. Using ***Keras*** for fast prototyping and ***TensorFlow*** for production. This trend is backed by Google.

2. Using ***PyTorch*** for prototyping and ***Caffe2*** for production. This trend is backed by Facebook.

## 4.3 Machine Learning and Deep Learning frameworks and libraries with MapReduce

## 4.3.1 Deeplearning4j

優點:

JAVA

缺點:

Java/Scala並非DL/ML的主流語言

H2O似乎比較受歡迎

## 4.3.2 Apache Spark MLlib and Spark ML

作者提醒,在Spark上開發算法需要非常了解**分散式環境,資料管理,行程管理,以及開發技術**.

> It is important to notice that the implementation of a seemingly simple algorithm (e.g. distributed multi-label kNN) for large-scale data mining is not trivial (Ramirez-Gallego et al. 2017; Gonzalez-Lopez et al. 2018). It requires deep understanding about underlying scalable and distributed environment (e.g. Apache Spark), its data and processing management as well as programming skills. Therefore, ML algorithms for large-scale data mining are different in complexity and implementation from general purpose ones.

優點:

許多套件

in-memory process

缺點:

主要處理tabular data

記憶體消耗

MLlib/ML仍還屬於發展階段

## 4.3.3 H2O, Sparkling Water and Deep Water

***Sparkling Water***: 結合Spark

***Deep Water***: TensorFlow, MXNet, and Caffe.

優點:

許多業界使用

基於基礎建設優化算法

目標可以讓ML/DL的流程可以透過UI界面自動化

缺點:

– UI flow, the web-based UI for H2O does not support direct interaction with Spark.

– H2O is more general purpose and aims at a different problem in comparison with (specific)
DL libraries e.g., TensorFlow or DL4j.

## 5. Conclusions

1. 大部分的framework都由企業或是研究單位建構.
2. 框架都有提供 high level Deep Learning wrapper.
3. Apache Spark, Apache Flink and Cloudera Oryx 2 仍在發展中.
4. Vertical scalability for large-scale DL 受限於記憶體大小; horizontal scalability 受限於節點之間的latency.
5. 包含傳統工具都能處理大規模的資料量.
6. Python是主流,主流工具都會支持Python,但作者沒提到和原生語言的效能差異多大.
7. 未來趨勢是工具會提供互動式分析/視覺化工具支援決策.
