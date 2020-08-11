1) Выбрать одну из СУБД

    Выбрал для стравнение mongodb

    Обе машины n1-standard-1
    По 30Гб жеского диска(для загрузки данных)

2) Загрузить в неё данные (10 Гб)

    Взял из данные из примера в задании из практики таблицу выгрузил из в csv, взял 40 csv файлов загрузил в базу

    Для постгреса:
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
    COPY test FROM PROGRAM 'awk FNR-1 /tmp/csv.*' DELIMITER ',' CSV HEADER;

    Для mongodb

    for f in csv.* ; do mongoimport --type csv -d test -c products --headerline $f; done

3) Сравнить скорость выполнения запросов на PosgreSQL
и выбранной СУБД

    Взял пару селектов по разным полям:
    select * from test where taxi_id = '1b7aef0344f45914e7942efad05553934fe222c880a1cb19de79fbf7c4c42213cb82f74d26f02a4ef8f42bdba220392fef9e31be732048be89111a59fedafbf4';
    Time: 107240.904 ms (01:47.241)

    db.products.find({trip_end_timestamp: "2014-08-13 19:45:00 UTC"}).explain("executionStats");
    {
        "queryPlanner" : {
                "plannerVersion" : 1,
                "namespace" : "test.products",
                "indexFilterSet" : false,
                "parsedQuery" : {
                        "trip_end_timestamp" : {
                                "$eq" : "2014-08-13 19:45:00 UTC"
                        }
                },
                "winningPlan" : {
                        "stage" : "COLLSCAN",
                        "filter" : {
                                "trip_end_timestamp" : {
                                        "$eq" : "2014-08-13 19:45:00 UTC"
                                }
                        },
                        "direction" : "forward"
                },
                "rejectedPlans" : [ ]
        },
        "executionStats" : {
                "executionSuccess" : true,
                "nReturned" : 171,
                "executionTimeMillis" : 174572,
                "totalKeysExamined" : 0,
                "totalDocsExamined" : 27074462,
                "executionStages" : {
                        "stage" : "COLLSCAN",
                        "filter" : {
                                "trip_end_timestamp" : {
                                        "$eq" : "2014-08-13 19:45:00 UTC"
                                }
                        },
                        "nReturned" : 171,
                        "executionTimeMillisEstimate" : 172339,
                        "works" : 27074464,
                        "advanced" : 171,
                        "needTime" : 27074292,
                        "needYield" : 0,
                        "saveState" : 212873,
                        "restoreState" : 212873,
                        "isEOF" : 1,
                        "invalidates" : 0,
                        "direction" : "forward",
                        "docsExamined" : 27074462
                }
        },
        "serverInfo" : {
                "host" : "mongodb-1",
                "port" : 27017,
                "version" : "3.6.3",
                "gitVersion" : "9586e557d54ef70f9ca4b43c26892cd55257e1a5"
        },
        "ok" : 1
    }


    select * from test where trip_end_timestamp = '2014-08-13 19:45:00 UTC';
    Time: 120037.884 ms (02:00.038)

    db.products.find({trip_end_timestamp: "2014-08-13 19:45:00 UTC"}).explain("executionStats")
    {
        "queryPlanner" : {
                "plannerVersion" : 1,
                "namespace" : "test.products",
                "indexFilterSet" : false,
                "parsedQuery" : {
                        "trip_end_timestamp" : {
                                "$eq" : "2014-08-13 19:45:00 UTC"
                        }
                },
                "winningPlan" : {
                        "stage" : "COLLSCAN",
                        "filter" : {
                                "trip_end_timestamp" : {
                                        "$eq" : "2014-08-13 19:45:00 UTC"
                                }
                        },
                        "direction" : "forward"
                },
                "rejectedPlans" : [ ]
        },
        "executionStats" : {
                "executionSuccess" : true,
                "nReturned" : 171,
                "executionTimeMillis" : 150173,
                "totalKeysExamined" : 0,
                "totalDocsExamined" : 27074462,
                "executionStages" : {
                        "stage" : "COLLSCAN",
                        "filter" : {
                                "trip_end_timestamp" : {
                                        "$eq" : "2014-08-13 19:45:00 UTC"
                                }
                        },
                        "nReturned" : 171,
                        "executionTimeMillisEstimate" : 148396,
                        "works" : 27074464,
                        "advanced" : 171,
                        "needTime" : 27074292,
                        "needYield" : 0,
                        "saveState" : 212769,
                        "restoreState" : 212769,
                        "isEOF" : 1,
                        "invalidates" : 0,
                        "direction" : "forward",
                        "docsExamined" : 27074462
                }
        },
        "serverInfo" : {
                "host" : "mongodb-1",
                "port" : 27017,
                "version" : "3.6.3",
                "gitVersion" : "9586e557d54ef70f9ca4b43c26892cd55257e1a5"
        },
        "ok" : 1
    }

    В итоге первый запрос:
    Postgresql 107240.904
    Mongodb 174572
    Второй: 
    Postgresql 120037.884
    Mongodb 150173

4) Описать что и как делали и с какими проблемами столкнулись
    
    Проблемы с mongodb - другая консоль, сначала было не понятно как загружать туда данные, оказалось немного проще.