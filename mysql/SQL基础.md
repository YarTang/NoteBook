#  SQL基础

SQL语句主要分为以下三种：

* **DDL**(Data Definition Language)，数据定义语句

* **DML**(Data Manipulation Language), 数据操纵语句

* **DCL**(Data Control Language), 数据控制语句

## DLL语句

1. 创建数据库

   使用命令 CREATE DATABASE *dbmame*

   ```sql
   create database test
   ```

   如果需要查看系统中所有的数据库，使用以下命令

   ```sql
   show databases
   ```

   在查看系统中的数据库后，可以使用命令  USE *dbname* 使用某个数据库

   ```sql
   use test
   ```

   使用以下命令查看数据库中的所有表

   ```sql
   show tables
   ```

2. 删除数据库

   DROP DATABASE *dbname*

   ```sq
   drop database test
   ```

3. 创建表

   CREATE TABLE *tablename* {

   *columnname_1 columntype_1 contraints*,

   *columnname_2 columntype_2 contraints*,

   ...

   *columnname_n columntype_n contraints*

   }

   ```sql
   create table emp (
   	ename varchar(4),
      	hiredate date,
       sal decimal(10,2),
       deptno int(2)
   );
   ```

4. 删除表

   DROP TABLE *tablename*

   ```sql
   drop table emp;
   ```

5. 修改表

   ALTER TABLE *tbl_name*
       [*alter_specification* [, *alter_specification* ]  *...* ]
       [*partition_options*]

   - 修改表的类型

     ALTER TABLE *tbl_name* MODIFY [columnname] columndefinition

     ```sql
     alter table emp modify ename varchar(20)
     ```

   - 增加字段

     ALTER TABLE *tbl_name* add columnname columndefinition

     ```sql
     alter table emp add age int(3)
     ```

   - 删除字段

     ALTER TABLE *tbl_name* DROP *col_name*

     ```sql
     alter table emp drop age
     ```

   - 字段改名

     ALTER TABLE  *tbl_name* CHANGE *old_tbl_name* *new_tbl_name* *new_tbl_definition*

     ```sql
     alter table emp change ename ename1 varchar(10)
     ```

   - 更改表名

     ALTER TABLE *tbl_name* RENAME *new_tbl_name*

     ```sql
     alter table emp rename emptest
     ```

## DML语句

1.插入记录

INSERT INTO *tbl_name* (*field1,filed2,field3...*) 

VALUES(*value1, value2, value3...*)

VALUES(*value1, value2, value3...*)

```sql
insert into emp (ename, hiredate, sal, deptno)
values('lisa', '2003-01-01', '2000', 2),
values('lisaa', '2004-01-01', '3000', 3),
```

2.更新记录

UPDATE *t1, t2...tn* set *t1.filed = expr1 ... tn.fieldn = exprn* [WHERE CONDITION]

```sql
update emp a, dept b set a.sal = a.sal * b.deptno, b.deptname = a.ename
where a.deptno = b.deptno;
```

3.删除记录

DELETE [*t1,t2,t3...*] FROM [*t1, t2, t3...*] [WHERE CONDITION]

```sql
delete from emp where emp.deptno = 1
delete t1, t2 from emp t1, dept t2 where a.deptno = b.deptno and a.deptno = 3;
```

4.查询记录

* 排序和限制

  SELECT * FROM *tbl_name* *[WHERE CONDITION] [ORDER BY field1 [DESC/ASC], field2 [DESC/ASC]...]*

  字段排序的优先级按照field出现次序，DESC表示按照字段进行降序排列，ASC表示按照升序排列

  ```sql
  select * from emp order by deptno;
  ```

* 聚合

  SELECT [*field1,field2...*] *fun_name* 

  FROM *tbl_name*

  *[WHERE where_condition]*

  *[GROUP BY field1,field2... [WITH ROLLUP]]*

  *[HAVING where_condition]*

  ```sql
  # 统计总人数
  select count(1) from emp;
  # 统计部门人数
  select deptno,count(1) from emp group by deptno
  # 统计部门人数以及总人数
  select deptno, count(1) from emp group by deptno rollup;
  # 统计人数大于1的部门
  select deptno, count(1) from emp group by deptno having count(1) > 1;
  ```

* 表连接

  ```sql
  # 内连接
  select * from emp a inner join dept b on a.deptno = b.deptno
  # 左连接
  select ename, deptname from emp a left join dept b on a.deptno = b.deptno
  # 右连接
  select ename, deptname from emp a right join dept b on a.deptno = b.deptno
  ```

* 记录联合

  SELECT * FROM t1

  UNION|UNIONALL

  SELECT * FROM t2

  UNION|UNIONALL

  SELECT * FROM t3

  **UNION ALL** 把结果直接合并在一起

  **UNION** 将合并结果中重复的去掉

## DCL语句