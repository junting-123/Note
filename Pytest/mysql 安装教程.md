# 【2026最新】 MySQL 数据库安装教程（超详细图文版）

> MySQL是一个开源的关系型数据库管理系统，由[Oracle公司](https://zhida.zhihu.com/search?content_id=275397693&content_type=Article&match_order=1&q=Oracle公司&zhida_source=entity)维护开发。它的主要用途是存储和管理结构化数据，通过SQL（结构化查询语言）进行数据的增删改查操作。
>
> MySQL提供了社区版（免费）和商业版（付费）两个版本。对于学习、个人开发、小规模生产环境来说，社区版完全够用。



## 一、下载MySQL

MySQL 官方下载地址：[MySQL :: Download MySQL Community Server](https://dev.mysql.com/downloads/mysql/)

![image-20260603134010433](https://cdn.jsdelivr.net/gh/junting-123/my-blog-images/img/20260603134011622.png)

![image-20260603135120917](https://cdn.jsdelivr.net/gh/junting-123/my-blog-images/img/20260603135121872.png)



## 二、 MySQL 安装与配置

### 2.1  解压安装包

1. 将下载的 ZIP 压缩包解压到指定目录（建议路径无中文、无空格，避免后续报错）

![image-20260603135348838](https://cdn.jsdelivr.net/gh/junting-123/my-blog-images/img/20260603135349819.png)

### 2.2 配置环境变量

> 配置环境变量后，可在任意命令行窗口操作 MySQL，无需切换到`bin`目录

1. 右键点击「此电脑」→「属性」→「高级系统设置」→「环境变量」

![image-20260603135503079](https://cdn.jsdelivr.net/gh/junting-123/my-blog-images/img/20260603135504013.png)



2. 在系统变量中新建MYSQL_HOME

![image-20260603135620047](https://cdn.jsdelivr.net/gh/junting-123/my-blog-images/img/20260603135620780.png)



3. 在「系统变量」中找到「Path」，点击「编辑」

![image-20260603135724326](https://cdn.jsdelivr.net/gh/junting-123/my-blog-images/img/20260603135725551.png)

![image-20260603135813095](https://cdn.jsdelivr.net/gh/junting-123/my-blog-images/img/20260603135919398.png)



4. 验证是否生效

打开 powellshell 命令行：

```
mysql
```

![image-20260603140058288](https://cdn.jsdelivr.net/gh/junting-123/my-blog-images/img/20260603140059270.png)

提示 Can't connect to MySQL server on ‘localhost’ 证明添加成功



### 2.3 初始化mysql

命令行：

```
mysqld --initialize-insecure
```

![image-20260603140309663](https://cdn.jsdelivr.net/gh/junting-123/my-blog-images/img/20260603140321120.png)

如果出现没有出现报错信息，则证明data目录初始化没有问题，此时再查看MySQL目录下已经有data目录生成。

![image-20260603140352473](https://cdn.jsdelivr.net/gh/junting-123/my-blog-images/img/20260603140353398.png)



### 2.4 注册安装 MySQL 服务

以**管理员身份**运行命令行：

```
sudo mysqld -install
```

![image-20260603140908544](https://cdn.jsdelivr.net/gh/junting-123/my-blog-images/img/20260603140909758.png)



2.5 启动 MySQL 服务

执行启动命令：

```
net start mysql  // 启动mysql服务
    
net stop mysql  // 停止mysql服务
```

启动成功提示：`MySQL 服务正在启动.. MySQL服务已经启动成功`



三、MySQL 登录与密码修改

1. 首次登录无需密码：

```
mysql -u root -p
```



2. 修改 root 密码

登陆后执行：

```
# 设置密码
mysqladmin -u root password <你的密码>

# 退出 mysql 命令行
exit；

# 验证密码
mysql -uroot -p<你的密码>
```

到这一步我们的 mysql 就设置密码成功，也安装好了我们的 mysql 数据库。



## 三、MySQL 可视化工具连接

### 3.1 DBeaver 下载

为了更便利操作MySQL，推荐免费工具**DBeaver**

下载地址：[Download | DBeaver Community](https://dbeaver.io/download/)

下载好后双击运行程序：

![image-20260603162659428](https://cdn.jsdelivr.net/gh/junting-123/my-blog-images/img/20260603162700423.png)

安装成功后，界面显示

![image-20260603162829299](https://cdn.jsdelivr.net/gh/junting-123/my-blog-images/img/20260603162830387.png)



### 3.2 连接到数据库 

1. 管理员身份运行 powershell 打开 mysql 服务

```
net start mysql
```



2. 连接数据库

![image-20260603163338607](https://cdn.jsdelivr.net/gh/junting-123/my-blog-images/img/20260603163339766.png)

连接好后显示：

![image-20260603163521429](https://cdn.jsdelivr.net/gh/junting-123/my-blog-images/img/20260603163522533.png)



### 3.3 测试连接

通过命令行测试连接情况：

```
# 连接mysql
mysql -u root -p

# 查看现有数据库
show databases;

# 创建一个新的测试数据库
create database test_connection;
```

查看 DBeaver 界面，连接成功

![image-20260603164241652](https://cdn.jsdelivr.net/gh/junting-123/my-blog-images/img/20260603164242658.png)