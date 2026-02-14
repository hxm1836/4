# 第1-2周详细学习计划：Python进阶 + Git全流程

> 📅 学习周期：第1-2周（共14天）  
> ⏰ 总时长：60小时（30小时/周）  
> 🎯 目标：掌握Python进阶特性，熟练使用Git进行版本控制

---

## 时间分配总览

```
算法线（Python进阶）：36小时
  - Week 1: 18小时（OOP + 装饰器基础）
  - Week 2: 18小时（生成器 + 异常 + 上下文管理器）

工程线（Git全流程）：24小时
  - Week 1: 12小时（基础操作 + 分支管理）
  - Week 2: 12小时（远程协作 + 冲突解决 + 最佳实践）
```

---

# 第一周（Week 1）

## 时间分配

| 时间段 | 内容 | 时长 |
|--------|------|------|
| 周一-周五晚上 | Python OOP | 10小时 |
| 周一-周五晚上 | Git基础 | 6小时 |
| 周六 | Python实战 + Git实践 | 8小时 |
| 周日 | 项目 + 复盘 | 6小时 |
| **总计** | | **30小时** |

---

## Day 1（周一）：类与对象基础 + Git初始化

### 算法线：类与对象基础（2小时）

#### 1.1 为什么需要面向对象？（20分钟）

**对比示例**：
```python
# 不用OOP：数据和操作分离
student_data = {"name": "张三", "score": 85}

def get_grade(student):
    if student["score"] >= 90: return "A"
    elif student["score"] >= 80: return "B"
    else: return "C"

# 问题：
# 1. 数据验证困难：student["score"] = -100 不会报错
# 2. 容易出错：student["scor"] = 95 创建了新key
# 3. 难以维护：修改结构需要改很多地方

# 用OOP：封装数据和行为
class Student:
    def __init__(self, name, score):
        self.name = name
        self.score = score
    
    def get_grade(self):
        if self.score >= 90: return "A"
        elif self.score >= 80: return "B"
        else: return "C"
    
    @property
    def score(self):
        return self._score
    
    @score.setter
    def score(self, value):
        if not 0 <= value <= 100:
            raise ValueError("分数必须在0-100之间")
        self._score = value
```

**OOP三大核心**：
1. **封装**：数据和操作绑定，隐藏实现细节
2. **继承**：代码复用，建立类层次
3. **多态**：同一接口，不同实现

#### 1.2 类的定义与`__init__`（40分钟）

```python
class BankAccount:
    """银行账户类"""
    
    def __init__(self, account_number, holder_name, balance=0):
        """初始化方法
        
        Args:
            account_number: 账户号码
            holder_name: 持有人姓名
            balance: 初始余额（默认0）
        """
        if balance < 0:
            raise ValueError("初始余额不能为负")
        
        self.account_number = account_number
        self.holder_name = holder_name
        self._balance = balance  # 用_表示"私有"
    
    def deposit(self, amount):
        """存款"""
        if amount <= 0:
            raise ValueError("金额必须为正")
        self._balance += amount
        return self._balance
    
    def withdraw(self, amount):
        """取款"""
        if amount > self._balance:
            raise ValueError(f"余额不足，当前余额{self._balance}")
        self._balance -= amount
        return self._balance
    
    def get_balance(self):
        """查询余额"""
        return self._balance

# 使用
account = BankAccount("6228481234567890", "张三", 1000)
account.deposit(500)
account.withdraw(200)
print(account.get_balance())  # 1300
```

**关键点**：
- `self`：代表实例本身，必须是第一个参数
- `__init__`：初始化方法，创建对象时自动调用
- 命名约定：`_variable`表示"受保护"，`__variable`表示"私有"

#### 1.3 实例属性 vs 类属性（30分钟）

```python
class Student:
    # 类属性：所有实例共享
    school = "清华大学"
    student_count = 0
    
    def __init__(self, name):
        # 实例属性：每个实例独有
        self.name = name
        Student.student_count += 1

stu1 = Student("张三")
stu2 = Student("李四")

print(Student.student_count)  # 2
print(stu1.school)  # 清华大学（访问类属性）
print(stu1.name)    # 张三（访问实例属性）

# ⚠️ 陷阱：通过实例修改类属性
class Demo:
    data = []  # 类属性是列表

obj1 = Demo()
obj2 = Demo()

obj1.data.append(1)  # 修改了类属性！
print(obj2.data)     # [1] - 所有实例都受影响

obj1.data = [2]      # 创建了实例属性
print(obj2.data)     # [1] - obj2仍然访问类属性
print(Demo.data)     # [1] - 类属性未变
```

#### 1.4 练习（30分钟）

