---
tags: Book, Unfinished, Recsys
---

# Practical Recommender Systems

---

<https://www.manning.com/books/practical-recommender-systems>

January 2019

不討論算法細節,大多分享自身經驗.

---

[TOC]

## Part 1 Getting ready for recommender systems

### 1. Waht's a recommender

### 1.1 Real-life recommendations

說明生活中常見的推薦行為,以及比較recommendation(推薦) 和 commercial(商業廣告).

**推薦**是根據用戶過去的行為決定;**商業廣告**則是要考量店家的利益.作者認為二者間的的差異也可以是模糊的.

### 1.1.1 Recommender systems are at home on the internet

- **non-personalized** e.g. top-10推薦
- **semi-personalized** e.g. 根據人口統計特徵或目前的地理位置推薦
- **Personalized recommendations** e.g. 用戶有興趣的不只是熱賣商品,還有非熱賣商品或是**長尾商品**

### 1.1.2 The long tail

brick-and-mortar shops (實體店面)  
實體店面的限制在於沒有足夠儲存空間以及商品展示空間展示給消費者看,導致只能以熱賣商品為主.  
反之,線上店面就沒有太多限制,尤其是賣非數位商品,可以賣少量但多樣的商品給不同類型的顧客,達成長尾經濟

### 1.1.3 The Netflix recommender system

從Netflix界面觀察推薦系統以及動機.

- Netflix花了許多成本製作原創影集,並只能在Netflix上播放
- 若用戶觀看非原創影集,Netfliubuntu editorx還要付錢給片商.

作者觀點: 即使頁面顯示個人化的推薦結果,但基於Netflix的商業考量,Netflix原創影集會擺在第二列(顯眼位置)

作者觀點: Netflix可能同時考量diversity而並非只注重accuracy,推薦的影集預測不會被給太高分,但它仍可能是用戶會想看的影集,作者猜想可能原因是Netflix並非只看重rating.

RANKING: 作者的實際體驗發現在不同時間或不同情境,Netflix會推薦不一樣的影集內容.

作者觀點: 熱門類別中,Netflix並不一定是將最熱門影集放在最左邊並依序排列,可能還會基於和用戶的相關性重新排序熱門影集.

BOOSTING: 不太了解作者的說法.

SOCIAL MEDIA CONNECTION: 因為用戶抗議，Netflix以移除此功能.

TASTE PROFIL: Netflix以移除此功能.

### 1.1.4 Recommender system definition

A recommender system calculates and provides relevant content to the user based  
on knowledge of the user, content, and interactions between the user and the item.

作者**猜想**Netflix的Top Picks推薦流程:

![figure 1.9](src/figure_1_9.png)

1. 收到Top Picks需求
2. 啟動推薦系統pipeline,從資料庫取出符合用戶品味的候選商品(canditate items)
3. 挑選前N個商品
4. 預測商品rating,刪除low rating商品
5. 增加rating視為商品特徵
6. 根據用戶的品味,情境,人物特徵的相關性排序商品.此作法可以增加商品的diversity,而非只看rating
7. 刪除低相關性商品
8. pipeline回傳結果
9. 系統回傳結果

作者示範資料在推薦流程中的如何被使用:

![figure 1.10](src/figure_1_10.png)

### 1.2 Taxonomy of recommender systems

作者用以下八種維度來分類推薦系統:

1. Domain
    - 不同Domain推薦錯誤的後果差異很大.
    - 有些Domain可以重複推薦相同商品

2. Purpose

    以NetFlix為例:
    - 從用戶角度思考,在特定時間可以看到和用戶最相關的影集;從Netflix角度思考,讓用戶每個月都會花錢訂閱.
    - Netflix透過影集觀看數量可以衡量表現如何.在這段作者想表達使用proxy goal必須謹慎小心,可能並非是預期效應帶來的結果.e.g. Netflix的使用時數可能不是很好的proxy,因為用戶可能花了很多時間在-搜尋他想看的影集,或許他已經找到了,但網站卻沒有反應.
    - 同時也需要加入商業考量,如何用最小的成本滿足用戶的需求.e.g. 推薦老片比推薦新片所要付的版權費更低,更甚至推薦Netflix原創影集.
    - Purpose也需考量是提供資訊,幫助,或者教導性質訊息給用戶?但通常也是為了賣出更多商品是主要目的.
    - 哪種類型的會員是推薦系統的服務對象.新客或忠實客戶?
    - 網站是否有自動化服務(automatic consumption)? e.g. Spotify會根據音樂或歌手自動持續播放音樂.作者只提出議題,並沒說明對於設計推薦系統的影響.

3. Context

    用戶接收到推薦結果的環境.e.g. 裝置,地理位置,時間,用戶目前在做的事,天氣,甚至用戶心理狀態!

    當用戶在辦公室內搜尋咖啡店時,搜尋結果可能是較遠但高品質的咖啡店;反之若用戶正在戶外且正下著大雨,搜尋結果應該要是最近的咖啡店.

4. Personalization level
    - NON-PERSONALIZED
    - SEMI/SEGMENT-PERSONALIZED
    - PERSONALIZED

5. Whose opinions

    看不懂作者想表達什麼

6. Privacy and trustworthiness

    Beware: if customers start feeling manipulated, they’ll stop trusting your recommendations and eventually find what they need somewhere else.

7. Interfaces

    - 輸出類型可以是預測,推薦,或是篩選機制
    - 輸出結果是否是可解釋性的?model accuracy-model interpretation trade-off

    ![figure 1.15](src/figure_1_15.png)

8. Algorithms

    - COLLABORATIVE FILTERING
    - CONTENT-BASED FILTERING
    - HYBRID RECOMMENDER

1.3 Machine learning and the Netflix Prize

    Netflix並未使用比賽獲勝的模型,作者猜想可能原因是模型為了表現能力而過於複雜,但這並不能保證能改善推薦效果.

1.4 The MovieGEEKs website

    demo網站說明

1.5 Building a recommender system

    demo網站說明

### 2. User behavior and how to collect it

### 2.1 How (I think) Netflix gathers evidence while you browse

    略

### 2.1.1 The evidence Netflix collects

![table 2.1](src/table_2_1.png)

### 2.2 Finding useful user behavior

    略

### 2.2.1 Capturing visitor impressions

    略

### 2.2.2 What you can learn from a shop browser

- page view
    他可以表示用戶對這商品很有興趣,但也有可能是用戶正在隨機點擊或是在找商品,所以越多的點擊並不一定是好的訊號.
    好的推薦系統應該是讓用戶在少量的page view可以找到他想要的商品.
- page duration
    直觀來說,用戶在頁面停留的越久代表越有興趣.但或許用戶不在位子上或是正在做別的事.

    作者解法: 定義不同時間段的可能含意
    < 5s: 沒興趣  
    \> 5s: 有興趣  
    \> 1min: 非常有興趣  
    \> 5mins: 可能離開了
    \> 10mins: ...
- expansion clicks
    透過觀察用戶是否點擊`接續閱讀`來代表用戶是否對商品有興趣.
- social media links
    用戶的分享連結等行為.
- save for later
    用戶的儲存行為也能視為對商品有興趣的信號.
