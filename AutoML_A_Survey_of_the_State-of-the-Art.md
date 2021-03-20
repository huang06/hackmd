---
tags: Paper, AutoML, NAS
---

# AutoML: A Survey of the State-of-the-Art

---

<https://arxiv.org/abs/1908.00709>

last revised 8 Jul 2020 (this version, v5)]

---
[TOC]

## Abstract

> We introduce AutoML methods according to the pipeline, covering data preparation, feature engineering, hyperparameter optimization, and neural architecture search (NAS). We focus more on NAS, as it is currently very hot sub-topic of AutoML.

## 1. Intorduction

### AutoML

> There are various deﬁnitions of AutoML. For example, according to [8], AutoML is designed to reduce the demand for data scientists and enable domain experts to automatically build ML applications without much requirement for statistical and ML knowledge. In [9], AutoML is defined as a combination of automation and ML.

> **In a word, AutoML can be understood to involve the automated construction of an ML pipeline on the limited computational budget.**

![](https://i.imgur.com/h9rr5Sa.png)

### NAS

> NAS consists of three important components: the search space of neural architectures, AO methods, and model estimation methods.

> NAS aims to search for a robust and well-performing neural architecture by selecting and combining different basic operations from a predeﬁned search space.

## 2. Data Preparation

![](https://i.imgur.com/qkk4yU6.png)

## 2.1. Data Collection

> However, it is usually challenging to ﬁnd a proper dataset through the above approaches for some particular tasks, such as those related to medical care or other private matters. Two types of methods are proposed to solve this problem: data searching and data synthesis.

### 2.1.1. Data Searching

> However, there are some problems with using Web data.

1. filter unrelated data.
2. Web data may be incorrectly labeled or even unlabeled.

### 2.1.2. Data Synthesis

> For some particular tasks, such as autonomous driving, it is not possible to test and adjust a model in the real world during the research phase, due to safety hazards. Therefore, a practical approach to generating data is to use a data simulator that matches the real world as closely as possible.

- OpenAI Gym
- Game engine
- Generative Adversarial Networks (GANs)

## 2.2. Data Cleaning

> BoostClean [84] attempts to automate this process by treating it as a boosting problem.

> AlphaClean [85] transforms data cleaning into a hyper-parameter optimization problem.

## 2.3. Data Augmentation

![](https://i.imgur.com/r4odt8y.png)

> The above augmentation techniques still require human to select augmentation operations and then form a speciﬁc DA policy for speciﬁc tasks, which requires much expertise and time. Recently, there are many methods [99, 100, 101, 102, 103, 104, 105, 106, 107, 108, 109] proposed to search for augmentation policy for different tasks.

> In order to improve search efficiency, a number of improved algorithms have subsequently been proposed using different search strategies, such as gradient descent-based [100,101],Bayesian-based optimization [102], online hyper-parameter learning [108], greedy-based search [103] and random search[106].

## 3. Feature Engineering

> Feature engineering consists of three sub-topics
> 1. feature selection
> 2. feature extraction
> 3. feature construction

> Feature extraction and construction are **variants of feature transformation**, by which a new set of features is created [110].

feature selection: 刪除不必要的特徵

feature extraction: 減少特徵維度

feature construction: 擴充特徵維度

> In most cases, **feature extraction** aims to reduce the dimensionality of features by applying specific mapping functions, while **feature construction** is used to expand original feature spaces, and the purpose of **feature selection** is to reduce feature redundancy by selecting important features. Thus, the essence of automatic feature engineering is, to some degree, a dynamic combination of these three processes.

## 3.1. Feature Selection

挑選特徵的子集合。

> Feature selection builds a feature subset based on the original  feature  set  by  reducing  irrelevant  or  redundant features.

According to [111], there are four basic steps in a typical process of feature selection (see Figure 5), as follows:

![](https://i.imgur.com/GjakqGC.png)

> The **search strategy** for feature selection involves three types of algorithms:  **complete search, heuristic search, and random search.**

> The most commonly used random search methods are simulated annealing (SA) and genetic algorithms (GAs).

> Methods of subset evaluation can be divided into threedifferent categories...

## 3.2. Feature Construction

增加新的特徵。

> Feature construction is a process that constructs new features from the basic feature space or raw data to enhance the robustness and generalizability of the model.

- standardization
- normalization
- feature discretization
- decision tree-based methods [114,113]
- enetic algo-rithms [115]
- annotation-based approaches [116]
- ...

> Then,  after  selecting  possible  operations and constructing a new feature,  feature-selection techniques are applied to measure the new feature.

## 3.3. Feature Extraction

特徵降維。

> Feature extraction is a dimensionality-reduction process performed via some mapping functions. It extracts informative and non-redundant features according to certain metrics.

不像feature selection挑選子集，feature extraction則是改變原始特徵。

> Unlike feature selection, feature extraction altersthe original features.

- principal component analysis (PCA)
- independent component analysis
- isomap
- nonlinear dimensionality reduction
- linear discriminantanalysis (LDA)
- autoencoder-based algorithms

## 4. Model Generation

![](https://i.imgur.com/LOk5PjF.png)

- **Search Space.**  The search space defines the design principles of neural architectures.  Different scenarios require different search spaces.  We summarize four commonly used search space, including entire-structured, cell-based, hierarchical, and morphism-based search space.
- **Architecture Optimization Method.**  The AO method defines how to guide the search to efficiently find the model architecture with high performance after thesearch space is defined.
- **Model Estimation Method.**  Once a model is generated,its performance needs to be evaluated.  The simplestway is to train the model to convergence on the training set, and then do the evaluation on the validation set,  whereas  such  method  is  time-consuming  and resource-intensive.  Some advanced methods maybe accelerate the process of estimation but loss fidelity. Thus, how to balance efficiency and effectiveness of evaluation is a problem worth studying.

## 4.1. Search Space

TBD

## 4.2. Architecture Optimization

動機: 傳統是透過ValidationSet的表現來調整網路結構的超參數，但需要依賴專家以及時間資源進行試錯。

> Traditionally, the architecture of a neural network is regarded as a set of static hyper-parameters that are tuned based on the performance observed on the validation set.  However, this process highly depends on human experts and requires a lot of time and resources for trial and error. 

### 4.2.1. Evolutionary Algorithm

![](https://i.imgur.com/1Gnfgg0.png)

- Encoding Scheme
- Steps
  - Selection: 從候選集中選擇子集
        - fitness selection
        - rank selection
        - tournament selection
    - Crossover: 產生子代
    - Mutation: 變異
    - Update: 刪除不要的網路

### 4.2.2. Reinforcement Learning

![](https://i.imgur.com/CiUvcHY.png)

> **The agent** is usually a recurrent neural network (RNN) that executes an action $A_t$ at each step $t$ to sample a new architecture from the search space and receives an observation of the state $S_t$ together with a reward scalar $R_t$ from the environment to update the agent’s sampling strategy.

> **Environment** refers to the use of standard neural network training procedure to train and evaluate the network generated by the agent, after which the corresponding results (such as accuracy) are returned.

### 4.2.3. Gradient Descent

將search space轉換為連續且可微的問題。

> A pioneering algorithm, namely **DARTS** [17], propose to search for neural architectures over the **continuous and differentiable search space** by using a softmax function to relax the discrete space,...

### 4.2.4. Surrogate Model-Based Optimization

> SMBO algorithms diﬀer from the surrogate models, which can be broadly divided into Bayesian optimization methods (including Gaussian process (GP) [164], random forests (RF) [37], Tree-structured Parzen Estimator (TPE) [165]), and neural networks [161, 166, 18, 163].

### 4.2.5. Grid and Random Search

不少研究說明Random Search並不比SOTA NAS差.

### 4.2.6. Hybrid Optimization Method

作者先說明上述單一算法的優缺點。

1. EA得到不錯的robustness，但是需要大量計算資源以及crossover和mutation都帶有隨機性質。
2. RL可以學到複雜的網路架構，但是因為需要嘗試大量的action去取得positive reward，導致不能確保efficiency和stability。
3. gradient descent-based methods可透過連續可微空間提升搜尋效率，但受限於在super-network搜尋child-network而缺乏network diversity。

後續提到混搭方法.

> The architecture optimization methods mentioned above have their own advantages and disadvantages.

> 1) The evolutionary algorithm (EA) is a mature global optimization method with high robustness. However, EA requires a lot of computational resources [26, 25], and its evolution operations (such as crossover and mutations) are performed randomly.

> 2) Although the RL-based methods (e.g., ENAS [13]) can learn complex architectural patterns, the searching efficiency and stability of the RL agent are not guaranteed, because the RL agent needs to try amounts of actions to get a positive reward.

> 3) The gradient descent-based methods (e.g., DARTS [17]) greatly improve searching efficiency by relaxing the categorical candidate operations to continuous variables. Nevertheless, in essence, they all search for a child network from a super network, which limits the diversity of neural architectures.

## 4.3. Hyper-parameter Optimization

> Most NAS methods use the same set of hyper-parameters for all candidate architectures during the whole search stage, so after ﬁnding the most promising neural architecture, it is necessary to redesign a set of hyperparameters, and use this set of hyperparameters to retrain or ﬁne-tune the architecture.

### 4.3.1. Grid and Random Search

![](https://i.imgur.com/jqbb5oT.png)

簡單解釋差異

> **GS** divides the search space into regular intervals, and selects the best-performing point after evaluating all points; in contrast, **RS**, as its name suggests, selects the best point from a set of points drawn at random.

Grid Search 方法簡單,可以平行處理，但是參數空間太大會導致效率太差。

> GS is very simple and naturally supports parallel implementation, but it is computationally expensive and inefficient when the hyper-parameter space is very large, as the number of trials grows exponentially with the dimension-
ality of hyper-parameters.

相關研究 Random Search 比 Grid Search 表現較好，但需要花較多資源找尋最佳參數。

> Although [178] empirically and theoretically shows that random search is more practical and efficient than grid search, random search does not promise an optimum. This means that although the longer the search, the more likely it is to find the optimal hyper-parameters, it will consume more resources.

### 4.3.2. Bayesian Optimization

> BO is a surrogate model-based optimization (SMBO) method, which builds a probabilistic model mapping from hyperparameters to the objective metrics evaluated on the validation set.

> It well balances exploration (evaluating as many sets of hyperparameters as possible) and exploitation (allocating more resources to those promising hyperparameters).

作者以SMBO為例來說明BO流程。

![](https://i.imgur.com/DICXIyi.png)

After the initialization, the steps of SMBO are as follows:

1. The first step is to tune the probabilistic model $M$ to fit the records dataset $D$.
2. The acqisition function $S$ is used to select the next promising neural network architecture from the probabilistic model $M$.
3. After that, the performance of the selected neural architecture will be evaludated by $f$, which is an expensive step as it involves training the neural network on the training set and evaluating it on the validation set.
4. The records dataset $D$ is updated by appending a new pair of result $(\theta_{i},y_{i})$.

## 5. Model Estimation

為了評估新網路結構的表現，直覺作法是做taining/evaluation來確定。但這需要大量時間和計算資源。這章節總結過去文獻如何加速的方案。

> Once  a  new  neural  network  has  been  generated,  its performance must be evaluated.  An intuitive method is to train the network to convergence and then evaluate its performance.  However, this method requires extensive time and computing resources.

> Several  algorithms  have  been proposed for accelerating the process of model evaluation, and are summarized as follows.

## 5.1. Low fidelity

2種方向來提昇速度，1.資料處理量 2.網路結構複雜度

> First, the number of images or the resolution of images (in terms of image-classification tasks) can be decreased.

> Second, low-fidelity model evaluationcan  be  realized  by  reducing  the  model  size,  such  as  by training with fewer filters per layer.

## 5.2. Weight sharing

> In [12], once a network has been evaluated, it is dropped. Hence, the technique of weight sharing is used to accelerate the process of NAS.

## 5.3. Surrogate

TBD

## 5.4. Early stopping

> Early stopping was first used to prevent overfitting in classical ML. It is used in several recent studies [200,201,202] to accelerate model evaluation by stopping evaluations that are predicted to perform poorly on the validation set.

## 5.5. Resource-aware

> algorithms add computational cost to the loss function as a resource constraint.

1. the parameter size
2. the number of Multiply-ACcumulate (MAC) operations
3. the number of float-point operations (FLOPs)
4. the real latency

## 6. NAS Performance Summary

TBD

## 7. Open Problems and Future Work

## 7.1. Flexible Search Space

Search Space 仍需要人工設計,難免會有bias。

> Although these search spaces have been proven effective to generate well-performing neural architectures, they all based on human knowledge and experience, which inevitably introduces human bias and hence still does not break away from the human design paradigm.

## 7.2. Applying NAS to More Areas

目前NAS只著重在CV。

> As described in Section 6, the models designed by NAS algorithms have achieved comparable results in image classification tasks (CIFAR-10 and ImageNet) to manually-designed models.

## 7.3. Interpretability

PASS

## 7.4. Reproducibility

文獻作者沒有公開全部參數設計的細節，以及論文使用大量資源產生的結果一般人無法重現。

> most of the existing NAS algorithms still have many parameters that need to be set manually at the implementation level, but the original papers do not cover too much detail.

## 7.5. Robustness

文獻上NAS表現是基於乾淨的公開資料集，但在真實世界資料的mislabeling, noise, adversarial data等問題仍有待解決。

> NAS has been proven effective in searching for promising architectures on many open datasets (e.g., CIFAR-10 and ImageNet). Those datasets are usually for the research purpose; therefore, most of the images are well-labeled. However, in real-world situations, the data inevitably contains noise (e.g., mislabeling and inadequate information). Even worse, the data might be modified to be adversarial data with carefully designed noises.

## 7.6. Joint Hyperparameter and Architecture Optimization

> Most NAS studies consider hyperparameter optimization (HPO) and architecture optimization (AO) as two separate processes.

## 7.7. Complete AutoML Pipeline

> There are already many AutoML pipeline libraries pro-posed, but most of them only focus on some parts of theAutoML pipeline

## 7.8. Lifelong Learning

> Last  but  not  least,  most  AutoML  algorithms  focusonly on solving a specific task on some fixed datasets, e.g.,image classification on CIFAR-10 and ImageNet.  However,a high-quality AutoML system should have the capabilityof lifelong learning,  i.e.,  it can 1) **efficiently learn newdata**; 2) meanwhile **remember old knowledge**.

### 7.8.1. Learn new data

#### Meta-learning

> Most of the existing NAS methodscan search a well-performing architecture for a single task.However, for the new task, they have to search for a new architecture;  otherwise,  the  old  architecture  might  not be optimal.

#### Unsupervised learning

> Meta-learning based NAS methods focus more on labeled data, while in some cases,only a portion of the data may have labels or even none at all.

### 7.8.2.  Remember old knowledge

目前的問題是，拿新的資料給pre-trained model重新學習，卻會導致新模型對舊資料的表現下降。

思考：這也是transfer-learning的目前問題，但外界課程卻不去討論這件事。或許，case-by-case的應用就不需要考慮這件事，只要保證的未來新的資料必定不包含舊有資料。

> In addition, an AutoML system must be able to constantly acquire and learn from new data, without forgetting the knowledge from old data.  However, when we use new datasets to train a pretrained model, the performance of the model on the previous data sets will be greatly reduced.

Incremental learning 研究如何解決這問題.

> Incremental learning may alleviate this problem.
