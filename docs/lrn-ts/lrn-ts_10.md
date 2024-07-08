# 第八章：类

> 一些功能开发人员
> 
> 尽量*永远*不使用类
> 
> 对我来说太强烈了

在 TypeScript 创建和发布于 2010 年代初期的 JavaScript 世界与今天大不相同。例如箭头函数和`let`/`const`变量等功能，这些功能后来在 ES2015 中标准化，当时还只是遥远的希望。Babel 离第一个提交还有几年时间；其前身工具如 Traceur，将新的 JavaScript 语法转换为旧的，尚未完全普及。

TypeScript 早期的市场营销和功能设置是针对那个世界的。除了类型检查之外，还强调了其转译器，以类作为一个常见的例子。如今，TypeScript 的类支持只是支持所有 JavaScript 语言特性之一的一部分。TypeScript 既不鼓励也不阻止类的使用或任何其他流行的 JavaScript 模式。

# 类方法

TypeScript 通常将方法理解为独立函数。参数类型默认为`any`，除非给定类型或默认值；调用方法需要接受适当数量的参数；如果函数不是递归的话，通常可以推断返回类型。

此代码片段定义了一个`Greeter`类，其中包含一个接受类型为`number`的单个必需参数的`greet`类方法：

```
class Greeter {
    greet(name: string) {
        console.log(`${name}, do your stuff!`);
    }
}

new Greeter().greet("Miss Frizzle"); // Ok

new Greeter().greet();
//            ~~~~~
// Error: Expected 1 arguments, but got 0.
```

类构造函数在其参数方面与典型的类方法一样对待。TypeScript 将执行类型检查，以确保方法调用提供了正确数量和正确类型的参数。

这个`Greeted`构造函数也期望提供`message: string`参数：

```
class Greeted {
    constructor(message: string) {
        console.log(`As I always say: ${message}!`);
    }
}

new Greeted("take chances, make mistakes, get messy");

new Greeted();
// Error: Expected 1 arguments, but got 0.
```

我将在本章的子类上下文中涵盖构造函数。

# 类属性

要在 TypeScript 中从类的属性读取或写入，必须在类中明确声明它们。类属性使用与接口相同的语法声明：它们的名称后面可以选择跟上类型注释。

TypeScript 不会试图从构造函数中的赋值推断出类可能存在的成员。

在这个例子中，`destination`允许在`FieldTrip`类的实例上分配和访问，因为它被明确声明为`string`。在构造函数中的`this.nonexistent`赋值是不允许的，因为该类没有声明`nonexistent`属性：

```
class FieldTrip {
    destination: string;

    constructor(destination: string) {
        this.destination = destination; // Ok
        console.log(`We're going to ${this.destination}!`);

        this.nonexistent = destination;
        //   ~~~~~~~~~~~
        // Error: Property 'nonexistent' does not exist on type 'FieldTrip'.
    }
}
```

明确声明类属性允许 TypeScript 快速理解哪些实例中可以存在或不存在。稍后，在使用类实例时，TypeScript 利用这一理解，在代码尝试访问未知存在的类实例成员时，例如在本段续写中的`trip.nonexistent`，会给出类型错误：

```
const trip = new FieldTrip("planetarium");

trip.destination; // Ok

trip.nonexistent;
//   ~~~~~~~~~~~
// Error: Property 'nonexistent' does not exist on type 'FieldTrip'.
```

## 函数属性

让我们简要回顾一些 JavaScript 方法作用域和语法基础知识，因为如果您不习惯，它们可能会让人惊讶。JavaScript 包含两种语法用于声明类的成员为可调用函数：*方法* 和 *属性*。

我已经展示了在成员名称后加括号的方法，例如 `myFunction() {}`。方法的方法会将函数分配给类的原型，因此所有类实例使用相同的函数定义。

此 `WithMethod` 类声明了一个 `myMethod` 方法，所有实例均能引用：

```
class WithMethod {
    myMethod() {}
}

