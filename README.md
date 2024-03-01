Установил ВМ с Ubuntu 22.04  

![1](https://github.com/aneprincev/otus_my/blob/homework_06/6_1.png?raw=true)  
  
Установил PostgreSQL 15, создал таблицу с данными 

![2](https://github.com/aneprincev/otus_my/blob/homework_06/6_2.png?raw=true)  
  
Создал vhd на 10 GB, подключил к VM 

![3](https://github.com/aneprincev/otus_my/blob/homework_06/6_3.png?raw=true)  
  
Разметил, выбрал файловую систему и примонтировал по инструкции:  
https://www.digitalocean.com/community/tutorials/how-to-partition-and-format-storage-devices-in-linux#step-5-mount-the-new-filesystem

![4](https://github.com/aneprincev/otus_my/blob/homework_06/6_4.png?raw=true)  
  
Перезагрузил систему, диск остался примонтированным  
![5](https://github.com/aneprincev/otus_my/blob/homework_06/6_5.png?raw=true)  
  
Выполнил команду mv /var/lib/postgresql/15 /mnt/data,  
запустил кластер и получил ошибку:

``` sql
user@postgres1:~$ sudo -u postgres pg_ctlcluster 15 main start  
Error: /var/lib/postgresql/15/main is not accessible or does not exist    
``` 

Исправил ошибку, указав новый каталог, где будут храниться данные. 

Путем изменения файла конфига /etc/postgresql/15/main/postgresql.conf   

``` sql
#data_directory = '/var/lib/postgresql/15/main'         
#use data in another directory
data_directory = '/mnt/data/15/main'
```

![6](https://github.com/aneprincev/otus_my/blob/homework_06/6_6.png?raw=true)  
  
В psql проверил наличие ранее созднанной таблицы

![7](https://github.com/aneprincev/otus_my/blob/homework_06/6_7.png?raw=true) 
