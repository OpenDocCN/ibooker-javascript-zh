# 第八章：类

JavaScript 是一种面向对象编程语言吗？答案取决于你问的是谁（以及你如何提问）。但一般的共识是*是*，尽管有些限制条件。

在学术界之外，面向对象编程语言通常围绕类、接口和继承等概念展开。但直到最近，JavaScript 一直是一个特例——一个建立在函数和*原型*上的面向对象编程语言。然后，ES6 出现了，突然之间类作为一种本地语言结构就出现了，使情况变得复杂起来。它只是语法糖还是一次重大的语言演进呢？

答案介于两者之间。总体而言，ES6 类是建立在 JavaScript 原型的熟悉基础上的一种更高级的语言特性。但映射并非完全一致，类模型引入了一些新的细微差别，这些差别在原型模型中并没有完全体现出来。此外，很可能未来的类将支持新的面向对象特性，进一步拉开这两种重叠模型的差距。

底线是：如今，新的开发倾向于使用类，但基于原型的代码仍然普遍存在（远非过时）。本章重点介绍使用类的常见模式，同时也探讨了原型。

# 创建可重复使用的类

## 问题

你希望为自定义对象创建一个可重复使用的模板。

## 解决方案

使用`class`关键字，并为你的类取一个名字。在内部，添加一个构造函数来初始化你的对象。以下是一个完整的`Person`类示例：

```
class Person {
  constructor(firstName, lastName) {
    this.firstName = firstName;
    this.lastName = lastName;
  }
}

// Test the Person class by creating an object
// The constructor is invoked when you use the new keyword with the class
const newPerson = new Person('Luke', 'Takei');
console.log(newPerson.firstName);  // 'Luke'
```

在本例中，`Person`类是一个简单的包装，将两个公共字段（`firstName`和`lastName`）捆绑在一起。但是，很容易向你的类添加方法，这些方法像函数一样工作，但不包括`function`关键字。以下是如何编写`Person.swapNames()`方法的代码示例：

```
class Person {
  constructor(firstName, lastName, dateOfBirth) {
    this.firstName = firstName;
    this.lastName = lastName;
    this.dateOfBirth = dateOfBirth;
  }

  // This is a method
  swapNames() {
    // Use a handy shortcut (destructuring assignment) to assign both
    // properties at once
    [this.firstName, this.lastName] = [this.lastName, this.firstName];
  }
}

// Test the Person class
const newPerson = new Person('Luke', 'Takei', new Date(1990, 5, 22));
newPerson.swapNames();
console.log(newPerson.firstName);   // 'Takei'
```

## 讨论

JavaScript 类的本质是构造函数。事实上，在幕后，JavaScript 类*是*一个构造函数，并且所有方法都附加到该函数的*原型*上。这意味着像`Person.swapNames()`这样的方法在`Person`类的所有实例之间是共享的，因为它们共享同一个原型。（要深入了解这个幕后实现，请查看“使用构造函数模式制作自定义类”中的构造函数模式。）

类有其自己的语法要求，你必须遵循这些要求：

+   构造函数总是命名为`constructor`。

+   无论构造函数还是方法，都不使用关键字`function`，尽管在其他方面声明方式与函数相同。

当你编写一个构造函数时，你使用`this`来在当前对象上创建新的公共字段。然后，在你的类方法中，无论何时需要，都可以引用这些字段，只要记得始终在变量名前加上`this`前缀。你还可以在类代码外部访问这些字段，使用熟悉的点语法。

您可能想知道如何更改此可访问性——例如，使字段私有并用公共属性包装它们。答案是目前您无法做到这一点——至少不会引入自己的自制解决方案而带来其它复杂性。有关完整讨论，请参阅“向类添加属性”。

与函数类似，JavaScript 允许您在*表达式*中创建类。以下是一个示例：

```
const personExpression = class Person {
  constructor(firstName, lastName) {
    this.firstName = firstName;
    this.lastName = lastName;
  }
}

// This won't work, because there is no Person class to be found in scope
const newPerson = new Person('Luke', 'Takei');

// This works because you can create a new instance of the variable that holds
// the class expression
const newPerson = new personExpression('Luke', 'Takei');
```

这是一种专业的——但并非罕见的——技术。它允许您避免将类添加到当前作用域中。例如，在这个例子中，如果您担心可能已经有另一个`Person`类的定义，这可能会很有用。（解决名称冲突问题的另一种方法是使用模块，如“使用 ES6 模块组织您的 JavaScript 类”所述。）

## 另请参阅

对于老式的构造函数模式用于对象创建，请参阅“使用构造函数模式创建自定义类”。要了解如何创建类属性，请参考“向类添加属性”。要学习如何在继承关系中连接类，请参阅“从另一个类继承功能”。

## 额外内容：多个构造函数

在大多数面向对象的语言中，可以创建多个构造函数，因此创建类的代码可以选择指定哪些参数。但 JavaScript 不支持构造函数重载或方法重载。

这并不像看起来的那样有限，因为 JavaScript 在处理函数参数时非常宽松，并且从不强制您提供它们。因此，即使`Person`有一个单独的三参数构造函数，以下都是创建实例的有效方法，而不必提供每个参数：

```
const noDatePerson = new Person('Luke', 'Takei');
const firstNamePerson = new Person('Luke');
const noDataPerson = new Person();
```

每个类都有且只有一个构造函数，并且它总是运行的。即使在创建`Person`对象时没有指定任何参数，三参数构造函数仍然会运行并设置`this.firstName`、`this.lastName`和`this.birthDate`（这些都将被设置为`undefined`）。如果这样不可接受，您可以设置默认参数值，就像在普通函数中一样（参见“提供默认参数值”）。

###### 注意

