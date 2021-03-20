---
tags: Paper, Recommendation, Netflix
---

# Collaborative Filtering with Temporal Dynamics

---

<https://doi.org/10.1145/1721654.1721677>

April 2010

---

[TOC]

## ABSTRACT

顧客對產品的偏好會隨著時間而變化.  
對產品認知以及熱門程度會因為新商品出現而持續改變.同樣地,顧客的偏好也會重新定義.  
那麼在設計推薦系統或是顧客偏好模型時,對時間動態建模就是個關鍵了.  
然而這會有些挑戰,包含在與多個產品和客戶相交的生態系統中,許多不同的特徵同時發生變化,而它們中的許多相互影響,並且這些變化通常是微妙的,並且與一些數據實例(data instance)相關聯.  
經典的 time-window 或 instance decay 方法都無法運用,因為它們在丟棄數據實例(data instance)時會損失太多信號.  
需要一種更敏感的方法，該方法可以更好地區分 ``transient effects`` 和 ``long term patterns`` .  
作者提供的範例是創建一個模型,以跟踪整個數據生命週期中隨時間變化的行為. 這使我們能夠利用所有數據實例(data instance)的相關元件，而只丟棄和建模無關的元件。

## INTRODUCTION

 對 time drifting data 建模是 data mining 的核心問題之一.  
 對這類數據分析需要在對未來行為影響很小的 ``discounting temporary effects`` 之間找到適當的平衡，同時也能獲取反映數據固有性質的長期趨勢. 這類問題被廣泛稱為 ``concept drift``.

 對顧客偏好的時間變化建模帶來獨特的挑戰.  
 其中一種 time drift 是新商品/服務的出現改變了顧客的偏好.和這相關的包含季節改變或特定節日改變了``shopping patterns``. 作者稱之為**global concept drift**  
然而顧客行為也會被**localized factors**影響.例如,家庭結構的改變會大大改變購物方式.同樣,個人逐漸改變他們在電影和音樂中的品味.上述這些都無法透過捕捉**globalconcept drift**的方法捕捉到.

用戶偏好應該是一個時間錨定的問題，這反映了在兩個方面，一是用戶反饋表現的變化，比如在一個用戶為產品提供星級評級的系統中，一個曾經通過“3星”輸入來表示中性偏好的用戶，現在可能會對相同的“3星”反饋表示不滿；另一個方面，在許多情況下，系統不能將訪問同一賬戶的不同家庭成員分開，區分不同的人的一種方法就是假設時間相鄰的訪問是由同一成員(有時代表其他成員)完成的

所有這些事實說明，時間因素應該成為構建推薦系統的主要因素，而且這一算法有望在其他時間序列中獲得應用。我們使用的數據集是Netflix數據集，它規模大、時間長，而且我們在分析中還發現04年以後用戶的評分突然變高，而且老電影比新電影有更高的評分，我們會在第6部分中分析這一點。

本文最主要的貢獻是給出了一個可以應用於推薦系統中，建模用戶偏好時間漂移的方法。我們在廣泛使用的數據集上評估這個方法，這樣我們就可以和別人作比較。比較結果顯示加入時間因素以後，結果好了很多.

**個人心得**: 第3段看不太懂...

## PRELIMINARIES

解釋此篇論文:  

$m$ users $n$ items

A rating $r_{ui}(t)$ indicates the preference by user $u$
of item $i$ at day $t$, where high values mean stronger preferences.

The notation $\hat{r}_{ui}$ for the predicted value of $r_{ui}(t)$.

The set $K=\{(u,i,t) | r_{ui}(t) is known\}$

此篇論文採用RMSE評估推薦效果:  
$RMSE: \sqrt{\Sigma_{(u,i)\isin{TestSet}}{(r_{ui}-\hat{r}_{ui})^2/|TestSet|}}$

後續段落敘述CF相關延伸,沒重點.

## 3. TRACKING DRIFTING CUSTOMER PREFERENCES

但是，在許多應用包括我們推薦系統，我們還面臨著更為複雜的概念漂移形式，其中許多用戶的相互關聯的偏好在不同時間點以不同的方式漂移。

過去論文有3種方法處理上述問題:  

- instance selection  
  time window.這樣做的問題是所有窗內數據被同樣重視, 而丟棄所有窗外數據. 作者認為此方法適用在無法預期的時間變化上,對於漸進式時間變化不適用.
- instance weighting  
  根據和當前時間點的距離對數據加權. e.g. time decay function.
- ensemble learning  
  使用了不止一個分類器，通過它們和當前時間的關聯情況對分類器加權.

