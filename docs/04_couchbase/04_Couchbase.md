
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



1.6) **Конфигурируем вторую ноду**
- определяем ее как ноду для хранения данных

![This is an image](https://raw.githubusercontent.com/spendmail/otus_nosql_hw/main/docs/04_couchbase/screenshots/5.png)



1.7) **Конфигурируем третью ноду**
- определяем ее как ноду для хранения данных

![This is an image](https://raw.githubusercontent.com/spendmail/otus_nosql_hw/main/docs/04_couchbase/screenshots/6.png)



1.8) **Конфигурируем четвертую ноду**
- определяем ее как ноду для работы индексов, обработки запросов, поиска, аналитики и бэкапа

![This is an image](https://raw.githubusercontent.com/spendmail/otus_nosql_hw/main/docs/04_couchbase/screenshots/7.png)

![This is an image](https://raw.githubusercontent.com/spendmail/otus_nosql_hw/main/docs/04_couchbase/screenshots/8.png)



1.7) **Выполняем ребалансировку данных**

![This is an image](https://raw.githubusercontent.com/spendmail/otus_nosql_hw/main/docs/04_couchbase/screenshots/9.png)


