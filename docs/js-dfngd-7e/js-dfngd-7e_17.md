# 第十七章：JavaScript 工具和扩展

恭喜您达到本书的最后一章。如果您已经阅读了前面的所有内容，现在您对 JavaScript 语言有了详细的了解，并知道如何在 Node 和 Web 浏览器中使用它。本章是一种毕业礼物：它介绍了许多 JavaScript 程序员发现有用的重要编程工具，并描述了核心 JavaScript 语言的两个广泛使用的扩展。无论您是否选择为自己的项目使用这些工具和扩展，您几乎肯定会在其他项目中看到它们的使用，因此至少了解它们是很重要的。

本章涵盖的工具和语言扩展包括：

+   ESLint 用于在代码中查找潜在的错误和样式问题。

+   使用 Prettier 以标准化方式格式化您的 JavaScript 代码。

+   Jest 作为编写 JavaScript 单元测试的一体化解决方案。

+   npm 用于管理和安装程序依赖的软件库。

+   代码捆绑工具——如 webpack、Rollup 和 Parcel——将您的 JavaScript 代码模块转换为用于 Web 的单个捆绑包。

+   Babel 用于将使用全新语言特性（或语言扩展）的 JavaScript 代码转换为可以在当前 Web 浏览器中运行的 JavaScript 代码。

+   JSX 语言扩展（由 React 框架使用）允许您使用类似 HTML 标记的 JavaScript 表达式描述用户界面。

+   Flow 语言扩展（或类似的 TypeScript 扩展）允许您使用类型注释注释您的 JavaScript 代码，并检查代码是否具有类型安全性。

本章不会以任何全面的方式记录这些工具和扩展。目标只是以足够深度解释它们，以便您了解它们为何有用以及何时可能需要使用它们。本章涵盖的所有内容在 JavaScript 编程世界中被广泛使用，如果您决定采用工具或扩展，您会在网上找到大量文档和教程。

# 17.1 使用 ESLint 进行 Linting

在编程中，术语*lint*指的是在技术上正确但不雅观、可能存在错误或以某种方式不够优化的代码。*linter*是一种用于检测代码中 lint 的工具，*linting*是在代码上运行 linter 的过程（然后修复代码以消除 lint，使 linter 不再抱怨）。

今天 JavaScript 最常用的 linter 是[ESLint](https://eslint.org)。如果您运行它，然后花时间实际修复它指出的问题，它将使您的代码更清洁，更不容易出现错误。考虑以下代码：

```js
var x = 'unused';

export function factorial(x) {
    if (x == 1) {
      return 1;
    } else {
        return x * factorial(x-1)
    }
}
```

如果您在此代码上运行 ESLint，可能会得到如下输出：

```js
$ eslint code/ch17/linty.js

code/ch17/linty.js
  1:1   error    Unexpected var, use let or const instead      no-var
  1:5   error    'x' is assigned a value but never used        no-unused-vars
  1:9   warning  Strings must use doublequote                  quotes
  4:11  error    Expected '===' and instead saw '=='           eqeqeq
  5:1   error    Expected indentation of 8 spaces but found 6  indent
  7:28  error    Missing semicolon                             semi

✖ 6 problems (5 errors, 1 warning)
  3 errors and 1 warning potentially fixable with the `--fix` option.
```

有时 linter 可能看起来很挑剔。我们是使用双引号还是单引号真的很重要吗？另一方面，正确的缩进对于可读性很重要，使用`===`和`let`而不是`==`和`var`可以保护您免受微妙错误的影响。未使用的变量是代码中的累赘——没有理由保留它们。

ESLint 定义了许多 linting 规则，并具有添加许多其他规则的插件生态系统。但 ESLint 是完全可配置的，您可以定义一个配置文件来调整 ESLint 以强制执行您想要的规则，仅限于这些规则。

# 17.2 使用 Prettier 进行 JavaScript 格式化

一些项目使用 linter 的原因之一是强制执行一致的编码风格，以便当程序员团队共同工作在共享的代码库上时，他们使用兼容的代码约定。这包括代码缩进规则，但也可以包括诸如首选引号类型以及`for`关键字和其后的开括号之间是否应该有空格等内容。

强制代码格式规则的现代替代方法是采用类似 [Prettier](https://prettier.io) 的工具，自动解析和重新格式化所有代码。

假设你已经编写了以下函数，它可以工作，但格式不太规范：

```js
function factorial(x)
{
         if(x===1){return 1}
           else{return x*factorial(x-1)}
}
```

运行 Prettier 对这段代码进行了缩进修复，添加了缺失的分号，围绕二进制运算符添加了空格，并在 `{` 之后和 `}` 之前插入了换行符，使代码看起来更加传统：

```js
$ prettier factorial.js
function factorial(x) {
  if (x === 1) {
    return 1;
  } else {
    return x * factorial(x - 1);
  }
}
```

如果你使用 `--write` 选项调用 Prettier，它将简单地在原地重新格式化指定的文件，而不是打印重新格式化的版本。如果你使用 `git` 管理你的源代码，你可以在提交钩子中使用 `--write` 选项调用 Prettier，这样代码就会在检入之前自动格式化。

如果你配置你的代码编辑器在每次保存文件时自动运行 Prettier，Prettier 就会变得非常强大。我觉得写松散的代码然后看到它被自动修复很解放。

Prettier 是可配置的，但只有少数选项。你可以选择最大行长度、缩进量、是否应该使用分号、字符串是单引号还是双引号，以及其他一些内容。一般来说，Prettier 的默认选项是相当合理的。其思想是你只需为你的项目采用 Prettier，然后就再也不用考虑代码格式了。

就我个人而言，我非常喜欢在 JavaScript 项目中使用 Prettier。然而，在这本书中的代码中我没有使用它，因为在我的许多代码中，我依赖仔细的手动格式化来垂直对齐我的注释，而 Prettier 会搞乱它们。

# 17.3 使用 Jest 进行单元测试

写测试是任何非平凡编程项目的重要部分。像 JavaScript 这样的动态语言支持测试框架，大大减少了编写测试所需的工作量，几乎让测试编写变得有趣！JavaScript 有很多测试工具和库，许多都是以模块化的方式编写的，因此可以选择一个库作为测试运行器，另一个库用于断言，第三个库用于模拟。然而，在本节中，我们将描述[ Jest ](https://jestjs.io)，这是一个流行的框架，包含了你在一个单一包中所需的一切。

假设你已经编写了以下函数：

```js
const getJSON = require("./getJSON.js");

/**
 * getTemperature() takes the name of a city as its input, and returns
 * a Promise that will resolve to the current temperature of that city,
 * in degrees Fahrenheit. It relies on a (fake) web service that returns
 * world temperatures in degrees Celsius.
 */
module.exports = async function getTemperature(city) {
    // Get the temperature in Celsius from the web service
    let c = await getJSON(
        `https://globaltemps.example.com/api/city/${city.toLowerCase()}`
    );
    // Convert to Fahrenheit and return that value.
    return (c * 5 / 9) + 32;  // TODO: double-check this formula
};
```

对于这个函数的一个很好的测试集可能会验证 `getTemperature()` 是否获取了正确的 URL，并且是否正确地转换了温度标度。我们可以使用类似下面的基于 Jest 的测试来做到这一点。这段代码定义了 `getJSON()` 的模拟实现，以便测试实际上不会发出网络请求。由于 `getTemperature()` 是一个异步函数，所以测试也是异步的——测试异步函数可能有些棘手，但 Jest 让它相对容易：

```js
// Import the function we are going to test
const getTemperature = require("./getTemperature.js");

// And mock the getJSON() module that getTemperature() depends on
jest.mock("./getJSON");
const getJSON = require("./getJSON.js");

// Tell the mock getJSON() function to return an already resolved Promise
// with fulfillment value 0.
getJSON.mockResolvedValue(0);

// Our set of tests for getTemperature() begins here
describe("getTemperature()", () => {
    // This is the first test. We're ensuring that getTemperature() calls
    // getJSON() with the URL that we expect
    test("Invokes the correct API", async () => {
        let expectedURL = "https://globaltemps.example.com/api/city/vancouver";
        let t = await(getTemperature("Vancouver"));
        // Jest mocks remember how they were called, and we can check that.
        expect(getJSON).toHaveBeenCalledWith(expectedURL);
    });

    // This second test verifies that getTemperature() converts
    // Celsius to Fahrenheit correctly
    test("Converts C to F correctly", async () => {
        getJSON.mockResolvedValue(0);                // If getJSON returns 0C
        expect(await getTemperature("x")).toBe(32);  // We expect 32F

        // 100C should convert to 212F
        getJSON.mockResolvedValue(100);              // If getJSON returns 100C
        expect(await getTemperature("x")).toBe(212); // We expect 212F
    });
});
```

写好测试后，我们可以使用 `jest` 命令来运行它，然后我们发现我们的一个测试失败了：

```js
$ jest getTemperature
 FAIL  ch17/getTemperature.test.js
  getTemperature()
    ✓ Invokes the correct API (4ms)
    ✕ Converts C to F correctly (3ms)

  ● getTemperature() › Converts C to F correctly

    expect(received).toBe(expected) // Object.is equality

    Expected: 212
    Received: 87.55555555555556

      29 |         // 100C should convert to 212F
      30 |         getJSON.mockResolvedValue(100); // If getJSON returns 100C
    > 31 |         expect(await getTemperature("x")).toBe(212); // Expect 212F
         |                                           ^
      32 |     });
      33 | });
      34 |

      at Object.<anonymous> (ch17/getTemperature.test.js:31:43)

