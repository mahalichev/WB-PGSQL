## Задание

1. Создать таблицу с текстовым полем и заполнить случайными или сгенерированными данным в размере 1 млн строк
2. Посмотреть размер файла с таблицей
3. 5 раз обновить все строчки и добавить к каждой строчке любой символ
4. Посмотреть количество мертвых строчек в таблице и когда последний раз приходил автовакуум
5. Подождать некоторое время, проверяя, пришел ли автовакуум
6. 5 раз обновить все строчки и добавить к каждой строчке любой символ
7. Посмотреть размер файла с таблицей
8. Отключить Автовакуум на конкретной таблице
9. 10 раз обновить все строчки и добавить к каждой строчке любой символ
10. Посмотреть размер файла с таблицей
11. Объясните полученный результат
12. Не забудьте включить автовакуум

Задание со *:
Написать анонимную процедуру, в которой в цикле 10 раз обновятся все строчки в искомой таблице. 
Не забыть вывести номер шага цикла.

## Решение

``` sql
# 1. Создать таблицу с текстовым полем и заполнить случайными или сгенерированными данным в размере 1 млн строк

postgres=# \c wb_psql
wb_psql=# CREATE TABLE task3.random (value text);
CREATE TABLE
wb_psql=# INSERT INTO task3.random
wb_psql-# SELECT md5(random()::text)
wb_psql-# FROM generate_series(1, 1000000);
INSERT 0 1000000

# 2. Посмотреть размер файла с таблицей

wb_psql=# SELECT pg_size_pretty(pg_relation_size('task3.random'));
 pg_size_pretty
----------------
 65 MB
(1 row)

# 3. 5 раз обновить все строчки и добавить к каждой строчке любой символ

wb_psql=# UPDATE task3.random SET value = value || '1';
UPDATE 1000000
wb_psql=# UPDATE task3.random SET value = value || '2';
UPDATE 1000000
wb_psql=# UPDATE task3.random SET value = value || '3';
UPDATE 1000000
wb_psql=# UPDATE task3.random SET value = value || '4';
UPDATE 1000000
wb_psql=# UPDATE task3.random SET value = value || '5';
UPDATE 1000000

# 4. Посмотреть количество мертвых строчек в таблице и когда последний раз приходил автовакуум

wb_psql=# SELECT n_live_tup, n_dead_tup, last_autovacuum
wb_psql-# FROM pg_stat_user_tables
wb_psql-# WHERE relname = 'random';
 n_live_tup | n_dead_tup |        last_autovacuum
------------+------------+-------------------------------
    1000000 |    4999814 | 2025-03-15 17:00:31.772059+03
(1 row)

wb_psql=# SELECT pg_size_pretty(pg_relation_size('task3.random'));
 pg_size_pretty
----------------
 391 MB
(1 row)

# 5. Подождать некоторое время, проверяя, пришел ли автовакуум

wb_psql=# SELECT n_live_tup, n_dead_tup, last_autovacuum
wb_psql-# FROM pg_stat_user_tables
wb_psql-# WHERE relname = 'random';
 n_live_tup | n_dead_tup |        last_autovacuum
------------+------------+-------------------------------
    1004748 |          0 | 2025-03-15 17:01:44.137267+03
(1 row)

wb_psql=# SELECT pg_size_pretty(pg_relation_size('task3.random'));
 pg_size_pretty
----------------
 391 MB
(1 row)

# 6. 5 раз обновить все строчки и добавить к каждой строчке любой символ

wb_psql=# UPDATE task3.random SET value = value || '1';
UPDATE 1000000
wb_psql=# UPDATE task3.random SET value = value || '2';
UPDATE 1000000
wb_psql=# UPDATE task3.random SET value = value || '3';
UPDATE 1000000
wb_psql=# UPDATE task3.random SET value = value || '4';
UPDATE 1000000
wb_psql=# UPDATE task3.random SET value = value || '5';
UPDATE 1000000

# 7. Посмотреть размер файла с таблицей

wb_psql=# SELECT n_live_tup, n_dead_tup, last_autovacuum
wb_psql-# FROM pg_stat_user_tables
wb_psql-# WHERE relname = 'random';
 n_live_tup | n_dead_tup |        last_autovacuum
------------+------------+-------------------------------
    1004748 |    4999309 | 2025-03-15 17:01:44.137267+03
(1 row)


wb_psql=# SELECT pg_size_pretty(pg_relation_size('task3.random'));
 pg_size_pretty
----------------
 414 MB
(1 row)

# 8. Отключить Автовакуум на конкретной таблице

wb_psql=# ALTER TABLE task3.random SET (autovacuum_enabled = off);
ALTER TABLE

# 9. 10 раз обновить все строчки и добавить к каждой строчке любой символ

wb_psql=# UPDATE task3.random SET value = value || '1';
UPDATE 1000000
wb_psql=# UPDATE task3.random SET value = value || '2';
UPDATE 1000000
wb_psql=# UPDATE task3.random SET value = value || '3';
UPDATE 1000000
wb_psql=# UPDATE task3.random SET value = value || '4';
UPDATE 1000000
wb_psql=# UPDATE task3.random SET value = value || '5';
UPDATE 1000000
wb_psql=# UPDATE task3.random SET value = value || '6';
UPDATE 1000000
wb_psql=# UPDATE task3.random SET value = value || '7';
UPDATE 1000000
wb_psql=# UPDATE task3.random SET value = value || '8';
UPDATE 1000000
wb_psql=# UPDATE task3.random SET value = value || '9';
UPDATE 1000000
wb_psql=# UPDATE task3.random SET value = value || '0';
UPDATE 1000000

# 10. Посмотреть размер файла с таблицей
wb_psql=# SELECT n_live_tup, n_dead_tup, last_autovacuum
wb_psql-# FROM pg_stat_user_tables
wb_psql-# WHERE relname = 'random';
 n_live_tup | n_dead_tup |        last_autovacuum
------------+------------+-------------------------------
    1011545 |    9997668 | 2025-03-15 17:03:43.738765+03
(1 row)

wb_psql=# SELECT pg_size_pretty(pg_relation_size('task3.random'));
 pg_size_pretty
----------------
 841 MB
(1 row)

# 11. Объясните полученный результат

Каждое выполнение UPDATE создаёт новые версии строк. Так как для данной таблицы выключен автовакуум,
старые версии строк остаются на месте, что объясняет резкое увеличение размера файла с таблицей.

# 12. Не забудьте включить автовакуум

wb_psql=# ALTER TABLE task3.random SET (autovacuum_enabled = on);
ALTER TABLE

# Написать анонимную процедуру, в которой в цикле 10 раз обновятся все строчки в искомой таблице.

DO $$
DECLARE
    i INT = 0;
    s TEXT;
BEGIN
    WHILE i < 10 LOOP
        RAISE NOTICE 'Step: %', i+1;
        s := i::text;
        UPDATE task3.random SET value = value || s;
        i := i + 1;
    END LOOP;
END $$;
```

