---
tags: Paper
---

# Past, Present, and Future of Recommender Systems: An Industry Perspective

---

<https://doi.org/10.1145/2959100.2959144>

2016

Youtube <https://www.youtube.com/watch?v=QOiZRe0UnRw>  

Slides <https://www.slideshare.net/justinbasilico/past-present-future-of-recommender-systems-an-industry-perspective>

---

[TOC]

## 1. Introduction

從產業角度看推薦系統的**過去**,**當下**以及**未來**.

## 2. PAST: THE NETFLIX PRIZE

2006年,Netflix舉辦Netflix Prize競賽,預測用戶評分並以RMSE衡量.此競賽讓數千支隊伍聚焦在這單一指標上,簡化的推薦系統問題也讓公司學到了些東西.

後續從此競賽而生的算法: SVD++, RBM. 但上述原始代碼只設計用來處理1億筆資料,但實際資料卻有50億筆,且還不考慮算法如何處理後續新增的新評分.Netflix改良了上述算法才在正式環境使用.

後續包含加入時間因素的算法,以及評分噪音問題,ensemble算法.

## 3. PRESENT: RECOMMENDATION BEYOND RATING PREDICTION

### 3.1 Recommender Systems in Industry

主要說明各個產業的推薦系統現況.  
Video: Netflix, Youtube  
Music: Spotify  
E-Commerce: Amazon, eBay  
News: Google News  
Social networks: Twitter, Facebook, Quora

> Typically the way algorithms are chosen in industry applications is by running randomized, controlled experiments (A/B tests) that measure the value of one approach over another.

上述應用作者都有提供相對應的論文,以及最後也提到都會透過實際上線驗證該採用何種算法.

### 3.2 Everything is a Recommendation

把推薦系統作為主要的網站內容. 連搜尋功能都能加入推薦算法(自動完成功能).

### 3.3 Beyond Explicit Feedback

Implicit feedback.

> For example, Bayesian Personalized Ranking (BPR) [21], uses it to compute a personalized ranking. Implicit and explicit feedback can also be combined in different ways, for example using SVD++, logistic ordinal regression [17] to provide a mapping, or taking a Bayesian approach like Matchbox [27].

### 3.4 Ranking

目前推薦應用都採用排序.個人推薦和整體熱門對用戶很重要卻也互相衝突.可行解作法就是採用ranking.

> The traditional ``pointwise`` approach for Learning to Rank (LTR) treats ranking as a simple binary classification problem where the only input are positive and negative examples. Typical models used in this context include Logistic Regression, or Gradient Boosted Decision Trees.  
> ``Pairwise`` algorithms optimize a loss function defined on pairwise preferences to minimize the number of inversions.  
> Going further, ``listwise`` approaches try to directly optimize the ranking of the whole list instead of individual pairs. RankALS [28], for example, defines an objective function that directly includes the ranking optimization and then uses Alternating Least Squares for optimizing.  
> These approaches use rank-specific information retrieval metrics to measure the performance of a ranking model. Ideally, we would like to directly optimize our models those same metrics. However, they are not differentiable and standard methods cannot be directly applied. Some methods like CLiMF [25] use a smoothed version of the objective function to run gradient descent.

## 4. FUTURE: RESEARCH DIRECTIONS  WITH INDUSTRIAL APPLICABILITY

### 4.1 Full Page Optimization

如何從整體頁面去思考如何推薦?  

> when optimizing a whole page, we not only optimize for relevance, but also other aspects like diversity and freshness.  
> An ideal full page optimization algorithm would also be able to personalize how recommendations are mixed with other elements of the user experience by understanding a user’s browsing or attention behavior [15]. It may also be useful to adapt a page within a session as a user’s actions reveal their current intents.

相關論文已經開始討論

> For example, [1] present an approach in the context of news involving a sequential click model for the user and a relevance model that promotes ``diversity`` through submodular functions. Another example is [30], where the layout of elements on a search results page is performed using a learned function over both the page elements and layout.

### 4.2 Personalizing How We Recommend

個人化推薦: 從推薦什麼(What),到如何推薦(How).

1. 推薦清單的 diversity, novelty, popularity 等參數原本是由系統設定而生成. 但根據不同用戶應該要有不同的比例分配. 不同國家以及不同語言的風俗名情也該要考慮到.
2. 還能個人化的內容包含: 圖片,描述, metadata...等.

### 4.3 Indirect Feedback

用戶只會看到系統推薦給他的商品清單.

- Challenges
  - Users can only click on what you show
  - No cunterfactuals
  - Implicit data has no real "negatives"
- Potential solutions
  - Attention models
  - Context is also indirect/implicit feedback
  - Explore/explit approaches and learning across time
  - ...

### 4.4 Value-aware Recommendations

各種推薦算法最終的目的都是一樣的: 優化算法讓用戶願意使用我們的推薦結果. 然而這些算法都是基於商品的價值都是等值的假設.

> In e-commerce some items might have a higher margin for the retailer while giving the same satisfaction to the user [32].

大部分電商比較在乎能夠長期回流(long-term)的推薦方法,而非只能促進短期點擊率(short-term)但對長期使用者滿意度沒有幫助.

> We also may want to make sure that our algorithms are not biased against certain kinds of users, for example not reinforcing prejudices. As such, it is important to also keep in mind a user’s values.  
> ...  
> However, there is a clear need to come up with algorithms that care holistically about the long-term value of the item being recommended so that given a similar probability user engagement, the algorithm can steer towards long-term gains. We call this family of algorithms value-aware recommendations.
