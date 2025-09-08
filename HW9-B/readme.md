Описание/Пошаговая инструкция выполнения домашнего задания:

Настройте выполнение контрольной точки раз в 30 секунд.

```
postgres=# show checkpoint_timeout;
 checkpoint_timeout 
--------------------
 30s
(1 row)
```

10 минут c помощью утилиты pgbench подавайте нагрузку.
```
postgres@tpt480:~$ pgbench -c8 -P 6 -T 600 -U postgres postgres
pgbench (16.9 (Ubuntu 16.9-0ubuntu0.24.04.1))
starting vacuum...end.
progress: 6.0 s, 6085.3 tps, lat 1.307 ms stddev 0.574, 0 failed
progress: 12.0 s, 6025.3 tps, lat 1.325 ms stddev 0.596, 0 failed
progress: 18.0 s, 6090.2 tps, lat 1.311 ms stddev 0.578, 0 failed
progress: 24.0 s, 6070.5 tps, lat 1.315 ms stddev 0.597, 0 failed
progress: 30.0 s, 6033.8 tps, lat 1.323 ms stddev 0.592, 0 failed
progress: 36.0 s, 5992.7 tps, lat 1.332 ms stddev 0.598, 0 failed
progress: 42.0 s, 6022.8 tps, lat 1.326 ms stddev 0.602, 0 failed
progress: 48.0 s, 5937.1 tps, lat 1.345 ms stddev 0.699, 0 failed
progress: 54.0 s, 5966.7 tps, lat 1.338 ms stddev 0.622, 0 failed
progress: 60.0 s, 6004.0 tps, lat 1.330 ms stddev 0.585, 0 failed
progress: 66.0 s, 5935.7 tps, lat 1.345 ms stddev 0.591, 0 failed
progress: 72.0 s, 6000.4 tps, lat 1.331 ms stddev 0.590, 0 failed
progress: 78.0 s, 6029.3 tps, lat 1.324 ms stddev 0.578, 0 failed
progress: 84.0 s, 6075.5 tps, lat 1.314 ms stddev 0.574, 0 failed
progress: 90.0 s, 5933.7 tps, lat 1.346 ms stddev 0.597, 0 failed
progress: 96.0 s, 5833.0 tps, lat 1.369 ms stddev 0.628, 0 failed
progress: 102.0 s, 5862.6 tps, lat 1.362 ms stddev 0.610, 0 failed
progress: 108.0 s, 5815.7 tps, lat 1.373 ms stddev 0.709, 0 failed
progress: 114.0 s, 5636.2 tps, lat 1.417 ms stddev 0.634, 0 failed
progress: 120.0 s, 5078.0 tps, lat 1.572 ms stddev 0.694, 0 failed
progress: 126.0 s, 5088.2 tps, lat 1.569 ms stddev 0.693, 0 failed
progress: 132.0 s, 5099.0 tps, lat 1.566 ms stddev 0.698, 0 failed
progress: 138.0 s, 5123.2 tps, lat 1.558 ms stddev 0.689, 0 failed
progress: 144.0 s, 5118.7 tps, lat 1.560 ms stddev 0.695, 0 failed
progress: 150.0 s, 5077.8 tps, lat 1.572 ms stddev 0.704, 0 failed
progress: 156.0 s, 5018.3 tps, lat 1.591 ms stddev 0.718, 0 failed
progress: 162.0 s, 5186.7 tps, lat 1.539 ms stddev 0.768, 0 failed
progress: 168.0 s, 5860.3 tps, lat 1.362 ms stddev 0.741, 0 failed
progress: 174.0 s, 5915.5 tps, lat 1.350 ms stddev 0.605, 0 failed
progress: 180.0 s, 5928.1 tps, lat 1.347 ms stddev 0.598, 0 failed
progress: 186.0 s, 5879.3 tps, lat 1.358 ms stddev 0.608, 0 failed
progress: 192.0 s, 5764.0 tps, lat 1.385 ms stddev 0.627, 0 failed
progress: 198.0 s, 5106.3 tps, lat 1.564 ms stddev 0.696, 0 failed
progress: 204.0 s, 5085.4 tps, lat 1.570 ms stddev 0.698, 0 failed
progress: 210.0 s, 5087.5 tps, lat 1.569 ms stddev 0.704, 0 failed
progress: 216.0 s, 5067.3 tps, lat 1.575 ms stddev 0.743, 0 failed
progress: 222.0 s, 4953.4 tps, lat 1.612 ms stddev 0.763, 0 failed
progress: 228.0 s, 5020.8 tps, lat 1.590 ms stddev 0.832, 0 failed
progress: 234.0 s, 5129.0 tps, lat 1.557 ms stddev 0.696, 0 failed
progress: 240.0 s, 5145.7 tps, lat 1.552 ms stddev 0.691, 0 failed
progress: 246.0 s, 5958.0 tps, lat 1.340 ms stddev 0.596, 0 failed
progress: 252.0 s, 5944.3 tps, lat 1.343 ms stddev 0.600, 0 failed
progress: 258.0 s, 5529.0 tps, lat 1.444 ms stddev 0.652, 0 failed
progress: 264.0 s, 5075.7 tps, lat 1.573 ms stddev 0.690, 0 failed
progress: 270.0 s, 5060.5 tps, lat 1.578 ms stddev 0.715, 0 failed
progress: 276.0 s, 5001.3 tps, lat 1.596 ms stddev 0.709, 0 failed
progress: 282.0 s, 5107.6 tps, lat 1.563 ms stddev 0.700, 0 failed
progress: 288.0 s, 5062.9 tps, lat 1.577 ms stddev 0.800, 0 failed
progress: 294.0 s, 5129.5 tps, lat 1.556 ms stddev 0.686, 0 failed
progress: 300.0 s, 5138.8 tps, lat 1.554 ms stddev 0.689, 0 failed
progress: 306.0 s, 5105.8 tps, lat 1.564 ms stddev 0.698, 0 failed
progress: 312.0 s, 5120.5 tps, lat 1.559 ms stddev 0.681, 0 failed
progress: 318.0 s, 5154.6 tps, lat 1.549 ms stddev 0.687, 0 failed
progress: 324.0 s, 5094.0 tps, lat 1.567 ms stddev 0.739, 0 failed
progress: 330.0 s, 5117.8 tps, lat 1.560 ms stddev 0.698, 0 failed
progress: 336.0 s, 4966.2 tps, lat 1.608 ms stddev 0.735, 0 failed
progress: 342.0 s, 5062.7 tps, lat 1.577 ms stddev 0.710, 0 failed
progress: 348.0 s, 5113.6 tps, lat 1.561 ms stddev 0.688, 0 failed
progress: 354.0 s, 5038.6 tps, lat 1.585 ms stddev 0.796, 0 failed
progress: 360.0 s, 5109.8 tps, lat 1.562 ms stddev 0.695, 0 failed
progress: 366.0 s, 5091.6 tps, lat 1.568 ms stddev 0.707, 0 failed
progress: 372.0 s, 5102.3 tps, lat 1.565 ms stddev 0.692, 0 failed
progress: 378.0 s, 5131.5 tps, lat 1.556 ms stddev 0.688, 0 failed
progress: 384.0 s, 5102.0 tps, lat 1.565 ms stddev 0.695, 0 failed
progress: 390.0 s, 5841.8 tps, lat 1.367 ms stddev 0.607, 0 failed
progress: 396.0 s, 5900.7 tps, lat 1.353 ms stddev 0.614, 0 failed
progress: 402.0 s, 5474.8 tps, lat 1.458 ms stddev 0.700, 0 failed
progress: 408.0 s, 5118.2 tps, lat 1.560 ms stddev 0.688, 0 failed
progress: 414.0 s, 5032.0 tps, lat 1.587 ms stddev 0.794, 0 failed
progress: 420.0 s, 5118.5 tps, lat 1.560 ms stddev 0.693, 0 failed
progress: 426.0 s, 5069.0 tps, lat 1.575 ms stddev 0.749, 0 failed
progress: 432.0 s, 5132.2 tps, lat 1.556 ms stddev 0.693, 0 failed
progress: 438.0 s, 5133.2 tps, lat 1.555 ms stddev 0.687, 0 failed
progress: 444.0 s, 5126.3 tps, lat 1.558 ms stddev 0.698, 0 failed
progress: 450.0 s, 5070.7 tps, lat 1.575 ms stddev 0.703, 0 failed
progress: 456.0 s, 4416.5 tps, lat 1.808 ms stddev 0.860, 0 failed
progress: 462.0 s, 4189.5 tps, lat 1.906 ms stddev 0.866, 0 failed
progress: 468.0 s, 4957.0 tps, lat 1.611 ms stddev 0.723, 0 failed
progress: 474.0 s, 4976.0 tps, lat 1.604 ms stddev 0.862, 0 failed
progress: 480.0 s, 5086.0 tps, lat 1.570 ms stddev 0.704, 0 failed
progress: 486.0 s, 5153.7 tps, lat 1.549 ms stddev 0.680, 0 failed
progress: 492.0 s, 5129.7 tps, lat 1.557 ms stddev 0.691, 0 failed
progress: 498.0 s, 4854.5 tps, lat 1.645 ms stddev 0.746, 0 failed
progress: 504.0 s, 4232.2 tps, lat 1.887 ms stddev 0.843, 0 failed
progress: 510.0 s, 4263.0 tps, lat 1.873 ms stddev 0.828, 0 failed
progress: 516.0 s, 4736.2 tps, lat 1.686 ms stddev 0.761, 0 failed
progress: 522.0 s, 4873.5 tps, lat 1.638 ms stddev 0.806, 0 failed
progress: 528.0 s, 5143.8 tps, lat 1.552 ms stddev 0.680, 0 failed
progress: 534.0 s, 5068.2 tps, lat 1.575 ms stddev 0.788, 0 failed
progress: 540.0 s, 5086.2 tps, lat 1.570 ms stddev 0.728, 0 failed
progress: 546.0 s, 5095.5 tps, lat 1.567 ms stddev 0.696, 0 failed
progress: 552.0 s, 5113.9 tps, lat 1.561 ms stddev 0.688, 0 failed
progress: 558.0 s, 5152.3 tps, lat 1.550 ms stddev 0.675, 0 failed
progress: 564.0 s, 5139.9 tps, lat 1.553 ms stddev 0.679, 0 failed
progress: 570.0 s, 5081.0 tps, lat 1.571 ms stddev 0.714, 0 failed
progress: 576.0 s, 4984.7 tps, lat 1.602 ms stddev 0.712, 0 failed
progress: 582.0 s, 4117.3 tps, lat 1.939 ms stddev 0.893, 0 failed
progress: 588.0 s, 4337.8 tps, lat 1.841 ms stddev 0.826, 0 failed
progress: 594.0 s, 4669.0 tps, lat 1.710 ms stddev 0.927, 0 failed
progress: 600.0 s, 5133.8 tps, lat 1.555 ms stddev 0.697, 0 failed
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 8
number of threads: 1
maximum number of tries: 1
duration: 600 s
number of transactions actually processed: 3164469
number of failed transactions: 0 (0.000%)
latency average = 1.514 ms
latency stddev = 0.708 ms
initial connection time = 23.190 ms
tps = 5274.235472 (without initial connection time)
```

