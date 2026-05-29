# Ubuntu 24.04上安装 Jenkins

> [Jenkins](https://link.zhihu.com/?target=https%3A//www.jenkins.io/) 是一种免费的，开放的[持续集成](https://zhida.zhihu.com/search?content_id=225316958&content_type=Article&match_order=1&q=持续集成&zhida_source=entity)工具，主要用于任务自动化。它有助于简化持续开发，测试以及新提交代码的部署。
>
> 本文将介绍如何在 Ubuntu 22.04 / Ubuntu 20.04 上安装 Jenkins。



## 1.使用 apt 命令安装 Java

作为一个 Java 应用程序，Jenkins 要求 Java 8 及更高版本，检查系统上是否安装了 Java

```bash
java --version
```

如没安装，执行如下命令：

```bash
sudo apt install -y openjdk-17-jre-headless
java --version
```

## 2.下载安装 Jenkins

下载地址[War Jenkins Packages]([Download and deploy](https://www.jenkins.io/download/))

下载该版本的 `jenkins.war` 文件

![image-20260526232237349](https://cdn.jsdelivr.net/gh/junting-123/my-blog-images/img/20260526232238429.png)

![image-20260526232257394](https://cdn.jsdelivr.net/gh/junting-123/my-blog-images/img/20260526232258415.png)

下载完成之后就把该 `war` 包上传至服务器

先在Linux服务器上创建一个 `/soft/jenkins` 的文件夹用于存放 `jenkins` 的安装包，并进入到该目录下上传文件：

```
# 创建 /soft/jenkins 文件夹
mkdir /soft/jenkins

# 进入到 /soft/jenkins 文件夹下
cd /soft/jenkins

# 安装 lrzsz 软件包
sudo apt-get install lrzsz -y

# 使用 rz 命令将下载好的安装包上传到该目录下
rz

# 查看是否上传成功
ls

# 上传成功，直接用java -jar 命令行启动，可以通过httpPort指定端口号
java -jar jenkins.war --httpPort=8080
```

![image-20260526233323351](https://cdn.jsdelivr.net/gh/junting-123/my-blog-images/img/20260526233324551.png)

看到successfully，说明启动成功。这个时候就可以去游览器上访问下 `ip:port`，比如：192.168.171.128:8080



## 3.配置Jenkins

在游览器上展示页面如下，就可以进行初始化了：

![image-20260526234043164](https://cdn.jsdelivr.net/gh/junting-123/my-blog-images/img/20260526234044102.png)

将管理员账户密码 复制到对应的地方，点击继续。

![image-20260526234105174](https://cdn.jsdelivr.net/gh/junting-123/my-blog-images/img/20260526234106170.png)

进入到创建管理员页面，填写账户信息后 保存并完成

![image-20260526234609978](https://cdn.jsdelivr.net/gh/junting-123/my-blog-images/img/20260526234610901.png)

进入以下页面配置 `jenkins` 的 `url` ，一般使用默认的就行了，`保存并完成`

![image-20260526234718268](https://cdn.jsdelivr.net/gh/junting-123/my-blog-images/img/20260526234719208.png)

初始化完成

![image-20260526234752795](https://cdn.jsdelivr.net/gh/junting-123/my-blog-images/img/20260526234753784.png)

可以点击 `开始使用 Jenkins` 直接登录进入 `Jenkins`

![image-20260526234843271](https://cdn.jsdelivr.net/gh/junting-123/my-blog-images/img/20260526234844338.png)

现在，可以开始使用你的Jenkins啦！