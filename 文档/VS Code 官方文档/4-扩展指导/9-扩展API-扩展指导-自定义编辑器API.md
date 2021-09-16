# 自定义编辑器 API

[原文链接，戳我前往](https://code.visualstudio.com/api/extension-guides/custom-editors)

更新版本：截至2021-09-04

------

翻译 by [赫雯勒莉特翡翠](https://github.com/HeveraletLaidCenx)

## 术语~的对照表

|中文（常用英文表述）|英文|
|----|----|
|着色器|shader|
|所见即所得的（WYSIWYG）|What You See Is What You Get|
|热退出|hot exit|

表中部分：

* 在中文表述中常直接用英文替代的
* 认为直译并不合适的

在中文之后的括号中说明了直接使用对应的英文。

------

## 概述

自定义编辑器 API 让扩展可以创建 完全可自定义 读/写 的编辑器，可以代替 **VS Code** 的标准文本编辑器，用来编辑特定类型的资源。

自定义编辑器 API 有很多使用场景，比如：

* 直接在 **VS Code** 中预览资源，比如 着色器 或者 3D 模型。
* 给 Maekdown 或者 XAML 等语言创建所见即所得的编辑器。
* 给 CSV、JSON、XML 等数据文件提供替代性的视觉渲染。
* 给 二进制文件 或者 文本文件 构建完全可自定义的编程体验。

本文提供了对 自定义编辑器 API 的概述，和实现一个 自定义编辑器 的基础。

我们将带你了解 自定义编辑器 的两种类型，以及它们之间的区别，让你可以了解在你的使用情境中更适合选择哪一种。

然后我们会分别介绍这两种类型的情况下，应该怎么创建一个表现良好的编辑器 的基础知识。

虽然 自定义编辑器 是一个强大的、比较新的可扩展点，但是实现一个基础的 自定义编辑器 其实并不困难~

但——是——，如果你正打算建立你的第一个 **VS Code** 扩展，你可能应该考虑一下推迟对 自定义编辑器 的深入研究 —— 直到你更熟悉 **VS Code** 扩展API 的基础知识。

自定义编辑器 涉及到很多 **VS Code** 的概念 —— 比如 [Webview](https://code.visualstudio.com/api/extension-guides/webview) 和 文本文档 等。所以如果你直接一头扎进 自定义编辑器 的内容中，那你很可能面临 需要同时学习大量新的基础概念 的局面，这可能会把你淹没，让你不知所措。

不过，如果你已经准备好了，然后正在考虑整一个超酷的 自定义编辑器 的话，那我们走起！

请务必下载 [自定义编辑器扩展示例](https://github.com/microsoft/vscode-extension-samples/tree/main/custom-editor-sample) ，这样你就可以跟着本文的思路走，然后看看 自定义编辑器 API 是怎么组合到一起的。

## 相关链接

* [自定义编辑器扩展示例](https://github.com/microsoft/vscode-extension-samples/tree/main/custom-editor-sample)

### 用到的 VS Code 扩展API

* [`window.registerCustomEditorProvider`](https://code.visualstudio.com/api/references/vscode-api#window.registerCustomEditorProvider)（窗口.注册自定义编辑器提供程序）
* [`CustomTextEditorProvider`](https://code.visualstudio.com/api/references/vscode-api#CustomTextEditorProvider)（自定义文本编辑器提供程序）

## 自定义编辑器 API 的基础知识

自定义编辑器 是一种替代性的视图，可以代替 **VS Code** 的标准文本编辑器，用来显示特定类型的资源。

自定义编辑器有两个部分：

* 实现和用户交互的 视图部分
* 用于和底层资源交互的 文档模型部分

视图部分 是通过 [Webview](https://code.visualstudio.com/api/extension-guides/webview) 来实现的。这让你可以用标准 HTML/CSS/JS 来构建 自定义编辑器 的 UI。虽然 Webview 不能直接访问 **VS Code** 扩展API ，但是它可以通过 来回传递消息 来实现和扩展之间的通信。不妨看看 [Webview 文档](https://code.visualstudio.com/api/extension-guides/webview) 来了解关于 Webview 的更多信息 和 使用 Webview 的最佳实践。

而对于 文档模型部分，它负责教会你的扩展 如何理解之后将要处理的 资源文件。`CustomTextEditorProvider`（自定义文本编辑器提供程序） 使用 **VS Code** 的标准 [TextDocument（文本文档）](https://code.visualstudio.com/api/references/vscode-api#TextDocument) 作为它的 文档模型，对文件的所有更改都使用 **VS Code** 的 标准文本编辑API 来表示。另一方面， `CustomReadonlyEditorProvider`（自定义只读编辑器提供程序） 和 `CustomEditorProvider`（自定义编辑器提供程序） 让你可以提供你自己的文档模型，这让它们可以被用作 非文本的文件格式。

自定义编辑器 中，每个资源文件都有一个单独的文档模型，但是这个文档可能会有多个编辑器实例（视图）。举个例子，想象一下，你打开了一个有 `CustomTextEditorProvider`（自定义文本编辑器提供程序） 的文件，然后运行了 **View: Split editor**（视图: 拆分编辑器） 命令。这种情况下，还是只有一个 `TextDocument`（文本文档） ，因为在工作区中只有一个该资源文件的副本，但是执行命令之后你能看到这个资源文件的两个 Webview 视图。

### `CustomEditor` 和 `CustomTextEditor`

自定义编辑器 分为两类：

* 自定义文本编辑器
* 自定义编辑器

二者的主要区别在于它们如何定义它们的文档模型。

`CustomTextEditorProvider`（自定义文本编辑器提供程序） 使用 **VS Code** 的标准 [`TextDocument`（文本文档）](https://code.visualstudio.com/api/references/vscode-api#TextDocument) 作为它的文档模型。你可以把 `CustomTextEditor`（自定义文本编辑器） 用于任何基于文本的文件类型。`CustomTextEditor`（自定义文本编辑器） 相对更容易实现，因为 **VS Code** 已经知道如何处理文本文件，因此可以轻松地实现 保存文件、备份文件 和 热退出 等常规操作。

而对于 `CustomEditorProvider`（自定义编辑器提供程序） ，允许你的扩展使用自己的 文档模型。这意味着你可以把 `CustomEditor`（自定义编辑器） 用于 二进制的文件格式（比如图片），但是这也意味着你的扩展要负责更多内容，包括实现 保存文件、备份文件。如果你的自定义编辑器是 只读 的话（比如一个用于预览的 自定义编辑器），那可以跳过大部分这块的复杂内容。

要想确定你应该选择哪种 自定义编辑器 也很简单：如果你只处理基于文本的文件格式，那就用 `CustomTextEditorProvider`（自定义文本编辑器提供程序） ，如果你需要处理二进制格式的文件，那就用 `CustomEditorProvider`（自定义编辑器提供程序） 。

### 作用点

`customEditors`（自定义编辑器） [作用点](https://code.visualstudio.com/api/references/contribution-points) 是你的扩展用来告诉 **VS Code** ：扩展是如何提供你的自定义编辑器的。比如，**VS Code** 需要知道你的 自定义编辑器 要处理什么类型的文件，也需要知道在任意 UI 中怎么才能识别你的 自定义编辑器。

下边是 [自定义编辑器扩展示例](https://github.com/microsoft/vscode-extension-samples/tree/main/custom-editor-sample) 的一个基础的 `customEditor`（自定义编辑器） 作用点：

```json
"contributes": {
  "customEditors": [
    {
      "viewType": "catEdit.catScratch",
      "displayName": "Cat Scratch",
      "selector": [
        {
          "filenamePattern": "*.cscratch"
        }
      ],
      "priority": "default"
    }
  ]
}
```

`customEditors`（自定义编辑器） 是一个数组所以你的扩展可以建立多个 自定义编辑器 作用点。让我们来分解看看 自定义编辑器 入口本身：

|字段名|翻译|字段含义|
|----|----|----|
|`viewType`|视图类型|你的 自定义编辑器 的唯一ID，它让 **VS Code** 能把 `package.json` 文件中的 自定义编辑器作用点 和 你在代码中实现的自定义编辑器 联系起来。它必须在所有的扩展中都是唯一的，所以相比于很笼统的、常见的、容易重复的 `viewType` （比如 `"preview"`（预览） ），请确保用一个足够独特的名字来区分，比如说： `"myAmazingExtension.svgPreview"`（我的超赞扩展.SVG预览）|
|`displayName`|显示名称|在 **VS Code** UI 中用来标识你的 自定义扩展 的名称，这个名称会显示在用户的 VS Code UI 里，比如下拉菜单中的 **View: Reopen with**（视图: 用 ... 来重新打开）|
|`selector`|选择器|用来区分 处理哪些文件时， 自定义编辑器 才应该处于活跃状态。`selector`（选择器） 是一个is an array of one or more glob patterns. These glob patterns are matched against file names to determine if the custom editor can be used for them. A `filenamePattern` such as `*.png` will enable the custom editor for all PNG files.

  You can also create more specific patterns that match on file or directory names, for example `**/translations/*.json`.

* `priority` * (optional) Specifies when the custom editor is used.

  `priority` controls when a custom editor is used when a resource is open. Possible values are:

  * `"default"` * Try to use the custom editor for every file that matches the custom editor's `selector`. If there are multiple custom editors for a given file, the user will have to select which custom editor they want to use.
  * `"option"` * Do not use the custom editor by default but allow users to switch to it or configure it as their default.

### Custom editor activation[#](https://code.visualstudio.com/api/extension-guides/custom-editors#custom-editor-activation)

When a user opens one of your custom editors, VS Code fires an `onCustomEditor:VIEW_TYPE` activation event. During activation, your extension must call `registerCustomEditorProvider` to register a custom editor with the expected `viewType`.

It's important to note that `onCustomEditor` is only called when VS Code needs to create an instance of your custom editor. If VS Code is merely showing the user some information about an available custom editor—such as with the **View: Reopen with** command—your extension will not be activated.

## Custom Text Editor[#](https://code.visualstudio.com/api/extension-guides/custom-editors#custom-text-editor)

Custom text editors let you create custom editors for text files. This can be anything from plain unstructured text to [CSV](https://en.wikipedia.org/wiki/Comma-separated_values) to JSON or XML. Custom text editors use VS Code's standard [TextDocument](https://code.visualstudio.com/api/references/vscode-api#TextDocument) as their document model.

The [custom editor extension sample](https://github.com/microsoft/vscode-extension-samples/tree/main/custom-editor-sample) includes a simple example custom text editor for cat scratch files (which are just JSON files that end with a `.cscratch` file extension). Let's take a look at some of the important bits of implementing a custom text editor.

### Custom Text Editor lifecycle[#](https://code.visualstudio.com/api/extension-guides/custom-editors#custom-text-editor-lifecycle)

VS Code handles the lifecycle of both the view component of custom text editors (the webviews) and the model component (`TextDocument`). VS Code calls out to your extension when it needs to create a new custom editor instance and cleans up the editor instances and document model when the user closes their tabs.

To understand how this all works in practice, let's work through what happens from an extension's point of view when a user opens a custom text editor and then when a user closes a custom text editor.

**Opening a custom text editor**

Using the [custom editor extension sample](https://github.com/microsoft/vscode-extension-samples/tree/main/custom-editor-sample), here's what happens when the user first opens a `.cscratch` file:

1. VS Code fires an `onCustomEditor:catCustoms.catScratch` activation event.

   This activates our extension if it has not already been activated. During activation, our extension must ensure the extension registers a `CustomTextEditorProvider` for `catCustoms.catScratch` by calling `registerCustomEditorProvider`.

2. VS Code then invokes `resolveCustomTextEditor` on the registered `CustomTextEditorProvider` for `catCustoms.catScratch`.

   This method takes the `TextDocument` for the resource that is being opened and a `WebviewPanel`. The extension must fill in the initial HTML contents for this webview panel.

Once `resolveCustomTextEditor` returns, our custom editor is displayed to the user. What is drawn inside the webview is entirely up to our extension.

This same flow happens every time a custom editor is opened, even when you split a custom editor. Every instance of a custom editor has its own `WebviewPanel`, although multiple custom text editors will share the same `TextDocument` if they are for the same resource. Remember: think of the `TextDocument` as being the model for the resource while the webview panels are views of that model.

**Closing custom text editors**

When a user closes a custom text editor, VS Code fires the `WebviewPanel.onDidDispose` event on the `WebviewPanel`. At this point, your extension should clean up any resources associated with that editor (event subscriptions, file watchers, etc.)

When the last custom editor for a given resource is closed, the `TextDocument` for that resource will also be disposed provided there are no other editors using it and no other extensions are holding onto it. You can check the `TextDocument.isClosed` property to see if the `TextDocument` has been closed. Once a `TextDocument` is closed, opening the same resource using a custom editor will cause a new `TextDocument` to be opened.

### Synchronizing changes with the TextDocument[#](https://code.visualstudio.com/api/extension-guides/custom-editors#synchronizing-changes-with-the-textdocument)

Since custom text editors use a `TextDocument` as their document model, they are responsible for updating the `TextDocument` whenever an edit occurs in a custom editor as well as updating themselves whenever the `TextDocument` changes.

**From webview to `TextDocument`**

Edits in custom text editors can take many different forms—clicking a button, changing some text, dragging some items around. Whenever a user edits the file itself inside the custom text editor, the extension must update the `TextDocument`. Here's how the cat scratch extension implements this:

1. User clicks the **Add scratch** button in the webview. This [posts a message](https://code.visualstudio.com/api/extension-guides/webview#scripts-and-message-passing) from the webview back to the extension.
2. The extension receives the message. It then updates its internal model of the document (which in the cat scratch example just consists of adding a new entry to the JSON).
3. The extension creates a `WorkspaceEdit` that writes the updated JSON to the document. This edit is applied using `vscode.workspace.applyEdit`.

Try to keep your workspace edit to the minimal change required to update the document. Also keep in mind that if you are working with a language such as JSON, your extension should try to observe the user's existing formatting conventions (spaces vs tabs, indent size, etc.).

**From `TextDocument` to webviews**

When a `TextDocument` changes, your extension also needs to make sure its webviews reflect the documents new state. TextDocuments can be changed by user actions such as undo, redo, or revert file; by other extensions using a `WorkspaceEdit`; or by a user who opens the file in VS Code's default text editor. Here's how the cat scratch extension implements this:

1. In the extension, we subscribe to the `vscode.workspace.onDidChangeTextDocument` event. This event is fired for every change to the `TextDocument` (including changes that our custom editor makes!)
2. When a change comes in for a document that we have an editor for, we post a message to the webview with its new document state. This webview then updates itself to render the updated document.

It's important to remember that any file edits that a custom editor triggers will cause `onDidChangeTextDocument` to fire. Make sure your extension does not get into an update loop where the user makes an edit in the webview, which fires onDidChangeTextDocument, which causes the webview to update, which causes the webview to trigger another update on your extension, which fires `onDidChangeTextDocument`, and so on.

Also remember that if you are working with a structured language such as JSON or XML, the document may not always be in a valid state. Your extension must either be able gracefully handle errors or display an error message to the user so that they understand what is wrong and how to fix it.

Finally, if updating your webviews is expensive, consider [debouncing](https://davidwalsh.name/javascript-debounce-function) the updates to your webview.

## Custom Editor[#](https://code.visualstudio.com/api/extension-guides/custom-editors#custom-editor)

`CustomEditorProvider` and `CustomReadonlyEditorProvider` let you create custom editors for binary file formats. This API gives your full control over the file is displayed to users, how edits are made to it, and lets your extension hook into `save` and other file operations. Again, if you are building an editor for a text based file format, strongly consider using a [`CustomTextEditor`](https://code.visualstudio.com/api/extension-guides/custom-editors#custom-text-editor) instead as they are far simpler to implement.

The [custom editor extension sample](https://github.com/microsoft/vscode-extension-samples/tree/main/custom-editor-sample) includes a simple example custom binary editor for paw draw files (which are just jpeg files that end with a `.pawdraw` file extension). Let's take a look at what goes into building a custom editor for binary files.

### CustomDocument[#](https://code.visualstudio.com/api/extension-guides/custom-editors#customdocument)

With custom editors, your extension is responsible for implementing its own document model with the `CustomDocument` interface. This leaves your extension free to store whatever data it needs on a `CustomDocument` in order to your custom editor, but it also means that your extension must implement basic document operations such as saving and backing up file data for hot exit.

There is one `CustomDocument` per opened file. Users can open multiple editors for a single resource—such as by splitting the current custom editor—but all those editors will be backed by the same `CustomDocument`.

### Custom Editor lifecycle[#](https://code.visualstudio.com/api/extension-guides/custom-editors#custom-editor-lifecycle)

**supportsMultipleEditorsPerDocument**

By default, VS Code only allows there to be one editor for each custom document. This limitation makes it easier to correctly implement a custom editor as you do not have to worry about synchronizing multiple custom editor instances with each other.

If your extension can support it however, we recommend setting `supportsMultipleEditorsPerDocument: true` when registering your custom editor so that multiple editor instances can be opened for the same document. This will make your custom editors behave more like VS Code's normal text editors.

**Opening Custom Editors** When the user opens a file that matches the `customEditor` contribution point, VS Code fires an `onCustomEditor` [activation event](https://code.visualstudio.com/api/references/activation-events) and then invokes the provider registered for the provided view type. A `CustomEditorProvider` has two roles: providing the document for the custom editor and then providing the editor itself. Here's a ordered list of what happens for the `catCustoms.pawDraw` editor from the [custom editor extension sample](https://github.com/microsoft/vscode-extension-samples/tree/main/custom-editor-sample):

1. VS Code fires an `onCustomEditor:catCustoms.pawDraw` activation event.

   This activates our extension if it has not already been activated. We must also make sure our extension registers a `CustomReadonlyEditorProvider` or `CustomEditorProvider` for `catCustoms.pawDraw` during activation.

2. VS Code calls `openCustomDocument` on our `CustomReadonlyEditorProvider` or `CustomEditorProvider` registered for `catCustoms.pawDraw` editors.

   Here our extension is given a resource uri and must return a new `CustomDocument` for that resource. This is the point at which our extension should create its document internal model for that resource. This may involve reading and parsing the initial resource state from disk or initializing our new `CustomDocument`.

   Our extension can define this model by creating a new class that implements `CustomDocument`. Remember that this initialization stage is entirely up to extensions; VS Code does not care about any additional information extensions store on a `CustomDocument`.

3. VS Code calls `resolveCustomEditor` with the `CustomDocument` from step 2 and a new `WebviewPanel`.

   Here our extension must fill in the initial html for the custom editor. If we need, we can also hold onto a reference to the `WebviewPanel` so that we can reference it later, for example inside commands.

Once `resolveCustomEditor` returns, our custom editor is displayed to the user.

If the user opens the same resource in another editor group using our custom editor—for example by splitting the first editor—the extension's job is simplified. In this case, VS Code just calls `resolveCustomEditor` with the same `CustomDocument` we created when the first editor was opened.

**Closing Custom Editors**

Say we have two instance of our custom editors open for the same resource. When the user closes these editors, VS Code signals our extension so that it can clean up any resources associated with the editor.

When the first editor instance is closed, VS Code fires the `WebviewPanel.onDidDispose` event on the `WebviewPanel` from the closed editor. At this point, our extension must clean up any resources associated with that specific editor instance.

When the second editor is closed, VS Code again fires `WebviewPanel.onDidDispose`. However now we've also closed all the editors associated with the `CustomDocument`. When there are no more editors for a `CustomDocument`, VS Code calls the `CustomDocument.dispose` on it. Our extension's implementation of `dispose` must clean up any resources associated with the document.

If the user then reopens the same resource using our custom editor, we will go back through the whole `openCustomDocument`, `resolveCustomEditor` flow with a new `CustomDocument`.

### Readonly Custom editors[#](https://code.visualstudio.com/api/extension-guides/custom-editors#readonly-custom-editors)

Many of the following sections only apply to custom editors that support editing and, while it may sound paradoxical, many custom editors don't require editing capabilities at all. Consider a image preview for example. Or a visual rendering of a memory dump. Both can be implemented using custom editors but neither need to be editable. That's where `CustomReadonlyEditorProvider` comes in.

A `CustomReadonlyEditorProvider` lets you create custom editors that do not support editing. They can still be interactive but don't support operations such as undo and save. It is also much simpler to implement a readonly custom editor compared to a fully editable one.

### Editable Custom Editor Basics[#](https://code.visualstudio.com/api/extension-guides/custom-editors#editable-custom-editor-basics)

Editable custom editors let you hook in to standard VS Code operations such as undo and redo, save, and hot exit. This makes editable custom editors very powerful, but also means that properly implementing is much more complex than implementing an editable custom text editor or a readonly custom editor.

Editable custom editors are implemented by `CustomEditorProvider`. This interface extends `CustomReadonlyEditorProvider`, so you'll have to implement basic operations such as `openCustomDocument` and `resolveCustomEditor`, along with a set of editing specific operations. Let's take a look at the editing specific parts of `CustomEditorProvider`.

**Edits**

Changes to a editable custom document are expressed through edits. An edit can be anything from a text change, to an image rotation, to reordering a list. VS Code leaves the specifics of what an edit does entirely up to your extension, but VS Code does need to know when an edit takes places. Editing is how VS Code marks documents as dirty, which in turn enables auto save and back ups.

Whenever a user makes an edit in any of the webviews for your custom editor, your extension must fire a `onDidChangeCustomDocument` event from its `CustomEditorProvider`. The `onDidChangeCustomDocument` event can fired two event types depending on your custom editor implementation: `CustomDocumentContentChangeEvent` and `CustomDocumentEditEvent`.

**CustomDocumentContentChangeEvent**

A `CustomDocumentContentChangeEvent` is a bare-bones edit. It's only function is to tell VS Code that a document has been edited.

When an extension fires a `CustomDocumentContentChangeEvent` from `onDidChangeCustomDocument`, VS Code will mark the associated document as being dirty. At this point, the only way for the document to become non-dirty is for the user to either save or revert it. Custom editors that use `CustomDocumentContentChangeEvent` do not support undo/redo.

**CustomDocumentEditEvent**

A `CustomDocumentEditEvent` is a more complex edit that allows for undo/redo. You should always try to implement your custom editor using `CustomDocumentEditEvent` and only fallback to using `CustomDocumentContentChangeEvent` if implementing undo/redo is not possible.

A `CustomDocumentEditEvent` has the following fields:

* `document` — The `CustomDocument` the edit was for.
* `label` — Optional text that that describes what type of edit was made (for example: "Crop", "Insert", ...)
* `undo` — Function invoked by VS Code when the edit needs to be undone.
* `redo` — Function invoked by VS Code when the edits needs to be redone.

When an extension fires a `CustomDocumentEditEvent` from `onDidChangeCustomDocument`, VS Code marks the associated document as being dirty. To make the document no longer dirty, a user can then either save or revert the document, or undo/redo back to the document's last saved state.

The `undo` and `redo` methods on an editor are called by VS Code when that specific edits needs to be undone or reapplied. VS Code maintains an internal stack of edits, so if your extension fires `onDidChangeCustomDocument` with three edits, let's call them `a`, `b`, `c`:

```
onDidChangeCustomDocument(a);
onDidChangeCustomDocument(b);
onDidChangeCustomDocument(c);
```

The following sequence of user actions results in these calls:

```
undo — c.undo()
undo — b.undo()
redo — b.redo()
redo — c.redo()
redo — no op, no more edits
```

To implement undo/redo, your extension must update it's associated custom document's internal state, as well as updating all associated webviews for the document so that they reflect the document's new state. Keep in mind that there may be multiple webviews for a single resource. These must always show the same document data. Multiple instances of an image editor for example must always show the same pixel data but may allow each editor instance to have its own zoom level and UI state.

### Saving[#](https://code.visualstudio.com/api/extension-guides/custom-editors#saving)

When a user saves a custom editor, your extension is responsible for writing the saved resource in its current state to disk. How your custom editor does this depends largely on your extension's `CustomDocument` type and how your extension tracks edits internally.

The first step to saving is getting the data stream to write to disk. Common approaches to this include:

* Track the resource's state so that it can be quickly serialized.

  A basic image editor for example may maintain a buffer of pixel data.

* Replay edit since the last save to generate the new file.

  A more efficient image editor for example might track the edits since the last save, such as `crop`, `rotate`, `scale`. On save, it would then apply these edits to file's last saved state to generate the new file.

* Ask a `WebviewPanel` for the custom editor for file data to save.

  Keep in mind though that custom editors can be saved even when they are not visible. For this reason, it is recommended that that your extension's implementation of `save` does not depend on a `WebviewPanel`. If this is not possible, you can use the `WebviewPanelOptions.retainContextWhenHidden` setting so that the webview stays alive even when it is hidden. `retainContextWhenHidden` does have significant memory overhead so be conservative about using it.

After getting the data for the resource, you generally should use the [workspace FS api](https://code.visualstudio.com/api/references/vscode-api#FileSystem) to write it to disk. The FS APIs take a `UInt8Array` of data and can write out both binary and text based files. For binary file data, simply put the binary data into the `UInt8Array`. For text file data, use `Buffer` to convert a string into a `UInt8Array`:

```
const writeData = Buffer.from('my text data', 'utf8');
vscode.workspace.fs.writeFile(fileUri, writeData);
```

## Next steps[#](https://code.visualstudio.com/api/extension-guides/custom-editors#next-steps)

If you'd like to learn more about VS Code extensibility, try these topics:

* [Extension API](https://code.visualstudio.com/api) * Learn about the full VS Code Extension API.
* [Extension Capabilities](https://code.visualstudio.com/api/extension-capabilities/overview) * Take a look at other ways to extend VS Code.