# 第十一章：类

当 TypeScript 在 2012 年首次发布时，JavaScript 生态系统和 JavaScript 语言的特性与今天的情况无法相提并论。TypeScript 不仅引入了类型系统，还通过语法丰富了已有语言，使您能够在模块、命名空间和类型之间抽象代码的部分。

TypeScript 的一个特性是类（classes），在面向对象编程中是一个基本要素。TypeScript 的类最初受到了 C# 的很大影响，如果你了解这两种编程语言背后的人物，这一点并不奇怪。^(1) 但它们也是基于被废弃的 ECMAScript 4 提案中的概念设计的。

随着时间的推移，JavaScript 获得了 TypeScript 和其他语言先驱创造的许多语言特性；类似私有字段、静态块和装饰器的类现在已经成为 ECMAScript 标准的一部分，并已经被部署到浏览器和服务器的语言运行时中。

这使得 TypeScript 处于一个甜蜜点，既保留了它在语言早期引入的创新，又符合 TypeScript 团队对类型系统所有未来功能的基线标准。尽管原始设计接近 JavaScript 的最终形式，但也有一些值得一提的差异。

在本章中，我们将探讨 TypeScript 和 JavaScript 中类的行为方式，我们表达自己的可能性，以及标准设计和原始设计之间的差异。我们将研究关键字、类型和泛型，并着眼于 TypeScript 在 JavaScript 中添加的内容，以及 JavaScript 自身带来的内容。

# 11.1 选择正确的可见性修饰符

## 问题

TypeScript 中有两种属性可见性和访问的风格：一种是通过特殊的关键字语法 — `public`、`protected`、`private` — 另一种是通过实际的 JavaScript 语法，当属性以井号字符开头时。你应该选择哪一种呢？

## 解决方案

最好选择 JavaScript 原生语法，因为它在运行时有一些影响，你不希望错过。如果你依赖于涉及可见性修饰符变体的复杂设置，请使用 TypeScript 的修饰符。它们不会消失。

## 讨论

TypeScript 的类已经存在了相当长的时间，虽然它们受到了随后几年的 ECMAScript 类的巨大启发，但 TypeScript 团队还决定引入当时传统基于类的面向对象编程中有用和流行的特性。

其中之一就是*属性可见性修饰符*，也称为*访问修饰符*。可见性修饰符是您可以放在成员（属性和方法）前面的特殊关键字，用来告诉编译器它们如何从软件的其他部分看到和访问。

###### 注意

所有的可见性修饰符以及 JavaScript 的私有字段，都可以作用于方法和属性。

默认的可见性修饰符是 `public`，可以显式地写出来，也可以省略不写：

```
class Person {
  public name; // modifier public is optional
  constructor(name: string) {
    this.name = name;
  }
}

const myName = new Person("Stefan").name; // works
```

另一个修饰符是 `protected`，限制了对类和子类的可见性：

```
class Person {
  protected name;
  constructor(name: string) {
    this.name = name;
  }
  getName() {
    // access works
    return this.name;
  }
}

const myName = new Person("Stefan").name;
//                                   ^
// Property 'name' is private and only accessible within
// class 'Person'.(2341)

class Teacher extends Person {
  constructor(name: string) {
    super(name);
  }

  getFullName() {
    // access works
    return `Professor ${this.name}`;
  }
}
```

`protected` 访问可以在派生类中被重写为 `public`。`protected` 访问还禁止从不同子类的类引用中访问成员。因此，尽管这样做是有效的：

```
class Player extends Person {
  constructor(name: string) {
    super(name);
  }

  pair(p: Player) {
    // works
    return `Pairing ${this.name} with ${p.name}`;
  }
}
```

使用基类或不同的子类是行不通的：

```
class Player extends Person {
  constructor(name: string) {
    super(name);
  }

  pair(p: Person) {
    return `Pairing ${this.name} with ${p.name}`;
    //                                    ^
    // Property 'name' is protected and only accessible through an
    // instance of class 'Player'. This is an instance of
    // class 'Person'.(2446)
  }
}
```

最后一个可见性修饰符是 `private`，它只允许在同一类内部访问：

```
class Person {
  private name;
  constructor(name: string) {
    this.name = name;
  }
}

const myName = new Person("Stefan").name;
//                                   ^
// Property 'name' is protected and only accessible within
// class 'Person' and its subclasses.(2445)

class Teacher extends Person {
  constructor(name: string) {
    super(name);
  }

  getFullName() {
    return `Professor ${this.name}`;
    //                        ^
    // Property 'name' is private and only accessible
    // within class 'Person'.(2341)
  }
}
```

在构造函数中也可以使用可见性修饰符作为定义属性并初始化它们的快捷方式：

```
class Category {
  constructor(
    public title: string,
    public id: number,
    private reference: bigint
  ) {}
}

// transpiles to

class Category {
  constructor(title, id, reference) {
    this.title = title;
    this.id = id;
    this.reference = reference;
  }
}
```

尽管这里描述的所有功能，应该注意的是 TypeScript 的可见性修饰符是编译时的注释，在编译步骤后会被擦除。通常，如果它们不是通过类描述而是在构造函数中初始化，整个属性声明可能会被移除，就像我们在最后一个示例中看到的那样。

它们在编译时检查期间也是有效的，这意味着 TypeScript 中的 `private` 属性在转换为 JavaScript 后将完全可访问；因此，您可以通过断言实例 `as any` 来绕过 `private` 访问检查，或者在编译后直接访问它们。它们也是*可枚举的*，这意味着它们的名称和值在通过 `JSON.stringify` 或 `Object.getOwnPropertyNames` 序列化时变得可见。简而言之：一旦它们离开类型系统的边界，它们就会像常规的 JavaScript 类成员一样行事。

###### 注意

除了可见性修饰符，还可以将`readonly`修饰符添加到类属性中。

由于对属性的有限访问是一种不仅在类型系统内合理的特性，ECMAScript 还为常规的 JavaScript 类采用了类似的称为*私有字段*的概念。

而不是能见度修饰符，私有字段实际上通过在成员名称前面添加井号或*哈希*的形式引入了新的语法。

###### 提示

引入私有字段的新语法在社区内引发了关于井号的美感和审美的激烈争论。一些参与者甚至称其为可恶的。如果这个增加也让你感到不快，也许将井号看作是一种小围栏，用来保护你不希望所有人都能访问的东西，会让井号语法变得更加令人愉悦。

井号成为属性名称的一部分，这意味着访问时也需要在前面加上这个符号：

```
class Person {
  #name: string;

  constructor(name: string) {
    this.#name = name;
  }

  // we can use getters!
  get name(): string {
    return this.#name.toUpperCase();
  }
}

const me = new Person("Stefan");
console.log(me.#name);
//              ^
// Property '#name' is not accessible outside
// class 'Person' because it has a private identifier.(18013)

console.log(me.name); // works
```

私有字段完全是 JavaScript；TypeScript 编译器不会删除任何内容，并且它们保持其功能——在类内部隐藏信息——即使在编译步骤之后。使用最新的 ECMAScript 版本作为目标进行转译后，其结果看起来几乎与 TypeScript 版本相同，只是没有类型注解：

```
class Person {
  #name;

  constructor(name) {
    this.#name = name;
  }

  get name() {
    return this.#name.toUpperCase();
  }
}
```

私有字段在运行时代码中无法访问，它们也不可枚举，这意味着它们的内容不会以任何方式泄漏。

现在的问题是 TypeScript 中存在私有可见性修饰符和私有字段。可见性修饰符一直存在，并且与 `protected` 成员结合使用具有更多的多样性。另一方面，私有字段尽可能接近 JavaScript，并且在 TypeScript 的目标是“JavaScript 语法类型化”时基本命中该标记。那么你应该选择哪一个？

