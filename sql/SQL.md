## DBeaver基础操作【附实操图】

## 1、连接数据库

1、下载sql数据库

网址：[MySQL 官方下载地址：[MySQL :: Download MySQL Community Server](https://dev.mysql.com/downloads/mysql/)](https://dev.mysql.com/downloads/file/?id=552803)



使用可视化工具：DBeaver



1、管理员模式下使用命令行启动：

```
net start MySQL
```





## 2、建立数据库

DBeaver 操作

1. 建立数据库

<img src="https://cdn.jsdelivr.net/gh/junting-123/my-blog-images/img/20260711141549196.png" alt="image-20260711141548141" style="zoom:50%;" />

![image-20260711141615594](https://cdn.jsdelivr.net/gh/junting-123/my-blog-images/img/20260711141616531.png)

2. 建立成功

<img src="https://cdn.jsdelivr.net/gh/junting-123/my-blog-images/img/20260711141649579.png" alt="image-20260711141641290" style="zoom:50%;" />



## 3、导入数据

DBeaver 操作

1. 导入数据

<img src="https://cdn.jsdelivr.net/gh/junting-123/my-blog-images/img/20260711141754845.png" alt="image-20260711141753844" style="zoom:50%;" />



<img src="https://cdn.jsdelivr.net/gh/junting-123/my-blog-images/img/20260711141933025.png" alt="image-20260711141931287" style="zoom:50%;" />

2. 免费版仅支持 csv 格式，导入csv表

<img src="https://cdn.jsdelivr.net/gh/junting-123/my-blog-images/img/20260711142845340.png" alt="image-20260711142844024" style="zoom:50%;" />

3. 导出表成功

<img src="C:/Users/79153/AppData/Roaming/Typora/typora-user-images/image-20260711143027824.png" alt="image-20260711143027824" style="zoom:50%;" />



## 4、SQL 编辑器

DBeaver 操作

1. 打开sql脚本编辑器：

<img src="https://cdn.jsdelivr.net/gh/junting-123/my-blog-images/img/20260711143406942.png" alt="image-20260711143405828" style="zoom:50%;" />



2. 查看连接的数据库：
   ![image-20260711143723231](https://cdn.jsdelivr.net/gh/junting-123/my-blog-images/img/20260711143724308.png)



3. 保存脚本

写完数据库脚本，ctrl+s进行保存，保存出现的位置如下：

```
# 从 test 表中查询所有列的数据，并且给这张表起了一个别名叫做 t
select * from test t ;
```

![image-20260711144329612](https://cdn.jsdelivr.net/gh/junting-123/my-blog-images/img/20260711144330521.png)

点击导出数据：

<img src="https://cdn.jsdelivr.net/gh/junting-123/my-blog-images/img/20260711150537025.png" alt="image-20260711145147198" style="zoom:50%;" />