
# Домашнее задание №2
## Базовые возможности mongoDB

### Необходимо:
 - установить MongoDB одним из способов: ВМ, докер;
 - заполнить данными;
 - написать несколько запросов на выборку и обновление данных

### Решение:
1) **Скачивание образа mongo 4.4.17**
```
docker pull mongo:4.4.17
```
<details>
<summary>Output</summary><pre>
4.4.17: Pulling from library/mongo
eaead16dc43b: Pull complete
8a00eb9f68a0: Pull complete
f683956749c5: Pull complete
b33b2f05ea20: Pull complete
3a342bea915a: Pull complete
d03ea960fc93: Pull complete
2456308b9b0d: Pull complete
6c5246c2368c: Pull complete
2e81bc3f9929: Pull complete
Digest: sha256:5012ddc14de6e0959b09ff59cb144ede632d70aa117c3c0b4e4a6e99faf8569d
Status: Downloaded newer image for mongo:4.4.17
docker.io/library/mongo:4.4.17
</pre></details>

```
docker images
```
<details>
<summary>Output</summary><pre>
REPOSITORY                      TAG       IMAGE ID       CREATED        SIZE
mongo                           4.4.17    af86910c16da   3 weeks ago    438MB
</pre></details>

2) **Запуск контейнера**
```
mkdir -p ~/mongodata
docker run -it -v ~/mongodata:/data/db -p 27017:27017 --name mongodb -d mongo:4.4.17
```
<details>
<summary>Output</summary><pre>
16179b529e062fc8503c8b72bf8ab8d2c7f3e88d1592c5b9c3f0097a85773258
</pre></details>

```
docker ps
```
<details>
<summary>Output</summary><pre>
CONTAINER ID   IMAGE          COMMAND                  CREATED         STATUS         PORTS                                           NAMES
16179b529e06   mongo:4.4.17   "docker-entrypoint.s…"   8 seconds ago   Up 7 seconds   0.0.0.0:27017->27017/tcp, :::27017->27017/tcp   mongodb
</pre></details>


3) **Подключение и создание пользователя**
```
mongo -host localhost -port 27017
```
<details>
<summary>Output</summary><pre>
MongoDB shell version v3.6.8
connecting to: mongodb://localhost:27017/
Implicit session: session { "id" : UUID("1e04e7d3-fd1d-42cc-b357-cffc6d0917ae") }
MongoDB server version: 4.4.17
WARNING: shell and server versions do not match
Server has startup warnings: 
{"t":{"$date":"2022-11-16T06:15:30.707+00:00"},"s":"I",  "c":"STORAGE",  "id":22297,   "ctx":"initandlisten","msg":"Using the XFS filesystem is strongly recommended with the WiredTiger storage engine. See http://dochub.mongodb.org/core/prodnotes-filesystem","tags":["startupWarnings"]}
{"t":{"$date":"2022-11-16T06:15:31.431+00:00"},"s":"W",  "c":"CONTROL",  "id":22120,   "ctx":"initandlisten","msg":"Access control is not enabled for the database. Read and write access to data and configuration is unrestricted","tags":["startupWarnings"]}
</pre></details>

```
use hw02
```
<details>
<summary>Output</summary><pre>
switched to db hw02
</pre></details>

```
db.createUser({
    user: "user",
    pwd: "passme",
    roles: [
        {role: "readWrite", db: "hw02"}
    ],
    passwordDigestor: "server"
})
```
<details>
<summary>Output</summary><pre>
Successfully added user: {
	"user" : "user",
	"roles" : [
		{
			"role" : "readWrite",
			"db" : "hw02"
		}
	],
	"passwordDigestor" : "server"
}
</pre></details>

4) **Импорт тестовых данных**
```
curl -o ~/mongodata/customers.json https://raw.githubusercontent.com/spendmail/otus_nosql_hw/main/docs/02_mongo/mall_customers.json
mongoimport --host=localhost --port=27017 --type json -d hw02 -c customers ~/mongodata/customers.json -u user --jsonArray
```
<details>
<summary>Output</summary><pre>
2022-11-16T13:18:11.551+0700	connected to: localhost:27017
2022-11-16T13:18:11.565+0700	imported 200 documents
</pre></details>

5) **Запись данных**
```
mongo --username user --password --authenticationDatabase hw02 --host localhost --port 27017
use hw02
db.customers.insertOne({"customer_id": "0201", "genre": "Male", "age": 31, "annual_income": 132, "spending_score": 85});
```
<details>
<summary>Output</summary><pre>
{
	"acknowledged" : true,
	"insertedId" : ObjectId("637480be5def556a7c1e4ff7")
}
</pre></details>

