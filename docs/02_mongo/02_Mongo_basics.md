
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
<summary>Output</summary>
4.4.17: Pulling from library/mongo<br>
eaead16dc43b: Pull complete <br>
8a00eb9f68a0: Pull complete <br>
f683956749c5: Pull complete <br>
b33b2f05ea20: Pull complete <br>
3a342bea915a: Pull complete <br>
d03ea960fc93: Pull complete <br>
2456308b9b0d: Pull complete <br>
6c5246c2368c: Pull complete <br>
2e81bc3f9929: Pull complete <br>
Digest: sha256:5012ddc14de6e0959b09ff59cb144ede632d70aa117c3c0b4e4a6e99faf8569d<br>
Status: Downloaded newer image for mongo:4.4.17<br>
docker.io/library/mongo:4.4.17
</details>

```
docker images
```
<details>
<summary>Output</summary>
REPOSITORY                      TAG       IMAGE ID       CREATED        SIZE<br>
mongo                           4.4.17    af86910c16da   3 weeks ago    438MB<br>
</details>

2) **Запуск контейнера**
```
mkdir -p ~/mongodata
docker run -it -v ~/mongodata:/data/db -p 27017:27017 --name mongodb -d mongo:4.4.17
```
<details>
<summary>Output</summary>
16179b529e062fc8503c8b72bf8ab8d2c7f3e88d1592c5b9c3f0097a85773258
</details>

```
docker ps
```
<details>
<summary>Output</summary>
CONTAINER ID   IMAGE          COMMAND                  CREATED         STATUS         PORTS                                           NAMES<br>
16179b529e06   mongo:4.4.17   "docker-entrypoint.s…"   8 seconds ago   Up 7 seconds   0.0.0.0:27017->27017/tcp, :::27017->27017/tcp   mongodb<br>
</details>


3) **Подключение и создание пользователя**
```
mongo -host localhost -port 27017
```
<details>
<summary>Output</summary>
MongoDB shell version v3.6.8<br>
connecting to: mongodb://localhost:27017/<br>
Implicit session: session { "id" : UUID("1e04e7d3-fd1d-42cc-b357-cffc6d0917ae") }<br>
MongoDB server version: 4.4.17<br>
WARNING: shell and server versions do not match<br>
Server has startup warnings: <br>
{"t":{"$date":"2022-11-16T06:15:30.707+00:00"},"s":"I",  "c":"STORAGE",  "id":22297,   "ctx":"initandlisten","msg":"Using the XFS filesystem is strongly recommended with the WiredTiger storage engine. See http://dochub.mongodb.org/core/prodnotes-filesystem","tags":["startupWarnings"]}<br>
{"t":{"$date":"2022-11-16T06:15:31.431+00:00"},"s":"W",  "c":"CONTROL",  "id":22120,   "ctx":"initandlisten","msg":"Access control is not enabled for the database. Read and write access to data and configuration is unrestricted","tags":["startupWarnings"]}
</details>

```
use hw02
```
<details>
<summary>Output</summary>
switched to db hw02
</details>

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
<summary>Output</summary>
Successfully added user: {<br>
	"user" : "user",<br>
	"roles" : [<br>
		{<br>
			"role" : "readWrite",<br>
			"db" : "hw02"<br>
		}<br>
	],<br>
	"passwordDigestor" : "server"<br>
}<br>
</details>

4) **Импорт тестовых данных**
```
curl -o ~/mongodata/customers.json https://raw.githubusercontent.com/spendmail/otus_nosql_hw/main/docs/02_mongo/mall_customers.json
mongoimport --host=localhost --port=27017 --type json -d hw02 -c customers ~/mongodata/customers.json -u user --jsonArray
```
<details>
<summary>Output</summary>
2022-11-16T13:18:11.551+0700	connected to: localhost:27017<br>
2022-11-16T13:18:11.565+0700	imported 200 documents
</details>

5) **Запись данных**
```
mongo --username user --password --authenticationDatabase hw02 --host localhost --port 27017
use hw02
db.customers.insertOne({"customer_id": "0201", "genre": "Male", "age": 31, "annual_income": 132, "spending_score": 85});
```
<details>
<summary>Output</summary>
{<br>
	"acknowledged" : true,<br>
	"insertedId" : ObjectId("637480be5def556a7c1e4ff7")<br>
}
</details>

```
db.customers.insertMany([
    {"customer_id": "0202", "genre": "Male", "age": 32, "annual_income": 133, "spending_score": 86},
    {"customer_id": "0203", "genre": "Male", "age": 33, "annual_income": 134, "spending_score": 87},
]);
```
<details>
<summary>Output</summary>
{<br>
	"acknowledged" : true,<br>
	"insertedIds" : [<br>
		ObjectId("637480cd5def556a7c1e4ff8"),<br>
		ObjectId("637480cd5def556a7c1e4ff9")<br>
	]<br>
}
</details>