创建`Rectangle`类：
```python
class Rectangle:
    """矩形类
    
    要求：
    1. 属性：width, height
    2. 方法：area(), perimeter(), is_square()
    3. 类方法：create_square(side)
    4. 静态方法：is_valid_dimension(value)
    5. 在__init__中验证宽高为正数
    """
    pass

# 测试
rect = Rectangle(4, 5)
assert rect.area() == 20
assert rect.perimeter() == 18
assert rect.is_square() == False

square = Rectangle.create_square(5)
assert square.is_square() == True
```

### 工程线：Git初始化（1小时）

#### 配置Git（15分钟）

```bash
# 设置用户信息
git config --global user.name "你的姓名"
git config --global user.email "your.email@example.com"

# 查看配置
git config --list

# 设置默认分支名为main
git config --global init.defaultBranch main

# 设置默认编辑器
git config --global core.editor "vim"  # 或 "code" (VS Code)
```

#### 创建第一个仓库（45分钟）

```bash
# 1. 创建学习目录
mkdir ai-learning
cd ai-learning

# 2. 初始化Git仓库
git init

# 3. 创建项目结构
mkdir -p week01-python-basics
cd week01-python-basics

# 4. 创建README
cat > README.md << 'EOF'
# Week 1: Python基础

## 学习内容
- 面向对象编程
- 类与对象
- 继承与多态

## 代码示例
见各个.py文件
EOF

# 5. 创建第一个Python文件
cat > student.py << 'EOF'
class Student:
    def __init__(self, name, age):
        self.name = name
        self.age = age
    
    def introduce(self):
        return f"我是{self.name}，今年{self.age}岁"

if __name__ == "__main__":
    stu = Student("张三", 20)
    print(stu.introduce())
EOF

# 6. 查看状态
git status

# 7. 添加到暂存区
git add .

# 8. 提交
git commit -m "feat: 初始化项目，添加Student类"

# 9. 查看历史
git log --oneline
```

**Git三区模型**：
```
工作区 (Working Directory)
   ↓ git add
暂存区 (Staging Area)
   ↓ git commit
仓库 (Repository)
```

---

## Day 2（周二）：继承与多态 + Git基础操作

### 算法线：继承与多态（2小时）

#### 2.1 为什么需要继承？（20分钟）

```python
# 不用继承：代码重复
class Teacher:
    def __init__(self, name, age, subject):
        self.name = name
        self.age = age
        self.subject = subject
    
    def introduce(self):
        return f"我是{self.name}老师"

class Student:
    def __init__(self, name, age, grade):
        self.name = name  # 重复
        self.age = age    # 重复
        self.grade = grade
    
    def introduce(self):  # 重复
        return f"我是{self.name}同学"

# 用继承：提取公共部分
class Person:
    def __init__(self, name, age):
        self.name = name
        self.age = age
    
    def introduce(self):
        return f"我是{self.name}"

class Teacher(Person):
    def __init__(self, name, age, subject):
        super().__init__(name, age)  # 调用父类构造
        self.subject = subject
    
    def introduce(self):  # 重写父类方法
        return f"我是{self.name}老师，教{self.subject}"

class Student(Person):
    def __init__(self, name, age, grade):
        super().__init__(name, age)
        self.grade = grade
    
    def introduce(self):
        return f"我是{self.name}同学，{self.grade}年级"
```

#### 2.2 super()详解（30分钟）

```python
class Animal:
    def __init__(self, name):
        print(f"Animal.__init__: {name}")
        self.name = name
    
    def speak(self):
        return "发出声音"

class Dog(Animal):
    def __init__(self, name, breed):
        # 方式1：使用super()（推荐）
        super().__init__(name)
        
        # 方式2：直接调用父类（不推荐）
        # Animal.__init__(self, name)
        
        print(f"Dog.__init__: {breed}")
        self.breed = breed
    
    def speak(self):
        # 调用父类方法
        parent_speak = super().speak()
        return f"{parent_speak}：汪汪！"

dog = Dog("旺财", "金毛")
print(dog.speak())
```

**super()的优势**：
1. 支持多重继承（后面会讲）
2. 代码更灵活，父类改名不影响子类
3. 遵循MRO（方法解析顺序）

#### 2.3 多态（30分钟）

```python
# 多态：不同的类，相同的接口
class Animal:
    def speak(self):
        pass

class Dog(Animal):
    def speak(self):
        return "汪汪"

class Cat(Animal):
    def speak(self):
        return "喵喵"

class Cow(Animal):
    def speak(self):
        return "哞哞"

# 多态的威力：无需知道具体类型
def animal_sound(animal: Animal):
    """接收任何Animal子类"""
    print(animal.speak())

animals = [Dog(), Cat(), Cow()]
for animal in animals:
    animal_sound(animal)  # 输出不同的声音

# 鸭子类型：如果它叫起来像鸭子，那它就是鸭子
class Robot:
    def speak(self):
        return "哔哔"

animal_sound(Robot())  # 也能工作！
```

#### 2.4 练习：设计类层次（40分钟）