new WithMethod().myMethod === new WithMethod().myMethod; // true
```

另一种语法是声明一个其值恰好为函数的属性。这会为类的每个实例创建一个新的函数，对于箭头函数 `() =>`，这可以保证 `this` 的作用域始终指向类实例（以时间和内存开销换取）。

此 `WithProperty` 类包含一个名为 `myProperty`，类型为 `() => void` 的单一属性，每个类实例将重新创建该属性：

```
class WithProperty {
    myProperty: () => {}
}

new WithMethod().myProperty === new WithMethod().myProperty; // false
```

函数属性可以使用与类方法和独立函数相同的语法提供参数和返回类型。毕竟，它们是分配给类成员的值，其值恰好是一个函数。

此 `WithPropertyParameters` 类具有类型为 `(input: string) => number` 的 `takesParameters` 属性：

```
class WithPropertyParameters {
    takesParameters = (input: boolean) => input ? "Yes" : "No";
}

const instance = new WithPropertyParameters();

instance.takesParameters(true); // Ok

instance.takesParameters(123);
//                       ~~~
// Error: Argument of type 'number' is not
// assignable to parameter of type 'boolean'.
```

## 初始化检查

启用严格的编译器设置后，TypeScript 将检查每个声明的属性，确保其类型不包括 `undefined`，在构造函数中被赋值。这种严格的初始化检查非常有用，因为它可以防止代码意外地忘记为类属性赋值。

下面的 `WithValue` 类没有为其 `unused` 属性赋值，这在 TypeScript 中被识别为类型错误：

```
class WithValue {
    immediate = 0; // Ok
    later: number; // Ok (set in the constructor)
    mayBeUndefined: number | undefined; // Ok (allowed to be undefined)

    unused: number;
    // Error: Property 'unused' has no initializer
    // and is not definitely assigned in the constructor.

    constructor() {
        this.later = 1;
    }
}
```

如果没有严格的初始化检查，可能允许类实例访问一个可能为 `undefined` 的值，尽管类型系统说它不可能。

如果没有发生严格的初始化检查，这个例子将编译通过，但生成的 JavaScript 代码在运行时会崩溃：

```
class MissingInitializer {
    property: string;
}

new MissingInitializer().property.length;
// TypeError: Cannot read property 'length' of undefined
```

十亿美元的错误再次发生！

使用 TypeScript 的 `strictPropertyInitialization` 编译器选项配置严格的属性初始化检查详见第十二章 “使用 IDE 功能”。

### 明确分配的属性

尽管严格的初始化检查大多数时候很有用，您可能会遇到一些情况，其中一个类属性被有意允许在类构造函数后未分配。如果您确信某个属性不应该被应用严格的初始化检查，您可以在其名称后加上 `!` 来禁用检查。这样做断言给 TypeScript，该属性在第一次使用前会被赋值为非 `undefined` 的值。

这个`ActivitiesQueue`类可以在任意次数中单独重新初始化，因此它的`pending`属性必须用`!`断言：

```
class ActivitiesQueue {
    pending!: string[]; // Ok

    initialize(pending: string[]) {
        this.pending = pending;
    }

    next() {
        return this.pending.pop();
    }
}

const activities = new ActivitiesQueue();

activities.initialize(['eat', 'sleep', 'learn'])
activities.next();
```

###### 警告

需要在类属性上禁用严格的初始化检查通常是代码设置方式不适合类型检查的标志。不要为了属性添加`!`断言并降低类型安全性，考虑重构类以不再需要断言。

## 可选属性

类似接口，TypeScript 中的类可以通过在声明名称后添加`?`将属性声明为可选。可选属性的行为与类型为包括`| undefined`的联合的属性大致相同。如果它们在构造函数中没有被明确设置，严格的初始化检查不会介意。

这个`OptionalProperty`类将其`property`标记为可选，因此允许在类构造函数中不分配它：

```
class MissingInitializer {
    property?: string;
}

