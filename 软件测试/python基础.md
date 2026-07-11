# Python 基础知识



## 1、数据结构

1、字符串（str）

eg:"nihaoa!" (一串字符，表示文本内容)



2、整数（int）

eg: 6；-34



3、浮点数（float）

eg:6.0；3.87



4、布尔类型（bool）

True；False



5、空值类型（None）



```
# 返回数据类型
print(type("6.8"))
# 预期返回值<class 'str'>

# 求长度
print(len("你好吗"))

# 索引获取单个字符
s = "你好吗"
print(s[0])
```



6、列表

```
shopping_list = ["笔","本子"]
# 列表加东西（对象.方法名）
shopping_list.append("电脑")
# 列表里删除东西
shopping_list.remove("本子")
print(shopping_list)

# 列表
num = [1,55,3,5,6]
# 打印列表最大值
print(max(num))
# 打印列表最小值
print(min(num))
# 打印排序好的列表
print(sorted(num))
```



7、字典

字典用于储存键值对应关系

键  ： 值

key：value

```
contacts = {"小李":"美丽，大方",
		  "小西":"帅气，聪明"}
# 添加到字典
contacts["小五"] = "可爱，灵敏"

# 查询功能
query = input("请输入你想要了解的人：")
if query in contacts:
	print("你查询的" + query + "如下：")	
	print(contacts[query])
else:
	print("你查询的" + query + "不存在！")
	
```



## 2、Print 输出

print（“xx”）可以直接输出字符串

```
# 直接输出字符串
print("nihai")

# 字符串中有"\"转义符，则后面跟着的会打印出来
print("He said \"Let\'s go\"")

# 换行
print("Hello!\nHi")

# 三引号换字符串
print("""你好你是谁？
      我是小白，你呢？
      我是小黑，很高兴认识你！
      我也是，有机会一起打球啊。""")
```



## 3、变量赋值

通过变量名获取值

```
my_iphone = "12345678910"

# 输出带空格
print("打我电话"my_iphone)

# 输出无空格
print("打我电话" + my_iphone) 
```



变量名约定俗成命名法：
1、字母全部小写

2、不同单词用下划线分隔



注释：ctrl+/



## 4、Input 获取输入

input 获取用户输入

```
user_name = str(input("请输入你的名字："))
print("你好，{user_name}！欢迎使用我们的程序。")

# 或者
user_name = str(input("请输入你的名字："))
print("你好，"+ user_name + "欢迎使用我们的程序。")
```



## 5、If 条件语句

判断结果是否为bool值，根据true or false，进行执行

```
if [条件]：
	[执行语句]
else：
	[执行语句]
```



嵌套条件语句：

```
if [条件1]:
	if [条件2]：
		[执行语句A]
	else:
		[执行语句B]
else:
	[执行语句C]
```



多个条件：

```
if [条件1]：
	[语句A]
elif [条件2]：
	[语句B]
elif [条件3]：
	[语句C]
else:
	[语句D]
```



逻辑运算：

与：and

或：or

非：not

优先级：not > and > or



## 6、for 循环语句

循环语句

```
temperature_list = {"1号":35,"2号":37,"3号":38,"4号":36,"5号":39}
for staff_id,temperature in temperature_list.items():
	if temperature >= 38:
		print(f"{staff_id}发烧了，烧到了{temperature}度")	
	else:
		print("全员体温正常")
```



注意

temperature_list = ["1号":35,"2号":37,"3号":38,"4号":36,"5号":39]

temperature_list.keys()              # 所有键

temperature_list.values()	      # 所有值

temperature_list.items()			# 所有键值对



range用法，从1 加到100

```
sum = 0

for i in range(1,101):
    sum = sum + i
print("The sum of numbers from 1 to 100 is:", sum)
```

| 关键字       | 什么时候用                                 |
| :----------- | :----------------------------------------- |
| 不加任何控制 | 需要完整遍历所有元素，每个都处理           |
| `break`      | 需要提前终止循环（找到目标后不再继续）     |
| `continue`   | 需要跳过某些特殊情况（不符合条件的不处理） |



## 7、while 循环

循环次数未知

```
while [条件A]：
	[行动B]
```



