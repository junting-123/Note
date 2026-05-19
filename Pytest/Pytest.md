# Pytest测试框架
`提示：这里可以添加本文要记录的大概内容：`

例如：随着人工智能的不断发展，机器学习这门技术也越来越重要，很多人都开启了学习机器学习，本文就介绍了机器学习的基础内容。

---

`提示：以下是本篇文章正文内容，下面案例可供参考`

# 一、引言

测试框架：抽象出来的工具集合，提供大量组件、工具、功能

- 用例发现
- 用例管理
- 环境管理
- 用例执行
- 测试报告



大部分语言都有测试框架：

- java：junit，testng

- ###### python：unittest，**pytest**

  

# 二、启动与规则

## 1. 安装
在终端命令行安装：

```c
pip install pytest #安装
pip install pytest -U #升级到最新版
pip show pytest #终端查看安装目录
```

## 2. 启动
在python中编写一个测试用例test_simple.py：

```c
def test_addition(): #测试用例：验证基本算术
    assert 1 + 1 == 2  #assert是pytest的断言，如果表达式为True则测试通过
    
 def test_multiplication():
    a = 2 
    b = 3   
    assert a*b == 4
```
1. 终端命令行启动

```
pytest test_simple.py #或直接输入Pytest运行全部用例
```

2. 代码启动

同文件夹下创建一个新的文件main.py：

```
import pytest
pytest.main()
```

---

## 3. 结果输出

启动后运行结果如下：

```
========================================= test session starts =====================================
platform win32 -- Python 3.12.9, pytest-9.0.2, pluggy-1.6.0
rootdir: E:\ruance\code
collected 2 items                                                                                                             

test_simple.py .F                                                                                [100%] 

========================================== FAILURES ===============================================
_______________________________________ test_multiplication _______________________________________ 

    def test_multiplication():
        a = 2
        b = 3
>       assert a*b == 4
E       assert (2 * 3) == 4

test_simple.py:7: AssertionError
====================================short test summary info ======================================== 
FAILED test_simple.py::test_multiplication - assert (2 * 3) == 4
====================================1 failed, 1 passed in 0.15s ====================================
```

代表分别为：

- 执行环境：版本、根目录、用例数量
- 执行过程：文件名称、**用例结果**、执行进度
- 失败详情：用例内容、断言提示
- 整体摘要：结果情况、结果数量、花费时间

用例结果缩写如下

| **缩写** | **单词** | **含义**                   |
| -------- | -------- | -------------------------- |
| .        | passed   | 通过                       |
| F        | failed   | 失败（用例执行时报错）     |
| E        | error    | 出错（fixture执行时报错）  |
| s        | skipped  | 跳过                       |
| X        | xpassed  | 预期外的通过（不符合预期） |
| x        | xfailed  | 预期内的失败（符合预期）   |

## 4. 用例规则

### 4.1 用例发现规则

测试框架在识别、加载用例的过程，称为：**用例发现**

pytest的用例发现步骤：

1. 遍历所有目录，例外 venv 或 . 开头的目录
2. 打开python文件，test_ 开头或者  _test 结尾的文件
3. 遍历所有的Test开头类
4. 收集所有的 test_ 开头的**函数或方法**

### 4.2 用例内容规则

Pytest 对用例的要求：

1. 可调用的（函数、方法、类、对象）
2. 名字 test_ 开头
3. 没有参数（参数有另外的含义）
4. 没有返回值（默认为None）

## 5. 配置框架

配置可以改变 pytest 默认的规则

1. 命令参数
2. ini配置文件

所有的配置方式，可以获取

```
pytest -h 
```

- 哪些配置
- 分别什么方式
  - -开头：参数
  - 小写字母开头：ini配置
  - 大写字母开头：环境遍历
- 配置文件：pytest.ini

常用参数

- -v：增加详细程度
- -s：在用例中正常的使用**输入输出**
- -x：快速退出，当遇到失败的用例停止执行
- -m：`用例筛选`

## 6. 标记mark

> 标记可以让用例与众不同，进而可以被筛选的

### 6.1 用户自定义标记

用户自定义标记只能实现用例筛选

步骤:

1. 先注册 (ini文件)

```
[pytest] #注册mark

markers = 
    api: 接口测试
    web: UI测试
    ut: 单元测试
    login: 登录相关
    pay: 支付相关
```

2. 再标记

```
import pytest

def add(a,b):
    return a+b

class TestAdd:  # 方法（类）
    
    @pytest.mark.api # 标记mark
    def test_add1(self): # 函数
        res = add(1,3)
        assert res == 4

    @pytest.mark.ui
    def test_add2(self):
        res = add("1","3")
        assert res == "13"

    @pytest.mark.pay
    def test_add3(self):
        res = add([1],[3])
        assert res == [1,3] 
    
def test_input():
    a = input("number")
```

3. 后筛选

```
pytest -m web
```

### 6.2 框架内置标记

用户自定义标记为用例增加特殊执行效果



和用户自定义标记区别：

