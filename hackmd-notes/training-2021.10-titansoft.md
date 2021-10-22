# 喬叔的 Elasticsearch 基礎實務班 (2021.10.16,17,21 鈦坦科技企業內訓)

[![hackmd-github-sync-badge](https://hackmd.io/YZNLmE4ZRs28v5XX5lplDg/badge)](https://hackmd.io/mu2E-Wf9QRW-hTE3_wUveQ?both)

> ## 相關連結
>
> 課程講義電子檔 (10/21下課刪檔)：[Elasticsearch 基礎實務班-202110鈦坦科技企業內訓客製版_v2](https://drive.google.com/file/d/13ddMmsvtmEXKyB7wkEsxzUEqG_bezx2k/view?usp=sharing)
>
> Facebook 粉絲頁：[喬叔 - Elastic Stack 技術交流](https://www.facebook.com/Joe.ElasticStack)
>
> 課程補充資訊網站：https://es.joecwu.com/
>
> 課程示範操作原始碼：https://es.joecwu.com/training-materials
>
> 課後問卷連結：https://forms.gle/g8nWE6xaQze6d5LQ6

:::info
:bulb: **提示**: 如果你不太熟 HackMD，[這邊](https://hackmd.io/features-tw?both) 有使用的技巧範例。
:::

---
索引

[TOC]

第一天的課程
===

# 1. Elasticsearch 快速上手


啟動時，如果要使用參數指定 node name：
```
./bin/elasticsearch -E node.name=node1
```
 
 修改 `config/jvm.options` 調整 JVM heap size
 ```
-Xms4g
-Xmx4g 
 ```
 
 Docker-compose版調整JVM heap size
 ```
 environment:
   ES_JAVA_OPTS: "-Xms4g -Xmx4g"
 ```

如果兩個Elasticsearch node沒有成為Cluster
> 試著刪掉 data folder並逐一重啟


查詢 indices without system 內建
```
GET _cat/indices/*,-.*?v
```

存入一筆doc
```
PUT /movies/_doc/1
{
  "title": "Star Wars: Episode VII – The Force Awakens",
  "director": "J.J. Abrams",
  "year": 2015
}
```

取出一筆doc
```
# With metadata
GET /movies/_doc/1

# Without metadata
GET /movies/_doc/1/_source

# Only specified fields
GET /movies/_doc/1/?_source=title,director
```

Serach
```
POST _search/
{
 "query": {
   "query_string": {
     "default_field": "title",
     "query": "wars"
   }
  }
}

# or
GET _search?q=wars
```

從特定 indices search
```
GET <indices>/_search

e.g.
GET /movies/_search?q=wars
```

Filter with multiple fields
```
E.g. year=2014 and genres has Adventure

POST /movies/_search
{
  "query": {
    "bool": {
      "filter": [
        {
          "term": {
            "year": "2014"
          }
        },
        {
          "term": {
            "genres.keyword": "Adventure"
          }
        }
      ]
    }
  }
}
```

查詢時回傳的 took 代表在 elasticsearch 內部查詢實際占用的時間 ms (不包含網路延遲)


[find structure tool](https://www.elastic.co/guide/en/elasticsearch/reference/current/find-structure.html) 提供文件，幫忙找出 mapping 的定義



# 2. Elasticsearch 基本介紹、集群架構與 Lucene 簡介



# 3. Data In/Data Out 基本操作

## Alias

## Index Alias

### Add alias with date math
```
PUT movies

POST _aliases
{
  "actions": [
    {
      "add": {
        "index": "movies",
        "alias": "<movies-{now{yyyy.MM.dd|+08:00}}>"
      }
    }
  ]
}
```

### Get alias

```
GET _alias/movies*
```

### 情境 1 - 提供單一名稱，查詢時對照到多個 Index，寫入時指定其中一個 Index

```
PUT <test-{now-2d}>
PUT <test-{now-1d}>
PUT <test-{now}>

POST _aliases
{
  "actions": [
    {
      "add": {
        "index": "test-*",
        "alias": "test"
      }
    },
    {
      "add": {
        "index": "<test-{now}>",
        "alias": "test",
        "is_write_index": true
      }
    }
  ]
}

GET test*/_alias
```

### 情境 2 - 搭配 Filter 做到權限控管、商業邏輯封裝的使用。
```
POST _aliases
{
  "actions": [
    {
      "add": {
        "index": "movies",
        "alias": "crime-movies",
        "filter": {
          "bool": {
            "filter": {
              "term": {
                "genres.keyword": "Crime"
              }
            }
          }
        }
      }
    }
  ]
}

GET crime-movies/_search
```

### 情境 3 - 避免 Client 端直接存取原始 Index，支援 Index 名稱上的版控

```
PUT drama_v1

POST _aliases
{
  "actions": [
    {
      "add": {
        "index": "drama_v1",
        "alias": "drama"
      }
    }
  ]
}
```

### 情境 4 - 做到 blue/green deployment

```
PUT drama_v2

## option1: keep drama_v1

POST _aliases
{
  "actions": [
    {
      "remove": {
        "index": "drama_v1",
        "alias": "drama"
      }
    },
    {
      "add": {
        "index": "drama_v2",
        "alias": "drama"
      }
    }
  ]
}

## option2: remove drama_v1 directly

POST _aliases
{
  "actions": [
    {
      "add": {
        "index": "drama_v2",
        "alias": "drama"
      }
    },
    {
      "remove_index": {
        "index": "drama_v1"
      }
    }
  ]
}

GET drama*/_alias
```

Alias Index 可以使用wildcard來設定，但只會包含到當下存在的index。


# 4. Elasticsearch Under the Hood

## how indexing request work

![](https://i.imgur.com/OmXYo4q.jpg)


## how searching request work

![](https://i.imgur.com/rbhWbgN.jpg)



# 5. Text Analysis 的基本介紹

ngram
```
min gram= 2
max gram= 2

Apple
------
Ap
 pp
  pl
   le



min gram= 2
max gram= 3

Apple
------
Ap
 pp
  pl
   le
App
 ppl
  ple


```

edge-ngram
```
min gram = 3
max gram = 5

Apple
------
App
Appl
Apple



```


第二天的課程
===


# 6. Mapping 的介紹與管理方式

![](https://i.imgur.com/8fzfrJY.png)
> 要移掉這些欄位，可以將 filebeats 使用的 index templates，改成自訂的，避免直接產生 ECS 的 fields.


### 情境題 - 客服部門只能存取最近一年的資料，其他人能正常存取全部資料

```
PUT _component_template/audit-logs_settings
{
  "version": 1,
  "template": {
    "settings": {
      "index.auto_expand_replicas": "1-3"
    }
  }
}

PUT _component_template/audit-logs_mappings
{
  "version": 1,
  "template": {
    "mappings": {
      "properties": {
        "@timestamp": {
          "type": "date"
        }
      }
    }
  }
}

PUT _component_template/audit-logs_cs_alias
{
  "version": 1,
  "template": {
    "aliases": {
      "cs-{index}": {
        "index": "audit-logs*",
        "filter": {
          "bool": {
            "filter": {
              "range": {
                "@timestamp": {
                  "gte": "now-1y"
                }
              }
            }
          }
        }
      }
    }
  }
}

PUT _index_template/audit-logs
{
  "index_patterns": [
    "audit-logs*"
  ],
  "template": {
    "mappings": {},
    "settings": {},
    "aliases": {}
  },
  "priority": 100,
  "composed_of": [ "audit-logs_settings", "audit-logs_mappings", "audit-logs_cs_alias" ]
}

PUT audit-logs-order/_doc/1
{
  "@timestamp": "2021-10-16",
  "test": 1
}


PUT audit-logs-order/_doc/2
{
  "@timestamp": "2019-10-16",
  "test": 2
}

PUT audit-logs-payment/_doc/1
{
  "@timestamp": "2019-10-16",
  "test": 123
}

GET audit-logs-order
GET audit-logs-payment

GET cs-audit-logs-order/_search
GET cs-audit-logs-payment/_search

GET audit-logs-order/_search
GET audit-logs-payment/_search

```


# 7. Search - Query & Aggregation


![](https://i.imgur.com/FMqMXfX.jpg)



# 8. Configuration & Go to production

```
GET _cluster/settings?include_defaults=true&flat_settings


PUT _cluster/settings
{
  "persistent": {
    "cluster.routing.allocation.enable": "primaries"
  }
}
```


# 9. Elasticsearch Security & RBAC


第三天的課程
===

# 10. Elasticsearch 資料管理、Index Lifecycle Management


### Rollover API

```
# ----- Preparation -----
# PUT <ro-{now/d}-000001>
PUT %3Cro-%7Bnow%2Fd%7D-000001%3E
{
  "aliases": {
    "ro": {
      "is_write_index": true
    }
  }
}

GET _alias/ro
GET ro/_search

# ----- Rollover -----
# first try
POST ro/_rollover
{
  "conditions": {
    "max_age": "5m",
    "max_docs": 10,
    "max_primary_shard_size": "1mb"
  }
}

# import data
POST _reindex
{
  "source": {
    "index": "top_rated_movies"
  },
  "dest": {
    "index": "ro"
  },
  "script": {
    "source": "ctx._id=null",
    "lang": "painless"
  }
}

# second try, meet criteria and perform rollover
POST ro/_rollover
{
  "conditions": {
    "max_age": "5m",
    "max_docs": 10,
    "max_primary_shard_size": "1mb"
  }
}

# check index & alias status
GET _alias/ro

```


### Force Merge

```
POST shrink_v4/_forcemerge?max_num_segments=1
```

### Custom Allocation

- add `node.attr` in `elasticsearch.yml`
```
# node-1
node.attr.rack: r1
node.attr.pet: cat

# node-2
node.attr.rack: r2
node.attr.pet: dog
```

- restart ES & check node attributes

```
GET _cat/nodeattrs?v
```

- create index w/ `index.routing.allocation` settings.

```

# create index
PUT attr_test
{
  "settings": {
    "number_of_shards": 5, 
    "number_of_replicas": 1, 
    "index.routing.allocation.include.rack": "r1,r2",
    "index.routing.allocation.exclude.pet": "dog"
  }
}

GET _cat/shards/attr_test?v

GET _cluster/allocation/explain

# not working
PUT attr_test/_settings
{
  "index.routing.allocation.include.rack": "r1,r2"
}

# because attributes have not been removed
GET attr_test/_settings?include_defaults=true&filter_path=**.routing.**

PUT attr_test/_settings
{
  "index.routing.allocation.include.rack": "r1,r2",
  "index.routing.allocation.exclude.pet": null
}

GET _cat/shards/attr_test?v

# [備用補充] 如果要反向的話，要手動移掉已分派好的 shards

POST /_cluster/reroute
{
  "commands": [
    {
      "cancel": {
        "index": "attr_test",
        "shard": 4,
        "node": "node-2"
      }
    }
  ]
}

```


#### Create Searchable Snapshot on Kibana

- go to kibana > stack management > data > snapshot and restore > repositorys to register new repository.


- mount searchable snapshot with fully mounted
```
POST /_snapshot/demo/daily-snapshot-2021.10.20-aq4f2xodq7efc8vd4iswya/_mount?wait_for_completion=true
{
  "index": "movies", 
  "renamed_index": "ss_movies", 
  "index_settings": { 
    "index.number_of_replicas": 0
  }
}
```

- search from searchable snapshot

```
GET ss_movies/_search
```

- we can also reindex from searchable snapshot
```
POST _reindex
{
  "source": {
    "index": "ss_movies"
  },
  "dest": {
    "index": "movies"
  }
}
```

- mount searchable snapshot with partial mounted
```
DELETE ss_movies

POST /_snapshot/demo/daily-snapshot-2021.10.20-aq4f2xodq7efc8vd4iswya/_mount?wait_for_completion=true&storage=shared_cache
{
  "index": "movies", 
  "renamed_index": "ss_movies", 
  "index_settings": { 
    "index.number_of_replicas": 0
  }
}
```

- query will failed due to no available shared cache
```
GET ss_movies/_search
```

- update `elasticsearch.yml` to setup shared cache
```
xpack.searchable.snapshot.shared_cache.size: 10MB
```

- after restart ES, we can search now.
```
GET ss_movies/_search
```

- we can also check cache stats of searchable snapshot
```
GET /_searchable_snapshots/cache/stats
```


### Data Stream

```
# create data stream but failed
PUT /_data_stream/my-data-stream

# create index template w/ data stream declaration
PUT _index_template/ds-test
{
  "index_patterns": [
    "my-data-stream*"
  ],
  "data_stream": {},
  "template": {
    "settings": {
      "number_of_shards": 3,
      "number_of_replicas": 1
    }
  }
}

# this step is optional
PUT /_data_stream/my-data-stream

GET my-data-stream/_settings

# must declare op_type=create
PUT my-data-stream/_doc/1
{}

# must provide timestamp
PUT my-data-stream/_doc/1?op_type=create
{}

PUT my-data-stream/_doc/1?op_type=create
{
  "@timestamp": "2021-10-20T00:00:00"
}

# search data
GET my-data-stream/_search

# unable to delete backing index
DELETE .ds-my-data-stream-2021.10.15-000001

# delete data stream
DELETE _data_stream/my-data-stream

```

### Create ILM

#### Create ILM on Kibana

1. create demo-ilm

2. Hot

- Rollover on 10 docs
- enable Force Merge: 1 segment only
- Shrink to 1 shard
- Read only

3. Warm

- Move data into phase when 1 minute
- Replicas: 1
- Read only

4. Cold

- Move data into phase when 2 minute
- Freeze

5. Frozen

- Move data into phase when 3 minute
- Searchable snapshot

6. Delete

- After 10 minutes

#### Create Index Template on Kibana

1. create index template

- Name: ilm-demo
- Idnex pattern: ilm-demo*
- Data stream: true
- Priority: 100
- Version: 1

- Setup Index Settings
```
{
  "number_of_shards": 8
}
```

#### Back to ILM and apply to Index Template

- apply to: ilm-demo
- alias for rollover: ilm-demo

#### Indexing Docs

```
POST ilm-demo/_doc/
{
  "@timestamp": "2021-10-20T00:00:00"
}

```

- option 2: index + alias case, we have to manual create first rolling index.

```
# manual create first rolling index (only for index w/ alias case, no need for data stream)
PUT joe-test-000001
{
  "aliases": {
    "joe-test": {
      "is_write_index": true
    }
  }
}
```

#### Setup ILM Poll Interval for DEMO

```
# ILM related settings: https://www.elastic.co/guide/en/elasticsearch/reference/current/ilm-settings.html
PUT _cluster/settings
{
  "transient": {
    "indices.lifecycle.poll_interval": "10s"
  }
}

POST ilm-demo/_doc/
{
  "@timestamp": "2021-10-20T01:00:00"
}

```

#### keep indexing & check index status

```
GET ilm-demo/_ilm/explain

POST ilm-demo/_doc/?refresh=true
{
  "@timestamp": "2021-10-20T00:00:00"
}

GET ilm-demo/_search

GET _cat/indices/*demo*?v
# check for rollover (after poll interver)
# check for shrink (after 1 mins)

# after index covert to frozen index
GET ilm-demo/_search?ignore_throttled=false

# after some data moving to searchable snapshop, index will be renamed to partial-shard-xxxxx


# tips: monitor specific index
GET ilm-demo/_ilm/explain?filter_path=*.*003.*
GET .*demo*008/_search

GET _cat/segments?index=*demo*&v
```

# 11. Elasticsearch Ingest Pipeline



## Ingest Pipeline

### Dissect

- also create on Kibana

```

POST _ingest/pipeline/_simulate
{
  "pipeline": {
  "description" : "parse multiple patterns",
  "processors": [
    {
      "dissect": {
        "field": "message",
        "pattern" : "%{clientip} %{ident} %{auth} [%{@timestamp}] \"%{verb} %{request} HTTP/%{httpversion}\" %{status} %{size}"
       }
    }
  ]
},
"docs":[
    {
      "_source": {
        "message": "8.8.8.8 - - [30/Apr/1998:22:00:52 +0000] \"GET /english/venues/cities/images/montpellier/18.gif HTTP/1.0\" 200 3171"
      }
    }
  ]
}


```


### Grok

```

# `field`：從哪個欄位中讀取資料。
# `patterns`：Grok patterns 的描述字串，可以使用已經定義好的 patterns，也可以自己定義。(上例是將比對到的結果，存放到 `pet` 欄位中)
# `pattern_definitions`：可以自行指定相同字串比對的規則，值的宣告中，也可以使用 `|` 來定義多個值。
# `trace_match`：是否要回傳 `_grok_match_index` 這個 grok 執行結果的資訊。


POST _ingest/pipeline/_simulate
{
  "pipeline": {
  "description" : "parse multiple patterns",
  "processors": [
    {
      "grok": {
        "field": "message",
        "patterns": ["%{FAVORITE_DOG:pet}", "%{FAVORITE_CAT:pet}"],
        "pattern_definitions" : {
          "FAVORITE_DOG" : "beagle",
          "FAVORITE_CAT" : "burmese"
        },
        "trace_match": true
      }
    }
  ]
},
"docs":[
  {
    "_source": {
      "message": "I love burmese cats!"
    }
  }
  ]
}

```


### GeoIP

- 可以在 kibana 直接操作

```
PUT _ingest/pipeline/geoip
{
  "description" : "Add geoip info",
  "processors" : [
    {
      "geoip" : {
        "field" : "clientip"
      }
    }
  ]
}
PUT geoiop_test/_doc/1?pipeline=geoip
{
  "clientip": "8.8.8.8"
}
GET geoiop_test/_doc/1

```

- testing data for kibana UI

```
[
  {
    "_source": {
      "clientip": "8.8.8.8"
    }
  }
]
```

### uriparts

```
# create uri_parts on Kibana

PUT _ingest/pipeline/uri_parts
{
  "processors": [
    {
      "uri_parts": {
        "field": "request"
      }
    }
  ]
}

```

### Pipeline

```
# add dissect, geoip, uri_parts

# test data
[
  {
    "_source": {
      "message": "8.8.8.8 - - [30/Apr/1998:22:00:52 +0000] \"GET /english/venues/cities/images/montpellier/18.gif HTTP/1.0\" 200 3171"
    }
  },
  {
    "_source": {
      "message": "61.195.147.131 - - [09/Jan/2015:19:12:06 +0000] \"GET /inventoryService/inventory/purchaseItem?userId=20253471&itemId=23434300 HTTP/1.1\" 500 17"
    }
  }
]

# add one more processor - K/V
# field: url.query
# field split: &
# value split: =
# Ignore missing

```

### Enrich Processor

```
# prepare source index
PUT /users/_doc/1?refresh=wait_for
{
  "email": "mardy.brown@asciidocsmith.com",
  "first_name": "Mardy",
  "last_name": "Brown",
  "city": "New Orleans",
  "county": "Orleans",
  "state": "LA",
  "zip": 70116,
  "web": "mardy.asciidocsmith.com"
}

# create policy
PUT /_enrich/policy/users-policy
{
  "match": {
    "indices": "users",
    "match_field": "email",
    "enrich_fields": [
      "first_name",
      "last_name",
      "city",
      "zip",
      "state"
    ]
  }
}

# execute policy
POST /_enrich/policy/users-policy/_execute

# check index & alias created by enrich policy
GET .enrich-users-policy

# create ingest pipeline (可以從 kibana 建立)
PUT /_ingest/pipeline/user_lookup
{
  "processors" : [
    {
      "enrich" : {
        "description": "Add 'user' data based on 'email'",
        "policy_name": "users-policy",
        "field" : "email",
        "target_field": "user",
        "max_matches": "1"
      }
    }
  ]
}

# testing data for kibana
[
  {
    "_source": {
      "email": "mardy.brown@asciidocsmith.com"
    }
  }
]

# ingest new data
PUT /test_user_lookup/_doc/1?pipeline=user_lookup
{
  "email": "mardy.brown@asciidocsmith.com"
}

# check result
GET test_user_lookup/_search
```





# 12. Elastic Stack 的基本介紹



## Elastic Stack - Filebeat

- 下載 dataset - Apache Access Logs

- create `filebeat_user` role
```
# cluster privileges
- manage_ilm, monitor, manage_index_templates


# index privileges
- filebeat-*
- create, create_index, index, manage
```

- create `filebeat` user w/ `filebeat_user` priviledge only.

- 設定 filebeat.yml

```
- type: filestream
  enabled: true
  paths:
    - /Users/joecwu/Training/materials/filebeat/filebeat-logs/*

output.elasticsearch:
  hosts: ["localhost:9200"]
  protocol: "https"
  ssl.certificate_authorities: ["/Users/joecwu/Training/beats/filebeat-7.15.1-darwin-x86_64/elasticsearch-ca.pem"]
  username: "filebeat"
  password: "changeme"

```

- [bypass] create keystore & setup password (if needed)

```
./filebeat keystore create
./filebeat keystore add ES_PWD

```

- setup modules if needed

```
./filebeat modules enable {module_name}
```

- init kibana dashboard & ES index template

```
./filebeat setup -e
```

- start filebeat

```
./filebeat -e
```


### 收集 Elasticsearch Logs

```
# Module: elasticsearch
# Docs: https://www.elastic.co/guide/en/beats/filebeat/7.x/filebeat-module-elasticsearch.html

- module: elasticsearch
  # Server log
  server:
    enabled: true

    # Set custom paths for the log files. If left empty,
    # Filebeat will choose the paths depending on your OS.
    var.paths:
      - /Users/joecwu/Training/elasticsearch-*/logs/*_server.json

  gc:
    enabled: true
    # Set custom paths for the log files. If left empty,
    # Filebeat will choose the paths depending on your OS.
    var.paths:
      - /Users/joecwu/Training/elasticsearch-*/logs/gc.log.[0-9]*
      - /Users/joecwu/Training/elasticsearch-*/logs/gc.log

  audit:
    enabled: true
    # Set custom paths for the log files. If left empty,
    # Filebeat will choose the paths depending on your OS.
    var.paths:
      - /Users/joecwu/Training/elasticsearch-*/logs/*_audit.json

  slowlog:
    enabled: true
    # Set custom paths for the log files. If left empty,
    # Filebeat will choose the paths depending on your OS.
    var.paths:
      - /Users/joecwu/Training/elasticsearch-*/logs/*_index_search_slowlog.json
      - /Users/joecwu/Training/elasticsearch-*/logs/*_index_indexing_slowlog.json

  deprecation:
    enabled: true
    # Set custom paths for the log files. If left empty,
    # Filebeat will choose the paths depending on your OS.
    var.paths:
      - /Users/joecwu/Training/elasticsearch-*/logs/*_deprecation.json
```





# 同學問答區

- `"action.auto_create_index": false` 設定的時候，ILM rollover 會不會被影響，而無法新增 index

> 不會，rollover 自己會建立新的 index，不是透過 indexing doc 時的 `auto create` 的這種行為。<br />
> 另外 component template 有 `allow_auto_create: true` 的設定，可以用在自己定義的 index_template 中，來避開這個設定。<br />
> 但是在 Kibana 中好像這屬性的設定 UI 還沒加上。(7.15.1)
> [name=Joe Wu] [color=red]



- Kibana 的 discover 中，以 JSON 檢視，可以看到 `fields` 的欄位，是否可以不儲存 `_source` 直接用這個欄位的值，來節省空間?

> 結論：非 `text` 的欄位可以考慮這樣做，但要考慮好不儲存 `_source` 的取捨。<br /><br />
> 這個 `fields` 欄位的由來，是來自於 Doc Values，是在 `_search` 時，加上 `docvalue_fields=<field_name>` 的方式取得。
> 不過 `text` 類型的欄位，不會存 doc value。
> `keyword`, numeric 類型的欄位可以這樣用。
> 效能上不會有什麼影響，搜尋還是用 inverted index，doc values 本來就是會在 indexing 時儲存，查詢時只是針對這部份取資料出來，所以評估效能上不會有影響。
> [name=Joe Wu] [color=red]
> :::warning
> 注意 :zap: kibana discover 還是會要有 `_source` 才能使用，所以至少安排一個欄位在 `_source`，剩下的欄位，沒有在 `_source` 中的，Kibana discover 不會出現，要另外設定 `scripted_filed`，把值從 doc_value 中抓出來，才能在 discover 中出現。
> :::


- shard allocation 針對不同 rack 時，如何做到 primary shard 被分配到其中一個 rack 時，replica 要在另一邊

> 參考 [shard allocation awareness - force awareness](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-cluster.html#forced-awareness)
> [name=Joe Wu] [color=red]