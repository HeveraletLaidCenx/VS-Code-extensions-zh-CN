# Textmate 语法

[原文链接，戳我前往](https://macromates.com/manual/en/language_grammars)

------

翻译 by [赫雯勒莉特翡翠](https://github.com/HeveraletLaidCenx)

## 术语~的对照表

|中文（常用英文表述）|英文|
|----|----|
|转义字符串|escape sequences|
|基类|baseclass|

表中部分：

* 在中文表述中常直接用英文替代的
* 认为直译并不合适的

在中文之后的括号中说明了直接使用对应的英文。

------

## 概述

语法 用于为 文档元素（比如 关键词、注释、字符串 或者类似的内容） 分配名称【标记】。

这样做的目的是让内容可以应用 样式（语法高亮） ，并让 文本编辑器 “智能” 地了解到 光标 所在的上下文，比如：

* 你可能希望在 按下某个键 或者 插入制表符（Tab键） 时，能根据上下文采取不同的操作
* 或者你可能想要在你的文本文档中的不是普通文本的地方（比如 HTML的标签）中禁用 拼写检查。

语法 只用于 解析文档 和 向分解出的部分分配名称【标记】。

之后 [作用域选择器](https://macromates.com/manual/en/scope_selectors) 可以用于 应用样式、首选项 以及决定 按键 和 制表符（Tab） 触发器应该如何扩展。

关于此概念的更详细介绍，请参阅 [对 作用域 的介绍](https://macromates.com/blog/2005/introduction-to-scopes/) 。

## 示例语法

你可以通过：

> 打开 包编辑器（窗口 → 显示包编辑器） 然后从左下角的 添加按钮 选择 `新语言`

来创建一个新 Textmate语法。

【译注：上方的描述是对于 MacOS 的 Textmate编辑器的操作】

这将向你提供一个 初始语法，如下所示。现在让我们从解释它开始。

```typescript
1  {  scopeName = 'source.untitled';
2     fileTypes = ( );
3     foldingStartMarker = '\{\s*$';
4     foldingStopMarker = '^\s*\}';
5     patterns = (
6        {  name = 'keyword.control.untitled';
7           match = '\b(if|while|for|return)\b';
8        },
9        {  name = 'string.quoted.double.untitled';
10           begin = '"';
11           end = '"';
12           patterns = ( 
13              {  name = 'constant.character.escape.untitled';
14                 match = '\\.';
15              }
16           );
17        },
18     );
19  }
```

它的格式是 [属性表格式](https://macromates.com/manual/en/appendix#property-list-format) ，在它的 根级别，有5个 键值对：

### `scopeName`（作用域名称）

> 位于上方代码的第 1 行。

它的值应该是 语法 的 **唯一**名称，格式遵循 `以点分隔的名称` 这个约定，也就是每个 新部分（最左侧的部分） 用于指定名称。

一般来说，它是个 `由两部分组成的名称` ：

* 第一部分是 `text`（文本） 或 `source`（源代码）
* 第二部分是 语言的名称 或 文档类型。

但是如果你打算处理一个 现有的类型，你可能想从该 现有类型 派生 名称，比如：

* Markdown 的是 `text.html.markdown`（文本.HTML.Markdown）
* Ruby on Rails （`rhtml` 文件）的是 `text.html.rails`（文本.HTML.Rails）

这种情况下，从 `text.html`（文本.HTML） 派生 的好处是，任何在 `text.html` 作用域 中生效的内容，将也在 `text.html.[something]`（文本.HTML.一些内容） 作用域中生效，但是其优先级低于 该指定作用域 `text.html.[something]` 中的内容。

### `fileTypes`（文件类型）

> 位于上方代码的第 2 行。

它的值是一个 默认情况下要应用语法的 文件扩展名 的数组。

当 TextMate 不清楚该用什么 语法 处理 用户打开的文件 时，会参考此项。如果用户从状态栏中弹出的语言窗口中选择了语法的话，TextMate 将记住该选择。

### `foldingStartMarker`（折叠区域开始标记） / `foldingStopMarker`（折叠区域结束标记）

> 位于上方代码的第 3~4 行。

这些是用于和文档中的行尝试匹配的 正则表达式。如果某一行匹配了其中一个模式（但没有两个都匹配），它将成为 折叠标记 （更多相关信息请参阅 [折叠](https://macromates.com/manual/en/navigation_overview#customizing-foldings) ）

### `patterns`（模式）

> 位于上方代码的第 5~18 行。

这部分是 包含了用于解析文档的实际规则的数组。

在这个例子中，包含两个规则（第 6~8 行 和 第 9~17 行），规则将在下一节中解释。

------

在这个例子中，还有两个额外的（根级别）键没有被使用，它们是：

* `firstLineMatch`（首行匹配）—— 一个正则表达式，当文档首次加载时，试图匹配文档的首行。如果匹配成功，则 语法 用于该文档（除非用户覆写）。比如：`^#!/.*\bruby\b` 。
* `repository`（仓库） —— 一个可以在语法中被从其它地方包含的 规则的字典（即键值对）。它的键是 规则名称，值是 对应的实际规则。更多解释和例子请参考对 `include` 规则键的描述。

## 语言规则

语言规则 负责匹配文档中的部分。

通常，规则将指定一个名称【标记】，它将被分配给 匹配上规则的文档部分。

有两种方法可以用来匹配文档。它可以提供一个或两个正则表达式。

### 第一种：单个正则表达式匹配

就像上面第 6~8 行的第一条规则中的 `match`（匹配） 键一样，与该正则表达式成功匹配的所有内容都会获得该规则指定的名称【标记】。

比如上边的第一条规则，会将名称【标记】 `keyword.control.untitled`（关键词.控制.无标题的） 指定给这些关键字：`if`、`while`、`for` 以及 `return`。接下来我们可以使用一个 `keyword.control`（关键词. 控制） 的 [作用域选择器](https://macromates.com/manual/en/scope_selectors) 来让我们的 [主题](https://macromates.com/manual/en/themes) 将这些 关键词 风格化。

### 第二种：起始/结束 模型

另一种匹配类型是上面第 9~17 行所使用的第二个规则。它给出了两个使用 `begin`（起始） 和 `end`（结束） 键 的正则表达式。该规则的名称【标记】会被分配给从 起始匹配部分 到 结束匹配部分 的内容（包括两个匹配项）。如果 结束规则 没有成功匹配，则会匹配到文档的末尾。

这种形式中，规则可以拥有 成功匹配 起始匹配部分 和 结束匹配部分 之间部分对应的下层规则。

在我们的例子中，这里我们匹配了起始和结束都是引号字符的字符串，匹配字符串（第 13~15 行）中的转义字符被标记为 `constant.character.escape.untitled`（常量.字符.转义字符.无标题的）。

### 注意

> 正则表达式每次只能匹配 **文档中的一行** 。这意味着 **不能使用多行匹配模式** 。

这样的原因是技术性的：能够在任意一行重新启动解析器，并且只需要重新解析受编辑影响的最小行数。

大多数情况下，可以使用 起始/结束 模型来克服这个限制。

## 规则键

以下是所有可以在 规则 中使用的 键 的列表。

### `name`（名称【标记】）

分配给成功匹配的内容的名称【标记】。

用于 风格化 和 特定于作用域的设置及操作，意味着通常情况下它应该从 标准名称 （请参阅后边的 [命名约定](https://macromates.com/manual/en/language_grammars#naming-conventions) ）之一派生而来。

### `match`（匹配）

用来标识应该被分配名称【标记】的文本内容的正则表达式。

例子： `'\b(true|false)\b'` 。

### `begin`（起始）、`end`（结束）

这些键允许跨多行的匹配，并且不能和 `match`（匹配） 键同时存在（因为两种同级的匹配模式在匹配时只能选择其中一种）。

`begin`（起始）、`end`（结束） 中的每个都是一个正则表达式模式。`begin`（起始） 是开始 块 的模式；`end`（结束） 是结束 块 的模式。

通过使用 正常的正则表达式反向引用，可以在`begin` 模式 中引用 `end`模式 的捕获内容。这经常被用于 here-docs，比如：

```typescript
{
  name = 'string.unquoted.here-doc';
    
  // 匹配 here-doc 标记
  begin = '<<(\w+)';
    
  // 匹配 here-doc 的结束
  end = '^\1$';
}
```

`begin`（起始）/`end`（结束） 规则可以使用 `pattern`（模式） 键 来使用 嵌套模式。比如，我们可以：

```typescript
{
  begin = '<%';
  end = '%>';
  patterns = (
    { match = '\b(def|end)\b'; … },
    …
  );
};
```

上面的规则将匹配 `<% … %>` 块 中的 关键词 `def` 和 `end`。（不过对于嵌入式语言，请参阅后面的 `include`（包含） 键。

### `contentName`（内容名称【标记】）

这个键和 `name`（名称【标记】） 键类似，但只给 匹配 `begin`（起始）/`end`（结束） 模式 **之间的** 内容分配。

比如，要想获取 `#if 0` 和 `#endif` 之间的文本，将其标记为注释，我们可以：

```typescript
{
  begin = '#if 0(\s.*)?$';
  end = '#endif';
  contentName = 'comment.block.preprocessor';
};
```

### `captures`（捕获）、`beginCaptures`（起始捕获）、`endCaptures`（结束捕获）

这些键允许你向  `match`（匹配）、`begin`（起始） 、或 `end`（结束） 模式的捕获内容 分配属性。

给 `begin`（起始）/`end`（结束） 规则使用 `captures` 键 和 同时给 `beginCaptures`（起始捕获） 和 `endCaptures`（结束捕获） 赋予相同值 的效果完全一样，可以看做一个省略写法。

这些 键 的 值 是一个字典。键 是 捕获数字，值 是 分配给捕获文本的属性的字典。目前仅支持 `name` 属性，例子如下：

```typescript
{
  match = '(@selector\()(.*?)(\))';
  captures = {
    1 = { name = 'storage.type.objc'; };
    3 = { name = 'storage.type.objc'; };
  };
};
```

在这个例子中，我们匹配了形似 `@selector(windowWillClose:)`（@选择器(窗口将要关闭:)） 的文本，但是 `storage.type.objc`（存储.类型.对象） 这个名称【标记】将只被分配给 `@selector(` 和 `)`.

### `include`（包括）

这允许你 引用一种不同的语言、递归引用语法本身 或者 在这个文件的仓库中声明的规则。

1. 要 引用另一种语言 的话，使用该语言的 作用域名称 即可：

    ```typescript
    {
      begin = '<\?(php|=)?';
      end = '\?>';
      patterns = (
        { include = "source.php"; }
      );
    }
    ```

2. 要 引用语法本身 的话，使用 `$self`：

    ```typescript
    {
      begin = '\(';
      end = '\)';
      patterns = (
        { include = "$self"; }
      );
    }
    ```

3. 要 引用当前语法仓库中引用规则 的话，请在名称前加一个井号 `#`：

    ```typescript
    patterns = ({
        begin = '"';
        end = '"';
        patterns = (
          { include = "#escaped-char"; },
          { include = "#variable"; }
        );
      },
      …
    ); // 模式的结束
    repository = {
      escaped-char = { match = '\\.'; };
      variable = { match = '\$[a-zA-Z0-9_]+'; };
    };
    ```

    这也可以用来匹配 递归结构，比如平衡字符：

    ```typescript
    patterns = ({
        name = 'string.unquoted.qq.perl';
        begin = 'qq\(';
        end = '\)';
        patterns = (
          { include = '#qq_string_content'; },
        );
      },
     …
    ); // 模式的结束
    repository = {
      qq_string_content = {
        begin = '\(';
        end = '\)';
        patterns = (
          { include = '#qq_string_content'; },
        );
      };
    };
    ```

     这将可以正确匹配类似这样的字符串：`qq( this (is (the) entire) string)` 。

## 命名约定

TextMate 是 格式自由 的。因此你基本上可以将 任何你想要的名称【标记】 分配给 你可以用语法系统标记的任何文档部分，并在之后在 [作用域选择器](https://macromates.com/manual/en/scope_selectors) 中使用该名称【标记】。

然而，为了：

* 让一个 [主题](https://macromates.com/manual/en/themes) 可以适配尽可能多的语言，而不是每个语言都有特定的一堆规则
* 功能（主要是 [首选项](https://macromates.com/manual/en/preferences_items)） 也能被跨语言重用，比如： 你可能希望在不管使用什么语言时，输入 字符串 和 注释 时都不想自动配对撇号。因此只进行一次设置就让它跨语言生效是很有意义的。

在开始了解 命名约定 之前，请记住以下内容：

1. 一个 最简的主题 只会为 以下 11 组【我们一般称之为 `顶级作用域` ，或是 `根源分组` 】中的 10 组分配样式（因为 `meta` 没有视觉样式） 。
  
    所以，你应该将你的名称【标记】分类，而不是全都扔在 `keyword`（关键词） 分类下 (as your formal language definition may insist) 。

    你应该考虑一下，“我是不是想让这俩元素的样式看起来不一样？”，如果是的话，那它们可能应该被分在不同的 根源分组 中。

2. 即使你应该 将你的名称【标记】分类，当你找到 你想将元素放置于其中的分组（比如，`storage`（存储） ）时，在决定 创建一个新的 下层分类 之前，你应该先考虑 重用目前该组中已有的 下层分类（对于 `storage`（存储来说，已有的是 `modifier`（修饰符） 或者 `type`（类型） ）。

    而你应该在你所选择的 下层分类 上附加尽可能多的信息。

    比如，如果你正准备匹配 `static`（静态） 存储修饰符，那么，你应该使用 `storage.modifier.static.«language»`（存储.修饰符.静态.某种语言名） 而不是仅仅将其命名为 `storage.modifier`（存储.修饰符） 。

    因为 作用域选择器处理 `storage.modifier`（存储.修饰符） 时，将匹配所有符合的，而添加了语言名称的话，它将可以专门针对想要生效的目标，而无视其它的存储修饰符。

3. 将 语言名称 放在 名称【标记】 的最后。

    这条看起来似乎有点多余，因为你通常也可以正常使用 这样的作用域选择器： `source.«language» storage.modifier`（源.某种语言名 存储.修饰符） ，但是对于嵌入式语言来说，有时候这种形式就不可行了。

------

那么现在让我们来看看 当前正在使用的 已经预置好的 11 个 根源分组吧~

它将展现为一个 层次列表 的形式，其中包含一些对它们的预期用途的解释。

虽然它是 分层列表 格式，但是其中的 下层的作用域 的 实际作用域名称 是以 `.` 连在 上层的作用域 之后的，比如：`double-slash`（双斜线） 的 实际作用域名称 是 `comment.line.double-slash`（注释.行.双斜线） 。

* `comment`（注释） —— 用于注释

  * `line`（行） —— 行注释，我们进一步细化它，以便可以从 作用域 中提取 注释开始的字符（串）类型。

    * `double-slash`（双斜线） —— `// 注释`
    * `double-dash`（双连接号） —— `-- 注释`
    * `number-sign`（井号） —— `# 注释`
    * `percentage`（百分号） —— `% 注释`
    * *character*（字符） —— 其它类型的行注释

  * `block` —— 多行注释，比如：

    ```typescript
    /*
    注释内容 
    */
    ```

    和

    ```html
    <!--
    注释内容
    -->
    ```

    。

    * `documentation`（文档） —— 嵌入式文档

* `constant`（常量） —— 各种形式的常量

  * `numeric`（数字的） —— 表示数字的，比如 `42`、`1.3f`、`0x4AB1U`。

  * `character`（字符） —— 代表字符的，比如 `&lt;`、`\e`、`\031`。
    * `escape`（转义） —— 转义字符串，比如 `\e` 将被分类为 `constant.character.escape`（常量.字符串.转义）。

  * `language`（语言） —— 通常由语言提供的 “特殊的” 常量，比如 `true`、`false`、`nil`、`YES`、`NO` 等。

  * `other`（其它） —— 其它常量，比如 CSS 中的 颜色。

* `entity`（实体） —— 实体 指的是 文档 中 较大的部分，比如 章节、类、函数、标签 。

    我们不会将 整个实体 的 作用域 标记为 `entity.*`（实体.\*） ，对于这部分内容，我们会用 `meta.*`（元数据.\*） 进行标记。

    `entity.*`（实体.\*） 用于标记 较大的实体中的 “占位符”，比如，如果该实体是一个章节，那我们将用 `entity.name.section`（实体.名称.章节） 来标记 该章节的标题。

  * `name`（名称） —— 用于给 较大的实体 命名

    * `function`（函数） —— 函数 的名称
    * `type`（类型） —— 类 或 类型声明 的名称
    * `tag`（标签） —— 标签 的名称
    * `section`（章节） —— 章节/标题 的 名称

  * `other`（其它） —— 其他实体

    * `inherited-class`（继承类） —— 超类/基类 的名称
    * `attribute-name`（属性名称） —— 属性（主要是在标签中的） 的名称

* `invalid`（无效的） —— “无效” 的 内容

  * `illegal`（不合法的、非法的） —— 不合法的内容，比如 HTML 中的 与号 `&` 或者 小于号 `<` ，它们不是 实体/标签 的一部分。
  * `deprecated`（不推荐的） —— 不推荐使用的内容，比如 使用了不推荐使用的 API函数 ，或者对于 严格HTML 使用了样式。

* `keyword`（关键词） —— 关键词（当无法对应到其他分类的时候）

  * `control`（控制） —— 主要涉及一些 流控制，比如 `continue`（继续）、`while`（当）、`return`（返回） 等。
  * `operator`（运算符） —— 可以是 文本型的（比如 `or`） 或者 字符型的 运算符
  * `other`（其它） —— 其它关键词

* `markup`（标记） —— 用于 标记语言，主要用于其中的 较大的文本子集。

  * `underline`（下划线） —— 标记了下划线的文本

    * `link`（链接） —— 用于 链接，方便起见，这是派生于 `markup.underline`（标记.下划线） 类型的，这样一来，如果 主题 没有特别指定 `markup.underline.link`（标记.下划线.链接） 的样式的话，它将继承 下划线类型 的样式。

  * `bold`（粗体） —— 粗体文本 （看起来粗壮的文本 或者类似的内容 最好 派生于这个名称）。

  * `heading`（标题） —— 章节标题。也可以选择性地在下一个元素提供 标题的级别，比如为 HTML 中的 `<h2>…</h2>`（二级标题） 标记为 `markup.heading.2.html`（标记.标题.2.HTML语言）。

  * `italic`（斜体） —— 斜体文本 （强调文本 或者类似的内容 最好 派生于这个名称）。

  * `list`（列表） —— 列表项

    * `numbered`（有编号的） —— 有编号的列表项
    * `unnumbered`（无编号的） —— 无编号的列表项

  * `quote`（引用） —— 用引号括起来的文本，有时是 块引用。

  * `raw`（原生） —— 逐字记录的文本，比如 代码列表。通常情况下，对于 `markup.raw`（标记.原生） 分类来说，拼写检查是禁用的。

  * `other`（其它） —— 其它的标记结构

* `meta`（元数据） —— meta（元数据） 作用域 通常用于标记 文档中较大的部分。

    比如， 声明函数的一整行 将被标记为 `meta.function`（元数据.函数） 。

    它的子集可能是 `storage.type`（存储.类型）、`entity.name.function`（实体.名称.函数）、`variable.parameter`（变量.参数） 等。只有这些子集将被风格化。

    有时，作用域的 meta（元数据） 部分将仅被用于限制 被风格化的更一般的元素，但是大多数时候，元数据作用域 被用于在 作用域选择器 中 激活 捆绑项目。

    比如，在 Objective-C（面向对象的C语言） 中，类 的 接口声明 和 实现 都会有一个 元数据作用域，允许同一个 标签触发器 根据上下文来以不同的方式展开。

* `storage`（存储） —— 和 “存储” 相关的内容

  * `type`（类型） —— 某些内容的类型，比如 `class`（类）、`function`（函数）、`int`（整形）、`var`（变量） 等。
  * `modifier`（修饰符） —— 存储 的 修饰符，比如 `static`（静态）、`final`（最终）、`abstract`（抽象） 等。

* `string`（字符串） —— 字符串

  * `quoted`（带引号的） —— 带引号的字符串

    * `single`（单） —— 单引号字符串 如 `'foo'`
    * `double`（双） —— 双引号字符串 如 `"foo"`
    * `triple`（三） —— 三引号字符串 如 `"""Python"""`.
    * `other`（其它） —— 其他类型的引号字符串 如 `$'shell'`、`%s{...}`

  * `unquoted`（无引号的） —— 像 here-docs 和 here-strings 这样的东西

  * `interpolated`（以内插值替换的） —— 会被 “评估” 的字符串，比如 ``date``、`$(pwd)` 。

  * `regexp`（正则表达式） —— 正则表达式 如 `/(\w+)/`

  * `other`（其它） —— 其他类型的字符串（应该很少使用这类）

* `support`（支持） —— 由 框架 或 库 提供的内容

  * `function`（函数） —— 由 框架/库 提供的 函数。比如，Objective-C（面向对象的C语言） 中的 `NSLog` 的分类就是 `support.function`（支持.函数） 。
  * `class`（类） —— 由 框架/库 提供的 类
  * `type` —— 由 框架/库 提供的 类型，这个分类或许只会被用于从 C语言 派生出来的语言中，它们具有 `typedef`（类型定义） 和 `struct`（结构） 。大部分其他语言会将引入的新类型作为 `class`（类） 。
  * `constant`（常量） —— 由 框架/库 提供的 常量（magic values（魔法值）） 。
  * `variable`（变量） —— 由 框架/库 提供的 变量。比如 AppKit 中的 `NSApp` 。
  * `other`（其它） —— 以上这些应该已经包括了所有的内容了，但是如果还有其它没覆盖到的，请把它丢在 `support.other`（支持.其它） 这个分类里。

* `variable`（变量） —— 变量，并非所有语言都能轻松地 识别 和 标记 变量。

  * `parameter`（参数） —— 当 变量被声明为参数 时分到此类
  * `language` —— 保留的语言变量，比如 `this`（此）、`super`（超）、`self`（自身） 等。
  * `other`（其它） —— 其它变量，比如 `$some_variables` 这种
