# 扩展API —— 概述

[原文链接，戳我前往](https://code.visualstudio.com/api)

------

翻译 by [赫雯勒莉特翡翠](https://github.com/HeveraletLaidCenx)

## 术语~的对照表

|中文（常用英文表述）|英文|
|----|----|
|扩展|extensions|
|应用程序接口（API）|Application Programming Interface|
|界面（UI）|User Interface|
|构建|build|
|运行|run|
|调试（Debug）|debug|
|调试器|debugger|
|测试|test|
|发布|publish|
|最佳实践|best practice|
|主题|theme、theming|
|组件|components|
|视图|views|
|工作台|workbench|
|超文本标记语言（HTML）|HyperText Markup Language|
|层叠样式表（CSS）|Cascading Style Sheet|
|（JS）|JavaScript|
|网页视图（Webview）|Webview|
|编程语言|programming language|
|运行时（runtime）|runtime|
|你好世界（Hello World）|Hello World|
|宿主（host）|host|
|代码空间|Codespaces|
|计划中的API|Proposed API|
|作用点|Contribution Points|
|发布版本|release|
|编写|authoring|
|开发者、开发人员|developer|
|问题（IS）|issue|

表中部分：

* 在中文表述中常直接用英文替代的
* 认为直译并不合适的

在中文之后的括号中说明了直接使用对应的英文。

------

## 概述

**Visual Studio Code** 在构建的时候就考虑了扩展性。从 UI 到编辑体验，**VS Code** 的所有部分几乎都能通过 **扩展API** 来自定义和增强。而事实上，**VS Code** 的很多核心功能功能也是作为 [扩展](https://github.com/microsoft/vscode/tree/main/extensions) ，以 **扩展API** 构建的。

这系列文档的内容包括：

* 如何构建、运行、Debug、测试、发布一个扩展
* 如何利用 **VS Code** 提供的丰富的扩展API
* 在哪儿能找到帮助你入门的 [指导手册](https://code.visualstudio.com/api/extension-guides/overview) 和 [代码示例](https://github.com/microsoft/vscode-extension-samples)
* 为了达到最佳实践，请遵循我们的 [扩展准则](https://code.visualstudio.com/api/references/extension-guidelines)

> 如果你在寻找已经发布的扩展，请前往 [**VS Code** 扩展市场](https://marketplace.visualstudio.com/vscode) 。

## 扩展能做到什么？

这里展示一些可以用 **扩展API** 实现的功能示例：

* 用颜色或者文件图标主题来更改 **VS Code** 的外观：[主题](https://code.visualstudio.com/api/extension-capabilities/theming)
* 在 UI 中添加自定义的 **组件** 和 **视图**：[扩展 VS Code 的工作台](https://code.visualstudio.com/api/extension-capabilities/extending-workbench)
* 创建一个 **Webview** 来显示用 HTML/CSS/JS 构建的自定义网页：[Webview 指导](https://code.visualstudio.com/api/extension-guides/webview)
* 添加对一种编程语言的支持：[语言扩展概述](https://code.visualstudio.com/api/language-extensions/overview)
* 对某个特定的 **runtime** 提供 Debug 支持：[调试器扩展指导](https://code.visualstudio.com/api/extension-guides/debugger-extension)

如果想对 **扩展API** 有一个更全面的了解，请参阅 [扩展功能概述](https://code.visualstudio.com/api/extension-capabilities/overview) 。

而在 [扩展指导概述](https://code.visualstudio.com/api/extension-guides/overview) 中，还有一个包括了代码示例和指导的列表，展示了各种 **扩展API** 的用法。

## 如何构建扩展？

构建一个好的扩展需要付出大量的努力。

这些是 **扩展API** 文档的各个部分能给你提供的帮助：

* **入门** —— 用国际惯例的 [Hello World](https://github.com/microsoft/vscode-extension-samples/tree/main/helloworld-sample) 示例来展示构建扩展的一些基本概念。
* **扩展功能** —— 将庞大的 **扩展API** 分为更小的类别，把你引导到更详细的文章。
* **扩展指导** —— 包括解释了 **扩展API** 特定用法的指导和代码示例。
* **语言扩展** —— 包括说明了如何添加对一种编程语言的支持的指导和代码示例。
* **测试与发布** —— 包括对各种扩展开发文章的深入指导，比如 [测试](https://code.visualstudio.com/api/working-with-extensions/testing-extension) 和 [发布](https://code.visualstudio.com/api/working-with-extensions/publishing-extension) 扩展。
* **高级文章** —— 解释了如 [扩展 host](https://code.visualstudio.com/api/advanced-topics/extension-host)、[支持远程开发和 Github 代码空间](https://code.visualstudio.com/api/advanced-topics/remote-extensions)、[计划中的API](https://code.visualstudio.com/api/advanced-topics/using-proposed-api) 等高级概念。
* **参考资料** —— 包括对 [VS Code API](https://code.visualstudio.com/api/references/vscode-api)、[作用点](https://code.visualstudio.com/api/references/contribution-points) 以及许多其它文章的详尽参考资料。

## 更新了什么内容？

**VS Code** 每月更新，而这也同样适用于 **扩展API** 。每月会实装新功能和 API ，让 **VS Code** 的扩展能有更多的功能和更广的使用范围。

想了解 **扩展API** 的最新情况的话，可以查看每月发布的版本说明，其中有专门的章节，包括：

* [扩展编写](https://code.visualstudio.com/updates#_extension-authoring) —— 了解最新版本中，有哪些新的 **扩展API** 可用。
* [计划中的 **扩展API**](https://code.visualstudio.com/updates#_proposed-extension-apis) —— 查看即将实装的计划中的API，并提交反馈

## 寻求帮助

如果对扩展开发有疑问，可以试试到下面的地方提问：

* [Stack Overflow](https://stackoverflow.com/questions/tagged/visual-studio-code) —— 有 [超多的问题](https://stackoverflow.com/questions/tagged/visual-studio-code) 带有 `visual-studio-code` 标签，而其中超过一半已经有了答案。搜索你的问题，提问，或回答有关 **VS Code** 扩展开发的问题来帮助同是天涯沦落人的开发者同伴。
* [Gitter Channel](https://gitter.im/Microsoft/vscode) 和 [VS Code Dev Slack](https://aka.ms/vscode-dev-community) —— 扩展开发者的公共聊天室。**VS Code** 团队的成员也经常会下场加入讨论。

> 如果要提交有关文档的反馈，请在 [Microsoft/vscode-docs](https://github.com/microsoft/vscode-docs/issues) 上发起新 issue。而如果是关于扩展的问题无法找到答案，或者关于 **扩展API** 的问题，请到 [Microsoft/vscode](https://github.com/microsoft/vscode/issues) 提交issue。
