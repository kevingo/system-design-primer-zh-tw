# 設計一個鍵值對的快取來儲存最近網頁伺服器搜尋的結果

*注意：本文件某些連結直接連到 [系統設計主題的索引](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E7%B3%BB%E7%B5%B1%E8%A8%AD%E8%A8%88%E4%B8%BB%E9%A1%8C%E7%9A%84%E7%B4%A2%E5%BC%95) 來避免重複的內容。你可以參考連結來取得相關重點、設計的取捨和選擇。*

## 步驟一：描述使用情境與限制

> 蒐集問題的需求、資訊和範圍。
> 透過詢問問題來瞭解使用情境和限制。
> 討論你的假設。

在這裡沒有面試者會幫你釐清上面的問題，所以我們會預先定義一些使用情境和限制。

### 使用情境

#### 我們將所要解決的問題限縮在以下範圍

* **使用者**發送一個搜尋的請求，這個請求最後可以在快取中找到對應資料
* **使用者**發送一個搜尋的請求，這個請求最後所需要的資料並沒有在快取中找到
* **服務**為高可用

### 限制與假設

#### 狀態假設

* 流量不是均勻分布的
    * 熱門的查詢結果應該會經常存在快取中
    * 需要決定資料何時會過去/更新
* 從快取中拿取的資料需要快速的查閱表
* 伺服器之間的溝通必須是低延遲
* 快取服務的記憶體是有限的
    * 需要決定哪些資料要保存/移除
    * 需要在快取中保存數百萬個查詢
* 1000 萬個使用者
* 每月 100 億次查詢

#### 計算使用量

**向你的面試人員詢問你是否可以用比較粗略的方式來計算使用量**

* 快取會依序儲存以下資訊：鍵：查詢、內容值查詢結果
    * `query` - 50 bytes
    * `title` - 20 bytes
    * `snippet` - 200 bytes
    * 總共： 270 bytes
* 如果 100 億次的查詢都是唯一的，並且全部儲存，則一個月會有 2.7 TB 的快取資料
    * 每個搜尋 270 bytes * 每月 100 億次搜尋
    * 假設記憶體有限，我們需要決定哪些內容會過期
* 每秒 4000 次請求

一些筆記：

* 每月 250 萬秒
* 每秒 1 次請求 = 每月 250 萬次請求
* 每秒 40 次請求 = 每月 1 億次請求
* 每秒 400 次請求 = 每月 10 億 次請求

## 步驟二：進行高階設計

> 提出所有重要元件的高階設計

