# 第二周详细学习计划：生成器 + 异常 + 上下文管理器 + Git进阶

> 📅 学习周期：第2周  
> ⏰ 总时长：30小时  
> 🎯 目标：掌握Python高级特性，精通Git协作流程

---

## 时间分配

| 时间段 | 内容 | 时长 |
|--------|------|------|
| 周一-周五晚上 | Python进阶特性 | 10小时 |
| 周一-周五晚上 | Git进阶操作 | 6小时 |
| 周六 | 综合实战 | 8小时 |
| 周日 | 项目完善 + 总结 | 6小时 |
| **总计** | | **30小时** |

---

# Day 1（周一）：迭代器协议 + Git stash

## 算法线：迭代器协议（2小时）

### 1.1 什么是迭代？（20分钟）

```python
# 可迭代对象：可以被for循环遍历的对象
items = [1, 2, 3, 4, 5]
for item in items:
    print(item)

# 问题：为什么列表可以被for循环？

# 答案：因为列表实现了迭代器协议

# 手动迭代的过程
items = [1, 2, 3]
iterator = iter(items)  # 获取迭代器
print(next(iterator))   # 1
print(next(iterator))   # 2
print(next(iterator))   # 3
# print(next(iterator))  # StopIteration异常

# for循环的本质
items = [1, 2, 3]
iterator = iter(items)
while True:
    try:
        item = next(iterator)
        print(item)
    except StopIteration:
        break
```

### 1.2 实现自己的迭代器（40分钟）

```python
class Countdown:
    """倒计时迭代器"""
    
    def __init__(self, start):
        self.current = start
    
    def __iter__(self):
        """返回迭代器对象本身"""
        return self
    
    def __next__(self):
        """返回下一个值"""
        if self.current <= 0:
            raise StopIteration
        
        self.current -= 1
        return self.current + 1

# 使用
counter = Countdown(5)
for num in counter:
    print(num)  # 5, 4, 3, 2, 1

# 只能迭代一次！
for num in counter:
    print(num)  # 什么都不输出（已经耗尽）

# 解决方案：分离迭代器和可迭代对象
class Countdown:
    """倒计时（可迭代对象）"""
    
    def __init__(self, start):
        self.start = start
    
    def __iter__(self):
        """返回新的迭代器"""
        return CountdownIterator(self.start)

class CountdownIterator:
    """倒计时迭代器"""
    
    def __init__(self, start):
        self.current = start
    
    def __iter__(self):
        return self
    
    def __next__(self):
        if self.current <= 0:
            raise StopIteration
        self.current -= 1
        return self.current + 1

# 现在可以多次迭代
counter = Countdown(5)
for num in counter:
    print(num)  # 5, 4, 3, 2, 1
for num in counter:
    print(num)  # 5, 4, 3, 2, 1（再次迭代）
```

### 1.3 实用迭代器示例（40分钟）

```python
# 1. 文件行迭代器（处理大文件）
class FileLineIterator:
    """逐行读取文件，不一次性加载到内存"""
    
    def __init__(self, filename):
        self.filename = filename
        self.file = None
    
    def __iter__(self):
        self.file = open(self.filename, 'r', encoding='utf-8')
        return self
    
    def __next__(self):
        line = self.file.readline()
        if not line:
            self.file.close()
            raise StopIteration
        return line.strip()

# 使用：处理GB级别的日志文件
for line in FileLineIterator('large_log.txt'):
    if 'ERROR' in line:
        print(line)

# 2. 斐波那契数列迭代器
class Fibonacci:
    """无限斐波那契数列"""
    
    def __init__(self):
        self.a, self.b = 0, 1
    
    def __iter__(self):
        return self
    
    def __next__(self):
        current = self.a
        self.a, self.b = self.b, self.a + self.b
        return current

# 使用：获取前10个斐波那契数
fib = Fibonacci()
for i, num in enumerate(fib):
    if i >= 10:
        break
    print(num)

# 3. 范围迭代器（类似range）
class MyRange:
    """自定义range"""
    
    def __init__(self, start, end=None, step=1):
        if end is None:
            self.start = 0
            self.end = start
        else:
            self.start = start
            self.end = end
        self.step = step
    
    def __iter__(self):
        current = self.start
        while current < self.end:
            yield current
            current += self.step
    
    def __len__(self):
        return max(0, (self.end - self.start + self.step - 1) // self.step)

# 使用
for i in MyRange(5):
    print(i)  # 0, 1, 2, 3, 4

for i in MyRange(1, 10, 2):
    print(i)  # 1, 3, 5, 7, 9
```

### 1.4 练习（20分钟）

```python
"""
实现一个数据批次迭代器BatchIterator

功能：将数据分批返回（用于机器学习的batch processing）

示例：
data = [1, 2, 3, 4, 5, 6, 7, 8, 9]
for batch in BatchIterator(data, batch_size=3):
    print(batch)
# 输出：
# [1, 2, 3]
# [4, 5, 6]
# [7, 8, 9]
"""

class BatchIterator:
    # 你的代码
    pass
```

## 工程线：Git stash（1小时）

### 2.1 为什么需要stash？（15分钟）

```bash
# 场景：你正在feature分支开发，突然需要修复main分支的bug

# 当前状态
git status
# On branch feature/new-feature
# Changes not staged for commit:
#   modified: feature.py

# 问题：有未提交的修改，无法切换分支
git checkout main
# error: Your local changes would be overwritten by checkout

# 方案1：先提交（不好，因为功能还没完成）
git add .
git commit -m "WIP: 未完成的功能"

# 方案2：使用stash暂存
git stash save "feature开发到一半"
# Saved working directory and index state

# 现在可以切换分支了
git checkout main
# 修复bug...
git add .
git commit -m "fix: 修复紧急bug"

# 切回feature分支
git checkout feature/new-feature

# 恢复之前的工作
git stash pop
# 继续开发...
```

