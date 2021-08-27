# 扩展功能概述

[原文链接，戳我前往](https://code.visualstudio.com/api/extension-capabilities/overview)

------

翻译 by [赫雯勒莉特翡翠](https://github.com/HeveraletLaidCenx)

## 术语~的对照表

|中文（常用英文表述）|英文|
|----|----|
|限制|restrictions|
|访问|access|
|文档对象模型（DOM）|Document Object Model|
|通用功能|Common Capabilities|
|上下文菜单|context menu|
|工作区|workspace|
|全局的|global|
|快速选取|Quick Pick|
|文件选择器|file picker|
|进度|progress|
|长时间运行的|long-running|
|《黑客帝国》【一部著名电影】|The Matrix|
|工作区|workspace|
|将 A 移植到 B|port A to B|
|声明式的|declarative|
|括号匹配|bracket matching|
|自动缩进|auto-indentation|
|语法高亮|syntax highlighting|
|编程型的|programmatic|
|打包、捆绑|bundle|
|片段|snippets|
|注入|injections|
|hover 本意悬停，这里指鼠标悬停提示|Hovers|
|诊断错误|diagnostic errors|
|【VS 系列的一个功能】|CodeLens|
|暴露【使可见可操作】|expose|
|语言服务器|Language Server|
|库|library|
|内联的、内嵌的|inline|
|【一类用于进行静态代码分析的查错程序或插件的统称】|Lint、Linter|
|代码格式化程序|code fromatter|
|上下文感知、情境感知|context-aware|
|折叠|folding|
|面包屑导航|Breadcrumbs|
|大纲|outline|
|文件资源管理器|File Explorer|
|右键点击|right-click|
|活动栏|Activity Bar|
|状态栏|Status Bar|
|渲染|render|
|源代码控制提供程序|Source Control providers|
|适配器|adapter|
|实现者|implementors|
|自动化|automate|
|动态的|dynamically|
|会话|sessions|
|跟踪|track|
|生命周期|lifecycle|
|元素|element|
|优化|optimize|
|底层网络技术|underlying web technologies|
|直接的|direct|

表中部分：

* 在中文表述中常直接用英文替代的
* 认为直译并不合适的

在中文之后的括号中说明了直接使用对应的英文。

------

## 概述

**Visual Studio Code** 为扩展 扩展的功能 提供了很多方式。有时候可能很难找到达到设计功能所需要使用的合适的 [**作用点**](https://code.visualstudio.com/api/references/contribution-points) 和 [**VS Code 扩展API**](https://code.visualstudio.com/api/references/vscode-api) 。这里我们把扩展的功能分成了几类，他们分别描述：

* 你的扩展可以使用的一些功能
* 使用这些功能的更详细内容的文章的链接
* 一些扩展的灵感

但是，我们也对扩展进行了 [限制](https://code.visualstudio.com/api/extension-capabilities/overview#restrictions) 来保证 **VS Code** 的稳定性和性能表现。比如，扩展不能访问 **VS Code** UI 的 DOM 。

## 通用功能

[通用功能](https://code.visualstudio.com/api/extension-capabilities/common-capabilities) 是你在任何扩展中都可以使用的核心功能。

比如包括：

* 注册 命令、配置、键位绑定，或者是上下文菜单中的项目。
* 存储工作区或者全局数据。
* 显示通知消息。
* 使用 快速选取 来收集用户的输入。
* 打开系统的文件选择器来让用户选择文件或者文件夹。
* 使用 进度API 来指示长时间运行的操作。

## 主题

[主题](https://code.visualstudio.com/api/extension-capabilities/theming) 控制 **VS Code** 的外观，包括编辑器中的源代码和整体 **VS Code** UI 的配色。如果你曾想通过把 **VS Code** 的界面变成不同明暗的绿色，让你看上去像在编写《黑客帝国》风格的代码，又或者是想创造一个极致简约的灰度配色工区，那么主题就是你在找的东西。

一些 **扩展的灵感**

* 更改你的源代码的配色。
* 更改你的 **VS Code** UI的配色。
* 将现有的 **TextMate** 主题移植到 **VS Code** 。
* 添加自定义的文件图标。

## 声明式语言功能

[声明式语言功能](https://code.visualstudio.com/api/language-extensions/overview#declarative-language-features) 为一种编程语言提供了基本的文字编辑的支持，比如 括号匹配、自动缩进，和 语法高亮。这些都是以声明式的方法完成的，不需要编写任何代码。

对于更高级的语言功能，比如 智能感知提示 或者 Debug纠错，请参阅 [编程型语言功能](https://code.visualstudio.com/api/extension-capabilities/overview#programmatic-language-features)。

一些 **扩展的灵感**

* 将常用的 **JavaScript** 代码片段打包到一个扩展中。
* 教会 **VS Code** 一种新的编程语言的规则。
* 添加或替换一种编程语言的语法。
* 通过语法注入来扩展一个现有的语法。
* 将现有的 **TextMate** 语法移植到 **VS Code** 。

## 编程型语言功能

[编程型语言功能](https://code.visualstudio.com/api/language-extensions/overview#programmatic-language-features) 添加了丰富的编程语言支持，比如 鼠标悬停提示、转到定义、诊断错误、智能感知提示 和 CodeLens 。这些语言功能通过 [vscode.languages.*](https://code.visualstudio.com/api/references/vscode-api#languages)（vscode.语言） API 暴露。扩展可以直接使用这些 API，也可以编写一个语言服务器并使用 [**VS Code 语言服务器库**](https://github.com/microsoft/vscode-languageserver-node) 将其适配到 **VS Code** 。

虽然我们提供了一个包含 [语言功能](https://code.visualstudio.com/api/language-extensions/programmatic-language-features) 和它们的预期用法，但是这并不限制你创造性地使用这些API。比如，CodeLens 和 鼠标悬停提示 就是个内嵌显示额外信息的好方法。而 诊断错误 可以被用来高亮拼写或代码风格的错误。

一些 **扩展的灵感**

* 添加一个展示某个 API 的使用方法示例的 鼠标悬停提示。
* 使用 诊断 来报告源代码中的拼写或者 Linter 错误。
* 为 HTML 注册一个新的 代码格式化程序。
* 提供丰富的【*：不确定此处rich的释义，因为参考 rich text，即富文本，已经在中文里形成了一个专有名词，此处查阅了相关关键词，不确定是否存在类似的“富-”前缀这种用法】、上下文感知的 智能感知提示。
* 为一种语言添加 内容折叠、面包屑导航 和 大纲 的支持。

## 工作台扩展

[工作台扩展](https://code.visualstudio.com/api/extension-capabilities/extending-workbench) 用于扩展 **VS Code** 工作台的 UI。比如 在 文件资源管理器 中添加一个新的右键点击操作，甚至使用 [**VS Code 的 树视图API**](https://code.visualstudio.com/api/extension-guides/tree-view) 来创建一个自定义的资源管理器。而如果你的扩展需要一个完全自定义的 UI，可以使用 [Webview API](https://code.visualstudio.com/api/extension-guides/webview) ，用标准的 HTML/CSS/JS 来建立你自己的文档 预览 或者 UI。

一些 **扩展的灵感**

* 向 文件资源管理器 添加一个自定义上下文菜单操作。
* 在 侧边栏 里创建一个新的、交互式的树视图。
* 定义一个新的 活动栏视图。
* 在 状态栏 显示新的信息。
* 使用 **WebView API** 渲染自定义内容。
* Contribute 源代码控制提供程序。【*：本条未能准确理解意思】

## Debug

你可以通过编写 [调试器扩展](https://code.visualstudio.com/api/extension-guides/debugger-extension) ，将 **VS Code** 的 Debug UI 连接到某个特定的 调试器 或 runtime ，来使用 **VS Code** 的 [Debug](https://code.visualstudio.com/docs/editor/debugging) 功能。

一些 **扩展的灵感**

* 通过建立 [Debug 适配器实现者](https://microsoft.github.io/debug-adapter-protocol/implementors/adapters/) 来把 **VS Code** 的 Debug UI 连接到某个 调试器 或 runtime。
* 指定一个调试器扩展所支持的语言。
* 为 调试器使用的 Debug 配置属性 提供 丰富的智能感知提示 和 鼠标悬停提示信息。
* 提供 Debug 配置片段。

另一方面，**VS Code** 也提供了一套 [Debug 扩展API](https://code.visualstudio.com/api/references/vscode-api#debug)，让你可以在任意 **VS Code** 调试器上实现调试相关的功能，以此来让用户的体验自动化。

一些 **扩展的灵感**

* 基于 动态创建的 Debug 配置 启动 Debug 会话。
* 跟踪 Debug 会话 的生命周期。
* 以编程方式 创建、管理 断点。

## 扩展准则

为了让你的扩展融入 **VS Code** 的用户界面中，请参阅 [扩展准则](https://code.visualstudio.com/api/references/extension-guidelines) ，其中你可以学到如何实现创建 扩展 UI 的最佳实践、遵循首选的 **VS Code** 的工作流。

## 限制

我们对扩展做了一些限制，下面介绍具体的限制和作出限制的目的

### 不允许访问 DOM

扩展不能访问 **VS Code** UI 的 DOM。

你编写的扩展 **做不到** 将自定义 CSS 样式应用到 **VS Code** UI ，或者向 **VS Code** UI 中添加 HTML 元素。

在 **VS Code** 中，我们不断尝试优化底层网络技术来提供一个 始终可用、高速响应 的编辑器，因为这些技术和我们的产品在不断发展中，我们也会继续调整对 DOM 的使用。为了确保扩展不影响 **VS Code** 的稳定性和性能，并且让我们能在不破坏现有的扩展的情况下改进我们的 DOM，我们阻止了扩展对 DOM 的直接访问，转而在 [扩展宿主](https://code.visualstudio.com/api/advanced-topics/extension-host) 中运行扩展。