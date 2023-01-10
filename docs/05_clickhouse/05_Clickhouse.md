
# Домашнее задание №5
## ClickHouse

### Необходимо:
1. Развернуть инстанс;
2. Выполнить импорт тестовой БД;
3. Выполнить несколько запросов и оценить скорость выполнения;

### Подготовка:

0.1) **Установка clickhouse-клиента**
```
sudo apt update
sudo apt install -y clickhouse-client
```



### Решение:



1.1) **Запуск контейнера**
```
docker-compose up -d
```
<details>
<summary>Output</summary><pre>
Creating network "05_clickhouse_default" with the default driver
Pulling clickhouse (yandex/clickhouse-server:21.3.20-alpine)...
21.3.20-alpine: Pulling from yandex/clickhouse-server
59bf1c3509f3: Pull complete
4aecfe00e445: Pull complete
a5cca1455543: Pull complete
Digest: sha256:b434983027bdddfb8c9ff20b2cdc5acbe1ad443a7bb382b221c29cdd08d44e40
Status: Downloaded newer image for yandex/clickhouse-server:21.3.20-alpine
Creating clickhouse ... done
</pre></details>




2.1) **Скачивание тестовых данных**
```
curl https://datasets.clickhouse.com/hits/tsv/hits_v1.tsv.xz | unxz --threads=`nproc` > hits_v1.tsv
```
<details>
<summary>Output</summary><pre>
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  802M  100  802M    0     0  10.0M      0  0:01:19  0:01:19 --:--:-- 10.1M
</pre></details>



```
curl https://datasets.clickhouse.com/visits/tsv/visits_v1.tsv.xz | unxz --threads=`nproc` > visits_v1.tsv
```
<details>
<summary>Output</summary><pre>
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  405M  100  405M    0     0  10.0M      0  0:00:40  0:00:40 --:--:-- 10.4M
</pre></details>



2.2) **Создание базы данных**
```
clickhouse-client --port 19000 --query "CREATE DATABASE IF NOT EXISTS tutorial"
```




2.3) **Создание таблиц**
```
clickhouse-client --port 19000 <<- "QUERY"
CREATE TABLE IF NOT EXISTS tutorial.hits_v1
(
`WatchID` UInt64,
`JavaEnable` UInt8,
`Title` String,
`GoodEvent` Int16,
`EventTime` DateTime,
`EventDate` Date,
`CounterID` UInt32,
`ClientIP` UInt32,
`ClientIP6` FixedString(16),
`RegionID` UInt32,
`UserID` UInt64,
`CounterClass` Int8,
`OS` UInt8,
`UserAgent` UInt8,
`URL` String,
`Referer` String,
`URLDomain` String,
`RefererDomain` String,
`Refresh` UInt8,
`IsRobot` UInt8,
`RefererCategories` Array(UInt16),
`URLCategories` Array(UInt16),
`URLRegions` Array(UInt32),
`RefererRegions` Array(UInt32),
`ResolutionWidth` UInt16,
`ResolutionHeight` UInt16,
`ResolutionDepth` UInt8,
`FlashMajor` UInt8,
`FlashMinor` UInt8,
`FlashMinor2` String,
`NetMajor` UInt8,
`NetMinor` UInt8,
`UserAgentMajor` UInt16,
`UserAgentMinor` FixedString(2),
`CookieEnable` UInt8,
`JavascriptEnable` UInt8,
`IsMobile` UInt8,
`MobilePhone` UInt8,
`MobilePhoneModel` String,
`Params` String,
`IPNetworkID` UInt32,
`TraficSourceID` Int8,
`SearchEngineID` UInt16,
`SearchPhrase` String,
`AdvEngineID` UInt8,
`IsArtifical` UInt8,
`WindowClientWidth` UInt16,
`WindowClientHeight` UInt16,
`ClientTimeZone` Int16,
`ClientEventTime` DateTime,
`SilverlightVersion1` UInt8,
`SilverlightVersion2` UInt8,
`SilverlightVersion3` UInt32,
`SilverlightVersion4` UInt16,
`PageCharset` String,
`CodeVersion` UInt32,
`IsLink` UInt8,
`IsDownload` UInt8,
`IsNotBounce` UInt8,
`FUniqID` UInt64,
`HID` UInt32,
`IsOldCounter` UInt8,
`IsEvent` UInt8,
`IsParameter` UInt8,
`DontCountHits` UInt8,
`WithHash` UInt8,
`HitColor` FixedString(1),
`UTCEventTime` DateTime,
`Age` UInt8,
`Sex` UInt8,
`Income` UInt8,
`Interests` UInt16,
`Robotness` UInt8,
`GeneralInterests` Array(UInt16),
`RemoteIP` UInt32,
`RemoteIP6` FixedString(16),
`WindowName` Int32,
`OpenerName` Int32,
`HistoryLength` Int16,
`BrowserLanguage` FixedString(2),
`BrowserCountry` FixedString(2),
`SocialNetwork` String,
`SocialAction` String,
`HTTPError` UInt16,
`SendTiming` Int32,
`DNSTiming` Int32,
`ConnectTiming` Int32,
`ResponseStartTiming` Int32,
`ResponseEndTiming` Int32,
`FetchTiming` Int32,
`RedirectTiming` Int32,
`DOMInteractiveTiming` Int32,
`DOMContentLoadedTiming` Int32,
`DOMCompleteTiming` Int32,
`LoadEventStartTiming` Int32,
`LoadEventEndTiming` Int32,
`NSToDOMContentLoadedTiming` Int32,
`FirstPaintTiming` Int32,
`RedirectCount` Int8,
`SocialSourceNetworkID` UInt8,
`SocialSourcePage` String,
`ParamPrice` Int64,
`ParamOrderID` String,
`ParamCurrency` FixedString(3),
`ParamCurrencyID` UInt16,
`GoalsReached` Array(UInt32),
`OpenstatServiceName` String,
`OpenstatCampaignID` String,
`OpenstatAdID` String,
`OpenstatSourceID` String,
`UTMSource` String,
`UTMMedium` String,
`UTMCampaign` String,
`UTMContent` String,
`UTMTerm` String,
`FromTag` String,
`HasGCLID` UInt8,
`RefererHash` UInt64,
`URLHash` UInt64,
`CLID` UInt32,
`YCLID` UInt64,
`ShareService` String,
`ShareURL` String,
`ShareTitle` String,
`ParsedParams` Nested(
Key1 String,
Key2 String,
Key3 String,
Key4 String,
Key5 String,
ValueDouble Float64),
`IslandID` FixedString(16),
`RequestNum` UInt32,
`RequestTry` UInt8
)
ENGINE = MergeTree()
PARTITION BY toYYYYMM(EventDate)
ORDER BY (CounterID, EventDate, intHash32(UserID))
SAMPLE BY intHash32(UserID)
QUERY
```



