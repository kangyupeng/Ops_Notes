# Zabbix



## zabbix 简介

### zabbix 原理

### zabbix 架构

### zabbix 功能

### zabbix 发展





## zabbix部署

### 环境、版本

| Key    | Value      |
| ------ | ---------- |
| OS     | CentOS 7.2 |
| zabbix | 3.2        |
|        |            |



### 服务端单机部署实施步骤

1. 关闭防火墙

   ```bash
   # systemctl stop firewalld.service
   # systemctl disable firewalld.service
   ```

2. 关闭selinux

3. 配置yum源

   如果不是7的系统或者不打算安装3.2版本的，都可以根据自己的情况修改地址

   ```bash
   # rpm -ivh http://mirrors.aliyun.com/epel/7/x86_64/Packages/e/epel-release-7-11.noarch.rpm 
   # rpm -ivh http://repo.zabbix.com/zabbix/3.2/rhel/7/x86_64/zabbix-release-3.2-1.el7.noarch.rpm 
   ```

4. 搭建一个LAMP架构平台

   1. 配置yum并安装LAMP

      ```bash
      # wget -P /etc/yum.repos.d http://mirrors.aliyun.com/repo/Centos-7.repo
      # yum -y install mariadb mariadb-server php php-mysql httpd
      ```

   2. 启动并初始化数据库

      ```bash
      # systemctl start mariadb.service           
      # systemctl enable mariadb.service
      # mysql_secure_installation   ##这一步是为了初始化数据库，安装好数据库后必须要做的
      
      Enter current password for root (enter for none):    ##初始化第一步提示直接敲回车
      
      Set root password? [Y/n]  y    ##第二步这里输入y，输入以后就是设置root的密码了，要输入两次你要设置的密码就会提示你 successfully! 即可
      
      ## 后面的选项全部 y 过去就可以初始化完毕了！
      ```

   3. 创建zabbix数据库及用户

      ```bash
      # mysql -uroot -pqwert.12345  -e "create database zabbix default character set utf8 collate utf8_bin;"
      # mysql -uroot -pqwert.12345  -e "grant all on zabbix.* to \"zabbix\"@\"%\" identified by \"zabbix\";"
      # mysql -uzabbix -pzabbix -e "show databases;"
      +--------------------+
      | Database           |
      +--------------------+
      | information_schema |
      | zabbix             |
      +--------------------+
      ```

   4. 启动Apache服务并检测是否能正常访问

      ```bash
      # systemctl start httpd.service
      # systemctl enable httpd.service
      
      # yum -y install elinks
      # elinks -dump 127.0.0.1
         haha
      ```

5. 安装zabbix以及相关的一些软件

   在服务端安装zabbix-get可以方便之后向agent端发起测试采集数据的请求

   ```bash
   # yum -y install zabbix-server-mysql zabbix-web-mysql zabbix-get
   ```

6. 导入数据到zabbix数据库中，这里需要用到数据库的root用户和密码。

   ```bash
   # cd /usr/share/doc/zabbix-server-mysql-3.2.11/
   # zcat create.sql.gz | mysql -uroot -p*** zabbix
   ```

7. 修改zabbix的配置文件

   ```bash
   # vim /etc/zabbix/zabbix_server.conf
   DBHost=localhost
   DBName=zabbix
   DBUser=zabbix
   DBPassword=zabbix
   ```

   这里的DBPassword指明密码的时候不能用引号，因为有的人密码可能会设置得比较复杂，有些复杂的符号，但是不管是单引号，双引号都是不可以的，不然后续会出现问题。

8. 修改时区，把时区改成 Asia/Shanghai

   ```bash
   # vim /etc/httpd/conf.d/zabbix.conf
   
           php_value date.timezone Asia/Shanghai
   ```

9. 启动zabbix服务，并添加到开机自启动

   ```bash
   # systemctl start zabbix-server.service
   # systemctl enable zabbix-server.service
   ```

10. 重启Apache服务

    ```bash
    # systemctl restart httpd.service 
    ```

11. 在web界面进行server端的最后配置

    1. 在firefox浏览器的网址栏输入： http://IP/zabbix/setup.php

    2. 来到 welcome to 页面后，点击右下角的next stop

    3. 来到check of pre-requisites页面，我们可以看到，我们的check项后都是OK的，继续点击右下角的next stop

    4. 来到configure DB connection界面，需要填写我们之前创建的zabbix数据库以及zabbix用户的帐号密码，继续下一步

    5. 来到zabbix server details页面后继续点击下一步

    6. 来到pre-installation summary页面后继续点击下一步

    7. 来到Install界面，可以看到 恭喜你已经成功安装了zabbix

    8. 最后一步了，输入我们的帐号密码进行登录即可

       username：Admin

       Password：zabbix

    9. 之后登录到zabbix的管理界面可以直接输入下面这个地址进去即可：

       http://IP/zabbix

12. 安装完毕



### 服务端监控自己配置





### agent端加入到监控





