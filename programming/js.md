# JavaScript Google 开发规范指南

## 目录
## 语言规范

### `var` 关键字

总是用 `var` 关键字定义变量。

#### 描述

如果不显式使用 `var` 关键字定义变量，变量会进入到全局上下文中，可能会和已有的变量发生冲突。另外，如果不使用var声明，很难说变量存在的作用域是哪个（可能在局部作用域里，也可能在 document 或者 window 上）。所以，要一直使用 `var` 关键字定义变量。

### 常量

+ 使用字母全部大写（如 `NAMES_LIKE_THIS` ）的方式命名
+ 可以使用 `@const` 来标记一个常量 **指针**（指向变量或属性，自身不可变）
+ 由于IE的兼容问题，不要使用 `const` 关键字

#### 常量值

如果一个值是恒定的，它命名中的字母要全部大写（如 `CONSTANT_VALUE_CASE` ）。字母全部大写意味着这个值不可以被改写。

原始类型（ `number` 、 `string` 、 `boolean` ）是常量值。

对象的表现会更主观一些，当它们没有暴露出变化的时候，应该认为它们是常量。但是这个不是由编译器决定的。

#### 常量指针（变量和属性）

用 `@const` 注释的变量和属性意味着它是不能更改的。使用 `const` 关键字可以保证在编译的时候保持一致。使用 `const` 效果相同，但是由于 IE 的兼容问题，我们不使用 `const` 关键字。

另外，不应该修改用 `@const` 注释的方法。

#### 例子

注意， `@const` 不一定是常量值，但命名类似 `CONSTANT_VALUE_CASE` 的 **一定** 是常量指针。

```js
/**
* 以毫秒为单位的超时时长
* @type {number}
*/
goog.example.TIMEOUT_IN_MILLISECONDS = 60;
```

1 分钟 60 秒永远也不会改变，这是个常量。全部大写的命名意味其为常量值，所以它不能被重写。
开源的编译器允许这个符号被重写，这是因为 **没有** `@const` 标记。


```js
/**
* Map of URL to response string.
* @const
*/
MyClass.fetchedUrlCache_ = new goog.structs.Map();
```

在这个例子中，指针没有变过，但是值却是可以变化的，所以这里用了驼峰式的命名，而不是全部大写的命名。

### 分号

一定要使用分号。

依靠语句间隐式的分割，可能会造成细微的调试的问题，千万不要这样做。

很多时候不写分号是很危险的：


```js
// 1.
MyClass.prototype.myMethod = function() {
    return 42;
}  // 这里没有分号.

(function() {
    // 一些局部作用域中的初始化代码
})();

var x = {
    'i': 1,
    'j': 2
}  //没有分号.

// 2.  试着在IE和firefox下做一样的事情.
//没人会这样写代码，别管他.
[normalVersion, ffVersion][isIE]();

var THINGS_TO_EAT = [apples, oysters, sprayOnCheese]  //这里没有分号

// 3. 条件语句
-1 == resultOfOperation() || die();
```

#### 发生了什么？

1. js 错误。返回 42 的函数运行了，因为后面有一对括号，而且传入的参数是一个方法，然后返回的 42 被调用，导致出错了。

2. 你可能会得到一个 `no sush property in undefined` 的错误，因为在执行的时候，解释器将会尝试执行 `x[normalVersion, ffVersion][isIE]()` 这个方法。

3. `die` 这个方法只有在 `resultOfOperation()` 是 `NaN` 的时候执行，并且 `THINGS_TO_EAT` 将会被赋值为 `die()` 的结果。

#### 为什么？

js 语句要求以分号结尾，除非能够正确地推断分号的位置。在这个例子当中，函数声明、对象和数组字面量被写在了一个语句当中。右括号（`)`、`}`、`]`）不足以证明这条语句已经结束了，如果下一个字符是运算符或者`(`、`{`、`[`，js 将不会结束语句。

这个错误让人震惊，所以一定要确保用分号结束语句。

#### 澄清：分号和函数

函数表达式后面要分号结束，但是函数声明就不需要。例如：

```js
var foo = function() {
    return true;
}; // 这里要分号

function foo() {
    return true;
} // 这里不用分号
```

### 嵌套函数

可以使用。

嵌套函数非常有用，比如在创建持续任务或者隐藏工具方法的时候。可以放心的使用。

### 块内函数声明

不要使用块内函数声明。

不要这样做：

```js
if (x) {
    function foo() {}
}
```

虽然大多数脚本引擎支持功能区块内声明，但 ECMAScript 并未认可。若与他人的及 EcmaScript 所建议的不一致，即可视为不好的实现方式。ECMAScript 只允许函数声明语句列表, 在根语句列表脚本或者函数。相反，使用一个变量初始化函数表达式在块内定义一个函数块

```js
if (x) {
    var foo = function() {}
}
```

### 异常

可以抛出异常。

如果你做一些比较复杂的项目你基本上无法避免异常，比如使用一个应用程序开发框架。可以大胆试一试。

### 自定义异常

可以自定义异常。

如果没有自定义异常，返回的错误信息来自一个有返回值的函数是难处理的，是不雅的。坏的解决方案包括传递引用的类型来保存错误信息或总是返回有一个潜在的错误成员的对象。这些基本上为原始的异常处理 hack。在适当的时候使用自定义的异常。

### 标准功能

总是优先于非标准功能。

为了最大的可移植性和兼容性，总是使用标准功能而不是非标准功能（例如，采用 `string.charAt(3)` 而非 `string[3]` ，用DOM的功能访问元素而不是使用特定于一个具体应用的简写）。

### 原始类型的包装对象

没有理由使用原始类型的包装对象，更何况他们是危险的：

```js
var x = new Boolean(false);
if (x) {
    // 显示 hi
    alert('hi');
}
```

不要这样做！

然而类型转换是可以的。

```js
var x = Boolean(0);

if (x) {
    // 永远都不显示。
    alert('hi');
}

typeof Boolean(0) == 'boolean';
typeof new Boolean(0) == 'object';
```

这是非常有用的进行数字、字符串和布尔值转换的方式。

### 多重的原型继承

不可取。

多重原型继承是 JavaScript 实现继承的方式。如果你有一个以用户定义的 class B 作为原型的用户自定义 class D，则得到多重原型继承。这样的继承出现容易但难以正确创造！

出于这个原因，最好是使用 Closure库中的 `goog.inherits()` 或类似的东西。

```js
function D() {
    goog.base(this)
}
goog.inherits( D, B );

D.prototype.method =function() {
    // ...
};
```