```
clickhouse-client --port 19000 <<- "QUERY"
CREATE TABLE IF NOT EXISTS tutorial.visits_v1
(
`CounterID` UInt32,
`StartDate` Date,
`Sign` Int8,
`IsNew` UInt8,
`VisitID` UInt64,
`UserID` UInt64,
`StartTime` DateTime,
`Duration` UInt32,
`UTCStartTime` DateTime,
`PageViews` Int32,
`Hits` Int32,
`IsBounce` UInt8,
`Referer` String,
`StartURL` String,
`RefererDomain` String,
`StartURLDomain` String,
`EndURL` String,
`LinkURL` String,
`IsDownload` UInt8,
`TraficSourceID` Int8,
`SearchEngineID` UInt16,
`SearchPhrase` String,
`AdvEngineID` UInt8,
`PlaceID` Int32,
`RefererCategories` Array(UInt16),
`URLCategories` Array(UInt16),
`URLRegions` Array(UInt32),
`RefererRegions` Array(UInt32),
`IsYandex` UInt8,
`GoalReachesDepth` Int32,
`GoalReachesURL` Int32,
`GoalReachesAny` Int32,
`SocialSourceNetworkID` UInt8,
`SocialSourcePage` String,
`MobilePhoneModel` String,
`ClientEventTime` DateTime,
`RegionID` UInt32,
`ClientIP` UInt32,
`ClientIP6` FixedString(16),
`RemoteIP` UInt32,
`RemoteIP6` FixedString(16),
`IPNetworkID` UInt32,
`SilverlightVersion3` UInt32,
`CodeVersion` UInt32,
`ResolutionWidth` UInt16,
`ResolutionHeight` UInt16,
`UserAgentMajor` UInt16,
`UserAgentMinor` UInt16,
`WindowClientWidth` UInt16,
`WindowClientHeight` UInt16,
`SilverlightVersion2` UInt8,
`SilverlightVersion4` UInt16,
`FlashVersion3` UInt16,
`FlashVersion4` UInt16,
`ClientTimeZone` Int16,
`OS` UInt8,
`UserAgent` UInt8,
`ResolutionDepth` UInt8,
`FlashMajor` UInt8,
`FlashMinor` UInt8,
`NetMajor` UInt8,
`NetMinor` UInt8,
`MobilePhone` UInt8,
`SilverlightVersion1` UInt8,
`Age` UInt8,
`Sex` UInt8,
`Income` UInt8,
`JavaEnable` UInt8,
`CookieEnable` UInt8,
`JavascriptEnable` UInt8,
`IsMobile` UInt8,
`BrowserLanguage` UInt16,
`BrowserCountry` UInt16,
`Interests` UInt16,
`Robotness` UInt8,
`GeneralInterests` Array(UInt16),
`Params` Array(String),
`Goals` Nested(
ID UInt32,
Serial UInt32,
EventTime DateTime,
Price Int64,
OrderID String,
CurrencyID UInt32),
`WatchIDs` Array(UInt64),
`ParamSumPrice` Int64,
`ParamCurrency` FixedString(3),
`ParamCurrencyID` UInt16,
`ClickLogID` UInt64,
`ClickEventID` Int32,
`ClickGoodEvent` Int32,
`ClickEventTime` DateTime,
`ClickPriorityID` Int32,
`ClickPhraseID` Int32,
`ClickPageID` Int32,
`ClickPlaceID` Int32,
`ClickTypeID` Int32,
`ClickResourceID` Int32,
`ClickCost` UInt32,
`ClickClientIP` UInt32,
`ClickDomainID` UInt32,
`ClickURL` String,
`ClickAttempt` UInt8,
`ClickOrderID` UInt32,
`ClickBannerID` UInt32,
`ClickMarketCategoryID` UInt32,
`ClickMarketPP` UInt32,
`ClickMarketCategoryName` String,
`ClickMarketPPName` String,
`ClickAWAPSCampaignName` String,
`ClickPageName` String,
`ClickTargetType` UInt16,
`ClickTargetPhraseID` UInt64,
`ClickContextType` UInt8,
`ClickSelectType` Int8,
`ClickOptions` String,
`ClickGroupBannerID` Int32,
`OpenstatServiceName` String,
`OpenstatCampaignID` String,
`OpenstatAdID` String,
`OpenstatSourceID` String,
`UTMSource` String,
`UTMMedium` String,
`UTMCampaign` String,
`UTMContent` String,
`UTMTerm` String,
`FromTag` String,
`HasGCLID` UInt8,
`FirstVisit` DateTime,
`PredLastVisit` Date,
`LastVisit` Date,
`TotalVisits` UInt32,
`TraficSource` Nested(
ID Int8,
SearchEngineID UInt16,
AdvEngineID UInt8,
PlaceID UInt16,
SocialSourceNetworkID UInt8,
Domain String,
SearchPhrase String,
SocialSourcePage String),
`Attendance` FixedString(16),
`CLID` UInt32,
`YCLID` UInt64,
`NormalizedRefererHash` UInt64,
`SearchPhraseHash` UInt64,
`RefererDomainHash` UInt64,
`NormalizedStartURLHash` UInt64,
`StartURLDomainHash` UInt64,
`NormalizedEndURLHash` UInt64,
`TopLevelDomain` UInt64,
`URLScheme` UInt64,
`OpenstatServiceNameHash` UInt64,
`OpenstatCampaignIDHash` UInt64,
`OpenstatAdIDHash` UInt64,
`OpenstatSourceIDHash` UInt64,
`UTMSourceHash` UInt64,
`UTMMediumHash` UInt64,
`UTMCampaignHash` UInt64,
`UTMContentHash` UInt64,
`UTMTermHash` UInt64,
`FromHash` UInt64,
`WebVisorEnabled` UInt8,
`WebVisorActivity` UInt32,
`ParsedParams` Nested(
Key1 String,
Key2 String,
Key3 String,
Key4 String,
Key5 String,
ValueDouble Float64),
`Market` Nested(
Type UInt8,
GoalID UInt32,
OrderID String,
OrderPrice Int64,
PP UInt32,
DirectPlaceID UInt32,
DirectOrderID UInt32,
DirectBannerID UInt32,
GoodID String,
GoodName String,
GoodQuantity Int32,
GoodPrice Int64),
`IslandID` FixedString(16)
)
ENGINE = CollapsingMergeTree(Sign)
PARTITION BY toYYYYMM(StartDate)
ORDER BY (CounterID, StartDate, intHash32(UserID), VisitID)
SAMPLE BY intHash32(UserID)
QUERY
```




