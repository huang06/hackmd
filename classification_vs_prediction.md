---
tags: Article, Machine Learning
---

# Classification vs. Prediction

---

<https://www.fharrell.com/post/classification/>

2019-08-04

從醫學角度看待機器學習的分類與預測的差異.

---

**分類**結合預測和決策性質,可能因為決策過於武斷,會忽略了當分類失敗的成本.反之只做**預測**可以被決策者所使用.

> It is important to distinguish prediction and  classification. In many decisionmaking contexts, **classification** represents a premature decision, because classification combines prediction and decision making and usurps the decision maker in specifying costs of wrong decisions. The classification rule must be reformulated if costs/utilities or sampling criteria change. **Predictions** are separate from decisions and can be used by any decision maker.

**分類**適合用在輸出的結果不帶有隨機性質,不能用在2筆同樣的輸入資料,但確有不同的輸出的情況.這情況就需要對**趨勢**建模.

> Classification is best used with non-stochastic/deterministic outcomes that occur frequently, and not when two individuals with identical inputs can easily have different outcomes. For the latter, modeling tendencies (i.e., probabilities) is key.

若不用機率角度思考,機器學習直接採用分類方法而不考量到 risk prediction models.這情況導致把LR歸類是個分類方法(但其實不是).

> The situation has gotten acute: many machine learning experts actually label logistic regression as a classification method (it is not).

重要的是要清楚分類代表什麼,它代表的是決策.

> It is important to think about what classification really implies. Classification is in effect a decision.

最佳的決策,需要充分利用可用的資料,訂定預測目標,以及應用 loss/utility/cost function 去制定決策.舉例,最小化成本或是最大化效益.不同的使用者要有不同的 utility function.

> Optimum decisions require making full use of available data, developing predictions, and applying a loss/utility/cost function to make a decision that, for example, minimizes expected loss or maximizes expected utility. Different end users have different utility functions.

分類是強制選擇. 行銷有固定廣告預算,分析師比起去分類哪些人值不值得下預算,他比較想去對機率建模,建立lift curve,根據購買機率對顧客由高到低排序. 為了獲得"最大收益",行銷人員選擇topN的顧客作為投放目標.所以這情境不需要分類.

> Classification is a forced choice. In marketing where the advertising budget is fixed, analysts generally know better than to try to classify a potential customer as someone to ignore or someone to spend resources on. Instead, they model probabilities and create a lift curve, whereby potential customers are sorted in decreasing order of estimated probability of purchasing a product. To get the “biggest bang for the buck”, the marketer who can afford to advertise to n persons picks the n highest-probability customers as targets. This is rational, and classification is not needed here.

因為最終要做出二分類決策,所以就要用二分類方法?這並不對.

case1,當預測疾病的機率值介於中間,要做的決策應該是"沒有決策",因為需要更多資料！  
case2,決策是可以撤銷的,例如,醫師開始以較低劑量的藥物開始治療患者，然後再決定是否更改劑量或藥物.

在外科治療中,手術的決定是不可撤銷的,但是,手術時間的選擇取決於外科醫生和患者,並取決於疾病和症狀的嚴重程度. **無論如何,如果需要二分類,則必須在所有都已知的情況下進行分類，而不是只在資料分析中進行**.

> A frequent argument from data users, e.g., physicians, is that ultimately they need to make a binary decision, so binary classification is needed. This is simply not true. First of all, it is often the case that the best decision is “no decision; get more data” when the probability of disease is in the middle. In many other cases, the decision is revocable, e.g., the physician starts the patient on a drug at a lower dose and decides later whether to change the dose or the medication. In surgical therapy the decision to operate is irrevocable, but the choice of when to operate is up to the surgeon and the patient and depends on severity of disease and symptoms. At any rate, if binary classification is needed, it must be done **at the point of care when all utilities are known**, not in a data analysis.

什麼時候適合用分類方法?先看問題是機械式問題或是帶有隨機性的.

> When are forced choices appropriate? I think that one needs to consider whether the problem is **mechanistic** or **stochastic/probabilistic**. Machine learning advocates often want to apply methods made for the former to problems where biologic variation, sampling variability, and measurement errors exist.

**信噪比**也是個重要的指標,實驗可以不斷重現與否.

> It may be best to apply classification techniques instead just to high signal:noise ratio situations such as those in which there there is a known gold standard and one can replicate the experiment and get almost the same result each time. An example is pattern recognition - visual, sound, chemical composition, etc.

以降雨預測為例,我不想知道用分類器去告訴我明天是否下雨,攜帶雨具的造成的loss/disutility由我自己去權衡.

> The U.S. Weather Service has always phrased rain forecasts as probabilities. I do not want a classification of “it will rain today.” There is a slight loss/disutility of carrying an umbrella, and I want to be the one to make the tradeoff.

無論是從事信用風險評分,天氣預報,氣候預測,行銷,診斷患者的疾病,還是評估患者的預後.我都不想使用分類方法,我想要具有可靠區間或信賴區間的風險估計.

> Whether engaging in credit risk scoring, weather forecasting, climate forecasting, marketing, diagnosis a patient’s disease, or estimating a patient’s prognosis, I do not want to use a classification method. I want risk estimates with credible intervals or confidence intervals.

分類方法應用在樣本偏差的資料是個已知議題,若透過取樣來讓樣本平衡,以$\frac{1}{2}$患病率的樣本所訓練出來的分類器也不能適用在患病率$\frac{1}{1000}$的群體.  
邏輯回歸可以通過(1)將導致患病率如此之低的變量作為預測變量，或(2)(僅)為另一個患病率更高的數據集重新校準截距來優雅地處理這種情況.

> Logistic regression on the other hand elegantly handles this situation by either (1) having as predictors the variables that made the prevalence so low, or (2) recalibrating the intercept (only) for another dataset with much higher prevalence.

選擇方法的關鍵之一是要對於衡量準確度的規則具有敏感度,且要有有正確的統計屬性.機器分類的專家很少有背景來理解這個重要的問題,而用了不正確的準確性評分(例如正確分類的比例)將導致假模型.

> One of the key elements in choosing a method is having a sensitive accuracy scoring rule with the correct statistical properties. Experts in machine classification seldom have the background to understand this enormously important issue, and choosing an improper accuracy score such as proportion classified correctly will result in a bogus model.
