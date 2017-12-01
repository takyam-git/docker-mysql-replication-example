# usage

```bash
# clone
$ git clone https://github.com/takyam-git/docker-mysql-replication-example.git
# change working directory
$ cd docker-mysql-replication-example
# create master volume
$ docker volume create --name=mysql_volume
# launch master mysql
$ docker-compose up -d mysql
# create database
$ mysql -u root -h 127.0.0.1 -e "CREATE DATABASE repl_example;"
$ mysql -u root -h 127.0.0.1 -e "SHOW DATABASES;"
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| repl_example       |
| sys                |
+--------------------+
# create table
$ mysql -uroot -h 127.0.0.1 -e "CREATE TABLE repl_example.test (id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY, foo VARCHAR(255) NOT NULL, bar TEXT);"
$ mysql -uroot -h 127.0.0.1 -e "SHOW TABLES FROM repl_example;"
+------------------------+
| Tables_in_repl_example |
+------------------------+
| test                   |
+------------------------+
$ mysql -uroot -h 127.0.0.1 -e "SHOW CREATE TABLE repl_example.test;"
+-------+----------------------------------------+
| Table | Create Table                           |
+-------+----------------------------------------+
| test  | CREATE TABLE `test` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
  `foo` varchar(255) NOT NULL,
  `bar` text,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 |
+-------+----------------------------------------+
$ mysql -uroot -h 127.0.0.1 -e "SELECT * FROM repl_example.test;"
# insert records
$ mysql -uroot -h 127.0.0.1 -e "INSERT INTO repl_example.test (foo, bar) VALUES ('foo1', 'bar1'), ('foo2', 'bar2'), ('foo3', 'bar3');"
$ mysql -uroot -h 127.0.0.1 -e "SELECT * FROM repl_example.test;"
+----+------+------+
| id | foo  | bar  |
+----+------+------+
|  1 | foo1 | bar1 |
|  2 | foo2 | bar2 |
|  3 | foo3 | bar3 |
+----+------+------+
# launch slave
$ docker-compose up -d mysql-slave
dockermysqlreplicationexample_mysql_1 is up-to-date
Creating dockermysqlreplicationexample_mysql-slave_1
# check replicated
$ mysql -u root -h 127.0.0.1 --port 13306 -e "SHOW DATABASES;"
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| repl_example       |
| sys                |
+--------------------+
$ mysql -uroot -h 127.0.0.1 --port 13306 -e "SELECT * FROM repl_example.test;"
+----+------+------+
| id | foo  | bar  |
+----+------+------+
|  1 | foo1 | bar1 |
|  2 | foo2 | bar2 |
|  3 | foo3 | bar3 |
+----+------+------+
# insert new record to master
$ mysql -uroot -h 127.0.0.1 -e "INSERT INTO repl_example.test (foo, bar) VALUES ('foo4', 'bar4');"
# check inserted on master
$ mysql -uroot -h 127.0.0.1 -e "SELECT * FROM repl_example.test;"
+----+------+------+
| id | foo  | bar  |
+----+------+------+
|  1 | foo1 | bar1 |
|  2 | foo2 | bar2 |
|  3 | foo3 | bar3 |
|  4 | foo4 | bar4 |
+----+------+------+
# check replicated to slave
$ mysql -uroot -h 127.0.0.1 --port 13306 -e "SELECT * FROM repl_example.test;"
+----+------+------+
| id | foo  | bar  |
+----+------+------+
|  1 | foo1 | bar1 |
|  2 | foo2 | bar2 |
|  3 | foo3 | bar3 |
|  4 | foo4 | bar4 |
+----+------+------+
```