设计一个图形类层次：
```python
"""
Shape (抽象基类)
  ├── Rectangle
  │     └── Square
  └── Circle

要求：
1. Shape有抽象方法 area(), perimeter()
2. 所有子类必须实现这些方法
3. Square继承自Rectangle
4. 写测试代码验证多态
"""

from abc import ABC, abstractmethod
import math

class Shape(ABC):
    @abstractmethod
    def area(self):
        pass
    
    @abstractmethod
    def perimeter(self):
        pass

# 你的代码...
```

### 工程线：Git基础操作（1小时）

#### 常用命令实践（60分钟）

```bash
# 1. 修改文件
echo "# 新增内容" >> README.md

# 2. 查看修改
git status        # 查看哪些文件被修改
git diff          # 查看具体改了什么

# 3. 暂存部分修改
git add README.md

# 4. 查看暂存的修改
git diff --staged

# 5. 提交
git commit -m "docs: 更新README"

# 6. 查看历史
git log                    # 完整历史
git log --oneline          # 简洁显示
git log --graph --oneline  # 图形化显示

# 7. 查看某次提交的内容
git show <commit-id>

# 8. 撤销操作
# 撤销工作区修改
git restore <file>

# 撤销暂存
git restore --staged <file>

# 修改最后一次提交信息
git commit --amend -m "新的提交信息"

# 9. 忽略文件
cat > .gitignore << 'EOF'
# Python
__pycache__/
*.pyc
*.pyo
.pytest_cache/

# 编辑器
.vscode/
.idea/
*.swp

# 环境
venv/
.env

# 数据
*.csv
*.xlsx
EOF

git add .gitignore
git commit -m "chore: 添加.gitignore"
```

**今日Git练习**：
创建3个以上的commit，练习：
- 修改文件
- 查看diff
- 撤销修改
- 修改commit信息

---

## Day 3（周三）：魔术方法 + Git分支管理

### 算法线：魔术方法（2小时）

#### 3.1 什么是魔术方法？（20分钟）

魔术方法（Magic Methods）：双下划线开头和结尾的方法，让对象支持Python的内置操作。

```python
class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y
    
    # 让对象可以打印
    def __str__(self):
        return f"Point({self.x}, {self.y})"
    
    # 让对象可以用repr()
    def __repr__(self):
        return f"Point(x={self.x}, y={self.y})"
    
    # 让对象可以相加
    def __add__(self, other):
        return Point(self.x + other.x, self.y + other.y)
    
    # 让对象可以比较
    def __eq__(self, other):
        return self.x == other.x and self.y == other.y

p1 = Point(1, 2)
p2 = Point(3, 4)

print(p1)           # Point(1, 2) - 调用__str__
print(p1 + p2)      # Point(4, 6) - 调用__add__
print(p1 == p2)     # False - 调用__eq__
```

#### 3.2 常用魔术方法（90分钟）

**1. 对象创建与销毁**
```python
class Resource:
    def __init__(self, name):
        self.name = name
        print(f"创建资源：{name}")
    
    def __del__(self):
        print(f"销毁资源：{self.name}")

r = Resource("数据库连接")
# 程序结束时自动调用__del__
```

**2. 字符串表示**
```python
class Book:
    def __init__(self, title, author):
        self.title = title
        self.author = author
    
    def __str__(self):
        """用户友好的字符串（print时调用）"""
        return f"《{self.title}》- {self.author}"
    
    def __repr__(self):
        """开发者友好的字符串（调试时用）"""
        return f"Book(title='{self.title}', author='{self.author}')"

book = Book("Python编程", "张三")
print(book)      # 《Python编程》- 张三
print(repr(book))  # Book(title='Python编程', author='张三')
```

**3. 运算符重载**
```python
class Vector:
    def __init__(self, x, y):
        self.x = x
        self.y = y
    
    def __add__(self, other):
        return Vector(self.x + other.x, self.y + other.y)
    
    def __sub__(self, other):
        return Vector(self.x - other.x, self.y - other.y)
    
    def __mul__(self, scalar):
        return Vector(self.x * scalar, self.y * scalar)
    
    def __str__(self):
        return f"Vector({self.x}, {self.y})"

v1 = Vector(1, 2)
v2 = Vector(3, 4)
print(v1 + v2)  # Vector(4, 6)
print(v1 * 2)   # Vector(2, 4)
```

**4. 比较运算符**
```python
class Student:
    def __init__(self, name, score):
        self.name = name
        self.score = score
    
    def __eq__(self, other):
        return self.score == other.score
    
    def __lt__(self, other):
        return self.score < other.score
    
    def __le__(self, other):
        return self.score <= other.score

students = [
    Student("张三", 85),
    Student("李四", 90),
    Student("王五", 78)
]

# 可以排序了！
students.sort()
for s in students:
    print(f"{s.name}: {s.score}")
```

