
# Домашнее задание №3
## Кластерные возможности mongodb

### Необходимо:
1. Построить шардированный кластер из 3 кластерных нод (по 3 инстанса с репликацией) и с кластером конфига (3 инстанса).
2. Добавить балансировку, нагрузить данными, выбрать хороший ключ шардирования, посмотреть как данные перебалансируются между шардами.
3. Поронять разные инстансы, посмотреть, что будет происходить, поднять обратно. Описать что произошло.
4. Настроить аутентификацию и многоролевой доступ.

### Решение:





0) **Запуск контейнеров**
```
docker-compose up -d
```
<details>
<summary>Output</summary><pre>
Creating network "03_mongo_cluster_mongo_cluster_network" with driver "bridge"
Creating shard_2_server_2 ... done
Creating shard_3_server_2 ... done
Creating shard_1_server_3 ... done
Creating balancer_1       ... done
Creating shard_2_server_3 ... done
Creating shard_3_server_3 ... done
Creating shard_1_server_2 ... done
Creating balancer_2       ... done
Creating shard_3_server_1 ... done
Creating shard_1_server_1 ... done
Creating shard_2_server_1 ... done
Creating config_server_1  ... done
Creating config_server_2  ... done
Creating config_server_3  ... done
</pre></details>





1.1) **Создаем репликасет с конфигурацией**
```
docker-compose exec config_server_1 mongo --quiet --eval "rs.initiate({
  _id: \"RScfg\",
  configsvr: true,
  version: 1,
  members: [
    {_id: 0, host: \"config_server_1\"},
    {_id: 1, host: \"config_server_2\"},
    {_id: 2, host: \"config_server_3\"}
 ]
})"
```
<details>
<summary>Output</summary><pre>
{
	"ok" : 1,
	"$gleStats" : {
		"lastOpTime" : Timestamp(1669229173, 1),
		"electionId" : ObjectId("000000000000000000000000")
	},
	"lastCommittedOpTime" : Timestamp(0, 0)
}
</pre></details>





1.2) **Создаем первый шард**
```
docker-compose exec shard_1_server_1 mongo --quiet --eval "rs.initiate({
  _id: \"RS1\",
  version: 1,
  members: [
    {_id: 0, host: \"shard_1_server_1\"},
    {_id: 1, host: \"shard_1_server_2\"},
    {_id: 2, host: \"shard_1_server_3\"}
  ]
})"
```
<details>
<summary>Output</summary><pre>
{ "ok" : 1 }
</pre></details>





1.3) **Создаем второй шард**
```
docker-compose exec shard_2_server_1 mongo --quiet --eval "rs.initiate({
  _id: \"RS2\",
  version: 1,
  members: [
    {_id: 0, host: \"shard_2_server_1\"},
    {_id: 1, host: \"shard_2_server_2\"},
    {_id: 2, host: \"shard_2_server_3\"}
  ]
})"
```
<details>
<summary>Output</summary><pre>
{ "ok" : 1 }
</pre></details>





1.4) **Создаем третий шард**
```
docker-compose exec shard_3_server_1 mongo --quiet --eval "rs.initiate({
  _id: \"RS3\",
  version: 1,
  members: [
    {_id: 0, host: \"shard_3_server_1\"},
    {_id: 1, host: \"shard_3_server_2\"},
    {_id: 2, host: \"shard_3_server_3\"}
  ]
})"
```
<details>
<summary>Output</summary><pre>
{ "ok" : 1 }
</pre></details>





2.1) **Добавляем первый шард в балансир**
```
docker-compose exec balancer_1 mongo --quiet --eval "
sh.addShard(\"RS1/shard_1_server_1\")
sh.addShard(\"RS1/shard_1_server_2\")
sh.addShard(\"RS1/shard_1_server_3\")
"
```
<details>
<summary>Output</summary><pre>
{
	"shardAdded" : "RS1",
	"ok" : 1,
	"operationTime" : Timestamp(1669229612, 1),
	"$clusterTime" : {
		"clusterTime" : Timestamp(1669229612, 1),
		"signature" : {
			"hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
			"keyId" : NumberLong(0)
		}
	}
}
</pre></details>





2.2) **Добавляем второй шард в балансир**
```
docker-compose exec balancer_1 mongo --quiet --eval "
sh.addShard(\"RS2/shard_2_server_1\")
sh.addShard(\"RS2/shard_2_server_2\")
sh.addShard(\"RS2/shard_2_server_3\")
"
```
<details>
<summary>Output</summary><pre>
{
	"shardAdded" : "RS2",
	"ok" : 1,
	"operationTime" : Timestamp(1669229666, 1),
	"$clusterTime" : {
		"clusterTime" : Timestamp(1669229666, 1),
		"signature" : {
			"hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
			"keyId" : NumberLong(0)
		}
	}
}
</pre></details>





