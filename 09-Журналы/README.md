1. Установим кластер
```
sudo apt update
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
echo "deb http://apt.postgresql.org/pub/repos/apt/ `lsb_release -cs`-pgdg main" |sudo tee  /etc/apt/sources.list.d/pgdg.list
sudo apt update
sudo apt install postgresql-14
sudo -i -u postgres
```
2. Установим параметры и перезагрузим конфигурацию кластера
```
psql
ALTER SYSTEM SET checkpoint_timeout TO '30s';
ALTER SYSTEM SET log_checkpoints = on;
select pg_reload_conf();
```
3. Запомним текущую позицию добавления в журнале предзаписи. ```SELECT pg_current_wal_insert_lsn(); => 0/10BCE4F8```
4. Прогоним бенчмарк

```
pgbench -i
pgbench -c5 -P 30 -T 600  postgres
```
5. Получим текущую позицию и посчитаем разницу
```
psql
SELECT pg_current_wal_insert_lsn() => 0/2E6F7978
SELECT pg_size_pretty('0/2E6F7978'::pg_lsn - '0/10BCE4F8'::pg_lsn); => 475 MB
475 / 20 => 23.75Mb
```

6. Проверим логи чекпоинтов
```
2022-04-06 22:21:39.388 UTC [3500] LOG:  received SIGHUP, reloading configuration files
2022-04-06 22:21:39.389 UTC [3500] LOG:  parameter "checkpoint_timeout" changed to "30s"
2022-04-06 22:21:39.389 UTC [3500] LOG:  parameter "log_checkpoints" changed to "on"
2022-04-06 22:21:39.530 UTC [3502] LOG:  checkpoint complete: wrote 3435 buffers (21.0%); 0 WAL file(s) added, 0 removed, 7 recycled; write=245.335 s, sync=0.009 s, total=245.377 s; sync files=14, longest=0.009 s, average=0.001 s; distance=117246 kB, estimate=118662 kB
2022-04-06 22:22:09.560 UTC [3502] LOG:  checkpoint starting: time
2022-04-06 22:22:36.101 UTC [3502] LOG:  checkpoint complete: wrote 1752 buffers (10.7%); 0 WAL file(s) added, 0 removed, 1 recycled; write=26.477 s, sync=0.017 s, total=26.541 s; sync files=16, longest=0.008 s, average=0.002 s; distance=15743 kB, estimate=108370 kB
2022-04-06 22:22:39.104 UTC [3502] LOG:  checkpoint starting: time
2022-04-06 22:23:06.059 UTC [3502] LOG:  checkpoint complete: wrote 1947 buffers (11.9%); 0 WAL file(s) added, 1 removed, 1 recycled; write=26.881 s, sync=0.022 s, total=26.955 s; sync files=8, longest=0.008 s, average=0.003 s; distance=23699 kB, estimate=99903 kB
2022-04-06 22:23:09.062 UTC [3502] LOG:  checkpoint starting: time
2022-04-06 22:23:36.101 UTC [3502] LOG:  checkpoint complete: wrote 1986 buffers (12.1%); 0 WAL file(s) added, 0 removed, 1 recycled; write=26.980 s, sync=0.021 s, total=27.040 s; sync files=15, longest=0.010 s, average=0.002 s; distance=23933 kB, estimate=92306 kB
2022-04-06 22:23:39.102 UTC [3502] LOG:  checkpoint starting: time
2022-04-06 22:24:06.053 UTC [3502] LOG:  checkpoint complete: wrote 1917 buffers (11.7%); 0 WAL file(s) added, 2 removed, 0 recycled; write=26.890 s, sync=0.023 s, total=26.952 s; sync files=9, longest=0.018 s, average=0.003 s; distance=23463 kB, estimate=85421 kB
2022-04-06 22:24:09.056 UTC [3502] LOG:  checkpoint starting: time
2022-04-06 22:24:36.112 UTC [3502] LOG:  checkpoint complete: wrote 2064 buffers (12.6%); 0 WAL file(s) added, 0 removed, 1 recycled; write=26.980 s, sync=0.032 s, total=27.056 s; sync files=16, longest=0.015 s, average=0.002 s; distance=23871 kB, estimate=79266 kB
2022-04-06 22:24:39.114 UTC [3502] LOG:  checkpoint starting: time
2022-04-06 22:25:06.063 UTC [3502] LOG:  checkpoint complete: wrote 1920 buffers (11.7%); 0 WAL file(s) added, 0 removed, 1 recycled; write=26.878 s, sync=0.024 s, total=26.950 s; sync files=9, longest=0.015 s, average=0.003 s; distance=23989 kB, estimate=73739 kB
2022-04-06 22:25:09.066 UTC [3502] LOG:  checkpoint starting: time
2022-04-06 22:25:36.123 UTC [3502] LOG:  checkpoint complete: wrote 2058 buffers (12.6%); 0 WAL file(s) added, 1 removed, 1 recycled; write=26.984 s, sync=0.031 s, total=27.058 s; sync files=16, longest=0.015 s, average=0.002 s; distance=23606 kB, estimate=68725 kB
2022-04-06 22:25:39.126 UTC [3502] LOG:  checkpoint starting: time
2022-04-06 22:26:06.073 UTC [3502] LOG:  checkpoint complete: wrote 1927 buffers (11.8%); 0 WAL file(s) added, 0 removed, 1 recycled; write=26.887 s, sync=0.016 s, total=26.948 s; sync files=9, longest=0.016 s, average=0.002 s; distance=23978 kB, estimate=64251 kB
2022-04-06 22:26:09.076 UTC [3502] LOG:  checkpoint starting: time
2022-04-06 22:26:36.132 UTC [3502] LOG:  checkpoint complete: wrote 2051 buffers (12.5%); 0 WAL file(s) added, 2 removed, 0 recycled; write=26.981 s, sync=0.029 s, total=27.057 s; sync files=14, longest=0.017 s, average=0.003 s; distance=23591 kB, estimate=60185 kB
2022-04-06 22:26:39.134 UTC [3502] LOG:  checkpoint starting: time
2022-04-06 22:27:06.071 UTC [3502] LOG:  checkpoint complete: wrote 1904 buffers (11.6%); 0 WAL file(s) added, 0 removed, 1 recycled; write=26.883 s, sync=0.019 s, total=26.938 s; sync files=8, longest=0.014 s, average=0.003 s; distance=22950 kB, estimate=56461 kB
2022-04-06 22:27:09.074 UTC [3502] LOG:  checkpoint starting: time
2022-04-06 22:27:36.148 UTC [3502] LOG:  checkpoint complete: wrote 2064 buffers (12.6%); 0 WAL file(s) added, 1 removed, 1 recycled; write=26.980 s, sync=0.039 s, total=27.074 s; sync files=14, longest=0.014 s, average=0.003 s; distance=23893 kB, estimate=53204 kB
2022-04-06 22:27:39.150 UTC [3502] LOG:  checkpoint starting: time
2022-04-06 22:28:06.094 UTC [3502] LOG:  checkpoint complete: wrote 1900 buffers (11.6%); 0 WAL file(s) added, 0 removed, 1 recycled; write=26.885 s, sync=0.015 s, total=26.945 s; sync files=8, longest=0.015 s, average=0.002 s; distance=24028 kB, estimate=50287 kB
2022-04-06 22:28:09.097 UTC [3502] LOG:  checkpoint starting: time
2022-04-06 22:28:36.092 UTC [3502] LOG:  checkpoint complete: wrote 2058 buffers (12.6%); 0 WAL file(s) added, 0 removed, 2 recycled; write=26.909 s, sync=0.034 s, total=26.996 s; sync files=14, longest=0.017 s, average=0.003 s; distance=23702 kB, estimate=47628 kB
2022-04-06 22:28:39.096 UTC [3502] LOG:  checkpoint starting: time
2022-04-06 22:29:06.064 UTC [3502] LOG:  checkpoint complete: wrote 1891 buffers (11.5%); 0 WAL file(s) added, 0 removed, 1 recycled; write=26.893 s, sync=0.016 s, total=26.969 s; sync files=9, longest=0.016 s, average=0.002 s; distance=23331 kB, estimate=45199 kB
2022-04-06 22:29:09.066 UTC [3502] LOG:  checkpoint starting: time
2022-04-06 22:29:36.116 UTC [3502] LOG:  checkpoint complete: wrote 2046 buffers (12.5%); 0 WAL file(s) added, 0 removed, 1 recycled; write=26.985 s, sync=0.022 s, total=27.051 s; sync files=12, longest=0.014 s, average=0.002 s; distance=23053 kB, estimate=42984 kB
2022-04-06 22:29:39.118 UTC [3502] LOG:  checkpoint starting: time
2022-04-06 22:30:06.072 UTC [3502] LOG:  checkpoint complete: wrote 1930 buffers (11.8%); 0 WAL file(s) added, 1 removed, 1 recycled; write=26.881 s, sync=0.017 s, total=26.955 s; sync files=9, longest=0.017 s, average=0.002 s; distance=24343 kB, estimate=41120 kB
2022-04-06 22:30:09.074 UTC [3502] LOG:  checkpoint starting: time
2022-04-06 22:30:36.121 UTC [3502] LOG:  checkpoint complete: wrote 2266 buffers (13.8%); 0 WAL file(s) added, 0 removed, 1 recycled; write=26.982 s, sync=0.031 s, total=27.048 s; sync files=14, longest=0.013 s, average=0.003 s; distance=22531 kB, estimate=39261 kB
2022-04-06 22:30:39.124 UTC [3502] LOG:  checkpoint starting: time
2022-04-06 22:31:06.070 UTC [3502] LOG:  checkpoint complete: wrote 1929 buffers (11.8%); 0 WAL file(s) added, 0 removed, 2 recycled; write=26.881 s, sync=0.016 s, total=26.946 s; sync files=10, longest=0.016 s, average=0.002 s; distance=24088 kB, estimate=37744 kB
2022-04-06 22:31:09.073 UTC [3502] LOG:  checkpoint starting: time
2022-04-06 22:31:36.122 UTC [3502] LOG:  checkpoint complete: wrote 2024 buffers (12.4%); 0 WAL file(s) added, 0 removed, 1 recycled; write=26.983 s, sync=0.020 s, total=27.050 s; sync files=11, longest=0.013 s, average=0.002 s; distance=23845 kB, estimate=36354 kB
2022-04-06 22:31:39.126 UTC [3502] LOG:  checkpoint starting: time
2022-04-06 22:32:06.052 UTC [3502] LOG:  checkpoint complete: wrote 1902 buffers (11.6%); 0 WAL file(s) added, 1 removed, 1 recycled; write=26.881 s, sync=0.008 s, total=26.927 s; sync files=9, longest=0.008 s, average=0.001 s; distance=23731 kB, estimate=35092 kB
2022-04-06 22:33:09.115 UTC [3502] LOG:  checkpoint starting: time
```

