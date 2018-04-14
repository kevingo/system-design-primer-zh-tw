# 設計 Twitter timeline 和搜尋功能

*注意：本文件某些連結直接連到 [系統設計主題的索引](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E7%B3%BB%E7%B5%B1%E8%A8%AD%E8%A8%88%E4%B8%BB%E9%A1%8C%E7%9A%84%E7%B4%A2%E5%BC%95) 來避免重複的內容。你可以參考連結來取得相關重點、設計的取捨和選擇。*

**設計 Facebook feed** 以及**搜尋**功能和本篇是類似的問題。

## 步驟一：描述使用情境與限制

> 蒐集問題的需求、資訊和範圍。
> 透過詢問問題來瞭解使用情境和限制。
> 討論你的假設。

在這裡沒有面試者會幫你釐清上面的問題，所以我們會預先定義一些使用情境和限制。

### 使用情境

#### 我們將所要解決的問題限縮在以下範圍

* **使用者**撰寫一則 tweet 訊息
    * **服務本身**會透過通知機制和 email 來推送 tweets 給跟隨者
*  **使用者**會瀏覽 timeline (使用者的活動狀況)
* **使用者**檢視自己的 timeline 首頁 (會顯示其跟隨者的活動狀態)
* **使用者**搜尋關鍵字
* **服務**本身為高可用

#### 不包含在此範圍

* **服務本身**將 tweets 推送到 Twitter Firehose 或其他串流服務
* **服務本身**根據使用者的隱私設定來決定 tweets 的可見性
    * 如果使用者沒有追蹤被回覆的人，隱藏 @reply 的 tweet
    * 這是根據 '隱藏 retweets' 的設定
* 分析功能

### 限制與假設

#### 狀態假設

一般假設

* 流量不是均勻分布的
* 撰寫一則 tweet 是很快速的
    * 除非你有數百萬個跟隨者，否則向所有的跟隨者發送 tweets 應該很快
* 一億個活躍使用者
* 每天 5 億個 tweets 或每月 150 億個 tweets
    * 每則 tweet 平均被轉發給 10 個使用者
    * 平均每天發送 50 億則 tweets
    * 每個月平個發送 1500 億則 tweets
* 每月 250 億次讀取請求
* 每月 100 億次搜尋請求

Timeline

* 瀏覽 timeline 應該要很快
* Twitter 服務的讀取請求應該大於寫入請求
    * 架構上應該要朝向快速讀取 tweets 來設計
* 採集 tweets 是以寫入請求為主

搜尋

* 搜尋應該要很快
* 搜尋主要是以讀取請求為主

#### 計算使用量

**向你的面試人員詢問你是否可以用比較粗略的方式來計算使用量**

* 每一則 tweet 的大小：
    * `tweet_id` - 8 bytes
    * `user_id` - 32 bytes
    * `text` - 140 bytes
    * `media` - 10 KB average
    * 總共：~10 KB
* 每月新 tweets 的大小為 150 TB
    * 每條 tweet 10 KB * 每天 5 億條 * 每月 30 天
    * 三年總共 5.4 PB
* 每秒 10 萬次的讀取請求
    * 每月 250 億次讀取請求 * (每秒 400 次請求 / 每月 10 億次請求)
* 每秒 6000 個 tweets
    * 每月 150 億次 tweets * (每秒 400 次請求 / 每月 10 億次請求)
* 每秒 fanout 的 tweets 有 6 萬則
    * 每月 fanout 1500 億則 tweets * (每秒 400 次請求 / 每月 10 億次請求)
* 4,000 search requests per second

一些筆記：

* 每月 250 萬秒
* 每秒 1 次請求 = 每月 250 萬次請求
* 每秒 40 次請求 = 每月 1 億次請求
* 每秒 400 次請求 = 每月 10 億 次請求

## 步驟二：進行高階設計

> 提出所有重要元件的高階設計

