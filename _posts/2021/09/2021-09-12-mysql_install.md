

大家好，我是指北君，mysql是大家最常用的数据库，也是大家学习编程前需要提前准备的，但是，mysql的安装比较繁琐，难倒了一批入门的小白，所以，今天指北君就带大家安装mysql，此文章只要是针对windows系统的，后续指北君也会针对macOS系统写一篇，下面开始正题吧



### 1 下载并安装MySQL

首先输入如下命令下载Yum Repository，大概25KB的样子

```plain
[root@localhost ~]# wget -i -c http://dev.mysql.com/get/mysql57-community-release-el7-10.noarch.rpm
```



下载完成后，就可以用以下命令直接yum安装了。

```plain
[root@localhost ~]# yum -y install mysql57-community-release-el7-10.noarch.rpm
```



然后开始安装MySQL服务器，这一步会比较耗时

```plain
[root@localhost ~]# yum -y install mysql-community-server
```



如果命令行出现如下图片语句，则说明MySQL安装完成

![img](https://img-blog.csdn.net/20180531164100716)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)



### 2 MySQL数据库设置

 首先用以下命令启动MySQL

```plain
[root@localhost ~]# systemctl start  mysqld.service
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

通过以下命令查看MySQL运行状态

```html
[root@localhost ~]# systemctl status mysqld.service
```

![img](https://img-blog.csdn.net/20180531164553246)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

如上图所示，此时MySQL已经开始正常运行

接着我们找进入mysql的初始密码，通过如下命令可以在日志文件中找出密码：

```plain
[root@localhost ~]# grep "password" /var/log/mysqld.log
```

![img](https://img-blog.csdn.net/20180531164427771)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)



 如下命令进入数据库：

```plain
[root@localhost ~]# mysql -uroot -p
```

输入初始密码（是上面图片最后面的 no;e!5>>alfg）

```plain
mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY 'new password';
```

将‘new password’换成你自己的密码



### 3 开启mysql的远程访问

执行以下命令开启远程访问限制（注意：下面命令开启的IP是 192.168.0.1，如要开启所有的，用%代替IP）：

```plain
grant all privileges on *.* to 'root'@'192.168.0.1' identified by 'password' with grant option;
```



![img](https://img-blog.csdn.net/20180918150432333?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM2NTgyNjA0/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

然后再输入下面两行命令

```plain
mysql> flush privileges; 
```

```plain
mysql> exit
```



![img](https://img-blog.csdn.net/20180531193815519)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)



### 4 为firewalld添加开放端口

通过如下命令行分别为mysql和tomcat添加端口号，其端口号分别为3306和8080

```plain
[root@localhost ~]# firewall-cmd --zone=public --add-port=3306/tcp --permanent
```

```plain
[root@localhost ~]# firewall-cmd --zone=public --add-port=8080/tcp --permanent
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

然后再重新载入

```plain
[root@localhost ~]# firewall-cmd --reload
```

![img](https://img-blog.csdn.net/20180531195102403)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)



### 5 更改mysql的语言

首先重新登录mysql，然后输入status：

![img](https://img-blog.csdn.net/2018053119584461)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)



可以看到，绿色箭头处不是utf-8

因此我们先退出mysql，然后再到、etc目录下的my.cnf文件下修改一下文件内容

![img](https://img-blog.csdn.net/20180531200259161)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

进入文件后，新增四行代码：

![img](https://img-blog.csdn.net/20180531201748668)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

保存更改后的my.cnf文件后，重启下mysql，然后输入status再次查看，你就会发现变化啦

![img](https://img-blog.csdn.net/20180531200538548)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

最后，到Windows下用cmd命令启动mysql就ok了