### 方法和属性定义

```js
// 构造函数
function SomeConstructor() {
    this.someProperty = 1;
}

Foo.prototype.someMethod = function() {
    // ...
};
```

虽然有多种使用 `new` 关键词来创建对象方法和属性的途径，首选的创建方法的途径是：

```js
Foo.prototype.bar = function() {
    // ...
};
```

其他特性的首选创建方式是在构造函数中初始化字段：

```js
// 构造
function Foo() {
    this.bar = value;
}
```

#### 为什么？

当前的 JavaScript 引擎优化基于一个对象的“形状”，给对象添加一个属性（包括覆盖原型设置的值）改变了形式，会降低性能。

### 删除

请使用 `this.foo = null` 。

```js
o.prototype.dispose = function() {
    this.property_ = null;
};
```

而不是：

```js
Foo.prototype.dispose = function() {
    delete his.property_;
};
```

在现代的 JavaScript 引擎中，改变一个对象属性的数量比重新分配值慢得多。应该避免删除关键字，除非有必要从一个对象的迭代的关键字列表删除一个属性，或改变 `if (key in obj)` 结果。

### 闭包

可以使用，但是要小心。

创建闭包可能是JS最有用的和经常被忽视的功能。在[这里][1]很好地描述说明了闭包的工作。

[1]: http://jibbering.com/faq/notes/closures/

要记住的一件事情，一个闭包的指针指向包含它的范围。因此，附加一个闭包的 DOM 元素，可以创建一个循环引用，所以，内存会泄漏。例如，下面的代码：

```js
function foo(element, a, b) {
    element.onclick = function() {
        /* 使用 a 和 b */
    };
}
```

闭包能保持元素 a 和 b 的引用即使它从未使用。因为元素还保持对闭包的一个引用，我们有一个循环引用，不会被垃圾收集清理。在这些情况下，代码的结构可以如下：

```js
function foo(element, a, b) {
    element.onclick = bar(a, b);
}

function bar(a, b) {
    return function() {
        /* 使用 a 和 b */
    }
}
```

### eval()函数

只用于反序列化（如评估 RPC 响应）。

若用于 `eval()` 的字符串含有用户输入，则 `eval()` 会造成混乱的语义，使用它有风险。通常有一个更好
更清晰、更安全的方式来编写你的代码，所以一般是不会允许其使用的。然而，`eval` 相对比非 `eval` 使反序列化更容易，因此它的使用是可以接受的（例如评估 RPC 响应）。

反序列化是将一系列字节存到内存中的数据结构转化过程。例如，你可能会写的对象是：

```js
users = [
    {
        name: 'Eric',
        id: 37824,
        email: 'jellyvore@myway.com'
    },
    {
        name: 'xtof',
        id: 31337,
        email: 'b4d455h4x0r@google.com'
    },
    // ...
];
```

将这些数据读入内存跟得出文件的字符串表示形式一样容易。

同样， `eval()` 函数可以简化解码RPC的返回值。例如，您可以使用 `XMLHttpRequest` 生成 RPC，在响应时服务器返回 JavaScript：

```js
var userOnline = false;
var user = 'nusrat';
var xmlhttp = new XMLHttpRequest();

xmlhttp.open('GET', 'http://chat.google.com/isUserOnline?user=' + user, false);
xmlhttp.send('');

// 服务器返回：userOnline = true;
if (xmlhttp.status == 200) {
  eval(xmlhttp.responseText);
}
// userOnline 现在为 true
```

### `with() {}`

不建议使用。

使用 `with` 会影响程序的语义。因为 `with` 的目标对象可能会含有和局部变量冲突的属性，使你程序的语义发生很大的变化。例如，这是做什么用？

```js
with (foo) {
    var x = 3;
    return x;
}
```

答案：什么都有可能。局部变量 `x` 可能会被 `foo` 的一个属性覆盖，它甚至可能有 `setter` 方法，在此情况下将其赋值为 3 可能会执行很多其他代码。不要使用 `with` 。

### `this`

只在构造函数对象、方法，和创建闭包的时候使用。

`this` 的语义可能会非常诡异。有时它指向全局对象（很多时候）、调用者的作用域链（在 `eval` 里）、DOM 树的一个节点（当使用HTML属性来做为事件句柄时）、新创建的对象（在一个构造函数中）、或者其他的对象（如果函数被 `call()` 或 `apply()` 方式调用）。

正因为 `this` 很容易被弄错，故将其使用限制在以下必须的地方：

+ 在构造函数中
+ 在对象的方法中（包括闭包的创建）

### `for-in` 循环

只使用在对象、映射、哈希的键值迭代中。

`for-in` 循环经常被不正确的用在元素数组的循环中。由于并不是从 `0` 到 `length - 1` 进行循环，而是遍历对象中和它原型链上的所有的键，所以很容易出错。这里有一些失败的例子：

```js
function printArray(arr) {
    for (var key in arr) {
        print(arr[key]);
    }
}

// 这样可以
printArray([0,1,2,3]);

var a = new Array(10);
// 这样不行
printArray(a);

a = document.getElementsByTagName('*');
// 这样不行
printArray(a);

a = [0,1,2,3];
a.buhu = 'wine';
// 这样不行
printArray(a);

a = new Array;
a[3] = 3;
// 这样不行
printArray(a);
```

在数组循环时常用的一般方式：

```js
function printArray(arr) {
    var l = arr.length;
    for (var i = 0; i < l; i++) {
        print(arr[i]);
    }
}
```

### 关联数组

不要将映射，哈希，关联数组当作一般数组来使用。

不允许使用关联数组……确切的说在数组，你不可以使用非数字的索引。如果你需要一个映射或者哈希，在这种情况下你应该使用对象来代替数组，因为在功能上你真正需要的是对象的特性而不是数组的。

数组仅仅是用来拓展对象的（像在JS中你曾经使用过的 `Date` 、 `RegExp` 和 `String` 对象一样的）。

### 多行的字符串字面量

不要使用。

不要这样做：

```js
var myString = 'A rather long string of English text, an error message \
              actually that just keeps going and going -- an error \
              message to make the Energizer bunny blush (right through \
              those Schwarzenegger shades)! Where was I? Oh yes, \
              you\'ve got an error and all the extraneous whitespace is \
              just gravy.  Have a nice day.';
```

在编译时每一行头部的空白符不会被安全地去除掉；斜线后的空格也会导致棘手的问题；虽然大部分脚本引擎都会支持，但是它不是ECMAScript规范的一部分。

使用字符串连接来代替：