1. 不需注册，可直接使用
2. 不仅可筛选，还可增加特殊效果
3. 不同的标记，增加不同的特殊效果
   - skip： 无条件跳过
   - skipif：有条件跳过
   -  xfail：预期失败
   - parametrize:  参数化
   - usefixtures:  使用fixture



数据驱动测试 = 参数化测试 + 数据文件

根据数据文件的内容，动态决定用例的数量、内容



## 7. 数据驱动测试参数

数据文件，驱动用例执行数量、内容，创建data.csv

```
a,b,c
1,1,2
2,3,5
3,3,6
4,4,7
```

用例：

```
    @pytest.mark.ddt
    @pytest.mark.parametrize(
        "a,b,c",
        read_csv("data.csv")
    )
    def test_ddt(self,a,b,c):
        res = add(int(a),int(b))
        assert res == int(c)
 
```

## 8. 夹具fixture

夹具：在用例**执行之前**、**执行之后**，自动运行代码

场景：

* 之前：加密参数 / 之后：解密结果
* 之前：启动浏览器 / 之后：关闭浏览器
* 之前：注册、登录账号 / 之后：删除账号

### 8.1 创建fixture

```
@pytest.fixture
def f():
    
    # 前置操作
    yield # 开始执行用例
    # 后置操作
```

1. 创建函数
2. 添加装饰器
3. 添加yield关键字

### 8.2 使用fixture

1. 在用例参数列表中，加入fixture名字即可
2. 给**用例**加上“usefixture”标记

```
def test_1(f):
    pass

@pytest.mark.usefixtures("f")
def test_2():
    pass
```

3. 全部自动执行

```
@pytest.fixture(autouse=True)
```

### 8.3 高级用法

1. 自动使用
2. 依赖使用
   * linux：使用linux进行编译
   * git：使用git进行版本控制
   * fixture：使用fixture进行前后置自动操作
3. 返回内容：接口自动化封装：接口关联
4. 范围共享
   * 默认范围：function
   * 全局范围：session
     * 使用**conftest.py**

## 9. 插件管理

pytest插件生态是pytest的优势之处



插件类型分为两类：

* 不需要安装：内置插件
* 需要安装：第三方插件



插件的启用管理：

* 启用：**-p** abc
* 禁用：**-p no: **abc



插件使用方式：

* 参数
* 配置文件
* fixture
* mark

## 10. 常用第三方插件

pytest有1400+插件：https://docs.pytest.org/en/stable/reference/plugin_list.html

### 10.1 pytest-html

用途：生成HTML测试报告

安装：

```
pip install pytest-html
```

使用：

```
pytest --html=report.html --self-contained-html
```

写入pytest.ini配置文件中：

```
--html=report.html --self-contained-html
```

### 10.2 pytest-xdist

用途：分布式执行

安装：

```
pip install pytest-xdist
```

使用：

```
pytest -n N
```

> 只有在任务本身耗时长，超出调用成本很多的时候，才有意义

> 分布式执行，有并发问题：资源竞争、乱序

### 10.3 pytest-rerunfailures

用途：用例失败之后，重新执行

安装：

```
pip install pytest-rerunfailures
```

使用：

```
--reruns 5 --reruns-delay 1
```



### 10.4 pytest-result-log

用途：把用例的执行结果记录到**日志文件**中

安装：

```
pip install pytest-result-log
```

使用，写入pytest.ini配置文件中：

```
log_file = ./logs/pytest.log
log_file_level = INFO
log_file_format = %(levelname)-8s %(asctime)s [%(name)s:%(lineno)s] :%(message)s
log_file_date_format = %Y-%m-%d %H:%M:%S

# 记录用例执行结果
result_log_enable = 1
# 记录用例分割线
result_log_seperator = 1
# 分割线等级
result_log_level_seperate = warnig
# 异常信息等级
result_log_level_verbose = info
```

* 注意：新结果会覆盖之前日志

## 11. 企业级测试报告allure

allure是一个测试报告框架

安装：

```
pip install allure-pytest
# 先安装 Scoop（如果没有）
Set-ExecutionPolicy RemoteSigned -Scope CurrentUser
irm get.scoop.sh | iex
# 安装 allure与所需java环境
scoop install openjdk
scoop install allure
```

配置（pytest.ini）：

```
--alluredir=./allure-results
```

生成并查看报告：

```
# 生成报告
allure generate --clean report 
# 查看报告
allure serve ./allure-results
```

也可以main函数：

```
import pytest
import os

pytest.main()

# 执行命令
os.system("allure generate --clean report && allure serve ./allure-results")
```

* 这里os功能类似于shell，它让你能在 Python 代码中**执行操作系统相关的命令和操作**。

allure支持对用例进行**分组和关联**（敏捷开发术语）

```
@allure.epic        #史诗 项目
@allure.feature		#主题 模块
@allure.story		#故事 功能
@allure.title		#标题 用例
```

使用相同装饰器的用例，自动在同一组

## 12. Web自动化测试

pytest仅进行用例管理，不会控制浏览器，需要借助新的工具：selenium

​	1.只了解selenium

​	2.搜索关于selenium的pytest插件