**5. 容器类型魔术方法**
```python
class MyList:
    def __init__(self):
        self._items = []
    
    def __len__(self):
        """支持len()"""
        return len(self._items)
    
    def __getitem__(self, index):
        """支持索引访问"""
        return self._items[index]
    
    def __setitem__(self, index, value):
        """支持索引赋值"""
        self._items[index] = value
    
    def __contains__(self, item):
        """支持in运算符"""
        return item in self._items
    
    def append(self, item):
        self._items.append(item)

my_list = MyList()
my_list.append(1)
my_list.append(2)
print(len(my_list))     # 2
print(my_list[0])       # 1
print(1 in my_list)     # True
```

#### 3.3 练习：实现分数类（30分钟）

```python
class Fraction:
    """分数类
    
    要求：
    1. 支持加减乘除：+, -, *, /
    2. 支持比较：==, <, >
    3. 自动约分
    4. 字符串表示：Fraction(3, 4) 显示为 "3/4"
    """
    pass

# 测试
f1 = Fraction(1, 2)
f2 = Fraction(1, 3)
print(f1 + f2)  # 5/6
print(f1 > f2)  # True
```

### 工程线：Git分支管理（1小时）

#### 3.1 为什么需要分支？（10分钟）

```
不用分支的问题：
main ───●───●───●───●───●───●
       开发A 开发B 开发A 开发B...
       
问题：
1. 功能混在一起
2. 无法独立测试
3. 一个bug影响所有人

用分支：
main    ───●───────────●────────●───
            ↓           ↑        ↑
feature-a   └──●──●──●──┘        │
                                 │
feature-b   ─────●──●──●──●──●───┘

优势：
1. 独立开发
2. 互不影响
3. 随时切换
```

#### 3.2 分支操作（50分钟）

```bash
# 1. 查看分支
git branch              # 查看本地分支
git branch -a           # 查看所有分支（包括远程）

# 2. 创建分支
git branch feature-1    # 创建分支（不切换）
git branch feature-2

# 3. 切换分支
git checkout feature-1  # 切换到feature-1

# 4. 创建并切换（推荐）
git checkout -b feature-3

# 5. 在分支上开发
echo "新功能A" > feature_a.py
git add .
git commit -m "feat: 实现功能A"

# 6. 切换回main
git checkout main

# 7. 合并分支
git merge feature-1
# 如果有冲突，会提示你解决

# 8. 删除已合并的分支
git branch -d feature-1

# 9. 强制删除未合并的分支
git branch -D feature-2

# 10. 查看分支历史
git log --graph --oneline --all
```

**分支命名规范**：
- `feature/功能名`：新功能
- `bugfix/问题描述`：bug修复
- `hotfix/紧急修复`：紧急修复
- `refactor/重构内容`：代码重构

**今日Git练习**：
1. 创建2个功能分支
2. 在不同分支上开发
3. 合并分支到main
4. 删除已合并的分支

---

## Day 4（周四）：三种方法 + 装饰器入门

### 算法线：实例方法、类方法、静态方法（1小时）

#### 4.1 三种方法对比（30分钟）

```python
class MyClass:
    class_var = "我是类变量"
    
    def __init__(self, value):
        self.instance_var = value
    
    # 1. 实例方法：需要访问实例数据
    def instance_method(self):
        print(f"实例方法，可访问：")
        print(f"  实例变量: {self.instance_var}")
        print(f"  类变量: {self.class_var}")
    
    # 2. 类方法：需要访问类数据，不需要实例
    @classmethod
    def class_method(cls):
        print(f"类方法，可访问：")
        print(f"  类变量: {cls.class_var}")
        print(f"  不能访问实例变量！")
    
    # 3. 静态方法：不需要访问类或实例
    @staticmethod
    def static_method():
        print(f"静态方法：")
        print(f"  不能访问类变量")
        print(f"  不能访问实例变量")
        print(f"  就是个普通函数，放在类里组织代码")

# 调用
obj = MyClass("实例数据")
obj.instance_method()
MyClass.class_method()
MyClass.static_method()
```

#### 4.2 实际应用场景（30分钟）

```python
class Date:
    """日期类：演示三种方法的典型用法"""
    
    def __init__(self, year, month, day):
        self.year = year
        self.month = month
        self.day = day
    
    # 实例方法：操作实例数据
    def to_string(self):
        return f"{self.year}-{self.month:02d}-{self.day:02d}"
    
    # 类方法：替代构造函数（工厂方法）
    @classmethod
    def from_string(cls, date_string):
        """从字符串创建日期：'2024-01-15'"""
        year, month, day = map(int, date_string.split('-'))
        return cls(year, month, day)
    
    @classmethod
    def today(cls):
        """获取今天的日期"""
        import datetime
        now = datetime.date.today()
        return cls(now.year, now.month, now.day)
    
    # 静态方法：工具函数
    @staticmethod
    def is_leap_year(year):
        """判断闰年"""
        return (year % 4 == 0 and year % 100 != 0) or (year % 400 == 0)
    
    @staticmethod
    def is_valid_date(year, month, day):
        """验证日期"""
        if month < 1 or month > 12:
            return False
        if day < 1 or day > 31:
            return False
        return True

# 使用
date1 = Date(2024, 1, 15)              # 普通构造
date2 = Date.from_string("2024-06-20")  # 类方法构造
date3 = Date.today()                    # 类方法构造

print(Date.is_leap_year(2024))  # True - 静态方法
```

