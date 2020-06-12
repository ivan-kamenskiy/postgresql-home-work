1 вариант:
• создайте виртуальную машину c Ubuntu 18.04 LTS (bionic) в GCE типа n1-standard-1 в default VPC в любом регионе и зоне, например us-central1-a

    Создал ВМ instance-2

• поставьте на нее PostgreSQL через sudo apt

    Установил 
    sudo apt install -y postgresql

• проверьте что кластер запущен через sudo -u postgres pg_lsclusters

    sudo -u postgres pg_lsclusters
    Ver Cluster Port Status Owner    Data directory              Log file
    10  main    5432 online postgres /var/lib/postgresql/10/main /var/log/postgresql/postgresql-10-main.log

• зайдите из под пользователя postgres в psql и сделайте произвольную таблицу с произвольным содержимым

    postgres=# create table test(c1 text);
    postgres=# insert into test values('1');
    \q
    sudo su - postgres
    postgres@instance-2:~$ psql
    psql (10.12 (Ubuntu 10.12-0ubuntu0.18.04.1))
    Type "help" for help.

    postgres=# create table test(c1 text);
    CREATE TABLE
    postgres=# insert into test values('1');
    INSERT 0 1
    postgres=#\q

• остановите postgres например через sudo -u postgres pg_ctlcluster 10 main stop

    sudo -u postgres pg_ctlcluster 10 main stop
    Warning: stopping the cluster using pg_ctlcluster will mark the systemd unit as failed. Consider using systemctl:

• создайте новый standard persistent диск GKE через Compute Engine -> Disks в том же регионе и зоне что GCE инстанс размером например 10GB

    disk-1

• добавьте свеже-созданный диск к виртуальной машине - надо зайти в режим ее редактирования и дальше выбрать пункт attach existing disk

    Добавил

• проинициализируйте диск согласно инструкции и подмонтировать файловую систему, только не забывайте менять имя диска на актуальное, в вашем случае это скорее всего будет /dev/sdb - https://www.digitalocean.com/community/tutorials/how-to-partition-and-format-storage-devices-in-linux

    Смонтировал согласно иннструкции

• сделайте пользователя postgres владельцем /mnt/data - chown -R postgres:postgres /mnt/data/

    sudo chown -R postgres:postgres /mnt/data/

• перенесите содержимое /var/lib/postgres/10 в /mnt/data - mv /var/lib/postgresql/10 /mnt/data

    sudo su - postgres
    mv /var/lib/postgresql/10 /mnt/data

• попытайтесь запустить кластер - sudo -u postgres pg_ctlcluster 10 main start

    Выходит ошибка
    Error: /var/lib/postgresql/10/main is not accessible or does not exist

• напишите получилось или нет и почему

    потомучто директория, которая описана в конфиге отсутсвует, поэтому postgresql не стартует
    решения 2 
    1) поменять настройки в конфиге
    2) сделать ссылку на директорию 
        Пример: ln -s /mnt/data/10 /var/lib/postgresql/10

• задание: найти конфигурационный параметр в файлах раположенных в /etc/postgresql/10/main который надо поменять и поменяйте его

    sudo vi /etc/postgresql/10/main/postgresql.conf

• напишите что и почему поменяли

    Потому что по этому пути находятся наши данные поле того как мы их переместили
    Меняем data_directory = '/mnt/data/10/main'

• попытайтесь запустить кластер - sudo -u postgres pg_ctlcluster 10 main start
• напишите получилось или нет и почему

    Получилось, потому что у на все правильно))

• зайдите через через psql и проверьте содержимое ранее созданной таблицы

    postgres=# select * from test;
    c1
    ---
    1
    (1 row)