2.3) **Добавляем третий шард в балансир**
```
docker-compose exec balancer_1 mongo --quiet --eval "
sh.addShard(\"RS3/shard_3_server_1\")
sh.addShard(\"RS3/shard_3_server_2\")
sh.addShard(\"RS3/shard_3_server_3\")
"
```
<details>
<summary>Output</summary><pre>
{
	"shardAdded" : "RS3",
	"ok" : 1,
	"operationTime" : Timestamp(1669230021, 12),
	"$clusterTime" : {
		"clusterTime" : Timestamp(1669230022, 1),
		"signature" : {
			"hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
			"keyId" : NumberLong(0)
		}
	}
}
</pre></details>





2.5) **Создаем коллекции**
```
docker-compose exec balancer_1 mongo --quiet --eval "
sh.enableSharding(\"shop\")
db.createCollection(\"items\")
db.items.createIndex( { _id: \"hashed\" } )
sh.shardCollection(
  \"shop.items\",
  { _id: \"hashed\"}
)
db.createCollection(\"customers\")
sh.shardCollection(
  \"shop.customers\",
  { _id: 1},
  true
)
"
```
<details>
<summary>Output</summary><pre>
{
	"collectionsharded" : "shop.customers",
	"collectionUUID" : UUID("9f01b4f1-ee03-4a5d-873e-a19e798a3fdf"),
	"ok" : 1,
	"operationTime" : Timestamp(1669230345, 75),
	"$clusterTime" : {
		"clusterTime" : Timestamp(1669230345, 75),
		"signature" : {
			"hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
			"keyId" : NumberLong(0)
		}
	}
}
</pre></details>





2.6) **Заполняем данными созданные коллекции**
```
docker-compose exec balancer_1 mongo --quiet --eval "
for (let i = 1; i <= 1000; ++i) {
    db.items.insertOne({ _id: i, name: i, price: i })
}
"
```
<details>
<summary>Output</summary><pre>
{ "acknowledged" : true, "insertedId" : 1000 }
</pre></details>

```
docker-compose exec balancer_2 mongo --quiet --eval "
for (let i = 1; i <= 50; ++i) {
    db.customers.insertOne({ _id: i, name: i, phone: i })
}
"
```
<details>
<summary>Output</summary><pre>
{ "acknowledged" : true, "insertedId" : 50 }
</pre></details>





2.7) **Поиск данных**
```
docker-compose exec balancer_2 mongo --quiet --eval "db.items.find().sort({ _id : 1 })"
```
<details>
<summary>Output</summary><pre>
{ "_id" : 1, "name" : 1, "price" : 1 }
{ "_id" : 2, "name" : 2, "price" : 2 }
{ "_id" : 3, "name" : 3, "price" : 3 }
{ "_id" : 4, "name" : 4, "price" : 4 }
{ "_id" : 5, "name" : 5, "price" : 5 }
{ "_id" : 6, "name" : 6, "price" : 6 }
{ "_id" : 7, "name" : 7, "price" : 7 }
{ "_id" : 8, "name" : 8, "price" : 8 }
{ "_id" : 9, "name" : 9, "price" : 9 }
{ "_id" : 10, "name" : 10, "price" : 10 }
...
</pre></details>

```
docker-compose exec balancer_2 mongo --quiet --eval "db.customers.find().sort({ _id : 1 })"
```
<details>
<summary>Output</summary><pre>
{ "_id" : 1, "name" : 1, "phone" : 1 }
{ "_id" : 2, "name" : 2, "phone" : 2 }
{ "_id" : 3, "name" : 3, "phone" : 3 }
{ "_id" : 4, "name" : 4, "phone" : 4 }
{ "_id" : 5, "name" : 5, "phone" : 5 }
{ "_id" : 6, "name" : 6, "phone" : 6 }
{ "_id" : 7, "name" : 7, "phone" : 7 }
{ "_id" : 8, "name" : 8, "phone" : 8 }
{ "_id" : 9, "name" : 9, "phone" : 9 }
{ "_id" : 10, "name" : 10, "phone" : 10 }
...
</pre></details>





2.8) **Поиск данных в первом шарде**
```
docker-compose exec shard_1_server_1 mongo --quiet --eval "db.items.find().sort({ _id : 1 })"
```
<details>
<summary>Output</summary><pre>

