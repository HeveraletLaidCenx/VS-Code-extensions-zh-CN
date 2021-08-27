# 通用功能

[原文链接，戳我前往](https://code.visualstudio.com/api/extension-capabilities/common-capabilities)

------

翻译 by [赫雯勒莉特翡翠](https://github.com/HeveraletLaidCenx)

## 术语~的对照表

|中文（常用英文表述）|英文|
|----|----|
|块|blocks|
|扩展特定的|extension-specific|
|数据存储|data storage|
|键值对|key/value pairs|
|指向|pointing to|
|本地的|local|
|路径|path|
|状态|states|
|共享|sharing|
|已经不理会的|dismissed|
|已经查看的|viewed|
|标志、旗帜|flags|
|实例|instances|
|输出通道|Output Channel|
|日志、记录（Log）|log|

表中部分：

* 在中文表述中常直接用英文替代的
* 认为直译并不合适的

在中文之后的括号中说明了直接使用对应的英文。

------

## 概述

通用功能 是你的扩展的重要的构建块。几乎所有扩展都使用了其中的一些功能。下面展示了你该如何使用它们。

## 命令

命令是 **VS Code** 运行的核心。你可以打开命令面板来执行命令，给命令绑定自定义的键位，右键点击来调用上下文菜单中的命令。

扩展可以：

* 用 [vscode.commands](https://code.visualstudio.com/api/references/vscode-api#commands)（vscode.命令） API 注册、执行命令。
* 用 [contributes.commands](https://code.visualstudio.com/api/references/contribution-points#contributes.commands)（建立作用点.命令） 作用点，使命令在命令面板中可用。

在 [扩展指导 / 命令](https://code.visualstudio.com/api/extension-guides/command) 文章中获得有关命令的更多信息。

## 配置

扩展可以通过 [contributes.configuration](https://code.visualstudio.com/api/references/contribution-points#contributes.configuration)（建立作用点.配置） 作用点 来建立扩展特定的设置，并通过 [workspace.getConfiguration](https://code.visualstudio.com/api/references/vscode-api#workspace.getConfiguration)（工作区.获取配置） API 来读取它们。

## 键位绑定

扩展可以添加自定义的键位绑定。详情请参阅 [contributes.keybindings](https://code.visualstudio.com/api/references/contribution-points#contributes.keybindings)（建立作用点.键位绑定） 和 [键位绑定](https://code.visualstudio.com/docs/getstarted/keybindings) 相关的文章。

## 上下文菜单

扩展可以注册自定义上下文菜单项，将在用户在 **VS Code** UI 的不同位置右键点击呼出的菜单中显示。更多内容请参阅 [contributes.menus](https://code.visualstudio.com/api/references/contribution-points#contributes.menus)（建立作用点.菜单） 作用点。

## 数据存储

对于存储数据，有4个选择：

* [ExtensionContext.workspaceState](https://code.visualstudio.com/api/references/vscode-api#ExtensionContext.workspaceState)（扩展上下文.工作区状态）: 一个可以让你在其中记录键值对的工作区存储。**VS Code** 会管理这部分存储，并在再次打开同一个工作区的时候恢复它们。
* [ExtensionContext.globalState](https://code.visualstudio.com/api/references/vscode-api#ExtensionContext.globalState)（扩展上下文.全局状态）: 一个可以让你在其中记录键值对的全局存储。**VS Code** 会管理这部分存储，并在每次扩展激活的时候恢复它。你可以在 `globalState` 中的 `setKeysForSync` 方法里设置同步的键，来选择性地在全局存储中同步键值对。
* [ExtensionContext.storagePath](https://code.visualstudio.com/api/references/vscode-api#ExtensionContext.storagePath)（扩展上下文.存储路径）: 一个指向你的扩展拥有读写访问权限的，工作区特定的存储路径。如果你需要存储只从当前工作区访问的大文件，这是个不错的选择。
* [ExtensionContext.globalStoragePath](https://code.visualstudio.com/api/references/vscode-api#ExtensionContext.globalStoragePath)（扩展上下文.全局存储路径）: 一个指向你的扩展拥有读写访问权限的全局存储路径。如果你需要存储所有工作区都可以访问的大文件，这是个不错的选择。

`ExtensionContext`（扩展上下文）可以在 [扩展入口文件](https://code.visualstudio.com/api/get-started/extension-anatomy#extension-entry-file) 中的 `activate` 函数里找到。

### setKeysForSync（为同步设置键） 示例

如果你的扩展需要保存一些跨不同设备的用户状态，那么使用 `vscode.ExtensionContext.globalState.setKeysForSync`（vscode.扩展上下文.全局状态.为同步设置键） 来把状态提供给 [设置同步](https://code.visualstudio.com/docs/editor/settings-sync) 。

可以使用下面的格式

```typescript
// 激活时
const versionKey = context.extension.id + '.version';
context.globalState.setKeysForSync([versionKey]);

// 稍后显示页面
const currentVersion = context.extension.packageJSON.version;
const lastVersionShown = context.globalState.get(versionKey);
if (!isHigher(currentVersion, lastVersionShown)) {
    return;
}
context.globalState.set(versionKey, currentVersion);
// 显示页面
```

通过跨设备共享状态，共享 已经不理会的 或者 已经查看的 标志，可以帮助避免用户看到多个欢迎或者升级页面实例的问题。

## 显示通知

几乎所有扩展都会需要在某些情况下向用户显示信息。**VS Code** 为显示不同严重程度的通知消息提供了三个 API ：

* 信息消息： [window.showInformationMessage](https://code.visualstudio.com/api/references/vscode-api#window.showInformationMessage)（窗口.显示信息消息）
* 警告消息： [window.showWarningMessage](https://code.visualstudio.com/api/references/vscode-api#window.showWarningMessage)（窗口.显示警告消息）
* 错误消息：[window.showErrorMessage](https://code.visualstudio.com/api/references/vscode-api#window.showErrorMessage)（窗口.显示错误消息）

## 快速选取

通过 [vscode.QuickPick](https://code.visualstudio.com/api/references/vscode-api#QuickPick)（vscode.快速选取） API，你可以轻松地收集用户的输入，或者让用户从多个选项中进行选择。可以查看这个展示它用法的例子： [快速输入示例](https://github.com/microsoft/vscode-extension-samples/tree/main/quickinput-sample) 。

## 文件选择器

扩展可以使用 [vscode.window.showOpenDialog](https://code.visualstudio.com/api/references/vscode-api#vscode.window.showOpenDialog)（vscode.窗口.显示打开文件对话） API 来打开系统的文件选择器，来选择文件或者文件夹。

## 输出通道

输出面板会显示 [输出通道](https://code.visualstudio.com/api/references/vscode-api#OutputChannel) 的集，非常适合用来 Log 。你可以通过 [window.createOutputChannel](https://code.visualstudio.com/api/references/vscode-api#window.createOutputChannel)（窗口.创建输出通道） API 来轻松地使用它。

## 进度 API

你可以使用 [vscode.Progress](https://code.visualstudio.com/api/references/vscode-api#Progress)（vscode.进度） API 来向用户展示进度更新的情况。

通过使用 [ProgressLocation](https://code.visualstudio.com/api/references/vscode-api#ProgressLocation)（进度位置） 选项，可以将进度显示在不同的位置：

* 在通知区域中
* 在源代码控制视图中
* **VS Code** 窗口中的一般进度

可以查看 [进度示例](https://github.com/microsoft/vscode-extension-samples/tree/main/progress-sample) 来了解进度 API 的用法。
