# Python 编程风格指南

## 目录

## 语言规范

### 导入

```python
import module-or-package
from package-prefix import non-prefix-module
from package-prefix import non-prefix-module as lite-name
```

+ 仅针对包和模块
+ 全路径导入包
+ 导入时不要使用相对名称，避免重复导入

### 异常

+ 不要使用两个参数的形式
+ 不要使用过时的字符串异常
+ 不要用 `except:` 捕获所有异常，也不要捕获 `Exception` 或 `StandardError`
+ 使用 `finally` 子句清理资源
+ 使用 `as`， 即 `except Error as error:`
+ 尽量减少 `try`/`except` 块中的代码量
+ 包或模块应定义特定域的异常基类 `class Error(Exception): pass`

### 全局变量

避免使用，除非

+ 脚本默认选项
+ 模块级常量
+ 缓存或作为返回值使用
+ 仅在模块内部可用，并通过模块级公共函数访问

### 使用默认迭代器和操作符

> 简单、通用、高效，没有额外方法调用，但无法通过方法名区分对象类型

例如

```python
for key in adict: ...
if key not in adict: ...
if obj in alist: ...
for line in afile: ...
for k, v in dict.iteritems(): ...
```

而不是

```python
for key in adict.keys(): ...
if not adict.has_key(key): ...
for line in afile.readline(): ...
```

### 生成器

> 执行一次返回一个迭代器，立刻被挂起，直到下次生成

+ 简化代码
+ 局部变量和控制流的状态都被保存
+ 占内存少
+ 文档字符串使用 `Yields:` 而非 `Returns:`

### Lambda

适用于单行函数

### 条件表达式

适用于单行函数，其他情况推荐完整的 if 语句

```python
x = 1 if conf else 2
```

### 默认参数值

+ 推荐使用，不要为罕见的例外定义大量函数
+ 不支持重载，倒是可以伪造重载行为
+ 默认参数只在模块加载时求值一次，避免使用列表或字典这类可变类型

比如

```python
def foo(a, b=None):
    if b is None:
        b = []
```

而不是

```python
def foo(a, b=[]):
def foo(a, b=time.time()):
def foo(a, b=FLAGS.my_things):
```

### 属性

> 代替简单，轻量级的访问和设置函数

+ 获取和设置属性的标准方法，提高可读性
+ 需要修饰器说明，必须继承 `object` 类，隐藏运算符重载副作用

例如

```python
import math

class Square(object):
    def __init__(self, side):
        self.side = side

    def __get_area(self):
        return self.side ** 2;

    def __set_area(self, area):
        self.side = math.sqrt(area)

    def ___get_area(self):
        return self.__get_area()

    def ___set_area(self, area):
        self._SetArea(area)

    area = property(___get_area, ___set_area,
                    doc="""Gets or sets the area""")

    @property
    def perimeter(self):
        return self.side * 4
```

### True/False 求值

> 尽可能使用隐式 `false`

虽然 `0`, `None`，`[]`，`{}`，`""` 都被认为是 `false`

+ 不要用 `==` 或 `!=` 比较，针对 `None` 要用 `is` 和 `is not`
+ `if x:` 表示的是 `if x is not None:`，如果测试变量设为其他值，表达式能是假
+ 不要用 `==` 与 `false` 比较，使用 `if not x` 与 `x is not None`
+ 空序列是 `false`，所以 `if not seq:` 比 `len(seq)` 好很多
+ 处理整数时可能会将 `None` 当作 `0` 处理，可将已知是整数的值与 `0` 直接比较
+ 字符串 `'0'` 是 `true`

### 过时的语言特性

> 使用字符串方法取代字符串模块，即 `apply()`，`filter()`，`map()` 和 `reduce()`

比如

```python
words = foo.split(':')

[x[1] for x in my_list if x[2] == 5]

map(math.sqrt, data)

fn(*args, **kwargs)
```

而不是

```python
words = strings.split(foo, ':')

map(lambda x: x[1], filter(lambda x: x[2] == 5, my_list))

apply(fn, args, kwargs)
```

### 词法作用域

+ 嵌套的函数可以引用外层定义的变量，但不能赋值
+ 块中某个名称的赋值会导致其全部引用当作局部变量，甚至是赋值前的处理

比如

```python
def get_addr(summand1):
    def addr(summand2):
        return summand1 + summand2

    return addr

sum = get_addr(summand1)(summand2)
```

通常会带来更加清晰、优雅的代码，但也会让人迷惑

```python
i = 4
def foo(x):
    def bar():
        # 会当成局部变量哦
        print i;

    for i in x:
        print i,

    bar()

foo([1, 2, 3])

# 输出的并不是 1 2 3 4
1 2 3 3
```

### 函数与方法修饰器

常见的是 `@classmethod` 和 `@staticmethod`，也允许自定义

```python
class C(object):
    @my_decorator
    def method(self):
        # balabala
```

等效于

```python
class C(object):
    def method(self):
        # balabala
    method = my_decorator(method)
```

+ 优雅的在函数上指定一些转换，减少重复代码并保持已有函数不变
+ 可在函数参数和返回值上执行任何操作，无法恢复导入执行时装饰器代码的失败
+ 避免对外界依赖，确保一个参数调用的装饰器在所有情况下都是成功的

### 线程