</pre></details>

```
docker-compose exec shard_1_server_1 mongo --quiet --eval "db.customers.find().sort({ _id : 1 })"
```
<details>
<summary>Output</summary><pre>

</pre></details>




2.9) **Поиск данных во втором шарде**
```
docker-compose exec shard_2_server_1 mongo --quiet --eval "db.items.find().sort({ _id : 1 })"
```
<details>
<summary>Output</summary><pre>
{ "_id" : 1, "name" : 1, "price" : 1 }
{ "_id" : 2, "name" : 2, "price" : 2 }
{ "_id" : 3, "name" : 3, "price" : 3 }
{ "_id" : 4, "name" : 4, "price" : 4 }
{ "_id" : 5, "name" : 5, "price" : 5 }
{ "_id" : 6, "name" : 6, "price" : 6 }
{ "_id" : 7, "name" : 7, "price" : 7 }
{ "_id" : 8, "name" : 8, "price" : 8 }
{ "_id" : 9, "name" : 9, "price" : 9 }
{ "_id" : 10, "name" : 10, "price" : 10 }
...
</pre></details>

```
docker-compose exec shard_2_server_1 mongo --quiet --eval "db.customers.find().sort({ _id : 1 })"
```
<details>
<summary>Output</summary><pre>
{ "_id" : 1, "name" : 1, "phone" : 1 }
{ "_id" : 2, "name" : 2, "phone" : 2 }
{ "_id" : 3, "name" : 3, "phone" : 3 }
{ "_id" : 4, "name" : 4, "phone" : 4 }
{ "_id" : 5, "name" : 5, "phone" : 5 }
{ "_id" : 6, "name" : 6, "phone" : 6 }
{ "_id" : 7, "name" : 7, "phone" : 7 }
{ "_id" : 8, "name" : 8, "phone" : 8 }
{ "_id" : 9, "name" : 9, "phone" : 9 }
{ "_id" : 10, "name" : 10, "phone" : 10 }
...
</pre></details>





3.1) **Останавливаем первый балансир**
```
docker-compose stop balancer_1
```
<details>
<summary>Output</summary><pre>
Stopping balancer_1 ... done
</pre></details>






3.2) **Запрашиваем данные у первого балансира**
```
docker-compose exec balancer_1 mongo --quiet --eval "db.items.find().sort({ _id : 1 })"
```
<details>
<summary>Output</summary><pre>
ERROR: No container found for balancer_1_1
(Контейнер с именем balancer_1_1 не найден)
</pre></details>





3.3) **Останавливаем secondary сервер во втором шарде**
```
docker-compose stop shard_2_server_2
```
<details>
<summary>Output</summary><pre>
Stopping shard_2_server_2 ... done
</pre></details>





3.4) **Запрашиваем данные со второго шарда**
```
docker-compose exec shard_2_server_1 mongo --quiet --eval "db.items.find().sort({ _id : 1 })"
```
<details>
<summary>Output</summary><pre>
{ "_id" : 1, "name" : 1, "price" : 1 }
{ "_id" : 2, "name" : 2, "price" : 2 }
{ "_id" : 3, "name" : 3, "price" : 3 }
{ "_id" : 4, "name" : 4, "price" : 4 }
{ "_id" : 5, "name" : 5, "price" : 5 }
{ "_id" : 6, "name" : 6, "price" : 6 }
{ "_id" : 7, "name" : 7, "price" : 7 }
{ "_id" : 8, "name" : 8, "price" : 8 }
{ "_id" : 9, "name" : 9, "price" : 9 }
{ "_id" : 10, "name" : 10, "price" : 10 }
{ "_id" : 11, "name" : 11, "price" : 11 }
{ "_id" : 12, "name" : 12, "price" : 12 }
{ "_id" : 13, "name" : 13, "price" : 13 }
{ "_id" : 14, "name" : 14, "price" : 14 }
{ "_id" : 15, "name" : 15, "price" : 15 }
{ "_id" : 16, "name" : 16, "price" : 16 }
{ "_id" : 17, "name" : 17, "price" : 17 }
{ "_id" : 18, "name" : 18, "price" : 18 }
{ "_id" : 19, "name" : 19, "price" : 19 }
{ "_id" : 20, "name" : 20, "price" : 20 }
(данные в порядке)
</pre></details>



/////////////////////////////////////////////////////////////////////////////////////////////////////////////////////


qwerty) **qwerty**
```
qwerty
```
<details>
<summary>Output</summary><pre>
qwerty
</pre></details>