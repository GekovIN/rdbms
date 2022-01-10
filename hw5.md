# ДЗ 5

## DML: вставка, обновление, удаление, выборка данных 

```sql
-- очищаем таблицы
delete from kitchen.product;
delete from kitchen.product_category;
delete from kitchen.organization_terminal;
delete from kitchen.organization;

-- перестартуем последовательности
alter sequence kitchen.organization_id_seq restart;
alter sequence kitchen.organization_terminal_id_seq restart;
alter sequence kitchen.product_category_id_seq restart;
alter sequence kitchen.product_id_seq restart;

-- наполняем данными
insert into kitchen.organization values (default, 'organization_0', 'organization_address_0', true),
                                        (default, 'organization_1', 'organization_address_1', true),
                                        (default, 'organization_2', 'organization_address_2', true)
returning *;

insert into kitchen.organization_terminal values (default, 'terminal_0', 'terminal_address_0', true, 1),
                                                 (default, 'terminal_1', 'terminal_address_1 ', true, 1),
                                                 (default, 'terminal_2', 'terminal_address_2', true, 2),
                                                 (default, 'terminal_3', 'terminal_address_3', true, 2),
                                                 (default, 'terminal_4', 'terminal_address_4', true, 2)
returning *;

insert into kitchen.product_category values (default, 'breakfast', 'dishes for breakfast', true, 1),
                                            (default, 'lunch', 'dishes for lunch', true, 1),
                                            (default, 'dinner', 'dishes for dinner', true, 1)
returning *;

insert into kitchen.product values (default, 'eggs', 'some eggs', 'eggs_code', 199.99, true, 1),
                                   (default, 'beacon', 'some beacon', 'beacon_code', 120.75, true, 1),
                                   (default, 'orange juice', 'some orange juice', 'orange_juice_code', 99.50, true, 1),

                                   (default, 'soup with mushrooms', 'some mushroom soup', 'soup_code', 250, true, 2),
                                   (default, 'soup with cheese', 'some cheese soup', 'soup_code', 250, true, 2),
                                   (default, 'beef stroganoff', 'some beef', 'beef_code', 399, true, 2),
                                   (default, 'tea', 'some tea', 'tea_code', 79.90, true, 2),
                                   (default, 'apple juice', 'some apple juice', 'apple_juice_code', 59.90, true, 2),

                                   (default, 'pasta', 'some pasta', 'pasta_code', 399.90, true, 3),
                                   (default, 'red vine', 'some red vine', 'vine_code', 599.90, true, 3),
                                   (default, 'white vine', 'some white vine', 'vine_code', 599.90, true, 3)
returning *;


-- select с регулярным выражением - ищем все супы и все соки
select * from kitchen.product where name like 'soup%';
select * from kitchen.product where name like '%juice';

-- select с left join - получаем все организации с их терминалами, также получаем organization_2, у которой нет терминалов
select o.name, ot.name from kitchen.organization o
                                left join kitchen.organization_terminal ot on o.id = ot.organization_id;

-- select с inner join - получаем все организации с их терминалами, не получаем organization_2, т.к. у неё нет терминалов
select o.name, ot.name from kitchen.organization o
                                inner join kitchen.organization_terminal ot on o.id = ot.organization_id;

-- update с from - ставим цену для апельсинового сока такую же как у яблочного
update kitchen.product as p
set price = pp.price from (select * from kitchen.product where name = 'apple juice') as pp
where p.name = 'orange juice'
returning p.name, p.price, pp.name, pp.price;

-- delete с using - удаляем все продукты из категории завтрака
delete from kitchen.product
    using kitchen.product_category pc
where pc.id = product.product_category_id
  and pc.name = 'breakfast';

```
Для того, чтобы воспользоваться утилитой COPY для начала нужно предоставить юзеру developer права группы pg_write_server_files:
```sql
grant pg_write_server_files to developer;
```

Затем копируем данные о всех продуктах категории обеда в файл:
```sql
copy (select * from kitchen.product p join kitchen.product_category pc on p.product_category_id = pc.id where pc.name = 'dinner') to 'C:\my\tmp\product_data.copy';
```
Копирование успешно:
![image](https://user-images.githubusercontent.com/41448520/148828012-f2f2178e-6003-4dc5-8422-8a955a33498d.png)
