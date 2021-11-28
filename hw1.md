# ДЗ 1

## Схема БД для сервиса доставки готовых блюд

[СХЕМА](https://dbdiagram.io/d/61a362748c901501c0d5c984)

Сервис представляет собой услугу по доставке готовых блюд на указанный пользователем адрес.
<br/>&emsp;При регистрации пользователь указывает свой адрес, который привязывается к соответствующему терминалу доставки.
В зависимости от выбранного адреса пользователю демонстрируется меню доставки соответствующей организации.
<br/> &emsp;У пользователя есть возможность выбрать конкретные блюда (product), а также модификаторы к ним (product_modifier).
При осуществлении заказа создаются сущности order_item и order_item_modifier для поддержания истории заказа пользователя (если делать связть непосредственно order -> product, то при изменении меню, изменится и история пользователя - цены, названия и т.п.)
<br/>&emsp;У пользователя имеется возможность совершить оплату онлайн, либо при получении заказа. За за тип и статус заказа отвечает таблица payment.

### Сущности БД:

1. **(organization)** Организация - сеть общественного питания, осуществляющая приготовление и доставку блюд
2. **(organization_terminal)** Терминал организации - непосредственная точка приготовления и доставки заказа
3. **(product_category)** Категория продуктов (блюд)
4. **(product)** Продукт (блюдо)
5. **(product_modifier)** Модификатор продукта (блюда)
6. **(modifier_to_product)** Список модификаторов доступных конкретному продукту (блюду)
7. **(order)** Заказ
8. **(order_item)** Элемент заказа (выбранный продукт\блюдо). 
9. **(order_item_modifier)** Модификатор, выбранный к элементу заказа
10. **(payment)** Оплата за заказ
11. **(user)** Клиент
12. **(user_address)** Адресс клиента (доставки)

### Отношения:

organization_terminal -> organization (many-to-one)<br/>
product_category -> organization (many-to-one)<br/>
product -> product_category (many-to-one)<br/>
product_modifier -> product (many-to-many через таблицу modifier_to_product)<br/>
order -> order_terminal (many-to-one)<br/>
order -> user (many-to-one)<br/>
order_item -> order (many-to-one)<br/>
order_item -> product (many-to-one)<br/>
order_item_modifier -> order_item (many_to_one)<br/>
order_item_modifier -> product_modifier (many_to_one)<br/>
payment -> order (one-to-one)<br/>
user_address -> user (many-to-one)<br/>
user_address -> organization_terminal (many-to_one)<br/>