2.4) **Вставка данных**
```
time clickhouse-client --port 19000 --query "INSERT INTO tutorial.hits_v1 FORMAT TSV" --max_insert_block_size=100000 < hits_v1.tsv
```
<details>
<summary>Output</summary><pre>
real	0m22,603s
user	0m19,963s
sys	0m3,401s
</pre></details>



```
time clickhouse-client --port 19000 --query "INSERT INTO tutorial.visits_v1 FORMAT TSV" --max_insert_block_size=100000 < visits_v1.tsv
```
<details>
<summary>Output</summary><pre>
real	0m12,306s
user	0m6,331s
sys	0m1,268s
</pre></details>



2.5) **Оптимизация таблиц**
```
time clickhouse-client --port 19000 --query "OPTIMIZE TABLE tutorial.hits_v1 FINAL"
```
<details>
<summary>Output</summary><pre>
real	0m15,912s
user	0m0,034s
sys	0m0,019s
</pre></details>



```
time clickhouse-client --port 19000 --query "OPTIMIZE TABLE tutorial.visits_v1 FINAL"
```
<details>
<summary>Output</summary><pre>
real	0m5,780s
user	0m0,025s
sys	0m0,014s
</pre></details>



3.1) **Запрос количества записей**
```
time clickhouse-client --port 19000 --query "SELECT COUNT(*) FROM tutorial.hits_v1"
```
<details>
<summary>Output</summary><pre>
8873898

