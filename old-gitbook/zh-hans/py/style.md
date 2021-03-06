#### 风格规范

##### 行长度

+ 行宽 80 个字符
+ 不要用反斜杠连接行，括号内的元素会隐式连接
+ 长的导入模块语句和 URL 是例外

##### 分号

> 不要使用

##### 括号

+ 宁缺毋滥
+ 除非实际行连接，否则不能在返回语句或条件语句中使用括号，当然元组肯定需要

##### 缩进

四个空格

##### 空行

+ 顶级定义之间空两行
+ 方法定义之间空一行

##### 空格

+ 标点、二元操作符两边有空格
+ 括号内不需要
+ 索引、切片的左括号前不需要
+ 逗号、分号、冒号前面也不需要空格
+ 在关键字参数或默认参数时，`=` 两侧不要空格
+ 不要垂直对齐

##### 注释

> 确保模块、函数、方法和行内注释使用正确风格

+ 文档字符串：一句概述，空行，同缩进写详情
+ 模块：包含许可证样板
+ 函数和方法：必须有文档字符串，除非外部不可见，非常短小或足够简单
+ 类：应该有文档字符串，公共属性也要描述
+ 块注释和行注释：技巧性部分是最需要注释的
+ 行注释至少离开 2 个空格

##### 类

> 如果不是派生类，就显式继承 `object`，即使是嵌套类

这样做是为了属性可以正常工作，避免潜在兼容性影响，以及一些特殊方法和默认语义

```python
__new__, __init__, __delatter__, __getattribute__,
__setatter__, __hash__, __repr__, __str__
```

##### 字符串

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

##### 文件和 sockets

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

##### 导入格式

> 每个导入占一行

根据完整包路径按字典顺序排序，忽略大小写，分组顺序：

+ 标准库导入
+ 第三方导入
+ 应用程序指定导入

##### 语句

+ 通常每个语句占一行
+ 测试需要，if 语句可单独一行，但 `try`/`except` 绝对不行

##### 访问控制

+ 琐碎不太重要的访问函数可以直接用公有变量取代他们，避免额外函数调用开销
+ 通过属性添加更多功能保持语法一致性

##### 命名

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

##### main

> 即使被用作脚本，也应该是可导入的，并别不会产生副作用

应该总是检查

```python
def main():
    ...

if __name__ == '__main__':
    main()
```

