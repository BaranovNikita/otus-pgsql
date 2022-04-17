1. ```sudo apt update```
2. ```sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'```
3. ```wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -```
4. ```sudo apt -y update```
5. ```sudo apt -y install postgresql-14```
6. ```sudo -u postgres pg_lsclusters``` - (14  main    5432 online postgres /var/lib/postgresql/14/main /var/log/postgresql/postgresql-14-main.log)
7. ```sudo -i -u postgres && psql```
8. ```ALTER SYSTEM SET deadlock_timeout = 200;```
9. ```ALTER SYSTEM SET log_lock_waits = on;```
10. ```SELECT pg_reload_conf();```
11. Для воспроизведения блокировки создадим тестовую таблицу и попытаемся обновить запись из двух транзакций
```
create table test (id int, text varchar);
insert into test values(1, 'привет');
begin;
update test set text='hello' where id = 1;
```
И во второй сессии
```=
begin;
update test set text='бонжур' where id = 1;
```
Видим что выполнение операции "повисло"
Делаем ```commit;``` в первой сессии и во второй сессии завершается выполнение апдейта и тоже можем сделать коммит.  
12. Смотрим журнал
```
2022-04-17 20:23:30.400 UTC [5045] postgres@postgres STATEMENT:  update test set text='hello' where id = 1;
2022-04-17 20:25:07.907 UTC [5127] postgres@postgres LOG:  process 5127 still waiting for ShareLock on transaction 735 after 200.061 ms
2022-04-17 20:25:07.907 UTC [5127] postgres@postgres DETAIL:  Process holding the *lock*: 5045. Wait queue: 5127.
2022-04-17 20:25:07.907 UTC [5127] postgres@postgres CONTEXT:  while updating tuple (0,1) in relation "test"
2022-04-17 20:25:07.907 UTC [5127] postgres@postgres STATEMENT:  update test set text='бонжур' where id = 1;
2022-04-17 20:26:04.243 UTC [5127] postgres@postgres LOG:  process 5127 acquired ShareLock on transaction 735 after 56536.223 ms
2022-04-17 20:26:04.243 UTC [5127] postgres@postgres CONTEXT:  while updating tuple (0,1) in relation "test"
2022-04-17 20:26:04.243 UTC [5127] postgres@postgres STATEMENT:  update test set text='бонжур' where id = 1;
```

Видим сообщения о возникших блокировках  
13. Блокировка строки в трех сессиях
сессия №1:  
```begin; update test set text='как дела' where id = 1;``` - успешно выполнено  
сессия №2:  
```begin; update test set text='how are you?' where id = 1;``` - блокировка  
сессия №3:  
```begin; update test set text='Wie geht es Ihnen?' where id = 1;``` - блокировка  
сессия №4:  
```SELECT locktype, mode, granted, pid, pg_blocking_pids(pid) AS wait_for FROM pg_locks WHERE relation = 'test'::regclass;```  
Видим список блокировок в таблице:  
| locktype | mode | granted | pid | wait_for | описание |
|-----|---|---|---|---|---|
|  relation   |  RowExclusiveLock | t  | 5208  | {5196}  | транзакция третьей сессии. ждет вторую |
|  relation   |  RowExclusiveLock | t  | 5127  | {}  | транзакция первой сессии |
|  relation   |  RowExclusiveLock | t  | 5196  | {5127}  | транзакция второй сессии. ждет первую |
|  tuple   |  ExclusiveLock | t  | 5196  | {5127}  | блокировка версии строки |
|  tuple   |  ExclusiveLock | f  | 5208  | {5196}  | блокировка версии строки |

После коммита в первой транзакции ситуация меняется:
| locktype | mode | granted | pid | wait_for | описание |
|-----|---|---|---|---|---|
|  relation   |  RowExclusiveLock | t  | 5208  | {5196}  | транзакция третьей сессии. ждет вторую |
|  relation   |  RowExclusiveLock | t  | 5196  | {}  | транзакция второй сессии. теперь ничего не ждет |

Так же пропали ExclusiveLock для tuple так как только одна транзакция ожидается эту строку. После коммита во второй сесии - блокировок больше нет.