```js
var myString = 'A rather long string of English text, an error message ' +
   'actually that just keeps going and going -- an error ' +
   'message to make the Energizer bunny blush (right through ' +
   'those Schwarzenegger shades)! Where was I? Oh yes, ' +
   'you\'ve got an error and all the extraneous whitespace is ' +
   'just gravy.  Have a nice day.';
```

### 数组和对象字面量

建议使用。

使用数组和对象字面量来代替数组和对象构造函数。

数组构造函数容易在参数上出错。

```js
// 长度为3
var a1 = new Array(x1, x2, x3);

// 长度为 2
var a2 = new Array(x1, x2);

// 如果 x1 是自然数，长度就是 x1
// 如果 x1 是非自然数的数值，会抛异常
// 其他情况都是创建只有一个元素的数组，里面是 x1 的数值
var a3 = new Array(x1);

// 长度为0
var a4 = new Array();
```

由此，如果有人将代码从2个参数变成了一个参数，那么这个数组就会有一个错误的长度。

为了避免这种怪异的情况，永远使用可读性更好的数组字面量。

```js
var a = [x1, x2, x3];
var a2 = [x1, x2];
var a3 = [x1];
var a4 = [];
```

对象构造函数虽然没有相同的问题，但是对于可读性和一致性，还是应该使用对象字面量。

```js
var o = new Object();

var o2 = new Object();
o2.a = 0;
o2.b = 1;
o2.c = 2;
o2['strange key'] = 3;
```

应该写成：

```js
var o = {};

var o2 = {
    a: 0,
    b: 1,
    c: 2,
    'strange key': 3
};
```

### 修改内置对象原型

不建议。

强烈禁止修改如 `Object.prototype` 和 `Array.prototype` 等对象的原型。修改其他内置原型如 `Function.prototype` 危险性较小，但在生产环境中还是会引发一些难以调试的问题，也应当避免。

### Internet Explorer 中的条件注释

不要使用。

不要这样做：

```js
var f = function () {
    /*@cc_on if (@_jscript) { return 2* @*/  3; /*@ } @*/
};
```

条件注释会在运行时改变 JavaScript 语法树，阻碍自动化工具。

## 格式规则

### 命名

通常来说，使用 `functionNamesLikeThis` ，`variableNamesLikeThis` ，`ClassNamesLikeThis` ，`EnumNamesLikeThis` ，`methodNamesLikeThis` ，`CONSTANT_VALUES_LIKE_THIS` ，`foo.namespaceNamesLikeThis.bar` 和 `filenameslikethis.js` 这种格式的命名方式。

#### 属性和方法

+ **私有** 属性和方法应该以下划线开头命名。
+ **保护** 属性和方法应该以无下划线开头命名（像公共属性和方法一样）。

#### 方法和函数参数

可选函数参数以 `opt_` 开头。

参数数目可变的函数应该具有以 `var_args` 命名的最后一个参数。你可能不会在代码里引用 `var_args`；使用 `arguments` 对象。

可选参数和数目可变的参数也可以在注释 `@param` 中指定。尽管这两种惯例都被编译器接受，但更加推荐两者一起使用。

#### `getter` 和 `setter`

EcmaScript 5 不鼓励使用属性的 `getter` 和 `setter`。然而，如果使用它们，那么 `getter` 就不要改变属性的可见状态。

```js
// 错误，不要这样做。
var foo = { get next() { return this.nextId++; } };
```

#### 存取函数

属性的 `getter`和 `setter` 方法不是必需的。然而，如果使用它们，那么读取方法必须以 `getFoo()` 命名，并且写入方法必须以 `setFoo(value)` 命名。（对于布尔型的读取方法，也可以使用 `isFoo()` ，并且这样往往听起来更自然。）

#### 命名空间

JavaScript 没有原生的对封装和命名空间的支持。

全局命名冲突难以调试，并且当两个项目尝试整合的时候可能引起棘手的问题。为了能共享共用的 JavaScript 代码，我们采用一些约定来避免冲突。

##### 为全局代码使用命名空间

在全局范围内 **总是** 使用唯一的项目或库相关的伪命名空间进行前缀标识。如果你正在进行 Project Sloth 项目，一个合理的伪命名空间为 `sloth.*` 。

```js
var sloth = {};

sloth.sleep = function() {
   // ...
};
```

很多 JavaScript 库，包括 Closure Library 和 Dojo toolkit 给你高级功能来声明命名空间。保持你的命名空间声明一致。

```js
goog.provide('sloth');

sloth.sleep = function() {
   // ...
};
```

##### 尊重命名空间所有权

当选择一个子命名空间的时候，确保父命名空间知道你在做什么。如果你开始了一个为 sloths 创建 hats 的项目，确保 Sloth 这一组命名空间知道你在使用 `sloth.hats` 。

##### 外部代码和内部代码使用不同的命名空间

“外部代码” 指的是来自你的代码库外并独立编译的代码。内部名称和外部名称应该严格区分开。如果你正在使用一个能调用 `foo.hats.*` 中的东西的外部库，你的内部代码不应该定义 `foo.hats.*` 中的所有符号，因为如果其他团队定义新符号就会把它打破。

```js
foo.require('foo.hats');

/**
*错误，不要这样做。
* @constructor
* @extends {foo.hats.RoundHat}
*/
foo.hats.BowlerHat = function() {
};
```

如果你在外部命名空间中需要定义新的 API，那么你应该明确地导出且仅导出公共的 API 函数。为了一致性和编译器更好的优化你的内部代码，你的内部代码应该使用内部 API 的内部名称调用它们。

```js
foo.provide('googleyhats.BowlerHat');

foo.require('foo.hats');

/**
* @constructor
* @extends {foo.hats.RoundHat}
*/
googleyhats.BowlerHat = function() {
   // ...
};
goog.exportSymbol('foo.hats.BowlerHat', googleyhats.BowlerHat);
```

##### 为长类型的名称提供别名提高可读性

如果对完全合格的类型使用本地别名可以提高可读性，那么就这样做。本地别名的名称应该符合类型的最后一部分。

```js
/**
* @constructor
*/
some.long.namespace.MyClass = function() {
};

/**
* @param {some.long.namespace.MyClass} a
*/
some.long.namespace.MyClass.staticHelper = function(a) {
   // ...
};

myapp.main = function() {
   var MyClass = some.long.namespace.MyClass;
   var staticHelper = some.long.namespace.MyClass.staticHelper;
   staticHelper(new MyClass());
};
```

