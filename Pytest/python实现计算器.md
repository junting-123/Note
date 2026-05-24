

# 前言

**基于客户端-服务端架构的计算器长稳自动化测试系统**

- **项目背景**：开发了一款支持加减乘除运算的计算器服务，采用客户端-服务端（C/S）架构。将服务端及其依赖环境通过**Docker **进行容器化打包，并部署至虚拟机中，实现了运行环境的一致性与快速交付**。**
- **自动化测试**：以 **pytest** 为核心构建本地自动化测试框架，对服务端接口进行功能正确性验证，并引入 **JMeter** 脚本实现服务端的压力与稳定性测试。
- **工程化与可视化**：通过 **Jenkins** 搭建 CI 流水线，实现测试任务的 24x7 小时无人值守执行；集成 **Allure** 报告框架，生成包含历史趋势与性能指标的可视化报告，便于质量分析与问题追溯。
  - **技术栈总结**：`pytest` + `JMeter` + `Jenkins` + `Docker` + `Allure`



1. **Jenkins 执行测试** → 跑 pytest + JMeter
2. **pytest 生成 Allure 原始数据**（`--alluredir`）
3. **Python 脚本用 Matplotlib 画性能趋势图**，保存为 PNG
4. **将 PNG 图片附加到 Allure 报告中**（使用 `allure.attach.file()`）
5. **Allure 生成最终 HTML 报告**，里面既有用例结果，又有性能图表
6. **Jenkins 展示 Allure 报告**



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





**在本地写 Dockerfile 打包服务端代码 → 导出镜像 → 传输到虚拟机 → 在虚拟机里运行容器**



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



### 1.2 MobaXterm远程连接

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

# 暴露应用程序需要的端口（Flask 默认 8080）
EXPOSE 8080

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
docker run -d -p 8080:8080 calculator-web

# 验证容器是否运行成功 
docker ps
# 应该看到 calculator-web 容器状态为 Up

# 测试
curl "http://localhost:8080/calc?a=10&b=5&op=add"
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



## 四、接口测试

项目会进行接口测试和jmeter性能测试，代码测试仓库包含：

项目根目录/

```
├── test_api.py              # pytest 接口测试脚本
├── test_data.json           # pytest 测试用例数据
├── jmeter/
│   └── calculator_test.jmx  # JMeter 压力测试计划
└── Jenkinsfile              # Jenkins Pipeline 脚本
```



#### 4.1 test_api测试用例

根据测试用例的常用方法，编写测试用例，先用表格记录：

![image-20260521001039449](https://cdn.jsdelivr.net/gh/junting-123/my-blog-images/img/20260521001041566.png)

#### 4.2 test_data.json数据写入

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

#### 4.3 test_api.py测试脚本

代码如下：

```python
import json
import pytest
import requests

# 从 JSON 文件加载测试数据
data = json.load(open("test_data.json", encoding='utf-8'))

# 服务端的地址，后续测试会用这个地址拼接完整请求的URL
BASE_URL = "http://192.168.0.104:8080"

# pytest 提供的功能，用于批量执行测试
@pytest.mark.parametrize("op, cases", data.items())
# 函数名必须以 test_ 开头，pytest 才能识别;op操作符eg：“add",cases对应测试用例表
def test_calculator(op, cases):
    for case in cases:
        resp = requests.get(f"{BASE_URL}/calc", params={"a": case["a"], "b": case["b"], "op": op})
        assert resp.json()["result"] == case["expected"], f'失败：{case["desc"]}'
```

终端运行：

```bash
pytest test_api.py -v 
```

运行成功就可以查看结果了！



## 五、性能测试

#### 5.1 Jmeter安装

Jmter下载路径：[Jmeter 官网下载](http://jmeter.apache.org/download_jmeter.cgi)

![image-20260521143335803](https://cdn.jsdelivr.net/gh/junting-123/my-blog-images/img/20260521143340532.png)



#### 5.2 Jmeter环境变量配置

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



#### 5.3 Jmeter测试计划

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

File -> Save Test Plan as...，然后在命令行里这样执行：

```bash
# 基本命令：-n 无界面模式，-t 指定脚本，-l 保存原始数据
jmeter -n -t E:\code\calculator\calculatortest\jmeter\calculator_test.jmx -l E:\code\calculator\calculatortest\jmeter\results.jtl

# 生成报告：-e 生成报告，-o 指定报告目录（目录必须为空）
jmeter -n -t E:\code\calculator\calculatortest\jmeter\calculator_test.jmx -l E:\code\calculator\calculatortest\jmeter\results.jtl -e -o E:\code\calculator\calculatortest\jmeter\report
```

结果会直接生成一个包含图表和详细数据的 HTML 报告，看起来更直观，也方便分享。



7.输出结果

生成的report ->html文件浏览器中打开，结果如下：

![image-20260523191856530](https://cdn.jsdelivr.net/gh/junting-123/my-blog-images/img/20260523191857904.png)



## 六、CI/CD全自动工作流（Jenkins）

### 6.1 CI/CD总体流程

在初步学习了接口测试和性能测试之后，我们尝试着将其形成全自动工作流。

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



### 6.2 Jenkins拉取最新代码

如下是CD部署流程：

![image-20260523193756563](https://cdn.jsdelivr.net/gh/junting-123/my-blog-images/img/20260523193757581.png)



1.本地代码推送至github

cd到推送到github的文件夹，终端执行：

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



2.Jenkins拉取

Jenkins下载：[Download and deploy](https://www.jenkins.io/download/)

在存放 `jenkins.war` 文件的目录下，启动一个终端：

```bash
java -jar jenkins.war
```













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