首先，无论你选择哪种修饰符，它们都完成了在编译时告知属性访问是否不应该的目标。这是你得到的第一个反馈，告诉你可能有问题，这也是我们在使用 TypeScript 时的目标。因此，如果需要隐藏外部信息，每个工具都会发挥其作用。

但是当你进一步查看时，这又取决于你的设置。如果你已经设置了具有精细可见性规则的项目，可能无法立即将它们迁移到原生 JavaScript 版本中。此外，JavaScript 中缺少 `protected` 可见性可能对你的目标造成问题。如果已有的东西运作正常，就没必要进行更改。

如果在运行时可见性出现问题，显示了你想要隐藏的细节：如果你依赖于其他人使用你的代码作为库，他们不应能够访问所有内部信息，则私有字段是正确的选择。它们在浏览器和其他语言运行时中得到了很好的支持，并且 TypeScript 针对较旧的平台提供了 polyfill。

# 11.2 显式定义方法重写

## 问题

在你的类层次结构中，你从基类扩展并在子类中重写特定方法。当你重构基类时，你可能会携带旧的、未使用的方法，因为没有任何东西告诉你基类已经改变。

## 解决方案

打开 `noImplicitOverride` 标志，并使用 `override` 关键字来标志重写。

## 讨论

你希望在画布上绘制形状。你的软件能够接收具有 `x` 和 `y` 坐标的点集合，并根据特定的渲染函数，在 HTML 画布上绘制多边形、矩形或其他元素。

你决定使用一个类层次结构，其中基类 `Shape` 接受一个任意列表的 `Point` 元素，并在它们之间绘制线条。这个类通过设置器和获取器来处理内务管理，同时也实现了 `render` 函数本身：

```
type Point = {
  x: number;
  y: number;
};

class Shape {
  points: Point[];
  fillStyle: string = "white";
  lineWidth: number = 10;

  constructor(points: Point[]) {
    this.points = points;
  }

  set fill(style: string) {
    this.fillStyle = style;
  }

  set width(width: number) {
    this.lineWidth = width;
  }

  render(ctx: CanvasRenderingContext2D) {
    if (this.points.length) {
      ctx.fillStyle = this.fillStyle;
      ctx.lineWidth = this.lineWidth;
      ctx.beginPath();
      let point = this.points[0];
      ctx.moveTo(point.x, point.y);
      for (let i = 1; i < this.points.length; i++) {
        point = this.points[i];
        ctx.lineTo(point.x, point.y);
      }
      ctx.closePath();
      ctx.stroke();
    }
  }
}
```

要使用它，从 HTML 画布元素创建一个二维上下文，创建一个 `Shape` 的新实例，并将上下文传递给 `render` 函数：

```
const canvas = document.getElementsByTagName("canvas")[0];
const ctx = canvas?.getContext("2d");

const shape = new Shape([
  { x: 50, y: 140 },
  { x: 150, y: 60 },
  { x: 250, y: 140 },
]);
shape.fill = "red";
shape.width = 20;

shape.render(ctx);
```

现在我们想要使用已建立的基类，并为特定形状（如矩形）派生子类。我们保留内务管理方法，并特别重写 `constructor` 和 `render` 方法：

```
class Rectangle extends Shape {
  constructor(points: Point[]) {
    if (points.length !== 2) {
      throw Error(`Wrong number of points, expected 2, got ${points.length}`);
    }
    super(points);
  }

  render(ctx: CanvasRenderingContext2D) {
    ctx.fillStyle = this.fillStyle;
    ctx.lineWidth = this.lineWidth;
    let a = this.points[0];
    let b = this.points[1];
    ctx.strokeRect(a.x, a.y, b.x - a.x, b.y - a.y);
  }
}
```

使用`Rectangle`的方式几乎相同：

```
const rectangle = new Rectangle([
  {x: 130, y: 190},
  {x: 170, y: 250}
]);
rectangle.render(ctx);
```

随着软件的演变，我们不可避免地会改变类、方法和函数，我们代码库中的某人会将`render`方法重命名为`draw`：

```
class Shape {
  // see above

  draw(ctx: CanvasRenderingContext2D) {
    if (this.points.length) {
      ctx.fillStyle = this.fillStyle;
      ctx.lineWidth = this.lineWidth;
      ctx.beginPath();
      let point = this.points[0];
      ctx.moveTo(point.x, point.y);
      for (let i = 1; i < this.points.length; i++) {
        point = this.points[i];
        ctx.lineTo(point.x, point.y);
      }
      ctx.closePath();
      ctx.stroke();
    }
  }
}
```

事实上，这并不是一个问题，但如果我们在代码中没有使用`Rectangle`的`render`方法，也许是因为我们将这个软件发布为库，并没有在我们的测试中使用它，没有任何东西告诉我们`Rectangle`中的`render`方法仍然存在，并且与原始类没有任何连接。

这就是为什么 TypeScript 允许你用`override`关键字注释你想要覆盖的方法。这是一种来自 TypeScript 的语法扩展，将在 TypeScript 将你的代码转译为 JavaScript 时被移除。

当一个方法标记为`override`关键字时，TypeScript 会确保基类中存在相同名称和签名的方法。如果你将`render`重命名为`draw`，TypeScript 会告诉你在基类`Shape`中没有声明`render`方法：

```
class Rectangle extends Shape {
  // see above

  override render(ctx: CanvasRenderingContext2D) {
//         ^
// This member cannot have an 'override' modifier because it
// is not declared in the base class 'Shape'.(4113)
    ctx.fillStyle = this.fillStyle;
    ctx.lineWidth = this.lineWidth;
    let a = this.points[0];
    let b = this.points[1];
    ctx.strokeRect(a.x, a.y, b.x - a.x, b.y - a.y);
  }
}
```

这个错误是一个很好的保护措施，确保重命名和重构不会破坏你现有的合约。

###### 注意

即使`constructor`可以被视为一个被覆盖的方法，它的语义是不同的，并且通过其他规则处理（例如，在实例化子类时确保调用`super`）。

通过在你的*tsconfig.json*中打开`noImplicitOverrides`标志，你可以进一步确保需要用`override`关键字标记函数。否则，TypeScript 会抛出另一个错误：

```
class Rectangle extends Shape {
  // see above

  draw(ctx: CanvasRenderingContext2D) {
// ^
// This member must have an 'override' modifier because it
// overrides a member in the base class 'Shape'.(4114)
    ctx.fillStyle = this.fillStyle;
    ctx.lineWidth = this.lineWidth;
    let a = this.points[0];
    let b = this.points[1];
    ctx.strokeRect(a.x, a.y, b.x - a.x, b.y - a.y);
  }
}
```

###### 注意

像实现定义类基本形状的接口这样的技术已经提供了一个坚实的基线，可以防止你遇到这类问题。因此，在创建类层次结构时，将`override`关键字和`noImplictOverrides`视为额外的保护措施是很好的。

当你的软件需要依赖类层次结构工作时，使用`override`与`noImplicitAny`一起是确保你不会忘记任何事情的好方法。类层次结构，像任何层次结构一样，随着时间的推移往往变得复杂，因此尽可能采取任何保障措施。

# 11.3 描述构造函数和原型

## 问题

你想动态实例化特定抽象类的子类，但 TypeScript 不允许你实例化抽象类。

## 解决方案

使用*constructor interface*模式描述你的类。

## 讨论

如果你在 TypeScript 中使用类层次结构，TypeScript 的结构特性有时会妨碍你。例如，看看下面的类层次结构，我们想要根据不同的规则过滤一组元素：

