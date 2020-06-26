#  SQL基础

SQL语句主要分为以下三种：

* **DDL**(Data Definition Language)，数据定义语句

* **DML**(Data Manipulation Language), 数据操纵语句

* **DCL**(Data Control Language), 数据控制语句

## DLL语句

1. 创建数据库

   使用命令 CREATE DATABASE ***dbmame***，dbname 是创建的数据库名

   ```sql
   create database test
   ```

   如果需要查看系统中所有的数据库，使用以下命令

   ```sql
   show databases
   ```

   在查看系统中的数据库后，可以使用命令  USE ***dbname*** 使用某个数据库

   ```sql
   use test
   ```

   使用以下命令查看数据库中的所有表

   ```sql
   show tables
   ```

2. 删除数据库

   DROP DATABASE ***dbname***

   ```sq
   drop database test
   ```

3. 创建表

4. 删除表

5. 修改表

## DML语句

## DCL语句