# 設計一個網路爬蟲程式

*注意：本文件某些連結直接連到 [系統設計主題的索引](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E7%B3%BB%E7%B5%B1%E8%A8%AD%E8%A8%88%E4%B8%BB%E9%A1%8C%E7%9A%84%E7%B4%A2%E5%BC%95) 來避免重複的內容。你可以參考連結來取得相關重點、設計的取捨和選擇。*

## 步驟一：描述使用情境與限制

> 蒐集問題的需求、資訊和範圍。
> 透過詢問問題來瞭解使用情境和限制。
> 討論你的假設。

在這裡沒有面試者會幫你釐清上面的問題，所以我們會預先定義一些使用情境和限制。

### 使用情境

#### 我們將所要解決的問題限縮在以下範圍

* **此服務**主要針對多個網址進行抓取內容：
    * 產生包含搜尋關鍵字的頁面的字詞反向索引
    * 產生搜尋結果頁面的標題和描述片段(snippet)
        * 標題和描述片段是固定的，不會隨著搜尋字詞而變動
* **使用者** 輸入搜尋關鍵字，透過爬蟲服務後返回搜尋結果的標題和描述片段
    * 僅需要描繪此使用情境中的高階元件，不需要深入研究
* **服務本身**為高可用

#### 不包含在此範圍

* 搜尋分析
* 個人化的搜尋結果
* 網頁排名

### 限制與假設

#### 狀態假設

* 流量不是均勻分布的
    * 某些搜尋很熱門，但另外一些搜尋可能僅有寥寥數次
* 只支援匿名使用者
* 產生搜尋結果應該很快
* 網路爬蟲程式不應該遇到無窮迴圈的問題
    * 當 graph 包含了包含了循環時，可能會發生無窮迴圈的問題，是我們必須注意的
* 總共包含 10 億個連結需要抓取
    * 需要定期抓取頁面以確保資料是定期更新的
    * 平均更新頻率為每週一次，對於熱門網站則更加頻繁
        * 每月抓取 40 億筆連結
    * 每個網站的儲存大小為 500 KB
        * 為了簡單起見，新頁面和具有多個分頁的每個頁面大小假設為一樣
* 每個月搜尋 1000 億次

