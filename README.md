# 12_07 Стасенко Григорий Домашнее задание к занятию «Репликация и масштабирование. Часть 2» 12_07

## Задание 1
Опишите основные преимущества использования масштабирования методами:

* активный master-сервер и пассивный репликационный slave-сервер;
* master-сервер и несколько slave-серверов;
* активный сервер со специальным механизмом репликации — distributed replicated block device (DRBD);
* SAN-кластер.
Дайте ответ в свободной форме.

````
 * активный master-сервер и пассивный репликационный slave-сервер:
   Позволяет сократить время, затрачиваемое на чтение данных из БД, защитить данные от потери, в случае отказа master.
   Скорость записи остается неизменной. В случае падения master, slave занимает его место.

 * master-сервер и несколько slave-серверов:
   Аналогично предыдущему пункту, но больше отказоустойчивости ввиду избыточности серверов.

 * активный сервер со специальным механизмом репликации — distributed replicated block device (DRBD):
   Позволяет увеличить скорость чтения и записи данных, а так же повысить отказоустойчивость за счет технологии отдаленно напоминающей RAID.

 * SAN-кластер:
   Позволяет получить наилучшую производительность и отказоустойчивость, благодаря масштабируемости, высокой гибкости, быстродействию и удобному централизированному управлению.

````
---

## Задание 2
Разработайте план для выполнения горизонтального и вертикального шаринга базы данных. База данных состоит из трёх таблиц:

пользователи,
книги,
магазины (столбцы произвольно).
Опишите принципы построения системы и их разграничение или разбивку между базами данных.

Пришлите блоксхему, где и что будет располагаться. Опишите, в каких режимах будут работать сервера.

Допустим, мы имеем следующую БД:

![image](https://github.com/Nightnek/HW_12_07/assets/127677631/d569306d-9b21-4708-85e8-4cb555ec07e9)


Вертикальное шардирование:
Таблицы магазинов и таблицы книг мы расположим на одном сервере, таблицу клиентов- на другом.
Дело в том, что у клиентов может быть по несколько книг.
В случае, если сеть магазинов разрастется и увеличится, мы сможем распределить их на разные машины.

Для реализации данной схемы:
1) готовим 3 машины
2) на одной создаем таблицу Customers
3) на второй создаем Stores и Books
4) Настраиваем DB Server и заполняем данными нашу БД.

![image](https://github.com/Nightnek/HW_12_07/assets/127677631/89ee767e-40d0-41a1-b71c-18bcae30ccd1)


Горизонтальное шардирование:
1) готовим 3 машины
2) копируем БД на 2 машины для хранения
3) наодной оставляем четные ID
4) на второй оставляем не четные ID
5)  Настраиваем DB Server

![image](https://github.com/Nightnek/HW_12_07/assets/127677631/5bea5c2c-d2df-46b0-bc85-1be7843f9f6e)


---

## Задание 3*
Выполните настройку выбранных методов шардинга из задания 2.

Пришлите конфиг Docker и SQL скрипт с командами для базы данных.

````
Докер:

version: '2.0'

services:
  db:
    image: mysql:8.3
    container_name: db
    networks:
      - repl_sql
    ports:
      - 3306:3306
      - 33060:33060
    restart: always
    environment:
        MYSQL_ROOT_PASSWORD: 12345
    command:
        - --mysql-native-password=ON
    volumes:
      - ./mysql_repl/mysql:/var/lib/mysql
      - ./mysql_repl/my.cnf:/etc/my.cnf:ro

  db1:
    image: mysql:8.3
    container_name: db1
    networks:
      - repl_sql
    restart: always
    depends_on:
      - db
    environment:
      MYSQL_ROOT_PASSWORD: 12345
    command:
      - --mysql-native-password=ON
    volumes:
    - ./mysql_repl1/mysql:/var/lib/mysql
    - ./mysql_repl1/my.cnf:/etc/my.cnf:ro

  db2:
    image: mysql:8.3
    container_name: db2
    networks:
      - repl_sql
    restart: always
    depends_on:
      - db
    environment:
      MYSQL_ROOT_PASSWORD: 12345
    command:
      - --mysql-native-password=ON
    volumes:
      - ./mysql_repl2/mysql:/var/lib/mysql
      - ./mysql_repl2/my.cnf:/etc/my.cnf:ro

networks:
  repl_sql:
    driver: bridge

Настройка Мастера:
CREATE USER repl@'%';
GRANT REPLICATION SLAVE ON *.* TO repl@'%';

Настройка Слейв:
change master to master_host='db', master_user='repl', master_log_file='mysql-bin.000003', master_log_pos=602, get_master_public_key =
 1;

Проверка работы:
mysql> SHOW PROCESSLIST;
+----+-----------------+------------------+------+-------------+------+-----------------------------------------------------------------+------------------+
| Id | User            | Host             | db   | Command     | Time | State                                                           | Info             |
+----+-----------------+------------------+------+-------------+------+-----------------------------------------------------------------+------------------+
|  5 | event_scheduler | localhost        | NULL | Daemon      |  426 | Waiting on empty queue                                          | NULL             |
| 10 | root            | localhost        | NULL | Query       |    0 | init                                                            | SHOW PROCESSLIST |
| 25 | repl            | 172.19.0.4:43598 | NULL | Binlog Dump |   28 | Source has sent all binlog to replica; waiting for more updates | NULL             |
| 26 | repl            | 172.19.0.3:44476 | NULL | Binlog Dump |    0 | Source has sent all binlog to replica; waiting for more updates | NULL             |
+----+-----------------+------------------+------+-------------+------+-----------------------------------------------------------------+------------------+

Создаем БД:
CREATE DATABASE test;

Создаем таблицу:
create table users ( id INT NOT NULL PRIMARY KEY, user_id int NOT NULL unique, name VARCHAR(50) NOT NULL, department VARCHAR(50));



````
---
