# ДЗ 8

## Репликация 

### Физическая репликация

1. Создано две ВМ - мастер и реплика

![image](https://user-images.githubusercontent.com/41448520/169370782-1454a22e-8297-471a-8c8a-67467a8d8b78.png)

2. На мастер ВМ:
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

3. На реплике ВМ:
    - Останавливаем кластер:<br>`sudo pg_ctlcluster 12 main stop`
    - Удаляем директорию main:<br>`sudo rm -rf /var/lib/postgresql/12/main/`
    - Реплицируем кластер с мастера:<br>`sudo -u postgres pg_basebackup -h 51.250.108.37 -R -D /var/lib/postgresql/12/main -U postgres -W`
    - Поднимаем кластер:<br>`sudo pg_ctlcluster 12 main start`
    - Проверяем, что кластер реплицировался и находится в recovery:<br>
      ![image](https://user-images.githubusercontent.com/41448520/169373663-7840dc46-9508-47ec-894d-c7c6672ad16e.png)
4. На мастере создаем таблицу и добавляем в неё данные:<br>
   ![image](https://user-images.githubusercontent.com/41448520/169374438-ba9ee70e-c07d-46be-99a1-24dfbd5616a8.png)
5. На реплике проверяем:<br>
   ![image](https://user-images.githubusercontent.com/41448520/169375386-aefbf10f-5d0b-4b44-8e90-9b8d7c98d03a.png)
6. И повторяем для надежности:<br>
   Мастер:<br>
   ![image](https://user-images.githubusercontent.com/41448520/169375518-376cd763-d166-40c9-a1ad-e97676948f29.png)
   <br>Реплика:<br>
   ![image](https://user-images.githubusercontent.com/41448520/169375569-83087bba-8f8b-44f5-8127-724d158e2bb8.png)


