установил на виртуальную машину ubuntu 24.04.
в ее репозиториях нет постгреса-15.  Пришлось ставить 15 вирсию из официального postgresql.org репа.
Было предупреждение, что 15 версия устарела.
Почсле установки 15ой версии, создал кластер main 15.Также создал табличку и добавил в нее данные.


>перенесите содержимое /var/lib/postgres/15 в /mnt/data - mv /var/lib/postgresql/15/mnt/data
>попытайтесь запустить кластер - sudo -u postgres pg_ctlcluster 15 main start
>напишите получилось или нет и почему

после переноса файлов инстанса (15 main) постгреса, он перестал запускаться 

```
root@otus1:/home/serge# pg_ctlcluster  15 main start
Error: /var/lib/postgresql/15/main is not accessible or does not exist
```

в конфигурационном файле 
/etc/postgresql/15/main/postgresql.conf:
параметре data_directory меняем путь к папке с данными  на актуальный

data_directory = '/mnt/data/15/main'


root@otus1:/home/serge# su - postgres
postgres@otus1:~$ psql  -p 5433
psql (16.9 (Ubuntu 16.9-0ubuntu0.24.04.1), server 15.14 (Ubuntu 15.14-1.pgdg24.04+1))
Type "help" for help.

postgres=# \dt
        List of relations
 Schema | Name | Type  |  Owner   
--------+------+-------+----------
 public | test | table | postgres
(1 row)

postgres=# select * from test;
 c1 
----
 1
(1 row)

