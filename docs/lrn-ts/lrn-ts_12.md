# 第十章：泛型

> 您
> 
> 在类型系统中声明？
> 
> 全新的（类型化）世界！

到目前为止，您学到的所有类型语法都是用于在编写时完全知道它们的类型的类型。但有时，一段代码可能旨在根据调用方式与不同类型一起工作。

拿这个 `identity` 函数作为例子，在 JavaScript 中，它意味着接收任何可能类型的输入，并将相同的输入作为输出返回。您将如何描述其参数类型和返回类型？

```
function identity(input) {
    return input;
}

identity("abc");
identity(123);
identity({ quote: "I think your self emerges more clearly over time." });
```

我们可以将 `input` 声明为 `any`，但那么函数的返回类型也将是 `any`：

```
function identity(input: any) {
    return input;
}

let value = identity(42); // Type of value: any
```

鉴于 `input` 可以是任何输入，我们需要一种方式来表明 `input` 类型与函数返回的类型之间存在关系。TypeScript 使用 *泛型* 来捕获类型之间的关系。

在 TypeScript 中，诸如函数之类的构造物可以声明任意数量的泛型 *类型参数*：这些类型参数用作构造物中的类型，以表示每个实例中可以不同的某种类型。类型参数可以为构造物的每个实例提供不同的类型，称为 *类型参数*，但在该实例内部保持一致。

类型参数通常使用像`T`和`U`这样的单字母名称，或者像`Key`和`Value`这样的帕斯卡命名。在本章涵盖的所有结构中，泛型可以使用`<`和`>`括号声明，例如`someFunction<T>`或`SomeInterface<T>`。

# 泛型函数

函数可以通过在参数括号之前的尖括号内放置类型参数的别名来进行泛型化。该类型参数随后可用于参数类型注释、返回类型注释以及函数体内的类型注释。

下面版本的 `identity` 声明了一个类型参数 `T` 用于其 `input` 参数，这使得 TypeScript 能够推断函数的返回类型为 `T`。每次调用 `identity` 时，TypeScript 可以推断出不同的 `T` 类型：

```
function identity<T>(input: T) {
    return input;
}

const numeric = identity("me"); // Type: "me"
const stringy = identity(123); // Type: 123
```

箭头函数也可以是泛型的。它们的泛型声明也放置在其参数列表之前的 `(` 之前。

以下箭头函数在功能上与前面的声明相同：

```
const identity = <T>(input: T) => input;

identity(123); // Type: 123
```

###### 警告

泛型箭头函数的语法在 *.tsx* 文件中有一些限制，因为它与 JSX 语法冲突。参见第十三章，“配置选项”以获取解决方法，以及配置 JSX 和 React 支持。

通过这种方式向函数添加类型参数，可以使其能够在不同的输入下重复使用，同时保持类型安全，并避免使用`any`类型。

## 显式泛型调用类型

大多数情况下，在调用泛型函数时，TypeScript 将能够根据函数调用的方式推断出类型参数。例如，在前面示例中的 `identity` 函数中，TypeScript 的类型检查器使用提供给 `identity` 的参数来推断相应的函数参数类型参数。

不幸的是，与类成员和变量类型一样，有时从函数调用中没有足够的信息来告诉 TypeScript 其类型参数应该解析为什么。如果泛型结构提供了其类型参数未知的另一个泛型结构，这种情况通常会发生。

对于无法推断其类型参数的任何类型参数，TypeScript 将默认为 `unknown` 类型。

例如，下面的 `logWrapper` 函数接受一个带有参数类型设置为 `logWrapper` 的类型参数 `Input` 的回调函数。如果 `logWrapper` 被调用时带有明确声明其参数类型的回调函数，TypeScript 可以推断出类型参数。然而，如果参数类型是隐式的，TypeScript 将无法知道 `Input` 应该是什么类型：

```
function logWrapper<Input>(callback: (input: Input) => void) {
    return (input: Input) => {
        console.log("Input:", input);
        callback(input);
    };
}

// Type: (input: string) => void
logWrapper((input: string) => {
    console.log(input.length);
});

// Type: (input: unknown) => void
logWrapper((input) => {
    console.log(input.length);
    //                ~~~~~~
    // Error: Property 'length' does not exist on type 'unknown'.
});
```