![Imgur](http://i.imgur.com/48tEA2j.png)

## 步驟三：設計核心元件

> 深入每個核心元件的細節

### 使用案例：使用者張貼一則 tweet

我們可以在 [關連式資料庫](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E9%97%9C%E9%80%A3%E5%BC%8F%E8%B3%87%E6%96%99%E5%BA%AB%E7%AE%A1%E7%90%86%E7%B3%BB%E7%B5%B1rdbms) 中儲存使用者自己的 tweet 在其 timeline 上 (這是使用者自己的活動列表)。而我們同時也應該討論關於 [使用 SQL 或 NoSQL 之間的權衡](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#sql-%E6%88%96-nosql)。

發送 tweets 和呈現使用者的 home timeline (顯示使用者之跟隨者的 tweets) 是複雜的。當系統想要將 tweets 發送給所有的跟隨者 (每秒發送 6 萬則 tweets)，使用傳統的 [關連式資料庫](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E9%97%9C%E9%80%A3%E5%BC%8F%E8%B3%87%E6%96%99%E5%BA%AB%E7%AE%A1%E7%90%86%E7%B3%BB%E7%B5%B1rdbms) 負擔太大。我們也許會想要使用寫入比較快的 **NoSQL 資料庫** 或 **記憶體快取**。循序的從記憶體中讀取 1 MB 的資料大約需要 250 微秒，而從 SSD 讀取需要 4 倍的時間，從硬碟中讀取則需要 80 倍的時間。<sup><a href=https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E6%AF%8F%E5%80%8B%E9%96%8B%E7%99%BC%E8%80%85%E9%83%BD%E6%87%89%E8%A9%B2%E7%9F%A5%E9%81%93%E7%9A%84%E5%BB%B6%E9%81%B2%E6%95%B8%E9%87%8F%E7%B4%9A>1</a></sup>

我們也可以將照片或影片儲存在 **物件資料庫** 中。

* **使用者**撰寫一則 tweet 到 **網頁伺服器**，這個伺服器是以 [反向代理伺服器](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E5%8F%8D%E5%90%91%E4%BB%A3%E7%90%86%E7%B6%B2%E9%A0%81%E4%BC%BA%E6%9C%8D%E5%99%A8) 的方式執行。
* **網頁伺服器** 將請求轉給 **API 伺服器** 負責寫入資料
* **API 伺服器**將使用者的 tweet 儲存在**關聯式資料庫**中。
* **API 伺服器**同時會和**發送 tweets 的服務**進行溝通，進行以下幾項行動：
    * 到**使用者圖形服務**進行查詢，找出使用**記憶體快取**儲存的使用者的跟隨者名單
    * 在**記憶體快取**中儲存 tweet 到*使用者跟隨者的 timeline*
        * O(n) 操作：1,000 跟隨者 = 1,000 次的查詢和寫入
    * 將 tweet 儲存在**搜尋索引服務**中來加快搜尋操作
    * 將影音儲存在**物件資料庫**中
    * 使用**通知服務**來發送通知給跟隨者：
        * 使用**佇列** (未畫在架構圖上) 來非同步的發送通知

**向你的面試者詢問他預期你的程式碼要寫到什麼程度**.

在我們使用的 Redis **記憶體快取**服務上可以使用 Redis 的 list 來儲存 tweet 資料：

```
           tweet n+2                   tweet n+1                   tweet n
| 8 bytes   8 bytes  1 byte | 8 bytes   8 bytes  1 byte | 8 bytes   8 bytes  1 byte |
| tweet_id  user_id  meta   | tweet_id  user_id  meta   | tweet_id  user_id  meta   |
```

新的 tweet 可以被放在**記憶體快取**中，用來被發佈至使用者的 home timeline (跟隨者的活動狀況)。

我們可以使用 twitter 的 [**REST API**](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E5%85%B7%E8%B1%A1%E7%8B%80%E6%85%8B%E8%BD%89%E7%A7%BB-rest)：

```
$ curl -X POST --data '{ "user_id": "123", "auth_token": "ABC123", \
    "status": "hello world!", "media_ids": "ABC987" }' \
    https://twitter.com/api/v1/tweet
```

回應：

```
{
    "created_at": "Wed Sep 05 00:37:15 +0000 2012",
    "status": "hello world!",
    "tweet_id": "987",
    "user_id": "123",
    ...
}
```

內部通訊上，我們可以使用 [遠端程式呼叫](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E9%81%A0%E7%AB%AF%E7%A8%8B%E5%BC%8F%E5%91%BC%E5%8F%AB-rpc) 的方式。

### 使用案例：使用者瀏覽 home timeline

* **客戶端**將一個 home timeline 的請求發送給**網頁伺服器**
* **網頁伺服器**將請求轉發給**負責讀取請求的 API 伺服器**
* **負責讀取請求的 API 伺服器**會呼叫 **timeline 服務**，進行以下操作：
    * 從**記憶體快取**中取得 timeline 資料，包含 tweet id 和使用者 id
    * 透過 **Tweet Info 服務**，使用 [multiget](http://redis.io/commands/mget) 指令來取得每個 tweet 的額外資訊 - O(n)
    * 透過 **使用者資訊服務**，使用  [multiget](http://redis.io/commands/mget) 指令來取得每個使用者的額外資訊 - O(n)

REST API：

```
$ curl https://twitter.com/api/v1/home_timeline?user_id=123
```

回應：

```
{
    "user_id": "456",
    "tweet_id": "123",
    "status": "foo"
},
{
    "user_id": "789",
    "tweet_id": "456",
    "status": "bar"
},
{
    "user_id": "789",
    "tweet_id": "579",
    "status": "baz"
},
```

### 使用案例：使用者瀏覽自己的 timeline

* **客戶端**發送一個 home timeline 的請求到 **網站伺服器**
* **網站伺服器**將請求轉送到 **API 伺服器**
* **API 伺服器**從 **SQL 資料庫**將自己的 timelime 資訊取出

REST API 的部分和 home timeline 類似，除了要抓出來的 tweets 是自己發送的，而不是跟隨者的 tweets。

### 使用案例：使用者搜尋關鍵字

* **客戶端**會發送搜尋請求到**網頁伺服器**
* **網頁伺服器**會轉發請求到**搜尋 API**
* **搜尋 API** 透過**搜尋服務**進行以下行為：
    * 將搜尋語句進行分析/斷詞，決定哪一部分需要被搜尋：
        * 刪除標記
        * 將長文句分解成詞
        * 修正錯字
        * 將大小寫歸一化
        * 將查詢語句轉換成布林操作
    * 透過**搜尋集群服務** (例如： [Lucene](https://lucene.apache.org/)) 來查詢結果：
        * 透過 [Scatter gathers](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E4%BB%8D%E5%9C%A8%E9%80%B2%E8%A1%8C%E4%B8%AD) 方法來搜尋集群中的每台伺服器來查詢是否有任何搜尋回傳結果
        * 合併結果、排序並回傳結果

REST API：

```
$ curl https://twitter.com/api/v1/search?query=hello+world
```

除了回傳比對到的 tweets 之外，這部分的回傳值應該和 home timeline 類似。

## 步驟四：擴展你的設計

> 根據你設定的限制條件，提出目前設計架構上的瓶頸，並提出解決方法

![Imgur](http://i.imgur.com/jrUBAF7.png)

**重要提醒：不要一開始就從最初的設計跳到最後階段**

描述你如何進行 1) **負載壓力測試**、2) **描述瓶頸**、3) 解決瓶頸，並提出替代方案、4) 重複以上步驟。可以參考 [在 AWS 上設計可以乘載百萬使用者的系統](../scaling_aws/README.md) 章節作為參考，學習如何一步一步來擴展你的初始架構設計。

針對初始設計所會遇到的瓶頸進行討論，並且知道如何解決是很重要的。舉例來說，你可以透過增加一台**負載平衡器**來加入多個**網頁伺服器**來解決什麼問題？**CDN**呢？**主-從架構**？每個選擇的替代方案和權衡條件是什麼？

我們會介紹一些元件來使系統設計更完整，並且解決擴展性的問題。這裡沒有顯示內部負載平衡器，以免讓整個架構太混亂。

*為了避免重複贅述*，請參考 [系統設計主題索引](https://github.com/donnemartin/system-design-primer#index-of-system-design-topics) 中提到的各種架構的取捨與選擇：

* [DNS](https://github.com/donnemartin/system-design-primer#domain-name-system)
* [CDN](https://github.com/donnemartin/system-design-primer#content-delivery-network)
* [負載平衡器](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E8%B2%A0%E8%BC%89%E5%B9%B3%E8%A1%A1%E5%99%A8)
* [水平擴展](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E6%B0%B4%E5%B9%B3%E6%93%B4%E5%B1%95)
* [網頁伺服器 (反向代理)](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E5%8F%8D%E5%90%91%E4%BB%A3%E7%90%86%E7%B6%B2%E9%A0%81%E4%BC%BA%E6%9C%8D%E5%99%A8)
* [API 伺服器 (應用層)](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E6%87%89%E7%94%A8%E5%B1%A4)
* [快取](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E5%BF%AB%E5%8F%96)
* [關聯式資料庫 (RDBMS)](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E9%97%9C%E9%80%A3%E5%BC%8F%E8%B3%87%E6%96%99%E5%BA%AB%E7%AE%A1%E7%90%86%E7%B3%BB%E7%B5%B1rdbms)
* [SQL 主從轉移](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E5%AE%B9%E9%8C%AF%E8%BD%89%E7%A7%BB)
* [主從複寫](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E4%B8%BB%E5%BE%9E%E8%A4%87%E5%AF%AB)
* [非同步](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E9%9D%9E%E5%90%8C%E6%AD%A5%E6%A9%9F%E5%88%B6)
* [一致性模式](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E4%B8%80%E8%87%B4%E6%80%A7%E6%A8%A1%E5%BC%8F)
* [可用性模式](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E5%8F%AF%E7%94%A8%E6%80%A7%E6%A8%A1%E5%BC%8F)

**Fanout 服務**可能是潛在的系統瓶頸。當你有數百萬個跟隨者時，可能會需要花費好幾分鐘來完成整個 fanout 的流程。這會和 @reply 的 tweet 發生競爭 (race condition)，我們可以透過重新排列 tweets 來舒緩這種情況。

我們還可以避免對具有大量跟隨者的使用者進行 fanout。取而代之的，我們可以主動尋找這些熱門的使用者，將他們的搜尋結果合併後，再一次把他們的 home timeline 回傳。

其他優化的方式還有：

* 在**記憶體快取**中，針對 home timeline 部分只保留數百則 tweets
* 在**記憶體快取**中，只保留活躍使用者的資訊
    * 如果使用者在過去三十天內沒有使用的紀錄時，我們可以從 **SQL 資料庫**中重建 home timeline 的資訊
        * 使用 **使用者 Graph 服務**來尋找使用者的跟隨者
        * 從 **SQL 資料庫**中把使用者的 tweets 取出並寫入**記憶體快取**中
* 只保留一個月的 tweets 資料在 **Tweet Info 服務**中
* 僅保留活躍的使用者在**使用者 Info 服務**中
* **搜尋集群**可能會將 tweets 儲存在記憶體中以降低延遲

我們同樣會想要來關心 **SQL 資料庫**所面臨的瓶頸。

儘管**記憶體快取**可以降低資料庫的負擔，但 **SQL 可讀副本**的架構不見得足夠處理快取未命中的情形。我們可能會想要採用其他 SQL 擴展的模式。

大量的寫入會導致 **SQL 主從複寫**資料庫的壓力，我們也需要額外的擴展技巧。

* [聯邦式資料庫](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E8%81%AF%E9%82%A6%E5%BC%8F%E8%B3%87%E6%96%99%E5%BA%AB)
* [分片](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E5%88%86%E7%89%87)
* [反正規化](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E5%8F%8D%E6%AD%A3%E8%A6%8F%E5%8C%96)
* [SQL 優化](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#sql-%E5%84%AA%E5%8C%96)

我們也可以考慮將某些資料移到 **NoSQL 資料庫**。

## Additional talking points

> Additional topics to dive into, depending on the problem scope and time remaining.

#### NoSQL

* [Key-value store](https://github.com/donnemartin/system-design-primer#key-value-store)
* [Document store](https://github.com/donnemartin/system-design-primer#document-store)
* [Wide column store](https://github.com/donnemartin/system-design-primer#wide-column-store)
* [Graph database](https://github.com/donnemartin/system-design-primer#graph-database)
* [SQL vs NoSQL](https://github.com/donnemartin/system-design-primer#sql-or-nosql)

### Caching

* Where to cache
    * [Client caching](https://github.com/donnemartin/system-design-primer#client-caching)
    * [CDN caching](https://github.com/donnemartin/system-design-primer#cdn-caching)
    * [Web server caching](https://github.com/donnemartin/system-design-primer#web-server-caching)
    * [Database caching](https://github.com/donnemartin/system-design-primer#database-caching)
    * [Application caching](https://github.com/donnemartin/system-design-primer#application-caching)
* What to cache
    * [Caching at the database query level](https://github.com/donnemartin/system-design-primer#caching-at-the-database-query-level)
    * [Caching at the object level](https://github.com/donnemartin/system-design-primer#caching-at-the-object-level)
* When to update the cache
    * [Cache-aside](https://github.com/donnemartin/system-design-primer#cache-aside)
    * [Write-through](https://github.com/donnemartin/system-design-primer#write-through)
    * [Write-behind (write-back)](https://github.com/donnemartin/system-design-primer#write-behind-write-back)
    * [Refresh ahead](https://github.com/donnemartin/system-design-primer#refresh-ahead)

### Asynchronism and microservices

* [Message queues](https://github.com/donnemartin/system-design-primer#message-queues)
* [Task queues](https://github.com/donnemartin/system-design-primer#task-queues)
* [Back pressure](https://github.com/donnemartin/system-design-primer#back-pressure)
* [Microservices](https://github.com/donnemartin/system-design-primer#microservices)

### Communications

* Discuss tradeoffs:
    * External communication with clients - [HTTP APIs following REST](https://github.com/donnemartin/system-design-primer#representational-state-transfer-rest)
    * Internal communications - [RPC](https://github.com/donnemartin/system-design-primer#remote-procedure-call-rpc)
* [Service discovery](https://github.com/donnemartin/system-design-primer#service-discovery)

### Security

Refer to the [security section](https://github.com/donnemartin/system-design-primer#security).

### Latency numbers

See [Latency numbers every programmer should know](https://github.com/donnemartin/system-design-primer#latency-numbers-every-programmer-should-know).

### Ongoing

* Continue benchmarking and monitoring your system to address bottlenecks as they come up
* Scaling is an iterative process
