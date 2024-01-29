# mysql 事务


<!--more-->

## 前言

mysql的事务不管是面试还是工作中，都有理由去掌握并应用。因为这关系到数据一致性问题，如果数据一致性(包括缓存)处理不好，就会带来毁灭性的灾难。

## 相关语法

- 查看全局，会话事务的隔离级别

  ```sql
  #8.0后
  #全局
  select @@global.transaction_isolation;
  #会话
  select @@session.transaction_isolation;
  #全局
  SELECT @@transaction_isolation;
  ```

  

  ```sql
  #全局
  SELECT @@global.tx_isolation;
  #会话
  SELECT @@session.tx_isolation;
  #全局
  SELECT @@tx_isolation;
  ```

- 设置

  ````sql
  SET [SESSION | GLOBAL] TRANSACTION ISOLATION LEVEL {READ UNCOMMITTED | READ COMMITTED | REPEATABLE READ | SERIALIZABLE}
  ````

- 概念

  - `READ UNCOMMITTED` 读未提交，一个事务可以读到另一个事务的未提交(commit) 的数据，该隔离级别会导致脏读现象

  - `READ COMMITTED` 读已提交，一个事物能读到另一个事务的已提交的(commit) 的数据，但是如果这个事务重复的读有可能会导致数据不一致的情况，后面详细介绍，该隔离级别会导致两次读数据不一致的现象。
  - `REPEATABLE READ` 可重复的读，一个事务能读到另一个事务的已提交的(commit) 的数据，并且重复读数据得到的是相同的结果，该隔离级别会导致幻读
  - `SERIALIZABLE` 串行，并发量最低，数据正确度最高，最高隔离级别

- 总结
  ![](/img/mysql/6.png)

## 演示

- READ UNCOMMITTED 读未提交

  ```sql
  SET SESSION TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;
  ```

  终端1会话隔离级别为
  ![](/img/mysql/7.png "工作流程")
  设置终端1当前会话隔离级别为`READ UNCOMMITTED`
  ![](/img/mysql/8.png "工作流程")
  再开一个终端叫终端2，查看数据为level 为0
  ![](/img/mysql/9.png "工作流程")
  在终端1上，起事务更新数据 level 为1
  ![](/img/mysql/10.png "工作流程")
  在终端2上设置会话隔离级别为`READ UNCOMMITTED`，并查看数据，发现level已经变成了1
  ![](/img/mysql/11.png "工作流程")

- READ COMMITTED 读已提交
  设置终端1，终端2的隔离级别为 READ COMMITTED

  ````sql
  SET SESSION TRANSACTION ISOLATION LEVEL READ COMMITTED;
  ````

  终端1起事务，更新数据
  ![](/img/mysql/12.png "工作流程")
  终端2上查看数据，发现level并没有发生改变
  ![](/img/mysql/13.png "工作流程")
  终端1提交数据
  ![](/img/mysql/14.png "工作流程")
  终端2上查看数据，发现level变成了9
  ![](/img/mysql/15.png "工作流程")

- REPEATABLE READ 可重复读
  设置终端1，终端2的隔离级别为 `REPEATABLE READ`

  ```sql
  SET SESSION TRANSACTION ISOLATION LEVEL REPEATABLE READ;
  ```

  终端1起事务，更新数据
  ![](/img/mysql/16.png "工作流程")

  终端2起事务，查看数据
  ![](/img/mysql/17.png "工作流程")
  终端1提交数据
  ![](/img/mysql/18.png "工作流程")
  终端2查看数据，发现level还是9
  ![](/img/mysql/19.png "工作流程")
  终端2提交，再次查看发现level变成了199了
  ![](/img/mysql/20.png "工作流程")

  如果终端1，插入数据会怎么样，其他查询事务会看见吗
  终端2起事务，查看数据
  ![](/img/mysql/21.png "工作流程")
  终端1插入数据

- SERIALIZABLE 串行
  终端1，终端2，设置隔离级别为 `SERIALIZABLE`

   ```sql
   SET SESSION TRANSACTION ISOLATION LEVEL SERIALIZABLE;
   ```

  终端1，起事务插入数据
  ![](/img/mysql/22.png "工作流程")
  终端2，起事务，查看数据发现，卡住了直到超时
  ![](/img/mysql/23.png "工作流程")

  