### 2.2 stash操作详解（45分钟）

```bash
# 1. 暂存修改
git stash                       # 暂存，使用默认消息
git stash save "描述信息"       # 暂存，使用自定义消息
git stash -u                    # 包含未跟踪的文件
git stash --all                 # 包含未跟踪和忽略的文件

# 2. 查看stash列表
git stash list
# stash@{0}: On feature: 描述信息
# stash@{1}: On main: 另一个暂存
# stash@{2}: WIP on develop

# 3. 查看stash内容
git stash show                  # 查看最新stash的修改摘要
git stash show -p               # 查看完整diff
git stash show stash@{1}        # 查看指定stash

# 4. 恢复stash
git stash pop                   # 恢复最新stash，并删除
git stash pop stash@{1}         # 恢复指定stash，并删除

git stash apply                 # 恢复最新stash，但保留
git stash apply stash@{1}       # 恢复指定stash，但保留

# 5. 删除stash
git stash drop                  # 删除最新stash
git stash drop stash@{1}        # 删除指定stash
git stash clear                 # 删除所有stash

# 6. 从stash创建分支
git stash branch new-branch     # 从最新stash创建分支
git stash branch bug-fix stash@{2}  # 从指定stash创建分支

# 实战演练
# 场景1：保存当前工作，切换到其他分支
git stash save "实现用户登录功能"
git checkout main
# 做其他工作...
git checkout -
git stash pop

# 场景2：清理工作区
# 有很多未完成的修改，想重新开始
git stash save "旧的实现方案"
# 重新编写...
# 如果新方案不好，可以恢复：
git stash pop
```

---

# Day 2（周二）：生成器 + Git rebase基础

## 算法线：生成器（2小时）

### 2.1 生成器是什么？（20分钟）

```python
# 问题：迭代器需要写很多代码
class Countdown:
    def __init__(self, start):
        self.start = start
    def __iter__(self):
        return CountdownIterator(self.start)

class CountdownIterator:
    def __init__(self, start):
        self.current = start
    def __iter__(self):
        return self
    def __next__(self):
        if self.current <= 0:
            raise StopIteration
        self.current -= 1
        return self.current + 1

# 生成器：用yield实现，简洁得多！
def countdown(start):
    """倒计时生成器"""
    while start > 0:
        yield start
        start -= 1

# 使用完全相同
for num in countdown(5):
    print(num)  # 5, 4, 3, 2, 1

# 生成器的本质：
# 1. yield会暂停函数执行
# 2. 下次调用next()时从yield处继续
# 3. 函数结束时自动抛出StopIteration
```

### 2.2 生成器的执行流程（30分钟）

```python
def simple_generator():
    print("开始")
    yield 1
    print("继续")
    yield 2
    print("再继续")
    yield 3
    print("结束")

# 创建生成器对象（不执行函数体）
gen = simple_generator()
print(type(gen))  # <class 'generator'>

# 第一次调用next
print(next(gen))  
# 输出：
# 开始
# 1

# 第二次调用next
print(next(gen))
# 输出：
# 继续
# 2

# 第三次调用next
print(next(gen))
# 输出：
# 再继续
# 3

# 第四次调用next
# print(next(gen))  
# 输出：
# 结束
# StopIteration

# 生成器方法
def counter():
    count = 0
    while True:
        value = yield count  # yield可以接收send的值
        if value is not None:
            count = value
        else:
            count += 1

gen = counter()
print(next(gen))        # 0
print(next(gen))        # 1
print(gen.send(10))     # 10（重置为10）
print(next(gen))        # 11
```

### 2.3 生成器的实际应用（50分钟）

**1. 处理大文件**
```python
def read_large_file(file_path, chunk_size=1024):
    """逐块读取大文件"""
    with open(file_path, 'r', encoding='utf-8') as f:
        while True:
            chunk = f.read(chunk_size)
            if not chunk:
                break
            yield chunk

# 使用：处理100GB的文件，内存只用几KB
for chunk in read_large_file('huge_file.txt'):
    process(chunk)

def read_lines(file_path):
    """逐行读取（推荐）"""
    with open(file_path, 'r', encoding='utf-8') as f:
        for line in f:
            yield line.strip()

# 筛选含有ERROR的行
error_lines = (line for line in read_lines('log.txt') if 'ERROR' in line)
for line in error_lines:
    print(line)
```

**2. 数据Pipeline**
```python
# 链式处理数据
def read_csv(filename):
    """读取CSV"""
    with open(filename, 'r') as f:
        next(f)  # 跳过表头
        for line in f:
            yield line.strip().split(',')

def filter_by_age(rows, min_age):
    """筛选年龄"""
    for row in rows:
        age = int(row[2])
        if age >= min_age:
            yield row

def extract_names(rows):
    """提取姓名"""
    for row in rows:
        yield row[0]

# 使用Pipeline
rows = read_csv('users.csv')
adults = filter_by_age(rows, 18)
names = extract_names(adults)

for name in names:
    print(name)
```

**3. 无限序列**
```python
def fibonacci():
    """无限斐波那契数列"""
    a, b = 0, 1
    while True:
        yield a
        a, b = b, a + b

# 获取前10个
import itertools
for num in itertools.islice(fibonacci(), 10):
    print(num)

def primes():
    """无限素数序列"""
    yield 2
    primes_list = [2]
    candidate = 3
    while True:
        is_prime = True
        for p in primes_list:
            if p * p > candidate:
                break
            if candidate % p == 0:
                is_prime = False
                break
        if is_prime:
            primes_list.append(candidate)
            yield candidate
        candidate += 2
```

