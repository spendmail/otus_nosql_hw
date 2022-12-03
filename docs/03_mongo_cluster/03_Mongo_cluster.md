
# Домашнее задание №3
## Кластерные возможности mongodb

### Необходимо:
1. Построить шардированный кластер из 3 кластерных нод (по 3 инстанса с репликацией) и с кластером конфига (3 инстанса).
2. Добавить балансировку, нагрузить данными, выбрать хороший ключ шардирования, посмотреть как данные перебалансируются между шардами.
3. Поронять разные инстансы, посмотреть, что будет происходить, поднять обратно. Описать что произошло.
4. Настроить аутентификацию и многоролевой доступ.

### Решение:





0) **Запуск контейнеров с демонами mongod**
```
mkdir -p /tmp/mongo/{config_server_1,config_server_2,config_server_3,shard_1_server_1,shard_1_server_2,shard_1_server_3,shard_2_server_1,shard_2_server_2,shard_2_server_3,shard_3_server_1,shard_3_server_2,shard_3_server_3}
sudo chmod -R 777 /tmp/mongo/{config_server_1,config_server_2,config_server_3,shard_1_server_1,shard_1_server_2,shard_1_server_3,shard_2_server_1,shard_2_server_2,shard_2_server_3,shard_3_server_1,shard_3_server_2,shard_3_server_3}
docker-compose up -d
```
<details>
<summary>Output</summary><pre>
Creating network "03_mongo_cluster_mongo_cluster_network" with driver "bridge"
Creating shard_1_server_2 ... done
Creating shard_2_server_3 ... done
Creating shard_1_server_3 ... done
Creating shard_3_server_2 ... done
Creating balancer_1       ... done
Creating shard_3_server_3 ... done
Creating shard_2_server_2 ... done
Creating shard_1_server_1 ... done
Creating balancer_2       ... done
Creating shard_3_server_1 ... done
Creating shard_2_server_1 ... done
Creating config_server_1  ... done
Creating config_server_2  ... done
Creating config_server_3  ... done
</pre></details>





1.1) **Инициализация трех конфиг-серверов**
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
		"lastOpTime" : Timestamp(1670082509, 1),
		"electionId" : ObjectId("000000000000000000000000")
	},
	"lastCommittedOpTime" : Timestamp(0, 0)
}
</pre></details>





1.2) **Инициализируем первый шард из трех нод: primary, secondary и arbiter**
```
docker-compose exec shard_1_server_1 mongo --quiet --eval "rs.initiate({
  _id: \"RS1\",
  version: 1,
  members: [
    {_id: 0, host: \"shard_1_server_1\", priority: 3},
    {_id: 1, host: \"shard_1_server_2\"},
    {_id: 2, host: \"shard_1_server_3\", arbiterOnly : true}
  ]
})"
```
<details>
<summary>Output</summary><pre>
{ "ok" : 1 }
</pre></details>





1.3) **Инициализируем второй шард из трех нод: primary, secondary и arbiter**
```
docker-compose exec shard_2_server_1 mongo --quiet --eval "rs.initiate({
  _id: \"RS2\",
  version: 1,
  members: [
    {_id: 0, host: \"shard_2_server_1\", priority: 3},
    {_id: 1, host: \"shard_2_server_2\"},
    {_id: 2, host: \"shard_2_server_3\", arbiterOnly : true}
  ]
})"
```
<details>
<summary>Output</summary><pre>
{ "ok" : 1 }
</pre></details>