練習使用比較傳統的系統或技術 - 不要用既有的像是 [solr](http://lucene.apache.org/solr/) 或 [nutch](http://nutch.apache.org/) 框架。

#### 計算使用量

**向你的面試人員詢問你是否可以用比較粗略的方式來計算使用量**

* 每月儲存 2PB 的資料
    * 每個頁面 500 KB * 每月抓取 40 億筆連結
    * 三年共 72 PB 的資料
* 每秒 1,600 次寫入請求
* 每秒 40,000 次搜尋請求

一些筆記：

* 每月 250 萬秒
* 每秒 1 次請求 = 每月 250 萬次請求
* 每秒 40 次請求 = 每月 1 億次請求
* 每秒 400 次請求 = 每月 10 億 次請求

## 步驟二：進行高階設計

> 提出所有重要元件的高階設計

### 使用情境：抓取多個 URL 的內容

我們假設一開始有一個根據熱門程度排序好的列表，而我們會根據這份列表來抓取網頁內容。如果這個假設不成立，或是沒有這個列表時，可以從一些其他服務來取得，例如：[Yahoo](https://www.yahoo.com/) 或 [DMOZ](http://www.dmoz.org/) 等。

我們建立一張資料表 `crawled_links` 來儲存抓取過後的網頁連結和它們對應的頁面標記。

我們可以把要抓取的網頁連結，和已經處理過的網頁連結，分別建立兩個資料表 `links_to_crawl` 和 `crawled_links`，這兩個資料表可以儲存在鍵值對的 **NoSQL 資料庫**，例如 [Redis](https://redis.io/)，儲存時可以針對熱門程度來排序。我們也可以針對 [使用 SQL 或 NoSQL 的情況進行討論](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#sql-%E6%88%96-nosql)。

* **爬蟲程式**會在一個迴圈中，針對每一個要抓取的頁面進行以下行為：
    * 選取排序高的頁面進行抓取
        * 在 **NoSQL 資料庫** 中檢查 `crawled_links` 資料表是否頁面有某些頁面具有相同的頁面標記
            * 如果有類似的頁面標記，調整他們的優先順序一併抓取
                * 這可以避免我們陷入一個迴圈中
                * 繼續檢查
            * 如果沒有的話，就直接抓取頁面
                * 在 **反向索引服務** 佇列中增加一個抓取網頁的工作來產生 [反向索引](https://en.wikipedia.org/wiki/Search_engine_indexing)
                * 增加一個工作到 **文件服務** 佇列中來產生靜態的標題和描述片段(snippet)
                * 產生頁面標記
                * 從 **NoSQL 資料庫** 的 `links_to_crawl` 資料表中將此連結移除
                * 在 **NoSQL 資料庫** 中的 `crawled_links` 資料表中插入此網頁連結和網頁標記

**向你的面試者詢問他預期你的程式碼要寫到什麼程度**.

`PagesDataStore` 是一個在 **網頁爬蟲程式** 中抽象的類別，它用來和 **NoSQL 資料庫** 進行溝通：

```
class PagesDataStore(object):

    def __init__(self, db);
        self.db = db
        ...

    def add_link_to_crawl(self, url):
        """Add the given link to `links_to_crawl`."""
        ...

    def remove_link_to_crawl(self, url):
        """Remove the given link from `links_to_crawl`."""
        ...

    def reduce_priority_link_to_crawl(self, url)
        """Reduce the priority of a link in `links_to_crawl` to avoid cycles."""
        ...

    def extract_max_priority_page(self):
        """Return the highest priority link in `links_to_crawl`."""
        ...

    def insert_crawled_link(self, url, signature):
        """Add the given link to `crawled_links`."""
        ...

    def crawled_similar(self, signature):
        """Determine if we've already crawled a page matching the given signature"""
        ...
```

`Page` 是一個在 **網頁爬蟲程式** 中的抽象類別，用來包裝一個網頁頁面，包含了它的內容、子連結和頁面標記：

```
class Page(object):

    def __init__(self, url, contents, child_urls, signature):
        self.url = url
        self.contents = contents
        self.child_urls = child_urls
        self.signature = signature
```

`Crawler` 是我們 **網頁爬蟲** 的主程式，當中會用到 `Page` 和 `PagesDataStore`：

```
class Crawler(object):

    def __init__(self, data_store, reverse_index_queue, doc_index_queue):
        self.data_store = data_store
        self.reverse_index_queue = reverse_index_queue
        self.doc_index_queue = doc_index_queue

    def create_signature(self, page):
        """Create signature based on url and contents."""
        ...

    def crawl_page(self, page):
        for url in page.child_urls:
            self.data_store.add_link_to_crawl(url)
        page.signature = self.create_signature(page)
        self.data_store.remove_link_to_crawl(page.url)
        self.data_store.insert_crawled_link(page.url, page.signature)

    def crawl(self):
        while True:
            page = self.data_store.extract_max_priority_page()
            if page is None:
                break
            if self.data_store.crawled_similar(page.signature):
                self.data_store.reduce_priority_link_to_crawl(page.url)
            else:
                self.crawl_page(page)
```

### 處理重複

我們需要注意讓爬蟲程式不會陷入到無窮迴圈中。當你的 Graph 結構中包含迴圈的時候就會發生這種問題。

**向你的面試者詢問他預期你的程式碼要寫到什麼程度**.

我們希望移除重複的網址：

* `sort | unique` 對於一個小的串列集合來說，可以使用 `sort | unique` 這樣的語法
* 但你有 10 億筆連結時，可以透過 **MapReduce** 這樣的框架來處理，取出次數為 1 的網址即可：

```
class RemoveDuplicateUrls(MRJob):

    def mapper(self, _, line):
        yield line, 1

    def reducer(self, key, values):
        total = sum(values)
        if total == 1:
            yield key, total
```

要找出內容重複則是更複雜的問題。我們可以根據網頁的內容產生一些可以用計算相似程度的指標，可以參考 [Jaccard index](https://en.wikipedia.org/wiki/Jaccard_index) 和 [cosine 相似度](https://en.wikipedia.org/wiki/Cosine_similarity)。

### 決定什麼時候要更新抓取的網頁內容

我們的爬蟲程式需要經常的去更新網頁內容以保持內容是最新的。我們可以保存一個 `timestamp` 欄位來標記該頁面最後抓取的時間。設定一個期間，比如說一週，所有的頁面需要被更新。較為熱門的頁面更新的頻率可以更快一些。

儘管我們沒有要深入討論如何進行頁面內容的分析，但我們可以做一些簡單的資料探勘來計算特定頁面更新的平均時間，並透過統計資訊來幫助我們來決定何時要重新抓取頁面。

我們也可以支援 `Robots.txt` 檔案，讓網站管理者可以自行決定抓取的頻率。

### 使用情境：使用者輸入搜尋關鍵詞來查看相關頁面的標題和描述片段

* 使用者發送請求到以 [反向代理](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E5%8F%8D%E5%90%91%E4%BB%A3%E7%90%86%E7%B6%B2%E9%A0%81%E4%BC%BA%E6%9C%8D%E5%99%A8) 形式運作的 **網頁伺服器**
* **網頁伺服器** 轉送請求到 **查詢 API 伺服器**
* **查詢 API 伺服器** 會進行以下行為：
    * 解析查詢語句
        * 刪除標記
        * 將查詢語句文字轉換為詞
        * 修正錯字
        * 將大小寫正規化
        * 將查詢轉換為布林操作
    * 使用 **反向索引服務** 來尋找匹配查詢詞的網頁內容
        * **反向索引服務** 會針對匹配的結果進行排序，並回傳排名第一的結果
    * 使用 **文件服務** 來回傳標題和描述片段

我們可以使用公開的 [**REST API**](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E5%85%B7%E8%B1%A1%E7%8B%80%E6%85%8B%E8%BD%89%E7%A7%BB-rest)：

```
$ curl https://search.com/api/v1/search?query=hello+world
```

回應：

```
{
    "title": "foo's title",
    "snippet": "foo's snippet",
    "link": "https://foo.com",
},
{
    "title": "bar's title",
    "snippet": "bar's snippet",
    "link": "https://bar.com",
},
{
    "title": "baz's title",
    "snippet": "baz's snippet",
    "link": "https://baz.com",
},
```

針對內部通訊，我們可以使用 [遠端程式呼叫](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E9%81%A0%E7%AB%AF%E7%A8%8B%E5%BC%8F%E5%91%BC%E5%8F%AB-rpc)。

## Step 4: Scale the design

> Identify and address bottlenecks, given the constraints.

![Imgur](http://i.imgur.com/bWxPtQA.png)

**Important: Do not simply jump right into the final design from the initial design!**

State you would 1) **Benchmark/Load Test**, 2) **Profile** for bottlenecks 3) address bottlenecks while evaluating alternatives and trade-offs, and 4) repeat.  See [Design a system that scales to millions of users on AWS](../scaling_aws/README.md) as a sample on how to iteratively scale the initial design.

It's important to discuss what bottlenecks you might encounter with the initial design and how you might address each of them.  For example, what issues are addressed by adding a **Load Balancer** with multiple **Web Servers**?  **CDN**?  **Master-Slave Replicas**?  What are the alternatives and **Trade-Offs** for each?

We'll introduce some components to complete the design and to address scalability issues.  Internal load balancers are not shown to reduce clutter.

*To avoid repeating discussions*, refer to the following [system design topics](https://github.com/donnemartin/system-design-primer#index-of-system-design-topics) for main talking points, tradeoffs, and alternatives:

* [DNS](https://github.com/donnemartin/system-design-primer#domain-name-system)
* [Load balancer](https://github.com/donnemartin/system-design-primer#load-balancer)
* [Horizontal scaling](https://github.com/donnemartin/system-design-primer#horizontal-scaling)
* [Web server (reverse proxy)](https://github.com/donnemartin/system-design-primer#reverse-proxy-web-server)
* [API server (application layer)](https://github.com/donnemartin/system-design-primer#application-layer)
* [Cache](https://github.com/donnemartin/system-design-primer#cache)
* [NoSQL](https://github.com/donnemartin/system-design-primer#nosql)
* [Consistency patterns](https://github.com/donnemartin/system-design-primer#consistency-patterns)
* [Availability patterns](https://github.com/donnemartin/system-design-primer#availability-patterns)

Some searches are very popular, while others are only executed once.  Popular queries can be served from a **Memory Cache** such as Redis or Memcached to reduce response times and to avoid overloading the **Reverse Index Service** and **Document Service**.  The **Memory Cache** is also useful for handling the unevenly distributed traffic and traffic spikes.  Reading 1 MB sequentially from memory takes about 250 microseconds, while reading from SSD takes 4x and from disk takes 80x longer.<sup><a href=https://github.com/donnemartin/system-design-primer#latency-numbers-every-programmer-should-know>1</a></sup>

Below are a few other optimizations to the **Crawling Service**:

* To handle the data size and request load, the **Reverse Index Service** and **Document Service** will likely need to make heavy use sharding and replication.
* DNS lookup can be a bottleneck, the **Crawler Service** can keep its own DNS lookup that is refreshed periodically
* The **Crawler Service** can improve performance and reduce memory usage by keeping many open connections at a time, referred to as [connection pooling](https://en.wikipedia.org/wiki/Connection_pool)
    * Switching to [UDP](https://github.com/donnemartin/system-design-primer#user-datagram-protocol-udp) could also boost performance
* Web crawling is bandwidth intensive, ensure there is enough bandwidth to sustain high throughput

## Additional talking points

> Additional topics to dive into, depending on the problem scope and time remaining.

### SQL scaling patterns

* [Read replicas](https://github.com/donnemartin/system-design-primer#master-slave-replication)
* [Federation](https://github.com/donnemartin/system-design-primer#federation)
* [Sharding](https://github.com/donnemartin/system-design-primer#sharding)
* [Denormalization](https://github.com/donnemartin/system-design-primer#denormalization)
* [SQL Tuning](https://github.com/donnemartin/system-design-primer#sql-tuning)

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
