# 喬叔的 Elasticsearch 基礎實務班 (2021.10.23~24)

[![hackmd-github-sync-badge](https://hackmd.io/YZNLmE4ZRs28v5XX5lplDg/badge)](https://hackmd.io/7hR1u16HQ4WsHLmtKAa72g)

> ## 相關連結
>
> Facebook 粉絲頁：[喬叔 - Elastic Stack 技術交流](https://www.facebook.com/Joe.ElasticStack)
>
> Zoom：https://zoom.us/j/92157530512?pwd=TEhkeS9nUmhlVnk0bTJtVk1QSVhtQT09
> 
> 課程補充資訊網站：https://es.joecwu.com/
>
> 課程示範操作原始碼：https://es.joecwu.com/training-materials
>
> 課後問卷連結：https://forms.gle/m2PKVZ5p9rTapRhf9
> 
> Facebook 實務班：https://www.facebook.com/groups/esclass
> 

:::info
:bulb: **提示**: 如果你不太熟 HackMD，[這邊](https://hackmd.io/features-tw?both) 有使用的技巧範例。
:::

---
索引

[TOC]

第一天的課程
===

# 1. Elasticsearch 快速上手

指定 node.name
```
./bin/elasticsearch -E node.name=node1
```
multi-node elasticsearch cluster & kibana docker-compose ver.

```yaml
version: '2.2'
services:
  es01:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.15.1
    container_name: es01
    environment:
      - node.name=es01
      - cluster.name=es-docker-cluster
      - discovery.seed_hosts=es02,es03
      - cluster.initial_master_nodes=es01,es02,es03
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    command: >
      sh -c "/usr/share/elasticsearch/bin/elasticsearch-plugin install analysis-icu && \
             /usr/share/elasticsearch/bin/elasticsearch-plugin install -b https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v7.15.1/elasticsearch-analysis-ik-7.15.1.zip && \
             /usr/local/bin/docker-entrypoint.sh"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - data01:/usr/share/elasticsearch/data
      - ./IKAnalyzer.cfg.xml:/usr/share/elasticsearch/config/analysis-ik/IKAnalyzer.cfg.xml
      - ./mydict.dic:/usr/share/elasticsearch/config/analysis-ik/mydict.dic
    ports:
      - 9200:9200
    networks:
      - elastic
  es02:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.15.1
    container_name: es02
    environment:
      - node.name=es02
      - cluster.name=es-docker-cluster
      - discovery.seed_hosts=es01,es03
      - cluster.initial_master_nodes=es01,es02,es03
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    command: >
      sh -c "/usr/share/elasticsearch/bin/elasticsearch-plugin install analysis-icu && \
             /usr/share/elasticsearch/bin/elasticsearch-plugin install -b https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v7.15.1/elasticsearch-analysis-ik-7.15.1.zip && \
             /usr/local/bin/docker-entrypoint.sh"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - data02:/usr/share/elasticsearch/data
      - ./IKAnalyzer.cfg.xml:/usr/share/elasticsearch/config/analysis-ik/IKAnalyzer.cfg.xml
      - ./mydict.dic:/usr/share/elasticsearch/config/analysis-ik/mydict.dic
    networks:
      - elastic
  es03:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.15.1
    container_name: es03
    environment:
      - node.name=es03
      - cluster.name=es-docker-cluster
      - discovery.seed_hosts=es01,es02
      - cluster.initial_master_nodes=es01,es02,es03
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    command: >
      sh -c "/usr/share/elasticsearch/bin/elasticsearch-plugin install analysis-icu && \
             /usr/share/elasticsearch/bin/elasticsearch-plugin install -b https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v7.15.1/elasticsearch-analysis-ik-7.15.1.zip && \
             /usr/local/bin/docker-entrypoint.sh"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - data03:/usr/share/elasticsearch/data
      - ./IKAnalyzer.cfg.xml:/usr/share/elasticsearch/config/analysis-ik/IKAnalyzer.cfg.xml
      - ./mydict.dic:/usr/share/elasticsearch/config/analysis-ik/mydict.dic
    networks:
      - elastic
  kibana:
    image: docker.elastic.co/kibana/kibana:7.15.1
    container_name: kibana
    ports:
      - 5601:5601
    environment:
      - ELASTICSEARCH_HOSTS=http://es01:9200
    networks:
      - elastic

volumes:
  data01:
    driver: local
  data02:
    driver: local
  data03:
    driver: local

networks:
  elastic:
    driver: bridge
```

_cat/indices 小技巧
```
GET _cat/indices?v&index=m*

GET _cat/indices/m*?v

#去除掉系統建立的 indices
GET _cat/indices/*,-.*?v
```

針對 _doc 的部份，舊版與新版的差異

```
# 早期版本 5.0
PUT movies/top/1 
PUT movies/_doc/1 => 建議

# 早期版本 6.0
PUT movies/_doc/1  => 預設 
PUT movies/top/1  => 可支援 (要經過設定)

# 版本 7.0
PUT movies/_doc/1  => 只支援

# 未來 8.0
PUT movies/1
```
![](https://i.imgur.com/C5GRhvq.png)

修改 mapping 之後，如果要整批更新現有 index 的資料，以套用新的 mapping 規則，可使用 `_reindex` API
```
POST _reindex
{
  "source": {
    "index": "movies"
  },
  "dest": {
    "index": "movies_v2"
  }
}
```


# 2. Elasticsearch 基本介紹、集群架構與 Lucene 簡介

* Shard 減少，搜尋效能比較好

[shard allocation](https://www.elastic.co/guide/en/elasticsearch/reference/current/index-modules-allocation.html)

 * 由 cluster 來判斷誰當 master ，master只做決定，故不用效能最好的node。
 * 每個 node 都存有 cluster狀態


# 3. Data In/Data Out 基本操作
 * Filebeat 就是使用到  _bulk API

 * 分析效率
 * ![](https://i.imgur.com/OatynqK.png)

# 4. Elasticsearch Under the Hood

How indexing request works
![](https://i.imgur.com/rCNwKlq.jpg)

What happened if the node hosting the primary
shard process failed?
![](https://i.imgur.com/nXqFvnz.png)

What happened if replication failed?
![](https://i.imgur.com/IRsOa35.png)


What happened if the node hosting the primary
shard be isolated? (network issue)
![](https://i.imgur.com/NyizX4g.png)

* ES 自己並沒有 load banlancer


How searching request works
![](https://i.imgur.com/f4HGSwQ.jpg)



第二天的課程
===

# 5. Text Analysis 的基本介紹

Ngram
```
N = 數字
min_gram: 2
max_gram: 3

Apple
---------
Ap
 pp
  pl
   le
App
 ppl
  ple
```

EdgeNgram
```
N = 數字
min_gram: 3
max_gram: 6

Macbook
---------
Mac
Macb
Macbo
Macboo
```


Synonym Demo
```
PUT /test_index
{
  "settings": {
    "index": {
      "analysis": {
        "analyzer": {
          "synonym": {
            "tokenizer": "standard",
            "filter": [ "synonym" ]
          }
        },
        "filter": {
          "synonym": {
            "type": "synonym",
            "lenient": true,
            "synonyms": [ "foo, bar" ]
          }
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "title": {
        "type": "text",
        "analyzer": "synonym"
      }
    }
  }
}

DELETE test_index

POST test_index/_analyze
{
  "analyzer": "synonym",
  "text": "foo"
}

PUT test_index/_doc/1
{
  "title": "this is foo" 
}


GET test_index/_search
{
  "query": {
    "match": {
      "title": {
        "query": "bar",
        "analyzer": "standard"
      }
    }
  }
}
```

- 例子說明，輸入是新聞相關的分詞，可以利用wiki的辭庫，當作分詞器的外部來源，並定期更新

# 6. Mapping 的介紹與管理方式



# 7. Search - Query & Aggregation

![](https://i.imgur.com/kLPP5RL.jpg)


Alias example
```

PUT test_alias_v1
{
  "settings": {
    "number_of_shards": 2
  },
  "aliases": {
    "test_alias": {}
  }
}

GET _alias

GET test_alias
```

For Windows Environment to bulk import data into ES by using Postman

[Postman Chrome Extension](https://chrome.google.com/webstore/detail/postman/fhbjgbiflinjbdggehcddcbncdddomop/related?asid=26c77780&hl=zh-TW)

![](https://i.imgur.com/0LvbI5t.png)

![](https://i.imgur.com/ar8ZCJF.png)

- POST: http://localhost:9200/shakespeare/_bulk
- Header: Content-Type application/x-ndjson
- Body: binary 選檔案


# 8. Configuration & Go to production

#### cluster.name 不要用預設，以免增加node 跑到別人家去

# 9. Elastic Stack - Logstash, Beats, Kibana 的基本介紹



---


# 同學問答區

- 請問 在一台電腦(server)中  [name=Ryan]
    1. 要怎麼配置 適合的node數量
    2. 甚麼等級的電腦才合適
    3. 一開始使用AWS提供的ES服務，但配置上有鎖死(可支援硬碟大小與CPU)，滿好奇的是甚麼原因
    

> 記憶體沒有超過 64GB 的話，基本上起一台 node 即可，JVM heap 設定 <30 GB，剩下的記憶體留給 OS filesystem。
> 官方 practice 是建議不用給太 powerful 的硬體。
> [name=Joe]
> 
> [capacity planning](https://www.elastic.co/webinars/elasticsearch-sizing-and-capacity-planning)

- 如果要升級，是否 5.0 -> 7.15?

> 要一步一步的升，參考 https://www.elastic.co/guide/en/elasticsearch/reference/7.15/setup-upgrade.html
> [name=Joe]


- query cache 會存在哪個地方? 如果資料更新了，是從cache加速再從新搜尋嗎?[name=Ryan]

> JVM heap. segment file level cache -> 不會有新的資料或更新的問題
> [name=Joe]

- 所以在 Mapping的fields 那個 'original' 是可以自訂的對吧 [name=生魚片]
> 對，可以自行透過 mapping 來指定
> [name=Joe]


![](https://i.imgur.com/TRaZLyG.png)

- stored_fields 可以用來存圖片嗎?(效率好嗎[name=Ryan]
> 可以存，會使用 base64 儲存，所以儲存空間佔用較大，要存的話會建議另外存成 stored field，並考慮在 _source 中排除，但不建議特別做這樣的設計，如果有其他地方存的話更好，讓 ES 專注當搜尋引擎，不用當 image store。
> [name=Joe]


- Delete api 提到的批次處理，如果時間真的很久，這一次的request 如果已經timeout，要怎麼知道，該執行有做完呢?[name=Ryan]
> `GET _tasks?detailed=true&actions=*/delete/byquery`
> 可使用 `_tasks` API 來查看背景執行的 tasks
> [name=Joe]


- 若設計是把 log(每天約萬筆，需要永久保留) 都放在同一個index其實這樣是不健康的設計? [name=Ryan]
> 我產品目前 log 每天約莫為一千萬筆(上課之後要來做減少 log 的 review )( size 約 4GB 上下)，設計上是一天一個 index
> ![](https://i.imgur.com/Qvun7z0.png)[name=生魚片] 
> 我的經驗是，已經將資料做index區分，分別是info與debug，info為永久保留，debug為每日需要delete的部分。所以想探討說 **info這個index 是可以切成多個的嗎?**[name=Ryan]
> 資料會一直增長下去的話，就應該要考慮切。可以使用 Rollover 的機制，除了照時間切，也可以照大小來切，當 index 長到 50G 時，就觸發 rollover，產生新的 index 來收新的資料。 <br />
> 這部份也是 ILM 的機制之一。
> [name=Joe]

- 如果使用scroll做pagination的話, 需要做跳頁, Ex: 目前在第1頁要跳至第100頁, 這樣的話是需要scroll 99次嗎? [name=Lu Jason]
> 沒錯，這樣的使用情境可能就不適合用 scroll api。<br />
> 其實在搜尋引擎的應用上，是否真的有"跳到第100頁"的這種需求? <br />
> 真的要做，可以用 search after 來做，應該會是 ES 中比較好的解法。
> [name=Joe]


- 使用多組 analyzer 來處理文字的範例
> 可以參考 App Search 的使用方式 https://ithelp.ithome.com.tw/articles/10251102
> [name=Joe]

- 有時候我們 index 需要一直開著的話，那應該要用複製index嗎? [name=Ryan]
> 一般會使用donwtime的方式，不建議複製index 

- 如果node的analysis安裝不一致 會影響搜尋結果的分數?而不會讓搜尋失敗 [name=Ryan]



- 在實務操作上，遇到fields 上限 滿了，這時候應該選擇切index，還是調高數量上限呢? [name=Ryan]

> 建議更改格式變成Array list的方式
```
{
  "allcase": {
    "1": {
      "aaa": 1
    },
    "2": {
      "bbb": 1,
      "ccc": 1
    }
  }
}

{
  "allcases": [
    {
      "test_id": "1",
      "result": [
        {
        "name": "aaa",
        "value": 1
        }
      ]
    },
    {
      "test_id": "2",
      "result": [
        {
        "name": "bbb",
        "value": 1
        },
        {
        "name": "ccc",
        "value": 1
        }
      ]
    }
  ]
}
```


- kibana的Index patterns定義field的格式，是否只影響到visualization以及aggregation，而實際index的資料型態不變呢? [name=jwc911037]
> 是的



---

# Zoom 對話記錄

## Day 1

```
09:17:20	 From Tony Pai : 請問可以用 docker 嗎？
09:18:26	 From Tony Pai : 了解，謝謝
09:36:49	 From Allen Chen : 請問一個用 brew 啟動 一個用解壓縮啟動  兩個是不是互相看不到
09:39:28	 From Allen Chen : 了解 謝謝老師
我剛剛有看到兩個 cluster name 不一樣
09:56:15	 From Alan Lu : 我剛沒聽到
09:56:21	 From Alan Lu : 不是我
10:00:43	 From Ryan : 請問 在一台電腦(server)中 1.要怎麼配置 適合的node書量 2.甚麼等級的電腦才合適
10:01:13	 From DAKY 的iPad : 開三個node結果電腦變磚塊了重開中。。
10:02:54	 From jwc911037 : 嗨 喬老師~我的windows起兩個node後 看log似乎cluster UUID是一樣的，不過從dev tool看似乎兩個node沒有並在一起，可以再從哪個部分檢查呢?
10:04:58	 From KevinKe : hi 喬老師~ 請問M1啟兩個node會分家是正常嗎?
10:05:46	 From Allen Chen : hi 老師，所以一個node中 已經有data 也會影響兩個node 互相加入對嗎
10:07:05	 From jwc911037 : 好~我試試看 剛我是在起完node 1後直接複製整個資料夾 可能data/有影響到
10:11:41	 From Ryan : 了解 謝謝答覆
10:13:33	 From Tony Pai : 剛好
10:13:36	 From Alan Lu : 2
10:13:37	 From HH : 2
10:13:37	 From Tony Pai : 2
10:13:38	 From Sean : 2
10:13:39	 From Gemmy : 2
10:13:39	 From Allen Chen : 2
10:13:40	 From WEIZHE : 2
10:13:40	 From bill : 2
10:13:41	 From Eason Lin : 2
10:13:41	 From Lu Jason : 2
10:13:42	 From jwc911037 : 2
10:13:42	 From Jocelin : 2
10:13:43	 From 生魚片的iPadPro : 3
10:13:44	 From KevinKe : 2
10:13:45	 From Ryan : 3
10:13:46	 From papaisme : 2
10:13:51	 From 生魚片的iPadPro : 😂
10:17:28	 From Ryan : 陰影+1
10:17:32	 From Tony Pai : 我踩過...
10:25:09	 From Ryan : 請問這個跟kibana 的搜尋語法是一致的嗎
10:26:39	 From 生魚片的iPadPro : 常用的透過欄位資料排序順便教一下
10:27:39	 From 生魚片的iPadPro : 例如按照size大小排序
10:29:09	 From papaisme : 剛剛看起來的確是按照大小排序~~從 b -> kb -> mb
10:31:49	 From Allen Chen : 請問老師，什麼樣的情況　health  會變yellow
10:52:06	 From WEIZHE : 方便再貼一次hackmd位置嗎?
10:52:23	 From Joe Wu : https://hackmd.io/7hR1u16HQ4WsHLmtKAa72g?both
10:52:27	 From WEIZHE : 感謝
10:55:50	 From jwc911037 : 2
10:55:53	 From bill : 2
10:55:54	 From Eason Lin : 2
10:55:54	 From Sean : 3
10:55:55	 From Gemmy : ]
10:55:55	 From Lu Jason : 2
10:55:57	 From Gemmy : 2
10:55:58	 From Tony Pai : 2
10:56:00	 From KevinKe : 2
10:56:04	 From WEIZHE : 2
10:56:05	 From YiHeng Tsai : 2
10:56:24	 From Ryan : 3
10:56:36	 From DAKY 的iPad : 3
10:56:58	 From Allen Chen : 老師 您剛剛有提到
types 這塊會被棄用
但一些語法裡面　像
delete 還是會出現 _doc

這個部分有點不清楚
10:58:56	 From Ryan : 如果使用的是5.0 能夠升級到 7.0嗎
11:00:36	 From Ryan : 慢慢爬山的概念
11:02:01	 From Alan Lu : 請問這二天的課程會教到 Logstash 嗎?
11:02:07	 From WEIZHE : 想請問，我如何指定只找出title為Star Wars: Episode VII – The Force Awakens的資料呢?
11:02:53	 From Allen Chen : 謝謝老師，這樣我知道了
types 只剩 _doc

DELETE movies/999  <= 將來有可能這使用嗎?
11:03:17	 From Alan Lu : 瞭解
11:03:51	 From Ryan : 希望logstash 放多點時間+1
11:05:20	 From Lu Jason : 請問如果我搜尋像是tar的話, 要怎麼下關鍵字？
11:06:56	 From Ryan : 關於保留字的坑，可以補充嗎
11:07:44	 From Ryan : ES 打上去有保留字，會造成資料上不去
11:08:43	 From Tony Pai : 會不會是前面有講到的相同 index 不同 type 的問題
11:09:07	 From Ryan : 對對對
11:09:20	 From Ryan : 這個是 新版本已經改掉了?
11:10:48	 From Tony Pai : 喔喔 我這邊的 type 指的是資料型態
11:10:49	 From Ryan : 了解，一時想不到當初的例子 5.0
11:13:54	 From jwc911037 : 好奇 為何query一個index中沒有的關鍵字 elastic search怎麼從分數判斷沒有資料符合
11:21:10	 From YiHeng Tsai : 好奇詢問~那filter 多次被cache 之後，會對效能有影響嗎
11:21:29	 From Ryan : 類似問題
- query cache 會存在哪個地方? 如果資料更新了，是從cache加速再從新搜尋嗎?
11:41:47	 From papaisme : 有沒有什麼方式可以一次將一個indexing 裡面所有的資料 按照新的設定全部更新
11:42:03	 From DAKY 的iPad : 請問搜尋中文的話會有編碼的問題麼？big5
11:42:31	 From Tony Pai : 之前有碰到一樣的問題，原來可以用 mapping 解啊～

傻傻地開了新的 index 去 workaround
11:44:09	 From Ryan : 了解 謝謝
11:47:11	 From Tony Pai : 請問如果是 beat 在送資料到 ES indexing 的階段被擋住的話，一樣就是用 mapping 去加 keyword type 去解嗎？
11:50:38	 From papaisme : Okok 好的~~了解~~謝謝
11:52:32	 From Allen Chen : 老師不好意思 array 欄位要怎麼搜尋
11:57:11	 From Ryan : 剛剛提到的 remapping 有相關資料可以提供嗎
11:57:46	 From WEIZHE : 想知道怎麼查詢movies已經建立了那些Index
11:57:47	 From Ryan : 是的
12:00:00	 From WEIZHE : 了解  謝謝
12:07:56	 From 生魚片的iPhone : Lunch time
13:28:21	 From papa : Elastic 的 Shard 可以像是 MongoDB 一樣自己決定 Shard Key 嗎??
13:29:30	 From Ryan : 所以說 切node 盡量是不同的硬碟，以免硬碟壞了 還是都沒資料了
13:35:57	 From Ryan : 這與 snapshot 有關係嗎
13:53:13	 From Allen Chen : 老師您好，如果一個cluster中
有 1 個 Dedicated master-eligible  X 
跟 多個 master-eligible

master 的工作 每次都不會是固定的節點

但是當請求資料量龐大的時候
是否 master 幾乎就都是 X來擔任
13:53:13	 From YiHeng Tsai : 請問~有最高限制的shard數嗎?
13:54:26	 From Allen Chen : 了解 所以被選上 就不會再換人了嗎
13:54:58	 From Ryan : 這邊會探討 master 這效能 當作選舉條件嗎?
13:57:15	 From Allen Chen : 這樣 如果有一個節點
指派 Dedicated master-eligible
是否選舉時 會更容易被選上
13:59:13	 From Allen Chen : 不然 被指派為 Dedicated master-eligible
不當master 他就沒事做了~
13:59:23	 From Ryan : 了解 謝謝
14:00:41	 From Allen Chen : 了解 謝謝! 待命當master
14:52:32	 From jwc911037 : 那這樣的情況下只能把原本node 1起回來 整體的service才會恢復囉?
14:52:51	 From jwc911037 : 看起來只有node2時沒有master
14:54:55	 From WEIZHE : 想請教有現成的API可以監聽Document有version、_primary_term或_seq_no的變化嗎?
14:56:29	 From WEIZHE : 例如 setting存在elasticsearch，然後想要監聽如果value有變化的話，對應做action
14:57:46	 From WEIZHE : 收到!
14:57:49	 From WEIZHE : 感謝你!!
15:05:32	 From 生魚片的iPadPro : 各位都是拿es做產品功能的嗎？ 我只拿來處理log orz
15:06:08	 From Joe Wu : 我遇到過大概7成是拿來處理 logs，3成是做產品功能~
15:07:31	 From Joe Wu : 或是logs太複雜，處理的需求都快變成和產品功能一樣了…
15:08:27	 From jwc911037 : 嗯嗯 我這裡是主要處理log，一部分將log parsing出來當monitor XD
15:09:09	 From Alan Lu : Log + 1
15:10:07	 From jwc911037 : 想問一下 如果是Ingest node的資源分配 有沒有甚麼指標或是準則可以參考呢?
15:23:01	 From Ryan : 通常是甚麼情境會用這兩隻API，快速上傳!?
15:24:01	 From Ryan : 原來 filebeat 就是~
15:43:19	 From Lu Jason : 請問 max_result_window 如果設很大會有什麼影響嗎？
15:45:22	 From Ryan : sorting 是單一node 的效能嗎
15:46:38	 From jwc911037 : ES sorting的機制是以所有的文件做sorting 還是就回傳結果的文件做sort而已?
(比如說總共有10筆，size =3，sorting是拿10筆還是3筆做呢?
15:47:58	 From jwc911037 : 更正一下10筆 是符合條件的10筆
15:48:02	 From jwc911037 : 喔喔了解了 感謝
16:32:01	 From Ryan : 如果使用 Filebeat 會幫我們避免這些問題嗎
16:32:08	 From WEIZHE : 想請問如果node 1回來之後，會怎麼做把資料做同步呢?
16:34:21	 From WEIZHE : 了解 感謝
16:34:35	 From Ryan : 這邊指的load balancer 是ES 自己的嗎
16:49:01	 From Allen Chen : 老師，請問搜尋預設都會去 replica Shard嗎
16:51:52	 From Tony Pai : 3000?
16:51:58	 From jwc911037 : 990*3?
16:52:11	 From jwc911037 : 喔喔要在家10
17:04:17	 From Lu Jason : OK
17:04:18	 From Tony Pai : ok
17:04:20	 From WEIZHE : OK!
17:04:21	 From DAKY 的iPad : Ok
17:04:21	 From YiHeng Tsai : OK
17:04:24	 From 生魚片的iPadPro  To  Joe Wu(privately) : sure
17:04:24	 From Eason Lin : ok
17:04:26	 From Gemmy : OK
17:04:26	 From papa : 沒問題~~
17:04:28	 From Allen Chen : ok
17:04:31	 From jwc911037 : 賀
17:04:34	 From 生魚片的iPadPro : sure
17:04:43	 From HH : 可以 :)
17:18:50	 From Alan Lu : 我要去打疫苗了，先離開了，明天見。
17:19:29	 From Allen Chen : 老師，
Transaction log那邊
請問什麼情境 是 shard 被重新打開

是還沒有執行 refresh 機器重新啟動嗎
所以要重讀取 log 走refresh
17:20:55	 From DAKY 的iPad : 謝謝
17:20:56	 From Tony Pai : 謝謝～
17:20:58	 From 生魚片的iPadPro : 獲益良多
17:20:59	 From KevinKe : 感謝
17:21:24	 From Gemmy : 謝謝
17:21:57	 From Bill : 謝謝 感恩
17:22:08	 From HH : 謝謝喬叔~
17:22:43	 From Ryan : 這個章節講到部分，會是維護這台機器的人，會很常操作的項目嗎?
會是甚麼實務情境需?

感覺我好像無腦用了 ES 很久
17:23:25	 From 生魚片的iPadPro : 我也都沒這樣操作過 只有透過index看空間 做刪除
17:23:56	 From 生魚片的iPadPro : 都透過kibana封裝這些動作
17:24:01	 From Ryan : 原來如此
17:32:11	 From Jocelin : 謝謝～
17:32:12	 From 生魚片的iPadPro : OK
17:32:14	 From YiHeng Tsai : 感謝
17:32:16	 From WEIZHE : 謝謝老師
17:32:18	 From jwc911037 : 謝謝><
17:32:19	 From papa : 感謝老師
17:32:19	 From Allen Chen : 謝謝老師
17:32:21	 From Gemmy : 謝謝老師..
17:32:23	 From Ryan : 感謝
17:32:26	 From Lu Jason : 謝謝老師

```


## Day 2

```
09:00:59	 From Joe Wu : https://hackmd.io/7hR1u16HQ4WsHLmtKAa72g?both
09:39:04	 From jwc911037 : analyzer有辦法設定多個嗎? 有時在做語意分析的時候可能會同時使用Ngram和keyword
09:42:42	 From jwc911037 : 喔喔原來要用不同欄位切開才能用~感謝!!
09:42:56	 From papa : 有可能 indexing 和 search 用到的 analyzer 是不同的~~進而導致搜尋效果不佳嗎
09:45:46	 From papa : 了解~~
09:47:43	 From Sean Chen : 我這邊ok
10:01:07	 From Ryan : 有時候我們 index 需要一直開著的話，那應該要用複製index嗎?
10:07:54	 From Alan Lu : 用同義字的方式，是不是就不用設字根了? (STEM)
10:10:08	 From jwc911037 : symnonyn.txt這個檔案如果更新的話 ES會動態更新嗎?
10:10:41	 From Alan Lu : 不用回答我的問題了，我知道了，謝謝。
10:11:16	 From jwc911037 : 我的也是~
10:13:36	 From WEIZHE : 抱歉，方便於markdown貼上剛剛synonym設定的語法嗎?  感謝!
10:21:04	 From WEIZHE : 感恩
10:23:34	 From Allen Chen : 請問 lenient 的屬性 可以再請您解釋一次 不太清楚
10:26:06	 From Allen Chen : 了解! 謝謝老師
10:42:40	 From Ryan : 請問 analyzer 與 tokenizer ，這兩個差別是?
搞不清楚
10:45:02	 From Ryan : 那為什麼外掛可以分別下載?
10:54:49	 From Allen Chen : 老師 請問你剛剛字典的檔案 是使用 extra_main 嗎
10:55:26	 From jwc911037 : 剛有提到下載外部套件 ES會自動檢查套件的版本，如果碰到ES需要升級，但套件還沒更新到該版次是否就只能放棄這個套件呢?
10:55:33	 From Allen Chen : 喔喔 好 我知道了 謝謝
11:08:38	 From Allen Chen : 老師 如果一個 node 有自訂辭庫
一個沒有
搜尋結果會採用哪一種方式呢
11:13:09	 From Allen Chen : 喔喔 兩個coordinator 看誰處理 搜尋結果不同
11:13:24	 From Allen Chen : 了解 謝謝
11:36:37	 From Ryan : 在實務操作上，遇到fields 上限 滿了，這時候應該選擇切index，還是調高數量上限呢?
11:37:00	 From Ryan : 真的遇到了0.0
11:37:19	 From Ryan : 因為紀錄很多 測試項目
11:38:34	 From Ryan : 我們為了這個操作 還把 一部份 塞成 text 之後自己轉譯
11:39:59	 From Ryan : 資料格例如這樣
Bucket.AllCase.1
Bucket.AllCase.2
....
Bucket.AllCase.N
11:49:52	 From Ryan : 了解 謝謝
12:07:57	 From Ryan : 請問 nested搜尋 在kibana 也可以操作嗎
12:09:18	 From Ryan : 好的謝謝 有釣竿就行了
13:11:21	 From Allen Chen : 了解 謝謝
13:11:24	 From Ryan : Joe 可以貼實機操作 到MD嗎
13:11:53	 From Ryan : 地震?
13:11:59	 From Ryan : 幹
13:12:01	 From KevinKe : 好大
13:12:24	 From Alan Lu : 超大
13:12:31	 From Ryan : 對不起....
13:12:36	 From Eason Lin : 我要嚇死惹
13:12:45	 From Allen Chen : 晃好久
13:12:46	 From Alan Lu : 先開門
13:12:52	 From YiHeng Tsai : 南部現在開始
13:12:54	 From Tony Pai : 跑出去了 XD
13:13:08	 From HH : 台中很晃!!! 目前停了
13:13:17	 From jwc911037 : 現在換台南晃XD
13:13:26	 From KevinKe : 各位保重
13:13:27	 From Lu Jason : 沒事
13:13:31	 From HH : 還有餘震...
13:13:32	 From Tony Pai : 餘震
13:13:36	 From Ryan : 杯子都灑出來了...
13:13:38	 From jwc911037 : safe
13:13:49	 From Gemmy : Safe
13:13:57	 From HH : 安全!
13:14:18	 From Daky : 湯差點灑出來XD
13:14:47	 From Tony Pai : 我回來了
13:15:28	 From Ryan : 嚇倒忘光了
13:15:31	 From Ryan : 了
13:15:34	 From Ryan : 了解
13:21:24	 From Ryan : 這跟 Kibana Index Patterns 是一樣的意思嗎?
13:25:21	 From jwc911037 : kibana的Index patterns定義field的格式，是否只影響到visualization以及aggregation，而實際index的資料型態不變呢?
13:43:53	 From Ryan : 請問 若是換成中文 "蔡文" 也可以找到 蔡英文?
13:44:07	 From Allen Chen : 請問 slop 具體如何計算
13:53:55	 From Ryan : 請問 
如果上去是 timestamp 1514790000 就變成還需要轉譯過才可以搜尋 或是直接用 int 去搜尋區間?
13:55:16	 From Allen Chen : 老師請問

如果 input 一個時間給他

他預設都會是+0的時間嗎

還是有系統地區可以設定
13:56:26	 From Ryan : 還是說 需要 上傳到ES的時候 直接轉譯過一層 會比較順
13:56:39	 From Allen Chen : 了解!
client 資料要自己處理好
14:11:08	 From Ryan : 請問 alias 是怎麼產生出來的?
14:12:01	 From Ryan : 再次覺得 ES 水很深
14:29:44	 From Ryan : 請教 依照joe的經驗，在甚麼情況還會用"關聯資料庫"?
14:30:44	 From Allen Chen : 老師 
請問可以提供剛剛 index alias 的範例 code 嗎
14:32:37	 From Ryan : PS C:\> Invoke-RestMethod "http://localhost:9200/shakespeare/_bulk?pretty" -Method Post -ContentType 'application/x-ndjson' -InFile "c:\es\shakespeare.json"
Invoke-RestMethod : {
  "error" : {
    "root_cause" : [
      {
        "type" : "illegal_argument_exception",
        "reason" : "The bulk request must be terminated by a newline [\\n]"
      }
    ],
    "type" : "illegal_argument_exception",
    "reason" : "The bulk request must be terminated by a newline [\\n]"
  },
  "status" : 400
}
14:33:32	 From Ryan : 我的chrome 沒有下載@@?
14:34:20	 From Ryan : 是的
14:34:52	 From Ryan : 我自己另存的
14:35:05	 From Tony Pai : 一樣
14:35:07	 From Tony Pai : mac
14:35:34	 From jwc911037 : 我是開chrome另存新檔 可以用~~
14:36:05	 From Tony Pai : 想請問那個 `@` 的作用是？
14:36:20	 From jwc911037 : 不過有些錯誤
took errors items
 ---- ------ -----
10138  False {@{index=}, @{index=}, @{index=}, @{index=}...}
14:36:26	 From Lu Jason : 我用linux , 用curl的方式下載下來後可以
14:37:07	 From Tony Pai : 我 OK 了，因為我把 @shakespeare.json 改成 sharespeare.json
14:37:20	 From Tony Pai : 才會噴那個錯誤
14:37:36	 From Gemmy : (windows) 我的也是有些錯誤, 但我只有2637個
14:38:30	 From Ryan : 目前下載克服了

-------------------
took errors items
---- ------ -----
8815  False {@{index=}, @{index=}, @{index=}, @{index=}...}
14:39:19	 From Gemmy : 我愈作愈少個..剩took 1270
14:40:12	 From jwc911037 : 有
14:40:12	 From Ryan : 有
14:40:20	 From Gemmy : 有
14:40:51	 From Ryan : 因為json內容的關係嗎!?
14:41:08	 From Ryan : 還是要從 老師這邊 pass 出來一份
14:41:12	 From Allen Chen : 老師 這個指令沒辦法在 kibana console 模擬嗎 只能用系統指令
14:42:11	 From 生魚片的iPadPro : 我是用wsl2 正常
14:42:15	 From jwc911037 : 想問一下這行log的意思~
[2021-10-24T14:41:03,833][INFO ][o.e.m.j.JvmGcMonitorService] [node2] [gc][10207] overhead, spent [808ms] collecting in the last [1.7s]

剛我懷疑node1太忙 改打node2 (:9201)有秀這個
14:42:58	 From 生魚片的iPadPro : 對
14:43:14	 From Allen Chen : 可能是 windows 路徑斜線的問題嗎?
14:43:18	 From 生魚片的iPadPro : es我用docker跑的
14:45:09	 From Ryan : 這樣是成功嗎
{
  "took" : 8122,
  "errors" : false,
  "items" : [
    {
      "index" : {
        "_index" : "shakespeare",
        "_type" : "_doc",
        "_id" : "0",
        "_version" : 2,
        "result" : "updated",
        "_shards" : {
          "total" : 2,
          "successful" : 1,
          "failed" : 0
        },
        "_seq_no" : 111396,
        "_primary_term" : 1,
        "status" : 200
      }
    },
14:45:12	 From Ryan : 這是postman
14:47:11	 From 生魚片的iPadPro : 11萬筆
14:50:03	 From 生魚片的iPadPro : 我剛用cmd
14:50:15	 From 生魚片的iPadPro : 得用” 雙引號
14:50:25	 From 生魚片的iPadPro : 一樣有curl
14:50:39	 From 生魚片的iPadPro : 不過我是win11
14:51:14	 From Ryan : win 10 homw
------
curl --help
Usage: curl [options...] <url>
     --abstract-unix-socket <path> Connect via abstract Unix domain socket
     --alt-svc <file name> Enable alt
…..
14:54:24	 From WEIZHE : 我OK
15:17:09	 From Ryan : 不確定後面會不會提到kibana

因為我自己的學習經驗是 從kibana 圖表 轉 aggr 出來，也方便debug

想請問 新版本的 kibana  怎麼匯出 語法出來
15:25:35	 From Ryan : 瞭改 感謝示範
15:39:03	 From Ryan : 他已經回傳了 怎麼換?
----------------
 陰影+1
15:39:17	 From Ryan : 我要21
15:40:46	 From Ryan : 對欸 就是一模一樣的操作 把size　弄很大
－－－－－－－－－－－－－－－－－－－
只能這樣做對吧？
15:40:58	 From Tony Pai : 那是不要定義 size 好呢？還是要自己去算？
15:42:13	 From Ryan : 用時間區間　減少量！？
15:43:51	 From Ryan : 難怪我很常把 ES 弄掛...
15:44:14	 From Allen Chen : 老師
所以  upper_bound = 0
是因為
所有沒有計入排名的對象中 第一名總和的人
沒有超過 計入排名的最小值 是這個意思嗎
15:44:17	 From Tony Pai : 喔喔～
15:44:22	 From Tony Pai : 我以為是 maximum
15:45:14	 From Allen Chen : 了解 是所有人的資料都回傳了
15:45:30	 From YiHeng Tsai : 詢問個問題哦~假設我先用類似unique count 一個欄位，又需要算總和，會有精準度的設定方式嗎?
15:49:08	 From Tony Pai : 原來如此！
15:49:13	 From Ryan : +1
15:49:19	 From Allen Chen : 如果 terms 欄位內容的種類 是變動的

就沒辦法設定一個size 保持 誤差永遠為0
15:49:44	 From Tony Pai : 1 index 1 shard 就沒事的意思？XD
15:50:28	 From Allen Chen : 了解了
15:50:28	 From jwc911037 : sum_other_doc_count剛還是有點不太懂 可以再解說一次呢?
15:54:05	 From Tony Pai : 照這樣說 doc_count_error_upper_bound 只要愈大，這份回傳資料的可信度就愈低？
15:54:32	 From Tony Pai : 有點算錯沒關係的就還好 XD
15:56:09	 From Allen Chen : 喔喔喔 我了解了 是三個加總回來的總數
15:56:16	 From Allen Chen : 好 謝謝
16:26:36	 From bill  To  Joe Wu(privately) : 老師 抱歉 我臨時有點急事 要先離開 不好意思
16:28:27	 From Ryan : 可以這樣解釋嗎

當設定了 initial_master_nodes = 5
就會等到 5台都好，才會能夠服務
16:28:52	 From Alan Lu : 那如果是三台都斷呢?
16:28:57	 From Alan Lu : 三台都不是M?
16:29:08	 From Alan Lu : 瞭解
16:29:23	 From Allen Chen : 老師那恢復以後Ｍ　會移轉回去嗎
16:29:23	 From Lu Jason : 所以同時斷兩台也是依樣會一直等待嗎？
16:45:14	 From Tony Pai : 請問 index health 一直都是 yellow 的狀態，有什麼方法可以修復成 green 呢？
16:45:50	 From Ryan : 請問 優化效能，有甚麼 to do list , 或是官方文章?
16:46:16	 From Tony Pai : 謝謝～
16:47:38	 From Ryan : 了解 感謝
16:57:04	 From Ryan : 舉手舉手~~~
Filebeat 和 ES 中間要怎麼選?
在 Filebeat 約100多個節點，目前是串 Redis (Load banlance) + Logstash 做前處理
17:03:56	 From Alan Lu : 請問 Beats 可以直接把資料進 Elasticsearch，那 Beats 和 Logstash 的功能是不是有重壘或用 Beats 是否就可以不用 Logstash 呢?
17:05:12	 From Alan Lu : 瞭解，謝謝
17:06:27	 From jwc911037 : 想問個 有些人會選擇使用fluentd，不知有人是否知道fluentd和filebeat之間的優缺點? XD
17:08:34	 From WEIZHE : Beats <-> fluentbit
17:09:55	 From Ryan : 這類集群管理 有套裝的嗎

通通配好(雙手攤XDDDD
17:10:18	 From 生魚片的iPadPro : 我選fluentd只是因為簡單
17:14:01	 From 生魚片的iPadPro : 我用到security是資安說弱點掃描到我們的es 不用登入就能用 只好開帳號密碼
17:14:39	 From YiHeng Tsai : 應該還會有https的問題吧?XD我們公司有遇到這個問題XD
17:15:29	 From 生魚片的iPadPro : 沒對外 是沒有要求https
17:26:06	 From Ryan : 這邊log stash  的效能 也是根據 node 來操作嗎?
17:26:38	 From Ryan : 會比獨立架設一台logstash 來的簡單又不用管理效能，只需要加Node就好
17:27:49	 From Ryan : 如何避免log訊息遺漏呢?可能會需要一個的CheckLoglsAlive機制嗎?
17:28:09	 From Ryan : OMG
17:28:22	 From jwc911037 : dead letter queue?
17:29:40	 From Ryan : 對 重複!!!
17:33:26	 From Ryan : 難怪放乖乖都沒用
17:33:50	 From 生魚片的iPadPro : 多大量的資料啊
17:34:12	 From Ryan : 日均 5G 小小
17:34:29	 From 生魚片的iPadPro : 那很迷你
17:34:37	 From Ryan : 可能真的開太小台
17:34:45	 From 生魚片的iPadPro : 我一天 300G (raw)
17:35:20	 From Ryan : 瞭改
17:35:21	 From 生魚片的iPadPro : 但我機器都256G
17:35:27	 From Ryan : 記憶體!?
17:35:30	 From Ryan : 好猛
17:35:30	 From 生魚片的iPadPro : yes
17:35:40	 From 生魚片的iPadPro : 處理行情跟交易
17:36:04	 From jwc911037 : 感謝喬老師😀
17:36:12	 From 生魚片的iPadPro : 👏👏👏滿滿的收穫
17:36:14	 From Gemmy : 謝謝老師...
17:36:14	 From Lu Jason : 謝謝老師
17:36:15	 From Jocelin : 謝謝～～
17:36:16	 From Uerica_onedoggo : 感謝大家的熱情參與～～～～
17:36:19	 From YiHeng Tsai : 非常感謝
17:36:20	 From KevinKe : 謝謝～
17:36:22	 From Ryan : 感謝 真的補了很多
17:36:24	 From Tony Pai : 謝謝～
17:36:26	 From Allen Chen : 謝謝老師
17:36:32	 From papa : 感謝喬叔~~
17:36:34	 From 生魚片的iPadPro : 上完課還有很多債要還
17:36:34	 From WEIZHE : 謝謝老師
17:36:43	 From Ryan : 真的 回去還債
17:37:51	 From 生魚片的iPadPro : 流行很久了 只是 都拿來做elk 都沒細用
17:37:54	 From 生魚片的iPadPro : 在產品上
17:38:06	 From 生魚片的iPadPro : 只都是知道用在log
17:38:14	 From Ryan : 黑阿 我們就是做log 而已
17:39:26	 From 生魚片的iPadPro : Joe是Pioneer
17:41:05	 From 生魚片的iPadPro : 我們只有開kibana來查log Orz
17:41:16	 From YiHeng Tsai : 超感謝老師這次鐵人賽寫的項目~我剛好最近在摸APM相關...這次課程也收穫很多
17:41:28	 From YiHeng Tsai : 我們公司用超多elastic stack的
17:41:51	 From Ryan : 要去報名了這樣XDD
17:41:58	 From YiHeng Tsai : 高雄某大廠XD
17:42:34	 From YiHeng Tsai : 好XD可能會需要
17:42:50	 From 生魚片的iPadPro : 海量時序資料解決方案可以找我 ：XD
17:43:38	 From Ryan : 股票 or 房地產這樣嗎XDD
17:43:47	 From 生魚片的iPadPro : 對
17:43:52	 From Ryan : 一看就是大咖
17:43:54	 From 生魚片的iPadPro : 交易 物聯網 都適合
17:44:20	 From 生魚片的iPadPro : 主打金融市場 物聯網之類的
17:44:34	 From 生魚片的iPadPro : 感謝各位～～～兩天學到超多～～
17:45:02	 From Ryan : 早點上到課 就不會踩到坑XDD
17:45:07	 From Ryan : 血淚阿~
17:45:38	 From 生魚片的iPadPro : OK
17:45:49	 From YiHeng Tsai : 踩坑+1...
希望能多多推廣elastic stack的東西XD
17:46:05	 From YiHeng Tsai : 少走些冤枉路

```