计算用户输入后的平均值

```
user_num = input("请输入数字，我会帮你求平均值(q为中止程序):")
total = 0
count = 0

while user_num !='q':
    num = float(user_num)
    total += num 
    count += 1
    user_num = input("请输入数字，我会帮你求平均值(q为中止程序):")

if count == 0:
    print("没有输入有效数字，无法计算平均值。")
else:
    result = total / count
    print("平均值为:", result)
```



## 8、封装函数

如果想频繁调用某个函数值，则将其封装

```
# 封装扇形面积函数
def calculator_sector_1():
    central_angle_1 = 160
    radius_1 = 30
    sector_area_1 = central_angle_1 / 360 * 3.14 * radius_1 ** 2
    print(f"此扇形面积为：{sector_area_1}")
    
# 调用函数    
calculator_sector_1()
```



可以通过参数，让函数入参值自定义性更高

```
# 入参值可自定义
def calculator_sector(central_angle,radius):
    sector_area = central_angle / 360 * 3.14 * radius ** 2
    print(f"此扇形面积为：{sector_area}")

calculator_sector(160,30)
```



## 9、return 返回值

函数没写 return 默认返回 None，函数只会打印值，而不会返回赋值

```
def fun():
    a = 3
    print(a)

a = fun()
print(a)

# 预期输出3;None
```

函数写 return 添加返回值

```
def fun():
    a = 3
    print(a)
    return a

a = fun()
print(a)

# 预期输出3;3
```

 

封装 BMI 计算函数

```
def calculator_BMI(height,weight):
    bmi_value = weight / (height ** 2)
    if bmi_value <= 18.5:
        category = "偏瘦"
    elif 18.5 < bmi_value <= 25:
        category = "正常"
    elif 25 < bmi_value <= 30:
        category = "偏胖"
    else:
        category = "肥胖"
    print(f"你的BMI分类为:{category}")
    return bmi_value

height = float(input("请输入身高（米）："))
weight = float(input("请输入体重（公斤）："))

calculator_BMI(height, weight)
print(calculator_BMI(height,weight))
```



## 10、Python 引入内置函数

程序员不要重复造轮子，Python 内置了很多函数，可以直接引用。

以下三种方式引用内置函数：

```
# import 语句
import statistics
print(statistics.median([1,34,26]))
print(statistics.mean([18,35,26]))

# from import 语句
from statistics import median,mean
print(median([1,34,26]))
print(mean([18,35,26]))

# from import * 语句
from statistics import *
print(median([1,34,26]))
print(mean([18,35,26]))
```



## 11、面向对象编程

### 11.1 概念

**1、面向过程编程：**
把要完成某个具体任务的代码，拆分成一个个步骤，按照顺序依次完成。

缺点：
随着程序长度和逻辑复杂度的增加，代码的清晰度可能由此降低。



**2、面向对象编程**

它以对象为核心，并不会聚焦于第一步，而是模拟真实世界，先考虑各个对象有什么性质、能做什么事情。

**属性类**

eg：杯子属性：有材质，透明度，颜色、重量等

然后用类创建对象，类是创建对象的模板，对象是类的实例。

**方法类**

eg：杯子行为逻辑：移动、放大、变色、旋转



方法——放在类里面的函数

属性——放在类里面的变量



相当于把事务先分解到对象身上，描述各个对象的作用，然后才是他们之间的交互。



### 11.2 封装

封装表示写类的人，将内部实现细节隐藏起来，使用类的人只能通过外部接口访问和使用。

其中接口可以被理解为提供使用的方法。



比如洗衣机功能，你不需要知道内部运转逻辑，它提供洗衣，烘干衣服按钮，你只需要知道按钮的使用方式。



优点：
封装能减少我们对不必要细节的精力投入。



### 11.3 继承

面向对象编程允许创建有层次的类，就像现实中的儿子继承爸爸，爸爸继承爷爷。类也可以有子类和父类，来表示从属关系。



比如小学生和初中生都有学号、年级、姓名等相似属性，多次定义重复冗余。我们就可以创建一个学生的父类，然后让小学生和初中生去继承这个类。

这样父类的那些属性、方法都可以被继承。不需要反复定义。

