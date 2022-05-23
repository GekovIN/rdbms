# ДЗ 9
## Внутренняя архитектура СУБД MySQL 

1. Подготавливаем скрипт init.sql для MySQL

```sql
-- Создаем БД
CREATE DATABASE kitchen;
use kitchen;

-- Создаем таблицы
 
create table if not exists product_modifier
(
	id          INT AUTO_INCREMENT not null primary key,
    name        VARCHAR(50),
    description VARCHAR(255),
    code        VARCHAR(50),
    price       DECIMAL(19,2),
    active      BIT
);

create table if not exists organization
(
    id      INT AUTO_INCREMENT
        not null primary key,
    name    VARCHAR(50) not null,
    address VARCHAR(100),
    active  BIT default true
);

create table if not exists product_category
(
    id              INT AUTO_INCREMENT not null primary key,
    name            VARCHAR(50) not null,
    description     VARCHAR(255),
    active          BIT,
    organization_id INT  not null,
	
    constraint organization___fk foreign key (organization_id) references organization (id)
);

create table if not exists product
(
    id                  INT AUTO_INCREMENT not null primary key,
    name                VARCHAR(50)        not null,
    description         VARCHAR(255),
    code                VARCHAR(50),
    price               DECIMAL(19,2) not null,
    active              BIT default true,
    product_category_id INT,
	
    constraint product_category___fk foreign key (product_category_id) references product_category (id)
);

create table if not exists modifier_to_product
(
    product_id  INT not null,
    modifier_id INT not null,
	
	constraint product__fk foreign key (product_id) references product (id),
    constraint modifier___fk foreign key (modifier_id) references product_modifier (id)
);

create table if not exists organization_terminal
(
    id              INT AUTO_INCREMENT not null primary key,
    name            VARCHAR(50) not null,
    address         VARCHAR(100),
    active          BIT default true,
    organization_id INT  not null,
    constraint terminal_organization___fk foreign key (organization_id) references organization (id)
);

create table if not exists payment
(
    id           INT AUTO_INCREMENT not null primary key,
    payment_type VARCHAR(20)   not null,
    status       VARCHAR(20),
    created_date TIMESTAMP not null,
    updated_date TIMESTAMP
);

create table if not exists user
(
    id            INT AUTO_INCREMENT not null primary key,
    name          VARCHAR(50),
    email         VARCHAR(20),
    mobile_number VARCHAR(20)   not null,
    birth_date    DATE,
    enabled       BIT default true,
    created_date  TIMESTAMP not null,
    updated_date  TIMESTAMP
);

create table if not exists kitchen_order
(
    id            INT AUTO_INCREMENT not null primary key,
    order_number  VARCHAR(100) not null,
    total_cost    DECIMAL(19,2) not null,
    status        VARCHAR(20) not null,
    status_reason VARCHAR(255),
    created_date  TIMESTAMP not null,
    updated_date  TIMESTAMP,
    terminal_id   INT,
    user_id       INT not null,
    payment_id    INT,
	
	constraint terminal___fk foreign key (terminal_id) references organization_terminal (id),
    constraint user___fk foreign key (user_id) references user (id),
    constraint payment___fk foreign key (payment_id) references payment(id)
);
create index order__number_index on kitchen_order (order_number);
create index order__create_date_index on kitchen_order (created_date);

create table if not exists order_item
(
    id         INT AUTO_INCREMENT not null primary key,
    name       VARCHAR(50)        not null,
    price      DECIMAL(19,2) not null,
    order_id   INT         not null,
    product_id INT         not null,
	
	constraint order___fk foreign key (order_id) references kitchen_order (id),
    constraint product___fk foreign key (product_id) references product (id)
);
create index order_id__index on order_item (order_id);

create table if not exists order_item_modifier
(
    id            INT AUTO_INCREMENT not null primary key,
    name          VARCHAR(50)        not null,
    price         DECIMAL(19,2) not null,
    order_item_id INT         not null,
    modifier_id   INT         not null,
	
	constraint order_item___fk foreign key (order_item_id) references order_item (id),
    constraint item_modifier___fk foreign key (modifier_id) references product_modifier (id)
);
create index order_item_id__index on order_item_modifier (order_item_id);
create index created_date__index on user (created_date);
create unique index user_mobile_number_uindex on user (mobile_number);

create table if not exists user_address
(
    id          INT AUTO_INCREMENT not null primary key,
    city        VARCHAR(50),
    street      VARCHAR(50),
    home        VARCHAR(50),
    building    VARCHAR(10),
    apartment   VARCHAR(10),
    floor       VARCHAR(5),
    doorphone   VARCHAR(10),
    comment     VARCHAR(255),
    actual      BIT default true,
    terminal_id INT,
    user_id     INT not null,
    
	constraint address_terminal___fk foreign key (terminal_id) references organization_terminal (id),
	constraint address_user___fk foreign key (user_id) references user (id)
);
```

2. Заливаем проект на ВМ и поднимаем docker-compose<br>

![image](https://user-images.githubusercontent.com/41448520/169650634-053f9645-1c29-432b-9400-b0ab8d76d512.png)

3. Подключаемся к контейнеру<br><br>
   проверяем БД<br>
 ![image](https://user-images.githubusercontent.com/41448520/169650669-8c8ed0a5-84fa-4659-90cb-4e33a69a42b0.png)
   
   проверяем таблицы<br>
 ![image](https://user-images.githubusercontent.com/41448520/169650684-ee95a734-009c-4df4-9126-6b0c5f8e059f.png)

4. Смотрим текущее значение inno_db_buffer_size<br>
   ![image](https://user-images.githubusercontent.com/41448520/169881926-bebedb5e-2f4a-438b-9118-46385051bd5f.png)

5. Считаем рекомендованное значение в Gb<br>
   ![image](https://user-images.githubusercontent.com/41448520/169882342-9f236e4c-55ed-4ff7-8bad-4ebaf9cf1a9f.png)

6. Выставляем в custom.conf/my.cfg 1Gb (на ВМ всего 2Gb)<br>
   ![image](https://user-images.githubusercontent.com/41448520/169882842-f55b9e96-fbb6-4f2c-b69a-3862e3c0acb3.png)

7. Перезапускаем контейнер и проверяем настройку<br>
   ![image](https://user-images.githubusercontent.com/41448520/169883144-bfa11a8e-46a8-4fd8-8041-9e79f72e8c91.png)
 
8. Установим и запустим sysbench
   ![image](https://user-images.githubusercontent.com/41448520/169885778-f7b54e6b-b8e9-4b95-8888-00c573231108.png)