Test Suites: 1 failed, 1 total
Tests:       1 failed, 1 passed, 2 total
Snapshots:   0 total
Time:        1.403s
Ran all test suites matching /getTemperature/i.
```

我们的 `getTemperature()` 实现使用了错误的公式将摄氏度转换为华氏度。它将乘以 5 再除以 9，而不是乘以 9 再除以 5。如果我们修复代码并再次运行 Jest，我们可以看到测试通过。而且，作为一个奖励，如果我们在调用 `jest` 时添加 `--coverage` 参数，它将计算并显示我们测试的代码覆盖率：

```js
$ jest --coverage getTemperature
 PASS  ch17/getTemperature.test.js
  getTemperature()
    ✓ Invokes the correct API (3ms)
    ✓ Converts C to F correctly (1ms)

------------------|--------|---------|---------|---------|------------------|
File              | % Stmts| % Branch|  % Funcs|  % Lines| Uncovered Line #s|
------------------|--------|---------|---------|---------|------------------|
All files         |   71.43|      100|    33.33|    83.33|                  |
 getJSON.js       |   33.33|      100|        0|       50|                 2|
 getTemperature.js|     100|      100|      100|      100|                  |
------------------|--------|---------|---------|---------|------------------|
Test Suites: 1 passed, 1 total
Tests:       2 passed, 2 total
Snapshots:   0 total
Time:        1.508s
Ran all test suites matching /getTemperature/i.
```

运行我们的测试为我们正在测试的模块提供了 100% 的代码覆盖率，这正是我们想要的。它只为 `getJSON()` 提供了部分覆盖，但我们对该模块进行了模拟，并且并不打算测试它，所以这是预期的。

# 17.4 使用 npm 进行包管理

在现代软件开发中，编写的任何非平凡程序都可能依赖于第三方软件库。例如，如果您在 Node 中编写 Web 服务器，可能会使用 Express 框架。如果您正在创建要在 Web 浏览器中显示的用户界面，则可能会使用像 React、LitElement 或 Angular 这样的前端框架。包管理器使查找和安装这些第三方包变得容易。同样重要的是，包管理器会跟踪代码所依赖的包，并将此信息保存到文件中，以便其他人想要尝试您的程序时，他们可以下载您的代码和依赖项列表，然后使用自己的包管理器安装代码所需的所有第三方包。

npm 是随 Node 捆绑的包管理器，并在§16.1.5 中介绍。它对于客户端 JavaScript 编程和 Node 服务器端编程同样有用。

如果您尝试其他人的 JavaScript 项目，那么在下载他们的代码后，您通常会首先执行`npm install`。这会读取*package.json*文件中列出的依赖项，并下载项目需要的第三方包并将其保存在*node_modules/*目录中。

您还可以输入`npm install <package-name>`来将特定包安装到项目的*node_modules/*目录中：

```js
$ npm install express
```

除了安装命名的包外，npm 还会在项目的*package.json*文件中记录依赖关系。以这种方式记录依赖关系是让其他人通过输入`npm install`来安装这些依赖关系的原因。

另一种依赖关系是开发人员工具的依赖，这些工具是开发人员想要在项目上工作时需要的，但实际上不需要运行代码。例如，如果项目使用 Prettier 来确保所有代码格式一致，那么 Prettier 就是一个“dev dependency”，您可以使用`--save-dev`来安装和记录其中之一：

```js
$ npm install --save-dev prettier
```

有时您可能希望全局安装开发工具，以便它们可以在任何地方访问，即使不是正式项目的代码也可以使用*package.json*文件和*node_modules/*目录。为此，您可以使用`-g`（全局）选项：

```js
$ npm install -g eslint jest
/usr/local/bin/eslint -> /usr/local/lib/node_modules/eslint/bin/eslint.js
/usr/local/bin/jest -> /usr/local/lib/node_modules/jest/bin/jest.js
+ jest@24.9.0
+ eslint@6.7.2
added 653 packages from 414 contributors in 25.596s

$ which eslint
/usr/local/bin/eslint
$ which jest
/usr/local/bin/jest
```

除了“install”命令，npm 还支持“uninstall”和“update”命令，其功能如其名称所示。npm 还有一个有趣的“audit”命令，您可以使用它来查找并修复依赖项中的安全漏洞：

```js
$ npm audit --fix

                       === npm audit security report ===

found 0 vulnerabilities
 in 876354 scanned packages
