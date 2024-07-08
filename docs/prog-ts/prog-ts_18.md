# 附录 E. 三斜杠指令

三斜杠指令只是普通的 JavaScript 注释，TypeScript 会寻找这些指令来做一些事情，比如调整特定文件的编译器设置，或者指示你的文件依赖于另一个文件。将你的指令放在文件顶部，任何代码之前。三斜杠指令看起来像这样（每个指令都是三斜杠 `///`，后面跟着一个 XML 标签）：

```
/// <directive attr="value" />
```

TypeScript 支持少量三斜杠指令。表 E-1 列出了你可能会使用的指令：

`amd-module`

转到 “amd-module 指令” 了解更多。

`lib`

`lib` 指令是告诉 TypeScript 你的模块依赖于 TypeScript 的哪些 `lib` 的一种方式，如果你的项目没有 *tsconfig.json*，在 *tsconfig.json* 中声明你依赖的 `lib` 是几乎总是更好的选择。

`path`

当使用 TSC 的 `outFile` 选项时，请使用 `path` 指令声明对另一个文件的依赖，以便该文件在编译输出中出现在依赖文件之前。如果你的项目使用 `import` 和 `export`，你可能永远不会使用此指令。

`type`

转到 “类型指令” 了解更多关于 `type` 指令的信息。

表 E-1\. 三斜杠指令

| 指令 | 语法 | 用它来… |
| --- | --- | --- |
| `amd-module` | `<amd-module name="MyComponent" />` | 在编译到 AMD 模块时声明导出名称 |
| `lib` | `<reference lib="dom" />` | 声明你的类型声明依赖于 TypeScript 内置的哪些 `lib` |
| `path` | `<reference path="./path.ts" />` | 声明你的模块依赖哪些 TypeScript 文件 |
| `type` | `<reference types="./path.d.ts" />` | 声明你的模块依赖哪些类型声明文件 |

# 内部指令

你可能永远不会在你自己的代码中使用 `no-default-lib` 指令（表 E-2）。

表 E-2\. 内部三斜杠指令

| 指令 | 语法 | 用它来… |
| --- | --- | --- |
| `no-default-lib` | `<reference no-default-lib="true" />` | 告诉 TypeScript 在这个文件中完全不使用任何 `lib` |

# 废弃的指令

你永远不应该使用 `amd-dependency` 指令（表 E-3），而是坚持使用普通的 `import`。

表 E-3\. 废弃的三斜杠指令

| 指令 | 语法 | 可以用… |
| --- | --- | --- |
| `amd-dependency` | `<amd-dependency path="./a.ts" name="MyComponent" />` | `import` |