<img src="https://cdn.jsdelivr.net/gh/junting-123/my-blog-images/img/20260704134817349.png" alt="image-20260704134816254" style="zoom:50%;" />

```
class 小学生（学生）：
	...
	
class 初中生（学生）：
	...
```



### 11.4 多态

同样的接口，因为对象具体类的不同，而有不同表现。

比如小学生和初中生都要写作业（方法一样），但是他们的作业难度不同，所以作业不能直接定义在父类里面，而是要分别定义在子类里面。

引用的时候，无需判断，统一调用同一名称的方法。

```
class 小学生（学生）：
	def 写作业（self）：
		# 写简单作业
	...
	
class 初中生（学生）：
	def 写作业（self）：
		# 写复杂作业
	...
	
弟弟 = 小学生（）
哥哥 = 初中生（）
我家孩子 = [弟弟，哥哥]
for 孩子 in 我家孩子：
	孩子.写作业               # 自动调用类里面不同方法，不用判断
```



## 12、类及构造函数

1、命名规则

变量名：下划线命名法（user_name ; list_count）

类名：首字母大写（UserCount ; CustomerOrder）

```
# 没有构造函数
class CuteCat:
    name = "Vioda"  # 所有猫都叫 Vioda
    age = 3
    breed = "英短"

cat1 = CuteCat()
cat2 = CuteCat()
# cat1 和 cat2 完全相同，不能定制！
```

```
# 使用构造函数后，每个实例独立，更具灵活性
class CuteCat:
    def __init__(self, name, age, breed):
        self.name = name
        self.age = age
        self.breed = breed

# 可以创建不同的猫
cat1 = CuteCat("Vioda", 3, "英短")
cat2 = CuteCat("Mimi", 5, "布偶")
cat3 = CuteCat("Tom", 2, "橘猫")

print(cat1.name, cat1.age)  # Vioda 3
print(cat2.name, cat2.age)  # Mimi 5
print(cat3.name, cat3.age)  # Tom 2
```

取名：

"构造函数"强调的不是赋值动作，而是创建完整对象。有构造函数，同一个类可以生成无数个不同个体。



**核心区别**：

- **类属性**：所有实例共享同一个值
- **实例属性**（通过 `self` 设置）：每个实例拥有自己的值

如果没有构造函数，你就必须手动一个个赋值。



定义一个学生类：

```
# 定义一个学生类
# 要求：
# 1.属性包含学生姓名、学号、以及语数英三科的成绩
# 2.能够设置学生某科目的成绩
# 3.能够打印出学生的所有科目成绩

class Student:
	def __init__(self,name,student_id):     # 双下划线（前后各两个）
		self.name = name
		self.student_id = student_id
		self.grades = {"语文":0,"数学":0,"英语":0}
		
	def set_grade(self,course,grade):
		if course in self.grades:
			self.grades[course] = grade
			
	def print_grades(self):
		print(f"学生{self.name}(学号:{self.student_id})的成绩为：")
		for course in self.grades:
			print(f"{course}: {self.grades[course]}分")

a = Student("a","1001")
b = Student("b","1002")
print(a.name)

b.set_grade("数学",95)
print(b.grades)
```



继承

```
# 类：人力系统
# 员工分两类：全职员工 FullTimeEmployee,兼职员工 PartTimeEmployee
# 员工都有“姓名 name”、“工号 id”属性
# 都具备“打印信息 print_info”(打印姓名、工号)方法
# 全职有“月薪 monthly_salary”属性
# 兼职有“日薪 daily_salary”属性、“每月工作天数 work_days”的属性
# 全职和兼职都有“计算月薪 calculator_monthly_pay”的方法，但计算过程不一样

class Employee:
    def __init__(self,name,id):
        self.name = name
        self.id = id

    def print_info(self):
        print(f"员工名字: {self.name}, 工号: {self.id}")

class FullTimeEmployee(Employee):
    def __init__(self,name,id,monthly_salary):
        super().__init__(name,id)
        self.monthly_salary = monthly_salary

    def calculator_mothly_pay(self):
        return self.monthly_salary

class PartTimeEmployee(Employee):
    def __init__(self,name,id,daily_salary,work_days):
        super().__init__(name,id)
        self.daily_salary = daily_salary
        self.work_days = work_days
        
    def calculator_monthly_pay(self):
        return self.daily_salary * self.work_days
    
zhangsan = FullTimeEmployee("张三","1001",8000)
lisi = PartTimeEmployee("李四","1002",200,15)
zhangsan.print_info()
lisi.print_info()

print(f"{zhangsan.name}的月薪为: {zhangsan.calculator_mothly_pay()}")
print(f"{lisi.name}的月薪为: {lisi.calculator_monthly_pay()}")
```