为了避免默认为 `unknown`，函数可以以显式的泛型类型参数调用，明确告诉 TypeScript 应该将该类型参数解析为什么。TypeScript 将对泛型调用执行类型检查，以确保请求的参数与提供的类型参数匹配。

在这个例子中，之前看到的 `logWrapper` 被提供了一个显式的 `string` 作为其 `Input` 泛型。然后 TypeScript 可以推断出回调函数的 `input` 参数的泛型类型 `Input` 解析为 `string` 类型：

```
// Type: (input: string) => void
logWrapper<string>((input) => {
    console.log(input.length);
});

logWrapper<string>((input: boolean) => {
    //             ~~~~~~~~~~~~~~~~~~~~~~~
    // Argument of type '(input: boolean) => void' is not
    // assignable to parameter of type '(input: string) => void'.
    //   Types of parameters 'input' and 'input' are incompatible.
    //     Type 'string' is not assignable to type 'boolean'.
});
```

就像在变量上明确指定类型注解一样，泛型函数上也可以始终指定明确的类型参数，但通常并不是必需的。许多 TypeScript 开发人员通常只在需要时才指定它们。

下面的 `logWrapper` 使用明确指定了 `string` 作为类型参数和函数参数类型。两者都可以移除：

```
// Type: (input: string) => void
logWrapper<string>((input: string) => { /* ... */ });
```

用于指定类型参数的 `Name<Type>` 语法将与本章节中的其他泛型结构相同。

## 多个函数类型参数

函数可以定义任意数量的类型参数，用逗号分隔。泛型函数的每次调用可以为其类型参数解析其自己的一组值。

在这个例子中，`makeTuple` 声明了两个类型参数，并返回一个以只读元组的形式分别为一个，然后另一个的值为类型化的值：

```
function makeTuple<First, Second>(first: First, second: Second) {
    return [first, second] as const;
}

let tuple = makeTuple(true, "abc"); // Type of value: readonly [boolean, string]
```

请注意，如果函数声明了多个类型参数，调用该函数时必须明确声明泛型类型的全部或不含任何泛型类型。TypeScript 尚不支持仅推断泛型调用中的某些类型。

在这里，`makePair` 也接受两个类型参数，因此要么两者都不明确指定，要么都明确指定：

```
function makePair<Key, Value>(key: Key, value: Value) {
    return { key, value };
}

// Ok: neither type argument provided
makePair("abc", 123); // Type: { key: string; value: number }

// Ok: both type arguments provided
makePair<string, number>("abc", 123); // Type: { key: string; value: number }
makePair<"abc", 123>("abc", 123); // Type: { key: "abc"; value: 123 }

makePair<string>("abc", 123);
//       ~~~~~~
// Error: Expected 2 type arguments, but got 1.
```

###### 提示

尽量不要在任何通用结构中使用超过一个或两个类型参数。与运行时函数参数一样，使用越多，代码的可读性和理解性就越差。

# 泛型接口

接口也可以声明为通用的。它们遵循与函数类似的通用规则：它们可以在名称后的`<`和`>`之间声明任意数量的类型参数。该通用类型可以稍后在其声明的其他位置使用，例如在成员类型中。

下面的`Box`声明具有`T`类型参数作为属性。使用类型参数声明为`Box`的对象强制`inside: T`属性与该类型参数匹配：

```
interface Box<T> {
    inside: T;
}

let stringyBox: Box<string> = {
    inside: "abc",
};

let numberBox: Box<number> = {
    inside: 123,
}

let incorrectBox: Box<number> = {
    inside: false,
    // Error: Type 'boolean' is not assignable to type 'number'.
}
```

有趣的事实：内置的`Array`方法在 TypeScript 中被定义为通用接口！ `Array`使用类型参数`T`来表示数组中存储的数据类型。其`pop`和`push`方法大致如下：

```
interface Array<T> {
    // ...

    /**
 * Removes the last element from an array and returns it.
 * If the array is empty, undefined is returned and the array is not modified.
 */
    pop(): T | undefined;

    /**
 * Appends new elements to the end of an array,
 * and returns the new length of the array.
 * @param items new elements to add to the array.
 */
    push(...items: T[]): number;

    // ...
}
```

## 推断泛型接口类型

