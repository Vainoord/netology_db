# Домашнее задание к занятию "6.4. PostgreSQL"

## Задача 1

Используя docker поднимите инстанс PostgreSQL (версию 13). Данные БД сохраните в volume.

Подключитесь к БД PostgreSQL используя `psql`.

Воспользуйтесь командой `\?` для вывода подсказки по имеющимся в `psql` управляющим командам.

**Найдите и приведите** управляющие команды для:
- вывода списка БД
- подключения к БД
- вывода списка таблиц
- вывода описания содержимого таблиц
- выхода из psql

#### Ответ

Создание контейнера:
```
docker create \
--name home_psql \
-e POSTGRES_PASSWORD=postgres \
-p 5432:5432 \
-v vl_psql_data:/var/lib/postgresql/data \
-v vl_backups:/var/lib/postgresql/backup \
postgres:13.8

docker start home_psql
```

- вывод списка БД: `\l`
- подключение к БД: `\c`
- вывод списка таблиц БД: `\dt`
- вывод описания содержимого таблиц: `\dS`
- выход из psql: `\q`

## Задача 2

Используя `psql` создайте БД `test_database`.

Изучите [бэкап БД](https://github.com/netology-code/virt-homeworks/tree/master/06-db-04-postgresql/test_data).

Восстановите бэкап БД в `test_database`.

Перейдите в управляющую консоль `psql` внутри контейнера.

Подключитесь к восстановленной БД и проведите операцию ANALYZE для сбора статистики по таблице.

Используя таблицу [pg_stats](https://postgrespro.ru/docs/postgresql/12/view-pg-stats), найдите столбец таблицы `orders`
с наибольшим средним значением размера элементов в байтах.

**Приведите в ответе** команду, которую вы использовали для вычисления и полученный результат.

Вывод операции `ANALYZE`:
```
test_database=# ANALYZE VERBOSE orders;
INFO:  analyzing "public.orders"
INFO:  "orders": scanned 1 of 1 pages, containing 8 live rows and 0 dead rows; 8 rows in sample, 8 estimated total rows
ANALYZE
```
Вывод наибольшего среднего размера элементов в байтах находится в столбце `avg_width`:
```
test_database=# SELECT tablename, attname, avg_width
test_database=# FROM pg_stats
test_database=# WHERE tablename = 'orders'
test_database=# ORDER BY avg_width DESC LIMIT 1
test_database=# ;
 tablename | attname | avg_width
-----------+---------+-----------
 orders    | title   |        16
(1 row)
```

## Задача 3

Архитектор и администратор БД выяснили, что ваша таблица orders разрослась до невиданных размеров и
поиск по ней занимает долгое время. Вам, как успешному выпускнику курсов DevOps в нетологии предложили
провести разбиение таблицы на 2 (шардировать на orders_1 - price>499 и orders_2 - price<=499).

Предложите SQL-транзакцию для проведения данной операции.

Можно ли было изначально исключить "ручное" разбиение при проектировании таблицы orders?

#### Ответ

Создаем две таблицы, с наследованием от orders:
```
test_database=# CREATE TABLE orders_1
test_database=# (CHECK (price > 499))
test_database=# INHERITS (orders)
test_database=# ;

test_database=# CREATE TABLE orders_2
test_database-# (CHECK (price <= 499))
test_database-# INHERITS (orders)
test_database-# ;
```
Добавляем данные из родительской таблицы в дочерние:
```
test_database=# INSERT INTO orders_1
test_database-# SELECT * FROM orders
test_database-# WHERE price > 499
;
INSERT 0 3
test_database=# INSERT INTO orders_2
test_database-# SELECT * FROM orders
test_database-# WHERE price <= 499
test_database-# ;
INSERT 0 5
```
Создаем индексы по столбцу price:  
```
test_database=# CREATE INDEX orders_from_499 on orders_1 (price);
CREATE INDEX
test_database=# CREATE INDEX orders_upto_499 on orders_2 (price);
CREATE INDEX
```
Создаем функцию, которая будет определять таблицу для записи новых данных:
```
test_database=# CREATE OR REPLACE FUNCTION insert_order_trigger()
test_database-# RETURNS TRIGGER AS $$
test_database$# BEGIN
test_database$#   IF (NEW.range > 499) THEN
test_database$#     INSERT INTO orders_1 VALUES (NEW.*);
test_database$#   ELSE
test_database$#     INSERT INTO orders_2 VALUES (NEW.*);
test_database$#   END IF;
test_database$#   RETURN NULL;
test_database$# END;
test_database$# $$
test_database-# LANGUAGE plpgsql;
CREATE FUNCTION
```
Создаем триггер, вызывающий функцию `insert_order_trigger()`:
```
test_database=# CREATE TRIGGER order_insertion
test_database-# BEFORE INSERT ON orders
test_database-# FOR EACH ROW EXECUTE FUNCTION in
test_database-# FOR EACH ROW EXECUTE FUNCTION insert_order_trigger();
CREATE TRIGGER
```
Для существующей таблицы использован метод Partitioning Using Inheritance (две таблицы наследуют колонки родительской таблицы).  
Новую таблицу можно разделить методом Declarative Partitioning, используя диапазоны значений price (но нужно знать минимальные и максимальные значения price, чтобы указать их в диапазоне). Но это также ручной метод разбиения таблицы.


## Задача 4

Используя утилиту `pg_dump` создайте бекап БД `test_database`.

Как бы вы доработали бэкап-файл, чтобы добавить уникальность значения столбца `title` для таблиц `test_database`?

#### Ответ

Бэкап создан:  
`pg_dump -U postgres -d test_database > /var/lib/postgresql/backup/test_database_dump.sql
`  

Уникальность значения добавляется с помощью ограничителя UNIQUE при создании/редактировании таблицы:
```
CREATE TABLE public.orders (
    id integer NOT NULL,
    title character varying(80) NOT NULL UNIQUE,
    price integer DEFAULT 0
);
```
---

### Как cдавать задание

Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.

---
