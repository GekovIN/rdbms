# ДЗ 4

## DDL скрипты для PostgreSQL

```sql
-- Создаем БД и пользователя от имени суперюзера postgres. Даем права на БД пользователю.

DROP DATABASE IF EXISTS kitchen_delivery;
CREATE DATABASE kitchen_delivery;

DROP USER IF EXISTS developer;
CREATE USER developer WITH PASSWORD 'dev_pass';

GRANT ALL PRIVILEGES ON DATABASE kitchen_delivery TO developer;

-- Подключаемся в БД kitchen_delivery от имени пользователя developer
```
![image](https://user-images.githubusercontent.com/41448520/147409233-35337fb7-a619-4580-88b2-4df39bf18858.png)

```sql
-- Создаем схему
DROP SCHEMA IF EXISTS kitchen CASCADE;
CREATE SCHEMA kitchen;

-- Создаем таблицы

create table if not exists kitchen.product_modifier
(
    id          bigserial
        constraint product_modifier_pk
            primary key,
    name        varchar,
    description varchar,
    code        varchar,
    price       numeric(19, 2),
    active      boolean
);

create table if not exists kitchen.organization
(
    id      bigserial
        constraint organization_pk
            primary key,
    name    varchar not null,
    address varchar,
    active  boolean default true
);

create table if not exists kitchen.product_category
(
    id              bigserial
        constraint product_category_pk
            primary key,
    name            varchar not null,
    description     varchar,
    active          boolean,
    organization_id bigint  not null
        constraint organization___fk
            references kitchen.organization
);

create table if not exists kitchen.product
(
    id                  bigserial
        constraint product_pk
            primary key,
    name                varchar        not null,
    description         varchar,
    code                varchar,
    price               numeric(19, 2) not null,
    active              boolean default true,
    product_category_id bigint
        constraint product_category___fk
            references kitchen.product_category
);

create table if not exists kitchen.modifier_to_product
(
    product_id  bigint not null
        constraint product__fk
            references kitchen.product,
    modifier_id bigint not null
        constraint modifier___fk
            references kitchen.product_modifier
);

alter table kitchen.modifier_to_product
    owner to developer;

create table if not exists kitchen.organization_terminal
(
    id              bigserial
        constraint organization_terminal_pk
            primary key,
    name            varchar not null,
    address         varchar,
    active          boolean default true,
    organization_id bigint  not null
        constraint organization___fk
            references kitchen.organization
);


create table if not exists kitchen.payment
(
    id           bigserial
        constraint payment_pk
            primary key,
    type         varchar   not null,
    status       varchar,
    created_date timestamp not null,
    updated_date timestamp
);

create table if not exists kitchen."user"
(
    id            bigserial
        constraint user_pk
            primary key,
    name          varchar,
    email         varchar,
    mobile_number varchar   not null,
    birth_date    date,
    enabled       boolean default true,
    created_date  timestamp not null,
    updated_date  timestamp
);

create table if not exists kitchen."order"
(
    id            bigserial
        constraint order_pk
            primary key,
    order_number  varchar        not null,
    total_cost    numeric(19, 2) not null,
    status        varchar        not null,
    status_reason varchar,
    created_date  timestamp      not null,
    updated_date  timestamp,
    terminal_id   bigint
        constraint terminal___fk
            references kitchen.organization_terminal,
    user_id       bigint         not null
        constraint user___fk
            references kitchen."user",
    payment_id    integer
        constraint payment___fk
            references kitchen.payment
);

create index if not exists order__number_index
    on kitchen."order" (order_number);

create index if not exists order__create_date_index
    on kitchen."order" (created_date);

create table if not exists kitchen.order_item
(
    id         bigserial
        constraint order_item_pk
            primary key,
    name       varchar        not null,
    price      numeric(19, 2) not null,
    order_id   bigint         not null
        constraint order___fk
            references kitchen."order",
    product_id bigint         not null
        constraint product___fk
            references kitchen.product
);

create index if not exists order_id__index
    on kitchen.order_item (order_id);

create table if not exists kitchen.order_item_modifier
(
    id            bigserial
        constraint order_item_modifier_pk
            primary key,
    name          varchar        not null,
    price         numeric(19, 2) not null,
    order_item_id bigint         not null
        constraint order_item___fk
            references kitchen.order_item,
    modifier_id   bigint         not null
        constraint modifier___fk
            references kitchen.product_modifier
);

create index if not exists order_item_id__index
    on kitchen.order_item_modifier (order_item_id);

create index if not exists created_date__index
    on kitchen."user" (created_date);

create unique index if not exists user_mobile_number_uindex
    on kitchen."user" (mobile_number);

create table if not exists kitchen.user_address
(
    id          bigserial
        constraint user_address_pk
            primary key,
    city        varchar,
    street      varchar,
    home        varchar,
    building    varchar,
    apartment   varchar,
    floor       varchar,
    doorphone   varchar,
    comment     varchar,
    actual      boolean default true,
    terminal_id bigint
        constraint terminal___fk
            references kitchen.organization_terminal,
    user_id     bigint not null
        constraint user___fk
            references kitchen."user"
);


```
