
# Домашнее задание №2
## Базовые возможности mongoDB

### Необходимо:
 - установить MongoDB одним из способов: ВМ, докер;
 - заполнить данными;
 - написать несколько запросов на выборку и обновление данных

### Решение:
1) **Скачивание образа mongo 4.4.17**
```
# docker pull mongo:4.4.17
# docker images
> mongo 4.4.17 af86910c16da 2 weeks ago 438MB
```

2) **Запуск контейнера**
```
# mkdir -p ~/mongodata
# docker run -it -v ~/mongodata:/data/db -p 27017:27017 --name mongodb -d mongo:4.4.17
# docker ps
> 85dbd4a12079 mongo:4.4.17 "docker-entrypoint.s…" 8 seconds ago Up 7 seconds 0.0.0.0:27017->27017/tcp, :::27017->27017/tcp mongodb
```

3) **Создание пользователя**
```
# mongo -host localhost -port 27017
use hw02
db.createUser({
    user: "user",
    pwd: "passme",
    roles: [
        {role: "readWrite", db: "hw02"}
    ],
    passwordDigestor: "server"
})
```

4) **Импорт тестовых данных**
```
# curl -o ~/mongodata/customers.json https://raw.githubusercontent.com/spendmail/otus_nosql_hw/main/docs/02_mongo/mall_customers.json
# mongoimport --host=localhost --port=27017 --type json -d hw02 -c customers ~/mongodata/customers.json -u user --jsonArray
```

5) **Запись данных**
```
# mongo --username user --password --authenticationDatabase hw02 --host localhost --port 27017
use hw02
db.customers.insertOne({"customer_id": "0201", "genre": "Male", "age": 31, "annual_income": 132, "spending_score": 85});
db.customers.insertMany([
    {"customer_id": "0202", "genre": "Male", "age": 32, "annual_income": 133, "spending_score": 86},
    {"customer_id": "0203", "genre": "Male", "age": 33, "annual_income": 134, "spending_score": 87},
]);
```

6) **Чтение данных**
```
use hw02
db.customers.find().pretty()
```

7) **Обновление данных**
```
use hw02
db.customers.updateOne({"customer_id": "0201"}, {$set: {"genre": "Female"}})
db.customers.updateMany({"customer_id": "0202"}, {$set: {"genre": "Female"}})
```

8) **Создание индекса и поиск**
```
db.customers.explain().find({"customer_id": "0200"})
db.customers.createIndex({"customer_id": 1}, {unique: true})
db.customers.explain().find({"customer_id": "0200"})
> "queryPlanner" : {
>     "winningPlan" : {
>         "stage" : "FETCH",
>         "inputStage" : {
>             "stage" : "IXSCAN",
```

9) **Удаление данных**
```
use hw02
db.customers.deleteMany({customer_id : "0202"})
db.customers.drop()
db.dropDatabase()
show dbs
```
