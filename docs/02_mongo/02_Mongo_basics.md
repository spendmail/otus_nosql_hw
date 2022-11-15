
# Домашнее задание №2
## Базовые возможности mongoDB

### Необходимо:
 - установить MongoDB одним из способов: ВМ, докер;
 - заполнить данными;
 - написать несколько запросов на выборку и обновление данных

### Ответ:
1) **Установка в Docker под ubuntu 20.04**
 ```
# docker pull mongo
# docker images
mongo   latest  b70536aeb250    2 weeks ago    695MB
# mkdir -p ~/mongodata
# docker run -it -v ~/mongodata:/data/db -p 27017:27017 --name mongodb -d mongo
# docker ps
95998d03c0a3 mongo "docker-entrypoint.s…" 7 seconds ago Up 6 seconds 0.0.0.0:27017->27017/tcp, :::27017->27017/tcp mongodb
```