new MissingInitializer().property?.length; // Ok

new MissingInitializer().property.length;
// Error: Object is possibly 'undefined'.
```

## 只读属性

与接口类似，TypeScript 中的类可以通过在声明名称前添加`readonly`关键字将属性声明为只读。`readonly`关键字完全存在于类型系统中，并且在编译为 JavaScript 时会被移除。

声明为`readonly`的属性只能在声明它们或构造函数中分配初始值。任何其他位置 —— 包括类本身的方法 —— 只能从属性中读取，而不能写入它们。

在这个例子中，`Quote`类的`text`属性在构造函数中被赋值，但其他用法会导致类型错误：

```
class Quote {
    readonly text: string;

    constructor(text: string) {
        this.text = ;
    }

    emphasize() {
        this.text += "!";
        //   ~~~~
        // Error: Cannot assign to 'text' because it is a read-only property.
    }
}

const quote = new Quote(
    "There is a brilliant child locked inside every student."
);

Quote.text = "Ha!";
// Error: Cannot assign to 'text' because it is a read-only property.
```

###### 警告

外部使用你的代码的用户，比如你发布的任何 npm 包的消费者，可能不会尊重`readonly`修饰符 —— 特别是如果他们正在编写 JavaScript 并且没有类型检查。如果你需要真正的只读保护，请考虑使用`#`私有字段和/或`get()`函数属性。

初始值为原始值的属性声明为`readonly`，与其他属性相比有一个小小的怪癖：如果可能的话，它们被推断为其值的狭窄*字面*类型，而不是更宽的*原始*类型。 TypeScript 对更积极的初始类型缩小感到舒适，因为它知道这个值后来不会改变；这与`const`变量比`let`变量具有更窄类型类似。

在这个例子中，类的属性最初都声明为字符串字面值，因此为了将其中一个扩展为`string`，需要一个类型注释：

```
class RandomQuote {
    readonly explicit: string = "Home is the nicest word there is.";
    readonly implicit = "Home is the nicest word there is.";

    constructor() {
        if (Math.random () > 0.5) {
            this.explicit = "We start learning the minute we're born." // Ok;

            this.implicit = "We start learning the minute we're born.";
            // Error: Type '"We start learning the minute we're born."' is
            // not assignable to type '"Home is the nicest word there is."'.
        }
    }
}

const quote = new RandomQuote();

quote.explicit; // Type: string
quote.implicit; // Type: "Home is the nicest word there is."
```

明确扩展属性的类型通常不是很必要。但在构造函数中的条件逻辑的情况下有时会有用，比如`RandomQuote`中的构造函数：

# 类作为类型

类在类型系统中相对独特，因为类声明既创建一个运行时值（类本身），又创建一个可以用于类型注释的类型。

这个 `Teacher` 类的名称用于注释一个 `teacher` 变量，告诉 TypeScript 它只能被分配给可以分配给 `Teacher` 类的值，比如 `Teacher` 类的实例：

```
class Teacher {
    sayHello() {
        console.log("Take chances, make mistakes, get messy!");
    }
}

let teacher: Teacher;

teacher = new Teacher(); // Ok

teacher = "Wahoo!";
// Error: Type 'string' is not assignable to type 'Teacher'.
```

有趣的是，TypeScript 将认为任何包含与类的所有相同成员的对象类型都可以分配给这个类。这是因为 TypeScript 的结构类型检查只关心对象的形状，而不关心它们的声明方式。

在这里，`withSchoolBus` 接受一个类型为 `SchoolBus` 的参数。任何对象，只要它恰好具有类型为 `() => string[]` 的 `getAbilities` 属性，比如 `SchoolBus` 类的一个实例，就可以满足这一要求：

