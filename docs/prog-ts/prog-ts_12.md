# 第十二章：构建和运行 TypeScript

如果您在生产环境中部署和运行 JavaScript 应用程序，那么您也知道如何运行 TypeScript 应用程序——一旦您将其编译为 JavaScript，这两者并没有太大的不同。本章讨论的是生产和构建 TypeScript 应用程序，但在 TypeScript 应用程序中并没有太多独特的内容——它大部分也适用于 JavaScript 应用程序。我们将其分为四个部分，涵盖以下内容：

+   您必须做的事情来构建任何 TypeScript 应用程序

+   在服务器上构建和运行 TypeScript 应用程序

+   在浏览器中构建和运行 TypeScript 应用程序

+   为 TypeScript 应用程序构建和发布到 NPM

# 构建您的 TypeScript 项目

构建 TypeScript 项目非常简单。在本节中，我们将涵盖您需要理解的核心概念，以便为计划运行的任何环境构建项目。

## 项目布局

我建议将您的源 TypeScript 代码保存在顶级 *src/* 文件夹中，并将其编译为顶级 *dist/* 文件夹。这种文件结构是一种常见的约定，将源代码和生成的代码分为两个顶级文件夹可以在日后与其他工具集成时简化您的生活。它还使得可以更轻松地将生成的工件排除在源代码控制之外。

尽量坚持这种约定：

```
my-app/
├──dist/
│ ├──index.d.ts
│ ├──index.js
│ └──services/
│   ├──foo.d.ts
│   ├──foo.js
│   ├──bar.d.ts
│   └──bar.js
├──src/
│ ├──index.ts
│ └──services/
│   ├──foo.ts
│   └──bar.ts
```

## 工件

当您将 TypeScript 程序编译为 JavaScript 时，TSC 可以为您生成几种不同的工件（表 12-1）。

表 12-1\. TSC 可以为您生成的工件

| 类型 | 文件扩展名 | tsconfig.json 标志 | 默认是否生成？ |
| --- | --- | --- | --- |
| **JavaScript** | *.js* | `{"emitDeclarationOnly": false}` | 是 |
| **源映射** | *.js.map* | `{"sourceMap": true}` | 否 |
| **类型声明** | *.d.ts* | `{"declaration": true}` | 否 |
| **声明映射** | *.d.ts.map* | `{"declarationMap": true}` | 否 |

第一种工件——JavaScript 文件——应该很熟悉。TSC 将您的 TypeScript 代码编译为 JavaScript，然后您可以使用类似 NodeJS 或 Chrome 的 JavaScript 平台运行它。如果您运行 `tsc yourfile.ts`，TSC 将对 *yourfile.ts* 进行类型检查并将其编译为 JavaScript。

第二种类型的工件——源映射——是特殊文件，将生成的每个 JavaScript 片段链接回生成它的 TypeScript 文件的特定行和列。这对于调试代码很有帮助（Chrome DevtTools 将显示您的 TypeScript 代码，而不是生成的 JavaScript），并且可以将 JavaScript 异常堆栈跟踪的行和列映射回 TypeScript（如果您提供了源映射，像“错误监控”中提到的工具会自动执行此查找）。

第三种工件——类型声明——允许其他 TypeScript 项目利用您生成的类型。

最后，声明映射被用来加快 TypeScript 项目的编译时间。你将在“项目引用”中详细了解它们。本章的其余部分将讨论如何以及为何生成这些工件。

## 调整你的编译目标

JavaScript 可能是一个不寻常的语言来处理：它不仅具有快速发展的规范，每年发布一个版本，而且作为程序员，你并不能总是控制你的程序运行的平台实现了哪个 JavaScript 版本。此外，许多 JavaScript 程序是*同构*的，这意味着你可以在服务器或客户端上运行它们。例如：

+   如果你在你控制的服务器上运行后端 JavaScript 程序，那么你可以精确控制它将运行的 JavaScript 版本。

+   如果你将你的后端 JavaScript 程序作为开源项目发布，你不知道消费者的 JavaScript 平台将支持哪个 JavaScript 版本。在 NodeJS 环境中，你能做的最好的是声明支持的 NodeJS 版本范围，但在浏览器环境中，你就没那么幸运了。

+   如果你在浏览器中运行你的 JavaScript，你不知道人们将使用哪个浏览器来运行它——最新版本的 Chrome、Firefox 或支持大多数现代 JavaScript 特性的 Edge，稍微过时的其中一个浏览器版本，可能缺少一些前沿功能的浏览器，像 Internet Explorer 8 这样的古老浏览器，或者在你车库里运行的 PlayStation 4 上的嵌入式浏览器。你能做的最好的是定义人们的浏览器需要支持的最小功能集，为尽可能多的这些功能提供 polyfill，并尝试检测用户是否在真正旧的浏览器上，你的应用无法运行，并显示他们需要升级的消息。

+   如果你发布了一个同构的 JavaScript 库（例如，在浏览器和服务器上都能运行的日志库），那么你必须支持一个最低的 NodeJS 版本和一系列浏览器 JavaScript 引擎和版本。

并非每个 JavaScript 环境都能在开箱即用时支持每个 JavaScript 特性，但你仍应该尝试用最新的语言版本编写代码。有两种方法可以做到这一点：

1.  *转译*（即自动转换）应用程序，从最新版本的 JavaScript 转换为目标平台支持的最老的 JavaScript 版本。我们为语言特性如`for..of`循环和`async`/`await`，可以自动转换为`for`循环和`.then`调用，做到这一点。

1.  *Polyfill*（即提供实现）任何现代特性，在你运行的 JavaScript 运行时中缺失。我们为 JavaScript 标准库（如`Promise`、`Map`和`Set`）以及原型方法（如`Array.prototype.includes`和`Function.prototype.bind`）提供这些功能。

TSC 内置支持将你的代码转译为旧版本的 JavaScript，但不会自动为你提供 polyfill。这值得再强调一下：TSC 会为旧环境转译大多数 JavaScript 特性，但不会提供缺失特性的实现。

TSC 提供了三个设置选项，帮助你调整目标环境：

+   `target` 设置你想要编译到的 JavaScript 版本：`es5`、`es2015` 等等。

+   `module` 设置你想要的模块系统：`es2015` 模块、`commonjs` 模块、`systemjs` 模块等等。

+   `lib` 告诉 TypeScript 目标环境中可用的 JavaScript 特性：`es5` 特性、`es2015` 特性、`dom` 等等。它并不实现这些特性——这就是 polyfill 的作用——但它告诉 TypeScript 这些特性是可用的（无论是原生支持还是通过 polyfill）。

你计划在哪个环境中运行应用程序决定了你应该用 `target` 编译到哪个 JavaScript 版本，以及应该设置哪些 `lib`。如果不确定，通常 `es5` 对于两者都是一个安全的默认值。你设置 `module` 取决于你是在目标 NodeJS 还是浏览器环境，以及如果是后者，你使用的模块加载器是什么。

###### 提示

如果你需要支持一组不寻常的平台，请查阅 Juriy Zaytsev（又名 Kangax）的 [兼容性表格](http://kangax.github.io/compat-table/es5/)，了解你的目标平台原生支持哪些 JavaScript 特性。

让我们更深入地了解 `target` 和 `lib`；关于 “在服务器上运行 TypeScript” 和 “在浏览器中运行 TypeScript”，我们将留给各自的章节。

### target

TSC 的内置转译器支持将大多数 JavaScript 特性转译为旧版本的 JavaScript，这意味着你可以使用最新的 TypeScript 版本编写代码，并将其转译为你需要支持的任何 JavaScript 版本。由于 TypeScript 支持最新的 JavaScript 特性（例如 `async`/`await`，目前并非所有主要 JavaScript 平台都支持），你几乎总是会利用这个内置转译器将你的代码转换为 NodeJS 和浏览器今天能理解的代码。

让我们看看 TSC 在转译为旧版 JavaScript 时支持哪些具体的 JavaScript 特性，以及不支持哪些特性（表 12-2 和 表 12-3）^(1)。

###### 注意

在过去，JavaScript 语言每隔几年就会发布一个新版本，版本号逐步增加（ES1，ES3，ES5，ES6）。从 2015 年开始，JavaScript 语言现在采用每年发布一次的周期，每个语言版本都以发布的年份命名（ES2015，ES2016 等）。然而，一些 JavaScript 特性在实际进入特定 JavaScript 版本之前就已经获得了 TypeScript 的支持；我们称这些特性为“ESNext”（即下一个修订版本）。

表 12-2\. TSC 进行了转译

| 版本 | 功能 |
| --- | --- |
| ES2015 | `const`, `let`, `for..of` 循环, 数组/对象展开 (`...`), 带标签的模板字符串, 类, 生成器, 箭头函数, 函数默认参数, 函数剩余参数, 解构声明/赋值/参数 |
| ES2016 | 指数运算符 (`**`) |
| ES2017 | `async` 函数，等待 promise |
| ES2018 | `async` 迭代器 |
| ES2019 | catch 子句中的可选参数 |
| ESNext | 数字分隔符 (`123_456`) |

表 12-3\. TSC 不会进行转译

| 版本 | 功能 |
| --- | --- |
| ES5 | 对象的 getter/setter |
| ES2015 | 正则表达式 `y` 和 `u` 标志 |
| ES2018 | 正则表达式 `s` 标志 |
| ESNext | BigInt (`123n`) |

要设置转译目标，请打开您的 *tsconfig.json* 并将 `target` 字段设置为：

+   `es3` 代表 ECMAScript 3

+   `es5` 代表 ECMAScript 5（如果不确定使用什么版本，这是一个很好的默认值）

+   `es6` 或 `es2015` 代表 ECMAScript 2015

+   `es2016` 代表 ECMAScript 2016

+   `es2017` 代表 ECMAScript 2017

+   `es2018` 代表 ECMAScript 2018

+   `esnext` 代表最新的 ECMAScript 修订版本

例如，要编译为 ES5：

```
{
  "compilerOptions": {
    "target": "es5"
  }
}
```

### lib

正如我所提到的，将您的代码转译为较旧的 JavaScript 版本有一个小问题：虽然大多数语言特性可以安全地转译（`let` 转为 `var`，`class` 转为 `function`），但是如果您的目标环境不支持较新的库特性，仍然需要自己 *polyfill* 功能。例如像 `Promise` 和 `Reflect` 这样的实用工具，以及像 `Map`、`Set` 和 `Symbol` 这样的数据结构。如果您的目标是最新版本的 Chrome、Firefox 或 Edge 等先进环境，通常不需要任何 polyfills；但是如果您的目标是几个版本之前的浏览器，或者大多数 NodeJS 环境，则需要 polyfill 遗漏的特性。

幸运的是，您不需要自己编写 polyfill。相反，您可以从流行的 polyfill 库如 [`core-js`](https://www.npmjs.com/package/core-js) 安装它们，或者通过运行经过类型检查的 TypeScript 代码通过 [`@babel/polyfill`](https://babeljs.io/docs/en/babel-polyfill) 自动将 polyfills 添加到您的代码中。

###### 提示

如果你计划在浏览器中运行你的应用程序，请注意不要通过包含每一个 polyfill 来使你的 JavaScript 捆绑包膨胀，无论你的代码运行的浏览器是否实际需要它们 —— 你的目标平台可能已经支持一些你正在 polyfill 的功能。相反，使用像 [Polyfill.io](https://polyfill.io/v2/docs/) 这样的服务，只加载你的用户浏览器需要的那些 polyfill。

一旦你在代码中添加了 polyfill，就需要告诉 TSC 你的环境已经支持了你 polyfill 的功能 —— 输入你的 *tsconfig.json* 的 `lib` 字段。例如，如果你已经 polyfill 了所有 ES2015 特性以及 ES2016 的 `Array.prototype.includes`，你可以使用这个配置：

```
{
  "compilerOptions": {
    "lib": ["es2015", "es2016.array.includes"]
  }
}
```

如果你在浏览器中运行你的代码，还要为 DOM 类型声明启用，比如 `window`、`document`，以及在浏览器中运行 JavaScript 时获得的所有其他 API：

```
{
  "compilerOptions": {
  "lib": ["es2015", "es2016.array.include", `"dom"`]
  }
}

```

要获取支持的 libs 的完整列表，请运行 `tsc --help`。

## 启用源映射

源映射是一种将你的编译后代码与生成它的源代码进行关联的方法。大多数开发者工具（如 Chrome DevTools）、错误报告和日志框架以及构建工具都支持源映射。由于典型的构建流水线可能会生成与最初的代码非常不同的代码（例如，你的流水线可能会将 TypeScript 编译为 ES5 JavaScript，使用 Rollup 进行树摇，使用 Prepack 进行预评估，然后使用 Uglify 进行压缩），在整个构建流水线中使用源映射可以大大简化调试生成的 JavaScript 的过程。

通常建议在开发中使用源映射，并在浏览器和服务器环境中将源映射发布到生产环境。不过，有一个注意事项：如果你依赖于某种程度上的安全性通过混淆来保护你的浏览器代码，那么不要在生产环境中将源映射发布到浏览器中。

## 项目引用

随着你的应用程序增长，TSC 对你的代码进行类型检查和编译所需的时间会越来越长。这个时间大致与你的代码库的大小成正比增长。在本地开发时，缓慢的增量编译时间会严重拖慢你的开发速度，并使得使用 TypeScript 变得困难。

为了解决这个问题，TSC 提供了一个名为 *项目引用* 的功能，大大加快了编译时间，包括增量编译时间。对于任何具有几百个或更多文件的项目，项目引用都是必不可少的。

像这样使用它们：

1.  将你的 TypeScript 项目分割成多个项目。一个项目只是一个包含 *tsconfig.json* 和一些 TypeScript 代码的文件夹。尝试以这样一种方式分割你的代码，使得通常一起更新的代码位于同一个文件夹中。

1.  在每个项目文件夹中，创建一个 *tsconfig.json*，至少包括以下内容：

    ```
    {
      "compilerOptions": {
        "composite": true,
        "declaration": true,
        "declarationMap": true,
        "rootDir": "."
      },
      "include": [
        "./**/*.ts"
      ],
      "references": [
        {
          "path": "../myReferencedProject",
          "prepend": true
        }
      ],
    }
    ```

    关键在于：

    +   `composite` 表示这个文件夹是一个更大的 TypeScript 项目的子项目。

    +   `declaration`，告诉 TSC 为这个项目生成 *.d.ts* 声明文件。项目引用的工作方式是，项目可以访问彼此的声明文件和生成的 JavaScript，但不能访问它们的源 TypeScript 文件。这创建了一个边界，超出这个边界 TSC 将不会尝试重新检查或重新编译你的代码：如果你在子项目 *A* 中更新了一行代码，TSC 不必重新检查你的其他子项目 *B*；TSC 只需要检查 *B* 的类型声明以查找类型错误。这是使项目引用在重建大型项目时如此高效的核心行为。

    +   `declarationMap`，告诉 TSC 为生成的类型声明构建源映射。

    +   `references`，是一个包含你的子项目依赖的子项目数组。每个引用的 `path` 应该指向一个包含 *tsconfig.json* 文件的文件夹，或直接指向一个 TSC 配置文件（如果你的配置文件不叫 *tsconfig.json*）。`prepend` 将连接由你引用的子项目生成的 JavaScript 和源映射到你的子项目生成的 JavaScript 和源映射中。注意，只有当你使用 `outFile` 时，`prepend` 才有用处——如果你不使用 `outFile`，你可以放弃 `prepend`。

    +   `rootDir`，明确指定该子项目应该相对于根项目 (`.`) 进行编译。或者，你可以指定一个 `outDir`，作为根项目 `outDir` 的子文件夹。

1.  创建一个根 *tsconfig.json*，引用任何尚未被其他子项目引用的子项目：

    ```
    {
      "files": [],
      "references": [
        {"path": "./myProject"},
        {"path": "./mySecondProject"}
      ]
    }
    ```

1.  现在当你使用 TSC 编译你的项目时，使用 `build` 标志告诉 TSC 考虑项目引用：

    ```
    tsc --build # Or, tsc -b for short
    ```

###### 警告

目前项目引用是 TypeScript 的一个新功能，有些地方还不够成熟。在使用时，请务必注意以下几点：

+   在克隆或重新获取后，重新构建整个项目（使用 `tsc -b`），以重新生成任何丢失或过时的 *.d.ts* 文件。或者，检查你生成的 *d.ts* 文件。

+   不要在项目引用中使用 `noEmitOnError: false` —— TSC 将始终将选项硬编码为 `true`。

+   手动确保给定的子项目不被多于一个其他子项目预置。否则，双重预置的子项目将在你的编译输出中显示两次。注意，如果你只是引用而不是预置，那就没问题。

## 错误监控

TypeScript 在编译时会警告你有关错误，但你也需要一种方法来了解用户在运行时遇到的异常，以便你可以尝试在编译时防止它们（或者至少修复导致运行时错误的 bug）。使用像 [Sentry](https://sentry.io) 或 [Bugsnag](https://bugsnag.com) 这样的错误监控工具来报告和整理你的运行时异常。

# 在服务器上运行 TypeScript

要在 NodeJS 环境中运行你的 TypeScript 代码，只需将你的代码编译成 ES2015 JavaScript（或者如果你的目标是旧版 NodeJS，则编译成 ES5），并且将你的*tsconfig.json*的 module 标志设置为`commonjs`：

```
{
  "compilerOptions": {
    "target": "es2015",
    "module": "commonjs"
  }
}
```

这将把你的 ES2015 的`import`和`export`语句编译成`require`和`module.exports`，因此你的代码可以在 NodeJS 上运行，无需进一步打包。

如果你正在使用源映射（你应该！），你需要将你的源映射提供给你的 NodeJS 进程。只需从 NPM 获取[`source-map-support`](https://www.npmjs.com/package/source-map-support)包，并按照该包的设置说明进行设置。大多数进程监控、日志记录和错误报告工具（如[PM2](https://www.npmjs.com/package/pm2)，[Winston](https://www.npmjs.com/package/winston)和[Sentry](https://sentry.io)）都内置了对源映射的支持。

# 在浏览器中运行 TypeScript

编译 TypeScript 以在浏览器中运行需要比在服务器上运行 TypeScript 多一些工作。

首先，选择一个模块系统进行编译。一个很好的经验法则是，如果要发布供他人使用的库（例如在 NPM 上），最好使用`umd`以确保与各种项目中可能使用的模块打包工具兼容。

如果你只打算自己使用代码而不将其发布到 NPM，请根据你正在使用的模块打包工具来决定编译到哪种格式。查看你的打包工具的文档——例如，Webpack 和 Rollup 最适合 ES2015 模块，而 Browserify 需要 CommonJS 模块。以下是一些指南：

+   如果你正在使用[SystemJS](https://github.com/systemjs/systemjs)模块加载器，请将`module`设置为`systemjs`。

+   如果你通过像[Webpack](https://webpack.js.org)或[Rollup](https://github.com/rollup/rollup)这样的 ES2015 模块感知的模块打包工具运行你的代码，请将`module`设置为`es2015`或更高版本。

+   如果你正在使用 ES2015 感知的模块打包工具，并且你的代码使用动态导入（参见“动态导入”），请将`module`设置为`esnext`。

+   如果你正在为其他项目构建一个库，并且在`tsc`之后没有任何其他构建步骤，请通过将`module`设置为`umd`来最大化与人们使用的不同加载器的兼容性。

+   如果你正在使用像[Browserify](https://github.com/browserify/browserify)这样的 CommonJS 打包工具打包你的模块，请将`module`设置为`commonjs`。

+   如果你计划使用[RequireJS](https://requirejs.org)或其他 AMD 模块加载器加载你的代码，请将`module`设置为`amd`。

+   如果你希望你的顶层导出在`window`对象上全局可用（例如，如果你是墨索里尼的侄孙），请将`module`设置为`none`。请注意，如果你的代码处于模块模式下，TSC 将尝试通过编译成`commonjs`来遏制你对其他软件工程师施加痛苦的热情（参见“模块模式与脚本模式”）。

接下来，配置你的构建流水线，将所有的 TypeScript 编译成单个 JavaScript 文件（通常称为“捆绑包”）或一组 JavaScript 文件。虽然 TSC 可以通过 `outFile` TSC 标志为小型项目执行此操作，但该标志仅限于生成 SystemJS 和 AMD 捆绑包。由于 TSC 不支持构建插件和智能代码分割，就像 Webpack 这样的专用构建工具一样，你很快就会发现自己需要一个更强大的打包工具。

因此，对于前端项目，你应该从一开始就使用更强大的构建工具。无论你使用什么构建工具，都有适用于该工具的 TypeScript 插件，例如：

+   [`ts-loader`](http://bit.ly/2Gw3uH2) 适用于 [Webpack](https://webpack.js.org)

+   [`tsify`](http://bit.ly/2KOaZgw) 适用于 [Browserify](http://bit.ly/2IDpfGe)

+   [`@babel/preset-typescript`](http://bit.ly/2vc2Sjy) 适用于 [Babel](https://babeljs.io)

+   [`gulp-typescript`](http://bit.ly/2vanubN) 适用于 [Gulp](https://gulpjs.com)

+   [`grunt-ts`](http://bit.ly/2PgUXuq) 适用于 [Grunt](https://gruntjs.com)

尽管详细讨论如何优化 JavaScript 捆绑包以实现快速加载超出了本书的范围，但一些简短的建议（不特定于 TypeScript）包括：

+   保持代码模块化，避免在代码中引入隐式依赖（当你将东西分配给 `window` 全局变量或其他全局变量时可能会发生这种情况），这样你的构建工具就能更准确地分析项目的依赖关系。

+   使用动态导入来延迟加载不需要在初始页面加载时就加载的代码，这样可以避免不必要地阻塞页面渲染。

+   充分利用构建工具的自动代码分割功能，以避免加载过多的 JavaScript 代码，从而不必要地减慢页面加载速度。

+   制定页面加载时间测量策略，可以是通过合成数据或理想情况下使用真实用户数据。随着应用程序的增长，初始加载时间可能会变得越来越慢；只有在能够测量加载时间的情况下，你才能优化它。像 [New Relic](https://newrelic.com) 和 [Datadog](https://www.datadoghq.com) 这样的工具在这方面非常宝贵。

+   尽量使生产构建与开发构建尽可能相似。两者差异越大，只会在生产环境中才会出现的难以修复的错误就越多。

+   最后，在将 TypeScript 部署到浏览器中运行时，需要制定一个策略来填充缺失的浏览器功能。这可以是一组标准的 polyfill，作为每个捆绑包的一部分进行发布，或者根据用户浏览器支持的功能动态选择 polyfill。

# 将你的 TypeScript 代码发布到 NPM

编译 TypeScript 代码以便其他 TypeScript 和 JavaScript 项目可以使用。在将 TypeScript 编译为供外部使用的 JavaScript 时，有一些最佳实践需要牢记：

+   生成源映射，这样你就可以调试自己的代码。

+   编译为 ES5，以便其他人可以轻松地构建和运行你的代码。

+   在选择要编译到的模块格式时要注意（UMD、CommonJS、ES2015 等）。

+   生成类型声明，以便其他 TypeScript 用户可以为你的代码提供类型。

首先用 `tsc` 将你的 TypeScript 编译为 JavaScript，并生成相应的类型声明。确保配置你的*tsconfig.json*以最大化与流行的 JavaScript 环境和构建系统的兼容性（关于此更多信息，请参见“构建您的 TypeScript 项目”）：

```
{
"compilerOptions": {
  "declaration": true,
  "module": "umd",
  "sourceMaps": true,
  "target": "es5"
  }
}
```

接下来，在你的*.npmignore*中将你的 TypeScript 源代码列入黑名单，以避免将其发布到 NPM 时使包大小膨胀。并且在你的*.gitignore*中排除生成的工件，以避免污染你的 Git 仓库：

```
# .npmignore