与通用函数一样，通用接口类型参数可以从使用中推断出。 TypeScript 将尽其所能从声明为接受通用类型的位置提供的值的类型推断类型参数。

此`getLast`函数声明了一个类型参数`Value`，然后用于其`node`参数。 TypeScript 可以根据传入参数的类型推断出`Value`。当推断出的类型参数与值的类型不匹配时，甚至可以报告类型错误。允许向`getLast`提供不包含`next`属性的对象，或其推断出的`Value`类型参数与给定对象的`value`和`next.value`不匹配是允许的，但是，这是一个类型错误：

```
interface LinkedNode<Value> {
    next?: LinkedNode<Value>;
    value: Value;
}

function getLast<Value>(node: LinkedNode<Value>): Value {
    return node.next ? getLast(node.next) : node.value;
}

// Inferred Value type argument: Date
let lastDate = getLast({
    value: new Date("09-13-1993"),
});

// Inferred Value type argument: string
let lastFruit = getLast({
    next: {
        value: "banana",
    },
    value: "apple",
});

// Inferred Value type argument: number
let lastMismatch = getLast({
    next: {
        value: 123
    },
    value: false,
//  ~~~~~
// Error: type 'boolean' is not assignable to type 'number'.
});
```

注意，如果接口声明了类型参数，则任何引用该接口的类型注释都必须提供相应的类型参数。在这里，对`CrateLike`的使用不正确，因为没有包含类型参数：

```
interface CrateLike<T> {
    contents: T;
}

let missingGeneric: CrateLike = {
    //              ~~~~~~~~~
    // Error: Generic type 'Crate<T>' requires 1 type argument(s).
    inside: "??"
};
```

在本章的后面，我将展示如何为类型参数提供默认值，以避免此要求。

# 泛型类

类，就像接口一样，也可以声明任意数量的类型参数，以后可以在其成员上使用。类的每个实例可以具有不同的类型参数集。

此`Secret`类声明了`Key`和`Value`类型参数，然后将它们用于成员属性、构造函数参数类型以及方法的参数和返回类型：

```
class Secret<Key, Value> {
    key: Key;
    value: Value;

    constructor(key: Key, value: Value) {
        this.key = key;
        this.value = value;
    }

    getValue(key: Key): Value | undefined {
        return this.key === key
            ? this.value
            : undefined;
    }
}

const storage = new Secret(12345, "luggage"); // Type: Secret<number, string>

storage.getValue(1987); // Type: string | undefined
```

与通用接口一样，使用类的类型注释必须告诉 TypeScript 该类上的任何通用类型是什么。在本章的后面，我将展示如何为类提供类型参数的默认值，以避免此要求。

## 显式通用类类型

实例化泛型类遵循与调用泛型函数相同的类型参数推断规则。如果类型参数可以从传递给类构造函数的参数的类型推断出来，例如之前的 `new Secret(12345, "luggage")`，TypeScript 将使用推断类型。否则，如果无法从传递给其构造函数的参数推断出类类型参数，则类型参数将默认为 `unknown`。

此 `CurriedCallback` 类声明了一个接受泛型函数的构造函数。如果泛型函数具有已知类型——例如来自显式类型参数类型注释——那么类实例的 `Input` 类型参数可以由此知情。否则，类实例的 `Input` 类型参数将默认为 `unknown`：

```
class CurriedCallback<Input> {
    #callback: (input: Input) => void;

    constructor(callback: (input: Input) => void) {
        this.#callback = (input: Input) => {
            console.log("Input:", input);
            callback(input);
        };
    }

    call(input: Input) {
        this.#callback(input);
    }
}

// Type: CurriedCallback<string>
new CurriedCallback((input: string) => {
    console.log(input.length);
});

// Type: CurriedCallback<unknown>
new CurriedCallback((input) => {
    console.log(input.length);
    //                ~~~~~~
    // Error: Property 'length' does not exist on type 'unknown'.
});
```

类实例也可以通过与其他泛型函数调用相同的方式提供显式的类型参数来避免默认为 `unknown`。

在这里，之前的 `CurriedCallback` 现在为其 `Input` 类型参数提供了一个显式的 `string`，因此 TypeScript 可以推断出回调的 `Input` 类型参数解析为 `string`：