```
class SchoolBus {
    getAbilities() {
        return ["magic", "shapeshifting"];
    }
}

function withSchoolBus(bus: SchoolBus) {
    console.log(bus.getAbilities());
}

withSchoolBus(new SchoolBus()); // Ok

// Ok
withSchoolBus({
    getAbilities: () => ["transmogrification"],
});

withSchoolBus({
    getAbilities: () => 123,
    //                  ~~~
    // Error: Type 'number' is not assignable to type 'string[]'.
});
```

###### 小贴士

在大多数真实世界的代码中，开发者不会在需要类类型的地方传递对象值。这种结构检查行为可能看起来意外，但并不经常发生。

# 类与接口

回顾第七章，“接口”中，我向你展示了接口如何允许 TypeScript 开发者在代码中设置对象形状的期望。通过在类名后添加 `implements` 关键字，接着是接口的名称，允许一个类声明其实例应该可以分配给这些接口中的每一个。任何不匹配的地方将被类型检查器指出为类型错误。

在这个例子中，`Student` 类通过包含其属性 `name` 和方法 `study` 正确地实现了 `Learner` 接口，但 `Slacker` 缺少了 `study` 方法，因此导致类型错误：

```
interface Learner {
    name: string;
    study(hours: number): void;
}

class Student implements Learner {
    name: string;

    constructor(name: string) {
        this.name = name;
    }

    study(hours: number) {
        for (let i = 0; i < hours; i+= 1) {
            console.log("...studying...");
        }
    }
}

class Slacker implements Learner {
   // ~~~~~~~
   // Error: Class 'Slacker' incorrectly implements interface 'Learner'.
   //  Property 'study' is missing in type 'Slacker'
   //  but required in type 'Learner'.
    name = "Rocky";
}
```

###### 注意

接口用于被类实现是使用方法语法声明接口成员作为函数的典型原因，正如 `Learner` 接口所示。

将类标记为实现接口不会改变类的使用方式。如果类已经恰好匹配接口，TypeScript 的类型检查器将允许将其实例用于需要接口实例的地方。 TypeScript 甚至不会从接口推断类的方法或属性的类型：如果我们在 `Slacker` 示例中添加了一个 `study(hours) {}` 方法，TypeScript 将会将 `hours` 参数视为隐式的 `any`，除非我们对其进行类型注释。

这个 `Student` 类的版本会因为没有为其成员提供类型注解而导致隐式的 `any` 类型错误：

```
class Student implements Learner {
    name;
    // Error: Member 'name' implicitly has an 'any' type.

    study(hours) {
        // Error: Parameter 'hours' implicitly has an 'any' type.
    }
}
```

实现接口纯粹是一种安全检查。它不会为你在类定义中复制任何接口成员。而是通过实现接口向类型检查器表明你的意图，并在类定义中暴露类型错误，而不是在稍后使用类实例的地方。它的目的类似于为变量添加类型注解，即使它具有初始值。

## 实现多个接口

TypeScript 中的类允许声明为实现多个接口。类的实现接口列表可以是任意数量的接口名称，用逗号分隔。

在此示例中，两个类都必须至少有一个 `grades` 属性来实现 `Graded`，并且一个 `report` 属性来实现 `Reporter`。`Empty` 类由于未能正确实现任何接口而存在两个类型错误：

```
interface Graded {
    grades: number[];
}

interface Reporter {
    report: () => string;
}

class ReportCard implements Graded, Reporter {
    grades: number[];

    constructor(grades: number[]) {
        this.grades = grades;
    }

    report() {
        return this.grades.join(", ");
    }
}

class Empty implements Graded, Reporter { }
   // ~~~~~
   // Error: Class 'Empty' incorrectly implements interface 'Graded'.
   //   Property 'grades' is missing in type 'Empty'
   //   but required in type 'Graded'.
   // ~~~~~
   // Error: Class 'Empty' incorrectly implements interface 'Reporter'.
   //   Property 'report' is missing in type 'Empty'
   //   but required in type 'Reporter'.
```

