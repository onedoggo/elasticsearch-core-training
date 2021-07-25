# 喬叔的 Elasticsearch 基礎實務班 2021-07-24~25

## 上課 Google Meet 對話記錄 07-24

```
你上午8:54
1
Zonyo Tao上午8:54
1
陳包包上午8:54
1
楊德倫上午8:54
1
李居翰上午8:54
1
Huang Core上午8:54
1
Kuang Li上午8:54
1
Grace J上午8:54
1
吳信賢上午8:54
1
林泳伸上午8:54
1
Solaris T上午8:54
1
Tracy Tung上午8:54
1
你上午8:58
先請大家確認
1: 聽得到我的聲音
2: 看得到分享的螢幕
陳包包上午8:58
1
2
Grayson上午8:58
1
Huang Core上午8:58
12
李居翰上午8:58
12
Grace J上午8:58
1 & 2
Tracy Tung上午8:58
12
YenTing Chen上午8:58
1 2
林泳伸上午8:58
12
Kuang Li上午8:58
12
Mark Weng上午8:58
1&2
陳怡升EasonC13上午8:58
12
你上午9:03
=================
陳怡升EasonC13上午9:04
1
陳包包上午9:04
1
Tracy Tung上午9:04
1
Roberson Liou上午9:04
1
Mark Weng上午9:04
1
吳信賢上午9:04
1
王慕羣上午9:04
1
Grayson上午9:04
1
Zonyo Tao上午9:04
1
Grace J上午9:04
1
Sam Yang上午9:04
1
Huang Core上午9:04
2
Solaris T上午9:04
1
楊嘉榮上午9:04
1
林泳伸上午9:04
2
HW Chen上午9:04
2
楊德倫上午9:04
1
李居翰上午9:04
2
Kuang Li上午9:04
1
Yuen-Hsien Tseng上午9:05
1
陳怡升EasonC13上午9:21
老師，我的 config 在 libexec 路徑下
bin 也是
陳怡升EasonC13上午9:22
/usr/local/Cellar/elasticsearch-full/7.13.4
Uerica Wang上午9:28
打擾一下，喬叔你視訊拍自己的畫面有點問題，畫面有殘影喔~
Zonyo Tao上午9:38
process turn off 是用 ctrl+c 嗎
剛剛沒注意到 老師怎麼把正在跑的 process 關掉
Sam Yang上午9:47
用brew的話是不是就不能裝兩個
陳怡升EasonC13上午9:47
我成功了，第二個要設定一下
第一個是 brew
學長做到哪步？
李居翰上午9:47
windows 版的起第二個node沒有自動加入 node1 
Sam Yang上午9:47
我剛剛也是第一個用brew第二個用下載的
但是兩個連不到的樣子
Roberson Liou上午9:47
老師我用 windows 跑了兩個 node
不過用 cat 看只有看到一個 IP
但 9200 跟 9201 都有通
Mark Weng上午9:48
如果是用Llinux ubuntu apt裝的話 可以裝兩個嗎?
陳怡升EasonC13上午9:48
GET _cat/nodes
兩個連不到，因為 cluster name 不同
要改 cluster name 跟改 data path
Sam Yang上午9:49
喔喔 對耶
沒關係 我直接用download的就好了
楊德倫上午9:50
建議都下載 zip 來進行 clustering，brew 指令安裝的部分，先省略
Mark Weng上午9:50
好的好的
HW Chen上午9:52
老師，請問我已經開啟第二個 node 了，不過在使用 nodes?v 還是只有看到一個，會需要做什麼才能看到兩個嗎？
陳怡升EasonC13上午9:54
HW
兩個連不到，因為 cluster name 不同
要改 cluster name 跟改 data path
楊德倫上午9:54
我第一個 node 會先出現 started，後面沒有多餘的訊息，再開啟第二個 node，此時第二個 node 會去偵測並連接第一個 node (via 9300 port)，所以如果還是沒連接，建議先開第一個，確認它有 started，同時沒有多餘訊息，再開啟第二個 node
王慕羣上午9:56
因為我一開始只有解壓縮一次 然後就把壓縮檔刪掉了 後來我的做法是先把 data/ 裡面的內容刪乾淨 然後再重啟 應該就會有不同的 node 出現了
陳怡升EasonC13上午9:56
我卡在 kibana 不能用
HW Chen上午9:57
可以了，謝謝大家
楊德倫上午9:58
如果真的不行，node 全刪，再解壓成兩個資料夾，設定 -1 和 -2 作為 postfix，先開啟第一個，確定 started 狀態後，再開啟第二個，然看看看第一個有沒有連接成功的訊息
陳怡升EasonC13上午9:59
{"statusCode":404,"error":"Not Found","message":"Not Found"}
Sam Yang上午10:00
我可以
Tracy Tung上午10:00
1
楊德倫上午10:00
1
YenTing Chen上午10:00
1
林泳伸上午10:00
1
HW Chen上午10:00
1
王慕羣上午10:00
1
李居翰上午10:00
1
蔡仁耀上午10:00
1
Mark Weng上午10:00
1
Kuang Li上午10:00
1
Grace J上午10:00
1
Roberson Liou上午10:00
1
李居翰上午10:01
GET /_cat/nodes?v
王慕羣上午10:03
所以現在是下面這樣子嗎？

1. 用網址列輸入 http://127.0.0.1:9200/_cat/nodes?v 可以
2. 用 kibana 輸入 GET _cat/nodes?v 不行
你上午10:03
對
王慕羣上午10:04
XD
陳怡升EasonC13上午10:05
對
我再來試試看 homebrew 版本
楊德倫上午10:07
嗨 Eason，建議先用下載 tar 包的版本，包括 es 和 kibana，homebrew 先暫時不要使用，單純用 Terminal 來執行 es 跟 kibana 即可
陳怡升EasonC13上午10:07
[warning][environment] Detected an unhandled Promise rejection.
楊德倫上午10:08
mic
王慕羣上午10:08
joe 沒開 mic
Mark Weng上午10:08
老師你的聲音靜音了
陳怡升EasonC13上午10:09
  log   [10:06:54.189] [warning][config][plugins][security] Generating a random key for xpack.security.encryptionKey. To prevent sessions from being invalidated on restart, please set xpack.security.encryptionKey in the kibana.yml or use the bin/kibana-encryption-keys command.
  log   [10:06:54.190] [warning][config][plugins][security] Session cookies will be transmitted over insecure connections. This is not recommended.
  log   [10:06:54.233] [warning][config][plugins][reporting] Generating a 
Solaris T上午10:09
請問如果elasticsearch是cluster 那kibana需要cluster嗎？
王慕羣上午10:13
可能要改用 postman 或自己打 curl 了
要不然或是用 elasticsearch head
陳怡升EasonC13上午10:14
2
Yuen-Hsien Tseng上午10:14
2
李居翰上午10:15
GET /_cat/nodes?v
這樣吧@@
Huang Core上午10:17
老師需要切換回原本畫面喔
分享畫面被Eason切走哩
Sam Yang上午10:18
meet可以釘選
Uerica Wang上午10:18
老師的畫面是另一個帳號喔~
Huang Core上午10:18
喔喔
Solaris T上午10:19
老師的畫面沒有出來
Huang Core上午10:19
直接用釘選的 直接選擇老師畫面
楊德倫上午10:19
釘選老師的畫面 (ppt 那個)
Huang Core上午10:19
就不會受到Eason那邊畫面影響哩
Uerica Wang上午10:19
嗯嗯嗯，大家要取消目前畫面的釘選，或是直接找老師的畫面來釘選~
陳怡升EasonC13上午10:21
解決了，路徑問題
陳怡升EasonC13上午10:28
get _search 我找不到 movies
楊德倫上午10:28
先

PUT /movies/_doc/1
{
 "title": "Star Wars: Episode VII – The Force Awakens",
 "director": "J.J. Abrams",
 "year": 2015
}
陳怡升EasonC13上午10:28
有
我打 get /movies/_search 就有找到了
但 get _search 不行
楊德倫上午10:30
GET _cat/indices?v
Sam Yang上午10:32
所以所有的資料的type都是_doc嗎？
Huang Core上午10:36
那我想問一下說，那我們現在都是用_doc ，那萬一以後升級8.X的話 ， _doc 就被移除了，這樣東西不會掛掉嗎
楊德倫上午10:39
1
YenTing Chen上午10:39
1
Kuang Li上午10:39
1
Tracy Tung上午10:39
1
王慕羣上午10:39
1
李居翰上午10:39
1
Roberson Liou上午10:39
1
Huang Core上午10:39
1
Mark Weng上午10:39
1
林泳伸上午10:39
1
楊嘉榮上午10:39
1
陳包包上午10:39
1
Grace J上午10:39
1
Sam Yang上午10:39
1
吳信賢上午10:39
1
Yuen-Hsien Tseng上午10:39
2
蔡仁耀上午10:39
1
陳怡升EasonC13上午10:39
1
HW Chen上午10:39
1
Roberson Liou上午10:54
請問老師 ELK 預設的 query 就會是 ignore case 對嗎?
Huang Core上午10:59
老師想問一下 他的 query 他是以開頭去做查詢的嗎??

我現在Title資料有
Star Wars VII
Star Wars XII

我搜尋 II 或搜尋 Wars 都是沒查到 
Grayson上午11:00
想問一下再塞資料的時候，指定 _doc _id 跟不指定 _id 的效能會有差異嗎？
陳怡升EasonC13上午11:00
老師剛剛找的文件，關鍵字是下什麼？
Grayson上午11:02
了解
Mark Weng上午11:10
想問一下 query的時候用POST跟用GET是相同的對嗎?
好的謝謝
Grayson上午11:19
意思是說，es會根據 filter 查詢的條件去做「選擇性的cache」嗎？
陳怡升EasonC13上午11:20
想問一下 query的時候用POST跟用GET是相同的對嗎?,
好奇剛剛的回應
Solaris T上午11:20
所以這樣會先做filter再去做query嗎？?
Grayson上午11:20
根據頻率嗎？還是有部分的 rule ?
林泳伸上午11:23
不太懂為何 filter 操作要跟 bool query 做搭配？有點不太懂 bool query 代表的意思
王慕羣上午11:27
1
Tracy Tung上午11:27
1
Mark Weng上午11:27
1
李居翰上午11:27
1
陳包包上午11:27
1
Kuang Li上午11:27
1
陳怡升EasonC13上午11:27
1
Sam Yang上午11:27
1
YenTing Chen上午11:27
1
Roberson Liou上午11:27
1
Grace J上午11:27
1
蔡仁耀上午11:27
1
Grayson上午11:27
1
HW Chen上午11:27
1
林泳伸上午11:27
1
你上午11:27
=============
Roberson Liou上午11:27
發現使用  "genres": "Crime" 查詢 hit 是 0 筆
改用小寫  "genres": "Crime" 就 work 了
王慕羣上午11:27
1
Tracy Tung上午11:27
1
Roberson Liou上午11:28
1
陳怡升EasonC13上午11:28
真的欸
1
Solaris T上午11:39
請問哪如果又想可以搜尋"J.J. Abrams"又想可以只找"Abrams" 也是用這樣增加欄位的方式嗎？還是有別的設定方式？
王慕羣上午11:42
這邊的操作看不太懂 所以預設是會多存一個 keyword 子欄位 我不用另外再存 keyword 的意思？
陳怡升EasonC13上午11:42
title.keyword 跟 title.original 的差異？
喔喔...
HW Chen上午11:43
請問，根據剛剛所說的
王慕羣上午11:43
那之後還可以改 keyword 子欄位的 mapping 嗎？
HW Chen上午11:43
director.original 也不能改 type 了對嗎？
陳怡升EasonC13上午11:44
老師這個怎叫的
王慕羣上午11:46
想問一下 如果有幾千萬筆資料 然後想要改 mapping 的話 只能手動重新 update 才行嗎？
王慕羣上午11:49
如果想用原本的 director 欄位直接搜尋 director.keyword 欄位 有這種寫法嗎？
剛剛寫

"director.*": {
  "value": "J.J. Abrams"
}

是沒有作用的
陳怡升EasonC13上午11:52
請問 .original 是不是會讓資料庫空間變大？
Roberson Liou上午11:54
想請問在 intellisense 的部分的大小寫差別
(截圖已貼到 hackmd 最後面)
林泳伸上午11:54
有發現 ，下列查不到，但如果是用 abrams 卻查得到，差別在哪？
{
   "query": {
     "term": {
         "director": "j.j."
      }
    }
}
陳怡升EasonC13上午11:55
所以他拆分時就全部轉小寫了嗎
王慕羣上午11:55
回樓上 預設 analyzer 應該是會把 . 拿掉 明天說不定會講？
Roberson Liou上午11:56
j.j 查的到
林泳伸上午11:57
了解，所以這部分就跟 analyzer 的機制有關係，可能不是那麼單純以空格拆字
王慕羣上午11:57
1
Roberson Liou上午11:57
1
楊德倫上午11:57
1
林泳伸上午11:57
1
Mark Weng上午11:57
1
吳信賢上午11:57
1
陳包包上午11:57
1
Kuang Li上午11:57
1
Tracy Tung上午11:57
1
YenTing Chen上午11:57
1
Grace J上午11:57
1
李居翰上午11:57
1
Sam Yang上午11:57
1
蔡仁耀上午11:58
1
林泳伸下午12:01
想請問教學的錄影，會連 meet 的對話一起附上去嗎？
Huang Core下午12:05
想問說我這_mapping 一開始要先設計好，這是我 一開始在
PUT /movies/_doc/1
{
  "title":"Star Wars VII",
  "director":"J.J Abrams",
  "year":2015,
  "value": 123701728012013568,
  "Genres":["Action","Adventure","Fantasy"]
}

是在這裡面再增加設定嗎??
Sam Yang下午12:06
之後會解釋term, bool, match_query, query_string這些不同的query的差異嗎？
Huang Core下午12:08
另外想問說 這 dev Tools Console 上面有註解語法可以使用讓我再上面寫註解嗎?
好的謝謝
那大概下午是幾點上課呢
你下午12:09
下午1點開始上課
王慕羣下午12:09
謝謝老師
Huang Core下午12:09
好謝謝老師
Mark Weng下午12:09
好的謝謝老師
Roberson Liou下午12:09
謝謝老師~
林泳伸下午12:09
謝謝
Tracy Tung下午12:09
感謝
楊德倫下午12:10
感謝
林泳伸下午1:00
1
王慕羣下午1:00
1
陳怡升EasonC13下午1:00
1
Kuang Li下午1:00
1
楊德倫下午1:00
1
李居翰下午1:00
1
Roberson Liou下午1:00
1
Solaris T下午1:00
1
吳信賢下午1:00
1
蔡仁耀下午1:00
1
YenTing Chen下午1:00
1
Grace J下午1:01
 1
Tracy Tung下午1:01
1
Grayson下午1:21
inverted index 是可以被查到的嗎？
Mark Weng下午1:25
還在memory的時候如果關掉elasticsearch 資料會消失嗎?
陳怡升EasonC13下午1:29
所有 node 都有同樣的 segment 嗎？只是茶的時候會跑的不同？
Solaris T下午1:33
哪如果不設定replica 那所有的資料都會擠在同一台server嗎 ?
陳怡升EasonC13下午1:35
應該無論如何所有資料都會在同一台吧（？
只是查詢會分開（？
同一台都會有所有資料（？
咦，酷，我說錯了
Solaris T下午1:37
所以index的時候，elastic會自己把primary shard放到不同server上去?
Sam Yang下午1:38
為什麼會說replica可以提高查詢效能？
陳怡升EasonC13下午1:38
所以 a0 的 primary shard 只會有一個？其他都會是replica shard
Solaris T下午1:38
了解  謝謝
Sam Yang下午1:39
a跟b是兩個不同的Index?
Grace J下午1:39
請問replica shrd的寫入時間是在primary shard
陳怡升EasonC13下午1:39
所以 a0 的 primary shard 只會有一個？其他都會是replica shard，是嗎？
Grace J下午1:39
寫入segment之後嗎？
Sam Yang下午1:39
所以說只有一個index的話，replica並不會提高查詢效能吧？
陳怡升EasonC13下午1:40
不能對 replica shard 寫入囉？
大家來寫共筆QQ https://hackmd.io/YZNLmE4ZRs28v5XX5lplDg?edit
Sam Yang下午1:42
了解
陳怡升EasonC13下午1:45
那如果有四個 Shard 但是有八台機器，會不會有差？
假設每台都有四個 Shard 的 replicate
王慕羣下午1:45
後面的資料應該會在 master 組合後才回 client 資料吧
Solaris T下午1:46
似乎應該是八台機器 4個primary 4個replica
陳怡升EasonC13下午1:48
1. 單一請求
如果要提升，如果能把資料分散到多台機器的話
（只吃 primary Shard）
但如果分太多，合併起來會有 overhead
2. 有很大量查詢請求
能被善用 Cluster，因此每台 Node 都能處理每個 Shard 的搜尋
是這樣嗎？
李居翰下午1:50
如果資料量很大 資料可以放在NFS,NAS,S3等地方嗎?
陳怡升EasonC13下午1:51
這時候單一請求速度最快就是縮短成 1/2
但如果大量請求，速度可以縮短成 1/4 ？
HW Chen下午1:52
如果用這個 GoodNote 的圖來說明 index 可以怎麼理解？
陳怡升EasonC13下午1:52
在 NAS 上面用 docker 建一個 node (?
陳怡升EasonC13下午1:55
這時候如果我重開一個 Movies2 然後拆成 4 個 Shard，是不是一個好的優化策略？
然後把舊的資料讀過去
Sam Yang下午1:56
應該是看你同時查詢的負荷量多少吧
同時查詢量少的話就拆四個primary shard
Grayson下午1:57
分散後放在 share 的資料，是用固定的比例來放資料嗎？
Solaris T下午1:59
請問一份Document 會直接被分shard還是一份index下去分shard?
了解  謝謝
王慕羣下午2:03
想問一下就以單純的資料寫入查詢來說 loading 比較大的應該是 data node 比較重，master node 比較輕？
王慕羣下午2:05
可是後面在把各個 shard 拿回來合併資料時 master node 會不會又要花更多 loading？
李居翰下午2:20
請問有官方或老師有 建議的node不同角色 如何配置的架構圖嗎,各node角色需要的資源如何計算呢?
Sam Yang下午2:21
我以為是REST API是9200~9299，cluster nodes之間溝通是9300~9399 不是這樣嗎？
Sam Yang下午2:23
喔喔好 可能我聽錯了
HW Chen下午2:27
老師，想問一下實務可能剛開始資料量不大，後續可能客戶變多了，所以資料量逐漸增加，這個情況下不能事後更改 primary shard 數量，通常會怎麼處理比較合適？
楊德倫下午2:27
請問 Primary shards 在 indexing 之後不能修改的情況下，新增一台主機後，若要達到資料分散儲存的目標，是不是直接調整 replica 的數量就好? 以後增加很多主機，而當初建立的 Primary shards 數量一直維持同一個值，會有什麼影響嗎?
李居翰下午2:30
Data role 的 data_hot data_warm  data_warm....  hot  warm 是怎麼判定的? hot 要做什麼過程才會變 warm
HW Chen下午2:32
以剛剛的例子：
如果是用時間來切 index
但是 primary shard 數量還是跟原本一樣
這樣一個 primary shard 資料量就會不斷增加對嗎？

如果這樣的話會不會導致查詢效能低落
楊德倫下午2:33
所以若是不確定未來會增加幾台主機，可能先以目前主機數量的 1.5 或 2 倍 primary shards 的值先設定，待未來主機增加的時候，讓 elasticsearch 自動分配，請問可以這麼做嗎?
楊德倫下午2:36
原來如此，謝謝回答
Sam Yang下午2:41
雜音好像有點嚴重（？
王慕羣下午2:41
有雜音
Mark Weng下午2:41
還是有
Uerica Wang下午2:41
沒有~
王慕羣下午2:41
還是有
Uerica Wang下午2:41
還是有
陳怡升EasonC13下午2:41
原來不是我的問題XD
Uerica Wang下午2:42
Ok
Zonyo Tao下午2:42
ok
Mark Weng下午2:42
沒有了
林泳伸下午2:42
可以
王慕羣下午2:42
ok
Roberson Liou下午2:42
ok
Solaris T下午2:42
OK
王慕羣下午2:42
又有一點點了
Mark Weng下午2:43
我這邊還好
李居翰下午2:43
OK
林泳伸下午2:43
可以
王慕羣下午2:43
ok
Zonyo Tao下午2:43
沒問題
Sam Yang下午2:43
我這裡沒有
HW Chen下午2:54
用空間換時間的概念
Zonyo Tao下午2:54
請問 stored_fields 是一開始在 mapping 就要設定好嗎
王慕羣下午3:06
想問一下，這個 cluster 不是有兩個 node 嗎？關了一個 node，為什麼會出現 503 的問題？
HW Chen下午3:12
請問資料量大的時候會不會有資料注入速度的瓶頸，比如原本五分鐘內可以查到最新的資料，但是量變大後變成只能一小時過後才能看到最新的資料？
Grayson下午3:15
一秒是根據 refresh 的時間嗎？
你下午3:15
yes refresh
Tracy Tung下午3:18
理論上 update 比 index 耗能是嗎？
Solaris T下午3:22
請問get不是本來就會返回多筆資料，跟mget的差異是?
Solaris T下午3:23
喔喔 不好意思 跟query搞混了
陳怡升EasonC13下午3:34
server.maxPayload
改掉就可以了，去 config
Roberson Liou下午3:35
老師我在坐上一個 lab 的時候
在 create index 的時候有發生問題
原因應該是早上建立過 movies 的 index 了
不過用 _delete_by_query match_all api 把資料刪掉之後，用 get indices 去看發現原本的 index 還是在。
想請問要怎麼刪除原本的 index 呢？
HW Chen下午3:41
1
Kuang Li下午3:42
1
蔡仁耀下午3:42
1
李居翰下午3:42
1
Grace J下午3:42
1
Mark Weng下午3:42
1
Tracy Tung下午3:42
1
王慕羣下午3:42
1
Roberson Liou下午3:42
1
林泳伸下午3:42
1
Sam Yang下午3:42
1
楊德倫下午3:42
1
Yuen-Hsien Tseng下午3:42
1
陳怡升EasonC13下午3:45
所以 timeout 才會失敗嗎？
Sam Yang下午3:54
包在bool裡面的是代表，裡面的兩個條件要同時符合的資料才會算是hit嗎？
陳怡升EasonC13下午3:56
size 是回傳幾筆，還是看過幾筆？
陳怡升EasonC13下午4:02
search 雖然預設只回傳前面幾筆，但他還是會遍歷一次？因為有?track_total_hits=true
Sam Yang下午4:13
BM25
Uerica Wang下午4:15
1
Sam Yang下午4:15
profile裡面的那些breakdown分別代表的是elasticsearch在進行搜尋的各個步驟嗎
Solaris T下午4:15
1
Roberson Liou下午4:15
1
Steven Huang下午4:15
1
Tracy Tung下午4:15
1
Grace J下午4:15
1
陳包包下午4:15
1
吳信賢下午4:15
1
林泳伸下午4:15
1
Kuang Li下午4:15
1
Huang Core下午4:16
1
李居翰下午4:16
1
蔡仁耀下午4:16
1
Yuen-Hsien Tseng下午4:16
1
Sam Yang下午4:17
關於這每一層的細節還有怎麼去優化 之後會提到嗎
楊德倫下午4:43
硬體防火牆嗎?
HW Chen下午4:44
https://www.haproxy.com/
Zonyo Tao下午4:44
請問 indexing 這個名詞是包含以下兩個 還是單指其中某一個?
1. 新增一個 document
2. 新增一個 index
第二個問題
執行動詞只受限於新增, 還是包含增刪修?
Solaris T下午4:47
如果一下送進來爆量的index request的話 node會直接掛掉嗎？還是會先緩存?
Solaris T下午4:48
了解  謝謝
王慕羣下午4:54
replica 少了一份
王慕羣下午5:02
39?
Tracy Tung下午5:10
1
Solaris T下午5:10
1
林泳伸下午5:10
1
Zonyo Tao下午5:10
1
王慕羣下午5:10
1
李居翰下午5:10
1
Roberson Liou下午5:10
1
蔡仁耀下午5:10
1
Kuang Li下午5:10
1
陳怡升EasonC13下午5:10
1
Huang Core下午5:10
1
吳信賢下午5:10
1
Yuen-Hsien Tseng下午5:10
1
Grace J下午5:10
1
蔡仁耀下午5:27
請問 刪除 底層行為會跟修改 依樣嗎
Roberson Liou下午5:28
想請問老師 segment merge 觸發的條件為何?
官方文件是說會看 maximun number of thread
這段有點不太懂
https://www.elastic.co/guide/en/elasticsearch/reference/current/index-modules-merge.html
蔡仁耀下午5:28
了解!
Roberson Liou下午5:31
好的謝謝老師
蔡仁耀下午5:31
還有一個問題請教，DELETE 會標記後刪除，但是有重新新增發現 version 還是會往上加，所以不管甚麼行為都會記錄一次 version 嗎?
蔡仁耀下午5:33
了解
Uerica Wang下午5:35
今天感謝大家的課程參與~剛剛中間麥克風有雜音的部分很不好意思，我們會再檢查一下我們的收音設備，也感謝各位即時反應！若今日線上教學的過程中，還有其他需改進的部分，也請各位即時與我們反應，好讓我們明天的課程能調整的更好：），最後，今日也請各位早點休息，我們明日見~🤗🤗🤗
Roberson Liou下午5:35
謝謝老師~
蔡仁耀下午5:35
謝謝!
Tracy Tung下午5:35
謝謝
Huang Core下午5:35
謝謝老師
Sam Yang下午5:35
謝謝！
王慕羣下午5:35
很硬的課程 謝謝老師
Yuen-Hsien Tseng下午5:35
謝謝
Grace J下午5:35
謝謝老師
楊德倫下午5:35
謝謝
Zonyo Tao下午5:35
thanks
YenTing Chen下午5:36
謝謝
林泳伸下午5:36
謝謝～
Kuang Li下午5:36
謝謝
Steven Huang下午5:36
謝謝
```


