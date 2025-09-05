запустили кластер постгрес-16

```
psql (16.9 (Ubuntu 16.9-0ubuntu0.24.04.1))
SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, compression: off)
Type "help" for help.

postgres=# \c
SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, compression: off)
You are now connected to database "postgres" as user "postgres".
postgres=# 
```
создаем новую БД testdb и переходим в нее
```
postgres=# ﻿CREATE DATABASE testdb;
CREATE DATABASE
postgres=# \c testdb
SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, compression: off)
You are now connected to database "testdb" as user "postgres".
```
создаем схему, таблицу и добавляем данные
```
testdb=#testdb=# CREATE SCHEMA testnm;
CREATE SCHEMA
testdb=# CREATE TABLE t1(c1 integer);
CREATE TABLE
testdb=# INSERT INTO t1 values(1);
INSERT 0 1
```
Добавляем роль и назначаем гранты
```
testdb=# CREATE role readonly;
CREATE ROLE
testdb=# grant connect on DATABASE testdb TO readonly;
GRANT
testdb=# grant usage on SCHEMA testnm to readonly;
GRANT
testdb=# grant SELECT on all TABLEs in SCHEMA testnm TO readonly;
GRANT
testdb=# CREATE USER testread with password 'test123';
CREATE ROLE
testdb=# grant readonly TO testread;
GRANT ROLE
```
переключаемся в нового пользователя и пытаемся прочитать из таблицы
```
testdb=# \c testdb testread
Password for user testread: 
connection to server at "localhost" (127.0.0.1), port 5432 failed: FATAL:  password authentication failed for user "testread"
connection to server at "localhost" (127.0.0.1), port 5432 failed: FATAL:  password authentication failed for user "testread"
Previous connection kept
testdb=# \c testdb testread
Password for user testread: 
SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, compression: off)
You are now connected to database "testdb" as user "testread".
testdb=> \c
SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, compression: off)
You are now connected to database "testdb" as user "testread".
testdb=> SELECT * FROM t1;
ERROR:  permission denied for table t1 
```
Доступ запрещен, тк таблицу по умолчанию мы создали в схеме паблик, а у этого юзера нет прав на нее

Пересоздаю таблицу под пользователем postgres с указанием схемы
```
DROP TABLE
testdb=# CREATE TABLE testnm.t1(c1 integer);
CREATE TABLE
```

Во второй сессии подключаюсь пользователем testread к БД testdb
и читаем данные
```
testdb=> \c
SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, compression: off)
You are now connected to database "testdb" as user "testread".

testdb=> SELECT * FROM testnm.t1;
 c1 
----
(0 rows)
```
пробуем создать вторую табличку
```
testdb=> create table t2(c1 integer); 
ERROR:  permission denied for schema public
LINE 1: create table t2(c1 integer);
```
Отказано, тк прав на созданием табличек у пользователя нет.

Переходим в другую сессию с пользователем postgres и создаем новую роль с возможностью создания объектов,
добавляем эту новую роль пользователю
```
testdb=# CREATE role role2;
CREATE ROLE
testdb=# grant create on SCHEMA testnm TO role2;
GRANT
testdb=# grant role2 TO testread;
GRANT ROLE
```
создаем пользователем testread другую таблицу и вставляем строчку
```
testdb=> create table testnm.t2(c1 integer); 
CREATE TABLE
testdb=> insert into testnm.t2 values (2);
INSERT 0 1
```

