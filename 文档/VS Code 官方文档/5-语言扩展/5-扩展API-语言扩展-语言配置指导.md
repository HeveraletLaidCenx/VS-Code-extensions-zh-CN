# 语言配置指导

[原文链接，戳我前往](https://code.visualstudio.com/api/language-extensions/language-configuration-guide)

------

翻译 by [赫雯勒莉特翡翠](https://github.com/HeveraletLaidCenx)

## 术语~的对照表

|中文（常用英文表述）|英文|
|----|----|
|词语模式【有译为“字模式”】|Word pattern|

表中部分：

* 在中文表述中常直接用英文替代的
* 认为直译并不合适的

在中文之后的括号中说明了直接使用对应的英文。

------

## 概述

[`contributes.languages`（建立作用点.语言）](https://code.visualstudio.com/api/references/contribution-points#contributes.languages) 作用点 让你可以定义一个用来控制 以下这些声明式语言规则 的 语言配置。

* 注释切换
* 括号定义
* 自动关闭
* 内容自动环绕
* 折叠
* 词语模式
* 缩进规则

这个 [语言配置的例子](https://github.com/microsoft/vscode-extension-samples/tree/main/language-configuration-sample) 展示了一个用于配置 JavaScript 文件的编辑体验的 语言配置。

本篇指导解释了 `language-configuration.json`（语言配置.json） 文件中的内容，它大概长成这样：

**注意：如果你的 语言配置文件 的文件名是、或者以 `language-configuration.json` 结尾，那么你可以在 VS Code 中获得 代码自动补全 和 验证。**

```json
{
  "comments": {
    "lineComment": "//",
    "blockComment": ["/*", "*/"]
  },
  "brackets": [
    ["{", "}"],
    ["[", "]"],
    ["(", ")"]
  ],
  "autoClosingPairs": [
    { "open": "{", "close": "}" },
    { "open": "[", "close": "]" },
    { "open": "(", "close": ")" },
    { "open": "'", "close": "'", "notIn": ["string", "comment"] },
    { "open": "\"", "close": "\"", "notIn": ["string"] },
    { "open": "`", "close": "`", "notIn": ["string", "comment"] },
    { "open": "/**", "close": " */", "notIn": ["string"] }
  ],
  "autoCloseBefore": ";:.,=}])>` \n\t",
  "surroundingPairs": [
    ["{", "}"],
    ["[", "]"],
    ["(", ")"],
    ["'", "'"],
    ["\"", "\""],
    ["`", "`"]
  ],
  "folding": {
    "markers": {
      "start": "^\\s*//\\s*#?region\\b",
      "end": "^\\s*//\\s*#?endregion\\b"
    }
  },
  "wordPattern": "(-?\\d*\\.\\d\\w*)|([^\\`\\~\\!\\@\\#\\%\\^\\&\\*\\(\\)\\-\\=\\+\\[\\{\\]\\}\\\\\\|\\;\\:\\'\\\"\\,\\.\\<\\>\\/\\?\\s]+)",
  "indentationRules": {
    "increaseIndentPattern": "^((?!\\/\\/).)*(\\{[^}\"'`]*|\\([^)\"'`]*|\\[[^\\]\"'`]*)$",
    "decreaseIndentPattern": "^((?!.*?\\/\\*).*\\*/)?\\s*[\\}\\]].*$"
  }
}
```

## 注释切换

**VS Code** 提供了两个用于切换注释的命令，分别是 `Toggle Line Comment`（切换行注释） 和 `Toggle Block Comment`（切换块注释） 。你可以指定 `comments.lineComment`（注释.行注释） 和 `comments.blockComment`（注释.块注释） 来控制 **VS Code** 如何注释 代码行 / 代码块。

```json
{
  "comments": {
    "lineComment": "//",
    "blockComment": ["/*", "*/"]
  }
}
```

## 括号定义

当你把光标移动到在这里定义的括号时，**VS Code** 将把 光标所在位置的括号 和 它对应的括号 一起高亮显示。

```json
{
  "brackets": [
    ["{", "}"],
    ["[", "]"],
    ["(", ")"]
  ]
}
```

此外，当你使用 `Go to Bracket`（转到括号） 或者 `Select to Bracket`（选择括号所有内容） 命令时，**VS Code** 也将用这里的定义来查找最近的和与之匹配的括号。

## 自动关闭

当你输入一个 `'` 时，**VS Code** 将自动创建一对单引号，并将你的光标放在它们中间，就像这样：`'|'` 。这一节定义了可以这么做的 符号对。

```json
{
  "autoClosingPairs": [
    { "open": "{", "close": "}" },
    { "open": "[", "close": "]" },
    { "open": "(", "close": ")" },
    { "open": "'", "close": "'", "notIn": ["string", "comment"] },
    { "open": "\"", "close": "\"", "notIn": ["string"] },
    { "open": "`", "close": "`", "notIn": ["string", "comment"] },
    { "open": "/**", "close": " */", "notIn": ["string"] }
  ]
}
```

其中 `notIn`（不在...中） 的值规定了在对应的代码范围中禁用 自动关闭 功能。比如，当你编写如下代码时：

```es6
// ES6's Template String【一行注释】
`ES6's Template String`;【反单引号包裹的字符串】
```

此时的单引号将不会自动关闭。

而不需要设置 `notIn`（不在...中） 属性的 规则也可以用一种更加简单的语法来表示，像下面这样：

```json
{
  "autoClosingPairs": [
    ["{", "}"],
    ["[", "]"]
  ]
}
```

用户可以通过设置中的 `editor.autoClosingQuotes`（编辑器.自动关闭引号） 和 `editor.autoClosingBrackets`（编辑器.自动关闭括号） 来调整 自动关闭行为。

### 在...之前自动关闭

默认情况下，**VS Code** 只有在光标右侧有一个空格的时候，才会自动关闭 符号对。所以当你在以下的 JSX 代码中输入 `{` 时，它将不会自动关闭。

```JSX
const Component = () =>
  <div className={>
                  ↑ 此处在默认情况下不会自动关闭
  </div>
```

但是，这个定义可以通过以下定义来覆写：

```json
{
  "autoCloseBefore": ";:.,=}])>` \n\t"
}
```

现在，当你在相同情况下，在 `>` 之前输入 `{` 时，**VS Code** 将会自动关闭它~

## 内容自动环绕

当你在 **VS Code** 中选择一个范围，然后输入一个 左括号 `(` 时，**VS Code** 会自动将选中的内容用一对括号包裹（环绕）起来。这个功能就叫做 `Autosurrounding`（内容自动环绕），在此处可以为某种语言定义 内容自动环绕的括号对：

```json
{
  "surroundingPairs": [
    ["{", "}"],
    ["[", "]"],
    ["(", ")"],
    ["'", "'"],
    ["\"", "\""],
    ["`", "`"]
  ]
}
```

用户可以通过设置中的 `editor.autoSurround` 来调整 内容自动环绕行为。

## 折叠

在 **VS Code** 中，折叠是基于缩进定义的，或者是由建立的 折叠范围提供程序 作用点 定义的：

* 带有标记的 基于缩进的 折叠：如果给定语言没有 折叠范围提供程序 可用的话，或者 用户将设置中的 `editor.foldingStrategy`（编辑器.折叠策略） 设置成了 `indentation`（基于缩进），则会使用 基于缩进的折叠。
  
  折叠区域：当 一行内容 比 下边的内容 的缩进小时开始，当有一行具有 与之相同的 或者 比它更小的 缩进时结束。空行不计。
  
  ```text
    A：一些内容【可折叠区域起始】
      可折叠区域的内容
      ...
    情况1：其他一些内容【可折叠区域结束，此行缩进和 A 相同】
  情况2：其他一些内容【可折叠区域结束，此行缩进比 A 小】
  ```

  此外，语言配置可以定义 起始标记 和 结束标记。它们在 `folding.markers` 中通过 `start` 和 `end` 的正则表达式来定义。当检测到符合规则的行时，在这两个标记之间的内容将被创建为可折叠区域。折叠标记 必须是非空的，它们通常看起来类似 `//#region` 和 `//#endregion`。

以下 JSON 文件创建了 `//#region` 和 `//#endregion` 标记。

```json
{
  "folding": {
    "markers": {
      "start": "^\\s*//\\s*#?region\\b",
      "end": "^\\s*//\\s*#?endregion\\b"
    }
  }
}
```

* 语言服务器折叠：语言服务器 会响应 带有一个折叠范围列表的 [`textDocument/foldingRange`（文本文档/折叠范围）](https://microsoft.github.io/language-server-protocol/specification#textDocument_foldingRange) 请求，**VS Code** 将把这些范围渲染为 折叠标记。更多与 语言服务器协议中的折叠支持 的内容请参阅 [编程型语言功能](https://code.visualstudio.com/api/language-extensions/programmatic-language-features) 。

## 词语模式

`wordPattern`（词语模式） 定义了在编程语言中，什么被看作一个单词。如果设置了它的话，代码建议功能 将用它来确定单词边界。

注意，这里的设置不会影响 与词相关的编辑器命令，与词相关的编辑器命令是由编辑器设置中的 `editor.wordSeparators`（编辑器.词语分离器） 选项控制的。

```json
{
  "wordPattern": "(-?\\d*\\.\\d\\w*)|([^\\`\\~\\!\\@\\#\\%\\^\\&\\*\\(\\)\\-\\=\\+\\[\\{\\]\\}\\\\\\|\\;\\:\\'\\\"\\,\\.\\<\\>\\/\\?\\s]+)"
}
```

## 缩进规则

`indentationRules`（缩进规则） 定义了：当你输入、粘贴，或者移动行的时候，编辑器应该如何调整 当前行 或者 下一行 的缩进。

```json
{
  "indentationRules": {
    "increaseIndentPattern": "^((?!\\/\\/).)*(\\{[^}\"'`]*|\\([^)\"'`]*|\\[[^\\]\"'`]*)$",
    "decreaseIndentPattern": "^((?!.*?\\/\\*).*\\*/)?\\s*[\\)\\}\\]].*$"
  }
}
```

比如， `if (true) {` 这条语句会匹配 `increaseIndentPattern`（增加缩进模式），然后如果你在这个语句的 `{` 之后按下 `Enter` 键，编辑器将会自动缩进一次然后你的代码就会变成这样：

```typescript
if (true) {
  console.log();
```

除了 `increaseIndentPattern`（增加缩进模式） 和 `decreaseIndentPatter`（减少缩进模式），还有两条其他的缩进规则：

* `indentNextLinePattern`（缩进下一行模式） —— 如果一行内容符合这个模式，那么 **只有这一行的下一行** 会被缩进一行after it should be indented once.
* `unIndentedLinePattern`（不进行缩进的行模式） —— 如果一行内容符合这个模式，那么它的缩进将不被改变，它也不会被评估为其他的缩进规则。

如果对于某种编程语言，没有设置 缩进规则的话，编辑器将会在某行以 左括号 结尾时进行缩进，并在你输入 右括号 后反缩进。这里的括号是由 `brackets`（括号） 定义的。

注意， `editor.formatOnPaste`（编辑器.粘贴内容的格式） 选项是由 [`DocumentRangeFormattingEditProvider`（文档范围格式化编辑提供程序）](https://code.visualstudio.com/api/references/vscode-api#DocumentRangeFormattingEditProvider) 控制的，不会被 自动缩进 所影响。

## Enter 时的规则

`onEnterRules`（Enter 时的规则） 定义了一系列规则，这些规则会在 在编辑器中按下 `Enter` 键的时候被评估。

```json
{
  "onEnterRules": [
    {
      "beforeText": "^\\s*(?:def|class|for|if|elif|else|while|try|with|finally|except|async).*?:\\s*$",
      "action": { "indent": "indent" }
    }
  ]
}
```

当按下 `Enter` 时，会根据以下属性，检查之前、之后、上一行的文本。

* `beforeText`（在文本之前）（强制的）。匹配光标前文本的正则表达式（仅限当前行）。
* `afterText`（在文本之后）。匹配光标后文本的正则表达式（仅限当前行）。
* `previousLineText`（前一行文本）。匹配光标上方一行文本的正则表达式。

如果所有指定的属性都匹配的话，则认为该规则匹配，且不会再进一步评估 `onEnterRules`（Enter 时的规则）。`onEnterRule`（Enter 时的规则） 可以指定以下的操作：

* `indent`（缩进）（强制的）。将是 `none`、`indent`、`outdent`、`indentOutdent` 中的一个。

  * `none` 表示新行将会继承当前行的缩进
  * `indent` 表示新行将会相对当前行进行缩进
  * `outdent` 表示新行将会想对当前行反缩进
  * `indentOutdent` 表示之后的两行，第一行缩进，第二行反缩进

* `appendText`（追加文本）。将会在新行和缩进之后追加一个字符串。

* `removeText`（移除文本）。要从新行的缩进中移除的字符的数量。
