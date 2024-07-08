# 第十章：React 测试

为了跟上我们的竞争对手，我们必须在确保质量的同时快速前进。一个至关重要的工具是*单元测试*。单元测试使我们能够验证我们应用程序的每个部分或单元是否按预期功能。^(1)

实践函数式技术的一个好处是它们适合编写可测试的代码。纯函数天然适合测试。不可变性易于测试。将应用程序组合成设计用于特定任务的小函数可以生成可测试的函数或代码单元。

在这一节中，我们将演示可以用来单元测试 React 应用程序的技术。这一章不仅涵盖了测试，还包括可以用来帮助评估和改进代码及测试的工具。

# ESLint

在大多数编程语言中，在运行任何东西之前，代码需要编译。编程语言对编码风格有严格的规则，并且在代码格式化合适之前不会编译。JavaScript 没有这些规则，也没有编译器。我们编写代码，跨越手指，然后在浏览器中运行它，看它是否工作。好消息是，有工具可以用来分析我们的代码，并使我们坚持特定的格式指南。

分析 JavaScript 代码的过程称为*hinting*或*linting*。JSHint 和 JSLint 是最初用于分析 JavaScript 并提供关于格式的反馈的工具。[ESLint](http://eslint.org) 是支持新兴 JavaScript 语法的最新代码检查工具。此外，ESLint 是可插拔的。这意味着我们可以创建并分享可以添加到 ESLint 配置中以扩展其功能的插件。

在 Create React App 中，ESLint 受到原生支持，我们已经看到 lint 警告和错误出现在控制台中。

我们将使用一个名为 [`eslint-plugin-react`](https://oreil.ly/3yeXO) 的插件。此插件将分析我们的 JSX 和 React 语法，以及我们的 JavaScript。

让我们将 `eslint` 安装为开发依赖项。我们可以通过 npm 安装 `eslint`：

```
npm install eslint --save-dev

# or

yarn add eslint --dev
```

在使用 ESLint 之前，我们需要定义一些配置规则，以便我们同意遵循这些规则。我们将这些规则定义在项目根目录下的配置文件中。这个文件可以格式化为 JSON 或 YAML。[YAML](http://yaml.org) 是一种数据序列化格式，类似于 JSON，但语法更简单，对人类更友好。

ESLint 包含一个工具，帮助我们设置配置。有几家公司已经创建了 ESLint 配置文件，我们可以用作起点，或者我们可以创建自己的配置文件。

我们可以通过运行 `eslint --init` 并回答关于我们编码风格的一些问题来创建一个 ESLint 配置：

```
npx eslint --init

How would you like to configure ESLint?
To check syntax and find problems

What type of modules does your project use?
JavaScript modules (import/export)

Which framework does your project use?
React

Does your project use TypeScript?
N

Where does your code run? (Press space to select, a to toggle all,
i to invert selection)
Browser

What format do you want your config file to be in?
JSON

Would you like to install them now with npm?
Y
```

运行 `npx eslint --init` 后，会发生三件事情：

1.  `eslint-plugin-react` 被安装在本地的 *./node_modules* 文件夹中。

1.  这些依赖项会自动添加到 *package.json* 文件中。

1.  一个配置文件 *.eslintrc.json* 被创建并添加到我们项目的根目录中。

如果我们打开 *.eslintrc.json*，我们会看到一个设置对象：

```
{
  "env": {
    "browser": true,
    "es6": true
  },
  "extends": [
    "eslint:recommended",
    "plugin:react/recommended"
  ],
  "globals": {
    "Atomics": "readonly",
    "SharedArrayBuffer": "readonly"
  },
  "parserOptions": {
    "ecmaFeatures": {
      "jsx": true
    },
    "ecmaVersion": 2018,
    "sourceType": "module"
  },
  "plugins": ["react"],
  "rules": {}
}
```

重要的是，如果查看 `extends` 键，我们会看到我们的 `--init` 命令初始化了 `eslint` 和 `react` 的默认配置。这意味着我们不必手动配置所有规则，而是提供给我们这些规则。

让我们通过创建一个 *sample.js* 文件来测试我们的 ESLint 配置和这些规则：

```
const gnar = "gnarly";

const info = ({
  file = __filename,
  dir = __dirname
}) => (
  <p>
    {dir}: {file}
  </p>
);

switch (gnar) {
  default:
    console.log("gnarly");
    break;
}
```

这个文件有一些问题，但不会导致浏览器出错。从技术上讲，这段代码完全可以运行。让我们在这个文件上运行 ESLint，看看基于我们自定义规则得到的反馈：

```
npx eslint sample.js

3:7 error 'info' is assigned a value but never used no-unused-vars
4:3 error 'file' is missing in props validation react/prop-types
4:10 error 'filename' is not defined no-undef
5:3 error 'dir' is missing in props validation react/prop-types
5:9 error 'dirname' is not defined no-undef
7:3 error 'React' must be in scope when using JSX react/react-in-jsx-scope

✖ 6 problems (6 errors, 0 warnings)
```

ESLint 已经对我们的代码进行了静态分析，并根据我们的配置选择报告了一些问题。关于属性验证有错误，ESLint 还抱怨 `__filename` 和 `__dirname`，因为它不会自动包含 Node.js 全局对象。最后，ESLint 的默认 React 警告告诉我们，在使用 JSX 时，必须在作用域中包含 React。

命令 `eslint .` 将会检查我们整个目录。为此，我们很可能需要让 ESLint 忽略一些 JavaScript 文件。*.eslintignore* 文件就是我们可以添加文件或目录让 ESLint 忽略的地方：

```
dist/assets/
sample.js
```

这个 *.eslintignore* 文件告诉 ESLint 忽略我们的新 *sample.js* 文件，以及 *dist/assets* 文件夹中的任何内容。如果不忽略 *assets* 文件夹，ESLint 将分析客户端 *bundle.js* 文件，并且很可能在该文件中找到许多问题。

让我们在 *package.json* 文件中为运行 ESLint 添加一个脚本：

```
{
  "scripts": {
    "lint": "eslint ."
  }
}
```

现在，我们可以随时使用 `npm run lint` 运行 ESLint，并且它将分析我们项目中除了被忽略的文件之外的所有文件。

## ESLint 插件

有很多插件可以添加到你的 ESLint 配置中，以帮助你编写代码。对于 React 项目，你肯定会想要安装 [`eslint-plugin-react-hooks`](https://reactjs.org/docs/hooks-rules.html)，这是一个用于强制执行 React Hooks 规则的插件。这个包是由 React 团队发布的，旨在帮助修复与 Hooks 使用相关的 bug。

首先要安装它：

```
npm install eslint-plugin-react-hooks --save-dev

# OR

yarn add eslint-plugin-react-hooks --dev
```

然后，打开 *.eslintrc.json* 文件，并添加以下内容：

```
{
  "plugins": [
    // ...
    "react-hooks"
  ],
  "rules": {
    "react-hooks/rules-of-hooks": "error",
    "react-hooks/exhaustive-deps": "warn"
  }
}
```

这个插件将检查以“use”开头的函数（假设是一个 hook）是否遵循了 Hooks 的规则。

一旦添加了这些内容，我们将编写一些示例代码来测试这个插件。调整 *sample.js* 中的代码。即使这段代码无法运行，我们也在测试插件是否正常工作：

```
function gnar() {
  const [nickname, setNickname] = useState(
    "dude"
  );
  return <h1>gnarly</h1>;
}
```

从这段代码中将会弹出几个错误，但最重要的是，有一个错误告诉我们，在不是组件或者 hook 的函数中尝试调用 `useState`：

```
4:35 error React Hook "useState" is called in function "gnar" that is neither
a React function component nor a custom React Hook function
react-hooks/rules-of-hooks
```

在我们学习如何使用 Hooks 的过程中，这些示例将帮助我们。

另一个有用的 ESLint 插件是 `eslint-plugin-jsx-a11y`。A11y 是一个数字符号，意味着在“a”和“y”之间有 11 个字母，用于表示可访问性。当我们考虑无障碍性时，我们构建工具、网站和技术，可以被残障人士使用。

此插件将分析你的代码，并确保它不违反任何无障碍规则。无障碍应该是我们所有人关注的一个领域，使用这个插件将促进编写无障碍 React 应用程序的良好实践。

要安装，我们将再次使用 npm 或 yarn：

```
npm install eslint-plugin-jsx-a11y

// or

yarn add eslint-plugin-jsx-a11y
```

接下来我们将在我们的配置文件中添加 *.eslintrc.json*：

```
{
  "extends": [
    // ...
    "plugin:jsx-a11y/recommended"
  ],
  "plugins": [
    // ...
    "jsx-a11y"
  ]
}
```

现在让我们来测试一下。我们将调整我们的 *sample.js* 文件，添加一个没有 alt 属性的图像标签。为了使图像通过 lint 检查，它必须具有 alt 属性，或者如果图像不影响用户对内容的理解，则为空字符串：

```
function Image() {
  return <img src="/img.png" />;
}
```

如果我们再次运行 `npm run lint` 进行 lint，我们会看到一个由 `jsx/a11y` 插件引发的新错误：

```
5:10 error img elements must have an alt prop, either with meaningful text,
or an empty string for decorative images
```

还有许多其他 ESLint 插件可以用来静态分析你的代码，你可以花费几周的时间调整你的 ESLint 配置，使其达到完美。如果你想把你的配置提升到下一个水平，可以在 [Awesome ESLint 仓库](https://github.com/dustinspecker/awesome-eslint) 中找到许多有用的资源。

# Prettier

Prettier 是一款具有意见的代码格式化工具，适用于各种项目。自发布以来，Prettier 对 Web 开发人员日常工作的影响非常显著。根据历史记录，争论语法占据了平均 JavaScript 开发人员日常工作时间的 87%，但现在 Prettier 负责代码格式化，并定义了每个项目应使用的代码语法规则。节省的时间是显著的。此外，如果你曾经在 Markdown 表格上使用过 Prettier，你会看到它快速、清晰的格式化效果，真的是一种难以置信的景象。

ESLint 曾负责许多项目的代码格式化，但现在责任已经明确划分。ESLint 处理代码质量问题，而 Prettier 负责代码格式化。

要使 Prettier 与 ESLint 协作，我们需要稍微调整项目的配置。你可以全局安装 Prettier 来开始使用：

```
sudo npm install -g prettier
```

现在你可以在任何项目中使用 Prettier。

## 按项目配置 Prettier

要向项目添加 Prettier 配置文件，你可以创建一个 *.prettierrc* 文件。该文件将描述项目的默认设置：

```
{
  "semi": true,
  "trailingComma": none,
  "singleQuote": false,
  "printWidth": 80
}
```

这些是我们首选的默认设置，但当然，选择最合理的选项。有关更多 Prettier 格式化选项，请查看 [Prettier 文档](https://prettier.io/docs/en/options.html)。

让我们用一些代码来替换当前 *sample.js* 文件中的内容，以便进行格式化：

```
console.log("Prettier Test")
```

现在让我们尝试从终端或命令提示符中运行 Prettier CLI：

```
prettier --check "sample.js"
```

Prettier 运行测试并显示以下消息：`在上述文件中找到了代码样式问题。忘记运行 Prettier？` 要从 CLI 运行它，我们可以传递 `write` 标志：

```
prettier --write "sample.js"
```

一旦我们这样做了，我们会看到一个输出，显示 Prettier 格式化文件所花费的某些毫秒数。如果我们打开文件，我们会看到根据 *.prettierrc* 文件中提供的默认值内容已经改变了。如果你觉得这个过程看起来很费力且可以加快速度，那么你是对的。让我们开始自动化吧！

首先，我们将通过安装配置工具和插件集成 ESLint 和 Prettier：

```
npm install eslint-config-prettier eslint-plugin-prettier --save-dev
```

配置 (`eslint-config-prettier`) 关闭任何可能与 Prettier 冲突的 ESLint 规则。插件 (`eslint-plugin-prettier`) 将 Prettier 规则整合到 ESLint 规则中。换句话说，当我们运行我们的 `lint` 脚本时，Prettier 也会运行。

我们将这些工具整合到 *.eslintrc.json* 中：

```
{
  "extends": [
    // ...
    "plugin:prettier/recommended"
  ],
  "plugins": [
    //,
  "prettier"],
  "rules": {
    // ...
    "prettier/prettier": "error"
  }
}
```

确保在你的代码中打破一些格式规则，以确保 Prettier 正常工作。例如，在 *sample.js* 中：

```
console.log("Prettier Test");
```

运行 lint 命令 `npm run lint` 将产生以下输出：

```
1:13 error Replace `'Prettier·Test')` with `"Prettier·Test");` prettier/prettier
```

所有的错误都被找到了。现在你可以运行 Prettier 写命令，并为一个文件进行格式化：

```
prettier --write "sample.js"
```

或者在特定文件夹中的所有 JavaScript 文件中：

```
prettier --write "src/*.js"
```

## VSCode 中的 Prettier

如果你在使用 VSCode，强烈建议在你的编辑器中设置 Prettier。配置非常快速，并且在编写代码时将为你节省大量时间。

你首先需要安装 VSCode 的 Prettier 扩展。只需按照 [此链接](https://oreil.ly/-7Zgz) 并点击安装。安装完成后，你可以使用 Mac 上的 Control + Command + P 或 PC 上的 Ctrl + Shift + P 手动格式化文件或突出显示的代码片段。为了获得更好的结果，你可以在保存时格式化你的代码。这需要在 VSCode 中添加一些设置。

要访问这些设置，请选择 Code 菜单，然后选择 Preferences，然后选择 Settings。（如果你着急的话，可以在 Mac 上使用 Command + 逗号或在 PC 上使用 Ctrl + 逗号。）然后你可以点击右上角的小纸片图标打开 VSCode 的 JSON 设置。在这里，你需要添加一些有用的键：

```
{
  "editor.formatOnSave": true
}
```

现在，当你保存任何文件时，Prettier 将根据 *.prettierrc* 的默认设置格式化它！相当不错。如果你希望强制执行格式化，即使你的项目中没有 *.prettierrc* 配置文件，你也可以在设置中搜索 Prettier 选项来设置默认值。

如果你使用不同的编辑器，Prettier 也可能支持它。有关其他代码编辑器的具体说明，请查看[文档中的编辑器集成部分](https://prettier.io/docs/en/editors.html)。

# React 应用程序的类型检查

当您处理较大的应用程序时，可能希望包含类型检查以帮助精确定位某些类型的错误。在 React 应用程序中，有三种主要的类型检查解决方案：`prop-types`库、Flow 和 TypeScript。在下一节中，我们将更详细地看看如何设置这些工具以提高代码质量。

## PropTypes

在本书的第一版中，PropTypes 是核心 React 库的一部分，并且是向应用程序添加类型检查的推荐方式。如今，由于 Flow 和 TypeScript 等其他解决方案的出现，该功能已移至自己的库中，以减小 React 的捆绑包大小。尽管如此，PropTypes 仍然是广泛使用的解决方案。

要将 PropTypes 添加到您的应用程序中，请安装`prop-types`库：

```
npm install prop-types --save-dev
```

我们将通过创建一个最小的`App`组件来测试这一点，该组件将渲染一个库的名称：

```
import React from "react";
import ReactDOM from "react-dom";

function App({ name }) {
  return (
    <div>
      <h1>{name}</h1>
    </div>
  );
}

ReactDOM.render(
  <App name="React" />,
  document.getElementById("root")
);
```

然后我们将导入`prop-types`库，并使用`App.propTypes`来定义每个属性的类型：

```
import PropTypes from "prop-types";

function App({ name }) {
  return (
    <div>
      <h1>{name}</h1>
    </div>
  );
}

App.propTypes = {
  name: PropTypes.string
};
```

`App`组件有一个名为`name`的属性，应始终是一个字符串。如果将不正确类型的值作为名称传递，则会抛出错误。例如，如果我们使用布尔值：

```
ReactDOM.render(
  <App name="React" />,
  document.getElementById("root")
);
```

我们的控制台会向我们报告一个问题：

```
Warning: Failed prop type: Invalid prop name of type boolean supplied to App,
expected string. in App
```

当为属性提供不正确类型的值时，警告只会在开发模式下出现。在生产环境中，警告和错误渲染不会出现。

当验证属性时，当然还有其他类型可用。例如，我们可以为公司是否使用技术添加一个布尔值：

```
function App({ name, using }) {
  return (
    <div>
      <h1>{name}</h1>
      <p>
        {using ? "used here" : "not used here"}
      </p>
    </div>
  );
}

App.propTypes = {
  name: PropTypes.string,
  using: PropTypes.bool
};

ReactDOM.render(
  <App name="React" using={true} />,
  document.getElementById("root")
);
```

较长的类型检查列表包括：

+   `PropTypes.array`

+   `PropTypes.object`

+   `PropTypes.bool`

+   `PropTypes.func`

+   `PropTypes.number`

+   `PropTypes.string`

+   `PropTypes.symbol`

另外，如果要确保提供了值，可以将`.isRequired`链接到任何这些选项的末尾。例如，如果必须提供字符串，则会使用：

```
App.propTypes = {
  name: PropTypes.string.isRequired
};

ReactDOM.render(
  <App />,
  document.getElementById("root")
);
```

然后，如果未为此字段提供值，将在控制台中显示以下警告：

```
index.js:1 Warning: Failed prop type: The prop name is marked as required in App,
but its value is undefined.
```

也许还有一些情况，您不在乎值是什么，只要提供了一个值就行。在这种情况下，您可以使用`any`。例如：

```
App.propTypes = {
  name: PropTypes.any.isRequired
};
```

这意味着可以提供布尔值、字符串、数字——任何东西。只要`name`不是`undefined`，类型检查就会成功。

除了基本的类型检查外，还有一些其他实用程序在许多实际情况下都非常有用。考虑一个组件，其中有两个`status`选项：`Open`和`Closed`：

```
function App({ status }) {
  return (
    <div>
      <h1>
        We're {status === "Open" ? "Open!" : "Closed!"}
      </h1>
    </div>
  );
}

ReactDOM.render(
  <App status="Open" />,
  document.getElementById("root")
);
```

状态是一个字符串，所以我们可能倾向于使用字符串检查：

```
App.propTypes = {
  status: PropTypes.string.isRequired
};
```

这样做效果很好，但如果传入的字符串值除了`Open`和`Closed`之外还有其他值，则将验证该属性。我们实际希望执行的类型检查是枚举检查。枚举类型是特定字段或属性的受限选项列表。我们将调整`propTypes`对象如下所示：

```
App.propTypes = {
  status: PropTypes.oneOf(["Open", "Closed"])
};
```

现在，如果传递给`PropTypes.oneOf`的数组之外的任何值，都会出现警告。

对于在 React 应用程序中配置 PropTypes 的所有选项，请查看[文档](https://oreil.ly/pO2Js)。

## Flow

Flow 是一个由 Facebook 开源项目使用和维护的类型检查库。它是一个通过静态类型注解来检查错误的工具。换句话说，如果您创建了一个特定类型的变量，Flow 将检查确保使用的值是正确的类型。

让我们启动一个 Create React App 项目：

```
npx create-react-app in-the-flow
```

然后我们将 Flow 添加到项目中。Create React App 不假设您想使用 Flow，因此不会随库一起发布，但是将其整合非常顺利：

```
npm install --save flow-bin
```

安装完成后，我们将添加一个 npm 脚本，在输入 `npm run flow` 时运行 Flow。在 *package.json* 中，只需将以下内容添加到 `scripts` 键：

```
{
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test",
    "eject": "react-scripts eject",
    "flow": "flow"
  }
}
```

现在运行 `flow` 命令将对我们的文件进行类型检查。但在使用之前，我们需要创建一个 *.flowconfig* 文件。为此，我们运行：

```
npm run flow init
```

这将创建一个如下所示的配置文件的框架：

```
[ignore]

[include]

[libs]

[lints]

[options]

[strict]
```

在大多数情况下，您会将此留空以使用 Flow 的默认设置。如果您想要超出基础设置的 Flow 配置，请在[文档](https://flow.org/en/docs/config/)中探索更多选项。

Flow 最酷的功能之一是您可以逐步采用 Flow。必须将整个项目添加类型检查可能会让人感到不知所措。但是使用 Flow，这不是必需的。您只需在要进行类型检查的文件顶部添加一行 `//@flow`，然后 Flow 将自动仅检查这些文件。

另一种选项是添加 VSCode 扩展程序来帮助处理代码补全和参数提示中的 Flow 语法。如果已设置了 Prettier 或 linting 工具，则这将帮助您的编辑器处理 Flow 的意外语法。您可以在[市场中找到它](https://oreil.ly/zdaPv)。

让我们打开 *index.js* 文件，为了简单起见，将所有内容保留在同一个文件中。确保在文件顶部添加 `//@flow`：

```
//@flow

import React from "react";
import ReactDOM from "react-dom";

function App(props) {
  return (
    <div>
      <h1>{props.item}</h1>
    </div>
  );
}

ReactDOM.render(
  <App item="jacket" />,
  document.getElementById("root")
);
```

现在我们将为属性定义类型：

```
type Props = {
  item: string
};

function App(props: Props) {
  //...
}
```

然后运行 Flow `npm run flow`。在某些版本的 Flow 中，您可能会看到以下警告：

```
Cannot call ReactDOM.render with root bound to container because null [1] is
incompatible with Element [2]
```

这个警告的存在是因为如果 `document.getElementById("root")` 返回 `null`，应用程序将崩溃。为了防范此类情况（并清除错误），我们可以采取以下两种方法之一。第一种方法是使用 `if` 语句检查 `root` 是否不为 `null`：

```
const root = document.getElementById("root");

if (root !== null) {
  ReactDOM.render(<App item="jacket" />, root);
}
```

另一种选项是使用 Flow 语法为根常量添加类型检查：

```
const root = document.getElementById("root");

ReactDOM.render(<App item="jacket" />, root);
```

无论哪种情况，您都将清除错误并看到您的代码没有错误！

*无错误！*

我们可以完全信任这个，但尝试打破它感觉是个好主意。让我们向应用程序传递一个不同的属性类型：

```
ReactDOM.render(<App item={3} />, root);
```

太棒了，我们搞砸了！现在我们得到一个读取如下的错误：

```
Cannot create App element because number [1] is incompatible with string [2]
in property item.
```

让我们切换回去并为数字添加另一个属性。我们还将调整组件和属性定义：

```
type Props = {
  item: string,
  cost: number
};

function App(props: Props) {
  return (
    <div>
      <h1>{props.item}</h1>
      <p>Cost: {props.cost}</p>
    </div>
  );
}

ReactDOM.render(
  <App item="jacket" cost={249} />,
  root
);
```

运行这个工作，但如果我们移除了 cost 值会怎么样？

```
ReactDOM.render(<App item="jacket" />, root);
```

我们会立即收到一个错误：

```
Cannot create App element because property cost is missing in props [1] but
exists in Props [2].
```

如果`cost`确实不是一个必需的值，我们可以在属性定义中使用问号在属性名称后面，`cost?`来将其定义为可选的：

```
type Props = {
  item: string,
  cost?: number
};
```

如果我们再次运行它，我们就不会看到错误。

这只是 Flow 提供的所有不同功能的冰山一角。要了解更多信息并了解库中的更改，请访问[文档站点](https://flow.org/en/docs/getting-started/)。

## TypeScript

TypeScript 是 React 应用程序中的另一个流行的类型检查工具。它是 JavaScript 的开源超集，这意味着它为语言添加了额外的功能。由 Microsoft 创建，TypeScript 旨在用于大型应用程序，帮助开发人员发现错误并更快地迭代项目。

TypeScript 有越来越多的支持者，因此生态系统中的工具不断改进。我们已经熟悉的一个工具是 Create React App，它有一个我们可以使用的 TypeScript 模板。让我们设置一些基本的类型检查，类似于我们之前使用 PropTypes 和 Flow 做的，以了解如何在我们自己的应用程序中开始使用它。

我们将从生成另一个 Create React App 开始，这次使用一些不同的标志：

```
npx create-react-app my-type --template typescript
```

现在让我们浏览我们的脚手架项目的功能。请注意，在`src`目录中，现在文件扩展名是*.ts*或*.tsx*。我们还会找到一个*.tsconfig.json*文件，其中包含所有我们的 TypeScript 设置。稍后再详细介绍。

此外，如果您查看*package.json*文件，会发现列出并安装了与 TypeScript 相关的新依赖项，如库本身和用于 Jest、React、ReactDOM 等的类型定义。任何以`@types/`开头的依赖项描述了库的类型定义。这意味着库中的函数和方法都有类型，因此我们不必描述库的所有类型。

###### 注意

如果您的项目不包括 TypeScript 功能，可能是使用旧版本的 Create React App。要摆脱这个问题，您可以运行`npm uninstall -g create-react-app`。

让我们尝试将我们的组件从流程课程中放入我们的项目中。只需将以下内容添加到*index.ts*文件中：

```
import React from "react";
import ReactDOM from "react-dom";

function App(props) {
  return (
    <div>
      <h1>{props.item}</h1>
    </div>
  );
}

ReactDOM.render(
  <App item="jacket" />,
  document.getElementById("root")
);
```

如果我们用`npm start`运行项目，我们应该会看到我们的第一个 TypeScript 错误。这在目前阶段是预期的：

```
Parameter 'props' implicitly has an 'any' type.
```

这意味着我们需要为这个`App`组件添加类型规则。我们将从为流程组件定义类型时一样开始为`AppProps`类型添加字符串`item`：

```
type AppProps = {
  item: string;
};

ReactDOM.render(
  <App item="jacket" />,
  document.getElementById("root")
);
```

然后我们将在组件中引用`AppProps`：

```
function App(props: AppProps) {
  return (
    <div>
      <h1>{props.item}</h1>
    </div>
  );
}
```

现在组件将无错误地渲染，如果需要，我们也可以解构`props`：

```
function App({ item }: AppProps) {
  return (
    <div>
      <h1>{item}</h1>
    </div>
  );
}
```

我们可以通过将不同类型的值作为`item`属性的值传递来打破它：

```
ReactDOM.render(
  <App item={1} />,
  document.getElementById("root")
);
```

这会立即触发一个错误：

```
Type 'number' is not assignable to type 'string'.
```

错误还告诉了我们出问题的确切行数。这在调试过程中非常有用。

TypeScript 不仅有助于属性验证，还可以使用 TypeScript 的*类型推断*来帮助我们对钩子值进行类型检查。

考虑一个`fabricColor`的状态值，初始状态为`purple`。组件可能看起来像这样：

```
type AppProps = {
  item: string;
};

function App({ item }: AppProps) {
  const [fabricColor, setFabricColor] = useState(
    "purple"
  );
  return (
    <div>
      <h1>
        {fabricColor} {item}
      </h1>
      <button
        onClick={() => setFabricColor("blue")}
      >
        Make the Jacket Blue
      </button>
    </div>
  );
}
```

注意，我们还没有向类型定义对象添加任何内容。相反，TypeScript 推断`fabricColor`的类型应与其初始状态的类型匹配。如果我们尝试用数字而不是另一个字符串颜色`blue`设置`fabricColor`，将会抛出错误：

```
<button onClick={() => setFabricColor(3)}>
```

错误看起来像这样：

```
Argument of type '3' is not assignable to parameter of type string.
```

TypeScript 为我们提供了一种相当低成本的类型检查方式。当然，你可以进一步定制，但这应该能让你开始为应用程序添加类型检查。

了解更多关于 TypeScript，请查看[官方文档](https://oreil.ly/97_Px)和 GitHub 上惊艳的[React+TypeScript Cheatsheets](https://oreil.ly/vmran)。

# 测试驱动开发

测试驱动开发（TDD）是一种实践，而不是一种技术。这并不意味着你简单地为你的应用程序编写测试。相反，它是让测试驱动开发过程的实践。为了实践 TDD，您应遵循以下步骤：

首先编写测试

这是最关键的一步。你首先在测试中声明你要构建的内容及其应该如何工作。测试的步骤包括红、绿和金。

运行测试并观察它们失败（红色）

在编写代码之前运行测试并观察它们失败。

编写使测试通过所需的最小代码（绿色）

特别专注于使每个测试通过；不要添加超出测试范围之外的任何功能。

重构代码和测试（金色）

一旦测试通过，现在是时候仔细查看您的代码和测试了。尽量以尽可能简单和美观的方式表达您的代码。^(2)

TDD 为我们提供了一种优秀的方式来处理 React 应用程序，特别是在测试 Hooks 时。在实际编写之前，想象一个 Hook 应该如何工作通常更容易。实践 TDD 将使您能够独立于 UI 构建和验证功能或应用程序的整个数据结构。

## TDD 与学习

如果您是 TDD 的新手，或者是您正在测试的语言的新手，可能会发现在编写代码之前编写测试有些挑战。这是可以预期的，可以在您熟悉之前先编写代码而不是测试。尝试分批处理：少量代码，少量测试，依此类推。一旦习惯编写测试，编写测试就会更容易。

在本章的其余部分中，我们将编写已存在代码的测试。严格来说，我们并未实践 TDD。但在下一节中，我们将假装我们的代码不存在，以便体验 TDD 工作流程的感觉。

# 整合 Jest

在我们开始编写测试之前，我们需要选择一个测试框架。你可以使用任何 JavaScript 测试框架为 React 编写测试，但是官方的 React 文档建议使用 Jest，这是一个 JavaScript 测试运行器，让你可以通过 JSDOM 访问 DOM。访问 DOM 是很重要的，因为你想要检查 React 渲染的内容，以确保你的应用程序正常工作。

## 创建 React 应用和测试

使用 Create React App 初始化的项目已经安装了 `jest` 包。我们可以创建另一个 Create React App 项目来开始，或者使用现有项目：

```
npx create-react-app testing
```

现在我们可以开始考虑使用一个小例子进行测试。我们将在 *src* 文件夹中创建两个新文件：*functions.js* 和 *functions.test.js*。请记住，Jest 已经配置并安装在 Create React App 中，所以你只需开始编写测试即可。在 *functions.test.js* 中，我们将存根化测试。换句话说，我们将写下我们认为函数应该做的事情。

我们希望我们的函数接受一个值，将其乘以二，并返回它。所以我们将在测试中模拟这个过程。`test` 函数是 Jest 提供的用于测试单个功能的函数：

*functions.test.js*

```
test("Multiplies by two", () => {
  expect();
});
```

第一个参数 `Multiplies by two` 是测试名称。第二个参数是包含应该测试的内容的函数，第三个（可选）参数指定超时。默认超时是五秒。

接下来我们要做的是存根化将数字乘以二的函数。这个函数将被称为我们的“系统正在测试的对象”（*SUT*）。在 *functions.js* 中创建函数：

```
export default function timesTwo() {...}
```

我们将它导出，以便我们可以在测试中使用 SUT。在测试文件中，我们希望导入函数，并使用 `expect` 来编写断言。在断言中，我们会说，如果我们将 4 传递给 `timesTwo` 函数，我们期望它应该返回 8：

```
import { timesTwo } from "./functions";

test("Multiplies by two", () => {
  expect(timesTwo(4)).toBe(8);
});
```

Jest 的“匹配器”由 `expect` 函数返回，并用于验证结果。为了测试函数，我们将使用 `.toBe` 匹配器。这会验证结果对象是否与发送给 `.toBe` 的参数匹配。

让我们运行测试并使用 `npm test` 或 `npm run test` 观察它们失败。Jest 将提供每个失败的具体细节，包括堆栈跟踪：

```
FAIL  src/functions.test.js
  ✕ Multiplies by two (5ms)

  ● Multiplies by two

    expect(received).toBe(expected) // Object.is equality

    Expected: 8
    Received: undefined

      2 |
      3 | test("Multiplies by two", () => {
    > 4 |   expect(timesTwo(4)).toBe(8);
        |                       ^
      5 | });
      6 |

      at Object.<anonymous> (src/functions.test.js:4:23)

Test Suites: 1 failed, 1 total
Tests:       1 failed, 1 total
Snapshots:   0 total
Time:        1.048s
Ran all test suites related to changed files.
```

花时间编写测试并运行它们以查看它们失败，这显示了我们的测试按预期工作。这种失败反馈代表了我们的待办事项列表。我们的任务是编写最小量的代码，使我们的测试通过。

现在，如果我们在 *functions.js* 文件中添加适当的功能，我们就可以让测试通过：

```
export function timesTwo(a) {
  return a * 2;
}
```

`.toBe` 匹配器帮助我们测试单个值的相等性。如果我们想测试对象或数组，我们可以使用 `.toEqual`。让我们通过测试再进行一次循环。在测试文件中，我们将测试对象数组的相等性。

我们有一个来自 Guy Fieri 在拉斯维加斯餐厅的菜单项目列表。重要的是我们建立一个他们点菜单项的对象，以便顾客可以得到他们想要的并知道他们应该支付多少。我们将首先存根化测试：

```
test("Build an order object", () => {
  expect();
});
```

然后我们会存根化我们的函数：

```
export function order(items) {
  // ...
}
```

现在我们将在测试文件中使用订单函数。我们还假设我们有一个订单的初始数据列表，我们需要进行转换：

```
import { timesTwo, order } from "./functions";

const menuItems = [
  {
    id: "1",
    name: "Tatted Up Turkey Burger",
    price: 19.5
  },
  {
    id: "2",
    name: "Lobster Lollipops",
    price: 16.5
  },
  {
    id: "3",
    name: "Motley Que Pulled Pork Sandwich",
    price: 21.5
  },
  {
    id: "4",
    name: "Trash Can Nachos",
    price: 19.5
  }
];

test("Build an order object", () => {
  expect(order(menuItems));
});
```

记住，我们将使用 `toEqual`，因为我们正在检查对象的值而不是数组。我们希望结果等于什么？嗯，我们想要创建一个看起来像这样的对象：

```
const result = {
  orderItems: menuItems,
  total: 77
};
```

所以我们只需将其添加到测试中并在断言中使用它：

```
test("Build an order object", () => {
  const result = {
    orderItems: menuItems,
    total: 77
  };
  expect(order(menuItems)).toEqual(result);
});
```

现在我们将完成 *functions.js* 文件中的函数：

```
export function order(items) {
  const total = items.reduce(
    (price, item) => price + item.price,
    0
  );
  return {
    orderItems: items,
    total
  };
}
```

当我们检查终端时，我们会发现我们的测试现在通过了！现在这可能看起来像一个微不足道的例子，但如果你正在获取数据，很可能你会测试数组和对象的形状匹配。

Jest 中另一个常用的函数是 `describe()`。如果你使用过其他测试库，你可能以前见过类似的函数。这个函数通常用于包装几个相关的测试。例如，如果我们对一些类似功能进行了几个测试，我们可以将它们包装在一个 `describe` 语句中：

```
describe("Math functions", () => {
  test("Multiplies by two", () => {
    expect(timesTwo(4)).toBe(8);
  });
  test("Adds two numbers", () => {
    expect(sum(4, 2)).toBe(6);
  });
  test("Subtracts two numbers", () => {
    expect(subtract(4, 2)).toBe(2);
  });
});
```

当你用 `describe` 语句包装测试时，测试运行器会创建一个测试块，这样在终端中的测试输出看起来更加有组织和易读：

```
Math functions
    ✓ Multiplies by two
    ✓ Adds two numbers
    ✓ Subtracts two numbers (1ms)
```

随着你写更多的测试，将它们分组在 `describe` 块中可能是一个有用的增强。

这个过程代表了典型的 TDD 循环。我们先写测试，然后写代码使测试通过。一旦测试通过，我们可以仔细查看代码，看看是否有任何值得重构以提高清晰度或性能的地方。在处理 JavaScript（或者其他任何语言）时，这种方法非常有效。

# 测试 React 组件

现在我们对编写测试背后的过程有了基本的理解，我们可以开始将这些技术应用于 React 组件测试。

React 组件提供了创建和管理 DOM 更新时，React 需要遵循的指令。我们可以通过渲染它们并检查结果的 DOM 来测试这些组件。

我们不是在浏览器中运行我们的测试；我们是在使用 Node.js 在终端中运行它们。Node.js 没有标准浏览器自带的 DOM API。Jest 包含了一个名为 `jsdom` 的 npm 包，用于在 Node.js 中模拟浏览器环境，这对于测试 React 组件是至关重要的。

对于每个组件测试，我们可能需要将我们的 React 组件树渲染到一个 DOM 元素中。为了演示这个工作流程，让我们重新审视我们的 *Star* 组件在 *Star.js* 中：

```
import { FaStar } from "react-icons/fa";

export default function Star({ selected = false }) {
  return (
    <FaStar color={selected ? "red" : "grey"} id="star" />
  );
}
```

然后在 *index.js* 中，我们将导入并渲染这个 star：

```
import Star from "./Star";

ReactDOM.render(
  <Star />,
  document.getElementById("root")
);
```

现在让我们来编写我们的测试。我们已经编写了星星的代码，所以这里我们不会参与 TDD。如果你必须将测试整合到现有的应用程序中，这是你会这样做的方式。在一个名为 *Star.test.js* 的新文件中，首先导入 React、ReactDOM 和 `Star`：

```
import React from "react";
import ReactDOM from "react-dom";
import Star from "./Star";

test("renders a star", () => {
  const div = document.createElement("div");
  ReactDOM.render(<Star />, div);
});
```

我们还要编写测试。记住，我们供给 `test` 的第一个参数是测试的名称。然后我们将通过创建一个我们可以将星星渲染到其中的 div 来执行一些设置，使用 `ReactDOM.render`。创建元素后，我们可以编写断言：

```
test("renders a star", () => {
  const div = document.createElement("div");
  ReactDOM.render(<Star />, div);
  expect(div.querySelector("svg")).toBeTruthy();
});
```

我们期望，如果我们试图选择创建的 `div` 内部的 `svg` 元素，结果将是真的。当我们运行测试时，我们应该看到测试通过了。为了验证我们在不应该得到有效断言时是否不会得到有效断言，我们可以将选择器更改为找到假的东西，观察测试失败：

```
expect(
  div.querySelector("notrealthing")
).toBeTruthy();
```

[文档](https://oreil.ly/ah7ZU)提供了关于所有可用自定义匹配器的更多详细信息，以便你可以精确测试你想要测试的内容。

当你生成你的 React 项目时，你可能已经注意到除了像 React 和 ReactDOM 这样的基础包之外，还安装了一些来自 `@testing-library` 的包。React Testing Library 是由 Kent C. Dodds 发起的项目，旨在推广良好的测试实践，并扩展了 React 生态系统中的测试工具。Testing Library 是一个涵盖了许多库的测试包的大伞——不仅仅是针对 React。

选择 React Testing Library 的一个潜在原因是在测试失败时获得更好的错误消息。当我们测试断言时看到的当前错误：

```
expect(
  div.querySelector("notrealthing")
).toBeTruthy();
```

是：

```
expect(received).toBeTruthy()

Received: null
```

[添加 React Testing Library](https://oreil.ly/ah7ZU)可以让我们的 Create React App 项目更加强大。我们已经安装好了。首先，我们需要从 `@testing-library/jest-dom` 中导入 `toHaveAttribute` 函数：

```
import { toHaveAttribute } from "@testing-library/jest-dom";
```

接下来，我们要扩展 `expect` 的功能，以包含这个函数：

```
expect.extend({ toHaveAttribute });
```

现在我们不再使用 `toBeTruthy`，这将给我们提供难以阅读的消息，我们可以使用 `toHaveAttribute`：

```
test("renders a star", () => {
  const div = document.createElement("div");
  ReactDOM.render(<Star />, div);
  expect(
    div.querySelector("svg")
  ).toHaveAttribute("id", "hotdog");
});
```

现在当我们运行测试时，我们会看到一个错误告诉我们具体是什么问题：

```
    expect(element).toHaveAttribute("id", "hotdog")
    // element.getAttribute("id") === "hotdog"

    Expected the element to have attribute:
      id="hotdog"
    Received:
      id="star"
```

现在应该很容易修复这个问题了：

```
expect(div.querySelector("svg")).toHaveAttribute(
  "id",
  "star"
);
```

使用多个自定义匹配器意味着你需要导入、扩展和使用：

```
import {
  toHaveAttribute,
  toHaveClass
} from "@testing-library/jest-dom";

expect.extend({ toHaveAttribute, toHaveClass });

expect(you).toHaveClass("evenALittle");
```

不过，还有一种更快的方法。如果你发现自己导入了太多这些匹配器以至于无法列出或跟踪它们，你可以导入 `extend-expect` 库：

```
import "@testing-library/jest-dom/extend-expect";

// Remove this --> expect.extend({ toHaveAttribute, toHaveClass });
```

断言将继续按预期运行（有点双关意味）。关于 Create React App 的另一个有趣事实是，在随 CRA 一起提供的 *setupTests.js* 文件中，已经包含了 `extend-expect` 辅助函数的一行。如果你查看 *src* 文件夹，你会看到 *setupTests.js* 包含：

```
// jest-dom adds custom jest matchers for asserting on DOM nodes.
// allows you to do things like:
// expect(element).toHaveTextContent(/react/i)
// learn more: https://github.com/testing-library/jest-dom
import "@testing-library/jest-dom/extend-expect";
```

所以如果你正在使用 Create React App，你甚至不必在测试文件中包含导入。

## 查询

查询是 React 测试库的另一个特性，允许您根据特定条件进行匹配。为了演示如何使用查询，让我们调整`Star`组件以包含一个标题。这将允许我们编写一种常见的测试样式——基于文本匹配的测试：

```
export default function Star({ selected = false }) {
  return (
    <>
      <h1>Great Star</h1>
      <FaStar
        id="star"
        color={selected ? "red" : "grey"}
      />
    </>
  );
}
```

让我们暂停一下思考我们要测试的内容。我们希望组件被渲染，现在我们想要测试看看`h1`是否包含正确的文本。React 测试库的一部分功能`render`将帮助我们做到这一点。`render`将替换我们使用`ReactDOM.render()`的需要，因此测试会有所不同。首先从 React Testing Library 中导入`render`：

```
import { render } from "@testing-library/react";
```

`render`将接受一个参数：我们要渲染的组件或元素。该函数返回一个查询对象，可用于检查该组件或元素中的值。我们将使用的查询是`getByText`，它将为查询找到第一个匹配的节点，并在没有匹配元素时抛出错误。要返回所有匹配节点的列表，请使用`getAllBy`返回一个数组：

```
test("renders an h1", () => {
  const { getByText } = render(<Star />);
  const h1 = getByText(/Great Star/);
  expect(h1).toHaveTextContent("Great Star");
});
```

`getByText`通过传递给它的正则表达式找到`h1`元素。然后我们使用 Jest 的匹配器`toHaveTextContent`来描述`h1`应包含的文本。

运行测试，它们会通过。如果我们更改传递给`toHaveTextContent()`函数的文本，测试将失败。

## 测试事件

编写测试的另一个重要部分是测试组件的事件。让我们使用和测试我们在第七章中创建的`Checkbox`组件：

```
export function Checkbox() {
  const [checked, setChecked] = useReducer(
    checked => !checked,
    false
  );

  return (
    <>
      <label>
        {checked ? "checked" : "not checked"}
        <input
          type="checkbox"
          value={checked}
          onChange={setChecked}
        />
      </label>
    </>
  );
}
```

这个组件使用`useReducer`来切换复选框。我们的目标是创建一个自动化测试，点击这个复选框并将`checked`的值从默认的`false`更改为`true`。编写一个检查复选框的测试也会触发`useReducer`并测试这个钩子。

让我们来桩测试：

```
import React from "react";

test("Selecting the checkbox should change the value of checked to true", () => {
  // .. write a test
});
```

我们需要做的第一件事是选择我们想要在其上触发事件的元素。换句话说，我们想在自动化测试中点击哪个元素？我们将使用 Testing Library 的查询之一来找到我们要找的元素。由于输入具有标签，我们可以使用`getByLabelText()`：

```
import { render } from "@testing-library/react";
import { Checkbox } from "./Checkbox";

test("Selecting the checkbox should change the value of checked to true", () => {
  const { getByLabelText } = render(<Checkbox />);
});
```

当组件首次渲染时，其标签文本为`not checked`，因此我们可以通过正则表达式搜索匹配该字符串：

```
test("Selecting the checkbox should change the value of checked to true", () => {
  const { getByLabelText } = render(<Checkbox />);
  const checkbox = getByLabelText(/not checked/);
});
```

目前，这个正则表达式是区分大小写的，所以如果您想搜索任何大小写，可以在末尾添加一个`i`。根据您希望的查询选择的宽松程度，谨慎使用这种技术：

```
const checkbox = getByLabelText(/not checked/i);
```

现在我们已经选择了我们的复选框。现在我们只需要触发事件（点击复选框）并编写一个断言，确保在点击复选框时`checked`属性设置为`true`：

```
mport { render, fireEvent } from "@testing-library/react"

test("Selecting the checkbox should change the value of checked to true", () => {
  const { getByLabelText } = render(<Checkbox />);
  const checkbox = getByLabelText(/not checked/i);
  fireEvent.click(checkbox);
  expect(checkbox.checked).toEqual(true);
});
```

您还可以通过再次触发事件并检查属性是否在切换时设置为`false`来为此复选框测试添加反向切换。我们改变了测试的名称以更准确地描述它：

```
test("Selecting the checkbox should toggle its value", () => {
  const { getByLabelText } = render(<Checkbox />);
  const checkbox = getByLabelText(/not checked/i);
  fireEvent.click(checkbox);
  expect(checkbox.checked).toEqual(true);
  fireEvent.click(checkbox);
  expect(checkbox.checked).toEqual(false);
});
```

在这种情况下，选择复选框非常容易。我们有一个标签可以用来找到我们想要检查的输入。如果您无法轻松访问 DOM 元素，Testing Library 提供了另一个实用工具，可用于检查任何 DOM 元素。您将首先向要选择的元素添加一个属性：

```
<input
  type="checkbox"
  value={checked}
  onChange={setChecked}
  data-testid="checkbox" // Add the data-testid= attribute
/>
```

然后使用查询`getByTestId`：

```
test("Selecting the checkbox should change the value of checked to true", () => {
  const { getByTestId } = render(<Checkbox />);
  const checkbox = getByTestId("checkbox");
  fireEvent.click(checkbox);
  expect(checkbox.checked).toEqual(true);
});
```

这样做的效果相同，但在访问其他难以访问的 DOM 元素时特别有用。

一旦测试了这个`Checkbox`组件，我们就可以自信地将其整合到应用程序的其余部分并重复使用它。

## 使用代码覆盖率

*代码覆盖率*是报告实际测试了多少行代码的过程。它提供了一个度量标准，可以帮助您确定何时已编写足够的测试。

Jest 随附了 Istanbul，这是一个 JavaScript 工具，用于审查您的测试并生成描述已覆盖多少语句、分支、函数和行的报告。

要运行带有代码覆盖率的 Jest，只需在运行`jest`命令时添加`coverage`标志：

```
npm test -- --coverage
```

这份报告告诉您在测试过程中每个文件中的代码执行了多少，并报告所有已导入测试的文件。

Jest 还生成了一个报告，您可以在浏览器中运行，该报告提供了有关测试覆盖了哪些代码的更多详细信息。运行 Jest 并生成覆盖率报告后，您会注意到根目录下添加了一个*coverage*文件夹。在 Web 浏览器中打开此文件：*/coverage/lcov-report/index.html*。它将以交互式报告显示您的代码覆盖率。

这份报告告诉您代码的覆盖率，以及每个子文件夹的单独覆盖情况。您可以深入查看子文件夹，了解其中的各个文件的覆盖情况。如果您选择*components/ui*文件夹，您将看到测试覆盖您用户界面组件的程度。

您可以通过单击文件名查看哪些行在单个文件中已被覆盖。

代码覆盖率是衡量测试覆盖范围的重要工具。这是帮助您了解何时已为代码编写足够单元测试的一个基准。在每个项目中都达到 100%的代码覆盖率并不常见。以 85%以上为目标是一个不错的指标。^(3)

测试通常会感觉像是一个额外的步骤，但围绕 React 测试的工具从未如此完善。即使您不对所有代码进行测试，开始考虑如何将测试实践纳入其中也能帮助您在构建可投入生产的应用程序时节省时间和金钱。

^(1) 要了解单元测试的简要介绍，请参阅马丁·福勒的文章，[“单元测试”](http://martinfowler.com/bliki/UnitTest.html)。

^(2) 欲了解更多有关这一开发模式的信息，请参阅杰夫·麦克沃特和詹姆斯·本德的[“红-绿-重构”](https://oreil.ly/Hr6Me)。

^(3) 查看马丁·福勒的文章，[“测试覆盖率”](https://oreil.ly/Hbb-D)。
