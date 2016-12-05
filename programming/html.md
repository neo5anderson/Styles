# HTML 编程风格指南

## 目录

## 语法

+ 用两个空格来代替制表符，这是唯一能保证在所有环境下获得一致展现的方法。
+ 嵌套元素应当缩进一次（即两个空格）。
+ 对于属性的定义，确保全部使用双引号，绝不要使用单引号。
+ 不要在自闭合（`self-closing`）元素的尾部添加斜线 -- HTML5 -规范中明确说明这是可选的。
+ 不要省略可选的结束标签（`closing tag`）（例如，`</li>` 或 `</body>`）。

```html
<!DOCTYPE html>
<html lang="zh">
<head>
    <meta charset="UTF-8">
    <title>Page title</title>
</head>
<body>
    <img src="images/company-logo.png" alt="Company">
    <h1 class="hello-world">Hello, world!</h1>
</body>
</html>
```

## 文档类型

推荐使用 HTML5 的文档类型申明： `<!DOCTYPE html>`。

（建议使用 text/html 格式的 HTML。避免使用 XHTML。XHTML 以及它的属性，比如 `application/xhtml+xml` 在浏览器中的应用支持与优化空间都十分有限）。

HTML 中最好不要将无内容元素的标签闭合，例如：使用 `<br>` 而非 `<br />`。

> 此处的空白元素指的是以下元素：area, base, br, col, command, embed, hr, img, input, keygen, link, meta, param, source, track, wbr。

## 语言属性

根据 HTML5 规范：

> 强烈建议为 html 根元素指定 lang 属性，从而为文档设置正确的语言。这将有助于语音合成工具确定其所应该采用的发音，有助于翻译工具确定其翻译时所应遵守的规则等等。 更多关于 lang 属性的知识可以从 此规范 中了解。

[这里列出了语言代码表][1]。

[1]: https://www.sitepoint.com/web-foundations/iso-2-letter-language-codes/

```html
<html lang="zh">
  <!-- ... -->
</html>
```

## IE 兼容模式

IE 支持通过特定的 `<meta>` 标签来确定绘制当前页面所应该采用的 IE 版本。除非有强烈的特殊需求，否则最好是设置为 `edge mode`，从而通知 IE 采用其所支持的最新的模式。

阅读这篇 [stack overflow][2] 上的文章可以获得更多有用的信息。

[2]: http://stackoverflow.com/questions/6771258/what-does-meta-http-equiv-x-ua-compatible-content-ie-edge-do

```html
<meta http-equiv="X-UA-Compatible" content="IE=Edge">
```

## HTML 验证

一般情况下，建议使用能通过标准规范验证的 HTML 代码，除非在性能优化和控制文件大小上不得不做出让步。

使用诸如 W3C HTML validator 这样的工具来进行检测。

规范化的 HTML 是显现技术要求与局限的显著质量基线，它促进了 HTML 被更好地运用。

不推荐

```html
<title>测试</title>
<article>这仅仅只是一个测试。
```

推荐

```html
<!DOCTYPE html>
<meta charset="utf-8">
<title>测试</title>
<article>这仅仅只是一个测试。</article>
```

## 省略可选标签

HTML5 规范中规定了 HTML 标签是可以省略的。但从可读性来说，在开发的源文件中最好不要这样做，因为省略标签可能会导致一些问题。

省略一些可选的标签确实使得页面大小减少，这很有用，尤其是对于一些大型网站来说。为了达到这一目的，我们可以在开发后期对页面进行压缩处理，在这个环节中这些可选的标签完全就可以省略掉了。

## 脚本加载

出于性能考虑，脚本异步加载很关键。一段脚本放置在 `<head>` 内，比如 `<script src="main.js"></script>`，其加载会一直阻塞 DOM 解析，直至它完全地加载和执行完毕。这会造成页面显示的延迟。特别是一些重量级的脚本，对用户体验来说那真是一个巨大的影响。

异步加载脚本可缓解这种性能影响。如果只需兼容 IE10+，可将 HTML5 的 `async` 属性加至脚本中，它可防止阻塞 DOM 的解析，甚至你可以将脚本引用写在 `<head>` 里也没有影响。