```

当您为项目本地安装类似 ESLint 这样的工具时，eslint 脚本会出现在*./node_modules/.bin/eslint*中，这使得运行命令变得笨拙。幸运的是，npm 捆绑了一个名为“npx”的命令，您可以使用它来运行本地安装的工具，如`npx eslint`或`npx jest`。（如果您使用 npx 调用尚未安装的工具，它将为您安装它。）

npm 背后的公司还维护着[*https://npmjs.com*](https://npmjs.com)包仓库，其中包含数十万个开源包。但您不必使用 npm 包管理器来访问这个包仓库。替代方案包括[yarn](https://yarnpkg.com)和[pnpm](https://pnpm.js.org)。

# 17.5 代码捆绑

如果您正在编写一个大型的 JavaScript 程序以在网络浏览器中运行，那么您可能需要使用一个代码捆绑工具，特别是如果您使用作为模块交付的外部库。多年来，网络开发人员一直在使用 ES6 模块(§10.3)，早在网络上支持`import`和`export`关键字之前。为了做到这一点，程序员使用一个代码捆绑工具，从程序的主入口点（或入口点）开始，并遵循`import`指令的树，以找到程序所依赖的所有模块。然后，它将所有这些单独的模块文件组合成一个 JavaScript 代码的单个捆绑包，并重写`import`和`export`指令，使代码在这种新形式下工作。结果是一个单个的代码文件，可以加载到不支持模块的网络浏览器中。

ES6 模块现在几乎被所有的网络浏览器支持，但网络开发人员在发布生产代码时仍倾向于使用代码捆绑工具。开发人员发现，当用户首次访问网站时，加载一个中等大小的代码捆绑包比加载许多小模块时用户体验更好。

###### 注意

网络性能是一个众所周知的棘手话题，有很多要考虑的变量，包括浏览器供应商的持续改进，因此确保加载代码的最快方式的唯一方法是进行彻底的测试和仔细的测量。请记住，有一个完全在您控制之下的变量：代码大小。较少的 JavaScript 代码总是比较多的 JavaScript 代码加载和运行更快！

有许多优秀的 JavaScript 捆绑工具可供选择。常用的捆绑工具包括[webpack](https://webpack.js.org)、[Rollup](https://rollupjs.org/guide/en)和[Parcel](https://parceljs.org)。捆绑工具的基本功能大致相同，它们的区别在于可配置性或易用性。Webpack 已经存在很长时间，拥有庞大的插件生态系统，可高度配置，并且可以支持旧的非模块化库。但它也可能复杂且难以配置。另一端是 Parcel，它被设计为一个零配置的替代方案，只需简单地做正确的事情。

除了执行基本的捆绑外，捆绑工具还可以提供一些附加功能：

+   一些程序可能有多个入口点。例如，一个具有多个页面的网络应用程序可以为每个页面编写不同的入口点。打包工具通常允许您为每个入口点创建一个捆绑包，或者创建一个支持多个入口点的单个捆绑包。

+   程序可以使用`import()`的功能形式(§10.3.6)，而不是静态形式，在实际需要时动态加载模块，而不是在程序启动时静态加载它们。这通常是改善程序启动时间的好方法。支持`import()`的捆绑工具可能能够生成多个输出捆绑包：一个在启动时加载，一个或多个在需要时动态加载。如果动态加载的模块共享依赖关系，那么确定要生成多少个捆绑包就变得棘手了，您可能需要手动配置捆绑工具来解决这个问题。

+   捆绑工具通常可以输出一个*源映射*文件，定义了捆绑包中代码行与原始源文件中对应行之间的映射关系。这使得浏览器开发工具可以自动显示 JavaScript 错误的原始未捆绑位置。

+   有时当你将一个模块导入到你的程序中时，你可能只使用其中的一部分功能。一个好的打包工具可以分析代码以确定哪些部分是未使用的，可以从捆绑包中省略。这个功能被戏称为“tree-shaking”。

+   打包工具通常具有基于插件的架构，并支持插件，允许导入和捆绑实际上不是 JavaScript 代码文件的“模块”。假设你的程序包含一个大型的 JSON 兼容数据结构。代码打包工具可以配置允许你将该数据结构移动到一个单独的 JSON 文件中，然后通过类似`import widgets from "./big-widget-list.json"`的声明将其导入到你的程序中。同样，将 CSS 嵌入到 JavaScript 程序中的 web 开发人员可以使用打包工具插件，允许他们使用`import`指令导入 CSS 文件。但是请注意，如果导入的不是 JavaScript 文件，你正在使用一个非标准的 JavaScript 扩展，并使你的代码依赖于打包工具。

+   在像 JavaScript 这样不需要编译的语言中，运行一个打包工具感觉像是一个编译步骤，每次在运行代码之前都必须运行一个打包工具，这让人感到沮丧。打包工具通常支持文件系统监视器，检测项目目录中任何文件的编辑，并自动重新生成必要的捆绑包。有了这个功能，你通常可以保存你的代码，然后立即重新加载你的 web 浏览器窗口以尝试它。

+   一些打包工具还支持开发人员的“热模块替换”模式，每次重新生成捆绑包时，它会自动加载到浏览器中。当这个功能起作用时，对开发人员来说是一种神奇的体验，但在幕后进行了一些技巧使其工作，它并不适用于所有项目。

# 17.6 使用 Babel 进行转译

[Babel](https://babeljs.io) 是一个工具，将使用现代语言特性编写的 JavaScript 编译成不使用这些现代语言特性的 JavaScript。由于它将 JavaScript 编译成 JavaScript，因此有时称为“转译器”。Babel 的创建是为了让 web 开发人员能够使用 ES6 及更高版本的新语言特性，同时仍针对只支持 ES5 的 web 浏览器。

诸如`**`指数运算符和箭头函数之类的语言特性可以相对容易地转换为`Math.pow()`和`function`表达式。其他语言特性，如`class`关键字，需要进行更复杂的转换，而且一般来说，Babel 输出的代码并不是为了人类可读性。然而，像打包工具一样，Babel 可以生成源映射，将转换后的代码位置映射回原始源位置，这在处理转换后的代码时非常有帮助。

浏览器供应商正在更好地跟上 JavaScript 语言的演变，今天几乎不需要编译掉箭头函数和类声明。当你想要使用最新功能，如数字文字中的下划线分隔符时，Babel 仍然可以帮助。

像本章描述的大多数其他工具一样，你可以使用 npm 安装 Babel，并使用 npx 运行它。Babel 读取一个 *.babelrc* 配置文件，告诉它如何转换你的 JavaScript 代码。Babel 定义了“预设”，你可以根据想要使用的语言扩展和你想要多么积极地转换标准语言特性来选择。Babel 的一个有趣的预设是用于通过缩小来进行代码压缩（去除注释和空格，重命名变量等）。

如果你使用 Babel 和一个代码捆绑工具，你可以设置代码捆绑器在构建捆绑包时自动运行 Babel 来处理你的 JavaScript 文件。如果是这样，这可能是一个方便的选项，因为它简化了生成可运行代码的过程。例如，Webpack 支持一个“babel-loader”模块，你可以安装并配置它在捆绑时运行 Babel 来处理每个 JavaScript 模块。

即使今天对核心 JavaScript 语言的转换需求较少，Babel 仍然常用于支持语言的非标准扩展，我们将在接下来的章节中描述其中的两个语言扩展。

# 17.7 JSX：JavaScript 中的标记表达式

JSX 是核心 JavaScript 的扩展，使用类似 HTML 的语法来定义元素树。JSX 与 React 框架最为密切相关，用于 Web 上的用户界面。在 React 中，使用 JSX 定义的元素树最终会被渲染成 HTML 在 Web 浏览器中。即使你没有计划自己使用 React，但由于其流行，你可能会看到使用 JSX 的代码。本节将解释你需要了解的内容以理解它。（本节关于 JSX 语言扩展，不是关于 React，仅解释 React 的部分内容以提供 JSX 语法的上下文。）

你可以将 JSX 元素视为一种新类型的 JavaScript 表达式语法。JavaScript 字符串字面量用引号界定，正则表达式字面量用斜杠界定。同样，JSX 表达式字面量用尖括号界定。这是一个非常简单的例子：

```js
let line = <hr/>;
```

如果你使用 JSX，你将需要使用 Babel（或类似工具）将 JSX 表达式编译成常规 JavaScript。转换足够简单，以至于一些开发人员选择在不使用 JSX 的情况下使用 React。Babel 将此赋值语句中的 JSX 表达式转换为简单的函数调用：

```js
let line = React.createElement("hr", null);
```

JSX 语法类似 HTML，并且像 HTML 元素一样，React 元素可以具有以下属性：

```js
let image = <img src="logo.png" alt="The JSX logo" hidden/>;
```

当一个元素有一个或多个属性时，它们成为传递给`createElement()`的第二个参数的对象的属性：

```js
let image = React.createElement("img", {
              src: "logo.png",
              alt: "The JSX logo",
              hidden: true
            });
