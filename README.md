Выключаем auto commit и проверяем  
``` sql
postgres=# \set AUTOCOMMIT OFF
postgres=# \echo :AUTOCOMMIT
OFF
```
Создаем и наполняем таблицу 
``` sql
create table persons(id serial, first_name text, second_name text);   
insert into persons(first_name, second_name) values('ivan', 'ivanov');  
insert into persons(first_name, second_name) values('petr', 'petrov');  
commit;
```
Проверяем запросом результат
``` sql
postgres=# select * from persons;
 id | first_name | second_name 
----+------------+-------------
  1 | ivan       | ivanov
  2 | petr       | petrov
(2 rows)
```

Проверяем текущий уровень изоляции
``` sql
postgres=# show transaction isolation level;
 transaction_isolation 
-----------------------
 read committed
(1 row)
```

Начинаем новую транзакцию в двух сессиях, уровень изоляции дэфолтный - read committed (не меняем).

В первой сессии добавляем новую запись 
``` sql
insert into persons(first_name, second_name) values('sergey', 'sergeev');
```
Во второй сессии выполняем запрос
``` sql
select * from persons;
```

Видим результат
``` sql
postgres=# select * from persons;
 id | first_name | second_name 
----+------------+-------------
  1 | ivan       | ivanov
  2 | petr       | petrov
(2 rows)
```
Новая запись не отображается, т.к. установлен дефолтный уровень изоляции read committed и отключен автокомит.

Вставка первой транзакцией ещё не зафиксирована,    
фиксируем её 
``` sql
commit;
```
Повторно проверяем во второй транзакции
``` sql
select * from persons;
```

Результат
``` sql
postgres=*# select * from persons;
 id | first_name | second_name 
----+------------+-------------
  1 | ivan       | ivanov
  2 | petr       | petrov
  4 | sergey     | sergeev
(3 rows)
```
Новая запись отображается по причине того, что в первой транзакции был commit - данные зафиксированы.

Фиксируем вторую транзакцию
``` sql
commit;
```
Устанавливаем в сессиях уровень изоляции транзакций repeatable read и проверяем запросом 
``` sql
postgres=# set transaction isolation level repeatable read;
SET
postgres=*# show transaction isolation level;
 transaction_isolation 
-----------------------
 repeatable read
(1 row)
```
В первой сессии добавляем новую запись 
``` sql
postgres=*# insert into persons(first_name, second_name) values('sveta', 'svetova');
INSERT 0 1
```

Во второй сессии выполняем запрос
``` sql
postgres=*# select * from persons;
 id | first_name | second_name 
----+------------+-------------
  1 | ivan       | ivanov
  2 | petr       | petrov
  4 | sergey     | sergeev
(3 rows)
```
Вставленные данные не отображаются, т.к. в первой сессии не было фиксации 
фиксируем вставку в первой сессии
``` sql
commit;
```

Во второй сессии повторно выполняем запрос
``` sql
postgres=*# select * from persons;
 id | first_name | second_name 
----+------------+-------------
  1 | ivan       | ivanov
  2 | petr       | petrov
  4 | sergey     | sergeev
(3 rows)
```
Вставленные данные не отображаются, даже при фиксации транзакции в первой сессии.  

Вторая транзакция видит данные (снимок данных) на момент своего начала  
и не ввидит остальные завершенные изменения, сделанные в других сессиях в этот момент (фантомное чтение).  

Неповторяющее чтение так же не работает - сколько бы не читали данные - реезульт будет тот же.
 

Фиксируем транзакцию во второй сессии и видим изменения
``` sql
postgres=*# commit;
COMMIT
postgres=# select * from persons;
 id | first_name | second_name 
----+------------+-------------
  1 | ivan       | ivanov
  2 | petr       | petrov
  4 | sergey     | sergeev
  5 | sveta      | svetova
(4 rows)
```
