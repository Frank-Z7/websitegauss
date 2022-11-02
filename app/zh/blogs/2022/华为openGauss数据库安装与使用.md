---
title: '华为openGauss数据库安装与使用'

date: '2021-12-29'

category: 'blog'
tags: ['华为openGauss数据库安装与使用']

archives: '2021-12'

author: 'Vector '

summary: '华为openGauss数据库安装与使用'

img: '/zh/blogs/2022/title/img6.png'

times: '12:30'
---

# 华为 openGauss 数据库安装与使用<a name="ZH-CN_TOPIC_0000001187213634"></a>

主要参考博客:[opengauss 单机部署-墨天轮](https://www.modb.pro/doc/4705)

[企业版安装 | openGauss](https://opengauss.org/zh/docs/2.0.0/docs/installation/%E4%BC%81%E4%B8%9A%E7%89%88%E5%AE%89%E8%A3%85.html)

## 1. 虚拟机安装<a name="section187171915319"></a>

先做安装准备，点击链接[下载](https://download3.vmware.com/software/wkst/file/VMware-workstation-full-16.1.2-17966106.exe)VMware Workstation Pro16，此处为 Windows 10 使用。

- 1.1 VMware 安装

  打开下载好的 exe 文件，即开始安装：

  <img src='./figures/615c183c2ab3f51d914bfbb6.png'>

  安装位置默认在 C 盘，点击更改可以修改安装位置，我安装到了 E:\\VMware\\下，安装路径尽量不要有中文，记得勾选 PATH 按钮，这样不用自己再添加环境变量，可勾选增强型键盘驱动程序，此功能可更好地处理国际键盘和带有额外按键的键盘：

  <img src='./figures/615c15c42ab3f51d91484e93.png'>

  一直点击下一步：

  <img src='./figures/615c11832ab3f51d914222f4.png'>

  <img src='./figures/615c11832ab3f51d914222dd.png'>

  <img src='./figures/615c11832ab3f51d914222e9.png'>

  <img src='./figures/615c11832ab3f51d91422301.png'>

  点击输入许可证，密钥可以自己购买，或者百度搜索以下，多尝试几个，下面是我当时安装使用的密钥，不知道现在失效没有：

  <img src='./figures/615c183c2ab3f51d914bfbaf.png'>

  安装后可能要求重启系统，重启后进入软件。依次点击导航栏中的 帮助 -\> 关于 VMware Workstation ，查看许可证信息的状态，如下图所示即为激活成功。

  <img src='./figures/615c15c42ab3f51d91484e9e.png'>

- 1.2 虚拟机部署 centos

  可以在官方网站下载 centos7，只有 centos7.6 支持安装 opengauss，如果找不到 7.6 版本的 centos，也可安装稍高版本的 centos，安装完之后需要在系统文件中做相关修改，我下载的是 centos7.9，文件太大了，需要下一段时间，记得更改下载保存的位置，我放在了 E:\\Linux\\下。我第一次安装时不知道必须安装 centos7，安装成了 centos8，而重新安装时部分截图忘记保存，所以下面部分截图出现的 centos8，大家视为 centos7 就好。

  <img src='./figures/615c15c42ab3f51d91484ead.png'>

  下载完成，打开 VMware 选择新建虚拟机：

  <img src='./figures/615c191d2ab3f51d914d3f1b.png'>

  <img src='./figures/615c191d2ab3f51d914d3f25.png'>

  浏览文件，选择 centos7 的下载目录，选择镜像文件：

  <img src='./figures/615c191d2ab3f51d914d3f32.png'>

  设置虚拟机的名称和账户名，以及密码：

  <img src='./figures/615c191d2ab3f51d914d3f43.png'>

  选择虚拟机的安装位置：

  <img src='./figures/615c19272ab3f51d914d4e90.png'>

  设置磁盘的容量，默认为 20GB，我修改为了 40GB，点击下一步即可：

  <img src='./figures/615c19272ab3f51d914d4e97.png'>

  自定义硬件可以根据自己的需求，修改 centos 的设置：

  <img src='./figures/615c19272ab3f51d914d4e9d.png'>

  内存大小默认为 1GB，我设置为了 2GB：

  <img src='./figures/615c19272ab3f51d914d4ea8.png'>

  网络适配器选择 NAT 模式，设置完成之后点击确定：

  <img src='./figures/615c19272ab3f51d914d4eaf.png'>

  <img src='./figures/615c19302ab3f51d914d5de4.png'>

  等待安装：

  <img src='./figures/615c19302ab3f51d914d5dd8.png'>

  <img src='./figures/615c19302ab3f51d914d5df7.png'>

  <img src='./figures/615c19302ab3f51d914d5e02.png'>

  <img src='./figures/615c19302ab3f51d914d5e11.png'>

  <img src='./figures/615c193f2ab3f51d914d72c2.png'>

  中间会出现这个页面让你设置，如果你没赶快进行操作，就跳过去了，设置不设置都没有关系，安装完成之后也可以设置：

  <img src='./figures/615c193f2ab3f51d914d72ba.png'>

  如下是，点击各个按钮进行时间、显示、输入法的设置：

  <img src='./figures/615c193f2ab3f51d914d72fc.png'>

  <img src='./figures/615c19492ab3f51d914d811b.png'>

  <img src='./figures/615c19492ab3f51d914d8137.png'>

  <img src='./figures/615c19492ab3f51d914d8153.png'>

  设置完成之后继续安装，安装完毕，输入设置的密码之后，回车：

  <img src='./figures/615c193f2ab3f51d914d72e9.png'>

  安装成功！

  <img src='./figures/615c19492ab3f51d914d8161.png'>

- 1.3 centos 配置

  - 1.3.1 设置系统版本

    因为 opengauss 要求的 centos 版本是 7.6，因此我们需要修改/etc/redhat-release 文件：

    <img src='./figures/615c15c42ab3f51d91484ed6.png'>

    ```
     #进入管理员模式
     su
     #打开文件，进行编辑
     vi /etc/redhat-release
    ```

    修改成如下内容 CentOS Linux release 7.6 \(Core\)：

    <img src='./figures/615c15c42ab3f51d91484ec6.png'>

  - 1.3.2 网络设置

    使用 ifconfig 或者 ip addr 可以查看自己的 ip 地址

    <img src='./figures/615c16922ab3f51d914979b2.png'>

    我的网卡的名字为 ens-33，接下来，给网卡增加 DNS：echo 'DNS1=114.114.114.114'\>\>/etc.sysconfig/network-scripts/ifcfg-ens33

    重启网卡：systemctl restart network，测试是否可以访问：ping www.baidu.com

    <img src='./figures/615c16922ab3f51d914979bf.png'>

    如上图所示，则可以访问。

  - 1.3.3 修改主机名

    ```
     echo "vector" > /etc/hostname
     echo "192.168.48.128 vector" >>/etc/hostd
    ```

    最后系统重启后记得查看主机名是否修改成功：

    ```
     cat /etc/hostname
    ```

  - 1.3.4 配置 YUM 源

    删除系统自带的 yum 源

    ```
     rm -rf /etc/yum.repos.d/*
    ```

    下载阿里云 yum 源

    ```
     wget -O /etc/yum.repos.d/CentOS-Base http://mirrors.aliyun.com/repo/Centos7.repo
    ```

    生成仓库缓存

    ```
     yum makecache
    ```

    安装 python3.6，一定要装 3.6 版本

    ```
     sudo yum install epel-release
     sudo yum install python36
    ```

  - 1.3.5 关闭防火墙

    ```
     sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config
    ```

  - 1.3.6 设置字符集

    ```
    cat >>/etc/profile<<EOF
    export LANG=en_US.UTF-8
    EOF
    ```

  - 1.3.7 修改地区和时间

    ```
    cp /usr / share/zoneinfo/Asia/Shanghai /etc/localtime
    ```

  - 1.3.8 关闭交换内存

    swapoff -a

  - 1.3.9 设置 root 用户远程登录

    修改/etc/ssh/sshd_config 文件，去掉 PermitRootLogin 前的\#号，并改 no 为 yes。修改之后我的系统文件中内容如下：

    <img src='./figures/615c16922ab3f51d914979c5.png'>

  - 1.3.10 重启虚拟机

    ```
    reboot
    ```

## 2. openGauss 安装<a name="section178293429494"></a>

- 2.1 安装前准备

  我下载的安装包是[企业版 2.0.0 版本](https://opengauss.obs.cn-south-1.myhuaweicloud.com/2.0.0/x86/openGauss-2.0.0-CentOS-64bit-all.tar.gz)，刚开始装的是极简版，但是极简版缺少安装包，缺少工具，最后回归企业版。安装过程，参考了[官方文档](https://opengauss.org/zh/docs/2.0.0/docs/installation/%E4%BC%81%E4%B8%9A%E7%89%88%E5%AE%89%E8%A3%85.html)。

  将下载好的安装包解压，我放在了目录/opt/software/openGauss/：

  ```
  #先创建文件夹
  mkdir -p /opt/software/openGauss
  #设置访问权限
  chmod 755 -R /opt/software
  ```

  - 不建议把安装包的存放目录规划到 openGauss 用户的根目录或其子目录下，可能导致权限问题。
  - openGauss 用户须具有/opt/software/openGauss 目录的读写权限。

  在安装包所在的目录下，解压安装包 openGauss-2.0.0-CentOS-64bit-all.tar.gz。安装包解压后，会有 om 安装包和 server 安装包。继续解压 om 安装包，会在/opt/software/openGauss 路径下自动生成 script 子目录，并且在 script 目录下生成 gs_preinstall 等各种 om 工具脚本。

  建议跟我目录放的一样，不然容易出问题，解压命令如下：

  ```
  cd /opt/software/openGauss
  tar -zxvf openGauss-2.0.0-CentOS-64bit-all.tar.gz
  tar -zxvf openGauss-2.0.0-CentOS-64bit-om.tar.gz
  ```

  <img src='./figures/615c16932ab3f51d914979dd.png'>

  在该目录下获取 XML 文件 script/gspylib/etc/conf/cluster_config_template.xml，重命名为 cluster_config.xml 放在/opt/software/openGauss/下，并将以下模板修改为自己的信息放入 xml 文件，第 37 行的 15400 表示设置了数据库的端口号，以下模板只需要更改两点：ip 地址和主机名：

  ```
  <?xml version="1.0" encoding="UTF-8"?>
  <ROOT>
      <!-- openGauss整体信息 -->
      <CLUSTER>
          <!-- 数据库名称 -->
          <PARAM name="clusterName" value="dbCluster" />
          <!-- 数据库节点名称(hostname) -->
          <PARAM name="nodeNames" value="node1_hostname" />
          <!-- 数据库安装目录-->
          <PARAM name="gaussdbAppPath" value="/opt/huawei/install/app" />
          <!-- 日志目录-->
          <PARAM name="gaussdbLogPath" value="/var/log/omm" />
          <!-- 临时文件目录-->
          <PARAM name="tmpMppdbPath" value="/opt/huawei/tmp" />
          <!-- 数据库工具目录-->
          <PARAM name="gaussdbToolPath" value="/opt/huawei/install/om" />
          <!-- 数据库core文件目录-->
          <PARAM name="corePath" value="/opt/huawei/corefile" />
          <!-- 节点IP，与数据库节点名称列表一一对应 -->
          <PARAM name="backIp1s" value="192.168.0.1"/>
      </CLUSTER>
      <!-- 每台服务器上的节点部署信息 -->
      <DEVICELIST>
          <!-- 节点1上的部署信息 -->
          <DEVICE sn="node1_hostname">
              <!-- 节点1的主机名称 -->
              <PARAM name="name" value="node1_hostname"/>
              <!-- 节点1所在的AZ及AZ优先级 -->
              <PARAM name="azName" value="AZ1"/>
              <PARAM name="azPriority" value="1"/>
              <!-- 节点1的IP，如果服务器只有一个网卡可用，将backIP1和sshIP1配置成同一个IP -->
              <PARAM name="backIp1" value="192.168.0.1"/>
              <PARAM name="sshIp1" value="192.168.0.1"/>

  	    <!--dbnode-->
  	    <PARAM name="dataNum" value="1"/>
  	    <PARAM name="dataPortBase" value="15400"/>
  	    <PARAM name="dataNode1" value="/opt/huawei/install/data/dn"/>
              <PARAM name="dataNode1_syncNum" value="0"/>
          </DEVICE>
      </DEVICELIST>
  </ROOT>
  ```

  根据我的 ip 地址 192.168.48.128 和我的主机名 vector 更改之后文件内容如下：

  ```
  <?xml version="1.0" encoding="UTF-8"?>
  <ROOT>
      <!-- openGauss整体信息 -->
      <CLUSTER>
          <!-- 数据库名称 -->
          <PARAM name="clusterName" value="dbCluster" />
          <!-- 数据库节点名称(hostname) -->
          <PARAM name="nodeNames" value="vector" />
          <!-- 数据库安装目录-->
          <PARAM name="gaussdbAppPath" value="/opt/huawei/install/app" />
          <!-- 日志目录-->
          <PARAM name="gaussdbLogPath" value="/var/log/omm" />
          <!-- 临时文件目录-->
          <PARAM name="tmpMppdbPath" value="/opt/huawei/tmp" />
          <!-- 数据库工具目录-->
          <PARAM name="gaussdbToolPath" value="/opt/huawei/install/om" />
          <!-- 数据库core文件目录-->
          <PARAM name="corePath" value="/opt/huawei/corefile" />
          <!-- 节点IP，与数据库节点名称列表一一对应 -->
          <PARAM name="backIp1s" value="192.168.48.128"/>
      </CLUSTER>
      <!-- 每台服务器上的节点部署信息 -->
      <DEVICELIST>
          <!-- 节点1上的部署信息 -->
          <DEVICE sn="vector">
              <!-- 节点1的主机名称 -->
              <PARAM name="name" value="vector"/>
              <!-- 节点1所在的AZ及AZ优先级 -->
              <PARAM name="azName" value="AZ1"/>
              <PARAM name="azPriority" value="1"/>
              <!-- 节点1的IP，如果服务器只有一个网卡可用，将backIP1和sshIP1配置成同一个IP -->
              <PARAM name="backIp1" value="192.168.48.128"/>
              <PARAM name="sshIp1" value="192.168.48.128"/>

  	    <!--dbnode-->
  	    <PARAM name="dataNum" value="1"/>
  	    <PARAM name="dataPortBase" value="15400"/>
  	    <PARAM name="dataNode1" value="/opt/huawei/install/data/dn"/>
              <PARAM name="dataNode1_syncNum" value="0"/>
          </DEVICE>
      </DEVICELIST>
  </ROOT>
  ```

  执行以下命令准备安装环境：

  ```
  cd /opt/software/openGauss/script
  ./gs_preinstall -U omm -G dbgrp -L -X /opt/software/openGauss/cluster_config.xml
  ```

  <img src='./figures/615c14052ab3f51d9145c394.png'>

  如上表示预安装成功！

- 2.2 执行安装

  - 2.2.1 检查

    检查安装包和 openGauss 配置文件在规划路径下是否已存在，如果没有，重新执行预安装，确保预安装成功，再执行以下步骤。

  - 2.2.2 切换用户

    登录到 openGauss 的主机，并切换到 omm 用户：

    ```
    su omm
    ```

    omm 指的是前置脚本 gs_preinstall 中-U 参数指定的用户。

    安装脚本 gs_install 必须以前置脚本中指定的 omm 执行，否则，脚本执行会报错。

  - 2.2.3 安装

    使用 gs_install 安装 openGauss。

    ```
    gs_install -X /opt/software/openGauss/cluster_config.xml
    ```

    /opt/software/openGauss/cluster_config.xml 为 openGauss 配置文件的路径。在执行过程中，用户需根据提示输入数据库的密码，密码具有一定的复杂度，为保证用户正常使用该数据库，请记住输入的数据库密码。

    设置的密码要符合复杂度要求：

    - 最少包含 8 个字符。
    - 不能和用户名、当前密码（ALTER）、或当前密码反序相同。
    - 至少包含大写字母（A-Z），小写字母（a-z），数字，非字母数字字符（限定为\~!@\#$%^&\*\(\)-\_=+|\[\{\}\];:,<.\>/?）四类字符中的三类字符。

      <img src='./figures/615c14052ab3f51d9145c3a9.png'>

      执行如下命令检查数据库状态是否正常:

      ```
      gs_om -t status
      ```

      <img src='./figures/615c16932ab3f51d914979e7.png'>

      cluster_state 显示“Normal”表示数据库可正常使用。

      <img src='./figures/615c14a32ab3f51d9146b96f.png'>

      如首次安装数据库不成功，则卸载后重新安装，卸载方式如下：

      ```
      gs_uninstall ‐‐delete‐data
      ```

  - 2.2.4 初始化数据库

    使用 SQL 语句创建数据库 database 时，指定数据库的字符集为 GBK。

    ```
    #后面跟的是端口号,我的是15400
    gsql -d postgres -p 15400
    ```

    ```
    CREATE DATABASE mydb WITH ENCODING 'GBK' template = template0;
    ```

    <img src='./figures/615c14a32ab3f51d9146b994.png'>

    显示如下信息:

    ```
    CREATE DATABASE
    ```

    创建 schema：

    ```
    CREATE SCHEMA tpcds;
    ```

    创建表：

    ```
    CREATE TABLE tpcds.warehouse_t1
    (
        W_WAREHOUSE_SK            INTEGER               NOT NULL,
        W_WAREHOUSE_ID            CHAR(16)              NOT NULL,
        W_WAREHOUSE_NAME          VARCHAR(20)                   ,
        W_WAREHOUSE_SQ_FT         INTEGER                       ,
        W_STREET_NUMBER           CHAR(10)                      ,
        W_STREET_NAME             VARCHAR(60)                   ,
        W_STREET_TYPE             CHAR(15)                      ,
        W_SUITE_NUMBER            CHAR(10)                      ,
        W_CITY                    VARCHAR(60)                   ,
        W_COUNTY                  VARCHAR(30)                   ,
        W_STATE                   CHAR(2)                       ,
        W_ZIP                     CHAR(10)                      ,
        W_COUNTRY                 VARCHAR(20)                   ,
        W_GMT_OFFSET              DECIMAL(5,2)
    );
    ```

    <img src='./figures/615ffa572ab3f51d91af9b67.jpg'>

    查看表信息：

    <img src='./figures/615ffb2b2ab3f51d91b0c00c.jpg'>

    ```
    insert into tpcds.warehouse_t1(w_warehouse_sk,w_warehouse_id) values(12,'000001');
    insert into tpcds.warehouse_t1(w_warehouse_sk,w_warehouse_id) values(25,'000002');
    select w_warehouse_sk, w_warehouse_id from tpcds.warehouse_t1;
    ```

    向数据库中添加数据之后查看：

    <img src='./figures/615ffbbb2ab3f51d91b187c6.jpg'>

    如果不知道自己的端口号,可根据以下方式查看:

    查看自己的 cluster_config.xml 文件,查看自己将端口号设置为了多少.

    使用如下命令查看:

    ```
    gs_om -t status --detail
    cd /opt/huawei/install/data/dn
    ```

    <img src='./figures/615c14a32ab3f51d9146b960.png'>

- 2.3 JDBC 连接数据库

  - 2.3.1 准备 java 环境

    查看 centos 的 java 环境，centos 自带 java1.8，需要安装配套的 javac，注意要是 1.8.0 版。

    ```
    yum install java-1.8.0-openjdk-devel.x86_64
    ```

    下载驱动包 2.0.0 版本[postgresql.jar](https://opengauss.obs.cn-south-1.myhuaweicloud.com/2.0.0/x86/openGauss-2.0.0-JDBC.tar.gz)，放在路径/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.302.b08-0.el7_9.x86_64/jre/lib/ext 下：

    ```
    cp postgresql.jar /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.302.b08-0.el7_9.x86_64/jre/lib/ext
    ```

    <img src='./figures/615c16f62ab3f51d914a1b92.png'>

    测试是否具备运行 java 代码的环境：

    ```
    java -version
    javac -version
    ```

    <img src='./figures/615c16f62ab3f51d914a1ba8.png'>

    已具备运行环境！

  - 2.3.2 准备好连接的 java 代码

    记得替换成你设置的用户名、密码、端口号，如果你是按照我前面的操作，用户名应该是 omm，

    ```
    import java.sql.Connection;
    import java.sql.DriverManager;
    import java.sql.PreparedStatement;
    import java.sql.SQLException;
    import java.sql.Statement;
    import java.sql.CallableStatement;

    public class test{//keep
    	public static Connection getConnect(String username, String passwd)
        {
            //驱动类。
            String driver = "org.postgresql.Driver";
            //数据库连接描述符。将15400替换为自己的端口号
            String sourceURL = "jdbc:postgresql://127.0.0.1:15400/postgres";
            Connection conn = null;

            try
            {
                //加载驱动。
                Class.forName(driver);
            }
            catch( Exception e )
            {
                e.printStackTrace();
                return null;
            }

            try
            {
                 //创建连接。
                conn = DriverManager.getConnection(sourceURL, username, passwd);
                System.out.println("Connection succeed!");
            }
            catch(Exception e)
            {
                e.printStackTrace();
                return null;
            }

            return conn;
        };

        //try to connect
    	public static void main(String[] args)
    	{
    		// TODO Auto-generated method stub
    		Connection conn = getConnect("user", "password");//replace by my user and password
    		//BatchInsertData(conn);
    		try
    		{
    			conn.close();
    		}
    		catch (SQLException e)
    		{
    			e.printStackTrace();
    		}
    	}
    }
    ```

  - 2.3.3 配置服务端远程连接

    以操作系统用户 omm 登录数据库。

    配置 listen_addresses，即远程客户端连接使用的数据库主节点 ip 或者主机名。

    使用如下命令查看数据库主节点目前的 listen_addresses 配置。

    ```
    gs_guc check -I all -c "listen_addresses"
    ```

    使用如下命令把要查询出的 ip 追加到 listen_addresses 后面，多个配置项之间用英文逗号分隔。例如，追加 ip 地址 10.11.12.13。

    ```
    gs_guc set -I all -c "listen_addresses='localhost,10.11.12.13'"
    ```

    执行如下命令重启 openGauss

    ```
    gs_om -t stop && gs_om -t start
    ```

    <img src='./figures/615c15482ab3f51d9147a2ba.png'>

  - 2.3.4 连接

    首先需要启动数据库

    ```
    su omm
    gs_om -t start
    ```

    运行 java 代码

    ```
    javac test.java
    java test
    ```

    <img src='./figures/615c15482ab3f51d9147a2b3.png'>

  - 2.3.5 操纵数据

    使用如下 java 代码访问并对表中数据进行查询（记得替换用户、密码和端口）：

    ```
    import java.sql.Connection;
    import java.sql.DriverManager;
    import java.sql.PreparedStatement;
    import java.sql.SQLException;
    import java.sql.Statement;
    import java.sql.CallableStatement;
    import java.sql.ResultSet;
    import java.sql.SQLException;

    public class gausstest{//keep
    	public static Connection getConnect(String username, String passwd)
        {
            //驱动类。
            String driver = "org.postgresql.Driver";
            //数据库连接描述符。
            String sourceURL = "jdbc:postgresql://127.0.0.1:15400/postgres";
            Connection conn = null;

            try
            {
                //加载驱动。
                Class.forName(driver);
            }
            catch( Exception e )
            {
                e.printStackTrace();
                return null;
            }

            try
            {
                 //创建连接。
                conn = DriverManager.getConnection(sourceURL, username, passwd);
                System.out.println("Connection succeed!");
            }
            catch(Exception e)
            {
                e.printStackTrace();
                return null;
            }

            return conn;
        };

        //try to connect
    	public static void main(String[] args) throws SQLException
    	{
    		// TODO Auto-generated method stub
    		Connection conn = getConnect("user", "password");//replace by my user and password
    		//BatchInsertData(conn);
    		Statement st = conn.createStatement();
    		String sql = "select w_warehouse_sk,w_warehouse_id from tpcds.warehouse_t1";
    		ResultSet rs = st.executeQuery(sql);
    		 while(rs.next()) {
                  int w_warehouse_sk = rs.getInt("w_warehouse_sk");
                  String w_warehouse_id = rs.getString("w_warehouse_id");
                  System.out.println("w_warehouse_sk = " + w_warehouse_sk + "; w_warehouse_id = " + w_warehouse_id);
             }
    		try
    		{
    			conn.close();
    			st.close();
    			rs.close();
    		}
    		catch (SQLException e)
    		{
    			e.printStackTrace();
    		}
    	}
    }
    ```

    <img src='./figures/615ffdad2ab3f51d91b42898.jpg'>

## 3. 遇到的问题<a name="section1680211152813"></a>

我感觉我把所有能遇到的问题都遇到了,最后成功是重装一遍,什么问题没遇到。

- 3.1 使用 gs_ctl 提示找不到命令

如下图所示：

<img src='./figures/615c13152ab3f51d91446977.png'>

参看博客[Linux 下解决命令未找到的问题 - ML。 - 博客园 \(cnblogs.com\)](https://www.cnblogs.com/mnote/p/8832806.html)，对于本问题主要使用的命令是：

```
#进入管理员模式
su
which gs_ctl
```

<img src='./figures/615c16f62ab3f51d914a1b6d.png'>

接下来需要做的是把查找出的路径直接链接到/usr/bin 下。操作如下：

```
ln -s xxx/xxx /usr/bin
```

以上 xxx 代表你查出来的路径。

<img src='./figures/615c533b2ab3f51d91a72523.jpg'>

- 3.2 gs_om 命令找不到

  不得不说极简版安装包下没有 gs_om 文件，我搜遍了也没有，在企业版中，我因为懒得重装把我同学下载的企业版中的 gs\_之类的文件全拷过来了，但是后来遇到了其他问题，我又重装了，不知道我这个操作最终会带来什么影响。

- 3.3 sudo 和 su 都用不了

  sudo chmod -R 777 / 修改根目录权限问题修复，参考了[ 关于不小心 777 导致没法 sudo 权限后的修改解决办法\_空木格子的博客-CSDN 博客](https://blog.csdn.net/qq_39543212/article/details/84107240)

  我应该是因为 sudo 用不了提示 sudo: must be setuid root，然后我进入根目录下修改了某个文件为 777，直接导致 su 也用不了。这下好了，要用 su 让我先用 sudo 修改相关文件，要用 sudo 让我先用 su 修改文件！

  解决这个问题需要先进入安全模式，进入方法为：在开机的过程中按 shift 或 ESC 键，好像在系统中按 F1 还是 F2 也可以。

  此时，已经进入到具有 root 权限的字符界面，输入以下命令解决了。

  ```
  ls -l /usr/bin/sudo
  chown root:root /usr/bin/sudo
  chmod 4755 /usr/bin/sudo
  ```

- 3.4 预安装失败

  <img src='./figures/615c53892ab3f51d91a7b1e6.png'>

  本问题先参考了链接[openGaussDB 初体验（上） - 云+社区 - 腾讯云 \(tencent.com\)](https://cloud.tencent.com/developer/article/1675265)以下内容，但是没有解决。

  <img src='./figures/615c14052ab3f51d9145c37e.png'>

  我解决这个问题的过程是这样的：找到虚拟网络编辑器，电脑连了自己的热点（我听我同学说她的用校园网就不行），然后还原默认设置：

  <img src='./figures/615c16f62ab3f51d914a1b7d.png'>

  <img src='./figures/615c14a32ab3f51d9146b955.png'>

  然后配置了静态的 ip 地址，参考了[ CentOS 7 连接不到网络解决方法\(设置静态 ip\)\_gaokcl 的博客-CSDN 博客\_centos7 无法连接网络](https://blog.csdn.net/gaokcl/article/details/82834925?utm_medium=distribute.pc_relevant.none-task-blog-2~default~CTRLIST~default-2.no_search_link&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2~default~CTRLIST~default-2.no_search_link)。但是神奇的是，这样就可以了。不过后来还是重装了。

- 3.5 重装 openGauss 时端口被占用

  报错：\[GAUSS-50601\] : The port \[15400\] is occupied or the ip address is incorrectl，有两种方法：

  修改 xml 文件中的端口号

  杀掉占用端口的进程

- 3.6 右上角网络连接图标消失

  参考了[centos7 右上角网络连接图标消失\_shuest 的博客-CSDN 博客\_centos7 右上角没有网络图标](https://blog.csdn.net/zs391077005/article/details/106885104?utm_medium=distribute.pc_relevant.none-task-blog-2~default~CTRLIST~default-1.no_search_link&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2~default~CTRLIST~default-1.no_search_link)

  ```
  chkconfig network off
  chkconfig network on
  service NetworkManager stop
  service NetworkManager start
  ```

  但是有可能遇到后两条命令用不了，然后又去查怎么办，最后也没解决，我重装了。累了累了。

- 3.7 循环显示登录界面无法进入

  看图吧，我最后又进安全模式解决的，最后修改/etc/selinux/config 配置，将 SELINUX 选项由 SELINUX=enforcing 改成 SELINUX=disabled，重启系统后发现就可以正常登陆系统了：

  <img src='./figures/615c14052ab3f51d9145c371.png'>

- 3.8 Connection refused

  首先需要启动数据库，不启动数据库会出现如下错误：

  未设置服务端远程连接也会出现以上问题，见 2.3.3

  <img src='./figures/615c15482ab3f51d9147a2aa.png'>

- 3.9 加载驱动出现问题

  以下是开发流程：

  <img src='./figures/采用JDBC开发应用程序的流程.png'>

  驱动需要按照 2.3.1 所说，放在指定文件夹下，不然在加载驱动的时候会出现问题。

- 3.10 unreported exception SQLException

  在本地编译 java 服务的时候，编译报错：未报告的异常错误; 必须对其进行捕获或声明以便抛出。

  <img src='./figures/615fff622ab3f51d91b644eb.jpg'>

  添加代码 throw SQLException 即可：

  <img src='./figures/615ffeef2ab3f51d91b5bb72.jpg'>