```
abstract class FilterItem {
  constructor(private property: string) {};
  someFunction() { /* ... */ };
  abstract filter(): void;
}

class AFilter extends FilterItem {
  filter() { /* ... */ }
}

class BFilter extends FilterItem {
  filter() { /* ... */ }
}
```

`FilterItem`抽象类需要被其他类实现。在这个例子中，`AFilter`和`BFilter`，都是`FilterItem`的具体化，作为过滤器的基线：

```
const some: FilterItem = new AFilter('afilter'); // ok
```

当我们不是直接使用实例时，情况变得有趣。假设我们希望根据从 AJAX 调用获取的令牌实例化新的过滤器。为了方便我们选择过滤器，我们将所有可能的过滤器存储在映射中：

```
declare const filterMap: Map<string, typeof FilterItem>;

filterMap.set('number', AFilter);
filterMap.set('stuff', BFilter);
```

映射的泛型被设置为一个`string`（用于从后端获取的令牌）以及与`FilterItem`类型签名相补充的一切内容。我们在这里使用`typeof`关键字，以便能够将类添加到映射中，而不是对象。毕竟，我们之后要对它们进行实例化。

到目前为止，一切都按预期进行。问题出现在当你想从映射中获取一个类并创建一个新对象时：

```
let obj: FilterItem;
// get the constructor
const ctor = filterMap.get('number');

if(typeof ctor !== 'undefined') {
  obj = new ctor();
//          ^
// cannot create an object of an abstract class
}
```

这是一个问题！在这一点上，TypeScript 只知道我们从`filterMap`获取了一个`FilterItem`，我们无法实例化`FilterItem`。抽象类混合了类型信息（*类型命名空间*）和实际实现（*值命名空间*）。首先，让我们看看类型：我们期望从`filterMap`获取什么？让我们创建一个接口（或类型别名），定义`FilterItem`的*形状*应该是什么：

```
interface IFilter {
  new(property: string): IFilter;
  someFunction(): void;
  filter(): void;
}

declare const filterMap: Map<string, IFilter>;
```

注意`new`关键字。这是 TypeScript 定义构造函数类型签名的一种方式。如果我们用实际接口替换抽象类，将会出现大量错误。无论将`implements IFilter`命令放在哪里，似乎都没有任何实现能够满足我们的合同：

```
abstract class FilterItem implements IFilter { /* ... */ }
// ^
// Class 'FilterItem' incorrectly implements interface 'IFilter'.
// Type 'FilterItem' provides no match for the signature
// 'new (property: string): IFilter'.

filterMap.set('number', AFilter);
//                      ^
// Argument of type 'typeof AFilter' is not assignable
// to parameter of type 'IFilter'. Type 'typeof AFilter' is missing
// the following properties from type 'IFilter': someFunction, filter
```

这里发生了什么？看起来既不是实现也不是类本身能够获取我们在接口声明中定义的所有属性和函数。为什么？

JavaScript 类很特殊；它们不仅有我们可以轻松定义的一种类型，而是两种：静态侧的类型和实例侧的类型。如果我们将我们的类转译到 ES6 之前的形式，一个构造函数和一个原型，这可能会更清晰：

```
function AFilter(property) { // this is part of the static side
  this.property = property;  // this is part of the instance side
}

// a function of the instance side
AFilter.prototype.filter = function() {/* ... */}

// not part of our example, but on the static side
Afilter.something = function () { /* ... */ }
```

一个用来创建对象的类型。一个用来描述对象本身的类型。因此，让我们拆分它，并为它创建两个类型声明：

```
interface FilterConstructor {
  new (property: string): IFilter;
}

interface IFilter {
  someFunction(): void;
  filter(): void;
}
```

第一个类型，`FilterConstructor`，是*构造函数接口*。这里列出了所有静态属性和构造函数本身。构造函数返回一个实例：`IFilter`。`IFilter`包含了实例侧的类型信息。我们声明的所有函数。

通过拆分这些内容，我们的后续类型定义也变得更加清晰：

```
declare const filterMap: Map<string, FilterConstructor>;  /* 1 */

filterMap.set('number', AFilter);
filterMap.set('stuff', BFilter);

let obj: IFilter;  /* 2 */
const ctor = filterMap.get('number');
if(typeof ctor !== 'undefined') {
  obj = new ctor('a');
}
```

1.  我们将`FilterConstructor`类型的实例添加到映射中。这意味着我们只能添加能够产生所需对象的类。

1.  最终我们想要的是一个`IFilter`的实例。当使用`new`调用构造函数时，这就是构造函数返回的内容。

我们的代码再次编译通过，并且我们得到了所有的自动完成和工具支持。更好的是，我们不能将抽象类添加到映射中，因为它们不会产生有效的实例：

```
filterMap.set('notworking', FilterItem);
//                          ^
// Cannot assign an abstract constructor type to a
// non-abstract constructor type.
```

构造函数接口模式在整个 TypeScript 和标准库中广泛使用。要有一个概念，请查看*lib.es5.d.ts*中的`ObjectContructor`接口。

# 11.4 在类中使用泛型

## 问题

TypeScript 的泛型通常设计用于大量推断，但在类中，这并不总是有效。

## 解决方案

如果无法从参数中推断出泛型类型，请在实例化时显式注释泛型类型；否则，它们默认为 `unknown` 并接受广泛的值。使用泛型约束和默认参数可以提供额外的安全性。

## 讨论

类还允许使用泛型。我们不仅可以将泛型类型参数添加到函数中，还可以将泛型类型参数添加到类中。虽然类方法中的泛型类型参数仅在函数范围内有效，但类的泛型类型参数则在整个类中有效。

创建一个集合，这是一个简单的数组包装器，带有一组有限的便利函数。我们可以在 `Collection` 类定义中添加 `T`，并在整个类中重复使用这个类型参数：

```
class Collection<T> {
  items: T[];
  constructor() {
    this.items = [];
  }

  add(item: T) {
    this.items.push(item);
  }

  contains(item: T): boolean {
    return this.items.includes(item);
  }
}
```

有了这个，我们可以明确地用泛型类型注释替换 `T`，例如，允许一个仅包含数字或仅包含字符串的集合：

```
const numbers = new Collection<number>();
numbers.add(1);
numbers.add(2);

const strings = new Collection<string>();
strings.add("Hello");
strings.add("World");
```

作为开发者，我们并不需要显式地注释泛型类型参数。TypeScript 通常尝试从使用中推断泛型类型。如果我们*忘记*添加泛型类型参数，TypeScript 会回退到 `unknown`，允许我们添加任何内容：

```
const unknowns = new Collection();
unknowns.add(1);
unknowns.add("World");
```

让我们暂且停留在这一点上。TypeScript 对我们非常诚实。我们构造一个新的 `Collection` 实例时，我们不知道项目的类型是什么。`unknown` 是对集合状态最准确的描述。它伴随着所有的缺点：我们可以添加任何内容，并且每次检索值时都需要进行类型检查。尽管 TypeScript 在这一点上只能做出唯一可能的事情，但我们可能希望做得更好。为 `Collection` 指定一个具体的 `T` 类型对其正确运行是必不可少的。

让我们看看是否可以依赖推断。TypeScript 对类的推断与对函数的推断完全相同。如果有某种类型的参数，TypeScript 将采用这种类型并替换泛型类型参数。类被设计用于保持状态，并且状态在其使用过程中发生变化。状态还定义了我们的泛型类型参数 `T`。为了正确推断 `T`，我们需要在构造时要求一个参数，也许是一个初始值：

```
class Collection<T> {
  items: T[];
  constructor(initial: T) {
    this.items = [initial];
  }

  add(item: T) {
    this.items.push(item);
  }

  contains(item: T): boolean {
    return this.items.includes(item);
  }
}

// T is number!
const numbersInf = new Collection(0);
numbersInf.add(1);
```

