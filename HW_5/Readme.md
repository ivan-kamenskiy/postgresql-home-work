1 вариант:
• сделать проект <firstname>-<lastname>-<yyyymmdd>-05
    
    Ivan-Kamenskiy-20200306-05

• сделать инстанс Google Cloud Engine типа n1-standard-1 с ОС Ubuntu 18.04
    
    Сделано

• поставить на него PostgreSQL 11 из пакетов собираемых postgres.org

    Сделано

• настроить кластер PostgreSQL 11 на максимальную производительность не
обращая внимание на возможные проблемы с надежностью в случае
аварийной перезагрузки виртуальной машины

    Так как нам не важны проблемы с надежностью:

        synchronous_commit = off
        checkpoint_timeout = 1d

    То что взял с https://pgtune.leopard.in.ua/#/

        max_connections = 100
        shared_buffers = 1GB
        effective_cache_size = 3GB
        maintenance_work_mem = 256MB
        checkpoint_completion_target = 0.7
        wal_buffers = 16MB
        default_statistics_target = 100
        random_page_cost = 1.1
        effective_io_concurrency = 300
        work_mem = 5242kB
        min_wal_size = 1GB
        max_wal_size = 4GB

• нагрузить кластер через утилиту

    Выполнил бенчмарк черезе pgbench 
    postgres@instance-1:~$ pgbench -c 10 -j 2 -t 10000 test


• написать какого значения tps удалось достичь, показать какие параметры в
какие значения устанавливали и почему
    
    С настройками:
    tps = 1668.231179 (including connections establishing)
    tps = 1668.457460 (excluding connections establishing)

    С настройками по умолчанию:
    tps = 941.105759 (including connections establishing)
    tps = 941.159402 (excluding connections establishing)