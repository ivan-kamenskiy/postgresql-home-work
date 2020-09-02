1 вариант:
Развернуть CockroachDB в GKE или GCE
Потесировать dataset с чикагскими такси
Или залить 10Гб данных и протестировать скорость запросов в сравнении с 1 инстансом PostgreSQL
Описать что и как делали и с какими проблемами столкнулись


1. Развернул 3 ВМ для CockroachDB в разных регионах. 
        
        Пример взял с https://eax.me/cockroachdb/

2. Полнял ВМ с postgresql загрузил в нее 10 Гб данных с taxi_trips
3. Сделал бекап загрузил в CockroachDB:
    
        IMPORT PGDUMP 'https://storage.googleapis.com/taxi-trips/test/test.sql.gz';

4. Проверил скорость запросов CockroachDB разнорегиональной с postgresql
    1. select
    
    Postgresql:

        test=# select  trip_start_timestamp from test where trip_start_timestamp= '2014-06-30 20:45:00';
        Time: 108489.250 ms (01:48.489)

        test=# select * from test where taxi_id = '1b7aef0344f45914e7942efad05553934fe222c880a1cb19de79fbf7c4c42213cb82f74d26f02a4ef8f42bdba220392fef9e31be732048be89111a59fedafbf4';
        Time: 110541.913 ms (01:50.542)




    CockroachDB:

        select  trip_start_timestamp from test where trip_start_timestamp= '2014-06-30 20:45:00';
        Time: 31.672001819s

        select * from test where taxi_id = '1b7aef0344f45914e7942efad05553934fe222c880a1cb19de79fbf7c4c42213cb82f74d26f02a4ef8f42bdba220392fef9e31be732048be89111a59fedafbf4';
        Time: 29.878412071s

    2. insert 

    Postgresql:

        test=# INSERT INTO test(
        test(# unique_key,
        test(# taxi_id,
        test(# trip_start_timestamp,
        test(# trip_end_timestamp,
        test(# trip_seconds,
        test(# trip_miles,
        test(# pickup_census_tract,
        test(# dropoff_census_tract,
        test(# pickup_community_area,
        test(# dropoff_community_area,
        test(# fare,
        test(# tips,
        test(# tolls,
        test(# extras,
        test(# trip_total,
        test(# payment_type,
        test(# company,
        test(# pickup_latitude,
        test(# pickup_longitude,
        test(# pickup_location,
        test(# dropoff_latitude,
        test(# dropoff_longitude,
        test(# dropoff_location)
        test-# VALUES (
        test(# 'bc3b5605c9957c7d85aa0fba788cc0215b9a3fe5',
        test(# 'b6bda386ee20bbe5e239adc88498c8e04137de6c0c28144cd3ab636cf093697c0058c54f289bc5e0b6dd291440309b80228b1c9648c4ace8bb22a9efb996da99',
        test(# '2018-06-03 14:45:00',
        test(# '2018-06-03 14:45:00',
        test(# '180',
        test(# '0.6',
        test(# '17031080100',
        test(# '17031081201',
        test(# '8',
        test(# '8',
        test(# '4.5',
        test(# '1',
        test(# '0',
        test(# '0',
        test(# '6',
        test(# 'Credit Card',
        test(# 'City Service',
        test(# '41.907520075',
        test(# '-87.6266589',
        test(# 'POINT (-87.6266589003 41.90752007470001)',
        '41.899155613',
        '-87.626210532',
        'POINT (-87.6262105324 41.8991556134)'
        test(# '41.899155613',
        test(# '-87.626210532',
        test(# 'POINT (-87.6262105324 41.8991556134)'
        test(# );
        INSERT 0 1
        Time: 2.265 ms

        test=# INSERT INTO test(
        test(# unique_key,
        test(# taxi_id,
        test(# trip_start_timestamp,
        test(# trip_end_timestamp)
        test-# VALUES (
        test(# 'bc3b5605c9957c7d85aa0fba788cc0215b9a3fe5',
        test(# 'b6bda386ee20bbe5e239adc88498c8e04137de6c0c28144cd3ab636cf093697c0058c54f289bc5e0b6dd291440309b80228b1c9648c4ace8bb22a9efb996da99',
        test(# '2018-06-03 14:45:00',
        test(# '2018-06-03 14:45:00');
        INSERT 0 1
        Time: 20.416 ms

    CockroachDB:

        INSERT INTO test(
        unique_key,
        taxi_id,
        trip_start_timestamp,
        trip_end_timestamp,
        trip_seconds,
        trip_miles,
        pickup_census_tract,
        dropoff_census_tract,
        pickup_community_area,
        dropoff_community_area,
        fare,
        tips,
        tolls,
        extras,
        trip_total,
        payment_type,
        company,
        pickup_latitude,
        pickup_longitude,
        pickup_location,
        dropoff_latitude,
        dropoff_longitude,
        dropoff_location)
        VALUES (
        'bc3b5605c9957c7d85aa0fba788cc0215b9a3fe5',
        'b6bda386ee20bbe5e239adc88498c8e04137de6c0c28144cd3ab636cf093697c0058c54f289bc5e0b6dd291440309b80228b1c9648c4ace8bb22a9efb996da99',
        '2018-06-03 14:45:00',
        '2018-06-03 14:45:00',
        '180',
        '0.6',
        '17031080100',
        '17031081201',
        '8',
        '8',
        '4.5',
        '1',
        '0',
        '0',
        '6',
        'Credit Card',
        'City Service',
        '41.907520075',
        '-87.6266589',
        'POINT (-87.6266589003 41.90752007470001)',
        '41.899155613',
        '-87.626210532',
        'POINT (-87.6262105324 41.8991556134)'
        );
        INSERT 1

        Time: 393.235169ms

        INSERT INTO test(
        unique_key,
        taxi_id,
        trip_start_timestamp,
        trip_end_timestamp)
        VALUES (
        'bc3b5605c9957c7d85aa0fba788cc0215b9a3fe5',
        'b6bda386ee20bbe5e239adc88498c8e04137de6c0c28144cd3ab636cf093697c0058c54f289bc5e0b6dd291440309b80228b1c9648c4ace8bb22a9efb996da99',
        '2018-06-03 14:45:00',
        '2018-06-03 14:45:00');
        INSERT 1

        Time: 305.720233ms

    3. update
    Postgresql:

        update test set trip_total = 7 WHERE trip_total = 6;
        UPDATE 231823
        Time: 113480.676 ms (01:53.481)

        update test set pickup_community_area = 7 WHERE pickup_latitude = '41.907520075';
        UPDATE 131537
        Time: 125297.517 ms (02:05.298)



    CockroachDB:



5. Проблемы
    1. Проблемы с загрузкой csv, так особо и не понял как грузить, те скриты которые использовал для Postgresql не работали для CockroachDB, пришлось делать дамп
    2. Долго искал нормальную инструкцию по установке и добавлению в кластер, оказалось все просто(правда не использовал сертификат)
6. Результаты (я не добавлял нигде индексы)
    1. CockroachDB бодрее работает с select
    2. Со вставками строк CockroachDB показывает уже намного хуже результат
    3. С update у CockroachDB совсем все плохо оставил на 2 часа так и не выполнился
