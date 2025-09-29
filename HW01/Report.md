1. Создание проекта 

ssh -i /var/lib/postgresql/.ssh/id_rsa rukomoykinav@89.169.188.99

2. Настройка SSH-доступа

postgres@Work01:~$ ssh-keygen -t rsa -b 2048
```bash
Generating public/private rsa key pair.
Enter file in which to save the key (/var/lib/postgresql/.ssh/id_rsa):
Created directory '/var/lib/postgresql/.ssh'.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /var/lib/postgresql/.ssh/id_rsa
Your public key has been saved in /var/lib/postgresql/.ssh/id_rsa.pub
The key fingerprint is:
SHA256:hLPpbbNMBB/nPWFgdo1HyE4XfiulfesS5KEpy/ShckM postgres@Work01
The key's randomart image is:
+---[RSA 2048]----+
|          +..=o. |
|       . o o=.+  |
|      + o .ooo...|
|       B + o.++..|
|      o S . Bo.o.|
|     . o E + +. o|
|      . O = . .. |
|       = O . ..  |
|        = .   .. |
+----[SHA256]-----+
postgres@Work01:~$ ls -l /var/lib/postgresql/.ssh/id_rsa
-rw------- 1 postgres postgres 1876 сен 28 02:57 /var/lib/postgresql/.ssh/id_rsa
postgres@Work01:~$ chmod 600  /var/lib/postgresql/.ssh/id_rsa
postgres@Work01:~$ ls -l /var/lib/postgresql/.ssh/id_rsa
-rw------- 1 postgres postgres 1876 сен 28 02:57 /var/lib/postgresql/.ssh/id_rsa
postgres@Work01:~$ cd /var/lib/postgresql/.ssh/
postgres@Work01:~/.ssh$ ls -latr
total 16
drwxr-xr-x 10 postgres postgres 4096 сен 28 02:57 ..
-rw-------  1 postgres postgres 1876 сен 28 02:57 id_rsa
-rw-r--r--  1 postgres postgres  397 сен 28 02:57 id_rsa.pub
drwx------  2 postgres postgres 4096 сен 28 02:57 .
postgres@Work01:~/.ssh$ chmod 600 id_rsa.pub
postgres@Work01:~/.ssh$ ls -latr
total 16
drwxr-xr-x 10 postgres postgres 4096 сен 28 02:57 ..
-rw-------  1 postgres postgres 1876 сен 28 02:57 id_rsa
-rw-------  1 postgres postgres  397 сен 28 02:57 id_rsa.pub
drwx------  2 postgres postgres 4096 сен 28 02:57 .
```
3. Установка PostgreSQL

sudo apt update && sudo apt upgrade -y && sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list' && wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add - && sudo apt-get update && sudo apt-get -y install postgresql && sudo apt install unzip && sudo apt -y install mc

4. Подключение к PostgreSQL

postgres=# \conninfo
           Connection Information
      Parameter       |        Value
----------------------+---------------------
 Database             | postgres
 Client User          | postgres
 Socket Directory     | /var/run/postgresql
 Server Port          | 5432
 Options              |
 Protocol Version     | 3.0
 Password Used        | false
 GSSAPI Authenticated | false
 Backend PID          | 5119
 SSL Connection       | false
 Superuser            | on
 Hot Standby          | off
(12 rows)


postgres=# SELECT version();
-[ RECORD 1 ]------------------------------------------------------------------------------------------------------------------------------
version | PostgreSQL 18.0 (Ubuntu 18.0-1.pgdg24.04+3) on x86_64-pc-linux-gnu, compiled by gcc (Ubuntu 13.3.0-6ubuntu2~24.04) 13.3.0, 64-bit

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
