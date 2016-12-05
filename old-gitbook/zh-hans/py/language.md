#### 语言规范

##### 导入

```python
import module-or-package
from package-prefix import non-prefix-module
from package-prefix import non-prefix-module as lite-name
```

+ 仅针对包和模块
+ 全路径导入包
+ 导入时不要使用相对名称，避免重复导入

##### 异常

+ 不要使用两个参数的形式
+ 不要使用过时的字符串异常
+ 不要用 `except:` 捕获所有异常，也不要捕获 `Exception` 或 `StandardError`
+ 使用 `finally` 子句清理资源
+ 使用 `as`， 即 `except Error as error:`
+ 尽量减少 `try`/`except` 块中的代码量
+ 包或模块应定义特定域的异常基类 `class Error(Exception): pass`

##### 全局变量

避免使用，除非

+ 脚本默认选项
+ 模块级常量
+ 缓存或作为返回值使用
+ 仅在模块内部可用，并通过模块级公共函数访问

##### 使用默认迭代器和操作符

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

##### 生成器

> 执行一次返回一个迭代器，立刻被挂起，直到下次生成

+ 简化代码
+ 局部变量和控制流的状态都被保存
+ 占内存少
+ 文档字符串使用 `Yields:` 而非 `Returns:`

##### Lambda

适用于单行函数

##### 条件表达式

适用于单行函数，其他情况推荐完整的 if 语句

```python
x = 1 if conf else 2
```

##### 默认参数值

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

##### 属性

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

##### True/False 求值

> 尽可能使用隐式 `false`

虽然 `0`, `None`，`[]`，`{}`，`""` 都被认为是 `false`

+ 不要用 `==` 或 `!=` 比较，针对 `None` 要用 `is` 和 `is not`
+ `if x:` 表示的是 `if x is not None:`，如果测试变量设为其他值，表达式能是假
+ 不要用 `==` 与 `false` 比较，使用 `if not x` 与 `x is not None`
+ 空序列是 `false`，所以 `if not seq:` 比 `len(seq)` 好很多
+ 处理整数时可能会将 `None` 当作 `0` 处理，可将已知是整数的值与 `0` 直接比较
+ 字符串 `'0'` 是 `true`

##### 过时的语言特性

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

##### 词法作用域

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

##### 函数与方法修饰器

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

##### 线程

+ 不要依赖内建的原子性
+ 优先使用队列模块的 `Queue` 数据类型作为线程间的通信方式
+ 使用 `threading` 模块以及锁原语
+ 使用 `threading.Condition` 取代低级锁

##### 鼓励使用

+ 嵌套/局部/内部类或函数
+ 简单情况下使用列表推导和生成器表达式，不必借助 `map()`，`filter()` 或 `lambda`

##### 威力过大的特性

避免使用：

+ 元类
+ 字节码访问
+ 任意编译
+ 动态继承
+ 对象父类重定义
+ 导入黑客
+ 反射
+ 系统内修改

##### pylint

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

