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

## 步驟四：擴展你的設計

> 根據你設定的限制條件，提出目前設計架構上的瓶頸，並提出解決方法

![Imgur](http://i.imgur.com/bWxPtQA.png)

**重要提醒：不要一開始就從最初的設計跳到最後階段**

描述你如何進行 1) **負載壓力測試**、2) **描述瓶頸**、3) 解決瓶頸，並提出替代方案、4) 重複以上步驟。可以參考 [在 AWS 上設計可以乘載百萬使用者的系統](../scaling_aws/README.md) 章節作為參考，學習如何一步一步來擴展你的初始架構設計。

針對初始設計所會遇到的瓶頸進行討論，並且知道如何解決是很重要的。舉例來說，你可以透過增加一台**負載平衡器**來加入多個**網頁伺服器**來解決什麼問題？**CDN**呢？**主-從架構**？每個選擇的替代方案和權衡條件是什麼？

我們會介紹一些元件來使系統設計更完整，並且解決擴展性的問題。這裡沒有顯示內部負載平衡器，以免讓整個架構太混亂。

*為了避免重複贅述*，請參考 [系統設計主題索引](https://github.com/donnemartin/system-design-primer#index-of-system-design-topics) 中提到的各種架構的取捨與選擇：

* [域名系統](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E5%9F%9F%E5%90%8D%E7%B3%BB%E7%B5%B1)
* [負載平衡器](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E8%B2%A0%E8%BC%89%E5%B9%B3%E8%A1%A1%E5%99%A8)
* [水平擴展](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E6%B0%B4%E5%B9%B3%E6%93%B4%E5%B1%95)
* [網頁伺服器 (反向代理)](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E5%8F%8D%E5%90%91%E4%BB%A3%E7%90%86%E7%B6%B2%E9%A0%81%E4%BC%BA%E6%9C%8D%E5%99%A8)
* [API 伺服器 (應用層)](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E6%87%89%E7%94%A8%E5%B1%A4)
* [快取](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E5%BF%AB%E5%8F%96)
* [NoSQL](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#nosql)
* [一致性模式](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E4%B8%80%E8%87%B4%E6%80%A7%E6%A8%A1%E5%BC%8F)
* [可用性模式](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E5%8F%AF%E7%94%A8%E6%80%A7%E6%A8%A1%E5%BC%8F)

某些搜尋很熱門，但其他的可能只有被搜尋過寥寥數次而已。針對熱門的搜尋可以透過 **記憶體快取**，像是 Redis 或 Memcached 來降低回應時間及降低 **反向索引服務** 和 **文件服務** 的負擔。**記憶體快取** 對於處理非均勻流量和突發的流量也很有幫助。從記憶體中循序讀取 1 MB 的資料大約需要花費 250 微秒，然而，從 SSD 中讀取則需要花 4 倍以上的時間，而從硬碟中讀取則要花費 80 倍以上的時間<sup><a href=https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E6%AF%8F%E5%80%8B%E9%96%8B%E7%99%BC%E8%80%85%E9%83%BD%E6%87%89%E8%A9%B2%E7%9F%A5%E9%81%93%E7%9A%84%E5%BB%B6%E9%81%B2%E6%95%B8%E9%87%8F%E7%B4%9A>1</a></sup>

針對 **網頁爬蟲程式**，底下還有一些優化的建議：

* 為了處理可預見的資料量和請求，**反向索引服務** 和 **文件服務** 可以採用分片或複寫的機制
* DNS 查詢可能會是瓶頸，**爬蟲程式** 可以使用自己的 DNS 查詢方法來讓更新更頻繁一點
* **爬蟲程式** 可以藉由在一段時間內保持連線開啟來增加效能並減少記憶體使用量，可以參考 [連線池](https://en.wikipedia.org/wiki/Connection_pool) 的說明
    * 使用 [UDP](https://github.com/donnemartin/system-design-primer#user-datagram-protocol-udp) 協定一樣可以增加效能
* 網路爬蟲需要有足夠的頻寬，確保你的頻寬足以讓你維持較高的處理能力

## 其他的重點

> 根據問題的範圍和剩餘的時間來深入探討其他主題

### SQL 擴展模式

* [可讀副本](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E4%B8%BB%E5%BE%9E%E8%A4%87%E5%AF%AB)
* [聯邦式資料庫](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E8%81%AF%E9%82%A6%E5%BC%8F%E8%B3%87%E6%96%99%E5%BA%AB)
* [分片](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E5%88%86%E7%89%87)
* [反正規化](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E5%8F%8D%E6%AD%A3%E8%A6%8F%E5%8C%96)
* [SQL 優化](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#sql-%E5%84%AA%E5%8C%96)

#### NoSQL

* [鍵-值對的資料庫](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E9%8D%B5-%E5%80%BC%E5%B0%8D%E7%9A%84%E8%B3%87%E6%96%99%E5%BA%AB)
* [文件類型資料庫](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E6%96%87%E4%BB%B6%E9%A1%9E%E5%9E%8B%E8%B3%87%E6%96%99%E5%BA%AB)
* [列儲存型資料庫](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E5%88%97%E5%84%B2%E5%AD%98%E5%9E%8B%E8%B3%87%E6%96%99%E5%BA%AB)
* [圖形資料庫](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E5%9C%96%E5%BD%A2%E8%B3%87%E6%96%99%E5%BA%AB)
* [SQL 或 NoSQL](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#sql-%E6%88%96-nosql)

### 快取

* 在哪裡進行快取
    * [使用者端快取](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E5%AE%A2%E6%88%B6%E7%AB%AF%E5%BF%AB%E5%8F%96)
    * [CDN 快取](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#cdn-%E5%BF%AB%E5%8F%96)
    * [網站伺服器快取](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E7%B6%B2%E7%AB%99%E4%BC%BA%E6%9C%8D%E5%99%A8%E5%BF%AB%E5%8F%96)
    * [資料庫快取](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E8%B3%87%E6%96%99%E5%BA%AB%E5%BF%AB%E5%8F%96)
    * [應用程式快取](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E6%87%89%E7%94%A8%E7%A8%8B%E5%BC%8F%E5%BF%AB%E5%8F%96)
* 什麼資訊需要快取
    * [資料庫查詢級別的快取](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E8%B3%87%E6%96%99%E5%BA%AB%E6%9F%A5%E8%A9%A2%E7%B4%9A%E5%88%A5%E7%9A%84%E5%BF%AB%E5%8F%96)
    * [物件級別的快取](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E7%89%A9%E4%BB%B6%E7%B4%9A%E5%88%A5%E7%9A%84%E5%BF%AB%E5%8F%96)
* 什麼時候要更新快取
    * [快取模式](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E5%BF%AB%E5%8F%96%E6%A8%A1%E5%BC%8F)
    * [寫入模式](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E5%AF%AB%E5%85%A5%E6%A8%A1%E5%BC%8F)
    * [事後寫入(回寫)](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E4%BA%8B%E5%BE%8C%E5%AF%AB%E5%85%A5%E5%9B%9E%E5%AF%AB)
    * [更新式快取](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E6%9B%B4%E6%96%B0%E5%BC%8F%E5%BF%AB%E5%8F%96)

### 非同步機制與微服務

* [訊息佇列](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E8%A8%8A%E6%81%AF%E4%BD%87%E5%88%97)
* [工作佇列](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E5%B7%A5%E4%BD%9C%E4%BD%87%E5%88%97)
* [被壓機制](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E8%83%8C%E5%A3%93%E6%A9%9F%E5%88%B6)
* [微服務](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E5%BE%AE%E6%9C%8D%E5%8B%99)

### 通訊

* 討論以下的選擇：
    * 和客戶端進行外部通訊 - [使用 REST 的 HTTP APIs](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E5%85%B7%E8%B1%A1%E7%8B%80%E6%85%8B%E8%BD%89%E7%A7%BB-rest)
    * 內部通訊 - [RPC](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E9%81%A0%E7%AB%AF%E7%A8%8B%E5%BC%8F%E5%91%BC%E5%8F%AB-rpc)
* [服務發現](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E6%9C%8D%E5%8B%99%E7%99%BC%E7%8F%BE)

### 資訊安全

參考 [資訊安全](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E8%B3%87%E8%A8%8A%E5%AE%89%E5%85%A8) 章節。

### 延遲數

參考 [每個開發者都應該知道的延遲數量級](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E6%AF%8F%E5%80%8B%E9%96%8B%E7%99%BC%E8%80%85%E9%83%BD%E6%87%89%E8%A9%B2%E7%9F%A5%E9%81%93%E7%9A%84%E5%BB%B6%E9%81%B2%E6%95%B8%E9%87%8F%E7%B4%9A) 章節。

### 後續步驟

* 持續監控系統，並進行壓力測試來解決隨時出現的系統瓶頸。
* 擴展是一個持續迭代的過程