这样做虽然可行，但在 API 设计方面还有很多不足之处。如果我们没有初始值会怎么样？虽然其他类可能有可以用于推断的参数，但对于包含各种项目的集合来说，这可能并没有太多意义。

对于 `Collection`，通过注释提供类型是非常重要的。唯一剩下的方法是确保我们不要忘记添加注释。为此，我们可以利用 TypeScript 的泛型默认参数和底部类型 `never`：

```
class Collection<T = never> {
  items: T[];
  constructor() {
    this.items = [];
  }

  add(item: T) {
    this.items.push(item);
  }

  contains(item: T): boolean {
    return this.items.includes(item);
  }
}
```

我们将通用类型参数 `T` 默认设置为 `never`，这为我们的类添加了一些非常有趣的行为。`T` 仍然可以通过注释显式地替换为每一种类型，正如以前一样，但一旦我们忘记注释，类型就不是 `unknown`，而是 `never`。这意味着我们的集合不兼容任何值，一旦我们尝试添加某些内容就会出现许多错误：

```
const nevers = new Collection();
nevers.add(1);
//     ^
// Argument of type 'number' is not assignable
// to parameter of type 'never'.(2345)
nevers.add("World");
//     ^
// Argument of type 'string' is not assignable
// to parameter of type 'never'.(2345)
```

这种后备方法使我们的通用类的使用更加安全。

# 11.5 决定何时使用类或命名空间

## 问题

TypeScript 提供了许多面向对象概念的语法，比如命名空间、静态类和抽象类。这些特性在 JavaScript 中并不存在，那么你该怎么办呢？

## 解决方案

坚持使用命名空间声明进行额外类型声明，尽可能避免抽象类，并优先使用 ECMAScript 模块而不是静态类。

## 讨论

我们从那些在传统面向对象编程语言（如 Java 或 C#）中工作过的人们那里看到的一件事是，他们倾向于将所有东西包装在一个类中。在 Java 中，因为类是结构化代码的唯一方式，你别无选择。在 JavaScript（因此也是 TypeScript）中，有很多其他可能性可以做到你想要的，而无需任何额外步骤。其中之一是静态类或具有静态方法的类：

```
// Environment.ts

export default class Environment {
  private static variableList: string[] = []
  static variables(): string[] { /* ... */ }
  static setVariable(key: string, value: any): void  { /* ... */ }
  static getValue(key: string): unknown  { /* ... */ }
}

// Usage in another file
import * as Environment from "./Environment";

console.log(Environment.variables());
```

虽然这样做有效且（去掉类型注解的话）是有效的 JavaScript，但这对于可以轻松地只是简单、无聊的函数的事情来说太过仪式化了：

```
// Environment.ts
const variableList: string = []

export function variables(): string[] { /* ... */ }
export function setVariable(key: string, value: any): void  { /* ... */ }
export function getValue(key: string): unknown  { /* ... */ }

// Usage in another file
import * as Environment from "./Environment";

console.log(Environment.variables());
```

对于用户的接口来说完全一样。您可以访问模块作用域变量，就像您访问类中的静态属性一样，但它们会自动在模块作用域内。您决定导出什么内容和使什么内容可见，而不是一些 TypeScript 字段修饰符。此外，您不会创建一个无用的 `Environment` 实例。

即使实现变得更简单了。查看 `variables()` 的类版本：

```
export default class Environment {
  private static variableList: string[] = [];
  static variables(): string[] {
    return this.variableList;
  }
}
```

与模块版本相比：

```
const variableList: string = []

export function variables(): string[] {
  return variableList;
}
```

没有 `this` 意味着少考虑。作为附加好处，您的捆绑器在做树摇时更容易，因此最终只会保留您实际使用的东西：

```
// Only the variables function and variableList
// end up in the bundle
import { variables } from "./Environment";

console.log(variables());
```

这就是为什么始终优先选择正确的模块而不是具有静态字段和方法的类。那只是增加了一些样板，没有额外的好处。

就像静态类一样，具有 Java 或 C# 背景的人们依恋命名空间，这是 TypeScript 在 ECMAScript 模块标准化之前引入的一个特性，用于组织代码。它们允许你在文件之间分割东西，并用参考标记器将它们合并：

```
// file users/models.ts
namespace Users {
  export interface Person {
    name: string;
    age: number;
  }
}

// file users/controller.ts

/// <reference path="./models.ts" />
namespace Users {
  export function updateUser(p: Person) {
    // do the rest
  }
}
```

当时，TypeScript 甚至有一个捆绑功能。它应该仍然有效。但正如注意到的那样，这是在 ECMAScript 引入模块之前。现在有了模块，我们有了一种方法来组织和结构化代码，与 JavaScript 生态系统的其他部分兼容。这是一个优点。

那么我们为什么需要命名空间呢？如果你想要扩展来自第三方依赖项的定义，例如存在于 node 模块中的定义，命名空间仍然是有效的。比如说，你想要扩展全局的`JSX`命名空间，并确保`img`元素包含 alt 文本：

```
declare namespace JSX {
  interface IntrinsicElements {
    "img": HTMLAttributes & {
      alt: string;
      src: string;
      loading?: 'lazy' | 'eager' | 'auto';
    }
  }
}
```

或者你想要在环境模块中编写复杂的类型定义。但除此之外呢？它的用处就不多了。

命名空间将你的定义封装到一个对象中，写起来像这样：

```
export namespace Users {
  type User = {
    name: string;
    age: number;
  };

  export function createUser(name: string, age: number): User {
    return { name, age };
  }
}
```

这会生成一些非常复杂的内容：

```
export var Users;
(function (Users) {
    function createUser(name, age) {
        return {
            name, age
        };
    }
    Users.createUser = createUser;
})(Users || (Users = {}));
```

这不仅会增加冗余代码，还会妨碍你的捆绑器正常摇树！使用它们也会变得有些啰嗦：

```
import * as Users from "./users";

Users.Users.createUser("Stefan", "39");
```

放弃它们会使事情变得简单得多。坚持使用 JavaScript 提供的功能。在声明文件之外不使用命名空间使你的代码清晰、简洁和整洁。

最后但并非最不重要的，还有抽象类。抽象类是结构化更复杂的类层次结构的一种方式，你可以预定义一些行为，但将一些特性的实际实现留给继承自你的抽象类的类：

```
abstract class Lifeform {
  age: number;
  constructor(age: number) {
    this.age = age;
  }

  abstract move(): string;
}

class Human extends Lifeform {
  move() {
    return "Walking, mostly...";
  }
}
```

所有`Lifeform`的子类都要实现`move`方法。这是基本上每种基于类的编程语言中都存在的概念。问题在于，JavaScript 并非传统的基于类的语言。例如，像下面这样的抽象类生成了一个有效的 JavaScript 类，但在 TypeScript 中却不允许实例化：

```
abstract class Lifeform {
  age: number;
  constructor(age: number) {
    this.age = age;
  }
}

const lifeform = new Lifeform(20);
//               ^
// Cannot create an instance of an abstract class.(2511)
```

如果你写普通的 JavaScript 但依赖于 TypeScript 来提供隐式文档形式的信息，比如函数定义看起来像这样，那么这可能会导致一些不需要的情况：

```
declare function moveLifeform(lifeform: Lifeform);
```

+   你或者你的用户可能会将其视为将`Lifeform`对象传递给`moveLifeform`的邀请。在内部，它调用`lifeform.move()`。

+   `Lifeform`可以在 JavaScript 中被实例化，因为它是一个有效的类。

+   方法`move`在`Lifeform`中不存在，因此破坏了你的应用程序！

