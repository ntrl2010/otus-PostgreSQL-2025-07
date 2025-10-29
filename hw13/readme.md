подключаемся к рабочему ПГ-кластеру 
```
[sudo] password for serge: 
Ver Cluster Port Status Owner    Data directory               Log file
16  main    5432 online postgres /var/lib/postgresql/16/main  /var/log/postgresql/postgresql-16-main.log
16  main2   5433 online postgres /var/lib/postgresql/16/main2 /var/log/postgresql/postgresql-16-main2.log
serge@tpt480:~$ psql -h localhost -U postgres 
Password for user postgres: 
psql (16.9 (Ubuntu 16.9-0ubuntu0.24.04.1))
SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, compression: off)
Type "help" for help.
```



Создаем новую БД hw13 и табличку с 100 записями


```
postgres=# create database hw13;
CREATE DATABASE
postgres=# \c hw13
SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, compression: off)
You are now connected to database "hw13" as user "postgres".
hw13=# \dt
Did not find any relations.
hw13=# \dt+
Did not find any relations.
hw13=# create table student as select generate_series(1,100) as id, md5(random()::text)::char(10) as fio;
SELECT 100
hw13=# select * from student limit 10
hw13-# ;
 id |    fio     
----+------------
  1 | 4c99e380a3
  2 | 603dc17ffe
  3 | 3b78b8ee3b
  4 | e1e5c382f7
  5 | e759de4101
  6 | f250eeef88
  7 | 3d8e8a9d6c
  8 | a35fae13db
  9 | d63a933ce5
 10 | e685157f9b
(10 rows)
```

выхожу из psql, переключаю unix-пользователя  на postgres, создаю каталог для бекапа, возвращаемся в БД

```
hw13=# \q
serge@tpt480:~$ sudo  su - postgres
postgres@tpt480:~$ mkdir /tmp/backup1
postgres@tpt480:~$ 
logout
serge@tpt480:~$ psql -h localhost -U postgres 
Password for user postgres: 
psql (16.9 (Ubuntu 16.9-0ubuntu0.24.04.1))
SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, compression: off)
Type "help" for help.

postgres=# \c hw13
SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, compression: off)
You are now connected to database "hw13" as user "postgres".
hw13=# \c
SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, compression: off)
You are now connected to database "hw13" as user "postgres".
```


попытка создать бекап таблички, не проходит, тк права нужны от юзера, запустившего psql.
```
hw13=# \copy student to '/tmp/backup1/table1.csv' with delimiter ',' 
/tmp/backup1/table1.csv: Permission denied
hw13=# \copy student to '/tmp/table1.csv' with delimiter ',' 
COPY 100
hw13=# \copy student to '/tmp/backup2/table1.csv' with delimiter ',' 
COPY 100
```

создаем вторую табличку и наливаем в нее данные из предыдущего бекапа.
```
hw13=# create table student2 ( id integer , fio char(10));
CREATE TABLE
hw13=# \copy student2 from /tmp/backup2/table1.csv with delimiter ','
```

данные совпадают
```

hw13=# select * from student limit 10;
 id |    fio     
----+------------
  1 | 4c99e380a3
  2 | 603dc17ffe
  3 | 3b78b8ee3b
  4 | e1e5c382f7
  5 | e759de4101
  6 | f250eeef88
  7 | 3d8e8a9d6c
  8 | a35fae13db
  9 | d63a933ce5
 10 | e685157f9b
(10 rows)

hw13=# select * from student2 limit 10;
 id |    fio     
----+------------
  1 | 4c99e380a3
  2 | 603dc17ffe
  3 | 3b78b8ee3b
  4 | e1e5c382f7
  5 | e759de4101
  6 | f250eeef88
  7 | 3d8e8a9d6c
  8 | a35fae13db
  9 | d63a933ce5
 10 | e685157f9b
(10 rows)
```

создаем бинарный дамп БД
```

serge@tpt480:~$ pg_dump -d hw13 --create -U postgres -h localhost -Fc  > /tmp/dump_hw13.gz
Password: 
serge@tpt480:~$ ls -l /tmp/dump_hw13.gz 
-rw-rw-r-- 1 serge serge 4890 окт 29 14:04 /tmp/dump_hw13.gz
serge@tpt480:~$ file /tmp/dump_hw13.gz 
/tmp/dump_hw13.gz: PostgreSQL custom database dump - v1.15-0
```


восстанавливаем бекап из бинарного дампа в новую БД и только одну таблицу
```
serge@tpt480:~$ psql -h localhost -U postgres 
Password for user postgres: 
psql (16.9 (Ubuntu 16.9-0ubuntu0.24.04.1))
SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, compression: off)
Type "help" for help.

postgres=# \c hw13b
SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, compression: off)
You are now connected to database "hw13b" as user "postgres".
hw13b=# \dt+
                                       List of relations
 Schema |   Name   | Type  |  Owner   | Persistence | Access method |    Size    | Description 
--------+----------+-------+----------+-------------+---------------+------------+-------------
 public | student2 | table | postgres | permanent   | heap          | 8192 bytes | 
(1 row)

hw13b=# select * from student2 limit 10;
 id |    fio     
----+------------
  1 | 4c99e380a3
  2 | 603dc17ffe
  3 | 3b78b8ee3b
  4 | e1e5c382f7
  5 | e759de4101
  6 | f250eeef88
  7 | 3d8e8a9d6c
  8 | a35fae13db
  9 | d63a933ce5
 10 | e685157f9b
(10 rows)
```