如需兼容老旧的浏览器，实践表明可使用用来动态注入脚本的脚本加载器。你可以考虑 yepnope 或 labjs。注入脚本的一个问题是：一直要等到 CSS 对象文档已就绪，它们才开始加载（短暂地在 CSS 加载完毕之后），这就对需要及时触发的 JS 造成了一定的延迟，这多多少少也影响了用户体验吧。

终上所述，兼容老旧浏览器(IE9-)时，应该遵循以下最佳实践。

脚本引用写在 `body` 结束标签之前，并带上 `async` 属性。这虽然在老旧浏览器中不会异步加载脚本，但它只阻塞了 body 结束标签之前的 DOM 解析，这就大大降低了其阻塞影响。而在现代浏览器中，脚本将在 DOM 解析器发现 body 尾部的 script 标签才进行加载，此时加载属于异步加载，不会阻塞 CSSOM（但其执行仍发生在 CSSOM 之后）。

所有浏览器中，推荐

```html
<html>
  <head>
    <link rel="stylesheet" href="main.css">
  </head>
  <body>
    <!-- body 代码在此处 -->

    <script src="main.js" async></script>
  </body>
</html>
```

只在现代浏览器中，推荐

```html
<html>
  <head>
    <link rel="stylesheet" href="main.css">
    <script src="main.js" async></script>
  </head>
  <body>
    <!-- body 代码在此处 -->
  </body>
</html>
```

## 语义化

根据元素（有时被错误地称作“标签”）其被创造出来时的初始意义来使用它。打个比方，用 heading 元素来定义头部标题，p 元素来定义文字段落，用 a 元素来定义链接锚点，等等。

有根据有目的地使用 HTML 元素，对于可访问性、代码重用、代码效率来说意义重大。

以下示例列出了一些的语义化 HTML 主要情况：

不推荐

```html
<b>页面标题</b>
<div class="top-navigation">
  <div class="nav-item"><a href="#home">首页</a></div>
  <div class="nav-item"><a href="#news">新闻</a></div>
  <div class="nav-item"><a href="#about">关于</a></div>
</div>

<div class="news-page">
  <div class="page-section news">
    <div class="title">所有新文章</div>
    <div class="news-article">
      <h2>为好的文章
      <div class="intro">子标题</div>
      <div class="content">在 HTML 语义中，这个示例中的代码是很差很差的。</div>
      <div class="article-side-notes">我想我永远也不会认同这样写代码。</div>
      <div class="article-foot-notes">
        本文创建于 <div class="time">2000-01-01 00:00</div>
      </div>
    </div>

    <div class="section-footer">
      标签：示例、错误示例
    </div>
  </div>
</div>

<div class="page-footer">
  版权所有 balabala
</div>
```

推荐

```html
<!-- 页面的头部应该放在一个 header 元素中 -->
<header>
  <!-- 因为标题属于当前这个页面的结构，且它定义了整页的标题，所以，它应该使用 h1 标签 -->
  <h1>页面标题</h1>
</header>

<!-- 所有的页面导航应该放在一个 nav 元素中 -->
<nav class="main nav">
  <!-- 一个列表总是应该放在一个 ol（有序列表）或者 ul （无序列表）中 -->
  <ul>
    <li class="nav item"><a href="#home">首页</a></li>
    <li class="nav item"><a href="#news">新闻</a></li>
    <li class="nav item"><a href="#about">关于</a></li>
  </ul>
</nav>

<!-- 页面主要的内容部分应该放在一个 main 元素中（为了访问可用性，还应该使用 role="main"） -->
<main class="news page" role="main">
  <!-- 页面的一个章节应该放在一个 section 元素中，将页面分为语义化的章节 -->
  <section class="news section">
    <!-- 章节的标题等内容应该放在 section 中的 header 元素中 -->
    <header class="section header">
      <!-- 作为页面一个章节的标题，它应该使用一个标题元素，在此处，它使用 h2 -->
      <h2 class="section title">所有文章</h2>
    </header>

    <!-- 如果一个 section 可以被看作是一篇文章（新闻、博客日志、商品简介或者任何其它可复用的元素），那么应该使用 article 元素。 -->
    <article class="news article">
      <!-- 一篇文章同样可以包含一个 header 元素，用于包含文章的摘要、介绍以及标题等 -->
      <header>
        <!-- 因为此处的标题并不属性整个页面的，仅仅只属于这一篇文章，所以，此处不应该使用标题元素（当某一个页面就是这篇文章时，此文的标题就应该作为页面的标题，那个时候，文章标题应该使用 h1） -->
        <div class="article title">这篇文章的结构就很好了</div>
        <!-- small 元素可以作为可选的元素，在此处用于显示子标题 -->
        <small class="intro">Introduction sub-title</small>
      </header>

      <!-- 对于文章的内容而言，因为 html5 中并没有定义该语义标签，所以，我们才使用一个 div 标签 -->
      <div class="content">
        <p>这是一个有着良好语义定义的HTML文档中的一个段落。</p>
      </div>
      <!-- 如果文章中，根据上下文关系，有一些评注、笔记等内容，应该使用 aside 元素。 -->
      <aside class="article side notes">
        <p>我感觉这段话很不错。</p>
      </aside>
      <!-- article 元素同样可以有 footer 元素，当你的一篇文章有脚注时，可以使用footer -->
      <footer class="article foot notes">
        <!-- time 元素可以注解一个时间戳，使用 datetime 属性即可定义一个ISO标准时间，而 time 标签中的内容则可以是人类可读的友好文本。 -->
        <p>This article was created by David <time datetime="2015-04-25 11:16" class="time">刚刚</time></p>
      </footer>
    </article>

    <!-- 对于整个 section 而言，脚注同样需要放在 footer 元素中 -->
    <footer class="section footer">
      <p>标签：示例，正确示例</p>
    </footer>
  </section>
</main>

<!-- 然后，整个页面的脚注应该放在一个全局的 footer 元素中 -->
<footer class="page footer">
  版权所有 balabala
</footer>
```

