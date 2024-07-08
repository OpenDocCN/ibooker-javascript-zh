# 第五章：条件类型

在本章中，我们将仔细研究一种 TypeScript 独有的特性：*条件类型*。条件类型允许我们根据子类型检查来选择类型，使我们能够在类型空间中移动，并在设计接口和函数签名时获得更大的灵活性。

条件类型是一个强大的工具，允许您动态生成类型。正如在[此 GitHub 问题](https://oreil.ly/igPhB)中展示的那样，它使得 TypeScript 的类型系统变得完备，这既令人印象深刻，也有些可怕。当您手中有这么强大的功能时，很容易失去对实际需要的类型的关注，从而陷入死胡同或制作过于难以阅读的类型。在本书中，我们将彻底讨论条件类型的使用，始终重新评估我们所做的是否确实达到了我们的预期目标。

请注意，本章比其他章节要短得多。这不是因为条件类型没有多少值得说的：恰恰相反。这更多是因为我们将在随后的章节中看到条件类型的良好使用。在这里，我们希望专注于基础知识，并建立您可以在需要时使用和参考的术语。

# 5.1 管理复杂的函数签名

## 问题

您正在创建一个具有不同参数和返回类型的函数。使用函数重载来管理所有变化变得越来越复杂。

## 解决方案

使用条件类型来定义一组规则，用于参数和返回类型。

## 讨论

您创建软件，根据用户定义的输入将某些属性显示为标签。您区分`StringLabel`和`NumberLabel`以允许不同类型的过滤操作和搜索：

```
type StringLabel = {
  name: string;
};

type NumberLabel = {
  id: number;
};
```

用户输入可以是字符串或数字。`createLabel`函数将输入作为原始类型，并生成`StringLabel`或`NumberLabel`对象：

```
function createLabel(input: number | string): NumberLabel | StringLabel {
  if (typeof input === "number") {
    return { id: input };
  } else {
    return { name: input };
  }
}
```

基本功能完成后，您会发现您的类型太过广泛。如果输入一个`number`，`createLabel`的返回类型仍然是`NumberLabel | StringLabel`，但它只能是`NumberLabel`。解决方案？添加函数重载以明确定义类型关系，就像我们在第二章第六部分中学到的那样：

```
function createLabel(input: number): NumberLabel;
function createLabel(input: string): StringLabel;
function createLabel(input: number | string): NumberLabel | StringLabel {
  if (typeof input === "number") {
    return { id: input };
  } else {
    return { name: input };
  }
}
```

函数重载的工作方式是，重载本身定义了用法的类型，而最后一个函数声明定义了函数体实现的类型。使用`createLabel`，我们可以传入一个`string`并获得`StringLabel`，或传入一个`number`并获得`NumberLabel`，因为这些是外部可用的类型。

这在我们无法事先缩小输入类型的情况下是个问题。我们缺少一个向外界传递允许输入为`number`或`string`的函数类型：

```
function inputToLabel(input: string | number) {
  return createLabel(input);
  //                    ^
  // No overload matches this call. (2769)
}
```

为了规避这种情况，我们添加另一个重载，以匹配非常广泛的输入类型的实现函数签名：

```
function createLabel(input: number): NumberLabel;
function createLabel(input: string): StringLabel;
function createLabel(input: number | string): NumberLabel | StringLabel;
function createLabel(input: number | string): NumberLabel | StringLabel {
  if (typeof input === "number") {
    return { id: input };
  } else {
    return { name: input };
  }
}
```

我们在这里看到，我们已经需要三个重载和四个函数签名声明，以描述此功能的最基本行为。从这里开始，情况只会变得更糟。

我们希望扩展我们的函数，以能够复制现有的`StringLabel`和`NumberLabel`对象。这最终意味着需要更多的重载：

```
function createLabel(input: number): NumberLabel;
function createLabel(input: string): StringLabel;
function createLabel(input: StringLabel): StringLabel;
function createLabel(input: NumberLabel): NumberLabel;
function createLabel(input: string | StringLabel): StringLabel;
function createLabel(input: number | NumberLabel): NumberLabel;
function createLabel(
  input: number | string | StringLabel | NumberLabel
): NumberLabel | StringLabel;
function createLabel(
  input: number | string | StringLabel | NumberLabel
): NumberLabel | StringLabel {
  if (typeof input === "number") {
    return { id: input };
  } else if (typeof input === "string") {
    return { name: input };
  } else if ("id" in input) {
    return { id: input.id };
  } else {
    return { name: input.name };
  }
}
```

坦率地说，根据我们希望类型提示有多么表达，我们可以编写较少但也更多的函数重载。问题仍然显而易见：更多的多样性导致更复杂的函数签名。

TypeScript 工具包中的一个工具可以帮助解决这种情况：条件类型。条件类型允许我们根据某些子类型检查选择类型。我们询问泛型类型参数是否属于某个子类型，如果是，则从`true`分支返回类型，否则从`false`分支返回类型。

例如，如果`T`是`string`的子类型（即所有字符串或非常具体的字符串），则以下类型返回输入参数。否则，返回`never`：

```
type IsString<T> = T extends string ? T : never;

type A = IsString<string>; // string
type B = IsString<"hello" | "world">; // string
type C = IsString<1000>; // never
```

TypeScript 从 JavaScript 的三元运算符中借用了这种语法。就像 JavaScript 的三元运算符一样，它检查某些条件是否有效。但与编程语言中通常的一套条件不同，TypeScript 的类型系统仅检查输入类型的值是否包含在我们检查的值集合中。

借助这个工具，我们能够编写一个名为`GetLabel<T>`的条件类型。我们检查输入是否为`string`或`StringLabel`。如果是，则返回`StringLabel`；否则，我们知道它必须是`NumberLabel`：

```
type GetLabel<T> = T extends string | StringLabel ? StringLabel : NumberLabel;
```

此类型仅检查输入`string`、`StringLabel`、`number`和`NumberLabel`是否位于`else`分支中。如果我们希望安全，还应包括对可能产生`NumberLabel`的输入进行检查，通过嵌套条件类型：

```
type GetLabel<T> = T extends string | StringLabel
  ? StringLabel
  : T extends number | NumberLabel
  ? NumberLabel
  : never;
```

现在是连接泛型的时候了。我们在`cr⁠ea⁠te​Lab⁠el`中添加一个新的泛型类型参数`T`，它受到所有可能输入类型的限制。这个`T`参数作为`GetLabel<T>`的输入，将产生相应的返回类型：

```
function createLabel<T extends number | string | StringLabel | NumberLabel>(
  input: T
): GetLabel<T> {
  if (typeof input === "number") {
    return { id: input } as GetLabel<T>;
  } else if (typeof input === "string") {
    return { name: input } as GetLabel<T>;
  } else if ("id" in input) {
    return { id: input.id } as GetLabel<T>;
  } else {
    return { name: input.name } as GetLabel<T>;
  }
}
```

现在我们已经准备好处理所有可能的类型组合，并且仍然可以从`getLabel`中获取正确的返回类型，所有这些只需一行代码即可完成。

如果你仔细观察，你会发现我们需要解决返回类型的类型检查问题。不幸的是，TypeScript 在处理泛型和条件类型时无法进行适当的控制流分析。通过少量类型断言告诉 TypeScript 我们正在处理正确的返回类型。

另一个解决方法是将具有条件类型的函数签名视为对原始广泛类型函数的重载：

```
function createLabel<T extends number | string | StringLabel | NumberLabel>(
  input: T
): GetLabel<T>;
function createLabel(
  input: number | string | StringLabel | NumberLabel
): NumberLabel | StringLabel {
  if (typeof input === "number") {
    return { id: input };
  } else if (typeof input === "string") {
    return { name: input };
  } else if ("id" in input) {
    return { id: input.id };
  } else {
    return { name: input.name };
  }
}
```

这样，我们为外部世界提供了一个灵活的类型，准确告诉我们基于输入能得到什么输出。而对于实现来说，你可以享有来自各种类型广泛集合的全面灵活性。

这是否意味着在所有情况下你应该优先选择条件类型而不是函数重载？未必。在 Recipe 12.7 中，我们会看到在某些情况下函数重载是更好的选择。

# 5.2 使用 `never` 进行过滤

## 问题

你有各种类型的联合体，但你只想要所有字符串的子类型。

## 解决方案

使用分布式条件类型来过滤正确的类型。

## 讨论

假设你的应用程序中有一些遗留代码，尝试重新创建类似 *jQuery* 的框架。你有自己的 `ElementList` 类型，其中有帮助函数用于向 `HTMLElement` 对象添加或删除类名，或者绑定事件监听器到事件上。

此外，你还可以通过索引访问来访问列表中的每个元素。这种 `ElementList` 的类型可以使用数字索引访问类型以及常规字符串属性键来描述：

```
type ElementList = {
  addClass: (className: string) => ElementList;
  removeClass: (className: string) => ElementList;
  on: (event: string, callback: (ev: Event) => void) => ElementList;
  length: number;
  [x: number]: HTMLElement;
};
```

这个数据结构被设计成具有流畅的接口。这意味着如果你调用 `addClass` 或 `removeClass` 等方法，你会得到相同的对象返回，因此可以链式调用你的方法。

这些方法的示例实现可能如下所示：

```
// begin excerpt
  addClass: function (className: string): ElementList {
    for (let i = 0; i < this.length; i++) {
      this[i].classList.add(className);
    }
    return this;
  },
  removeClass: function (className: string): ElementList {
    for (let i = 0; i < this.length; i++) {
      this[i].classList.remove(className);
    }
    return this;
  },
  on: function (event: string, callback: (ev: Event) => void): ElementList {
    for (let i = 0; i < this.length; i++) {
      this[i].addEventListener(event, callback);
    }
    return this;
  },
// end excerpt
```

作为内置集合（如 `Array` 或 `NodeList`）的扩展，更改 `HTMLElement` 对象集上的东西变得非常方便：

```
declare const myCollection: ElementList;

myCollection
  .addClass("toggle-off")
  .removeClass("toggle-on")
  .on("click", (e) => {});
```

假设你需要维护你的 *jQuery* 替代品，并发现直接元素访问在某种程度上是不安全的。当你的应用程序的某些部分可以直接更改事物时，你将更难弄清楚变化来自何处，如果不是来自你精心设计的 `ElementList` 数据结构：

```
myCollection[1].classList.toggle("toggle-on");
```

由于你不能改变原始的库代码（太多部门依赖它），你决定在一个`Proxy`中封装原始的`ElementList`。

`Proxy` 对象接受一个原始目标对象和一个处理程序对象，定义如何处理访问。以下实现展示了一个 `Proxy`，只允许读取访问，并且只有当属性键的类型为 `string` 而不是表示数字的字符串时：

```
const safeAccessCollection = new Proxy(myCollection, {
  get(target, property) {
    if (
      typeof property === "string" &&
      property in target &&
      "" + parseInt(property) !== property
    ) {
      return target[property as keyof typeof target];
    }
    return undefined;
  },
});
```

###### 注意

在 `Proxy` 对象中，处理程序对象只接收字符串或符号属性。如果你使用数字进行索引访问，例如 `0`，JavaScript 会将其转换为字符串 `"0"`。

这在 JavaScript 中效果很好，但我们的类型不再匹配。`Proxy` 构造函数的返回类型再次是 `ElementList`，这意味着数字索引访问仍然保持完整：

```
// Works in TypeScript throws in JavaScript
safeAccessCollection[0].classList.toggle("toggle-on");
```

我们需要告诉 TypeScript，我们现在处理的是一个没有数字索引访问的对象，通过定义一个新类型来实现。

让我们看看 `ElementList` 的键。如果我们使用 `keyof` 操作符，我们会得到一个 `ElementList` 类型对象所有可能访问方法的联合类型：

```
// resolves to "addClass" | "removeClass" | "on" | "length" | number
type ElementListKeys = keyof ElementList;
```

它包含四个字符串以及所有可能的数字。现在我们有了这个联合类型，我们可以创建一个条件类型，来过滤掉不是字符串的一切：

```
type JustStrings<T> = T extends string ? T : never;
```

`JustStrings<T>`就是我们所谓的*分布条件类型*。由于条件中的`T`单独存在于条件中—而不是包裹在对象或数组中—TypeScript 将一个联合类型的条件类型视为条件类型的联合。事实上，TypeScript 对联合`T`的每个成员进行相同的条件检查。

在我们的案例中，它穿过了`keyof ElementList`的所有成员：

```
type JustElementListStrings =
  | "addClass" extends string ? "addClass" : never
  | "removeClass" extends string ? "removeClass" : never
  | "on" extends string ? "on" : never
  | "length" extends string ? "length" : never
  | number extends string ? number : never;
```

唯一进入`false`分支的条件是最后一个条件，我们检查`number`是否是`string`的子类型，它不是。如果我们解决每个条件，我们最终得到一个新的联合类型：

```
type JustElementListStrings =
  | "addClass"
  | "removeClass"
  | "on"
  | "length"
  | never;
```

具有`never`的联合有效地删除了`never`。如果您有一个没有可能值的集合，并将其与值集合合并，那么这些值将保留下来：

```
type JustElementListStrings =
  | "addClass"
  | "removeClass"
  | "on"
  | "length";
```

这确切是我们考虑安全访问的键列表！通过使用`Pick`辅助类型，我们可以创建一个类型，通过挑选所有类型为`string`的键来有效地创建一个`ElementList`的超类型：

```
type SafeAccess = Pick<ElementList, JustStrings<keyof ElementList>>;
```

如果我们悬停在上面，我们看到结果类型正是我们所期望的：

```
type SafeAccess = {
  addClass: (className: string) => ElementList;
  removeClass: (className: string) => ElementList;
  on: (event: string, callback: (ev: Event) => void) => ElementList;
  length: number;
};
```

让我们将类型作为注释添加到`safeAccessCollection`。由于可以分配给超类型，TypeScript 将从那一刻起将`safeAccessCollection`视为无法从数字索引访问的类型：

```
const safeAccessCollection: Pick<
  ElementList,
  JustStrings<keyof ElementList>
> = new Proxy(myCollection, {
  get(target, property) {
    if (
      typeof property === "string" &&
      property in target &&
      "" + parseInt(property) !== property
    ) {
      return target[property as keyof typeof target];
    }
    return undefined;
  },
});
```

现在，当我们尝试从`safeAccessCollection`访问元素时，TypeScript 将向我们报错：

```
safeAccessCollection[1].classList.toggle("toggle-on");
// ^ Element implicitly has an 'any' type because expression of
// type '1' can't be used to index type
// 'Pick<ElementList, "addClass" | "removeClass" | "on" | "length">'.
```

这正是我们所需要的。分布条件类型的威力在于我们可以更改联合的成员。我们将在配方 5.3 中看到另一个示例，我们将使用内置的辅助类型。

# 5.3 按种类分组元素

## 问题

来自配方 4.5 的您的`Group`类型运行良好，但组的每个条目的类型过于宽泛。

## 解决方案

使用`Extract`辅助类型从联合类型中选择正确的成员。

## 讨论

让我们回到 3.1 和 4.5 章节中来自玩具店示例。我们开始用精心制作的模型，通过辨别联合类型，我们可以获得有关每个可能值的精确信息：

```
type ToyBase = {
  name: string;
  description: string;
  minimumAge: number;
};

type BoardGame = ToyBase & {
  kind: "boardgame";
  players: number;
};

type Puzzle = ToyBase & {
  kind: "puzzle";
  pieces: number;
};

type Doll = ToyBase & {
  kind: "doll";
  material: "plush" | "plastic";
};

type Toy = Doll | Puzzle | BoardGame;
```

然后，我们找到了一种方法*派生*另一种名为`GroupedToys`的类型，该类型从`Toy`派生，我们从`kind`属性的联合类型成员作为映射类型的属性键，每个属性的类型为`Toy[]`：

```
type GroupedToys = {
  [k in Toy["kind"]]?: Toy[];
};
```

多亏了泛型，我们能够定义一个辅助类型`Group<Collection, Selector>`以便在不同情境下重复使用相同的模式：

```
type Group<
  Collection extends Record<string, any>,
  Selector extends keyof Collection
> = {
  [K in Collection[Selector]]: Collection[];
};

type GroupedToys = Partial<Group<Toy, "kind">>;
```

辅助类型运行良好，但有一个注意事项。如果我们悬停在生成的类型上，我们会看到，尽管`Group<Collection, Selector>`能够正确地选择`Toy`联合类型的辨别，但所有属性指向非常宽泛的`Toy[]`：

```
type GroupedToys = {
  boardgame?: Toy[] | undefined;
  puzzle?: Toy[] | undefined;
  doll?: Toy[] | undefined;
};
```

但我们难道不应该了解更多吗？例如，为什么`boardgame`指向`Toy[]`，当唯一现实的类型应该是`BoardGame[]`。同样适用于 puzzle 和 dolls，以及我们希望添加到收藏中的所有后续玩具。我们期望的类型应该更像这样：

```
type GroupedToys = {
  boardgame?: BoardGame[] | undefined;
  puzzle?: Puzzle[] | undefined;
  doll?: Doll[] | undefined;
};
```

我们可以通过*从*`Collection`联合类型中提取相应成员来实现这种类型。幸运的是，有一个辅助类型可以做到这一点：`Extract<T, U>`，其中`T`是集合，`U`是`T`的一部分。

`Extract<T, U>` 的定义如下：

```
type Extract<T, U> = T extends U ? T : never;
```

因为条件中的`T`是一个裸类型，`T`是一个*分布条件类型*，这意味着 TypeScript 检查`T`的每个成员是否是`U`的子类型，如果是，它将在联合类型中保留此成员。这对于从`Toy`中挑选正确的玩具组会如何工作呢？

假设我们想从`Toy`中挑选`Doll`。`Doll`有一些属性，但`kind`属性明显与其余部分不同。因此，要使类型只查找`Doll`意味着我们从`Toy`中提取每种类型，其中`{ kind: "doll" }`：

```
type ExtractedDoll = Extract<Toy, { kind: "doll" }>;
```

使用分布条件类型，联合的条件类型是条件类型的联合，因此会检查`T`的每个成员是否符合`U`：

```
type ExtractedDoll =
  BoardGame extends { kind: "doll" } ? BoardGame : never |
  Puzzle extends { kind: "doll" } ? Puzzle : never |
  Doll extends { kind: "doll" } ? Doll : never;
```

`BoardGame`和`Puzzle`都不是`{ kind: "doll" }`的子类型，因此它们解析为`never`。但是`Doll` *是* `{ kind: "doll" }`的子类型，所以解析为`Doll`：

```
type ExtractedDoll = never | never | Doll;
```

在与`never`的联合中，`never`会直接消失。因此结果类型是`Doll`：

```
type ExtractedDoll = Doll;
```

这正是我们所需要的。让我们把这个检查加入我们的`Group`辅助类型中。幸运的是，我们已经拥有所有的部件来从组的集合中提取特定类型：

+   `Collection`本身，最终用`Toy`替换。

+   在`Selector`中的鉴别特性，最终用`"kind"`替换。

+   我们想要提取的鉴别类型是一个字符串类型，巧合的是也是我们在`Group`中映射出的属性键：`K`

因此，在`Group<Collection, Selector>`中`Extract<Toy, { kind: "doll" }>`的泛型版本如下：

```
type Group<
  Collection extends Record<string, any>,
  Selector extends keyof Collection
> = {
  [K in Collection[Selector]]: Extract<Collection, { [P in Selector]: K }>[];
};
```

如果我们用`Toy`替换`Collection`，用`"kind"`替换`Selector`，那么类型读起来如下：

`[K in Collection[Selector]]`

对于`Toy["kind"]`的每个成员——在这种情况下，是`"boardgame"`，`"puzzle"`和`"doll"`——作为新对象类型的属性键。

`Extract<Collection, …​>`

从`Collection`中提取，联合类型`Toy`，每个成员都是`...​`的子类型

`{ [P in Selector]: K }`

遍历`Selector`的每个成员——在我们的案例中，它只是`"kind"`——并创建一个对象类型，当属性键为`"boardgame"`时指向`"boardgame"`，为`"puzzle"`时指向`"puzzle"`，依此类推。

这就是我们为每个属性键选择`Toy`的方式。结果正如预期的那样：

```
type GroupedToys = Partial<Group<Toy, "kind">>;
// resolves to:
type GroupedToys = {
  boardgame?: BoardGame[] | undefined;
  puzzle?: Puzzle[] | undefined;
  doll?: Doll[] | undefined;
};
```

太棒了！类型现在清晰多了，我们可以确保在选择棋盘游戏时不必处理拼图。但是一些新问题已经出现。

由于每个属性的类型都更加精细，并且不指向非常广泛的 `Toy` 类型，TypeScript 在正确解析组内每个集合时有些困难：

```
function groupToys(toys: Toy[]): GroupedToys {
  const groups: GroupedToys = {};
  for (let toy of toys) {
    groups[toy.kind] = groups[toy.kind] ?? [];
//  ^ Type 'BoardGame[] | Doll[] | Puzzle[]' is not assignable to
//    type '(BoardGame[] & Puzzle[] & Doll[]) | undefined'. (2322)
    groups[toy.kind]?.push(toy);
//                         ^
//  Argument of type 'Toy' is not assignable to
//  parameter of type 'never'.  (2345)
  }
  return groups;
}
```

问题在于 TypeScript 仍然认为 `toy` 可能是所有玩具，而 `group` 的每个属性指向一些非常具体的玩具。有三种方法来解决这个问题。

首先，我们可以再次检查每个成员。由于 TypeScript 认为 `toy` 是一个非常广泛的类型，缩小范围可以再次明确关系：

```
function groupToys(toys: Toy[]): GroupedToys {
  const groups: GroupedToys = {};
  for (let toy of toys) {
    switch (toy.kind) {
      case "boardgame":
        groups[toy.kind] = groups[toy.kind] ?? [];
        groups[toy.kind]?.push(toy);
        break;
      case "doll":
        groups[toy.kind] = groups[toy.kind] ?? [];
        groups[toy.kind]?.push(toy);
        break;
      case "puzzle":
        groups[toy.kind] = groups[toy.kind] ?? [];
        groups[toy.kind]?.push(toy);
        break;
    }
  }
  return groups;
}
```

这样做是有效的，但是我们要避免大量的重复和重复。

其次，我们可以使用类型断言来扩展 `groups[toy.kind]` 的类型，以便 TypeScript 可以确保索引访问：

```
function groupToys(toys: Toy[]): GroupedToys {
  const groups: GroupedToys = {};
  for (let toy of toys) {
    (groups[toy.kind] as Toy[]) = groups[toy.kind] ?? [];
    (groups[toy.kind] as Toy[])?.push(toy);
  }
  return groups;
}
```

这实际上就像我们对 `GroupedToys` 进行更改之前一样有效，并且类型断言告诉我们，我们在这里有意改变了类型以摆脱类型错误。

第三，我们可以进行一些间接操作。我们不直接将 `toy` 添加到组中，而是使用一个帮助函数 `assign`，在其中使用泛型：

```
function groupToys(toys: Toy[]): GroupedToys {
  const groups: GroupedToys = {};
  for (let toy of toys) {
    assign(groups, toy.kind, toy);
  }
  return groups;
}

function assign<T extends Record<string, K[]>, K>(
  groups: T,
  key: keyof T,
  value: K
) {
  // Initialize when not available
  groups[key] = groups[key] ?? [];
  groups[key]?.push(value);
}
```

在这里，我们通过使用 TypeScript 的泛型替换来缩小 `Toy` 联合的正确成员：

+   `groups` 是 `T`，一个 `Record<string, K[]>`。`K[]` 可能会非常广泛。

+   `key` 与 `T` 相关：`T` 的属性键。

+   `value` 的类型是 `K`。

当我们调用 `assign` 时，所有三个函数参数都与彼此相关，并且我们设计的类型关系方式使我们可以安全地访问 `groups[key]` 并将 `value` 推入数组。

此外，当我们调用 `assign` 时，每个参数的类型都符合我们刚刚设置的泛型类型约束。如果您想了解更多关于这种技术的信息，请查看 Recipe 12.6。

# 5.4 移除特定对象属性

## 问题

您希望创建一个通用的帮助对象类型，根据其类型而不是属性名称选择属性。

## 解决方案

在映射属性键时，使用条件类型和类型断言进行过滤。

## 讨论

TypeScript 允许您基于其他类型创建类型，因此您可以保持它们更新，而不必维护每一个派生类型。我们已经在早期的示例中看到了一些例子，比如 Recipe 4.5。在以下场景中，我们想根据其属性的类型调整现有对象类型。让我们看一个 `Person` 类型：

```
type Person = {
  name: string;
  age: number;
  profession?: string;
};
```

它由两个字符串组成 — `profession` 和 `name` — 以及一个数字：`age`。我们想创建一个仅包含字符串类型属性的类型：

```
type PersonStrings = {
  name: string;
  profession?: string;
};
```

TypeScript 已经有一些辅助类型来处理过滤属性名。例如，映射类型 `Pick<T>` 获取对象的一部分键，以创建一个仅包含这些键的新对象：

```
type Pick<T, K extends keyof T> = {
  [P in K]: T[P];
}

// Only includes "name"
type PersonName = Pick<Person, "name">;

// Includes "name" and "profession"
type PersonStrings = Pick<Person, "name" | "profession">;
```

如果我们想要移除某些属性，我们可以使用 `Omit<T>`，它与 `Pick<T>` 类似，只是我们通过一个稍微改变的属性集进行映射，其中我们移除不想包括的属性名称：

```
type Omit<T, K extends string | number | symbol> = {
  [P in Exclude<keyof T, K>]: T[P];
}

// Omits age, thus includes "name" and "profession"
type PersonWithoutAge = Omit<Person, "age">;
```

要基于它们的类型而不是名称选择正确的属性，我们需要创建一个类似的辅助类型，其中我们映射一个动态生成的属性名集，该集仅指向我们正在寻找的类型。我们从第 5.2 节知道，当在联合类型上使用条件类型时，我们可以使用`never`来过滤这个联合中的元素。

因此，第一种可能性是我们映射`Person`的所有属性键，并检查`Person[K]`是否是我们期望类型的子集。如果是，则返回该类型；否则返回`never`：

```
// Not there yet
type PersonStrings = {
  [K in keyof Person]: Person[K] extends string ? Person[K] : never;
};
```

这很好，但要注意：我们检查的类型不是联合类型，而是映射类型的类型。因此，与过滤属性键不同，我们将获得指向类型`never`的属性，这意味着我们将禁止设置某些属性。

另一个想法是将类型设置为`undefined`，将属性视为可选的，但正如我们在第 3.11 节中学到的那样，缺失的属性和`undefined`值并不相同。

我们实际想要做的是删除指向特定类型的属性键。这可以通过将条件放在对象的左侧而不是右侧来实现，属性是在左侧创建的。

就像`Omit`类型一样，我们需要确保映射特定的属性集。当映射`Person`的`keyof`时，可以通过类型断言改变属性键的类型。与常规类型断言一样，有一种故障安全机制，这意味着您不能断言它为任何东西：它必须在属性键的边界内。

我们想要断言`K`是集合的一部分，如果`Person[K]`是`string`类型。如果是这样，我们保留`K`；否则，我们用`never`过滤集合中的元素。由于`never`位于对象的左侧，属性将被删除：

```
type PersonStrings = {
  [K in keyof Person as Person[K] extends string ? K : never]: Person[K];
};
```

现在，我们只选择指向字符串值的属性键。有一个要注意的地方：可选的字符串属性比普通字符串具有更广泛的类型，因为`undefined`也可能是其可能的值之一。使用联合类型可以确保可选属性也被保留：

```
type PersonStrings = {
  [K in keyof Person as Person[K] extends string | undefined
    ? K
    : never]: Person[K];
};
```

下一步是使这种类型泛化。我们通过用`O`替换`Person`，用`T`替换`string`来创建类型`Select<O, T>`：

```
type Select<O, T> = {
  [K in keyof O as O[K] extends T | undefined ? K : never]: O[K];
};
```

这种新的辅助类型非常灵活。我们可以用它来从我们自己的对象类型中选择特定类型的属性：

```
type PersonStrings = Select<Person, string>;
type PersonNumbers = Select<Person, number>;
```

但是我们也可以找出，例如，字符串原型中返回数字的函数：

```
type StringFnsReturningNumber = Select<String, (...args: any[]) => number>;
```

一个反向的辅助类型`Remove<O, T>`，我们希望删除特定类型的属性键，与`Select<O, T>`非常相似。唯一的区别在于切换条件并在`true`分支中返回`never`：

```
type Remove<O, T> = {
  [K in keyof O as O[K] extends T | undefined ? never : K]: O[K];
};

type PersonWithoutStrings = Remove<Person, string>;
```

如果您创建对象类型的可序列化版本，这尤其有帮助：

```
type User = {
  name: string;
  age: number;
  profession?: string;
  posts(): string[];
  greeting(): string;
};

type SerializeableUser = Remove<User, Function>;
```

通过了解在映射键时可以使用条件类型，您突然就可以访问各种潜在的辅助类型。关于这一点的更多信息，请参阅 第八章。

# 5.5 条件中的类型推断

## 问题

您希望创建一个对象序列化的类，它会删除对象的所有不可序列化属性，如函数。如果您的对象具有 `serialize` 函数，则序列化器将使用该函数的返回值，而不是自行序列化对象。如何定义这种类型？

## 解决方案

使用递归条件类型来修改现有的对象类型。对于实现了 `serialize` 的对象，使用 `infer` 关键字将通用返回类型固定到一个具体类型。

## 讨论

序列化是将数据结构和对象转换为可以存储或传输的格式的过程。想象一下，将 JavaScript 对象的数据存储在磁盘上，然后通过再次反序列化它将其取回到 JavaScript 中。

JavaScript 对象可以包含任何类型的数据：像字符串或数字这样的原始类型，以及对象和甚至函数这样的复合类型。函数很有趣，因为它们不包含数据，而是行为：这是一些无法很好地序列化的内容。序列化 JavaScript 对象的一种方法是完全摒弃函数。这正是我们想在本课程中实现的内容。

我们从一个简单的对象类型 `Person` 开始，其中包含我们想要存储的数据的常规主题：人的姓名和年龄。它还有一个 `hello` 方法，生成一个字符串：

```
type Person = {
  name: string;
  age: number;
  hello: () => string;
};
```

我们希望序列化这种类型的对象。一个 `Serializer` 类包含一个空的构造函数和一个通用函数 `serialize`。注意，我们将通用类型参数添加到 `serialize` 而不是类本身。这样，我们可以为不同的对象类型重复使用 `serialize`。返回类型指向一个通用类型 `Serialize<T>`，这将是序列化过程的结果：

```
class Serializer {
  constructor() {}
  serialize<T>(obj: T): Serialize<T> {
    // tbd...
  }
}
```

我们稍后会处理具体的实现。现在让我们专注于 `Serialize<T>` 类型。首先想到的一个想法是仅丢弃函数属性。我们已经在 5.4 节 中定义了一个 `Remove<O, T>` 类型，它非常方便，因为它正是做这件事——删除特定类型的属性：

```
type Remove<O, T> = {
  [K in keyof O as O[K] extends T | undefined ? never : K]: O[K];
};

type Serialize<T> = Remove<T, Function>;
```

第一次迭代已经完成，并且适用于简单的一级深度对象。然而，对象可以很复杂。例如，`Person` 可以嵌套其他对象，这些对象本身也可能具有函数：

```
type Person = {
  name: string;
  age: number;
  profession: {
    title: string;
    level: number;
    printProfession: () => void;
  };
  hello: () => string;
};
```

要解决这个问题，我们需要检查每个属性是否是另一个对象，如果是，则再次使用 `Serialize<T>` 类型。名为 `NestSerialization` 的映射类型在条件类型中检查每个属性是否为 `object` 类型，在 `true` 分支中返回该类型的序列化版本，在 `false` 分支中返回该类型本身：

```
type NestSerialization<T> = {
  [K in keyof T]: T[K] extends object ? Serialize<T[K]> : T[K];
};
```

我们通过在`NestSerialization`中包装原始的`Remove<T, Function>`类型来重新定义`Serialize<T>`，从而有效地创建了一个*递归类型*：`Serialize<T>`使用`NestSerialization<T>`使用`Serialize<T>`，依此类推：

```
type Serialize<T> = NestSerialization<Remove<T, Function>>;
```

TypeScript 可以在一定程度上处理类型递归。在这种情况下，它可以看到在`NestSerialization`中确实存在一种条件来打破类型递归。

这就是序列化类型！现在来实现这个函数，这个函数奇怪地直接翻译了我们在 JavaScript 中的类型声明。我们检查每个属性，如果是对象，我们再次调用`serialize`。如果不是，则只转移该属性，但前提是它不是一个函数：

```
class Serializer {
  constructor() {}
  serialize<T>(obj: T): Serialize<T> {
    const ret: Record<string, any> = {};

    for (let k in obj) {
      if (typeof obj[k] === "object") {
        ret[k] = this.serialize(obj[k]);
      } else if (typeof obj[k] !== "function") {
        ret[k] = obj[k];
      }
    }
    return ret as Serialize<T>;
  }
}
```

注意，由于我们在`serialize`中生成了一个新对象，我们从一个非常广泛的`Record<string, any>`开始，这允许我们将任何字符串属性键设置为基本任何内容，并在最后断言我们创建了一个符合返回类型的对象。当您创建新对象时，这种模式很常见，但最终需要您确保百分之百正确。请广泛测试此函数。

第一次实现完成后，我们可以创建一个新的`Person`类型对象，并将其传递给我们新生成的序列化程序：

```
const person: Person = {
  name: "Stefan",
  age: 40,
  profession: {
    title: "Software Developer",
    level: 5,
    printProfession() {
      console.log(`${this.title}, Level ${this.level}`);
    },
  },
  hello() {
    return `Hello ${this.name}`;
  },
};

const serializer = new Serializer();
const serializedPerson = serializer.serialize(person);
console.log(serializedPerson);
```

结果如预期：`serializedPerson`的类型缺少所有方法和函数的信息。如果我们记录`serializedPerson`，我们还会看到所有方法和函数都已经消失了。类型与实现结果匹配：

```
[LOG]: {
  "name": "Stefan",
  "age": 40,
  "profession": {
    "title": "Software Developer",
    "level": 5
  }
}
```

但我们还没有完成。序列化程序有一个特殊功能。对象可以实现`serialize`方法，如果是这样，序列化程序将获取此方法的输出，而不是自行序列化对象。让我们扩展`Person`类型，以包含一个`serialize`方法：

```
type Person = {
  name: string;
  age: number;
  profession: {
    title: string;
    level: number;
    printProfession: () => void;
  };
  hello: () => string;
  serialize: () => string;
};

const person: Person = {
  name: "Stefan",
  age: 40,
  profession: {
    title: "Software Developer",
    level: 5,
    printProfession() {
      console.log(`${this.title}, Level ${this.level}`);
    },
  },
  hello() {
    return `Hello ${this.name}`;
  },
  serialize() {
    return `${this.name}: ${this.profession.title} L${this.profession.level}`;
  },
};
```

我们需要调整`Serialize<T>`类型。在运行`NestSerialization`之前，我们在条件类型中检查对象是否实现了`serialize`方法。我们通过询问`T`是否是包含`serialize`方法的类型的子类型来做到这一点。如果是这样，我们需要获取返回类型，因为那就是序列化的结果。

这就是`infer`关键字发挥作用的地方。它允许我们从条件中获取一个类型，并在`true`分支中将其用作类型参数。我们告诉 TypeScript，如果这个条件为真，则取出你在那里找到的类型并使其对我们可用：

```
type Serialize<T> = T extends { serialize(): infer R }
  ? R
  : NestSerialization<Remove<T, Function>>;
```

把`R`想象成一开始是`any`。如果我们将`Person`与`{ serialize(): any }`进行比较，我们就进入了`true`分支，因为`Person`有一个`serialize`函数，使其成为有效的子类型。但是`any`是广泛的，我们感兴趣的是在`any`位置的具体类型。`infer`关键字可以选择确切的类型。因此，`Serialize<T>`现在读取：

+   如果`T`包含一个`serialize`方法，则获取其返回类型并返回它。

+   否则，通过深度移除所有类型为`Function`的属性来开始序列化。

我们也希望在我们的 JavaScript 实现中反映该类型的行为。我们进行了一些类型检查（检查`serialize`是否可用以及它是否是一个函数），最终调用它。TypeScript 要求我们使用类型守卫明确表示，以确保这个函数确实存在：

```
class Serializer {
  constructor() {}
  serialize<T>(obj: T): Serialize<T> {
    if (
      // is an object
      typeof obj === "object" &&
      // not null
      obj &&
      // serialize is available
      "serialize" in obj &&
      // and a function
      typeof obj.serialize === "function"
    ) {
      return obj.serialize();
    }

    const ret: Record<string, any> = {};

    for (let k in obj) {
      if (typeof obj[k] === "object") {
        ret[k] = this.serialize(obj[k]);
      } else if (typeof obj[k] !== "function") {
        ret[k] = obj[k];
      }
    }
    return ret as Serialize<T>;
  }
}
```

有了这个改变，`serializedPerson` 的类型是`string`，而结果也如预期的那样：

```
[LOG]: "Stefan: Software Developer L5"
```

这个强大的工具在对象生成方面提供了很大帮助。并且，我们用 TypeScript 的类型系统作为声明性元语言来创建类型，最终以 JavaScript 的命令式写法看到相同的过程，这其中蕴含着美感。