如果您创建一个没有构造函数的类，JavaScript 会自动给它一个空的无参数构造函数。如果您决定使用类继承，这个细节将变得重要（参见“从另一个类继承功能”）。

处理可选参数的另一种方法是使用传递给构造函数的对象字面量。这样调用者可以选择设置他们想要使用的命名属性：

```
const partialInfoPerson1 = new Person({
  lastName: "Takei",
  birthDate: new Date(1990, 04, 23)
});
const partialInfoPerson2 = new Person({firstName: 'Luke', lastName: 'Takei'});
```

这是一种常见的 JavaScript 设计模式，详细描述在 “使用命名函数参数” 中。它提供的一个优点是你不需要担心对象字面量中属性的顺序。一个缺点是，没有任何东西可以防止你意外地创建错误命名的参数，这些参数将会被静默地忽略：

```
// The Person class will look for a firstName property in this object literal
// It will quietly ignore the firstname property
const partialInfoPerson2 = new Person({firstname: 'Luke'});
```

另一种可能的方法是为您的类创建一个单一的构造函数，但添加静态方法来创建不同配置的对象实例。根据实现的不同，这有时被称为 *建造者模式* 或 *工厂模式*。它在 “使用静态方法创建对象” 中有描述。

# 添加类属性

## 问题

您希望添加属性的获取器和设置器以包装类数据。

## 解决方案

首先，考虑属性是否是您用例的最佳解决方案。（正如讨论中所解释的那样，它们有众所周知的局限性，并且稍微有争议。）如果决定使用属性，可以为每个属性创建 `get` 和 `set` 方法。这里有一个计算属性 `age` 的示例，它是从存储在 `this.dateOfBirth` 中的日期计算出来的：

```
class Person {
  constructor(firstName, lastName, dateOfBirth) {
    this.firstName = firstName;
    this.lastName = lastName;
    this.dateOfBirth = dateOfBirth;
  }

  // This is a getter for the age property
  get age() {
    if (this.dateOfBirth instanceof Date) {
      // Calculate the difference in years
      const today = new Date();
      let age = today.getFullYear() - this.dateOfBirth.getFullYear();

      // Adjust if the bithday hasn't happened yet this year
      const monthDiff = today.getMonth() - this.dateOfBirth.getMonth();
      if (monthDiff < 0 ||
         (monthDiff === 0 && today.getDate() < this.dateOfBirth.getDate())) {
        age -= 1;
      }

      return age;
    }
  }
}

// Test the Person class
const newPerson = new Person('Luke', 'Takei', new Date(1990, 5, 22));
console.log(newPerson.age);
```

由您决定是否仅包含获取器、仅包含设置器，或者两者都包含。这里有一个使用属性模式应用基本验证到出生日期的示例：

```
class Person {
  constructor(firstName, lastName, date) {
    this.firstName = firstName;
    this.lastName = lastName;

    // Set the date using the property setter so a Person
    // can't be created in an invalid state
    this.dateOfBirth = date;
  }

  // Just return the date with no extra processing
  get dateOfBirth() {
    return this._dateOfBirth;
  }

  // Don't allow dates in the future
  set dateOfBirth(value) {
    if (value instanceof Date && value < Date.now()) {
      // This is a valid date
      this._dateOfBirth = value;
    }
    else {
      throw new TypeError('Birthdate needs to be a valid date in the past');
    }
  }
}

// Test the date restrictions
const newPerson = new Person('Luke', 'Takei', new Date(1990, 5, 22));
console.log(newPerson.dateOfBirth);

// This change is allowed
newPerson.dateOfBirth = new Date(2010, 10, 10);
console.log(newPerson.dateOfBirth);

// This change causes an error
newPerson.dateOfBirth = new Date(2035, 10, 10);
```

###### 注意

这个示例在试图设置无效值时抛出异常（“抛出标准错误”），以通知调用者。这是一个合理的设计决策，但在 JavaScript 中，当尝试设置属性（甚至更糟的是，尝试使用无效日期创建 `Person` 时）时发生错误不是预期的行为，并且调用代码可能未预料到可能的错误。（另一种选择——在设置属性时静默地忽略有问题的错误——也是有风险的。）最终，使用方法来提供潜在问题数据而不是属性可能是更好的方法。

## 讨论

有许多原因可能导致您考虑创建属性过程。一些例子包括：

+   计算一个值（如 `Person.age`）

+   将一个字段转换为另一种表示形式

+   在更新字段之前执行验证

+   为某些其他服务（如日志记录或测试）添加钩子，这些服务应在每次读取或设置字段时发生

+   使用某种惰性初始化，只有在需要时才创建或计算属性值

+   公开存储在字段中的对象的单个属性

本篇介绍两个示例。`Person.age` 属性是只读的计算属性。`Person.dateOfBirth` 属性是带有验证的可设置属性。

使用属性时，必须小心避免名称冲突。存储值的字段不能与属性或构造函数参数具有相同的名称。为了理解原因，让我们更仔细地看看`dateOfBirth`的例子。构造函数接受一个`date`参数，并像这样设置它：

```
this.dateOfBirth = date;
```

乍一看，您可能会认为此语句将日期存储在名为`this.dateOfBirth`的公共字段中（这是通常的模式）。但在这种情况下，`this.dateOfBirth`指的是`dateOfBirth`属性。其 setter 接管了：

```
set dateOfBirth(value) {
if (value instanceof Date && value < Date.now()) {
  // This is a valid date
  this._dateOfBirth = value;
}
else {
  throw new TypeError('Birthdate needs to be a valid date in the past');
}
```

如果新值通过测试，它将存储在名为`this._dateOfBirth`的公共字段中。这种笨拙的命名是必要的，因为`this.dateOfBirth`（属性）和`this._dateOfBirth`（字段）具有相同的作用域。如果两者使用相同的名称，您将会调用错误的那个（并触发无限调用序列，最终导致堆栈溢出）。

