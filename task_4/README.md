## Задание

1. Создать таблицу accounts(id integer, amount numeric);
2. Добавить несколько записей и подключившись через 2 терминала добиться ситуации взаимоблокировки (deadlock).
3. Посмотреть логи и убедиться, что информация о дедлоке туда попала.

## Решение

``` sql
# 1. Создать таблицу accounts(id integer, amount numeric);

postgres=# \c wb_psql
You are now connected to database "wb_psql" as user "postgres".
wb_psql=# CREATE TABLE task4.accounts (id integer, amount numeric);
CREATE TABLE

# 2. Добавить несколько записей и подключившись через 2 терминала добиться ситуации взаимоблокировки (deadlock).

wb_psql=# INSERT INTO task4.accounts
wb_psql-# VALUES (1, 100),
wb_psql-# (2, 200);
INSERT 0 2

(1) wb_psql=# BEGIN;
(2) wb_psql=# BEGIN;
(1) wb_psql=*# UPDATE task4.accounts SET amount = amount + 1000 WHERE id = 1;
UPDATE 1
(2) wb_psql=*# UPDATE task4.accounts SET amount = amount + 2000 where id = 2;
UPDATE 1
(1) wb_psql=*# UPDATE task4.accounts SET amount = amount + 2000 where id = 2;
UPDATE 1
(2) wb_psql=*# UPDATE task4.accounts SET amount = amount + 1000 WHERE id = 1;
ERROR:  deadlock detected
DETAIL:  Process 739 waits for ShareLock on transaction 1017; blocked by process 733.
Process 733 waits for ShareLock on transaction 1018; blocked by process 739.
HINT:  See server log for query details.
CONTEXT:  while updating tuple (0,1) in relation "accounts"

# 3. Посмотреть логи и убедиться, что информация о дедлоке туда попала.

mahalichev@DESKTOP:~$ tail /var/log/postgresql/postgresql-17-main.log
2025-03-15 18:00:21.239 MSK [739] postgres@wb_psql ERROR:  deadlock detected
2025-03-15 18:00:21.239 MSK [739] postgres@wb_psql DETAIL:  Process 739 waits for ShareLock on transaction 1017; blocked by process 733.
        Process 733 waits for ShareLock on transaction 1018; blocked by process 739.
        Process 739: UPDATE task4.accounts SET amount = amount + 1000 WHERE id = 1;
        Process 733: UPDATE task4.accounts SET amount = amount + 2000 where id = 2;
2025-03-15 18:00:21.239 MSK [739] postgres@wb_psql HINT:  See server log for query details.
2025-03-15 18:00:21.239 MSK [739] postgres@wb_psql CONTEXT:  while updating tuple (0,1) in relation "accounts"
2025-03-15 18:00:21.239 MSK [739] postgres@wb_psql STATEMENT:  UPDATE task4.accounts SET amount = amount + 1000 WHERE id = 1;
```
