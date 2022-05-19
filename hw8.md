# ДЗ 8

## Репликация

Создано две ВМ - мастер и реплика

![image](https://user-images.githubusercontent.com/41448520/169370782-1454a22e-8297-471a-8c8a-67467a8d8b78.png)

### Физическая репликация

1. На мастер ВМ:
    - Настраиваем postgresql.conf (/etc/postgresql/12/main/postgresql.conf), выставляем:<br>`listen_addresses = '*'`
    - Настраиваем pg_hba.conf (/etc/postgresql/12/main/pg_hba.conf), выставляем:<br>
        ```
        # IPv4 local connections:
      host    all             all             0.0.0.0/0            md5
      # IPv6 local connections:
      host    all             all             ::1/128                 md5
      # Allow replication connections from localhost, by a user with the
      # replication privilege.
      local   replication     all                                     peer
      host    replication     all             0.0.0.0/0            md5
      host    replication     all             ::1/128                 md5
      ```
   - Перезагружаем кластер для применения параметров:<br>`sudo pg_ctlcluster 12 main restart`
   - Создаем БД replica_db:<br>
     ![image](https://user-images.githubusercontent.com/41448520/169373459-8bd695fa-a7a6-4dae-88c1-159cf16033f0.png)

2. На реплике ВМ:
    - Останавливаем кластер:<br>`sudo pg_ctlcluster 12 main stop`
    - Удаляем директорию main:<br>`sudo rm -rf /var/lib/postgresql/12/main/`
    - Реплицируем кластер с мастера:<br>`sudo -u postgres pg_basebackup -h 51.250.108.37 -R -D /var/lib/postgresql/12/main -U postgres -W`
    - Поднимаем кластер:<br>`sudo pg_ctlcluster 12 main start`
    - Проверяем, что кластер реплицировался и находится в recovery:<br>
      ![image](https://user-images.githubusercontent.com/41448520/169373663-7840dc46-9508-47ec-894d-c7c6672ad16e.png)
3. На мастере создаем таблицу и добавляем в неё данные:<br>
   ![image](https://user-images.githubusercontent.com/41448520/169374438-ba9ee70e-c07d-46be-99a1-24dfbd5616a8.png)
4. На реплике проверяем:<br>
   ![image](https://user-images.githubusercontent.com/41448520/169375386-aefbf10f-5d0b-4b44-8e90-9b8d7c98d03a.png)
5. И повторяем для надежности:<br>
   Мастер:<br>
   ![image](https://user-images.githubusercontent.com/41448520/169375518-376cd763-d166-40c9-a1ad-e97676948f29.png)
   <br>Реплика:<br>
   ![image](https://user-images.githubusercontent.com/41448520/169375569-83087bba-8f8b-44f5-8127-724d158e2bb8.png)

### Логическая репликация

1. На мастере:
    - Настраиваем postgresql.conf (/etc/postgresql/12/main/postgresql.conf), выставляем:<br>`wal_level = logical`
    - Перезагружаем кластер:<br>`sudo pg_ctlcluster 12 main restart`
    - Создаем новую таблицу replica_l:<br>
      ![image](https://user-images.githubusercontent.com/41448520/169389735-4fdf9dcd-5407-4d55-a1f9-39e62d29770f.png)
    - Создаем публикацию для таблицы:<br>`create publication test_pub for table replica_l;`
    - Проверяем создание публикации:<br>`select * from pg_publication;` и `select * from pg_publication_tables;`
      ![image](https://user-images.githubusercontent.com/41448520/169390729-d875db9f-0eef-4448-ba1d-3a7d47c61477.png)
    - Добавляем значение в replica_l:
      ![image](https://user-images.githubusercontent.com/41448520/169391176-c2352468-9cc2-4a0a-9e6e-137e1104d041.png)
      
2. На реплике:
    - Выводим из реплики:<br>`sudo pg_ctlcluster 12 main promote`
    - Перезагружаем кластер:<br>`sudo pg_ctlcluster 12 main restart`
    - Создаем подписку:<br>`create subscription test_sub connection 'host=62.84.121.224 port=5432 user=postgres password=postgres dbname=replica_db' publication test_pub;`
    - Проверяем, что данные реплицировались и наличие подписки:<br>`select * from pg_subscription;`
      ![image](https://user-images.githubusercontent.com/41448520/169396322-30c6357a-78f7-46d8-891a-e6ec78a8376c.png)

      

