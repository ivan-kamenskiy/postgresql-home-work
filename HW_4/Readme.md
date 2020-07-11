2 вариант

1. Настройте сервер так, чтобы в журнал сообщений сбрасывалась информация о блокировках, удерживаемых более 200 миллисекунд. Воспроизведите ситуацию, при которой в журнале появятся такие сообщения.

    deadlock_timeout = 200ms

2. Смоделируйте ситуацию обновления одной и той же строки тремя командами UPDATE в разных сеансах. Изучите возникшие блокировки в представлении pg_locks и убедитесь, что все они понятны. Пришлите список блокировок и объясните, что значит каждая.

    Создал базу test, c таблицей t2.
    Создал 3 тарнцакции с апдейтом 1 строки
    На первой видно что transactionid granted - true, тоесть выдан на строку на других false, тоесть еще не выдан. 

        FROM pg_locks WHERE pid = 4725;
           locktype    | relation | virtxid | xid |       mode       | granted
        ---------------+----------+---------+-----+------------------+---------
         relation      | t2       |         |     | RowExclusiveLock | t
         virtualxid    |          | 6/16    |     | ExclusiveLock    | t
         transactionid |          |         | 575 | ExclusiveLock    | t
        (3 rows)

        test=# SELECT locktype, relation::REGCLASS, virtualxid AS virtxid, transactionid AS xid, mode, granted

        FROM pg_locks WHERE pid = 4726;
           locktype    | relation | virtxid | xid |       mode       | granted
        ---------------+----------+---------+-----+------------------+---------
         relation      | t2       |         |     | RowExclusiveLock | t
         virtualxid    |          | 5/10    |     | ExclusiveLock    | t
         tuple         | t2       |         |     | ExclusiveLock    | t
         transactionid |          |         | 575 | ShareLock        | f
         transactionid |          |         | 576 | ExclusiveLock    | t
        (5 rows)

        test=# SELECT locktype, relation::REGCLASS, virtualxid AS virtxid, transactionid AS xid, mode, granted

        FROM pg_locks WHERE pid = 4727;
           locktype    | relation | virtxid | xid |       mode       | granted
        ---------------+----------+---------+-----+------------------+---------
         relation      | t2       |         |     | RowExclusiveLock | t
         virtualxid    |          | 4/10    |     | ExclusiveLock    | t
         transactionid |          |         | 578 | ExclusiveLock    | t
         tuple         | t2       |         |     | ExclusiveLock    | f
        (4 rows)

3. Воспроизведите взаимоблокировку трех транзакций. Можно ли разобраться в ситуации постфактум, изучая журнал сообщений?
    
    В первой сессии  
    update t2 set c1 = 11 where c1=1;
    Во второй 
    update t2 set c1 = 12 where c1=2;
    В третий 
    update t2 set c1 = 13 where c1=3;
    update t2 set c1 = 111 where c1=1;
    Во второй 
    update t2 set c1 = 333 where c1=3;
    В первой сессии  
    update t2 set c1 = 222 where c1=2;

    Да можно.

4. Могут ли две транзакции, выполняющие единственную команду UPDATE одной и той же таблицы (без where), заблокировать друг друга?
    
    Да, можно как-то схитрить.

* Попробуйте воспроизвести такую ситуацию.