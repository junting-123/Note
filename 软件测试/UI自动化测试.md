# 基于 Playwright UI 自动化测试



## 1、录制自动化测试

1、安装

```
pip install pytest-playwright
```

只需要在代码里指定 `channel="msedge"`，Playwright 就会自动找到你电脑上的 Edge 并启动它。



2、录制

> 录制的方式进行使用

通过浏览器手动操作转换为相应代码。

```
playwright codegen https://pc-toutiao-python.itheima.net/#/login --channel=msedge
```

会自动打开两个窗口：

- 浏览器窗口：显示网页内容（自动跳转到被测网站）
- Inspector 窗口：显示和控制 Playwright 对浏览器的操作



选择 Playwrght ：

- 速度快

- 定位方便

- 很好和 AI 结合

  

```
from playwright.sync_api import Playwright, sync_playwright, expect


def run(playwright: Playwright) -> None:
    browser = playwright.chromium.launch(channel="msedge", headless=False)
    context = browser.new_context()
    page = context.new_page()
    
    page.goto("https://pc-toutiao-python.itheima.net/#/login")
    page.get_by_role("button", name="登录").click()
    page.get_by_role("menuitem", name="首页").click()
    page.get_by_text("内容管理").click()
    page.get_by_role("menuitem", name="发布文章").click()
    page.get_by_role("textbox", name="文章名称").click()
    page.get_by_role("textbox", name="文章名称").fill("你好")
    page.locator("iframe[title=\"在编辑区按ALT-F9打开菜单，按ALT-F10打开工具栏，按ALT-0查看帮助\"]").content_frame.locator("html").click()
    page.locator("iframe[title=\"在编辑区按ALT-F9打开菜单，按ALT-F10打开工具栏，按ALT-0查看帮助\"]").content_frame.locator("#tinymce").fill("你好")
    page.get_by_role("button", name="发表").click()

    # ---------------------
    context.close()
    browser.close()
```



**包含N个自动化动作，和一个断言**，是一个测试用例



