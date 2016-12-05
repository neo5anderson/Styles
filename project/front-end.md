# 前端项目规范

## 文件结构

```
template/
    vendor/
        第三方库、样式、脚本
    assets/
        css/
        img/
        js/
    index.html
    layout.html

build/
    vendor/
        第三方库、样式、脚本
    assets/
        css/
        img/
        js/
    index.html
    layout.html

gulp.js
scripts/
scss/
```

## 文件、资源命名

在 web 项目中，所有的文件名应该都遵循同一命名约定。以可读性而言，减号（-）是用来分隔文件名的不二之选。同时它也是常见的 URL 分隔符（i.e. `//example.com/blog/my-blog-entry` 或者 `//s.example.com/images/big-black-background.jpg`），所以理所当然的，减号应该也是用来分隔资源名称的好选择。

请确保文件命名总是以字母开头而不是数字。而以特殊字符开头命名的文件，一般都有特殊的含义与用处（比如 compass 中的下划线就是用来标记跳过直接编译的文件用的）。

资源的字母名称必须全为小写，这是因为在某些对大小写字母敏感的操作系统中，当文件通过工具压缩混淆后，或者人为修改过后，大小写不同而导致引用文件不同的错误，很难被发现。

还有一些情况下，需要对文件增加前后缀或特定的扩展名（比如 `.min.js`, `.min.css`），抑或一串前缀（比如 `3fa89b.main.min.css`）。这种情况下，建议使用点分隔符来区分这些在文件名中带有清晰意义的元数据。

不推荐

+ MyScript.js
+ myCamelCaseName.css
+ i_love_underscores.html
+ 1001-scripts.js
+ my-file-min.css

推荐

+ my-script.js
+ my-camel-case-name.css
+ i-love-underscores.html
+ thousand-and-one-scripts.js
+ my-file.min.css

## 协议头

建议在指向图片、样式表、JavaScript脚本或者其它媒体文件的URL地址中省略 `http:` 与 `https:` 协议部分，除非已知相应文件不能同时兼容这两个协议，比如：

我们不推荐下面这种方式：

```html
<script src="http://www.doraemoney.com/vendor/jquery/2.1.3/jquery.min.js"></script>
<link rel="stylesheet" href="http://assets/css/style.css"/>
<style>
  .example {
    background: #fff url(http://www.doraemoney.com/assets/img/example-background.png) no-repeat center;
  }
</style>
```

你应该像下面这样写：

```html
<script src="//www.doraemoney.com/vendor/jquery/2.1.3/jquery.min.js"></script>
<link rel="stylesheet" href="//assets/css/style.css"/>
<style>
  .example {
    background: #fff url(//www.doraemoney.com/assets/img/example-background.png) no-repeat center;
  }
</style>
```

另外的示例，比如通过 <img>标签引入图片，我们同样不推荐这样写：

```html
<img src="http://www.doraemoney.com/assets/img/logo.png" alt="叮当钱包" class="logo"/>
```

而应该这样写：

```html
<img src="//www.doraemoney.com/assets/img/logo.png" alt="叮当钱包" class="logo"/>
```

## 代码缩进

不允许在任何 `HTML`、`CSS` 以及 `JavaScript` 代码中使用 `Tab` 进行缩进，且所有的缩进只允许使用两个空格 `  ` ，如下面这样是正确的：

```html
<a class="brand">
  <img src="//www.doraemoney.com/assets/img/logo.png" alt="叮当钱包" />
</a>

@media screen and (min-width: 1100px) {
  body {
    font-size: 100%;
  }
}

(function(){
  var x = 10;

  function y(a, b) {
    return {
      result: (a + b) * x
    }

  }
}());
```

而下面这样均为错误的：

```html
<a class="brand">
    <img src="//www.doraemoney.com/assets/img/logo.png" alt="叮当钱包" />
</a>
```

## 大小写