```
// Type: CurriedCallback<string>
new CurriedCallback<string>((input) => {
    console.log(input.length);
});

new CurriedCallback<string>((input: boolean) => {
    //                       ~~~~~~~~~~~~~~~~~~~~~~
    // Argument of type '(input: boolean) => void' is not
    // assignable to parameter of type '(input: string) => void'.
    //   Types of parameters 'input' and 'input' are incompatible.
    //     Type 'string' is not assignable to type 'boolean'.
});
```

## 扩展泛型类

泛型类可以作为 `extends` 关键字后的基类使用。TypeScript 不会尝试从使用中推断基类的类型参数。任何没有默认值的类型参数都需要使用显式类型注释来指定。

下面的 `SpokenQuote` 类为其基类 `Quote<T>` 提供了 `string` 作为 `T` 类型参数：

```
class Quote<T> {
    lines: T;

    constructor(lines: T) {
        this.lines = lines;
    }
}

class SpokenQuote extends Quote<string[]> {
    speak() {
        console.log(this.lines.join("\n"));
    }
}

new Quote("The only real failure is the failure to try.").lines; // Type: string
new Quote([4, 8, 15, 16, 23, 42]).lines; // Type: number[]

new SpokenQuote([
    "Greed is so destructive.",
    "It destroys everything",
]).lines; // Type: string[]

new SpokenQuote([4, 8, 15, 16, 23, 42]);
//              ~~~~~~~~~~~~~~~~~~~~~~
// Error: Argument of type 'number' is not
// assignable to parameter of type 'string'.
```

泛型派生类可以通过将它们自己的类型参数传递给它们的基类来传递自己的类型参数。类型名称不必匹配；仅仅是为了好玩，这个 `AttributedQuote` 将一个名为 `Value` 的不同命名的类型参数传递给基类 `Quote<T>`：

```
class AttributedQuote<Value> extends Quote<Value> {
    speaker: string

    constructor(value: Value, speaker: string) {
        super(value);
        this.speaker = speaker;
    }
}

// Type: AttributedQuote<string>
// (extending Quote<string>)
new AttributedQuote(
    "The road to success is always under construction.",
    "Lily Tomlin",
);
```

## 实现泛型接口

泛型类也可以通过为其提供任何必要的类型参数来实现泛型接口。这与扩展泛型基类类似：基接口上的任何类型参数必须由类声明。

这里，`MoviePart` 类将 `ActingCredit` 接口的 `Role` 类型参数指定为 `string`。`IncorrectExtension` 类引发类型投诉，因为它的 `role` 类型为 `boolean`，尽管它将 `string[]` 作为 `ActingCredit` 的类型参数提供：

```
interface ActingCredit<Role> {
    role: Role;
}

class MoviePart implements ActingCredit<string> {
    role: string;
    speaking: boolean;

    constructor(role: string, speaking: boolean) {
        this.role = role;
        this.speaking = speaking;
    }
}

const part = new MoviePart("Miranda Priestly", true);

part.role; // Type: string

class IncorrectExtension implements ActingCredit<string> {
    role: boolean;
    //    ~~~~~~~
    // Error: Property 'role' in type 'IncorrectExtension' is not
    // assignable to the same property in base type 'ActingCredit<string>'.
    //   Type 'boolean' is not assignable to type 'string'.
}
```

## 方法泛型

类方法可以声明它们自己的泛型类型，与它们的类实例分开。对泛型类方法的每次调用可能对其类型参数的每个类型参数使用不同的类型参数。

此泛型 `CreatePairFactory` 类声明了一个 `Key` 类型，并包含一个还声明了一个单独的 `Value` 泛型类型的 `createPair` 方法。`createPair` 的返回类型然后被推断为 `{ key: Key, value: Value }`：

```
class CreatePairFactory<Key> {
    key: Key;

    constructor(key: Key) {
        this.key = key;
    }

    createPair<Value>(value: Value) {
        return { key: this.key, value };
    }
}

// Type: CreatePairFactory<string>
const factory = new CreatePairFactory("role");

// Type: { key: string, value: number }
const numberPair = factory.createPair(10);

// Type: { key: string, value: string }
const stringPair = factory.createPair("Sophie");
```

## 静态类泛型