这是由于一种错误的安全感。实际上你想要的是将一些预定义的实现放在原型链中，并且有一个合同告诉你应该期望什么：

```
interface Lifeform {
  move(): string;
}

class BasicLifeForm {
  age: number;
  constructor(age: number) {
    this.age = age;
  }
}

class Human extends BasicLifeForm implements Lifeform {
  move() {
    return "Walking";
  }
}
```

当你查找`Lifeform`时，你可以看到它的接口和它所期望的一切，但你很少会遇到意外实例化错误类的情况。

说了这么多关于何时*不*使用类和命名空间，那么何时应该使用它们呢？每当你需要同一个对象的多个实例，其中内部状态对对象功能至关重要时。

# 11.6 编写静态类

## 问题

基于类的面向对象编程教导你使用静态类来实现某些功能，但你想知道这些原则在 TypeScript 中是如何支持的。

## 解决方案

在 TypeScript 中传统的静态类不存在，但 TypeScript 对类成员有多种目的的静态修饰符。

## 讨论

静态类是不能实例化为具体对象的类。它们的目的是包含方法和其他成员，在代码的各个点访问时是相同的。静态类在只有类作为抽象手段的编程语言中是必需的，例如 Java 或 C#。在 JavaScript 以及随后的 TypeScript 中，有更多表达自己的方式。

在 TypeScript 中，我们不能声明类为 `static`，但可以在类上定义 `static` 成员。行为如预期：方法或属性不是对象的一部分，但可以从类本身访问。

正如我们在 Recipe 11.5 中看到的，只有静态成员的类在 TypeScript 中是一个反模式。函数存在；您可以每个模块保持状态。导出函数和模块范围的条目的组合通常是最佳选择：

```
// Anti-Pattern
export default class Environment {
  private static variableList: string[] = []
  static variables(): string[] { /* ... */ }
  static setVariable(key: string, value: any): void  { /* ... */ }
  static getValue(key: string): unknown  { /* ... */ }
}

// Better: Module-scoped functions and variables
const variableList: string = []

export function variables(): string[] { /* ... */ }
export function setVariable(key: string, value: any): void  { /* ... */ }
export function getValue(key: string): unknown  { /* ... */ }
```

但类的 `static` 部分仍然有用。我们在 Recipe 11.3 中建立了一个类由静态成员和动态成员组成的观点。

`constructor` 是类的静态特征的一部分，而属性和方法是类的动态特征的一部分。通过 `static` 关键字，我们可以添加这些静态特征。

让我们想象一个名为 `Point` 的类，描述二维空间中的一个点。它有 `x` 和 `y` 坐标，并且我们创建一个方法来计算此点与另一个点之间的距离：

```
class Point {
  x: number;
  y: number;

  constructor(x: number, y: number) {
    this.x = x;
    this.y = y;
  }

  distanceTo(point: Point): number {
    const dx = this.x - point.x;
    const dy = this.y - point.y;
    return Math.sqrt(dx * dx + dy * dy);
  }
}

const a = new Point(0, 0);
const b = new Point(1, 5);

const distance = a.distanceTo(b);
```

这是一个良好的行为，但是如果我们选择一个起点和终点可能会感觉有点奇怪，尤其是因为距离无论哪一个先都是相同的。`Point` 上的静态方法消除了顺序问题，我们有一个漂亮的 `distance` 方法，接受两个参数：

```
class Point {
  x: number;
  y: number;

  constructor(x: number, y: number) {
    this.x = x;
    this.y = y;
  }

  distanceTo(point: Point): number {
    const dx = this.x - point.x;
    const dy = this.y - point.y;
    return Math.sqrt(dx * dx + dy * dy);
  }

  static distance(p1: Point, p2: Point): number {
    return p1.distanceTo(p2);
  }
}

const a = new Point(0, 0);
const b = new Point(1, 5);

const distance = Point.distance(a, b);
```

在 JavaScript 的 ECMAScript 类之前使用构造函数/原型模式的类似版本如下所示：

```
function Point(x, y) {
  this.x = x;
  this.y = y;
}

Point.prototype.distanceTo = function(p) {
  const dx = this.x - p.x;
  const dy = this.y - p.y;
  return Math.sqrt(dx * dx + dy * dy);
}

Point.distance = function(a, b) {
  return a.distanceTo(b);
}
```

就像在 Recipe 11.3 中一样，我们可以轻松看到哪些部分是静态的，哪些部分是动态的。所有在 *原型* 中的东西属于动态部分。其他一切都是 *静态* 的。

但类不仅仅是构造函数/原型模式的语法糖。通过包含私有字段（在常规对象中不存在），我们可以做一些实际与类及其实例相关的事情。

如果我们希望隐藏 `distanceTo` 方法，因为它可能会令人困惑，我们更希望用户使用静态方法，只需在 `distanceTo` 前加上简单的私有修饰符，即可使其从外部无法访问，但仍然可以从静态成员中访问：

```
class Point {
  x: number;
  y: number;

  constructor(x: number, y: number) {
    this.x = x;
    this.y = y;
  }

  #distanceTo(point: Point): number {
    const dx = this.x - point.x;
    const dy = this.y - point.y;
    return Math.sqrt(dx * dx + dy * dy);
  }

  static distance(p1: Point, p2: Point): number {
    return p1.#distanceTo(p2);
  }
}
```

可见性也是反过来的。假设您有一个表示系统中某个 `Task` 的类，并且希望限制现有任务的数量。

我们使用一个名为`nextId`的静态私有字段，我们从`0`开始，并且我们增加这个私有字段以每个构造的实例`Task`。如果达到`100`，我们会抛出一个错误：

```
class Task {
  static #nextId = 0;
  #id: number;

  constructor() {
    if (Task.#nextId > 99) {
      throw "Max number of tasks reached";
    }
    this.#id = Task.#nextId++;
  }
}
```

如果我们希望通过后端的动态值来限制实例的数量，我们可以使用一个`static`实例化块来获取这些数据，并相应地更新静态私有字段：

```
type Config = {
  instances: number;
};

class Task {
  static #nextId = 0;
  static #maxInstances: number;
  #id: number;

  static {
    fetch("/available-slots")
      .then((res) => res.json())
      .then((result: Config) => {
        Task.#maxInstances = result.instances;
      });
    }

  constructor() {
    if (Task.#nextId > Task.#maxInstances) {
      throw "Max number of tasks reached";
    }
    this.#id = Task.#nextId++;
  }
}
```

与实例中的字段不同，在写作时，TypeScript 不会检查静态字段是否已被实例化。例如，如果我们从后端异步加载可用槽的数量，则在此期间我们可以构造实例，但不能检查是否达到了最大值。

因此，即使在 TypeScript 中没有静态类的构造，并且静态类仅被认为是反模式，但在许多情况下静态成员可能会有良好的用途。

# 11.7 使用严格属性初始化

## 问题

类保留状态，但没有任何信息告诉你这个状态是否已被初始化。

## 解决方案

通过在*tsconfig*中将`strictPropertyInitialization`设置为`true`来激活严格属性初始化。

## 讨论

类可以被视为用于创建对象的代码模板。您定义属性和方法，只有通过实例化才会分配实际的值。TypeScript 类将基本的 JavaScript 类与更多用于定义类型的语法相结合。例如，TypeScript 允许您以类似类型或接口的方式定义实例的属性：

```
type State = "active" | "inactive";

class Account {
  id: number;
  userName: string;
  state: State;
  orders: number[];
}
```

然而，这种标注只是定义了结构：它并没有设置任何具体的值。当被转译成常规 JavaScript 时，所有这些属性都会被擦除；它们只存在于*类型命名空间*中。

