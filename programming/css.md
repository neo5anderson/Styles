# CSS 编程风格指南

## 语法

+ 两个空格代替制表符
+ 每个选择器单独一行
+ 声明块的左括号前一个空格，声明块的右花括号应当单独成行
+ 声明语句 `:` 后面有一个空格
+ 为了获得更准确的错误报告，每条声明独占一行
+ 所有声明语句以分号结尾，包括最后一条声明语句的分号
+ 以逗号分隔的属性值，每个逗号后面都应该插入一个空格（例如下文 `box-shadow`）
+ 不要在 `rgb()`、`rgba()`、`hsl()`、`hsla()` 或 `rect()` 值的内部的逗号后面插入空格
+ 对于属性值或颜色参数，省略小于 1 的小数前面的 0 （例如，`.5` 代替 `0.5`；`-.5px` 代替 `-0.5px`）。
+ 十六进制值应该全部小写
+ 尽量使用简写形式的十六进制值，例如，用 `#fff` 代替 `#ffffff`
+ 双引号包裹选择器中的属性，例如，`input[type="text"]`，[例外][1] 并不推荐
+ 数值为 0 的不需要单位，例如，用 `margin: 0;` 代替 `margin: 0px;`。

[1]: https://mathiasbynens.be/notes/unquoted-attribute-values#css

不推荐

```css
.selector, .selector.secondary, .selector[type=text] {
  padding:15px;
  margin:0px 0px 15px;
  background-color:rgba(0, 0, 0, 0.5);
  box-shadow:0px 1px 2px #CCC,inset 0 1px 0 #FFFFFF
}
```

推荐

```css
.selector,
.selector.secondary,
.selector[type="text"] {
  padding: 15px;
  margin-bottom: 15px;
  background-color: rgba(0,0,0,.5);
  box-shadow: 0 1px 2px #ccc, inset 0 1px 0 #fff;
}
```

## 声明顺序

相关的属性声明应当归为一组，并按照下面的顺序排列：

+ Positioning
+ Box model
+ Typographic
+ Visual

由于定位（positioning）可以从正常的文档流中移除元素，并且还能覆盖盒模型（box model）相关的样式，因此排在首位。盒模型排在第二位，因为它决定了组件的尺寸和位置。

其他属性只是影响组件的内部（inside）或者是不影响前两组属性，因此排在后面。

完整的属性列表及其排列顺序请参考 Recess。

```css
.declaration.order {
  /* Positioning */
  position: absolute;
  top: 0;
  right: 0;
  bottom: 0;
  left: 0;
  z-index: 100;

  /* Box-model */
  display: block;
  float: right;
  width: 100px;
  height: 100px;

  /* Typography */
  font: normal 13px "Helvetica Neue", sans-serif;
  line-height: 1.5;
  color: #333;
  text-align: center;

  /* Visual */
  background-color: #f5f5f5;
  border: 1px solid #e5e5e5;
  border-radius: 3px;

  /* Misc */
  opacity: 1;
}
```

## 不要使用 `@import`

与 `<link>` 标签相比，`@import` 指令要慢很多，不光增加了额外的请求次数，还会导致不可预料的问题。替代办法有以下几种：

+ 使用多个 `<link>` 元素
+ 通过 `Sass` 或 `Less` 类似的 CSS 预处理器将多个 CSS 文件编译为一个文件
+ 通过 Rails、Jekyll 或其他系统中提供过 CSS 文件合并功能

请参考 [Steve Souders][2] 的文章了解更多知识。

[2]: http://www.stevesouders.com/blog/2009/04/09/dont-use-import/

不推荐

```html
<!-- @imports -->
<style>
  @import url("more.css");
</style>
```

推荐

```html
<!-- 使用 link 元素 -->
<link rel="stylesheet" href="core.css">
```

## 媒体查询（Media query）的位置

将媒体查询放在尽可能相关规则的附近。不要将他们打包放在一个单一样式文件中或者放在文档底部。如果你把他们分开了，将来只会被大家遗忘。下面给出一个典型的实例。

```css
.element { ... }
.element.avatar { ... }
.element.selected { ... }

@media (min-width: 480px) {
  .element { ...}
  .element.avatar { ... }
  .element.selected { ... }
}
```

## 带前缀的属性

当使用特定厂商的带有前缀的属性时，通过缩进的方式，让每个属性的值在垂直方向对齐，这样便于多行编辑。

> 在 Textmate 中，使用 Text → Edit Each Line in Selection (⌃⌘A)。在 Sublime Text 2 中，使用 Selection → Add Previous Line (⌃⇧↑) 和 Selection → Add Next Line (⌃⇧↓)。
推荐

```css
.selector {
  -webkit-box-shadow: 0 1px 2px rgba(0,0,0,.15);
          box-shadow: 0 1px 2px rgba(0,0,0,.15);
}
```

## 单行规则声明

对于只包含一条声明的样式，为了易读性和便于快速编辑，建议将语句放在同一行。对于带有多条声明的样式，还是应当将声明分为多行。