类的静态成员与实例成员是分离的，并且不与类的任何特定实例关联。它们无法访问任何类实例或特定于任何类实例的类型信息。因此，虽然静态类方法可以声明它们自己的类型参数，但无法访问类上声明的任何类型参数。

在这里，`BothLogger`类为其`instanceLog`方法声明了一个`OnInstance`类型参数，为其静态`staticLog`方法声明了一个单独的`OnStatic`类型参数。静态方法无法访问实例`OnInstance`，因为`OnInstance`是为类实例声明的。

```
class BothLogger<OnInstance> {
    instanceLog(value: OnInstance) {
        console.log(value);
        return value;
    }

    static staticLog<OnStatic>(value: OnStatic) {
        let fromInstance: OnInstance;
        //                ~~~~~~~~~~
        // Error: Static members cannot reference class type arguments.

        console.log(value);
        return value;
    }
}

const logger = new BothLogger<number[]>;
logger.instanceLog([1, 2, 3]); // Type: number[]

// Inferred OnStatic type argument: boolean[]
BothLogger.staticLog([false, true]);

// Explicit OnStatic type argument: string
BothLogger.staticLog<string>("You can't change the music of your soul.");
```

# 泛型类型别名

TypeScript 中的最后一个可以使用类型参数泛化的构造是类型别名。每个类型别名可以给予任意数量的类型参数，例如这个`Nullish`类型接收一个`T`：

```
type Nullish<T> = T | null | undefined;
```

泛型类型别名通常与函数一起使用，以描述泛型函数的类型：

```
type CreatesValue<Input, Output> = (input: Input) => Output;

// Type: (input: string) => number
let creator: CreatesValue<string, number>;

creator = text => text.length; // Ok

creator = text => text.toUpperCase();
//                ~~~~~~~~~~~~~~~~~~
// Error: Type 'string' is not assignable to type 'number'.
```

## 泛型判别联合

我在第四章，“对象”中提到过，判别联合是 TypeScript 中我最喜欢的功能，因为它们精美地结合了 JavaScript 的一种常见优雅模式与 TypeScript 的类型缩小。我最喜欢使用判别联合的方式是添加类型参数以创建一个通用的“结果”类型，该类型代表具有数据的成功结果或带有错误的失败结果。

此`Result`泛型类型具有必须用于将结果缩小到成功或失败的`succeeded`判别标志。这意味着任何返回`Result`的操作都可以指示错误或数据结果，并确保消费者需要检查结果是否成功：

```
type Result<Data> = FailureResult | SuccessfulResult<Data>;

interface FailureResult {
    error: Error;
    succeeded: false;
}

interface SuccessfulResult<Data> {
    data: Data;
    succeeded: true;
}

function handleResult(result: Result<string>) {
    if (result.succeeded) {
        // Type of result: SuccessfulResult<string>
        console.log(`We did it! ${result.data}`);
    } else {
        // Type of result: FailureResult
        console.error(`Awww... ${result.error}`);
    }

    result.data;
    //     ~~~~
    // Error: Property 'data' does not exist on type 'Result<string>'.
    //   Property 'data' does not exist on type 'FailureResult'.
}
```

综合起来，泛型类型和判别类型提供了一种出色的方式来建模可重用的类型，如`Result`。

# 通用修饰符

TypeScript 包含的语法允许您修改泛型类型参数的行为。

## 泛型默认值

到目前为止，我已经说明了如果在类型注释中使用泛型类型或作为类的基础（`extends`或`implements`），则必须为每个类型参数提供类型参数。您可以通过在类型参数声明后放置一个`=`符号，然后跟一个默认类型来避免显式提供类型参数。默认值将用于任何后续类型，其中未显式声明类型参数并且无法推断出来。

在这里，`Quote`接口接收一个`T`类型参数，如果未提供，则默认为`string`。`explicit`变量显式设置`T`为`number`，而`implicit`和`mismatch`均解析为`string`：

```
interface Quote<T = string> {
    value: T;
}

let explicit: Quote<number> = { value: 123 };

let implicit: Quote = { value: "Be yourself. The world worships the original." };

let mismatch: Quote = { value: 123 };
//                                     ~~~
// Error: Type 'number' is not assignable to type 'string'.
```

类型参数也可以默认为同一声明中较早的类型参数。由于每个类型参数为声明引入一个新类型，因此它们可作为该声明中后续类型参数的默认值。

