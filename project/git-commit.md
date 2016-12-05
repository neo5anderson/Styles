# Git 提交规范指南

## 格式化的作用

1. 提供更多的历史信息，方便快速浏览。

通过命令查看历史变动，每个 commit 占据一行。你只看行首，就知道每次提交的目的。

```sh
git log <last tag> HEAD --pretty=format:%s
```

2. 可以过滤某些 commit（比如文档改动），便于快速查找信息。

比如，下面的命令仅仅显示本次发布新增加的功能。

```sh
$ git log <last release> HEAD --grep feature
```

3. 可以直接从 commit 生成 Change log。

Change Log 是发布新版本时，用来说明与上一个版本差异的文档，例如：

```
0.0.4 (2016-08-21)

## 修复漏洞

+ 术语表：修复术语表内漏洞 (9e57eb)
+ 介绍：修复无法使用介绍漏洞 (6be0da)

## 特性

+ 二期：添加更好的冗余文内容 (0723fa)
+ 二期：添加冗余文内容 (7abe21)

## 突出的改动

+ 添加素食冗余文本内容以及肉食组件
```

## 提交格式

每次提交，Commit message 都包括三个部分：Header，Body 和 Footer。

```
<type>(<scope>): <subject>

<body>

<footer>
```

其中，Header 是必需的，Body 和 Footer 可以省略。

不管是哪一个部分，任何一行都不得超过 80 个字符。这是为了避免自动换行影响美观。

### Header

Header部分只有一行，包括三个字段：type（必需）、scope（可选）和subject（必需）。

1. type

type 用于说明 commit 的类别，只允许使用下面7个标识。

+ feat：新功能（feature）
+ fix：修补bug
+ docs：文档（documentation）
+ style： 格式（不影响代码运行的变动）
+ refactor：重构（即不是新增功能，也不是修改bug的代码变动）
+ test：增加测试
+ chore：构建过程或辅助工具的变动

如果 type 为 feat 和 fix，则该 commit 将肯定出现在 Change log 之中。其他情况（docs、chore、style、refactor、test）由你决定，要不要放入 Change log，建议是不要。

2. scope

scope 用于说明 commit 影响的范围，比如数据层、控制层、视图层等等，视项目不同而不同。

3. subject

subject 是 commit 目的的简短描述，不超过50个字符。

以动词开头，使用第一人称现在时，比如 change，而不是 changed 或 changes 第一个字母小写结尾不加句号

### Body

Body 部分是对本次 commit 的详细描述，可以分成多行。下面是一个范例。

```
More detailed explanatory text, if necessary.  Wrap it to 
about 72 characters or so. Further paragraphs come after blank lines.

- Bullet points are okay, too
- Use a hanging indent
```

有两个注意点。

1. 使用第一人称现在时，比如使用 change 而不是 changed 或 changes。
2. 应该说明代码变动的动机，以及与以前行为的对比。

### Footer

Footer 部分只用于两种情况。

1. 不兼容变动

如果当前代码与上一个版本不兼容，则 Footer 部分以 `BREAKING CHANGE` 开头，后面是对变动的描述、以及变动理由和迁移方法。

```
BREAKING CHANGE: isolate scope bindings definition has changed.
 
    To migrate the code follow the example below:
 
    Before:
 
    scope: {
      myAttr: 'attribute',
    }
 
    After:
 
    scope: {
      myAttr: '@',
    }
 
    The removed `inject` wasn't generaly useful for directives so there should be no code using it.
```

2. 关闭 Issue

如果当前 commit 针对某个issue，那么可以在 Footer 部分关闭这个 issue 。

```
Closes #234
```

也可以一次关闭多个 issue 。

```
Closes #123, #245, #992
```

### Revert

还有一种特殊情况，如果当前 commit 用于撤销以前的 commit，则必须以revert:开头，后面跟着被撤销 Commit 的 Header。

```
revert: feat(pencil): add 'graphiteWidth' option
 
This reverts commit 667ecc1654a317a13331b17617d973392f415f02.
```

Body部分的格式是固定的，必须写成 `This reverts commit <hash>.`，其中的 `hash` 是被撤销 commit 的 SHA 标识符。

如果当前 commit 与被撤销的 commit，在同一个发布（release）里面，那么它们都不会出现在 Change log 里面。如果两者在不同的发布，那么当前 commit，会出现在 Change log 的 `Reverts` 小标题下面。

## 生成 Change long

如果你的所有 Commit 都符合 Angular 格式，那么发布新版本时， Change log 就可以用脚本自动生成（[例1][1]，[例2][2]，[例3][3]）。

[1]: https://github.com/conventional-changelog/conventional-changelog/blob/master/CHANGELOG.md
[2]: https://github.com/karma-runner/karma/blob/master/CHANGELOG.md
[3]: https://github.com/btford/grunt-conventional-changelog/blob/master/CHANGELOG.md

生成的文档包括以下三个部分。

+ New features
+ Bug fixes
+ Breaking changes.

每个部分都会罗列相关的 commit ，并且有指向这些 commit 的链接。当然，生成的文档允许手动修改，所以发布前，你还可以添加其他内容。

[`conventional-changelog`][4] 就是生成 Change log 的工具，运行下面的命令即可。

```sh
npm install -g conventional-changelog
cd my-project
conventional-changelog -p angular -i CHANGELOG.md -w
```

上面命令不会覆盖以前的 Change log，只会在 `CHANGELOG.md` 的头部加上自从上次发布以来的变动。

如果你想生成所有发布的 Change log，要改为运行下面的命令。

```sh
conventional-changelog -p angular -i CHANGELOG.md -w -r 0
```

为了方便使用，可以将其写入package.json的scripts字段。

```json
{
  "scripts": {
    "changelog": "conventional-changelog -p angular -i CHANGELOG.md -w -r 0"
  }
}
```

以后，直接运行下面的命令即可。

```sh
npm run changelog
```