1.4) **Инициализируем третий шард из трех нод: primary, secondary и arbiter**
```
docker-compose exec shard_3_server_1 mongo --quiet --eval "rs.initiate({
  _id: \"RS3\",
  version: 1,
  members: [
    {_id: 0, host: \"shard_3_server_1\", priority: 3},
    {_id: 1, host: \"shard_3_server_2\"},
    {_id: 2, host: \"shard_3_server_3\", arbiterOnly : true}
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
	"operationTime" : Timestamp(1670082569, 1),
	"$clusterTime" : {
		"clusterTime" : Timestamp(1670082569, 1),
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
	"operationTime" : Timestamp(1670082611, 6),
	"$clusterTime" : {
		"clusterTime" : Timestamp(1670082611, 6),
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
	"operationTime" : Timestamp(1670082626, 3),
	"$clusterTime" : {
		"clusterTime" : Timestamp(1670082626, 3),
		"signature" : {
			"hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
			"keyId" : NumberLong(0)
		}
	}
}
</pre></details>





2.4) **Создаем коллекции**
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
	"collectionUUID" : UUID("a2ffd7ed-ce0e-4a99-899e-2e19e793e599"),
	"ok" : 1,
	"operationTime" : Timestamp(1670082654, 6),
	"$clusterTime" : {
		"clusterTime" : Timestamp(1670082654, 6),
		"signature" : {
			"hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
			"keyId" : NumberLong(0)
		}
	}
}
</pre></details>





2.5) **Заполняем данными созданные коллекции**
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





2.6) **Поиск данных**
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





2.7) **Поиск данных в первом шарде**
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




2.8) **Поиск данных в третьем шарде**
```
docker-compose exec shard_3_server_1 mongo --quiet --eval "db.items.find().sort({ _id : 1 })"
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
Type "it" for more
</pre></details>

```
docker-compose exec shard_3_server_1 mongo --quiet --eval "db.customers.find().sort({ _id : 1 })"
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
{ "_id" : 11, "name" : 11, "phone" : 11 }
{ "_id" : 12, "name" : 12, "phone" : 12 }
{ "_id" : 13, "name" : 13, "phone" : 13 }
{ "_id" : 14, "name" : 14, "phone" : 14 }
{ "_id" : 15, "name" : 15, "phone" : 15 }
{ "_id" : 16, "name" : 16, "phone" : 16 }
{ "_id" : 17, "name" : 17, "phone" : 17 }
{ "_id" : 18, "name" : 18, "phone" : 18 }
{ "_id" : 19, "name" : 19, "phone" : 19 }
{ "_id" : 20, "name" : 20, "phone" : 20 }
Type "it" for more

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





3.4) **Запрашиваем данные с третьего шарда**
```
docker-compose exec shard_3_server_1 mongo --quiet --eval "db.items.find().sort({ _id : 1 })"
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
Type "it" for more
(данные в порядке)
</pre></details>





3.5) **Возвращаю ноду**
```
docker-compose start shard_2_server_2
```
<details>
<summary>Output</summary><pre>
Starting shard_2_server_2 ... done
</pre></details>





4.1) **Генерируем файл для аутентификации и копируем его на ноды**
```
openssl rand -base64 756 > /tmp/keyfile
chmod 400 /tmp/keyfile
docker cp /tmp/keyfile config_server_1:/home/mongo/config_server_1/keyfile
docker exec config_server_1 chown mongodb: /home/mongo/config_server_1/keyfile
docker cp /tmp/keyfile config_server_2:/home/mongo/config_server_2/keyfile
docker exec config_server_2 chown mongodb: /home/mongo/config_server_2/keyfile
docker cp /tmp/keyfile config_server_3:/home/mongo/config_server_3/keyfile
docker exec config_server_3 chown mongodb: /home/mongo/config_server_3/keyfile
docker cp /tmp/keyfile shard_1_server_1:/home/mongo/shard_1_server_1/keyfile
docker exec shard_1_server_1 chown mongodb: /home/mongo/shard_1_server_1/keyfile
docker cp /tmp/keyfile shard_1_server_2:/home/mongo/shard_1_server_2/keyfile
docker exec shard_1_server_2 chown mongodb: /home/mongo/shard_1_server_2/keyfile
docker cp /tmp/keyfile shard_1_server_3:/home/mongo/shard_1_server_3/keyfile
docker exec shard_1_server_3 chown mongodb: /home/mongo/shard_1_server_3/keyfile
docker cp /tmp/keyfile shard_2_server_1:/home/mongo/shard_2_server_1/keyfile
docker exec shard_2_server_1 chown mongodb: /home/mongo/shard_2_server_1/keyfile
docker cp /tmp/keyfile shard_2_server_2:/home/mongo/shard_2_server_2/keyfile
docker exec shard_2_server_2 chown mongodb: /home/mongo/shard_2_server_2/keyfile
docker cp /tmp/keyfile shard_2_server_3:/home/mongo/shard_2_server_3/keyfile
docker exec shard_2_server_3 chown mongodb: /home/mongo/shard_2_server_3/keyfile
docker cp /tmp/keyfile shard_3_server_1:/home/mongo/shard_3_server_1/keyfile
docker exec shard_3_server_1 chown mongodb: /home/mongo/shard_3_server_1/keyfile
docker cp /tmp/keyfile shard_3_server_2:/home/mongo/shard_3_server_2/keyfile
docker exec shard_3_server_2 chown mongodb: /home/mongo/shard_3_server_2/keyfile
docker cp /tmp/keyfile shard_3_server_3:/home/mongo/shard_3_server_3/keyfile
docker exec shard_3_server_3 chown mongodb: /home/mongo/shard_3_server_3/keyfile
```
<details>
<summary>Output</summary><pre>