这种标注显然非常易读，并给开发者一个很好的想法，可以期望哪些属性。但不能保证这些属性实际存在。如果我们不初始化它们，一切都是缺失的或`undefined`。

TypeScript 对此有保护措施。通过在*tsconfig.json*中将`strictPropertyInitialization`标志设置为`true`，TypeScript 将确保在从类创建新对象时，所有你期望的属性都被初始化。

###### 注意

`strictPropertyInitialization`是 TypeScript 的`strict`模式的一部分。如果你在*tsconfig*中将`strict`设置为`true`——这是推荐的做法——你也会激活严格属性初始化。

一旦激活了这个选项，TypeScript 将在许多地方用红色波浪线向你打招呼：

```
class Account {
  id: number;
// ^ Property 'id' has no initializer and is
// not definitely assigned in the constructor.(2564)
  userName: string;
// ^ Property 'userName' has no initializer and is
// not definitely assigned in the constructor.(2564)
  state: State;
// ^ Property 'state' has no initializer and is
// not definitely assigned in the constructor.(2564)
  orders: number[];
// ^ Property 'orders' has no initializer and is
// not definitely assigned in the constructor.(2564)
}
```

太棒了！现在我们要确保每个属性都能收到一个值。有多种方法可以做到这一点。如果我们看一下`Account`示例，我们可以定义一些约束或规则，如果我们的应用程序域允许的话：

+   需要设置`id`和`userName`；它们控制与我们的后端通信并且在显示时是必需的。

+   `state`也需要设置，但其默认值为`active`。通常情况下，我们软件中的账户是活跃的，除非有意将其设置为`inactive`。

+   `orders`是一个包含订单 ID 的数组，但如果我们什么都没订购怎么办？一个空数组同样有效，或者也许我们将`orders`设置为尚未定义。

鉴于这些限制，我们已经可以排除两个错误。我们将`state`默认设置为`active`，并且我们使`orders`变为可选。还有可能将`orders`设置为`number[] | undefined`类型，这与可选相同：

```
class Account {
  id: number; // still errors
  userName: string; // still errors
  state: State = "active"; // ok
  orders?: number[]; // ok
}
```

另外两个属性仍然会报错。通过添加一个`constructor`并初始化这些属性，我们也排除了其他错误：

```
class Account {
  id: number;
  userName: string;
  state: State = "active";
  orders?: number[];

  constructor(userName: string, id: number) {
    this.userName = userName;
    this.id = id;
  }
}
```

这就是一个合适的 TypeScript 类！TypeScript 还允许使用构造函数的简写形式，通过添加`public`、`private`或`protected`等可见性修饰符，你可以将构造函数参数转换为具有相同名称和值的类属性。这是一个方便的功能，可以消除大量样板代码。重要的是不要在类形状中重新定义相同的属性：

```
class Account {
  state: State = "active";
  orders?: number[];

  constructor(public userName: string, public id: number) {}
}
```

如果你现在查看这个类，你会发现我们仅依赖于 TypeScript 的特性。转译后的类，即 JavaScript 等效物，看起来大不相同：

```
class Account {
  constructor(userName, id) {
    this.userName = userName;
    this.id = id;
    this.state = "active";
  }
}
```

一切都在`constructor`里面，因为`constructor`定义了一个实例。

###### 警告

尽管 TypeScript 的类快捷方式和语法看起来不错，但要小心不要过度依赖它们。最近几年，TypeScript 转变成了主要是在常规 JavaScript 上的类型语法扩展，但它们多年来存在的类特性仍然可用，并为您的代码添加了与您期望的不同的语义。如果你倾向于你的代码是“带有类型的 JavaScript”，那么当你进入 TypeScript 类特性的深处时要小心。

严格的属性初始化还理解复杂的情况，比如在通过`constructor`调用的函数内部设置属性。它还理解，异步类可能会使你的类处于潜在的未初始化状态。

假设你只需通过一个`id`属性初始化你的类，并从后端获取`userName`。如果你在构造函数内部进行异步调用，并在`fetch`调用完成后设置`userName`，你仍然会得到严格的属性初始化错误：

```
type User = {
  id: number;
  userName: string;
};

class Account {
  userName: string;
// ^ Property 'userName' has no initializer and is
// not definitely assigned in the constructor.(2564)
  state: State = "active";
  orders?: number[];

  constructor(public id: number) {
    fetch(`/api/getName?id=${id}`)
      .then((res) => res.json())
      .then((data: User) => (this.userName = data.userName ?? "not-found"));
  }
}
```

而这是真的！没有什么能告诉你`fetch`调用会成功，即使你捕获错误并确保属性会被初始化为一个备用值，你的对象在未初始化`userName`状态时也有一定时间的未初始化状态。

你可以做一些事情来解决这个问题。一个好的模式是有一个静态工厂函数，它异步工作，首先获取数据，然后调用期望两个属性的构造函数：

```
class Account {
  state: State = "active";
  orders?: number[];

  constructor(public id: number, public userName: string) {}

  static async create(id: number) {
    const user: User = await fetch(`/api/getName?id=${id}`).then((res) =>
      res.json()
    );
    return new Account(id, user.userName);
  }
}
```

这允许在非异步上下文中实例化两个对象，如果你可以访问这两个属性，或者在异步上下文中如果只有`id`可用。我们转换职责并完全从构造函数中删除`async`。

另一种技术是简单地忽略未初始化的状态。如果`userName`的状态对你的应用程序完全不相关，并且你只在需要时访问它，使用*definite assignment assertion*（感叹号）告诉 TypeScript 你将把这个属性视为已初始化：

```
class Account {
  userName!: string;
  state: State = "active";
  orders?: number[];

  constructor(public id: number) {
    fetch(`/api/getName?id=${id}`)
      .then((res) => res.json())
      .then((data: User) => (this.userName = data.userName));
  }
}
```

现在责任在你手上，有了感叹号你可以把 TypeScript 特定的语法称为不安全操作，包括运行时错误。

# 11.8 在类中使用这些类型

## 问题

你从基类扩展以重用功能，并且你的方法具有引用相同类实例的签名。你希望确保没有其他子类混入你的接口，但是你不想重写方法只是为了改变类型。

## 解决方案

使用`this`作为类型而不是实际的类类型。

## 讨论

在这个例子中，我们想使用类来模拟公告板软件中不同用户角色。我们从一个通用的`User`类开始，它通过用户 ID 进行标识，并具有打开主题的能力：

```
class User {
  #id: number;
  static #nextThreadId: number;

  constructor(id: number) {
    this.#id = id;
  }

  equals(user: User): boolean {
    return this.#id === user.#id;
  }

  async openThread(title: string, content: string): Promise<number> {
    const threadId = User.#nextThreadId++;
    await fetch("/createThread", {
      method: "POST",
      body: JSON.stringify({
        content,
        title,
        threadId,
      }),
    });
    return threadId;
  }
}
```

这个类还包含一个`equals`方法。在我们的代码库中的某个地方，我们需要确保两个用户引用是相同的，由于我们通过他们的 ID 来标识用户，我们可以轻松地比较数字。

`User`是所有用户的基类，因此如果我们添加具有更多特权的角色，我们可以轻松地继承基础`User`类。例如，`Admin`具有关闭主题的能力，并且它存储了一组其他我们可能在其他方法中使用的权限。

###### 注意

编程社区对继承是否是一种更好的技术存在很多争论，因为它的好处很少超过其缺点。尽管如此，JavaScript 的某些部分依赖于继承，比如 Web 组件。

由于我们从`User`继承，我们无需编写另一个`openThread`方法，我们可以重用相同的`equals`方法，因为所有管理员也是用户：

