## MySQL中的insert ignore into, replace into等的一些用法总结

mysql中常用的三种插入数据的语句：

**一.insert into**

　　表示插入数据，数据库会检查主键（PrimaryKey），如果出现重复会报错；

　　insert … select … where not exist：根据select的条件判断是否插入，可以不光通过primary 和unique来判断，也可通过其它条件。例如：

```sql
INSERT INTO books (name) SELECT 'MySQL Manual' FROM dual WHERE NOT EXISTS (SELECT id FROM books WHERE id = 1)
```

　　on duplicate key update：当primary或者unique重复时，则执行update语句，如update后为无用语句，如id=id，则同1功能相同，但错误不会被忽略掉。例如，为了实现name重复的数据插入不报错，可使用一下语句：

```sql
INSERT INTO books (name) VALUES ('MySQL Manual') ON duplicate KEY UPDATE id = id
```

 

**二.replace into**

　　如果存在primary or unique相同的记录，则先删除掉。再插入新记录。

```sql
REPLACE INTO books SELECT 1, 'MySQL Manual' FROM books
```

　　表示插入替换数据，需求表中有PrimaryKey，或者unique索引的话，如果数据库已经存在数据，则用新数据替换，如果没有数据效果则和insert into一样；

　　REPLACE语句会返回一个数，来指示受影响的行的数目。该数是被删除和被插入的行数的和。如果对于一个单行REPLACE该数为1，则一行被插入，同时没有行被删除。如果该数大于1，则在新行被插入前，有一个或多个旧行被删除。如果表包含多个唯一索引，并且新行复制了在不同的唯一索引中的不同旧行的值，则有可能是一个单一行替换了多个旧行。

　　MySQL replace into 有三种形式：

　　1. replace into tbl_name(col_name, ...) values(...)

　　2. replace into tbl_name(col_name, ...) select ...

　　3. replace into tbl_name set col_name=value, ...

**replace into 当表是多个唯一索引，并且即将插入的这样数据与多行都有冲突，则会删除多行并插入新数据** 

**三.insert ignore into**

　　表示如果中已经存在相同的记录，则忽略当前新数据。

　　当插入数据时，如出现错误时，如重复数据，将不返回错误，只以警告形式返回。所以使用ignore请确保语句本身没有问题，否则也会被忽略掉。例如：

```sql
INSERT IGNORE INTO books (name) VALUES ('MySQL Manual')
```

 

**四.INSERT IGNORE 与INSERT INTO的区别**

　　INSERT IGNORE会忽略数据库中已经存在 的数据，如果数据库没有数据，就插入新的数据，如果有数据的话就跳过这条数据。这样就可以保留数据库中已经存在数据，达到在间隙中插入数据的目的，如：

```sql
insert ignore into table(name) select name from table2
```

　　下面通过代码说明之间的区别，如下：

下面这段sql拿去跑一跑，一切都明了，skr skr～～

```sql
create table testtb(
    id int not null primary key,
    name varchar(50),
    age int
);
insert into testtb(id,name,age)values(1,"bb",13);
select * from testtb;
insert ignore into testtb(id,name,age)values(1,"aa",13);
select * from testtb;//仍是1，“bb”,13，因为id是主键，出现主键重复但使用了ignore则错误被忽略
replace into testtb(id,name,age)values(1,"aa",12);
select * from testtb; //数据变为1,"aa",12
```

 