real	0m0,073s
user	0m0,033s
sys	0m0,010s
</pre></details>


```
time clickhouse-client --port 19000 --query "SELECT COUNT(*) FROM tutorial.visits_v1"
```
<details>
<summary>Output</summary><pre>
1676861

real	0m0,067s
user	0m0,034s
sys	0m0,006s
</pre></details>



3.2) **Запрос StartURL c группировкой по Duration**
```
time clickhouse-client --port 19000 <<- "QUERY"
SELECT
StartURL AS URL,
AVG(Duration) AS AvgDuration
FROM tutorial.visits_v1
WHERE StartDate BETWEEN '2014-03-23' AND '2014-03-30'
GROUP BY URL
ORDER BY AvgDuration DESC
LIMIT 10
QUERY
```
<details>
<summary>Output</summary><pre>
http://itpalanija-pri-patrivative=0&ads_app_user	60127
http://renaul-myd-ukraine	58938
http://karta/Futbol/dynamo.kiev.ua/kawaica.su/648	56538
https://moda/vyikroforum1/top.ru/moscow/delo-product/trend_sms/multitryaset/news/2014/03/201000	55218
http://e.mail=on&default?abid=2061&scd=yes&option?r=city_inter.com/menu&site-zaferio.ru/c/m.ensor.net/ru/login=false&orderStage.php?Brandidatamalystyle/20Mar2014%2F007%2F94dc8d2e06e56ed56bbdd	51378
http://karta/Futbol/dynas.com/haberler.ru/messages.yandsearchives/494503_lte_13800200319	49078
http://xmusic/vstreatings of speeds	36925
http://news.ru/yandex.ru/api.php&api=http://toberria.ru/aphorizana	36902
http://bashmelnykh-metode.net/video/#!/video/emberkas.ru/detskij-yazi.com/iframe/default.aspx?id=760928&noreask=1&source	34323
http://censonhaber/547-popalientLog=0&strizhki-petro%3D&comeback=search?lr=213&text	31773

real	0m0,114s
user	0m0,024s
sys	0m0,013s
</pre></details>





3.3) **Запрос суммы посещений с агрегацией с условием**
```
time clickhouse-client --port 19000 <<- "QUERY"
SELECT
sum(Sign) AS visits,
sumIf(Sign, has(Goals.ID, 1105530)) AS goal_visits,
(100. * goal_visits) / visits AS goal_percent
FROM tutorial.visits_v1
WHERE (CounterID = 912887) AND (toYYYYMM(StartDate) = 201403) AND (domain(StartURL) = 'yandex.ru')
QUERY
```
<details>
<summary>Output</summary><pre>
10543	8553	81.12491700654462

real	0m0,085s
user	0m0,030s
sys	0m0,010s
</pre></details>



3.4) **Done**
```
docker-compose down -v
```
<details>
<summary>Output</summary><pre>
Stopping clickhouse ... done
Removing clickhouse ... done
Removing network 05_clickhouse_default
</pre></details>
