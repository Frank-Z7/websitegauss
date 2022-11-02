---
title: '高并发报错too many clients already或无法创建线程'

date: '2022-07-14'

category: 'blog'
tags: ['高并发报错too many clients already或无法创建线程']

archives: '2022-07'

author: '云和恩墨'

summary: '高并发报错too many clients already或无法创建线程'

img: '/zh/blogs/enmo/title/img.png'

times: '10:20'
---

# 高并发报错 too many clients already 或无法创建线程

本文出处：[https://www.modb.pro/db/432236](https://www.modb.pro/db/432236)

#### 问题现象

高并发执行 SQL，报错“sorry, too many clients already”；或报无法创建线程、无法 fork 进程等错误。

#### 原因分析

该类报错是由于操作系统线程资源不足引起，查看操作系统 ulimit -u，如果过小（例如小于 32768），则基本可以判断是操作系统限制引起的。

#### 处理办法

通过“ulimit -u”命令查看操作系统 max user processes 的值。

[root@MogDB36 mnt]# ulimit -u
unlimited

按如下简易公式计算需要设置的最小值。

value=max(32768，实例数目\*8192)

其中实例数目指本节点所有实例总数。

设置最小值方法为，修改/etc/security/limits.conf，追加如下两行：

- hard nproc [value]

- soft nproc [value]

  对于不同操作系统修改方式略有不同，centos6 以上版本可以修改/etc/security/ limits.d/90-nofile.conf 文件，方法同上。

另外，也可以直接通过如下命令设置，但 OS 重启会失效，可以添加到全局环境变量/etc/profile 文件中使其生效。

ulimit -u [values]

在大并发模式下，建议开启线程池，使数据库内部的线程资源受控。