```

像 HTML 元素一样，JSX 元素可以具有字符串和其他元素作为子元素。就像 JavaScript 的算术运算符可以用于编写任意复杂度的算术表达式一样，JSX 元素也可以任意深度地嵌套以创建元素树：

```js
let sidebar = (
  <div className="sidebar">
    <h1>Title</h1>
    <hr/>
    <p>This is the sidebar content</p>
  </div>
);
```

常规 JavaScript 函数调用表达式也可以任意深度地嵌套，这些嵌套的 JSX 表达式转换为一组嵌套的`createElement()`调用。当一个 JSX 元素有子元素时，这些子元素（通常是字符串和其他 JSX 元素）作为第三个及后续参数传递：

```js
let sidebar = React.createElement(
    "div", { className: "sidebar"},  // This outer call creates a <div>
    React.createElement("h1", null,  // This is the first child of the <div/>
                        "Title"),    // and its own first child.
    React.createElement("hr", null), // The second child of the <div/>.
    React.createElement("p", null,   // And the third child.
                        "This is the sidebar content"));
```

`React.createElement()`返回的值是 React 用于在浏览器窗口中呈现输出的普通 JavaScript 对象。由于本节是关于 JSX 语法而不是关于 React，我们不会详细介绍返回的元素对象或呈现过程。值得注意的是，你可以配置 Babel 将 JSX 元素编译为调用不同函数的调用，因此如果你认为 JSX 语法是表达其他类型嵌套数据结构的有用方式，你可以将其用于自己的非 React 用途。

JSX 语法的一个重要特点是你可以在 JSX 表达式中嵌入常规 JavaScript 表达式。在 JSX 表达式中，花括号内的文本被解释为普通 JavaScript。这些嵌套表达式允许作为属性值和子元素。例如：

```js
function sidebar(className, title, content, drawLine=true) {
  return (
    <div className={className}>
      <h1>{title}</h1>
      { drawLine && <hr/> }
      <p>{content}</p>
    </div>
  );
}
```

`sidebar()`函数返回一个 JSX 元素。它接受四个参数，这些参数在 JSX 元素中使用。花括号语法可能会让你想起使用`${}`在字符串中包含 JavaScript 表达式的模板字面量。由于我们知道 JSX 表达式编译为函数调用，因此包含任意 JavaScript 表达式并不奇怪，因为函数调用也可以用任意表达式编写。Babel 将此示例代码转换为以下内容：

```js
function sidebar(className, title, content, drawLine=true) {
  return React.createElement("div", { className: className },
                             React.createElement("h1", null, title),
                             drawLine && React.createElement("hr", null),
                             React.createElement("p", null, content));
}
```

这段代码易于阅读和理解：花括号消失了，生成的代码以自然的方式将传入的函数参数传递给`React.createElement()`。请注意我们在这里使用`drawLine`参数和短路`&&`运算符的巧妙技巧。如果你只用三个参数调用`sidebar()`，那么`drawLine`默认为`true`，并且外部`createElement()`调用的第四个参数是`<hr/>`元素。但如果将`false`作为第四个参数传递给`sidebar()`，那么外部`createElement()`调用的第四个参数将计算为`false`，并且永远不会创建`<hr/>`元素。这种使用`&&`运算符的习惯用法在 JSX 中是一种常见的习语，根据某些其他表达式的值有条件地包含或排除子元素。（这种习惯用法在 React 中有效，因为 React 简单地忽略`false`或`null`的子元素，并且不为它们生成任何输出。）

当你在 JSX 表达式中使用 JavaScript 表达式时，你不仅限于前面示例中的字符串和布尔值等简单值。任何 JavaScript 值都是允许的。事实上，在 React 编程中使用对象、数组和函数是非常常见的。例如，考虑以下函数：

```js
// Given an array of strings and a callback function return a JSX element
// representing an HTML <ul> list with an array of <li> elements as its child.
function list(items, callback) {
  return (
    <ul style={ {padding:10, border:"solid red 4px"} }>
      {items.map((item,index) => {
        <li onClick={() => callback(index)} key={index}>{item}</li>
      })}
    </ul>
  );
}
```

此函数将对象字面量用作`<ul>`元素上`style`属性的值。（请注意，这里需要双大括号。）`<ul>`元素只有一个子元素，但该子元素的值是一个数组。子数组是通过在输入数组上使用`map()`函数创建`<li>`元素数组而创建的数组。（这在 React 中有效，因为 React 库在渲染时会展平元素的子元素。具有一个数组子元素的元素与该元素的每个数组元素作为子元素相同。）最后，请注意每个嵌套的`<li>`元素都有一个`onClick`事件处理程序属性，其值是一个箭头函数。JSX 代码编译为以下纯 JavaScript 代码（我已使用 Prettier 格式化）：

```js
function list(items, callback) {
  return React.createElement(
    "ul",
    { style: { padding: 10, border: "solid red 4px" } },
    items.map((item, index) =>
      React.createElement(
        "li",
        { onClick: () => callback(index), key: index },
        item
      )
    )
  );
}
```

JSX 中对象表达式的另一个用途是使用对象扩展运算符（§6.10.4）一次指定多个属性。假设你发现自己编写了许多重复一组常见属性的 JSX 表达式。你可以通过将属性定义为对象的属性并将它们“扩展到”你的 JSX 元素中来简化表达式：

```js
let hebrew = { lang: "he", dir: "rtl" }; // Specify language and direction
let shalom = <span className="emphasis" {...hebrew}>שלום</span>;
```

Babel 将其编译为使用`_extends()`函数（此处省略）将`className`属性与`hebrew`对象中包含的属性组合在一起：

```js
let shalom = React.createElement("span",
                                 _extends({className: "emphasis"}, hebrew),
                                 "\u05E9\u05DC\u05D5\u05DD");
