# 為一個社群網站設計資料結構

*注意：本文件某些連結直接連到 [系統設計主題的索引](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E7%B3%BB%E7%B5%B1%E8%A8%AD%E8%A8%88%E4%B8%BB%E9%A1%8C%E7%9A%84%E7%B4%A2%E5%BC%95)來避免重複的內容。你可以參考連結來取得相關重點、設計的取捨和選擇。*

## 步驟一：描述使用情境與限制

> 蒐集問題的需求、資訊和範圍。
> 透過詢問問題來瞭解使用情境和限制。
> 討論你的假設。

在這裡沒有面試者會幫你釐清上面的問題，所以我們會預先定義一些使用情境和限制。

### 使用情境

#### 我們將所要解決的問題限縮在以下範圍

* **使用者**會尋找某人，並且看到此搜尋的最短路徑
* **服務**本身具備高可用 (High-Availability)

### 限制與假設

#### 狀態假設

* 流量並非均勻的分佈
    * 某些搜尋總是比較熱門，但有的搜尋可能只出現一次
* 資料並非僅存在於一台機器上
* 圖的邊上並沒有權重
* 一億名使用者
* 平均每個使用者有 50 位朋友
* 每月有 10 億次的搜尋朋友的行為

練習使用更傳統的系統 - 不要使用圖形化的解決方案，像是 [GraphQL](http://graphql.org/) 或是圖形化資料庫 [Neo4j](https://neo4j.com/)。

#### 計算使用量

**向你的面試人員詢問你是否可以用比較粗略的方式來計算使用量**

* 50 億個朋友之間的關係要記錄
    * 一億個使用者 * 每個使用者平均 50 位朋友
* 每秒 400 次搜尋

一些筆記：

* 每月 250 萬秒
* 每秒 1 次請求 = 每月 250 萬次請求
* 每秒 40 次請求 = 每月 1 億次請求
* 每秒 400 次請求 = 每月 10 億 次請求

## 步驟二：進行高階設計

> 提出所有重要元件的高階設計

![Imgur](http://i.imgur.com/wxXyq2J.png)

## 步驟三：設計核心元件

> 深入每個核心元件的細節

### 使用情境： 使用者會尋找某人，並且看到此搜尋的最短路徑

**向你的面試者詢問你的程式碼要撰寫到多詳細**.

如果沒有數百萬個使用者 (圖的點) 和數十億的朋友之間的關係 (圖的邊)，我們可以用一般的廣度優先搜尋演算法(BFS) 來解決未加權的最短路徑問題：

```
class Graph(Graph):

    def shortest_path(self, source, dest):
        if source is None or dest is None:
            return None
        if source is dest:
            return [source.key]
        prev_node_keys = self._shortest_path(source, dest)
        if prev_node_keys is None:
            return None
        else:
            path_ids = [dest.key]
            prev_node_key = prev_node_keys[dest.key]
            while prev_node_key is not None:
                path_ids.append(prev_node_key)
                prev_node_key = prev_node_keys[prev_node_key]
            return path_ids[::-1]

    def _shortest_path(self, source, dest):
        queue = deque()
        queue.append(source)
        prev_node_keys = {source.key: None}
        source.visit_state = State.visited
        while queue:
            node = queue.popleft()
            if node is dest:
                return prev_node_keys
            prev_node = node
            for adj_node in node.adj_nodes.values():
                if adj_node.visit_state == State.unvisited:
                    queue.append(adj_node)
                    prev_node_keys[adj_node.key] = prev_node.key
                    adj_node.visit_state = State.visited
        return None
```

我們不可能透過一台機器來服務所有的使用者，所以我們會需要使用到 [分片](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E5%88%86%E7%89%87) 的技術來讓使用者分散到不同的 **使用者伺服器** 上，並且透過 **查找服務** 來搜尋他們。

* **使用者** 發送請求到 **網頁伺服器**，這個伺服器是以 [反向代理伺服器](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E5%8F%8D%E5%90%91%E4%BB%A3%E7%90%86%E7%B6%B2%E9%A0%81%E4%BC%BA%E6%9C%8D%E5%99%A8) 的方式執行
* **網頁伺服器** 轉送請求到 **搜尋 API** 伺服器
* **搜尋 API** 伺服器轉送請求到 **使用者圖形服務**
* **使用者圖形服務** 會進行以下行為：
    * 使用 **查詢服務** 來尋找 **使用者伺服器**，這個伺服器儲存了目前使用者的資訊
    * 尋找正確的 **使用者伺服器** 來取得目前使用者的 `friend_ids` 清單
    * 使用目前使用者當成 `source`，以及 `friend_ids` 當成 ids 來執行廣度優先搜尋演算法(BFS)
    * 給定 id，為了找到 `adjacent_node`：
        * **使用者圖形服務** 需要 *再一次* 和 **查詢服務** 進行溝通，用來決定哪一台 **使用者伺服器** 用來儲存對應 id 的 `adjacent_node`。

**向你的面試者詢問他預期你的程式碼要寫到什麼程度**.

**注意**：為了簡化流程，例外處理並不在底下的程式碼中。向你的面試官詢問是否需要考慮例外處理。

**查詢服務** 實做：

```
class LookupService(object):

    def __init__(self):
        self.lookup = self._init_lookup()  # key: person_id, value: person_server

    def _init_lookup(self):
        ...

    def lookup_person_server(self, person_id):
        return self.lookup[person_id]
```

**使用者服務** 實作：

```
class PersonServer(object):

    def __init__(self):
        self.people = {}  # key: person_id, value: person

    def add_person(self, person):
        ...

    def people(self, ids):
        results = []
        for id in ids:
            if id in self.people:
                results.append(self.people[id])
        return results
```

**使用者** 實作：

```
class Person(object):

    def __init__(self, id, name, friend_ids):
        self.id = id
        self.name = name
        self.friend_ids = friend_ids
```

**使用者圖形服務** 實作：

```
class UserGraphService(object):

    def __init__(self, lookup_service):
        self.lookup_service = lookup_service

    def person(self, person_id):
        person_server = self.lookup_service.lookup_person_server(person_id)
        return person_server.people([person_id])

    def shortest_path(self, source_key, dest_key):
        if source_key is None or dest_key is None:
            return None
        if source_key is dest_key:
            return [source_key]
        prev_node_keys = self._shortest_path(source_key, dest_key)
        if prev_node_keys is None:
            return None
        else:
            # Iterate through the path_ids backwards, starting at dest_key
            path_ids = [dest_key]
            prev_node_key = prev_node_keys[dest_key]
            while prev_node_key is not None:
                path_ids.append(prev_node_key)
                prev_node_key = prev_node_keys[prev_node_key]
            # Reverse the list since we iterated backwards
            return path_ids[::-1]

    def _shortest_path(self, source_key, dest_key, path):
        # Use the id to get the Person
        source = self.person(source_key)
        # Update our bfs queue
        queue = deque()
        queue.append(source)
        # prev_node_keys keeps track of each hop from
        # the source_key to the dest_key
        prev_node_keys = {source_key: None}
        # We'll use visited_ids to keep track of which nodes we've
        # visited, which can be different from a typical bfs where
        # this can be stored in the node itself
        visited_ids = set()
        visited_ids.add(source.id)
        while queue:
            node = queue.popleft()
            if node.key is dest_key:
                return prev_node_keys
            prev_node = node
            for friend_id in node.friend_ids:
                if friend_id not in visited_ids:
                    friend_node = self.person(friend_id)
                    queue.append(friend_node)
                    prev_node_keys[friend_id] = prev_node.key
                    visited_ids.add(friend_id)
        return None
```

我們會使用公開的 [**REST API**]((https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E5%85%B7%E8%B1%A1%E7%8B%80%E6%85%8B%E8%BD%89%E7%A7%BB-rest))：

```
$ curl https://social.com/api/v1/friend_search?person_id=1234
```

回應：

```
{
    "person_id": "100",
    "name": "foo",
    "link": "https://social.com/foo",
},
{
    "person_id": "53",
    "name": "bar",
    "link": "https://social.com/bar",
},
{
    "person_id": "1234",
    "name": "baz",
    "link": "https://social.com/baz",
},
```

內部通訊上，我們可以使用 [遠端程式呼叫](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E9%81%A0%E7%AB%AF%E7%A8%8B%E5%BC%8F%E5%91%BC%E5%8F%AB-rpc) 的方式。

## 步驟四：擴展你的設計

> 根據你設定的限制條件，提出目前設計架構上的瓶頸，並提出解決方法

![Imgur](http://i.imgur.com/cdCv5g7.png)

**重要提醒：不要一開始就從最初的設計跳到最後階段**

描述你如何進行 1) **負載壓力測試**、2) **描述瓶頸**、3) 解決瓶頸，並提出替代方案、4) 重複以上步驟。可以參考 [在 AWS 上設計可以乘載百萬使用者的系統](../scaling_aws/README.md) 章節作為參考，學習如何一步一步來擴展你的初始架構設計。

