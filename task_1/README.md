## Задание

1. Развернуть ВМ (Linux) с PostgreSQL (у вас есть ВМ в ВБ, любой другой способ, в т.ч.
докер)
2. Залить Тайские перевозки в минимальном варианте
https://github.com/aeuge/postgres16book/tree/main/database
3. Посчитать количество поездок - select count(*) from book.tickets;
4. Не забываем ВМ остановить/удалить

## Решение

``` bash
mahalichev@DESKTOP:~$ sudo -u mahalichev psql
psql (17.4 (Ubuntu 17.4-1.pgdg24.04+2))
Type "help" for help.

mahalichev=# \c thai
You are now connected to database "thai" as user "mahalichev".
thai=# select count(*) from book.tickets;
  count
---------
 5185505
(1 row)
```

## Ответ

5185505 записей
