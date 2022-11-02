---
title: 'openGauss数据库的事务锁'

date: '2022-04-07'

category: 'blog'
tags: ['openGauss数据库的事务锁']

archives: '2022-04'

author: '云和恩墨交付战队'

summary: 'openGauss数据库的事务锁'

img: '/zh/blogs/enmo/title/img6.png'

times: '10:20'
---

# openGauss 数据库的事务锁

openGauss 数据库事务锁分为两类：表级锁和行级锁

## 表级锁

表级锁有 8 种模式：

**ACCESS SHARE**

只与 ACCESS EXCLUSIVE 冲突。

SELECT 命令在被引用的表上请求一个这种锁。通常，任何只读取表而不修改它的命令都请求这种锁模式。

**ROW SHARE**

与 EXCLUSIVE 和 ACCESS EXCLUSIVE 锁模式冲突。

SELECT FOR UPDATE 和 SELECT FOR SHARE 命令会自动在目标表上请求 ROW SHARE 锁（且所有被引用但不是 FOR SHARE/FOR UPDATE 的其他表上，还会自动加上 ACCESS SHARE 锁）。

**ROW EXCLUSIVE**

与 ROW SHARE 锁相同，ROW EXCLUSIVE 允许并发读取表，但是禁止修改表中数据。UPDATE，DELETE，INSERT 命令会自动在目标表上请求这个锁（且所有被引用的其他表上还会自动加上的 ACCESS SHARE 锁）。通常情况下，所有会修改表数据的命令都会请求表的 ROW EXCLUSIVE 锁。

**SHARE UPDATE EXCLUSIVE**

这个模式保护一个表的模式不被并发修改，以及禁止在目标表上执行垃圾回收命令（VACUUM ）。

VACUUM（不带 FULL 选项），ANALYZE，CREATE INDEX CONCURRENTLY 命令会自动请求这样的锁。

**SHARE**

SHARE 锁允许并发的查询，但是禁止对表进行修改。

CREATE INDEX（不带 CONCURRENTLY 选项）语句会自动请求这种锁。

**SHARE ROW EXCLUSIVE**

SHARE ROW EXCLUSIVE 锁禁止对表进行任何的并发修改，而且是独占锁，因此一个会话中只能获取一次。

任何 SQL 语句都不会自动请求这个锁模式。

**EXCLUSIVE**

EXCLUSIVE 锁允许对目标表进行并发查询，但是禁止任何其他操作。

这个模式只允许并发加 ACCESS SHARE 锁，也就是说，只有对表的读动作可以和持有这个锁模式的事务并发执行。

任何 SQL 语句都不会在用户表上自动请求这个锁模式。然而在某些操作的时候，会在某些系统表上请求它。

**ACCESS EXCLUSIVE**

这个模式保证其所有者（事务）是可以访问该表的唯一事务。

ALTER TABLE，DROP TABLE，TRUNCATE，REINDEX 命令会自动请求这种锁。

在 LOCK TABLE 命令没有明确声明需要的锁模式时，它是缺省锁模式。

**手动获取行级锁**

可以在事务块内部执行 lock table 命令获取指定的表级锁。

LOCK [ TABLE ] {[ ONLY ] name [, …]| {name [ * ]} [, …]}
[ IN {ACCESS SHARE | ROW SHARE | ROW EXCLUSIVE | SHARE UPDATE EXCLUSIVE | SHARE | SHARE ROW EXCLUSIVE | EXCLUSIVE | ACCESS EXCLUSIVE} MODE ]
[ NOWAIT ];

```
postgres=# begin;
BEGIN
postgres=# lock table stu ;
LOCK TABLE
postgres=# select pg_backend_pid();
 pg_backend_pid
-----------------
 140645423245056
(1 row)

postgres=# select * from pg_locks where pid ='140645423245056';
   locktype    | database | relation | page | tuple | bucket | virtualxid | transactionid | classid | objid | objsubid | virtualtransaction |       pid       |    sessionid    |        mode
     | granted | fastpath |      locktag
---------------+----------+----------+------+-------+--------+------------+---------------+---------+-------+----------+--------------------+-----------------+-----------------+----------------
-----+---------+----------+-------------------
 virtualxid    |          |          |      |       |        | 3/6288     |               |         |       |          | 3/6288             | 140645423245056 | 140645423245056 | ExclusiveLock
     | t       | t        | 3:1890:0:0:0:7
 relation      |    15103 |    16385 |      |       |        |            |               |         |       |          | 3/6288             | 140645423245056 | 140645423245056 | AccessExclusive
Lock | t       | f        | 3aff:4001:0:0:0:0
 transactionid |          |          |      |       |        |            |         73662 |         |       |          | 3/6288             | 140645423245056 | 140645423245056 | ExclusiveLock
     | t       | f        | 11fbe:0:0:0:0:6
(3 rows)


```