不要为命名空间起本地别名。命名空间应该只能使用 `goog.scope` 命名别名。

```js
myapp.main = function() {
   var namespace = some.long.namespace;
   namespace.MyClass.staticHelper(new namespace.MyClass());
};
```

避免访问一个别名类型的属性，除非它是一个枚举。

```js
/** @enum {string} */
some.long.namespace.Fruit = {
   APPLE: 'a',
   BANANA: 'b'
};

myapp.main = function() {
var Fruit = some.long.namespace.Fruit;
switch (fruit) {
  case Fruit.APPLE:
    // ...
  case Fruit.BANANA:
    // ...
}
};

myapp.main = function() {
   var MyClass = some.long.namespace.MyClass;
   MyClass.staticHelper(null);
};
```

永远不要在全局环境中创建别名。只在函数体内使用它们。

#### 文件名

为了避免在大小写敏感的平台上引起混淆，文件名应该小写。文件名应该以 `.js` 结尾，并且应该不包含除了 `-` 或 `_` （相比较 `_` 更推荐 `-` ）以外的其它标点。

### 自定义 toString() 方法

必须确保无误，并且无其他副作用。

你可以通过自定义 `toString()` 方法来控制对象如何字符串化他们自己。这没问题，但是你必须确保你的方法执行无误，并且无其他副作用。如果你的方法没有达到这个要求，就会很容易产生严重的问题。比如，如果 `toString()` 方法调用一个方法产生一个断言，断言可能要输出对象的名称，就又需要调用 `toString()` 方法。

### 延时初始化

可以使用。

并不总在变量声明的地方就进行变量初始化，所以延时初始化是可行的。

### 明确作用域

时常。

经常使用明确的作用域加强可移植性和清晰度。例如，在作用域链中不要依赖 `window` 。你可能想在其他应用中使用你的函数，这时此 `window` 就非彼 `window` 了。

### 代码格式

我们原则上遵循 C++ 格式规范，并且进行以下额外的说明。

#### 大括号

由于隐含分号的插入，无论大括号括起来的是什么，总是在同一行上开始你的大括号。例如：

```js
if (something) {
   // ...
} else {
   // ...
}
```

#### 数组和对象初始化表达式

当单行数组和对象初始化表达式可以在一行写开时，写成单行是允许的。

```js
// 之后无空格[或之前]
var arr = [1, 2, 3];
// 之后无空格[或之前]
var obj = {a: 1, b: 2, c: 3};
```

多行数组和对象初始化表达式缩进两个空格，括号的处理就像块一样单独成行。

```js
// 对象初始化表达式
var inset = {
   top: 10,
   right: 20,
   bottom: 15,
   left: 12
};

// 数组初始化表达式
this.rows_ = [
   '"Slartibartfast" <fjordmaster@magrathea.com>',
   '"Zaphod Beeblebrox" <theprez@universe.gov>',
   '"Ford Prefect" <ford@theguide.com>',
   '"Arthur Dent" <has.no.tea@gmail.com>',
   '"Marvin the Paranoid Android" <marv@googlemail.com>',
   'the.mice@magrathea.com'
];

// 在方法调用中使用
goog.dom.createDom(goog.dom.TagName.DIV, {
   id: 'foo',
   className: 'some-css-class',
   style: 'display:none'
}, 'Hello, world!');
```

长标识符或值在对齐的初始化列表中存在问题，所以初始化值不必对齐。例如：

```js
CORRECT_Object.prototype = {
   a: 0,
   b: 1,
   lengthyName: 2
};
```

不要像这样：

```js
WRONG_Object.prototype = {
   a          : 0,
   b          : 1,
   lengthyName: 2
};
```

#### 函数参数

如果可能，应该在同一行上列出所有函数参数。如果这样做将超出每行 80 个字符的限制，参数必须以一种可读性较好的方式进行换行。为了节省空间，在每一行你可以尽可能的接近 80 个字符，或者把每一个参数单独放在一行以提高可读性。缩进可能是四个空格，或者和括号对齐。下面是最常见的参数换行形式：

```js
// 四个空格，每行包括80个字符。适用于非常长的函数名，
// 重命名不需要重新缩进，占用空间小。
goog.foo.bar.doThingThatIsVeryDifficultToExplain = function(
   veryDescriptiveArgumentNumberOne, veryDescriptiveArgumentTwo,
   tableModelEventHandlerProxy, artichokeDescriptorAdapterIterator) {
   // ...
};

//四个空格，每行一个参数。适用于长函数名，
// 允许重命名，并且强调每一个参数。
goog.foo.bar.doThingThatIsVeryDifficultToExplain = function(
   veryDescriptiveArgumentNumberOne,
   veryDescriptiveArgumentTwo,
   tableModelEventHandlerProxy,
   artichokeDescriptorAdapterIterator) {
   // ...
};

// 缩进和括号对齐，每行80字符。 看上去是分组的参数，
// 占用空间小。
function foo(veryDescriptiveArgumentNumberOne, veryDescriptiveArgumentTwo,
          tableModelEventHandlerProxy, artichokeDescriptorAdapterIterator) {
   // ...
}

// 和括号对齐，每行一个参数。看上去是分组的并且
// 强调每个单独的参数。
function bar(veryDescriptiveArgumentNumberOne,
          veryDescriptiveArgumentTwo,
          tableModelEventHandlerProxy,
          artichokeDescriptorAdapterIterator) {
   // ...
}
```

当函数调用本身缩进，你可以自由地开始相对于原始声明的开头或者相对于当前函数调用的开头，进行4个空格的缩进。以下都是可接受的缩进风格。

```js
if (veryLongFunctionNameA(
      veryLongArgumentName) ||
   veryLongFunctionNameB(
   veryLongArgumentName)) {
veryLongFunctionNameC(veryLongFunctionNameD(
   veryLongFunctioNameE(
      veryLongFunctionNameF)));
}
```

#### 匿名函数传递

当在一个函数的参数列表中声明一个匿名函数时，函数体应该与声明的左边缘缩进两个空格，或者与 function 关键字的左边缘缩进两个空格。这是为了匿名函数体更加可读（即不被挤在屏幕的右侧）。

```js
prefix.something.reallyLongFunctionName('whatever', function(a1, a2) {
   if (a1.equals(a2)) {
      someOtherLongFunctionName(a1);
   } else {
      andNowForSomethingCompletelyDifferent(a2.parrot);
   }
});

var names = prefix.something.myExcellentMapFunction(
   verboselyNamedCollectionOfItems,
   function(item) {
      return item.name;
   });
```

