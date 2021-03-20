---
tags: Paper
---

# Dynamic Item-Based Recommendation Algorithm with Time Decay

---

<https://doi.org/10.1109/ICNC.2010.5582899>

2010

---

[TOC]

## ABSTRACTION

根據過往推薦系統方法主要著重在CF上。而經驗上itemCF的推薦效果優於userCF。  
在這篇論文，作者將``time decay``概念加入在item-item相似度計算裡，並且用3種time decay function比較推薦效果。最後實驗證明加入time decay的算法能有更加的推薦效果。

## INTRODUCTION

略

## RELATED WORK

這段作者主要說明itemCF比userCF還要適合的原因，以及列出常見的item-item相似度算法。

cosine:  
$sim(i,j)=cos(R_{*,i},R_{*,j})=\dfrac{R_{*,i},R_{*,j}}{||R_{*,i}||_2,||R_{*,j}||_2}$

prob:  
$sim(i,j)=P(i|j)=\dfrac{Num(i \cap j)}{Num(j)}$  
此算法計算user購買商品$i$的條件下也會購買商品$j$的機率，但這是非對稱算法($P(i|j) \neq P(j|i)$)。  
後續有多種算法改良方法:  
[8] multiplies $P(i|j)$ by $log_2 P(i)$  
[9] devides $P(i|j)$ by $P(i)$  
[7] $sim(i, j)=\dfrac{Num(i \cap j)}{Num(i) \times (Num(j))^\varkappa}$, if $\varkappa=1$, $sim(i,j)$ become symmetric similarity function.

## DYNAMIC ITEM-BASED TOP-N RECOMMENDATION ALGORITHMS WITH TIME DECAY

### A. Problems

item相似度在不同時間區間內購買應該要不一樣。  
e.g 我們可以合理地理解2種商品在**一天內**都被購買得出的相似度不同於2種商品在**一年**內都被購買所得出的相似度。

被時間因素影響的現象，作者稱之為**Time Decay**。

**個人心得**: 作者沒說明該如何處理消費者多次購買單樣商品的消費紀錄.

### B. Dynamic Similarity Function with time decay

$sim(i,j)=\sum\limits_{k=1}^{n}[R_{ki} \cdot R_{kj} \cdot \theta(|T_{ki}-T_{kj}|)]$  
$R_{ki}$: 若用戶$k$購買過商品$i$則為1,反之為0  
$R_{kj}$: 若用戶$k$購買過商品$j$則為1,反之為0  
$|T_{ki}-T_{kj}|$: 用戶$k$對商品$i, j$的購買時間差  
$\theta(x)$: Time Decay Function(TDF),在這篇論文當時間差$x$越大,TDF得到的值會越小

作者不對算法做標準化(normalization),因為在作者實驗中會降低precision.

### C. Time Decay Functions

作者提出挑選TDF的2個限制式:  
Constraint (i): $\forall{x},\theta(x) \in [0,1]$  
Constraint (ii): $\forall{x,x'}$,if $x>= x'$ then $\theta(x) <= \theta(x')$

後續作者提出3種符合限制式的TDF:

#### (1) *Concave Time Decay Function*

$\theta(x)=\alpha^{x}$  
The parameter $\alpha \in [0,1]$ is named concave Time Deecay Coefficient(TDC).  
The smaller $\alpha$ is, the stronger of the time decay's effectiveness is.

#### (2) *Convex Time Decay Function*

$\theta(x)=1-\beta^{t-x}$  
Parameter $t$ represents the value of the largest time interval in the training dataset.  
The parameter $\beta \in (0,1)$ is the convex TDC.  
The larger $\beta$ is, the stronger of the time decay's effectiveness is.

#### (3) *Linear Time Decay Function*

$\theta(x)=1-\dfrac{x}{t} \cdot \gamma$  
Parameter $t$ represents the value of the largest time interval in the training dataset.  
The parameter $\gamma \in [0,1]$ is the linear TDC.  
The larger $\gamma$ is, the stronger of the time decay's effectiveness is.

### D. Applying the Dynamic Model

此段落在說明 pseudo-code,  
沒重點,略過.

### E. Computational Complexity

upper bound: $O(m^{2}n)$.

因為資料集是稀疏矩陣,以上最壞情況不會發生,所以作者沒有採用矩陣而是linked list去減少不必要的計算.

## EXPERIMENTS AND RESULTS

### A. Experimental Dataset

A real e-commerence data released by Alibaba China.  
前4天紀錄為訓練集,後4天為測試集.

作者提供的下載網址已經失效...

### B. Evaluation Method

以下是作者定義的 ``precision``.

$CSI(i)=\dfrac{|T_{i} \cap V_{i}|}{Min(|T_{i},|V_{i}||)}$

$precision=\dfrac{1}{n}\sum\limits_{i=1}^{n}CSI(i)$

**個人心得**: 有趣的是作者在分母取推薦清單長度和用戶曾觀看數量之間的最小值,而非經典precision作法是根據推薦清單長度代表分母.

### C. Comparation of Algorithms

此章節呈現3種TDF的調參過程.

### D. Result Analysis

3種TDF調參後都能得到一樣高的precision,並且都優於itemCF,userCF.

**個人心得**: 如果拿其他經典推薦算法應用在此資料集,不知道作者的TDF能否維持優勢?

## CONCLUSION & FUTURE WORK

time decay的參數設定對於item相似度影響很大. 如果商品間的時間差過大導致TDF輸出值可能接近0,其實也沒必要做後續計算了.這有助於我們可以提前篩選並只對有符合時間差門檻的item相似度計算.