• задание со звездочкой: не удаляя существующий GCE инстанс сделайте новый, поставьте на его PostgreSQL, удалите файлы с данными из /var/lib/postgres, перемонтируйте внешний диск который сделали ранее от первой виртуальной машины ко второй и запустите PostgreSQL на второй машине так чтобы он работал с данными на внешнем диске, расскажите как вы это сделали и что в итоге получилось.

    1. Закоментировал /etc/fstab
    2. Выключил пострегрес на 1 ВМ
    3. Размонтировал диск umount /mnt/data
    4. Удалил Диск из первой ВМ (instance-2)
    5. Создал новую ВМ (instance-3)
    5. Подключил диск к второй ВМ (instance-3)
        5.1 Подключаем через консоль
        5.2 Монтируем через команду sudo mount -o defaults /dev/sdb1 /mnt/data  (так как у нас уже создана файловая система делать уже создавать ее уже не надо)
        5.3 Меняем влядельца sudo chown -R postgres:postgres /mnt/data/ так как UID может быть разным
        5.4 Добавляем настройку в /etc/fstab 
            LABEL=datapartition /mnt/data ext4 defaults 0 2
    6. Устновил postgresql на вторую машину 
        sudo apt install -y postgresql
    7. Останавливаем postgresql sudo -u postgres pg_ctlcluster 10 main stop
    7. Изменяем настройки в конфиге так же как и делали раньше 
        sudo vi /etc/postgresql/10/main/postgresql.conf
        data_directory = '/mnt/data/10/main'
    8. Стартуем postgresql sudo -u postgres pg_ctlcluster 10 main start
    9. Смотрим есть ли наша таблица 
        postgres=# select * from test ;
         c1
        ----
         1
        (1 row)

    Машины остановил, смонтирован диск к второй ВМ (instance-3)
    На 1 машине пострес естественно работать не будет, так как диск там не смонтирован.


2 вариант:
• сделать проект <firstname>-<lastname>-<yyyymmdd>-04

    На лекции говорили что делать не надо

• сделать в нем GCE инстанс с Ubuntu 18.04

    Сделал ВМ (docker-hw-2)

• поставить на нем Docker Engine

    Ставлю по оф инструкции https://docs.docker.com/engine/install/ubuntu/
    Добавил себя сразу в группу docker

• сделать каталог /var/lib/postgres

    sudo mkdir /var/lib/postgres

• развернуть контейнер с PostgreSQL 11 смонтировав в него /var/lib/postgres

    docker network create postgres
    docker run -d \
        --name some-postgres \
        -e POSTGRES_PASSWORD=mysecretpassword \
        -e PGDATA=/var/lib/postgresql/data/pgdata \
        --network postgres \
        -v /var/lib/postgres:/var/lib/postgresql/data \
        -p 5432:5432 \
        postgres

• развернуть контейнер с клиентом postgres

        docker run -d \
        --name client-postgres \
        -e POSTGRES_PASSWORD=mysecretpassword \
        -e PGDATA=/var/lib/postgresql/data/pgdata \
        --network postgres \
        postgres

• подключится из контейнера с клиентом к контейнеру с сервером и сделать таблицу с парой строк
    
    docker exec -it client-postgres bash
    psql -h some-postgres -U postgres
    create table test(id text);
    insert into test values('test');
    insert into test values('test2');

• подключится к контейнеру с сервером с ноутбука
    
    Создал резрешиение в vpc для входящего трафика TCP/5432

• удалить контейнер с сервером

    docker kill some-postgres
    docker rm some-postgres

• создать его заново

    docker network create postgres
    docker run -d \
        --name some-postgres \
        -e POSTGRES_PASSWORD=mysecretpassword \
        -e PGDATA=/var/lib/postgresql/data/pgdata \
        --network postgres \
        -v /var/lib/postgres:/var/lib/postgresql/data \
        -p 5432:5432 \
        postgres

• подключится снова из контейнера с клиентом к контейнеру с сервером

    docker exec -it client-postgres bash
    psql -h some-postgres -U postgres

• оставляйте в ЛК ДЗ комментарии что и как вы делали и как боролись с проблемами

    все прошло на ура)