## 多媒体回溯

对页面上的媒体而言，像图片、视频、canvas 动画等，要确保其有可替代的接入接口。图片文件我们可采用有意义的备选文本（`alt`），视频和音频文件我们可以为其加上说明文字或字幕。

提供可替代内容对可用性来说十分重要。试想，一位盲人用户如何能知晓一张图片是什么，要是没有 alt 的话。

图片的 `alt` 属性是可不填写内容的，纯装饰性的图片就可用这么做：`alt=""`。

不推荐

```html
<img src="luke-skywalker.jpg">
```

推荐

```html
<img src="luke-skywalker.jpg" alt="天行者 Luke 正骑着一匹外星球的马">
```

尽量用 alt 标签去描述图片，设想你需要对于那些只能通过语音或者看不见图片的用户表达图片到底是什么。

不推荐

```html
<img src="huge-spaceship-approaching-earth.jpg" alt="头部背景图片">
```

推荐

```html
<img src="huge-spaceship-approaching-earth.jpg" alt="一张从太空眺望地球的图片">
```

## 关注点分离

理解 web 中如何和为何区分不同的关注点，这很重要。这里的关注点主要指的是：信息（HTML 结构）、外观（CSS）和行为（JavaScript）。为了使它们成为可维护的干净整洁的代码，我们要尽可能的将它们分离开来。

严格地保证结构、表现、行为三者分离，并尽量使三者之间没有太多的交互和联系。

就是说，尽量在文档和模板中只包含结构性的 HTML；而将所有表现代码，移入样式表中；将所有动作行为，移入脚本之中。

在此之外，为使得它们之间的联系尽可能的小，在文档和模板中也尽量少地引入样式和脚本文件。

清晰的分层意味着：

+ 不使用超过一到两张样式表（i.e. `main.css`, `vendor.css`）
+ 不使用超过一到两个脚本（学会用合并脚本）
+ 不使用行内样式（`<style>.no-good {}</style>`）
+ 不在元素上使用 style 属性（`<hr style="border-top: 5px solid black">`）
+ 不使用行内脚本（`<script>alert('no good')</script>`）
+ 不使用表象元素（i.e. `<b>`, `<u>`, `<center>`, `<font>`）
+ 不使用表象 class 名（i.e. `red`, `left`, `center`）

不推荐

```html
<!DOCTYPE html>
<html>
<head>
  <link rel="stylesheet" href="base.css">
  <link rel="stylesheet" href="grid.css">
  <link rel="stylesheet" href="type.css">
  <link rel="stylesheet" href="modules/teaser.css">
</head>
<body>
  <h1 style="font-size: 3rem"></h1>
  <b>我是一个子标题，而且我加粗了！</b>
  <center>哈哈，我现在是居中的了</center>
  <script>
    alert('呃，千万别这么做！');
  </script>
  <div class="red">我是红色的，我将非常重要。</div>
</body>
</html>
```