**使用原则**：
- **实例方法**：需要访问或修改实例数据时
- **类方法**：作为替代构造函数，或需要访问类变量时
- **静态方法**：逻辑上属于这个类，但不需要访问类或实例数据

### 算法线：装饰器入门（1小时）

#### 4.3 装饰器是什么？（20分钟）

```python
# 问题：我想记录函数执行时间
def slow_function():
    import time
    time.sleep(1)
    return "完成"

# 笨办法：在每个函数里加代码
def slow_function():
    import time
    start = time.time()
    time.sleep(1)
    result = "完成"
    print(f"耗时: {time.time() - start:.2f}秒")
    return result

# 问题：如果有100个函数要计时呢？代码重复！

# 装饰器：把通用功能抽取出来
def timer(func):
    """装饰器：计时"""
    def wrapper(*args, **kwargs):
        import time
        start = time.time()
        result = func(*args, **kwargs)
        print(f"{func.__name__} 耗时: {time.time() - start:.2f}秒")
        return result
    return wrapper

# 使用装饰器
@timer
def slow_function():
    import time
    time.sleep(1)
    return "完成"

slow_function()  # 自动计时！
```

**装饰器本质**：
```python
@timer
def func():
    pass

# 等价于：
def func():
    pass
func = timer(func)
```

#### 4.4 装饰器原理（40分钟）

```python
# 1. 函数是一等公民
def say_hello():
    return "Hello"

# 函数可以赋值
func = say_hello
print(func())  # Hello

# 函数可以作为参数
def call_twice(func):
    func()
    func()

call_twice(say_hello)  # 调用两次

# 2. 闭包：内层函数可以访问外层函数的变量
def outer(x):
    def inner(y):
        return x + y  # inner可以访问outer的x
    return inner

add_5 = outer(5)
print(add_5(3))  # 8

# 3. 装饰器 = 高阶函数 + 闭包
def repeat(times):
    """带参数的装饰器"""
    def decorator(func):
        def wrapper(*args, **kwargs):
            results = []
            for _ in range(times):
                result = func(*args, **kwargs)
                results.append(result)
            return results
        return wrapper
    return decorator

@repeat(3)
def greet(name):
    return f"Hello, {name}!"

print(greet("张三"))  # ['Hello, 张三!', 'Hello, 张三!', 'Hello, 张三!']
```

### 工程线：Git远程仓库（30分钟）

#### 连接GitHub（30分钟）

```bash
# 1. 在GitHub上创建仓库
# 网页操作：New repository -> ai-learning

# 2. 生成SSH密钥（如果还没有）
ssh-keygen -t ed25519 -C "your.email@example.com"
# 一路回车

# 3. 查看公钥
cat ~/.ssh/id_ed25519.pub
# 复制输出内容

# 4. 在GitHub添加SSH密钥
# Settings -> SSH and GPG keys -> New SSH key
# 粘贴公钥

# 5. 测试连接
ssh -T git@github.com

# 6. 关联远程仓库
git remote add origin git@github.com:你的用户名/ai-learning.git

# 7. 推送到远程
git push -u origin main

# 8. 查看远程仓库
git remote -v
```

---

## Day 5（周五）：装饰器进阶 + Git远程协作

### 算法线：装饰器进阶（2小时）

#### 5.1 functools.wraps（20分钟）

```python
# 问题：装饰后函数的元信息丢失
def timer(func):
    def wrapper(*args, **kwargs):
        import time
        start = time.time()
        result = func(*args, **kwargs)
        print(f"耗时: {time.time() - start:.2f}秒")
        return result
    return wrapper

@timer
def calculate(n):
    """计算1到n的和"""
    return sum(range(n))

print(calculate.__name__)  # wrapper（不是calculate！）
print(calculate.__doc__)   # None（文档字符串丢失！）

# 解决：使用functools.wraps
from functools import wraps

def timer(func):
    @wraps(func)  # 保留原函数的元信息
    def wrapper(*args, **kwargs):
        import time
        start = time.time()
        result = func(*args, **kwargs)
        print(f"耗时: {time.time() - start:.2f}秒")
        return result
    return wrapper

@timer
def calculate(n):
    """计算1到n的和"""
    return sum(range(n))

print(calculate.__name__)  # calculate
print(calculate.__doc__)   # 计算1到n的和
```

