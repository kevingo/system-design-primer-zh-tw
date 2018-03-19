# 設計 Mint.com

*注意：本文件某些連結直接連到 [系統設計主題的索引](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E7%B3%BB%E7%B5%B1%E8%A8%AD%E8%A8%88%E4%B8%BB%E9%A1%8C%E7%9A%84%E7%B4%A2%E5%BC%95)來避免重複的內容。你可以參考連結來取得相關重點、設計的取捨和選擇。*

## 步驟一：描述使用情境與限制

> 蒐集問題的需求、資訊和範圍。
> 透過詢問問題來瞭解使用情境和限制。
> 討論你的假設。

在這裡沒有面試者會幫你釐清上面的問題，所以我們會預先定義一些使用情境和限制。

### 使用情境

#### 我們將所要解決的問題限縮在以下範圍

* **使用者** 連接到一個財務帳戶
* **服務內容** 從帳戶中取得交易紀錄
    * 每日更新
    * 進行交易的分類
        * 允許使用者手動自己手動覆蓋
        * 沒有自動重新分類功能
    * 每月針對各個類別進行花費分析
* **服務內容** 建議的預算
    * 允許使用者設定預算上限
    * 當接近或超過預算上限時會通知使用者
* **服務** 是高可用的

#### 不包含在此範圍

* **服務** 提供額外的日誌記錄與分析功能

### 限制與假設

#### 狀態假設

* 流量不是均勻分布的
* 每日的自動更新帳戶功能僅適用於過去三十天內處於活動狀態的使用者
* 新增或刪除一個帳戶的功能相對較少使用
* 預算通知不需要是即時的
* 1000 萬個使用者
    * 每個 user 有 10 個預算的類別 = 總共有 1 億筆的預算資料
    * 幾個類別範例：
        * 租屋 = $1,000
        * 食物 = $200
        * 加油 = $100
    *販賣者被用來決定每個交易的類別
        * 50,000 個販賣者
* 3000 萬個帳戶
* 每月 50 億筆交易資料
* 每月 5 億次的讀取請求
* 寫入和讀取的比例為 10:1
    * 寫入的需求很大，因為使用者每天都會進行交易，但相對比較少的人會每天訪問網站

#### 計算使用量

**向你的面試人員詢問你是否可以用比較粗略的方式來計算使用量**

* 每筆資料的大小：
    * `user_id` - 8 bytes
    * `created_at` - 5 bytes
    * `seller` - 32 bytes
    * `amount` - 5 bytes
    * 總共約： ~50 bytes
* 每月 250 GB 新的交易資料量
    * 每筆交易 50 bytes * 每月 50 億筆交易資料
    * 3 年內有 9 TB 的新交易資料
    * 假設絕大多數都是新的交易資料，而不是更新既有的資料
* 平均每秒 2000 筆交易
* 平均每秒 200 個讀取的請求

一些筆記：

* 每月 250 萬秒
* 每秒 1 次請求 = 每月 250 萬次請求
* 每秒 40 次請求 = 每月 1 億次請求
* 每秒 400 次請求 = 每月 10 億 次請求

## 步驟二：進行高階設計

> 提出所有重要元件的高階設計