![Imgur](http://i.imgur.com/KqZ3dSx.png)

## 步驟三：設計核心元件

> 深入每個核心元件的細節

### 使用案例：使用者發送查詢請求，最後讓快取命中結果並回傳

熱門的查詢可以透過**記憶體快取**，像是 Redis 或 Memcached 來降低讀取的延遲，並且避免**反向索引服務**和**文件服務**負擔過大。循序的從記憶體中讀取 1 MB 的資料大約需要 250 微秒，而從 SSD 讀取需要 4 倍的時間，從硬碟中讀取則需要 80 倍的時間。<sup><a href=https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E6%AF%8F%E5%80%8B%E9%96%8B%E7%99%BC%E8%80%85%E9%83%BD%E6%87%89%E8%A9%B2%E7%9F%A5%E9%81%93%E7%9A%84%E5%BB%B6%E9%81%B2%E6%95%B8%E9%87%8F%E7%B4%9A>1</a></sup>

由於快取的記憶體有限，我們可以使用「最近最少使用 (LRU)」的方法來讓舊有的資料過期。

* **客戶端** 發出一個請求到 **網頁伺服器** 上，這個伺服器是一個 [反向代理伺服器](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E5%8F%8D%E5%90%91%E4%BB%A3%E7%90%86%E7%B6%B2%E9%A0%81%E4%BC%BA%E6%9C%8D%E5%99%A8)
* **網頁伺服器**會轉送請求到**查詢 API 伺服器**
* **查詢 API**伺服器包含以下行為：
    * 解析查詢
        * 刪除標記
        * 將長文句分解成詞
        * 修正錯字
        * 將大小寫歸一化
        * 將查詢語句轉換成布林操作
    * 檢查**記憶體快取**來尋找和查詢詞相符合的結果
        * 如果在**記憶體快取**中找到對應的結果，則進行以下步驟：
            * 將快取的資料位置更新到 LRU 列表的前方
            * 從快取回傳結果
        * 否則，**查詢 API**伺服器則進行以下步驟：
            * 使用**反向索引服務**來尋找對應的結果
                * **反向索引服務**會針對符合的結果進行排序
            * 使用**文件服務**來回傳結果標題和描述
            * 將對應的結果更新到**記憶體快取**，並將資料位置更新到 LRU 列表前方

#### 快取實作

快取系統可以透過雙向鏈結串列來實作：新的項目會被加到開頭的位置，而過期的項目則會在尾巴的地方被移除。我們可以使用一個 hash table 來快速的查找每個鏈結串列中的節點。

**向你的面試者詢問他預期你的程式碼要寫到什麼程度**.

**查詢 API 伺服器** 實作：

```
class QueryApi(object):

    def __init__(self, memory_cache, reverse_index_service):
        self.memory_cache = memory_cache
        self.reverse_index_service = reverse_index_service

    def parse_query(self, query):
        """Remove markup, break text into terms, deal with typos,
        normalize capitalization, convert to use boolean operations.
        """
        ...

    def process_query(self, query):
        query = self.parse_query(query)
        results = self.memory_cache.get(query)
        if results is None:
            results = self.reverse_index_service.process_search(query)
            self.memory_cache.set(query, results)
        return results
```

**節點** 實作：

```
class Node(object):

    def __init__(self, query, results):
        self.query = query
        self.results = results
```

**鏈結串列** 實作：

```
class LinkedList(object):

    def __init__(self):
        self.head = None
        self.tail = None

    def move_to_front(self, node):
        ...

    def append_to_front(self, node):
        ...

    def remove_from_tail(self):
        ...
```

**快取** 實作：

```
class Cache(object):

    def __init__(self, MAX_SIZE):
        self.MAX_SIZE = MAX_SIZE
        self.size = 0
        self.lookup = {}  # key: query, value: node
        self.linked_list = LinkedList()

    def get(self, query)
        """Get the stored query result from the cache.

        Accessing a node updates its position to the front of the LRU list.
        """
        node = self.lookup[query]
        if node is None:
            return None
        self.linked_list.move_to_front(node)
        return node.results

    def set(self, results, query):
        """Set the result for the given query key in the cache.

        When updating an entry, updates its position to the front of the LRU list.
        If the entry is new and the cache is at capacity, removes the oldest entry
        before the new entry is added.
        """
        node = self.lookup[query]
        if node is not None:
            # Key exists in cache, update the value
            node.results = results
            self.linked_list.move_to_front(node)
        else:
            # Key does not exist in cache
            if self.size == self.MAX_SIZE:
                # Remove the oldest entry from the linked list and lookup
                self.lookup.pop(self.linked_list.tail.query, None)
                self.linked_list.remove_from_tail()
            else:
                self.size += 1
            # Add the new key and value
            new_node = Node(query, results)
            self.linked_list.append_to_front(new_node)
            self.lookup[query] = new_node
```

#### 什麼時候要更新快取

The cache should be updated when:

* The page contents change
* The page is removed or a new page is added
* The page rank changes

The most straightforward way to handle these cases is to simply set a max time that a cached entry can stay in the cache before it is updated, usually referred to as time to live (TTL).

Refer to [When to update the cache](https://github.com/donnemartin/system-design-primer#when-to-update-the-cache) for tradeoffs and alternatives.  The approach above describes [cache-aside](https://github.com/donnemartin/system-design-primer#cache-aside).

## Step 4: Scale the design

> Identify and address bottlenecks, given the constraints.

![Imgur](http://i.imgur.com/4j99mhe.png)

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
* [Consistency patterns](https://github.com/donnemartin/system-design-primer#consistency-patterns)
* [Availability patterns](https://github.com/donnemartin/system-design-primer#availability-patterns)

### Expanding the Memory Cache to many machines

To handle the heavy request load and the large amount of memory needed, we'll scale horizontally.  We have three main options on how to store the data on our **Memory Cache** cluster:

* **Each machine in the cache cluster has its own cache** - Simple, although it will likely result in a low cache hit rate.
* **Each machine in the cache cluster has a copy of the cache** - Simple, although it is an inefficient use of memory.
* **The cache is [sharded](https://github.com/donnemartin/system-design-primer#sharding) across all machines in the cache cluster** - More complex, although it is likely the best option.  We could use hashing to determine which machine could have the cached results of a query using `machine = hash(query)`.  We'll likely want to use [consistent hashing](https://github.com/donnemartin/system-design-primer#under-development).

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