类似`_dateOfBirth`这样的变量名中的前导下划线还有另一个目的。目前，JavaScript 没有任何创建私有字段的方法。但下划线表示该字段应该是类的私有字段。然后，您可以信任调用代码会避免使用这个字段。如果不遵循这种约定，几乎肯定会遇到问题，即调用代码意外使用字段而不是属性。即使遵循了这种模式，也不能保证调用代码会遵循它。

许多 JavaScript 开发人员认为，在 JavaScript 中更自然的模式是使用`setXxx()`和`getXxx()`方法：

```
class Person {
  constructor(firstName, lastName, date) {
    this.firstName = firstName;
    this.lastName = lastName;
    this.setDateOfBirth(date);
  }

  getDateOfBirth() {
    return this._dateOfBirth;
  }

  setDateOfBirth(value) {
    if (value instanceof Date && value < Date.now()) {
      // This is a valid date
      this._dateOfBirth = value;
    }
    else {
      throw new TypeError('Birthdate cannot be in the future');
    }
  }
}

const newPerson = new Person('Luke', 'Takei', new Date(1990, 5, 22));
console.log(newPerson.getDateOfBirth());

// This change is allowed
newPerson.setDateOfBirth (new Date(2010, 10, 10));
console.log(newPerson.getDateOfBirth());

// This change causes an error
newPerson.setDateOfBirth (new Date(2035, 10, 10));
```

这种方法有点繁琐，但有一些优点。它明确表明您正在调用方法并运行代码，而不仅仅是设置变量。因此，调用代码可以期望来自类型检查或其他副作用的异常。方法还可以防止这样的问题：

```
// This isn't the property you want (that's dateOfBirth) but JavaScript
//  creates it anyway, and you won't notice the mistake
person.DateOfBirth = new Date(2035, 10, 10);

// You can't call a function that doesn't exist, so this typo
// ("Data" instead of "Date") always fails and won't be ignored
person.setDataOfBirth(new Date(2035, 10, 10));
```

###### 注意

