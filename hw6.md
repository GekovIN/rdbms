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
![image](https://user-images.githubusercontent.com/41448520/149820749-d270ae5c-a79a-4172-acac-f07ba9f43567.png)
