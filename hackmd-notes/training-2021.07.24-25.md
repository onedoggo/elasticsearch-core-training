# 喬叔的 Elasticsearch 基礎實務班 (2021.07.24~25)

[![hackmd-github-sync-badge](https://hackmd.io/YZNLmE4ZRs28v5XX5lplDg/badge)](https://hackmd.io/YZNLmE4ZRs28v5XX5lplDg)

> ## 相關連結
>
> Facebook 粉絲頁：[喬叔 - Elastic Stack 技術交流](https://www.facebook.com/Joe.ElasticStack)
>
> Google Meet：https://meet.google.com/ybg-pbbo-qjq
> 
> 課程補充資訊網站：https://es.joecwu.com/
>
> 課程示範操作原始碼：https://es.joecwu.com/training-materials
>
> 課後問卷連結：https://forms.gle/xrQDGPGM2Kgfmp1h8

:::info
:bulb: **提示**: 如果你不太熟 HackMD，[這邊](https://hackmd.io/features-tw?both) 有使用的技巧範例。
:::

---
索引

[TOC]

第一天的課程
===

[Google Meet 對話記錄](https://hackmd.io/AIHzhUIlTM2Z7NIwXN_cYA#%E4%B8%8A%E8%AA%B2-Google-Meet-%E5%B0%8D%E8%A9%B1%E8%A8%98%E9%8C%84-07-27)

# 1. Elasticsearch 快速上手

如果是 Windows, 建議直接開起 CMD, 執行 elasticsearch位置/bin/elasticsearch
 
 
Elastic Search Brew 安裝:
config 在 libexec 路徑下

解壓縮安裝:
config 資料夾在母路徑下

開啟後會去周圍網路群找有沒有同樣的 cluster name 節點存在，如果沒有就把自己設定為 Master Node `master node changed {previous []...`

## 怎麼在一台電腦模擬兩個 Node?
> 若為測試用途，直接在同一機器上使用兩個不同名稱的 Elasticsearch目錄即可

下載下來後，解壓縮為兩個不同的資料夾，並且分別去執行

> 他會自動去開在下一個 PORT (9300 -> 9301)
> 也可改 config 讓他有不同的 port

也可透過設定，讓他讀取不同的 config path
（要找文件）

確認多個 node 有沒有建立成功：
 http://<es_host>:9200/_cat/nodes?v
 ?v 是方便人眼觀看，而拿掉?v是方便寫程式的時候對其做操作
 
 在同一台電腦建立兩個 node 可以單純將下載下來的 elasticsearch 放置不同資料夾後執行
 需要注意資料夾是否乾淨，如果是用複製已經啟動過的資料夾，則可能判定失敗
 解決的話可以將新的
 
 ## 怎麼更換 node name?
 - 方法1:用 ctrl+c 停掉目前執行中的 elasticsearch 然後使用 node.name: {node.name}
 - 方法2:開啟相對應的 elasticsearch目錄\config\elasticsearch.yml 當中會有一個註解起來的 node.name，解除註解後，重新開啟 elasticsearch 即可

//實務上，kibana 可以透過 HA 機制建立分流，而 elasticsearch 有自己的 clustered 機制

### 為何 kibana `/_cat/nodes?v` 無法顯示 2 個 node?
elastic 經過初始化之後會在 data 資料夾產生 id，簡單的方式是確保兩個 node 的是乾淨的然後再執行。

## Kibana

下載下來，解壓縮，然後 `bin/kibana` 執行

http://localhost:5601/app/dev_tools#/console

> Eason 卡了很久的錯誤，結果是相對路徑的問題

![](https://i.imgur.com/otFa5hS.png)



## 加入文件

```
PUT /movies/_doc/1
{
"title": "Star Wars: Episode VII – The Force Awakens", "director": "J.J. Abrams",
"year": 2015
}
```


所有 type 都是 _doc

未來升級 8.0 就會拿掉 _doc
未來升級，

## 搜尋目錄
```
get _cat/indices?v
```

get _cat 可以告訴你有哪些功能可以用，詳細請看官方文件


### 拿出文件

get _search

預設只會回傳前十筆
要看 total.value 會寫總共有幾筆文件，如果要找出所有文件，就要設那個值

get /movies/_search

/movies 是 index 名稱

GET _search 預設是 10 筆
尋出來後可以看 hits->total->value 來確認總數
確認後可以用 GET _search？size={數字} 來增加搜尋結果筆數
```
 "hits" : {
    "total" : {
      "value" : 41,
      "relation" : "eq"
    },
```

小技巧：可以匡起來執行
![](https://i.imgur.com/9KhX5Nm.png)


### Delete
```
delete /movies/_doc/1
```

Delete 也會 update version


### Query

去文件找關鍵字： query DSL

https://www.elastic.co/guide/en/elasticsearch/reference/7.9/query-dsl.html

一定不會被 cache

### Filter

es會根據 filter 查詢的條件去做「選擇性的cache」


E 會去把字拆開成小塊來搜尋用

那如何完整比對？
告訴 E 怎麼處理這個欄位，透過 _mapping 的與法進行宣告

61 頁，劃掉是因為 7 版已經拿掉了

當一個欄位的型態被確立時，Elastic 欄位不允許破壞性的更新，要重新砍掉重建

或者用新欄位

或者用替代性

```
//透過這個與法在 director 下安排一個新的屬性
PUT /movies/_mapping
{
  "properties":{
    "director": {
      "type": "text",
      "fields": {
        "original": {
          "type": "keyword"
        }
      }
    }
  }
}
```

改了，對之後新 index 的資料才有效
然後再重新 update index 即可，接著即可用下面的方法搜尋
```
POST /movies/_search
{
  "query": {
    "term": {
      "director.original": "J.J. Abrams"
    }
  }
}
```

或者直接用 .keyword 去找

```
POST /movies/_search
{
  "query": {
    "term": {
      "title.keyword": {
        "value": "The Assassination of Jesse James by the Coward Robert Ford"
      }
    }
  }
}
```

因此，在使用Elasticsearch前，必須先確定資料的格 式與使用方式，謹慎設計mapping後再做document indexing。

要看設定情況可看 `GET /movies/_mapping`

內建的 .keyword 有 `"ignore_above" : 256` 只看前 256 字
我們後來宣告的就沒有這個限制


### 修改 mapping
- re-index api
- 如果是非破壞性的更新可以直接加入其他 mapping fields

# 2. Elasticsearch 基本介紹、集群架構與 Lucene 簡介

Segment Files

每個 Node 透過 Shard 去產生自己的 Segment File

Primary Shard: 資料分開放

replica shard: 複製品
1. 如果毀損就能提供服務
2. 提升效能

Primary Shard 可放同一 Node 或不同 Node
replica shard 會放不同 Node

參考 93 頁的圖，實心底色是 Primary Shard，透明底色是 replica shard

Elastic 會在群集中，自己去平衡 Shard 的分佈

所以 a0 的 primary shard 只會有一個，其他都會是replica shard

查詢請求效能提升：
1. 單一請求
如果要提升，如果能把資料分散到多台機器的話
（只吃 primary Shard）
但如果分太多，合併起來會有 overhead
2. 有很大量查詢請求
能被善用 Cluster，因此每台 Node 都能處理每個 Shard 的搜尋

舉例來說
兩個 Shard，四台主機
這時候單一請求可讓兩台處理（Shard 上限）
但如果大量請求，可讓四台分別處理
![](https://i.imgur.com/fPdCYBl.png)

Elastic Search 建議用 SSD

### Voting only node:
建議節點數量是 3 以上的單數，以免衝突
這要透過節點投票
如果投票個數是偶數，要重投效率就不好

在MasterNode的選舉時，有時可能我們的Master-eligiblenode是 雙數，可能會􏰀成無法順利完成選舉，因此可以安排只負責投票， 但不參與選擇的 Voting only node。(在這種模式下，Node 本身也會 是 master-eligible的角色，因為必須是 master-eligitble才能參與選舉 ，而 Voting only node會宣告本身不要被選上) 注意:設為 voting_only 的 node，必須也設定為 master，而這樣的 node.roles 設 定，表示這個 node “不會” 被選為 master 。

### Dedicated master-eligible node
如果Cluster規模較大、或是非常重視Cluster的穩定性，可安排某 個 Node 只擔任 Master-eligible 的角色，這個 Node 就不會因為大量 的 indexing, searching, ingesting, transforming 等操作讓資源被吃光 而導致 Cluster 不穩定。

### Time series Index
隨時間增加 index
這樣 Primary Shard 也會逐漸變多

但官方 Primary Shard 預設是一個，可見

### Search 
搜尋時，會依照 Adaptive Replica Selection (參考一些之前的表現狀態決定要派此任務給誰做)

# 3. Data In/Data Out 基本操作
如果處理大量資料，在放資料進去 elasticsearch 的時候建議不要指定 _id
因為 _id是 PrimaryKey, 如果有指定，elasticsearch 會找遍所有 _id 看有沒有重複
而 如果讓 _id 由 elasticsearch 自己建立，他會透過類似 timestamp 的方式查找新增


# 4. Elasticsearch Under the Hood


一份資料 (Document) 這邊假設叫做 "Movie"

一台機器或一個服務我們可以稱作 node, 多個 node 叫 cluster

當 Movie 被放入 ElasticSearch 時(其中一個 node) 會生成一個 Index 可能叫 "Ｍovie" 

（這邊假設 Movie 有一千萬筆資料）

而剛開始建立 node 時可以設定 primary shard 有幾個

接著這個 Index 注入 node 的時候會先放到記憶體去做 Inverted Index

當資料量到達一定後，會被從記憶體中抽取出來變成一個 segment

segment 會被分類到 primary shard 中
（每一個 primary shard 會有 333-334 萬筆資料）

shard 會由 ElasticSearch 分配到不同 node 中

隨著時間演進，此時可能公司會添購一個新的機器，我們把它設定成 node4

因為原先 primary shard 我們設定成 3 所以 node 4 上面不會有 primary shard

這個設定不能事後更動

所以通常會建立 replica shard 在 node4

這樣 node4 也可以幫忙分擔搜尋的 request

![](https://i.imgur.com/6jiefSO.png)

### Indexing

Routing value: 123 (_id)

murmur3(123) mod `# of shard` = shard


# Optimistic Concurrency Control
version_type=external 是因為我們是從外部 API 去修改的

```
PUT /courses/_doc/1?version=1&version_type=external
{ "first_name": "Joe"
"last_name": "Wu"
}
```



## ILM

https://ithelp.ithome.com.tw/articles/10244575

GET movies/_doc/1 資料會被放在 index 的 source 欄位
GET movies/_doc/1?source=title  當 source  很龐大的話效能就很低落
所以這個情況可以在把資料放入 index 的時候就先另外除存想關注的資料，
可以用 stored_field 來處理

_source 可以為了空間不儲存
但後續就沒辦法使用 update API


## 實作時間

使用範例資料
https://es.joecwu.com/datasets.html

Multi Get API
Get 是 realtime，資料只要進去，就一定拿得到
依照 id去把 document 拉出來
Mget 就是一次拿多筆

## Search

Timeout:
允許接受部分的失敗，

from & size
From從哪裡開始，Size 回傳幾筆？

sort 預設會按照 `_score`，如果分數一樣會按照 lucene id 去排 ( invisible )

Score 計算自資料與搜尋條件的相關度，使用BM25演算法。

hits 代表有符合的
等等會說明

Search 選replica shard 因為降低 primary shard 負擔

From + Size，他會全部抓回 Coordinator 然後再來抓出 From, Size 然後再來 mget 再來回傳
所以 From, Size 太大，就會很吃資源，預設不會太大

Coordinator 需要很多記憶體

同時 Shard 越多，回傳的就會越多，因為不確定在哪一個，要全部傳回去，再來處理




第二天的課程
===
[第二天的課程對話內容](https://hackmd.io/AIHzhUIlTM2Z7NIwXN_cYA#%E4%B8%8A%E8%AA%B2-Google-Meet-%E5%B0%8D%E8%A9%B1%E8%A8%98%E9%8C%84-07-25)

# 5. Text Analysis 的基本介紹

## NGram & EdgeNGram
```
N => 數字

# Ngram
min: 2
max: 3

Apple
-------
Ap
 pp
  pl
   le
App
 ppl
  ple

# EdgeNgram
min: 2
max: 3

Apple
-------
Ap
App


```

多個 Analyzer 的結果：
```
GET movies/_search
{
  "query": {
    "bool": {
      "should": [
        {
          "match": {
            "title": {
              "query": "Star Wars",
              "analyzer": "standard"
            }
          }
        },
        {
        "match": {
            "title": {
              "query": "Star Wars",
              "analyzer": "xxx"
            }
          }
        }
      ]
    }
  }
}
```


定義 Filter 等，要用在還沒建立的 index 才有用

[Elastic 官方 - 日文 analyzer](https://www.elastic.co/blog/how-to-implement-japanese-full-text-search-in-elasticsearch)

![](https://i.imgur.com/Wkzyiul.png)

講者使用維基百科的辭典檔，讓 E 能跟著新聞時事的新字


![](https://i.imgur.com/CMBGHfd.png)


可以用多個 mapping 一起做

## 範例: 定義不同的 fields 以套用多種的 Analyzers

:::info
Pinyin 的用法，請參考 [Pinyin Github](https://github.com/medcl/elasticsearch-analysis-pinyin)
:::

```
PUT /chinese
{
  "mappings": {
    "properties": {
      "title": {
        "type": "text",
        "fields": {
          "ik": {
            "type": "text",
            "analyzer": "ik_smart"
          },
          "pinyin": {
            "type": "text",
            "analyzer": "pinyin"
          }
        }
      },
      "body": {
        "analyzer": "smartcn"
      }
    }
  }
}
```

關於字典， index 時以當時的字典資料 為主，要更新可用 reindex，有專門的 API 可用。




同義詞應用：兩種做法，第一種是在 index 時就把同義詞都一起變成 Token，另一種是在搜尋時把 Query 變成同義詞也跟著進去找。

前者用空間換時間
後者用時間換空間


# 6. Mapping 的介紹與管理方式


```
PUT nested_fields/_doc/1
{
   "group": "fans",
   "users": [
      {
         "first": "John",
         "last": "Smith"
      },
      {
         "first": "Alice",
         "last": "White"
      }
   ]
}

GET nested_fields/_search 
{
   "query": {
      "bool": {
         "must": [
            {
               "match": {
                  "users.first": "Alice"
               }
            },
            {
               "match": {
                  "users.last": "Smith"
               }
            }
         ]
      }
   }
}
```
ElasticSearch 會把 Object 攤平，所以要做的時候會一起出現

如何解決？改成 Nested


# 7. Search - Query & Aggregation


Fuzzy: 可以在搜尋建議時使用，就推薦使用者一開始就打正確的字，比較是針對標題去做 Fuzzy 等，避免對太長的資料如內文做模糊比對以免影響效能。

## Match Phrase

如果要允許間隔幾個位置依然能查詢得到，可使用 `slop`. [參考資料](https://www.elastic.co/guide/cn/elasticsearch/guide/current/slop.html)

```
#  "body" : "蔡英文跟柯文哲去基隆廟口夜市吃宵夜"

GET chinese/_search
{
  "query": {
    "match_phrase": {
      "body": {
        "query": "蔡英文基隆",
        "slop": 5
      }
    }
  }
}
```

## Index Alias

Alias -> Index
         Movie_v1
Movie -> Movie_v2 (新版)

Hot_Movies (filter: popularity > 100) -> Movie_v2

Client -> Movie(Alias) -> Movie_v2(Index)

Last_1_month_logs(Alias, filter: time < 1M) -> Logs


Filter Cache 更新機制：Filter 更新是 by segment file，因此在 Merge 時 Cache 會失效，且小 Segment 更新 Cache 較不吃資源

### Aggreation

現在的 Aggreation 如果只回傳前 N 個，會有漏算

doc_count_error_upper_bound 可以看誤差的可能

因此條大 Aggreation 值，直到 doc_count_error_upper_bound 為 0 即可。
sum_other_doc_count 是其他比較低的，但有算到的


![](https://i.imgur.com/hrHuwjt.jpg)




# 8. Configuration & Go to production


```
- Node 1 (M)
- Node 2
- Node 3
- Node 4
- Node 5
```

(N+1)/2

N=3, 2
N=4, 3
N=5, 3
N=6, 4
N=7, 4





# 9. Elastic Stack - Logstash, Beats, Kibana 的基本介紹



# 結論

ElasticSearch 適用於全文檢索


缺點：Elastic 沒有 Lock



---
# 課程期待

> 待補…
* 希望能聽到一些 tf-idf 的計算作法
* 如何優化中文搜尋（e.g. 斷詞）
* 希望可以成功整合起來Net Core , 正確的Log記錄以及像是做出產品搜尋的功能
* 在流量和資料量不變的情況下，如何評估從 standalone 升為 cluster 時，要如何配置 cpu 和 memory
* 如何在搜尋時做出商品的區分，舉例：

```
index 

iphone ： 斷詞成 iphone
iphone 手機： 斷詞成 iphone、手機
iphone 手機殼： 斷詞成 iphone、手機、手機殼

search

搜尋 “iphone” 希望只會找到 iphone、iphone 手機，不會找到 iphone 手機殼
搜尋 “iphone 手機殼” 希望只會找到 iphone 手機殼，不會找到 iphone 或是 iphone 手機

```
上面的需求能單用ES就達到這樣的效果嗎？還是要透過ES外的手法配合？

=> match_phrase query, 或是 直接用字典檔，把 `iphone手機殼` 切成一個獨立的 term.

### 期待學習到的內容

- 多語系的資料搜尋schema設計。
>  1. 文件中僅存在一種單一語系，直接依語系的分類來切不同的 index。
>  2. 文件中混合各種語系，使用 multi-fields 來進行不同語系的 text analysis。
> [name=Joe] [time=Jul 25, 2021] [color=green]


- 在資料屬性是一對多關係且更新頻率的有差異的情況下，要如何設計schema比較適當。例如：賣家與商品，如果查詢商品時必須回傳賣家資訊，是否把賣家資訊和商品資訊設計在同一個schema，但是會造成更新賣家資料會必須更新賣家的全部商品。
> 1. ES 是搜尋引擎，最好的方式是善用他的搜尋能力，若是要當 DB 使用的話，ES 不是關聯式資料庫，不適用 RDB 的概念來設計，使用 NoSQL document based DB 的使用原則，去正規化。
> 2. 真實搜尋的使用情境來設計，是否會需要同時以混合條件進行查詢? 不會的話就可以分開放，Application 端取得需要的資訊後進行組裝 (composition)，可參考微服務架構的設計模式。
> [name=Joe] [time=Jul 25, 2021] [color=green]

- 是否有辦法在配置較差的環境(測試環境：ram較小，cpu較少)，推估配置較好的環境的搜尋效能。如果已經有一台 single node 打算生成 cluster 的架構，假設在流量和資料量都不變的情況下，要如何評估要配置多少 cpu 和 memory 才能維持原有的服務品質。
> 因為資料的特性不同、使用的情境不同，都會有不同的配置方式，所以沒有一招打天下，Elasticsearch 預設已盡量依照常用的案例來配置預設值，但還是需要依照課程介紹的原理與實際應用情境來配置。
> 一個參考的數字，依照不同硬體特性的 Memory:Storage 比例配置建議
> 1. Hot Node (SSD): 1:30
> 2. Warm Node (HDD): 1:160
> 3. Frozen Node (效能更差的 storage): 1:1000+
> [name=Joe] [time=Jul 25, 2021] [color=green]

- 了解 tokenization 分詞等用法，對搜尋機制分數計算的影響等等
> 分數的計算與對於查詢的使用方式影響更大，可使用 _explain 可了解計分的細節。
> [name=Joe] [time=Jul 25, 2021] [color=green]

- 搜尋的時候如何能做到區別產品，例如搜尋手機不會出現手機殼，搜尋手機殼不會出現手機
> 1. 讓"手機殼"與"手機"各別為 term。
> 2. 或是你希望手機殼排序在前面，而手機排在後面?
> [name=Joe] [time=Jul 25, 2021] [color=green]

- 期待聽到實戰的乾貨, 點到一些實戰上會遇到的重點與如何解決, ES 文件眾多, 學習上要有脈絡才能抓住軸心. 希望上完這堂課後可以找到方向持續學習。


### 不在這次課程範圍內

- Log 我應該要怎麼樣記錄，才會是比較好調查的
> 結構化，可參考 Elastic Common Schema 的設計方式。
> [name=Joe] [time=Jul 25, 2021] [color=green]

- 如何控制Log量，才不至於讓系統量爆掉?
> 預估、觀察、調整，配合 Index Lifecycle Management
> [name=Joe] [time=Jul 25, 2021] [color=green]

- 希望可以 Net Core，知道如何可以好好的與他介接
> Client lib 的教學不在課程範圍內
> [name=Joe] [time=Jul 25, 2021] [color=green]

- 希望可以有辦法知道怎麼去處理，英文跟中文的，去知道要怎麼寫上推薦文章，比如今天他看了一篇體育類別的文章之後，下次我要怎麼推薦
> 我們有探討中、英文的 text analysis 差異，但是推薦的機制不在 Elasticsearch 的範圍內。
> Elasticsearch 適合做的，是 autocomplete 與 auto suggestion 這樣的功能。
> [name=Joe] [time=Jul 25, 2021] [color=green]

- 壓力測試工具介紹。
> 1. esrally：裡面包含了測試資料，可用於比較不同 ES 版本之間的效能差距、比較 cluster 或是 node 在硬體規格變化時的效能差距、不同 config 配置時的效能差距
> 2. 自己的應用資料，可使用各種能壓測 RESTful 的工具，並且自己依資料特性來設計合適的測試案例。
> [name=Joe] [time=Jul 25, 2021] [color=green]

- 希望可以聽到更多關於全文檢索的內容，包括像是如何評估使用者的搜尋滿意度有提高
> 這部份可以從應用程式的 log 來記錄 hit rate ，或是 UI behavior log 進行分析。
> [name=Joe] [time=Jul 25, 2021] [color=green]

- elk cluster 設置, logstash 是否需要搭配 rabbitmq, docker 與 elk cluster 應用
> 1. Queue: 如果資料流量大且會有 peak，或是不能接受遺失的可能性，用另外的 queue 會較可靠。
> 2. Docker 不在課程範圍，Elastic 官方有 Docker 與 K8S 相關的配置教學，這部份都有支援了。
> [name=Joe] [time=Jul 25, 2021] [color=green]

- 學會 ELK 基本操作及效能調校
> 效能調校的部份未來有規劃進階課程，另外目前籌劃的新書之中也會有章節進行介紹。
> [name=Joe] [time=Jul 25, 2021] [color=green]

- 希望可以幫公司建一套較完整的log分析+apm系統
> APM 不在這次的教學範圍，未來有考慮對於 Elastic 生態圈的其他產品開不同的課程。
> [name=Joe] [time=Jul 25, 2021] [color=green]



# 同學問答區

- 問題1

> 回答1

- Genres 大、小寫 intellisense 差別

> 不過測試用 `Genres` 是查不到的
> 
> ![](https://i.imgur.com/gnhym0y.png)

- Text Analysis的分隔符號是否可以自訂(明早的問題)

- 問題：先前用 ELK，GROK 一直無法正確轉換日期，我想要 UTC+8，他給我算成 +0，當時沒找到解法

要加 date timezone的filter ，我之前是用這樣可以解決，給你參考
filter {
  date {
    match => ["@timestamp", "yyyy-MM-dd HH:mm:ss", "yyyy MM dd HH:mm:ss", "ISO8601"]
    target => "@timestamp"
    timezone => "Asia/Taipei"
  }

}

## 多語系

case 1
name_zh, name_en, name_ja

今天聽到 fields 之後，原來可以用 fields 的方式處理，所以會變成這樣

case 2 mapping
name: {
    fields: {
        zh: "xxx", Analyzer: 'ik'
        en: "xxx", Analyzer: 'standard'
        ja: "xxx", Analyzer: 'kuromoji'
    }
}

### indexing `_source` size

case 1:
{
  "name_zh": "中文",
  "name_en": "",
  "name_ja": ""
}

case 2:
{
  "name": "xxxxx"
}

### query

依照使用者 locale or 使用者 input search keyword language
=> 決定要用哪個 fields 查詢.
=> query: name.ja