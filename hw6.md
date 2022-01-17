# ДЗ 6

## Индексы в PostgreSQL

1. Создаем индекс для поля order_number таблицы order, т.к.это бизнес идентификатор заказа. Демонстрируется пользователю, используется администрацией для идентификации заказов при взаимодействии с пользователем. Будет иметь высокую кардинальность
```sql
create index if not exists order__number_index on kitchen."order" (order_number);
```
2. Тестируем с помощью EXPLAIN:
```sql

-- Подготоваливаем данные для теста

truncate kitchen.organization_terminal cascade;
truncate kitchen.organization cascade;
truncate kitchen."order" cascade;
truncate kitchen."user" cascade;

-- перестартуем последовательности
alter sequence kitchen.organization_id_seq restart;
alter sequence kitchen.organization_terminal_id_seq restart;
alter sequence kitchen.order_id_seq restart;
alter sequence kitchen.user_id_seq restart;

-- наполняем данными
insert into kitchen.organization values (default, 'organization_0', 'organization_address_0', true);

insert into kitchen.organization_terminal values (default, 'terminal_0', 'terminal_address_0', true, 1);

insert into kitchen."user" (id, name, email, mobile_number, created_date) values (1, 'Ivan', 'email@email.com', '79998887766', now());

-- используем функцию generate_series, чтобы сгенерировать 100 000 записей в таблице order с уникальными order_number.
insert into kitchen."order" (id, order_number, total_cost, status, status_reason, created_date, updated_date, terminal_id, user_id)
SELECT i, 'order_' || i::text, 99, 'created', '', now(), now(), 1, 1
FROM generate_series(1, 100000) AS t(i);

-- Проверяем, что индекс работает

explain select * from kitchen."order" where order_number = 'order_78375';

```
Получаем Index Scan:

![image](https://user-images.githubusercontent.com/41448520/149820749-d270ae5c-a79a-4172-acac-f07ba9f43567.png)

3. Полнотекстовой поиск <br/> 
Предположим, что в поле order.status_reason будут хранится произвольные данные о состоянии выполнения заказа. В таком случае нам может понадобиться полнотекстовой поиск для поиска заказов по каким-либо ключевым словам из его статуса
  ```sql
-- Заполняем столбец status_reason различными рандомными значениями
DO
$do$
    BEGIN
        FOR i IN 1..99999 LOOP
                update kitchen."order" set status_reason = (select (array['Заказ выполнен', 'Заказ в процессе'])[floor(random() * 2 + 1)]) where id = i;
            END LOOP;
    END
$do$;

-- Небольшой части записей добавим статус 'Произошла ошибка'
DO
$do$
    BEGIN
        FOR i IN 1..10 LOOP
                update kitchen."order" set status_reason = 'Произошла ошибка' where id = (SELECT floor(random() * 100000 + 1)::int);
            END LOOP;
    END
$do$;

-- Создадим индекс на столбец status_reason
-- создаем новую колонку для хранения tsvector
alter table kitchen."order" add column status_lexeme tsvector;
update kitchen."order" set status_lexeme = to_tsvector(status_reason);

-- создаем сам индекс типа GIN
drop index if exists kitchen.idx_gin_order_status_reason;
create index idx_gin_order_status_reason
    on kitchen."order"
        using gin (status_lexeme);

-- Пробуем найти все записи, у которых статус содержит слово 'ошибка'
explain select * from kitchen."order" where "order".status_lexeme @@ to_tsquery('ошибка');
```
Получаем Index Scan:

![image](https://user-images.githubusercontent.com/41448520/149827563-f22c75bc-6874-4dcf-93e5-67c6c8941f86.png)


4. Создание частичного индекса<br/>
  Предположим, что мы не видим смысла индексировать всю таблицу order, а только последние 5000 заказов
  
```sql
-- создаем частичный индекс
drop index if exists kitchen.order__number_index;
create index if not exists order__number_index on kitchen."order" (order_number) where id > 95000;

-- проверяем его работу
explain select * from kitchen."order" where id > 95000 and order_number = 'order_95001';
explain select * from kitchen."order" where id > 1000 and order_number = 'order_1001';
```
Для первого запроса Index Scan:
![image](https://user-images.githubusercontent.com/41448520/149829544-859159b1-709b-41a0-ba67-8f57977de399.png)

Для второго запроса Seq Scan:
![image](https://user-images.githubusercontent.com/41448520/149829590-0a31ebd4-1071-47db-a212-804cd4642e44.png)

```sql
-- сравниваем размеры полного и частичного индексов 
drop index if exists kitchen.order__number_index;
-- create index if not exists order__number_index on kitchen."order" (order_number);
create index if not exists order__number_index on kitchen."order" (order_number) where id > 95000;

select pg_size_pretty(pg_table_size('kitchen.order__number_index'));
```
![image](https://user-images.githubusercontent.com/41448520/149829702-4c399068-29ac-49d7-8445-fcdc0a0711b4.png)
<br/>против<br/>
![image](https://user-images.githubusercontent.com/41448520/149829748-cbf78fa9-af84-4be1-a9f9-a152724a0e77.png)


5. Составные индексы<br/>
Предположим, что мы хотим иметь возможность искать по БД продукты, зная только название и цену
```sql
-- заполним таблицу product тестовыми данными
truncate kitchen.product cascade;
alter sequence kitchen.product_id_seq restart;

insert into kitchen.product (id, name, price)
SELECT i, 'product_' || i::text, 0
FROM generate_series(1, 100000) AS t(i);

DO
$do$
    BEGIN
        FOR i IN 1..100000 LOOP
                update kitchen.product set price = (SELECT floor(random() * 1000)::int) where id = i;
            END LOOP;
    END
$do$;

-- создаем составной индекс на поля name и price

create index if not exists product_name_price_index on kitchen.product(name, price);

-- смотрим на его работу
explain select * from kitchen.product where name = 'product_3016' and price = 183.00;
```
Получаем Index Scan

![image](https://user-images.githubusercontent.com/41448520/149831411-503b0a9e-74f0-49c5-afde-4b3f36a64943.png)