- search terms
    用戶輸入想搜尋的商品關鍵字,能確實地知道用戶想要的是什麼.
    當用戶輸入關鍵字搜尋商品，系統可能不會有相同關鍵字的商品，但系統可推薦與關鍵字相關的商品.

> **Linking searched items with the resulting consumption**  
Another thing to consider about search terms (what a customer types in the search  
field) is that it’s a good idea to connect what’s searched for with what’s consumed.  
Say a user searches for Star Wars and looks at Harlock: Space Pirate, which has a  
reference to Babylon A.D., and the user ends up watching that. Maybe it’s worth put-  
ting Babylon A.D. in the search result for Star Wars.

### 2.2.3 Act of buying

- 如果用戶購買商品的改變?這代表用戶的興趣改變或只是替人買禮物? 是否該視為outlier?
- 我們通常都會認為商品的售出次數增加,代表用戶對該商品越有正向評價.

### 2.2.4 Consuming products

用戶的商品使用過程,以音樂播放為例.

- Start playing
    代表用戶對這音樂有興趣,是個正面訊號.
- Stop playing
    音樂在前20秒或電影在前20分鐘被停止,可能是個負面訊號。
- Resume playing
    如果在短時間內又重新播放,代表用戶只是被某事中斷,或許我們可以忽略該次中斷;若隔24小時仍繼續播放該歌曲,用戶可能真的喜歡該歌曲!
- Speeding
    第一次播放就快轉,可能是個負面訊號;但曾被播放多次仍快轉,代表用戶只想跳過部份無聊的片段。用戶也可能只是為了解歌曲而多次快轉,進而確認是否喜歡該歌曲。
- Playing it to the end
    這是最期望的結果.這是個好的結果.
- Replaying
    音樂和電影被重播多次代表用戶的的確喜歡它。但如果是線上課程被重播多次，代表該課程對用戶而言太困難而需要重複多次...。

### 2.2.5 visitor ratings

大部分網站透過星等和評論可以更加了解用戶的意見.而Amazon則透過星星旁的說明提醒用戶該星星代表意義,確保用戶知道每個星等的意義.

- A SENSE OF CONTROL
    透過反問用戶是否滿意推薦的商品列表,讓用戶對系統有控制感.
    作者觀點: 作者認為透過資料集的rating測試推薦系統的表現能力是困難的.這只能看出系統的預測能力但不能看出系統能夠吸引更多的用戶.
- SAVING A RATING
- NEGATIVE USER RATINGS
    不喜歡此商品代表並不想給商品任何rating.但當附註說明一顆星代表非常討厭,用戶給商品一顆星時意義就不一樣了.
- VOTING
    用戶對其他人留言的支持程度

#### 2.2.6 Getting to know your customers the (old) Netflix way

Netflix曾透過讓用戶建立自己的profile去提供更好的推薦,以及能對新用戶推薦.

### 2.3  Identifying users

探討如何識別用戶的問題.  
已知問題1: 同個帳號被多人使用  
已知問題2: 跨裝置使用問題

### 2.4 Getting visitor data from other sources

### 2.5 The collector

Demo網站的資料格式說明

### 2.5.1 Building the project files

### 2.5.2 The data model

### 2.5.3 The snitch: Client-side evidence collector

### 2.5.4 Integrating the collector into MovieGEEKs

### 2.6 What users in the system are and how to model them

### 3. Monitoring the system

### 3.1 Why adding a dashboard is a good idea

### 3.1.1 Answering “How are we doing?”

### 3.2 Doing the analytics

### 3.2.1 Web analytics

可分類為off-site和on-site.  
office-site包含Opportunity(根據過去用戶數量觀察網站潛力),  Visibility(進入此網站的容易程度),Voice(有多少人談論此網站)  
此章節作者只討論on-site analytics,觀察用戶在網站上行為.

### 3.2.2 The basic statistics

- 造訪人數
- 商品銷售
- 總收入

### 3.2.3 Conversions

    Conversion rate(轉換率)來自於conversion funnel

    公式 = 目標達成數量/造訪數量
    
    作者觀點: 根據不同場景,轉換以及轉換率的定義不同.  
    作者以售車和電商舉例.電商期望的是用戶每次的造訪都會達成一次轉換.但若是賣車則不考慮客戶來過幾次,只要最後成交就算成功.
![fugure 3.6](src/figure_3_6.png)

### 3.2.4 Analyzing the path up to conversion

    SQL 範例

### 3.2.5 Conversion path

    SQL 範例

### 3.3 Personas

    用儀表板說明可觀察的personas

### 3.4 MovieGEEKs dashboard

    說明MovieGEEKs網站內容,不必筆記.

### 3.4.1 Auto-generating data to your log

### 3.4.2 Specification and design of the analytics dashboard

### 3.4.3 Analytics dashboard wireframe

### 3.4.4 Architecture

### 4. Ratings and how to calculate them

### 4.1 User-item preferences

### 4.1.1 Definition of ratings

### 4.1.2 User-item matrix

    討論稀疏矩陣問題.

### 4.2 Explicit or implicit ratings

    如何確認rating的可信度?
    作者觀點: 這是個domain-specific問題,並沒有確切的答案.
> The truth about users’ tastes  
It’s important to remember that the log data you collect is evidence, and you need to foster an objective view of what users do on the site. Translating it into ratings and opinions is a subjective process, which is something that has to be tweaked not only for each domain but also for each recommender algorithm.

### 4.2.1 How we use trusted sources for recommendations

    作者觀點: 無論是explicit或implicit rating,都有可能是假的,我們必須去思考是什麼動機會讓用戶去給商品高分或給低分.

### 4.3 Revisiting explicit ratings

    作者觀點: explicit rating會受到許多因素影響給分.e.g. 身邊朋友,給低分原因是整個商品都不好或只是細節不滿意.

### 4.4 What are implicit ratings

    作者觀點: 針對用戶在網站上的行為自定義符合的raing.e.g. 購買過代表正面,退貨代表負面.並應該有個調整機制隨時調整rating的計算方式.
    作者觀點: rating高的商品也並不代表應該被馬上推薦.可以透過切分時間建立推薦系統,但這也會面臨trade-off問題(accurate recommendation, data sparsity).
    作者觀點: 並非所有應用都要加入explicit rating機制.透過implicit rating也能達成推薦.

### 4.4.1 People suggestions

    implicit rating的應用

### 4.4.2 Considerations of calculating ratings

    作者觀點: 在計算rating之前,我們需要先想清楚資料的類型以及網站顯示推薦清單的方式.

    BINARY USER-ITEM MATRIX
        只用1,0表示
        缺點: 以前有興趣的商品,現在不見得想買. e.g. 技術書
    TIME-BASED APPROACH
        作者以Hacker News為例,加入time decay性質.
        rating = (score-1)/(item age in hours + 2) ** gravity
    BEHAVIOR-BASED APPROACH
        若矩陣只考慮購買行為,會有稀疏問題,可以將用戶對商品的其他行為也列入考慮,填補缺值.

### 4.5 Calculating implicit ratings

    作者觀點: 推薦系統的目的是計算用戶會購買的可能性有多少,而非是計算用戶會給商品多高的rating.