</pre></details>





4.2) **Создаем пользователей на нодах**
```
docker-compose exec config_server_1 mongo admin --quiet --eval "
db.createUser({user: \"cluster_admin\", pwd: \"passme\", roles: [\"clusterAdmin\"]})
db.createUser({user: \"db_owner\", pwd: \"passme\", roles: [{role: \"dbOwner\", db: \"*\"}]})
db.createUser({user: \"root\", pwd: \"passme\", roles: [\"root\"]})
db.createUser({user: \"shop_admin\", pwd: \"passme\", roles: [{role: \"readWrite\", db: \"shop\"}, {role: \"dbAdmin\", db: \"shop\"}]})
"

docker-compose exec shard_1_server_1 mongo admin --quiet --eval "
db.createUser({user: \"cluster_admin\", pwd: \"passme\", roles: [\"clusterAdmin\"]})
db.createUser({user: \"db_owner\", pwd: \"passme\", roles: [{role: \"dbOwner\", db: \"*\"}]})
db.createUser({user: \"root\", pwd: \"passme\", roles: [\"root\"]})
db.createUser({user: \"shop_admin\", pwd: \"passme\", roles: [{role: \"readWrite\", db: \"shop\"}, {role: \"dbAdmin\", db: \"shop\"}]})
"

docker-compose exec shard_2_server_1 mongo admin --quiet --eval "
db.createUser({user: \"cluster_admin\", pwd: \"passme\", roles: [\"clusterAdmin\"]})
db.createUser({user: \"db_owner\", pwd: \"passme\", roles: [{role: \"dbOwner\", db: \"*\"}]})
db.createUser({user: \"root\", pwd: \"passme\", roles: [\"root\"]})
db.createUser({user: \"shop_admin\", pwd: \"passme\", roles: [{role: \"readWrite\", db: \"shop\"}, {role: \"dbAdmin\", db: \"shop\"}]})
"

docker-compose exec shard_3_server_1 mongo admin --quiet --eval "
db.createUser({user: \"cluster_admin\", pwd: \"passme\", roles: [\"clusterAdmin\"]})
db.createUser({user: \"db_owner\", pwd: \"passme\", roles: [{role: \"dbOwner\", db: \"*\"}]})
db.createUser({user: \"root\", pwd: \"passme\", roles: [\"root\"]})
db.createUser({user: \"shop_admin\", pwd: \"passme\", roles: [{role: \"readWrite\", db: \"shop\"}, {role: \"dbAdmin\", db: \"shop\"}]})
"
```
<details>
<summary>Output</summary><pre>
Successfully added user: { "user" : "cluster_admin", "roles" : [ "clusterAdmin" ] }
Successfully added user: {
	"user" : "db_owner",
	"roles" : [
		{
			"role" : "dbOwner",
			"db" : "*"
		}
	]
}
Successfully added user: { "user" : "root", "roles" : [ "root" ] }
Successfully added user: {
	"user" : "shop_admin",
	"roles" : [
		{
			"role" : "readWrite",
			"db" : "shop"
		},
		{
			"role" : "dbAdmin",
			"db" : "shop"
		}
	]
}
</pre></details>





