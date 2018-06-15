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

為了解決平均每秒 400 次的讀取請求 (高峰時可能更高)，使用者資料可以從 **記憶體快取**，像是 Redis 或 Memcached 中取得，用以降低回應時間和降低下游服務的流量。這對連續進行多次搜尋的使用者以及連接良好的使用者特別有用。從記憶體中循序讀取 1 MB 的資料大約需要花費 250 微秒，然而，從 SSD 中讀取則需要花 4 倍以上的時間，而從硬碟中讀取則要花費 80 倍以上的時間<sup><a href=https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E6%AF%8F%E5%80%8B%E9%96%8B%E7%99%BC%E8%80%85%E9%83%BD%E6%87%89%E8%A9%B2%E7%9F%A5%E9%81%93%E7%9A%84%E5%BB%B6%E9%81%B2%E6%95%B8%E9%87%8F%E7%B4%9A>1</a></sup>

底下是一些額外的優化建議：

* 在**記憶體快取**中儲存完整或部分 BFS 遍歷的結果來加速查找過程
* 在離線時，批次計算完整或部分 BFS 結果，用來加速後續在 **NoSQL 資料庫** 中查找的過程
* 將透過 **使用者服務** 來查找朋友的過程批次處理，並且將此服務託管在同一台機器上避免伺服器之間的跳轉
    * 透過將 **使用者服務** 依照位置進行 [分片](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E5%88%86%E7%89%87) 來進一步改善料能，因為通常朋友之間的距離在圖形上會靠得很近
* 可以同時執行兩個 BFS 搜尋，一個從起始點開始，一個從終點開始，最後再進行合併
* 在進行 BFS 搜尋時，先從朋友數目多的使用者開始，因為他們更有可能減少當前使用者與搜尋目標之間的 [分離度](https://en.wikipedia.org/wiki/Six_degrees_of_separation) 的數量
* 在使用者要繼續搜尋之前，根據時間或跳轉的數量來設定一些限制，因為在某些情況下，搜尋可能需要花很多時間
* 使用像是 [Neo4j](https://neo4j.com/) 或是圖形查詢語言 [GraphQL](http://graphql.org/) (如果沒有限制使用 **圖形資料庫**)

## 其他想要談論的重點

> 根據你問題的範圍和面試剩餘的時間，深入討論以下主題

### SQL 擴展模式

* [主從複寫](https://github.com/donnemartin/system-design-primer/blob/master/README-zh-TW.md#%E4%B8%BB%E5%BE%9E%E8%A4%87%E5%AF%AB)
* [聯邦式資料庫](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E8%81%AF%E9%82%A6%E5%BC%8F%E8%B3%87%E6%96%99%E5%BA%AB)
* [分片](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E5%88%86%E7%89%87)
* [反正規化](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E5%8F%8D%E6%AD%A3%E8%A6%8F%E5%8C%96)
* [SQL 優化](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#sql-%E5%84%AA%E5%8C%96)

### NoSQL

* [鍵-值對的資料庫](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E9%8D%B5-%E5%80%BC%E5%B0%8D%E7%9A%84%E8%B3%87%E6%96%99%E5%BA%AB)
* [文件類型資料庫](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E6%96%87%E4%BB%B6%E9%A1%9E%E5%9E%8B%E8%B3%87%E6%96%99%E5%BA%AB)
* [列儲存型資料庫](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E5%88%97%E5%84%B2%E5%AD%98%E5%9E%8B%E8%B3%87%E6%96%99%E5%BA%AB)
* [圖形資料庫](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E5%9C%96%E5%BD%A2%E8%B3%87%E6%96%99%E5%BA%AB)
* [SQL 或 NoSQL](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#sql-%E6%88%96-nosql)

### 快取

* 快取在什麼位置
    * [客戶端快取](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E5%AE%A2%E6%88%B6%E7%AB%AF%E5%BF%AB%E5%8F%96)
    * [CDN 快取](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#cdn-%E5%BF%AB%E5%8F%96)
    * [網站伺服器快取](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E7%B6%B2%E7%AB%99%E4%BC%BA%E6%9C%8D%E5%99%A8%E5%BF%AB%E5%8F%96)
    * [資料庫快取](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E8%B3%87%E6%96%99%E5%BA%AB%E5%BF%AB%E5%8F%96)
    * [應用程式快取](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E6%87%89%E7%94%A8%E7%A8%8B%E5%BC%8F%E5%BF%AB%E5%8F%96)
* 什麼內容要快取
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
* [背壓機制](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E8%83%8C%E5%A3%93%E6%A9%9F%E5%88%B6)
* [微服務](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E5%BE%AE%E6%9C%8D%E5%8B%99)

### 通訊

* 評估以下兩種方式：
    * 和客戶端進行外部通訊 - [使用 REST 進行 HTTP APIs 呼叫](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E5%85%B7%E8%B1%A1%E7%8B%80%E6%85%8B%E8%BD%89%E7%A7%BB-rest)
    * 內部通訊 - [RPC](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E9%81%A0%E7%AB%AF%E7%A8%8B%E5%BC%8F%E5%91%BC%E5%8F%AB-rpc)
* [服務發現](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E6%9C%8D%E5%8B%99%E7%99%BC%E7%8F%BE)

### 安全

可以參考 [資訊安全](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E8%B3%87%E8%A8%8A%E5%AE%89%E5%85%A8) 章節。

### 延遲數目

可以參考 [每個開發者都應該知道的延遲數量級](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E6%AF%8F%E5%80%8B%E9%96%8B%E7%99%BC%E8%80%85%E9%83%BD%E6%87%89%E8%A9%B2%E7%9F%A5%E9%81%93%E7%9A%84%E5%BB%B6%E9%81%B2%E6%95%B8%E9%87%8F%E7%B4%9A) 章節。

### 持續調整與改善

* 持續的評估和監控你的系統，找出可能的瓶頸
* 擴展是一個不斷來回迭代的過程