这样做的关键因素是为了错误检测 -- 例如，CSS 校验器指出在 183 行有语法错误。如果是单行单条声明，你就不会忽略这个错误；如果是单行多条声明的话，你就要仔细分析避免漏掉错误了。

```css
/* 单行规则声明 */
.span1 { width: 60px; }
.span2 { width: 140px; }
.span3 { width: 220px; }

/* 多行规则声明 */
.sprite {
  display: inline-block;
  width: 16px;
  height: 15px;
  background-image: url(../img/sprite.png);
}
.icon           { background-position: 0 0; }
.icon.home      { background-position: 0 -20px; }
.icon.account   { background-position: 0 -40px; }
```

## 简写形式的属性声明

在需要显示地设置所有值的情况下，应当尽量限制使用简写形式的属性声明。常见的滥用简写属性声明的情况如下：

+ padding
+ margin
+ font
+ background
+ border
+ border-radius

大部分情况下，我们不需要为简写形式的属性声明指定所有值。例如，HTML 的 heading 元素只需要设置上、下边距（`margin`）的值，因此，在必要的时候，只需覆盖这两个值就可以。过度使用简写形式的属性声明会导致代码混乱，并且会对属性值带来不必要的覆盖从而引起意外的副作用。

MDN（Mozilla Developer Network）上一片非常好的关于 [shorthand properties][3] 的文章，对于不太熟悉简写属性声明及其行为的用户很有用。

[3]: https://developer.mozilla.org/en-US/docs/Web/CSS/Shorthand_properties

不推荐

```css
.element {
  margin: 0 0 10px;
  background: red;
  background: url("image.jpg");
  border-radius: 3px 3px 0 0;
}
```

推荐

```css
.element {
  margin-bottom: 10px;
  background-color: red;
  background-image: url("image.jpg");
  border-top-left-radius: 3px;
  border-top-right-radius: 3px;
}
```

## Less 和 Sass 中的嵌套

避免非必要的嵌套。这是因为虽然你可以使用嵌套，但是并不意味着应该使用嵌套。只有在必须将样式限制在父元素内（也就是后代选择器），并且存在多个需要嵌套的元素时才使用嵌套。

无嵌套

```css
.table > thead > tr > th { }
.table > thead > tr > td { }
```

嵌套

```css
.table > thead > tr {
  > th { }
  > td { }
}
```

## 注释

代码是由人编写并维护的。请确保你的代码能够自描述、注释良好并且易于他人理解。好的代码注释能够传达上下文关系和代码目的。不要简单地重申组件或 class 名称。

对于较长的注释，务必书写完整的句子；对于一般性注解，可以书写简洁的短语。

不推荐

```css
/* Modal header */
.modal.header {
  ...
}
```

推荐

```css
/* 包裹 .modal-title 与 .modal-close 的外围容器 */
.modal.header {
  ...
}
```

## class 命名（此条暂时未完全定下来）

class 名称中只能出现小写字符和破折号（dashe）（不是下划线，也不是驼峰命名法）。同时，破折号只有在多个单词必须连着同时存在才具有真实意义时使用，用于区分两个不同的单词，例如，`.button` 与 `.danger.button` ，但是不允许使用 `.danger-button` 或者 `.button.danger`。

不允许使用简写。但 class 名称应当尽可能短，并且意义明确。使用有意义的名称。使用有组织的或目的明确的名称，不要使用表现形式（presentational）的名称。

使用 `.js-*` class 来标识行为（与样式相对），并且不要将这些 class 包含到 CSS 文件中。

在为 Sass 和 Less 变量命名是也可以参考上面列出的各项规范。

不推荐

```css
.t { ... }
.red { ... }
.header { ... }
```

推荐

```css
.tweet { ... }
.important { ... }
.tweet.header { ... }
```

## 选择器

对于通用元素使用 class ，这样利于渲染性能的优化。对于经常出现的组件，避免使用属性选择器（例如，`[class^="..."]`）。浏览器的性能会受到这些因素的影响。

选择器要尽可能短，并且尽量限制组成选择器的元素个数，建议不要超过 3 。 只有在必要的时候才将 class 限制在最近的父元素内（也就是后代选择器）（例如，不使用带前缀的 class 时 -- 前缀类似于命名空间）。

不推荐

```css
span { ... }
.page-container #stream .stream-item .tweet .tweet-header .username { ... }
.avatar { ... }
```

推荐

```css
.avatar { ... }
.tweet-header .username { ... }
.tweet .avatar { ... }
```

## 代码组织

+ 以组件为单位组织代码段。
+ 制定一致的注释规范。
+ 使用一致的空白符将代码分隔成块，这样利于扫描较大的文档。
+ 如果使用了多个 CSS文件，将其按照组件而非页面的形式分拆，因为页面会被重组，而组件只会被移动。

```css
/*
 * 组件
 */

.element { ... }


/*
 * 组件
 *
 * 有的时候也可能一个组件是很复杂的
 */

.element { ... }

/* 组件的子组件 */
.element-heading { ... }
```

