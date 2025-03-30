## Задание

1. Развернуть ВМ (Linux) с PostgreSQL
2. Залить Тайские перевозки
https://github.com/aeuge/postgres16book/tree/main/database
3. Проверить скорость выполнения сложного запроса
4. Навесить индекс на внешние ключи
5. Проверить, помогли ли индексы на внешние ключи ускориться

## Решение

``` sql
# 2. Залить Тайские перевозки

mahalichev@DESKTOP:~$ wget https://storage.googleapis.com/thaibus/thai_small.tar.gz && tar -xf thai_small.tar.gz && psql < thai.sql
mahalichev@DESKTOP:~$ sudo -u postgres psql
psql (17.4 (Ubuntu 17.4-1.pgdg24.04+2))
Type "help" for help.

postgres=# \c thai
You are now connected to database "thai" as user "postgres".
thai=# VACUUM ANALYZE;
VACUUM

# 3. Проверить скорость выполнения сложного запроса
thai=# EXPLAIN ANALYZE
WITH all_place AS (
    SELECT count(s.id) as all_place, s.fkbus as fkbus
    FROM book.seat s
    group by s.fkbus
),
order_place AS (
    SELECT count(t.id) as order_place, t.fkride
    FROM book.tickets t
    group by t.fkride
)
SELECT r.id, r.startdate as depart_date, bs.city || ', ' || bs.name as busstation,
      t.order_place, st.all_place
FROM book.ride r
JOIN book.schedule as s
      on r.fkschedule = s.id
JOIN book.busroute br
      on s.fkroute = br.id
JOIN book.busstation bs
      on br.fkbusstationfrom = bs.id
JOIN order_place t
      on t.fkride = r.id
JOIN all_place st
      on r.fkbus = st.fkbus
GROUP BY r.id, r.startdate, bs.city || ', ' || bs.name, t.order_place,st.all_place
ORDER BY r.startdate
limit 10;

                                                                                      QUERY PLAN
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
 Limit  (cost=329687.17..329687.19 rows=10 width=56) (actual time=4992.157..4996.148 rows=10 loops=1)
   ->  Sort  (cost=329687.17..330040.42 rows=141301 width=56) (actual time=4968.192..4972.175 rows=10 loops=1)
         Sort Key: r.startdate
         Sort Method: top-N heapsort  Memory: 25kB
         ->  Group  (cost=277294.24..326633.71 rows=141301 width=56) (actual time=3646.528..4891.637 rows=144000 loops=1)
               Group Key: r.id, (((bs.city || ', '::text) || bs.name)), (count(t.id)), (count(s_1.id))
               ->  Incremental Sort  (cost=277294.24..324514.19 rows=141301 width=56) (actual time=3646.522..4747.369 rows=144000 loops=1)
                     Sort Key: r.id, (((bs.city || ', '::text) || bs.name)), (count(t.id)), (count(s_1.id))
                     Presorted Key: r.id
                     Full-sort Groups: 4500  Sort Method: quicksort  Average Memory: 27kB  Peak Memory: 27kB
                     ->  Merge Join  (cost=277065.85..316057.17 rows=141301 width=56) (actual time=3646.266..4600.170 rows=144000 loops=1)
                           Merge Cond: (r.id = t.fkride)
                           ->  Sort  (cost=20076.71..20436.71 rows=144000 width=32) (actual time=691.230..767.339 rows=144000 loops=1)
                                 Sort Key: r.id
                                 Sort Method: external merge  Disk: 6496kB
                                 ->  Hash Join  (cost=52.09..4291.50 rows=144000 width=32) (actual time=1.962..595.352 rows=144000 loops=1)
                                       Hash Cond: (r.fkbus = s_1.fkbus)
                                       ->  Hash Join  (cost=46.98..3587.99 rows=144000 width=28) (actual time=1.669..463.342 rows=144000 loops=1)
                                             Hash Cond: (br.fkbusstationfrom = bs.id)
                                             ->  Hash Join  (cost=45.75..3048.56 rows=144000 width=16) (actual time=1.615..332.031 rows=144000 loops=1)
                                                   Hash Cond: (s.fkroute = br.id)
                                                   ->  Hash Join  (cost=43.40..2641.51 rows=144000 width=16) (actual time=1.519..200.985 rows=144000 loops=1)
                                                         Hash Cond: (r.fkschedule = s.id)
                                                         ->  Seq Scan on ride r  (cost=0.00..2219.00 rows=144000 width=16) (actual time=0.014..62.602 rows=144000 loops=1)
                                                         ->  Hash  (cost=25.40..25.40 rows=1440 width=8) (actual time=1.484..1.486 rows=1440 loops=1)
                                                               Buckets: 2048  Batches: 1  Memory Usage: 73kB
                                                               ->  Seq Scan on schedule s  (cost=0.00..25.40 rows=1440 width=8) (actual time=0.023..0.728 rows=1440 loops=1)
                                                   ->  Hash  (cost=1.60..1.60 rows=60 width=8) (actual time=0.082..0.084 rows=60 loops=1)
                                                         Buckets: 1024  Batches: 1  Memory Usage: 11kB
                                                         ->  Seq Scan on busroute br  (cost=0.00..1.60 rows=60 width=8) (actual time=0.018..0.047 rows=60 loops=1)
                                             ->  Hash  (cost=1.10..1.10 rows=10 width=20) (actual time=0.042..0.043 rows=10 loops=1)
                                                   Buckets: 1024  Batches: 1  Memory Usage: 9kB
                                                   ->  Seq Scan on busstation bs  (cost=0.00..1.10 rows=10 width=20) (actual time=0.025..0.032 rows=10 loops=1)
                                       ->  Hash  (cost=5.05..5.05 rows=5 width=12) (actual time=0.276..0.278 rows=5 loops=1)
                                             Buckets: 1024  Batches: 1  Memory Usage: 9kB
                                             ->  HashAggregate  (cost=5.00..5.05 rows=5 width=12) (actual time=0.263..0.268 rows=5 loops=1)
                                                   Group Key: s_1.fkbus
                                                   Batches: 1  Memory Usage: 24kB
                                                   ->  Seq Scan on seat s_1  (cost=0.00..4.00 rows=200 width=8) (actual time=0.048..0.129 rows=200 loops=1)
                           ->  Finalize GroupAggregate  (cost=256989.14..292787.69 rows=141301 width=12) (actual time=2954.984..3611.503 rows=144000 loops=1)
                                 Group Key: t.fkride
                                 ->  Gather Merge  (cost=256989.14..289961.67 rows=282602 width=12) (actual time=2954.941..3323.062 rows=432000 loops=1)
                                       Workers Planned: 2
                                       Workers Launched: 2
                                       ->  Sort  (cost=255989.11..256342.37 rows=141301 width=12) (actual time=2892.406..2980.557 rows=144000 loops=3)
                                             Sort Key: t.fkride
                                             Sort Method: external merge  Disk: 3672kB
                                             Worker 0:  Sort Method: external merge  Disk: 3672kB
                                             Worker 1:  Sort Method: external merge  Disk: 3672kB
                                             ->  Partial HashAggregate  (cost=218976.78..241486.94 rows=141301 width=12) (actual time=2529.795..2762.001 rows=144000 loops=3)
                                                   Group Key: t.fkride
                                                   Planned Partitions: 4  Batches: 5  Memory Usage: 8241kB  Disk Usage: 27632kB
                                                   Worker 0:  Batches: 5  Memory Usage: 8241kB  Disk Usage: 27688kB
                                                   Worker 1:  Batches: 5  Memory Usage: 8241kB  Disk Usage: 27544kB
                                                   ->  Parallel Seq Scan on tickets t  (cost=0.00..80579.48 rows=2160348 width=12) (actual time=0.157..1118.624 rows=1728502 loops=3)
 Planning Time: 1.473 ms
 JIT:
   Functions: 81
   Options: Inlining false, Optimization false, Expressions true, Deforming true
   Timing: Generation 3.999 ms (Deform 1.390 ms), Inlining 0.000 ms, Optimization 3.156 ms, Emission 46.160 ms, Total 53.315 ms
 Execution Time: 5038.416 ms
(61 rows)

# 4. Навесить индекс на внешние ключи

thai=# CREATE INDEX idx_ride_fkschedule ON book.ride(fkschedule);
CREATE INDEX
thai=# CREATE INDEX idx_schedule_fkroute ON book.schedule(fkroute);
CREATE INDEX
thai=# CREATE INDEX idx_busroute_fkbusstationfrom ON book.busroute(fkbusstationfrom);
CREATE INDEX
thai=# CREATE INDEX idx_tickets_fkride ON book.tickets(fkride);
CREATE INDEX
thai=# CREATE INDEX idx_ride_fkbus ON book.ride(fkbus);
CREATE INDEX
thai=# CREATE INDEX idx_seat_fkbus ON book.seat(fkbus);
CREATE INDEX
thai=# VACUUM ANALYZE;
VACUUM

# 5. Проверить, помогли ли индексы на внешние ключи ускориться

thai=# EXPLAIN ANALYZE
WITH all_place AS (
    SELECT count(s.id) as all_place, s.fkbus as fkbus
    FROM book.seat s
    group by s.fkbus
),
order_place AS (
    SELECT count(t.id) as order_place, t.fkride
    FROM book.tickets t
    group by t.fkride
)
SELECT r.id, r.startdate as depart_date, bs.city || ', ' || bs.name as busstation,
      t.order_place, st.all_place
FROM book.ride r
JOIN book.schedule as s
      on r.fkschedule = s.id
JOIN book.busroute br
      on s.fkroute = br.id
JOIN book.busstation bs
      on br.fkbusstationfrom = bs.id
JOIN order_place t
      on t.fkride = r.id
JOIN all_place st
      on r.fkbus = st.fkbus
GROUP BY r.id, r.startdate, bs.city || ', ' || bs.name, t.order_place,st.all_place
ORDER BY r.startdate
limit 10;

                                                                                      QUERY PLAN
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
 Limit  (cost=331102.94..331102.96 rows=10 width=56) (actual time=4943.707..4947.358 rows=10 loops=1)
   ->  Sort  (cost=331102.94..331463.50 rows=144225 width=56) (actual time=4923.704..4927.346 rows=10 loops=1)
         Sort Key: r.startdate
         Sort Method: top-N heapsort  Memory: 25kB
         ->  Group  (cost=277619.52..327986.29 rows=144225 width=56) (actual time=3599.254..4846.915 rows=144000 loops=1)
               Group Key: r.id, (((bs.city || ', '::text) || bs.name)), (count(t.id)), (count(s_1.id))
               ->  Incremental Sort  (cost=277619.52..325822.91 rows=144225 width=56) (actual time=3599.245..4701.694 rows=144000 loops=1)
                     Sort Key: r.id, (((bs.city || ', '::text) || bs.name)), (count(t.id)), (count(s_1.id))
                     Presorted Key: r.id
                     Full-sort Groups: 4500  Sort Method: quicksort  Average Memory: 27kB  Peak Memory: 27kB
                     ->  Merge Join  (cost=277386.38..317169.66 rows=144225 width=56) (actual time=3598.877..4554.831 rows=144000 loops=1)
                           Merge Cond: (r.id = t.fkride)
                           ->  Sort  (cost=20076.71..20436.71 rows=144000 width=32) (actual time=675.727..752.439 rows=144000 loops=1)
                                 Sort Key: r.id
                                 Sort Method: external merge  Disk: 6496kB
                                 ->  Hash Join  (cost=52.09..4291.50 rows=144000 width=32) (actual time=1.860..587.246 rows=144000 loops=1)
                                       Hash Cond: (r.fkbus = s_1.fkbus)
                                       ->  Hash Join  (cost=46.98..3587.99 rows=144000 width=28) (actual time=1.533..457.002 rows=144000 loops=1)
                                             Hash Cond: (br.fkbusstationfrom = bs.id)
                                             ->  Hash Join  (cost=45.75..3048.56 rows=144000 width=16) (actual time=1.481..327.283 rows=144000 loops=1)
                                                   Hash Cond: (s.fkroute = br.id)
                                                   ->  Hash Join  (cost=43.40..2641.51 rows=144000 width=16) (actual time=1.393..198.039 rows=144000 loops=1)
                                                         Hash Cond: (r.fkschedule = s.id)
                                                         ->  Seq Scan on ride r  (cost=0.00..2219.00 rows=144000 width=16) (actual time=0.013..62.461 rows=144000 loops=1)
                                                         ->  Hash  (cost=25.40..25.40 rows=1440 width=8) (actual time=1.361..1.363 rows=1440 loops=1)
                                                               Buckets: 2048  Batches: 1  Memory Usage: 73kB
                                                               ->  Seq Scan on schedule s  (cost=0.00..25.40 rows=1440 width=8) (actual time=0.021..0.674 rows=1440 loops=1)
                                                   ->  Hash  (cost=1.60..1.60 rows=60 width=8) (actual time=0.075..0.077 rows=60 loops=1)
                                                         Buckets: 1024  Batches: 1  Memory Usage: 11kB
                                                         ->  Seq Scan on busroute br  (cost=0.00..1.60 rows=60 width=8) (actual time=0.017..0.044 rows=60 loops=1)
                                             ->  Hash  (cost=1.10..1.10 rows=10 width=20) (actual time=0.040..0.042 rows=10 loops=1)
                                                   Buckets: 1024  Batches: 1  Memory Usage: 9kB
                                                   ->  Seq Scan on busstation bs  (cost=0.00..1.10 rows=10 width=20) (actual time=0.025..0.030 rows=10 loops=1)
                                       ->  Hash  (cost=5.05..5.05 rows=5 width=12) (actual time=0.312..0.315 rows=5 loops=1)
                                             Buckets: 1024  Batches: 1  Memory Usage: 9kB
                                             ->  HashAggregate  (cost=5.00..5.05 rows=5 width=12) (actual time=0.301..0.306 rows=5 loops=1)
                                                   Group Key: s_1.fkbus
                                                   Batches: 1  Memory Usage: 24kB
                                                   ->  Seq Scan on seat s_1  (cost=0.00..4.00 rows=200 width=8) (actual time=0.049..0.148 rows=200 loops=1)
                           ->  Finalize GroupAggregate  (cost=257309.67..293849.01 rows=144225 width=12) (actual time=2923.089..3580.600 rows=144000 loops=1)
                                 Group Key: t.fkride
                                 ->  Gather Merge  (cost=257309.67..290964.51 rows=288450 width=12) (actual time=2923.043..3291.395 rows=432000 loops=1)
                                       Workers Planned: 2
                                       Workers Launched: 2
                                       ->  Sort  (cost=256309.64..256670.20 rows=144225 width=12) (actual time=2864.668..2952.883 rows=144000 loops=3)
                                             Sort Key: t.fkride
                                             Sort Method: external merge  Disk: 3672kB
                                             Worker 0:  Sort Method: external merge  Disk: 3672kB
                                             Worker 1:  Sort Method: external merge  Disk: 3672kB
                                             ->  Partial HashAggregate  (cost=218942.11..241483.53 rows=144225 width=12) (actual time=2514.635..2740.019 rows=144000 loops=3)
                                                   Group Key: t.fkride
                                                   Planned Partitions: 4  Batches: 5  Memory Usage: 8241kB  Disk Usage: 25360kB
                                                   Worker 0:  Batches: 5  Memory Usage: 8241kB  Disk Usage: 24344kB
                                                   Worker 1:  Batches: 5  Memory Usage: 8241kB  Disk Usage: 25352kB
                                                   ->  Parallel Seq Scan on tickets t  (cost=0.00..80531.55 rows=2160555 width=12) (actual time=0.205..1115.944 rows=1728502 loops=3)
 Planning Time: 1.648 ms
 JIT:
   Functions: 81
   Options: Inlining false, Optimization false, Expressions true, Deforming true
   Timing: Generation 3.853 ms (Deform 1.388 ms), Inlining 0.000 ms, Optimization 2.885 ms, Emission 43.189 ms, Total 49.927 ms
 Execution Time: 4954.085 ms
(61 rows)

```

## Ответ

В данном запросе добавление индексов на внешние ключи не помогло по нескольким причинам:

1. Происходит чтение всех данных из таблиц
2. Некоторые из этих таблиц имеют очень мало данных, для них последовательное чтение будет эффективнее