[Google JavaScript Style Guide](https://google.github.io/styleguide/jsguide.html)和经常查阅的[Airbnb JavaScript Style Guide](https://github.com/airbnb/javascript)都不鼓励使用属性的 getter 和 setter，但允许使用`setXxx()`和`getXxx()`方法。

使用属性时，还有一个要考虑的细微之处。在 JavaScript 内部，使用`Object.defineProperty()`方法来实现您的属性 getter 和 setter。大多数情况下，这样做完全正常。但在特定情况下，您可能决定使用`defineProperty()`，因为它允许您配置无法以其他方式设置的元数据细节。例如，如果您想使属性不可配置（因此其实现不能被更改）或不可枚举（因此它不会在`for...in`循环中显示），您需要显式调用`defineProperty()`。在这种情况下，通常的做法是在构造函数中调用`defineProperty`。

## 参见

如果你希望使用属性程序来响应属性变化并触发其他操作（比如日志记录），考虑使用代理而不是（“使用代理拦截和修改对象上的操作”）。有关使用 `Object.defineProperty()` 创建属性的更多信息，请参阅“自定义属性定义方式”。

## 额外：私有字段

目前，JavaScript 没有办法使成员变量（使用 `this` 创建的变量）私有化。许多解决方案都在使用，并且其中许多都是非常有创意且危险的。最流行的实现使用 `WeakMap` 来存储内部数据。它确实有效，但也增加了一层危险的自制复杂性。

更好的方法是使用下划线约定（如 `_firstName`）来命名那些不应在类外部访问的字段。未来，JavaScript 将弥补这一空缺并采纳某种版本的[*私有类字段*提案](https://github.com/tc39/proposal-class-fields)。目前，私有字段语法使用 `#` 来标识私有字段，可以在类块的开头声明，使得你的类自说明性更强。具体如下所示：

```
// A likely implementation of private field syntax in the near future
class Person {
  #firstName;
  #lastName;

  constructor(firstName, lastName) {
    this.#firstName = firstName;
    this.#lastName = lastName;
  }

  // Wrap the fields in properties
	get firstName() {
    return this.#firstName;
  }
  set firstName(name) {
    this.#firstName = name;
  }

  get lastName() {
    return this.#lastName;
  }
  set lastName(name) {
    this.#lastName = name;
  }
}
```

如果你想要立即尝试这些功能，可以使用[Babel](https://babeljs.io)来转译你的代码，尽管请注意语法可能会发生变化。

有趣的是，在这种情况下，JavaScript 类的功能比老式的构造函数模式*更少*（“使用构造函数模式创建自定义类”）。因为构造函数模式可以使用闭包来存储私有变量，如“使用闭包存储其状态的函数”中所解释的。

# 给类一个更好的字符串表示

## 问题

当将对象转换为字符串时，你想选择一个合适的文本表示方式。

## 解决方案

在你的类中添加一个名为 `toString()` 的方法，并返回你想要使用的字符串：

```
class Person {
  constructor(firstName, lastName) {
    this.firstName = firstName;
    this.lastName = lastName;
  }

  toString() {
    return `${this.lastName}, ${this.firstName}`;
  }
}

const newPerson = new Person('Luke', 'Takei');
console.log(newPerson.toString());   // 'Takei, Luke'
```

## 讨论

所有对象的默认 `toString()` 实现显示不太有帮助的文本 `[object Object]`。你可以通过添加 `toString()` 方法来设置自己的文本。

`toString()` 方法可以显式调用（就像这个例子中一样），或者当你的对象转换为字符串时可以隐式调用。例如，如果你将你的对象与一个字符串连接，`toString()` 就会自动调用：

```
const newPerson = new Person('Luke', 'Takei');
const message = 'The name is ' + newPerson;

// Now message = 'The name is Takei, Luke'
// which is much better than 'The name is [object Object]'
```

但是，仅对对象调用 `console.log()` 并不会触发你的 `toString()`。这是因为 `console.log()` 有额外的逻辑，遍历对象的属性并用其构建自定义字符串。你可以通过手动调用 `toString()` 或使用模板字面量（“使用模板字面量进行更清晰的字符串拼接”）来解决这个问题。下面是一个比较：

```
const newPerson = new Person('Luke', 'Takei');

console.log(newPerson);       // 'Person {firstName: "Luke", lastName: "Takei"}'
console.log(`${newPerson}`);  // 'Takei, Luke'
console.log(newPerson+'');    // 'Takei, Luke'
```

# 使用构造函数模式创建自定义类

## 问题

您希望在您的代码中创建一个可重用的类似类的实体。您希望使用传统的构造函数模式，因为它与您现有的代码匹配。

## 解决方案

构造函数模式是一种稍微过时但仍然可接受的对象创建模式。即使您打算使用正式类（“创建可重用类”），了解构造函数模式也很重要，因为您可能会在野外遇到它。它还可以帮助您理解 JavaScript 类的工作原理。

下面是一个来自“使用 ES6 类”的`Person`类的示例，但按照构造函数模式编写：

```
function Person(firstName, lastName) {
  // Store public data using 'this'
  this.firstName = firstName;
  this.lastName = lastName;

  // Add a nested function to represent a method
  this.swapNames = function() {
    [this.firstName, this.lastName] = [this.lastName, this.firstName];
  }
}

// Create a Person object
const newPerson = new Person('Luke', 'Takei');
console.log(newPerson.firstName);  // 'Luke'

newPerson.swapNames();
console.log(newPerson.firstName);  // 'Takei'
```

注意，使用基于函数的对象的代码与使用具有相同构造函数的基于类的代码相同。因此，通常可以将代码从构造函数模式迁移到正式类而不会破坏应用程序的其他部分。

## 讨论

类是 JavaScript 语言的一个相对晚期的引入者。在类存在之前，开发人员使用函数来代替。这是因为 JavaScript 允许您使用`new`关键字创建函数的新*实例*（函数对象）。每个函数都有自己的作用域，具有自己的局部数据。

构造函数模式存在多种变体。最常见的方法是创建一个函数，函数名为您的“类”的名称，并接受创建实例所需的所有构造函数参数。在函数内部，使用`this`来创建公共字段。您还可以创建普通变量，这些变量对外部代码不可见，仅由构造函数和任何嵌套函数使用。

有两种常见的方法来创建类似方法的函数。这里展示的方法使用函数表达式创建每个方法，并使用`this`使它们公开可访问。因为方法函数被包裹在构造函数内部，它们具有与构造函数相同的作用域，并且可以访问所有相同的变量和局部变量。 （从技术上讲，构造函数创建了一个闭包，如在“创建一个使用闭包存储其状态的函数”中所解释的。）

创建方法的另一种方式是将它们显式添加到构造函数的*原型*中。如果您尚未遇到原型，它们是一种基本（但大多数情况下隐藏的）组成部分，允许对象共享功能。当您尝试调用方法（如`Person.swapNames()`）时，JavaScript 会查找`Person`构造函数中的`swapNames()`函数。如果找不到它，JavaScript 会在原型中查找`swapNames()`函数。当涉及继承时，这个过程会变得更复杂，因为 JavaScript 会在整个*原型链*中搜索函数，如“从另一个类继承功能”中所解释的。

那么如何向原型添加函数呢？您可以直接使用`prototype`属性：

```
function Person(firstName, lastName) {
  this.firstName = firstName;
  this.lastName = lastName;
}

// Add function to the Person prototype to represent a method
Person.prototype.swapNames = function() {
  [this.firstName, this.lastName] = [this.lastName, this.firstName];
}

const newPerson = new Person('Luke', 'Takei');
newPerson.swapNames();
console.log(newPerson.firstName);  // 'Takei'
```

这个示例的行为与嵌套构造函数版本基本相同。但是有一个区别。以前，`swapNames()`在每个`Person`对象中独立存在。现在，在原型中设置了一个单独的`swapNames()`函数，并在所有`Person`实例之间共享。如果您计划创建将原型链接在一起的继承关系（参见“额外：原型链”），这一点很重要。如果您尝试使用闭包与私有变量（“使用闭包存储其状态的函数”）函数，因为附加到原型的函数不存在于构造函数的相同上下文中，并且无法访问在构造函数中定义的私有变量。

###### 注意

使用原型，您可以改变内置 JavaScript 对象的行为。例如，您可以为基本的`Array`或`String`类型添加功能。这听起来像一个很棒的功能，但它充满了复杂性，并且强烈不建议使用（除非用于构建框架）。模糊标准代码和自定义代码之间的区别会引起混淆，并且可能导致非标准模式、性能不佳的代码和隐藏的错误。如果有多个人尝试使用相同名称扩展内置对象，它也可能完全失败。

将构造函数模式与“创建可重用类”中显示的`class`关键字进行比较是很有趣的。在这两个示例中，大部分代码都是完全相同的：

+   您编写一个接受参数并初始化对象的构造函数。

+   您使用`this`关键字创建公开可访问的字段。

+   创建对象时使用`new`关键字（现在技术上是一个函数的实例，而不是一个类）。

但是也有一些细微的差异，最明显的是语法。在构造函数模式中没有专用属性，并且方法是单独声明的，而不是嵌套在构造函数中或显式附加到构造函数的原型（尽管在运行时确实是这样）。

## 参见

“使用 es6 类”演示了在现代 JavaScript 中创建自定义对象模板的首选方法，即使用`class`关键字。

# 在您的类中支持方法链

## 问题

您希望以一种使得多个方法可以快速连续调用的方式定义类方法。

## 解决方案

确保在支持方法链的每个方法的末尾返回当前对象。在自定义类中，通常只需添加`return this`语句即可。

这是一个自定义的`Book`对象的示例，其中包含两个方法，`raisePrice()`和`releaseNewEdition()`，两者都使用方法链：

```
class Book {
  constructor(title, author, price, publishedDate) {
    this.title = title;
    this.author = author;
    this.price = price;
    this.publishedDate = publishedDate;
  }

  raisePrice(percent) {
    const increase = this.price*percent;
    this.price += Math.round(increase)/100;
    return this;
  }

  releaseNewEdition() {
    // Set the pulishedDate to today
    this.publishedDate = new Date();
    return this;
  }
}

const book = new Book('I Love Mathematics', 'Adam Up', 15.99,
 new Date(2010, 2, 2));

// Raise the price 15% and then change the edition, using method chaining
console.log(book.raisePrice(15).releaseNewEdition());
```

## 讨论

直接在另一个方法的结果上调用一个方法，形成单一的代码语句，被称为 *方法链*。以下是一个使用字符串和 `replaceAll()` 方法的示例。因为 `replaceAll()` 返回一个新字符串，您可以在该字符串上再次调用 `replaceAll()`，并得到一个 *第三个* 字符串：

```
const safePieceOfHtml =
 originalPieceOfHtml.replaceAll('<', '&lt;').replaceAll('>', '&gt;');
```

方法链不一定要使用相同的方法。它可以与返回对象的任何方法一起工作。考虑以下代码如何通过将 `concat()` 和 `sort()` 的调用链接来连接两个数组，然后对结果数组进行排序：

```
const evens = [2, 4, 6, 8];
const odds = [1, 3, 5, 7, 9];

const evensAndOdds = evens.concat(odds).sort();
console.log(evensAndOdds);  // [1, 2, 3, 4, 5, 6, 7, 8, 9]
```

在内置的 JavaScript 对象和许多 JavaScript 库和框架中广泛使用链式调用。要在自己的类中使用此模式，您只需在方法末尾返回对 `this` 的引用即可。调用代码可以忽略此引用，或者使用它执行方法链。

在当前示例中，调用 `Book` 上的方法会更改对象并返回对更改后对象的引用。调用者可以忽略返回值，因为他们已经有了对 `Book` 对象的引用。但是，许多函数式编程纯粹主义者会做一些不同的事情。他们编写返回更改后对象 *副本* 的方法，同时保持原始对象不变。以下是如何实现此模式的示例：

```
class Book {
  constructor(title, author, price, publishedDate) {
    this.title = title;
    this.author = author;
    this.price = price;
    this.publishedDate = publishedDate;
  }

  getRaisedPriceBook(percent) {
    const increase = this.price*percent;
    return new Book(this.title, this.author, Math.round(increase)/100,
     this.publishedDate);
  }

  getNewEdition() {
    return new Book(this.title, this.author, this.price, new Date());
  }
}
```

这种模式不影响方法链工作的方式，但确实意味着调用者需要接受返回值，否则他们将看不到变化。

# 向类添加静态方法

## 问题

您希望创建一个与类相关但可以在不创建对象的情况下调用的实用方法。

## 解决方案

在方法前加上 `static` 关键字。确保您的方法不尝试使用任何实例字段、属性或方法。这里有一个名为 `Book.isEqual()` 的静态方法示例：

```
class Book {
  constructor(isbn, title, author, publishedDate) {
    this.isbn = isbn;
    this.title = title;
    this.author = author;
    this.publishedDate = publishedDate;
  }

  static isEqual(book, otherBook) {
    if (book instanceof Book && otherBook instanceof Book) {
      // Books are deemed equal if their ISBNs match,
      // irrespective of dashes
      return (book.isbn.replaceAll('-','') === otherBook.isbn.replaceAll('-',''));
    }
    else {
      return false;
    }
  }
}
```

您通过类名访问静态方法（如 `Book.isEqual()`）。不能通过对象变量访问它。

```
const firstPrinting = new Book('978-3-16-148410-0', 'A.I. Is Not a Threat',
 'Anne Droid', new Date(2019, 2, 2));
const secondPrinting = new Book('978-3-16-148410-0', 'A.I. Is Not a Threat',
 'A. Droid', new Date(2021, 2, 10));

// Compare the books with the static method
const sameBook = Book.isEqual(firstPrinting, secondPrinting);
// sameBook = true

// This doesn't work, because isEqual isn't available in Book instances
sameBook = firstPrinting.isEqual(firstPrinting, secondPrinting);
```

## 讨论

静态方法具有与类逻辑相关但不绑定于特定实例的功能。`Array.isArray()` 方法是一个很好的例子 —— 它允许您测试任何对象是否为数组，而无需首先创建数组对象。偶尔，类完全由静态方法组成。JavaScript 的 `Math` 类就是一个很好的例子。

在当前示例中，您可能希望为 `Book` 类提供与处理或验证 ISBN 相关的静态方法。您还可以使用静态方法来决定如何复制或比较某个类的对象。解决方案通过静态 `isEqual()` 方法演示了这一原则。您还可以添加一个 `compare()` 方法，使您能够对数组中的对象进行排序（如 “按属性值排序对象数组” 中所示）。

在静态方法中，`this`指的是当前类，而不是对象实例。这可能会导致问题，因为您的代码仍然允许在`this`中存储数据（或检索数据）。它只是可能没有您期望的效果。本质上，静态`this`中的所有内容都像一个类范围的全局变量，最好避免使用。

###### 提示

如果你想要一个静态方法调用另一个静态方法，可以使用`this`关键字。例如，如果你想要在`Book`类的另一个静态方法中调用静态方法`isEqual()`，你可以使用`Book.isEqual()`或`this.isEqual()`，这可能更清晰。

属性的`set`和`get`方法也可以是静态的，尽管它们的使用有时是有争议的。例如，您可以使用静态 getter 来存储常量，就像这样：

```
class Book {
  constructor(isbn, title, author, publishedDate) {
    this.isbn = isbn;
    this.title = title;
    this.author = author;
    this.publishedDate = publishedDate;
  }

  // Create a static, read-only Books.isnbnPrefix property
  static get isbnPrefix() {
    return '978-1';
  }
}
```

您可以编写一个静态 setter，它在您的应用程序中类似于全局变量。然而，由于没有静态构造函数，您将被迫在某个地方运行代码以分配初始值。这并不特别清晰，因此正在开发一种[new static property syntax](https://oreil.ly/7O28H)，目前更现代的浏览器版本已经支持。它允许您使用类似变量的语法设置公共静态属性：

```
class Book {
  // Create a static Book.isbnPrefix property
  static isbnPrefix = '978-1';

  constructor(isbn, title, author, publishedDate) {
    this.isbn = isbn;
    this.title = title;
    this.author = author;
    this.publishedDate = publishedDate;
  }
}
```

然而，最好完全避免这种语言特性——或者至少在 JavaScript 中的使用更为规范的未来数据时再考虑使用。

# 使用静态方法创建对象

## 问题

您想要创建一个生成预配置对象的方法，可能是为了绕过 JavaScript 的单构造函数限制。

## 解决方案

在您的类中添加一个静态方法，创建并返回您想要的对象。以下是一个`Book`类的示例，您可以通过构造函数或通过静态方法`Book.createSequel()`创建它：

```
class Book {
  constructor(title, firstName, lastName) {
    this.title = title;
    this.firstName = firstName;
    this.lastName = lastName;
  }

  static createSequel(prevBook, title) {
    return new Book(title, prevBook.firstName, prevBook.lastName);
  }
}
```

以下是如何使用静态方法：

```
// Create a Book with the usual constructor
const book = new Book('Good Design', 'Polly', 'Morfissim');

// Create a sequel with the static method
const sequel = Book.createSequel(book, 'Even Gooder Design');
console.log(sequel);
```

## 讨论

使用静态方法，您可以实现不同类型的*创建型*模式——基本上是帮助您创建类的预配置实例的模式。例如，JavaScript 的`Date`类有一个`now()`属性，返回一个新的`Date`对象，自动设置为当前日期和时间。

此方法特别适用于创建更复杂的对象组合。例如，您可以通过`Book.createTrilogy()`方法扩展前面的示例，以获取一个包含三个`Book`对象的数组。在这个示例中，`Book`对象共享一个`Author`对象，这意味着如果您更新`Author`对象，所有链接到它的`Book`实例都会看到这个变化：

```
class Author {
  constructor(firstName, lastName) {
    this.firstName = firstName;
    this.lastName = lastName;
  }
}

class Book {
  constructor(title, author) {
    this.title = title;
    this.author = author;
  }

  static createSequel(prevBook, title) {
    return new Book(title, prevBook.author);
  }

  static createTrilogy(author, title1, title2, title3) {
    return [new Book(title1, author),
      new Book(title2, author),
      new Book(title3, author)];
  }
}

// Create a trilogy of three books with a factory method
const author = new Author('Koh','Der');
const books = Book.createTrilogy(author, 'A Sea of Fire', 'A Sea of Ice',
 'A Sea of Water');
console.log(books);
```

与构造函数不同，您可以添加多少个静态方法来支持不同的对象创建场景。

###### 注意

有时这些静态方法被称为*工厂方法*，尽管这个描述在技术上并不精确。在面向对象设计理论中，当你不知道正在创建的对象的确切类型时，会使用工厂模式。例如，你可以编写一个`createBook()`方法，检查你提供的参数并返回`TechBook`类或`FictionBook`类的实例，它们都继承自基类`Book`。在 JavaScript 中也可以实现这种设计，但人们对语言如何处理这种更重的经典面向对象抽象的看法不一。

# 从另一个类继承功能

## 问题

你想创建一个自定义类，继承另一个类的功能。

## 解决方案

使用继承，一个或多个*子*类从*父*类派生。在代码中建模这一点时，声明子类时使用`extends`关键字：

```
public class SomeChild extends SomeParent {

}
```

这里有一个例子，展示了一个从名为`Shape`的基础父类继承的`Triangle`类：

```
// This is the parent class
class Shape {
  getArea() {
    return null;
  }
}

// This is a child class
class Triangle extends Shape {
  constructor(base, height) {
    // Call the base class constructor
    super();

    this.base = base;
    this.height = height;
  }

  getArea() {
    return this.base * this.height/2;
  }
}
```

在这个例子中，父类（`Shape`）没有任何有用的功能。`getArea()`方法只是一个占位符。但在其他情况下，基类本身可能会有用。例如，你可以使用`Book`类的继承来创建`EBook`子类或`Person`类的`Customer`。

在 JavaScript 这样的弱类型语言中，如果你只打算使用`Triangle`，似乎没有必要构建一个从`Shape`继承的`Triangle`。而且在这种情况下，通常确实如此！但是当你使用单个父类来标准化更多子类时，潜在的价值就显现出来：

```
class Circle extends Shape {
  constructor(radius) {
    super();
    this.radius = radius;
  }

  getArea() {
    return Math.PI * this.radius**2;
  }
}

class Square extends Shape {
  constructor(length) {
    super();
    this.length = length;
  }

  getArea() {
    return this.length**2;
  }
}
```

现在可以这样编写代码：

```
// Create an array of different shapes
const shapes = [new Triangle(15, 8), new Circle(8), new Square(7)];

// Sort them by area from smallest to largest
shapes.sort( (a,b) => a.getArea()-b.getArea() );

console.log(shapes);
// New order: Square, Triangle, Circle
```

当然，JavaScript 是一种弱类型语言，即使它们没有共享定义该方法的父类，你也可以在`Triangle`、`Circle`和`Square`对象上调用`getArea()`。但是使用继承来规范这种接口可以帮助明确这些要求。如果你需要使用`instan⁠ce​of`来测试对象是否是某种类型，这也很重要（“检查对象类型”）：

```
const triangle = new Triangle(15, 8);

if (triangle instanceof Shape) {
  // We end up here, because triangle is a Triangle which is a Shape
}
```

## 讨论

如果你不为子类编写构造函数，JavaScript 会自动创建一个。该构造函数调用基类构造函数（但不提供参数）。

如果为子类编写了构造函数，*必须*调用父类构造函数。否则，在尝试创建实例时会收到`ReferenceError`。要调用父类构造函数，使用`super()`关键字：

```
constructor(length) {
  super();
}
```

如果父类构造函数接受参数，你应该像创建对象时那样将它们传递给`super()`。以下是一个扩展`Book`的`EBook`类的示例：

```
class Book {
  constructor(title, author, publishedDate) {
    this.title = title;
    this.author = author;
    this.publishedDate = publishedDate;
  }
}

class EBook extends Book {
  constructor(title, author, publishedDate, format) {
    super(title, author, publishedDate);
    this.format = format;
  }
}
```

你还可以使用`super()`调用父类中的其他方法或属性。例如，如果一个子类想调用父类`formatString()`的实现，它会调用`super.formatString()`。

类是 JavaScript 相对较晚引入的功能。虽然它们支持继承，但你在传统面向对象语言中可能习惯的许多其他工具，如抽象基类、虚方法和接口，在 JavaScript 中都没有对应的概念。一些开发者喜欢 JavaScript 轻量级的特性和原型的重视，而另一些则觉得缺少构建大型复杂应用程序的重要工具。如果你属于后者，最好考虑使用 TypeScript，这是 JavaScript 的一个更严格的超集。

但继承并非没有其缺点。它可能会促使你编写紧密耦合的类，这些类相互依赖并且难以适应未来的变更。更糟糕的是，往往很难识别这些依赖关系，开发者变得不愿意对父类进行更改（这种情况被称为*脆弱基类*问题）。正因为这些问题，现代开发往往更喜欢聚合对象组而不是使用继承关系。例如，不是建立一个扩展`Person`的`Employee`类，而是创建一个包含`Person`属性及其它所需细节的`Employee`对象。这种模式称为*组合*：

```
class Person {
  constructor(firstName, lastName) {
    this.firstName = firstName;
    this.lastName = lastName;
  }
}

class Employee {
  constructor(person, department, hireDate) {
    // person is a full-fledged Person object
    this.person = person;

    // These properties hold the extra, nonperson information
    this.department = department;
    this.hireDate = hireDate;
  }
}

// Create an Employee object that's composed of a Person object
// and some extra details
const employee = new Employee(new Person('Mike', 'Scott'), 'Sales', new Date());
```

## 补充：原型链

你可能还记得 JavaScript 类功能创建对象的原型。这个原型包含了所有方法和属性的实现，并在该类的所有实例之间共享。原型也是实现继承的秘诀。当一个类扩展另一个类时，它们被链接在一个*原型链*中。

例如，考虑`Shape`和`Triangle`之间的关系。`Triangle`类有一个原型，其中保存了你为子类定义的任何内容。然而，该原型还有*它自己*的原型，即`Shape`类的原型，其中包含所有其成员。`Shape`原型也有它自己的原型：基础的`Object.prototype`，这样便形成了原型链。

继承可以无限层级地延伸，因此原型链可以变得非常长。当你调用像`Triangle.getArea()`这样的方法时，JavaScript 会搜索原型链。它首先在`Triangle`原型中查找方法，然后是`Shape`原型，最后是`Object`原型（如果找不到匹配的方法，则会报错）。

当然，JavaScript 类是相对较新的，而原型自从语言的第一个版本以来就存在。因此，即使不使用 JavaScript 类，你也可以创建类似继承的关系。有时这与老式的构造函数模式（“使用构造函数模式创建自定义类”）结合使用，结果是一些相当不优雅的代码。

```
// This will be the parent class
function Person(firstName, lastName) {
  this.firstName = firstName;
  this.lastName = lastName;
}

// Add the methods you want to the Person class
Person.prototype.greet = function() {
  console.log('I am ' + this.firstName + ' ' + this.lastName);
}

// This will be the child class
function Employee(firstName, lastName, department) {
  // The Object.call() method allows you to chain constructor functions
  // It binds the Person constructor to this object's context
  Person.call(this, firstName, lastName);

  // Add extra details
  this.department = department;
}

// Link the Person prototype to the Employee function
// This establishes the inheritance relationship
Employee.prototype = Object.create(Person.prototype);
Employee.prototype.constructor = Employee;

// Now add the methods you want to the Employee class
Employee.prototype.introduceJob = function() {
  console.log('I work in ' + this.department);
}

// When you create an instance of the Employee function, its prototype
// is chained back to the Person prototype
const newEmployee = new Employee('Luke', 'Takei', 'Tech Support');

// You can call Person methods and Employee methods
newEmployee.greet();          // 'I am Luke Takei'
newEmployee.introduceJob();   // 'I work in Tech Support'
```

这种模式现在*应该*大部分已经过时了，因为类给你提供了一个更清晰的方法来创建继承关系。但它仍然存在于许多长期存在的代码库中。

# 使用模块组织你的 JavaScript 类

## 问题

你想要将你的类封装在一个单独的命名空间中，以便于重用并防止与其他库的命名冲突。

## 解决方案

使用 ES6 引入的模块系统。有三个步骤：

1.  决定哪些功能代表一个完整的模块。将那些类、函数和全局变量的代码放在一个单独的脚本文件中。

1.  选择你想要*导出*（在其他文件的其他脚本中可用）的代码细节。

1.  在另一个脚本中，*导入*你想要使用的功能。

这是一个模块的示例；我们将其存储在名为 *lengthConverterModule.js* 的文件中：

```
const Units = {
  Meters: 100,
  Centimeters: 1,
  Kilometers: 100000,
  Yards: 91.44,
  Feet: 30.48,
  Miles: 160934,
  Furlongs: 20116.8,
  Elephants: 625,
  Boeing747s: 7100
};

class InvisibleLogger {
  static log() {
    console.log('Greetings from the invisible logger');
  }
}

class LengthConverter {
  static Convert(value, fromUnit, toUnit) {
    InvisibleLogger.log();
    return value*fromUnit/toUnit;
  }
}

export {Units, LengthConverter}
```

最重要的是末尾的 `export` 语句。它列出了所有将被其他代码文件访问的函数、变量和类。在这个例子中，`Units` 常量（实际上只是一个枚举）和 `LengthConverter` 类是可用的，而 `InvisibleLogger` 类则不是。

###### 注意

当你创建模块文件时，有时建议使用扩展名为 *.mjs*。 *.mjs* 扩展名清楚地表明你在使用 ES6 模块，它有助于 Node 和 Babel 等工具自动识别这些文件。但是，如果你的 web 服务器没有正确配置为使用正确的 MIME 类型（`text/javascript`）提供 *.mjs* 文件，那么 *.mjs* 扩展名也可能会引起问题，就像普通的 *.js* 文件一样。因此，在这个例子中我们不使用它。

现在你可以将需要的功能导入到另一个模块中。你可以将这个模块写成一个单独的文件，或者像我们在这里做的那样在网页中使用一个 `<script>` 块。但无论哪种方式，你的 `<script>` 标签必须包含 `type="module"` 属性。

这是完整的页面，包括触发 `doSampleConversion()` 测试的按钮：

```
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <meta http-equiv="X-UA-Compatible" content="ie=edge" />
    <title>Module Test</title>
  </head>
  <body>
    <h1>Module Test</h1>
    <button id="convertButton">Do Sample Conversion</button>

<script type="module">
  import {Units, LengthConverter} from './lengthConverterModule.js';

  function doSampleConversion() {
    const lengthInMiles = 495;

    // This works because you have access to LengthConverter and Units
    const lengthInElephants =
     LengthConverter.Convert(lengthInMiles, Units.Feet, Units.Yards);
    alert(lengthInElephants);

    // This wouldn't work, because you don't have access to InvisibleLogger
    //InvisibleLogger.log();
  }

  // Connect the button
  document.getElementById('convertButton').addEventListener('click',
   doSampleConversion);
</script>

  </body>
</html>
```

## 讨论

多年来，JavaScript 使用了许多模块系统，最显著的是 Node 和 npm。但自 ES6 以来，JavaScript 已经有了自己的模块标准，所有现代浏览器都原生支持。

在创建带有模块的解决方案之前，有几个你应该了解的考虑因素：

+   浏览器安全限制意味着你不能从本地文件系统运行模块示例。相反，你需要将你的示例托管在开发 Web 服务器上（如 “设置本地测试服务器” 中描述的）。

+   模块被锁定在它们自己独特的“模块”作用域中。你无法从普通的非模块脚本访问模块。同样，你也无法从开发者控制台访问模块。

+   你无法从页面的 HTML 中访问模块。这意味着你不能使用像`onclick`这样的 HTML 属性来连接事件处理程序，因为页面无法访问模块内部的事件处理程序。相反，你的模块代码需要通过`window`或`document`来访问周围的浏览器上下文。

+   模块会自动以严格模式执行（“使用严格模式捕捉常见错误”）。

模块功能只能被导入到另一个模块中。如果你想在网页中为一个模块创建一个`<script>`块，确保将`type`属性设置为`module`，否则模块导入功能将无法工作：

```
<script type="module">
```

当你从一个模块导入功能时，你必须在`import`语句的`from`部分指定模块的文件路径。模块支持一种方便的快捷方式，允许你以`./`开头的相对路径，因此`./lengthConverterModule.js`指向当前文件夹中的*lengthConverterModule.js*文件：

```
import {Units, LengthConverter} from './lengthConverterModule.js';
```

在导入模块功能时，你使用的命名具有相当大的灵活性。你可以将你的导入包装在一个*模块对象*中，这是一种特殊的容器，用于对所有内容进行命名空间管理。下面是一个将每种导出类型导入到名为`LConvert`的模块对象中的示例：

```
import * as LConvert from './lengthConverterModule.js';

// Now you can access LengthConverter as LConvert.LengthConverter
```

注意，在使用模块对象时不需要大括号。

你还可以在你的模块中设置一个*默认*导出：

```
export default LengthConverter
```

然后你可以使用任何名称进行导入：

```
import LConvert from './lengthConverterModule.js';
```

默认导出功能与其他模块系统中的类似功能匹配。这使得那些模块更容易迁移到 ES6 模块标准中。

ES6 模块很可能最终会成为 JavaScript 中主导的模块标准。但是今天，在 npm 中实现 ES 模块仍然存在一些问题。在可预见的未来，这意味着开发人员将至少需要处理两种模块标准：ES6 标准，这是现代浏览器本地识别的，以及较老的 CommonJS 标准，在 Node 和 npm 生态系统中成熟和广泛使用。

## 参见

有关如何在 Node 和 npm 中使用 CommonJS 模块的信息，请参见第十八章。