**4. 生成器表达式**
```python
# 列表推导式：一次性生成所有元素
squares_list = [x**2 for x in range(1000000)]  # 占用大量内存

# 生成器表达式：按需生成
squares_gen = (x**2 for x in range(1000000))   # 几乎不占内存

# 使用场景：只需要遍历一次
sum_of_squares = sum(x**2 for x in range(1000000))

# 链式处理
text = "hello world python programming"
# 转大写 -> 筛选长度>5 -> 反转
result = (word[::-1] for word in (word.upper() for word in text.split()) if len(word) > 5)
print(list(result))
```

### 2.4 练习（20分钟）

```python
"""
1. 实现一个生成器：读取目录下所有Python文件
   遍历目录树，yield每个.py文件的路径
   
2. 实现一个数据流处理Pipeline：
   读取日志 -> 解析时间戳 -> 筛选错误 -> 统计
"""

import os

def find_files(directory, extension):
    """查找指定扩展名的文件"""
    # 你的代码
    pass

# 测试
for filepath in find_files('.', '.py'):
    print(filepath)
```

## 工程线：Git rebase基础（1小时）

### 2.1 rebase vs merge（20分钟）

```bash
# 场景：feature分支需要同步main的更新

# 初始状态
main    ─●─●─●
         ↓
feature  └─●─●

# main有了新提交
main    ─●─●─●─●─●
         ↓
feature  └─●─●

# 方式1：merge（会产生合并提交）
git checkout feature
git merge main

main    ─●─●─●─●─●
         ↓       ↘
feature  └─●─●───●（merge commit）

# 方式2：rebase（历史是线性的）
git checkout feature
git rebase main

main    ─●─●─●─●─●
                 ↓
feature          └─●─●（移动到main最新提交后）

# 区别：
# merge: 保留分支历史，但产生额外的合并提交
# rebase: 历史线性，但会改写提交历史
```

### 2.2 rebase实践（40分钟）

```bash
# 1. 基本rebase
git checkout feature
git rebase main

# 如果有冲突：
# a. 解决冲突
# b. git add <解决的文件>
# c. git rebase --continue

# 如果想放弃rebase：
git rebase --abort

# 2. 交互式rebase（整理提交历史）
git rebase -i HEAD~3  # 整理最近3次提交

# 编辑器会打开：
# pick abc123 feat: 添加功能A
# pick def456 fix: 修复bug
# pick ghi789 feat: 添加功能B

# 可以做的操作：
# pick   - 保留提交
# reword - 修改提交信息
# edit   - 修改提交内容
# squash - 合并到上一个提交
# fixup  - 合并到上一个提交，丢弃commit信息
# drop   - 删除提交

# 示例：合并多个提交
# pick abc123 feat: 添加功能A
# squash def456 fix: 修复功能A的bug
# squash ghi789 feat: 完善功能A

# 保存后，三个提交会合并成一个

# 3. rebase实战
# 创建测试环境
git checkout main
echo "main file" > main.txt
git add . && git commit -m "main: 添加main文件"

git checkout -b feature
echo "feature 1" > feature.txt
git add . && git commit -m "feat: 功能1"
echo "feature 2" >> feature.txt
git add . && git commit -m "feat: 功能2"
echo "feature 3" >> feature.txt
git add . && git commit -m "feat: 功能3"

# 整理提交：将3个提交合并为1个
git rebase -i HEAD~3
# 将后两个改为squash

# 查看历史
git log --oneline
```

**什么时候用rebase？**
- ✅ 本地分支整理提交
- ✅ 同步上游更新（feature同步main）
- ❌ 不要rebase已推送到远程的公共分支
- ❌ 不要rebase别人正在基于其工作的分支

---

# Day 3（周三）：异常处理 + Git cherry-pick

## 算法线：异常处理（2小时）

### 3.1 为什么需要异常处理？（15分钟）

```python
# 不处理异常：程序崩溃
def divide(a, b):
    return a / b

print(divide(10, 2))  # 5.0
# print(divide(10, 0))  # ZeroDivisionError: division by zero
# 程序崩溃，后续代码不执行

# 处理异常：程序继续运行
def divide(a, b):
    try:
        return a / b
    except ZeroDivisionError:
        print("除数不能为0")
        return None

print(divide(10, 2))   # 5.0
print(divide(10, 0))   # 除数不能为0，返回None
print("程序继续运行")  # 这行会执行
```

### 3.2 try/except/else/finally（40分钟）