推荐

```html
<!DOCTYPE html>
<html>
<head>
  <!-- 将所有的样式表放在一到两个文件里面 -->
  <!-- 如果有来自第三方的库的样式表，那么，我们也应该将所有第三方样式表压缩至一个名为 vendor.css 的文件中 -->
  <link rel="stylesheet" href="vendor.css">
  <!-- 然后将自己的所有样式放在一个名为 main.css 的文件中 -->
  <link rel="stylesheet" href="main.css">
</head>
<body>
  <!-- 不要直接在元素中使用 style 属性，而应该为其指定合乎情理的的 class，然后在样式表中定义他们的样式 -->
  <h1 class="title"></h1>
  <!-- 不要使用表象元素，然后还要为其指定 class -->
  <div class="sub title">我是一个子标题，而且我加粗了！</div>
  <!-- 也许我们是评论需要居中，但是我们不应该使用 center，而应该很明确的通过 class 指定这个是 comment，然后在样式表中指定它的样式 -->
  <span class="comment">哈哈，我现在是居中的了</span>
  <!-- 如果你感觉某段话非常重要，想通过将其颜色修改为艳丽的红色以达到警示的效果，那么，你应该设置一个class，然后在样式表中设定它的样式 -->
  <div class="important">我非常重要。</div>

  <!-- 将你所有的 javascript 代码写进 .js 文件中，最后还要生成合并为一个文件 -->
  <script async src="main.js"></script>
</body>
</html>
```

## HTML 内容至上

不要让非内容信息污染了你的 HTML。现在貌似有一种倾向：通过 HTML 来解决设计问题，这是显然是不对的。HTML 就应该只关注内容。

HTML 标签的目的，就是为了不断地展示内容信息。

不要引入一些特定的 HTML 结构来解决一些视觉设计问题 不要将 img 元素当做专门用来做视觉设计的元素 以下例子展示了误将 HTML 用来解决设计问题的这两种情况：

不推荐

```html
<!-- 我们不应该仅仅为了一个视觉元素就轻易的引入一个新的元素  -->
<span class="text-box">
  <span class="square"></span>
  看我旁边的正方形
</span>
.text-box > .square {
  display: inline-block;
  width: 1rem;
  height: 1rem;
  background-color: red;
}
```

推荐

```html
<!-- 这样才清爽嘛 -->
<span class="text-box">
  看我旁边的正方形
</span>
/* 使用一个 :before 来解决视觉问题，然后将其置于内容的前面即可 */
.text-box:before {
  content: "";
  display: inline-block;
  width: 1rem;
  height: 1rem;
  background-color: red;
}
```

图片和 SVG 图形能被引入到 HTML 中的唯一理由是它们呈现出了与内容相关的一些信息。

不推荐

```html
<!-- 内容元素永远都不应该被用于装饰，他们应该以样式的形式引入页面的视觉中  -->
<span class="text-box">
  <img src="square.svg" alt="Square" />
  看我旁边的正方形
</span>
```

推荐

```html
<!-- 这样才成 -->
<span class="text-box">
  看我旁边的正方形
</span>
/* 同样使用 :before 来解决视觉问题，然后将其置于内容的前面即可 */
.text-box:before {
  content: "";
  display: inline-block;
  width: 1rem;
  height: 1rem;
  background: url(square.svg) no-repeat;
  background-size: 100%;
}
```

## Type 属性

省略样式表与脚本上的 type 属性。鉴于 HTML5 中以上两者默认的 type 值就是 `text/css` 和 `text/javascript`，所以 type 属性一般是可以忽略掉的。甚至在老旧版本的浏览器中这么做也是安全可靠的。

不推荐

```html
<link rel="stylesheet" href="main.css" type="text/css">
<script src="main.js" type="text/javascript"></script>
```

推荐

```html
<link rel="stylesheet" href="main.css">
<script src="main.js"></script>
```

## 属性顺序

HTML 属性应当按照以下给出的顺序依次排列，确保代码的易读性。

+ class
+ id, name
+ data-*
+ src, for, type, href
+ title, alt
+ aria-*, role

class 用于标识高度可复用组件，因此应该排在首位。id 用于标识具体组件，应当谨慎使用（例如，页面内的书签），因此排在第二位。