#### 使用goog.scope命名别名

`goog.scope` 可用于在使用 `Closure Library` 的工程中缩短命名空间的符号引用。

每个文件只能添加一个 `goog.scope` 调用。始终将它放在全局范围内。

开放的 `goog.scope(function() {` 调用必须在之前有一个空行，并且紧跟 `goog.provide` 声明、 `goog.require` 声明或者顶层的注释。调用必须在文件的最后一行闭合。在 scope 声明闭合处追加 `// goog.scope` 。注释与分号间隔两个空格。

和 C++ 命名空间相似，不要在 `goog.scope` 声明下面缩进。相反，从第0列开始。

只取不会重新分配给另一个对象（例如大多数的构造函数、枚举和命名空间）的别名。不要这样做：

```js
goog.scope(function() {
var Button = goog.ui.Button;

Button = function() {
   // ...
};

// ...
```

别名必须和全局中的命名的最后一个属性相同。

```js
goog.provide('my.module');

goog.require('goog.dom');
goog.require('goog.ui.Button');

goog.scope(function() {
  var Button = goog.ui.Button;
  var dom = goog.dom;

  // 在构造定以后给新类型一个别名
  my.module.SomeType = function() {
     // ...
  };

  var SomeType = my.module.SomeType;

  // 像往常一样定义方法
  SomeType.prototype.findButton = function() {
     // 同上，做个按钮的别名
     this.button = new Button(dom.getElement('my-button'));
  };

  // ...

});
```

#### 更多的缩进

事实上，除了初始化数组和对象和传递匿名函数外，所有被拆开的多行文本应与之前的表达式左对齐，或者以4个（而不是2个）空格作为一缩进层次。

```js
someWonderfulHtml = '' +
                  getEvenMoreHtml(someReallyInterestingValues, moreValues,
                        evenMoreParams, 'a duck', true, 72,
                        slightlyMoreMonkeys(0xfff)) +
                  '';

thisIsAVeryLongVariableName =
   hereIsAnEvenLongerOtherFunctionNameThatWillNotFitOnPrevLine();

thisIsAVeryLongVariableName = 'expressionPartOne' + someMethodThatIsLong() +
   thisIsAnEvenLongerOtherFunctionNameThatCannotBeIndentedMore();

someValue = this.foo(
   shortArg,
   'Some really long string arg - this is a pretty common case, actually.',
   shorty2,
   this.bar());

if (searchableCollection(allYourStuff).contains(theStuffYouWant) &&
   !ambientNotification.isActive() && (client.isAmbientSupported() ||
                                       client.alwaysTryAmbientAnyways())) {
ambientNotification.activate();
}
```

#### 空行

使用新的空行来划分一组逻辑上相关联的代码片段。例如：

```js
doSomethingTo(x);
doSomethingElseTo(x);
andThen(x);

nowDoSomethingWith(y);

andNowWith(z);
```

#### 二元和三元操作符

操作符始终跟随着前行, 这样你就不用顾虑分号的隐式插入问题。否则换行符和缩进还是遵循其他谷歌规范指南。

```js
// 合适的话就在一行里面
var x = a ? b : c;

// 缩紧四个空格也不错
var y = a ?
   longButSimpleOperandB : longButSimpleOperandC;

// 每个操作符都缩紧也成
var z = a ?
   moreComplicatedB :
   moreComplicatedC;
```

点号也应如此处理。

```js
var x = foo.bar().
   doSomething().
   doSomethingElse();
```

### 括号

只用在有需要的地方。

通常只在语法或者语义需要的地方有节制地使用。

绝对不要对一元运算符如 `delete` 、 `typeof` 和 `void` 使用括号或者在关键词如 `return` 、 `throw` 和其他的（ `case` 、 `in` 或者 `new` ）之后使用括号。

### 字符串

使用 `'` 代替 `"` 。

使用单引号（ `'` ）代替双引号（ `"` ）来保证一致性。当我们创建包含有HTML的字符串时这样做很有帮助。

```js
var msg = 'This is some HTML';
```

### 可见性（私有和保护类型字段）

鼓励使用 `@private` 和 `@protected` JSDoc 注释。

我们建议使用 JSDoc 注释 `@private` 和 `@protected` 来标识出类、函数和属性的可见程度。设置 `--jscomp_warning=visibility` 可令编译器对可见性的违规进行编译器警告。加了 `@private` 标记的全局变量和函数只能被同一文件中的代码所访问。被标记为 `@private` 的构造函数只能被同一文件中的代码或者它们的静态和实例成员实例化。 `@private` 标记的构造函数可以被相同文件内它们的公共静态属性和 `instanceof` 运算符访问。全局变量、函数和构造函数不能注释 `@protected` 。

```js
// 文件1
// AA_PrivateClass_ 和 AA_init_ 是全局的并且在同一个文件中所以能被访问

/**
* @private
* @constructor
*/
AA_PrivateClass_ = function() {
};

/** @private */
function AA_init_() {
   return new AA_PrivateClass_();
}

AA_init_();
```

标记 `@private` 的属性可以被同一文件中的所有的代码访问，如果属性属于一个类，那么所有自身含有属性的类的静态方法和实例方法也可访问。它们不能被不同文件下的子类访问或者重写。

标记 `@protected` 的属性可以被同一文件中的所有的代码访问，任何含有属性的子类的静态方法和实例方法也可访问。

注意这些语义和 C++、JAVA 中 private 和 protected 的不同，其许可同一文件中的所有代码访问的权限，而不是仅仅局限于同一类或者同一类层次。此外，不像 C++ 中，子类不可重写私有属性。

```js
// File 1.

/** @constructor */
AA_PublicClass = function() {
   /** @private */
   this.privateProp_ = 2;

   /** @protected */
   this.protectedProp = 4;
};

/** @private */
AA_PublicClass.staticPrivateProp_ = 1;

/** @protected */
AA_PublicClass.staticProtectedProp = 31;

/** @private */
AA_PublicClass.prototype.privateMethod_ = function() {};

/** @protected */
AA_PublicClass.prototype.protectedMethod = function() {};

// File 2.

/**
* @return {number}
*/
AA_PublicClass.prototype.method = function() {
   // 访问这两个属性是合法的
   return this.privateProp_ + AA_PublicClass.staticPrivateProp_;
};

// File 3.

/**
* @constructor
* @extends {AA_PublicClass}
*/
AA_SubClass = function() {
   // 访问这个静态方法是合法的
   AA_PublicClass.staticProtectedProp = this.method();
};

goog.inherits(AA_SubClass, AA_PublicClass);

/**
* @return {number}
*/
AA_SubClass.prototype.method = function() {
   // 访问这个保护的实例方法是合法的
   return this.protectedProp;
};
```