实际应用中，可能会存在一些接口定义使得同一个类无法实现两者。尝试声明一个类同时实现两个冲突接口将导致类至少有一个类型错误。

下面的 `AgeIsANumber` 和 `AgeIsNotANumber` 接口为 `age` 属性声明了非常不同的类型。`AsNumber` 类和 `NotAsNumber` 类都未能正确实现这两个接口：

```
interface AgeIsANumber {
    age: number;
}

interface AgeIsNotANumber {
    age: () => string;
}

class AsNumber implements AgeIsANumber, AgeIsNotANumber {
    age = 0;
 // ~~~
 // Error: Property 'age' in type 'AsNumber' is not assignable
 // to the same property in base type 'AgeIsNotANumber'.
 //   Type 'number' is not assignable to type '() => string'.
}
```

```
class NotAsNumber implements AgeIsANumber, AgeIsNotANumber {
    age() { return ""; }
 // ~~~
 // Error: Property 'age' in type 'NotAsNumber' is not assignable
 // to the same property in base type 'AgeIsANumber'.
 //   Type '() => string' is not assignable to type 'number'.
}
```

当两个接口描述非常不同的对象形状的情况下，通常表明你不应该尝试使用同一个类来实现它们。

# 扩展类

TypeScript 在 JavaScript 类扩展或子类化的概念上增加了类型检查。首先，基类上声明的任何方法或属性都将在子类（也称为派生类）上可用。

在此示例中，`Teacher` 声明了一个 `teach` 方法，可以被 `StudentTeacher` 子类的实例使用：

```
class Teacher {
    teach() {
        console.log("The surest test of discipline is its absence.");
    }
}

class StudentTeacher extends Teacher {
    learn() {
        console.log("I cannot afford the luxury of a closed mind.");
    }
}

const teacher = new StudentTeacher();
teacher.teach(); // Ok (defined on base)
teacher.learn(); // Ok (defined on subclass)

teacher.other();
 //     ~~~~~
 // Error: Property 'other' does not exist on type 'StudentTeacher'.
```

## 扩展可分配性

子类像派生接口扩展基础接口一样继承其基类的成员。子类的实例具有其基类的所有成员，因此可以在需要基类实例的地方使用。如果基类没有子类具有的所有成员，则在需要更具体的子类时不能使用基类。

以下 `Lesson` 类的实例不能在需要其派生类 `OnlineLesson` 的地方使用，但派生实例可以用来满足基类或子类的要求：

```
class Lesson {
    subject: string;

    constructor(subject: string) {
        this.subject = subject;
    }
}

class OnlineLesson extends Lesson {
    url: string;

    constructor(subject: string, url: string) {
        super(subject);
        this.url = url;
    }
}

let lesson: Lesson;
lesson = new Lesson("coding"); // Ok
lesson = new OnlineLesson("coding", "oreilly.com"); // Ok

let online: OnlineLesson;
online = new OnlineLesson("coding", "oreilly.com"); // Ok

online = new Lesson("coding");
// Error: Property 'url' is missing in type
// 'Lesson' but required in type 'OnlineLesson'.
```

根据 TypeScript 的结构类型，如果子类的所有成员在其基类上已经存在且类型相同，则仍允许使用基类的实例来代替子类的实例。

在此示例中，`LabeledPastGrades` 只向 `PastGrades` 添加了一个可选属性，因此基类的实例可以替换子类的实例：

```
class PastGrades {
    grades: number[] = [];
}

class LabeledPastGrades extends PastGrades {
    label?: string;
}

let subClass: LabeledPastGrades;

subClass = new LabeledPastGrades(); // Ok
subClass = new PastGrades(); // Ok
```

###### 提示

在大多数真实世界的代码中，子类通常在其基类之上添加新的必需类型信息。这种结构检查行为可能看起来出乎意料，但并不经常发生。

## 覆盖构造函数

与原始 JavaScript 类似，TypeScript 中的子类不需要定义自己的构造函数。没有自己构造函数的子类隐式使用其基类的构造函数。

