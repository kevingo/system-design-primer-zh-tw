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

快取應該要在以下情況發生時進行更新：

* 頁面的內容改變時
* 頁面被移除或新的頁面加入時
* 頁面的排名改變時

針對上述的情況，最直覺的方法就是為每個快取中的資料設定一個最長存活時間，通常我們稱為 TTL (time to live)。

請參考 [什麼時候要更新快取](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E4%BB%80%E9%BA%BC%E6%99%82%E5%80%99%E8%A6%81%E6%9B%B4%E6%96%B0%E5%BF%AB%E5%8F%96) 這個章節來決定你的方案和評估的準則。上述的方法細節可以參考 [快取模式](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E5%BF%AB%E5%8F%96%E6%A8%A1%E5%BC%8F) 章節。

## 步驟四：擴展你的設計

> 根據你設定的限制條件，提出目前設計架構上的瓶頸，並提出解決方法

![Imgur](http://i.imgur.com/4j99mhe.png)

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

### 擴展記憶體快去到多台機器上

為了處理大量的請求和記憶體需求，我們會進行水平擴展。底下有三種選擇來讓你了解如何儲存資料在我們的**快取**集群中：

* **每台機器在快取集群中有自己的快取節點** - 簡單的做法，即使它可能會導致較低的快取命中率
* **每台機器在快取集群中擁有快取的複製** - 簡單的做法，但這樣在使用記憶體上較沒有效率
* **快取是採用 [分片](https://github.com/donnemartin/system-design-primer#sharding) 式的架構** - 較為複雜的做法，但可能是比較好的架構。我們可以使用 hash 的機制來決定哪台機器會存放我們查詢的結果：`machine = hash(query)`. 可以考慮 [一致性的 hashing](https://github.com/donnemartin/system-design-primer/blob/master/README-zh-TW.md#%E4%BB%8D%E5%9C%A8%E9%80%B2%E8%A1%8C%E4%B8%AD)。

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