```

最后，还有一个 JSX 的重要特性我们还没有涉及。正如你所见，所有 JSX 元素在开角括号后立即以标识符开头。如果此标识符的第一个字母是小写（就像在这里的所有示例中一样），那么该标识符将作为字符串传递给`createElement()`。但如果标识符的第一个字母是大写，则将其视为实际标识符，并将该标识符的 JavaScript 值作为`createElement()`的第一个参数传递。这意味着 JSX 表达式`<Math/>`编译为将全局 Math 对象传递给`React.createElement()`的 JavaScript 代码。

对于 React 来说，将非字符串值作为`createElement()`的第一个参数传递的能力使得创建*组件*成为可能。组件是一种编写简单 JSX 表达式（使用大写组件名称）来表示更复杂表达式（使用小写 HTML 标签名称）的方式。

在 React 中定义一个新组件的最简单方法是编写一个以“props 对象”作为参数的函数，并返回一个 JSX 表达式。*props 对象*只是一个表示属性值的 JavaScript 对象，就像作为`createElement()`的第二个参数传递的对象一样。例如，这里是我们`sidebar()`函数的另一种写法：

```js
function Sidebar(props) {
  return (
    <div>
      <h1>{props.title}</h1>
      { props.drawLine && <hr/> }
      <p>{props.content}</p>
    </div>
  );
}
```

这个新的`Sidebar()`函数与之前的`sidebar()`函数非常相似。但这个函数以大写字母开头的名称，并接受一个对象参数而不是单独的参数。这使它成为一个 React 组件，并意味着它可以在 JSX 表达式中替代 HTML 标签名称使用：

```js
let sidebar = <Sidebar title="Something snappy" content="Something wise"/>;
```

这个`<Sidebar/>`元素编译如下：

```js
let sidebar = React.createElement(Sidebar, {
  title: "Something snappy",
  content: "Something wise"
});
```

这是一个简单的 JSX 表达式，但当 React 渲染它时，它会将第二个参数（Props 对象）传递给第一个参数（`Sidebar()`函数），并将该函数返回的 JSX 表达式替换为`<Sidebar>`表达式的位置。

# 17.8 使用 Flow 进行类型检查

[Flow](https://flow.org)是一种语言扩展，允许您在 JavaScript 代码中添加类型信息，并用于检查您的 JavaScript 代码（包括带注释和不带注释的代码）中的类型错误。要使用 Flow，您开始使用 Flow 语言扩展编写代码以添加类型注解。然后运行 Flow 工具分析您的代码并报告类型错误。一旦您修复了错误并准备运行代码，您可以使用 Babel（可能作为代码捆绑过程的一部分自动执行）来剥离代码中的 Flow 类型注解。（Flow 语言扩展的一个好处是，Flow 没有必须编译或转换的新语法。您使用 Flow 语言扩展向代码添加注解，而 Babel 只需剥离这些注解以将您的代码返回到标准 JavaScript。）

使用 Flow 需要承诺，但我发现对于中大型项目来说，额外的努力是值得的。为代码添加类型注解，每次编辑代码时运行 Flow，以及修复它报告的类型错误都需要额外的时间。但作为回报，Flow 将强制执行良好的编码纪律，并不允许你采取可能导致错误的捷径。当我在使用 Flow 的项目上工作时，我对它在我的代码中发现的错误数量感到印象深刻。在这些问题变成错误之前修复这些问题是一种很棒的感觉，并让我对我的代码正确性更有信心。

当我第一次开始使用 Flow 时，我发现有时很难理解它为什么会抱怨我的代码。然而，通过一些实践，我开始理解它的错误消息，并发现通常很容易对我的代码进行微小更改，使其更安全并满足 Flow 的要求。¹ 如果你仍然觉得自己在学习 JavaScript 本身，我不建议使用 Flow。但一旦你对这门语言有信心，将 Flow 添加到你的 JavaScript 项目中将推动你将编程技能提升到下一个水平。这也是为什么我将这本书的最后一节专门用于 Flow 教程的原因：因为了解 JavaScript 类型系统提供了另一种编程水平或风格的一瞥。

本节是一个教程，不打算全面涵盖 Flow。如果您决定尝试 Flow，几乎肯定会花时间阅读[*https://flow.org*](https://flow.org)上的文档。另一方面，您不需要在掌握 Flow 类型系统之前就能开始在项目中实际使用它：这里描述的 Flow 的简单用法将带您走很远。

## 17.8.1 安装和运行 Flow

像本章中描述的其他工具一样，您可以使用包管理器安装 Flow 类型检查工具，使用类似`npm install -g flow-bin`或`npm install --save-dev flow-bin`的命令。如果使用`-g`全局安装工具，那么可以使用`flow`运行它。如果在项目中使用`--save-dev`本地安装它，那么可以使用`npx flow`运行它。在使用 Flow 进行类型检查之前，首次在项目的根目录中运行`flow --init`以创建`.flowconfig`配置文件。您可能永远不需要向此文件添加任何内容，但 Flow 需要知道您的项目根目录在哪里。

运行 Flow 时，它会找到项目中的所有 JavaScript 源代码，但只会为已通过在文件顶部添加`// @flow`注释而“选择加入”类型检查的文件报告类型错误。这种选择加入的行为很重要，因为这意味着您可以为现有项目采用 Flow，然后逐个文件地开始转换代码，而不会受到尚未转换的文件上的错误和警告的困扰。

即使您只是通过`// @flow`注释选择加入，Flow 也可能能够找到代码中的错误。即使您不使用 Flow 语言扩展并且不向代码添加任何类型注释，Flow 类型检查工具仍然可以推断程序中的值，并在您不一致地使用它们时提醒您。

考虑以下 Flow 错误消息：

```js
Error ┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈ variableReassignment.js:6:3

Cannot assign 1 to i.r because:
 • property r is missing in number [1].

     2│ let i = { r: 0, i: 1 };    // The complex number 0+1i
 [1] 3│ for(i = 0; i < 10; i++) {  // Oops! The loop variable overwrites i
     4│     console.log(i);
     5│ }
     6│ i.r = 1;                   // Flow detects the error here
```

在这种情况下，我们声明变量`i`并将一个对象赋给它。然后我们再次使用`i`作为循环变量，覆盖了对象。Flow 注意到这一点，并在我们尝试像仍然保存对象一样使用`i`时标记错误。（一个简单的修复方法是写`for(let i = 0;`使循环变量局部于循环。）

这是 Flow 即使没有类型注释也能检测到的另一个错误：

```js
Error ┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈ size.js:3:14

Cannot get x.length because property length is missing in Number [1].

     1│ // @flow
     2│ function size(x) {
     3│     return x.length;
     4│ }
 [1] 5│ let s = size(1000);
```

Flow 看到`size()`函数接受一个参数。它不知道该参数的类型，但可以看到该参数应具有`length`属性。当看到使用数字参数调用此`size()`函数时，它会正确地标记此为错误，因为数字没有`length`属性。

## 17.8.2 使用类型注释

当声明 JavaScript 变量时，可以在变量名称后面加上冒号和类型来添加 Flow 类型注释：

```js
let message: string = "Hello world";
let flag: boolean = false;
let n: number = 42;
```

即使您没有为这些变量添加注释，Flow 也会知道这些变量的类型：它可以看到您为每个变量分配的值，并跟踪它们。但是，如果添加了类型注释，Flow 既知道变量的类型，又知道您已表达了该变量应始终为该类型的意图。因此，如果使用类型注释，如果将不同类型的值分配给该变量，Flow 将标记错误。对于变量，类型注释也特别有用，如果您倾向于在函数使用之前在函数顶部声明所有变量。

函数参数的类型注释与变量的注释类似：在函数参数名称后面跟着冒号和类型名称。在注释函数时，通常还会为函数的返回类型添加注释。这在函数体的右括号和左花括号之间。返回空值的函数使用 Flow 类型`void`。

在前面的示例中，我们定义了一个期望具有`length`属性的参数的`size()`函数。下面是如何将该函数更改为明确指定它期望一个字符串参数并返回一个数字。请注意，即使在这种情况下函数可以正常工作，Flow 现在也会标记错误，如果我们将数组传递给函数：

```js
Error ┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈ size2.js:5:18

Cannot call size with array literal bound to s because array literal [1]
is incompatible with string [2].

 [2] 2│ function size(s: string): number {
     3│     return s.length;
     4│ }
 [1] 5│ console.log(size([1,2,3]));
```

使用箭头函数的类型注解也是可能的，尽管这可能会将这个通常简洁的语法变得更冗长：

```js
const size = (s: string): number => s.length;
```

关于 Flow 的一个重要事项是，JavaScript 值`null`具有 Flow 类型`null`，JavaScript 值`undefined`具有 Flow 类型`void`。但这两个值都不是任何其他类型的成员（除非你明确添加它）。如果你声明一个函数参数为字符串，那么它必须是一个字符串，传递`null`、传递`undefined`或省略参数（基本上与传递`undefined`相同）都是错误的：