```python
# 完整语法
try:
    # 可能出错的代码
    result = risky_operation()
except SpecificError as e:
    # 处理特定异常
    handle_error(e)
except AnotherError as e:
    # 处理另一个异常
    handle_another_error(e)
except Exception as e:
    # 处理所有其他异常
    handle_general_error(e)
else:
    # 如果没有异常，执行这里
    print("操作成功")
finally:
    # 无论是否异常，都会执行
    cleanup()

# 示例1：文件读取
def read_file(filename):
    try:
        f = open(filename, 'r')
        content = f.read()
    except FileNotFoundError:
        print(f"文件 {filename} 不存在")
        return None
    except PermissionError:
        print(f"没有权限读取 {filename}")
        return None
    except Exception as e:
        print(f"未知错误: {e}")
        return None
    else:
        print("文件读取成功")
        return content
    finally:
        # 无论如何都要关闭文件
        try:
            f.close()
        except:
            pass

# 示例2：数据库操作
def update_database(data):
    connection = None
    try:
        connection = connect_to_database()
        cursor = connection.cursor()
        cursor.execute("UPDATE table SET value = ?", data)
        connection.commit()
    except DatabaseConnectionError as e:
        print(f"数据库连接失败: {e}")
        raise
    except DatabaseError as e:
        print(f"数据库操作失败: {e}")
        if connection:
            connection.rollback()
        raise
    else:
        print("数据更新成功")
    finally:
        if connection:
            connection.close()
            print("数据库连接已关闭")

# 示例3：多个except
def process_input(value):
    try:
        # 转换为整数
        num = int(value)
        # 除以10
        result = 100 / num
        # 访问列表
        data = [1, 2, 3]
        item = data[num]
    except ValueError:
        print("输入不是有效的数字")
    except ZeroDivisionError:
        print("不能除以0")
    except IndexError:
        print("索引超出范围")
    except Exception as e:
        print(f"未知错误: {type(e).__name__}: {e}")
    else:
        print(f"结果: {result}, 项: {item}")
        return result, item
```

### 3.3 raise抛出异常（30分钟）

```python
# 1. 抛出异常
def check_age(age):
    if age < 0:
        raise ValueError("年龄不能为负数")
    if age > 150:
        raise ValueError("年龄不合理")
    return True

try:
    check_age(-5)
except ValueError as e:
    print(f"错误: {e}")

# 2. 重新抛出异常
def process_data(data):
    try:
        result = risky_operation(data)
    except Exception as e:
        print(f"处理数据时出错: {e}")
        raise  # 重新抛出，让上层处理

# 3. 异常链（from）
def parse_config(filename):
    try:
        with open(filename) as f:
            config = json.load(f)
    except FileNotFoundError as e:
        raise ConfigError(f"配置文件不存在: {filename}") from e
    except json.JSONDecodeError as e:
        raise ConfigError(f"配置文件格式错误") from e

# 4. 异常传播
def level3():
    raise ValueError("level3 error")

def level2():
    level3()  # 不处理，向上传播

def level1():
    try:
        level2()
    except ValueError as e:
        print(f"在level1捕获: {e}")

level1()
```

### 3.4 自定义异常（30分钟）

```python
# 1. 基本自定义异常
class MyError(Exception):
    """自定义异常基类"""
    pass

class ValidationError(MyError):
    """验证错误"""
    pass

class DatabaseError(MyError):
    """数据库错误"""
    pass

# 使用
def validate_user(user):
    if not user.get('name'):
        raise ValidationError("用户名不能为空")
    if not user.get('email'):
        raise ValidationError("邮箱不能为空")

try:
    validate_user({'name': '张三'})
except ValidationError as e:
    print(f"验证失败: {e}")

# 2. 带属性的异常
class HTTPError(Exception):
    """HTTP错误"""
    def __init__(self, status_code, message):
        self.status_code = status_code
        self.message = message
        super().__init__(f"{status_code}: {message}")

def fetch_url(url):
    if not url.startswith('http'):
        raise HTTPError(400, "无效的URL")
    # 模拟请求失败
    raise HTTPError(404, "页面不存在")

try:
    fetch_url("invalid")
except HTTPError as e:
    print(f"HTTP错误 {e.status_code}: {e.message}")

# 3. 异常层次
class AppError(Exception):
    """应用错误基类"""
    pass

class AuthError(AppError):
    """认证错误"""
    pass

class LoginError(AuthError):
    """登录错误"""
    pass

class PermissionError(AuthError):
    """权限错误"""
    pass

# 可以捕获父类异常
try:
    raise LoginError("用户名或密码错误")
except AuthError as e:  # 捕获所有认证相关错误
    print(f"认证失败: {e}")
```

### 3.5 练习（5分钟）

```python
"""
创建一个用户管理系统的异常层次：

UserError (基类)
├── ValidationError
│   ├── UsernameError
│   └── PasswordError
└── AuthenticationError
    ├── LoginFailedError
    └── TokenExpiredError

实现一个register_user函数，使用这些异常
"""
```

## 工程线：Git cherry-pick（1小时)

### 3.1 什么是cherry-pick？（20分钟）

```bash
# 场景：想把某个分支的特定提交应用到当前分支

# 初始状态
main    ─●─●─●
         ↓
feature  └─A─B─C

# 只想要提交B，不想merge整个分支

# 使用cherry-pick
git checkout main
git cherry-pick <B的commit-id>

# 结果
main    ─●─●─●─B'  # B'是B的副本
         ↓
feature  └─A─B─C
```

### 3.2 cherry-pick实践（40分钟)

```bash
# 1. 基本用法
git cherry-pick <commit-id>

# 2. cherry-pick多个提交
git cherry-pick <commit1> <commit2> <commit3>

# 3. cherry-pick范围
git cherry-pick <start-commit>..<end-commit>  # 不包含start
git cherry-pick <start-commit>^..<end-commit> # 包含start

# 4. 冲突处理
git cherry-pick <commit-id>
# 如果有冲突：
# a. 解决冲突
# b. git add <文件>
# c. git cherry-pick --continue

# 放弃cherry-pick
git cherry-pick --abort

# 5. 实战练习
# 创建测试场景
git checkout main
echo "main 1" > main.txt
git add . && git commit -m "main: commit 1"

git checkout -b feature
echo "feature 1" > feature.txt
git add . && git commit -m "feat: feature 1"
echo "feature 2" >> feature.txt
git add . && git commit -m "feat: feature 2"
echo "feature 3" >> feature.txt
git add . && git commit -m "feat: feature 3"

# 只想要feature 2这个提交
git checkout main
git log feature --oneline  # 找到feature 2的commit-id
git cherry-pick <commit-id>

# 验证
git log --oneline
cat feature.txt
```

