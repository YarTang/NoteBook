# MySQL学习笔记

## 查看sql语句执行效率的方法

* 慢查询日志定位执行效率较低的SQL语句

* 通过explain语句分析

  > | id | select_type | table | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra |

  以上是查询的几个主要参数。

* show profile分析SQL

  1.通过set语句开启profiling

  > mysql> select @@profiling;
  >+-------------+
  > | @@profiling |
  > +-------------+
  > |           0 |
  > +-------------+
  > 1 row in set, 1 warning (0.00 sec)

  2.执行完SQL语句后通过show profiles语句查看当前query的ID

  > #执行查询SQL语句
  >
  > mysql>select count(*) from payment;
  >
  > 
  >
  > mysql>show profiles;
  >
  > +----------+------------+------------------------------+
  > | Query_ID | Duration   | Query                        |
  > +----------+------------+------------------------------+
  > |        1 | 0.00011550 | show warnings                |
  > |        2 | 0.00023425 | SELECT DATABASE()            |
  > |        3 | 0.10067725 | select count(*) from payment |
  > +----------+------------+------------------------------+

  3.show profile for query [ID]查看执行过程中线程每个状态和消耗时间

  >mysql> show profile for query 3;
  >+--------------------------------+----------+
  >| Status                         | Duration |
  >+--------------------------------+----------+
  >| starting                       | 0.009182 |
  >| Executing hook on transaction  | 0.000009 |
  >| starting                       | 0.000010 |
  >| checking permissions           | 0.000009 |
  >| Opening tables                 | 0.001797 |
  >| init                           | 0.000010 |
  >| System lock                    | 0.000007 |
  >| optimizing                     | 0.000278 |
  >| statistics                     | 0.000020 |
  >| preparing                      | 0.000020 |
  >| executing                      | 0.089198 |
  >| end                            | 0.000010 |
  >| query end                      | 0.000003 |
  >| waiting for handler commit     | 0.000013 |
  >| closing tables                 | 0.000010 |
  >| freeing items                  | 0.000081 |
  >| cleaning up                    | 0.000023 |
  >+--------------------------------+----------+
  >17 rows in set, 1 warning (0.01 sec)

## 索引的使用



