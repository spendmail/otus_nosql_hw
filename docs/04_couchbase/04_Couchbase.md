
# Домашнее задание №4
## Кластер Couchbase

### Необходимо:
1. Развернуть кластер Couchbase
2. Создать БД, наполнить небольшими тестовыми данными
3. Проверить отказоустойчивость

### Решение:

1.1) **Запуск контейнеров**
```
docker-compose up -d
```
<details>
<summary>Output</summary><pre>
Creating network "04_couchbase_default" with the default driver
Creating cb-node-2 ... done
Creating cb-node-4 ... done
Creating cb-node-1 ... done
Creating cb-node-3 ... done
</pre></details>





1.2) **Создаем новый кластер**
 - Создавать кластер будем через веб, согласно рекомендации
 - Веб доступен по адресу https://localhost:18091/ (сконфигурировано в docker-compose.yaml) 

![This is an image](https://raw.githubusercontent.com/spendmail/otus_nosql_hw/main/docs/04_couchbase/screenshots/1.png)




1.3) **Задаем имя кластера, логин и пароль администратора**

![This is an image](https://raw.githubusercontent.com/spendmail/otus_nosql_hw/main/docs/04_couchbase/screenshots/2.png)



1.4) **Соглашаемся с условиями использования**

![This is an image](https://raw.githubusercontent.com/spendmail/otus_nosql_hw/main/docs/04_couchbase/screenshots/3.png)




1.5) **Конфигурируем первую ноду**
 - определяем ее как ноду для хранения данных

![This is an image](https://raw.githubusercontent.com/spendmail/otus_nosql_hw/main/docs/04_couchbase/screenshots/4.png)



2.1) **Создаем bucket population**
```
docker-compose exec cb-node-1 bash -c "
couchbase-cli bucket-create \
--cluster localhost:8091 --username admin \
--password passme --bucket population --bucket-type couchbase \
--bucket-ramsize 100
"
```
<details>
<summary>Output</summary><pre>
SUCCESS: Bucket created
</pre></details>




2.2) **Создаем scope population_scope**
```
docker-compose exec cb-node-1 bash -c "
couchbase-cli collection-manage \
--cluster http://localhost:8091 \
--username admin \
--password passme \
--bucket population \
--create-scope population_scope
"
```
<details>
<summary>Output</summary><pre>
SUCCESS: Scope created
</pre></details>




2.3) **Создаем коллекцию us_population**
```
docker-compose exec cb-node-1 bash -c "
couchbase-cli collection-manage -c localhost:8091 \
--username admin \
--password passme \
--bucket population \
--create-collection population_scope.us_population
"
```
<details>
<summary>Output</summary><pre>
SUCCESS: Collection created
</pre></details>





2.4) **Выводим список коллекций**
```
docker-compose exec cb-node-1 bash -c "
couchbase-cli collection-manage -c localhost:8091 \
--username admin \
--password passme \
--bucket population \
--list-collections population_scope
"
```
<details>
<summary>Output</summary><pre>
Scope population_scope:
    - us_population
</pre></details>





2.5) **Заполняем коллекцию документами**
```
docker-compose exec cb-node-1 bash -c "
cbc create -u admin -P passme doc_1 -U couchbase://localhost/population --scope='population_scope' --collection='us_population' -V '{\"date\": \"2021\",\"value\": 331893745,}' ;
cbc create -u admin -P passme doc_2 -U couchbase://localhost/population --scope='population_scope' --collection='us_population' -V '{\"date\": \"2020\",\"value\": 331501080,}' ;
cbc create -u admin -P passme doc_3 -U couchbase://localhost/population --scope='population_scope' --collection='us_population' -V '{\"date\": \"2019\",\"value\": 328329953,}' ;
cbc create -u admin -P passme doc_4 -U couchbase://localhost/population --scope='population_scope' --collection='us_population' -V '{\"date\": \"2018\",\"value\": 326838199,}' ;
cbc create -u admin -P passme doc_5 -U couchbase://localhost/population --scope='population_scope' --collection='us_population' -V '{\"date\": \"2017\",\"value\": 325122128,}' ;
cbc create -u admin -P passme doc_6 -U couchbase://localhost/population --scope='population_scope' --collection='us_population' -V '{\"date\": \"2016\",\"value\": 323071755,}' ;
cbc create -u admin -P passme doc_7 -U couchbase://localhost/population --scope='population_scope' --collection='us_population' -V '{\"date\": \"2015\",\"value\": 320738994,}' ;
cbc create -u admin -P passme doc_8 -U couchbase://localhost/population --scope='population_scope' --collection='us_population' -V '{\"date\": \"2014\",\"value\": 318386329,}' ;
cbc create -u admin -P passme doc_9 -U couchbase://localhost/population --scope='population_scope' --collection='us_population' -V '{\"date\": \"2013\",\"value\": 316059947,}' ;
cbc create -u admin -P passme doc_10 -U couchbase://localhost/population --scope='population_scope' --collection='us_population' -V '{\"date\": \"2012\",\"value\": 313877662,}' ;
"
```
<details>
<summary>Output</summary><pre>
doc_1                Stored. CAS=0x173020f5ed050000
                     SYNCTOKEN=953,232759052850583,3
