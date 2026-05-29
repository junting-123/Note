# Jenkins Pipeline语法

> pipeline流水线，可以直观的展示每个阶段做的任务，以及每个阶段耗费的时间。
>
> pipeline不在使用鼠标来实现自动构建，也不要去看控制台日志，而是全程使用代码的方式来实现，构建完成后会展示一个视图，用来展示每个阶段完成的情况



写 Jenkins Pipeline 只需要掌握：

| 需要学的                                                    | 不需要深究的      |
| :---------------------------------------------------------- | :---------------- |
| Pipeline 基本语法（`pipeline`、`stages`、`stage`、`steps`） | Groovy 的高级特性 |
| `sh`、`echo`、`git` 等常用步骤                              | Groovy 元编程     |
| 简单的变量、条件判断（`if`）、循环（`for`）                 | 复杂闭包          |



## 1.Pipeline 的基本框架

如下所示：

```groovy
pipeline {
    agent any
    stages {
        stage('阶段名') {
            steps {
                // 要执行的命令
            }
        }
    }
}
```

- `pipeline {}`：最外层，固定写法
- `agent any`：定义任务在那台主机上运行，可以是any,none等
- `stages {}`：类似于一个大项目的集合，用来包含所有 `stage`子任务
- `stage('名字') {}`：类似一个项目中的单个任务，用来包含`steps`子层
- `steps {}`：里面写具体要执行的命令



编写pipeline脚本

```groovy
pipeline{
	agent any
	
	stages{
		stage('拉取代码'){
			step{
				echo '正在拉取代码...'
				git 'https://github.com/junting-123/calculator/blob/test/calculatorweb/calculatorweb.py'
			}
		}
		
		stage('打包Docker镜像'){
			steps{
				echo '正在打包...'
				script{
					def app = docker.build("你的镜像名称:${env.BUIL_ID}")
				}
			}
		}
		
		stage('推送Docker镜像'){
			steps{
				script{
					docker.withRgistry('docker-credentials-id'){
						app.push（）
					}
				}
			}
		}
	}
}
```





## 2.常用步骤

| 步骤      | 作用             | 示例                                            |
| :-------- | :--------------- | :---------------------------------------------- |
| `sh`      | 执行 shell 命令  | `sh 'docker build -t myapp .'`                  |
| `git`     | 拉取代码         | `git 'https://github.com/xxx/xxx.git'`          |
| `echo`    | 打印信息         | `echo '开始构建'`                               |
| `withEnv` | 设置环境变量     | `withEnv(['PATH=/usr/local/bin']) { sh '...' }` |
| `script`  | 包裹一段脚本代码 | 用于写 `if` 或 `for`                            |



## 3.变量和引用的写法

```groovy
pipeline {
    environment {
        APP_NAME = 'my-app'
        IMAGE_TAG = 'latest'
    }
    stages {
        stage('Build') {
            steps {
                sh "docker build -t ${APP_NAME}:${IMAGE_TAG} ."
                // 注意：sh 里面必须用双引号，变量才能替换
            }
        }
    }
}
```



## 4.条件判断（简单if）

```groovy
stage('Push') {
    steps {
        script {
            if (env.BRANCH_NAME == 'main') {
                sh 'docker push myapp:latest'
            } else {
                echo '非主分支，不推送'
            }
        }
    }
}
```



## 5.Jenkins默认工作目录修改及迁移

> Jenkins 的 `JENKINS_HOME` 路径，是Jenkins 的数据目录（包括插件、配置、job、用户等）存储的默认目录。默认的空间是C:\ProgramData\Jenkins.jenkins，git下拉的项目会在该目录的workspace下，造成C盘空间增大。

1.暂停Jenkins服务

直接关掉命令行窗口，或者按 `Ctrl + C`



2.**创建新目录**

在 D 盘创建你想要的目录，例如：

```
D:\jenkins_home
```



3.**迁移现有数据（重要！）**

把你当前的 Jenkins 主目录下的**所有内容**复制到新目录。

当前目录很可能是：

```
C:\Users\用户名\.jenkins
```



4.**用新参数启动 Jenkins**

打开命令行（CMD 或 PowerShell），**切换到 `jenkins.war` 所在的目录**，然后执行：

```bash
java -DJENKINS_HOME=D:\jenkins_home -jar jenkins.war
```

验证是否启动成功



5.永久方案：创建启动脚本

在 `jenkins.war` 所在的目录下，新建一个文本文件，命名为 `start_jenkins.bat`，内容如下：

```bash
@echo off
set JENKINS_HOME=D:\jenkins_home
java -DJENKINS_HOME=%JENKINS_HOME% -jar jenkins.war
pause
```

以后直接**双击这个 `start_jenkins.bat`** 文件，Jenkins 就会自动使用 D 盘的主目录启动。