所有的代码都应该是小写的，包括元素名称、属性、属性值（除非 text 或者 CDATA 的内容）、选择器、CSS属性以及属性值，如下面这样均是错误的：

```html
<A HREF="http://www.doraemoney.com" TITLE="叮当钱包官方网站首页" CLASS="Brand"/>叮当钱包</A>
```

正确的写法应该是：

```html
<a href="http://www.doraemoney.com" title="叮当钱包官方网站首页" class="brand">叮当钱包</a>
```

对于资源文件的命名，我们同样要求全部使用纯小写字母，同时，多个单词间使用中横线分割（`-`），同时，不允许在命名中出现空格，英文字母必须放在第一个位置，`-` 不允许出现在名称格式后缀小数点 . 的前面，如下面这样就是正确的命名方法：

+ logo.png
+ example-background.png
+ assets/css/base.css

而下面这些均是错误的：

+ logo-.png
+ -logo.png
+ 0logo.png
+ example background.png
+ Example-Background.png
+ ExampleBackground.png
+ Assets/Css/Base.css

## 尾随空格

尾随空格是绝对不允许的，容易让 diff 更加复杂，比如下面这样是不允许的：

```html
<h2>什么是叮当钱包？ </h2>
<p>叮当钱包的命名灵感来源于哆啦A梦的四次元口袋，叮当希望自己也可以像这个神奇口袋一样，在小伙伴需要帮助时，伸出温暖援（圆）手，给予大家“无所不能”的帮助。</p>
```

下面这样才是正确的：

```html
<h2>什么是叮当钱包？</h2>
<p>叮当钱包的命名灵感来源于哆啦A梦的四次元口袋，叮当希望自己也可以像这个神奇口袋一样，在小伙伴需要帮助时，伸出温暖援（圆）手，给予大家“无所不能”的帮助。</p>
```

注意仔细观察，在 `h2` 标签中 `？` 号后面有无空格。

## 编码格式

所有的文件，包括 `.html`、`.css`、`.js`、`scss`、`less` 等，必须全部使用 utf-8 编码格式，如下面这样的是正确的：

```html
<head>
  <meta charset="utf-8" />
</head>
```

## 代码注释

注释是你自己与你的小伙伴们了解代码写法和目的的唯一途径。特别是在写一些看似琐碎的无关紧要的代码时，由于记忆点不深刻，注释就变得尤为重要了。

编写自解释代码只是一个传说，没有任何代码是可以完全自解释的。而代码注释，则是永远也不嫌多。

当你写注释时一定要注意：不要写你的代码都干了些什么，而要写你的代码为什么要这么写，背后的考量是什么。当然也可以加入所思考问题或是解决方案的链接地址。

任何编写代码的人都必须根据需要撰写代码注释，对于团队开发来说，这是非常重要的，比如下方是HTML的注释规范：

```html
<!-- 这是单行注释 -->
<div class="foo"></bar>

<!--
  这是多行注释的标题

  这是多行注释中的一行
-->
```

不推荐

```js
var offset = 0;

if(includeLabels) {
  // 设置 offset 为20
  offset = 20;
}
```

推荐

```js
var offset = 0;

if(includeLabels) {
  // 如果饮食了 label，那么我们还需要 20 像素的宽度，以解决下面这个Bug
  // http://somebrowservendor.com/issue-tracker/ISSUE-1 
  offset = 20;
}
```

一些注释工具可以帮助你写出更好的注释。JSDoc 或 YUIDoc 就是用来写 JavaScript 注释用的。你甚至可以使用工具来为这些注释生成文档，这也是激励开发者们写注释的一个好方法，因为一旦有了这样方便的生成文档的工具，他们通常会开始花更多时间在注释细节上。

## 代码检查

对于比较宽松自由的编程语言来说，严格遵循编码规范和格式化风格指南就显得极为重要。遵循规范固然很好，但是有自动化流程来确保其执行情况，岂不更佳。

> 相信很好，要是能控制，那就更好了

对于 JavaScript，建议使用 JSLint 或 JSHint。

