# 文件图标主题

[原文链接，戳我前往](https://code.visualstudio.com/api/extension-guides/file-icon-theme)

------

翻译 by [赫雯勒莉特翡翠](https://github.com/HeveraletLaidCenx)

## 术语~的对照表

|中文（常用英文表述）|英文|
|----|----|
|可缩放的矢量图形（SVG）|Scalable Vector Graphics|
|处于折叠状态的文件夹|collaspsed folder|
|处于展开状态的文件夹|expanded folder|
|不敏感的【如对大小写不敏感，就是指不区分大小写都可以识别】|insensitive|
|敏感的|sensitive|
|折叠图标|Twisties|

表中部分：

* 在中文表述中常直接用英文替代的
* 认为直译并不合适的

在中文之后的括号中说明了直接使用对应的英文。

------

## 概述

在整个 **Visual Studio Code** 的 UI 中，在文件名旁边会显示对应的图标，而扩展可以建立新的文件图标集供用户从中选择。

## 添加一个新的文件图标主题

你可以从 图标（最好是SVG） 和 图标字体 来创建你自己的文件图标主题。可以看看这两个内置主题当做例子：[Minimal](https://github.com/microsoft/vscode/tree/main/extensions/theme-defaults) 和 [Seti](https://github.com/microsoft/vscode/tree/main/extensions/theme-seti) 。

首先，创建一个 **VS Code** 扩展并添加 `iconTheme`（图标主题） 作用点。

```json
{
  "contributes": {
    "iconThemes": [
      {
        "id": "turtles",
        "label": "Turtles",
        "path": "./fileicons/turtles-icon-theme.json"
      }
    ]
  }
}
```

|字段名|翻译|字段含义|
|----|----|----|
|`id`|标识符|是图标主题的标识符。它也在设置中也被用作标识符，所以确保它唯一且具有可读性（有意义、让人能区分出来这是你的主题）。|
|`label`|标签|会在文件图标主题选择器的下拉列表中显示。|
|`path`|路径|则指向扩展中定义了图标集的文件。如果你的图标集的名称符合 `*icon-theme.json` 的命名格式，那么你将在 **VS Code** 中获得 自动补全支持 和鼠标悬停提示。|

### 文件图标集 文件

文件图标集 文件是一个由 文件图标关联 和 图标定义 组成的 JSON 文件。

一个 图标关联 会将一个 文件类型（文件、文件夹、json文件…）映射到一个 图标定义。

图标定义 则定义了图标在什么位置：可以是一个图像文件，也可以是字体中的图形字符。

### 图标定义

`iconDefinitions`（图标定义） 部分包含了所有的定义。每个定义都有一个将被用来引用定义的 ID。而一个定义也可以被多种文件格式关联所引用

```json
{
  "iconDefinitions": {
    "_folder_dark": {
      "iconPath": "./images/Folder_16x_inverse.svg"
    }
  }
}
```

上边的这个图标定义，包括一个以 `_folder_dark` 为 ID 的定义。

在图标定义中支持如下属性：

* 当使用 SVG 或 PNG 格式的图像文件时：
  |字段名|翻译|字段含义|
  |----|----|----|
  |`iconPath`|图标路径|表示该图像的路径。|
* 当使用图形字符字体时：
  |字段名|翻译|字段含义|
  |----|----|----|
  |`fontCharacter`|字体字符|表示在字体中要使用的那个字符|
  |`fontColor`|字体颜色|要对该图形字符使用的颜色。|
  |`fontSize`|字体尺寸|图形字符的尺寸。默认情况下，使用字体规范中指定的大小。这个尺寸的值应该是一个相对于上级字体的相对尺寸，比如 150% 。|
  |`fontId`|字体ID|字体的ID。如果没有特别指明，那么会默认选择字体规范中的第一个字体。|

### 文件关联

图标可以和：文件夹、文件夹名称、文件、文件扩展名、文件名 和 [语言 ID](https://code.visualstudio.com/api/references/contribution-points#contributes.languages) 相关联。

此外，这些关联关联中的每一个都可以针对 `light`（亮色） 和 `highContrast`（高对比度）颜色主题进行细化。

每个 文件关联 都指向一个 图标定义。

```json
{
  "file": "_file_dark",
  "folder": "_folder_dark",
  "folderExpanded": "_folder_open_dark",
  "folderNames": {
    ".vscode": "_vscode_folder"
  },
  "fileExtensions": {
    "ini": "_ini_file"
  },
  "fileNames": {
    "win.ini": "_win_ini_file"
  },
  "languageIds": {
    "ini": "_ini_file"
  },
  "light": {
    "folderExpanded": "_folder_open_light",
    "folder": "_folder_light",
    "file": "_file_light",
    "fileExtensions": {
      "ini": "_ini_file_light"
    }
  },
  "highContrast": {}
}
```

|字段名|翻译|字段含义|
|----|----|----|
|`file`|文件|是默认的文件图标，所有不与已经设定的 扩展名、文件名 或者 语言ID 相匹配的文件都会显示成这个图标。目前，由 文件图标 定义的 **所有属性** 都将被继承（只与图形字符字体有关，对 fontSize（字体尺寸） 有用）。|
|`folder`|文件夹|是处于折叠状态的文件夹的图标，如果没有设置 `folderExpanded`（展开的文件夹） ，则也会被应用给展开的文件夹。对于特定的文件夹名称的图标，可以使用 `folderNames`（文件夹名称） 属性来关联。文件夹图标是选择性设置的。如果不设置的话，文件夹就不会显示图标。|
|`folderExpanded`|展开的文件夹|是处于展开状态的文件夹的图标。这个图标也是选择性设置的，不设置的话就会显示为给 `folder`（文件夹） 设置的图标。|
|`folderNames`|文件夹名称|会把文件夹名称关联到图标。这个设置的键 **仅仅是文件夹的名称** ，**不包括** 任何文件夹路径的内容。 **不支持** 格式或者通配符。文件夹名称的匹配是 **大小写不敏感** 的。|
|`folderNamesExpanded`|展开的文件夹的名称|会把展开的文件夹的名称关联到图标。设置的键部分的要求和上一条文件夹名称完全一样。|
|`rootFolder`|根文件夹|是处于折叠状态的工作区根文件夹的文件夹图标，如果没有设置 `rootFolderExpanded`（展开的根文件夹） ，则也会被应用给展开的工作区根文件夹。不设置的话就会显示为给 `folder`（文件夹） 设置的图标。|
|`rootFolderExpanded`|展开的根文件夹|是处于展开状态的工作区根文件夹的图标。不设置的话就会显示为给 `rootFolder`（根文件夹） 设置的图标。|
|`languageIds`|语言ID|把语言关联到图标。这个设置的键是在 [语言作用点](https://code.visualstudio.com/api/references/contribution-points#contributes.languages) 里定义的 语言ID 。文件的语言是根据 文件的扩展名 和 在语言作用点中定义的文件名 来判定的。注意：语言作用点的 `first line match`（匹配第一行） **不被考虑**【此处的“不被考虑”暂未找到更合适的翻译故暂时直译】 。|
|`fileExtensions`|文件扩展名|把文件扩展名关联到图标。这个设置的键是文件的扩展名。扩展名是文件名中， `.` 后的部分， **不包括 `.` 本身** 。在文件名中有多个 `.` 的，比如 `lib.d.ts` ，可以匹配多个扩展名： `d.ts` 和 `ts` 。扩展名也是 **大小写不敏感** 的。|
|`fileNames`|文件名|把文件名关联到图标。这个设置的键是 **不包括任何路径的，完整的文件名** 。 **不支持** 格式或者通配符。文件名称的匹配是 **大小写不敏感** 的。文件名的匹配是 **最优先的匹配** ，成功匹配文件名的文件的图标会优先使用 文件名关联的图标，而不是 文件扩展名 或者 语言ID 所关联的图标。|

> 匹配优先级：文件名匹配 > 文件扩展名匹配 > 语言匹配

`light`（亮色） 和 `highContrast`（高对比度） 部分的设置属性和上面列出的文件关联设置属性完全一致，设置它们会允许在应用对应主题时，用设置的图标覆盖默认的图标。

### 字体定义

`fonts`（字体） 部分让你可以声明任意数量的你想使用的 图形字符字体。

你可以在之后的 图标定义 中引用这些字体。如果 图标定义 没有特别指明 字体ID ，则第一个声明的字体会被用作其默认值。

把字体文件复制到你的扩展，并且对应地设置其路径。

推荐使用 [WOFF](https://developer.mozilla.org/docs/Web/Guide/WOFF) 字体。

* 把format（格式）设置为 `woff` 。
* weight（字重） 属性的值是在 [这里](https://developer.mozilla.org/docs/Web/CSS/font-weight#Values) 定义的。
* style（样式） 属性的值是在 [这里](https://developer.mozilla.org/docs/Web/CSS/@font-face/font-style#Values) 定义的。
* size（尺寸）属性应该是相对于图标使用的字体的尺寸的。因此，请使用百分比的形式。

```json
{
  "fonts": [
    {
      "id": "turtles-font",
      "src": [
        {
          "path": "./turtles.woff",
          "format": "woff"
        }
      ],
      "weight": "normal",
      "style": "normal",
      "size": "150%"
    }
  ],
  "iconDefinitions": {
    "_file": {
      "fontCharacter": "\\E002",
      "fontColor": "#5f8b3b",
      "fontId": "turtles-font"
    }
  }
}
```

### 文件图标主题 中的 文件夹图标

当文件夹的图标已经可以很好地展示文件夹的 展开/折叠 状态时，文件图标主题可以告诉资源管理器：不再显示默认的文件夹图标（旋转的三角形，也被称之为"Twisties"（折叠图标））。要实现这个功能，请在文件图标主题定义文件中，设置 `"hidesExplorerArrows":true`（隐藏资源管理器的箭头:真） 。