注意在 Javascript 中，一个类（如 `AA_PrivateClass_` ）和其构造函数类型是没有区别的。没办法确定一种类型是 public 而它的构造函数是 private。（因为构造函数很容易重命名从而躲避隐私检查）。

### JavaScript类型

鼓励和强制执行的编译器。

JSDoc 记录类型时，要尽可能具体和准确。我们支持的类型是基于 [EcmaScript 4 规范][2]。

[2]: http://wiki.ecmascript.org/doku.php?id=spec:spec

#### JavaScript 类型语言

ES4 提案包含指定 JavaScript 类型的语言。我们使用 JSDoc 这种语言表达函数参数和返回值的类型。

随着 ES4 提议的发展，这种语言已经改变了。编译器仍然支持旧的语法类型，但这些语法已经被弃用了。

语法名称 | 语法 | 描述 | 弃用语法
-- | -- | -- | -- 
原始类型 | `{null}`，`{undefined}`，`{boolean}`，`{number}` 和 `{string}` | 实例的名称 | - 
实例类型 | `{Object}` 实例对象或空，`{Function}` 一个实例函数或空，`{EventTarget}` 构造函数实现的 EventTarget 接口，或者为 null 的一个实例。 |  一个实例构造函数或接口函数。构造函数是 `@constructor` JSDoc标记定义的函数 。接口函数是 `@interface` JSDoc标记定义的函数。默认情况下，实例类型将接受空。这是唯一的类型语法，使得类型为空。此表中的其他类型的语法不会接受空。 | - 
枚举类型 | `{goog.events.EventType}` 字面量初始化对象的属性之一 `goog.events.EventType` 。| 一个枚举必须被初始化为一个字面量对象，或作为另一个枚举的别名,加注 `@enum` JSDoc 标记。这个属性是枚举实例。请注意，这是我们的类型系统中为数不多的 ES4 规范以外的事情之一。| -
应用类型 | `{Array.<string>}` 字符串数组。`{Object.<string, number>}` 一个对象，其中键是字符串，值是数字。 | 参数化类型，该类型应用一组参数类型。这个想法是类似于Java泛型。| -
联合类型 | `{(number/boolean)}` 一个数字或布尔值。 | 表明一个值可能有A型或B型。 括号在顶层表达式可以省略，但在子表达式不能省略，以避免歧义。`{number/boolean}` `{function(): (number/boolean)}` | `{(number,boolean)}` ， `{(number/boolean)}`
可为空的类型 | `{?number}` 一个数字或空。 | 空类型与任意其他类型组合的简称。这仅仅是语法糖（syntactic sugar）。 | `{number?}`
非空类型 | `{!Object}` 一个对象，值非空。 | 从非空类型中过滤掉null。最常用于实例类型，默认可为空。 | `{Object!}` 
记录类型 | `{{myNum: number, myObject}}` 给定成员类型的匿名类型。 | 表示该值有指定的类型的成员。在这种情况下， `myNum` 是 `number` 类型而 `myObject` 可为任何类型。 注意花括号是语法类型的一部分。例如，表示一个数组对象有一个 `length` 属性，你可以写 `Array.<{length}>` 。| -
函数类型 | `{function(string, boolean)}` 一个函数接受两个参数（一个字符串和一个布尔值），并拥有一个未知的返回值。 | 指定一个函数。 | -
函数返回类型 | `{function(): number}` 一个函数没有参数并返回一个数字。 | 指定函数的返回类型。 | - 
函数 `this` 类型 | `{function(this:goog.ui.Menu, string)}` 一个需要一个参数（字符串）的函数，执行上下文是 `goog.ui.Menu` | 指定函数类型的上下文类型。 | -
函数 `new` 类型 | `{function(new:goog.ui.Menu, string)}` 一个构造函数接受一个参数（一个字符串），并在使用“new”关键字时创建一个 `goog.ui.Menu` 新实例。| 指定构造函数所构造的类型。| -
可变参数 | `{function(string, ...[number]): number}` 一个函数，它接受一个参数（一个字符串），然后一个可变数目的参数，必须是数字。 | 指定函数的变量参数。| -
可变参数（ `@param` 注释）| `@param {...number} var_args` 带注释函数的可变数目参数。| 指定带注释函数接受一个可变数目的参数。| -
函数可选参数 | `{function(?string=, number=)}` 一个函数，它接受一个可选的、可以为空的字符串和一个可选的数字作为参数。“=”只用于函数类型声明。 | 指定函数的可选参数。| -
函数可选参数（ `@param` 注释）| `@param {number=} opt_argument` `number` 类型的可选参数。 | 指定带注释函数接受一个可选的参数。 | -
所有类型 | `{*}` | 表明该变量可以接受任何类型。 | -
未知类型 | `{?}` | 表明该变量可以接受任何类型，编译器不应该检查其类型。 | -

#### JavaScript 中的类型

类型举例 | 取值举例 | 描述
-- | -- | --
number | `1` `1.0` `-5` `1-e5` `Math.PI` | -
Number | `new Number(true)` | Number 对象
string | `'Hello'` `"World"` `String(42)` | 字符串 
String | `new String('Hello')` `new String(42)` | String 对象
boolean | `true` `false` `Boolean(0)` | Boolean 值
Boolean | `new Boolean(true)` | Boolean 对象
RegExp | `new RegExp('hello')` `/world/g` | - 
Date | `new Date` `new Date()` | -
null | `null` | -
undefined | undefined | -
void | `function f() { return; }` | 没有返回值
Array | `['foo', 0.3, null]` `[]` | 无类型数组
Array.<number> | `[11, 22, 33]` | 数字数组
Array.<Array.<string>> | `[['one', 'two', 'three']`, `['foo', 'bar']]` | 以字符串为元素的数组，作为另一个数组的元素
Object | `{}` `{foo: 'abc', bar: 123, baz: null}` | -
Object.<string> | `{'foo': 'bar'}` | 值为字符串的对象
Object.<number, string> | `var obj = {};` `obj[1] = 'bar';` | 键为整数，值为字符串的对象。注意，js 当中键总是会隐式转换为字符串。所以 `obj['1'] == obj[1]` 。键在 `for-in` 循环中，总是字符串类型。但在对象中索引时编译器会验证键的类型。
Function | `function(x, y) { return x * y; }` | Function 对象
function(number, number): number | `function(x, y) { return x * y; }` | 函数值
类 | `/** @constructor */ function SomeClass() {} new SomeClass();` | -
接口 | `/** @interface */ function SomeInterface() {} SomeInterface.prototype.draw = function() {};` | -
project.MyClass | `/** @constructor */ project.MyClass = function () {} new project.MyClass()` | -
project.MyEnum | `/** @enum {string} */ project.MyEnum = { /** The color blue. */ BLUE: '#0000dd', /** The color red. */ RED: '#dd0000' };` | -
枚举 | JSDoc中枚举的值都是可选的. | -
Element | `document.createElement('div')` | DOM元素
Node | `document.body.firstChild` | DOM节点
HTMLInputElement | `htmlDocument.getElementsByTagName('input')[0]` | 指明类型的DOM元素

