1 вариант:
Создать индексы на БД, которые ускорят доступ к данным.
В данном задании тренируются навыки:
- определения узких мест
- написания запросов для создания индекса
- оптимизации
Необходимо:
1) Создать индекс к какой-либо из таблиц вашей БД

    Создал базу test, с таблицей тест, взял данные из bigquery-public-data:chicago_taxi_trips.taxi_trips(2Гб) как из прошлого ДЗ
    create  table test (
    unique_key text,
    taxi_id text,
    trip_start_timestamp TIMESTAMP,
    trip_end_timestamp TIMESTAMP,
    trip_seconds bigint,
    trip_miles numeric,
    pickup_census_tract bigint,
    dropoff_census_tract bigint,
    pickup_community_area bigint,
    dropoff_community_area bigint,
    fare numeric,
    tips numeric,
    tolls numeric,
    extras numeric,
    trip_total numeric,
    payment_type text,
    company text,
    pickup_latitude numeric,
    pickup_longitude numeric,
    pickup_location text,
    dropoff_latitude numeric,
    dropoff_longitude numeric,
    dropoff_location text
    );
    COPY test FROM PROGRAM 'awk FNR-1 /var/lib/postgresql/jfjfj.csv.*' DELIMITER ',' CSV HEADER;

    Сделал индекс trip_start_timestamp для данного столбца:

    create index idx_test_trip_start_timestamp on test(trip_start_timestamp);

2) Прислать текстом результат команды explain, в которой используется данный индекс

    test-# select * from test where trip_start_timestamp= '2014-06-30 20:45:00';
    -[ RECORD 1 ]----------------------------------------------------------------------------------------------
    QUERY PLAN | Index Scan using idx_test_trip_start_timestamp on test  (cost=0.43..865.63 rows=220 width=399)
    -[ RECORD 2 ]----------------------------------------------------------------------------------------------
    QUERY PLAN |   Index Cond: (trip_start_timestamp = '2014-06-30 20:45:00'::timestamp without time zone)

    Если мы хотим только по индексу:

    test=# explain
    select  trip_start_timestamp from test where trip_start_timestamp= '2014-06-30 20:45:00';
    -[ RECORD 1 ]-------------------------------------------------------------------------------------------------
    QUERY PLAN | Index Only Scan using idx_test_trip_start_timestamp on test  (cost=0.43..865.63 rows=220 width=8)
    -[ RECORD 2 ]-------------------------------------------------------------------------------------------------
    QUERY PLAN |   Index Cond: (trip_start_timestamp = '2014-06-30 20:45:00'::timestamp without time zone)

3) Реализовать индекс для полнотекстового поиска

    CREATE INDEX idx_test_company ON test USING GIN (to_tsvector('english', company));
    
    explain
    test-# select company from test where to_tsvector('english', company) @@ to_tsquery('english', 'elite
    test'# ');
                                           QUERY PLAN
    -----------------------------------------------------------------------------------------
     Bitmap Heap Scan on test  (cost=321.73..108706.03 rows=33771 width=24)
       Recheck Cond: (to_tsvector('english'::regconfig, company) @@ '''elit'''::tsquery)
       ->  Bitmap Index Scan on idx_test_company  (cost=0.00..313.28 rows=33771 width=0)
             Index Cond: (to_tsvector('english'::regconfig, company) @@ '''elit'''::tsquery)
    (4 rows)

4) Реализовать индекс на часть таблицы или индекс на поле с функцией

    create index idx_test_trip_seconds on test(trip_seconds) where trip_seconds < 2;

    test=# explain
    select trip_seconds from test where trip_seconds < 2;
    -[ RECORD 1 ]------------------------------------------------------------------------------------------
    QUERY PLAN | Bitmap Heap Scan on test  (cost=7340.08..381200.63 rows=451744 width=8)
    -[ RECORD 2 ]------------------------------------------------------------------------------------------
    QUERY PLAN |   Recheck Cond: (trip_seconds < 2)
    -[ RECORD 3 ]------------------------------------------------------------------------------------------
    QUERY PLAN |   ->  Bitmap Index Scan on idx_test_trip_seconds  (cost=0.00..7227.14 rows=451744 width=0)

5) Создать индекс на несколько полей

    create index idx_test_unique_key_and_taxi_id on test(unique_key,taxi_id);

6) Написать комментарии к каждому из индексов

    COMMENT ON INDEX idx_test_trip_start_timestamp IS 'Индекс по полю trip_start_timestamp';
    COMMENT ON INDEX idx_test_company IS 'Индекс по полю company, для полнотекстового поиска';
    COMMENT ON INDEX idx_test_trip_seconds IS 'Индекс части таблице trip_seconds меньше 2, где по полю trip_seconds';
    COMMENT ON INDEX idx_test_unique_key_and_taxi_id IS 'Индекс по двум полям unique_key и taxi_id';

7) Описать что и как делали и с какими проблемами столкнулись

    Изначально было не понятно как сделать индек по полнотекстового поиска
    Из статьи https://postgrespro.ru/docs/postgrespro/9.5/textsearch-tables все понял.