## 13、读写文件

```
# 打开文件
f = open("/usr/demo/data.txt","r"，encoding="utf-8" )   # r 只读，w 只写
print(f.read())  # 会读全部的文件内容，并打印
print(f.read())  # 会读空字符串，并打印

# 读取字节
f = open("/usr/demo/data.txt","r"，encoding="utf-8" )   # r 只读，w 只写
print(f.read(10))  # 会读1-10个字节的文件内容
print(f.read(10))  # 会读11-20个字节的文件内容

# 读取一行
f = open("/usr/demo/data.txt","r"，encoding="utf-8" )   # r 只读，w 只写
print(f.readline())  # 会读1行文件内容，并打印
print(f.readline())  # 会读1行文件内容，并打印

# readline 方法
f = open("/usr/demo/data.txt","r"，encoding="utf-8" ) 
line = f.readline
while line != "":            # 判断当前行是否为空
	print(line)				 # 不为空则打印当前行
	line = f.readline()		 # 读取下一行
	
	
# readlines 方法
f = open("/usr/demo/data.txt","r"，encoding="utf-8" ) 
lines = f.readlines()       # 把每行内容存储到列表里
for line in lines:          # 遍历每行内容
	print(line)             # 打印当前行
	
	
f.close()   # 关闭文件，释放资源
```

![image-20260704153642531](https://cdn.jsdelivr.net/gh/junting-123/my-blog-images/img/20260704153643422.png)



```
f = open("./data.txt","r",encoding="utf-8")
content = f.read()
print(content)
f.close()

# 或是写成如下
with open("./data.txt","r",encoding="utf-8") as f:
    content = f.read()
    print(content)
```



文件打开形式
w : 只写

r：只读

a：附件内容

```
# data.txt：
你好
我是小李
你叫什么名字
```



```
# 在一个新的名字为"data.txt"的文件里，写下如下内容（w模式）：
# 我叫小何
# 很高兴认识你
# 我们做朋友吧

with open("./data.txt","w",encoding="utf-8") as f:
    f.write("我叫小何,\n很高兴认识你\n我们做朋友吧")

# 在一个"data.txt"的文件里，添加如下内容（a模式）：
with open("./data.txt","a",encoding="utf-8") as f:
    f.write("\n我叫小何,\n很高兴认识你\n我们做朋友吧")
```



## 14、单元测试

**1、自带 unittest 测试框架**

开发代码：my_caculator.py

```
def my_add(x,y):
	return x+y
```



测试代码：test_my_caculator.py

```
# unittest自带的、标准化的测试框架
import unittest                   
from my_caculator import my_add    # 引入函数

class TestMyAdd(unittest.TestCase): # 继承父类
# 定义不同的测试用例
	def test_positive_with_positive(self):  
		self.assertEqual(my_add(3,2),5)
		
	def test_negative_with_negative(self):
		self.assertEqual(my_add(-3,-5),-8)
```



测试测试名字必须以 test_ 开头，因为 unittest 这个库会自动搜寻 test_ 开头的方法，并且只把 test_ 开头的当成测试用例。



终端测试：

```
python -m unittest
```



**2、pytest 框架**

测试代码：test_my_caculator.py

```
# pytest 是第三方开发的、更现代、更强大的测试框架

import pytest                           # 引入 pytest
from my_calculator import my_add        # 引入被测试的函数

# pytest 不需要类，直接用函数就可以
# 测试用例

def test_positive_with_positive():
    assert my_add(3, 2) == 5             # 直接用 assert，不用 self.assertEqual

def test_negative_with_negative():
    assert my_add(-3, -5) == -8
```



终端：

```
pytest
```