4.3) **Останавливаем ноды**
```
docker-compose down -v
```
<details>
<summary>Output</summary><pre>
Stopping config_server_3  ... done
Stopping config_server_2  ... done
Stopping config_server_1  ... done
Stopping shard_2_server_1 ... done
Stopping shard_3_server_1 ... done
Stopping balancer_2       ... done
Stopping shard_1_server_1 ... done
Stopping shard_3_server_2 ... done
Stopping shard_2_server_2 ... done
Stopping shard_3_server_3 ... done
Stopping shard_1_server_3 ... done
Stopping shard_2_server_3 ... done
Stopping shard_1_server_2 ... done
Removing config_server_3  ... done
Removing config_server_2  ... done
Removing config_server_1  ... done
Removing shard_2_server_1 ... done
Removing shard_3_server_1 ... done
Removing balancer_2       ... done
Removing shard_1_server_1 ... done
Removing shard_3_server_2 ... done
Removing shard_2_server_2 ... done
Removing balancer_1       ... done
Removing shard_3_server_3 ... done
Removing shard_1_server_3 ... done
Removing shard_2_server_3 ... done
Removing shard_1_server_2 ... done
Removing network 03_mongo_cluster_mongo_cluster_network
</pre></details>





4.4) **Перезапускаем ноды с ключами для аутентификации**
```
docker-compose -f docker-compose-with-auth.yaml up -d
```
<details>
<summary>Output</summary><pre>
Creating network "03_mongo_cluster_mongo_cluster_network" with driver "bridge"
Creating shard_3_server_3 ... done
Creating shard_1_server_2 ... done
Creating balancer_1       ... done
Creating shard_2_server_3 ... done
Creating shard_2_server_2 ... done
Creating shard_1_server_3 ... done
Creating shard_3_server_2 ... done
Creating shard_3_server_1 ... done
Creating shard_1_server_1 ... done
Creating shard_2_server_1 ... done
Creating balancer_2       ... done
Creating config_server_1  ... done
Creating config_server_2  ... done
Creating config_server_3  ... done
</pre></details>





4.5) **Пробуем подключиться без аутентификации**
```
docker exec -it shard_1_server_1 mongo
use admin
db.system.users.find()
```
<details>
<summary>Output</summary><pre>
Error: error: {
	"operationTime" : Timestamp(1670085160, 24),
	"ok" : 0,
	"errmsg" : "command find requires authentication",
	"code" : 13,
	"codeName" : "Unauthorized",
	"lastCommittedOpTime" : Timestamp(1670085160, 24),
	"$clusterTime" : {
		"clusterTime" : Timestamp(1670085160, 32),
		"signature" : {
			"hash" : BinData(0,"bJFRni8okWrMRhczPgUkUO0fTKI="),
			"keyId" : NumberLong("7172949805021265943")
		}
	}
}
</pre></details>





