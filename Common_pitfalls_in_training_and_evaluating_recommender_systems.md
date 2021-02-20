---
tags: Paper
---

# Common pitfalls in training and evaluating recommender systems

---

<https://doi.org/10.1145/3137597.3137601>

September 2017

4種普遍發生在訓練,評估推薦系統的陷阱。

1. 訓練資料集受到原有算法影響
2. 測試資料集受到原有算法影響
3. CTR高不代表平台收入也會增加
4. 推薦系統不見得**額外**增加平台收入

---

[TOC]

## 1. INTRODUCTION

略過

## 2. A TYPICAL PROCEDURE OF PREPARING TRAINING AND TEST DATASETS

Figure 1. 以時間軸說明常見的train/test切法

$t_{0} - t_{1}:沒有推薦系統的時期$

$t_{1}:初版推薦系統R_{orig}開始上線運作$

$t_{1} - t_{s}:訓練集$

$t_{s} - t_{2}:測試集$

## 3. ISSUE 1: TRAINED MODEL COULD BE BIASED TOWARD HIGHLY REACHABLE PRODUCTS

## 3.1 The core problem

產品$p_{i}$頁面上若出現產品$p_{j}$的直接連結，很多使用者很可能會點擊$p_{i}$之後點擊$p_{j}$。

$p_{i} \rightarrow p_{j}會產生很強的正向關係$

> Thus, the information $p_{i}$ → $p_{j}$ (or more formally, ($x_{i}$ = $(x_{i,1}=p_{i}),p_{j})$ as a positive training instance) may become a strong positive signal simply because it is extremely easy to reach $p_{j}$ from $p_{i}$

但其實二商品間並沒有相關。

> As a result, the linked products may be little relevant to the current browsing product.

最終會導致新的推薦系統推出商品是因為過去受到layout影響所產生的易觸及商品。

> As a result, the recommendation algorithms are likely to output the highly reachable products, which is highly influenced by the layout of the product pages.

## 3.2 Selecting proper training data

作者提出的解決方法: 降低商品和易觸及商品之間的權重

> we should weaken the weights of the positive instances in which next clicked product is highly reachable.

## 3.3 Experiment

作者將訓練集設計成2種：

train-all: 原始訓練集

train-sel: 拿掉$p_{j}$是包含易觸及商品的訓練樣本

Table 2. 可以觀察到2種不同訓練集所產出的推薦結果中，易觸及商品的佔比差異。

## 3.4 Lessons learned

根據**點擊紀錄**訓練的推薦系統很有可能會學習到layout和現行算法的推薦規則。

> As a result, training a recommender system based on the clickstreams are likely to learn (1) the “layout” of the pages, and (2) the recommendation rules of the online recommender system.

理想解決方法，訓練集內只保留用戶**主動觸及**的商品，排除促銷商品(易觸及商品)以及現行推薦系統產生的商品

> Ideally, we should keep only the spontaneous clicks in the log to, at least partially, solve or bypass this issue.

## 4. ISSUE 2: THE ONLINE RECOMMEN-DATION ALGORITHM AFFECTS THEDISTRIBUTION OF THE TEST DATA

## 4.1 The Core problem

使用者沒機會點擊到原有算法**沒推薦到的商品集**.

> the online users have no chances to click on the products that appear only in $L_{new}$ but not in $L_{orig}$.

## 4.2 Selecting proper test data

> if a product $p_{k}$ appears in $L_{new}$ but not in $L_{orig}$, the product $p_{k}$ is less likely to be clicked even though $p_{k}$ might be a great recommendation given the context feature $xi$.

作者提出的解決方法: 測試集內只保留用戶**主動觸及**的商品，排除促銷商品(易觸及商品)以及現行推薦系統產生的商品

## 4.3 Experiment

略過

## 4.4 Lessons learned

過往研究都用上了全部的測試資料集,導致有利於原有算法,我們應該要小心挑選測試集以得到公平的評估.

> Previous  studies  sometimes  use  all  the  available  test  dataas the ground truth for evaluation.  Unfortunately, such anevaluation process inevitably favors the algorithms that sug-gest products close to the online recommendation algorithm.We should carefully select the test dataset to perform a fairer evaluation.

## 5. ISSUE 3: CLICK THROUGH RATES AREMEDIOCRE PROXY TO THE RECOM-MENDATION REVENUES

## 5.1

多數研究都採用CTR和其他算法比較. CTR指標是屬於user-centric,用來評估使用者對推薦的滿意程度. 也因為業界不願意提供收益相關數據,研究都只能透過提昇user-centric measures(CTR)來代表business-centric measures(Revenue)的提昇. 但是這並沒被仔細驗證.

## 5.2 Experiment

略過

## 5.3 Lessons learned

得到很多點擊,並不確保能讓平台帶來大量的收入

> As a result, even if a recommendation algorithm attracts many clicks, we cannot assure this algorithm will bring a large amount of revenue to the website.

## 6. ISSUE 4: EVALUATING RECOMMENDATION REVENUE IS NOT STRAIGHTFORWARD

## 6.1 The core problem

有可能推薦系統對用戶而言,只是**方便尋找**他要的東西,即便沒有推薦系統,用戶透過網站上其他途徑找到該商品.

> It is possible that the recommendation modules are served as a convenient tool for users to locate the desired items, but even without the recommendation module, the users can still discover these items
through another user interface provided by the website.

## 6.2 Experiment

略

## 6.3 Lessons learned

最極端的案例是電商公司可以將整個頁面塞滿推薦,然後宣稱100%的收益都從推薦而來XD,顯然地是讓人誤導的作法.

> In an extreme case, an EC company can fill in the entire page with recommendations and claim that nearly 100% of their revenue comes directly from recommendations. Apparently, such a claim is misleading.

好的驗證方法要透過A/B testing.但這在學術界很難進行.

> To proper evaluate the extra revenue contributed by a recommender system, we still need to leverage on A/B testing. Unfortunately, it is very difficult for the researchers in academia to perform A/B testing in practice.

## 7. RELATED WORKS

## 7.1 Common metrics to evaluate recommender systems

只專注在CTR,會讓算法傾向推薦熱門商品.

> A simple way to measure accuracy is click through rate – in
what percentage a user clicks a recommendation [10]. However, such a metric may favor the algorithms that tend to recommend popular items, because recommendation accuracy usually declines towards the long tail [25].

研究指出,加上考慮 diversity 可以提昇用戶對推薦的滿意程度.而diversity和accuracy無法二者兼顧,需要視需求決定.

> Diversity and accuracy are usually a trade-off. One can easily increase diversity by recommending unrelated items, but this usually sacrifices the accuracy.

## 7.2 Reviewing Previous Competitions and Publications

作者提到過往比賽,透過點擊資料去建立推薦模型,但問題是會推薦出highly reachable items. 所以比賽開始採用線上測試來驗證模型的好壞.

點擊行為和購買行為完全不一樣.作者以文獻[13]說明,2種行為是可以透過分類器分開.用戶會透過推薦內容持續地點擊,但這並不代表會購買.

## 8. DISCUSSION AND FUTURE WORK

略過
