
1. ```sudo apt update```
2. ```sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'```
3. ```wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -```
4. ```sudo apt -y update```
5. ```sudo apt -y install postgresql-14```
6. ```sudo -u postgres pg_lsclusters``` - (14  main    5432 online postgres /var/lib/postgresql/14/main /var/log/postgresql/postgresql-14-main.log)
7. ```sudo -i -u postgres```
8. Монтирование диска в яндексе и разметка его по инструкции
9. ```sudo chown -R postgres:postgres /mnt/data/```
10. ```mv /var/lib/postgresql/14 /mnt/data``` Под пользователем postgres
11. При попытке запуска получаем Error: /var/lib/postgresql/14/main is not accessible or does not exist так как данные физически перенесены в другое место
12. Изучаем conf файлы в папке /etc/postgresql/14/main. В файле postgresql.conf находим путь до data (данных) в параметре data_directory. Ставим туда '/mnt/data/14/main' 
13. Запуск успешный. Файлы на месте
14. Звездочка:
15. cd /var/lib/postgresql/
16. rm -rf 14
17. Останавливаем кластер pg
18. В консоли переаттачиваем диск от виртуалки otus к виртуалке otus2
19. Монтируем диск. Разметка и лейблы уже есть
20. Редактируем fstab
21. Редактируем data_directory в postgresql.conf
22. Запускаем кластер. Данные на месте