```js
Error ┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈ size3.js:3:18

Cannot call size with null bound to s because null [1] is incompatible
with string [2].

     1│ // @flow
 [2] 2│ const size = (s: string): number => s.length;
 [1] 3│ console.log(size(null));
```

如果你想允许`null`和`undefined`作为变量或函数参数的合法值，只需在类型前加上问号。例如，使用`?string`或`?number`代替`string`或`number`。如果我们将`size()`函数更改为期望类型为`?string`的参数，那么当我们将`null`传递给函数时，Flow 不会抱怨。但现在它有其他事情要抱怨：

```js
Error ┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈ size4.js:3:14

Cannot get s.length because property length is missing in null or
undefined [1].

     1│ // @flow
 [1] 2│ function size(s: ?string): number {
     3│     return s.length;
     4│ }
     5│ console.log(size(null));
```

Flow 在这里告诉我们的是，在我们的代码中，写`s.length`是不安全的，因为此处的`s`可能是`null`或`undefined`，而这些值没有`length`属性。这就是 Flow 确保我们不会偷懒的地方。如果一个值可能是`null`，Flow 会坚持要求我们在执行任何依赖于该值不是`null`的操作之前检查该情况。

在这种情况下，我们可以通过更改函数主体来解决问题如下：

```js
function size(s: ?string): number {
    // At this point in the code, s could be a string or null or undefined.
    if (s === null || s === undefined) {
        // In this block, Flow knows that s is null or undefined.
        return -1;
    } else {
        // And in this block, Flow knows that s is a string.
        return s.length;
    }
}
```

当函数首次调用时，参数可以有多种类型。但通过添加类型检查代码，我们在代码中创建了一个块，Flow 可以确定参数是一个字符串。当我们在该块内使用`s.length`时，Flow 不会抱怨。请注意，Flow 不要求你编写冗长的代码。如果我们只是用`return s ? s.length : -1;`替换`size()`函数的主体，Flow 也会满意。

Flow 语法允许在任何类型规范之前加上问号，以指示除了指定的类型外，`null`和`undefined`也是允许的。问号也可以出现在参数名后，以指示参数本身是可选的。因此，如果我们将参数`s`的声明从`s: ?string`更改为`s? : string`，那意味着可以用没有参数调用`size()`（或值为`undefined`，这与省略它相同），但如果我们用除`undefined`之外的参数调用它，那么该参数必须是一个字符串。在这种情况下，`null`不是合法值。

到目前为止，我们已经讨论了原始类型`string`、`number`、`boolean`、`null`和`void`，并演示了如何在变量声明、函数参数和函数返回值中使用它们。接下来的小节描述了 Flow 支持的一些更复杂的类型。

## 17.8.3 类型类

除了 Flow 了解的原始类型外，它还了解所有 JavaScript 的内置类，并允许你使用类名作为类型。例如，以下函数使用类型注解指示应使用一个 Date 对象和一个 RegExp 对象调用它：

```js
// @flow
// Return true if the ISO representation of the specified date
// matches the specified pattern, or false otherwise.
// E.g: const isTodayChristmas = dateMatches(new Date(), /^\d{4}-12-25T/);
export function dateMatches(d: Date, p: RegExp): boolean {
    return p.test(d.toISOString());
}
```

如果你使用`class`关键字定义自己的类，那些类会自动成为有效的 Flow 类型。然而，为了使其工作，Flow 确实要求你在类中使用类型注解。特别是，类的每个属性必须声明其类型。这里是一个简单的复数类示例，演示了这一点：

```js
// @flow
export default class Complex {
    // Flow requires an extended class syntax that includes type annotations
    // for each of the properties used by the class.
    i: number;
    r: number;
    static i: Complex;

    constructor(r: number, i:number) {
        // Any properties initialized by the constructor must have Flow type
        // annotations above.
        this.r = r;
        this.i = i;
    }

    add(that: Complex) {
        return new Complex(this.r + that.r, this.i + that.i);
    }
}

// This assignment would not be allowed by Flow if there was not a
// type annotation for i inside the class.
Complex.i = new Complex(0,1);
```

## 17.8.4 对象类型

描述对象的 Flow 类型看起来很像对象字面量，只是属性值被属性类型替换。例如，这里是一个期望具有数字 `x` 和 `y` 属性的对象的函数：

```js
// @flow
// Given an object with numeric x and y properties, return the
// distance from the origin to the point (x,y) as a number.
export default function distance(point: {x:number, y:number}): number {
    return Math.hypot(point.x, point.y);
}
```

在这段代码中，文本 `{x:number, y:number}` 是一个 Flow 类型，就像 `string` 或 `Date` 一样。与任何类型一样，你可以在前面加上问号来表示 `null` 和 `undefined` 也应该被允许。

在对象类型中，你可以在任何属性名称后面加上问号，表示该属性是可选的，可以省略。例如，你可以这样写一个表示 2D 或 3D 点的对象类型：

```js
{x: number, y: number, z?: number}
```

如果在对象类型中未标记属性为可选，则该属性是必需的，如果实际值中缺少适当的属性，Flow 将报告错误。然而，通常情况下，Flow 容忍额外的属性。如果你向上面的 `distance()` 函数传递一个具有 `w` 属性的对象，Flow 不会抱怨。

如果你希望 Flow 严格执行对象除了在其类型中明确声明的属性之外没有其他属性，你可以通过在花括号中添加竖线来声明*精确对象类型*：

```js
{| x: number, y: number |}
```

JavaScript 的对象有时被用作字典或字符串值映射。当以这种方式使用对象时，属性名称事先不知道，也不能在 Flow 类型中声明。如果你以这种方式使用对象，你仍然可以使用 Flow 来描述数据结构。假设你有一个对象，其中属性是世界主要城市的名称，这些属性的值是指定这些城市地理位置的对象。你可以这样声明这个数据结构：

```js
// @flow
const cityLocations : {[string]: {longitude:number, latitude:number}} = {
    "Seattle": { longitude: 47.6062, latitude: -122.3321 },
    // TODO: if there are any other important cities, add them here.
};
export default cityLocations;
```

## 17.8.5 类型别名

对象可以有许多属性，描述这样一个对象的 Flow 类型将会很长且难以输入。即使相对较短的对象类型也可能令人困惑，因为它们看起来非常像对象字面量。一旦我们超越了像 `number` 和 `?string` 这样的简单类型，为我们的 Flow 类型定义名称通常是有用的。事实上，Flow 使用 `type` 关键字来做到这一点。在 `type` 关键字后面跟上标识符、等号和 Flow 类型。一旦你这样做了，该标识符将成为该类型的别名。例如，这里是我们如何使用显式定义的 `Point` 类型重写上一节中的 `distance()` 函数：

```js
// @flow
export type Point = {
    x: number,
    y: number
};

// Given a Point object return its distance from the origin
export default function distance(point: Point): number {
    return Math.hypot(point.x, point.y);
}
```

请注意，此代码导出了 `distance()` 函数，并且还导出了 `Point` 类型。其他模块可以使用 `import type Point from './distance.js'` 如果他们想使用该类型定义。但请记住，`import type` 是一个 Flow 语言扩展，而不是真正的 JavaScript 导入指令。类型导入和导出被 Flow 类型检查器使用，但像所有其他 Flow 语言扩展一样，在代码运行之前它们都会被剥离。

最后，值得注意的是，与其定义一个代表点的 Flow 对象类型的名称，可能更简单和更清晰的是只定义一个 Point 类并将该类用作类型。

## 17.8.6 数组类型

描述数组的 Flow 类型是一个复合类型，还包括数组元素的类型。例如，这里是一个期望数字数组的函数，以及如果尝试使用具有非数字元素的数组调用该函数时 Flow 报告的错误：

