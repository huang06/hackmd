---
tags: Book, MLOps
---

# MLOps: Operationalizing Data Science

---

<https://www.oreilly.com/library/view/ml-ops-operationalizing/9781492074663/>

Released April 2020

Build,Management -> 可類比 mlflow tracking, models  
Deploy and Integration -> 可類比 如何交付以及如何使用  
Monitor -> 可類比 監測模型表現以及未來真實環境的改變

---

[TOC]

## Introducing the Four-Step ML Ops Approach

### Build

- feature engineering
- ML algorithm
- analytics pipelines

### Management

- a central repository where their provenance, versioning, consistency with regulatory rules and policies, approval for production, testing, deployment, and eventual replacement can be tracked.
- metadata - model artifacts
- MLplatform - metrics - models - datasets

### Deploy and Integration

- exporting visual data science workflows as Java code
- containerized deployment of model artifacts to public or private cloud environments
- model artifacts can be accessed via REST frameworks and APIs.

### Monitor

-  model fairness and other important business criteria, to provide full visibility into general business impact. 
-  BI tools connected to predictions and model inputs, or it may take the form of regularly scheduled reports on all critical business key performance indicators (KPIs).

## Build

### Data Considerations: Structures and Access

> How close is our training data to the data that the model will see in real life? And how can we be sure that the models will be able to access similar data structures in both places?

舉例,財務模型透過分析文字報表預測股價上升或下降,在實驗階段可以表現很好.但是,並不適用於高頻交易模型,因為在短時間內並無法即時地接收文字報表資訊進行預測.

> so the data scientist needs to consider whether the models will have access to a cache of recent text metrics or whether it’s reasonable to include such variables at all.

### Feature Engineering

> How do we create new variables that give us more accurate models? Can the variables be replicated in an operational environment?

舉例:年齡(numerical) -> 年齡區間(categorical)

> Consider variables like age or postal code, which may contribute to accuracy but which local laws and accepted standards of fairness might preclude from use in an ML model for a marketing campaign.

法規因素導致不能使用特定 features.

> the use of radios and networks to define a boundary-at the application level to ensure they are not active in prohibited times and places. 

> A global adherence to broadly accepted standards like these seems like the best and simplest way to ensure privacy and fairness consistent with local rules and cultural values, now and in the future as predictive modeling technologies continue to advance at a rapid pace.

> Complexity affects feature engineering because while  omplex features may yield greater accuracy, they usually come at some cost.

特徵工程的複雜度,包含特徵組合,對記憶體的消耗.

> Weather data could improve a model used in agriculture, but not all relevant inputs are accessible in an operational environment.

天氣資訊在農業應用可以提高效果,但在正式環境如何取得這些資訊呢?

> Finally, complexity ties into explainability, the idea that technical professionals should be able to use the structure of a model to gain business understanding and to account for the accuracy or inaccuracy of models to decision-makers and consumers.

可解釋性,解釋模型為何拒絕信用貸款.

### Model Testing

> How do we test our models in a way that reflects the production environment?

> The goal is to feed production data to the model in an environment where the model can be safely executed. Similarly, realistic production data should occur in the test stream also.

## Management

> Somebody must manage the model life cycle, including provenance, version control, approval, testing, deployment, and replacement.

新角色的需求 - ML Ops engineer

### The ML Ops Engineer

> Do we need a dedicated engineer to better operationalize our models?
Do we have somebody who is our ML Ops engineer without even
knowing it?

> An ML Ops engineer is someone with enough knowledge of machine learning models to understand how to deploy them and with enough knowledge of operational systems to understand how to integrate, scale, and monitor models.

