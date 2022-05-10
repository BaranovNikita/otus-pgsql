1. ```sudo apt update```
2. ```sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'```
3. ```wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -```
4. ```sudo apt -y update```
5. ```sudo apt -y install postgresql-14```
6. ```sudo -u postgres pg_lsclusters``` - (14  main    5432 online postgres /var/lib/postgresql/14/main /var/log/postgresql/postgresql-14-main.log)
7. `curl -s https://packagecloud.io/install/repositories/akopytov/sysbench/script.deb.sh | sudo bash`
8. `sudo apt -y install sysbench`
9. `git clone https://github.com/Percona-Lab/sysbench-tpcc.git`
10. `sudo -i -u postgres && psql`
11. `create database testdb;`
12. `\c testdb`
13. `CREATE USER test WITH PASSWORD 'test';`
14. `GRANT ALL PRIVILEGES ON DATABASE "testdb" to test;`
15. Запускаем prepare шаг для бенчамарка ```sudo -u postgres ./tpcc.lua prepare --pgsql-user=test --pgsql-password=test --pgsql-db=testdb --time=120 --threads=2 --report-interval=1 --tables=5 --scale=5 --use_fk=0 --trx_level=RC --db-driver=pgsql```
16. Запускаем бенч в 20 тредов на дефолтных настройках посгреса ```sudo -u postgres ./tpcc.lua --pgsql-user=test --pgsql-password=test --pgsql-db=testdb --time=1200 --threads=20 --report-interval=60 --tables=5 --scale=5 --use_fk=0 --trx_level=RC --db-driver=pgsql run``` и получаем в среднем 112 tps
17. Применяем настройки от pgtune с типом бд Web Application, 4RAM/2CPU/HDD и запускаем бенч. Получаем в средне 265tps. В конфиге изменились параметры памяти и ЦПУ<br/>
Конфиг:
```
# DB Version: 14
# OS Type: linux
# DB Type: web
# Total Memory (RAM): 4 GB
# CPUs num: 2
# Connections num: 100
# Data Storage: hdd

max_connections = 100
shared_buffers = 1GB
effective_cache_size = 3GB
maintenance_work_mem = 256MB
checkpoint_completion_target = 0.9
wal_buffers = 16MB
default_statistics_target = 100
random_page_cost = 4
effective_io_concurrency = 2
work_mem = 10485kB
min_wal_size = 1GB
max_wal_size = 4GB
max_worker_processes = 2
max_parallel_workers_per_gather = 1
max_parallel_workers = 2
max_parallel_maintenance_workers = 1
```
