# 前言

**项目名称**：
**基于客户端-服务端架构的计算器长稳自动化测试系统**

**项目描述**：
开发了一个带加减乘除功能的计算器服务，部署在虚拟机中作为服务端；本地编写 pytest 自动化测试框架，实现 24 小时不间断的接口功能正确性验证，并集成失败重试、随机数据生成、服务端异常恢复测试。

**个人职责与技术实现**：

- **服务端开发**：使用 Python/Flask 编写计算器 API，支持 `add/sub/mul/div`，处理除零、非法输入等异常；通过 Docker 打包并运行在 Ubuntu 虚拟机中。
- **客户端测试框架**：
  - 基于 `pytest + requests` 设计分层用例（参数化测试、边界值测试、随机输入测试）。
  - 编写 `conftest.py` 实现**会话级服务探活**与**自动重连**（服务重启后测试不中断）。
  - 使用 `pytest-repeat` 或自定义 `while + 日志轮转` 实现 24 小时持续运行，每轮测试结果实时记录。
- **稳定性验证**：
  - 模拟服务端中途重启（`docker restart`），验证客户端等待/重试机制。
  - 长时间运行下内存/CPU 监控（使用 `psutil` 记录资源曲线）。
- **结果输出**：生成带时间戳的 JSON 日志与 HTML 测试报告，24 小时内执行 **约 8.6 万次测试请求**，缺陷发现率 0%（或发现服务端连接池未释放问题并修复）。



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

2. 测试连同

```
ip addr show
```

显示虚拟机ip地址。



## 二、本地构建计算器程序打包成Docker镜像

> Docker 镜像的核心特性就是**可移植性**——镜像包含了运行程序所需的一切（代码、运行环境、依赖库、配置），只要目标机器安装了 Docker，就可以直接运行这个镜像，不管是在本地、虚拟机还是云服务器上。

### 2.1 本地编写calculator.py

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



### 2.2 在本地构建镜像

> 镜像是一种轻量级的、可执行的独立软件包。用来打包软件运行环境和基于运行环境的开发软件，它包含运行某个软件所需要的内容，包括代码、运行时、库、环境变量和配置文件。

1.安装Docker Desktop



假设你的项目目录是 `D:\calculator`，里面有：

- `calculator.py`
- `Dockerfile`

打开 PowerShell 或 CMD，进入项目目录：





2.3 用docker build 构建镜像

> **读取 Dockerfile 中的"说明书"，自动把"运行环境 + 代码"打包成一个镜像。**



2.4 用docker run 启动容器，验证服务正常