![Imgur](http://i.imgur.com/E8klrBh.png)

## 步驟三：設計核心元件

> 深入每個核心元件的細節

### 使用案例：使用者連接到一個財務帳戶

我們可以在 [關連式資料庫](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E9%97%9C%E9%80%A3%E5%BC%8F%E8%B3%87%E6%96%99%E5%BA%AB%E7%AE%A1%E7%90%86%E7%B3%BB%E7%B5%B1rdbms) 中為 1000 萬名使用者儲存相關資料。我們應該要進行 [SQL 或 NoSQL 的相關使用案例](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#sql-%E6%88%96-nosql) 討論。

* **客戶端** 發出一個請求到 **網頁伺服器** 上，這個伺服器是一個 [反向代理伺服器](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E5%8F%8D%E5%90%91%E4%BB%A3%E7%90%86%E7%B6%B2%E9%A0%81%E4%BC%BA%E6%9C%8D%E5%99%A8)。
* **網頁伺服器**會轉送請求到**帳戶 API 伺服器**。
* **帳戶 API 伺服器**更新 **SQL 資料庫**的 `accounts` 資料表中的相關資訊。

**向你的面試者詢問他預期你的程式碼要寫到什麼程度**.

`accounts` 資料表包含以下結構：

```
id int NOT NULL AUTO_INCREMENT
created_at datetime NOT NULL
last_update datetime NOT NULL
account_url varchar(255) NOT NULL
account_login varchar(32) NOT NULL
account_password_hash char(64) NOT NULL
user_id int NOT NULL
PRIMARY KEY(id)
FOREIGN KEY(user_id) REFERENCES users(id)
```

我們會在 `id`、`user_id` 和 `created_at` 三個欄位建立 [索引](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E4%BD%BF%E7%94%A8%E6%AD%A3%E7%A2%BA%E7%9A%84%E7%B4%A2%E5%BC%95) 來增加讀取速度 (log-time 的讀取時間，而不是掃描整張資料表)，同時確保資料會被保存在記憶體中。從記憶體中循序讀取 1MB 的資料大約需要 250 毫秒，但相同的資料，從 SSD 大概需要四倍以上的時間，而一般的硬碟則需要八十倍以上的時間<sup><a href=https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E6%AF%8F%E5%80%8B%E9%96%8B%E7%99%BC%E8%80%85%E9%83%BD%E6%87%89%E8%A9%B2%E7%9F%A5%E9%81%93%E7%9A%84%E5%BB%B6%E9%81%B2%E6%95%B8%E9%87%8F%E7%B4%9A>1</a></sup>。

我們會使用公開的 [**REST API**](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E5%85%B7%E8%B1%A1%E7%8B%80%E6%85%8B%E8%BD%89%E7%A7%BB-rest)：

```
$ curl -X POST --data '{ "user_id": "foo", "account_url": "bar", \
    "account_login": "baz", "account_password": "qux" }' \
    https://mint.com/api/v1/account
```

內部通訊上，我們可以使用 [遠端程式呼叫](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E9%81%A0%E7%AB%AF%E7%A8%8B%E5%BC%8F%E5%91%BC%E5%8F%AB-rpc) 的方式。

下一步，我們的服務會從帳戶中取出交易資訊。

### 使用案例：服務從帳戶中取出交易資訊

我們希望從帳戶中取出以下使用情境的資訊：

* 使用者第一次連接到帳戶
* 使用者手動更新帳戶內容
* 對於過去 30 天內持續活躍的使用者，每天自動的取得資訊

資料流：

* **客戶端**發送請求到**網頁伺服器**。
* **網頁伺服器**轉發請求到**帳戶 API 伺服器**。
* **帳戶 API 伺服器**發送一個工作到**佇列**中，例如：Amazon 的 SQS 或 [RabbitMQ](https://www.rabbitmq.com/)。
    * 擷取交易資料可能會花一些時間，我們可能希望採用 [異步處理加上佇列](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E9%9D%9E%E5%90%8C%E6%AD%A5%E6%A9%9F%E5%88%B6) 的機制來進行，即使這可能會增加額外的複雜度。
* 這個**交易資訊擷取的服務**包含以下內容：
    * 從**佇列**中取出工作，並針對帳戶擷取交易資訊，將結果視為原始日誌檔案存到**物件資料庫**中。
    * 使用**類別服務**來分類每個交易紀錄
    * 使用**預算服務**按照類別計算每月總支出
        * **預算服務**使用**通知機制**來讓使用者知道他們是否接近或超過所設定的預算
    * 使用分類好的交易資訊來更新**SQL 資料庫**中的 `transactions` 資料表
    * 使用按照類別匯總後的每月支出來更新**SQL 資料庫**中的 `monthly_spending` 資料表
    * 透過**通知服務**來通知使用者他的交易資料已經處理完畢：
        * 使用**佇列服務**來異步的發送通知

`transactions` 資料表可能包含以下的結構：

```
id int NOT NULL AUTO_INCREMENT
created_at datetime NOT NULL
seller varchar(32) NOT NULL
amount decimal NOT NULL
user_id int NOT NULL
PRIMARY KEY(id)
FOREIGN KEY(user_id) REFERENCES users(id)
```

我們可以在 `id`、`user_id` 和 `created_at` 欄位上建立 [索引](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E4%BD%BF%E7%94%A8%E6%AD%A3%E7%A2%BA%E7%9A%84%E7%B4%A2%E5%BC%95)。

`monthly_spending` 資料表可能包含以下結構：

```
id int NOT NULL AUTO_INCREMENT
month_year date NOT NULL
category varchar(32)
amount decimal NOT NULL
user_id int NOT NULL
PRIMARY KEY(id)
FOREIGN KEY(user_id) REFERENCES users(id)
```

我們可以建立 [索引](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E4%BD%BF%E7%94%A8%E6%AD%A3%E7%A2%BA%E7%9A%84%E7%B4%A2%E5%BC%95) 在 `id` 和 `user_id` 欄位。

#### 類別服務

對於**類別服務**來說，我們可以提供一個「賣家與類別」對應的對應表，當中列出熱門的賣家。如果我們預估有 50000 個賣家，並且每個項目會使用少於 255 bytes，那整個對應表也僅僅需要 12 MB 的記憶體空間。

**向你的面試者詢問他預期你的程式碼要寫到什麼程度**.

```
class DefaultCategories(Enum):

    HOUSING = 0
    FOOD = 1
    GAS = 2
    SHOPPING = 3
    ...

seller_category_map = {}
seller_category_map['Exxon'] = DefaultCategories.GAS
seller_category_map['Target'] = DefaultCategories.SHOPPING
...
```

對於那些最初沒有被挑中的賣家來說，我們可以藉由手動的方式讓使用者覆蓋掉我們的設定。使用 heap 可以在 O(1) 的時間內挑出前幾名的賣家。

```
class Categorizer(object):

    def __init__(self, seller_category_map, self.seller_category_crowd_overrides_map):
        self.seller_category_map = seller_category_map
        self.seller_category_crowd_overrides_map = \
            seller_category_crowd_overrides_map

    def categorize(self, transaction):
        if transaction.seller in self.seller_category_map:
            return self.seller_category_map[transaction.seller]
        elif transaction.seller in self.seller_category_crowd_overrides_map:
            self.seller_category_map[transaction.seller] = \
                self.seller_category_crowd_overrides_map[transaction.seller].peek_min()
            return self.seller_category_map[transaction.seller]
        return None
```

交易的實作部分：

```
class Transaction(object):

    def __init__(self, created_at, seller, amount):
        self.timestamp = timestamp
        self.seller = seller
        self.amount = amount
```

### 使用案例：服務建議一個預算

一開始的時候，我們可以用一個通用的預算範本，它是根據收入的級別來分配各個類別的預算金額。根據這種方式，我們可以不需要儲存 1 億筆預算的資料，而只需要儲存使用者自定義的部分。如果使用者覆蓋的我們預先定義的預算時，我們可以儲存在 `buget_overrides` 資料表：

```
class Budget(object):

    def __init__(self, income):
        self.income = income
        self.categories_to_budget_map = self.create_budget_template()

    def create_budget_template(self):
        return {
            'DefaultCategories.HOUSING': income * .4,
            'DefaultCategories.FOOD': income * .2
            'DefaultCategories.GAS': income * .1,
            'DefaultCategories.SHOPPING': income * .2
            ...
        }

    def override_category_budget(self, category, amount):
        self.categories_to_budget_map[category] = amount
```

對於**預算服務**來說，我們可以透過查詢 `transactions` 資料表來產生 `monthly_spending` 的彙整表。`monthly_spending` 的資料表中的資料會遠少於 50 億筆的交易資料，因為使用者通常一個月會有許多交易紀錄。 

而作為替代方案，我們可以在原始的交易資料上執行 **MapReduce**：

* 對每個交易紀錄進行分類
* 根據類別來彙總每月的花費

在原始的交易資料上進行分析可以大幅減少資料庫的負擔。

如果使用者更新了類別，我們可以呼叫**預算服務**來重新分析。

**向你的面試者詢問他預期你的程式碼要寫到什麼程度**.

日誌檔案的格式，使用 tab 進行分隔：

```
user_id   timestamp   seller  amount
```

**MapReduce** 實作部分：

```
class SpendingByCategory(MRJob):

    def __init__(self, categorizer):
        self.categorizer = categorizer
        self.current_year_month = calc_current_year_month()
        ...

    def calc_current_year_month(self):
        """Return the current year and month."""
        ...

    def extract_year_month(self, timestamp):
        """Return the year and month portions of the timestamp."""
        ...

    def handle_budget_notifications(self, key, total):
        """Call notification API if nearing or exceeded budget."""
        ...

    def mapper(self, _, line):
        """Parse each log line, extract and transform relevant lines.

        Argument line will be of the form:

        user_id   timestamp   seller  amount

        Using the categorizer to convert seller to category,
        emit key value pairs of the form:

        (user_id, 2016-01, shopping), 25
        (user_id, 2016-01, shopping), 100
        (user_id, 2016-01, gas), 50
        """
        user_id, timestamp, seller, amount = line.split('\t')
        category = self.categorizer.categorize(seller)
        period = self.extract_year_month(timestamp)
        if period == self.current_year_month:
            yield (user_id, period, category), amount

    def reducer(self, key, value):
        """Sum values for each key.

        (user_id, 2016-01, shopping), 125
        (user_id, 2016-01, gas), 50
        """
        total = sum(values)
        yield key, sum(values)
```

## 步驟四：擴展你的設計

> 根據你設定的限制條件，提出目前設計架構上的瓶頸，並提出解決方法

![Imgur](http://i.imgur.com/V5q57vU.png)

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

我們會增加一個額外的使用案例：**使用者**存取摘要清單和交易。

使用者的 session，根據分類來進行彙總，同時最近存取的交易紀錄可以放在**記憶體快取**，像是 Redis 或 Memcached。

* **客戶端**發送請求到**網頁伺服器**。
* **網頁伺服器**轉發請求到**帳戶 API 伺服器**。
    * 靜態檔案可以透過**物件儲存資料庫**來提供，像是 S3，同時還可以利用 **CDN** 來進行緩存。
* **負責讀取的 API 伺服器**會做以下事情：
    * 確認**記憶體快取**中有沒有資料：
        * 如果網址已經存在於**記憶體快取**中，就直接回傳相關資訊
        * 如果不存在於記憶體快取中
            * 如果資料在**資料庫**中，就從資料庫中回傳資訊
                * 將資訊存到**記憶體快取**中

請參考 [什麼時候要更新快取](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E4%BB%80%E9%BA%BC%E6%99%82%E5%80%99%E8%A6%81%E6%9B%B4%E6%96%B0%E5%BF%AB%E5%8F%96) 這個章節來決定你的方案和評估的準則。上述的方法細節可以參考 [快取模式](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E5%BF%AB%E5%8F%96%E6%A8%A1%E5%BC%8F) 章節。

我們也可以使用像是 Amazon Redshift 或 Google BigQuery 等資料倉儲的解決方案來建立一個**分析資料庫**，用來儲存 `monthly_spending` 這樣彙整式的資料表，而不是將它們放在**SQL 資料庫**中。

我們也許只想儲存每個月的 `transactoins` 資料在資料庫中，其餘的則是放在資料倉儲或**物件儲存資料庫**中。**物件儲存資料庫**，像是 Amazon S3，可以很容易地處理每個月 250 GB 的新資料。

為了解決*平均*每秒鐘 2000 次讀取的需求 (尖峰時可能更高)，我們需要將熱門的資料放在**記憶體快取**中，而不是資料庫。**記憶體快取**也很適合用來處理不均勻地流量或尖峰流量。**SQL 讀取副本**的機制也可以用在當快取沒有命中的時候，只要讀取副本的機制不會因為進行同步的時候發生問題即可。

*平均* 每秒 200 次寫入 (尖峰時可能更高) 對於一台**SQL 主從資料庫**來說可能會有點困難，我們需要透過底下這些 SQL 擴展的模式來幫助我們：

* [聯邦式資料庫](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E8%81%AF%E9%82%A6%E5%BC%8F%E8%B3%87%E6%96%99%E5%BA%AB)
* [分片](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E5%88%86%E7%89%87)
* [反正規化](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E5%8F%8D%E6%AD%A3%E8%A6%8F%E5%8C%96)
* [SQL 優化](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#sql-%E5%84%AA%E5%8C%96)

我們也可以考慮將某些資料移到 **NoSQL 資料庫**。

## 其他想要談論的重點

> 根據你問題的範圍和面試剩餘的時間，深入討論以下主題

#### NoSQL

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