**使用场景**：
- ✅ 把hotfix应用到多个分支
- ✅ 提取feature分支中的某个独立提交
- ✅ 撤销某次提交（cherry-pick一个revert commit）
- ❌ 不要滥用，优先考虑merge或rebase

---

# Day 4（周四）：上下文管理器 + Git工作流

## 算法线：上下文管理器（2小时）

### 4.1 with语句的作用（20分钟）

```python
# 不用with：容易忘记关闭资源
file = open('data.txt', 'r')
try:
    content = file.read()
    process(content)
finally:
    file.close()  # 必须记得关闭

# 用with：自动管理资源
with open('data.txt', 'r') as file:
    content = file.read()
    process(content)
# 自动关闭，即使出现异常

# with的本质：调用__enter__和__exit__
class FileManager:
    def __init__(self, filename):
        self.filename = filename
    
    def __enter__(self):
        print("打开文件")
        self.file = open(self.filename, 'r')
        return self.file
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        print("关闭文件")
        self.file.close()
        return False  # 不压制异常

with FileManager('data.txt') as f:
    print(f.read())
# 输出：
# 打开文件
# <文件内容>
# 关闭文件
```

### 4.2 实现自己的上下文管理器（40分钟）

**方式1：类实现**
```python
class DatabaseConnection:
    """数据库连接上下文管理器"""
    
    def __init__(self, host, port):
        self.host = host
        self.port = port
        self.connection = None
    
    def __enter__(self):
        """进入with块时调用"""
        print(f"连接到 {self.host}:{self.port}")
        self.connection = self._connect()
        return self.connection
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        """退出with块时调用
        
        Args:
            exc_type: 异常类型（如果有异常）
            exc_val: 异常值
            exc_tb: 异常追踪
            
        Returns:
            True: 压制异常，False: 不压制
        """
        print(f"关闭连接")
        if self.connection:
            self.connection.close()
        
        if exc_type is not None:
            print(f"发生异常: {exc_type.__name__}: {exc_val}")
        
        return False  # 不压制异常
    
    def _connect(self):
        # 模拟连接
        return {"connected": True}

# 使用
with DatabaseConnection('localhost', 5432) as conn:
    print(f"执行查询: {conn}")
    # raise ValueError("查询错误")  # 测试异常处理

class Timer:
    """计时器上下文管理器"""
    
    def __enter__(self):
        import time
        self.start = time.time()
        return self
    
    def __exit__(self, *args):
        import time
        self.end = time.time()
        self.elapsed = self.end - self.start
        print(f"耗时: {self.elapsed:.4f}秒")
        return False

# 使用
with Timer():
    # 耗时操作
    sum(range(1000000))
```

**方式2：contextlib.contextmanager装饰器**
```python
from contextlib import contextmanager
import time

@contextmanager
def timer():
    """计时器生成器"""
    start = time.time()
    try:
        yield  # yield之前是__enter__，之后是__exit__
    finally:
        end = time.time()
        print(f"耗时: {end - start:.4f}秒")

# 使用
with timer():
    sum(range(1000000))

@contextmanager
def temporary_directory():
    """临时目录"""
    import tempfile
    import shutil
    
    temp_dir = tempfile.mkdtemp()
    print(f"创建临时目录: {temp_dir}")
    
    try:
        yield temp_dir
    finally:
        print(f"删除临时目录: {temp_dir}")
        shutil.rmtree(temp_dir)

# 使用
with temporary_directory() as temp_dir:
    # 在临时目录中工作
    file_path = os.path.join(temp_dir, 'temp.txt')
    with open(file_path, 'w') as f:
        f.write("临时数据")
# 自动清理

@contextmanager
def change_directory(path):
    """临时切换目录"""
    import os
    original_dir = os.getcwd()
    try:
        os.chdir(path)
        yield
    finally:
        os.chdir(original_dir)

# 使用
print(f"当前目录: {os.getcwd()}")
with change_directory('/tmp'):
    print(f"临时切换到: {os.getcwd()}")
print(f"恢复目录: {os.getcwd()}")
```

### 4.3 实用上下文管理器（40分钟）

```python
# 1. 文件批量操作
@contextmanager
def atomic_write(filename):
    """原子写入：要么全部成功，要么回滚"""
    import tempfile
    import shutil
    
    temp_file = tempfile.NamedTemporaryFile(mode='w', delete=False)
    try:
        yield temp_file
        temp_file.close()
        shutil.move(temp_file.name, filename)
    except:
        temp_file.close()
        os.remove(temp_file.name)
        raise

# 使用
with atomic_write('important.txt') as f:
    f.write("重要数据\n")
    f.write("更多数据\n")
    # 如果这里出错，文件不会被破坏

# 2. 环境变量临时修改
@contextmanager
def temporary_env_var(key, value):
    """临时修改环境变量"""
    import os
    old_value = os.environ.get(key)
    os.environ[key] = value
    try:
        yield
    finally:
        if old_value is None:
            del os.environ[key]
        else:
            os.environ[key] = old_value

# 使用
with temporary_env_var('DEBUG', 'true'):
    print(os.environ['DEBUG'])  # true
print(os.environ.get('DEBUG'))  # None（已恢复）

# 3. 数据库事务
@contextmanager
def transaction(connection):
    """数据库事务"""
    try:
        yield connection
        connection.commit()
        print("事务提交")
    except Exception:
        connection.rollback()
        print("事务回滚")
        raise

# 使用
# with transaction(db_connection) as conn:
#     conn.execute("INSERT ...")
#     conn.execute("UPDATE ...")
#     # 如果出错，自动回滚

# 4. 多个上下文管理器
# 方式1：嵌套
with open('input.txt') as infile:
    with open('output.txt', 'w') as outfile:
        outfile.write(infile.read())

# 方式2：一行（Python 3.10+）
with (
    open('input.txt') as infile,
    open('output.txt', 'w') as outfile
):
    outfile.write(infile.read())
```