Измерьте, какой объем журнальных файлов был сгенерирован за это время. Оцените, какой объем приходится в среднем на одну контрольную точку.

```
root@tpt480:/home/serge# du -sh /var/lib/postgresql/16/main/pg_wal
3,3G	/var/lib/postgresql/16/main/pg_wal
```

Проверьте данные статистики: все ли контрольные точки выполнялись точно по расписанию. Почему так произошло?
Сравните tps в синхронном/асинхронном режиме утилитой pgbench. Объясните полученный результат.

```
postgres=# alter system set synchronous_commit = on;
ALTER SYSTEM
postgres=# select pg_reload_conf();
 pg_reload_conf 
----------------
 t
(1 row)
```
результат: initial connection time = 21.729 ms
tps = 328.739265 (without initial connection time)

в ассинхронном режиме TPS сильно выше, тк снижена нагрузка на дисковую систему.


Создайте новый кластер с включенной контрольной суммой страниц. Создайте таблицу. Вставьте несколько значений. Выключите кластер. Измените пару байт в таблице. Включите кластер и сделайте выборку из таблицы. Что и почему произошло? как проигнорировать ошибку и продолжить работу?

на 16ом постгресе из репов убунты
```
root@tpt480:/var/lib/postgresql/16/main/base/5# pg_createcluster 16 main2 --data-checksums
Unknown option: data-checksums

```

а если всетаки влезть в кишки таблицы и изменить байты, то нужно выполнить команду

```
alter system set ignore_checksum_failure = on;
```
