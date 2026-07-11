# 前言

**基于B/S架构的Web服务CI/CD自动化测试与部署平台**

**技术栈总结**：`Docker` +`pytest` +`Flask`+ `Allure`+ `JMeter` + `Jenkins` +`git`+`Natapp`+`飞书机器人`

**项目背景**：开发并维护一套完整的 CI/CD 自动化流水线，实现从代码推送到构建、测试、部署的全流程自动化。Web 服务与测试程序均通过 **Docker** 容器化运行于**虚拟机**，确保各环节环境一致性并提升交付效率。

**主要工作**：

- **全流程容器化**：使用 **Docker** 将 Web 服务打包为镜像，部署至虚拟机；接口测试与性能测试脚本同样运行在 Docker 容器中，实现测试环境与生产环境的高度一致；

- **接口自动化测试**：基于 **Pytest + Allure** 搭建接口自动化接口测试框架，生成可视化测试报告；
- **性能与稳定性测试**：引入 **JMeter** 进行性能与稳定性测试，验证服务在高并发场景下的响应能力与可靠性；
- **CI/CD 流水线**：基于 **Jenkins** 搭建自动化流水线，结合 **Git Webhook** 实现代码推送自动触发构建、测试与部署；实现**路径级精准触发**，不同目录变更自动匹配并执行对应测试任务；
- **实时通知**：集成**飞书机器人**，实时推送构建、测试与部署结果。



## 技术栈总结

| 类别              | 技术            | 用途                                               |
| :---------------- | :-------------- | :------------------------------------------------- |
| **后端服务**      | Python + Flask  | 实现加减乘除运算的计算器 API 服务                  |
| **容器化**        | Docker          | 将服务端及其依赖环境打包为镜像，确保环境一致性     |
| **自动化测试**    | pytest + Allure | 接口功能验证，生成可视化测试报告                   |
| **性能测试**      | JMeter          | 对服务端进行压力与稳定性测试，生成 HTML 性能报告   |
| **持续集成/部署** | Jenkins + Git   | 代码推送自动触发构建、测试与部署流程               |
| **内网穿透**      | Natapp          | 将本地 Jenkins 服务暴露至公网，接收 GitHub Webhook |
| **通知推送**      | 飞书机器人      | 实时推送构建、测试与部署结果至团队群聊             |
| **版本控制**      | Git + GitHub    | 代码托管与版本管理                                 |



必须补充的 3 个加分项

1. **异常场景测试（最重要）**
   - 除零测试
   - 超大数字（如 `10**100`）
   - 非数字参数
   - 服务端返回非 JSON
   - 网络超时
     → 这些才是测试的价值所在。
2. **测试数据与结果可视化**
   - 每 1 小时统计一次通过率，用 matplotlib 画成趋势图（哪怕只是截图附在简历 PDF 末尾）。
3. **一键环境重建**
   - 写一个 `start.sh` 脚本：自动启动虚拟机中的服务端（或用 Docker-compose 同时拉起服务端+客户端测试），面试官可以现场跑起来看。



## 一. **客户端-服务端**基础连通性测试

### 1.1 ping（确认网络互通）

> 要让虚拟机跟本地电脑互相通信（互相ping通），虚拟机必须是桥接模式，并且桥接到跟本地电脑同一个网卡。

**C/S 架构测试（Client/Server）**，全称是**客户端-服务器架构测试**，它是一种**软件测试方法**，专门用来验证那些分为“客户端”和“服务器”两部分的软件系统是否正常工作。

我们将虚拟机作为服务端用以启动计算器服务程序，本地电脑作为客户端用以进行测试。首先确认客户端-服务端连接情况。



1. 启动虚拟机中linux终端（ctrl+alt+T），执行：

```linux
# Linux
ip addr show | grep inet
```

典型输出：

192.168.0.104   <- 这个就是服务端IP



2. 启动本地PowerShell，执行：

```cmd
ipconfig
```

典型输出：

   IPv4 地址 : 192.168.0.101<- 这个就是客户端IP



3. ping（确认）网络层互通

服务端linux：

```
ping 192.168.0.101
```

输出如下：

```
64 bytes from 192.168.0.101: icmp_seq=1 ttl=128 time=1.09 ms
```

其中含义：

| 部分                 | 含义               | 你的数值   | 说明                       |
| :------------------- | :----------------- | :--------- | :------------------------- |
| `64 bytes`           | 接收到的数据包大小 | 64字节     | 正常，标准ping包大小       |
| `from 192.168.0.101` | 响应来自这个IP     | 本地电脑IP | ✅ 虚拟机成功收到本地的回复 |
| `icmp_seq=1`         | 第几个ping请求     | 第1个      | 序号连续说明没丢包         |
| `ttl=128`            | 生存时间           | 128        | Windows系统特征值          |
| `time=1.09 ms`       | 往返时间           | 1.09毫秒   | 很快，网络质量好           |



客户端windows：

```
 ping 192.168.0.104
```

输出如下：

192.168.0.104 的 Ping 统计信息:

 数据包: 已发送 = 4，已接收 = 4，丢失 = 0 (0% 丢失)



表示网络层面服务端和客户端互通！



### 1.2  MobaXterm 远程连接

> 用 MobaXterm 连接你的 Ubuntu 虚拟机，最大的好处是**用一个工具解决了你在 Windows 上管理 Linux 的所有需求**

1. 连接如下，其中Remote host为虚拟机ip：

