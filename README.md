# Домашнее задание к занятию "6.3. MySQL"

## Введение

Перед выполнением задания вы можете ознакомиться с
[дополнительными материалами](https://github.com/netology-code/virt-homeworks/tree/master/additional/README.md).

## Задача 1

Используя docker поднимите инстанс MySQL (версию 8). Данные БД сохраните в volume.

Изучите [бэкап БД](https://github.com/netology-code/virt-homeworks/tree/master/06-db-03-mysql/test_data) и
восстановитесь из него.

Перейдите в управляющую консоль `mysql` внутри контейнера.

Используя команду `\h` получите список управляющих команд.

Найдите команду для выдачи статуса БД и **приведите в ответе** из ее вывода версию сервера БД.

Подключитесь к восстановленной БД и получите список таблиц из этой БД.

**Приведите в ответе** количество записей с `price` > 300.

В следующих заданиях мы будем продолжать работу с данным контейнером.

#### Ответ

Версия БД:
`Server version:		8.0.30 MySQL Community Server - GPL`

Количество строк с `price` > 300:
```
mysql> select count(*) from orders where price > 300;
+----------+
| count(*) |
+----------+
|        1 |
+----------+
1 row in set (0.01 sec)
```

## Задача 2

Создайте пользователя test в БД c паролем test-pass, используя:
- плагин авторизации mysql_native_password
- срок истечения пароля - 180 дней
- количество попыток авторизации - 3
- максимальное количество запросов в час - 100
- аттрибуты пользователя:
    - Фамилия "Pretty"
    - Имя "James"

Предоставьте привелегии пользователю `test` на операции SELECT базы `test_db`.

Используя таблицу INFORMATION_SCHEMA.USER_ATTRIBUTES получите данные по пользователю `test` и
**приведите в ответе к задаче**.

#### Ответ

```
mysql> CREATE USER 'test'@'localhost' IDENTIFIED BY 'test-pass';

mysql> ALTER USER 'test'@'localhost' ATTRIBUTE
    -> '{"firstname": "James", "lastname": "Pretty"}';

mysql> ALTER USER 'test'@'localhost'
    -> WITH
    -> MAX_QUERIES_PER_HOUR 100
    -> FAILED_LOGIN_ATTEMPTS 3
    -> PASSWORD_LOCK_TIME 2
    -> PASSWORD EXPIRE INTERVAL 180 DAY;

mysql> GRANT SELECT ON test_db.* TO 'test'@'localhost';
Query OK, 0 rows affected, 1 warning (0.00 sec)

mysql> SELECT USER,HOST,ATTRIBUTE FROM INFORMATION_SCHEMA.USER_ATTRIBUTES WHERE USER='test';
+------+-----------+----------------------------------------------+
| USER | HOST      | ATTRIBUTE                                    |
+------+-----------+----------------------------------------------+
| test | localhost | {"lastname": "Pretty", "firstname": "James"} |
+------+-----------+----------------------------------------------+
1 row in set (0.00 sec)
```

## Задача 3

Установите профилирование `SET profiling = 1`.
Изучите вывод профилирования команд `SHOW PROFILES;`.

Исследуйте, какой `engine` используется в таблице БД `test_db` и **приведите в ответе**.

Измените `engine` и **приведите время выполнения и запрос на изменения из профайлера в ответе**:
- на `MyISAM`
- на `InnoDB`

#### Ответ

```
mysql> SET profiling = 1;
Query OK, 0 rows affected, 1 warning (0.00 sec)
```
По-умолчанию у таблицы `orders` `ENGINE` = `InnoDB`
```
mysql> SELECT TABLE_SCHEMA,TABLE_NAME,ENGINE,VERSION FROM information_schema.TABLES where TABLE_NAME = 'orders';
+--------------+------------+--------+---------+
| TABLE_SCHEMA | TABLE_NAME | ENGINE | VERSION |
+--------------+------------+--------+---------+
| test_db      | orders     | InnoDB |      10 |
+--------------+------------+--------+---------+
1 row in set (0.00 sec)
```
Сменив `ENGINE` таблицы, получаем следующие результаты:
```
mysql> ALTER TABLE orders ENGINE=MyISAM;
Query OK, 5 rows affected (0.09 sec)
Records: 5  Duplicates: 0  Warnings: 0

mysql> ALTER TABLE orders ENGINE=InnoDB;
Query OK, 5 rows affected (0.03 sec)
Records: 5  Duplicates: 0  Warnings: 0

mysql> SHOW PROFILES;
+----------+------------+--------------------------------------+
| Query_ID | Duration   | Query                                |
+----------+------------+--------------------------------------+
|       11 | 0.00017675 | SET @@profiling_history_size = 100   |
|       12 | 0.00032350 | ALTER TABLE orders SET ENGINE=MyISAM |
|       13 | 0.09105925 | ALTER TABLE orders ENGINE=MyISAM     |
|       14 | 0.03776225 | ALTER TABLE orders ENGINE=InnoDB     |
+----------+------------+--------------------------------------+
4 rows in set, 1 warning (0.00 sec)
```

## Задача 4

Изучите файл `my.cnf` в директории /etc/mysql.

Измените его согласно ТЗ (движок InnoDB):
- Скорость IO важнее сохранности данных
- Нужна компрессия таблиц для экономии места на диске
- Размер буффера с незакомиченными транзакциями 1 Мб
- Буффер кеширования 30% от ОЗУ
- Размер файла логов операций 100 Мб

Приведите в ответе измененный файл `my.cnf`.

#### Ответ

Файл `my.cnf` после внесения изменений:
```
skip-host-cache
skip-name-resolve
datadir=/var/lib/mysql
socket=/var/run/mysqld/mysqld.sock
secure-file-priv=/var/lib/mysql-files
user=mysql

pid-file=/var/run/mysqld/mysqld.pid
[client]
socket=/var/run/mysqld/mysqld.sock

# Amount of RAM for data cache
innodb_buffer_pool_size = 654M

# Log-file size
innodb_log_file_size = 100M

# The size in bytes of the buffer that InnoDB uses to write to the log files on disk
innodb_log_buffer_size = 1M

# Keep tables saparately in each file (ON) or in one file (OFF)
innodb_file_per_table ON

# Defines the method used to flush data to InnoDB data files and log files
innodb_flush_method = O_DSYNC

# Logs are written after each transaction commit and flushed to disk once per second
innodb_flush_log_at_trx_commit = 2

# Amount of RAM size for cache query
query_cache_size = 0
!includedir /etc/mysql/conf.d/

```
---

### Как оформить ДЗ?

Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.

---