在 JavaScript 中，如果子类确实声明了自己的构造函数，则必须通过 `super` 关键字调用其基类构造函数。子类构造函数可以声明任何参数，而不管其基类需要什么参数。TypeScript 的类型检查器将确保对基类构造函数的调用使用了正确的参数。

在这个例子中，`PassingAnnouncer` 的构造函数正确地使用一个 `number` 参数调用了基类构造函数，而 `FailingAnnouncer` 由于忘记进行此调用而导致类型错误：

```
class GradeAnnouncer {
    message: string;

    constructor(grade: number) {
        this.message = grade >= 65 ? "Maybe next time..." : "You pass!";
    }
}

class PassingAnnouncer extends GradeAnnouncer {
    constructor() {
        super(100);
    }
}

class FailingAnnouncer extends GradeAnnouncer {
    constructor() { }
 // ~~~~~~~~~~~~~~~~~
 // Error: Constructors for subclasses must contain a 'super' call.
}
```

根据 JavaScript 规则，在子类的构造函数中，在访问 `this` 或 `super` 之前必须调用基类构造函数。如果 TypeScript 在 `super()` 之前看到 `this` 或 `super` 被访问，将报告类型错误。

下面的 `ContinuedGradesTally` 类在其构造函数中在调用 `super()` 之前错误地引用了 `this.grades`：

```
class GradesTally {
    grades: number[] = [];

    addGrades(...grades: number[]) {
        this.grades.push(...grades);
        return this.grades.length;
    }
}

class ContinuedGradesTally extends GradesTally {
    constructor(previousGrades: number[]) {
        this.grades = [...previousGrades];
        // Error: 'super' must be called before accessing
        // 'this' in the constructor of a subclass.

        super();

        console.log("Starting with length", this.grades.length); // Ok
    }
}
```

## 被重写的方法

子类可以重新声明与基类相同名称的新方法，只要子类方法上的方法可以分配给基类上的方法即可。请记住，由于子类可以用在原始类可用的任何地方，新方法的类型必须能够替代原始方法的位置。

在这个例子中，`FailureCounter` 的 `countGrades` 方法被允许，因为它与基类 `GradeCounter` 的 `countGrades` 方法具有相同的第一个参数和返回类型。`AnyFailureChecker` 的 `countGrades` 因返回类型不正确而导致类型错误：

```
class GradeCounter {
    countGrades(grades: string[], letter: string) {
        return grades.filter(grade => grade === letter).length;
    }
}

class FailureCounter extends GradeCounter {
    countGrades(grades: string[]) {
        return super.countGrades(grades, "F");
    }
}

class AnyFailureChecker extends GradeCounter {
    countGrades(grades: string[]) {
        // Property 'countGrades' in type 'AnyFailureChecker' is not
        // assignable to the same property in base type 'GradeCounter'.
        //   Type '(grades: string[]) => boolean' is not assignable
        //   to type '(grades: string[], letter: string) => number'.
        //      Type 'boolean' is not assignable to type 'number'.
        return super.countGrades(grades, "F") !== 0;
    }
}

const counter: GradeCounter = new AnyFailureChecker();

// Expected type: number
// Actual type: boolean
const count = counter.countGrades(["A", "C", "F"]);
```

## 被重写的属性

子类也可以显式地重新声明其基类的同名属性，只要新类型可分配给基类的类型即可。与重写方法一样，子类必须与基类结构匹配。

大多数重新声明属性的子类都是为了将这些属性作为类型联合的更具体子集，或者使属性成为扩展自基类属性类型的类型。

在这个例子中，基类 `Assignment` 将其 `grade` 声明为 `number |` `undefined`，而子类 `GradedAssignment` 将其声明为一个始终存在的 `number`：

```
class Assignment {
    grade?: number;
}

class GradedAssignment extends Assignment {
    grade: number;

    constructor(grade: number) {
        super();
        this.grade = grade;
    }
}
```