+ 不要依赖内建的原子性
+ 优先使用队列模块的 `Queue` 数据类型作为线程间的通信方式
+ 使用 `threading` 模块以及锁原语
+ 使用 `threading.Condition` 取代低级锁

### 鼓励使用

+ 嵌套/局部/内部类或函数
+ 简单情况下使用列表推导和生成器表达式，不必借助 `map()`，`filter()` 或 `lambda`

### 威力过大的特性

避免使用：

+ 元类
+ 字节码访问
+ 任意编译
+ 动态继承
+ 对象父类重定义
+ 导入黑客
+ 反射
+ 系统内修改

### pylint

捕获容易忽视的错误，有时需要围绕着它写代码

通过注释抑制不想看到的警告

```python
# pylint: disable=redefined-builtin
dict = 'something awful'
```

通过 `_` 作为参数标识符，或 `unused_` 来消除未使用参数的警告

```python
def foo(a, unused_b, unused_c, d=None, e=None):
    _ = d, e
    return a
```

查看全部警告列表

```shell
pylint --list-msgs
```

查看某个具体信息

```shell
pylint --help-msg=Wxxx
```

## 风格规范

### 行长度

+ 行宽 80 个字符
+ 不要用反斜杠连接行，括号内的元素会隐式连接
+ 长的导入模块语句和 URL 是例外

### 分号

> 不要使用

### 括号

+ 宁缺毋滥
+ 除非实际行连接，否则不能在返回语句或条件语句中使用括号，当然元组肯定需要

### 缩进

四个空格

### 空行

+ 顶级定义之间空两行
+ 方法定义之间空一行

### 空格

+ 标点、二元操作符两边有空格
+ 括号内不需要
+ 索引、切片的左括号前不需要
+ 逗号、分号、冒号前面也不需要空格
+ 在关键字参数或默认参数时，`=` 两侧不要空格
+ 不要垂直对齐

### 注释

> 确保模块、函数、方法和行内注释使用正确风格

+ 文档字符串：一句概述，空行，同缩进写详情
+ 模块：包含许可证样板
+ 函数和方法：必须有文档字符串，除非外部不可见，非常短小或足够简单
+ 类：应该有文档字符串，公共属性也要描述
+ 块注释和行注释：技巧性部分是最需要注释的
+ 行注释至少离开 2 个空格

### 类

> 如果不是派生类，就显式继承 `object`，即使是嵌套类

这样做是为了属性可以正常工作，避免潜在兼容性影响，以及一些特殊方法和默认语义

```python
__new__, __init__, __delatter__, __getattribute__,
__setatter__, __hash__, __repr__, __str__
```

### 字符串

> 合理使用格式化字符串和加号

比如

```python
x = a + b
x = '%s, %s!' % (a, b)
x = '{}, {}!'.format(a, b)
```

而不是

```python
x = '%s%s' % (a, b)
x = 'name: ' + name + '; score: ' + str(n)
```

+ 避免在循环中使用 `+` 或者 `+=`，因为字符串是不可变的
+ 可通过添加列表之后再 `.join` 或者写入 `cStringIO.StringIO` 缓存
+ 单双引号保持一致性，多行的问题，可交由括号实现，保持缩进美观
+ 多行文档字符串使用的是三重双引号，通常使用隐式行链接，避免缩进问题
+ 三重单引号仅用于非文档字符串的多行字符串标识引用

### 文件和 sockets

> 结束时要显式的关闭它

除文件外，在没必要的情况下打开会产生许多副作用

+ 消耗有限的系统资源，比如文件描述符
+ 持有文件将阻止诸如移动、删除之类的操作
+ 逻辑上关闭他们后，可能仍会被共享的程序读写

企图通过析构实现自动关闭也是不现实的

+ 没有方法确保运行环境真正析构了，垃圾处理未必相同
+ 对文件意外的引用，可能会超出预期的持有时间

可通过 `with` 管理文件

```python
with open("hello.txt") as hello_file:
    for line in hello_file:
        print line
```

或者使用 `contextlib.closing()`

```python
import contextlib

with contextlib.closing(urllib.urlopen("http://www.python.org")) as page:
    for line in page:
        print line
```

### 导入格式

> 每个导入占一行

根据完整包路径按字典顺序排序，忽略大小写，分组顺序：

+ 标准库导入
+ 第三方导入
+ 应用程序指定导入

### 语句

+ 通常每个语句占一行
+ 测试需要，if 语句可单独一行，但 `try`/`except` 绝对不行

### 访问控制

+ 琐碎不太重要的访问函数可以直接用公有变量取代他们，避免额外函数调用开销
+ 通过属性添加更多功能保持语法一致性

### 命名

Guido 推荐的名字

```python
module_name
package_name
method_name
function_name
instance_var_name
function_parameter_name
local_var_name

ClassName
ExceptionName

GLOBAL_VAR_NAME
```

应该避免

+ 单字符名称，除了计算器和迭代器
+ 包名和模块名的连字符 `-`
+ 双下划线开头并结尾的名称，比如 `__init__`

命名约定

+ 内部：仅模块内部可用或类内保护或私有的
+ 保护的模块变量或函数添加 `_` 前缀
+ 私有的变量或方法添加 `__` 前缀
+ 大写首字母当类名
+ 小写加下划线做模块名

### main

> 即使被用作脚本，也应该是可导入的，并别不会产生副作用

应该总是检查

```python
def main():
    ...

if __name__ == '__main__':
    main()
```