## 行级锁

行级锁有 2 种模式：

**FOR UPDATE**

FOR UPDATE 会导致由 SELECT 语句检索到的行被锁定。这样避免它们在当前事务结束前被其他事务修改或者删除，即其他企图 UPDATE、 DELETE、 SELECT FOR UPDATE 这些行的事务将被阻塞，直到当前事务结束。

**FOR SHARE**

在每个检索出来的行上要求一个共享锁，而不是一个排他锁。一个共享锁阻塞其它事务执行 UPDATE、DELETE、SELECT，不阻塞 SELECT FOR SHARE。

**手动获取表级锁**

可以使用 select…for update|share 命令获取行级锁。

```
postgres=# begin;
BEGIN
postgres=# select *from stu for update;
 id | name
----+------
  1 | a
  2 | a
  3 | a
  4 | a
(4 rows)

postgres=# select pg_backend_pid();
 pg_backend_pid
-----------------
 140645423245056
(1 row)

postgres=#
postgres=# select * from pg_locks where pid ='140645423245056';
   locktype    | database | relation | page | tuple | bucket | virtualxid | transactionid | classid | objid | objsubid | virtualtransaction |       pid       |    sessionid    |     mode      |
 granted | fastpath |      locktag
---------------+----------+----------+------+-------+--------+------------+---------------+---------+-------+----------+--------------------+-----------------+-----------------+---------------+
---------+----------+-------------------
 relation      |    15103 |    16385 |      |       |        |            |               |         |       |          | 3/6292             | 140645423245056 | 140645423245056 | RowShareLock  |
 t       | t        | 3aff:4001:0:0:0:0
 virtualxid    |          |          |      |       |        | 3/6292     |               |         |       |          | 3/6292             | 140645423245056 | 140645423245056 | ExclusiveLock |
 t       | t        | 3:1894:0:0:0:7
 transactionid |          |          |      |       |        |            |         73665 |         |       |          | 3/6292             | 140645423245056 | 140645423245056 | ExclusiveLock |
 t       | f        | 11fc1:0:0:0:0:6
(3 rows)

```

**查询数据库中锁阻塞信息**

查询视图 pg_locks 可以获取数据库中事务锁阻塞信息

select \* from pg_locks;

```
postgres=# select * from pg_locks;
  locktype    | database | relation | page | tuple | bucket | virtualxid | transactionid | classid | objid | objsubid | virtualtransaction |       pid       |    sessionid    |        mode
    | granted | fastpath |      locktag
---------------+----------+----------+------+-------+--------+------------+---------------+---------+-------+----------+--------------------+-----------------+-----------------+----------------
-----+---------+----------+-------------------
virtualxid    |          |          |      |       |        | 9/48       |               |         |       |          | 9/48               | 140645423245056 | 140645423245056 | ExclusiveLock
    | t       | t        | 9:30:0:0:0:7
virtualxid    |          |          |      |       |        | 3/6309     |               |         |       |          | 3/6309             | 140645440026368 | 140645440026368 | ExclusiveLock
    | t       | t        | 3:18a5:0:0:0:7
relation      |    15103 |    11835 |      |       |        |            |               |         |       |          | 8/19332            | 140645607831296 | 140645607831296 | AccessShareLock
    | t       | t        | 3aff:2e3b:0:0:0:0
virtualxid    |          |          |      |       |        | 8/19332    |               |         |       |          | 8/19332            | 140645607831296 | 140645607831296 | ExclusiveLock
    | t       | t        | 8:4b84:0:0:0:7
relation      |    15103 |    16385 |      |       |        |            |               |         |       |          | 9/48               | 140645423245056 | 140645423245056 | AccessExclusive
Lock | t       | f        | 3aff:4001:0:0:0:0
relation      |    15103 |    16385 |      |       |        |            |               |         |       |          | 3/6309             | 140645440026368 | 140645440026368 | AccessExclusive
Lock | f       | f        | 3aff:4001:0:0:0:0
transactionid |          |          |      |       |        |            |         73670 |         |       |          | 9/48               | 140645423245056 | 140645423245056 | ExclusiveLock
    | t       | f        | 11fc6:0:0:0:0:6
transactionid |          |          |      |       |        |            |         73671 |         |       |          | 3/6309             | 140645440026368 | 140645440026368 | ExclusiveLock
    | t       | f        | 11fc7:0:0:0:0:6
(8 rows)

postgres=# select locktag_decode('11fc7:0:0:0:0:6');
              locktag_decode
---------------------------------------------
locktype:transactionid, transactionid:73671
(1 row)

```

