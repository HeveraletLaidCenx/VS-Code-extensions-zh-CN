# 代码片段指导

[原文链接，戳我前往](https://code.visualstudio.com/api/language-extensions/snippet-guide)

------

翻译 by [赫雯勒莉特翡翠](https://github.com/HeveraletLaidCenx)

## 术语~的对照表

|中文（常用英文表述）|英文|
|----|----|
|||

表中部分：

* 在中文表述中常直接用英文替代的
* 认为直译并不合适的

在中文之后的括号中说明了直接使用对应的英文。

------

## 概述

[`contributes.snippets`（建立作用点.代码片段）](https://code.visualstudio.com/api/references/contribution-points#contributes.snippets) 作用点允许你将 代码片段 打包到 **Visual Studio Code** 扩展 中以用于共享。

[创建代码片段](https://code.visualstudio.com/docs/editor/userdefinedsnippets#_creating-your-own-snippets) 文章中包含了与 创建代码片段 相关的所有信息。本篇指导及示例将仅向你展示 如何将你的 代码片段 转换成扩展以便共享。

推荐的工作流程是：

* 使用 `Preferences: Configure User Snippets`（首选项: 配置用户代码片段） 命令 创建并测试你的 代码片段
* 当你对代码片段满意后，将完整的 JSON 文件，比如 `snippets.json` 复制到扩展文件夹
* 在你的扩展的 `package.json` 中添加以下 代码片段作用点：

```json
{
  "contributes": {
    "snippets": [
      {
        "language": "javascript",
        "path": "./snippets.json"
      }
    ]
  }
}
```

你可以在 [这里](https://github.com/microsoft/vscode-extension-samples/tree/main/snippet-sample) 找到完成的源代码。

**提示**：通过在 `package.json` 中使用以下配置可以将你的扩展标记为一个 代码片段扩展：

```json
{
  "categories": ["Snippets"]
}
```

## 使用 TextMate 代码片段

你也可以通过使用 [yo code 扩展生成器](../2-入门/1-扩展API-入门-你的第一个扩展.md) 来将 TextMate 代码片段（.tmSnippets格式的文件）添加到你的 **VS Code** 扩展中。

扩展生成器中有一个 `New Code Snippets`（新的代码片段） 选项，让你可以指向一个 包含多个 `.tmSnippets` 格式文件的文件夹，它们将被打包到一个 **VS Code** 代码片段扩展 中。生成器还支持Sublime 代码片段（.sublime-snippets格式的文件）。

生成器最终会输出两个文件：一份扩展清单 `package.json` ，包含将代码集成到 **VS Code** 的元数据 和一个 `snippets.json` 文件，包含了转换为 **VS Code** 代码片段格式的 代码片段。

```text
.
├─ snippets                    // VS Code 集成
│   └─ snippets.json           // The JSON file w/ the snippets
└─ package.json                // 扩展的清单
```

最后将生成的 代码片段文件夹 复制到你的 `.vscode/extensions` 文件夹中，然后重启 **VS Code**。