![](https://i.imgur.com/hVLbxGs.png)

> The role involves a combination of understanding ML models and data transformations, plus engineering (usually at the level of scripting)

### Getting Out in Front of Model Proliferation

> Where did all these models come from?

公司內多個開發人員且每個人獨立開發,導致新模型一直被產生出來以及產生模型相依性(舉例:預測模型的資料來自於會員分群模型),以及AutoML工具的便利性讓新模型的產生速度更快,模型控管更為重要.

> The result is a greater need to track, govern, and manage models across the organization.

### Auditing, Approvals, and Version Control

> Why do we need to track models?

以結構化文件紀錄模型的metadata-有利於比較不同版本之間的差異,metadata也有助於自動化流程(e.g., approvals, electronic signatures, retraining)

> The advantage of a large body of models is related to change management, whereby anybody on the ML pipeline can determine exactly who made which change at what time and for what reason.
Change management is useful not only for compliance with outside regulations (like the approval process described before) but also for efficient collaboration, internal governance, and justifying changes to the model.

建立"變更管理"的效益

> The purpose of version control for models is to track the data and the parameters that were used to build each version, as well as the model outputs (e.g., coefficients, accuracy).

### Reusing and Repurposing Models from a Centrally Managed Repository

> How do we manage so many models coming from so many sources?

討論集中管理(Centralization)的重要性.

> The best way to centralize is to reuse and repurpose models fromcentrally managed repository that allows multiple people to use a single, approved model for multiple applications.

主要目的在於可以重複使用模型處理不同問題.

## Deploy and Integration

模型的整合對象包含:

1. database
2. website
3. device
4. inline operational system

> Why can’t we deploy without integrating, or integrate without deploying?

**什麼是佈署(Deployment)?** 將開發環境內的模型轉換成可執行的形式(e.g., a code snippet, or an API)的流程. PMML

**什麼是整合(Integration)?** 將佈署的模型嵌入外部系統中. REST APIs

### Where Deploy and Integrate Meet

> Which deployment methods and integration endpoints are the best match? Which combinations are not worth trying?

![](https://i.imgur.com/XnITzAd.png)

### Business App Development

> How do we combine the efforts of ML Ops and our application developers?

> The ideal approach to business app development is to entirely decouple machine learning models from the application, by providing them as a service—a set of APIs that developers can discover, test, and incorporate into their code.

對於應用程式開發人員,最理想的整合策略是採用API呼叫,有利於解耦(decouple)模型和應用程式.

資料科學家必須提供文件,包含:模型如何使用,輸入限制/預期輸出,適用範圍,準確度和信心水準,資料相依性.以及設計機制,紀錄模型的上線表現

當環境產生變化,模型表現不準確時,開發人員也必須要模型可以**替換**或是**停用**.minitor的細節在下節細講.

## Monitor

> It covers three types of metrics: statistical, performance, and business/ROI. 

### Statistical Metrics

> How accurate is the model now that it is running on real-world data?
How do we set a threshold and configure alerts when the model becomes inaccurate? How exactly do we measure accuracy on new or real-time data?

- Accuracy tracking
  檢查分類錯誤率,訓賴區間等指標是否低於門檻值.
- Champion–challenger
  目前運作的模型(Champion)和新模型(challenger)的表現比較.
- Population stability
  資料集的分佈是否隨時間維持著一致性.舉例,2008年初建立的房貸模型.

### Performance Metrics

> Is our infrastructure adequate to support our models?

>  Performance metrics include input/output, execution time, and number of records scored per second, plus the factors they depend on, such as memory and CPU usage.

### Business Metrics and ROI

> If the dashboards are showing that the model’s error rate is low, then why aren’t we seeing the increase in margins that we were hoping for?

不太理解這段表達重點

### When Models Drift

> Why aren’t the predictions accurate anymore? Is the model drifting?

舉例1,預測飲料銷售,因為夏天的溫度異常低,超過了模型的範圍值.

舉例2,在經濟不好的期間,顧客對於價格更加敏感,但是這個經濟情況在模型訓練階段沒有發生.

舉例3,預測模型對廣告活動預測80%準確率,但現實只有75%或更低.

> And when a model is generating predictions that effectively cost the business money because of inaccuracy, this activity is as important as the original development of the model

---

想法: 訓練一個即將上線的模型時,必須紀錄該模型的資料分佈,例如,數值型變數的分佈以及邊界值;類別型變數的分佈...等,用以觀察新舊資料的差異,作為儀表板的視覺呈現依據.

---

### Retraining and Remodeling

> The business is complaining that the predictions don’t make sense.
How do we get the model back on track?

> **Retraining** involves the same variables and model structure applied when the model was first developed, but it uses fresher data

> **Remodeling**, however, involves adding, changing, or deleting variables and is usually more work than retraining.

When a model is retrained and promoted to production, several steps ensue:

1. Model accuracy assessment
2. Model updates and explainability
3. Model diagnostics
4. Model versioning, approval, and audit

看不懂以上步驟.

### Monitoring Meets Automation

> Why do we need to do this manually? Can’t retraining take place automatically?

自動化進行champion-challenger比較,當有比現有更好模型時,啟動trigger通知相關人員.

### Case Study: Operationalizing Data Science in the Manufacturing Industry—Digital Twin Models

Digital Twin Models?

### Case Study: Operationalizing Data Science in the Insurance Industry—Dynamic Pricing Models

> A dynamic pricing strategy, in which a business sets prices for services based on market demands and other factors, helps insurers adjust price based on potential risks, customers’ willingness to pay, competitor pricing, and other variables.