*.ts # Ignore .ts files
!*.d.ts # Allow .d.ts files
```

```
# .gitignore

*.d.ts # Ignore .d.ts files
*.js # Ignore .js files
```

###### 注意

如果你坚持推荐的项目布局，并将源文件保存在*src/*中，生成的文件保存在*dist/*中，你的*.ignore*文件将会更简单：

```
# .npmignore

src/ # Ignore source files
```

```
# .gitignore

dist/ # Ignore generated files
```

最后，在你项目的*package.json*中添加一个 `"types"` 字段，指示它带有类型声明（注意这不是强制的，但对于使用 TypeScript 的任何消费者，这是一个有用的提示），并添加一个脚本来在发布之前构建你的包，以确保你的包的 JavaScript、类型声明和源映射始终保持更新并与你编译它们的 TypeScript 同步：

```
{
  "name": "my-awesome-typescript-project",
  "version": "1.0.0",
  "main": "dist/index.js",
  "types": "dist/index.d.ts",
  "scripts": {
    "prepublishOnly": "tsc -d"
  }
}
```

就这样！现在当你将你的包`npm publish`到 NPM 时，NPM 将自动将你的 TypeScript 编译为可供既使用 TypeScript 的人（具有完全的类型安全性）使用，也供使用 JavaScript 的人（如果他们的代码编辑器支持的话，有一定的类型安全性）使用的格式。

# 三斜杠指令

TypeScript 自带一个鲜为人知、使用稀少且大多已过时的特性，称为*三斜杠指令*。这些指令是特别格式化的 TypeScript 注释，用作给 TSC 的指令。

它们有几种不同的风格，在本节中，我们将只涵盖其中的两种：`types`，用于省略仅类型的完整模块导入，和`amd-module`，用于命名生成的 AMD 模块。有关完整参考，请参见附录 E。

## types 指令

当你从模块导入东西时，根据你导入的内容，当你将代码编译为 JavaScript 时，TypeScript 不会总是需要生成 `import` 或 `require` 调用。如果你有一个 `import` 语句，其导出仅在模块的类型位置中使用（即，你只是从一个模块中导入了一个类型），TypeScript 将不会为该 `import` 生成任何 JavaScript 代码——可以将其视为仅存在于类型级别。这个特性称为*导入省略*。

唯一的例外是用于副作用的导入：如果你导入整个模块（而不导入特定的导出或通配符），当你编译 TypeScript 时，该导入将生成 JavaScript 代码。例如，你可能这样做，以确保脚本模式模块中定义的环境类型在你的程序中可用（就像我们在“安全地扩展原型”中所做的那样）。例如：

```
// global.ts
type MyGlobal = number

