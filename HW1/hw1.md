

> посмотреть текущий уровень изоляции: show transaction isolation level
> начать новую транзакцию в обоих сессиях с дефолтным (не меняя) уровнем изоляции
> в первой сессии добавить новую запись insert into persons(first_name, second_name) values('sergey', 'sergeev');
> сделать select from persons во второй сессии
> видите ли вы новую запись и если да то почему?

Не видно, тк  в первой сессии транзакция еще не закоммителась. Сработала защита-изоляция клиентов.

первая сессия
``` 
BEGIN
postgres=*# 
postgres=*# insert into persons(first_name, last_name) values('sergey2', 'sergeev');
INSERT 0 1
```
вторая сессия:
```
​￼postgres=# select * from persons;
 id | first_name |  last_name   
​￼----+------------+--------------
  1 | Ivan       | Ivanov
  2 | Petr       | Updated11:38
  3 | Anna       | Changed
  4 | sergey     | sergeev
(4 rows)

postgres=# 
```
до коммита.
И после коммита:
```
​￼postgres=# select * from persons;
 id | first_name |  last_name   
​￼----+------------+--------------
  1 | Ivan       | Ivanov
  2 | Petr       | Updated11:38
  3 | Anna       | Changed
  4 | sergey     | sergeev
  5 | sergey2    | sergeev
(5 rows)

```

---
> начать новые но уже repeatable read транзации - set transaction isolation level repeatable read;
> в первой сессии добавить новую запись insert into persons(first_name, second_name) values('sveta', 'svetova');
> сделать select* from persons во второй сессии*
> видите ли вы новую запись и если да то почему?

не видно, новая запись  еще не закоммителась в первой сессии. 

>завершить первую транзакцию - commit;
>сделать select from persons во второй сессии
>видите ли вы новую запись и если да то почему?

не видно, тк вторая сессия находится еще в "изолированном" состоянии (начало второй транзакциии во второй сессии, которая еще не завершилась)

>завершить вторую транзакцию
>сделать select * from persons во второй сессии
>видите ли вы новую запись и если да то почему?

теперь изменения видны (появилась добавленная строчка)
