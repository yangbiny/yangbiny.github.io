---
title:  "Mysql Index"
date:   2013-12-15
desc: "Mysql Index 相关"
keywords: "Mysql"
categories: [Mysql.Database]
tags: [mysql]
icon: icon-html
---

# 锁

Mysql提供了行锁、表锁和页面锁来解决数据之间的竞争的情况。在这三种锁的基础上，MySQL提供了不同的实现，具体使用的锁是需要根据存储引擎决定。

锁的申请规则：

1. 申请读锁时，如果无锁可申请，如果只有读锁，可申请，如果有写锁，不可申请
2. 申请写锁时，如果无锁可申请，如果有读锁，不可申请，如果有写锁，不可申请

# 行锁

行锁：顾名思义是为一个数据行加锁，即当MySQL进行数据加锁的时候，加锁的粒度是针对一条记录，对于其他记录的访问是无影响的。

在InnoDB中，行锁的实现主要有三类：读锁（共享锁）、意向读锁、写锁（排他锁）、意向写锁和间隙锁。

申请锁的时候会先判断该记录是否存在有关联的锁结构（是否有其他事务申请了锁）。如果没有其他事务持有锁（会根据锁的类型去判断是否可以申请锁），则会创建一个锁记录，记录事务ID，标记获取锁的状态为成功和锁的类型，然后成功获取锁。如果有其他的事务申请了锁，则创建一个锁记录，记录事务ID，标记获取锁的状态为失败以及锁的类型。当其他事务释放锁的时候，会去查找对应记录的锁记录信息，然后修改状态为获取成功，并唤醒对应的线程。

### 读锁

Mysql默认查询数据的时候，是不会加锁的，如果需要在Mysql读取数据的时候加上读锁，则可以使用`select * from test for update;`的方式加上排他锁，也可以使用`select * from test lock in share mode` 的方式加上共享读锁。

当给某一条记录加上读锁的时候，会先获取整个表意向读锁。当需要对MySQL的表添加排他锁的时候，就可以通过判断是否存在意向读锁，去判断是否存在有读锁的存在。（写锁的后面会介绍）

### 写锁

写锁是MysQL在更新数据的时候，会先申请一个写锁（排他锁）。申请写锁的时候，也会和读锁一致，先去申请一个意向写锁。

从上述两个加锁的粒度来说，意向读锁和意向写锁并不冲突，因为他们只是一个标志，代表表的是否存在有读锁或者写锁，具体是否加锁成功，是根据具体记录的锁记录信息判断，是否存在锁竞争。



### 间隙锁

在隔离级别中，读未提交、读已提交均会出现幻读的情况。幻读主要是出现在范围查询的时候，如果其他的事务提交了一批数据，事务在第二次查询的时候，会出现两次查询的结果不一致。在可重复读中实际已经解决了幻读的情况。解决幻读的方式可以有两种：MVCC机制和间隙锁。

间隙锁：锁定的数据是一个范围。例如 `select * from test where id < 17 for update` 会锁定id小于17的，如果想要插入一条记录ID 为16 的数据，则会阻塞，直到释放该间隙锁。

使用间隙锁解决幻读的时候，如果其他的事务想要插入数据到该范围中，则会阻塞，直到该间隙锁释放。因为其他的事务无法插入数据，所以不会出现幻读的情况。间隙锁的时候需要在可重复读隔离级别下才可以使用。

# 表锁

下面的均是针对MyIsam进行分析。

表锁是比行锁粒度更大的一个锁，锁定的对象为一张表。当表进行修改的时候，如果有其他的事务想要修改就会被阻塞（concurrent_insert需要设置为0，否则允许并发）。

如果希望能够并行插入数据的时候，可以设置 `concurrent_insert` 的值进行指定。如果指定的值为1，则如果表不存在空洞（没有被删除的数据），则允许在表的最后插入数据；如果指定的值为2，则无论表是否存在空洞，都会在表的最后插入数据。

# 页面锁

页面锁是介于行锁和表锁之间。锁定的对象是一个页面的数据，加锁和释放锁的开销介于行锁和表锁之间。

# 加锁方式

Mysql中默认在执行insert、delete和update的时候，会执行加锁的操作（哪怕只是一个简单的update语句，也会执行加锁操作），并且锁的释放是在事务提交的时候会释放这个锁（在执行更新的时候，如果没有使用显式的事务，Mysql会创建一个隐式事务）。

<aside>
💡 为什么一定要在事务提交的时候，才能释放锁呢？
可以回忆一下Mysql事务的ACID中的特效：isolation。他需要保证两个事务之间的操作互不干扰。如果在提交前就释放了锁，其他的事务就可以读取到他的数据。如果说采用读未提交，会导致采用读已提交的事务也可以读取到未提交的数据，导致其他事务的可见性出现问题，导致脏读。

</aside>

对于`select`  查询语句，如果需要对查询加锁，可以选择两种方式：`select ... from table for update`和`select ... from table for share modle` 两种方式。第一种方式是会直接加上排他锁，其他事务无法读取该数据，第二种方式是添加共享锁，其他事务可以读取数据，但是不能修改。

<aside>
💡 下面这段摘抄自Mysql官方文档。
If you use FOR UPDATE with a storage engine that uses page or row locks, rows examined by the query are write-locked until the end of the current transaction. Using LOCK IN SHARE MODE sets a shared lock that permits other transactions to read the examined rows but not to update or delete them.

In addition, you cannot use FOR UPDATE as part of the SELECT in a statement such as CREATE TABLE new_table SELECT ... FROM old_table .... (If you attempt to do so, the statement is rejected with the error Can't update table 'old_table' while 'new_table' is being created.) This is a change in behavior from MySQL 5.5 and earlier, which permitted CREATE TABLE ... SELECT statements to make changes in tables other than the table being created.

</aside>

# 加锁过程

1. 尝试获取所需要的锁
  1. 如果获取成功，立即返回。
2. 开始自旋回环，增加自选计数（spin-count）
  1. 如果不自旋，那么操作系统就会回收CPU，需要等待操作系统的重新调度。因为锁的使用时间都很短，所以自旋获得锁的概率很高。
3. 开始N轮自选（由参数innodb_sync_spin_loops决定）
  1. 每一轮都会调用一个PAUSE逻辑，导致CPU进入PAUSE的X个周期
  2. 每轮检查锁是否可用，可用则直接退出。
  3. 再次尝试获取所需的锁
    1. 如果成功，返回 spins=1，rounds=M （M≤N），os-waits=0
    2. 如果失败，则判断终止任务还是继续自旋
  4. 当线程完成了spin-wait轮询次数，还是没有获取到锁，则会被挂起，交还给操作系统
  5. 将线程加入到锁对应的等待队列
  6. 收到锁通知，唤起线程，重新尝试获取锁。
    1. 如果成功，返回spins=1，rounds=N，os-waits=1
    2. 如果失败，重新循环，rounds=1
