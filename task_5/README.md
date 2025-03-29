## Задание

Протестировать падение производительности при использовании pgbouncer в разных режимах: statement, transaction, session

## Решение

``` sql
# 1. Загрузим тайские перевозки в среднем варианте (60 млн.строк, 6Гб)

mahalichev@DESKTOP:~$ wget https://storage.googleapis.com/thaibus/thai_small.tar.gz && tar -xf thai_small.tar.gz && psql < thai.sql
mahalichev@DESKTOP:~$ sudo -u postgres psql
postgres=# \c thai
You are now connected to database "thai" as user "postgres".
thai=# SELECT count(*) from book.tickets;
  count
---------
 5185505
(1 row)

thai=# \q

# 2. Создадим SQL файл для бенчмарка

mahalichev@DESKTOP:~$ echo "SELECT * FROM book.tickets WHERE id = random(1, 5185505);" >> ~/test.sql

# 3. Проведем бенчмарки для режима работы statement

mahalichev@DESKTOP:~$ /usr/lib/postgresql/17/bin/pgbench -c 4 -j 4 -T 30 -f ~/test.sql -h localhost -p 6432 -n -U postgres thai
number of transactions actually processed: 169
number of failed transactions: 0 (0.000%)
latency average = 720.939 ms
initial connection time = 27.060 ms
tps = 5.548320 (without initial connection time)

mahalichev@DESKTOP:~$ /usr/lib/postgresql/17/bin/pgbench -c 8 -j 4 -T 30 -f ~/test.sql -h localhost -p 6432 -n -U postgres thai
number of transactions actually processed: 213
number of failed transactions: 0 (0.000%)
latency average = 1153.004 ms
initial connection time = 47.606 ms
tps = 6.938395 (without initial connection time)

mahalichev@DESKTOP:~$ /usr/lib/postgresql/17/bin/pgbench -c 16 -j 4 -T 30 -f ~/test.sql -h localhost -p 6432 -n -U postgres thai
number of transactions actually processed: 225
number of failed transactions: 0 (0.000%)
latency average = 2200.228 ms
initial connection time = 87.907 ms
tps = 7.271972 (without initial connection time)

mahalichev@DESKTOP:~$ /usr/lib/postgresql/17/bin/pgbench -c 32 -j 4 -T 30 -f ~/test.sql -h localhost -p 6432 -n -U postgres thai
number of transactions actually processed: 377
number of failed transactions: 0 (0.000%)
latency average = 2766.118 ms
initial connection time = 171.092 ms
tps = 11.568559 (without initial connection time)

mahalichev@DESKTOP:~$ /usr/lib/postgresql/17/bin/pgbench -c 64 -j 4 -T 30 -f ~/test.sql -h localhost -p 6432 -n -U postgres thai
number of transactions actually processed: 361
number of failed transactions: 0 (0.000%)
latency average = 6077.421 ms
initial connection time = 388.214 ms
tps = 10.530782 (without initial connection time)

mahalichev@DESKTOP:~$ /usr/lib/postgresql/17/bin/pgbench -c 100 -j 4 -T 30 -f ~/test.sql -h localhost -p 6432 -n -U postgres thai
number of transactions actually processed: 460
number of failed transactions: 0 (0.000%)
latency average = 9051.844 ms
initial connection time = 524.506 ms
tps = 11.047473 (without initial connection time)

# 4. Проведем бенчмарки для режима работы transaction

mahalichev@DESKTOP:~$ /usr/lib/postgresql/17/bin/pgbench -c 4 -j 4 -T 30 -f ~/test.sql -h localhost -p 6432 -n -U postgres thai
number of transactions actually processed: 167
number of failed transactions: 0 (0.000%)
latency average = 733.025 ms
initial connection time = 26.647 ms
tps = 5.456843 (without initial connection time)

mahalichev@DESKTOP:~$ /usr/lib/postgresql/17/bin/pgbench -c 8 -j 4 -T 30 -f ~/test.sql -h localhost -p 6432 -n -U postgres thai
number of transactions actually processed: 200
number of failed transactions: 0 (0.000%)
latency average = 1229.832 ms
initial connection time = 48.358 ms
tps = 6.504952 (without initial connection time)

mahalichev@DESKTOP:~$ /usr/lib/postgresql/17/bin/pgbench -c 16 -j 4 -T 30 -f ~/test.sql -h localhost -p 6432 -n -U postgres thai
number of transactions actually processed: 200
number of failed transactions: 0 (0.000%)
latency average = 2472.745 ms
initial connection time = 88.312 ms
tps = 6.470542 (without initial connection time)

mahalichev@DESKTOP:~$ /usr/lib/postgresql/17/bin/pgbench -c 32 -j 4 -T 30 -f ~/test.sql -h localhost -p 6432 -n -U postgres thai
number of transactions actually processed: 396
number of failed transactions: 0 (0.000%)
latency average = 2640.559 ms
initial connection time = 169.065 ms
tps = 12.118646 (without initial connection time)

mahalichev@DESKTOP:~$ /usr/lib/postgresql/17/bin/pgbench -c 64 -j 4 -T 30 -f ~/test.sql -h localhost -p 6432 -n -U postgres thai
number of transactions actually processed: 393
number of failed transactions: 0 (0.000%)
latency average = 5555.817 ms
initial connection time = 331.636 ms
tps = 11.519458 (without initial connection time)

mahalichev@DESKTOP:~$ /usr/lib/postgresql/17/bin/pgbench -c 100 -j 4 -T 30 -f ~/test.sql -h localhost -p 6432 -n -U postgres thai
number of transactions actually processed: 432
number of failed transactions: 0 (0.000%)
latency average = 8642.541 ms
initial connection time = 633.038 ms
tps = 11.570671 (without initial connection time)

# 5. Проведем бенчмарки для режима работы session

mahalichev@DESKTOP:~$ /usr/lib/postgresql/17/bin/pgbench -c 4 -j 4 -T 30 -f ~/test.sql -h localhost -p 6432 -n -U postgres thai
number of transactions actually processed: 167
number of failed transactions: 0 (0.000%)
latency average = 732.561 ms
initial connection time = 26.623 ms
tps = 5.460300 (without initial connection time)

mahalichev@DESKTOP:~$ /usr/lib/postgresql/17/bin/pgbench -c 8 -j 4 -T 30 -f ~/test.sql -h localhost -p 6432 -n -U postgres thai
number of transactions actually processed: 208
number of failed transactions: 0 (0.000%)
latency average = 1183.148 ms
initial connection time = 47.018 ms
tps = 6.761623 (without initial connection time)

mahalichev@DESKTOP:~$ /usr/lib/postgresql/17/bin/pgbench -c 16 -j 4 -T 30 -f ~/test.sql -h localhost -p 6432 -n -U postgres thai
number of transactions actually processed: 195
number of failed transactions: 0 (0.000%)
latency average = 2562.431 ms
initial connection time = 88.027 ms
tps = 6.244070 (without initial connection time)

mahalichev@DESKTOP:~$ /usr/lib/postgresql/17/bin/pgbench -c 32 -j 4 -T 30 -f ~/test.sql -h localhost -p 6432 -n -U postgres thai
number of transactions actually processed: 386
number of failed transactions: 0 (0.000%)
latency average = 2686.462 ms
initial connection time = 170.590 ms
tps = 11.911577 (without initial connection time)

mahalichev@DESKTOP:~$ /usr/lib/postgresql/17/bin/pgbench -c 64 -j 4 -T 30 -f ~/test.sql -h localhost -p 6432 -n -U postgres thai
number of transactions actually processed: 419
number of failed transactions: 0 (0.000%)
latency average = 5277.775 ms
initial connection time = 369.022 ms
tps = 12.126323 (without initial connection time)

mahalichev@DESKTOP:~$ /usr/lib/postgresql/17/bin/pgbench -c 100 -j 4 -T 30 -f ~/test.sql -h localhost -p 6432 -n -U postgres thai
number of transactions actually processed: 416
number of failed transactions: 0 (0.000%)
latency average = 8763.022 ms
initial connection time = 513.162 ms
tps = 11.411589 (without initial connection time)
```

![image](https://github.com/user-attachments/assets/ff863340-09fe-4f72-b3eb-6112af0896c7)

## Отличие

**statement** - подключение отправляется в пулл после выполнения любого запроса.

**transaction** - подключение отправляется в пулл после выполнения транзакции (после COMMIT или ROLLBACK).

**session** - подключение отправляется в пулл только после того, как клиент отключился.