作者對 instance weighting 進行了廣泛的嘗試,發現調整衰減因子可以改進評估性能,其中,評估性能最好的是沒有衰減的時候,這說明儘管用戶的偏好是緩慢變化的，過去的偏好還是會嚴重影響未來的偏好,絕對不能簡單地丟棄過去的數據.  
至於 ensemble learning ,一方面它把用戶偏好分成很多小方面，這可能導致丟失一些 global patterns ;另一方面它需要把不同的用戶分開考慮,這嚴重違背了CF的原則,所以作者不是很想要它.

作者總結對用戶偏好建模的4原則：

1.模型要能考慮整個時間段而不僅僅是某個時間點上提取用戶行為特徵。

2.多種變化概念(changing concepts)都要能捕捉到. Some are user-dependent and some are item-dependent. Similarly, some are gradual while others are sudden.

3.除了能對每個user/item單獨建模,模型還要能在單一框架下整合這些概念.對user/item跨界互動能識別出higher level patterns.

4.作者不嘗試根據過去數據預測未來的的時序變化.雖然很有幫助但是太困難以及資料量並不足.作者的目標是從過去trasient noise 抽取出 persistent signal,由此預測用戶未來行為.

## 4. TIME-AWARE FACTOR MODEL

### 4.1 The anatomy of a factor model

回顧3種 factor models:

FunkSVD  
$\hat{r}_{ui}=q_{i}^{T}p_{u}$  
$\min\limits_{p*,q*}\sum\limits_{(u,i,t)\in{K}}(r_{ui}-q_{i}^{T}p_{u})^{2}+\lambda(||q_i||^{2}+||p_u||^{2})$

BaisSVD  
$\hat{r}_{ui}=u+b_{u}+b_{i}+q_{i}^{T}p_{u}$

SVD++(這篇論文修改自這算法)  
$\hat{r}_{ui}=u+b_{u}+b_{i}+q_{i}^{T}(p_{u}+|R(u)|^{-1/2}\sum\limits_{j\in{R(u)}}y_j)$

The decomposition of a rating into distinct portions is convenient here, as it allows us to treat different temporal aspects in separation.  
More specifically, we identify the following effects: (1) user biases($b_u$) change over time; (2) Item biases ($b_{i}$) change over time; (3) User preferences ($p_u$) change over time.  
On the other hand, we would not expect a significant temporal variation of item characteristics ($q_i$), as items, unlike humans, are static in their nature.  
We start with a detailed discussion of the temporal effects that are contained within the baseline predictors.

### 4.2 Time changing baseline predictors

2種 temporal variability 從 baseline predictors 可以看出.  
1.商品流行程度會隨時間變化.  
2.顧客會隨著時間改變 baseline rating. e.g.平均只會給3星會變成平均給4星.

**Template**:  
$b_{ui}(t)=\mu+b_{u}(t)+b_{i}(t)$  

作者觀察 movielens 資料集,並不期望電影的人氣每天都在波動,在長時間內才會發生變化.另外,作者觀察到用戶每天都可能發生變化並產生的不一致行為.作者認為和電影相比,對用戶建模需要比較精細的時間.

**Item bias**:  
$b_{i}(t)=b_{i}+b_{i,Bin(t)}$  
作者採10週為1個bin,資料集總共30個bins.

**User bias**:  
因為分桶會導致 user rating 不足,所以作者採用以下線性函數來捕捉gradual drift of user bias:  
$dev_{u}(t)=sign(t-t_{u}) \cdot |t-t_{u}|^{\beta}$  

第一版的 time-dependent user-bias:  
$b_{u}^{(1)}(t)=b_{u}+\alpha_{u} \cdot dev_{u}(t)$  

作者認為也可透過 splines(樣條函數) 描述 time-dependent user-bias:  
$b_{u}^{(2)}(t)=b_{u}+\dfrac{\sum\limits_{l=1}^{k_{u}}e^{-\gamma|t-t_{l}^{u}|}b_{t_{i}}^{u}}{\sum\limits_{l=1}^{k_{u}}e^{-\gamma|t-t_{l}^{u}|}}$  

However, in many applications there are sudden drifts emerging as “spikes” associated with a single day or session...To address such short lived effects, we assign a single parameter per user and day, absorbing the day-specific variability.This parameter is denoted by $b_{u,t}$.  
為了處理短暫時間內的峰值,作者在每天每位用戶上加上參數,負責吸收這些變化.

$b_{u}^{(3)}(t)=b_{u}+\alpha_{u} \cdot dev_{u}(t)+b_{u,t}$  