## 上課 Google Meet 對話記錄 07-25

```
Zonyo Tao上午8:58
1
Tracy Tung上午8:59
1
HW Chen上午8:59
1
林泳伸上午8:59
1
李居翰上午8:59
1
蔡仁耀上午8:59
1
Grace J上午8:59
1
wang uerica上午8:59
1
Steven Huang上午8:59
1
Solaris T上午8:59
1
Huang Core上午8:59
1
Kuang Li上午8:59
1
楊嘉榮上午8:59
1
吳信賢上午8:59
1
Grayson上午8:59
1
Roberson Liou上午8:59
1
Yuen-Hsien Tseng上午8:59
1
Grayson上午9:03
stop words 可以依據 不同的 index or type 做過濾嗎？？
了解
Grayson上午9:06
中文的部分有同義字嗎？
陳怡升EasonC13上午9:07
這就是昨天說的 ETL pipeline 嗎
Grayson上午9:10
tokenizer 是依據空白切分，有辦法指定使用其他的分隔符號嗎？
ＯＫ
林泳伸上午9:20
pdf 上的連結是 404 
Sam Yang上午9:21
不能直接按
要全部copy
換行斷掉了
林泳伸上午9:22
了解，謝謝
Sam Yang上午9:32
n-gram跟standard tokenizer可以兩個都使用嗎？
Grayson上午9:36
stopwords 是可以再增加的嗎？
陳包包上午9:36
老師我想請問一下這個token filter，analtzer 是直接改變資料，還是只針對查詢
如果只針對查詢，這樣我資料大的時候，不是很花資源
Grayson上午9:37
這字典檔會需要 reload 嗎？
or restart 
王慕羣上午9:37
想問一下如何判斷何時該用 char filter 或是 token filter？

我上次的需求是把想把文字丟到 jieba analyzer 處理時，把空白跟標點符號去掉。但發現 jieba analyzer 沒有處理空白跟標點符號。

所以後來用 char_filter 的 pattern_replace，但後來我認為這個用法可能會有些效能問題，而且可能容易錯誤。

後來就改用 token filter 的 stop filter，然後在 stop word 裡面設定我想要去掉的內容，感覺這樣子可能比較符合需求。

還是說一般不太會用到 char filter？
陳怡升EasonC13上午9:39
POST _analyze
{
  "tokenizer": "keyword",
  "filter": ["standard", "lowercase", "stop"],
  "text": "the handsome Eason Chen 13"
}
ERROR: The [standard] token filter has been removed.
Grayson上午9:41
這樣不就會變成，如果有多個 node 都要依序 reload 
陳怡升EasonC13上午9:41
我的解決了
POST _analyze
{
  "tokenizer": "standard",
  "filter": ["lowercase", "stop"],
  "text": "the handsome Eason Chen 13"
}
Sam Yang上午9:44
兩個問題
1. 如何同時使用ngram以及沒有ngram的結果
2. 可以根據一群文件中的document frequency去設定一個門檻值做stop word嗎？
Grayson上午9:44
但實際場景會有大量的字要更新 @@
Sam Yang上午9:45
一個fields只能有一個analyzer嗎？
query也可以同時用兩個analyzer
?
Sam Yang上午9:50
了解
Tracy Tung上午9:50
搜尋時可以用 cutoff_frequency 去濾掉高頻詞
這個算是有達到 Sam Yang 提出的問題 2 嗎
Grayson上午9:50
字典檔
ik
了解，我會再試試
Sam Yang上午9:52
但是沒辦法在indexing的階段就過濾掉吧？
Solaris T上午9:52
去掉高頻詞不會跟tf-idf衝突嗎？
林泳伸上午9:53
1
Mark Weng上午9:53
1
Solaris T上午9:53
1
蔡仁耀上午9:53
1
吳信賢上午9:53
1
王慕羣上午9:53
1
Tracy Tung上午9:53
1
Kuang Li上午9:53
1
李居翰上午9:53
1
HW Chen上午9:53
1
Roberson Liou上午9:53
1
楊德倫上午9:53
1
Huang Core上午9:53
1
Grayson上午9:53
1
Grace J上午9:53
1
YenTing Chen上午9:53
1
Grayson上午10:05
剛剛講的 ik hot restart 是要去底層的 java 嗎？
林泳伸上午10:06
202 頁 Test the mapping 左邊，POST posts/_analyze 好像不能成功，還是要 POST posts/index/_analyze 才可以成功運作
王慕羣上午10:07
hot reload 字典檔好像是 es 的痛？因為我裝 jieba plugin 也是都要 restart 才行，一直到最近好像才有 hot reload 的功能
Tracy Tung上午10:08
不過 jieba 新版我自己實測 hot reload 還是失敗
王慕羣上午10:09
https://github.com/sing1ee/elasticsearch-jieba-plugin 樓上是用這套嗎？
Tracy Tung上午10:09
對的
林泳伸上午10:10
噢噢！了解沒問題了
陳怡升EasonC13上午10:10
PUT movies/
{
  "settings": {
    "index":{
      "analysis":{
        "analyzer": {
          "eason_analyzer": {
            "tokenizer": "uax_url_email",
            "filter": ["lowercase", "stop",     { "type": "edge_ngram",
                "min_gram": 3,
                "max_gram": 5
              }]
          }
        }
      }
    }
  }
}
Zonyo Tao上午10:12
想問一下前面 query 加 analyzer 的問題

在 query 裡面放 analyzer 是針對搜尋的文字做 analyzer 處理
不是針對被搜尋的 field 做 analyzer 對嗎
(因為 field 應該在 indexing 階段 就完成 text analysis 了)
ok thanks
Grace J上午10:13
我想請問針對不同語言會有不同的tokenizer，比如說中文就有很多分詞起可供挑選，因為中文是我們認識的語言，所以可以自行判斷出分詞結果是否為我們想要的。但是如果針對沒學過的語言，我就無法判斷分詞結果是否符合需求，請問網路上是否有各種語系的文章與斷詞結果可供驗證。或是老師可以提供其他驗正的思路以供參考，謝謝！
王慕羣上午10:13
回 grace 我是直接用 es 的內建 analyzer
他有各個語言的建議 analyzer
但我也不知道正不正確 XD
Grace J上午10:14
我目前也是用內建的
王慕羣上午10:16
ko openkoreantext
ja kuromoji
zh jieba

其他就內建的
王慕羣上午10:17
還有漢字...
Grace J上午10:19
謝謝老師和慕羣分享 乾蝦
Grayson上午10:19
老師如果有看到日文那篇的話，方便在分享嗎？
王慕羣上午10:20
https://www.elastic.co/blog/how-to-implement-japanese-full-text-search-in-elasticsearch 是這篇嗎？
Grayson上午10:23
所以通常 日文場景的資料跟中文的資料會放在同一個 index 內嗎？
畢竟他用了不同的 andalyzer ... etc 
王慕羣上午10:23
回 grayson 我放在不同欄位
Grayson上午10:24
謝謝～
Grayson上午10:42
pinyin 是注音符號的那種嗎？
Yuen-Hsien Tseng上午10:44
請問，ik 代表什麼？是什麼字頭詞嗎？
Grayson上午10:44
那有針對相同的音去搜尋的相關套件嗎？
例如 「藍球」 要搜尋到 「籃球」
楊德倫上午10:46
有 套件下載 的功能，會不會也有需要維護/更新 repositories 或 resource.list 的需要? 例如定期更新 套件下載來源 的清單。
楊德倫上午10:49
謝謝回覆
Sam Yang上午10:53
JavaEye：10. 为什么给这个项目起IK Analyzer 这个名字呢？

linliangyi2007：呵呵，这个问题问的好，我很喜欢Diablo，尤其是Diablo II，我玩暗黑7年了。暗黑中有个角色，野蛮人哦，它的终极套装就是“不朽之王Immortal King”，IK诞生的那一天，刚好是我打出一整套套装的那一天，于是就用这个套装的名字做纪念了，呵呵，感谢暴雪，感谢JavaEye，感谢CCAV。。。     听说java也是这么命名的，当时的设计人员正在喝java咖啡来着……
Grayson上午10:53
了解，所以其實不需要真正翻譯成 注音符號
Sam Yang上午10:53
https://blog.csdn.net/code52/article/details/7243400
Grayson上午10:54
got it
王慕羣上午10:58
老師的麥克風又有一點點雜音
Solaris T上午10:59
請問像這樣多的field使用不同analyzer，是index的時候要自己把一樣的內容塞到不同的field嗎？
Grayson上午10:59
老師有推薦哪邊可以下載的到比較完整的字典檔嗎（比較通用的那種）？
Yuen-Hsien Tseng上午11:00
可以把剛剛Devtool的命令copy到訊息來嗎？
Grayson上午11:01
了解
Roberson Liou上午11:08
想請問老師如果 cluster 的 node 很多的狀況
一台一台更新 plugin 會蠻耗時的
想請問除了自己寫 script 做自動化以外，ELK 有內部的機制去同步 cluster 的 plugin 嗎?
楊德倫上午11:11
請問 我在 conf 資料夾內，自訂 seg.dic，也更新了 IKAnalyzer.cfg.xml，是否要重啟 elasticsearch service ?
Solaris T上午11:11
index可以備份嗎？elk有export import的機制還是直接複製data目錄下的資料就可以
楊德倫上午11:12
懂了，謝謝您
Solaris T上午11:14
好的  謝謝
Grayson上午11:16
所以在資料量大的時候，頻繁的更新字典檔的情況下，是不是就要常做 reindex 的動作
王慕羣上午11:18
回 grayson，我的做法是在原資料庫將這次有增加的詞用 like 全部掃一次，然後加上 es_update = true，然後再把 es_update = true 的重新做 update
但好累...
Grayson上午11:19
好像有點大工程＠＠
王慕羣上午11:19
我是每小時做一次
Grayson上午11:19
我的資料量可能不允許ＸＤ
楊德倫上午11:20
看來要設定 維護期間，跟郵局凌晨時間維護系統一樣。
Grayson上午11:20
感謝建議～
王慕羣上午11:21
我是大概 3000 萬筆，不過因為要更新的內容一次可能幾百筆，每小時跑一次也還好
陳怡升EasonC13上午11:28
如果 string 放到 int 欄位呢
Sam Yang上午11:39
物件指的是什麼？
Solaris T上午11:40
json裡面有物件的意思是?會被index嗎?
王慕羣上午11:40
所以 object 可以用 fields 來表示嗎？
Solaris T上午11:41
那需要為物件內的field指定type跟analyzer嗎?
王慕羣上午11:46
為什麼 date 跟 title 的 result 是 array？
王慕羣上午11:48
對
上一種做法
主要是怕之後取資料回來在 parse 會有問題
王慕羣上午11:52
使用 copy_to 應該會有 analyzer 的問題？因為這三個欄位的 analyzer 可能會不一樣？
Solaris T下午12:15
只需要指定routing id嗎？那些node是屬於這個routing id是由elasticsearch自己去處理？
陳怡升EasonC13下午12:17
就是剛剛的 copyto 嗎
聽起來不是
Grayson下午12:17
join 的功能有版本限制嗎？
Solaris T下午12:21
所以只要給固定的Rountiing ID就會有一樣結果 到一樣的node
了解  謝謝
王慕羣下午12:22
所以是說 假設我有一群資料是固定日本人會搜尋 所以可以把這群資料的 routing id 設定為 country = jp 然後把 shard 放在 japan 這樣搜尋會比較快？
你下午12:25
13:20 回來上課
Tracy Tung下午1:20
1
陳怡升EasonC13下午1:20
ㄅ1
林泳伸下午1:20
1
wang uerica下午1:20
1
Kuang Li下午1:20
1
Grace J下午1:20
1
吳信賢下午1:20
1
李居翰下午1:20
1
楊德倫下午1:20
1
YenTing Chen下午1:20
1
Steven Huang下午1:20
1
楊嘉榮下午1:20
1
蔡仁耀下午1:20
1
Yuen-Hsien Tseng下午1:20
1
Roberson Liou下午1:21
1
陳怡升EasonC13下午1:28
這樣是不是 Quick go Eason 只有一個 Quick 這樣是會回傳，但是Quick go Eason Chen 雖然有 Quick 但只 Match 25% 就不回傳？
王慕羣下午1:31
所以 match phrase 有看 position？
Steven Huang下午1:55
請問 ES的 效能一般要怎麼做 benchmark，會使用 load testing 之類的工具嘛？
Steven Huang下午1:59
謝謝喬叔！
王慕羣下午1:59
想問一下 match phrase 的 position 一定要連續，但可以跳 token 嗎？比如

「蔡 小明 英文 考試 不 及格」

我輸入「考試及格」要如何用 match_phrase 找到這份 doc？
你下午2:00
https://www.elastic.co/guide/en/elasticsearch/reference/7.14/common-options.html#date-math
王慕羣下午2:05
https://www.elastic.co/guide/cn/elasticsearch/guide/current/slop.html 剛剛看到了這篇
王慕羣下午2:07
實務上應該如何決定 slop 要多少會比較好？又或是中英文應該有不同的 slop 設定？
陳怡升EasonC13下午2:10
1
Roberson Liou下午2:11
1
Tracy Tung下午2:11
1
蔡仁耀下午2:11
1
李居翰下午2:11
1
Kuang Li下午2:11
1
Grace J下午2:11
1
林泳伸下午2:11
1
吳信賢下午2:11
1
wang uerica下午2:11
1
YenTing Chen下午2:11
1
王慕羣下午2:22
所以聽起來是 segment file 被 merge 之後 cache 就 invalidate 了？
Solaris T下午2:58
兩個C
HW Chen下午2:59
Node3 自己的呢？
Grace J下午2:59
A 應該有14個, G有13個
Zonyo Tao下午3:07
sum_other_doc_count 是指 bucket size == 1
的所有 bucket 總和嗎?
Zonyo Tao下午3:09
ok 了解
Tracy Tung下午3:14
問一個不相關的，請問剛剛那個圖上的 master node 會建議是配置在哪裡？
Tracy Tung下午3:15
理解 感謝
Solaris T下午3:16
所以如果我有配置某些是Agg Node某些是Data Node 所以我在LB上要指定request只打到agg node嗎?
謝謝
陳包包下午3:19
老師 您的畫面是不是沒有卡住了@@
Tracy Tung下午3:21
可以自己 pin 簡報
王慕羣下午3:21
正常
Solaris T下午3:21
看的到
林泳伸下午3:21
正常
王慕羣下午3:27
想問一下關於欄位設計上的問題，我現在有多語系的資料，設計成下面這樣

name_zh, name_en, name_ja

今天聽到 fields 之後，原來可以用 fields 的方式處理，所以會變成這樣

name: {
    fields: {
        zh: "xxx",
        en: "xxx",
        ja: "xxx"
    }
}

那在搜尋實務上，這兩種設計在 query 時，哪一種會比較好？或是兩種的優缺點？
Yuen-Hsien Tseng下午3:30
這樣寫，可以：
"buckets_path": "movies_per_year>votes"
這樣寫，會導致 error：
"buckets_path": "movies_per_year > votes"
亦即，不可以有空格！
陳怡升EasonC13下午3:44
好像csv
楊德倫下午4:00
請問安裝套件時，有可能做到 pip install -r requirements.txt 這種把套件列表寫在文字檔案的安裝方式嗎? 一次安裝指定的所有套件。
目前是逐一安裝
楊德倫下午4:01
了解，用 shell script 之類的方法安裝
好，謝謝老師
Zonyo Tao下午4:01
ansible 是這樣拚嗎?
ok thx
蔡仁耀下午4:14
請問如果參與選舉的 nodes 都掛掉了，還在嘗試連線的node 還是依樣嘗試修復嗎?
蔡仁耀下午4:15
了解
謝謝
陳怡升EasonC13下午4:17
所以 Elastic 就算只設定 8G，還是會用掉剩下的記憶體（？
Sam Yang下午4:19
上限不是32GB嗎，為什麼會建議不超過26
Solaris T下午4:19
不會進行swap的意思是?
如果沒記憶體可以用了  不可以使用swap? 直接out of memory ?
Sam Yang下午4:35
兩個問題
1. JVM Heap 上限是32GB，為何簡報內建議為不超過26GB
2. 若是實際使用上常常會需要使用terms很多的query的話，有什麼優化上的建議嗎？
Solaris T下午4:36
請問好像比較沒看XPack的東西   Xpack大概在那些情況下需要購買?
Sam Yang下午4:37
單純query而已
Sam Yang下午4:38
有的
王慕羣下午4:38
保哥
Sam Yang下午4:39
什麼時候會出版w
王慕羣下午4:39
+10
你下午4:41
https://www.elastic.co/subscriptions
李居翰下午4:41
請問如果是公司內部使用Elastic系列產品,不是賣服務的話 在Elastic License 2.0與Elastic License 那些是要付費的範圍呢?@@
Solaris T下午4:41
了解  那我再看看  主要是看安全性的部份
可以在index上做安全性嗎
陳怡升EasonC13下午4:48
先前用 ELK，GROK 一直無法正確轉換日期，我想要 UTC+8，他給我算成 +0，當時沒找到解法
HW Chen下午4:50
如果 application log 在多台上
會建議用 logstash 去個台抓，還是透過 filebeats 主動打到 logstash
Mark Weng下午4:51
要加 date timezone的filter ，我之前是用這樣可以解決，給你參考
filter {
  date {
    match => ["@timestamp", "yyyy-MM-dd HH:mm:ss", "yyyy MM dd HH:mm:ss", "ISO8601"]
    target => "@timestamp"
    timezone => "Asia/Taipei"
  }

}
陳怡升EasonC13下午4:55
好，我來研究看看
王慕羣下午5:06
1. 想請問一下 elastic cloud 跟 aws managed es 哪種會比較推薦？

之前沒有用 aws managed es 是因為部分 plugin 好像安裝不上去，所以後來就在 ec2 上面 self hosting

---

2. 想問一下關於 query 的時候，是否可以用一些現有已經存在 index 裡面的 doc 來做搜尋。

舉例，我要搜尋 top_movies 的內容

GET /top_movies
{
    query:{
        title: "ANOTHER_INDEX[123].name"
    }
}
Tracy Tung下午5:06
不好意思，我有問個手機、手機殼的問題，
但是我表單給的範例不夠清楚，所以我列在了課程期待區，想請問這個例子是 ES 可以處理的嗎？
Grace J下午5:07
老師我在昨天您回覆的"期待學習到的內容"的第二點，有增加comment，再麻煩您解答，謝謝！
Grayson下午5:08
可以拉長回看時間嗎，兩天的影片回看需要一點時間....
好的
Solaris T下午5:09
老師有打算開進階課程嗎?
Grace J下午5:14
對 有join
Grace J下午5:16
了解 謝謝老師
陳怡升EasonC13下午5:23
好，謝謝老師
李居翰下午5:23
hackmd 筆記有時限嗎?
楊德倫下午5:23
若是未來有機會上進階課程，需要有什麼先備知識嗎?
王慕羣下午5:23
謝謝老師 %%%%%%%%%%%%%%%%
Tracy Tung下午5:24
感謝 
Grace J下午5:24
謝謝老師
蔡仁耀下午5:24
謝謝!
Roberson Liou下午5:24
謝謝老師~
Tracy Tung下午5:24
%%%%
李居翰下午5:24
謝謝老師
wang uerica下午5:24
謝謝大家的參與跟回饋~~
YenTing Chen下午5:24
謝謝
Steven Huang下午5:24
謝謝老師
Sam Yang下午5:24
謝謝老師
楊德倫下午5:24
好，謝謝 ^^
HW Chen下午5:24
謝謝老師
Grayson下午5:24
謝謝老師～
吳信賢下午5:24
謝謝
Solaris T下午5:24
謝謝
Yuen-Hsien Tseng下午5:24
感謝
Kuang Li下午5:25
謝謝老師
```