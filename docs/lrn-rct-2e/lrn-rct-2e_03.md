# 第三章：JavaScript 函数式编程

当您开始探索 React 时，您可能会注意到函数式编程这个话题经常被提及。函数式技术在 JavaScript 项目中被越来越多地使用，特别是在 React 项目中。

很可能您已经在不知不觉中编写了函数式 JavaScript 代码。如果您对数组进行了映射或减少操作，那么您已经在成为函数式 JavaScript 程序员的路上了。函数式编程技术不仅是 React 的核心，也是 React 生态系统中许多库的核心。

如果您想知道这种函数式趋势是从哪里来的，答案是 20 世纪 30 年代，随着*lambda 演算*或λ-演算的发明^(1)。函数自 17 世纪以来一直是微积分的一部分。函数可以作为参数传递给函数，或者作为函数的结果返回。更复杂的函数，称为*高阶函数*，可以操作函数并将它们用作参数或结果，或者两者兼而有之。在 20 世纪 30 年代，阿隆佐·丘奇在普林斯顿大学进行这些高阶函数的实验时发明了λ-演算。

在 20 世纪 50 年代末，约翰·麦卡锡将从λ-演算中得出的概念应用到了一种名为 Lisp 的新编程语言中。Lisp 实现了高阶函数的概念和作为*一等公民*的函数。当函数可以像变量一样被声明并作为参数传递给函数时，该函数被认为是一等公民。这些函数甚至可以从函数中返回。

在本章中，我们将介绍函数式编程的一些关键概念，并讨论如何在 JavaScript 中实现函数式技术。

# 什么是函数式编程意味着

JavaScript 支持函数式编程，因为 JavaScript 函数是一等公民。这意味着函数可以像变量一样执行相同的操作。最新的 JavaScript 语法增加了语言改进，可以增强您的函数式编程技术，包括箭头函数、Promise 和展开操作符。

在 JavaScript 中，函数可以表示应用程序中的数据。您可能已经注意到，您可以使用`var`、`let`或`const`关键字声明函数，就像可以声明字符串、数字或任何其他变量一样：

```
var log = function(message) {
  console.log(message);
};

log("In JavaScript, functions are variables");

// In JavaScript, functions are variables
```

我们可以使用箭头函数来编写相同的函数。函数式程序员编写许多小函数，箭头函数语法使这一过程变得更加容易：

```
const log = message => {
  console.log(message);
};
```

既然函数是变量，我们可以将它们添加到对象中：

```
const obj = {
  message: "They can be added to objects like variables",
  log(message) {
    console.log(message);
  }
};

obj.log(obj.message);

// They can be added to objects like variables
```

这两个语句都执行同样的操作：它们将一个函数存储在名为`log`的变量中。此外，使用`const`关键字声明了第二个函数，这将防止它被覆盖。

我们还可以在 JavaScript 中向数组添加函数：

```
const messages = [
  "They can be inserted into arrays",
  message => console.log(message),
  "like variables",
  message => console.log(message)
];

messages1; // They can be inserted into arrays
messages3; // like variables
```

函数可以像其他变量一样作为参数传递给其他函数：

```
const insideFn = logger => {
  logger("They can be sent to other functions as arguments");
};

insideFn(message => console.log(message));

// They can be sent to other functions as arguments
```

它们也可以像变量一样从其他函数中返回：

```
const createScream = function(logger) {
  return function(message) {
    logger(message.toUpperCase() + "!!!");
  };
};

const scream = createScream(message => console.log(message));

scream("functions can be returned from other functions");
scream("createScream returns a function");
scream("scream invokes that returned function");

// FUNCTIONS CAN BE RETURNED FROM OTHER FUNCTIONS!!!
// CREATESCREAM RETURNS A FUNCTION!!!
// SCREAM INVOKES THAT RETURNED FUNCTION!!!
```

最后两个例子是高阶函数：可以接受或返回其他函数的函数。我们可以用箭头描述同样的`createScream`高阶函数：

```
const createScream = logger => message => {
  logger(message.toUpperCase() + "!!!");
};
```

如果在函数声明过程中看到多个箭头使用，这意味着你在使用高阶函数。

我们可以说 JavaScript 支持函数式编程，因为它的函数是一等公民。这意味着函数是数据。它们可以像变量一样被保存、检索或在你的应用程序中流动。

# 命令式与声明式

函数式编程是更大编程范式的一部分：*声明式编程*。声明式编程是一种编程风格，其结构化应用程序优先描述*发生了什么*而不是定义*如何发生*。

为了理解声明式编程，我们将其与*命令式编程*进行对比，后者仅关注如何使用代码实现结果。让我们考虑一个常见任务：使字符串符合 URL 规范。通常，这可以通过用连字符替换字符串中的所有空格来完成，因为空格不符合 URL 规范。首先，让我们查看这个任务的命令式方法：

```
const string = "Restaurants in Hanalei";
const urlFriendly = "";

for (var i = 0; i < string.length; i++) {
  if (string[i] === " ") {
    urlFriendly += "-";
  } else {
    urlFriendly += string[i];
  }
}

console.log(urlFriendly); // "Restaurants-in-Hanalei"
```

