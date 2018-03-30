# 在 AWS 上設計一個可以擴展到百萬使用者的系統

*注意：本文件某些連結直接連到 [系統設計主題的索引](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E7%B3%BB%E7%B5%B1%E8%A8%AD%E8%A8%88%E4%B8%BB%E9%A1%8C%E7%9A%84%E7%B4%A2%E5%BC%95)來避免重複的內容。你可以參考連結來取得相關重點、設計的取捨和選擇。*

## 步驟一：描述使用情境與限制

> 蒐集問題的需求、資訊和範圍。
> 透過詢問問題來瞭解使用情境和限制。
> 討論你的假設。

在這裡沒有面試者會幫你釐清上面的問題，所以我們會預先定義一些使用情境和限制。

### 使用情境

解決這個問題需要透過不斷來回修正的方式：1) **負載壓力測試**、2) **描述瓶頸**、3) 解決瓶頸，並提出替代方案、4) 重複以上步驟。上面這幾個步驟可以很好的幫助你解決可擴展系統架構的基本問題。

除非你本來就對 AWS 有相關背景知識，或是你打算申請的職位需要 AWS 的知識，否則對於 AWS 的細部知識並非必要。然而，**絕大多數在這裡提到的準則，你都可以用在其他非 AWS 的環境上**。

#### 我們將所要解決的問題限縮在以下範圍

* **使用者**會產生讀取和寫入的請求
    * **服務**本身會處理請求、儲存使用者資料並且回傳結果
* **服務**本身需要從服務少量的使用者一路擴展到百萬等級的使用者
    * 我們會討論一個通用性的原則來讓架構本身可以處理大量的使用者請求
* **服務**本身具備高可用 (High-Availability)

### 限制與假設

#### 狀態假設

* 流量並非均勻的分佈
* 資料為關連式
* 從 1 個使用者擴展到數千萬名使用者
    * 我們用以下表示方式來代表使用者不斷增加：
        * Users+
        * Users++
        * Users+++
        * ...
    * 1000 萬名使用者
    * 每月 10 億次寫入
    * 每月 1000 億次讀取
    * 讀寫比例為 100:1
    * 每次寫入大小為 1 KB

#### 計算使用量

**向你的面試人員詢問你是否可以用比較粗略的方式來計算使用量**

* 每月 1 TB 的新資料
    * 每次寫入 1 KB * 每月寫入 10 億次
    * 三年共 36 TB 的新資料
    * 假設大部分的寫入都是新資料，而不是更新既有的資料
* 平均每秒 400 次寫入
* 平均每秒 40,000 次寫入

一些筆記：

* 每月 250 萬秒
* 每秒 1 次請求 = 每月 250 萬次請求
* 每秒 40 次請求 = 每月 1 億次請求
* 每秒 400 次請求 = 每月 10 億 次請求

## 步驟二：進行高階設計

> 提出所有重要元件的高階設計