4.6) **Подключаемся с аутентификацией**
```
docker exec -it shard_1_server_1 mongo -u "root" -p "passme" --authenticationDatabase "admin"
use admin
db.system.users.find()
```
<details>
<summary>Output</summary><pre>
{ "_id" : "admin.cluster_admin", "userId" : UUID("73df7024-3b98-442d-abac-9a45bbdb5718"), "user" : "cluster_admin", "db" : "admin", "credentials" : { "SCRAM-SHA-1" : { "iterationCount" : 10000, "salt" : "Cf0Z40HCwjSAxzJtKpVNnw==", "storedKey" : "Wn5t7Mzhbohiw0qvpGTgI4MIHeE=", "serverKey" : "3B1Cjdjxg6QKM0b4vclJkqnqIYk=" }, "SCRAM-SHA-256" : { "iterationCount" : 15000, "salt" : "/LMSE//QJYACdlEvhVZOpjE4dVkU7gCfUlBmAw==", "storedKey" : "Dmc7lDiguPVW/nCfXGiyFMbULG4z4Vk8vvKMb1s34yo=", "serverKey" : "MqRB6uo/gbWlD2/YC+0grgKofl46lm0jx8RLQIfU/2M=" } }, "roles" : [ { "role" : "clusterAdmin", "db" : "admin" } ] }
{ "_id" : "admin.db_owner", "userId" : UUID("570a14ad-9c32-4269-84fa-4e0a64fb4754"), "user" : "db_owner", "db" : "admin", "credentials" : { "SCRAM-SHA-1" : { "iterationCount" : 10000, "salt" : "4VuMnMEp/RM7qOgm9I0A9A==", "storedKey" : "wjKxvRqu1rxe48UyUzaUDdfUy/M=", "serverKey" : "XTwH66Wv5lAV89eawpA8PjtxNE4=" }, "SCRAM-SHA-256" : { "iterationCount" : 15000, "salt" : "JFjUxIk4J3Svjg7i6hzxmXl0yKrlg3/9acdQ7w==", "storedKey" : "vQCTB0u+C6PkrU+nwlbKz2ryhpuqnJUkpIJjrxS6jh8=", "serverKey" : "VYDs9CSvQ+pwKGOn5VwSIfqztTpDo64DwkoBexCVMME=" } }, "roles" : [ { "role" : "dbOwner", "db" : "*" } ] }
{ "_id" : "admin.root", "userId" : UUID("85a6fed1-d705-4c1b-86f0-82a53b87ac89"), "user" : "root", "db" : "admin", "credentials" : { "SCRAM-SHA-1" : { "iterationCount" : 10000, "salt" : "GiCLyr+WEmGncuWmIDdvtA==", "storedKey" : "mkr6kQ2aTfKSgsekiFZpCaLZksA=", "serverKey" : "MBb+/MN5gaK5PeywIUDNuWxCvYU=" }, "SCRAM-SHA-256" : { "iterationCount" : 15000, "salt" : "RkajGdse1B1D44ZjLRTVWZPd8h+VOioRBT7CXQ==", "storedKey" : "XRxFa1bPVPUtnI0rkASNp52ccc37aZAu4IgP0mZFgVs=", "serverKey" : "tocY1ocWd4WA1fhbS1VOqj6P80MNVtfDbyus84ImWoM=" } }, "roles" : [ { "role" : "root", "db" : "admin" } ] }
{ "_id" : "admin.shop_admin", "userId" : UUID("325f20f3-0315-40cd-85f2-77f0550bff36"), "user" : "shop_admin", "db" : "admin", "credentials" : { "SCRAM-SHA-1" : { "iterationCount" : 10000, "salt" : "RQpf/4hcpGTJ3aJNvmL8KA==", "storedKey" : "8imWqYHL5ZQ/ixLyTXXQDnK37TE=", "serverKey" : "Ola4qjBNt+MdeNIkyFvddAN8KVg=" }, "SCRAM-SHA-256" : { "iterationCount" : 15000, "salt" : "lOOdyfcCbYeOKwFNlHRcmUNTOLzcoe3yIcFHvQ==", "storedKey" : "kbwu7+VlfGaku7+qLzcx6xkhLUfPpqa+oXra0FzITkM=", "serverKey" : "VBURbIjkwXvrY/HmYHrs/tlrtfFXZG5jN+016PLr7fM=" } }, "roles" : [ { "role" : "readWrite", "db" : "shop" }, { "role" : "dbAdmin", "db" : "shop" } ] }
</pre></details>
