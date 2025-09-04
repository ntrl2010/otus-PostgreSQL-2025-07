Установил postgres-16 на ноутбук i5 16GB RAM, ssd, Ubuntu 24.04
```
postgres@tpt480:~$ pg_lsclusters 
Ver Cluster Port Status Owner    Data directory              Log file
16  main    5432 online postgres /var/lib/postgresql/16/main /var/log/postgresql/postgresql-16-main.log

postgres@tpt480:~$ \psql
psql (16.9 (Ubuntu 16.9-0ubuntu0.24.04.1))
Type "help" for help.

postgres=# \conninfo
You are connected to database "postgres" as user "postgres" via socket in "/var/run/postgresql" at port "5432".
postgres=# \q
```

Инициализируем тестовую БД для утилиты pgbench и проводим первый тест 
```
postgres=# create database test;
CREATE DATABASE
postgres=# \q
postgres@tpt480:~$ pgbench -i test -h localhost
Password: 
dropping old tables...
NOTICE:  table "pgbench_accounts" does not exist, skipping
NOTICE:  table "pgbench_branches" does not exist, skipping
NOTICE:  table "pgbench_history" does not exist, skipping
NOTICE:  table "pgbench_tellers" does not exist, skipping
creating tables...
generating data (client-side)...
100000 of 100000 tuples (100%) done (elapsed 0.04 s, remaining 0.00 s)
vacuuming...
creating primary keys...
done in 0.22 s (drop tables 0.00 s, create tables 0.01 s, client-side generate 0.11 s, vacuum 0.05 s, primary keys 0.05 s).
postgres@tpt480:~$ 
postgres@tpt480:~$ pgbench -c 50 -j 2 -P 10 -T 60 test -h localhost
Password: 
pgbench (16.9 (Ubuntu 16.9-0ubuntu0.24.04.1))
starting vacuum...end.
progress: 10.0 s, 298.4 tps, lat 157.664 ms stddev 162.183, 0 failed
progress: 20.0 s, 315.2 tps, lat 158.089 ms stddev 176.247, 0 failed
progress: 30.0 s, 309.7 tps, lat 161.602 ms stddev 190.134, 0 failed
progress: 40.0 s, 310.8 tps, lat 160.789 ms stddev 186.905, 0 failed
progress: 50.0 s, 314.8 tps, lat 159.560 ms stddev 172.324, 0 failed
progress: 60.0 s, 330.2 tps, lat 151.408 ms stddev 169.684, 0 failed
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 50
number of threads: 2
maximum number of tries: 1
duration: 60 s
number of transactions actually processed: 18841
number of failed transactions: 0 (0.000%)
latency average = 158.344 ms
latency stddev = 176.609 ms
initial connection time = 419.828 ms
tps = 315.355498 (without initial connection time)
```
включаю ассинхронную запись на диск, перезапускаю постгрес и снова запускаю pgbench

```
postgres@tpt480:~$ \psql
psql (16.9 (Ubuntu 16.9-0ubuntu0.24.04.1))
Type "help" for help.

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
```
Значение TPS увеличилось в 10 раз, с 300 до 3000 попугайчиков

