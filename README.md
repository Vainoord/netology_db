# Домашнее задание к занятию "6.2. SQL"

## Введение

Перед выполнением задания вы можете ознакомиться с
[дополнительными материалами](https://github.com/netology-code/virt-homeworks/tree/master/additional/README.md).

## Задача 1

Используя docker поднимите инстанс PostgreSQL (версию 12) c 2 volume,
в который будут складываться данные БД и бэкапы.

Приведите получившуюся команду или docker-compose манифест.

#### Ответ

```
docker pull postgres:12.12-alpine3.16
docker volume create vl_psql_data
docker volume create vl_psql_backup

docker create \
--name home_psql \
-e POSTGRES_PASSWORD=postgres \
-p 5432:5432 \
-v vl_psql_data:/var/lib/postgresql/data \
-v vl_psql_backup:/var/lib/postgresql/backup \
postgres:12.12-alpine3.16

docker start home_psql
```
## Задача 2

В БД из задачи 1:
- создайте пользователя test-admin-user и БД test_db
- в БД test_db создайте таблицу orders и clients (спeцификация таблиц ниже)
- предоставьте привилегии на все операции пользователю test-admin-user на таблицы БД test_db
- создайте пользователя test-simple-user  
- предоставьте пользователю test-simple-user права на SELECT/INSERT/UPDATE/DELETE данных таблиц БД test_db

Таблица orders:
- id (serial primary key)
- наименование (string)
- цена (integer)

Таблица clients:
- id (serial primary key)
- фамилия (string)
- страна проживания (string, index)
- заказ (foreign key orders)

Приведите:
- итоговый список БД после выполнения пунктов выше,
- описание таблиц (describe)
- SQL-запрос для выдачи списка пользователей с правами над таблицами test_db
- список пользователей с правами над таблицами test_db

#### Ответ

Список баз:
```
test_db=# \l
                                 List of databases
   Name    |  Owner   | Encoding |  Collate   |   Ctype    |   Access privileges   
-----------+----------+----------+------------+------------+-----------------------
 postgres  | postgres | UTF8     | en_US.utf8 | en_US.utf8 |
 template0 | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres          +
           |          |          |            |            | postgres=CTc/postgres
 template1 | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres          +
           |          |          |            |            | postgres=CTc/postgres
 test_db   | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =Tc/postgres         +
           |          |          |            |            | postgres=CTc/postgres
(4 rows)
```

Список пользователей:
```
test_db=# \du
                                       List of roles
    Role name     |                         Attributes                         | Member of
------------------+------------------------------------------------------------+-----------
 postgres         | Superuser, Create role, Create DB, Replication, Bypass RLS | {}
 test-admin-user  | Superuser, Cannot login                                    | {}
 test-simple-user |                                                            | {}
```
Список таблиц базы test_db:
```
test_db=# \d+ clients
                                   Table "public.clients"
  Column  |  Type   | Collation | Nullable | Default | Storage  | Stats target | Description
----------+---------+-----------+----------+---------+----------+--------------+-------------
 id       | integer |           |          |         | plain    |              |
 fullname | text    |           |          |         | extended |              |
 country  | text    |           |          |         | extended |              |
 purchase | integer |           |          |         | plain    |              |
Foreign-key constraints:
    "clients_purchase_fkey" FOREIGN KEY (purchase) REFERENCES orders(id)
Access method: heap

test_db=# \d+ orders
                                   Table "public.orders"
 Column |  Type   | Collation | Nullable | Default | Storage  | Stats target | Description
--------+---------+-----------+----------+---------+----------+--------------+-------------
 id     | integer |           | not null |         | plain    |              |
 name   | text    |           |          |         | extended |              |
 price  | integer |           |          |         | plain    |              |
Indexes:
    "orders_pkey" PRIMARY KEY, btree (id)
Referenced by:
    TABLE "clients" CONSTRAINT "clients_purchase_fkey" FOREIGN KEY (purchase) REFERENCES orders(id)
Access method: heap
```
Права пользователя test-admin-user:
```
test_db=# SELECT * FROM information_schema.table_privileges WHERE grantee in ('test-admin-user');
 grantor  |     grantee     | table_catalog | table_schema | table_name | privilege_type | is_grantable | with_hierarchy
----------+-----------------+---------------+--------------+------------+----------------+--------------+----------------
 postgres | test-admin-user | test_db       | public       | orders     | INSERT         | YES          | NO
 postgres | test-admin-user | test_db       | public       | orders     | SELECT         | YES          | YES
 postgres | test-admin-user | test_db       | public       | orders     | UPDATE         | YES          | NO
 postgres | test-admin-user | test_db       | public       | orders     | DELETE         | YES          | NO
 postgres | test-admin-user | test_db       | public       | orders     | TRUNCATE       | YES          | NO
 postgres | test-admin-user | test_db       | public       | orders     | REFERENCES     | YES          | NO
 postgres | test-admin-user | test_db       | public       | orders     | TRIGGER        | YES          | NO
 postgres | test-admin-user | test_db       | public       | clients    | INSERT         | YES          | NO
 postgres | test-admin-user | test_db       | public       | clients    | SELECT         | YES          | YES
 postgres | test-admin-user | test_db       | public       | clients    | UPDATE         | YES          | NO
 postgres | test-admin-user | test_db       | public       | clients    | DELETE         | YES          | NO
 postgres | test-admin-user | test_db       | public       | clients    | TRUNCATE       | YES          | NO
 postgres | test-admin-user | test_db       | public       | clients    | REFERENCES     | YES          | NO
 postgres | test-admin-user | test_db       | public       | clients    | TRIGGER        | YES          | NO
(14 rows)
```
Права пользователя test-simple-user:
```
test_db=# SELECT * FROM information_schema.table_privileges WHERE grantee in ('test-simple-user');
 grantor  |     grantee      | table_catalog | table_schema | table_name | privilege_type | is_grantable | with_hierarchy
----------+------------------+---------------+--------------+------------+----------------+--------------+----------------
 postgres | test-simple-user | test_db       | public       | orders     | INSERT         | NO           | NO
 postgres | test-simple-user | test_db       | public       | orders     | SELECT         | NO           | YES
 postgres | test-simple-user | test_db       | public       | orders     | UPDATE         | NO           | NO
 postgres | test-simple-user | test_db       | public       | orders     | DELETE         | NO           | NO
 postgres | test-simple-user | test_db       | public       | clients    | INSERT         | NO           | NO
 postgres | test-simple-user | test_db       | public       | clients    | SELECT         | NO           | YES
 postgres | test-simple-user | test_db       | public       | clients    | UPDATE         | NO           | NO
 postgres | test-simple-user | test_db       | public       | clients    | DELETE         | NO           | NO
(8 rows)
```

## Задача 3

Используя SQL синтаксис - наполните таблицы следующими тестовыми данными:

Таблица orders

|Наименование|цена|
|------------|----|
|Шоколад| 10 |
|Принтер| 3000 |
|Книга| 500 |
|Монитор| 7000|
|Гитара| 4000|

Таблица clients

|ФИО|Страна проживания|
|------------|----|
|Иванов Иван Иванович| USA |
|Петров Петр Петрович| Canada |
|Иоганн Себастьян Бах| Japan |
|Ронни Джеймс Дио| Russia|
|Ritchie Blackmore| Russia|

Используя SQL синтаксис:
- вычислите количество записей для каждой таблицы
- приведите в ответе:
    - запросы
    - результаты их выполнения.

#### Ответ

Добавление данных в таблицы:
```
test_db=# INSERT INTO clients (id,fullname,country) VALUES (1,'Иванов Иван Иванович','USA');
test_db=# INSERT INTO clients (id,fullname,country) VALUES (2,'Петров Петр Петрович','Canada');
test_db=# INSERT INTO clients (id,fullname,country) VALUES (3,'Иоганн Себастьян Бах','Japan');
test_db=# INSERT INTO clients (id,fullname,country) VALUES (4,'Ронни Джеймс Дио','Russia');
test_db=# INSERT INTO clients (id,fullname,country) VALUES (5,'Ritchie Blackmore','Russia');

test_db=# INSERT INTO orders (id,name,price) VALUES (1,'Шоколад',10);
test_db=# INSERT INTO orders (id,name,price) VALUES (2,'Принтер',3000);
test_db=# INSERT INTO orders (id,name,price) VALUES (3,'Книга',500);
test_db=# INSERT INTO orders (id,name,price) VALUES (4,'Монитор',7000);
test_db=# INSERT INTO orders (id,name,price) VALUES (5,'Гитара',4000);
```
Количество записей в каждой таблице:
```
test_db=# SELECT COUNT(*) FROM clients;
 count
-------
     5
(1 row)

test_db=# SELECT COUNT(*) FROM orders;
 count
-------
     5
(1 row)
```
## Задача 4

Часть пользователей из таблицы clients решили оформить заказы из таблицы orders.

Используя foreign keys свяжите записи из таблиц, согласно таблице:

|ФИО|Заказ|
|------------|----|
|Иванов Иван Иванович| Книга |
|Петров Петр Петрович| Монитор |
|Иоганн Себастьян Бах| Гитара |

Приведите SQL-запросы для выполнения данных операций.

Приведите SQL-запрос для выдачи всех пользователей, которые совершили заказ, а также вывод данного запроса.

Подсказк - используйте директиву `UPDATE`.

#### Ответ

Обновление базы клиентов.
```
test_db=# UPDATE clients SET purchase=3 WHERE fullname='Иванов Иван Иванович';
UPDATE 1
test_db=# UPDATE clients SET purchase=4 WHERE fullname='Петров Петр Петрович';
UPDATE 1
test_db=# UPDATE clients SET purchase=5 WHERE fullname='Иоганн Себастьян Бах';
UPDATE 1
```
Выборка клиентов, которые делали заказ:
```
test_db=# SELECT id,fullname,purchase FROM clients WHERE purchase is not null;
 id |       fullname       | purchase
----+----------------------+----------
  1 | Иванов Иван Иванович |        3
  2 | Петров Петр Петрович |        4
  3 | Иоганн Себастьян Бах |        5
(3 rows)
```

## Задача 5

Получите полную информацию по выполнению запроса выдачи всех пользователей из задачи 4
(используя директиву EXPLAIN).

Приведите получившийся результат и объясните что значат полученные значения.

#### Ответ

```
test_db=# EXPLAIN ANALYZE VERBOSE SELECT id,fullname,purchase FROM clients WHERE purchase is not null;
                                                 QUERY PLAN                                                 
------------------------------------------------------------------------------------------------------------
 Seq Scan on public.clients  (cost=0.00..18.10 rows=806 width=40) (actual time=0.012..0.014 rows=3 loops=1)
   Output: id, fullname, purchase
   Filter: (clients.purchase IS NOT NULL)
   Rows Removed by Filter: 2
 Planning Time: 0.038 ms
 Execution Time: 0.029 ms
(6 rows)
```
EXPLAIN показывает следующие параметры:
 - Объекты сканирования для получения информации (таблица clients);
 - Стоимость запроса (cost в произвольных единицах);
 - Фактическое время выполнения запроса (actual time в миллисекундах) - показано из-за параметра ANALYZE в запросе. А также количество строк, которое вернулось при выполнении запроса и количество обращений к узлу запроса (таблице);
 - Что будет выведено (Output);
 - Фильтры запроса (Filter).

## Задача 6

Создайте бэкап БД test_db и поместите его в volume, предназначенный для бэкапов (см. Задачу 1).

Остановите контейнер с PostgreSQL (но не удаляйте volumes).

Поднимите новый пустой контейнер с PostgreSQL.

Восстановите БД test_db в новом контейнере.

Приведите список операций, который вы применяли для бэкапа данных и восстановления.

#### Ответ

Экспорт базы test_db и пользователей:
```
bash-5.1# pg_dump -U postgres test_db -f /var/lib/postgresql/backup/dump-test_db.sql

bash-5.1# pg_dumpall -h localhost -p 5432 -U postgres -v --roles-only -f /var/lib/postgres/backup/useraccts.sql

bash-5.1# ls -la /var/lib/postgresql/backup/
total 16
drwx------    2 postgres postgres      4096 Oct 10 16:37 .
drwxr-xr-x    1 postgres postgres      4096 Oct 10 18:42 ..
-rw-r--r--    1 root     root          2702 Oct 10 14:56 dump-test_db.sql
-rw-r--r--    1 root     root           753 Oct 10 16:37 useraccts.sql
```
Создаем новый контейнер home_psql2 с двумя volumes, data2 и backup:
```
docker volume create vl_psql_data2

docker create \
--name home_psql2 \
-e POSTGRES_PASSWORD=postgres \
-p 5433:5432 \
-v vl_psql_data2:/var/lib/postgresql/data \
-v vl_psql_backup:/var/lib/postgresql/backup \
postgres:12.12-alpine3.16

docker start home_psql2

docker exec -ti home_psql2 /bin/bash
```
Импортируем данные пользователей и создаем пустую базу test_db:
```
bash-5.1# psql -h localhost -d postgres -U postgres -f /var/lib/postgresql/backup/useraccts.sql
SET
SET
SET
psql:/var/lib/postgresql/backup/useraccts.sql:16: ERROR:  role "postgres" already exists
ALTER ROLE
CREATE ROLE
ALTER ROLE
CREATE ROLE
ALTER ROLE

postgres=# CREATE DATABASE test_db;
```
Импортируем данные из дампа в базу:
```
bash-5.1# psql -U postgres -d test_db -f /var/lib/postgresql/backup/dump-test_db.sql
SET
SET
SET
SET
SET
 set_config
------------

(1 row)

SET
SET
SET
SET
SET
SET
CREATE TABLE  
ALTER TABLE
COMMENT
CREATE TABLE
ALTER TABLE
COMMENT
COPY 5
COPY 5
ALTER TABLE
ALTER TABLE
GRANT
GRANT
GRANT
GRANT
```
---

### Как cдавать задание

Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.

---
