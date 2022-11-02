---
title: '卸载openGauss'

date: '2021-10-31'

category: 'blog'
tags: ['卸载openGauss']

archives: '2021-10'

author: 'easydb '

summary: '卸载openGauss'

img: '/zh/blogs/July/title/img1.png'

times: '12:30'
---

# 卸载 openGauss<a name="ZH-CN_TOPIC_0000001200594233"></a>

卸载 openGauss 的过程分为两步，包含卸载 openGauss 数据库和对 openGauss 服务器的环境做清理。

## 执行卸载 openGauss 数据库<a name="section4142182815270"></a>

openGauss 提供了卸载脚本帮助用户完整的卸载 openGauss 数据库。

## 操作步骤<a name="section118176390278"></a>

1.  确认主节点

    ```
    [omm@db02 ~]$ gs_om -t status --detail
    [  CMServer State   ]

    node        node_ip         instance                                 state
    ----------------------------------------------------------------------------
    AZ1 1  db01 192.168.0.43    1    /opt/huawei/data/cmserver/cm_server Primary
    AZ1 2  db02 192.168.0.22    2    /opt/huawei/data/cmserver/cm_server Standby
    AZ1 3  db03 192.168.0.242   3    /opt/huawei/data/cmserver/cm_server Standby
    ```

2.  以操作系统用户 omm 登录数据库主节点 db01 192.168.0.43。

    ```
    [root@db01 ~]# su - omm
    Last login: Mon Oct 25 14:24:27 CST 2021 on pts/0
    [omm@db01 ~]$ ip a | grep 43
        inet 192.168.0.43/24 brd 192.168.0.255 scope global dynamic noprefixroute eth0
    ```

3.  使用 gs_uninstall 卸载 openGauss。

    ```
    [omm@db01 ~]$ gs_uninstall --delete-data
    Checking uninstallation.
    Successfully checked uninstallation.
    Stopping the cluster.
    Successfully stopped the cluster.
    Successfully deleted instances.
    Uninstalling application.
    Successfully uninstalled application.
    Uninstallation succeeded.
    Successfully uninstall cluster, for more message please see /home/omm/gs_uninstall.log
    ```

4.  错误排查

    如果卸载失败请根据“$GAUSSLOG/om/gs_uninstall-YYYY-MM-DD_HHMMSS.log”中的日志信息排查错误。

## 服务器环境清理<a name="section115851720132817"></a>

在 openGauss 卸载完成后，如果不需要在环境上重新部署 openGauss，可以运行脚本 gs_postuninstall 对 openGauss 服务器上环境信息做清理。openGauss 环境清理是对环境准备脚本 gs_preinstall 所做设置的清理。

## 前提条件<a name="section975651019307"></a>

- openGauss 卸载执行成功。
- root 用户互信可用。
- 只能使用 root 用户执行 gs_postuninstall 命令。

## 操作步骤<a name="section1485102913306"></a>

1.  以 root 用户登录 openGauss 服务器。
2.  查看 root 用户互信是否建立，如果 root 用户没有建立互信，需要手工建立 root 用户互信

    ```
    # 检查互信
    [root@db01 ~]# ssh db02 date
    Mon Oct 25 15:15:56 CST 2021
    [root@db01 ~]# ssh db03 date
    Mon Oct 25 15:16:01 CST 2021
    [root@db01 ~]# ssh db01 date
    Mon Oct 25 15:16:07 CST 2021
    ```

3.  进入 script 路径下。

    ```
    [root@db01 ~]# cd /opt/software/GaussDB_Kernel/script/
    [root@db01 script]# ls
    checkRunStatus.py  gs_checkos    gs_lcctl          gs_resize      HADR.py                 nodegroup_migrate.sh  util
    cmd_sender.py      gs_checkperf  gs_om             gs_shrink      impl                    py_pstree.py
    CSVInfo.py         gs_collector  gs_postuninstall  gs_ssh         __init__.py             stage_step
    GaussRoach.py      gs_expand     gs_preinstall     gs_sshexkey    JsonToDbClustorInfo.py  SyncDataToStby.py
    gs_backup          gs_hotpatch   gspylib           gs_uninstall   killall                 uninstall_force.py
    gs_check           gs_install    gs_replace        gs_upgradectl  local                   uploader.py
    ```

4.  使用 gs_postuninstall 进行清理。若为环境变量分离的模式安装的数据库需要 source 环境变量分离文件 ENVFILE。

    ```
    [root@db01 script]# ./gs_postuninstall -U omm -X /opt/software/GaussDB_Kernel/clusterconfig.xml --delete-user --delete-group
    Parsing the configuration file.
    Successfully parsed the configuration file.
    Check log file path.
    Successfully checked log file path.
    Checking unpreinstallation.
    Successfully checked unpreinstallation.
    Deleting Cgroup.
    Successfully deleted Cgroup.
    Deleting the instance's directory.
    Successfully deleted the instance's directory.
    Start to delete the installation directory.
    Successfully deleted the installation directory.
    Deleting the temporary directory.
    Successfully deleted the temporary directory.
    Deleting remote OS user.
    Successfully deleted remote OS user.
    Deleting software packages and environmental variables of other nodes.
    Successfully deleted software packages and environmental variables of other nodes.
    Deleting logs of other nodes.
    Successfully deleted logs of other nodes.
    Deleting software packages and environmental variables of the local node.
    Successfully deleted software packages and environmental variables of the local nodes.
    Deleting local OS user.
    Successfully deleted local OS user.
    Deleting local node's logs.
    Successfully deleted local node's logs.
    Successfully cleaned environment.
    ```

    omm 为运行 openGauss 的操作系统用户名，/opt/software/GaussDB_Kernel/clusterconfig.xml 为 openGauss 配置文件路径。

    若为环境变量分离的模式安装的数据库需删除之前 source 的环境变量分离的 env 参数。

    unset MPPDB\\\_ENV\\\_SEPARATE\\\_PATH

5.  删除 openGauss 数据库各节点 root 用户的互信

    ```
    [root@db01 ~]# \rm -rf /root/.ssh
    ```

6.  错误排查

    如果一键式环境清理失败请根据“$GAUSSLOG/om/gs_postuninstall-YYYY-MM-DD_HHMMSS.log”中的日志信息排查错误。