#### 类型转换

在类型检测不准确的情况下，有可能需要添加类型的注释，并且把类型转换的表达式写在括号里，括号是必须的。如：

```js
/** @type {number} */ (x)
```

#### 可为空与可选的参数和属性

因为 JavaScript 是一个弱类型的语言，明白函数参数、类属性的可选、可为空和未定义之间的细微差别是非常重要的。

对象类型和引用类型默认可为空。如以下表达式：

```js
/**
* 传入值初始化的类
* @param {Object} value某个值
* @constructor
*/
function MyClass(value) {
   /**
   * Some value.
   * @type {Object}
   * @private
   */
   this.myValue_ = value;
}
```

告诉编译器 `myValue_` 属性为一对象或 null。如果 `myValue_` 永远都不会为 null, 就应该如下声明:

```js
/**
* 传入非null值初始化的类
* @param {!Object} value某个值
* @constructor
*/
function MyClass(value) {
   /**
    * Some value.
    * @type {!Object}
    * @private
    */
   this.myValue_ = value;
}
```

这样，如果编译器可以识别出 `MyClass` 初始化传入值为 null，就会发出一个警告。

函数的可选参数在运行时可能会是 undefined，所以如果他们是类的属性，那么必须声明：

```js
/**
* 传入可选值初始化的类
* @param {Object=} opt_value某个值（可选）
* @constructor
*/
function MyClass(opt_value) {
   /**
    * Some value.
    * @type {Object|undefined}
    * @private
    */
   this.myValue_ = opt_value;
}
```

这告诉编译器 `myValue_` 可能是一个对象，或 `null` ，或 `undefined` 。

注意: 可选参数 `opt_value` 被声明成 `{Object=}` ，而不是 `{Object|undefined}` 。这是因为可选参数可能是 undefined。虽然直接写 undefined也并无害处，但鉴于可阅读性还是写成上述的样子。

最后，属性的可为空和可选并不矛盾，下面的四种声明各不相同：

```js
/**
* 接受四个参数，两个可为空，两个可选
* @param {!Object} nonNull 必不为null
* @param {Object} mayBeNull 可为null
* @param {!Object=} opt_nonNull 可选但必不为null
* @param {Object=} opt_mayBeNull 可选可为null
*/
function strangeButTrue(nonNull, mayBeNull, opt_nonNull, opt_mayBeNull) {
   // ...
};
```

#### 类型定义

有时类型可以变得复杂。一个函数，它接受一个元素的内容可能看起来像：

```js
/**
* @param {string} tagName
* @param {(string|Element|Text|Array.<Element>|Array.<Text>)} contents
* @return {!Element}
*/
goog.createElement = function(tagName, contents) {
   // ...
};
```

你可以定义带 `@typedef` 标记的常用类型表达式。例如：

```js
/** @typedef {(string|Element|Text|Array.<Element>|Array.<Text>)} */
goog.ElementContent;

/**
* @param {string} tagName
* @param {goog.ElementContent} contents
* @return {!Element}
*/
goog.createElement = function(tagName, contents) {
   // ...
};
```

#### 模板类型

编译器对模板类型提供有限支持。它只能从字面上通过 `this` 参数的类型和 `this` 参数是否丢失推断匿名函数的 `this` 类型。

```js
/**
* @param {function(this:T, ...)} fn
* @param {T} thisObj
* @param {...*} var_args
* @template T
*/
goog.bind = function(fn, thisObj, var_args) {
   // ...
};
// 可能出现属性丢失警告
goog.bind(function() { this.someProperty; }, new SomeClass());
// 出现this未定义警告
goog.bind(function() { this.someProperty; });
```

### 注释

使用 JSDoc。

我们使用c++的注释风格。所有的文件、类、方法和属性都应该用合适的 JSDoc 的标签和 类型注释。除了直观的方法名称和参数名称外，方法的描述、方法的参数以及方法的返回值也要包含进去。

行内注释应该使用 `//` 的形式。

为了避免出现语句片段，要使用正确的大写单词开头，并使用正确的标点符号作为结束。

#### 注释语法

JSDoc 的语法基于 [JavaDoc][4] ，许多编译工具从 JSDoc 注释中获取信息从而进行代码验证和优化，所以这些注释必须符合语法规则。

[4]: http://www.oracle.com/technetwork/java/javase/documentation/index-137868.html

```js
/**
* A JSDoc comment should begin with a slash and 2 asterisks.
* Inline tags should be enclosed in braces like {@code this}.
* @desc Block tags should always start on their own line.
*/
```

#### JSDoc 缩进

如果你不得不进行换行，那么你应该像在代码里那样，使用四个空格进行缩进。

```js
/**
* 长参数和返回值的注释说明
* @param {string} foo 这是一个灰常灰常长的注释标记着这个参数在一行里面，在这
*       里换行了。
* @return {number} This returns something that has a description too long to
*       fit in one line.
*/
project.MyClass.prototype.method = function(foo) {
   return 5;
};
```

不必在 `@fileoverview` 标记中使用缩进。

虽然不建议，但依然可以对描述文字进行排版。

```js
/**
* 这是不推荐的缩进方法
* @param {string} foo 这是一个灰常灰常长的注释标记着这个参数在一行里面，在这
*                   里换行了。
* @return {number} This returns something that has a description too long to
*                   fit in one line.
*/
project.MyClass.prototype.method = function(foo) {
   return 5;
};
```

#### JSDoc 中的 HTML

像 JavaDoc 一样, JSDoc 支持很多的HTML标签，像 `<code>` ， `<pre>` ， `<tt>` ， `<strong>` ， `<ul>` ， `<ol>` ， `<li>` ， `<a>` 等。

这就意味着不建议采用纯文本的格式。所以，不要在 JSDoc 里使用空白符进行格式化。

