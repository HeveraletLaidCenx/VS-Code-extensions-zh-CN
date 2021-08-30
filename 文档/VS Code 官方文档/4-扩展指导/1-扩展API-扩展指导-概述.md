# 扩展指导

[原文链接，戳我前往](https://code.visualstudio.com/api/extension-guides/overview)

------

翻译 by [赫雯勒莉特翡翠](https://github.com/HeveraletLaidCenx)

## 术语~的对照表

|中文（常用英文表述）|英文|
|----|----|
|串行器|serializer|
|虚拟的|virtual|
|壳（Shell）|shell|
|源代码控制、源代码管理|source control|
|源代码控制管理器（SCM）|Source Control Manager|
|装饰器|decoractor|
|可主题化的|themable|
|国际化（i18n）【奇怪的缩写增加了，如果是大写的话，首字母是I不是L，这个词是因为原来的单词比较长，所以取了首尾字母，中间的18是中间的字符数，真有你们的.jpg】|internationalization|
|终端|terminal|
|语言服务器协议（LSP）【特别注明，LSP这个缩写在计算机领域还有其他意思，避免混淆】|Language Server Protocol|

表中部分：

* 在中文表述中常直接用英文替代的
* 认为直译并不合适的

在中文之后的括号中说明了直接使用对应的英文。

------

## 概述

在完成了新手教学任务 [Hello World 例子](https://code.visualstudio.com/api/get-started/your-first-extension) ，了解了 **Visual Studio Code** 扩展API 的基础知识之后，是时候来点实战内容啦 —— 构建一些实际的扩展。

在上一章， [扩展功能](https://code.visualstudio.com/api/extension-capabilities/overview) 中，提供了一些展示扩展 **做得到** 什么的高级概述，而本节将包括一个含有 详细代码指导 和 示例 的列表，展示 **怎么用** 某个特定的 **VS Code** API。

每个 指导 和 示例 中，包括如下内容：

* 对源代码的透彻的注释。
* 展示示例扩展用法的 gif 动态图片 或 图片。
* 运行示例扩展的说明。
* 示例扩展中用到的 **VS Code** 扩展API 的列表。
* 示例扩展中用到的 作用点 的列表。
* 类似于示例扩展的实际存在的扩展。
* 对 扩展 API 概念的解释。

## 指导 和 示例

以下是在 **VS Code** 网站上的指导，包括它们使用的 [**VS Code 扩展API**](https://code.visualstudio.com/api/references/vscode-api) 和 [**作用点**](https://code.visualstudio.com/api/references/contribution-points) 。

![扩展准则](../../../文档/VS%20Code%20官方文档/扩展准则.png)

还有一件事！ —— 别忘了参考 [扩展准则](https://code.visualstudio.com/api/references/extension-guidelines) 来了解创建扩展的最佳实践！

|**VS Code** 网站上的指导|扩展API 和 作用点|
|----|----|
|[命令](https://code.visualstudio.com/api/extension-guides/command)|[commands](https://code.visualstudio.com/api/references/vscode-api#commands)（命令）|
|[颜色主题](https://code.visualstudio.com/api/extension-guides/color-theme)|[contributes.themes](https://code.visualstudio.com/api/references/contribution-points#contributes.themes)（建立作用点.主题）|
|[文件图标主题](https://code.visualstudio.com/api/extension-guides/file-icon-theme)|[contributes.iconThemes](https://code.visualstudio.com/api/references/contribution-points#contributes.iconThemes)（建立作用点.图标主题）|
|[产品图标主题](https://code.visualstudio.com/api/extension-guides/product-icon-theme)|[contributes.productIconThemes](https://code.visualstudio.com/api/references/contribution-points#contributes.productIconThemes)（建立作用点.产品图标主题）|
|[树视图](https://code.visualstudio.com/api/extension-guides/tree-view)|[window.createTreeView](https://code.visualstudio.com/api/references/vscode-api#window.createTreeView)（窗口.创建树视图） </br>[window.registerTreeDataProvider](https://code.visualstudio.com/api/references/vscode-api#window.registerTreeDataProvider)（窗口.注册树的数据提供程序） </br>[TreeView](https://code.visualstudio.com/api/references/vscode-api#TreeView)（树视图） </br>[TreeDataProvider](https://code.visualstudio.com/api/references/vscode-api#TreeDataProvider)（树的数据提供器） </br>[contributes.views](https://code.visualstudio.com/api/references/contribution-points#contributes.views)（建立作用点.视图） </br>[contributes.viewsContainers](https://code.visualstudio.com/api/references/contribution-points#contributes.viewsContainers)（建立作用点.视图容器）|
|[Webview](https://code.visualstudio.com/api/extension-guides/webview)|[window.createWebviewPanel](https://code.visualstudio.com/api/references/vscode-api#window.createWebviewPanel) （窗口.创建 Webview 面板）</br>[window.registerWebviewPanelSerializer](https://code.visualstudio.com/api/references/vscode-api#window.registerWebviewPanelSerializer)（窗口.注册 Webview 面板串行器）|
|[自定义编辑器](https://code.visualstudio.com/api/extension-guides/custom-editors)|[window.registerCustomEditorProvider](https://code.visualstudio.com/api/references/vscode-api#window.registerCustomEditorProvider) （窗口.注册自定义编辑器提供程序）</br>[CustomTextEditorProvider](https://code.visualstudio.com/api/references/vscode-api#CustomTextEditorProvider) （自定义文本编辑器提供程序）</br>[contributes.customEditors](https://code.visualstudio.com/api/references/contribution-points#contributes.customEditors)（建立作用点.自定义编辑器）|
|[虚拟文档](https://code.visualstudio.com/api/extension-guides/virtual-documents)|[workspace.registerTextDocumentContentProvider](https://code.visualstudio.com/api/references/vscode-api#workspace.registerTextDocumentContentProvider) （工作区.注册文本文档内容提供程序）</br>[commands.registerCommand](https://code.visualstudio.com/api/references/vscode-api#commands.registerCommand)（命令.注册命令） </br>[window.showInputBox](https://code.visualstudio.com/api/references/vscode-api#window.showInputBox)（窗口.显示输入框）|
|[工作区信任](https://code.visualstudio.com/api/extension-guides/workspace-trust)|[workspace.isTrusted](https://code.visualstudio.com/api/references/vscode-api#workspace.isTrusted)（工作区.受信任的） </br>[workspace.onDidGrantWorkspaceTrust](https://code.visualstudio.com/api/references/vscode-api#workspace.onDidGrantWorkspaceTrust)（工作区.当工作区曾被信任过时触发） </br>capabilities.untrustedWorkspaces（功能.不受信任的工作区）|
|[任务提供程序](https://code.visualstudio.com/api/extension-guides/task-provider)|[tasks.registerTaskProvider](https://code.visualstudio.com/api/references/vscode-api#tasks.registerTaskProvider)（任务.注册任务提供程序） </br>[Task](https://code.visualstudio.com/api/references/vscode-api#Task)（任务） </br>[ShellExecution](https://code.visualstudio.com/api/references/vscode-api#ShellExecution)（通过 Shell 执行） </br>[contributes.taskDefinitions](https://code.visualstudio.com/api/references/contribution-points#contributes.taskDefinitions)（建立作用点.任务定义）|
|[源代码控制](https://code.visualstudio.com/api/extension-guides/scm-provider)|[workspace.workspaceFolders](https://code.visualstudio.com/api/references/vscode-api#workspace.workspaceFolders)（工作区.工作区文件夹） </br>[SourceControl](https://code.visualstudio.com/api/references/vscode-api#SourceControl)（源代码控制） </br>[SourceControlResourceGroup](https://code.visualstudio.com/api/references/vscode-api#SourceControlResourceGroup)（源代码控制资源组） </br>[scm.createSourceControl](https://code.visualstudio.com/api/references/vscode-api#scm.createSourceControl)（源代码控制管理器.创建源代码控制） </br>[TextDocumentContentProvider](https://code.visualstudio.com/api/references/vscode-api#TextDocumentContentProvider)（文本文档内容提供程序） </br>[contributes.menus](https://code.visualstudio.com/api/references/contribution-points#contributes.menus)（建立作用点.菜单）|
|[Debug 扩展](https://code.visualstudio.com/api/extension-guides/debugger-extension)|[contributes.breakpoints](https://code.visualstudio.com/api/references/contribution-points#contributes.breakpoints)（建立作用点.断点） </br>[contributes.debuggers](https://code.visualstudio.com/api/references/contribution-points#contributes.debuggers)（建立作用点.调试器） </br>[debug](https://code.visualstudio.com/api/references/vscode-api#debug)（Debug）|
|[Markdown 扩展](https://code.visualstudio.com/api/extension-guides/markdown-extension)|markdown.previewStyles（markdown.预览样式） </br>markdown.markdownItPlugins（markdown.markdown插件） </br>markdown.previewScripts（markdown.预览脚本）|
|[测试扩展](https://code.visualstudio.com/api/extension-guides/testing)|[TestController](https://code.visualstudio.com/api/references/vscode-api#TestController)（测试控制器） </br>[TestItem](https://code.visualstudio.com/api/references/vscode-api#TestItem)（测试项目）|
|[自定义数据扩展](https://code.visualstudio.com/api/extension-guides/custom-data-extension)|contributes.html.customData（建立作用点.HTML.自定义数据） </br>contributes.css.customData（建立作用点.CSS.自定义数据）|

下边这些是[VS Code 扩展示例仓库](https://github.com/microsoft/vscode-extension-samples) 中更多的代码的列表：

|GitHub 仓库中的示例|扩展API 和 作用点|
|----|----|
|[Webview 示例](https://github.com/microsoft/vscode-extension-samples/tree/main/webview-sample)|[window.createWebviewPanel](https://code.visualstudio.com/api/references/vscode-api#window.createWebviewPanel)（窗口.创建 Webview 面板） </br>[window.registerWebviewPanelSerializer](https://code.visualstudio.com/api/references/vscode-api#window.registerWebviewPanelSerializer)（窗口.注册 Webview 面板串行器）|
|[状态栏示例](https://github.com/microsoft/vscode-extension-samples/tree/main/statusbar-sample)|[window.createStatusBarItem](https://code.visualstudio.com/api/references/vscode-api#window.createStatusBarItem)（窗口.创建状态栏项目） </br>[StatusBarItem](https://code.visualstudio.com/api/references/vscode-api#StatusBarItem)（状态栏项目）|
|[树视图示例](https://github.com/microsoft/vscode-extension-samples/tree/main/tree-view-sample)|[window.createTreeView](https://code.visualstudio.com/api/references/vscode-api#window.createTreeView)（窗口.创建树视图） </br>[window.registerTreeDataProvider](https://code.visualstudio.com/api/references/vscode-api#window.registerTreeDataProvider)（窗口.注册树的数据提供程序） </br>[TreeView](https://code.visualstudio.com/api/references/vscode-api#TreeView)（树视图） </br>[TreeDataProvider](https://code.visualstudio.com/api/references/vscode-api#TreeDataProvider)（树的数据提供程序） </br>[contributes.views](https://code.visualstudio.com/api/references/contribution-points#contributes.views)（建立作用点.视图） </br>[contributes.viewsContainers](https://code.visualstudio.com/api/references/contribution-points#contributes.viewsContainers)（建立作用点.视图容器）|
|[任务提供程序示例](https://github.com/microsoft/vscode-extension-samples/tree/main/task-provider-sample)|[tasks.registerTaskProvider](https://code.visualstudio.com/api/references/vscode-api#tasks.registerTaskProvider)（任务.注册任务提供程序） </br>[Task](https://code.visualstudio.com/api/references/vscode-api#Task)（任务） </br>[ShellExecution](https://code.visualstudio.com/api/references/vscode-api#ShellExecution)（通过 Shell 执行） </br>[contributes.taskDefinitions](https://code.visualstudio.com/api/references/contribution-points#contributes.taskDefinitions)（建立作用点.任务定义）|
|[多根源（源文件夹）示例](https://github.com/microsoft/vscode-extension-samples/tree/main/basic-multi-root-sample)|[workspace.getWorkspaceFolder](https://code.visualstudio.com/api/references/vscode-api#workspace.getWorkspaceFolder)（工作区.获取工作区文件夹） </br>[workspace.onDidChangeWorkspaceFolders](https://code.visualstudio.com/api/references/vscode-api#workspace.onDidChangeWorkspaceFolders)（工作区.当工作区文件夹发生改变时触发）|
|[自动补全提供程序示例](https://github.com/microsoft/vscode-extension-samples/tree/main/completions-sample)|[languages.registerCompletionItemProvider](https://code.visualstudio.com/api/references/vscode-api#languages.registerCompletionItemProvider)（语言.注册自动补全项目提供程序） </br>[CompletionItem](https://code.visualstudio.com/api/references/vscode-api#CompletionItem)（自动补全项目） </br>[SnippetString](https://code.visualstudio.com/api/references/vscode-api#SnippetString)（片段字符串）|
|[文件系统提供程序示例](https://github.com/microsoft/vscode-extension-samples/tree/main/fsprovider-sample)|[workspace.registerFileSystemProvider](https://code.visualstudio.com/api/references/vscode-api#workspace.registerFileSystemProvider)（工作区.注册文件系统提供程序）|
|[编辑器装饰器示例](https://github.com/microsoft/vscode-extension-samples/tree/main/decorator-sample)|[TextEditor.setDecorations](https://code.visualstudio.com/api/references/vscode-api#TextEditor.setDecorations)（文本编辑器.设置装饰器） </br>[DecorationOptions](https://code.visualstudio.com/api/references/vscode-api#DecorationOptions)（装饰器选项） </br>[DecorationInstanceRenderOptions](https://code.visualstudio.com/api/references/vscode-api#DecorationInstanceRenderOptions)（装饰器实例渲染选项）</br> [ThemableDecorationInstanceRenderOptions](https://code.visualstudio.com/api/references/vscode-api#ThemableDecorationInstanceRenderOptions)（可主题化的装饰器实例渲染选项） </br>[window.createTextEditorDecorationType](https://code.visualstudio.com/api/references/vscode-api#window.createTextEditorDecorationType)（窗口.创建文本编辑器装饰器类型） </br>[TextEditorDecorationType](https://code.visualstudio.com/api/references/vscode-api#TextEditorDecorationType)（文本编辑器装饰器类型） </br>[contributes.colors](https://code.visualstudio.com/api/references/contribution-points#contributes.colors)（建立作用点.颜色）|
|[国际化示例](https://github.com/microsoft/vscode-extension-samples/tree/main/i18n-sample)|【不是忘了写，是原文这里就没有东西】|
|[终端示例](https://github.com/microsoft/vscode-extension-samples/tree/main/terminal-sample)|[window.createTerminal](https://code.visualstudio.com/api/references/vscode-api#window.createTerminal)（窗口.创建终端） </br>[window.onDidChangeActiveTerminal](https://code.visualstudio.com/api/references/vscode-api#window.onDidChangeActiveTerminal)（窗口.当 活跃的终端 改变了的时候触发） </br>[window.onDidCloseTerminal](https://code.visualstudio.com/api/references/vscode-api#window.onDidCloseTerminal)（窗口.当关闭终端的时候触发） </br>[window.onDidOpenTerminal](https://code.visualstudio.com/api/references/vscode-api#window.onDidOpenTerminal)（窗口.在启动终端时触发） </br>[window.Terminal](https://code.visualstudio.com/api/references/vscode-api#window.Terminal)（窗口.终端） </br>[window.terminals](https://code.visualstudio.com/api/references/vscode-api#window.terminals)（窗口.终端）|
|[Vim 示例](https://github.com/microsoft/vscode-extension-samples/tree/main/vim-sample)|[commands](https://code.visualstudio.com/api/references/vscode-api#commands)（命令） </br>[StatusBarItem](https://code.visualstudio.com/api/references/vscode-api#StatusBarItem)（状态栏项目） </br>[window.createStatusBarItem](https://code.visualstudio.com/api/references/vscode-api#window.createStatusBarItem)（窗口.创建状态栏项目） </br>[TextEditorCursorStyle](https://code.visualstudio.com/api/references/vscode-api#TextEditorCursorStyle)（文本编辑器光标样式） </br>[window.activeTextEditor](https://code.visualstudio.com/api/references/vscode-api#window.activeTextEditor)（窗口.活跃的文本编辑器） </br>[Position](https://code.visualstudio.com/api/references/vscode-api#Position)（位置） </br>[Range](https://code.visualstudio.com/api/references/vscode-api#Range)（范围） </br>[Selection](https://code.visualstudio.com/api/references/vscode-api#Selection)（选择） </br>[TextEditor](https://code.visualstudio.com/api/references/vscode-api#TextEditor)（文本编辑器） </br>[TextEditorRevealType](https://code.visualstudio.com/api/references/vscode-api#TextEditorRevealType)（文本编辑器显示类型） </br>[TextDocument](https://code.visualstudio.com/api/references/vscode-api#TextDocument)（文本文档）|
|[源代码控制示例](https://github.com/microsoft/vscode-extension-samples/tree/main/source-control-sample)|[workspace.workspaceFolders](https://code.visualstudio.com/api/references/vscode-api#workspace.workspaceFolders)（工作区.工作区文件夹） </br>[SourceControl](https://code.visualstudio.com/api/references/vscode-api#SourceControl)（源代码控制） </br>[SourceControlResourceGroup](https://code.visualstudio.com/api/references/vscode-api#SourceControlResourceGroup)（源代码控制资源组） </br>[scm.createSourceControl](https://code.visualstudio.com/api/references/vscode-api#scm.createSourceControl)（源代码控制管理器.创建源代码控制） </br>[TextDocumentContentProvider](https://code.visualstudio.com/api/references/vscode-api#TextDocumentContentProvider)（文本文档内容提供程序） </br>[contributes.menus](https://code.visualstudio.com/api/references/contribution-points#contributes.menus)（建立作用点.菜单）|
|[注释 API 示例](https://github.com/microsoft/vscode-extension-samples/tree/main/comment-sample)|【不是忘了写，是原文这里就没有东西】|
|[文档编辑示例](https://github.com/microsoft/vscode-extension-samples/tree/main/document-editing-sample)|[commands](https://code.visualstudio.com/api/references/vscode-api#commands)（命令）|
|[入门示例](https://github.com/microsoft/vscode-extension-samples/tree/main/getting-started-sample)|[contributes.walkthroughs](https://code.visualstudio.com/api/references/contribution-points#contributes.walkthroughs)（建立作用点.演练）|
|[测试扩展](https://github.com/microsoft/vscode-extension-samples/tree/main/test-provider-sample)|[TestController](https://code.visualstudio.com/api/references/vscode-api#TestController)（测试控制器） </br>[TestItem](https://code.visualstudio.com.azurewebsites.net/api/references/vscode-api#TestItem)（测试项目）|

## 语言扩展示例

下面这些是 [语言扩展](https://code.visualstudio.com/api/language-extensions/overview) 示例：

|示例|VS Code 网站上的指导|
|----|----|
|[代码片段示例](https://github.com/microsoft/vscode-extension-samples/tree/main/snippet-sample)|[/api/language-extensions/snippet-guide](https://code.visualstudio.com/api/language-extensions/snippet-guide)|
|[语言配置示例](https://github.com/microsoft/vscode-extension-samples/tree/main/language-configuration-sample)|[/api/language-extensions/language-configuration-guide](https://code.visualstudio.com/api/language-extensions/language-configuration-guide)|
|[语言服务器（LSP）示例](https://github.com/microsoft/vscode-extension-samples/tree/main/lsp-sample)|[/api/language-extensions/language-server-extension-guide](https://code.visualstudio.com/api/language-extensions/language-server-extension-guide)|
|[语言服务器（LSP）日志流示例](https://github.com/microsoft/vscode-extension-samples/tree/main/lsp-log-streaming-sample)|N/A（不适用）【原文就是这样】|
|[语言服务器（LSP）多根源服务器示例](https://github.com/microsoft/vscode-extension-samples/tree/main/lsp-multi-server-sample)|[https://github.com/microsoft/vscode/wiki/Adopting-Multi-Root-Workspace-APIs#language-client--language-server](https://github.com/microsoft/vscode/wiki/Adopting-Multi-Root-Workspace-APIs#language-client--language-server)|