針對初始設計所會遇到的瓶頸進行討論，並且知道如何解決是很重要的。舉例來說，你可以透過增加一台**負載平衡器**來加入多個**網頁伺服器**來解決什麼問題？**CDN**呢？**主-從架構**？每個選擇的替代方案和權衡條件是什麼？

我們會介紹一些元件來使系統設計更完整，並且解決擴展性的問題。這裡沒有顯示內部負載平衡器，以免讓整個架構太混亂。

*為了避免重複贅述*，請參考 [系統設計主題索引](https://github.com/donnemartin/system-design-primer#index-of-system-design-topics) 中提到的各種架構的取捨與選擇：

* [域名系統(DNS)](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E5%9F%9F%E5%90%8D%E7%B3%BB%E7%B5%B1)
* [負載平衡器](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E8%B2%A0%E8%BC%89%E5%B9%B3%E8%A1%A1%E5%99%A8)
* [水平擴展](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E6%B0%B4%E5%B9%B3%E6%93%B4%E5%B1%95)
* [網頁伺服器 (反向代理)](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E5%8F%8D%E5%90%91%E4%BB%A3%E7%90%86%E7%B6%B2%E9%A0%81%E4%BC%BA%E6%9C%8D%E5%99%A8)
* [API 伺服器 (應用層)](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E6%87%89%E7%94%A8%E5%B1%A4)
* [快取](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E5%BF%AB%E5%8F%96)
* [一致性模式](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E4%B8%80%E8%87%B4%E6%80%A7%E6%A8%A1%E5%BC%8F)
* [可用性模式](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E5%8F%AF%E7%94%A8%E6%80%A7%E6%A8%A1%E5%BC%8F)

To address the constraint of 400 *average* read requests per second (higher at peak), person data can be served from a **Memory Cache** such as Redis or Memcached to reduce response times and to reduce traffic to downstream services.  This could be especially useful for people who do multiple searches in succession and for people who are well-connected.  Reading 1 MB sequentially from memory takes about 250 microseconds, while reading from SSD takes 4x and from disk takes 80x longer.<sup><a href=https://github.com/donnemartin/system-design-primer#latency-numbers-every-programmer-should-know>1</a></sup>

Below are further optimizations:

* Store complete or partial BFS traversals to speed up subsequent lookups in the **Memory Cache**
* Batch compute offline then store complete or partial BFS traversals to speed up subsequent lookups in a **NoSQL Database**
* Reduce machine jumps by batching together friend lookups hosted on the same **Person Server**
    * [Shard](https://github.com/donnemartin/system-design-primer#sharding) **Person Servers** by location to further improve this, as friends generally live closer to each other
* Do two BFS searches at the same time, one starting from the source, and one from the destination, then merge the two paths
* Start the BFS search from people with large numbers of friends, as they are more likely to reduce the number of [degrees of separation](https://en.wikipedia.org/wiki/Six_degrees_of_separation) between the current user and the search target
* Set a limit based on time or number of hops before asking the user if they want to continue searching, as searching could take a considerable amount of time in some cases
* Use a **Graph Database** such as [Neo4j](https://neo4j.com/) or a graph-specific query language such as [GraphQL](http://graphql.org/) (if there were no constraint preventing the use of **Graph Databases**)

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