doc_2                Stored. CAS=0x173020f5edcc0000
                     SYNCTOKEN=688,232284995489980,3
doc_3                Stored. CAS=0x173020f5eedb0000
                     SYNCTOKEN=439,224283278953648,3
doc_4                Stored. CAS=0x173020f5efe90000
                     SYNCTOKEN=979,40769804272736,3
doc_5                Stored. CAS=0x173020f5f0bf0000
                     SYNCTOKEN=212,223610377559718,3
doc_6                Stored. CAS=0x173020f5f1ab0000
                     SYNCTOKEN=477,187119947610596,3
doc_7                Stored. CAS=0x173020f5f2660000
                     SYNCTOKEN=730,245023863899599,3
doc_8                Stored. CAS=0x173020f5f33e0000
                     SYNCTOKEN=613,34702824440941,3
doc_9                Stored. CAS=0x173020f5f41d0000
                     SYNCTOKEN=354,20474814932630,3
doc_10               Stored. CAS=0x173020f5f5360000
                     SYNCTOKEN=818,82613304871264,3

</pre></details>





2.6) **Запрашиваем созданные документы**
```
docker-compose exec cb-node-1 bash -c "
cbc cat doc_1 -u admin -P passme -U couchbase://localhost/population --scope='population_scope' --collection='us_population' ;
cbc cat doc_2 -u admin -P passme -U couchbase://localhost/population --scope='population_scope' --collection='us_population' ;
cbc cat doc_3 -u admin -P passme -U couchbase://localhost/population --scope='population_scope' --collection='us_population' ;
cbc cat doc_4 -u admin -P passme -U couchbase://localhost/population --scope='population_scope' --collection='us_population' ;
cbc cat doc_5 -u admin -P passme -U couchbase://localhost/population --scope='population_scope' --collection='us_population' ;
cbc cat doc_6 -u admin -P passme -U couchbase://localhost/population --scope='population_scope' --collection='us_population' ;
cbc cat doc_7 -u admin -P passme -U couchbase://localhost/population --scope='population_scope' --collection='us_population' ;
cbc cat doc_8 -u admin -P passme -U couchbase://localhost/population --scope='population_scope' --collection='us_population' ;
cbc cat doc_9 -u admin -P passme -U couchbase://localhost/population --scope='population_scope' --collection='us_population' ;
cbc cat doc_10 -u admin -P passme -U couchbase://localhost/population --scope='population_scope' --collection='us_population' ;
"
```
<details>
<summary>Output</summary><pre>
doc_1                CAS=0x173020f5ed050000, Flags=0x0, Size=36, Datatype=0x00
{"date": "2021","value": 331893745,}
doc_2                CAS=0x173020f5edcc0000, Flags=0x0, Size=36, Datatype=0x00
{"date": "2020","value": 331501080,}
doc_3                CAS=0x173020f5eedb0000, Flags=0x0, Size=36, Datatype=0x00
{"date": "2019","value": 328329953,}
doc_4                CAS=0x173020f5efe90000, Flags=0x0, Size=36, Datatype=0x00
{"date": "2018","value": 326838199,}
doc_5                CAS=0x173020f5f0bf0000, Flags=0x0, Size=36, Datatype=0x00
{"date": "2017","value": 325122128,}
doc_6                CAS=0x173020f5f1ab0000, Flags=0x0, Size=36, Datatype=0x00
{"date": "2016","value": 323071755,}
doc_7                CAS=0x173020f5f2660000, Flags=0x0, Size=36, Datatype=0x00
{"date": "2015","value": 320738994,}
doc_8                CAS=0x173020f5f33e0000, Flags=0x0, Size=36, Datatype=0x00
{"date": "2014","value": 318386329,}
doc_9                CAS=0x173020f5f41d0000, Flags=0x0, Size=36, Datatype=0x00
{"date": "2013","value": 316059947,}
doc_10               CAS=0x173020f5f5360000, Flags=0x0, Size=36, Datatype=0x00
{"date": "2012","value": 313877662,}
</pre></details>