![Imgur](http://i.imgur.com/B8LDKD7.png)

## 步驟三：設計核心元件

> 深入每個核心元件的細節

### 使用情境：使用者發送一個讀取或寫入請求

#### 目標

* 當只有 1 到 2 個使用者時，你只需要基本的架構
    * 只需要一台機器
    * 需要擴展時使用垂直擴展
    * 做好監控來發現瓶頸

#### 從一台機器開始

* **網頁伺服器** 在 EC2
    * 儲存使用者資料
    * [**MySQL 資料庫**](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E9%97%9C%E9%80%A3%E5%BC%8F%E8%B3%87%E6%96%99%E5%BA%AB%E7%AE%A1%E7%90%86%E7%B3%BB%E7%B5%B1rdbms)

使用 **垂直擴展**:

* 只要單純的選擇規格比較好的機器即可
* 透過以下指標和方式來關注何時需要進行擴展
    * 使用基本的監控指標來決定系統瓶頸：CPU、記憶體、IO、Network
    * 使用 CloudWatch、top、nagios、statsd、 graphite 等工具
* 垂直擴展可能是成本很高的
* 垂直擴展不具備容錯轉移/高可用的特性

*其他的選擇或相關細節：*

* 相對於**垂直擴展**的另一個選擇是 [**水平擴展**](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E6%B0%B4%E5%B9%B3%E6%93%B4%E5%B1%95)

#### 從 SQL 資料庫開始，並且將 NoSQL 納入考量

我們的假設是這個系統是使用關連式資料，所以我們可以從一台 **MySQL 資料庫**開始規化。

*其他的選擇或相關細節：*

* 參考 [關聯式資料庫 (RDBMS)](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E9%97%9C%E9%80%A3%E5%BC%8F%E8%B3%87%E6%96%99%E5%BA%AB%E7%AE%A1%E7%90%86%E7%B3%BB%E7%B5%B1rdbms) 章節
* 討論使用 [SQL 或 NoSQL](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#sql-%E6%88%96-nosql) 的理由

#### 指定一個公開的 IP 位置

* IP 可以提供一個公開的 IP 位置，不會隨著你重開機 IP 就變更
* 綁定一個 IP 位置可以幫助你的服務具有容錯功能，當發生問題時，只要指向另外一個 IP 位置即可

#### 使用 DNS

使用像是 Route 53 這樣的 **DNS** 服務來將域名指向伺服器的公開 IP 位置。

*其他的選擇或相關細節：*

* 參考 [域名系統](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E5%9F%9F%E5%90%8D%E7%B3%BB%E7%B5%B1) 章節

#### 保護網頁伺服器

* 在防火牆部分只開啟需要的 port
    * 伺服器只允許來自底下幾個 port 得請求
        * HTTP 80
        * HTTPS 443
        * 只允許某些白名單上的 IP 位置使用 22 port 來 ssh 到這台機器
    * 避免伺服器任意的對外連線

*其他的選擇或相關細節：*

* 參考 [資訊安全](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E8%B3%87%E8%A8%8A%E5%AE%89%E5%85%A8) 章節

## 步驟四：擴展你的設計

> 根據你設定的限制條件，提出目前設計架構上的瓶頸，並提出解決方法

### Users+

![Imgur](http://i.imgur.com/rrfjMXB.png)

#### 假設

我們的使用者數量開始增加，一台機器的負擔越來越大。根據我們的**監控與負載測試**顯示， **MySQL 資料庫**的記憶體和 CPU 資源負擔逐漸增加，同時使用者的資料佔滿了我們的硬碟。

我們現在已經可以使用**垂直擴展**的方式來解決這樣的問題，但是，這種方式成本很高，而且它無法針對單ㄧ**MySQL 資料庫**或**網頁伺服器**來進行擴展。

#### 目標

* 減輕單一機器的負擔，並且允許單獨縮放
    * 將靜態的內容單獨儲存在**物件儲存資料庫**中。
    * 將 **MySQL 資料庫**移到單獨的機器
* 缺點
    * 這個改變會增加系統複雜度，同時**網頁伺服器**會需要可以連接到**物件儲存資料庫**和 **MySQL 資料庫**。
    * 必須採取額外的安全措施來保護新的元件的安全
    * AWS 的成本可能也會增加，但同時你也需要考量自己管理類似系統的成本

#### 將靜態資獨立儲存

* 考慮使用受託的**物件儲存服務**，像是 S3 來儲存靜態資料
    * 具有高度可擴展和高可靠的優點
    * 具有伺服器端加密功能
* 考慮將以下資料移至 S3
    * 使用者靜態資料
    * JS
    * CSS
    * 圖片
    * 影片

#### 將 MySQL 資料庫轉移至單獨的機器

* 考慮使用像是 RDS 這樣受託的 **MySQL 資料庫**
    * 管理上簡單，並且容易擴展
    * 多可用區域
    * 加密靜態的資料

#### 保護系統資訊安全

* 加密傳輸中和被儲存的資料
* 使用虛擬私有網路
    * 將**網頁伺服器**設置於一個公開的子網路，讓他可以傳送和接收從 Internet 上來的流量
    * 其他的部分建立一個私有網路，避免對外連線
    * 每個伺服器或資料庫只開放特定的 port 給白名單上的 IP 進行連線
* 在剩下的系統架構練習題中，這樣的模式同樣應該運用到新的資源上

*其他的選擇或相關細節：*

* 閱讀 [資訊安全](https://github.com/donnemartin/system-design-primer#security) 章節

### Users++

![Imgur](http://i.imgur.com/raoFTXM.png)

#### 狀態假設

根據我們的**監控與壓力測試**顯示，在尖峰時段，**單一台網頁伺服器**已經成為系統的瓶頸了，在某些情境下，它的回應時間變慢，在系統越來越成熟的情況下，我們應該要朝向更加高可用和高擴展的目標前進

#### 目標

* 底下的目標嘗試解決**網頁伺服器**擴展的問題
    * 根據**監控與壓力測試**的結果，你也許只要採取一到兩種做法即可
* 使用 [**水平擴展**](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E6%B0%B4%E5%B9%B3%E6%93%B4%E5%B1%95) 來解決流量增加的問題，並且提高可用性
    * 增加一個 [**負載平衡器**](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E8%B2%A0%E8%BC%89%E5%B9%B3%E8%A1%A1%E5%99%A8)。比如說使用 Amazon 的 ELB 或自己架設 HAProxy
        * ELB 是高可用的
        * 如果你打算自己架設**負載平衡器**，記得設定為 [雙主動切換模式](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E9%9B%99%E4%B8%BB%E5%8B%95%E5%88%87%E6%8F%9B%E6%A8%A1%E5%BC%8Faa-mode) 或在不同的區域設定 [主動到備用切換模式](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E4%B8%BB%E5%8B%95%E5%88%B0%E5%82%99%E7%94%A8%E5%88%87%E6%8F%9B%E6%A8%A1%E5%BC%8Fap-mode) 以達到高可用
        * 將處理 SSL 相關工作設定在**負載平衡器**上進行，如此一來可以降低後端伺服器的工作，同時在管理憑證上也更為簡便
    * 在多個可用區域 (availability zones) 中配置數台**網頁伺服器**
    * 在多個可用區域中使用多台 **MySQL** 伺服器來達到 [主從複寫](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E4%B8%BB%E5%BE%9E%E8%A4%87%E5%AF%AB)
* 將**網頁伺服器**從[**應用伺服器**](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E6%87%89%E7%94%A8%E5%B1%A4)中分離出來
    * 你可以分別擴展與設定這兩種類型的伺服器
    * **網頁伺服器**可以使用[**反向代理伺服器**](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E5%8F%8D%E5%90%91%E4%BB%A3%E7%90%86%E7%B6%B2%E9%A0%81%E4%BC%BA%E6%9C%8D%E5%99%A8)
    * 比如說，你可以使用**應用伺服器**來處理**讀取 API** 的需求，其餘的伺服器則處理**寫入 API** 的部分
* 將靜態 (甚至某些動態部分) 的內容移到 [CDN](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E5%85%A7%E5%AE%B9%E5%82%B3%E9%81%9E%E7%B6%B2%E8%B7%AFcdn) 上，像是 CloudFront，這樣可以降低讀取速度與伺服器的負擔

*其他的選擇或相關細節：*

* 詳細的內容請參考上述內文中的各個連結

### Users+++

![Imgur](http://i.imgur.com/OZCxJr0.png)

**注意：** **內部負載平衡器**並沒有畫出來，避免造成整個架構過於複雜

#### 狀態假設

根據我們的**監控與壓力測試**顯示，我們的服務在讀取與寫入的比例是 100:1，而我們的資料庫因為大量的讀取請求造成效能低落

#### 目標

* **MySQL Database** 以下的目標嘗試來解決**MySQL 資料庫**擴展的問題
    * 根據你的**監控與壓力測試**結果，你可能只需要採用底下一到兩種做法即可
* 將底下的資料移到[**記憶體快取**](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E5%BF%AB%E5%8F%96)中，像是 Elasticcache，這樣一來可以降低資料庫的負擔與延遲：
    * 在 **MySQL 資料庫**中經常被存取的資料
        * 首先，你可以先設定 **MySQL 資料庫**去快取你的資料。在實作**記憶體快取**之前，確認這樣的設計是否滿足你的需求
    * **網頁伺服器**的 session 資料
        * 這樣一來，**網頁伺服器**會變成無狀態模式，以利於**自動擴展**
    * 從記憶體中循序讀取 1 MB 的資料大約需要花費 250 微秒，然而，從 SSD 中讀取則需要花 4 倍以上的時間，而從硬碟中讀取則要花費 80 倍以上的時間<sup><a href=https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E6%AF%8F%E5%80%8B%E9%96%8B%E7%99%BC%E8%80%85%E9%83%BD%E6%87%89%E8%A9%B2%E7%9F%A5%E9%81%93%E7%9A%84%E5%BB%B6%E9%81%B2%E6%95%B8%E9%87%8F%E7%B4%9A>1</a></sup>
* 增加 [**MySQL 讀取副本**](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E4%B8%BB%E5%BE%9E%E8%A4%87%E5%AF%AB) 來降低主資料庫的負擔
* 增加更多**網頁伺服器**和**應用程式伺服器**來改善回應速度

*其他的選擇或相關細節：*

* 詳細的內容請參考上述內文中的各個連結

#### 增加 MySQL 可讀副本

* 除了增加並擴展**記憶體快取**之外，**MySQL 可讀副本**也可以降低主資料庫的負擔。
* 在**網頁伺服器**調整實作邏輯，將讀寫分離
* 在 **MySQL 可讀副本**前加上**負載平衡器**(在這裡並未畫在架構圖上)
* 大部分的服務要不就是著重讀取，要不就是著重寫入

*其他的選擇或相關細節：*

* 閱讀 [關連式資料庫管理系統 (RDBMS)](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E9%97%9C%E9%80%A3%E5%BC%8F%E8%B3%87%E6%96%99%E5%BA%AB%E7%AE%A1%E7%90%86%E7%B3%BB%E7%B5%B1rdbms) 章節

### Users++++

![Imgur](http://i.imgur.com/3X8nmdL.png)

#### 狀態假設

根據**監控與壓力測試**顯示，我們尖峰的流量集中在上班時間的美國地區，在下班後流量就急速下降。我們認為可以藉由自動的調整伺服器的數量來降低整體成本，透過**自動擴展**的機制來達到 DevOps 的自動化。

#### 目標

* 增加**自動擴展**機制
    * 滿足尖峰流量需求
    * 離峰時段藉由關閉機器來降低成本
* DevOps 自動化
    * 使用 Chef、Puppet、Ansible 等等工具
* 持續的監控來觀測系統瓶頸
    * **從機器的觀點** - 檢視每個 EC2 機器
    * **從匯總的觀點** - 檢視每個負載平衡器的狀態
    * **Log 分析** - 使用 CloudWatch、CloudTrail、Loggly、Splunk、Sumo
    * **外部網站性能監控** - Pingdom 或 New Relic
    * **處理事件和通知** - PagerDuty
    * **錯誤的報告** - Sentry

#### 增加自動擴展機制

* 考慮使用託管的服務，像是 AWS 的 **Autoscaling**
    * 為**網頁伺服器**和**應用程式伺服器**分別設定一個自動擴展的群組，將每個群組放在多個可用區中
    * 設定最小和最大的機器數量
    * 透過 CloudWatch 來觸發機器的擴展或縮減
        * 可以透過每日預估系統負載的時段來自動擴展或縮減
        * 透過觀測以下指標來進行自動擴展或縮減
            * CPU 負載
            * 延遲狀況
            * 網路流量
            * 其他客製化的指標
    * 缺點
        * 自動擴展會增加複雜度
        * 自動擴展或縮減會需要花上一點時間來滿足當前的系統需求

### Users+++++

![Imgur](http://i.imgur.com/jj3A5N8.png)

**注意：** **自動擴展** 群組並沒有畫在架構圖上以避免過於混亂

#### 狀態假設

隨著服務的架構朝著我們設定的目標和限制調整，我們持續進行**監控與壓力測試**來發現並解決新的問題。

#### 目標

根據我們對問題的假設，持續的討論關於可擴展性的相關議題：

* 如果 **MySQL 資料庫**當中儲存的資料開始變得太大，我們可能會考慮只將某個時間內的資料儲存在資料庫中，其餘的資料則放在 Redshift 這樣的資料倉儲中
    * 像 Redshift 這樣的資料倉儲可以輕鬆的儲存每個月 1 TB 新資料的規模
* 針對每秒平均 40,000 次的讀取需求，我們可以透過**記憶體快取**服務來解決，這對於處理不均勻流量與尖峰流量也會很有幫助
    * **SQL 可讀副本**可能會在讀取不到快取時遇到問題，我們需要透過其他 SQL 擴展模式來解決這個問題
* 每秒平均 400 次寫入 (尖峰時可能更高) 可能會對單一 **SQL 主從寫入**模式的資料庫產生壓力，也需要額外的擴展技術來解決

SQL 資料庫擴展模式包含以下方法：

* [聯邦式資料庫](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E8%81%AF%E9%82%A6%E5%BC%8F%E8%B3%87%E6%96%99%E5%BA%AB)
* [分片](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E5%88%86%E7%89%87)
* [反正規化](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E5%8F%8D%E6%AD%A3%E8%A6%8F%E5%8C%96)
* [SQL 優化](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#sql-%E5%84%AA%E5%8C%96)

要進一步處理高讀取和寫入的請求，我們應該要考慮將適當的資料移到像是 DynamoDB 這樣的 [**NoSQL 資料庫**](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#nosql)。

我們可以進一步將 [**應用程式伺服器**](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E6%87%89%E7%94%A8%E5%B1%A4) 分離出來，以便於單獨擴展。批次處理或是不需要即時計算的部分可以透過 [**非同步機制**](https://github.com/kevingo/system-design-primer-zh-tw/blob/master/README-zh-TW.md#%E9%9D%9E%E5%90%8C%E6%AD%A5%E6%A9%9F%E5%88%B6) 加上**佇列**和**工作佇列**：

* 以一個相簿服務來說，一張照片上傳後，縮圖的顯示可以被拆解成以下步驟：
    * **客戶端** 上傳照片
    * **應用伺服器** 將建立縮圖的工作丟到**佇列服務**中。比如說：SQS
    * 在 EC2 或 Lambda 上的 **Worker** 會到**佇列服務**上將工作抓取下來後處理：
        * 建立一個縮圖
        * 更新**資料庫**
        * 將縮圖儲存到**物件儲存**服務上

*其他的選擇或相關細節：*

* 詳細的內容請參考上述內文中的各個連結

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