```html
<a class="..." id="..." data-modal="toggle" href="#">
  Example link
</a>

<input class="form-control" type="text">

<img src="..." alt="...">
```

## 布尔（boolean）型属性

布尔型属性可以在声明时不赋值。XHTML 规范要求为其赋值，但是 HTML5 规范不需要。

更多信息请参考 WhatWG section on boolean attributes：

> 元素的布尔型属性如果有值，就是 true，如果没有值，就是 false。 如果一定要为其赋值的话，请参考 WhatWG 规范：

> 如果属性存在，其值必须是空字符串或 [...] 属性的规范名称，并且不要再收尾添加空白符。

简单来说，就是不用赋值。

## 可用性

如果 HTML5 语义化标签使用得当，许多可用性问题已经引刃而解。ARIA 规则在一些语义化的元素上可为其添上默认的可用性角色属性，使用得当的话已使网站的可用性大部分成立。假如你使用 nav, aside, main, footer 等元素，ARIA 规则会在其上应用一些关联的默认值。 更多细节可参考 ARIA specification

另外一些角色属性则能够用来呈现更多可用性情景（i.e. `role="tab"`）。

### Tab Index 在可用性上的运用

检查文档中的 tab 切换顺序并传值给元素上的 `tabindex`，这可以依据元素的重要性来重新排列其 tab 切换顺序。你可以设置 `tabindex="-1"` 在任何元素上来禁用其 tab 切换。

当你在一个默认不可聚焦的元素上增加了功能，你应该总是为其加上 `tabindex` 属性使其变为可聚焦状态，而且这也会激活其 CSS 的伪类 `:focus`。选择合适的 `tabindex` 值，或是直接使用 `tabindex="0"` 将元素们组织成同一 tab 顺序水平，并强制干预其自然阅读顺序。

### 微格式在 SEO 和可用性上的运用

如果 SEO 和可用性环境条件允许的话，建议考虑采用微格式。微格式是通过在元素标签上申明一系列特定数据来达成特定语义的方法。

谷歌、微软和雅虎对如何使用这些额外的数据一定程度上的达成一致，如果正确的使用，这将给搜索引擎优化带来巨大的好处。

你可以访问 schema.org 获得更多内容细节。

看一个电影网站的简单例子：

不带微格式

```html
<div>
 <h1>阿凡达</h1>
 <span>导演：詹姆斯·卡梅隆 James Cameron</span>
 <span>科幻</span>
 <a href="../movies/avatar-theatrical-trailer.html">预告片</a>
</div>
```

带有微格式

```html
<div itemscope itemtype ="http://schema.org/Movie">
  <h1 itemprop="name">阿凡达</h1>
  <div itemprop="director" itemscope itemtype="http://schema.org/Person">
  导演： <span itemprop="name">导演：詹姆斯·卡梅隆</span> <span itemprop="englishName"> James Cameron</span>
  </div>
  <span itemprop="genre">科幻</span>
  <a href="../movies/avatar-theatrical-trailer.html" itemprop="trailer">预告片</a>
</div>
```

## ID 和锚点

通常一个比较好的做法是将页面内所有的头部标题元素都加上 ID. 这样做，页面 URL 的 hash 中带上对应的 ID 名称，即形成描点，方便跳转至对应元素所处位置。

打个比方，当你在浏览器中输入 URL `http://your-site.com/about#best-practices`，浏览器将定位至以下 H3 上。

```html
<h3 id="best-practices">Best practices</h3>
```

## 格式化规则

在每一个块状元素，列表元素和表格元素后，加上一新空白行，并对其子孙元素进行缩进。内联元素写在一行内，块状元素还有列表和表格要另起一行。

> 如果由于换行的空格引发了不可预计的问题，那将所有元素并入一行也是可以接受的，格式警告总好过错误警告。

推荐

```html
<blockquote>
  <p><em>Space</em>, the final frontier.</p>
</blockquote>

<ul>
  <li>Moe</li>
  <li>Larry</li>
  <li>Curly</li>
</ul>

<table>
  <thead>
    <tr>
      <th scope="col">Income</th>
      <th scope="col">Taxes</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>$ 5.00</td>
      <td>$ 4.50</td>
    </tr>
  </tbody>
</table>
```

## HTML 引号

使用双引号(`""`) 而不是单引号(`''`) 。

不推荐

```html
<div class='news-article'></div>
```

推荐

```html
<div class="news article"></div>
```

