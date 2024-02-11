Создал ВМ с Ubuntu 20.04/22.04 в VirtualBox  
![1](https://github.com/aneprincev/otus_my/blob/homework_03/3_1.png?raw=true)


Поставил Docker Engine  
![2](https://github.com/aneprincev/otus_my/blob/homework_03/3_2.png?raw=true)


Сделал каталог /var/lib/postgres 
![3](https://github.com/aneprincev/otus_my/blob/homework_03/3_3.png?raw=true)


Развернул контейнер с PostgreSQL 15 смонтировав в него /var/lib/postgresql 
``` sql
sudo docker network create pg-net  
sudo docker run --name pg-server --network pg-net -e POSTGRES_PASSWORD=postgres -d -p 5432:5432 -v /var/lib/postgres:/var/lib/postgresql/data postgres:15
```


Развернул контейнер с клиентом postgres,  
подключился из контейнера с клиентом к контейнеру с сервером и сделал таблицу с парой строк  
``` sql
sudo docker network create pg-net  
sudo docker run --name pg-server --network pg-net -e POSTGRES_PASSWORD=postgres -d -p 5432:5432 -v /var/lib/postgres:/var/lib/postgresql/data postgres:15  
sudo docker run -it --rm --network pg-net --name pg-client postgres:15 psql -h pg-server -U postgres  
create database otus;  
\c otus  
CREATE TABLE test (i serial, amount int);  
INSERT INTO test(amount) VALUES (111);  
INSERT INTO test(amount) VALUES (555);  
```
![4](https://github.com/aneprincev/otus_my/blob/homework_03/3_4.png?raw=true)


Подключится к контейнеру с сервером с ноутбука/компьютера извне инстансов GCP/ЯО/места установки докера 
- Реализовал
подключением из другого инстанса ВМ
![5](https://github.com/aneprincev/otus_my/blob/homework_03/3_5.png?raw=true)


Удалить контейнер с сервером, создать его заново, подключится снова из контейнера с клиентом к контейнеру с сервером и проверить, что данные остались на месте 
- Проверил, данные после удаления остались на хост машине, и после запуска контейнера с опцией -v стали доступными из контейнера
