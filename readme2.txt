postgres@tpt480:~$ pg_lsclusters 
Ver Cluster Port Status Owner    Data directory              Log file
16  main    5432 online postgres /var/lib/postgresql/16/main /var/log/postgresql/postgresql-16-main.log
postgres@tpt480:~$ \psql
psql (16.9 (Ubuntu 16.9-0ubuntu0.24.04.1))
Type "help" for help.

postgres=# alter system set synchronous_commin = off;
ERROR:  unrecognized configuration parameter "synchronous_commin"
postgres=# alter system set synchronous_commit = off;
ALTER SYSTEM
postgres=# select pg_reload_conf();
 pg_reload_conf 
----------------
 t
(1 row)

postgres=# \q
postgres@tpt480:~$ pgbench -c 50 -j 2 -P 10 -T 60 test -h localhost
Password: 
pgbench (16.9 (Ubuntu 16.9-0ubuntu0.24.04.1))
starting vacuum...end.
progress: 10.0 s, 2751.9 tps, lat 17.285 ms stddev 22.662, 0 failed
progress: 20.0 s, 3092.8 tps, lat 16.196 ms stddev 20.377, 0 failed
progress: 30.0 s, 3087.1 tps, lat 16.185 ms stddev 19.036, 0 failed
progress: 40.0 s, 3042.1 tps, lat 16.397 ms stddev 21.133, 0 failed
progress: 50.0 s, 3029.2 tps, lat 16.535 ms stddev 20.204, 0 failed
progress: 60.0 s, 3047.4 tps, lat 16.403 ms stddev 20.459, 0 failed
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 50
number of threads: 2
maximum number of tries: 1
duration: 60 s
number of transactions actually processed: 180555
number of failed transactions: 0 (0.000%)
latency average = 16.494 ms
latency stddev = 20.648 ms
initial connection time = 446.141 ms
tps = 3029.536432 (without initial connection time)
postgres@tpt480:~$ 
postgres@tpt480:~$ pgbench -c 50 -j 2 -P 10 -T 60 test -h localhost
Password: 
pgbench (16.9 (Ubuntu 16.9-0ubuntu0.24.04.1))
starting vacuum...end.
progress: 10.0 s, 2801.1 tps, lat 17.001 ms stddev 22.419, 0 failed
progress: 20.0 s, 2833.8 tps, lat 17.643 ms stddev 22.570, 0 failed
progress: 30.0 s, 2741.4 tps, lat 18.262 ms stddev 25.004, 0 failed
progress: 40.0 s, 2673.0 tps, lat 18.645 ms stddev 24.175, 0 failed
progress: 50.0 s, 2633.8 tps, lat 19.031 ms stddev 25.649, 0 failed
progress: 60.0 s, 2660.9 tps, lat 18.786 ms stddev 25.435, 0 failed
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 50
number of threads: 2
maximum number of tries: 1
duration: 60 s
number of transactions actually processed: 163490
number of failed transactions: 0 (0.000%)
latency average = 18.217 ms
latency stddev = 24.228 ms
initial connection time = 440.616 ms
tps = 2742.816702 (without initial connection time)
postgres@tpt480:~$ pgbench -i postgres
dropping old tables...
creating tables...
generating data (client-side)...
100000 of 100000 tuples (100%) done (elapsed 0.08 s, remaining 0.00 s)
vacuuming...
creating primary keys...
done in 0.20 s (drop tables 0.01 s, create tables 0.00 s, client-side generate 0.10 s, vacuum 0.04 s, primary keys 0.04 s).
postgres@tpt480:~$ pgbench -c8 -P 6 -T 60 -U postgres postgres
pgbench (16.9 (Ubuntu 16.9-0ubuntu0.24.04.1))
starting vacuum...end.
progress: 6.0 s, 6007.1 tps, lat 1.323 ms stddev 0.655, 0 failed
progress: 12.0 s, 5832.7 tps, lat 1.369 ms stddev 0.702, 0 failed
progress: 18.0 s, 5690.8 tps, lat 1.403 ms stddev 0.694, 0 failed
progress: 24.0 s, 5314.0 tps, lat 1.502 ms stddev 0.915, 0 failed
progress: 30.0 s, 5495.3 tps, lat 1.452 ms stddev 0.890, 0 failed
progress: 36.0 s, 4938.3 tps, lat 1.616 ms stddev 0.987, 0 failed
progress: 42.0 s, 5753.0 tps, lat 1.388 ms stddev 0.746, 0 failed
progress: 48.0 s, 5758.8 tps, lat 1.386 ms stddev 0.663, 0 failed
progress: 54.0 s, 5577.5 tps, lat 1.431 ms stddev 0.820, 0 failed
progress: 60.0 s, 5665.7 tps, lat 1.409 ms stddev 0.731, 0 failed
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 8
number of threads: 1
maximum number of tries: 1
duration: 60 s
number of transactions actually processed: 336208
number of failed transactions: 0 (0.000%)
latency average = 1.424 ms
latency stddev = 0.786 ms
initial connection time = 25.635 ms
tps = 5605.562680 (without initial connection time)
postgres@tpt480:~$ psql
psql (16.9 (Ubuntu 16.9-0ubuntu0.24.04.1))
Type "help" for help.

postgres=# SELECT txid_current();
CREATE TABLE test(i int);
INSERT INTO test VALUES (10),(20),(30);

select * from test;
 txid_current 
--------------
       699939
(1 row)

CREATE TABLE
INSERT 0 3
 i  
