# 06-db-02-sql - Карпов Андрей devops-21
## Задача 1

Используя docker поднимите инстанс PostgreSQL (версию 12) c 2 volume, 
в который будут складываться данные БД и бэкапы.

Приведите получившуюся команду или docker-compose манифест.
### Ответ:
```
version: '3.8'
services:
  db:
    image: postgres:12-bullseye
    restart: always
    environment:
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=postgres
    ports:
      - '5432:5432'
    volumes:
      - db:/var/lib/postgresql/data
      - backup:/var/backups
volumes:
  db:
    driver: local
  backup:
    driver: local
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

### Ответ:
- итоговый список БД после выполнения пунктов выше,
```
test_db=# \dt
          List of relations
 Schema |  Name   | Type  |  Owner
--------+---------+-------+----------
 public | clients | table | postgres
 public | orders  | table | postgres
(2 rows)
```
- описание таблиц (describe)
```
test_db=# \d+ clients
                                                                   Table "public.clients"
              Column               |       Type        | Collation | Nullable |               Default               | Storage  | Stats target | Description
-----------------------------------+-------------------+-----------+----------+-------------------------------------+----------+--------------+-------------
 id                                | integer           |           | not null | nextval('clients_id_seq'::regclass) | plain    |              |
 фамилия                           | character varying |           |          |                                     | extended |              |
 страна проживания                 | character varying |           |          |                                     | extended |              |
 заказ                             | integer           |           |          |                                     | plain    |              |
Foreign-key constraints:
    "clients_fk" FOREIGN KEY ("заказ") REFERENCES orders(id)
Access method: heap

test_db=# \d+ orders
                                                              Table "public.orders"
          Column          |       Type        | Collation | Nullable |              Default               | Storage  | Stats target | Description
--------------------------+-------------------+-----------+----------+------------------------------------+----------+--------------+-------------
 id                       | integer           |           | not null | nextval('orders_id_seq'::regclass) | plain    |              |
 наименование             | character varying |           |          |                                    | extended |              |
 цена                     | integer           |           |          |                                    | plain    |              |
Indexes:
    "orders_pk" PRIMARY KEY, btree (id)
Referenced by:
    TABLE "clients" CONSTRAINT "clients_fk" FOREIGN KEY ("заказ") REFERENCES orders(id)
Access method: heap

```
- SQL-запрос для выдачи списка пользователей с правами над таблицами test_db
```
test_db=# SELECT
    grantee, table_name, privilege_type
FROM
    information_schema.table_privileges
WHERE
    grantee in ('test-admin-user','test-simple-user')
    and table_name in ('orders','clients') order by grantee,table_name;
```
- список пользователей с правами над таблицами test_db
```
     grantee      | table_name | privilege_type
------------------+------------+----------------
 test-admin-user  | clients    | INSERT
 test-admin-user  | clients    | SELECT
 test-admin-user  | clients    | UPDATE
 test-admin-user  | clients    | DELETE
 test-admin-user  | clients    | TRUNCATE
 test-admin-user  | clients    | REFERENCES
 test-admin-user  | clients    | TRIGGER
 test-admin-user  | orders     | INSERT
 test-admin-user  | orders     | SELECT
 test-admin-user  | orders     | UPDATE
 test-admin-user  | orders     | DELETE
 test-admin-user  | orders     | TRUNCATE
 test-admin-user  | orders     | REFERENCES
 test-admin-user  | orders     | TRIGGER
 test-simple-user | clients    | DELETE
 test-simple-user | clients    | INSERT
 test-simple-user | clients    | SELECT
 test-simple-user | clients    | UPDATE
 test-simple-user | orders     | DELETE
 test-simple-user | orders     | SELECT
 test-simple-user | orders     | UPDATE
 test-simple-user | orders     | INSERT
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

### Ответ:
```
test_db=# INSERT INTO public.orders (наименование,цена) 
  VALUES ('Шоколад',10),('Принтер',3000),('Книга',500),('Монитор',7000),('Гитара',4000);
INSERT 0 5
 
test_db=# INSERT INTO public.clients (фамилия,"страна проживания") VALUES ('Иванов Иван Иванович','USA'),('Петров Петр Петрович','Canada'),('Иоганн Себастьян Бах','Japan'),('Ронни Джеймс Дио','Russia'),('Ritchie Blackmore','Russia');
INSERT 0 5

test_db=# select count(*) from orders;
 count
-------
     5
(1 row)

test_db=# select count(*) from clients;
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

## Задача 5

Получите полную информацию по выполнению запроса выдачи всех пользователей из задачи 4 
(используя директиву EXPLAIN).

Приведите получившийся результат и объясните что значат полученные значения.

## Задача 6

Создайте бэкап БД test_db и поместите его в volume, предназначенный для бэкапов (см. Задачу 1).

Остановите контейнер с PostgreSQL (но не удаляйте volumes).

Поднимите новый пустой контейнер с PostgreSQL.

Восстановите БД test_db в новом контейнере.

Приведите список операций, который вы применяли для бэкапа данных и восстановления. 
