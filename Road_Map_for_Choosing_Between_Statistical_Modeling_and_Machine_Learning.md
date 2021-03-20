---
tags: Article, Machine Learning
---

# Road Map for Choosing Between Statistical Modeling and Machine Learning

---

<https://www.fharrell.com/post/stat-ml/>

2019-09-15

討論統計模型和機器學習選擇

---

[TOC]

## A statistical model may be the better choice if

- Uncertainty is inherent and the signal:noise ratio is not large—even with identical twins, one twin may get colon cancer and the other not; one should model tendencies instead of doing classification when there is randomness in the outcome
- One doesn’t have perfect training data, e.g., cannot repeatedly test one subject and have outcomes assessed without error
- One wants to isolate effects of a small number of variables
- Uncertainty in an overall prediction or the effect of a predictor is sought
- Additivity is the dominant way that predictors affect the outcome, or interactions are relatively small in number and can be pre-specified
- The sample size isn’t huge
- One wants to isolate (with a predominantly additive effect) the effects of “special” variables such as treatment or a risk factor
- One wants the entire model to be interpretable

## Machine learning may be the better choice if

- The signal:noise ratio is large and the outcome being predicted doesn’t have a strong component of randomness; e.g., in visual pattern recognition an object must be an ``E`` or not an ``E``
- The learning algorithm can be trained on an unlimited number of exact replications (e.g., 1000 repetitions of each letter in the alphabet or of a certain word to be translated to German)
- Overall prediction is the goal, without being able to succinctly describe the impact of any one variable (e.g., treatment)
- One is not very interested in estimating uncertainty in forecasts or in effects of selected predictors
- Non-additivity is expected to be strong and can’t be isolated to a few pre-specified variables (e.g., in visual pattern recognition the letter ``L`` must have both a dominating vertical component **and** a dominating horizontal component **and** these two must intersect at their endpoints)
- The sample size is huge
- One does not need to isolate the effect of a special variable such as treatment
- One does not care that the model is a “black box”