```js
/**
* 计算长度有三个因素：
*   发送的条目
*   接收的条目
*   上次的时间戳
*/
```

上面的注释会变成这样：

```js
计算长度有三个因素：发送的条目 接收的条目 上次的时间戳
```

所以，用下面的方式代替：

```js
/**
* 计算长度有三个因素：
* <ul>
*   <li>发送的条目
*   <li>接收的条目
*   <li>上次的时间戳
* </ul>
*/
```

[JavaDoc][3] 风格指南对于如何编写良好的doc注释是非常有帮助的。

[3]: http://www.oracle.com/technetwork/java/javase/documentation/index-137868.html

#### 顶层/文件层注释

版权声明和作者信息是可选的。顶层注释的目的是为了让不熟悉代码的读者了解文件中有什么。它需要描述文件内容，依赖关系以及兼容性的信息。例如：

```js
/**
* @fileoverview 文件描述，包含使用说明和他的依赖信息
*/
```

#### Class 评论

类必须记录说明与描述和一个类型的标签，标识的构造函数。类必须加以描述，若是构造函数则需标注出。

```js
/**
* 让事情变得容易且有趣的类
* @param {string} arg1 具体的事情
* @param {Array.<number>} arg2 待办列表
* @constructor
* @extends {goog.Disposable}
*/
project.MyClass = function(arg1, arg2) {
   // ...
};

goog.inherits(project.MyClass, goog.Disposable);
```

#### 方法和功能注释

参数和返回类型应该被记录下来。如果方法描述从参数或返回类型的描述中明确可知则可以省略。方法描述应该由一个第三人称表达的句子开始。

```js
/**
* 操作一个类实例，然后返回点东西
* @param {project.MyClass} obj 特定类的实例，这个会是一个长长的注释，使得注释
*   本身用到了换行。
* @return {boolean} 是否出发事件
*/
function PR_someMethod(obj) {
   // ...
}
```

#### 属性评论

```js
/** @constructor */
project.MyClass = function() {
   /**
   * 最大处理数值
   * @type {number}
   */
   this.someProperty = 4;
}
```

### 为 `goog.provide` 提供依赖

只提供顶级符号。

一个类上定义的所有成员应该放在一个文件中。所以，在一个在相同类中定义的包含多个成员的文件中只应该提供顶级的类（例如枚举、内部类等）。

要这样写：

```js
goog.provide('namespace.MyClass');
```

不要这样写：

```js
goog.provide('namespace.MyClass');
goog.provide('namespace.MyClass.Enum');
goog.provide('namespace.MyClass.InnerClass');
goog.provide('namespace.MyClass.TypeDef');
goog.provide('namespace.MyClass.CONSTANT');
goog.provide('namespace.MyClass.staticMethod');
```

命名空间的成员也应该提供：

```js
goog.provide('foo.bar');
goog.provide('foo.bar.method');
goog.provide('foo.bar.CONSTANT');
```

### 编译

必需。

对于所有面向客户的代码来说，使用 JS 编辑器是必需的，如使用 Closure Compiler。

#### `True` 和 `False` 布尔表达式

下边的布尔表达式都返回false：

+ null
+ undefined
+ ''空字符串
+ 数字0

但是要小心，因为以下这些返回true：

+ 字符串"0"
+ []空数组
+ {}空对象

下面这样写不好：

```js
while (x != null) {
```

你可以写成这种更短的代码（只要你不期望x为0、空字符串或者false）：

```js
while (x) {
```

如果你想检查字符串是否为null或空，你可以这样写：

```js
if (y != null && y != '') {
```

但是以下这样会更简练更好：

```js
if (y) {
```

注意：还有很多不直观的关于布尔表达式的例子，这里是一些：

```js
Boolean('0') == true
   '0' != true

0 != null
   0 == []
   0 == false

Boolean(null) == false
   null != true
   null != false

Boolean(undefined) == false
   undefined != true
   undefined != false

Boolean([]) == true
   [] != true
   [] == false

Boolean({}) == true
   {} != true
   {} != false
```

#### 条件（三元）操作符（？：）

以下这种写法可以三元操作符替换：

```js
if (val != 0) {
   return foo();
} else {
   return bar();
}
```

你可以这样写来代替：

```js
return val ? foo() : bar();
```

三元操作符在生成HTML代码时也是很有用的：

```js
var html = '<input type="checkbox"' +
   (isChecked ? ' checked' : '') +
   (isEnabled ? '' : ' disabled') +
   ' name="foo">';
```

#### `&&` 和 `||`

二元布尔操作符是可短路的，只有在必要时才会计算到最后一项。

`||` 被称作为 `default` 操作符，因为可以这样：

```js
/** @param {*=} opt_win */
function foo(opt_win) {
   var win;
   if (opt_win) {
      win = opt_win;
   } else {
      win = window;
   }

   // ...
}
```

你可以这样写：

```js
/** @param {*=} opt_win */
function foo(opt_win) {
   var win = opt_win || window;
   // ...
}
```

`&&` 也可以用来缩减代码。例如，以下这种写法可以被缩减：

```js
if (node) {
   if (node.kids) {
      if (node.kids[index]) {
         foo(node.kids[index]);
      }
   }
}
```

你可以这样写：

```js
if (node && node.kids && node.kids[index]) {
   foo(node.kids[index]);
}
```

或者这样写：

```js
var kid = node && node.kids && node.kids[index];
if (kid) {
   foo(kid);
}
```

然而以下这样写就有点过头了：

```js
node && node.kids && node.kids[index] && foo(node.kids[index]);
```

#### 遍历节点列表

节点列表是通过给节点迭代器加一个过滤器来实现的。这表示获取他的属性，如 `length` 的时间复杂度为 `O(n)`，通过 `length` 来遍历整个列表需要 `O(n^2)`。

```js
var paragraphs = document.getElementsByTagName('p');
for (var i = 0; i < paragraphs.length; i++) {
   doSomething(paragraphs[i]);
}
```

这样写更好：

```js
var paragraphs = document.getElementsByTagName('p');
for (var i = 0, paragraph; paragraph = paragraphs[i]; i++) {
   doSomething(paragraph);
}
```

这种方法对所有的集合和数组(只要数组不包含被认为是 `false` 值的元素) 都适用。

在上面的例子中，你也可以通过 `firstChild` 和 `nextSibling` 属性来遍历子节点。

```js
var parentNode = document.getElementById('foo');
for (var child = parentNode.firstChild; child; child = child.nextSibling) {
   doSomething(child);
}
```