![image-20260517233812824](https://cdn.jsdelivr.net/gh/junting-123/my-blog-images/img/20260517233816660.png)

2. 测试是否远程连接

```
ip addr show
```

显示虚拟机ip地址。



## 二、本地构建程序打包成镜像

> Docker 镜像的核心特性就是**可移植性**——镜像包含了运行程序所需的一切（代码、运行环境、依赖库、配置），只要目标机器安装了 Docker，就可以直接运行这个镜像，不管是在本地、虚拟机还是云服务器上。

整体流程：

calculator.py → Dockerfile → Docker镜像 → Docker容器



### 2.1 本地编写calculatorweb.py

#### 2.1.1 编写计算器功能

```python
# calculator.py
def add(a, b):
    return a + b

def subtract(a, b):
    return a - b

def multiply(a, b):
    return a * b

def divide(a, b):
    if b == 0:
        return "错误：除数不能为0"
    return a / b

def main():
    print("=" * 40)
    print("      简单计算器")
    print("=" * 40)
    
    while True:
        print("\n请选择运算：")
        print("1. 加法 (+)")
        print("2. 减法 (-)")
        print("3. 乘法 (*)")
        print("4. 除法 (/)")
        print("5. 退出")
        
        choice = input("\n请输入选择 (1-5): ")
        
        if choice == '5':
            print("再见！")
            break
        
        if choice not in ['1', '2', '3', '4']:
            print("无效选择，请重新输入")
            continue
        
        try:
            num1 = float(input("请输入第一个数字: "))
            num2 = float(input("请输入第二个数字: "))
        except ValueError:
            print("错误：请输入有效的数字")
            continue
        
        if choice == '1':
            result = add(num1, num2)
            print(f"\n{num1} + {num2} = {result}")
        elif choice == '2':
            result = subtract(num1, num2)
            print(f"\n{num1} - {num2} = {result}")
        elif choice == '3':
            result = multiply(num1, num2)
            print(f"\n{num1} × {num2} = {result}")
        elif choice == '4':
            result = divide(num1, num2)
            print(f"\n{num1} ÷ {num2} = {result}")

if __name__ == "__main__":
    main()
```



#### 2.2.2 程序封装成 Web API

> 由于后续要进行接口测试和压力测试，服务端必须是一个**网络服务**。所以给计算器程序，套上一层HTTP通话的外壳，在这里我们使用Flask，它是一个轻量级Python Web 框架

```bash
# 下载Flask
pip install flask
```

calculatorweb.py如下：

```python
from flask import Flask, request, jsonify

app = Flask(__name__)

def add(a, b):
    return a + b

def subtract(a, b):
    return a - b

def multiply(a, b):
    return a * b

def divide(a, b):
    if b == 0:
        return "错误：除数不能为0"
    return a / b

@app.route('/calc', methods=['GET'])
def calc():
    a = float(request.args.get('a'))
    b = float(request.args.get('b'))
    op = request.args.get('op')
    
    if op == 'add':
        return jsonify({'result': add(a, b)})
    elif op == 'sub':
        return jsonify({'result': subtract(a, b)})
    elif op == 'mul':
        return jsonify({'result': multiply(a, b)})
    elif op == 'div':
        return jsonify({'result': divide(a, b)})

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=8080)
```

运行代码，显示如下

![image-20260520224056746](https://cdn.jsdelivr.net/gh/junting-123/my-blog-images/img/20260520224058055.png)



#### 2.2.3 测试Web程序



1. 打开一个新的终端输入：

```
curl "http://127.0.0.1:8080/calc?a=10&b=5&op=add"
```



2. 在浏览器测试

直接访问：

```txt
http://127.0.0.1:8080/calc?a=10&b=5&op=add
```



3. 从其他设备（手机、另一台电脑）

确保在同一个局域网，访问：

```
http://192.168.0.101:8080/calc?a=10&b=5&op=add
```

**预期返回：** `{"result":15}`



### 2.2 安装Docker Desktop

> 镜像是一种轻量级的、可执行的独立软件包。用来打包软件运行环境和基于运行环境的开发软件，它包含运行某个软件所需要的内容，包括代码、运行时、库、环境变量和配置文件。



#### 2.2.1 安装Docker Desktop

网址：https://hub.docker.com/

![image-20260519150106975](https://cdn.jsdelivr.net/gh/junting-123/my-blog-images/img/20260519150109293.png)

下载Docker Desktop.



#### 2.2.2 安装WSL2

如果你是第一次安装 Docker Desktop，需要启用 WSL2：

```
wsl --install 
```

因为 Docker 容器本质上必须运行在 Linux 内核之上，所以需要安装WSL 2内核，它利用 Windows 自带的轻量级虚拟机技术，这也是目前最高效、体验最好的方式。



#### 2.2.3 验证安装是否成功

重启后，桌面会出现Docker图标，双击启动（首次启动可能需要几分钟，状态栏出现鲸鱼图标即启动成功）。打开Windows终端（或CMD、PowerShell），输入以下命令：

```bash
docker --version  # 查看版本，出现版本号说明安装成功
docker run hello-world  # 运行测试容器，出现"Hello from Docker!"说明正常工作
```

如果报错可能是**网络连接问题**——Docker无法连接到位于国外的官方镜像仓库。这在国内是普遍现象，主要是因为网络访问不稳定或速度过慢导致的超时。

修改配置并重启Docker Desktop：

![image-20260520165300629](https://cdn.jsdelivr.net/gh/junting-123/my-blog-images/img/20260520165327150.png)

修改的Docker Engine如下：

```
{
  "registry-mirrors": [
    "https://docker.m.daocloud.io",
    "https://docker.nju.edu.cn",
    "https://mirror.baidubce.com"
  ]
}
```

出现以下信息，说明**Docker 安装成功了！**

![image-20260520165511559](https://cdn.jsdelivr.net/gh/junting-123/my-blog-images/img/20260520165512476.png)



### 2.3 创建Dockerfile

> Dockerfile是用于构建 Docker 镜像的文本文件，其中包含了一系列指令和参数，用于定义镜像的内容、环境和运行方式

在项目`calculator.py` **同一个文件夹**里，创建一个名为 `Dockerfile`（没有扩展名）的文件：

- `calculatorweb.py`
- `dockerfile`
- `requirements.txt`

其中requirements.txt:

```txt
flask
```

dockerfile基本结构如下：

```text
# 使用官方 Python 镜像作为基础镜像
FROM python:3.12

# 设置工作目录 /app
WORKDIR /app

# 复制当前目录下的所有文件到工作目录
COPY . /app

# 安装应用程序依赖
RUN pip install -r requirements.txt

# 暴露应用程序需要的端口
EXPOSE 8081

# 定义容器启动时运行的命令
CMD ["python", "calculatorweb.py"]
```



### 2.4  构建镜像并运行docker

> 读取 Dockerfile 中的"说明书"，自动把"运行环境 + 代码"打包成一个镜像。



从Dockerfile构建镜像并运行docker

```
# 构建
docker build -t calculator-web .

# 验证镜像是否创建成功
docker images
# 应该看到 calculator-web 在列表中

# 运行（注意端口映射：主机端口:容器端口）
docker run -d -p 8081:8081 calculator-web

# 验证容器是否运行成功 
docker ps
# 应该看到 calculator-web 容器状态为 Up

# 测试
curl "http://localhost:8081/calc?a=10&b=5&op=add"
# 应该返回 {"result":15}
```



## 三、镜像上传至虚拟机

### 3.1 将 Docker 镜像保存为文件

首先，需要把你的 `calculator-app` 镜像导出成一个 `.tar` 文件。这个文件就可以像其他普通文件一样，被复制和移动了。

命令行运行：

```bash
docker save -o calculator-web.tar calculator-web
```

- `docker save`: 用来导出镜像的命令。
- `-o calculator-app.tar`: 指定输出的文件名，这里命名为 `calculator-app.tar`。
- `calculator-app`: 你要导出的本地镜像名称。

**验证一下**：命令执行成功后，你应该能在当前目录（通常是 `E:\code\calculator`）下看到一个新生成的 `calculator-app.tar` 文件。



注意：由于Docker Desktop 开启后，VMware 虚拟机能开机，但很可能会变得非常卡顿或直接报错无法启动。所以在打包好tar文件后，关闭 Docker。



### 3.2 上传文件到虚拟机

接下来，我们把刚才生成的 `calculator-app.tar` 文件上传到虚拟机里。

1. 通过SSH连接虚拟机



2. 使用 MobaXterm 内置的 SFTP 功能，将本地 `calculator-app.tar` 文件拖拽到虚拟机的 `/root/` 或 `/home/` 目录下。

### 3.3 在虚拟机里加载 Docker 镜像

1. 虚拟机中安装docker

> Docker 是一个独立的容器化平台，每个需要运行 Docker 容器的地方（无论是你的本地电脑、虚拟机，还是云服务器），都必须自己安装一个 Docker 引擎

```
#更新软件包并安装依赖
sudo apt update
sudo apt install -y ca-certificates curl

#添加 Docker 的官方 GPG 密钥

# 创建存放密钥的目录
sudo install -m 0755 -d /etc/apt/keyrings
# 下载并安装 GPG 密钥
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
# 设置密钥文件权限
sudo chmod a+r /etc/apt/keyrings/docker.asc

# 添加 Docker 软件源
# 将 Docker 仓库添加到 APT 源列表
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
# 更新 APT 索引
sudo apt update

# 正式安装 Docker 引擎及相关组件
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

#验证安装并运行测试
sudo docker run hello-world

#出于安全考虑，Docker 命令默认需要 root 权限。如果你不想每次都输入 #sudo，可以执行下面的命令将当前用户加入 docker 用户组
sudo usermod -aG docker $USER
```

**重要：** 执行完此命令后，需要**退出并重新登录**虚拟机（或关闭当前终端窗口重新打开），权限才会生效。之后就能直接运行 `docker ps` 等命令而无需加 `sudo` 了。



2. 在虚拟机中加载镜像：

```bash
# 进入文件所在目录
cd /root/

# 加载 Docker 镜像
docker load -i calculator-web.tar

# 验证镜像
docker images

# 运行容器（后台运行，端口映射 8080:8080）
docker run -d -p 8080:8080 --name calculator calculator-web

# 查看容器状态
docker ps
# 看到 calculator 容器状态为 Up
```

好了，现在程序已经在服务端通过 Docker 运行起来了。接下来，你需要从客户端（本地电脑）通过网络访问虚拟机里的计算器服务，然后执行自动化测试。



### 3.4 自动部署编写 jenkinsfile

```
calculatorweb/              		
├── calculator.py                      
├── requirenments           
├── dockerfile              
└── jenkinsfile  					# 主流水线
└── jenkinsfiles/  
    └── send_feishu.groovy			# 子脚本
    └── check_path.groovy	
```



send_feishu.groovy：

```groovy
def call(Map params) {
    def webhook = "https://open.feishu.cn/open-apis/bot/v2/hook/d27ef06c-cdad-4eff-a7d0-46f089c661bf"
    def status = params.RESULT
    def statusText = status == 'SUCCESS' ? '✅ 通过' : '❌ 失败'
    def statusColor = status == 'SUCCESS' ? 'green' : 'red'
    
    sh """
        curl -X POST -H 'Content-Type: application/json' \
        -d '{
            "msg_type": "interactive",
            "card": {
                "header": {
                    "title": {"content": "Jenkins 部署通知", "tag": "plain_text"},
                    "template": "${statusColor}"
                },
                "elements": [{
                    "tag": "div",
                    "text": {
                        "content": "**项目**: ${params.JOB_NAME}\\n**构建号**: #${params.BUILD_NUMBER}\\n**部署结果**: ${statusText}",
                        "tag": "lark_md"
                    }
                }]
            }
        }' ${webhook}
    """
    echo "部署通知已发送"
}

return this
```



check_path:

```groovy
def call(String targetPath){
    // 获取本次构建的变更文件列表
    def changeLogSets = currentBuild.changeSets
    def changedFiles = []

    // 收集所有变更的文件路径
    for (changeLogSet in changeLogSets){
        for (entry in changeLogSet.getItems()){
            for (path in entry.getAffectedPaths()){
                changedFiles.add(path)
            }
        }
    }

    // 如果没有变更记录 (手动构建、定时执行)， 默认执行
    if (changedFiles.isEmpty()){
        println "⏭️ 无变更记录（手动/定时构建），默认执行"
        return true
    }

    // 检查是否包含目标路径的变更
    for (file in changedFiles){
        if (file.startsWith(targetPath)){
            println "✅ 检测到目标路径变更： ${file}"
            return true
        }
    }

    // 没有相关更新
    println "❌ 没有检测到 ${targetPath} 的变更,跳过构建"
    return false
}

    return this
```



jenkinsfile :

```groovy
pipeline {
    agent any

    // 可选：添加参数，允许手动构建时跳过路径检查
    parameters {
        booleanParam(name: 'SKIP_PATH_CHECK', defaultValue: false, description: '手动构建时跳过路径检查')
    }

    stages {
        // 新增：检查变更路径的 stage
        stage('检查变更路径') {
            when {
                expression { !params.SKIP_PATH_CHECK }  // 手动构建时可跳过检查
            }
            steps {
                script {
                    // 加载 checkPath 函数
                    def check_path = load "${env.WORKSPACE}/calculatorweb/jenkinsfiles/check_path.groovy"
                    
                    // 检查是否是 calculatorweb 目录的变更
                    def shouldRun = check_path('calculatorweb/')
                    
                    if (!shouldRun) {
                        echo "⏭️ 没有检测到 calculatorweb 目录的变更，跳过构建"
                        currentBuild.result = 'SUCCESS'
                        // 使用 error 但标记为 ABORTED，这样 post 不会执行
                        currentBuild.description = 'SKIPPED'
                        return
                    }
                }
            }
        }
    
        stage('拉取代码') {
            when{
                expression { currentBuild.description != 'SKIPPED' }
            }
            steps {
                retry(3){ // 失败后重试
                    git branch: 'master',
                        url: 'git@github.com:junting-123/calculator.git',
                        credentialsId: ''
                }
            }
        }

        stage('清理旧镜像') {
            when {
                expression { currentBuild.description != 'SKIPPED' }
            }
            steps {
                sh '''
                    echo "=== 删除旧镜像 ==="
                    docker rmi calculatorweb:latest || true
                '''
            }
        }
    
        stage('构建镜像') {
            when{
                expression { currentBuild.description != 'SKIPPED' }
            }
            steps {
                sh '''
                    cd calculatorweb
                    docker build -t calculatorweb:latest .
                '''
            }
        }

        stage('部署到服务器') {
            when {
                expression { currentBuild.description != 'SKIPPED' }
            }
            steps {
                sh '''
                    echo "=== 停止占用 8081 端口的容器  ==="
                    # 查找占用 8081 端口的容器
                    CONTAINER_ID=$(docker ps -q --filter "publish=8081")
                    if [ -n "$CONTAINER_ID" ]; then
                        echo "停止容器: $CONTAINER_ID"
                        docker stop $CONTAINER_ID
                        docker rm $CONTAINER_ID
                    fi
            
                    # 同时也停止 calculatorweb 容器（如果存在）
                     docker stop calculatorweb || true
                    docker rm calculatorweb || true
                    
                    echo "=== 启动新容器 ==="
                    docker run -d \
                        --name calculatorweb \
                        -p 8081:8081 \
                        calculatorweb:latest
                    
                    echo "=== 部署完成 ==="
                '''
            }
        }

    }
    
    post {
        always {
            script {
                // 只有不是被跳过的构建，才执行
                if (currentBuild.description != 'SKIPPED'){
                def notify = load "${env.WORKSPACE}/calculatorweb/jenkinsfiles/send_feishu.groovy"

                notify([
                    JOB_NAME: env.JOB_NAME,
                    BUILD_NUMBER: env.BUILD_NUMBER,
                    BUILD_URL: env.BUILD_URL,
                    RESULT: currentBuild.result ?: 'SUCCESS'
                ])
                echo "📱 飞书通知已发送"
                }
                else{
                    echo "⏭️ 构建被跳过，不生成通知"
                }
            }
        }

        success {
            echo '✅ 部署通过！'
        }

        failure {
            echo '❌ 部署失败！'
        }

        aborted {
            echo '⏭️ 流水线被跳过（路径不匹配）'
        }
    }
}

```



## 四、接口测试

项目会进行接口测试和jmeter性能测试

### 4.1 test_api 测试用例

根据测试用例的常用方法，编写测试用例，先用表格记录：

![image-20260521001039449](https://cdn.jsdelivr.net/gh/junting-123/my-blog-images/img/20260521001041566.png)

### 4.2 test_data.json 数据写入

将数据写入文件夹中test_data.json

```
{
    "add": [
        {"a": 10, "b": 5, "expected": 15, "desc": "正整数相加"},
        {"a": 10, "b": -3, "expected": 7, "desc": "正数加负数"},
        {"a": -5, "b": -3, "expected": -8, "desc": "负数相加"}
    ],
    "sub": [
        {"a": 10, "b": 3, "expected": 7, "desc": "正整数相减"},
        {"a": 3, "b": 10, "expected": -7, "desc": "结果为负数"}
    ],
    "mul": [
        {"a": 4, "b": 5, "expected": 20, "desc": "正整数相乘"},
        {"a": 5, "b": 0, "expected": 0, "desc": "乘以零"}
    ],
    "div": [
        {"a": 10, "b": 2, "expected": 5, "desc": "整除"},
        {"a": 10, "b": 0, "expected": "错误：除数不能为0", "desc": "除零错误"}
    ]
}
```

### 4.3 test_api.py 测试脚本

代码如下：

```python
import json
import pytest
import requests

# 从 JSON 文件加载测试数据
data = json.load(open("test_data.json", encoding='utf-8'))

# 服务端的地址 - 根据你的部署情况选择
# 如果测试脚本在虚拟机上运行 
# BASE_URL = "http://localhost:8081"

# 如果测试脚本在本地 Windows 运行，访问虚拟机中的服务
BASE_URL = "http://192.168.171.128:8081"

# pytest 提供的功能，用于批量执行测试
@pytest.mark.parametrize("op, cases", data.items())
# 函数名必须以 test_ 开头，pytest 才能识别;op操作符eg：“add",cases对应测试用例表
def test_calculator(op, cases):
    for case in cases:
        resp = requests.get(f"{BASE_URL}/calc", params={"a": case["a"], "b": case["b"], "op": op})
        assert resp.json()["result"] == case["expected"], f'失败：{case["desc"]}'

# 添加这个入口，让脚本可以直接运行
if __name__ == "__main__":
    pytest.main([__file__, "-v"])
```

终端运行：

```bash
pytest test_api.py -v 
```

运行成功就可以查看结果了！



### 4.4 生成 Allure 报告数据

终端运行：

```
# 运行测试，并让 pytest 把报告数据存到 'allure-results' 文件夹
pytest test_api.py --alluredir=./allure-results
```

这一步执行后，你会看到一个 `allure-results` 文件夹，里面是一些 `.json` 文件，它们是报告的“原材料”。

```
# Allure 会自动启动一个本地服务，并打开你的默认浏览器
allure serve ./allure-results
```

执行这条命令后，Allure 会读取数据、生成一个网页版报告，并自动在你的浏览器中打开一个全新的、带图表的测试报告页面。

![image-20260528165123963](https://cdn.jsdelivr.net/gh/junting-123/my-blog-images/img/20260528165125229.png)

程序调整成熟后，我们将他打包上传。

### 4.5 编写 dockerfile

同目录下增加 requirements 和 dockerfile

requirenments 如下：

```txt
# 对应好自己的版本号
pytest==9.0.2
requests==2.33.1
allure-pytest==2.15.3
```

dockerfile 如下：

```
# 使用官方 Python 基础镜像
FROM python:3.12

# 设置工作目录
WORKDIR /test

# 复制依赖文件并安装
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# 复制项目文件
COPY . .

# 启动应用
CMD ["pytest", "test_api.py", "-v"]
```

这一步写完 dockerfile 后，将代码 git push到 github，后续会通过服务端的 jenkins 拉取代码并进行打包，在服务端构建镜像并自动测试。



### 4.6 编写 jenkinsfile 

将后续 jenkins 需要构建的流水线代码写在本地 Jenkinsfile 中，存在 Git 里，随代码一起版本管理。

结构：

```
api_test/              		
├── test_api.py             
├── test_data.json          
├── requirenments           
├── dockerfile              
└── jenkinsfile  					# 主流水线
└── jenkinsfiles/  
    └── send_feishu.groovy			# 子脚本
    └── check_path.groovy	
```



子目录 send_feishu 的 jenkinsflie ，这个流水线主要是将结果上传飞书 ：

```
pipeline {
    agent any
    
    parameters {
        string(name: 'JOB_NAME', defaultValue: '', description: '项目名称')
        string(name: 'BUILD_NUMBER', defaultValue: '', description: '构建编号')
        string(name: 'BUILD_URL', defaultValue: '', description: '构建地址')
        string(name: 'RESULT', defaultValue: 'SUCCESS', description: '构建结果')
    }
    
    stages {
        stage('发送飞书通知') {
            steps {
                script {
                    def webhook = "https://open.feishu.cn/open-apis/bot/v2/hook/d27ef06c-cdad-4eff-a7d0-46f089c661bf"
                    def status = params.RESULT
                    def statusText = status == 'SUCCESS' ? '✅ 通过' : '❌ 失败'
                    def statusColor = status == 'SUCCESS' ? 'green' : 'red'
                    def reportUrl = "${params.BUILD_URL}allure/"
                    
                    sh """
                        curl -X POST -H 'Content-Type: application/json' \
                        -d '{
                            "msg_type": "interactive",
                            "card": {
                                "header": {
                                    "title": {"content": "Jenkins 构建通知", "tag": "plain_text"},
                                    "template": "${statusColor}"
                                },
                                "elements": [{
                                    "tag": "div",
                                    "text": {
                                        "content": "**项目**: ${params.JOB_NAME}\\n**构建号**: #${params.BUILD_NUMBER}\\n**结果**: ${statusText}\\n**报告**: [点击查看](${reportUrl})",
                                        "tag": "lark_md"
                                    }
                                }]
                            }
                        }' ${webhook}
                    """
                    echo "飞书通知已发送"
                }
            }
        }
    }
    
    post {
        success {
            echo '✅ 飞书通知发送成功'
        }
        failure {
            echo '❌ 飞书通知发送失败'
        }
    }
}
```



check_path.groovy 脚本如下：

```
def call(String targetPath){
    // 获取本次构建的变更文件列表
    def changeLogSets = currentBuild.changeSets
    def changedFiles = []

    // 收集所有变更的文件路径
    for (changeLogSet in changeLogSets){
        for (entry in changeLogSet.getItems()){
            for (path in entry.getAffectedPaths()){
                changedFiles.add(path)
            }
        }
    }

    // 如果没有变更记录 (手动构建、定时执行)，默认执行
    if (changedFiles.isEmpty()){
        println "⏭️ 无变更记录（手动/定时构建），默认执行"
        return true
    }

    // 检查是否包含目标路径的变更
    for (file in changedFiles){
        if (file.startsWith(targetPath)){
            println "✅ 检测到目标路径变更： ${file}"
            return true
        }
    }

    // 没有相关更新
    println "❌ 没有检测到 ${targetPath} 的变更,跳过构建"
    return false
}

    return this
```



主流水线  jenkinsfile，讲子目录通过 load 加载：

```
pipeline {
    agent any

    // 可选：添加参数，允许手动构建时跳过路径检查
    parameters {
        booleanParam(name: 'SKIP_PATH_CHECK', defaultValue: false, description: '手动构建时跳过路径检查')
    }

    stages {
        // 新增：检查变更路径的 stage
        stage('检查变更路径') {
            when {
                expression { !params.SKIP_PATH_CHECK }  // 手动构建时可跳过检查
            }
            steps {
                script {
                    // 加载 checkPath 函数
                    def check_path = load "${env.WORKSPACE}/calculatortest/api_test/jenkinsfiles/check_path.groovy"
                    
                    // 检查是否是 api_test 目录的变更
                    def shouldRun = check_path('calculatortest/api_test/')
                    
                    if (!shouldRun) {
                        echo "⏭️ 没有检测到 api_test 目录的变更，跳过构建"
                        currentBuild.result = 'SUCCESS'
                        // 使用 error 但标记为 ABORTED，这样 post 不会执行
                        currentBuild.description = 'SKIPPED'
                        return
                    }
                }
            }
        }
    
        stage('拉取代码') {
            when{
                expression { currentBuild.description != 'SKIPPED' }
            }
            steps {
                retry(3){ // 失败后重试3次
                    git branch: 'master',
                        url: 'git@github.com:junting-123/calculator.git',
                        credentialsId: ''
                }
            }
        }
    
        stage('构建测试镜像') {
            when{
                expression { currentBuild.description != 'SKIPPED' }
            }
            steps {
                sh '''
                    cd calculatortest/api_test
                    docker build -t api-test:latest .
                '''
            }
        }
        
        stage('运行测试并生成Allure数据') {
            when{
                expression { currentBuild.description != 'SKIPPED' }
            }
            steps {
                sh '''
                    cd calculatortest/api_test
                    mkdir -p allure-results
                    docker run --rm \
                        -v $(pwd)/allure-results:/tests/allure-results \
                        api-test:latest \
                        pytest test_api.py --alluredir=/tests/allure-results
                '''
            }
        }
        
    }
    
    post {
        always {
            script {
                // 只有不是被跳过的构建，才执行报告和通知
                if (currentBuild.description != 'SKIPPED'){
                    allure([
                    includeProperties: false,
                    jdk: '',
                    properties: [],
                    reportBuildPolicy: 'ALWAYS',
                    results: [[path: 'calculatortest/api_test/allure-results']]
                ])

                // 2. 发送飞书通知
                def notify = load "${env.WORKSPACE}/calculatortest/api_test/jenkinsfiles/send_feishu.groovy"
                notify([
                    JOB_NAME: env.JOB_NAME,
                    BUILD_NUMBER: env.BUILD_NUMBER,
                    BUILD_URL: env.BUILD_URL,
                    RESULT: currentBuild.result ?: 'SUCCESS'
                ])
                echo "📱 飞书通知已发送"
                }
            }
        }
        success {
            echo '✅ 接口测试通过！'
        }
        failure {
            echo '❌ 接口测试失败！'
        }

        aborted {
            echo '⏭️ 流水线被跳过（路径不匹配）'
        }
    }
}
```



## 五、性能测试

### 5.1 Jmeter安装

Jmter下载路径：[Jmeter 官网下载](http://jmeter.apache.org/download_jmeter.cgi)

![image-20260521143335803](https://cdn.jsdelivr.net/gh/junting-123/my-blog-images/img/20260521143340532.png)



### 5.2 Jmeter环境变量配置

1.  下载安装包后，解压安装 Jmeter 即可

![image-20260521143625320](https://cdn.jsdelivr.net/gh/junting-123/my-blog-images/img/20260521143626408.png)



2. 环境变量配置

找到Jmeter下载后的文件所在目录

此电脑 -> 右键 -> 属性 -> 高级系统设置 -> 环境变量，然后在系统变量中添加JMETER_HOME和CLASSPATH两个变量，并且编辑Path变量的变量值。

- 配置JMETER_HOME变量

> 变量名：JMETER_HOME
> 变量值：E:\app\Jmeter\apache-jmeter-5.6.3

- 配置JMETER_HOME变量

> 变量名：CLASSPATH
> 变量值：%JMETER_HOME%\lib\ext\ApacheJMeter_core.jar; %JMETER_HOME%\lib\jorphan.jar;

- 配置Path变量

> 变量名：Path
> 变量值：%JMETER_HOME%\bin



3. 查看环境变量是否配置成功

终端运行：

```bash
# 查看是否能查看到Jmeter版本信息
jmeter -v

# 查看是否能启动jmeter
jmeter
```

预期输出如下：

![image-20260521145426865](https://cdn.jsdelivr.net/gh/junting-123/my-blog-images/img/20260521145427893.png)

并弹出Jmeter界面

![image-20260521145857145](https://cdn.jsdelivr.net/gh/junting-123/my-blog-images/img/20260521145858945.png)

**注意**：不管用使用哪一种方式打开，都会打开一个cmd窗口。如果关闭这个cmd窗口，打开的jmeter也会被关闭。



### 5.3 Jmeter测试计划

1.添加线程组

`右键点击“Test Plan” -> 添加 -> 线程（用户） -> 线程组`，可添加测试需要的线程组：

![image-20260521150800989](https://cdn.jsdelivr.net/gh/junting-123/my-blog-images/img/20260521150802328.png)



2.添加HTTP请求

JMeter的HTTP请求是性能测试中常用的功能，用于模拟用户向服务器发送HTTP请求并获取响应。

`右键点击线程组 -> 添加 -> 取样器 -> HTTP请求`，添加一个HTTP请求：

![image-20260521165925049](https://cdn.jsdelivr.net/gh/junting-123/my-blog-images/img/20260521165926982.png)



3.添加监听器

   **压测时一定要看结果**。在`右键点击线程组 -> 添加 -> 监听器`，添加两个最常用的：

- **聚合报告 (Aggregate Report)**：核心报告，能看吞吐量、平均响应时间、错误率等关键指标。
- **查看结果树 (View Results Tree)：**调试必备，能看每个请求的详细请求和响应数据，先用它确保脚本没问题，再跑正式压测。

![image-20260529161357016](https://cdn.jsdelivr.net/gh/junting-123/my-blog-images/img/20260529161358192.png)



4.运行测试

脚本调试好之后，点击上方绿色小箭头即可运行。JMeter 会模拟用户发请求，你可以实时在监听器里看到数据变化。



5.分析聚合报告

压测结束后，重点关注 **聚合报告** 里的这几项数据：

- **Samples**：总请求数，公式是 `线程数 × 循环次数`。
- **Average**：平均响应时间，单位是毫秒。核心接口建议保持在 500ms 或 1s 以内。
- **Error %**：**错误率**，要尽可能低（比如 < 1%），只要出错就说明系统有问题。
- **Throughput**：**吞吐量**，这是衡量系统处理能力的关键指标，代表每秒能处理的请求数，越高越好。
- **90% Line (90% 分位)**：**核心参考指标**，代表 90% 的请求响应时间都小于这个数值。相比平均值，这个指标更能体现对长尾用户的性能影响



注意：

**压测环境隔离**：压测时尽量**不要用 GUI 模式跑高并发**，JMeter 自身会很消耗性能。建议先在 GUI 上调试好脚本，然后保存脚本文件 (`.jmx`)，再通过命令行进行压测。



6.保存测试计划

File -> Save Test Plan as...，假设我们命名为xingneng.jmx，然后切换到所在目录，在命令行里这样执行：

```bash
# 切换到xingneng.jmx所在目录
cd xx

# 基本命令：-n 无界面模式，-t 指定脚本，-l 保存原始数据
jmeter -n -t xingneng.jmx -l results.jtl

# 基于 JTL 文件生成 HTML 报告
jmeter -g results.jtl -o report
```

结果会直接生成一个包含图表和详细数据的 HTML 报告，看起来更直观，也方便分享。



7.输出结果

生成的report ->html文件浏览器中打开，结果如下：

![image-20260523191856530](https://cdn.jsdelivr.net/gh/junting-123/my-blog-images/img/20260523191857904.png)



### 5.4 编写 dockerfile

为了在容器里跑 JMeter，需要写一个 dockerfile 来打包。

1. 创建 dockerfile

   在项目根目录下，创建名为 `dockerfile` 的文件，内容如下

   ```dockerfile
   FROM justb4/jmeter:5.6.3
   
   WORKDIR /opt/jmeter/scripts
   
   # 复制测试脚本
   COPY xingneng.jmx .
   
   # 默认执行测试，生成结果和HTML报告
   CMD ["-n", \
        "-t", "/opt/jmeter/scripts/xingneng.jmx", \
        "-l", "/opt/jmeter/results/result.jtl", \
        "-e", "-o", "/opt/jmeter/results/html-report"]
   ```

   同样，这一步写完 dockerfile 后，将代码 git push到 github，后续通过服务端的 jenkins 拉取代码并打包，在服务端构建镜像自动运行性能测试。



### 5.5 编写 jenkinsfile

目录结构如下：

```
jmeter-docker/
├── dockerfile
└── xingneng.jmx             
└── jenkinsfile  
└── jenkinsfiles/  
    └── send_feishu.groovy			# 子脚本(用于结果发送飞书)
    └── check_path.groovy			# 子脚本(用于github智能触发对应任务)
```



send_feishu.groovy 脚本如下：

```groovy
def call(Map params) {
    def webhook = "https://open.feishu.cn/open-apis/bot/v2/hook/d27ef06c-cdad-4eff-a7d0-46f089c661bf"
    def status = params.RESULT
    def statusText = status == 'SUCCESS' ? '✅ 通过' : '❌ 失败'
    def statusColor = status == 'SUCCESS' ? 'green' : 'red'
    def reportUrl = "${params.BUILD_URL}allure/"
    
    sh """
        curl -X POST -H 'Content-Type: application/json' \
        -d '{
            "msg_type": "interactive",
            "card": {
                "header": {
                    "title": {"content": "Jenkins 构建通知", "tag": "plain_text"},
                    "template": "${statusColor}"
                },
                "elements": [{
                    "tag": "div",
                    "text": {
                        "content": "**项目**: ${params.JOB_NAME}\\n**构建号**: #${params.BUILD_NUMBER}\\n**结果**: ${statusText}\\n**报告**: [点击查看](${reportUrl})",
                        "tag": "lark_md"
                    }
                }]
            }
        }' ${webhook}
    """
    echo "飞书通知已发送"
}

return this
```



check_path.groovy 脚本如下

```
def call(String targetPath){
    // 获取本次构建的变更文件列表
    def changeLogSets = currentBuild.changeSets
    def changedFiles = []

    // 收集所有变更的文件路径
    for (changeLogSet in changeLogSets){
        for (entry in changeLogSet.getItems()){
            for (path in entry.getAffectedPaths()){
                changedFiles.add(path)
            }
        }
    }

    // 如果没有变更记录 (手动构建、定时执行)，默认执行
    if (changedFiles.isEmpty()){
        println "⏭️ 无变更记录（手动/定时构建），默认执行"
        return true
    }

    // 检查是否包含目标路径的变更
    for (file in changedFiles){
        if (file.startsWith(targetPath)){
            println "✅ 检测到目标路径变更： ${file}"
            return true
        }
    }

    // 没有相关更新
    println "❌ 没有检测到 ${targetPath} 的变更,跳过构建"
    return false
}

    return this
```



主流水线 jenkinsfile 脚本如下：

```groovy
pipeline{
    agent any

     // 可选：添加参数，允许手动构建时跳过路径检查
    parameters {
        booleanParam(name: 'SKIP_PATH_CHECK', defaultValue: false, description: '手动构建时跳过路径检查')
    }

    stages {
        // 新增：检查变更路径的 stage
        stage('检查变更路径') {
            when {
                expression { !params.SKIP_PATH_CHECK }  // 手动构建时可跳过检查
            }
            steps {
                script {
                    // 加载 checkPath 函数
                    def check_path = load "${env.WORKSPACE}/calculatortest/jmeter_test/jenkinsfiles/check_path.groovy"
                    
                    // 检查是否是 api_test 目录的变更
                    def shouldRun = check_path('calculatortest/jmeter_test/')
                    
                    if (!shouldRun) {
                        echo "⏭️ 没有检测到 api_test 目录的变更，跳过构建"
                        currentBuild.result = 'SUCCESS'
                        // 使用 error 但标记为 ABORTED，这样 post 不会执行
                        currentBuild.description = 'SKIPPED'
                        // 添加 return，退出当前 stage 的 script 块
                        return
                    }
                }
            }
        }


        stage('拉取代码'){
            when{
                expression { currentBuild.description != 'SKIPPED' }
            }
            steps{
                retry(3){
                   git branch: 'master',
                    url: 'git@github.com:junting-123/calculator.git',
                    credentialsId: '' 
                }
            }
        }

        stage('构建镜像'){
            when{
                expression { currentBuild.description != 'SKIPPED' }
            }
            steps{
                sh '''
                    cd calculatortest/jmeter_test
                    docker build -t jmeter_test:latest .
                '''
                }
            }

        stage('运行测试并生成html数据'){
            when{
                expression { currentBuild.description != 'SKIPPED' }
            }
            steps{
                sh '''
                    cd calculatortest/jmeter_test
                    sudo rm -rf report   # 清空旧报告
                    sudo rm -f results.jtl
                    mkdir -p report

                    # 使用 Docker 容器运行 JMeter
                    docker run --rm \
                        --network host \
                        -v $(pwd):/tests \
                        jmeter_test:latest \
                        -n -t /tests/xingneng.jmx -l /tests/results.jtl -e -o /tests/report

                        sudo chown -R junting:junting report
                        sudo chown junting:junting results.jtl 2>/dev/null || true

                '''
                }
            }
        }

    post{
        always{
            script{
                if (currentBuild.description != 'SKIPPED'){
                     // 发布 HTML 报告到 Jenkins
            publishHTML target:[
                allowMissing: true,
                alwaysLinkToLastBuild: true,
                keepAll:true,
                reportDir: 'calculatortest/jmeter_test/report',
                reportFiles: 'index.html',
                reportName: 'Jmeter 性能测试报告'
            ]

            // 归档 JTL 文件
            archiveArtifacts artifacts: 'calculatortest/jmeter_test/results.jtl',
                                allowEmptyArchive: true

            // 加载并发送飞书通知
             def notify = load "${env.WORKSPACE}/calculatortest/jmeter_test/jenkinsfiles/send_feishu.groovy"
                notify([
                    JOB_NAME: env.JOB_NAME,
                    BUILD_NUMBER: env.BUILD_NUMBER,
                    BUILD_URL: env.BUILD_URL,
                    RESULT: currentBuild.result ?: 'SUCCESS',
                    REPORT_URL: "${env.BUILD_URL}JMeter_20性能测试报告/"  // 指向性能报告
                ])
                echo "📱 飞书通知已发送"
                }
                else{
                    echo "⏭️ 构建被跳过，不生成报告和通知"
                } 
            }
        }

        success {
            echo '✅ 性能测试通过！'
        }

        failure {
            echo '❌ 性能测试失败！'
        }

        aborted{
            echo '⏭️ 流水线被跳过（路径不匹配）'
        }
    }
}
```





## 六、CI/CD 持续部署工作流（Jenkins）

### 6.1 CI/CD总体流程

> 在初步学习了接口测试和性能测试之后，我们尝试着将其形成全自动工作流。Jenkins是一个流行的开源自动化服务器，可以帮助我们实现这样的流程。
>

CI：持续集成（Continuous Integration）

CD：持续交付（Continuous Delivery）或持续部署（Continuous Deployment）

流程图如下：

![image-20260523192746546](https://cdn.jsdelivr.net/gh/junting-123/my-blog-images/img/20260523192747534.png)

| 阶段              | 步骤     | 说明                         | 对应你的项目        |
| :---------------- | :------- | :--------------------------- | :------------------ |
| **① 代码提交**    | Step1-2  | 开发者推送代码，触发 Jenkins | `git push`          |
| **② CI 持续集成** | Step3-8  | 自动构建 + 测试验证          | pytest + JMeter     |
| **③ CD 持续部署** | Step9-15 | 自动发布 + 上线              | Docker 部署到虚拟机 |
| **④ 完成**        | End      | 服务正式运行                 | 计算器服务可用      |

Linux上Jenkins下载参考文章：[Ubuntu 24.04上安装 Jenkins-CSDN博客](https://blog.csdn.net/manmanmanzai/article/details/161432745?spm=1001.2014.3001.5501)



### 6.2 Jenkins拉取Git项目

接下来我们学习如何将本地代码推送至github，然后用Jenkins拉取Git项目，并打包成Docker镜像。

如下是CD部署流程：

![image-20260523193756563](https://cdn.jsdelivr.net/gh/junting-123/my-blog-images/img/20260523193757581.png)



#### 6.2.1 本地代码推送至 github

cd到推送到 github 的文件夹，终端执行：

```bash
# 初始化文件夹，用于将一个普通文件夹转换为 Git 仓库
git init

# 添加远程仓库
git remote add origin<远程仓库地址>

# 检查当前仓库
git remote -v

# 添加文件到暂存区
git add .

# 添加文件到本地仓库
git commit -m "first commit"

# 查看当前分支
git branch

# 提交代码至远程仓库
git push -u origin main
```



#### 6.2.2 在 github 上生成Token

> 无论你是用 Jenkins、命令行、还是任何第三方工具，都无法再用"账号+密码"的方式拉取代码。**Token 是替代密码的官方方案**，它更适合 CI/CD 场景，权限可控、可随时撤销

①**进入设置页面**：登录 GitHub，点击右上角头像 -> **Settings**。

②**找到入口**：在页面左侧最下方，点击 **Developer settings** -> **Personal access tokens** -> **Tokens (classic)** 。

③**生成新 Token**：点击 **Generate new token** (classic)。

④**设置权限 (最关键的一步)**：

- **Note**：给 Token 起个名字，比如 `Jenkins-K8s`。
- **Expiration**：建议设置过期时间（如 90 天或 1 年），这是为了保障安全，到期后需要重新生成 
- **Scopes (权限范围)**：**务必勾选 `repo`** 这一整项 。如果你之后想让 Jenkins 自动帮你创建 Webhook，还需要额外勾选 **`admin:repo_hook`** 。

⑤**复制 Token**：点击页面底部的绿色按钮生成。**注意：这个 Token 只会显示这一次，一定要立刻复制并保存好！** 



#### 6.2.4 在 Jenkins 中配置凭证

1. **进入凭据管理页面**：
   - 在 Jenkins 首页，点击左侧的 **Manage Jenkins** > **Credentials** > 点击 **System** 列下的 **Global credentials (unrestricted)** 链接。
   - 在打开的页面中，点击左侧的 **Add Credentials**。
2. **填写 Token 信息**：
   - **Kind (类型)**：虽然也可以选择 `Secret text`，但对于 Git 操作，选择 **`Username with password`** 是最稳定且不易出错的。
   - **Username (用户名)**：这里直接填写你的 **GitHub 用户名**。
   - **Password (密码)**：**不要填你的 GitHub 登录密码**。在这里 **粘贴** 你刚才从 GitHub 复制好的 **Token**。
   - **ID**：这是 Jenkins 内部使用的唯一标识，可以起一个有意义的名字，比如 `github-token`。
   - **Description (描述)**：简单的说明，比如 `GitHub Personal Access Token`。
3. **保存**：点击 **Create** 或 **Save** 按钮，完成配置。



#### 6.2.5 Jenkinsfile 拉取github

1.配置 Jenkins 插件

- Git Plugin：用于从Git拉取代码
- Docker Pipeline：用于Docker操作

进入Jenkins首页，选择“管理Jenkins -> 管理插件 -> 可用插件”，搜索并安装所需插件即可。



2.创建 Jenkins Job

- 在 Jenkins 首页选择“新。建Item”
- 输入任务名称“拉取git”并选择“流水线”，然后点击“确定”
- 在任务配置页面进行如下配置

在“流水线”部分添加如下代码：

```groovy
pipeline {
    agent any
    stages {
        stage('拉取代码') {
            steps {
                git branch: 'master',
                    url: 'https://github.com/用户名/仓库.git',
                    credentialsId: 'github-token'
            }
        }
    }
}
```

拉取成功后会出现在 jenkins_home\workspace 里面，那如何进一步实现Jenkins在你每次`git push`到GitHub时自动拉取代码并构建，最推荐的方法是配置**GitHub Webhook**，这样GitHub会在代码变更时主动通知Jenkins，实现真正的实时自动化。



#### 6.2.6 github Webhook触发Jenkins 自动拉取

1.在Jenkins中配置任务

①登录Jenkins，进入你的任务（Job）配置页面。

②在构建触发器部分，勾选 **GitHub hook trigger for GITScm polling**和**Poll SCM**。设置每小时检查一次更新

```
H/60 * * * *
```

③保存配置。

第二种方式是配置 GitHub Webhook，使得github每更新一次就会触发自动拉取。



2.在GitHub仓库中设置Webhook

①打开你的GitHub仓库，进入 **Settings** -> **Webhooks** -> **Add webhook**。

②在表单中填写：

- **Payload URL**: 填写 `http://<你的Jenkins地址>/github-webhook/`。例如：`http://192.168.1.100:8080/github-webhook/`。
- **Content type**: 选择 `application/json`。
- **Events**: 选择 `Just the push event`，这样只有在代码推送时才会触发。
- **SSL verification**：选Disable (not recommended)，允许使用HTTP协议。

③点击 **Add webhook** 保存。

*注意：<你的Jenkins地址>必须用公网地址，可使用内网穿透技术部署成公网ip。

内网穿透技术详见：[0成本在 Linux 上实现 NATAPP 内网穿透技术-CSDN博客](https://blog.csdn.net/manmanmanzai/article/details/161625604?spm=1001.2014.3001.5502)



这样，之后每次你用`git push`命令推送代码，GitHub就会自动通知Jenkins，Jenkins随即开始拉取代码并执行你的Pipeline。你可以在GitHub仓库的Webhook页面看到每次触发的历史记录和响应状态（应为绿色勾选，表示成功）。



### 6.3 接口自动化测试

#### 6.3.1 pipeline配置

> 由于 github 上已上传 dockerfile 和 jenkinsfile，所以我们采取从源代码管理（SCM）中获取流水线脚本方式，来做接口自动化测试。

1.创建任务

点击左上角的：新建 Item——>输入任务名称：`api_test`（或你喜欢的名字）——>选择 Pipeline——>点击 确定

 

2.配置 pipeline

| 配置项         | 选择/填写的内容                 |
| :------------- | :------------------------------ |
| **Definition** | 选择 `Pipeline script from SCM` |
| **SCM**        | 选择 `Git`                      |

展开后填写：

| 配置项                                     | 填写内容                                           |
| :----------------------------------------- | :------------------------------------------------- |
| **Repository URL**                         | `https://github.com/你的用户名/your-repo.git`      |
| **Credentials**                            | 点击添加 → 选择 GitHub 用户名/密码 或 Access Token |
| **Branches to build**                      | `*/main`（或 `*/master`）                          |
| **Script Path**                            | 填写 Jenkinsfile 相对于仓库根目录的完整路径        |
| **GitHub hook trigger for GITScm polling** | 构建触发器                                         |



3.验证是否成功部署：

```
# 查看镜像
docker images

# 查看容器是否在运行
docker ps | grep xx
```



#### 6.3.2 Allure 测试报告配置

1.Jenkins 中安装 Allure 插件

在Jenkins中生成和展示报告，首先需要安装对应的插件

- 进入 Jenkins：**Manage Jenkins → Plugins → Available plugins**

- 搜索 **"Allure"**，勾选并安装

- 重启 Jenkins（可勾选"安装完成后重启"）

  

2.配置 Allure 命令行工具

为了让Jenkins能生成报告，我们需要告诉它Allure命令行工具在哪里

- 进入：**Manage Jenkins → Tools → Allure Commandline installations**

- 点击 **Add Allure Commandline**

- Name 填 `Allure`，勾选 **Install automatically**

- 版本选择最新的稳定版（如 2.27.0 以上）

运行完成后，可以在 Jenkins 界面查看 Allure 报告：

![image-20260529143150039](https://cdn.jsdelivr.net/gh/junting-123/my-blog-images/img/20260601145124999.png)



#### 6.3.3 测试报告发送至飞书

1.飞书机器人配置

1. **打开飞书群聊** → 点击右上角 `···` → **设置** → **群机器人** 
2. 点击 **添加机器人** → 选择 **自定义机器人**
3. 给机器人起名，如 `CI测试报告`
4. **安全设置**（建议开启，否则URL泄露会有安全隐患）：
   - 勾选 **签名校验** → 复制保存生成的 `SEC_xxxx` 密钥
5. 点击完成，复制保存 **Webhook地址**（格式：`https://open.feishu.cn/open-apis/bot/v2/hook/xxx`）



2.验证连通性

```bash
curl -X POST -H 'Content-Type: application/json' \
-d '{
    "msg_type": "text",
    "content": {"text": "测试消息"}
}' https://open.feishu.cn/open-apis/bot/v2/hook/
你的Webhook地址
```

返回 `{"StatusCode":0}` 表示成功，你的飞书会收到“测试消息”的字样。



3.在 Jenkins 中配置 Webhook

①在你的 Jenkins 项目中点击 **Configure**

②找到 **"Feishu Configuration"** 或 **"飞书配置"** 区域

③将 **"Custom Webhook"** 改为 **"Yes"**

④在下方 **"Webhook URL"** 输入框粘贴复制的地址

⑤根据需要设置：

- **"Notify Success"**：是否通知成功（选是）
- **"Notify Failure"**：是否通知失败（选是）
- **"Notify Unstable"**：是否通知不稳定（选是）

⑥点击 **Save**



4.确认Allure报告可访问

你已经通过Allure插件成功生成了报告，报告地址格式为：

```txt
http://你的JenkinsIP:8080/job/你的任务名/构建编号/allure/
```



5.确认飞书机器人自动发送

![image-20260601144618577](https://cdn.jsdelivr.net/gh/junting-123/my-blog-images/img/20260601144619738.png)



### 6.4 性能自动化测试

#### 6.4.1 Html 测试报告配置

性能自动化测试 pipeline 配置与接口测试一致。Html 报告插件配置如下：

1. 在 Jenkins 首页，点击 **Manage Jenkins** > **Manage Plugins**。
2. 进入 **Available** 选项卡。
3. 在搜索框中输入 **HTML Publisher**。
4. 勾选搜索结果中的 **"HTML Publisher plugin"**。
5. 点击 **Download now and install after restart**（或 **Install without restart**）。
6. 安装完成后，**重启 Jenkins**（可以在管理页面通过 `http://你的Jenkins地址/safeRestart` 安全重启）。



#### 6.4.2 针对 html 图表不显示问题解决



1.首先确认是否是数据的原因

```bash
# 切换到保存数据的位置
cd /home/junting/.jenkins/workspace/jmeter_test/calculatortest/jmeter_test

# 查看 JTL 文件的前几行
head -5 results.jtl

# 查看 JTL 文件的后几行
tail -5 results.jtl

# 查看所有响应码
cat results.jtl | grep -v timeStamp | cut -d',' -f4 | sort | uniq -c
```



2.使用 Script Console 临时生效图表

> 如果数据没有问题，Meter 生成的 HTML 报告在 Jenkins 中图表不显示，可能原因是 Jenkins 默认的 Content Security Policy (CSP，内容安全策略) 为了安全，阻止了报告中 CSS 和 JavaScript 等动态内容的加载。

通过 Jenkins 的脚本控制台临时修改 CSP 配置。临时生效，Jenkins 服务重启后配置会失效，需要重新执行，适用于开发环境测试，或紧急验证报告时适用。步骤如下：

1. **打开 `Script Console`**：
   登录 Jenkins 后，点击 **Manage Jenkins** → **Script Console**。
2. **执行脚本**：
   在命令行输入框中粘贴以下命令，然后点击 **Run** 按钮。

```groovy
System.setProperty("hudson.model.DirectoryBrowserSupport.CSP", "")
```

3. **验证效果**：
   执行完该命令后，重新进入你的 `jmeter_test` 任务，再次查看 **JMeter 性能测试报告**，图表应该就能正常显示了。



3.修改 Jenkins 启动配置永久生效图表

通过修改 CSP 配置文件来实现 jenkins 图表显示永久生效。

```
# 切换到 jenkins 文件夹下
cd /home/xx/xx/jenkins

# 禁用csp
cat > start_jenkins.sh << 'EOF'
#!/bin/bash
JENKINS_WAR="/home/xx/jenkins.war"  # 改成实际路径
cd $(dirname $JENKINS_WAR)
nohup java -Dhudson.model.DirectoryBrowserSupport.CSP= -jar $JENKINS_WAR > jenkins.log 2>&1 &
echo "Jenkins started with CSP disabled"
EOF

# 启动
./start_jenkins.sh
```

访问 Jenkins，浏览器打开 http://192.168.171.128:8080



## 七、CI/CD自动测试工作流



### 7.1 自动测试架构设计

通过第六章我们已经完成了持续部署工作流：



① CD(持续部署)：

github中web每更新一次 -> 自动触发jenkins拉取git仓库 -> 构建docker镜像 -> 部署到服务端-> 部署结果发送飞书



② CI(持续集成)：

github中tetst每更新一次 -> 自动触发对应测试拉取git仓库 ->自动测试  -> 测试结果发送飞书



## 附件：

### 常用docker镜像管理命令速查

| 命令                               | 作用                |
| :--------------------------------- | :------------------ |
| `docker build -t`                  | 构建镜像            |
| `docker run`                       | 运行容器            |
| `docker ps`                        | 查看容器            |
| `docker stop`                      | 停止容器            |
| `docker images`                    | 查看本地所有镜像    |
| `docker image ls`                  | 同上（等价）        |
| `docker load -i 文件名.tar`        | 从 tar 文件加载镜像 |
| `docker save -o 文件名.tar 镜像名` | 保存镜像为 tar 文件 |
| `docker rmi 镜像名`                | 删除镜像            |
| `docker inspect 镜像名`            | 查看镜像详细信息    |

```
先删除容器，再删除镜像

# 1. 停止并删除所有容器
docker rm -f $(docker ps -aq)

# 2. 删除所有镜像
docker rmi -f $(docker images -q)
```



























三、jenkins

1.下载jenkins

[https://get.jenkins.io/war-stab](https://link.zhihu.com/?target=https%3A//get.jenkins.io/war-stable/)

2.选择对应版本的jenkins.war

3.

我们可以看到Jenkins的包是一个 war 包，它有两种启动方式：一种是将 war 包方到 tomcat 中启动，一种是直接启动，下面我演示直接启动的方式

启动：

```powershell
cd E:\app\jenkins  #转到jenkins.war所在文件夹下
java -jar jenkins.war --httpPort=8080
```

![image-20260519163204577](https://cdn.jsdelivr.net/gh/junting-123/my-blog-images/img/20260519163308706.png)

访问：[http://localhost:8080](https://link.zhihu.com/?target=http%3A//localhost%3A8080/)



![image-20260519163546193](https://cdn.jsdelivr.net/gh/junting-123/my-blog-images/img/20260519163547289.png)看上图让我们输入密码，它说密码已经保存到服务器上地址已经给你标到了下面，我们去这个地址下的initialAdminPassword文件复制密码（或者在日志输出也会打印）

输入密码后安装：

![image-20260519163825506](https://cdn.jsdelivr.net/gh/junting-123/my-blog-images/img/20260519163828881.png)