### 4.5.1 Looking at the behavioral data

    我們可以觀察用戶的點擊行為到最後購買成功的過程中,除了購買一定是給high rating之外,中間的點擊過程也能給予不同程度的正面分數,代表會購買的可能性

    implicit rating function: IR = (W1*#event1) + (W2 * #event2) + ... + (Wn * #eventn)
    用戶在特定行為的數量可能高於其他正常的用戶行為,此時我們可以考量用cut-off策略避免極端值產生. e.g. min(#eventn, relevant_maxn)

### 4.5.2 This could be considered a machine learning problem

    我們可以根據過去的線上行為透過ML預測用戶最終有無購買產品(分類問題)

### 4.6 How to implement implicit ratings

    此章節是SQL和Python實做

    作者觀點: 從儀表板觀察得知若用戶有瀏覽行為但沒購買此商品,我們可以推薦給他該商品.但是瀏覽次數很多但都不買,或許就不用推薦給他該商品了.

### 4.6.1 Adding the time aspect

    當最近行為的重要性高於過去的行為,若此假設成立,就適合用time decay加入rating中.  

    score = 1 / age of event in days

### 4.7 Less frequent items provide more value

    作者觀點: 用戶購買熱門商品並不帶有太多資訊價值,反之若購買的是冷門商品,我們可以從中得知用戶特殊的品味.  
    換句話說,越多人買的商品,所含有的價值越低.

    Inverse User Frequency (IUF): 觀念類似於TF-IDF的IDF,當我們認為.
    IUFi,u = log(N / (1+n)), n是用戶u購買商品i的次數, N是此類別的用戶數量.
    將IUF帶入implicit raing的計算過程:
    W * Ru,i = Ru,i * IUFi,u -> 提高user對冷門item的rating

### 5. Non-personalized recommendations

    此章節作者強調非個人化推薦仍有其重要性  
    1. 新網站並沒有歷史數據可以使用.  
    2. 瀏覽網站的大多數都是非會員用戶,個人的瀏覽紀錄也並不多.

### 5.1 What’s a non-personalized recommendation

### 5.1.1 What’s a commercial

    **廣告**是以商家角度思考,要求商品曝光度以及說服用戶該商品值得購買;  
    但**推薦**是從用戶角度思考,推薦用戶他想要的東西.

### 5.1.2 What does a recommendation do

### 5.2 How to make recommendations when you have no data

    作者強調: **no data at all means no recommendations**
    在沒有資料情況下,我們只能假設一些情況是用戶可能會願意買單的商品清單.

    1. 靜態推薦
        Tip: 商品排序不應該用資料庫預設排序方式產出推薦結果,而是要有自己的一套自定義排序作法.因為資料排序可能是因為一開始資料如何被寫入資料庫所決定e.g. 字母順序.

    2. 動態推薦
        Recency(越近排序越高)e.g. 電影.  
        Recency(越遠排序越高)e.g. 古董.  
        或許用戶在意的其他因素並非時間.作者在這段的舉例不是很懂.

### 5.2.1 Top 10: A chart of items

    Top10是我們常用的推薦手法,但我們仍需要時常觀察Top10的受歡迎程度.  
    作者觀點: 透過加入回饋機制(e.g. like)知道用戶的口味變化.

### 5.3 Implementing the chart and the groundwork for the recommender system component

    說明SQL實做方法,不做筆記

### 5.4 Seeded recommendations

> One thing that many sites take advantage of is you looking at a specific item, which can be used to create recommenders for associated items.  
These items could be said to be **seeds**.

### 5.4.1 Frequently bought items similar to the one you’re viewing

    作者針對超市以及賣船為例說明關聯銷售的可行性:  
    超市:通常一起購買的商品組(frequency set)
    賣船:賣船通常還會搭配救生衣,救生艇(frequently needed equipment) -> 大商品搭配小商品的銷售方式.

### 5.4.2 Association rules

    此章節列舉2種指標計算商品間的關聯性:  
    Confidence(A->B) = #同時買A和B的紀錄 / #有買A的紀錄  
    Support(A->B) = #同時買A和B的紀錄 / #所有紀錄  
    只看Confidence會讓人不夠確定是否有足夠數據支持買A也買B的案例,所以可從Support觀察此關聯性是否成立.

### 5.4.3 Implementing association rules

    示範Python實做

### 5.4.4 Saving the association rules in the database

    示範sql table
    作者觀點: 必須另外建立version table,避免將所有關聯規則寫入同一個table,透過version id管理未來產生的新規則.

### 5.4.5 Running the association rules calculator

    示範Python實做

### 5.4.6  Using different events to create the association rules

    作者觀點: 除了購物籃分析之外,我們也可以透過分析瀏覽紀錄和最後購買商品之間,有可能找到些關聯法則.

### 6. The user (and content) who came in from the cold

### 6.1 What’s a cold start

    討論cold-start以及gray sheep問題.

### 6.1.1 Cold products

    作者提出的解法:
    1. 在網站上開設展示新商品的區塊.
    2. 在推薦清單中加入新商品，新商品若不被買單,讓新商品逐漸被decay掉.

### 6.1.2 Cold visitors

    作者提出的解法:
    1. 先讓用戶對特定商品打分數(非大部分網站的作法)
    2. 從用戶的搜尋字詞紀錄用戶想要的東西

    但作者也提到,需要多少資訊量才足夠得知用戶的偏好?
    解法: ch12 mixed hybrid.
    個人化推薦若不可行,可以改用非個人化的熱門推薦.

### 6.1.3 Gray sheep

    直接看英文比較好理解定義...
> A gray sheep is a user who has such an individual taste that even if there’s data, there are likely no other consumers—or very, very few people—who’ve bought any of the products the gray sheep has.

### 6.1.4 Let’s look at real-life examples

    這段作者舉生活例子說明用戶在網站上的行為對推薦系統的影響.
    不太理裡作者最後提到,outlier對association rule和cf的影響差異.

    後續作者舉例藝術類的商品更會有cold product和gray sheep問題,因為藝術類商品賣出數量少,也很難和其他商品有關聯.
    即便透過可以透過標籤(tag)增加訊息,但也很難去為藝術類商品訂定標籤.

### 6.1.5 What can you do about cold starts

    作者提到利用Content-based filtering解決cold-start問題.

### 6.2 Keeping track of vistors

    因為用戶是捉摸不定的(elusive).用戶很可能會在不同地點不同裝置進入網站.並且有時還不會登入帳號.如何得知用戶身份是個問題.

### 6.2.1 Persisting anonymous users

    這章節討論用cookie和session追蹤用戶身份.

### 6.3 Addressing cold-start problems with algorithms

    作者見解: cold starts目前仍沒有完美的解法.機器學習也是要依賴資料才會有合理的結果.
    作者在這章節採用非機器學習的方法探討解決cold starts問題.

### 6.3.1 Using association rules to create recs for cold users

    如題,根據用戶看過或買過的商品給予關聯推薦.
    但後續提到透過**加權平均**重新排序推薦就不太明白作者的意思了.

### 6.3.2 Using domain knowledge and business rules

    此章節討論如何避免算法推薦不適當的商品.
    作者觀點: 不要直接在算法內寫入限制式,而是讓算法得出的結果透過規則篩選成最終的推薦清單.
![figure 6.6](src/figure_6_6.png)

### 6.3.3 Using segments

    分群策略替用戶篩選商品
    1. 地理位置: 只展示當地有銷售的商品.
    2. 性別: 服飾店依據性別篩選展示商品.
    3. 年齡: 電影依據用戶年齡推薦星際大戰或是漫威電影.
    上述可稱為 demographic recommendation

    作者提到,我們可以不只透過用戶特徵還有其他行為進行分群,這時可以用機器學習的clustering來建立難以直接辨別的群體.

### 6.3.4 Using categories to get around the gray sheep problem and how to introduce cold product

    如果只靠商品很難找到關聯的其他商品,作者建議採用商品的metadata建立關聯規則.

### 6.4 Those who doesn’t ask, won’t know

    或許我們可以透過直接詢問用戶而得知他們偏好.例如讓用戶決定要熱門或冷門商品推薦,因此得知要將用戶規為哪種群體或進行關聯分析.

    作者提到了**activate learning**可以解決此問題.但在此書不會介紹

### 6.4.1 When the visitor is no longer new

    當用戶已經不再是新用戶時,我們可以改變推薦策略.
    作者舉例,我們推薦改成只依據最近幾周或是最近20個事件.

### 6.5 Using association rules to start recommending things fast

    示範Python實做

### 6.5.1 Find the collected items

### 6.5.2 Retrieve association rules and order them according to confidence

### 6.5.3 Displaying the recs

### 6.5.4 Implementation evaluation

## Part 2 Recommender Algorithms

### 7. Finding similarities among users and among content

    此章節討論相似度的計算

### 7.1 Why similarity

    我們該去思考清楚,何謂相似?
    兩個人都喜歡科幻片,但這代表他們都會喜歡同一部科幻電影嗎?

    除了對商品的rating之外,用戶資訊(demographics)和商品資訊(metadata)也可以拿來比較相似度.

### 7.1.1 What’s a similarity function

    兩物品間的相似度可以轉化為距離來解釋.

### 7.2 Essential similarity functions

- Jaccard similarity
- Consine similarity
- Pearson similarity

![table 7.1](src/table_7_1.png)

### 7.2.1 Jaccard distance

    similarity = #同時有看兩部電影的用戶 / #看過其中一部電影的用戶

    作者觀點: 相似度0.5代表相似度很高嗎?不見得,還是要依據doamin決定.

### 7.2.2 Measuring distance with Lp-norms

- L1-norm  
    算法也稱之為曼哈頓距離(Manhattan distance).  

    計算二人對某一項商品的相似度:  
    $distance(u1,u2) = |r_{u1,i1} - r_{u2,i1}|$  

    $similarity(u1,u2) = \frac{1}{|r_{u1,i1}-r_{u2,i1}| + 1}$

    計算二人對所有商品的相似度:  
    $Sum\ of\ Absolute\ Difference\ (SAD)=\sum_{i=1}^{N}|r_{u1,i} - r_{u2,i}|$  

    MAE其實也是將l1-norm加總的值再除以商品數量:  
    $Mean\ absolute\ error\ (MAE)=\frac{1}{n}\sum_{i=1}^{N}|r_{u1,i} - r_{u2,i}|$
- L2-norm  
    算法起源於畢氏定理(Pythagorean theorem).

    計算二人對某一項商品的距離:  
    $distance(u1,u2)=\|r_{u1,i1} - r_{u2,i1}\|_2=\sqrt{\sum_{i=1}^{n}|r_{u1,i} - r_{u2,i}|^2}$

    機器學習常見的MSE:  
    $Root\ Mean\ Squared\ Error\ (RMSE)=\frac{1}{n}\sum_{i=1}^{n}|r_{u1,i} - r_{u2,i}|^2$

### 7.2.3 Cosine similarity

$Cosine\ Similarity=TBD...$

如果資料沒有限定rating scale,建議改採以下算法:  
$Adjusted\ Cosine\ Similarity=TBD...$

### 7.2.4 Finding similarity with Pearson’s correlation coefficient

作者用圖表說明pearson的用途,可以得知二位用戶的評分趨勢是否正相關.

![figure 7.6](src/figure_7_6.png)

$similarity=TBD...$

### 7.2.5 Test running a Pearson similarity

    作者示範pearson過程,還考慮到normalize以及只計算用戶間的有交集商品.

**Note on implementation**  
Remember when you have an array of ratings similar to Jesper’s, you need to calculate the mean of the items that were rated. If you look at Jesper’s ratings, you’d have the following array [4, 3, 0, 4, 4]. If you use a normal mean operation on that, you’ll
get 15 / 5. Although the zero doesn’t affect the sum (15), it still counts in the total
number of ratings (5).

### 7.2.6 Pearson correlation is similar to cosine

    adjusted cosine similarity 和 pearson correltion公式看似一樣.
    但是pearson correlation只計算用戶有共同購買過得商品.

    當一個用戶評分過很少的商品,令個用戶評分過很多商品,用cosine similarity計算會得到接近於0的相似度.

### 7.3 k-means clustering

    CF的問題在於要找和某一位用戶的相似用戶,需要把全部用戶都算過一次...
    作者認為,可以透過分群減少計算的成本

### 7.3.1 The k-means clustering algorithm

    先幫用用找最相似的群體,然後只計算該群內的相似用戶.

### 7.3.2 Translating k-means clustering into Python

    示範Python實做

**NOTEOF WARNING**  
K-means clustering, as with most other machine-learning algorithms, is difficult to make work correctly. It often responds with results that aren’t understandable or usable.

作者認為clustering可以應用的地方:  
    1.找出現有用戶所在群體,並觀察同一群的其他用戶.  
    2.歸類新用戶該屬於哪一群.

### 7.4 Implementing similarities

    示範SQL實做

### 7.4.1 Implementing the similarity in the MovieGEEKs site

    示範Python實做

### 7.4.2 Implementing the clustering in the MovieGEEKs site

    示範Python實做

作者認為還是要仔細觀察clustering的結果,避免產出奇怪的推薦.

### 8. Collaborative filteringin the neighborhood

### 8.1 Collaborative filtering: A history lesson

    說歷史

### 8.1.1 When information became collaboratively filtered

    1992年CF剛開始是被設計用來篩選,解決資訊過載的問題.

### 8.1.2 Helping each other

    CF有效的前提假設:  
    1.我們相信可以透過群體可以更了解彼此
    2.用戶的品味不會隨著時間變化

    說明user-based和item-based cf:
![figure 8.2](src/figure_8_2.png)

### 8.1.3 The rating matrix

    說明matrix

### 8.1.4 The collaborative filtering pipeline

    此章節簡單說明CF流程.
![figure 8.3](src/figure_8_3.png)

### 8.1.5 Should you use user-user or item-item collaborative filtering

    該選哪種cf?
    1.根據研究,商品通常只會被相同類型的用戶購買,所以比較穩定.
    2.選擇計算相似度成本比較低的.

### 8.1.6 Data requirements

    CF資料限制:
        1.用戶從沒給過rating,不會為他產生推薦.
        2.用戶和其他用戶的重疊商品很少,不會有好的推薦.
    作者強調,即使CF不需要domain knowledge,但是要建立一個好的推薦系統仍需要它.

### 8.2 Calculating recommendations

    略

### 8.3 Calculating similarities

    略

### 8.4 Amazon’s algorithm to precalculate item similarity

    略

**BEWARE OF THE 1 OR 2 ITEMS IN COMMON PROBLEM**  
    作者舉例常見的CF問題:  
    1.商品被太少用戶評分:如果只有一個rating,那平均值就等於rating,在計算average cosine similarity時,會讓分子為0,導致算法失效.
    2.用戶對太少商品評分: 不理解作者的解釋.  
    3.用戶太多商品的重疊會導致推薦的商品太過於general.  
    4.用戶太少商品也導致無法推薦.

**note**  
Remember, it’s a **tradeoff**. Narrow the number of users and you risk not finding any content to recommend. Make it too wide and you risk that the recommendations contain content that’s outside the user’s taste.

### 8.5 Ways to select the neighborhood

    作者定義**Neighborhood**: 和用戶現在尋找有相關的東西.

    3種找到Neighborhood的方法:
    1.Clustering
        潛在問題:cluster形狀怪異或找不到相近的群.
        作者解法:測試沒有clustering的推薦效果.
    2.Top-N
        強制生出N個推薦商品,缺點是也會包含到相似度極差的商品.
    3.Threshold
        只推薦相似距離在門檻內的商品

---
**NOTE**

Choosing between Top-N and threshold is choosing between quantity and quality.  
Choose the threshold method for quality; Top-N for quantity

---

![Figure 8.7](src/figure_8_7.png)

### 8.6 Finding the right neighborhood

    略

### 8.7 Ways to calculate predicted ratings

    此章節討論預測rating的作法,作者將方法分成迴歸以及分類2種.
    但在這段沒細講作法.

### 8.8 Prediction with item-based filtering

### 8.8.1 Computing item predictions

    作者透過加權平均法預測rating,我覺得這是值得參考的作法

![formula 1](src/formula_1.png)

### 8.9 Cold-start problems

    關於item cold start,作者提出exploit/explore方法.
    像擲骰子般隨機決定是否要在推薦清單加入new item.

### 8.10 A few words on machine learning terms

    只簡單說明一些ML名詞.
    - Offline
    - Model
    - Training

### 8.11 Collaborative filtering on the MovieGEEKs site

    作者思路:
    1. 先計算出相似矩陣
    2. 將相似度低於門檻的直接設0
    3. 將用戶重疊數量少的直接設0
    ...

### 8.11.1 Item-based filtering

    示範Python實作

### 8.12 What’s the difference between association rule recs and collaborative recs

    作者觀點: association rule只看購物籃裡面放什麼東西.但是CF看的是隨時間變化用戶買了什麼或對商品的評分行為.

### 8.13 Levers to fiddle with for collaborative filtering

    CF建立推薦系統必須考慮的點:
    1. rating
        - 只考慮正面rating嗎?
        - 只考慮最近的行為?
        - 要如何normalize rating?
    2. similarity
        - 是否要考慮overlap?
        - 是否只需要考慮只有positive rating的商品相似度?
    3. neighborhood
        - 要用哪種方法計算?
        - 如何決定neighborhood範圍(threshold/top-N)
    4. predict
        - 用回歸或是分類?
        - 用加權平均?
    5. recommendation
        - 應該將商品全部推出,還是只推出特定門檻以上的商品?

    作者觀點:
    overlap門檻設太高,會導致只推薦熱門商品,反而失去個人化效果.

### 8.14 Pros and cons of collaborative filtering

    - Sparsity: 老問題.
    - Gray Sheep: 用戶特殊偏好.
    - Number of ratings: 我們都期望用戶的rating數量夠多,推薦才會有效,但實際情形卻無法收集到足夠資料才會開時跑推薦系統(cold start problem).
    - Similarity: 因為沒有加入內容資訊,只依靠用戶的行為進行相似度計算,會讓熱賣商品會持續一直出現推薦清單上.

### 9. Evaluating and testing your recommender

    此章節談論如何衡量(evaluate)推薦系統.
    作者觀點: 大部分人都認同推薦系統最終還是要在現實環境中評估效果,不過重要的是要知道你的推薦系統是否朝著正確的方向發展.

### 9.1 Business wants lift, cross-sales, up-sales, and conversions

![figure 9.1](src/figure_9_1.png)

    作者觀點： 隨著新的資料不斷更新,我們也需要不斷地**維護**,**追蹤**推薦系統的表現有沒有變差.

    - verify: -
    - regresion test: 增加資料並觀察系統的表現是否變差.
    - friends and family: 不太懂作者的說明.
    - exploit/explore: 根據用戶對推薦系統的反應決定該用哪個算法.

### 9.2 Why is it important to evaluate

    為了證明推薦系統是有效的!
    作者在這段也提出一些值得思考的點
    1. 系統經常會過於over-fitting利害關係人的偏好:
    2. 重要的是要清楚地了解您所問的問題。

    測試的第一步: hypothesis
    舉例: 推薦系統B的點擊次數會高於推薦系統A(CTR)

### 9.3 How to interpret user behavior

    我們該如何解讀用戶對推薦結果的反應?
    1. 顯示推薦結果一次,但沒點擊: 假設其實用戶沒看到推薦結果?
    2. 顯示多次推薦結果但都沒點擊: 假設用戶對這沒興趣?
    3. 最理想行為: 顯示一次並點擊
    4. 對推薦結果點擊多次: 該如何解釋這現象?作者認為要依據domain決定推薦是否成功.

### 9.4 What to measure

    用戶角度:
    1. The site understands my tastes
    2. The site gives me a nice variety of recommendations
    3. The site surprises me
    店家角度:
    1. The site covers everything in the catalog.
    後續章節說明該如何轉化為可衡量系統的計算方法.

### 9.4.1 Understanding my taste: Minimizing prediction error

    略

### 9.4.2 Diversity

    馬太效應(the Matthew effect): 大者恆大,推薦系統只推薦你熱門的東西.
    過濾氣泡(filter bubble): 同溫層效應,讓你還沒體驗過得商品沒機會推薦給你
    作者觀點:同時兼顧個人化推薦以及diversity是困難的.(推薦論文: Improving Recommendation Diversity)

### 9.4.3 Coverage

    顧及Diversity其實也代表顧及了Coverage.從店家角度看,其實也希望各類商品都有機會被列入推薦.

    作者將Coverage又細分成user coverage和content coverage:
    user coverage: 推薦清單不為空的人數 / 總人數
    content coverage: 某類別商品出現在清單的數量 / 某類別中的總商品數

### 9.4.4 Serendipity

    因為Serendipity比較主觀而且很難去計算,所以建議是在比較不在乎推薦的產出再考慮這作法.

### 9.5 Before implementing the recommender

    - 驗證演算法
    - 驗證資料
    - 迴歸測試

### 9.5.1 Verify the algorithm

    透過簡單的資料集驗證是否有得到預期的結果.
    而資料的部份也要考量取得的可行性以及是否可能產出.

### 9.5.2 Regression testing

    回歸測試,不太了解作者的解釋,或許網路上的資訊比較清楚.
    作者舉例:
    將CF的流程拆分,讓算法能分開測試個別的預期結果.並且持續地增加未知資料集測試.

### 9.6 Types of evaluation

    - offline
    - controlled user
    - online
    每個衡量方法都有各自目的.

### 9.7 Offline evaluation

    對推薦系統而言,offline並不是好的評估方法,但目前也沒有更好作法,除非我們是Netflix能執行A/B test.

### 9.7.1 What to do when the algorithm doesn’t produce any recommendations

    當該用戶沒有推薦商品時,該去思考該產生哪些推薦給他.

### 9.8 Offline experiments

    - MAE
    - MSE
    - RMSE

作者透過下圖說明不同計算方法的差異性,一個人有多個評分，另   個人只有1個一個評分.  

(1+1+1+1+3)/5 = 1.4 -> 常見的測試方法,但是,看不出來推薦系統對rating較少用戶的影響.
((1+1+1+1)/4+3/1) = 4

![figure 9.6](src/figure_9_6.png)

作者觀點: 對用戶而言,預測rating準不準是其次,重點應該是推薦清單內的商品是否是他有興趣的商品,有可能給低分但卻是用戶有興趣的商品.

**MEASURING DECISION-SUPPORT METRICS**
觀察商品是否和用戶相關或確實被消費.

- Precisionfiltering
- Recall

![figure 9.7](src/figure_9_7.png)

**MEASURING RANKING METRICS**
考量排序

- MEAN AVERAGE PRECISION (MAP)

作者用很好懂的圖例書說明AP計算方式:

![figure 9.8](src/figure_9_8.png)

**DISCOUNTED CUMULATIVE GAIN**
這篇對DCG和nDCG的說明比較好懂  
[http://sofasofa.io/forum_main_post.php?postid=1002561](http://sofasofa.io/forum_main_post.php?postid=1002561)

$DCG = \sum{i=1}^{k}\frac{2^{rel[i]}-1}{\log_{2}{([i]+2)}}$

**NORMALIZED DISCOUNTED CUMULATIVE GAIN**
$nDCG = \frac{DCG}{IDCG}$

### 9.8.1 Preparing the data for the experiment

**HANDLING NEW USERS**  
作者也認為直接拿掉紀錄過少的user.

**SAMPLING**  
stratified sampling  
當數據量過大時,以維持資料分佈的前提下取樣,e.g., 維持10%男性90%女性.

**FINDING GOOD CANDIDATES FOR THE EVALUATION**
依據domain決定overlap等等門檻的值.

**SPLITTING THE DATA INTO TEST, TRAINING, AND VALIDATION SETS**  
train: 訓練集  
validation: 優化參數用  
test: 測試集,只能被用一次

作者接下來開始討論切分資料的方法,以及對應的驗證方法.

**RANDOM SPLITTING**  
作者觀點:隨機切分資料會導致很多問題.

1. rating在推薦系統中需要加入時間因素(time decay).
2. 切分後,user只出現在validation set.
3. 切分後,item指出現在validation set.

**SPLIT BY TIME**  
除非資料集沒有時間欄位,不然這是個合理作法.  
若用戶都指出現在test set裡,作者認為可以直接忽略那些無法驗證的用戶.

**SPLIT BY USERS**  
取用戶的前n個rating為train set,其餘為test set.

**CROSS-VALIDATION**  
作者提出的是傳統的cv,可惜沒提到和時間序列相關的cv.  
[https://towardsdatascience.com/time-series-nested-cross-validation-76adba623eb9
](https://towardsdatascience.com/time-series-nested-cross-validation-76adba623eb9)

## 9.9 Implementing the experiment in MovieGEEKs

    Python示範

## 9.10 Evaluating the test set

    Python示範

## 9.10.1 Starting out with the baseline predictor

    Python示範

## 9.10.2 Finding the right parameters

    Python示範

## 9.11 Online evaluation

- controlled experiments
- A/B testing

### 9.11.1 Controlled experiments

作者提出2種方法

1. 邀請受試者填寫問券(紀錄rating),給他多個推薦清單,並觀察用戶的選擇.
2. **FAMILY AND FRIENDS**,由受試者告訴我們推薦系統該如何改進,雖然他們可能不懂算法,但他們是代表end-user的角色.

### 9.11.2 A/B testing

因為太多無法控制的因素會影響用戶的決定,無法得知系統到底有沒有效.所以需要透過A/B testing去確認效果.

![figure 9.16](src/figure_9_16.png)

作者也提到A/B testing也有潛在問題:

1. 測試組的用戶可能因為不好的使用體驗而不再使用你的網站.

Read more about Netflix’s A/B test at [http://mng.bz/FJUB](http://mng.bz/FJUB)

### 9.12 Continuous testing with exploit/explore

當進行A/B testing時,會有一種情況是某個算法表現比較好,但有時是另個算法比較好.作者認為系統推薦熱門商品通常表現比較好
,但列出些新東西或許會有額外的獲得?作者用吃角子老虎機說明exploit/expore但我無法把他和推薦算法聯想在一起...

### 9.12.1 Feedback loops

作者觀點:

1. 透過loop過程中,觀察用戶是否只採用系統的推薦,或是自行找尋過哪些商品..等行為.
2. 作者認為仍需加入新商品在loop中,應該事前一章節提到的exploit/explore.
3. 很難透過一次性的評估就可以得知系統的好壞,第一次的評估只是作為未來評估的標準.

![figure 9.18](src/figure_9_18.png)

### **SUMMARY**

- 實做推薦系統之前就要先考慮到測試.
- Regression test可以避免無意中添加的錯誤.
- 作者認為Serendipity雖然很難衡量,但有其重要性.

### 10 Content-based filtering

本章節主要會討論的算法是TF-IDF和LDA.

### 10.1 Descriptive example

略

### 10.2 Content-based filtering

- 這章節概略說明CBF的流程

1. Content analyzer: 計算商品間相似度(事先訓練模型).
2. User profiler: 用戶過去的消費紀錄.
3. Item retriever: 回傳用戶消費紀錄相似商品.

![figure 10.4](src/figure_10_4.png)

![figure 10.5](src/figure_10_5.png)

### 10.3 Content analyzer

### 10.3.1 Feature extraction for the item profile

- 作者將商品Metadata分為2種: Facts, Tags
- Facts: 客觀資訊,不會改變.
- Tags: 可能有主觀意識的資訊.

### 10.3.2 Categorical data with small numbers

- 不常出現的資訊,作者認為要考量是否要列為metadata,因為在計算相似度時也難有結果.

### 10.3.3 Converting the year to a comparable feature

- 作者觀點: 將數值轉換為容易比較特徵. e.g. normalization.

![figure 10.7](src/figure_10_7.png)

### 10.4 Extracting metadata from descriptions

### 10.4.1 Preparing descriptions

- BAG-OF-WORDS model
- Remove stop words
- REMOVING THE HIGH AND LOWS
- STEMMING AND LEMMATIZING

### 10.5 Finding important words with TF-IDF

- tf(word ,document) = 1 + log(word frequency)
- idf(term) = log(Total number of docs/Number of docs containing term)
- 雖然TF-IDF已經不是主題模型的首選算法了,但TF-IDF可以應用在LDA的前處理工作.
- 作者最後也提到了Okapi BM25但沒有討論細節.

### 10.6 Topic modeling using the LDA

![figure 10.8](src/figure_10_8.png)

- GENERATIVE MODEL EXAMPLE  
很難理解作者用圖形說明LDA,直接看圖還比較好懂.

![figure 10.12](src/figure_10_12.png)

- GENERATING THE TOPICS  
簡單說明LDA的input和output

![figure 10.13](src/figure_10_13.png)

- GIBBS SAMPLING  
The goal is now to connect words with topics and documents with topics. Let’s start with the topics and words as shown in figure 10.14

- LDA MODEL  
作者只簡單說明輸出的結果

![figure 10.14](src/figure_10_14.png)

### 10.6.1 What knobs can you turn to tweak the LDA

- 作者介紹了視覺化套件**pyLDAvis**
- 說明調整alpha,beta的影響

If you enter a high alpha, then you’ll distribute each document over many topics; low alpha distributes only a few topics.  
The same is true for beta: a high beta leads to topics being more similar because the probabilities will be distributed on more words that are used to describe each
topic.

### 10.7 Finding similar content

- 計算2文件間的相似度: 把probability distribution視為vector透過cosine similarity計算相似度.
- 這可以解決cold-start問題

### 10.8 Creating the user profile

### 10.8.1 Creating the user profile with LDA

值得參考的流程:

1. 取得用戶消費過的商品清單
2. 針對每個商品:  
透過LDA取得相似商品  
根據similarity和rating得出新的rating
3. 依據rating排序
4. 依據相似度排序(optional)

參考論文:[Improving Collaborative Filtering Based Recommenders Using Topic Modelling](https://arxiv.org/abs/1402.6238)

### 10.8.2 Creating the user profile with TF-IDF

作者作法:

1. 事先抽取電影的所有特徵
2. 用戶看過的電影特徵預設值*rating,並加總就是user profile

這段作者沒講清楚流程,也沒提到TF-IDF,有點莫名其妙.

### 10.9 Content-based recommendations in MovieGEEKs

Python實做示範

### 10.10 Evaluation of the content-based recommender

略

### 10.11 Pros and cons of content-based filtering

優點:

1. 解決 item cold start
2. 不被商品熱門程度影響

缺點:

1. Conflates liking with importance:
(很難用中文說明..)if I like Sandra Bullock in action films
and Meg Ryan in comedies, but if I hate Meg Ryan in action films and Sandra Bullock in comedies, there’s no way for that to be captured in the feature vector
2. No serendipity
3. Limited understanding of content: 無法完全掌握到商品的所有特徵

### 11 Finding hidden genres with matrix factorization

此章節主要介紹SVD和funk-SVD

作者在文中提到一點是我之前沒想過的,用SVD並非直接要當作推薦算法,目的可以改成只要SVD產生的latent vectors,可以當作其他模型的pre-train vector!

### 11.1 Sometimes it’s good to reduce the amount of data

Talking about hidden things makes it sound as if there’s always something there, so it’s worthwhile to point out that if your data is random or it doesn’t have any signal, the reduction won’t provide any additional information.

### 11.2 Example of what you want to solve

不懂作者想表達的意思

### 11.3 A whiff of linear algebra

向量矩陣基本介紹

### 11.4 Constructing the factorization using SVD

解釋SVD元素

- U: User feature matrix.
- Sigma: Weights diagonal.可以解釋成每個維度的重要程度
- V: Item feature matrix.

**SOLVING THE PROBLEM OF THE ZEROS IN THE RATING MATRIX USING IMPUTATION**  
Imputation  
當資料有0的數值,作者提到用``填補平均值``,``normzlize``等方法處理.

### 11.4.1 Adding a new user by folding in

預測新用戶/新商品的latent vetor

$u_{kim} = r_{k}V^T\Sigma^{-1}$  
$i_{kim} = r_{k}U\Sigma^{-1}$

作者提到,SVD有方法預測新用戶的latent vaetor.當用戶的購買紀錄過少,其實沒必要為他重造SVD,我們可以用上述提及的作法來為他推薦商品.

### 11.4.2 How to do recommendations with SVD

作者提出3種SVD的推薦方法

1. 直接透過SVD還原矩陣,計算rating最大的商品進行推薦.
2. 透過矩陣I,找用戶過去購買紀錄的相似商品
3. 採用CF手法,根據矩陣I內的商品相似度,預測rating

**PROBLEMS WITH SVD**  
需要經常更新SVD  
無法解釋SVD的rating預測結果

### 11.4.3 Baseline Predicitons

考量到**rating bias**,作者提出另類填補缺值的作法:  
$b_{ui} = \mu + b_{u} + b_{i}$

base prediction = average of all ratings + user bias + item bias

**FINDING BIAS BY FINDING LEAST SQUARES**  

$$min_{b}\sum_{u,i\in{K}}{(r_{u,i}-\mu-b_{u}-b_{i})^{2}}$$

**A SIMPLER WAY TO CALCULATE BIASES**

$$b_{u} = \sum_{i\in{I_{u}}}{(r_{u,i}-\mu)} / |I_{i}|$$

$$b_{i} = \sum_{u\in{U_{i}}}{(r_{u,i}-b_{u}-\mu)} / |U_{i}|$$

baseline prediction能處理SVD空值問題,甚至是其他矩分解的相關作法

### 11.4.4 Temporal dynamic

考量bias會隨著時間而變化,所以rating也會隨著時間變化.

$b_{ui}(t) = \mu + b_{u}(t) + b_{i}(t)$

A good place to start is **Collaborative Filtering with Temporal Dynamics** by Yehuda Koren

### 11.5 Constructing the factorization using Funk SVD

Funk SVD = regularized SVD

### 11.5.1 Root Mean Squared Error

採用RMSE原因:  
You want the equation to penalize big errors, so you take the square of the difference. But you still want the error to come out to be on the same scale of the ratings so with that you end up with the RMSE

### 11.5.2 Gradient descent

略

### 11.5.3 Stochastic gradient descent

略

### 11.5.4 And finally, to the factorization

For each rating $r_{ui}$ in ratings, calculate

- $e_{ui} = r_{ui} - q_{i}p_{u}$
- $q_{i} = q_{i} + \gamma *(e_{ui}* p_{u} - \lambda * q_{i})$
- $p_{u} = p_{u} + \gamma *(e_{ui}* q_{i} - \lambda * p_{u})$

### 11.5.5 Adding biases

作者用圖片清楚說明Funk-SVD的組成要素

![figure 11.16](src/figure_11_16.png)

### 11.5.6 How to start and when to stop

> The difference between what you’re doing here and the evaluation you looked at in chapter 9 is that you’re not only looking for more precision and recall (that’s something you’ll look at when you’ve finished fine-tuning the parameters), but you’re also concentrating on training the algorithm to understand the domain. That sounds fancy, but let’s remember your goal, which is to provide recommendations for users online.

作者提出4個在算法中需要思考的參數:

- **Initialize the factors?**  
Decides where the gradient descend should start walking.
- **Learning rate?**  
How fast you should move at each step.
- **Regularization?**  
How much should the algorithm regularize the errors? Should it be the same for all the expressions or should you split it into factors and bias?
- **How many iterations?**  
How specialized should the algorithm become to the training data?

### 11.6 Doing recommendations with Funk SVD

作者提出2種推薦方法: BRUTE FORCE, NEIGHBORHOODS.

- BRUTE FORCE RECOMMENDATION CALCULATION  
顧名思義就是填滿原始user-item matrix.  
缺點是計算所有rating需要大量成本,只能靠減少factor數量或bias來降低計算成本.

- NEIGHBORHOODS RECOMMENDATION CALCULATION  
計算user和所有item的相似度(因為user, item factors在同個向量空間裡).  
但是,作者也提到我們必須要確定用戶是否有特定偏好.當用戶都偏好各式各樣的商品,會導致沒有商品向量會和用戶向量相似.  
可以用圖片理解user,item factors共存  
![figure 11.19](src/figure_11_19.png)

### 11.7 Funk SVD implementation in MovieGEEKs

Python實做示範.

### 11.7.1 What to do with outliers

不太明白作者對outlier的處理方法.

### 11.7.2 Keeping the model up to date

模型更新取決於資料的變動幅度.

作者見解:

1. 每次模型的更新,可以觀察用戶的偏好是否改變.
2. 注意用戶是否有消費過上次產生的推薦結果,注意否還要再推薦給他.
3. implicit data,我覺得這段意義不明.

### 11.7.3 Faster implementation

作者提到**ALS**但沒說細節

### 11.8 Explicit vs. implicit data

作者推薦論文:  
[Collaborative Filtering for Implicit Feedback Datasets” by Yifan Hu et al.](http://yifanhu.net/PUB/cf.pdf)

作者推薦github:  
[benfred/implicit](https://github.com/benfred/implicit)

### 11.9 Evaluation

作者作法:  
作者準備2份測試集,一份是購買數量夠多的用戶,一份是全體用戶,並觀察2種資料集的MAP.  
我的見解:  
未來實驗設計可以將不同類別的會員分開(e.g.消費頻率),並觀察模型應用在不同會員群的預測效果.

### 11.10 Levers to fiddle with for Funk SVD

此章節探討Funk-SVD的參數以及調整方向.

1. 參數通常都是會互相影響(dependent),所以通常需要藉由grid_search類似方式找尋最佳參數組合.
2. 透過調整regularization避免latent factor過大.
3. Bias initialization. 必須要先知道資料的rating scale以及計算整體平均,可以得知如何設定bias的初始值.  
e.g. 資料集的rating scale在1-10之間,平均rating是7,那麼user bias和item bias就都不應該大於1.5以上.
4. 不太了解作者說法...
    > Determining the right number of iterations is also a matter of taste. If you run through too many iterations on the early factors, you risk the chance that all signals will be pushed into the first dimension, while the remaining factors will only have a small signal.

### SUMMARY

1. SVD的缺點在於無法處理稀疏矩陣.
2. Baseline predictors可用來填補缺值以及計算biases.
3. Funk-SVD可應用於稀疏矩陣.
4. Gradient Descent,SGD應用於優化SVD參數

### 12 Taking the best of all algorithms: Implementing hybrid recommenders

下面的圖簡略說明混搭流程.  
不同資料內容可以對應建立不同的推薦系統,混搭可彌補每個系統自身的缺點,並有機會提高推薦效果.

![figure 12.1](src/figure_12_1.png)

### 12.1 The confused world of hybrids

作者將混搭分為3類:

![figure 12.2](src/figure_12_2.png)

### 12.2 The monolithic

定義: 在模型有包含多個元件並可拆分情況下,我們將多個模型的元件混搭,或是增加某個模型的元件.

> A monolithic recommender mixes components from different recommenders or even adds new steps to improve overall performance.

下圖作者以CBF混搭CF為例.

![figure 12.3](src/figure_12_3.png)

### 12.2.1 Mixing content-based features with behavioral data to improve collaborative filtering recommenders

作者觀點: 混搭模型的重點就是要利用到各種資料的特性.

作者以CBF+CF舉例:  
當矩陣過於稀疏,導致計算相似度時,沒有足夠的rating可以紀算相似度,作者在前處理階段增加一位new user,並對特定類型的電影訂定rating,這樣一來讓後續可以順利預測rating.

### 12.3 Mixed hybrid recommender

從下圖可知作者如何定義Mixed hybrid.

1. 情境1  
因為個別熱銷可能會有推薦商品數不足問題,此時我們可以藉助全體熱銷補足清單數量.

2. 情境2  
混合將多個算法產生的推薦商品,並根據分數排序出新的推薦清單.  
**注意!必須考量分數是否在同個scale上,是否需要normalize.**

![figure 12.4](src/figure_12_4.png)

作者另外推薦的論文: [Content-Boosted Collaborative Filtering for Improved Recommendations](www.cs.utexas.edu/~ai-lab/pubs/cbcf-aaai-02.pdf)

### 12.4 The ensemble

作者說明mixed和ensemble的差異
> The difference between an ensemble and a mixed recommender is that the hybrid might not show anything of the result for one recommender, while the mixed hybrid always shows everything.

從下圖可看出,ensemble是將多個算法產生的推薦結果送入另個hybrid算法,並產出全新的推薦清單.  
作者舉例,將hybrid算法視為``bagging``,將多個算法產出的商品清單中,出現次數最高的優先列為推薦商品.

![figure 12.6](src/figure_12_6.png)

### 12.4.1 Switched ensemble recommender

作者定義switched ensemble:  
根據條件,從多個算法中挑選其中一個算法的推薦結果.

我覺得這章節對於討論商業情境和對應的推薦算法,有很大的幫助.

舉例:

- 根據地理位置: 不同的城市應用不同的推薦算法.  
- 根據時間: 早上給一種算法結果;晚上給另一組算法結果.  
- 根據用戶行為: 消費次數少的消費者用``關聯分析``;消費次數多的消費者用``CF推薦``.
- 根據用戶行為: 如果是有登入帳號的消費者採用一種推薦算法;沒有登入的訪客採用另種推薦算法.

### 12.4.2 Weighted ensemble recommender

作者定義: 透過加權相加計算最終的rating

但是要如何得知適合的weights?作者認為可以用**linear regression**.

![figure 12.9](src/figure_12_9.png)

### 12.4.3 Linear regression

作者在這章節沒提太多迴歸細節.

### 12.5 Feature-weighted linear stacking (FWLS)

作者定義: 將上個章節提到的權重改成用function.  
有點像類神經網路的activation function.

參考論文: [Feature-Weighted Linear Stacking](https://arxiv.org/pdf/0911.0460.pdf)

### 12.5.1 Meta features: Weights as functions

$b(u,i) = f(u,i)*r{cf}(u,i)+g(u,i)*r_{cb}(u,i)$

![figure 12.10](src/figure_12_10.png)

### 12.5.2 The algorithm

下圖用數學公式說明FWLS. 最後的v是由linear regression決定.

![fwls](src/fwls.png)

下圖用案例說明FWLS.

![figure_12_12](src/figure_12_12.png)

下圖說明FWLS pipeline

![figure_12_13](src/figure_12_13.png)

### 12.6 Implementation

Python實做,未來如果要實做,值得參考這段作法.

### Summary

1. 推薦系統可以藉由增加多個算法來優化.
2. 混搭推薦系統可以結合多個推薦系統的長處得到更好的結果.
3. 並非複雜的功能能在真實世界運作.Netflix優勝的算法就是因此無法正式上線使用.
4. FWLS

## 13 Ranking and learning to rank