```
db.customers.insertMany([
    {"customer_id": "0202", "genre": "Male", "age": 32, "annual_income": 133, "spending_score": 86},
    {"customer_id": "0203", "genre": "Male", "age": 33, "annual_income": 134, "spending_score": 87},
]);
```
<details>
<summary>Output</summary><pre>
{
	"acknowledged" : true,
	"insertedIds" : [
		ObjectId("637480cd5def556a7c1e4ff8"),
		ObjectId("637480cd5def556a7c1e4ff9")
	]
}
</pre></details>

6) **Чтение данных**
```
use hw02
db.customers.find({"customer_id": "0202"})
```
<details>
<summary>Output</summary><pre>
{ "_id" : ObjectId("637480cd5def556a7c1e4ff8"), "customer_id" : "0202", "genre" : "Male", "age" : 32, "annual_income" : 133, "spending_score" : 86 }
</pre></details>

7) **Обновление данных**
```
use hw02
db.customers.updateOne({"customer_id": "0201"}, {$set: {"genre": "Female"}})
```
<details>
<summary>Output</summary><pre>
{ "acknowledged" : true, "matchedCount" : 1, "modifiedCount" : 1 }
</pre></details>

```
db.customers.updateMany({"customer_id": "0202"}, {$set: {"genre": "Female"}})
```
<details>
<summary>Output</summary><pre>
{ "acknowledged" : true, "matchedCount" : 1, "modifiedCount" : 1 }
</pre></details>

8) **Группировка данных с подсчетом количества**
```
use hw02
db.customers.aggregate([
    {"$group" : {_id:"$genre", count:{$sum:1}}}
])
```
<details>
<summary>Output</summary><pre>
{ "_id" : "Male", "count" : 89 }
{ "_id" : "Female", "count" : 114 }
</pre></details>

9) **Создание индекса и поиск**
```
db.customers.explain().find({"customer_id": "0200"})
```
<details>
<summary>Output</summary><pre>
{
	"queryPlanner" : {
		"plannerVersion" : 1,
		"namespace" : "hw02.customers",
		"indexFilterSet" : false,
		"parsedQuery" : {
			"customer_id" : {
				"$eq" : "0200"
			}
		},
		"queryHash" : "F2460F4B",
		"planCacheKey" : "F2460F4B",
		"winningPlan" : {
			"stage" : "COLLSCAN",
			"filter" : {
				"customer_id" : {
					"$eq" : "0200"
				}
			},
			"direction" : "forward"
		},
		"rejectedPlans" : [ ]
	},
	"serverInfo" : {
		"host" : "80f67f4a0a11",
		"port" : 27017,
		"version" : "4.4.17",
		"gitVersion" : "85de0cc83f4dc64dbbac7fe028a4866228c1b5d1"
	},
	"ok" : 1
}
</pre></details>

```
db.customers.createIndex({"customer_id": 1}, {unique: true})
```
<details>
<summary>Output</summary><pre>
{
	"createdCollectionAutomatically" : false,
	"numIndexesBefore" : 1,
	"numIndexesAfter" : 2,
	"ok" : 1
}
</pre></details>

```
db.customers.explain().find({"customer_id": "0200"})
```
<details>
<summary>Output</summary><pre>
{
	"queryPlanner" : {
		"plannerVersion" : 1,
		"namespace" : "hw02.customers",
		"indexFilterSet" : false,
		"parsedQuery" : {
			"customer_id" : {
				"$eq" : "0200"
			}
		},
		"queryHash" : "F2460F4B",
		"planCacheKey" : "76087015",
		"winningPlan" : {
			"stage" : "FETCH",
			"inputStage" : {
				"stage" : "IXSCAN",
				"keyPattern" : {
					"customer_id" : 1
				},
				"indexName" : "customer_id_1",
				"isMultiKey" : false,
				"multiKeyPaths" : {
					"customer_id" : [ ]
				},
				"isUnique" : true,
				"isSparse" : false,
				"isPartial" : false,
				"indexVersion" : 2,
				"direction" : "forward",
				"indexBounds" : {
					"customer_id" : [
						"[\"0200\", \"0200\"]"
					]
				}
			}
		},
		"rejectedPlans" : [ ]
	},
	"serverInfo" : {
		"host" : "80f67f4a0a11",
		"port" : 27017,
		"version" : "4.4.17",
		"gitVersion" : "85de0cc83f4dc64dbbac7fe028a4866228c1b5d1"
	},
	"ok" : 1
}
</pre></details>

10) **Удаление данных**
```
use hw02
db.customers.deleteMany({customer_id : "0202"})
```
<details>
<summary>Output</summary><pre>
{ "acknowledged" : true, "deletedCount" : 1 }
</pre></details>

```
db.customers.drop()
db.customers.count()
```
<details>
<summary>Output</summary><pre>
true
0
</pre></details>

```
db.dropDatabase()
```
<details>
<summary>Output</summary><pre>
{ "dropped" : "hw02", "ok" : 1 }
</pre></details>

```
show dbs
```
<details>
<summary>Output</summary><pre>
admin   0.000GB
config  0.000GB
local   0.000GB
</pre></details>