```js
Error ┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈ average.js:8:16

Cannot call average with array literal bound to data because string [1]
is incompatible with number [2] in array element.

 [2]  2│ function average(data: Array<number>) {
      3│     let sum = 0;
      4│     for(let x of data) sum += x;
      5│     return sum/data.length;
      6│ }
      7│
 [1]  8│ average([1, 2, "three"]);
```

表示数组的 Flow 类型是 `Array`，后面跟着尖括号中的元素类型。你也可以通过在元素类型后面加上开放和关闭方括号来表示数组类型。因此，在这个例子中，我们可以写成 `number[]` 而不是 `Array<number>`。我更喜欢尖括号表示法，因为，正如我们将看到的，还有其他使用这种尖括号语法的 Flow 类型。

所示的 Array 类型语法适用于具有任意数量元素的数组，所有元素都具有相同的类型。Flow 有一种不同的语法来描述*元组*的类型：一个具有固定数量元素的数组，每个元素可能具有不同的类型。要表示元组的类型，只需写出每个元素的类型，用逗号分隔，然后将它们都括在方括号中。

例如，一个返回 HTTP 状态码和消息的函数可能如下所示：

```js
function getStatus():[number, string] {
    return [getStatusCode(), getStatusMessage()];
}
```

返回元组的函数在不使用解构赋值的情况下很难处理：

```js
let [code, message] = getStatus();
```

解构赋值，再加上 Flow 的类型别名功能，使得元组易于处理，以至于你可能会考虑它们作为简单数据类型的替代方案：

```js
// @flow
export type Color = [number, number, number, number];  // [r, g, b, opacity]

function gray(level: number): Color {
    return [level, level, level, 1];
}

function fade([r,g,b,a]: Color, factor: number): Color {
    return [r, g, b, a/factor];
}

let [r, g, b, a] = fade(gray(75), 3);
```

现在我们有了一种表达数组类型的方法，让我们回到之前的`size()`函数，并修改它以接受一个数组参数而不是一个字符串参数。我们希望函数能够接受任意长度的数组，因此元组类型不合适。但我们也不希望将函数限制为仅适用于所有元素类型相同的数组。解决方案是类型`Array<mixed>`：

```js
// @flow
function size(s: Array<mixed>): number {
    return s.length;
}
console.log(size([1,true,"three"]));
```

元素类型`mixed`表示数组的元素可以是任何类型。如果我们的函数实际上对数组进行索引并尝试使用其中的任何元素，Flow 将坚持要求我们使用`typeof`检查或其他测试来确定元素的类型，然后再执行任何不安全的操作。（如果你愿意放弃类型检查，也可以使用`any`代替`mixed`：它允许你对数组的值做任何想做的事情，而不必确保这些值是你期望的类型。）

## 17.8.7 其他参数化类型

我们已经看到，当您将一个值注释为`Array`时，Flow 要求您还必须在尖括号内指定数组元素的类型。这个额外的类型被称为*类型参数*，而 Array 并不是唯一一个被参数化的 JavaScript 类。

JavaScript 的 Set 类是一个元素集合，就像数组一样，你不能单独使用`Set`作为一种类型，而是必须在尖括号内包含一个类型参数来指定集合中包含的值的类型。（尽管如果集合可能包含多种类型的值，你可以使用`mixed`或`any`。）以下是一个示例：

```js
// @flow
// Return a set of numbers with members that are exactly twice those
// of the input set of numbers.
function double(s: Set<number>): Set<number> {
    let doubled: Set<number> = new Set();
    for(let n of s) doubled.add(n * 2);
    return doubled;
}
console.log(double(new Set([1,2,3])));  // Prints "Set {2, 4, 6}"
```

Map 是另一种参数化类型。在这种情况下，必须指定两个类型参数；键的类型和值的类型：

```js
// @flow
import type { Color } from "./Color.js";

let colorNames: Map<string, Color> = new Map([
    ["red", [1, 0, 0, 1]],
    ["green", [0, 1, 0, 1]],
    ["blue", [0, 0, 1, 1]]
]);
```

Flow 还允许您为自己的类定义类型参数。以下代码定义了一个 Result 类，但使用一个 Error 类型和一个 Value 类型对该类进行参数化。我们在代码中使用占位符`E`和`V`来表示这些类型参数。当这个类的用户声明一个 Result 类型的变量时，他们将指定实际类型来替换`E`和`V`。变量声明可能如下所示：

```js
let result: Result<TypeError, Set<string>>;
```

下面是参数化类的定义方式：

```js
// @flow
// This class represents the result of an operation that can either
// throw an error of type E or a value of type V.
export class Result<E, V> {
    error: ?E;
    value: ?V;

    constructor(error: ?E, value: ?V) {
        this.error = error;
        this.value = value;
    }

    threw(): ?E { return this.error; }
    returned(): ?V { return this.value; }

    get():V {
        if (this.error) {
            throw this.error;
        } else if (this.value === null || this.value === undefined) {
            throw new TypeError("Error and value must not both be null");
        } else {
            return this.value;
        }
    }

}
```

甚至可以为函数定义类型参数：

```js
// @flow
// Combine the elements of two arrays into an array of pairs
function zip<A,B>(a:Array<A>, b:Array<B>): Array<[?A,?B]> {
    let result:Array<[?A,?B]> = [];
    let len = Math.max(a.length, b.length);
    for(let i = 0; i < len; i++) {
        result.push([a[i], b[i]]);
    }
    return result;
}

// Create the array [[1,'a'], [2,'b'], [3,'c'], [4,undefined]]
let pairs: Array<[?number,?string]> = zip([1,2,3,4], ['a','b','c'])
```

## 17.8.8 只读类型

Flow 定义了一些特殊的参数化“实用类型”，其名称以`$`开头。这些类型中的大多数都有我们这里不打算涵盖的高级用例。但其中两个在实践中非常有用。如果你有一个对象类型 T，并想要创建该类型的只读版本，只需编写`$ReadOnly<T>`。类似地，您可以编写`$ReadOnlyArray<T>`来描述一个具有类型 T 的只读数组。

使用这些类型的原因不是因为它们可以提供任何对象或数组不能被修改的保证（如果你想要真正的只读对象，请参见 §14.2 中的 `Object.freeze()`），而是因为它可以帮助你捕捉由无意修改引起的错误。如果你编写一个接受对象或数组参数并且不改变对象的任何属性或数组的元素的函数，那么你可以用 Flow 的只读类型注释函数参数。如果你这样做，那么如果你忘记并意外修改输入值，Flow 将报告错误。以下是两个示例：

```js
// @flow
type Point = {x:number, y:number};

// This function takes a Point object but promises not to modify it
function distance(p: $ReadOnly<Point>): number {
    return Math.hypot(p.x, p.y);
}

let p: Point = {x:3, y:4};
distance(p)  // => 5

// This function takes an array of numbers that it will not modify
function average(data: $ReadOnlyArray<number>): number {
    let sum = 0;
    for(let i = 0; i < data.length; i++) sum += data[i];
    return sum/data.length;
}

let data: Array<number> = [1,2,3,4,5];
average(data) // => 3
```

## 17.8.9 函数类型

我们已经看到如何添加类型注释来指定函数参数和返回类型的类型。但是当函数的一个参数本身是一个函数时，我们需要能够指定该函数参数的类型。

要使用 Flow 表达函数的类型，需要写出每个参数的类型，用逗号分隔，将它们括在括号中，然后跟上一个箭头和函数的返回类型。