$b_{u}^{(4)}(t)=b_{u}+\dfrac{\sum\limits_{l=1}^{k_{u}}e^{-\gamma|t-t_{l}^{u}|}b_{t_{i}}^{u}}{\sum\limits_{l=1}^{k_{u}}e^{-\gamma|t-t_{l}^{u}|}}+b_{u,t}$  

在此篇論文,作者採用 $b_{u}^{(3)}(t)$ 作為timeSVD++的 $b_{u}$

**個人心得**: 還無法輕易理解$b_{u}^{(2)}(t)$和$b_{u}^{(4)}(t)$代表意義...

根據Table 1.實驗得知,$b_{u,t}$具有顯著影響.  

後續接著討論週期性影響.e.g. 商品在特定季節或節日會特別熱門;用戶在週末和工作日也會有不同的購買行為.  

加上 time period 參數:  
$b_{i}(t)=b_{i}+b_{i,Bin(t)}+b_{i,period(t)}$  
$b_{u}(t)=b_{u}+\alpha_{u} \cdot dev_{u}(t)+b_{u,t}+b_{i,period(t)}$  

後續討論用戶的 rating scale.  
For example, different users employ different rating scales, and a single user can change his rating scale over time.  

$b_{ui}(t)=\mu+b_{u}+\alpha_{u} \cdot dev_{u}(t)+b_{u,t}+(b_{i}+b_{i,Bin(t)}) \cdot c_{u}(t)$  
$c_{u}(t)=c_u+c_{u,t}$, $c_{u,t}$ represents day-specific variability.

**個人心得**: 沒有看程式碼很難很難從字面上了解意思...

### 4.3 Time changing factor model

作者表示 temporal dynamics 不僅改變 base predictor, 它們還影響用戶的偏好, 進而影響用戶與項目之間的交互關係.

For example, a fan of the “psychological thrillers” genre may become a fan of “crime dramas” a year later. Similarly, humans change their perception on certain actors and directors.

This effect is modeled by taking the user factors (the vector $p_{u}$) as a function of time.

$p_{u}(t)^{T}=(p_{u1}(t),...,p_{uf}(t))$  
$p_{uk}(t)=p_{uk}+\alpha_{uk} \cdot dev_{u}(t)+p_{uk,t}  k=1,...,f$  

$p_{uk}$ captures the stationary portion of the factor.  
$\alpha_{uk} \cdot dev_{u}(t)$ approximates a possible portion that changes linearly over time.  
$p_{uk,t}$ absorbs the very local, day-specific variability.

最終得出了 timeSVD++:  
$\hat{r}_{ui}(t)=\mu+b_{i}(t)+b_{u}(t)+q_{i}^{T}\left(p_{u}(t)+|R(u)|^{-1/2}\sum\limits_{j\in{R(u)}}y_{j}\right)$

後續table2實驗說明SVD++優於過往SVD model.

## 5. TEMPORAL DYNAMICS AT NEIGHBORHOOD MODELS

第5章說明 neighborhood based CF的優點,以及如何將temporal dynamics應用在這.

優點1: 可解釋.  
優點2: 可以處理新加入的rating.

此章節沒著墨太多,後續作者提到實驗結果仍比傳統neighborhood-based方法好,證明加入 dynamic temporal 有意想不到的好處.

## 6. AN EXPLORATORY STUDY

參考 <https://blog.csdn.net/weixin_39655021/article/details/96504226>

04年以后用户的评分突然变高，而且老电影比新电影有更高的评分。下面我们用我们提出的模型解释一下这件事。

04年以后用户的评分突然变高可能有下面三个原因:  
1.2004年起，Netflix改进了他们的推荐系统，这使得用户可以看到更加符合他们口味的电影。  
2.人们对于级别的要求不那么严格了。这就是娱乐至死的后果啊！！  
3.很多人2004年之前都还没有评分。

去掉那些2004年之前没有评分的用户，结果并没有什么变化。第3条排除。

无疑baseline也有一些变化，但是变化最大的是interaction。所以，评分变高是因为2004年起，Netflix改进了他们的推荐系统，这使得用户可以看到更加符合他们口味的电影。

老电影比新电影有更高的评分可能有两个原因：  
1.老电影的匹配率更好(对应interaction)  
2.老电影就是更好(对应baseline)

只观察1500天以后的电影(摆脱2004年大跳变的影响)，我们可以发现，数据有力地支持了第一条假设：老电影比新电影得到了更准确的推荐。

## 7. RELATED WORKS

略

## 8. CONCLUSIONS

略
