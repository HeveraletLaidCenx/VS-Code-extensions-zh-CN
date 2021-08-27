# 扩展剖析

[原文链接，戳我前往](https://code.visualstudio.com/api/get-started/extension-anatomy)

------

翻译 by [赫雯勒莉特翡翠](https://github.com/HeveraletLaidCenx)

## 术语~的对照表

|中文（常用英文表述）|英文|
|----|----|
|注册|register|
|激活事件|Activation Event|
|绑定|bind|
|身份（ID）|Identity|
|函数|function|
|调用|invoke|
|功能|capabilities|
|文件结构|file structure|
|编译|compile|
|node 模块|node_modules|
|任务|Tasks|
|域|field|
|唯一识别|uniquely identify|
|入口|entry point|
|激活|activate|
|停用|deactivate|
|执行|execute|
|显式的|explicit|
|智能感知提示|IntelliSense|
|导入|import|
|别名|alias|
|引用|reference|

表中部分：

* 在中文表述中常直接用英文替代的
* 认为直译并不合适的

在中文之后的括号中说明了直接使用对应的英文。

------

## 概述

在上一篇文章中，你成功让一个基本的扩展运行起来了，但是透过现象看本质的话，它到底是怎么运行起来的呢？

`Hello World` 扩展做了3件事儿：

* 注册了 [onCommand](https://code.visualstudio.com/api/references/activation-events#onCommand)（以命令触发）[**激活事件**](https://code.visualstudio.com/api/references/activation-events)：`onCommand:helloworld.helloWorld`，因此扩展会在用户运行 `Hello World` 命令时被激活。
* 用 [contributes.commands](https://code.visualstudio.com/api/references/contribution-points#contributes.commands)（作用点.命令）[**作用点**](https://code.visualstudio.com/api/references/contribution-points) 来让 `Hello World` 命令在命令面板中可用，并且将它绑定到一个命令 ID `helloworld.helloWorld` 。
* 用 [commands.registerCommand](https://code.visualstudio.com/api/references/vscode-api#commands.registerCommand)（命令.注册命令）[**扩展API**](https://code.visualstudio.com/api/references/vscode-api) 来将一个函数绑定到注册了的命令 ID `helloworld.helloWorld` 上。

理解这仨概念，在 **VS Code** 扩展编写中至关重要：

* [**激活事件**](https://code.visualstudio.com/api/references/activation-events)：让你的扩展激活所依据的事件。
* [**作用点**](https://code.visualstudio.com/api/references/contribution-points)：你在 `package.json` [扩展清单](https://code.visualstudio.com/api/get-started/extension-anatomy#extension-manifest) 中为了扩展 **VS Code** 而做的静态声明。
* [**扩展API**](https://code.visualstudio.com/api/references/vscode-api)：一组你可以在你的扩展代码中调用的 **JavaScript** API 。

一般情况下，你的扩展会组合使用 **作用点** 和 **扩展API** 来达到扩展 **VS Code** 的功能的目的。[扩展功能概述](https://code.visualstudio.com/api/extension-capabilities/overview) 将帮助你找到你的扩展功能对应的 **作用点** 和 **扩展API** 。

让我们进一步看看 `Hello World` 例子的源代码，看看这些概念是在例子里是如何应用的。

## 扩展文件结构

```text
.
├─ .vscode
│    ├─ launch.json     // 关于启动和 Debug 扩展的配置文件
│    └─ tasks.json      // 关于编译 TypeScript 的构建任务的配置
├─ .gitignore          // 忽略构建输出和 node 模块
├─ README.md           // 对于你的扩展功能的一份可读的描述说明
├─ src
│    └─ extension.ts    // 扩展的源代码
├─ package.json        // 扩展的清单文件
└─ tsconfig.json       // TypeScript 的配置
```

你可以了解有关配置文件的更多信息：

* `launch.json` 用来配置 [**VS Code** 的 Debug](https://code.visualstudio.com/docs/editor/debugging)
* `tasks.json` 用于定义 [**VS Code** 的任务](https://code.visualstudio.com/docs/editor/tasks)
* `tsconfig.json` 请参考 [**TypeScript** 的手册](https://www.typescriptlang.org/docs/handbook/tsconfig-json.html)

但——是——，让我们把更多的目光放到 `package.json` 和 `extension.ts` 上，它们对于理解我们的第一个 `Hello World` 扩展例子 **至关重要**。

### 扩展清单

每个 **VS Code** 扩展都必须有一个 `package.json` 作为它的 [扩展清单](https://code.visualstudio.com/api/references/extension-manifest)。 `package.json` 包括了一个 **Node.js 域** （比如 `scripts`（脚本） 和 `devDependencies`（开发依赖））和 **VS Code 特定域** （比如 `publisher`（发布者）、`activationEvents`（激活事件）和 `contributes`（建立作用点））的组合。你可以在 [扩展清单参考](https://code.visualstudio.com/api/references/extension-manifest) 里找到所有 **VS Code 特定域** 的描述。这里列举其中一些最重要的域：

* `name`（名字）和 `publisher`（发布者）：**VS Code** 把 `<publisher>.<name>` 用作扩展的唯一 ID 。比如， `Hello World` 例子的 ID 是 `vscode-samples.helloworld-sample` 。 **VS Code** 使用此 ID 来唯一识别你的扩展。
* `main`：扩展的入口。
* `activationEvents` 和 `contributes`：[激活事件](https://code.visualstudio.com/api/references/activation-events) 和 [作用点](https://code.visualstudio.com/api/references/contribution-points) 。
* `engines.vscode`（引擎.vscode）: 指定了扩展依赖的 **VS Code 扩展API** 的所需最低版本。

> ↓ `package.json` 的文件内容

```json
{
  "name": "helloworld-sample",
  "displayName": "helloworld-sample",
  "description": "HelloWorld example for VS Code",
  "version": "0.0.1",
  "publisher": "vscode-samples",
  "repository": "https://github.com/microsoft/vscode-extension-samples/helloworld-sample",
  "engines": {
    "vscode": "^1.51.0"
  },
  "categories": ["Other"],
  "activationEvents": ["onCommand:helloworld.helloWorld"],
  "main": "./out/extension.js",
  "contributes": {
    "commands": [
      {
        "command": "helloworld.helloWorld",
        "title": "Hello World"
      }
    ]
  },
  "scripts": {
    "vscode:prepublish": "npm run compile",
    "compile": "tsc -p ./",
    "watch": "tsc -watch -p ./"
  },
  "devDependencies": {
    "@types/node": "^8.10.25",
    "@types/vscode": "^1.51.0",
    "tslint": "^5.16.0",
    "typescript": "^3.4.5"
  }
}
```

## 扩展入口文件

扩展入口文件会导出两个函数： `activate` （激活）和 `deactivate` （停用）。 `activate` 在你注册了的 **激活事件** 发生的时候执行。 `deactivate` 让你有机会在扩展被停用前进行清理。对于很多扩展来说，可能并不需要进行显式清理， `deactivate` 在这种情况下也不需要，可以移除。但是如果扩展需要在 **VS Code** 关闭时，或者扩展被禁用、卸载时执行操作，那就需要使用 `deactivate` 了。

**VS Code 扩展API** 在 [@types/vscode](https://www.npmjs.com/package/@types/vscode) 类型定义中声明。 `vscode` 类型定义的版本是由 `package.json` 中 `engines.vscode` 域中的值控制的。 `vscode` 类型在你代码中提供 智能感知提示、转到定义，以及其他的 **TypeScript** 语言功能。

> ↓ `extension.ts` 的文件内容

```typescript
// 'vscode' 模块包含了 VS Code 扩展API
// 导入该模块，并在之后的代码中用别名 "vscode"来引用它
import * as vscode from 'vscode';

// 这个方法会在你的扩展被激活时调用
// 你的扩展会在第一次执行命令的时候被激活
export function activate(context: vscode.ExtensionContext) {
  // 使用控制台来输出诊断信息（console.log）和错误（console.error）
  // 这行代码只会在你的扩展被激活时运行一次
  console.log('Congratulations, your extension "helloworld-sample" is now active!');

  // 命令已经在 package.json 文件中定义了
  // 现在用 registerCommand 提供对命令的实现
  // commandId 参数必须与 package.json 中的命令域中定义的一致
  let disposable = vscode.commands.registerCommand('helloworld.helloWorld', () => {
    // 放在这里的代码将在每次执行命令时都被运行

    // 向用户显示一个消息框
    vscode.window.showInformationMessage('Hello World!');
  });

  context.subscriptions.push(disposable);
}

// 这个方法会你的扩展被停用时调用
export function deactivate() {}
```