其中各列的描述如下：

locktype:被锁定对象的类型,共 12 种。

| Lock 等待事件    | 描述                    |
| ---------------- | ----------------------- |
| relation         | 对表加锁。              |
| extend           | 对表扩展空间时加锁。    |
| partition        | 对分区表加锁。          |
| partition_seq    | 对分区表的分区加锁。    |
| page             | 对表页面加锁。          |
| tuple            | 对页面上的 tuple 加锁。 |
| transactionid    | 对事务 ID 加锁。        |
| virtualxid       | 对虚拟事务 ID 加锁。    |
| object           | 加对象锁。              |
| cstore_freespace | 对列存空闲空间加锁。    |
| userlock         | 加用户锁。              |
| advisory         | 加 advisory 锁。        |

database:被锁定对象所在数据库的 OID。

relation:被锁定对象的 OID。

page:被锁定对象的页面编号。

tuple:被锁定对象的页面里边的行编号。

bucket:子表对应的 bucket number。

virtualxid:事务的虚拟 ID。

transactionid:事务的 ID。

classid:包含该被锁定对象的系统表的 OID。

objid:被锁定对象在其系统表内的 OID。

objsubid:对于表的一个字段，这是字段编号；对于其他对象类型，这个字段是零。

virtualtransaction:持有此锁或者在等待此锁的事务的虚拟 ID。

pid:持有或者等待这个锁的服务器线程的逻辑 ID。

sessionid:持有或者等待这个锁的会话 ID。

mode:这个线程持有的或者是期望的锁模式。

granted: 如果锁是持有锁，则为 TRUE。如果锁是等待锁，则为 FALSE。

fastpath:如果通过 fast-path 获得锁，则为 TRUE；如果通过主要的锁表获得，则为 FALSE。

locktag:会话等待锁信息，可通过 locktag_decode()函数解析。

**阻塞会话查询**

```
with lock as (select usename,granted,locktag,query_start,query,l.pid
from pg_locks l,pg_stat_activity a
where l.pid=a.pid and locktag in(select locktag from pg_locks where granted=‘f’))
select locker.usename locker_user,locker.query_start locker_query_start,locker.granted locker_granted,locker.query locker_query,locker.pid locker_pid,locked.pid locked_pid,locked.query locked_query,locked.query_start locked_query_start,locked.granted locked_granted,locked.usename locked_user,extract(epoch from now() - locked.query_start) as locked_times
from (select * from lock where granted=‘t’) as locker,(select * from lock where granted=‘f’) locked
where locker.locktag=locked.locktag
order by 1;
```

```
postgres=# with lock as (select usename,granted,locktag,query_start,query,l.pid
postgres(# from pg_locks l,pg_stat_activity a
postgres(# where l.pid=a.pid and locktag in(select locktag from pg_locks where granted='f'))
postgres-# select locker.usename locker_user,locker.query_start locker_query_start,locker.granted locker_granted,locker.query locker_query,locker.pid locker_pid,locked.pid locked_pid,locked.query locked_query,locked.query_start locked_query_start,locked.granted locked_granted,locked.usename locked_user,extract(epoch from now() - locked.query_start) as locked_times
postgres-# from (select * from lock where granted='t') as locker,(select * from lock where granted='f') locked
postgres-# where locker.locktag=locked.locktag
postgres-# order by 1;
 locker_user |      locker_query_start      | locker_granted |               locker_query                |   locker_pid    |   locked_pid    |               locked_query                |      l
ocked_query_start       | locked_granted | locked_user | locked_times
-------------+------------------------------+----------------+-------------------------------------------+-----------------+-----------------+-------------------------------------------+-------
------------------------+----------------+-------------+--------------
 omm         | 2021-12-13 16:13:04.75289+08 | t              | lock table stu  in ACCESS EXCLUSIVE mode; | 140645423245056 | 140645440026368 | lock table stu  in ACCESS EXCLUSIVE mode; | 2021-1
2-13 16:13:09.585616+08 | f              | omm         |   143.055227
(1 row)

```

**手动 kill 掉锁阻塞会话**

```
postgres=# select pg_terminate_backend(140645423245056);
 pg_terminate_backend
----------------------
 t
(1 row)
```

**锁等待超时**

当申请锁的操作等待时间超过数据库 lockwait_timeout 参数的设定值时，系统会报错。lockwait_timeout 参数默认 20 分钟。

当更新操作等待时间操作数据库 update_lockwait_timeout 参数的设定值时，系统会报错。update_lockwait_timeout 参数默认 2 分钟。

**死锁检测**

openGauss 数据库能够自动侦测到死锁，然后退出其中一个事务，从而允许其他事物执行。