#### 5.2 常用装饰器模式（60分钟）

**1. 日志记录**
```python
from functools import wraps
import logging

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

def log_function_call(func):
    """记录函数调用"""
    @wraps(func)
    def wrapper(*args, **kwargs):
        logger.info(f"调用 {func.__name__}")
        logger.info(f"  参数: args={args}, kwargs={kwargs}")
        result = func(*args, **kwargs)
        logger.info(f"  返回: {result}")
        return result
    return wrapper

@log_function_call
def add(a, b):
    return a + b

add(3, 5)
```

**2. 缓存结果**
```python
from functools import wraps

def cache(func):
    """缓存函数结果"""
    cached_results = {}
    
    @wraps(func)
    def wrapper(*args):
        if args in cached_results:
            print(f"从缓存返回: {args}")
            return cached_results[args]
        
        result = func(*args)
        cached_results[args] = result
        return result
    
    return wrapper

@cache
def fibonacci(n):
    """计算斐波那契数"""
    print(f"计算 fibonacci({n})")
    if n <= 1:
        return n
    return fibonacci(n-1) + fibonacci(n-2)

print(fibonacci(5))  # 第一次计算
print(fibonacci(5))  # 从缓存返回
```

**3. 重试机制**
```python
from functools import wraps
import time

def retry(max_attempts=3, delay=1):
    """失败时重试"""
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            for attempt in range(max_attempts):
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    if attempt == max_attempts - 1:
                        raise
                    print(f"失败，{delay}秒后重试... ({attempt + 1}/{max_attempts})")
                    time.sleep(delay)
        return wrapper
    return decorator

@retry(max_attempts=3, delay=2)
def unstable_network_call():
    """模拟不稳定的网络请求"""
    import random
    if random.random() < 0.7:
        raise ConnectionError("网络错误")
    return "成功"
```

**4. 权限检查**
```python
from functools import wraps

def require_auth(func):
    """需要认证"""
    @wraps(func)
    def wrapper(user, *args, **kwargs):
        if not user.get("is_authenticated"):
            raise PermissionError("需要登录")
        return func(user, *args, **kwargs)
    return wrapper

def require_role(role):
    """需要特定角色"""
    def decorator(func):
        @wraps(func)
        def wrapper(user, *args, **kwargs):
            if user.get("role") != role:
                raise PermissionError(f"需要{role}权限")
            return func(user, *args, **kwargs)
        return wrapper
    return decorator

@require_auth
@require_role("admin")
def delete_user(user, user_id):
    return f"删除用户 {user_id}"

# 测试
admin_user = {"is_authenticated": True, "role": "admin"}
normal_user = {"is_authenticated": True, "role": "user"}

print(delete_user(admin_user, 123))  # 成功
# delete_user(normal_user, 123)  # 报错：需要admin权限
```

#### 5.3 练习：实现装饰器（40分钟）

```python
"""
实现以下装饰器：

1. @validate_input - 验证输入参数
   确保所有参数都是正数
   
2. @measure_memory - 测量内存使用
   记录函数执行前后的内存差异
   
3. @rate_limit - 限制调用频率
   每秒最多调用N次
"""

# 你的代码...
```

### 工程线：Git远程协作（1小时）

#### 5.1 推送与拉取（30分钟）

```bash
# 1. 查看远程仓库
git remote -v

# 2. 推送分支
git push origin main
git push origin feature-1

# 3. 拉取更新
git pull origin main
# 等价于：
# git fetch origin main
# git merge origin/main

# 4. fetch vs pull
# fetch：只下载，不合并
git fetch origin
git log origin/main  # 查看远程分支的提交

# pull：下载并合并
git pull origin main

# 5. 推送本地分支到远程
git checkout -b feature-new
git push -u origin feature-new  # -u 设置上游分支

# 6. 删除远程分支
git push origin --delete feature-old

# 7. 查看所有分支（包括远程）
git branch -a
```

#### 5.2 协作流程（30分钟）

```bash
# GitHub Flow工作流

# 每天开始工作前：
git checkout main
git pull origin main

# 创建功能分支
git checkout -b feature/user-login

# 开发...
git add .
git commit -m "feat(auth): 实现登录接口"

# 继续开发...
git add .
git commit -m "feat(auth): 添加登录验证"

# 推送到远程
git push -u origin feature/user-login

# 在GitHub上创建Pull Request
# Code Review 后合并

# 本地清理
git checkout main
git pull origin main
git branch -d feature/user-login
```

---

## 周六（Day 6）：综合实战

### 上午：Python综合练习（4小时）

#### 项目：任务管理系统

**需求**：
1. 任务类（Task）：标题、描述、状态、优先级
2. 任务列表类（TaskList）：管理多个任务
3. 使用装饰器记录操作日志
4. 使用魔术方法支持迭代、索引
5. 单元测试