// app.ts
import './global'
```

使用 `tsc app.ts` 将 *app.ts* 编译为 JavaScript 后，你会注意到 `./global` 的导入并没有被省略：

```
// app.js
import './global'
```

如果你发现自己写了这样的导入语句，你可能需要首先确保你的导入确实需要使用副作用，并且没有其他方法可以重写你的代码，使得导入的值或类型更加明确（例如，`import {MyType} from './global'` — TypeScript 将会为你省略这部分 — 而不是 `import './global'`）。或者，看看是否可以在你的 *tsconfig.json* 的 `types`、`files` 或 `include` 字段中包含你的环境类型，避免导入整个模块。

如果以上两种方式对你的使用场景都不适用，并且你希望继续使用完整的模块导入但又避免生成 JavaScript 的 `import` 或 `require` 调用，可以使用 `types` 的三斜线指令。三斜线指令由三个斜线 `///` 开头，接着是几个可能的 XML 标签之一，每个标签有自己的一组必需属性。对于 `types` 指令来说，它看起来像这样：

+   声明对环境类型声明的依赖：

    ```
    /// <reference types="./global" />
    ```

+   声明对 *@types/jasmine/index.d.ts* 的依赖：

    ```
    /// <reference types="jasmine" />
    ```

你可能不会经常使用这个指令。如果确实使用了，请重新考虑如何在项目中使用类型，并考虑是否有办法减少对环境类型的依赖。