当申请锁的等待时间超过数据库参数 deadlock_timeout 的设定值时，系统会检查是否产生了死锁。deadlock_timeout 参数默认值为 1s。

**测试**

测试前修改数据库参数 log_lock_waits 参数值为 on，控制当一个会话为获得一个锁等到超过 deadlock_timeout 时，产生一个日志消息。默认是 off 不记录。

测试 1：会话 2 执行锁表操作时，发生锁等待，等待时间达到 1S 时，系统进行死锁检测。锁等待时间达到 lockwait_timeout 设置的 20min 时，会话 2 执行的锁表操作报错退出，提示锁等待超时。

```
#会话1：
postgres=# begin;
BEGIN
postgres=# lock table stu  in ACCESS EXCLUSIVE mode;
LOCK TABLE
postgres=# select now();
              now
-------------------------------
 2021-12-13 15:25:23.535138+08
(1 row)
#会话2：
postgres=# begin;
BEGIN
postgres=# lock table stu  in ACCESS EXCLUSIVE mode;
ERROR:  Lock wait timeout: thread 140645440026368 on node dn_6001 waiting for AccessExclusiveLock on relation 16385 of database 15103 after 1199000.033 ms
DETAIL:  blocked by hold lock thread 140645423245056, statement <select now();>, hold lockmode AccessExclusiveLock.
postgres=#
#日志：
2021-12-13 15:26:41.395 61b6f5ab.5064 postgres 140645440026368 gsql 73671 dn_6001 00000  2533274790414298 [BACKEND] LOG:  thread 140645440026368 still waiting for AccessExclusiveLock on relation 16385 of database 15103 after 1000.051 ms
2021-12-13 15:46:40.395 61b6f5ab.5064 postgres 140645440026368 gsql 73671 dn_6001 00000  2533274790414298 [BACKEND] LOG:  thread 140645440026368 still waiting for AccessExclusiveLock on relation 16385 of database 15103 after 1199000.033 ms
2021-12-13 15:46:40.395 61b6f5ab.5064 postgres 140645440026368 gsql 73671 dn_6001 YY003  2533274790414298 [BACKEND] ERROR:  Lock wait timeout: thread 140645440026368 on node dn_6001 waiting for AccessExclusiveLock on relation 16385 of database 15103 after 1199000.033 ms
2021-12-13 15:46:40.395 61b6f5ab.5064 postgres 140645440026368 gsql 73671 dn_6001 YY003  2533274790414298 [BACKEND] DETAIL:  blocked by hold lock thread 140645423245056, statement <select now();>, hold lockmode AccessExclusiveLock.
2021-12-13 15:46:40.395 61b6f5ab.5064 postgres 140645440026368 gsql 73671 dn_6001 YY003  2533274790414298 [BACKEND] STATEMENT:  lock table stu  in ACCESS EXCLUSIVE mode;

```

测试 2：更新操作，锁等待时间达到 update_lockwait_timeout 设置的 2min 时，会话 2 执行的 update 操作报错退出，提示锁等待超时。

```
#会话1：
postgres=# begin;
BEGIN
postgres=# select *from stu for update;
 id | name
----+------
  1 | a
  2 | a
  3 | a
  4 | a
(4 rows)

#会话2：
postgres=# begin;
BEGIN
postgres=#  update stu set name = 'b' where id=2;
ERROR:  Lock wait timeout: thread 140645607831296 on node dn_6001 waiting for ShareLock on transaction 73665 after 119000.041 ms
DETAIL:  blocked by hold lock thread 140645423245056, statement <select pg_backend_pid();>, hold lockmode ExclusiveLock.
postgres=#

#日志：
2021-12-13 14:56:24.802 61b6ee19.5067 postgres 140645607831296 gsql 73666 dn_6001 00000  2533274790414263 [BACKEND] LOG:  thread 140645607831296 still waiting for ShareLock on transaction 73665 after 119000.041 ms
2021-12-13 14:56:24.802 61b6ee19.5067 postgres 140645607831296 gsql 73666 dn_6001 YY003  2533274790414263 [BACKEND] ERROR:  Lock wait timeout: thread 140645607831296 on node dn_6001 waiting for ShareLock on transaction 73665 after 119000.041 ms
2021-12-13 14:56:24.802 61b6ee19.5067 postgres 140645607831296 gsql 73666 dn_6001 YY003  2533274790414263 [BACKEND] DETAIL:  blocked by hold lock thread 140645423245056, statement <select pg_backend_pid();>, hold lockmode ExclusiveLock.
2021-12-13 14:56:24.802 61b6ee19.5067 postgres 140645607831296 gsql 73666 dn_6001 YY003  2533274790414263 [BACKEND] STATEMENT:  update stu set name = 'b' where id=2;
```