### 4.4 练习（20分钟）

```python
"""
实现以下上下文管理器：

1. @suppress_exception - 压制特定异常
   with suppress_exception(ValueError):
       int("abc")  # 不会报错
       
2. @mock_time - 模拟时间（测试用）
   with mock_time("2024-01-01 10:00:00"):
       # time.time()返回模拟时间
       
3. @redirect_stdout - 重定向标准输出
   with redirect_stdout('output.txt'):
       print("写入文件")
"""
```

## 工程线：Git工作流最佳实践（1小时）

### 4.1 Git Flow vs GitHub Flow（30分钟）

**Git Flow（复杂项目）**
```
master    ───●───────────────●───────────────●───
             ↓               ↑               ↑
develop   ───●───●───●───●───●───●───●───●───●───
                 ↓       ↑       ↓       ↑
feature         └───●───┘       │       │
                                 │       │
release                          └───●───┘
                                     
hotfix    ───────────────────────────●───────────
                                     ↑
                                  回master和develop

分支说明：
- master: 生产环境，只接受merge
- develop: 开发主线
- feature/*: 功能开发
- release/*: 发布准备
- hotfix/*: 紧急修复
```

**GitHub Flow（简单项目，推荐）**
```
main    ───●───────●───────●───────●───
            ↓       ↑       ↓       ↑
feature-1   └───●───┘       │       │
                             │       │
feature-2   ─────────────────└───●───┘

流程：
1. 从main创建功能分支
2. 开发并提交
3. 创建Pull Request
4. Code Review
5. 合并到main
6. 删除功能分支
```

### 4.2 实践GitHub Flow（30分钟）

```bash
# 完整工作流实战

# 1. 开始新功能
git checkout main
git pull origin main
git checkout -b feature/user-profile

# 2. 开发（小步提交）
# 编写用户资料显示功能
git add user_profile.py
git commit -m "feat(user): 添加用户资料模型"

# 继续开发
git add templates/profile.html
git commit -m "feat(user): 添加资料页面模板"

# 添加测试
git add tests/test_user_profile.py
git commit -m "test(user): 添加资料测试"

# 3. 推送到远程
git push -u origin feature/user-profile

# 4. 在GitHub创建Pull Request
# 标题: feat: 实现用户资料功能
# 描述:
# ## 变更内容
# - 添加用户资料模型
# - 实现资料页面
# - 添加单元测试
#
# ## 测试
# - [x] 单元测试通过
# - [x] 手动测试通过
#
# Closes #123

# 5. Code Review期间可能需要修改
git add .
git commit -m "fix(user): 根据review修改"
git push

# 6. PR合并后，清理本地
git checkout main
git pull origin main
git branch -d feature/user-profile

# 7. 开始下一个功能
git checkout -b feature/next-feature
```

---

# Day 5（周五）：综合练习

## 上午：Python知识整合（2小时）

### 实现一个日志处理系统

```python
"""
功能需求：
1. 读取大型日志文件（使用生成器）
2. 解析日志行（使用迭代器）
3. 筛选特定级别的日志
4. 统计错误信息
5. 将结果写入文件（使用上下文管理器）
6. 完善的异常处理
"""

# log_processor.py
from contextlib import contextmanager
import re
from datetime import datetime
from typing import Iterator, Dict

class LogParseError(Exception):
    """日志解析错误"""
    pass

class LogProcessor:
    """日志处理器"""
    
    LOG_PATTERN = r'(\d{4}-\d{2}-\d{2} \d{2}:\d{2}:\d{2}) \[(\w+)\] (.+)'
    
    def __init__(self, input_file: str, output_file: str):
        self.input_file = input_file
        self.output_file = output_file
    
    def read_logs(self) -> Iterator[str]:
        """生成器：逐行读取日志"""
        try:
            with open(self.input_file, 'r', encoding='utf-8') as f:
                for line in f:
                    yield line.strip()
        except FileNotFoundError:
            raise LogParseError(f"日志文件不存在: {self.input_file}")
        except PermissionError:
            raise LogParseError(f"没有权限读取: {self.input_file}")
    
    def parse_log(self, line: str) -> Dict:
        """解析日志行"""
        match = re.match(self.LOG_PATTERN, line)
        if not match:
            raise LogParseError(f"无法解析日志: {line}")
        
        timestamp, level, message = match.groups()
        return {
            'timestamp': datetime.strptime(timestamp, '%Y-%m-%d %H:%M:%S'),
            'level': level,
            'message': message
        }
    
    def filter_by_level(self, logs: Iterator[Dict], level: str) -> Iterator[Dict]:
        """生成器：筛选特定级别的日志"""
        for log in logs:
            if log['level'] == level:
                yield log
    
    @contextmanager
    def write_results(self):
        """上下文管理器：写入结果"""
        f = None
        try:
            f = open(self.output_file, 'w', encoding='utf-8')
            yield f
        except Exception as e:
            raise LogParseError(f"写入失败: {e}")
        finally:
            if f:
                f.close()
    
    def process(self, level='ERROR'):
        """处理流程"""
        try:
            # 生成器Pipeline
            lines = self.read_logs()
            logs = (self.parse_log(line) for line in lines if line)
            filtered = self.filter_by_level(logs, level)
            
            # 统计
            count = 0
            errors = []
            
            for log in filtered:
                count += 1
                errors.append(log)
            
            # 写入结果
            with self.write_results() as f:
                f.write(f"共找到 {count} 条 {level} 日志\n\n")
                for error in errors:
                    f.write(f"{error['timestamp']} - {error['message']}\n")
            
            print(f"处理完成，结果写入: {self.output_file}")
            
        except LogParseError as e:
            print(f"处理失败: {e}")
            raise
        except Exception as e:
            print(f"未知错误: {e}")
            raise

# 使用
processor = LogProcessor('app.log', 'errors.txt')
processor.process('ERROR')
```