`KeyValuePair` 类型可以具有不同类型的 `Key` 和 `Value` 泛型，但默认保持它们相同 — 虽然因为 `Key` 没有默认值，它仍然需要可推断或提供：

```
interface KeyValuePair<Key, Value = Key> {
    key: Key;
    value: Value;
}

// Type: KeyValuePair<string, string>
let allExplicit: KeyValuePair<string, number> = {
    key: "rating",
    value: 10,
};

// Type: KeyValuePair<string>
let oneDefaulting: KeyValuePair<string> = {
    key: "rating",
    value: "ten",
};

let firstMissing: KeyValuePair = {
    //            ~~~~~~~~~~~~
    // Error: Generic type 'KeyValuePair<Key, Value>'
    // requires between 1 and 2 type arguments.
    key: "rating",
    value: 10,
};
```

所有默认类型参数在其声明列表中必须放在最后，类似于默认函数参数。没有默认值的泛型类型不能跟在有默认值的泛型类型之后。

在此处，`inTheEnd` 是允许的，因为所有没有默认值的泛型类型在具有默认值的泛型类型之前。`inTheMiddle` 是个问题，因为没有默认值的泛型类型跟在有默认值的类型后面：

```
function inTheEnd<First, Second, Third = number, Fourth = string>() {} // Ok

function inTheMiddle<First, Second = boolean, Third = number, Fourth>() {}
//                                                         // ~~~~~~
// Error: Required type parameters may not follow optional type parameters.
```

# 受约束的泛型类型

默认情况下，泛型类型可以赋予世界上任何类型：类、接口、基本类型、联合类型，你名字它。然而，某些函数只能与有限集合的类型一起使用。

TypeScript 允许一个类型参数声明自己需要 *扩展* 一种类型：这意味着它只允许别名为可分配给该类型的类型。约束类型参数的语法是在类型参数名称之后放置 `extends` 关键字，然后是要将其约束为的类型。

例如，通过创建一个 `WithLength` 接口来描述任何具有 `length: number` 属性的对象，我们可以让我们的泛型函数接受任何具有 `length` 属性的类型作为其 `T` 泛型。字符串、数组，甚至只是具有 `length: number` 属性的对象，都是允许的，而像 `Date` 这样缺少数值 `length` 的类型形状会导致类型错误：

```
interface WithLength {
    length: number;
}

function logWithLength<T extends WithLength>(input: T) {
    console.log(`Length: ${input.length}`);
    return input;
}

logWithLength("No one can figure out your worth but you."); // Type: string
logWithLength([false, true]); // Type: boolean[]
logWithLength({ length: 123 }); // Type: { length: number }

logWithLength(new Date());
//            ~~~~~~~~~~
// Error: Argument of type 'Date' is not
// assignable to parameter of type 'WithLength'.
//   Property 'length' is missing in type
//   'Date' but required in type 'WithLength'.
```

我将在 第十五章，“类型操作” 中详细介绍您可以使用泛型执行的更多类型操作。

## `keyof` 和受约束的类型参数

在 第九章，“类型修饰符” 中引入的 `keyof` 运算符也与受约束的类型参数非常配合。使用 `extends` 和 `keyof` 结合允许一个类型参数被限制为前一个类型参数的键。这也是指定泛型类型键的唯一方法。

以流行库 Lodash 的 `get` 方法的简化版本为例。它接受一个被定义为 `T` 类型的容器值，以及 `T` 中某个键的 `key` 名称，用于从 `container` 中检索。因为 `Key` 类型参数被限制为 `T` 的 `keyof`，TypeScript 知道此函数被允许返回 `T[Key]`：

```
function get<T, Key extends keyof T>(container: T, key: Key) {
    return container[key];
}

const roles = {
    favorite: "Fargo",
    others: ["Almost Famous", "Burn After Reading", "Nomadland"],
};

const favorite = get(roles, "favorite"); // Type: string
const others = get(roles, "others"); // Type: string[]

const missing = get(roles, "extras");
//                         ~~~~~~~~
// Error: Argument of type '"extras"' is not assignable
// to parameter of type '"favorite" | "others"'.
```

如果没有 `keyof`，将无法正确地为泛型 `key` 参数设置类型。