----
 10
 20
 30
(3 rows)

postgres=# SELECT i, xmin,xmax,cmin,cmax,ctid FROM test;
 i  |  xmin  | xmax | cmin | cmax | ctid  
----+--------+------+------+------+-------
 10 | 699941 |    0 |    0 |    0 | (0,1)
 20 | 699941 |    0 |    0 |    0 | (0,2)
 30 | 699941 |    0 |    0 |    0 | (0,3)
(3 rows)

postgres=# SELECT relname, n_live_tup, n_dead_tup, trunc(100*n_dead_tup/(n_live_tup+1))::float "ratio%", last_autovacuum 
FROM pg_stat_user_TABLEs WHERE relname = 'test';
 relname | n_live_tup | n_dead_tup | ratio% | last_autovacuum 
---------+------------+------------+--------+-----------------
 test    |          3 |          0 |      0 | 
(1 row)

postgres=# update test set i = 100 where i = 10;
UPDATE 1
postgres=# CREATE EXTENSION pageinspect;
ERROR:  extension "pageinspect" already exists
postgres=# \dx+
      Objects in extension "plpgsql"
            Object description             
-------------------------------------------
 function plpgsql_call_handler()
 function plpgsql_inline_handler(internal)
 function plpgsql_validator(oid)
 language plpgsql
(4 rows)

postgres=# \dt+
                                           List of relations
 Schema |       Name       | Type  |  Owner   | Persistence | Access method |    Size    | Description 
--------+------------------+-------+----------+-------------+---------------+------------+-------------
 public | accounts         | table | postgres | permanent   | heap          | 16 kB      | 
 public | pgbench_accounts | table | postgres | permanent   | heap          | 13 MB      | 
 public | pgbench_branches | table | postgres | permanent   | heap          | 1336 kB    | 
 public | pgbench_history  | table | postgres | permanent   | heap          | 17 MB      | 
 public | pgbench_tellers  | table | postgres | permanent   | heap          | 80 kB      | 
 public | test             | table | postgres | permanent   | heap          | 8192 bytes | 
(6 rows)

postgres=# SELECT lp as tuple, t_xmin, t_xmax, t_field3 as t_cid, t_ctid FROM heap_page_items(get_raw_page('test',0));
 tuple | t_xmin | t_xmax | t_cid | t_ctid 
-------+--------+--------+-------+--------
     1 | 699941 | 699942 |     0 | (0,4)
     2 | 699941 |      0 |     0 | (0,2)
     3 | 699941 |      0 |     0 | (0,3)
     4 | 699942 |      0 |     0 | (0,4)
(4 rows)

postgres=# SELECT * FROM heap_page_items(get_raw_page('test',0)) \gx
postgres=# \q
postgres@tpt480:~$ pgbench -c8 -P 6 -T 60 -U postgres postgres
pgbench (16.9 (Ubuntu 16.9-0ubuntu0.24.04.1))
starting vacuum...end.
progress: 6.0 s, 5946.8 tps, lat 1.337 ms stddev 0.625, 0 failed
progress: 12.0 s, 5940.7 tps, lat 1.344 ms stddev 0.694, 0 failed
progress: 18.0 s, 6068.2 tps, lat 1.316 ms stddev 0.661, 0 failed
progress: 24.0 s, 5946.5 tps, lat 1.343 ms stddev 0.674, 0 failed
progress: 30.0 s, 6054.9 tps, lat 1.319 ms stddev 0.642, 0 failed
progress: 36.0 s, 5830.3 tps, lat 1.369 ms stddev 0.696, 0 failed
progress: 42.0 s, 5454.8 tps, lat 1.464 ms stddev 0.790, 0 failed
progress: 48.0 s, 5634.8 tps, lat 1.417 ms stddev 0.738, 0 failed
progress: 54.0 s, 5899.2 tps, lat 1.353 ms stddev 0.674, 0 failed
progress: 60.0 s, 5694.0 tps, lat 1.402 ms stddev 0.689, 0 failed
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 8
number of threads: 1
maximum number of tries: 1
duration: 60 s
number of transactions actually processed: 350829
number of failed transactions: 0 (0.000%)
latency average = 1.365 ms
latency stddev = 0.690 ms
initial connection time = 25.359 ms
tps = 5848.986777 (without initial connection time)
postgres@tpt480:~$ 



Описание/Пошаговая инструкция выполнения домашнего задания:
Создать инстанс ВМ с 2 ядрами и 4 Гб ОЗУ и SSD 10GB
Установить на него PostgreSQL 15 с дефолтными настройками
Создать БД для тестов: выполнить pgbench -i postgres
Запустить pgbench -c8 -P 6 -T 60 -U postgres postgres
Применить параметры настройки PostgreSQL из прикрепленного к материалам занятия файла
Протестировать заново
Что изменилось и почему?
Создать таблицу с текстовым полем и заполнить случайными или сгенерированными данным в размере 1млн строк
Посмотреть размер файла с таблицей
5 раз обновить все строчки и добавить к каждой строчке любой символ
Посмотреть количество мертвых строчек в таблице и когда последний раз приходил автовакуум
Подождать некоторое время, проверяя, пришел ли автовакуум
5 раз обновить все строчки и добавить к каждой строчке любой символ
Посмотреть размер файла с таблицей
Отключить Автовакуум на конкретной таблице
10 раз обновить все строчки и добавить к каждой строчке любой символ
Посмотреть размер файла с таблицей
Объясните полученный результат
Не забудьте включить автовакуум)