14. Добавим данных ```insert into test values (2, 'два'), (3, 'три');```
15. Воспроизведем взаимную блокировку трех транзакций:  
сессия №1:  
```begin; update test set text='one' where id = 1;```  
сессия №2:  
```begin; update test set text='two' where id = 2;```  
сессия №3:  
```begin; update test set text='three' where id = 3;```  
сессия №1:  
```begin; update test set text='zwei' where id = 2;```  
сессия №2:  
```begin; update test set text='drei' where id = 3;```  
сессия №3:  
```begin; update test set text='ein' where id = 1;```    
Видим ошибку в сессии 3:  
```
ERROR:  deadlock detected
DETAIL:  Process 5208 waits for ShareLock on transaction 747; blocked by process 5127.
Process 5127 waits for ShareLock on transaction 748; blocked by process 5196.
Process 5196 waits for ShareLock on transaction 749; blocked by process 5208.
HINT:  See server log for query details.
CONTEXT:  while updating tuple (0,5) in relation "test"
```
Смотрим логи:
```
2022-04-17 21:12:02.291 UTC [5208] postgres@postgres LOG:  process 5208 detected deadlock while waiting for ShareLock on transaction 747 after 200.124 ms
2022-04-17 21:12:02.291 UTC [5208] postgres@postgres DETAIL:  Process holding the lock: 5127. Wait queue: .
2022-04-17 21:12:02.291 UTC [5208] postgres@postgres CONTEXT:  while updating tuple (0,5) in relation "test"
2022-04-17 21:12:02.291 UTC [5208] postgres@postgres STATEMENT:  update test set text='ein' where id = 1;
2022-04-17 21:12:02.291 UTC [5208] postgres@postgres ERROR:  deadlock detected
2022-04-17 21:12:02.291 UTC [5208] postgres@postgres DETAIL:  Process 5208 waits for ShareLock on transaction 747; blocked by process 5127.
	Process 5127 waits for ShareLock on transaction 748; blocked by process 5196.
	Process 5196 waits for ShareLock on transaction 749; blocked by process 5208.
	Process 5208: update test set text='ein' where id = 1;
	Process 5127: update test set text='zwei' where id = 2;
	Process 5196: update test set text='drei' where id = 3;
2022-04-17 21:12:02.291 UTC [5208] postgres@postgres HINT:  See server log for query details.
2022-04-17 21:12:02.291 UTC [5208] postgres@postgres CONTEXT:  while updating tuple (0,5) in relation "test"
2022-04-17 21:12:02.291 UTC [5208] postgres@postgres STATEMENT:  update test set text='ein' where id = 1;
2022-04-17 21:12:02.291 UTC [5196] postgres@postgres LOG:  process 5196 acquired ShareLock on transaction 749 after 7711.462 ms
2022-04-17 21:12:02.291 UTC [5196] postgres@postgres CONTEXT:  while updating tuple (0,11) in relation "test"
2022-04-17 21:12:02.291 UTC [5196] postgres@postgres STATEMENT:  update test set text='drei' where id = 3;
2022-04-17 21:13:10.453 UTC [5127] postgres@postgres LOG:  process 5127 acquired ShareLock on transaction 748 after 82975.589 ms
2022-04-17 21:13:10.453 UTC [5127] postgres@postgres CONTEXT:  while updating tuple (0,10) in relation "test"
2022-04-17 21:13:10.453 UTC [5127] postgres@postgres STATEMENT:  update test set text='zwei' where id = 2;
```
Данных в целом достаточно для изучения причины произошедшего  
16.Две транзакции на update по одной и той же таблице могут заблокировать друг друга.  
Для воспроизведения создадим таблицу с одним полем. Сделаем индекс по этому полю (но в порядке убывания). Отключим seqscan в одной из сессий для избежания последовательного сканирования. 
```
create table test2 (id int);
insert into test2 (id) (SELECT generate_series(1, 9999999));
CREATE INDEX ON test2(id DESC);
```
Запустим в двух сессиях обновление всех строк. Постепенно обновления в первом сеансе дойдут до строк второго - мы словим дедлок  
сессия №1: ```UPDATE test2 SET id = id + 1;```  
сессия №2: ```SET enable_seqscan = off;UPDATE test2 SET id = id + 9999999;```
