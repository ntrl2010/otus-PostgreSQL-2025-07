проверяем текущее значение таймаута
```
ostgres=# show deadlock_timeout;
 deadlock_timeout 
------------------
 1s
(1 row)
```

устанавливаем таймаут в 200мс и включаем логгирование
```
postgres=# show deadlock_timeout;
 deadlock_timeout 
------------------
 200ms
(1 row)
postgres=# ALTER SYSTEM SET log_lock_waits = on;
ALTER SYSTEM

```

применяем измененные настройки и проверяем текущее значение
```
postgres=# select pg_reload_conf();
 pg_reload_conf 
----------------
 t
(1 row)

postgres=# show deadlock_timeout;
 deadlock_timeout 
------------------
 200ms
(1 row)
postgres=# show log_lock_waits;
 log_lock_waits 
----------------
 on
(1 row)
```

смоделировал запрос на создание индексов, при незаконченной транзакции в другой сессии и в логе появилась строчки
```
2025-09-02 14:00:28.880 MSK [34206] postgres@testlock LOG:  process 34206 still waiting for ShareLock on relation 16454 of database 16388 after 200.207 ms
2025-09-02 14:00:28.880 MSK [34206] postgres@testlock DETAIL:  Process holding the lock: 32942. Wait queue: 34206.
2025-09-02 14:00:28.880 MSK [34206] postgres@testlock STATEMENT:  CREATE INDEX ON accounts(acc_no);
```

моделирование 3х команд UPDATE в разных сеансах. Вывод pg_locks
```
testlock=*# SELECT locktype, relation::REGCLASS, mode, granted, pid, pg_blocking_pids(pid) AS wait_for
FROM pg_locks WHERE relation = 'accounts'::regclass order by pid;
 locktype | relation |       mode       | granted |  pid  | wait_for 
----------+----------+------------------+---------+-------+----------
 relation | accounts | RowExclusiveLock | t       | 34206 | {36541}
 tuple    | accounts | ExclusiveLock    | t       | 34206 | {36541}
 relation | accounts | RowExclusiveLock | t       | 36541 | {}
```
процесс 34206 ждет 36541, 
RowExclusiveLock  
ExclusiveLock

Моделирование взаимоблокировки, на примере update одной и тойже строки в нескольких сессиях.
```
testlock=*# UPDATE accounts SET amount = amount - 10.00 WHERE acc_no = 2;
ERROR:  deadlock detected
DETAIL:  Process 36541 waits for ExclusiveLock on tuple (0,2) of relation 16454 of database 16388; blocked by process 34206.
Process 34206 waits for ShareLock on transaction 760; blocked by process 32942.
Process 32942 waits for ShareLock on transaction 759; blocked by process 36541.
HINT:  See server log for query details.
```