请注意前面示例中 `Key` 类型参数的重要性。如果仅提供 `T` 作为类型参数，并且允许 `key` 参数为 `T` 的任何 `keyof`，则返回类型将是 `Container` 中所有属性值的联合类型。这种不太具体的函数声明并未告知 TypeScript 每次调用可以通过类型参数具有特定的 `key`：

```
function get<T>(container: T, key: keyof T) {
    return container[key];
}

const roles = {
    favorite: "Fargo",
    others: ["Almost Famous", "Burn After Reading", "Nomadland"],
};

const found = get(roles, "favorite"); // Type: string | string[]
```

在编写泛型函数时，务必了解参数类型依赖于前一个参数类型的情况。在这些情况下，通常需要使用约束类型参数以正确地指定参数类型。

# Promises

现在你已经看到了泛型的工作原理，终于是时候讨论现代 JavaScript 的核心特性之一：Promises！回顾一下，在 JavaScript 中，Promise 表示可能仍处于挂起状态的某些操作，比如网络请求。每个 Promise 都提供了方法来注册回调函数，以便在待定操作“解析”（成功完成）或“拒绝”（抛出错误）时调用。

一种`Promise`在任意值类型上表示类似操作的能力，是 TypeScript 泛型的天然适用。`Promise`在 TypeScript 类型系统中以单一类型参数表示最终解析的值。

## 创建 Promises

在 TypeScript 中，`Promise`构造函数被定义为接受一个单一参数的类型。该参数的类型依赖于泛型`Promise`类声明的类型参数。简化形式大致如下：

```
class PromiseLike<Value> {
    constructor(
        executor: (
            resolve: (value: Value) => void,
            reject: (reason: unknown) => void,
        ) => void,
    ) { /* ... */ }
}
```

创建一个打算最终解析为值的 Promise 通常需要显式声明 Promise 的类型参数。如果没有明确的泛型类型参数，TypeScript 默认将参数类型视为`unknown`。通过显式提供 Promise 构造函数的类型参数，TypeScript 可以理解结果 Promise 实例的解析类型：

```
// Type: Promise<unknown>
const resolvesUnknown = new Promise((resolve) => {
    setTimeout(() => resolve("Done!"), 1000);
});

// Type: Promise<string>
const resolvesString = new Promise<string>((resolve) => {
    setTimeout(() => resolve("Done!"), 1000);
});
```

`Promise`的泛型`.then`方法引入了一个新的类型参数，表示返回的 Promise 的解析值。

例如，以下代码创建了一个`textEventually` Promise，在一秒后解析为`string`值，以及一个`lengthEventually`，在额外等待一秒后解析为`number`：

```
// Type: Promise<string>
const textEventually = new Promise<string>((resolve) => {
    setTimeout(() => resolve("Done!"), 1000);
});

// Type: Promise<number>
const lengthEventually = textEventually.then((text) => text.length)
```

## 异步函数

在 JavaScript 中声明带有`async`关键字的任何函数都会返回一个`Promise`。如果 JavaScript 中`async`函数返回的值不是 Thenable（具有`.then()`方法的对象；实际上几乎总是一个 Promise），则会像调用`Promise.resolve`一样将其包装在`Promise`中。TypeScript 识别这一点，并推断`async`函数的返回类型始终为`Promise`，无论返回的值是什么。

在这里，`lengthAfterSecond`直接返回一个`Promise<number>`，而`lengthImmediately`被推断为返回`Promise<number>`，因为它是`async`并直接返回`number`：

```
// Type: (text: string) => Promise<number>
async function lengthAfterSecond(text: string) {
    await new Promise((resolve) => setTimeout(resolve, 1000))
    return text.length;
}

// Type: (text: string) => Promise<number>
async function lengthImmediately(text: string) {
    return text.length;
}
```

任何在`async`函数上手动声明的返回类型都必须始终是`Promise`类型，即使函数在实现中没有明确提到 Promises：

```
// Ok
async function givesPromiseForString(): Promise<string> {
    return "Done!";
}

async function givesString(): string {
    //                        ~~~~~~
    // Error: The return type of an async function
    // or method must be the global Promise<T> type.
    return "Done!";
}
```

# 正确使用泛型