3.1) **Добавляем вторую ноду**
- определяем ее как ноду для хранения данных

![This is an image](https://raw.githubusercontent.com/spendmail/otus_nosql_hw/main/docs/04_couchbase/screenshots/5.png)





3.2) **Добавляем третью ноду**
- определяем ее как ноду для хранения данных

![This is an image](https://raw.githubusercontent.com/spendmail/otus_nosql_hw/main/docs/04_couchbase/screenshots/6.png)




3.3) **Добавляем четвертую ноду**
- определяем ее как ноду для Index, Query, Search, Analytics, Eventing, Backup

![This is an image](https://raw.githubusercontent.com/spendmail/otus_nosql_hw/main/docs/04_couchbase/screenshots/7.png)

![This is an image](https://raw.githubusercontent.com/spendmail/otus_nosql_hw/main/docs/04_couchbase/screenshots/8.png)




3.4) **Выполняем ребалансировку**

![This is an image](https://raw.githubusercontent.com/spendmail/otus_nosql_hw/main/docs/04_couchbase/screenshots/9.png)




3.5) **Выполняем ребалансировку**

![This is an image](https://raw.githubusercontent.com/spendmail/otus_nosql_hw/main/docs/04_couchbase/screenshots/9.png)





3.3) **Выполняем graceful failover для третьей ноды**

![This is an image](https://raw.githubusercontent.com/spendmail/otus_nosql_hw/main/docs/04_couchbase/screenshots/10.png)

![This is an image](https://raw.githubusercontent.com/spendmail/otus_nosql_hw/main/docs/04_couchbase/screenshots/11.png)




3.4) **Запрашиваем данные**
```
docker-compose exec cb-node-1 bash -c "
cbc cat doc_1 -u admin -P passme -U couchbase://localhost/population --scope='population_scope' --collection='us_population' ;
cbc cat doc_2 -u admin -P passme -U couchbase://localhost/population --scope='population_scope' --collection='us_population' ;
cbc cat doc_3 -u admin -P passme -U couchbase://localhost/population --scope='population_scope' --collection='us_population' ;
cbc cat doc_4 -u admin -P passme -U couchbase://localhost/population --scope='population_scope' --collection='us_population' ;
cbc cat doc_5 -u admin -P passme -U couchbase://localhost/population --scope='population_scope' --collection='us_population' ;
cbc cat doc_6 -u admin -P passme -U couchbase://localhost/population --scope='population_scope' --collection='us_population' ;
cbc cat doc_7 -u admin -P passme -U couchbase://localhost/population --scope='population_scope' --collection='us_population' ;
cbc cat doc_8 -u admin -P passme -U couchbase://localhost/population --scope='population_scope' --collection='us_population' ;
cbc cat doc_9 -u admin -P passme -U couchbase://localhost/population --scope='population_scope' --collection='us_population' ;
cbc cat doc_10 -u admin -P passme -U couchbase://localhost/population --scope='population_scope' --collection='us_population' ;
"
```
<details>
<summary>Output</summary><pre>
doc_1                CAS=0x173021bb4ceb0000, Flags=0x0, Size=36, Datatype=0x00
{"date": "2021","value": 331893745,}
doc_2                CAS=0x173021bb53c00000, Flags=0x0, Size=36, Datatype=0x00
{"date": "2020","value": 331501080,}
doc_3                CAS=0x173021bb5a3d0000, Flags=0x0, Size=36, Datatype=0x00
{"date": "2019","value": 328329953,}
doc_4                CAS=0x173021bb60a10000, Flags=0x0, Size=36, Datatype=0x00
{"date": "2018","value": 326838199,}
doc_5                CAS=0x173021bb67dc0000, Flags=0x0, Size=36, Datatype=0x00
{"date": "2017","value": 325122128,}
doc_6                CAS=0x173021bb6ee90000, Flags=0x0, Size=36, Datatype=0x00
{"date": "2016","value": 323071755,}
doc_7                CAS=0x173021bb7cd80000, Flags=0x0, Size=36, Datatype=0x00
{"date": "2015","value": 320738994,}
doc_8                CAS=0x173021bb84ba0000, Flags=0x0, Size=36, Datatype=0x00
{"date": "2014","value": 318386329,}
doc_9                CAS=0x173021bb8b020000, Flags=0x0, Size=36, Datatype=0x00
{"date": "2013","value": 316059947,}
doc_10               CAS=0x173021bb92540000, Flags=0x0, Size=36, Datatype=0x00
{"date": "2012","value": 313877662,}

</pre></details>





3.5) **Выполняем hard failover для второй ноды**

![This is an image](https://raw.githubusercontent.com/spendmail/otus_nosql_hw/main/docs/04_couchbase/screenshots/12.png)

![This is an image](https://raw.githubusercontent.com/spendmail/otus_nosql_hw/main/docs/04_couchbase/screenshots/13.png)



3.6) **Запрашиваем данные**
```
docker-compose exec cb-node-1 bash -c "
cbc cat doc_1 -u admin -P passme -U couchbase://localhost/population --scope='population_scope' --collection='us_population' ;
cbc cat doc_2 -u admin -P passme -U couchbase://localhost/population --scope='population_scope' --collection='us_population' ;
cbc cat doc_3 -u admin -P passme -U couchbase://localhost/population --scope='population_scope' --collection='us_population' ;
cbc cat doc_4 -u admin -P passme -U couchbase://localhost/population --scope='population_scope' --collection='us_population' ;
cbc cat doc_5 -u admin -P passme -U couchbase://localhost/population --scope='population_scope' --collection='us_population' ;
cbc cat doc_6 -u admin -P passme -U couchbase://localhost/population --scope='population_scope' --collection='us_population' ;
cbc cat doc_7 -u admin -P passme -U couchbase://localhost/population --scope='population_scope' --collection='us_population' ;
cbc cat doc_8 -u admin -P passme -U couchbase://localhost/population --scope='population_scope' --collection='us_population' ;
cbc cat doc_9 -u admin -P passme -U couchbase://localhost/population --scope='population_scope' --collection='us_population' ;
cbc cat doc_10 -u admin -P passme -U couchbase://localhost/population --scope='population_scope' --collection='us_population' ;
"
```
<details>
<summary>Output</summary><pre>
libcouchbase error: LCB_ERR_NO_MATCHING_SERVER (1010): The node the request was mapped to does not exist in the current cluster map. This may be the result of a failover
doc_2                CAS=0x173021bb53c00000, Flags=0x0, Size=36, Datatype=0x00
{"date": "2020","value": 331501080,}
doc_3                CAS=0x173021bb5a3d0000, Flags=0x0, Size=36, Datatype=0x00
{"date": "2019","value": 328329953,}
libcouchbase error: LCB_ERR_NO_MATCHING_SERVER (1010): The node the request was mapped to does not exist in the current cluster map. This may be the result of a failover
doc_5                CAS=0x173021bb67dc0000, Flags=0x0, Size=36, Datatype=0x00
{"date": "2017","value": 325122128,}
doc_6                CAS=0x173021bb6ee90000, Flags=0x0, Size=36, Datatype=0x00
{"date": "2016","value": 323071755,}
doc_7                CAS=0x173021bb7cd80000, Flags=0x0, Size=36, Datatype=0x00
{"date": "2015","value": 320738994,}
libcouchbase error: LCB_ERR_NO_MATCHING_SERVER (1010): The node the request was mapped to does not exist in the current cluster map. This may be the result of a failover
doc_9                CAS=0x173021bb8b020000, Flags=0x0, Size=36, Datatype=0x00
{"date": "2013","value": 316059947,}
doc_10               CAS=0x173021bb92540000, Flags=0x0, Size=36, Datatype=0x00
{"date": "2012","value": 313877662,}
</pre></details>




3.7) **Возвращаем ноды в работу и выполняем ребалансировку**

![This is an image](https://raw.githubusercontent.com/spendmail/otus_nosql_hw/main/docs/04_couchbase/screenshots/14.png)