```
class Admin extends User {
  #privileges: string[];
  constructor(id: number, privileges: string[] = []) {
    super(id);
    this.#privileges = privileges;
  }

  async closeThread(threadId: number) {
    await fetch("/closeThread", {
      method: "POST",
      body: "" + threadId,
    });
  }
}
```

设置完我们的类之后，我们可以通过实例化正确的类来创建`User`和`Admin`类型的新对象。我们还可以调用`equals`方法来比较两个用户是否可能相同：

```
const user = new User(1);
const admin = new Admin(2);

console.log(user.equals(admin));
console.log(admin.equals(user));
```

有一件事令人困扰：比较的方向。当然，比较两个数字是可交换的；如果我们比较一个`user`和一个`admin`，这应该没问题，但是如果我们考虑周围的类和子类型，还是有改进的空间：

+   如果我们检查一个`user`是否等于一个`admin`，这是可以的，因为它可能获得特权。

+   如果我们希望一个`admin`等于一个`user`，这是令人怀疑的，因为更广泛的超类型具有更少的信息。

+   如果我们有`Admin`相邻的另一个`Moderator`子类，我们绝对不希望能够将它们作为它们在基类之外不共享属性的比较。

仍然，在当前`equals`的开发方式下，所有比较都可以工作。我们可以通过改变我们想要比较的类型来解决这个问题。我们首先用`User`标注了输入参数，但实际上我们想要比较*同类型的另一个实例*。有一个类型可以做到这一点，那就是`this`：

```
class User {
  // ...

  equals(user: this): boolean {
    return this.#id === user.#id;
  }
}
```

这与我们从函数中了解到的可以擦除的`this`参数不同，我们在 Recipe 2.7 中学习过，因为`this`参数类型允许我们在函数范围内设置`this`全局变量的具体类型。`this`类型是对方法所在类的引用。并且随着实现的变化而变化。因此，如果我们在`User`中用`this`注释一个`user`，它在从`User`继承的类中变成一个`Admin`，或者一个`Moderator`，等等。因此，`admin.equals`期望与之比较的是另一个`Admin`类；否则，我们会得到一个错误：

```
console.log(admin.equals(user));
//                       ^
// Argument of type 'User' is not assignable to parameter of type 'Admin'.
```

反过来也可以工作。因为`Admin`包含了所有`User`的属性（毕竟它是一个子类），我们可以轻松地比较`user.equals(admin)`。

`this`类型也可以用作返回类型。看看这个实现*构建器模式*的`OptionBuilder`：

```
class OptionBuilder<T = string | number | boolean> {
  #options: Map<string, T> = new Map();
  constructor() {}

  add(name: string, value: T): OptionBuilder<T> {
    this.#options.set(name, value);
    return this;
  }

  has(name: string) {
    return this.#options.has(name);
  }

  build() {
    return Object.fromEntries(this.#options);
  }
}
```

它是`Map`的一个轻量包装，允许我们设置键值对。它具有链式接口，这意味着在每个`add`调用之后，我们都会得到当前实例的返回，允许我们在`add`调用之后进行另一个`add`调用。注意我们用`OptionBuilder<T>`标注了返回类型：

```
const options = new OptionBuilder()
  .add("deflate", true)
  .add("compressionFactor", 10)
  .build();
```

现在我们正在创建一个继承自`OptionBuilder`并将可能元素类型设置为`string`的`StringOptionBuilder`。我们还添加了一个`safeAdd`方法，用于在写入之前检查是否已经设置了某个特定值，以避免覆盖先前的设置：

```
class StringOptionBuilder extends OptionBuilder<string> {
  safeAdd(name: string, value: string) {
    if (!this.has(name)) {
      this.add(name, value);
    }
    return this;
  }
}
```

当我们开始使用新的构建器时，我们发现如果作为第一步有`add`，我们就不能合理地使用`safeAdd`：

```
const languages = new StringOptionBuilder()
  .add("en", "English")
  .safeAdd("de", "Deutsch")
// ^
// Property 'safeAdd' does not exist on type 'OptionBuilder<string>'.(2339)
  .safeAdd("de", "German")
  .build();
```

TypeScript 告诉我们，在类型为`OptionBuilder<string>`的情况下，`safeAdd`不存在。这个函数去哪了？问题在于`add`有一个非常宽泛的注释。当然，`StringOptionBuilder`是`OptionBuilder<string>`的子类型，但是有了注释，我们失去了更窄类型的信息。解决方案？使用`this`作为返回类型：

```
class OptionBuilder<T = string | number | boolean> {
  // ...

  add(name: string, value: T): this {
    this.#options.set(name, value);
    return this;
  }
}
```

与前面的例子效果相同。在`OptionBuilder<T>`中，`this`变成了`OptionBuilder<T>`。在`StringBuilder`中，`this`变成了`StringBuilder`。如果你返回`this`并省略返回类型注释，`this`就成为了*推断*的返回类型。因此，明确使用`this`取决于你的偏好（参见 Recipe 2.1）。

# 11.9 编写装饰器

## 问题

你希望记录你的方法执行情况以进行遥测，但是对每个方法手动添加日志很麻烦。

## 解决方案

编写一个名为 `log` 的类方法装饰器来注解您的方法。

## 讨论

*装饰器*设计模式在埃里克·伽马等人（Addison-Wesley）的著名书籍《设计模式：可复用面向对象软件的元素》中有所描述，并描述了一种可以动态添加或覆盖某些行为的技术。

起初作为面向对象编程中自然出现的设计模式，如今变得如此流行，以至于支持面向对象特性的编程语言都添加了装饰器作为一种语言特性，并使用了特殊的语法。在 Java（称为*注解*）或 C#（称为*属性*）以及 JavaScript 中都可以看到其形式。

ECMAScript 关于装饰器的提案已经在提案阶段 3（准备实施阶段）中停留了相当长的时间，但在 2022 年达到了阶段 3，并且随着所有功能达到阶段 3，TypeScript 是第一个采纳新规范的工具之一。

###### 警告

在 TypeScript 中，装饰器已经存在很长时间，使用 `experimentalDecorators` 编译器标志。随着 TypeScript 5.0 的推出，原生的 ECMAScript 装饰器提案完全实现并且无需标志。实际的 ECMAScript 实现与原始设计在根本上有所不同，如果您在 TypeScript 5.0 之前开发过装饰器，它们将无法与新规范一起使用。请注意，打开 `experimentalDecorators` 标志会关闭 ECMAScript 原生装饰器。此外，关于类型，*lib.decorators.d.ts* 包含 ECMAScript 原生装饰器的所有类型信息，而 *lib.decorators.legacy.d.ts* 中的类型包含旧的类型信息。确保您的设置正确，并且不要从错误的定义文件中使用类型。

装饰器允许我们在类中几乎任何地方进行装饰。对于这个例子，我们希望从一个方法装饰器开始，允许我们记录方法调用的执行。

装饰器被描述为具有 *value* 和 *context* 的函数，这两者都取决于您想装饰的类元素的类型。这些装饰器函数返回另一个函数，在您自己的方法之前执行（或在字段初始化之前，或在访问器调用之前等）。

方法的一个简单的 `log` 装饰器可能如下所示：

```
function log(value: Function, context: ClassMethodDecoratorContext) {
  return function (this: any, ...args: any[]) {
    console.log(`calling ${context.name.toString()}`);
    return value.call(this, ...args);
  };
}

class Toggler {
  #toggled = false;

  @log
  toggle() {
    this.#toggled = !this.#toggled;
  }
}

const toggler = new Toggler();
toggler.toggle();
```

