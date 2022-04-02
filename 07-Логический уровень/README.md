1. Установим кластер 13 версии 
```
sudo apt update
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
echo "deb http://apt.postgresql.org/pub/repos/apt/ `lsb_release -cs`-pgdg main" |sudo tee  /etc/apt/sources.list.d/pgdg.list
sudo apt update
sudo apt install postgresql-13
```
2. Зайдем под пользователем postgres и откроем psql
```
sudo -i -u postgres
psql
```
3. ```create database testdb;```
4. ```\c testdb```
5. ```create schema testnm;```
6. ```create table t1 (c1 int);```
7. ```insert into t1 values (1);```
8. ```create role readonly;```
9. ```GRANT CONNECT ON DATABASE testdb TO readonly;```
10. ```GRANT USAGE ON SCHEMA testnm  TO readonly;```
11. ```GRANT SELECT ON ALL TABLES IN SCHEMA testnm TO readonly;```
12. ```create user testread with password 'test123';```
13. ```grant readonly to testread;```
14. ```\c testdb testread``` (ловим ошибки про peer. Правим конфиг hba. Без подсказки)
15. ```\c testdb testread```
16. permission denied for table t1 при селекте так как нет права. мы выдали только на схему testnm, а при создании таблицы мы не указывали схему явно и она создалась в public
17. ```\c testdb postgres```
18. ```drop table t1;```
19. ```create table testnm.t1 (c1 int)```
20. ```insert into testnm.t1 values (1);```
21. ```\c testdb testread```
22. ```select * from testnm.t1;``` - permission denied for table t1 так как таблицу мы удаляли и создавали заново нужно перегрантить права под postgres (шаг 11)
23. после гранта селект отработал, чтобы не повторялось надо еще грантить default привелегии. Тогда права будут по умолчанию для новых таблиц
24. ```create table t2(c1 integer); insert into t2 values (2);``` - таблица создалась в public схеме, а у нашего юзера есть public роль, которая позваляет создавать любые сущности в схеме public (без подсказки не осилил)
25. Для того чтобы не повторилось надо удалить схему и роль public. Либо ревокнуть привелегии как это написано в шпаргалке
26. ```create table t3(c1 integer); insert into t2 values (2);``` - permission denied for schema public так как мы ревокнули права на создание 
