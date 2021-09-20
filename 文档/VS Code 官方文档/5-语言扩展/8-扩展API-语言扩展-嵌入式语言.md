# 嵌入式语言 的 语言服务器

[原文链接，戳我前往](https://code.visualstudio.com/api/language-extensions/embedded-languages)

------

翻译 by [赫雯勒莉特翡翠](https://github.com/HeveraletLaidCenx)

## 术语~的对照表

|中文（常用英文表述）|英文|
|----|----|
|插值|Interpolation|
|劫持|hijacks|
|内容分发网络（CDN）|Content Delivery Network|

表中部分：

* 在中文表述中常直接用英文替代的
* 认为直译并不合适的

在中文之后的括号中说明了直接使用对应的英文。

------

## 概述

**Visual Studio Code** 为编程语言提供了丰富的语言功能。就像你已经在 [语言服务器扩展指导](7-扩展API-语言扩展-语言服务器扩展指导.md) 中读到的那样，你可以编写 语言服务器 来支持任何一种编程语言。然而，要启用对 嵌入式语言 的支持的话，需要付出更多的努力。

今天，涌现出越来越多的嵌入式语言，比如：

* HTML 中的 JavaScript 和 CSS
* JavaScript 中的 JSX
* 模板语言 中的 插值，比如 Vue、Handlebars 和 Razor
* PHP 中的 HTML

本篇指导侧重于 实现 嵌入式语言 的语言功能。如果你想为嵌入式语言提供语法高亮的话，可以到 [语法高亮指导](2-扩展API-语言扩展-语法高亮指导.md) 中查看相关内容。

本篇指导包含两个例子，展示了构建此类语言服务器的两种方法： **Language Services**（语言服务） 和 **Request Forwarding**（请求转发） 。我们将分别查看这两个例子，并且总结它们的优缺点。

每个例子的源代码可以从以下地址找到：

* [语言服务实现的嵌入式语言的语言服务器](https://github.com/microsoft/vscode-extension-samples/tree/main/lsp-embedded-language-service)
* [请求转发实现的嵌入式语言的语言服务器](https://github.com/microsoft/vscode-extension-samples/tree/main/lsp-embedded-request-forwarding)

这是我们即将构建的嵌入式语言的语言服务器：

![嵌入式语言服务器协议示例](img/嵌入式语言服务器协议示例.gif)

出于演示目的，两个例子都建立了一种新的语言—— `html1` 。你可以创建一个扩展名为 `.html1` 的文件来测试以下功能：

* HTML 标签的 自动补全
* 在 `<style>`（样式） 标签中的 CSS样式 的自动补全
* 对于 CSS 的诊断（仅在语言服务的例子中）

## 语言服务

**语言服务** 是一个实现单一语言的 [编程型语言功能](https://code.visualstudio.com/api/language-extensions/programmatic-language-features) 的库。**语言服务器** 可以通过嵌入 语言服务 来处理嵌入式语言。

以下是对 **VS Code** 的 HTML 支持的概要：

* 内置的 [html 扩展](https://github.com/microsoft/vscode/tree/main/extensions/html) 只提供 语法高亮 和 HTML的语言配置。
* 内置的 [html 语言功能扩展](https://github.com/microsoft/vscode/tree/main/extensions/html-language-features) 包括了一个可以为 HTML 提供 编程型语言功能 的 HTML 语言服务器。
* HTML 语言服务器使用 [vscode-html-languageservice（vscode-html-语言服务）](https://github.com/microsoft/vscode-html-languageservice) 来支持 HTML。
* CSS 语言服务器 使用 [vscode-css-languageservice（vscode-css-语言服务）](https://github.com/microsoft/vscode-css-languageservice) 来支持 HTML 中的 CSS。

HTML 语言服务器会分析 HTML 文档，将它分解成 语言区域，然后使用对应的 语言服务 来处理 语言服务器 的请求。

比如：

* 对于在 `<|`（输入标签左侧的尖括号） 的自动补全请求，HTML语言服务器 使用 HTML语言服务 来提供 HTML的自动补全。
* 对于在 `<style>.foo { | }</style>`【译注：大概是样式标签中的CSS样式部分】 的自动补全请求，HTML语言服务器 使用 CSS语言服务 来提供 CSS的自动补全。

让我们检查一下 [lsp-embedded-language-service（语言服务器协议-嵌入式-语言-服务）](https://github.com/microsoft/vscode-extension-samples/tree/main/lsp-embedded-language-service) 这个例子，它是一个 实现了 对 HTML、CSS实现了自动补全，以及提供对 CSS的错误诊断 的 简化版的 HTML 语言服务器。

### 语言服务的例子

> **注意**：这个例子假设你已经学会了 [编程型语言功能](6-扩展API-语言扩展-编程型语言功能.md) 和 [语言服务器扩展指导](7-扩展API-语言扩展-语言服务器扩展指导.md) 文章中的内容。这个例子的代码建立在上一章中的 [lsp-sample（语言服务器协议的例子）](https://github.com/microsoft/vscode-extension-samples/tree/main/lsp-sample) 的基础上。

源代码可以从 [microsoft/vscode-extension-samples](https://github.com/microsoft/vscode-extension-samples/tree/main/lsp-embedded-language-service) 来查看。

和 [lsp-sample（语言服务器协议的例子）](https://github.com/microsoft/vscode-extension-samples/tree/main/lsp-sample) 相比，客户端 的代码是一样的。

就像上面提到的，服务器 会将文档分解为 不同的语言区域，以此来处理嵌入式的内容。

以下是个简单的例子：

```html
<div></div>
<style>.foo { }</style>
```

在这种情况下，服务器 会检测到 `<style>` 标签，并将 `.foo { }` 标记为一个 CSS区域 。

对于 给定位置 的自动补全请求，服务器 使用以下逻辑来计算响应：

* 如果 给定位置 属于 任意区域
  * 使用 对应区域的语言的虚拟文档 来处理它，同时用空格替换其他所有区域
* If the position falls out of any region
  * 使用 HTML 的虚拟文档 来处理它，同时用空格替换所有区域

比如，当在下面这个位置进行 自动补全 的时候：

```html
<div></div>
<style>.foo { | }</style>
```

服务器 判断 位置在区域中，计算 虚拟CSS文档，虚拟文档的内容如下（其中 `█` 表示空格）：

```html
███████████
███████.foo { | }████████
```

然后 服务器 使用 `vscode-css-languageservice`（vscode-css-语言服务） 来分析这个文档，并且计算出它的 自动补全项目列表 。因为内容现在不包含 HTML ，所以用 CSS语言服务 处理它 完全没问题。通过将 所有非CSS的内容 用空格替换，我们不需要手动偏移位置。

处理 自动补全请求 的 服务器 代码：

```typescript
connection.onCompletion(async (textDocumentPosition, token) => {
  const document = documents.get(textDocumentPosition.textDocument.uri);
  if (!document) {
    return null;
  }

  const mode = languageModes.getModeAtPosition(document, textDocumentPosition.position);
  if (!mode || !mode.doComplete) {
    return CompletionList.create();
  }
  const doComplete = mode.doComplete!;

  return doComplete(document, textDocumentPosition.position);
});
```

负责处理所有被归入 CSS区域 的 语言服务器请求 的 CSS 模式：

```typescript
export function getCSSMode(
  cssLanguageService: CSSLanguageService,
  documentRegions: LanguageModelCache<HTMLDocumentRegions>
): LanguageMode {
  return {
    getId() {
      return 'css';
    },
    doComplete(document: TextDocument, position: Position) {
      // 获取虚拟 CSS 文档，其中 所有非CSS内容 都用空格来代替
      const embedded = documentRegions.get(document).getEmbeddedDocument('css');
      // 用 vscode-css-languageservice（vscode-css-语言服务） 计算响应
      const stylesheet = cssLanguageService.parseStylesheet(embedded);
      return cssLanguageService.doComplete(embedded, position, stylesheet);
    }
  };
}
```

这是个处理嵌入式语言的简单而有效的方法。然而，用这种方法实现有一些缺点：

* 你需要不断地更新你的 语言服务器 依赖的 语言服务。
* 将 和你编写语言服务器所用语言不同的语言服务 包含进来很具挑战性。比如，对于一个 用PHP编写的PHP语言服务器，可能会发现要想包含一个 用TypeScript编写的 `vscode-css-languageservice`（vscode-css-语言服务） 会极端麻烦。

为了解决上述问题，让我们隆重介绍 **request forwarding**（请求转发）——

## 请求转发

简而言之，请求转发 和 语言服务 的工作方式类似。请求转发 也会接受 语言服务器请求、计算虚拟内容、计算响应。

主要的区别在于：

* 语言服务 方法使用 库 来计算 语言服务器的响应，而 请求转发 会将 请求 发回 **VS Code** 来查询所有 语言服务器，并转发它们的响应。
* 分配是在 **语言客户端** 发生的，不是在 **语言服务器** 发生的。

又是那个简单的例子：

```html
<div></div>
<style>.foo { | }</style>
```

此时 自动补全 通过以下方式实现：

* 语言客户端 用 `workspace.registerTextDocumentContentProvider`（工作区.注册文本文档内容提供程序） 为 `embedded-content`（嵌入的内容） 注册一个 虚拟文本文档提供程序。

* 语言客户端 劫持 对`<FILE_URI>`的自动补全请求。

* 语言客户端确定 归入CSS区域的请求位置。

* 语言客户端 构建一个新的 URI，比如 `embedded-content://css/<FILE_URI>.css`（嵌入式的-内容://css/<文件URI>.css）

* 然后 语言客户端 调用

    ```typescript
    commands.executeCommand('vscode.executeCompletionItemProvider', ...)
    ```

  * **VS Code** 的 CSS语言服务器 响应这个 提供程序请求。
  * 虚拟文本文档提供程序 提供内容，所有非CSS内容 都用空格替代
  * 语言客户端 从 **VS Code** 接收响应，并将它作为响应发送。

通过这种方法，我们可以在即使我们的代码没有包括任何理解 CSS 的库的情况下，计算 CSS 的自动补全。只要 **VS Code** 更新了它的 CSS语言服务器，我们就可以在不用更新我们的代码的情况下，获得最新的 CSS语言支持。

现在让我们来看看示例代码：

### 请求转发的例子

> **注意**：这个例子假设你已经学会了 [编程型语言功能](6-扩展API-语言扩展-编程型语言功能.md) 和 [语言服务器扩展指导](7-扩展API-语言扩展-语言服务器扩展指导.md) 文章中的内容。这个例子的代码建立在上一章中的 [lsp-sample（语言服务器协议的例子）](https://github.com/microsoft/vscode-extension-samples/tree/main/lsp-sample) 的基础上。

源代码可以在 [microsoft/vscode-extension-samples](https://github.com/microsoft/vscode-extension-samples/tree/main/lsp-embedded-request-forwarding) 找到。

保持一个 从 文档URI 到 它们的虚拟文档 的映射，然后向它们提供对应的请求：

```typescript
const virtualDocumentContents = new Map<string, string>();

workspace.registerTextDocumentContentProvider('embedded-content', {
  provideTextDocumentContent: uri => {
    // 移除开头的 `/` 和结尾的 `.css` 来获取 原始URI
    const originalUri = uri.path.slice(1).slice(0, -4);
    const decodedUri = decodeURIComponent(originalUri);
    return virtualDocumentContents.get(decodedUri);
  }
});
```

通过使用 语言客户端 的 `middleware`（中间件） 选项，我们劫持自动补全的请求：

```typescript
let clientOptions: LanguageClientOptions = {
  documentSelector: [{ scheme: 'file', language: 'html' }],
  middleware: {
    provideCompletionItem: async (document, position, context, token, next) => {
      // 如果不在 `<style>`（样式） 标签内，不进行请求转发。
      if (
        !isInsideStyleRegion(
          htmlLanguageService,
          document.getText(),
          document.offsetAt(position)
        )
      ) {
        return await next(document, position, context, token);
      }

      const originalUri = document.uri.toString();
      virtualDocumentContents.set(
        originalUri,
        getCSSVirtualContent(htmlLanguageService, document.getText())
      );

      const vdocUriString = `embedded-content://css/${encodeURIComponent(originalUri)}.css`;
      const vdocUri = Uri.parse(vdocUriString);
      return await commands.executeCommand<CompletionList>(
        'vscode.executeCompletionItemProvider',
        vdocUri,
        position,
        context.triggerCharacter
      );
    }
  }
};
```

## 潜在的问题

在实现 嵌入式语言服务器 的过程中，我们遇到了很多问题。虽然我们目前还没有完美的解决方案，但是我们向提醒你一下，因为你也可能遇到这些问题。

### 难以实现语言功能

通常，跨语言区域边界工作的语言功能 会更难实现。

比如，自动补全 或者 鼠标悬停提示内容 可能很容易就可以实现，因为你可以检测嵌入内容的语言，并且根据嵌入内容计算响应。

然而，像 格式化、重命名 这类的语言功能，可能需要特殊的处理。比如：

* 对于 格式化 来说，你需要处理 单个文档 中的 多区域 的 缩进 和 格式化设置。
* 对于 重命名 来说，在 不同的文档 中的 不同区域 工作可能颇具挑战。

### 语言服务 可能是 有状态的，因此难以嵌入

**VS Code** 的 HTML支持 提供 HTML、CSS、JavaScript 语言功能。虽然 HTML 和 CSS 语言服务 是 无状态的，但是 JavaScript 语言功能 是由 TypeScript 服务器驱动的。我们只在 HTML 中提供基础的 JavaScript 支持，以为内很难将 项目的状态 告知 TypeScript。

比如，如果你包含了一个 指向一个托管于CDN的`lodash`库的`<script>`（脚本）标签，你将不能在该标签中获得 `_.` 的自动补全。

### 编码 和 解码

文档的主要语言 可能具有 与它的嵌入语言不同的 编码或转义规则。

比如，根据 [HTML 规范](https://www.w3.org/TR/html401/appendix/notes.html#h-B.3.2) ，这个 HTML 文档是无效的：

```html
<SCRIPT type="text/javascript">
  document.write ("<EM>This won't work</EM>")
</SCRIPT>
```

这种情况下，如果嵌入的 JavaScript 的 语言服务器 返回了一个 包含 `</`, 的结果，它会被转义成： `<\/`。

## 总结

这两种方法各有优劣。

语言服务：

* 【优点】 对于 语言服务器 和 用户体验 的完全控制。
* 【优点】 不依赖其他的 语言服务器。所有代码都在一个仓库中。
* 【优点】 语言服务器 可以在所有 [遵循LSP（语言服务器协议） 的代码编辑器](https://microsoft.github.io/language-server-protocol/implementors/tools/) 中重用。
* 【缺点】 可能难以嵌入 用其他语言编写的语言服务。
* 【缺点】 需要持续维护，以此来从 语言服务依赖 获取新功能。

请求转发：

* 【优点】 避免了 嵌入式语言服务不是由语言服务器的语言编写的 这个问题（比如，在 Razor语言服务器 中嵌入 C#编译器 来支持 C#）。
* 【优点】 不需要维护就可以从其他语言服务的上游获取新功能。
* 【缺点】 暂时不能处理 从语言服务器推送的诊断错误。
  > LSP（语言服务器协议） 在 3.17 版本提供了一个 诊断拉取模型，让这项成为了可能。
* 【缺点】 因为缺少控制，难以向其他 语言服务器 共享状态。
* 【缺点】 跨语言的功能可能难以实现。（比如，当存在 `<div class="foo">` 时，为 `.foo` 提供 CSS自动补全）。

总的来说，我们建议 通过嵌入语言服务来构建语言服务器，因为这种方法能让你更好地控制用户体验，也可以让 服务器 可以重用于 任何 遵循LSP（语言服务器协议） 的编辑器。

然而，如果你的情况比较简单，可以在没有 上下文环境 或者 语言服务器状态 的情况下处理嵌入式内容，或者对你来说，打包 Node.js 库是个问题的话，你也可以选择用 请求转发 的方法。