**项目结构**：
```
week01-python-basics/
└── task_manager/
    ├── __init__.py
    ├── task.py          # Task类
    ├── task_list.py     # TaskList类
    ├── decorators.py    # 装饰器
    ├── tests/
    │   ├── test_task.py
    │   └── test_task_list.py
    └── main.py
```

**实现要求**：

```python
# task.py
class Task:
    """任务类
    
    要求：
    1. 属性：id, title, description, status, priority
    2. 状态：pending, in_progress, completed
    3. 优先级：low, medium, high
    4. 实现__str__, __repr__, __eq__
    5. 实现状态转换方法
    """
    pass

# task_list.py
class TaskList:
    """任务列表
    
    要求：
    1. 支持添加、删除、查找任务
    2. 实现__len__, __getitem__, __iter__
    3. 支持按状态、优先级筛选
    4. 支持保存到文件、从文件加载
    5. 所有修改操作使用装饰器记录日志
    """
    pass

# decorators.py
def log_operation(func):
    """记录操作日志
    
    要求：
    1. 记录函数名、参数、返回值
    2. 记录时间戳
    3. 保存到log.txt
    """
    pass
```

### 下午：Git实战（4小时）

#### Git工作流实践

```bash
# 1. 创建项目分支结构
git checkout main
git checkout -b develop

# 2. 功能开发
git checkout -b feature/task-model
# 实现Task类
git add task.py
git commit -m "feat(task): 实现Task类基础功能"
git commit -m "feat(task): 添加状态转换方法"

# 3. 合并到develop
git checkout develop
git merge feature/task-model
git branch -d feature/task-model

# 4. 继续开发第二个功能
git checkout -b feature/task-list
# 实现TaskList类
git add task_list.py
git commit -m "feat(tasklist): 实现TaskList类"

# 5. 模拟冲突解决
# 在两个分支上修改同一文件
git checkout develop
echo "# Develop版本" > README.md
git add README.md
git commit -m "docs: 更新README"

git checkout feature/task-list
echo "# Feature版本" > README.md
git add README.md
git commit -m "docs: 添加使用说明"

# 尝试合并，产生冲突
git checkout develop
git merge feature/task-list
# 解决冲突...
git add README.md
git commit -m "merge: 解决README冲突"

# 6. 推送到远程
git push origin develop
git push origin main
```

---

## 周日（Day 7）：复盘与总结

### 上午：完善项目（3小时）

1. 添加单元测试（pytest）
2. 添加类型提示
3. 完善文档
4. Code Review自己的代码

```bash
# 安装pytest
pip install pytest

# 运行测试
pytest task_manager/tests/ -v

# 测试覆盖率
pip install pytest-cov
pytest --cov=task_manager task_manager/tests/
```

### 下午：学习总结（3小时）

#### 写周总结（2小时）

创建 `notes/week01-summary.md`：

```markdown
# 第一周学习总结

## 本周学习内容

### Python面向对象
1. ✅ 类与对象：封装、self、__init__
2. ✅ 继承与多态：super()、方法重写
3. ✅ 魔术方法：__str__, __add__, __eq__ 等
4. ✅ 三种方法：实例方法、类方法、静态方法
5. ✅ 装饰器基础：@语法、闭包、functools.wraps

### Git版本控制
1. ✅ Git配置：user.name, user.email
2. ✅ 基础命令：add, commit, status, log
3. ✅ 分支管理：branch, checkout, merge
4. ✅ 远程仓库：remote, push, pull
5. ✅ 工作流：GitHub Flow

## 重点概念

### 1. 类与实例的关系
- 类是模板，实例是具体对象
- self代表实例本身
- __init__不是构造函数，__new__才是

### 2. 继承与super()
- super()支持多重继承
- 遵循MRO（方法解析顺序）
- 子类可以重写父类方法

### 3. 装饰器本质
- 装饰器是返回函数的函数
- @符号是语法糖
- functools.wraps保留元信息

### 4. Git三区模型
- 工作区 -> 暂存区 -> 仓库
- add是暂存，commit是提交
- push是推送到远程

## 遇到的问题与解决

### 问题1：实例属性 vs 类属性混淆
**现象**：修改类属性时所有实例都受影响

**原因**：类属性是共享的

**解决**：
- 类属性用于共享数据（如计数器）
- 实例属性用于独立数据
- 通过类名修改类属性：`ClassName.class_var = value`

### 问题2：装饰器丢失函数信息
**现象**：`func.__name__`变成`wrapper`

**解决**：使用`@wraps(func)`

### 问题3：Git合并冲突
**现象**：两个分支修改了同一文件

**解决**：
1. 打开冲突文件
2. 手动编辑，保留需要的部分
3. `git add` 标记为已解决
4. `git commit` 完成合并

## 项目总结

### 任务管理系统
- **代码量**：约300行
- **测试覆盖率**：85%
- **Git提交**：15次
- **学到的**：
  - 类的组合使用
  - 装饰器实际应用
  - 单元测试编写

## 本周数据

- **学习时长**：32小时（超额完成）
- **代码行数**：500+ 行
- **Git提交**：28次
- **完成练习**：12个

## 下周计划

### Week 2重点
1. 生成器与迭代器
2. 异常处理机制
3. 上下文管理器
4. Git进阶操作

### 预习内容
- yield关键字
- 迭代器协议
- with语句原理

## 自我评价

### 做得好的
- ✅ 每天坚持学习
- ✅ 完成了所有练习
- ✅ 养成了每天commit的习惯

### 需要改进的
- ⚠️ 有些概念理解不够深入
- ⚠️ 代码质量还需提高
- ⚠️ 需要多做项目实践

## 资源推荐

### 文档
- [Python官方文档](https://docs.python.org/zh-cn/3/)
- [Git官方文档](https://git-scm.com/book/zh/v2)

### 视频
- B站：Python面向对象编程
- YouTube：Git & GitHub Tutorial

### 练习
- LeetCode：Python专题
- GitHub：搜索interesting-python-projects
```