6) **Чтение данных**
```
use hw02
db.customers.find({"customer_id": "0202"})
```
<details>
<summary>Output</summary>
{ "_id" : ObjectId("637480cd5def556a7c1e4ff8"), "customer_id" : "0202", "genre" : "Male", "age" : 32, "annual_income" : 133, "spending_score" : 86 }
</details>

7) **Обновление данных**
```
use hw02
db.customers.updateOne({"customer_id": "0201"}, {$set: {"genre": "Female"}})
```
<details>
<summary>Output</summary>
{ "acknowledged" : true, "matchedCount" : 1, "modifiedCount" : 1 }
</details>

```
db.customers.updateMany({"customer_id": "0202"}, {$set: {"genre": "Female"}})
```
<details>
<summary>Output</summary>
{ "acknowledged" : true, "matchedCount" : 1, "modifiedCount" : 1 }
</details>

8) **Группировка данных с подсчетом количества**
```
use hw02
db.customers.aggregate([
    {"$group" : {_id:"$genre", count:{$sum:1}}}
])
```
<details>
<summary>Output</summary>
{ "_id" : "Male", "count" : 89 }<br>
{ "_id" : "Female", "count" : 114 }<br>
</details>

9) **Создание индекса и поиск**
```
db.customers.explain().find({"customer_id": "0200"})
```
<details>
<summary>Output</summary>
{<br>
	"queryPlanner" : {<br>
		"plannerVersion" : 1,<br>
		"namespace" : "hw02.customers",<br>
		"indexFilterSet" : false,<br>
		"parsedQuery" : {<br>
			"customer_id" : {<br>
				"$eq" : "0200"<br>
			}<br>
		},<br>
		"queryHash" : "F2460F4B",<br>
		"planCacheKey" : "F2460F4B",<br>
		"winningPlan" : {<br>
			"stage" : "COLLSCAN",<br>
			"filter" : {<br>
				"customer_id" : {<br>
					"$eq" : "0200"<br>
				}<br>
			},<br>
			"direction" : "forward"<br>
		},<br>
		"rejectedPlans" : [ ]<br>
	},<br>
	"serverInfo" : {<br>
		"host" : "80f67f4a0a11",<br>
		"port" : 27017,<br>
		"version" : "4.4.17",<br>
		"gitVersion" : "85de0cc83f4dc64dbbac7fe028a4866228c1b5d1"<br>
	},<br>
	"ok" : 1<br>
}<br>
</details>

```
db.customers.createIndex({"customer_id": 1}, {unique: true})
```
<details>
<summary>Output</summary>
{<br>
	"createdCollectionAutomatically" : false,<br>
	"numIndexesBefore" : 1,<br>
	"numIndexesAfter" : 2,<br>
	"ok" : 1<br>
}<br>
</details>

```
db.customers.explain().find({"customer_id": "0200"})
```
<details>
<summary>Output</summary>
{<br>
	"queryPlanner" : {<br>
		"plannerVersion" : 1,<br>
		"namespace" : "hw02.customers",<br>
		"indexFilterSet" : false,<br>
		"parsedQuery" : {<br>
			"customer_id" : {<br>
				"$eq" : "0200"<br>
			}<br>
		},<br>
		"queryHash" : "F2460F4B",<br>
		"planCacheKey" : "76087015",<br>
		"winningPlan" : {<br>
			"stage" : "FETCH",<br>
			"inputStage" : {<br>
				"stage" : "IXSCAN",<br>
				"keyPattern" : {<br>
					"customer_id" : 1<br>
				},<br>
				"indexName" : "customer_id_1",<br>
				"isMultiKey" : false,<br>
				"multiKeyPaths" : {<br>
					"customer_id" : [ ]<br>
				},<br>
				"isUnique" : true,<br>
				"isSparse" : false,<br>
				"isPartial" : false,<br>
				"indexVersion" : 2,<br>
				"direction" : "forward",<br>
				"indexBounds" : {<br>
					"customer_id" : [<br>
						"[\"0200\", \"0200\"]"<br>
					]<br>
				}<br>
			}<br>
		},<br>
		"rejectedPlans" : [ ]<br>
	},<br>
	"serverInfo" : {<br>
		"host" : "80f67f4a0a11",<br>
		"port" : 27017,<br>
		"version" : "4.4.17",<br>
		"gitVersion" : "85de0cc83f4dc64dbbac7fe028a4866228c1b5d1"<br>
	},<br>
	"ok" : 1<br>
}<br>
</details>

10) **Удаление данных**
```
use hw02
db.customers.deleteMany({customer_id : "0202"})
```
<details>
<summary>Output</summary>
{ "acknowledged" : true, "deletedCount" : 1 }
</details>

```
db.customers.drop()
db.customers.count()
```
<details>
<summary>Output</summary>
true<br>
0<br>
</details>

```
db.dropDatabase()
```
<details>
<summary>Output</summary>
{ "dropped" : "hw02", "ok" : 1 }
</details>

```
show dbs
```
<details>
<summary>Output</summary>
admin   0.000GB<br>
config  0.000GB<br>
local   0.000GB<br>
</details>