在这个例子中，我们循环遍历字符串中的每个字符，替换空格。这个程序的结构只关心如何实现这样的任务。我们使用`for`循环和`if`语句，并用等号操作符设置值。仅仅看代码本身并不能告诉我们太多信息。命令式程序需要大量注释才能理解正在发生的事情。

现在让我们看看相同问题的声明式方法：

```
const string = "Restaurants in Hanalei";
const urlFriendly = string.replace(/ /g, "-");

console.log(urlFriendly);
```

在这里，我们使用`string.replace`和正则表达式来替换字符串中所有空格的实例为连字符。使用`string.replace`描述了应该发生的事情：字符串中的空格应该被替换。如何处理空格的细节被抽象在`replace`函数内部。在声明式程序中，语法本身描述了应该发生什么，如何发生的细节被抽象了起来。

由于代码本身描述了正在发生的事情，声明式程序易于推理。例如，阅读以下示例中的语法。它详细描述了从 API 加载成员后发生的事情：

```
const loadAndMapMembers = compose(
  combineWith(sessionStorage, "members"),
  save(sessionStorage, "members"),
  scopeMembers(window),
  logMemberInfoToConsole,
  logFieldsToConsole("name.first"),
  countMembersBy("location.state"),
  prepStatesForMapping,
  save(sessionStorage, "map"),
  renderUSMap
);

getFakeMembers(100).then(loadAndMapMembers);
```

