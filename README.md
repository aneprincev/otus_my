1. Создайте новый кластер PostgresSQL 14  
``` sql
user@postgres:~$ pg_lsclusters
Ver Cluster Port Status Owner    Data directory    Log file
15  main    5432 online postgres /mnt/data/15/main /var/log/postgresql/postgresql-15-main.log
``` 

2. Зайдите в созданный кластер под пользователем postgres
``` sql
user@postgres:~$ sudo -u postgres psql
[sudo] password for user: 
psql (15.5 (Ubuntu 15.5-1.pgdg22.04+1))
Type "help" for help.

postgres=# 
```  

3. Создайте новую базу данных testdb
``` sql
postgres=# create database testdb;
CREATE DATABASE
postgres=#
``` 

4. Зайдите в созданную базу данных под пользователем postgres
``` sql
postgres=# \c testdb
You are now connected to database "testdb" as user "postgres".
testdb=# select current_user;
 current_user 
--------------
 postgres
```  

5. Создайте новую схему testnm
``` sql
testdb=# create schema testnm;
CREATE SCHEMA
testdb=#
```  

6. Создайте новую таблицу t1 с одной колонкой c1 типа integer
``` sql
testdb=# create table t1(c1 int);
CREATE TABLE
testdb=#
```  

7. Вставьте строку со значением c1=1
``` sql
testdb=# insert into t1 (c1) values (1);
INSERT 0 1
testdb=#
```  

8. Создайте новую роль readonly
``` sql
testdb=# create role readonly;
CREATE ROLE
```  

9. Дайте новой роли право на подключение к базе данных testdb
``` sql
testdb=# GRANT CONNECT ON DATABASE testdb TO readonly;
GRANT
```  

10. Дайте новой роли право на использование схемы testnm
``` sql
testdb=# GRANT USAGE ON SCHEMA testnm TO readonly;
GRANT
```  

11. Дайте новой роли право на select для всех таблиц схемы testnm
``` sql
testdb=# GRANT SELECT ON ALL TABLES in SCHEMA testnm TO readonly;
GRANT
```  

12. Создайте пользователя testread с паролем test123
``` sql
testdb=# CREATE USER testread with PASSWORD 'test123';
CREATE ROLE
``` 

13. Дайте роль readonly пользователю testread
``` sql
testdb=# GRANT readonly TO testread;
CREATE ROLE
``` 

14. Зайдите под пользователем testread в базу данных testdb
``` sql
user@postgres:~$ psql -U testread -h localhost -d testdb
Password for user testread: 
psql (15.5 (Ubuntu 15.5-1.pgdg22.04+1))
SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, compression: off)
Type "help" for help.

testdb=>
``` 

15. Сделайте select * from t1;
``` sql
testdb=> select * from t1;
``` 

16. Получилось?
``` sql
Нет
``` 

17. Напишите что именно произошло
``` sql
ERROR:  permission denied for table t1
```

18. У вас есть идеи почему? ведь права то дали?
``` sql
Для роли readonly выданы гранты для работы с схемой testnm,

а t1 была создана под пользователем postgres в схеме public
``` 

19. Посмотрите на список таблиц
``` sql
testdb=> \dt
        List of relations
 Schema | Name | Type  |  Owner   
--------+------+-------+----------
 public | t1   | table | postgres
``` 

20. Schema public для t1  

21. Почему так получилось с таблицей
``` sql
Схема public создается по умолчанию при создании БД, 
если явно схему не указывать при создании таблицы, то таблица создается в public 
```

22. Вернитесь в базу данных testdb под пользователем postgres
``` sql
postgres=# \c testdb
You are now connected to database "testdb" as user "postgres".
testdb=# 
``` 

23. Удалите таблицу t1
``` sql
testdb=# drop table t1;
DROP TABLE
```

24. Создайте ее заново но уже с явным указанием имени схемы testnm
``` sql
testdb=# create table testnm.t1(c1 int);
CREATE TABLE
```

25. Вставьте строку со значением c1=1
``` sql
testdb=# insert into testnm.t1 (c1) values (1);
INSERT 0 1
```

26. Зайдите под пользователем testread в базу данных testdb
``` sql
user@postgres:~$ psql -U testread -h localhost -d testdb
Password for user testread: 
psql (15.5 (Ubuntu 15.5-1.pgdg22.04+1))
SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, compression: off)
Type "help" for help.

testdb=> 
```

27. Сделайте select * from testnm.t1;
``` sql
testdb=> select * from testnm.t1;
```

28. Получилось?
``` sql
Нет
ERROR:  permission denied for table t1
```

29. Причина
``` sql
Необходимо перевыдать гранты для новых таблиц
GRANT SELECT ON ALL TABLES in SCHEMA testnm TO readonly;
```

30. Как сделать так чтобы такое больше не повторялось?
``` sql
Использовать 
ALTER default privileges in SCHEMA testnm grant SELECT on TABLES to readonly; 
```



37. Теперь попробуйте выполнить команду create table t2(c1 integer); insert into t2 values (2);
``` sql
Не получилось создать таблицу, как указанно в инструкции. 
testdb=> create table t2(c1 integer);
ERROR:  permission denied for schema public

Причина в обновленной 15-й версии https://www.postgresql.org/docs/release/15.0/ 
для обычных пользователей нет гранта на создание (CREATE) объектов в public схеме 

Разрешаем создавать командой
testdb=# GRANT CREATE ON SCHEMA public TO readonly;
GRANT


Запрещаем
testdb=# REVOKE CREATE on SCHEMA public FROM readonly;
REVOKE

```