#### 整理代码（1小时）

1. 清理无用代码
2. 统一代码风格
3. 添加注释和文档
4. 确保所有测试通过
5. 最后一次commit和push

```bash
# 代码格式化
pip install black
black task_manager/

# 类型检查
pip install mypy
mypy task_manager/

# 最后的commit
git add .
git commit -m "refactor: 代码格式化和优化"
git push origin main
```

---

# 第一周检验清单

## Python基础

**面向对象**
- [ ] 能独立创建类并实例化对象
- [ ] 理解self的含义
- [ ] 掌握继承和super()的用法
- [ ] 能实现5个以上魔术方法
- [ ] 理解实例属性vs类属性的区别

**三种方法**
- [ ] 知道何时使用实例方法
- [ ] 能用类方法作为替代构造函数
- [ ] 知道静态方法的使用场景

**装饰器**
- [ ] 理解装饰器的本质
- [ ] 能写简单的装饰器
- [ ] 会使用@wraps保留函数信息
- [ ] 了解常见装饰器模式（日志、缓存、计时）

## Git版本控制

**基础操作**
- [ ] 能独立初始化仓库
- [ ] 熟练使用add、commit、status、log
- [ ] 能写规范的commit message
- [ ] 会使用.gitignore

**分支管理**
- [ ] 能创建、切换、删除分支
- [ ] 理解分支合并的概念
- [ ] 能解决简单的合并冲突

**远程协作**
- [ ] 配置好SSH密钥
- [ ] 能推送代码到GitHub
- [ ] 理解fetch和pull的区别

## 项目产出

- [ ] GitHub上有ai-learning仓库
- [ ] 完成任务管理系统项目
- [ ] 本周至少7天有commit记录
- [ ] 有单元测试和文档

---

# Week 2 预告

## 主要内容

### Python进阶
1. **生成器与迭代器**：yield、迭代器协议
2. **异常处理**：try/except/finally、自定义异常
3. **上下文管理器**：with语句、__enter__/__exit__

### Git进阶
1. **Git进阶操作**：rebase、cherry-pick、stash
2. **冲突解决技巧**
3. **Git最佳实践**

### 实战项目
**数据处理Pipeline**：
- 使用生成器处理大文件
- 完善的异常处理
- 上下文管理器管理资源
- 完整的Git工作流

---

# 常用命令速查

## Python相关

```python
# 类定义
class MyClass:
    def __init__(self):
        pass
    
    @classmethod
    def class_method(cls):
        pass
    
    @staticmethod
    def static_method():
        pass

# 装饰器
from functools import wraps

def my_decorator(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)
    return wrapper

# 带参数的装饰器
def decorator_with_args(arg):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            return func(*args, **kwargs)
        return wrapper
    return decorator
```

## Git命令

```bash
# 配置
git config --global user.name "name"
git config --global user.email "email"

# 基础
git init
git add .
git commit -m "message"
git status
git log --oneline

# 分支
git branch                  # 查看分支
git branch name            # 创建分支
git checkout name          # 切换分支
git checkout -b name       # 创建并切换
git merge name             # 合并分支
git branch -d name         # 删除分支

# 远程
git remote add origin url
git push -u origin main
git pull origin main
git fetch origin

# 撤销
git restore file           # 撤销工作区
git restore --staged file  # 撤销暂存
git reset --soft HEAD~1    # 软重置
```

---

**Week 1 开始！加油！🚀**

记住：
1. 每天都要commit
2. 不懂就问（AI助手随时待命）
3. 动手比看更重要
4. 保持节奏，坚持到底