`log` 函数遵循原始[装饰器提案](https://oreil.ly/76JuE)中定义的 `ClassMethodDecorator` 类型：

```
type ClassMethodDecorator = (value: Function, context: {
  kind: "method";
  name: string | symbol;
  access: { get(): unknown };
  static: boolean;
  private: boolean;
  addInitializer(initializer: () => void): void;
}) => Function | void;
```

许多装饰器上下文类型可用。*lib.decorator.d.ts* 定义了以下装饰器：

```
type ClassMemberDecoratorContext =
    | ClassMethodDecoratorContext
    | ClassGetterDecoratorContext
    | ClassSetterDecoratorContext
    | ClassFieldDecoratorContext
    | ClassAccessorDecoratorContext
    ;

/**
 * The decorator context types provided to any decorator.
 */
type DecoratorContext =
    | ClassDecoratorContext
    | ClassMemberDecoratorContext
    ;
```

您可以从名称中准确地看出它们目标类的哪个部分。

请注意，我们还没有编写详细的类型。我们大多数时候使用 `any`，主要是因为类型可能变得非常复杂。如果我们想为所有参数添加类型，我们需要大量使用泛型：

```
function log<This, Args extends any[], Return>(
  value: (this: This, ...args: Args) => Return,
  context: ClassMethodDecoratorContext
): (this: This, ...args: Args) => Return {
  return function (this: This, ...args: Args) {
    console.log(`calling ${context.name.toString()}`);
    return value.call(this, ...args);
  };
}
```

泛型类型参数对于描述我们要传递的方法是必要的。我们想捕捉以下类型：

+   `This`是`this`参数类型的通用类型参数（参见 Recipe 2.7）。我们需要将`this`设置为在对象实例上下文中运行装饰器。

+   然后我们有方法的参数作为`Args`。正如我们在 Recipe 2.4 中学到的那样，一个方法或函数的参数可以描述为一个元组。

+   最后但同样重要的是`Return`类型参数。该方法需要返回特定类型的值，我们需要指定这一点。

有了这三者，我们能够以最通用的方式描述输入方法和输出方法，适用于所有类。我们可以使用泛型约束来确保我们的装饰器只在某些情况下工作，但对于`log`，我们希望能够记录每个方法调用。

###### 注意

在撰写本文时，TypeScript 中的 ECMAScript 装饰器还比较新。随着时间的推移，类型信息会变得更好，因此您获取的类型信息可能已经好得多。

我们还想要在`constructor`方法被调用之前记录我们的类字段及其初始值：

```
class Toggler {
  @logField #toggled = false;

  @log
  toggle() {
    this.#toggled = !this.#toggled;
  }
}
```

为此，我们创建另一个名为`logField`的装饰器，它作用于`ClassFieldDecoratorContext`。[装饰器提案](https://oreil.ly/76JuE)如下描述了用于类字段的装饰器：

```
type ClassFieldDecorator = (value: undefined, context: {
  kind: "field";
  name: string | symbol;
  access: { get(): unknown, set(value: unknown): void };
  static: boolean;
  private: boolean;
}) => (initialValue: unknown) => unknown | void;
```

注意*value*是`undefined`。初始值被传递给替换方法：

```
type FieldDecoratorFn = (val: any) => any;

function logField<Val>(
  value: undefined,
  context: ClassFieldDecoratorContext
): FieldDecoratorFn {
  return function (initialValue: Val): Val {
    console.log(`Initializing ${context.name.toString()} to ${initialValue}`);
    return initialValue;
  };
}
```

有一件事感觉不对劲。为什么我们需要为不同类型的成员使用不同的装饰器？我们的`log`装饰器难道不能处理所有情况吗？我们的装饰器在特定*装饰器上下文*中被调用，我们可以通过`kind`属性（我们在 Recipe 3.2 中看到的模式）来识别正确的上下文，因此编写一个根据上下文进行不同装饰器调用的`log`函数是非常简单的，对吧？

嗯，是的也不是。当然，有一个正确分支的包装器函数是正确的方法，但是正如我们所见，类型定义非常复杂。要找到能处理它们所有的*一个*函数签名几乎是不可能的，除非到处都用`any`。请记住：我们需要正确的函数签名类型；否则，装饰器将无法与类成员一起工作。

多个不同的函数签名只会引起*函数重载*。因此，我们不是为所有可能的装饰器找到一个函数签名，而是为*字段装饰器*、*方法装饰器*等创建重载。在这里，我们可以像单个装饰器一样对它们进行类型化。实现的函数签名采用`any`作为`value`，并将所有必需的装饰器上下文类型汇总到一个联合中，以便我们之后进行适当的区分检查：

```
function log<This, Args extends any[], Return>(
  value: (this: This, ...args: Args) => Return,
  context: ClassMethodDecoratorContext
): (this: This, ...args: Args) => Return;
function log<Val>(
  value: Val,
  context: ClassFieldDecoratorContext
): FieldDecoratorFn;
function log(
  value: any,
  context: ClassMethodDecoratorContext | ClassFieldDecoratorContext
) {
  if (context.kind === "method") {
    return logMethod(value, context);
  } else {
    return logField(value, context);
  }
}
```

而不是把所有实际代码都弄到`if`分支中，我们宁愿调用原始方法。如果您不想将`logMethod`或`logField`函数暴露出来，那么可以将它们放在一个模块中，只导出`log`。

###### 提示

有很多不同类型的装饰器，它们都有些许不同的各种字段。*lib.decorators.d.ts*中的类型定义非常好，但如果您需要更多信息，请查看[TC39 提案中的原始装饰器提案](https://oreil.ly/76JuE)。它不仅包含所有类型装饰器的广泛信息，还包含完整的 TypeScript 类型定义，从而完整地展示了整个画面。

我们还想做最后一件事：调整`logMethod`以在调用之前和之后都记录日志。对于普通方法来说，暂时存储返回值就很容易：

```
function log<This, Args extends any[], Return>(
  value: (this: This, ...args: Args) => Return,
  context: ClassMethodDecoratorContext
) {
  return function (this: This, ...args: Args) {
    console.log(`calling ${context.name.toString()}`);
    const val = value.call(this, ...args);
    console.log(`called ${context.name.toString()}: ${val}`);
    return val;
  };
}
```

但对于异步方法，事情变得更加有趣。调用异步方法会产生一个`Promise`。`Promise`本身可能已经执行完毕，或者执行被推迟到以后。这意味着如果我们继续使用之前的实现，*called*日志消息可能会在方法实际产生值之前出现。

作为一种解决方法，我们需要将日志消息链式化为`Promise`产生结果后的下一步。为此，我们需要检查方法是否实际上是一个`Promise`。JavaScript 的 Promises 很有趣，因为它们只需具有`then`方法就可以被等待。这是我们可以在辅助方法中检查的内容：

```
function isPromise(val: any): val is Promise<unknown> {
  return (
    typeof val === "object" &&
    val &&
    "then" in val &&
    typeof val.then === "function"
  );
}
```

有了这个，我们根据是否有`Promise`来决定是直接记录日志还是延迟记录：

```
function logMethod<This, Args extends any[], Return>(
  value: (this: This, ...args: Args) => Return,
  context: ClassMethodDecoratorContext
): (this: This, ...args: Args) => Return {
  return function (this: This, ...args: Args) {
    console.log(`calling ${context.name.toString()}`);
    const val = value.call(this, ...args);
    if (isPromise(val)) {
      val.then((p: unknown) => {
        console.log(`called ${context.name.toString()}: ${p}`);
        return p;
      });
    } else {
      console.log(`called ${context.name.toString()}: ${val}`);
    }

    return val;
  };
}
```

修饰器可以变得非常复杂，但最终它们是使 JavaScript 和 TypeScript 中的类更具表现力的有用工具。

^(1) C#和 TypeScript 都由 Microsoft 制作，Anders Hejlsberg 在这两种编程语言中都有很大的参与。