扩展属性联合类型的允许值集合是不允许的，因为这样做将使子类属性不再可分配给基类属性的类型。

在这个例子中，`VagueGrade` 的 `value` 尝试在基类 `NumericGrade` 的 `number` 类型之上添加 `| string`，导致类型错误：

```
class NumericGrade {
    value = 0;
}

class VagueGrade extends NumericGrade {
    value = Math.random() > 0.5 ? 1 : "...";
    // Error: Property 'value' in type 'NumberOrString' is not
    // assignable to the same property in base type 'JustNumber'.
    //   Type 'string | number' is not assignable to type 'number'.
    //     Type 'string' is not assignable to type 'number'.
}

const instance: NumericGrade = new VagueGrade();

// Expected type: number
// Actual type: number | string
instance.value;
```

# 抽象类

有时创建一个基类，它本身不声明某些方法的实现，而是期望子类提供它们，这样做可以很有用。通过在类名前面加上 TypeScript 的 `abstract` 关键字和任何预期为抽象的方法前面加上 `abstract` 关键字来标记一个类为抽象。这些抽象方法声明在抽象基类中不提供具体实现；相反，它们的声明方式与接口相同。

在此示例中，`School` 类及其 `getStudentTypes` 方法被标记为 `abstract`。因此，其子类——`Preschool` 和 `Absence`——应当实现 `getStudentTypes`：

```
abstract class School {
    readonly name: string;

    constructor(name: string) {
        this.name = name;
    }

    abstract getStudentTypes(): string[];
}

class Preschool extends School {
    getStudentTypes() {
        return ["preschooler"];
    }
}

class Absence extends School { }
   // ~~~~~~~
   // Error: Nonabstract class 'Absence' does not implement
   // inherited abstract member 'getStudentTypes' from class 'School'.
```

抽象类不能直接实例化，因为它没有某些方法的定义，其实现可能假定存在。只有非抽象（“具体的”）类可以被实例化。

继续 `School` 示例，尝试调用 `new School` 将导致 TypeScript 类型错误：

```
let school: School;

school = new Preschool("Sunnyside Daycare"); // Ok

school = new School("somewhere else");
// Error: Cannot create an instance of an abstract class.
```

抽象类通常在期望消费者填写类细节的框架中使用。类可以用作类型注解，以指示值必须遵守该类，就像早期示例中的 `school: School` 一样，但创建新实例必须使用子类完成。

# 成员可见性

JavaScript 允许在类成员名称前面加 `#` 标记为“私有”类成员。私有类成员只能由该类的实例访问。JavaScript 运行时通过在类外部的代码尝试访问私有方法或属性时抛出错误来强制执行此隐私。

TypeScript 的类支持早于 JavaScript 的真正的 `#` 隐私功能，虽然 TypeScript 支持私有类成员，但它还允许在仅存在于类型系统中的类方法和属性上定义略微更加微妙的隐私定义。通过在类成员的声明名称之前添加以下关键字之一，可以实现 TypeScript 的成员可见性：

`public`（默认）

允许任何人在任何地方访问

`protected`

只允许类本身及其子类访问

`private`

只允许类本身访问

这些关键字仅存在于类型系统中。当代码编译为 JavaScript 时，它们与所有其他类型系统语法一起被移除。

在这里，`Base` 声明了两个 `public` 成员，一个 `protected` 成员，一个 `private` 成员和一个真正的私有成员 `#truePrivate`。`Subclass` 可以访问 `public` 和 `protected` 成员，但不能访问 `private` 或 `#truePrivate`：

