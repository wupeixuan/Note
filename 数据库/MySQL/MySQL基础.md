数据操作语言 DML语言：
```
SELECT - 从数据库表中获取数据
UPDATE - 更新数据库表中的数据
DELETE - 从数据库表中删除数据
INSERT INTO - 向数据库表中插入数据
```
数据定义语言 DDL语言:
```
CREATE DATABASE - 创建新数据库
ALTER DATABASE - 修改数据库
CREATE TABLE - 创建新表
ALTER TABLE - 变更（改变）数据库表
DROP TABLE - 删除表
CREATE INDEX - 创建索引（搜索键）
DROP INDEX - 删除索引
```

```
--创建表
CREATE TABLE `student`(
   `id` INT UNSIGNED AUTO_INCREMENT,
   `name` VARCHAR(100) NOT NULL,
   PRIMARY KEY ( `id` )
)ENGINE=InnoDB DEFAULT CHARSET=utf8;

CREATE TABLE `studentbak`(
   `id` INT UNSIGNED AUTO_INCREMENT,
   `name` VARCHAR(100) NOT NULL,
   PRIMARY KEY ( `id` )
)ENGINE=InnoDB DEFAULT CHARSET=utf8;

CREATE TABLE `grade` (
  `name` varchar(100) NOT NULL,
  `score` int(11) NOT NULL DEFAULT '0'
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

--删除表
drop table runoob_tbl;


--插入数据
INSERT INTO student (name) VALUES ('武培轩');

--查询数据
SELECT * FROM student;
SELECT * FROM student WHERE id=1;
SELECT * FROM student WHERE name like '%轩';
SELECT * FROM student order by id DESC;

--inner join
select a.id,a.name,b.score from student a inner join grade b on a.name=b.name;

--left join
select a.id,a.name,b.score from student a left join grade b on a.name=b.name;

--right join
select a.id,a.name,b.score from student a right join grade b on a.name=b.name;

--is null/is not null
SELECT * FROM grade WHERE score IS NULL;

SELECT * FROM grade WHERE score IS NOT NULL;

--更新数据
UPDATE student SET name='王鹏鹏' WHERE id=2;

--删除数据
DELETE FROM student WHERE id=3;

--创建索引
ALTER TABLE student ADD INDEX (id);

--删除索引
ALTER TABLE student DROP INDEX id;

--复制表
INSERT INTO studentbak SELECT * FROM student;

```