## amd-module 指令

当将你的 TypeScript 代码编译为 AMD 模块格式（在你的 *tsconfig.json* 中指定为 `{"module": "amd"}`）时，默认情况下 TypeScript 将生成匿名的 AMD 模块。你可以使用 AMD 的三斜线指令来为生成的模块指定名称。

假设你有以下代码：

```
export let LogService = {
  log() {
    // ...
  }
}
```

编译为 `amd` 模块格式时，TSC 生成了以下 JavaScript 代码：

```
define(['require', 'exports'], function(require, exports) {
  exports.__esModule = true
  exports.LogService = {
    log() {
      // ...
    }
  }
})
```

如果你熟悉 AMD 模块格式，你可能注意到这是一个匿名的 AMD 模块。要为你的 AMD 模块指定名称，可以在你的代码中使用 `amd-module` 的三斜线指令：

```
/// <amd-module name="LogService" /> ![1](img/1.png)
export let LogService = { ![2](img/2.png)
  log() {
    // ...
  }
}
```

![1](img/#co_building_and_running_typescript_CO1-1)

我们使用 `amd-module` 指令，并在其上设置了 `name` 属性。

![2](img/#co_building_and_running_typescript_CO1-2)

其余代码保持不变。

使用 TSC 重新编译到 AMD 模块格式后，我们现在得到以下 JavaScript 代码：

```
*`/// <amd-module name='LogService' />`*
define(`'LogService'``,` ['require', 'exports'], function(require, exports) {
  exports.__esModule = true
  exports.LogService = {
    log() {
      *`// ...`*
    }
  }
})

```

当编译为 AMD 模块时，使用 `amd-module` 指令可以使你的代码更易于捆绑和调试（或者，如果可能的话，切换到更现代的模块格式，如 ES2015 模块）。

# 概要

本章我们涵盖了构建和在生产环境中运行 TypeScript 应用程序所需的一切内容，无论是在浏览器还是服务器上。我们讨论了如何选择要编译的 JavaScript 版本，如何标记环境中可用的库（以及如何在缺失时填充库），以及如何构建和发布带有源映射的应用程序，以便在生产中进行调试和在本地开发时更轻松。然后，我们探讨了如何将您的 TypeScript 项目模块化以保持快速编译时间。最后，我们介绍了如何在服务器和浏览器上运行您的 TypeScript 应用程序，如何将您的 TypeScript 代码发布到 NPM 供他人使用，import elision 的工作原理，以及对于 AMD 用户如何使用三斜杠指令命名您的模块。

^(1) 如果您使用了 TSC 不会转译的语言特性，并且您的目标环境也不支持它，通常可以找到一个 Babel 插件来为您转译它。要找到最新的插件，请在您喜欢的搜索引擎中搜索“babel plugin <feature name>”。