```
class Base {
    isPublicImplicit = 0;
    public isPublicExplicit = 1;
    protected isProtected = 2;
    private isPrivate = 3;
    #truePrivate = 4;
}

class Subclass extends Base {
    examples() {
        this.isPublicImplicit; // Ok
        this.isPublicExplicit; // Ok
        this.isProtected; // Ok

        this.isPrivate;
        // Error: Property 'isPrivate' is private
        // and only accessible within class 'Base'.

        this.#truePrivate;
        // Property '#truePrivate' is not accessible outside
        // class 'Base' because it has a private identifier.
    }
}

new Subclass().isPublicImplicit; // Ok
new Subclass().isPublicExplicit; // Ok

new Subclass().isProtected;
//             ~~~~~~~~~~~
// Error: Property 'isProtected' is protected
// and only accessible within class 'Base' and its subclasses.

new Subclass().isPrivate;
//             ~~~~~~~~~~~
// Error: Property 'isPrivate' is private
// and only accessible within class 'Base'.
```

TypeScript 的成员可见性与 JavaScript 的真正私有声明的关键区别在于 TypeScript 的存在仅存在于类型系统中，而 JavaScript 的私有成员也存在于运行时。 TypeScript 中声明为 `protected` 或 `private` 的类成员在编译为 JavaScript 代码时，与显式或隐式声明为 `public` 的代码相同。与接口和类型注解一样，可见性关键字在输出 JavaScript 时会被擦除。只有 `#` 私有字段在运行时的 JavaScript 中才是真正私有的。

可见性修饰符可以与 `readonly` 一起标记。要同时声明成员为 `readonly` 和显式可见性，可见性应该先声明。

此 `TwoKeywords` 类将其 `name` 成员声明为 `private` 和 `readonly`：

```
class TwoKeywords {
    private readonly name: string;

    constructor() {
        this.name = "Anne Sullivan"; // Ok
    }

    log() {
        console.log(this.name); // Ok
    }
}

const two = new TwoKeywords();

two.name = "Savitribai Phule";
 // ~~~~
 // Error: Property 'name' is private and
 // only accessible within class 'TwoKeywords'.
 // ~~~~
 // Error: Cannot assign to 'name'
 // because it is a read-only property.
```

请注意，不允许将 TypeScript 的旧成员可见性关键字与 JavaScript 的新 `#` 私有字段混合使用。私有字段默认始终为私有，因此无需额外使用 `private` 关键字标记它们。

## 静态字段修饰符

JavaScript 允许在类本身上声明成员——而不是其实例——使用 `static` 关键字。 TypeScript 支持在其自身上使用 `static` 关键字和/或与 `readonly` 和/或一个可见性关键字结合使用。当组合使用时，可见性关键字首先出现，然后是 `static`，然后是 `readonly`。

这个 `HasStatic` 类将它们全部整合到一起，使其 `static` 的 `prompt` 和 `answer` 属性都是 `readonly` 和 `protected`：

```
class Question {
    protected static readonly answer: "bash";
    protected static readonly prompt =
        "What's an ogre's favorite programming language?";

    guess(getAnswer: (prompt: string) => string) {
        const answer = getAnswer(Question.prompt);

        // Ok
        if (answer === Question.answer) {
            console.log("You got it!");
        } else {
            console.log("Try again...")
        }
    }
}

Question.answer;
//       ~~~~~~
// Error: Property 'answer' is protected and only
// accessible within class 'HasStatic' and its subclasses.
```

使用只读和/或可见性修饰符来限制静态类字段，对于阻止这些字段在其类外被访问或修改很有用。

# 摘要

本章介绍了关于类的大量类型系统特性和语法：

+   声明和使用类的方法和属性

+   将属性标记为 `readonly` 和/或可选

+   将类名用作类型注释的类型

+   实现接口以强制执行类实例的形状

+   扩展类，以及子类的可赋值性和覆盖规则

+   将类和方法标记为抽象

+   向类字段添加类型系统修饰符

###### 提示

现在您已经完成了阅读本章，可以将所学的内容练习到 [*https://learningtypescript.com/classes*](https://learningtypescript.com/classes) 上。

> 为什么面向对象编程开发人员总是穿着西装？
> 
> 因为它们拥有类。