这里是一个期望传递回调函数的示例函数。请注意我们为回调函数的类型定义了一个类型别名：

```js
// @flow
// The type of the callback function used in fetchText() below
export type FetchTextCallback = (?Error, ?number, ?string) => void;

export default function fetchText(url: string, callback: FetchTextCallback) {
    let status = null;
    fetch(url)
        .then(response => {
            status = response.status;
            return response.text()
        })
        .then(body => {
            callback(null, status, body);
        })
        .catch(error => {
            callback(error, status, null);
        });
}
```

## 17.8.10 Union 类型

让我们再次回到 `size()` 函数。创建一个什么都不做，只返回数组长度的函数并没有太多意义。数组有一个完全好用的 `length` 属性。但如果 `size()` 能够接受任何类型的集合对象（数组或 Set 或 Map）并返回集合中元素的数量，那么它可能会有用。在常规的未类型化 JavaScript 中，编写一个这样的 `size()` 函数很容易。但是在 Flow 中，我们需要一种方式来表达一个允许数组、Set 和 Map 的类型，但不允许任何其他类型值。

Flow 将这种类型称为 *Union 类型*，并允许你通过简单列出所需类型并用竖线字符分隔它们来表达它们：

```js
// @flow
function size(collection: Array<mixed>|Set<mixed>|Map<mixed,mixed>): number {
    if (Array.isArray(collection)) {
        return collection.length;
    } else {
        return collection.size;
    }
}
size([1,true,"three"]) + size(new Set([true,false])) // => 5
```

Union 类型可以用“或”这个词来阅读——“一个数组或一个 Set 或一个 Map”——因此，这种 Flow 语法使用与 JavaScript 的 OR 运算符相同的竖线字符是有意的。

我们之前看到在类型前面加一个问号允许 `null` 和 `undefined` 值。现在你可以看到，`?` 前缀只是一个为类型添加 `|null|void` 后缀的快捷方式。

一般来说，当你用 Union 类型注释一个值时，Flow 不会允许你使用该值，直到你进行足够的测试以确定实际值的类型。在我们刚刚看过的 `size()` 示例中，我们需要明确检查参数是否为数组，然后再尝试访问参数的 `length` 属性。请注意，我们不必区分 Set 参数和 Map 参数，然而：这两个类都定义了 `size` 属性，因此只要参数不是数组，`else` 子句中的代码就是安全的。

## 17.8.11 枚举类型和辨别联合

Flow 允许你使用原始字面量作为只包含一个单一值的类型。如果你写 `let x:3;`，那么 Flow 将不允许你给该变量赋值除了 3 之外的任何值。定义只有一个成员的类型通常不太有用，但字面量类型的联合可能会有用。你可能可以想象出这些类型的用途，例如：

```js
type Answer = "yes" | "no";
type Digit = 0|1|2|3|4|5|6|7|8|9;
```

如果你使用由字面量组成的类型，你需要理解只有字面值是允许的：

```js
let a: Answer = "Yes".toLowerCase(); // Error: can't assign string to Answer
let d: Digit = 3+4;                  // Error: can't assign number to Digit
```

当 Flow 检查你的类型时，它实际上并不执行计算：它只检查计算的类型。Flow 知道 `toLowerCase()` 返回一个字符串，`+` 运算符在数字上返回一个数字。尽管我们知道这两个计算返回的值都在类型内，但 Flow 无法知道这一点，并在这两行上标记错误。

像`Answer`和`Digit`这样的字面类型的联合类型是*枚举类型*的一个例子。枚举类型的一个典型用例是表示扑克牌的花色：

```js
type Suit = "Clubs" | "Diamonds" | "Hearts" | "Spades";
```

更相关的例子可能是 HTTP 状态码：

```js
type HTTPStatus =
    | 200    // OK
    | 304    // Not Modified
    | 403    // Forbidden
    | 404;   // Not Found
```

新手程序员经常听到的建议之一是避免在代码中使用字面量，而是定义符号常量来代表这些值。这样做的一个实际原因是避免拼写错误的问题：如果你拼错了一个字符串字面量，比如“Diamonds”，JavaScript 可能不会抱怨，但你的代码可能无法正常工作。另一方面，如果你拼错了一个标识符，JavaScript 很可能会抛出一个你会注意到的错误。然而，在使用 Flow 时，这个建议并不总是适用。如果你用类型 Suit 注释一个变量，然后尝试将一个拼写错误的 suit 赋给它，Flow 会提醒你错误。

字面类型的另一个重要用途是创建*辨别联合体*。当你使用联合类型（由实际不同类型组成，而不是字面量）时，通常需要编写代码来区分可能的类型。在前一节中，我们编写了一个函数，它可以接受一个数组、一个 Set 或一个 Map 作为其参数，并且必须编写代码来区分数组输入和 Set 或 Map 输入。如果你想创建一个对象类型的联合体，可以通过在每个单独的对象类型中使用字面类型来使这些类型易于区分。

举个例子来说明。假设你正在 Node 中使用工作线程（§16.11），并且正在使用`postMessage()`和“message”事件在主线程和工作线程之间发送基于对象的消息。工作线程可能想要向主线程发送多种类型的消息，但我们希望编写一个描述所有可能消息的 Flow 联合类型。考虑以下代码：

```js
// @flow
// The worker sends a message of this type when it is done
// reticulating the splines we sent it.
export type ResultMessage = {
    messageType: "result",
    result: Array<ReticulatedSpline>, // Assume this type is defined elsewhere.
};

// The worker sends a message of this type if its code failed with an exception.
export type ErrorMessage = {
    messageType: "error",
    error: Error,
};

// The worker sends a message of this type to report usage statistics.
export type StatisticsMessage = {
    messageType: "stats",
    splinesReticulated: number,
    splinesPerSecond: number
};

// When we receive a message from the worker it will be a WorkerMessage.
export type WorkerMessage = ResultMessage | ErrorMessage | StatisticsMessage;

// The main thread will have an event handler function that is passed
// a WorkerMessage. But because we've carefully defined each of the
// message types to have a messageType property with a literal type,
// the event handler can easily discriminate among the possible messages:
function handleMessageFromReticulator(message: WorkerMessage) {
    if (message.messageType === "result") {
        // Only ResultMessage has a messageType property with this value
        // so Flow knows that it is safe to use message.result here.
        // And Flow will complain if you try to use any other property.
        console.log(message.result);
    } else if (message.messageType === "error") {
        // Only ErrorMessage has a messageType property with value "error"
        // so knows that it is safe to use message.error here.
        throw message.error;
    } else if (message.messageType === "stats") {
        // Only StatisticsMessage has a messageType property with value "stats"
        // so knows that it is safe to use message.splinesPerSecond here.
        console.info(message.splinesPerSecond);
    }
}
```

# 17.9 总结

JavaScript 是当今世界上使用最广泛的编程语言。它是一种活跃的语言，不断发展和改进，周围有着繁荣的库、工具和扩展生态系统。本章介绍了其中一些工具和扩展，但还有许多其他内容需要了解。JavaScript 生态系统蓬勃发展，因为 JavaScript 开发者社区活跃而充满活力，同行们通过博客文章、视频和会议演讲分享他们的知识。当你结束阅读这本书，加入这个社区时，你会发现有很多信息源可以让你与 JavaScript 保持联系并继续学习。

祝一切顺利，David Flanagan，2020 年 3 月

¹ 如果你有 Java 编程经验，可能在第一次编写使用类型参数的通用 API 时会遇到类似的情况。我发现学习 Flow 的过程与 2004 年 Java 添加泛型时经历的过程非常相似。
