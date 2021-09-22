# 【注意：本篇文档未翻译完成】使用 语言服务API

[原文链接，戳我前往](https://github.com/microsoft/TypeScript/wiki/Using-the-Language-Service-API)

------

翻译 by [赫雯勒莉特翡翠](https://github.com/HeveraletLaidCenx)

## 术语~的对照表

|中文（常用英文表述）|英文|
|----|----|
|解耦、分离|decoupling|
|解耦编译器流水线阶段|decoupling compiler pipeline phases|

表中部分：

* 在中文表述中常直接用英文替代的
* 认为直译并不合适的

在中文之后的括号中说明了直接使用对应的英文。

------

> 有关 通用 TypeScript 编译器架构 和 分层的概述，请参阅 [架构概述](https://github.com/microsoft/TypeScript/wiki/Architectural-Overview)

## 概述

语言服务对象 可以被简单地看作一个 长期存在的程序，或者编译环境。

## 设计目的

设计 语言服务 时，主要想实现两个目标：

1：**按需处理**

语言服务 被设计用于实现 不论程序大小的快速响应。实现这一点的唯一方法就是 只做所需的最少的工作。所有 语言服务接口 都仅计算回答查询所需要的必要信息。

比如：

* 一个对 `getSyntacticDiagnostics`（获取语法诊断） 的调用，只需要解析对应的文件，而不需要执行 绑定 或者 类型检查。
* 一个对 `getCompletionsAtPosition`（获取指定位置的自动补全） 的调用，只会尝试解析 和对应的类型有关的声明，而不会处理其它无关的声明。

2：**解耦编译器流水线阶段**

语言服务 设计了 对于 编译器流水线 的 解耦不同阶段，这些阶段通常在 命令行编译 期间，按顺序发生；它允许 语言服务 灵活安排这些不同阶段的顺序。

For instance, the language service reports diagnostics on a file per file basis, all while making a distinction between syntactic and semantic errors of each file. This ensures that the host can supply an optimal experience by retrieving syntax errors for a given file without having to pay the cost of querying other files, or performing a full semantic check. It also allows the host to skip querying for syntax errors for files that have not changed. Similarly, the language service allows for emitting a single file (`getEmitOutput`) without having to emit or even type check the whole program.

## Language Service Host

The host is described by the LanguageServiceHost API, and it abstracts all interactions between the language service and the external world. The language service host defers managing, monitoring and maintaining input files to the host.

The language service will only ask the host for information as part of host calls. No asynchronous events or background processing are expected. The host is expected to manage threading if needed.

The host is expected to supply the full set of files comprising the context. Refer to [reference resolution in the language service](https://github.com/microsoft/TypeScript/wiki/Using-the-Language-Service-API#reference-resolution-in-the-language-service) for more details.

## ScriptSnapshot

A `ScriptSnapshot` represents the state of the text of an input file to the language service at a given point of time. The ScriptSnapshot is mainly used to allow for an efficient incremental parsing. A `ScriptSnapshot` is meant to answer two questions:

1. What is the current text?
2. Given a previous snapshot, what are the change ranges?

Incremental parsing asks the second question to ensure it only re-parses changed regions.

> For users who do not want to opt into incremental parsing, use `ts.ScriptSnapshot.fromString()`.

## Reference resolution in the language service

There are two means of declaring dependencies in TypeScript: import statements, and triple-slash reference comments (`/// <reference path="..." />`). Reference resolution for a program is the process of walking the dependency graph between files, and generating a sorted list of files comprising the program.

In the command-line compiler (`tsc`) this happens as part of building the program. A `createProgram` call starts with a set of root files, parses them in order, and walks their dependency declaration (both imports and triple-slash references) resolving references to actual files on disk and then pulling them into the compilation process.

This work flow is decoupled in the language service into two phases, allowing the host to interject at any point and change the resolution logic if needed. It also allows the host to fully manage program structure and optimize file state change.

> To resolve references originating from a file, use `ts.preProcessFile`. This method will resolve both imports and triple-slash references from a given file. Also worth noting is that this relies solely on the scanner, and does not require a full parse, so as to allow for fast context resolution which is suited to editor interactions.

## Document Registry

A language service object corresponds to a single project. So if the host is handling multiple projects it will need to maintain multiple instances of the LanguageService objects; each instance of the language service holds state about the files in the context. Most of the state that a language service object holds is syntactic (text + AST). The projects can share files (at minimum, the library file `lib.d.ts`).

The document registry is simply a store of SourceFile objects. If multiple language service instances share the same `DocumentRegistry` instance they will be able to share `SourceFile` objects, allowing for more efficient memory utilization.

A more advanced use of the document registry is to serialize `SourceFile` objects to disk and re-hydrate them when needed.

The Language service comes with a default `DocumentRegistry` implementation allowing for sharing SourceFiles between different `LanguageService` instances. Use `createDocumentRegistry` to create one, and pass it to all your `createLanguageService` calls.