## 下午：Git综合练习（1小时）

```bash
# 模拟真实项目协作流程

# 1. 初始化项目
mkdir python-log-processor
cd python-log-processor
git init
git checkout -b main

# 2. 创建项目结构
cat > README.md << 'EOF'
# Python Log Processor

日志处理工具

## 功能
- 解析日志
- 筛选级别
- 统计错误
EOF

git add README.md
git commit -m "docs: 初始化README"

# 3. 开发核心功能
git checkout -b feature/log-parser

# 创建代码文件...
git add log_processor.py
git commit -m "feat: 实现LogProcessor类"

git add tests/test_log_processor.py
git commit -m "test: 添加单元测试"

# 4. 开发过程中main有更新，需要同步
git checkout main
echo "## 安装" >> README.md
git add README.md
git commit -m "docs: 添加安装说明"

# 回到feature分支，rebase同步
git checkout feature/log-parser
git rebase main

# 5. 功能完成，创建PR（模拟）
git checkout main
git merge feature/log-parser --no-ff
git branch -d feature/log-parser

# 6. 发现bug，紧急修复
git checkout -b hotfix/fix-encoding
# 修复代码...
git add log_processor.py
git commit -m "fix: 修复编码错误"

git checkout main
git merge hotfix/fix-encoding
git branch -d hotfix/fix-encoding

# 7. 整理历史（交互式rebase）
git rebase -i HEAD~5

# 8. 推送到远程（如果有）
# git remote add origin git@github.com:username/repo.git
# git push -u origin main
```

---

# 周六（Day 6）：综合项目

## 项目：数据ETL Pipeline（7小时）

### 项目需求

构建一个数据提取-转换-加载(ETL)工具：
1. 从多个源读取数据（CSV, JSON, 日志文件）
2. 数据清洗和转换
3. 写入目标格式
4. 完整的异常处理
5. 使用生成器处理大文件
6. 上下文管理器管理资源
7. 完善的Git工作流

### 项目结构

```
etl_pipeline/
├── README.md
├── requirements.txt
├── .gitignore
├── src/
│   ├── __init__.py
│   ├── readers/
│   │   ├── __init__.py
│   │   ├── base.py
│   │   ├── csv_reader.py
│   │   └── json_reader.py
│   ├── transformers/
│   │   ├── __init__.py
│   │   ├── base.py
│   │   └── data_cleaner.py
│   ├── writers/
│   │   ├── __init__.py
│   │   ├── base.py
│   │   └── csv_writer.py
│   ├── pipeline.py
│   └── exceptions.py
├── tests/
│   ├── test_readers.py
│   ├── test_transformers.py
│   └── test_pipeline.py
└── examples/
    ├── sample_data.csv
    └── run_pipeline.py
```

### 核心代码示例

```python
# src/exceptions.py
class ETLError(Exception):
    """ETL错误基类"""
    pass

class ReaderError(ETLError):
    """读取错误"""
    pass

class TransformError(ETLError):
    """转换错误"""
    pass

class WriterError(ETLError):
    """写入错误"""
    pass

# src/readers/base.py
from abc import ABC, abstractmethod
from typing import Iterator, Dict
from contextlib import contextmanager

class BaseReader(ABC):
    """读取器基类"""
    
    def __init__(self, source: str):
        self.source = source
    
    @abstractmethod
    def read(self) -> Iterator[Dict]:
        """生成器：逐条读取数据"""
        pass
    
    @contextmanager
    def open_source(self):
        """上下文管理器：打开数据源"""
        pass

# src/readers/csv_reader.py
import csv
from .base import BaseReader

class CSVReader(BaseReader):
    """CSV读取器"""
    
    def read(self) -> Iterator[Dict]:
        try:
            with open(self.source, 'r', encoding='utf-8') as f:
                reader = csv.DictReader(f)
                for row in reader:
                    yield row
        except FileNotFoundError:
            raise ReaderError(f"文件不存在: {self.source}")
        except Exception as e:
            raise ReaderError(f"读取CSV失败: {e}")

# src/pipeline.py
class ETLPipeline:
    """ETL管道"""
    
    def __init__(self, reader, transformer, writer):
        self.reader = reader
        self.transformer = transformer
        self.writer = writer
    
    def run(self):
        """执行ETL流程"""
        try:
            # 读取数据（生成器）
            data = self.reader.read()
            
            # 转换数据（生成器）
            transformed = self.transformer.transform(data)
            
            # 写入数据
            self.writer.write(transformed)
            
            print("ETL流程完成")
        
        except ETLError as e:
            print(f"ETL错误: {e}")
            raise
        except Exception as e:
            print(f"未知错误: {e}")
            raise
```

### Git工作流

