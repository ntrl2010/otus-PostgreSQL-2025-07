запуск кластера и проверка
```
postgres@tpt480:~$ pg_lsclusters 
Ver Cluster Port Status Owner    Data directory              Log file
16  main    5432 online postgres /var/lib/postgresql/16/main /var/log/postgresql/postgresql-16-main.log
postgres@tpt480:~$ \psql
psql (16.9 (Ubuntu 16.9-0ubuntu0.24.04.1))
Type "help" for help.
```
Запуск pgbench 

```
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
```
tps = 3029.536432  на дефолтных настройках
добавляем в конец /etc/postgresql/16/main/postgresql.conf
```
max_connections = 40
shared_buffers = 1GB
effective_cache_size = 3GB
maintenance_work_mem = 512MB
checkpoint_completion_target = 0.9
wal_buffers = 16MB
default_statistics_target = 500
random_page_cost = 4
effective_io_concurrency = 2
work_mem = 6553kB
min_wal_size = 4GB
max_wal_size = 16GB
```
рестартим  сервис и запускаем бенч

```
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
```
TPS стал 5600, поднялся почти в два раза.  Думаю, в основном, это изза увеличенных в объеме буферов

создаем табличку с миллионом строчек
```
postgres@tpt480:~$ 
postgres@tpt480:~$ psql
psql (16.9 (Ubuntu 16.9-0ubuntu0.24.04.1))
Type "help" for help.

postgres=# CREATE TABLE test(id serial, tname char(100) );
ERROR:  relation "test" already exists
postgres=# \dt+
                                           List of relations
 Schema |       Name       | Type  |  Owner   | Persistence | Access method |    Size    | Description 
--------+------------------+-------+----------+-------------+---------------+------------+-------------
 public | accounts         | table | postgres | permanent   | heap          | 16 kB      | 
 public | pgbench_accounts | table | postgres | permanent   | heap          | 14 MB      | 
 public | pgbench_branches | table | postgres | permanent   | heap          | 744 kB     | 
 public | pgbench_history  | table | postgres | permanent   | heap          | 18 MB      | 
 public | pgbench_tellers  | table | postgres | permanent   | heap          | 168 kB     | 
 public | test             | table | postgres | permanent   | heap          | 8192 bytes | 
(6 rows)

postgres=# drop table test;
DROP TABLE
postgres=# CREATE TABLE test(id serial, tname char(100) );
CREATE TABLE
postgres=# INSERT INTO test(tname) SELECT 'testdata' FROM generate_series(1,1000000);
INSERT 0 1000000
postgres=# SELECT pg_size_pretty(pg_total_relation_size('test'));
 pg_size_pretty 
----------------
 135 MB
(1 row)
```

обновляем 5 раз строчки с добавлением символа. 
Смотрим колво мертвых строчек и последний вакуум
```
postgres=# update test set tname=tname||'f';
UPDATE 1000000
postgres=# update test set tname=tname||'w';
UPDATE 1000000
postgres=# update test set tname=tname||'u';
UPDATE 1000000
postgres=# update test set tname=tname||'y';
UPDATE 1000000
postgres=# update test set tname=tname||'t';
UPDATE 1000000
postgres=# SELECT relname, n_live_tup, n_dead_tup, trunc(100*n_dead_tup/(n_live_tup+1))::float "ratio%", last_autovacuum
postgres-# ;
ERROR:  column "relname" does not exist
LINE 1: SELECT relname, n_live_tup, n_dead_tup, trunc(100*n_dead_tup...
               ^
postgres=# SELECT relname, n_live_tup, n_dead_tup, trunc(100*n_dead_tup/(n_live_tup+1))::float "ratio%", last_autovacuum FROM pg_stat_user_TABLEs WHERE relname = 'test';
 relname | n_live_tup | n_dead_tup | ratio% |        last_autovacuum        
---------+------------+------------+--------+-------------------------------
 test    |    1000000 |    1999953 |    199 | 2025-09-04 16:29:22.646063+03
(1 row)
```
ждем некорое время и снова смотрим тожесамое

```
postgres=# SELECT relname, n_live_tup, n_dead_tup, trunc(100*n_dead_tup/(n_live_tup+1))::float "ratio%", last_autovacuum FROM pg_stat_user_TABLEs WHERE relname = 'test';
 relname | n_live_tup | n_dead_tup | ratio% |        last_autovacuum        
---------+------------+------------+--------+-------------------------------
 test    |    1000000 |          0 |      0 | 2025-09-04 16:30:22.239907+03
(1 row)
```
мертвых строчек нет, все "съел вакуум"

Опять 5 раз обновляем строчки с добавлением символа
```
postgres=# update test set tname=tname||'q';
UPDATE 1000000
postgres=# update test set tname=tname||'w';
UPDATE 1000000
postgres=# update test set tname=tname||'e';
UPDATE 1000000
postgres=# update test set tname=tname||'r';
UPDATE 1000000
postgres=# update test set tname=tname||'t';
UPDATE 1000000
```

смотрим размер таблицы и отключаем автовакуум на ней
```
postgres=# SELECT relname, n_live_tup, n_dead_tup, trunc(100*n_dead_tup/(n_live_tup+1))::float "ratio%", last_autovacuum FROM pg_stat_user_TABLEs WHERE relname = 'test';
 relname | n_live_tup | n_dead_tup | ratio% |        last_autovacuum        
---------+------------+------------+--------+-------------------------------
 test    |    1000000 |    1000000 |     99 | 2025-09-04 16:30:22.239907+03
(1 row)

postgres=# SELECT pg_size_pretty(pg_total_relation_size('test'));
 pg_size_pretty 
----------------
 539 MB
(1 row)

postgres=# ALTER TABLE test SET (autovacuum_enabled = off);
ALTER TABLE
postgres=# update test set tname=tname||'h';
UPDATE 1000000
postgres=# SELECT pg_size_pretty(pg_total_relation_size('test'));
 pg_size_pretty 
----------------
 539 MB
(1 row)
```
опять добавляем строчки и смотрим размер
```
...
postgres=# update test set tname=tname||'a';
UPDATE 1000000
postgres=# SELECT pg_size_pretty(pg_total_relation_size('test'));
 pg_size_pretty 
----------------
 1078 MB
(1 row)
```

Файл с таблицей раздуло изза выключенного автовакуума

