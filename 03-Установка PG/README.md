
1. Установка докера по официальной инструкции https://docs.docker.com/engine/install/ubuntu/
2. Шаги после инстала докера https://docs.docker.com/engine/install/linux-postinstall/
3. Создадим папку ```sudo mkdir /var/lib/postgres```
4. Запустим контейнер
```
    docker run -d \
    --name some-postgres \
    -p 6432:5432 \
    -e POSTGRES_PASSWORD=12345 \
    -e PGDATA=/var/lib/postgresql/data/pgdata \
    -v /var/lib/postgres:/var/lib/postgresql/data \
    postgres:14.2
```
5. Создам сеть ```docker network connect pg```
6. Приаттачу пг контейнер к сети ```docker network connect pg some-postgres```
7. Создадим контейнер с клиентом ```docker run -d     --name postgres-client -e POSTGRES_PASSWORD=12345    postgres:14.2``` (можно найти более легковестный контейнер с утилитой psql)
8. Приаттачу клиент контейнер к сети ```docker network connect pg postgres-client```
9. Зайдем в контейнер клиент ```docker exec -it postgres-client bash```
10. Подключимся к пг ```psql -h some-postgres -U postgres```
11. Создадим таблицу ```create table persons(id serial, first_name text, second_name text); insert into persons(first_name, second_name) values('ivan', 'ivanov'); insert into persons(first_name, second_name) values('petr', 'petrov');```
12. Выйдем из контейнера и закроем ssh соединение ```exit; exit;```
13. С ноутбука подключился к посгресу в яндекс облаке ```psql -h IP -p 6432```
14. Удаляем контейнер ```docker rm some-postgres```
15. Пересоздаем контейнер командой из пункта 4
16. Коннектимся к посгресу и проверяем что все данные на месте так как мы вынесли данные в вольюм
