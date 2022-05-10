1. В яндех облаке создаю 4 виртуалки (otus1...4)
2. Создаю базы на трех инстансах (db1, db2, db3) и пользователей для этих баз<br>
```
create database db1;
create role user1 with password 'user1';
grant all privileges ON DATABASE db1 TO user1;
```
3. На этих машинках создаю таблицы test, test2<br>
```
create table test (id integer);
create table test2 (id integer);
```
4. Разрешаю подключеник к кластеру из локальной сети (на первых двух машинках)
5. На первой машинке создаю публикацию первой таблицы `create publication publication1 for table test;`
6. На второй машинке создаю публикацию второй таблицы `create publication publication2 for table test2;`
7. В конфиге выставляю wal_level в *logical* и в listen_addresses добавляю локальную подсеть
8. На первой машине создаю подписку на публикацию со второй машины `create subscription subsc1 CONNECTION 'host=10.128.0.13 port=5432 user=user2 password=user2 dbname=db2' publication publication2 with ( copy_data = true);`
9. На второй машине делаю аналогичную подписку на первую
10. На третьей машине создаю две подписки (на первую и вторую машины)<br>
```
create subscription subsc1 CONNECTION 'host=10.128.0.13 port=5432 user=user2 password=user2 dbname=db2' publication publication2 with ( copy_data = true);
create subscription subsc2 CONNECTION 'host=10.128.0.23 port=5432 user=user1 password=user1 dbname=db1' publication publication1 with ( copy_data = true);
```
11. Теперь при изменении записей в таблицу test или test2 на первой и второй виртуалке записи меняются сразу на трех кластерах ПГ.
12. На 3 и 4 кластрах выставляем wal_level в *replica*
13. На 3 кластере разрешаем подключение четвертой машины для репликации по адресу из лкокальной сети<br> `host    replication     all             10.128.0.28/32          md5`
14. На 4 виртуалке останавливаем кластер и чистим main каталог
15. Делаем бекап с третьей машины и копируем его на четвертую (можно через pg_basebackup). В данный момент четвертая машина полностью копирует остальные