元素抓取器：
![image-20260705142356241](https://cdn.jsdelivr.net/gh/junting-123/my-blog-images/img/20260705142438517.png)

通过 **Pick locator** ，点一下页面上的按钮/输入框，自动帮你生成定位它的代码。



3、技巧

默认情况下，Playwright 使用**面向用户**的定位方式：用户能够看到什么，就定位什么

```
page.get_by_role("textbox",name="登录").click()
```



有的网页，会让用户看到的内容不断变化，从而导致元素定位失败

这个时候，就要使用另一种方式：**面向网页**定位方式

```
page.locator("#chat-textarea").click()
```



## 2、Playwright 常用语句

### 2.1 页面导航

决定被测页面是哪一个

- goto : 直接跳转

- reload : 刷新

- go_back ：后退

- go_forward : 前进

  

### 2.2 页面内容

- titile : 网页标题（HTML中的title）
- url ：当前的网页
- content：当前网页内容（HTML）



### 2.3 页面动作

- set_viewport_size ：设置窗口大小
- screenshot : 截图
- evaluate : 执行 JS 代码



## 3、POM 设计模式

### 3.1 POM原理介绍



**什么是POM设计？**

页面对象模型（Page Object Model，简称 POM），是业内公认最佳的一种 UI 自动化设计模式。专门用于实现 UI 自动化测试框架的核心底层设计模式。

它将页面与代码进行映射，核心思想是：将每一个网页或页面组件，封装成一个独立的类（Class），这个类里包含该页面上的所有元素定位和操作动作。



**与传统测试区别？**

传统测试行为我们关注的是每一个操作步骤。POM的设计思路上，不再纠结每一个操作步骤，而是将整个业务流程进行分解。

基于主流程分解其中的参与页面，将主流程基于不同的页面分解为不同子流程。通过子流程的拼接最终实现一个主流程的执行。

基于不同的页面的子流程的拼接，我们可以成功执行不同的核心主业务流程。从而实现到对系统的完整业务流程的覆盖。



用户从登录到修改个人信息的业务流程测试：

1. 访问 url ，进入登录页
2. 输入账号密码，点击登录按钮
3. 点击个人信息页，点击修改按钮
4. 输入修改数据
5. 点击保存按钮
6. 校验修改是否成功



基于POM设计模式实现：

1. 访问登录页，执行登录流程
2. 访问个人信息页，执行修改信息流程
3. 校验最终结果是否成功



### 3.2 POM设计思路与实现

整体所有的自动化框架都需要满足代码与数据分离，逻辑代码与测试代码分离。

POM 核心结构，主体分为四层：

1. 基类：类似于关键字驱动层。是自动化测试底层。主要用于为页面对象提供操作行为。
2. 页面对象类：是 POM 的设计核心层级。用于封装不同的页面，页面包含有核心元素，核心业务。

3. 测试用例层：测试代码。用于管理和实现测试执行。
4. 测试数据层：用于管理和存放测试过程中所需要的数据。



### 3.3 POM设计代码

目录：

```
AUTOTEST/
├── pages/                             # 页面
│   ├── orangehrm_login_page.py        # 登录页面类
│   ├── orangehrm_home_page.py         # 首页页面类
├── test_data/ 
│   ├── login_data.json                # 测试用例
├── utils/ 
│   ├── data_loader.py                 # 数据驱动模块
├── test_cases/ 
├── test_login_orangehrm               # 测试脚本逻辑
```



pages页面：

```
# orangehrm_login_page.py

from playwright.sync_api import Page

class LoginPage:

    def __init__(self,page:Page):
        self.page = page
        self.username_input = page.get_by_role("textbox",name="Username")
        self.password_input = page.get_by_role("textbox",name="Password")
        self.login_button = page.get_by_role("button",name="Login")
    
    def enter_username(self,username:str):
        self.username_input.fill(username)

    def enter_password(self,password:str):
        self.password_input.fill(password)

    def click_login(self):
        self.login_button.click()

    def login(self,username:str,password:str):
        self.enter_username(username)
        self.enter_password(password)
        self.click_login()
```



```
# orangehrm_home_page.py

from playwright.sync_api import Page,expect

class HomePage:

    def __init__(self,page:Page):
        self.page = page
        self.upgrade_button = page.get_by_role("button",name="Upgrade")
        self.performance_link = page.get_by_role("link",name="Performance")
        self.dashboard_link = page.get_by_role("link",name="Dashboard")

    def is_upgrade_button_visible(self):
        # 用于验证某个条件是否为真
        expect(self.upgrade_button).to_be_visible()
    
    def click_performance(self):
        self.performance_link.click()

    def click_dashboard(self):
        self.dashboard_link.click()
```



test_data：

```
# login_data.json

[
  {
    "case_name": "正确用户名和密码",
    "username": "Admin",
    "password": "admin123",
    "expected": "success"
  },
  {
    "case_name": "错误密码",
    "username": "Admin",
    "password": "wrongpassword",
    "expected": "failure"
  },
  {
    "case_name": "错误用户名",
    "username": "wronguser",
    "password": "admin123",
    "expected": "failure"
  },
  {
    "case_name": "空用户名和密码",
    "username": "",
    "password": "",
    "expected": "failure"
  }
]
```



utiles：

```
# data_loader.py

# 数据驱动测试的辅助模块，从外部的JSON文件中加载测试数据，让测试用例和数据分离
import json
import os

class DataLoader:
    def __init__(self):
        # 在初始化时计算并保存项目根目录
        current_dir = os.path.dirname(os.path.abspath(__file__))
        self.project_root = os.path.dirname(current_dir)

    def load_json_data(self, filename: str):
        json_path = os.path.join(self.project_root, "test_data", filename)
        with open(json_path, "r", encoding="utf-8") as f:
            return json.load(f)

    def load_login_data(self):
        return self.load_json_data("login_data.json")
```



test_cases：

```
# test_login_orangehrm.py

import sys
import os

# 将项目根目录添加到 Python 搜索路径（必须在所有 import 之前）
sys.path.insert(0, os.path.dirname(os.path.dirname(os.path.abspath(__file__))))

import pytest
from playwright.sync_api import Page, expect
from pages.orangehrm_login_page import LoginPage
from pages.orangehrm_home_page import HomePage
from utils.data_loader import DataLoader

# pytest 的参数化装饰器。一条测试代码，自动覆盖多组测试数据，从而实现数据驱动测试
@pytest.mark.parametrize(
        # 参数名（在测试函数中接收数据）
        "login_data",
        # 返回测试数据列表（从 JSON 加载）
        DataLoader().load_login_data(),
        # 测试用例的显示名称（用 case_name 字段命名）
        ids=lambda x: x["case_name"]
        )

# 测试逻辑
def test_login(page: Page, login_data: dict) -> None:
    # 页面对象实例化
    login_page = LoginPage(page)
    home_page = HomePage(page)

    # 1. 打开登录页面
    page.goto(
        "https://opensource-demo.orangehrmlive.com/web/index.php/auth/login",
        timeout=60000
    )

    # 2. 输入用户名和密码（从 JSON 读取）
    login_page.enter_username(login_data["username"])
    login_page.enter_password(login_data["password"])
    login_page.click_login()

    # 3. 根据预期结果进行断言
    if login_data["expected"] == "success":
        # 登录成功：验证升级按钮可见，并执行后续操作
        home_page.is_upgrade_button_visible()
        # home_page.click_performance()
        # home_page.click_dashboard()
        print(f"✅ 通过：{login_data['case_name']}")
    else:
        # 登录失败：验证错误提示可见
        error_message = page.get_by_text("Required")
        expect(error_message).to_be_visible()
        print(f"✅ 通过：{login_data['case_name']}（预期失败，验证通过）")
```



### 3.4 运行代码

```
pytest ./test_cases/login_orangehrm1.py --headed --browser-channel=msedge --alluredir=./allure-results
```



--headed参数作用：

当你使用 `pytest` 命令运行测试时，pytest-playwright 插件会：

1. 在后台启动一个浏览器实例。
2. 执行你的测试代码（如 `page.goto` 和 `login_page.enter_username`）。
3. 测试结束后，自动关闭浏览器。

这个后台浏览器实例是可见的，但它不会自动弹到前台，因此你会看到一个空白的浏览器窗口。在运行测试时，添加 `--headed` 参数，这会让浏览器窗口正常显示在前台



--browser-channel=msedge：

指定已有的edge加载



--alluredir=./allure-results：
显示 Allure 报告



