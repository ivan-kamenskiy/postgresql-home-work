1 вариант:
Развернуть CockroachDB в GKE или GCE
Потесировать dataset с чикагскими такси
Или залить 10Гб данных и протестировать скорость запросов в сравнении с 1 инстансом PostgreSQL
Описать что и как делали и с какими проблемами столкнулись

1. Устанавливаем базу ectd на отдельно ВМ, кластер сделать не стал
2. Устанавливаем базу postgres-11 и patroni
3. Создаем 2 конфига:
    Первый:
        scope: patroni_cluster_1_2
        name: member_3

        restapi:
          listen: 0.0.0.0:8008
          connect_address: 10.128.0.37:8008

        etcd:
          hosts: etcd:2379
          protocol: http

        bootstrap:
          dcs:
            ttl: 30
            loop_wait: 10
            retry_timeout : 10
            maximum_lag_on_failover: 1048576
            postgresql:
              use_pg_rewind: true
              use_slots: true
              parameters:
                wal_keep_segments: 100

          initdb:
          - encoding: UTF8
          - data-checksums

          pg_hba:
          - host replication replicator 0.0.0.0/0 md5
          - host all all 0.0.0.0/0 md5

        postgresql:
          listen: 0.0.0.0:5432
          connect_address: 10.128.0.37:5432
          data_dir: /var/lib/postgresql/patroni_cluster_1/member_1/data
          bin_dir: /usr/lib/postgresql/11//bin
          authentication:
            replication:
              username: replicator
              password: test
            superuser:
              username: postgres
              password: test

    Второй:

        scope: patroni_cluster_1_2
        name: member_1

        restapi:
          listen: 0.0.0.0:8008
          connect_address: 10.128.0.38:8008

        etcd:
          hosts: etcd:2379
          protocol: http

        bootstrap:
          dcs:
            ttl: 30
            loop_wait: 10
            retry_timeout : 10
            maximum_lag_on_failover: 1048576
            postgresql:
              use_pg_rewind: true
              use_slots: true
              parameters:
                wal_keep_segments: 100

          initdb:
          - encoding: UTF8
          - data-checksums

          pg_hba:
          - host replication replicator 0.0.0.0/0 md5
          - host all all 0.0.0.0/0 md5

        postgresql:
          listen: 0.0.0.0:5432
          connect_address: 10.128.0.38:5432
          data_dir: /var/lib/postgresql/patroni_cluster_1/member_1/data
          bin_dir: /usr/lib/postgresql/11//bin
          authentication:
            replication:
              username: replicator
              password: test
            superuser:
              username: postgres
              password: test

4. Запускаем patroni deb_patroni_1.yml > patroni_member_1.log 2>&1 & и patroni deb_patroni_2.yml > patroni_member_2.log 2>&1 &
5. Проверям:

    postgres@instance-2:~$ patronictl -d etcd://etcd:2379 list patroni_cluster_1_2
    + Cluster: patroni_cluster_1_2 (6867445601898573826) --------+
    |  Member  |     Host    |  Role  |  State  | TL | Lag in MB |
    +----------+-------------+--------+---------+----+-----------+
    | member_1 | 10.128.0.38 |        | running |  1 |         0 |
    | member_3 | 10.128.0.37 | Leader | running |  1 |           |
    +----------+-------------+--------+---------+----+-----------+