```bash
# 1. 初始化
git init
git checkout -b main
git add README.md .gitignore
git commit -m "chore: 初始化项目"

# 2. 创建develop分支
git checkout -b develop

# 3. 开发readers
git checkout -b feature/readers
git add src/readers/
git commit -m "feat(readers): 实现CSV和JSON读取器"
git checkout develop
git merge feature/readers

# 4. 开发transformers
git checkout -b feature/transformers
git add src/transformers/
git commit -m "feat(transformers): 实现数据清洗器"
git checkout develop
git merge feature/transformers

# 5. 开发writers
git checkout -b feature/writers
git add src/writers/
git commit -m "feat(writers): 实现CSV写入器"
git checkout develop
git merge feature/writers

# 6. 整合pipeline
git checkout -b feature/pipeline
git add src/pipeline.py
git commit -m "feat: 实现ETL管道"
git checkout develop
git merge feature/pipeline

# 7. 添加测试
git add tests/
git commit -m "test: 添加单元测试"

# 8. 发布到main
git checkout main
git merge develop --no-ff
git tag -a v1.0.0 -m "Release version 1.0.0"
```

---

# 周日（Day 7）：复盘与总结

## 上午：完善项目（3小时）

1. **代码质量检查**
```bash
# 安装工具
pip install black flake8 mypy pytest pytest-cov

# 格式化
black src/

# 代码检查
flake8 src/

# 类型检查
mypy src/

# 测试覆盖率
pytest --cov=src tests/
```

2. **文档完善**
- 更新README
- 添加使用示例
- API文档

## 下午：学习总结（3小时）

### 创建Week 2总结

```markdown
# 第二周学习总结

## 本周学习内容

### Python高级特性
1. ✅ 迭代器协议：__iter__, __next__
2. ✅ 生成器：yield, 生成器表达式
3. ✅ 异常处理：try/except/else/finally
4. ✅ 自定义异常：异常层次
5. ✅ 上下文管理器：__enter__, __exit__

### Git进阶
1. ✅ git stash：暂存工作
2. ✅ git rebase：整理历史
3. ✅ git cherry-pick：选择性合并
4. ✅ Git工作流：GitHub Flow
5. ✅ 分支策略：feature/hotfix

## 重点概念

### 1. 生成器的优势
- 惰性计算：按需生成
- 内存高效：不一次性加载
- 适合处理大数据

### 2. 异常处理原则
- 具体异常先捕获
- 避免捕获所有异常
- finally用于清理资源
- 自定义异常增加语义

### 3. 上下文管理器
- 自动资源管理
- 保证清理代码执行
- @contextmanager简化实现

### 4. Git rebase vs merge
- rebase：线性历史，适合私有分支
- merge：保留历史，适合公共分支

## 项目总结

### ETL Pipeline
- **代码量**：800+行
- **测试覆盖率**：90%
- **Git提交**：25次
- **学到的**：
  - 生成器处理大文件
  - 完整的异常处理体系
  - Git分支管理策略

## 本周数据

- **学习时长**：31小时
- **代码行数**：1200+行
- **Git提交**：35次
- **完成练习**：15个

## 第1-2周总体回顾

### 完成的学习目标
- [x] Python面向对象编程
- [x] 装饰器
- [x] 生成器与迭代器
- [x] 异常处理
- [x] 上下文管理器
- [x] Git全流程

### 阶段产出
1. 任务管理系统
2. 日志处理系统
3. ETL Pipeline
4. GitHub仓库：ai-learning

## 下周计划

### Week 3-4重点
1. NumPy深入
2. 手写神经网络前向传播
3. 数学基础（线性代数）

### 需要复习的
- 装饰器的高级用法
- Git rebase的更多场景
```

---

# 第二周检验清单

## Python高级特性

**迭代器**
- [ ] 理解迭代器协议
- [ ] 能实现自定义迭代器
- [ ] 知道迭代器和可迭代对象的区别

**生成器**
- [ ] 理解yield的工作原理
- [ ] 能用生成器处理大文件
- [ ] 会写生成器表达式
- [ ] 理解send()和close()

**异常处理**
- [ ] 掌握try/except/else/finally
- [ ] 能设计异常层次
- [ ] 会使用raise和raise from
- [ ] 理解异常传播

**上下文管理器**
- [ ] 理解__enter__和__exit__
- [ ] 能用类实现上下文管理器
- [ ] 会用@contextmanager装饰器
- [ ] 知道常见使用场景

## Git进阶

**stash**
- [ ] 会暂存和恢复工作
- [ ] 理解stash的使用场景

**rebase**
- [ ] 理解rebase vs merge
- [ ] 会使用交互式rebase
- [ ] 知道何时不该用rebase

**cherry-pick**
- [ ] 会选择性应用提交
- [ ] 能处理cherry-pick冲突

**工作流**
- [ ] 理解GitHub Flow
- [ ] 能执行完整的开发流程
- [ ] 会处理各种冲突

## 项目产出

- [ ] 完成ETL Pipeline项目
- [ ] 单元测试覆盖率>80%
- [ ] 代码通过格式检查
- [ ] Git历史清晰
- [ ] 有完整的文档

---

# Week 3 预告

## 主要内容

### NumPy深入
1. 数组创建与索引
2. 广播机制
3. 矩阵运算
4. **手写神经网络前向传播**

### 数学基础
1. 线性代数：向量、矩阵
2. 矩阵运算：点积、转置
3. 神经网络数学基础

### 工程
1. Python工程化
2. 虚拟环境管理
3. 项目结构规范

---

**Week 2 完成！继续加油！🚀**

两周坚持下来，你已经：
- ✅ 掌握了Python核心特性
- ✅ 学会了专业的Git工作流
- ✅ 完成了3个实战项目
- ✅ 养成了良好的编码习惯

下一步：用NumPy手写神经网络，开始真正的AI之旅！
