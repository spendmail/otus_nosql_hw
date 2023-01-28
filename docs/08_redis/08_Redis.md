
# Домашнее задание №8
## Redis

### Необходимо:
1. Сохранить большой json (~20МБ) в виде разных структур - строка, hset, zset, list.
2. Протестировать скорость сохранения и чтения.
3. Предоставить отчет.
4. Настроить редис кластер на 3х нодах с отказоусточивостью, затюнить таймауты.



### Решение:



1.) **Запуск контейнеров**
```
docker-compose up -d
```
<details>
<summary>Output</summary><pre>
Creating network "08_redis_default" with the default driver
Pulling redis (redis:7.0.2)...
7.0.2: Pulling from library/redis
b85a868b505f: Pull complete
b09642bd3b88: Pull complete
e0678a951c8d: Pull complete
d5d7c0a1681b: Pull complete
954286b64dd1: Pull complete
58024fcab1ef: Pull complete
Digest: sha256:d581aded52343c461f32e4a48125879ed2596291f4ea4baa7e3af0ad1e56feed
Status: Downloaded newer image for redis:7.0.2
Creating master ... done
Creating slave1 ... done
Creating slave2 ... done
</pre></details>




2.) **Проверяем статус мастера**
```
echo "INFO replication" | redis-cli -p 16379
```
<details>
<summary>Output</summary><pre>
# Replication
role:master
connected_slaves:2
slave0:ip=172.27.0.3,port=6379,state=online,offset=56,lag=0
slave1:ip=172.27.0.4,port=6379,state=online,offset=56,lag=0
master_failover_state:no-failover
master_replid:aa47463d12224ff2c19340d59d2d7e1b06ee8da7
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:56
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:56
</pre></details>




3.) **Наполняем данными**
```
for x in {1..200000}; do echo "SET   hw:string:$x $x"       >> data.txt; done;
for x in {1..200000}; do echo "HSET  hw:hashtable:$x id $x" >> data.txt; done;
for x in {1..200000}; do echo "RPUSH hw:list:$x $x"         >> data.txt; done;
for x in {1..200000}; do echo "SADD  hw:set:$x $x"          >> data.txt; done;
for x in {1..200000}; do echo "ZADD  hw:zset:$x $x $x"      >> data.txt; done;
time cat data.txt | redis-cli -p 16379 --pipe
```
<details>
<summary>Output</summary><pre>
All data transferred. Waiting for the last reply...
Last reply received from server.
errors: 0, replies: 1000000

real	0m2,085s
user	0m0,075s
sys	0m0,106s
</pre></details>





4.) **Запрашиваем количество ключей на слейве**
```
echo "INFO" | redis-cli -p 26379 | grep "db0:keys"
```
<details>
<summary>Output</summary><pre>
db0:keys=1000000,expires=0,avg_ttl=0
</pre></details>




5.) **Запрашиваем различные данные на слейвах**
```
echo "GET hw:string:10001" | redis-cli -p 26379
```
<details>
<summary>Output</summary><pre>
"10001"
</pre></details>


```
echo "HGET hw:hashtable:100 id" | redis-cli -p 26379
```
<details>
<summary>Output</summary><pre>
"100"
</pre></details>


```
echo "HKEYS hw:hashtable:100" | redis-cli -p 26379
```
<details>
<summary>Output</summary><pre>
1) "id"
</pre></details>




```
echo "HVALS hw:hashtable:100" | redis-cli -p 26379
```
<details>
<summary>Output</summary><pre>
1) "100"
</pre></details>



```
echo "LLEN hw:list:200" | redis-cli -p 36379
```
<details>
<summary>Output</summary><pre>
(integer) 1
</pre></details>



```
echo "SCARD hw:set:2000" | redis-cli -p 36379
```
<details>
<summary>Output</summary><pre>
(integer) 1
</pre></details>





```
echo "ZCOUNT hw:zset:100 5 100" | redis-cli -p 36379
```
<details>
<summary>Output</summary><pre>
(integer) 1
</pre></details>










6.) **Остановка контейнеров**
```
docker-compose down -v
```
<details>
<summary>Output</summary><pre>
Stopping slave1 ... done
Stopping slave2 ... done
Stopping master ... done
Removing slave1 ... done
Removing slave2 ... done
Removing master ... done
Removing network 08_redis_default

</pre></details>



8.) **Вывод**
 - Redis - незаменимый key-value nosql storage, если требуется хранить: сессии, кэш, одноразовые коды, куки, ключи, токены и т.д. 
 - От memcached его выгодно отличает поддержка типов данных, таких как: списки, хэш-мапы, множества и битовые строки.
 - Скорость записи и чтения можно сравнить с прямым доступом к памяти.
 - Возможность сохранения данных на диск гарантирует персистентность.  
 - Возможность построения кластера с поддержкой репликации с автоматическим выбором мастера и шардирования делает из redis'а полноценный production-ready инструмент для быстрых записи и доступа к простым данным.
