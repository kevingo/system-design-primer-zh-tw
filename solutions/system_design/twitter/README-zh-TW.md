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

### Use case: User views the user timeline

* The **Client** posts a home timeline request to the **Web Server**
* The **Web Server** forwards the request to the **Read API** server
* The **Read API** retrieves the user timeline from the **SQL Database**

The REST API would be similar to the home timeline, except all tweets would come from the user as opposed to the people the user is following.

### Use case: User searches keywords

* The **Client** sends a search request to the **Web Server**
* The **Web Server** forwards the request to the **Search API** server
* The **Search API** contacts the **Search Service**, which does the following:
    * Parses/tokenizes the input query, determining what needs to be searched
        * Removes markup
        * Breaks up the text into terms
        * Fixes typos
        * Normalizes capitalization
        * Converts the query to use boolean operations
    * Queries the **Search Cluster** (ie [Lucene](https://lucene.apache.org/)) for the results:
        * [Scatter gathers](https://github.com/donnemartin/system-design-primer#under-development) each server in the cluster to determine if there are any results for the query
        * Merges, ranks, sorts, and returns the results

REST API:

```
$ curl https://twitter.com/api/v1/search?query=hello+world
```

The response would be similar to that of the home timeline, except for tweets matching the given query.

## Step 4: Scale the design

> Identify and address bottlenecks, given the constraints.

![Imgur](http://i.imgur.com/jrUBAF7.png)

**Important: Do not simply jump right into the final design from the initial design!**

State you would 1) **Benchmark/Load Test**, 2) **Profile** for bottlenecks 3) address bottlenecks while evaluating alternatives and trade-offs, and 4) repeat.  See [Design a system that scales to millions of users on AWS](../scaling_aws/README.md) as a sample on how to iteratively scale the initial design.

It's important to discuss what bottlenecks you might encounter with the initial design and how you might address each of them.  For example, what issues are addressed by adding a **Load Balancer** with multiple **Web Servers**?  **CDN**?  **Master-Slave Replicas**?  What are the alternatives and **Trade-Offs** for each?

We'll introduce some components to complete the design and to address scalability issues.  Internal load balancers are not shown to reduce clutter.

*To avoid repeating discussions*, refer to the following [system design topics](https://github.com/donnemartin/system-design-primer#index-of-system-design-topics) for main talking points, tradeoffs, and alternatives:

* [DNS](https://github.com/donnemartin/system-design-primer#domain-name-system)
* [CDN](https://github.com/donnemartin/system-design-primer#content-delivery-network)
* [Load balancer](https://github.com/donnemartin/system-design-primer#load-balancer)
* [Horizontal scaling](https://github.com/donnemartin/system-design-primer#horizontal-scaling)
* [Web server (reverse proxy)](https://github.com/donnemartin/system-design-primer#reverse-proxy-web-server)
* [API server (application layer)](https://github.com/donnemartin/system-design-primer#application-layer)
* [Cache](https://github.com/donnemartin/system-design-primer#cache)
* [Relational database management system (RDBMS)](https://github.com/donnemartin/system-design-primer#relational-database-management-system-rdbms)
* [SQL write master-slave failover](https://github.com/donnemartin/system-design-primer#fail-over)
* [Master-slave replication](https://github.com/donnemartin/system-design-primer#master-slave-replication)
* [Consistency patterns](https://github.com/donnemartin/system-design-primer#consistency-patterns)
* [Availability patterns](https://github.com/donnemartin/system-design-primer#availability-patterns)

The **Fanout Service** is a potential bottleneck.  Twitter users with millions of followers could take several minutes to have their tweets go through the fanout process.  This could lead to race conditions with @replies to the tweet, which we could mitigate by re-ordering the tweets at serve time.

We could also avoid fanning out tweets from highly-followed users.  Instead, we could search to find tweets for high-followed users, merge the search results with the user's home timeline results, then re-order the tweets at serve time.

Additional optimizations include:

* Keep only several hundred tweets for each home timeline in the **Memory Cache**
* Keep only active users' home timeline info in the **Memory Cache**
    * If a user was not previously active in the past 30 days, we could rebuild the timeline from the **SQL Database**
        * Query the **User Graph Service** to determine who the user is following
        * Get the tweets from the **SQL Database** and add them to the **Memory Cache**
* Store only a month of tweets in the **Tweet Info Service**
* Store only active users in the **User Info Service**
* The **Search Cluster** would likely need to keep the tweets in memory to keep latency low

We'll also want to address the bottleneck with the **SQL Database**.

Although the **Memory Cache** should reduce the load on the database, it is unlikely the **SQL Read Replicas** alone would be enough to handle the cache misses.  We'll probably need to employ additional SQL scaling patterns.

The high volume of writes would overwhelm a single **SQL Write Master-Slave**, also pointing to a need for additional scaling techniques.

* [Federation](https://github.com/donnemartin/system-design-primer#federation)
* [Sharding](https://github.com/donnemartin/system-design-primer#sharding)
* [Denormalization](https://github.com/donnemartin/system-design-primer#denormalization)
* [SQL Tuning](https://github.com/donnemartin/system-design-primer#sql-tuning)

We should also consider moving some data to a **NoSQL Database**.

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
