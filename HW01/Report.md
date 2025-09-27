# Домашняя работа Рукомойкин Андрей
## Урок 1
1. Создание проекта 

Skip

2. Настройка SSH-доступа

Skip

3. Установка PostgreSQL

Skip

4. Подключение к PostgreSQL

testdb=# \conninfo
You are connected to database "testdb" as user "postgres" via socket in "/var/run/postgresql" at port "5433".

5. 

\set AUTOCOMMIT off


create table shipments(id serial, product_name text, quantity int, destination text);
insert into shipments(product_name, quantity, destination) values('bananas', 1000, 'Europe');
insert into shipments(product_name, quantity, destination) values('coffee', 500, 'USA');
commit;

Объект существует в обоих сессиях

testdb=# \dt shipments
           List of relations
 Schema |   Name    | Type  |  Owner
--------+-----------+-------+----------
 public | shipments | table | postgres
(1 row)

6. Изучение уровней изоляции

Окно №1

```bash
testdb=# insert into shipments(product_name, quantity, destination) values('sugar', 300, 'Asia');
INSERT 0 1
testdb=*# select * from shipments;
 id | product_name | quantity | destination
----+--------------+----------+-------------
  1 | bananas      |     1000 | Europe
  2 | coffee       |      500 | USA
  3 | sugar        |      300 | Asia
(3 rows)

Окно №2
```bash
testdb=# select * from shipments;
 id | product_name | quantity | destination
----+--------------+----------+-------------
  1 | bananas      |     1000 | Europe
  2 | coffee       |      500 | USA
(2 rows)

Как следует из названия "transaction_isolation = read committed" только те транзакции, которые были commit видны другим пользователям, остальные же видят старые данные на основании MVСС. Однако сразу после commit данные становятся видны всем
```bash
testdb=*# select * from shipments;
 id | product_name | quantity | destination
----+--------------+----------+-------------
  1 | bananas      |     1000 | Europe
  2 | coffee       |      500 | USA
  3 | sugar        |      300 | Asia
(3 rows)


В случае если происходит обрыв сессию, транзакция откатывается обратно. 

7. Эксперименты с уровнем изоляции Repeatable Read

testdb=*# SHOW TRANSACTION ISOLATION LEVEL;
 transaction_isolation
-----------------------
 repeatable read
(1 row)

testdb=*# select * from shipments;
 id | product_name | quantity | destination
----+--------------+----------+-------------
  1 | bananas      |     1000 | Europe
  2 | coffee       |      500 | USA
  3 | sugar        |      300 | Asia

testdb=*# select * from shipments;
 id | product_name | quantity | destination
----+--------------+----------+-------------
  1 | bananas      |     1000 | Europe
  2 | coffee       |      500 | USA
  3 | sugar        |      300 | Asia
  4 | bananas      |     2000 | Africa

В случае Repeatable Read  данные в текущей сессии не буду меняться, даже после завершения и commit в соседней и все данные будут читаться на момент начала транзакции. 