7. Отметим, что при работе pgbench точки создавались примерно каждые 30 сек, а выполнялись 27. Затем заметно реже (около 1 раза в минуту) так как прекратились изменения данных

8. Сравним показатели при синхронном и асинхронном режиме
```
ALTER SYSTEM SET synchronous_commit TO on;
select pg_reload_conf();
pgbench -c5 -P 30 -T 500  postgres
tps = 730.148930 (without initial connection time)
```
```
ALTER SYSTEM SET synchronous_commit TO off;
select pg_reload_conf();
pgbench -c5 -P 30 -T 300  postgres
tps = 2587.117567 (without initial connection time)
```
9. Видим, что в асинхронном режиме записи скорость кратно возрасла, т.к. pg не дожидался подтверждения записи контрольной точки на диск.
10. Создаем новый кластер и заполняем тестовыми данными
```
sudo pg_createcluster 14 test -- --data-checksums
psql -p 5433
create table test (i int);
insert into test values (1),(2),(3);
```
11. Получаем расположение файла с данными и меняем его содержимое
```
SELECT pg_relation_filepath('test');
base/13760/16384
sudo -u postgres pg_ctlcluster 14 test stop
меняем файл /var/lib/postgresql/14/test/base/13760/16384
```
12. Запускаем кластер и пробуем получить данные

```
sudo -u postgres pg_ctlcluster 14 test start

postgres=# select * from test;
WARNING:  page verification failed, calculated checksum 55033 but expected 56454
ERROR:  invalid page in block 0 of relation base/13760/16384
```
13. Получили ошибку, так как контрольная сумма страницы не совпала с ожидаемой. Такие ошибки можно игнорировать с помощью параметра ignore_checksum_failure
