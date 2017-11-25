---
title: "Oracle, Mysql and Postgres Scripts"
date: 2017-11-24T11:07:39-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

For database access, we are going to prepare three scripts for Oracle, Mysql and Postgres.
The three databases are all relational databases and the SQL scripts are very similar. There
a slightly differences in the DDL and you should only focus on the database your are going
to use for your project and ignore others. Also, the framework support other type of SQL and
NoSQL databases as well but you might need to update pom.xml and service.yml to add dependencies
and database configurations.



Oracle
```
DROP TABLE world CASCADE CONSTRAINTS;
CREATE TABLE  world (
  id int NOT NULL,
  randomNumber int NOT NULL,
  PRIMARY KEY  (id)
);

BEGIN
FOR loop_counter IN 1..10 LOOP
INSERT INTO world (id, randomNumber)
VALUES (loop_counter, dbms_random.value(1,10)
       );
END LOOP;
COMMIT;
END;

DROP TABLE fortune CASCADE CONSTRAINTS;
CREATE TABLE fortune (
  id int NOT NULL,
  message varchar2(2048) NOT NULL,
  PRIMARY KEY  (id)
);

INSERT INTO fortune (id, message) VALUES (1, 'fortune: No such file or directory');
INSERT INTO fortune (id, message) VALUES (2, 'A computer scientist is someone who fixes things that aren''t broken.');
INSERT INTO fortune (id, message) VALUES (3, 'After enough decimal places, nobody gives a damn.');
INSERT INTO fortune (id, message) VALUES (4, 'A bad random number generator: 1, 1, 1, 1, 1, 4.33e+67, 1, 1, 1');
INSERT INTO fortune (id, message) VALUES (5, 'A computer program does what you tell it to do, not what you want it to do.');
INSERT INTO fortune (id, message) VALUES (6, 'Emacs is a nice operating system, but I prefer UNIX. — Tom Christaensen');
INSERT INTO fortune (id, message) VALUES (7, 'Any program that runs right is obsolete.');
INSERT INTO fortune (id, message) VALUES (8, 'A list is only as strong as its weakest link. — Donald Knuth');
INSERT INTO fortune (id, message) VALUES (9, 'Feature: A bug with seniority.');
INSERT INTO fortune (id, message) VALUES (10, 'Computers make very fast, very accurate mistakes.');
INSERT INTO fortune (id, message) VALUES (11, '<script>alert("This should not be displayed in a browser alert box.");</script>');
INSERT INTO fortune (id, message) VALUES (12, 'フレームワークのベンチマーク');
```

Mysql
```
# modified from SO answer http://stackoverflow.com/questions/5125096/for-loop-in-mysql
DROP DATABASE IF EXISTS hello_world;
CREATE DATABASE hello_world;
USE hello_world;

DROP TABLE IF EXISTS world;
CREATE TABLE  world (
  id int(10) unsigned NOT NULL auto_increment,
  randomNumber int NOT NULL default 0,
  PRIMARY KEY  (id)
)
ENGINE=INNODB;

DROP PROCEDURE IF EXISTS load_data;

DELIMITER #
CREATE PROCEDURE load_data()
BEGIN

declare v_max int unsigned default 10;
declare v_counter int unsigned default 0;

  TRUNCATE TABLE world;
  START TRANSACTION;
  while v_counter < v_max do
    INSERT INTO world (randomNumber) VALUES ( floor(0 + (rand() * 10)) );
    SET v_counter=v_counter+1;
  end while;
  commit;
END #

DELIMITER ;

CALL load_data();

DROP TABLE IF EXISTS fortune;
CREATE TABLE  fortune (
  id int(10) unsigned NOT NULL auto_increment,
  message varchar(2048) CHARACTER SET 'utf8' NOT NULL,
  PRIMARY KEY  (id)
)
ENGINE=INNODB;

INSERT INTO fortune (message) VALUES ('fortune: No such file or directory');
INSERT INTO fortune (message) VALUES ('A computer scientist is someone who fixes things that aren''t broken.');
INSERT INTO fortune (message) VALUES ('After enough decimal places, nobody gives a damn.');
INSERT INTO fortune (message) VALUES ('A bad random number generator: 1, 1, 1, 1, 1, 4.33e+67, 1, 1, 1');
INSERT INTO fortune (message) VALUES ('A computer program does what you tell it to do, not what you want it to do.');
INSERT INTO fortune (message) VALUES ('Emacs is a nice operating system, but I prefer UNIX. — Tom Christaensen');
INSERT INTO fortune (message) VALUES ('Any program that runs right is obsolete.');
INSERT INTO fortune (message) VALUES ('A list is only as strong as its weakest link. — Donald Knuth');
INSERT INTO fortune (message) VALUES ('Feature: A bug with seniority.');
INSERT INTO fortune (message) VALUES ('Computers make very fast, very accurate mistakes.');
INSERT INTO fortune (message) VALUES ('<script>alert("This should not be displayed in a browser alert box.");</script>');
INSERT INTO fortune (message) VALUES ('フレームワークのベンチマーク');

```


Postgres

```

DROP TABLE IF EXISTS world;
CREATE TABLE  world (
  id integer NOT NULL,
  randomNumber integer NOT NULL default 0,
  PRIMARY KEY  (id)
);

INSERT INTO world (id, randomnumber)
SELECT x.id, random() * 10 + 1 FROM generate_series(1,10) as x(id);

DROP TABLE IF EXISTS fortune;
CREATE TABLE fortune (
  id integer NOT NULL,
  message varchar(2048) NOT NULL,
  PRIMARY KEY  (id)
);

INSERT INTO fortune (id, message) VALUES (1, 'fortune: No such file or directory');
INSERT INTO fortune (id, message) VALUES (2, 'A computer scientist is someone who fixes things that aren''t broken.');
INSERT INTO fortune (id, message) VALUES (3, 'After enough decimal places, nobody gives a damn.');
INSERT INTO fortune (id, message) VALUES (4, 'A bad random number generator: 1, 1, 1, 1, 1, 4.33e+67, 1, 1, 1');
INSERT INTO fortune (id, message) VALUES (5, 'A computer program does what you tell it to do, not what you want it to do.');
INSERT INTO fortune (id, message) VALUES (6, 'Emacs is a nice operating system, but I prefer UNIX. — Tom Christaensen');
INSERT INTO fortune (id, message) VALUES (7, 'Any program that runs right is obsolete.');
INSERT INTO fortune (id, message) VALUES (8, 'A list is only as strong as its weakest link. — Donald Knuth');
INSERT INTO fortune (id, message) VALUES (9, 'Feature: A bug with seniority.');
INSERT INTO fortune (id, message) VALUES (10, 'Computers make very fast, very accurate mistakes.');
INSERT INTO fortune (id, message) VALUES (11, '<script>alert("This should not be displayed in a browser alert box.");</script>');
INSERT INTO fortune (id, message) VALUES (12, 'フレームワークのベンチマーク');

```

Above scripts can be found in https://github.com/networknt/light-example-4j/tree/master/rest/swagger/database/dbscript
You can just copy the dbscript folder from database.bak to database folder as scripting for these three databases
are out of scope in this tutorial.

In the next step, we are going to start three databases with Docker and make them ready to be
connected from the project we are building. 