声明式方法更易读，因此更容易推理。如何实现每个函数的细节都被抽象化了。这些小函数命名良好，并以描述数据成员如何从加载到保存并打印在地图上的方式组合在一起，这种方法不需要太多注释。基本上，声明式编程产生了更易于推理的应用程序，当一个应用程序更容易推理时，它就更容易扩展。有关声明式编程范式的更多详细信息，请参阅[声明式编程 wiki](https://oreil.ly/7MbkB)。

现在，让我们考虑构建文档对象模型（DOM）的任务，或者[DOM](https://www.w3.org/DOM)。命令式方法关注 DOM 如何构建：

```
const target = document.getElementById("target");
const wrapper = document.createElement("div");
const headline = document.createElement("h1");

wrapper.id = "welcome";
headline.innerText = "Hello World";

wrapper.appendChild(headline);
target.appendChild(wrapper);
```

该代码关注创建元素，设置元素并将它们添加到文档中。在构建 DOM 的 10,000 行代码中进行更改，添加功能或扩展将非常困难。

现在让我们看看如何使用 React 组件声明性地构建 DOM：

```
const { render } = ReactDOM;

const Welcome = () => (
  <div id="welcome">
    <h1>Hello World</h1>
  </div>
);

render(<Welcome />, document.getElementById("target"));
```

React 是声明性的。在这里，`Welcome`组件描述了应该渲染的 DOM。`render`函数使用组件中声明的指令构建 DOM，抽象出 DOM 如何被渲染的细节。我们可以清楚地看到我们想要将`Welcome`组件渲染到 ID 为`target`的元素中。

# Functional Concepts

现在您已经了解了函数式编程及其“功能性”或“声明性”含义，我们将介绍函数式编程的核心概念：不可变性、纯度、数据转换、高阶函数和递归。

## 不可变性

变异是改变的意思，所以*不可变*意味着不可改变。在功能性程序中，数据是不可变的。它永远不会改变。

如果您需要与公众分享您的出生证明，但又想删除或遮盖私人信息，您基本上有两个选择：您可以拿一支大尖笔在原始出生证明上涂抹并划掉您的私人数据，或者您可以找一台复印机。找到一台复印机，复印您的出生证明，并用大尖笔在复印件上写字会更可取。这样，您就可以有一个被遮盖的出生证明可以分享，而您的原件则完好无损。

这就是应用程序中不可变数据的工作方式。我们不改变原始数据结构，而是构建这些数据结构的更改副本并使用它们。

要理解不可变性如何工作，让我们看看什么是数据变异。考虑一个代表颜色`lawn`的对象：

```
let color_lawn = {
  title: "lawn",
  color: "#00FF00",
  rating: 0
};
```

我们可以构建一个函数来评估颜色，并使用该函数来更改`color`对象的评级：

```
function rateColor(color, rating) {
  color.rating = rating;
  return color;
}

console.log(rateColor(color_lawn, 5).rating); // 5
console.log(color_lawn.rating); // 5
```

在 JavaScript 中，函数参数是对实际数据的引用。像这样设置颜色的评分会改变或突变原始颜色对象。 （想象一下，如果你让一家企业涂黑重要细节并分享你的出生证明，他们返回的是原件，你会希望企业有常识复印你的出生证明并将原件无损返回。）我们可以重写`rateColor`函数，使其不会损害原始物品（`color`对象）：

```
const rateColor = function(color, rating) {
  return Object.assign({}, color, { rating: rating });
};

console.log(rateColor(color_lawn, 5).rating); // 5
console.log(color_lawn.rating); // 0
```

在这里，我们使用`Object.assign`来更改颜色评分。`Object.assign`就像复印机。它获取一个空对象，将颜色复制到该对象中，并在副本上覆盖评分。现在我们可以得到一个新评分的颜色对象，而无需更改原始对象。

我们可以使用箭头函数和对象扩展运算符来编写相同的函数。这个`rateColor`函数使用扩展运算符将颜色复制到一个新对象中，然后覆盖其评分：

```
const rateColor = (color, rating) => ({
  ...color,
  rating
});
```

这个`rateColor`函数的版本与之前的完全相同。它将颜色视为不可变对象，语法更少，看起来更清晰一些。请注意，我们用括号包裹返回的对象。使用箭头函数时，这是一个必需的步骤，因为箭头不能简单地指向对象的花括号。

让我们考虑一个颜色名称的数组：

```
let list = [{ title: "Rad Red" }, { title: "Lawn" }, { title: "Party Pink" }];
```

我们可以创建一个使用`Array.push`将颜色添加到该数组的函数：

```
const addColor = function(title, colors) {
  colors.push({ title: title });
  return colors;
};

console.log(addColor("Glam Green", list).length); // 4
console.log(list.length); // 4
```

然而，`Array.push`并非是一个不可变函数。这个`addColor`函数通过向其添加另一个字段来改变原始数组。为了保持`colors`数组的不可变性，我们必须改用`Array.concat`。

```
const addColor = (title, array) => array.concat({ title });

console.log(addColor("Glam Green", list).length); // 4
console.log(list.length); // 3
```

`Array.concat`用于连接数组。在这种情况下，它获取一个带有新颜色标题的新对象，并将其添加到原始数组的副本中。

您还可以使用扩展运算符来连接数组，就像它用于复制对象一样。这是先前`addColor`函数的新兴 JavaScript 等效版本：

```
const addColor = (title, list) => [...list, { title }];
```

这个函数将原始列表复制到一个新数组中，然后将包含颜色标题的新对象添加到该副本中。它是不可变的。

## 纯函数

*纯函数*是根据其参数计算值并返回的函数。纯函数至少接受一个参数并始终返回一个值或另一个函数。它们不会引起副作用，不会设置全局变量，也不会改变应用程序状态的任何内容。它们将其参数视为不可变数据。

为了理解纯函数，让我们先看一个不纯的函数：

```
const frederick = {
  name: "Frederick Douglass",
  canRead: false,
  canWrite: false
};

function selfEducate() {
  frederick.canRead = true;
  frederick.canWrite = true;
  return frederick;
}

selfEducate();
console.log(frederick);

// {name: "Frederick Douglass", canRead: true, canWrite: true}
```

`selfEducate`函数不是一个纯函数。它不接受任何参数，也不返回值或函数。它还在其范围之外更改了一个变量：`Frederick`。一旦调用`selfEducate`函数，某些关于“世界”的东西就会改变。它引起了副作用：

```
const frederick = {
  name: "Frederick Douglass",
  canRead: false,
  canWrite: false
};

const selfEducate = person => {
  person.canRead = true;
  person.canWrite = true;
  return person;
};

console.log(selfEducate(frederick));
console.log(frederick);

// {name: "Frederick Douglass", canRead: true, canWrite: true}
// {name: "Frederick Douglass", canRead: true, canWrite: true}
```

# 纯函数是可测试的

纯函数自然是*可测试*的。它们不改变其环境或“世界”的任何内容，因此不需要复杂的测试设置或拆卸。纯函数运行所需的一切都通过参数访问。测试纯函数时，您可以控制参数，从而可以估计结果。此`selfEducate`函数也是不纯的：它会引起副作用。调用此函数会改变发送到它的对象。如果我们能将发送到此函数的参数视为不可变数据，则会得到纯函数。

让我们让这个函数接受一个参数：

```
const frederick = {
  name: "Frederick Douglass",
  canRead: false,
  canWrite: false
};

const selfEducate = person => ({
  ...person,
  canRead: true,
  canWrite: true
});

console.log(selfEducate(frederick));
console.log(frederick);

// {name: "Frederick Douglass", canRead: true, canWrite: true}
// {name: "Frederick Douglass", canRead: false, canWrite: false}
```

最后，此版本的`selfEducate`是一个纯函数。它根据发送到它的参数——`person`来计算一个值。它返回一个新的`person`对象，而不是改变发送到它的参数，因此没有副作用。

现在让我们来看一个会改变 DOM 的不纯函数的例子：

```
function Header(text) {
  let h1 = document.createElement("h1");
  h1.innerText = text;
  document.body.appendChild(h1);
}

Header("Header() caused side effects");
```

`Header`函数创建一个标题——一个具有特定文本的元素，并将其添加到 DOM 中。此函数是不纯的。它不返回函数或值，并且引起副作用：更改的 DOM。

在 React 中，UI 是用纯函数表示的。在以下示例中，`Header`是一个纯函数，可以用来创建`h1`元素，就像前面的示例中一样。但是，此函数本身不会引起副作用，因为它不会改变 DOM。此函数将创建一个`h1`元素，应用程序的其他部分将使用该元素来更改 DOM：

```
const Header = props => <h1>{props.title}</h1>;
```

纯函数是函数式编程的另一个核心概念。它们将极大地简化您的生活，因为它们不会影响应用程序的状态。编写函数时，请尝试遵循以下三条规则：

1.  该函数应至少接受一个参数。

1.  该函数应返回一个值或另一个函数。

1.  该函数不应更改或改变其任何参数。

## 数据转换

如果数据是不可变的，应用程序中的任何内容如何变化？函数式编程完全是关于将数据从一种形式转换为另一种形式。我们将使用函数生成转换后的副本。这些函数使我们的代码更少命令式，从而减少了复杂性。

您不需要特殊的框架来理解如何生成基于其他数据集的数据集。JavaScript 已经内置了执行此任务所需的工具。有两个核心函数必须掌握以精通函数式 JavaScript：`Array.map`和`Array.reduce`。

在本节中，我们将看看这些和其他一些核心函数如何将数据从一种类型转换为另一种类型。

考虑这个高中数组：

```
const schools = ["Yorktown", "Washington & Liberty", "Wakefield"];
```

我们可以使用`Array.join`函数获得这些和其他一些字符串的逗号分隔列表：

```
console.log(schools.join(", "));

// "Yorktown, Washington & Liberty, Wakefield"
```

`Array.join` 是一个内置的 JavaScript 数组方法，我们可以使用它从数组中提取一个带有分隔符的字符串。原始数组仍然完好无损；`join` 只是提供了不同的视角。产生此字符串的具体细节对程序员来说是抽象的。

如果我们想创建一个函数，用于创建一个以字母“W”开头的学校的新数组，我们可以使用 `Array.filter` 方法：

```
const wSchools = schools.filter(school => school[0] === "W");

console.log(wSchools);
// ["Washington & Liberty", "Wakefield"]
```

`Array.filter` 是 JavaScript 中的内置函数，可以从源数组生成一个新数组。这个函数接受一个 *谓词* 作为其唯一参数。谓词是一个始终返回布尔值（`true` 或 `false`）的函数。`Array.filter` 对数组中的每个项目调用一次这个谓词。该项目作为参数传递给谓词，返回值用于决定是否将该项目添加到新数组中。在这种情况下，`Array.filter` 正在检查每个学校是否以“W”开头的名字。

当需要从数组中删除项目时，应该使用 `Array.filter` 而不是 `Array.pop` 或 `Array.splice`，因为 `Array.filter` 是不可变的。在下一个示例中，`cutSchool` 函数返回新的数组，过滤掉特定的学校名称：

```
const cutSchool = (cut, list) => list.filter(school => school !== cut);

console.log(cutSchool("Washington & Liberty", schools).join(", "));

// "Yorktown, Wakefield"

console.log(schools.join("\n"));

// Yorktown
// Washington & Liberty
// Wakefield
```

在这种情况下，`cutSchool` 函数用于返回一个不包含“Washington & Liberty”的新数组。然后，使用这个新数组与 `join` 函数一起创建包含剩余两个学校名称的字符串。`cutSchool` 是一个纯函数。它接受学校列表和应该删除的学校名称，然后返回一个不包含特定学校的新数组。

函数式编程中另一个关键的数组函数是 `Array.map`。`Array.map` 方法不像谓词那样接受一个谓词，而是接受一个函数作为其参数。这个函数将为数组中的每个项目调用一次，并将其返回值添加到新数组中：

```
const highSchools = schools.map(school => `${school} High School`);

console.log(highSchools.join("\n"));

// Yorktown High School
// Washington & Liberty High School
// Wakefield High School

console.log(schools.join("\n"));

// Yorktown
// Washington & Liberty
// Wakefield
```

在这种情况下，`map` 函数用于在每个学校名称后附加“High School”。`schools` 数组仍然完好无损。

在最后的例子中，我们从一个字符串数组生成了一个字符串对象数组。`map` 函数可以生成对象、值、数组、其他函数——任何 JavaScript 类型的数组。这是 `map` 函数为每个学校返回一个对象的示例：

```
const highSchools = schools.map(school => ({ name: school }));

console.log(highSchools);

// [
// { name: "Yorktown" },
// { name: "Washington & Liberty" },
// { name: "Wakefield" }
// ]
```

从包含字符串的数组生成了一个包含对象的数组。

如果需要创建一个纯函数，用于更改对象数组中的一个对象，也可以使用 `map`。在下面的例子中，我们将名为“Stratford”的学校改名为“HB Woodlawn”，而不会改变 `schools` 数组：

```
let schools = [
  { name: "Yorktown" },
  { name: "Stratford" },
  { name: "Washington & Liberty" },
  { name: "Wakefield" }
];

let updatedSchools = editName("Stratford", "HB Woodlawn", schools);

console.log(updatedSchools[1]); // { name: "HB Woodlawn" }
console.log(schools[1]); // { name: "Stratford" }
```

`schools` 数组是一个包含对象的数组。`updatedSchools` 变量调用 `editName` 函数，我们向其发送要更新的学校、新学校和 `schools` 数组。这会更改新数组，但不会编辑原始数组：

```
const editName = (oldName, name, arr) =>
  arr.map(item => {
    if (item.name === oldName) {
      return {
        ...item,
        name
      };
    } else {
      return item;
    }
  });
```

在 `editName` 中，使用 `map` 函数基于原始数组创建一个新对象数组。 `editName` 函数可以完全在一行中编写。 下面是使用简写 `if/else` 语句的相同函数示例：

```
const editName = (oldName, name, arr) =>
  arr.map(item => (item.name === oldName ? { ...item, name } : item));
```

如果需要将数组转换为对象，可以结合使用 `Array.map` 和 `Object.keys`。 `Object.keys` 是一个方法，用于从对象中返回键数组。

假设我们需要将 `schools` 对象转换为一个学校数组：

```
const schools = {
  Yorktown: 10,
  "Washington & Liberty": 2,
  Wakefield: 5
};

const schoolArray = Object.keys(schools).map(key => ({
  name: key,
  wins: schools[key]
}));

console.log(schoolArray);

// [
// {
// name: "Yorktown",
// wins: 10
// },
// {
// name: "Washington & Liberty",
// wins: 2
// },
// {
// name: "Wakefield",
// wins: 5
// }
// ]
```

在这个例子中，`Object.keys` 返回一个学校名称数组，我们可以对该数组使用 `map` 来生成一个相同长度的新数组。新对象的 `name` 将使用键设置，而 `wins` 则设置为相应的值。

到目前为止，我们已经学习了如何使用 `Array.map` 和 `Array.filter` 转换数组。 我们还学习了如何通过组合 `Object.keys` 和 `Array.map` 将数组转换为对象。 在我们的功能工具中，还需要一个工具，即将数组转换为基本类型和其他对象的能力。

`reduce` 和 `reduceRight` 函数可以用于将数组转换为任何值，包括数字、字符串、布尔值、对象，甚至是函数。

假设我们需要在一个数字数组中找到最大数。 我们需要将数组转换为一个数字，因此可以使用 `reduce`：

```
const ages = [21, 18, 42, 40, 64, 63, 34];

const maxAge = ages.reduce((max, age) => {
  console.log(`${age} > ${max} = ${age > max}`);
  if (age > max) {
    return age;
  } else {
    return max;
  }
}, 0);

console.log("maxAge", maxAge);

// 21 > 0 = true
// 18 > 21 = false
// 42 > 21 = true
// 40 > 42 = false
// 64 > 42 = true
// 63 > 64 = false
// 34 > 64 = false
// maxAge 64
```

`ages` 数组已被减少为单个值：最大年龄 `64`。 `reduce` 接受两个参数：回调函数和原始值。 在本例中，原始值为 `0`，将最初的最大值设置为 `0`。 每个数组项都会调用回调函数一次。 第一次调用此回调时，`age` 等于数组中的第一个值 `21`，`max` 等于初始值 `0`。 回调返回两个数中较大的那个，`21` 将成为下一次迭代期间的 `max` 值。 每次迭代都会将每个 `age` 与 `max` 值进行比较，并返回两者中较大的值。 最后，比较并返回数组中的最后一个数字。

如果我们从前面的函数中移除 `console.log` 语句，并使用简写 `if/else` 语句，则可以使用以下语法计算任何数字数组中的最大值：

```
const max = ages.reduce((max, value) => (value > max ? value : max), 0);
```

# `Array.reduceRight`

`Array.reduceRight` 的工作方式与 `Array.reduce` 相同；区别在于它从数组末尾开始减少，而不是从开头开始。

有时我们需要将数组转换为对象。 以下示例使用 `reduce` 将包含颜色的数组转换为哈希：

```
const colors = [
  {
    id: "xekare",
    title: "rad red",
    rating: 3
  },
  {
    id: "jbwsof",
    title: "big blue",
    rating: 2
  },
  {
    id: "prigbj",
    title: "grizzly grey",
    rating: 5
  },
  {
    id: "ryhbhsl",
    title: "banana",
    rating: 1
  }
];

const hashColors = colors.reduce((hash, { id, title, rating }) => {
  hash[id] = { title, rating };
  return hash;
}, {});

console.log(hashColors);

// {
// "xekare": {
// title:"rad red",
// rating:3
// },
// "jbwsof": {
// title:"big blue",
// rating:2
// },
// "prigbj": {
// title:"grizzly grey",
// rating:5
// },
// "ryhbhsl": {
// title:"banana",
// rating:1
// }
// }
```

在这个例子中，发送给`reduce`函数的第二个参数是一个空对象。这是我们的哈希的初始值。在每次迭代中，回调函数使用方括号表示法向哈希添加一个新的键，并将该键的值设置为数组的`id`字段。`Array.reduce`可以这样使用，将数组减少为一个单一值，即在本例中是一个对象。

我们甚至可以使用`reduce`将数组转换为完全不同的数组。考虑将一个包含多个相同值实例的数组减少为一个唯一值数组。可以使用`reduce`方法完成这个任务：

```
const colors = ["red", "red", "green", "blue", "green"];

const uniqueColors = colors.reduce(
  (unique, color) =>
    unique.indexOf(color) !== -1 ? unique : [...unique, color],
  []
);

console.log(uniqueColors);

// ["red", "green", "blue"]
```

在这个例子中，`colors`数组被减少为一个包含不同值的数组。发送给`reduce`函数的第二个参数是一个空数组。这将是`distinct`的初始值。当`distinct`数组中还没有包含特定颜色时，它将被添加进去。否则，它将被跳过，并返回当前的`distinct`数组。

`map`和`reduce`是任何函数式程序员的主要工具，JavaScript 也不例外。如果你想成为一名熟练的 JavaScript 工程师，那么你必须掌握这些函数。从一个数据集创建另一个数据集的能力是一项必备技能，对任何类型的编程范式都有用。

## 高阶函数

使用*高阶函数*对于函数式编程也是必不可少的。我们已经提到过高阶函数，甚至在本章中使用了一些。高阶函数是可以操作其他函数的函数。它们可以接受函数作为参数，或者返回函数，或者两者兼而有之。

第一类高阶函数是期望其他函数作为参数的函数。`Array.map`、`Array.filter`和`Array.reduce`都接受函数作为参数。它们都是高阶函数。

让我们看看如何实现一个高阶函数。在下面的例子中，我们创建了一个`invokeIf`回调函数，它将测试一个条件，并在条件为真时调用一个回调函数，在条件为假时调用另一个回调函数：

```
const invokeIf = (condition, fnTrue, fnFalse) =>
  condition ? fnTrue() : fnFalse();

const showWelcome = () => console.log("Welcome!!!");

const showUnauthorized = () => console.log("Unauthorized!!!");

invokeIf(true, showWelcome, showUnauthorized); // "Welcome!!!"
invokeIf(false, showWelcome, showUnauthorized); // "Unauthorized!!!"
```

`invokeIf`期望两个函数：一个用于真，一个用于假。通过将`showWelcome`和`showUnauthorized`都发送到`invokeIf`来演示这一点。当条件为真时，将调用`showWelcome`。当条件为假时，将调用`showUnauthorized`。

返回其他函数的高阶函数可以帮助我们处理 JavaScript 中的异步复杂性。它们可以帮助我们创建可以在我们方便时使用或重复使用的函数。

*柯里化*是一种涉及高阶函数使用的函数式技术。

下面是柯里化的一个例子。`userLogs` 函数保存了一些信息（用户名），并返回一个函数，当其他信息（消息）可用时可以使用和重复使用该函数。在这个例子中，所有的日志消息都将以相关的用户名作为前缀。请注意，我们在这里使用了`getFakeMembers`函数，它从第二章返回一个承诺：

```
const userLogs = userName => message =>
  console.log(`${userName} -> ${message}`);

const log = userLogs("grandpa23");

log("attempted to load 20 fake members");
getFakeMembers(20).then(
  members => log(`successfully loaded ${members.length} members`),
  error => log("encountered an error loading members")
);

// grandpa23 -> attempted to load 20 fake members
// grandpa23 -> successfully loaded 20 members

// grandpa23 -> attempted to load 20 fake members
// grandpa23 -> encountered an error loading members
```

`userLogs` 是高阶函数。`log` 函数是从 `userLogs` 生成的，每次使用 `log` 函数时，“grandpa23”都会被添加到消息前面。

## 递归

递归是一种涉及创建函数并调用自身的技术。通常，在面对需要循环的挑战时，可以使用递归函数代替。考虑从 10 开始倒数的任务。我们可以创建一个`for`循环来解决这个问题，或者我们可以使用递归函数。在这个例子中，`countdown`就是递归函数：

```
const countdown = (value, fn) => {
  fn(value);
  return value > 0 ? countdown(value - 1, fn) : value;
};

countdown(10, value => console.log(value));

// 10
// 9
// 8
// 7
// 6
// 5
// 4
// 3
// 2
// 1
// 0
```

`countdown` 函数期望一个数字和一个函数作为参数。在这个例子中，它被调用时传入了一个值为`10`和一个回调函数。当调用`countdown`时，会调用回调函数，该函数记录当前值。接下来，`countdown`检查该值是否大于`0`。如果是，则`countdown`使用递减后的值重新调用自身。最终，值将为`0`，并且`countdown`将该值返回到调用堆栈的顶部。

递归是一种特别适合异步过程的模式。当数据可用或计时器完成时，函数可以在准备好时重新调用自身。

`countdown` 函数可以修改为带有延迟的倒计时。这个修改后的`countdown`函数可以用来创建一个倒计时时钟：

```
const countdown = (value, fn, delay = 1000) => {
  fn(value);
  return value > 0
    ? setTimeout(() => countdown(value - 1, fn, delay), delay)
    : value;
};

const log = value => console.log(value);
countdown(10, log);
```

在这个例子中，我们通过最初使用数字`10`调用`countdown`一次，并在一个记录倒计时的函数中创建了一个 10 秒的倒计时。而不是立即重新调用自身，`countdown`函数等待一秒钟再重新调用自身，从而创建一个时钟。

递归是搜索数据结构的一个好技术。你可以使用递归来迭代子文件夹，直到找到只包含文件的文件夹为止。你也可以使用递归来迭代 HTML DOM，直到找到不包含任何子元素的元素为止。在下一个例子中，我们将使用递归深入迭代对象，以检索嵌套的值：

```
const dan = {
  type: "person",
  data: {
    gender: "male",
    info: {
      id: 22,
      fullname: {
        first: "Dan",
        last: "Deacon"
      }
    }
  }
};

deepPick("type", dan); // "person"
deepPick("data.info.fullname.first", dan); // "Dan"
```

`deepPick` 可以用来访问 `Dan` 的类型，在第一个对象中立即存储，或者深入到嵌套对象中查找 `Dan` 的名字。通过发送使用点符号表示法的字符串，我们可以指定在对象中查找嵌套值的位置：

```
const deepPick = (fields, object = {}) => {
  const [first, ...remaining] = fields.split(".");
  return remaining.length
    ? deepPick(remaining.join("."), object[first])
    : object[first];
};
```

`deepPick` 函数要么返回一个值，要么反复调用自身，直到最终返回一个值。首先，该函数将点符号字段字符串拆分为数组，并使用数组解构将第一个值与剩余值分开。如果有剩余值，`deepPick` 会使用略有不同的数据重新调用自身，从而使其深入挖掘一层。

此函数继续调用自身，直到字段字符串不再包含点号，这意味着没有更多的剩余字段。在此示例中，您可以看到 `deepPick` 迭代时 `first`、`remaining` 和 `object[first]` 的值如何变化：

```
deepPick("data.info.fullname.first", dan); // "Dan"

// First Iteration
// first = "data"
// remaining.join(".") = "info.fullname.first"
// object[first] = { gender: "male", {info} }

// Second Iteration
// first = "info"
// remaining.join(".") = "fullname.first"
// object[first] = {id: 22, {fullname}}

// Third Iteration
// first = "fullname"
// remaining.join("." = "first"
// object[first] = {first: "Dan", last: "Deacon" }

// Finally...
// first = "first"
// remaining.length = 0
// object[first] = "Deacon"
```

递归是一种强大的函数技术，实现起来很有趣。

## 组合

函数式程序将其逻辑分解为小的纯函数，这些函数专注于特定的任务。最终，您需要将这些较小的函数组合在一起。具体来说，您可能需要将它们组合、按顺序或并行调用，或者将它们组合成更大的函数，直到最终形成一个应用程序。

在组合方面，有许多不同的实现、模式和技术。您可能熟悉的其中一种是链式调用。在 JavaScript 中，可以使用点符号将函数链接在一起，以在前一个函数的返回值上执行操作。

字符串有一个替换方法。替换方法返回一个模板字符串，该模板字符串也将有一个替换方法。因此，我们可以使用点符号将替换方法链在一起，以转换字符串：

```
const template = "hh:mm:ss tt";
const clockTime = template
  .replace("hh", "03")
  .replace("mm", "33")
  .replace("ss", "33")
  .replace("tt", "PM");

console.log(clockTime);

// "03:33:33 PM"
```

在此示例中，模板是一个字符串。通过将替换方法链接到模板字符串的末尾，我们可以用新值替换字符串中的小时、分钟、秒和时间。模板本身保持不变，可以重复使用以创建更多时钟时间显示。

`both` 函数是一个将值传递到两个单独函数的管道函数。平民小时的输出成为 `appendAMPM` 的输入，并且我们可以将这两个函数结合成一个来改变日期：

```
const both = date => appendAMPM(civilianHours(date));
```

然而，这种语法难以理解，因此难以维护或扩展。当我们需要将值通过 20 个不同的函数时会发生什么？

更优雅的方法是创建一个高阶函数，我们可以用它来将函数组合成更大的函数：

```
const both = compose(
  civilianHours,
  appendAMPM
);

both(new Date());
```

这种方法看起来好多了。它易于扩展，因为我们可以在任何时候添加更多函数。这种方法还使得更改组合函数的顺序变得容易。

`compose` 函数是一个高阶函数。它接受函数作为参数并返回单个值：

```
const compose = (...fns) => arg =>
  fns.reduce((composed, f) => f(composed), arg);
```

`compose`接受函数作为参数并返回一个单一函数。在这个实现中，展开运算符用于将这些函数参数转换为一个名为`fns`的数组。然后返回一个函数，该函数期望一个参数`arg`。当调用此函数时，`fns`数组从我们想要通过函数发送的参数开始进行管道传输。参数成为`compose`的初始值，然后每个缩减回调的迭代返回。请注意，回调函数接受两个参数：组合的值和一个函数`f`。每个函数都使用`compose`调用，这是前一个函数输出的结果。最终，将调用最后一个函数并返回最后的结果。

这是一个简单的`compose`函数示例，旨在说明组合技术。当处理超过一个参数或处理非函数参数时，该函数变得更加复杂。

## 把一切放在一起

现在我们已经介绍了函数式编程的核心概念，让我们将这些概念付诸实践，并为我们构建一个小型 JavaScript 应用程序。

我们的挑战是构建一个滴答作响的时钟。时钟需要以民用时间显示小时、分钟、秒和上午/下午时间。每个字段必须始终有两位数，这意味着需要为类似于 1 或 2 的单个数字值应用前导零。时钟必须每秒滴答一次并更改显示。

首先，让我们回顾一下时钟的命令式解决方案：

```
// Log Clock Time every Second
setInterval(logClockTime, 1000);

function logClockTime() {
  // Get Time string as civilian time
  let time = getClockTime();

  // Clear the Console and log the time
  console.clear();
  console.log(time);
}

function getClockTime() {
  // Get the Current Time
  let date = new Date();
  let time = "";

  // Serialize clock time
  let time = {
    hours: date.getHours(),
    minutes: date.getMinutes(),
    seconds: date.getSeconds(),
    ampm: "AM"
  };

  // Convert to civilian time
  if (time.hours == 12) {
    time.ampm = "PM";
  } else if (time.hours > 12) {
    time.ampm = "PM";
    time.hours -= 12;
  }

  // Prepend a 0 on the hours to make double digits
  if (time.hours < 10) {
    time.hours = "0" + time.hours;
  }

  // prepend a 0 on the minutes to make double digits
  if (time.minutes < 10) {
    time.minutes = "0" + time.minutes;
  }

  // prepend a 0 on the seconds to make double digits
  if (time.seconds < 10) {
    time.seconds = "0" + time.seconds;
  }

  // Format the clock time as a string "hh:mm:ss tt"
  return time.hours + ":" + time.minutes + ":" + time.seconds + " " + time.ampm;
}
```

这个解决方案有效，注释帮助我们理解发生了什么。然而，这些函数很大且复杂。每个函数做了很多事情。它们难以理解，需要注释，并且难以维护。让我们看看函数式方法如何产生一个更可扩展的应用程序。

我们的目标是将应用程序逻辑分解为更小的部分：函数。每个函数将专注于一个单一任务，并将它们组合成更大的函数，以便我们可以用来创建时钟。

首先，让我们创建一些给出值并管理控制台的函数。我们需要一个给出一秒的函数，一个给出当前时间的函数，以及几个将在控制台上记录消息并清除控制台的函数。在函数式程序中，我们应该尽可能使用函数而不是值。在需要时，我们将调用函数来获取值：

```
const oneSecond = () => 1000;
const getCurrentTime = () => new Date();
const clear = () => console.clear();
const log = message => console.log(message);
```

接下来，我们需要一些用于转换数据的函数。这三个函数将用于将`Date`对象变异为一个可用于我们时钟的对象：

`serializeClockTime`

获取一个日期对象并返回一个包含小时、分钟和秒的时钟时间对象。

`civilianHours`

获取时钟时间对象并返回一个对象，其中小时转换为民用时间。例如：1300 变为 1:00。

`appendAMPM`

接收时钟时间对象并将上午或下午时间附加到该对象。

```
const serializeClockTime = date => ({
  hours: date.getHours(),
  minutes: date.getMinutes(),
  seconds: date.getSeconds()
});

const civilianHours = clockTime => ({
  ...clockTime,
  hours: clockTime.hours > 12 ? clockTime.hours - 12 : clockTime.hours
});

const appendAMPM = clockTime => ({
  ...clockTime,
  ampm: clockTime.hours >= 12 ? "PM" : "AM"
});
```

这三个函数用于在不改变原始数据的情况下转换数据。它们将其参数视为不可变对象。

接下来，我们需要几个高阶函数：

`display`

采取一个目标函数，并返回一个函数，该函数将时间发送到目标。在这个例子中，目标是`console.log`。

`formatClock`

采取一个模板字符串，并使用它根据字符串中的标准返回格式化的时钟时间。在这个例子中，模板是“hh:mm:ss tt”。从那里，`formatClock`将用小时、分钟、秒和日间时间替换占位符。

`prependZero`

采取一个对象的键作为参数，并在该对象键下存储的值前加零。如果值小于 10，它将键入一个特定字段并在值前加零。

```
const display = target => time => target(time);

const formatClock = format => time =>
  format
    .replace("hh", time.hours)
    .replace("mm", time.minutes)
    .replace("ss", time.seconds)
    .replace("tt", time.ampm);

const prependZero = key => clockTime => ({
  ...clockTime,
  key: clockTime[key] < 10 ? "0" + clockTime[key] : clockTime[key]
});
```

这些高阶函数将被调用来创建用于格式化每次滴答的时钟时间的函数。`formatClock`和`prependZero`将在一开始时调用，设置所需的模板或关键字。它们返回的内部函数将每秒调用一次，用于格式化显示时间。

现在我们已经拥有了构建滴答时钟所需的所有函数，我们需要将它们组合在一起。我们将使用在上一节定义的`compose`函数来处理组合：

`convertToCivilianTime`

一个单独的函数，接受时钟时间作为参数，并使用平民时间的两个小时将其转换为平民时间。

`doubleDigits`

一个单独的函数，接收平民时钟时间，并确保小时、分钟和秒显示为双位数字，在需要的位置前加零。

`startTicking`

通过设置一个间隔，每秒调用一个回调函数来启动时钟。回调函数由我们所有的函数组成。每秒钟，控制台将被清空，`currentTime`将被获取、转换、平民化、格式化并显示。

```
const convertToCivilianTime = clockTime =>
  compose(
    appendAMPM,
    civilianHours
  )(clockTime);

const doubleDigits = civilianTime =>
  compose(
    prependZero("hours"),
    prependZero("minutes"),
    prependZero("seconds")
  )(civilianTime);

const startTicking = () =>
  setInterval(
    compose(
      clear,
      getCurrentTime,
      serializeClockTime,
      convertToCivilianTime,
      doubleDigits,
      formatClock("hh:mm:ss tt"),
      display(log)
    ),
    oneSecond()
  );

startTicking();
```

这个声明式版本的时钟实现了与命令式版本相同的结果。然而，这种方法有许多好处。首先，这些函数都易于测试和重用。它们可以用于未来的时钟或其他数字显示器。此外，这个程序易于扩展。没有副作用。除了函数本身之外，没有全局变量。仍然可能存在 bug，但它们会更容易找到。

在本章中，我们介绍了函数式编程的原则。整本书在讨论 React 的最佳实践时，我们将继续演示许多 React 概念是基于函数技术的。在下一章，我们将带着对指导其开发的原则有了更深入的理解，正式深入 React。

^(1) Dana S. Scott, [“λ-Calculus: Then & Now”](https://oreil.ly/k0EpX).