就像本章早期的`Promise<Value>`实现一样，尽管泛型可以在代码中描述类型给予我们很大的灵活性，但它们很快就会变得相当复杂。 TypeScript 的最佳实践通常是仅在必要时使用泛型，并清楚地说明它们的用途。

###### 警告

在 TypeScript 中，大多数你编写的代码不应该过度使用泛型以至于令人困惑。然而，对于实用程序库的类型，特别是通用模块，有时可能需要大量使用它们。理解泛型特别有助于能够有效地使用这些实用程序类型。

## 泛型的黄金法则

一个快速的测试，可以帮助确定是否需要一个类型参数是它应该至少用两次。泛型描述类型之间的关系，因此，如果泛型类型参数只出现在一个地方，它不可能定义多个类型之间的关系。

每个函数类型参数应该用于一个参数，然后也用于至少一个其他参数和/或函数的返回类型。

例如，这个`logInput`函数仅使用其`Input`类型参数一次，来声明其`input`参数：

```
function logInput<Input extends string>(input: Input) {
    console.log("Hi!", input);
}
```

不像本章早期的`identify`函数那样，`logInput`不对其类型参数做任何操作，如返回或声明更多参数。因此，声明`Input`类型参数没有多少用处。我们可以在不声明它的情况下重写`logInput`：

```
function logInput(input: string) {
    console.log("Hi!", input);
}
```

*《Effective TypeScript》*（丹·范德坎，O'Reilly，2019）包含了几条关于如何使用泛型的极好建议，其中包括一节名为“泛型的黄金法则”。我强烈推荐阅读*《Effective TypeScript》*，特别是如果你在代码中发现自己花费了大量时间与泛型纠结的话。

## 泛型命名约定

许多语言（包括 TypeScript）的类型参数的标准命名约定是默认将第一个类型参数称为“T”（表示“type”或“template”），如果存在后续类型参数，则称为“U”、“V”等。

如果关于类型参数如何使用的上下文信息已知，则惯例有时会扩展到使用术语的第一个字母来表示其用途：例如，状态管理库可能会将通用状态称为“S”。数据结构中的“K”和“V”通常指键和值。

不幸的是，用一个字母命名类型参数可能会和用一个字符命名函数或变量一样令人困惑：

```
// What on earth are L and V?!
function labelBox<L, V>(l: L, v: V) { /* ... */ }
```

当单个字母`T`的泛型意图不明确时，最好使用描述性的泛型类型名称，以指示类型的用途：

```
// Much more clear.
function labelBox<Label, Value>(label: Label, value: Value) { /* ... */ }
```

当一个结构有多个类型参数，或者单个类型参数的目的不明确时，请考虑使用完整的名称以提高可读性，而不是使用单个字母的缩写。

# 总结

在本章中，通过允许它们使用类型参数，你使类、函数、接口和类型别名变得“通用”：

+   使用类型参数表示在结构的不同用法之间的不同类型

+   在调用泛型函数时提供显式或隐式的类型参数

+   使用泛型接口来表示通用对象类型

+   向类添加类型参数，以及这如何影响它们的类型

+   在类型别名中添加类型参数，特别是在有歧义的类型联合中

+   修改泛型类型参数的默认值 (`=`) 和约束 (`extends`)

+   Promises 和 `async` 函数如何使用泛型来表示异步数据流

+   泛型的最佳实践，包括它们的黄金法则和命名约定

因此，本书的 *特性* 部分到此结束。恭喜：你现在了解了 TypeScript 类型系统中大多数项目的最重要的语法和类型检查特性！

接下来的章节 *用法* 将介绍如何配置 TypeScript 在你的项目中运行，如何与外部依赖交互，并调整其类型检查和生成的 JavaScript。这些是在你自己的项目中使用 TypeScript 的重要特性。

在 TypeScript 语法中还有一些其他的杂项类型操作。你不需要完全理解它们就可以在大多数 TypeScript 项目中工作，但它们很有趣，也很有用。我把它们放在了第 IV 部分，“额外学分”之后的第 III 部分，“用法”中，如果你有时间，可以当作一个有趣的小礼物。

###### 提示

现在你已经完成了本章的阅读，可以在[*https://learningtypescript.com/generics*](https://learningtypescript.com/generics)上练习所学内容。

> 泛型为何会激怒开发者？
> 
> 它们总是在输入参数。
