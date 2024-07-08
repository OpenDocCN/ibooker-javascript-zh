# 第七章：对象

JavaScript 中有两类广泛的类型。一方面是一小组*原始*类型，比如字符串和数字。另一方面是真正的对象，所有这些对象都源自 JavaScript 的`Object`。

JavaScript 的内置对象很容易识别。它们有构造函数，通常会使用`new`关键字实例化它们。像数组、`Date`、错误对象、`Map`和`Set`集合，以及`RegExp`正则表达式等基本组件都是对象。

JavaScript 对象在重要方面也与传统面向对象编程语言中的对象不同。例如，JavaScript 允许你创建基本`Object`类型的实例，并在运行时附加新属性和函数。事实上，你可以取一个活动对象——任何对象——并修改其成员，而无需遵守类定义。

在本章中，你将更仔细地查看 JavaScript 的`Object`类型的功能和怪癖。你将看到如何使用核心`Object`功能来检查、扩展和复制所有类型的对象。在下一章中，你将进一步学习如何规范自定义对象的最佳实践。

# 检查对象是否为某种类型

## 问题

你有一个神秘对象，想要确定它的类型。

## 解决方案

使用`instanceof`运算符：

```
const mysteryObject = new Date(2021, 2, 1);

if (mysteryObject instanceof Date) {
  // We end up here because mysteryObject is a Date
}
```

你可以使用非运算符（`!`）来测试一个对象是否*不是*某种类型的实例。但确保使用括号将`!`应用于整个`instanceof`条件：

```
if (!(mysteryObject instanceof Date)) {
  // You get here if mysteryObject isn't a Date
}

// Don't make this mistake!
if (!mysteryObject instanceof Date) {
  // This code never runs
}
```

`instanceof`运算符存在一个缺陷。它无法处理原始值，比如数字、字符串、布尔值、`BigInt`值、`null`和`undefined`。以下是问题的演示：

```
const testNumber = 42;
if (testNumber instanceof Number) {
  // This code never runs
}

const testString = 'Hello';
if (testString instanceof String) {
  // This code never runs
}

// The following two tests work because the primitives are wrapped in objects,
// but that's uncommon in modern JavaScript.
const numberObject = new Number(42);
if (numberObject instanceof Number) {
  // This code runs
}

const stringObject = new String('Hello');
if (stringObject instanceof String) {
  // This code runs
}
```

如果你要测试一个可能包含原始数据类型之一的变量，解决方案是使用`typeof`运算符。与`instanceof`不同，`typeof`会为你提供九个预定义的字符串值之一（如“检查是否存在、非空字符串”中所述）。如果得到一个`object`值，你可以使用`instanceof`运算符深入挖掘：

```
const mysteryPrimitive = 42;
const mysteryObject = new Date();

if (typeof mysteryPrimitive === 'number') {
  // This code runs
}

if (typeof mysteryObject === 'object') {
  // This code runs, because a Date is an object, not a primitive

  if (mysteryObject instanceof Date) {
    // This code also runs
  }
}
```

## 讨论

`instanceof`运算符通过检查对象的*原型链*来工作，这是在“额外：原型链”中解释的一个概念。根据对象的构造方式，原型链中可能有几种类型（类似于传统面向对象编程语言中对象从一系列类继承的方式）。例如，每个对象在其链的基础上都有`Object`原型，因此这总是成立：

```
if (mysteryObject instanceof Object) {
  // This is true, unless mysteryObject is a primitive type
}
```

记住，原始值不仅包括数字、字符串和布尔值。它们还包括专门的`BigInt`和`Symbol`，以及特殊值`null`和`undefined`。如果你使用`instanceof Object`测试，所有这些值都会返回`false`。

# 使用对象字面量来捆绑数据

## 问题

你想要将几个变量组合在一起，创建一个基本的数据包。

## 解决方案

使用*对象字面量*语法创建`Object`类型的新实例。您不使用`new`关键字，甚至不命名`Object`类型。相反，您只需编写一组`{}`括号，其中包含一个逗号分隔的属性列表。每个属性由属性名称后跟冒号，然后是属性值组成：

```
const employee = {
  employeeId: 402,
  firstName: 'Lisa',
  lastName: 'Stanecki',
  birthDate: new Date(1995, 8, 15)
};

console.log(employee.firstName);  // 'Lisa'
```

当然，您可以在创建对象后添加附加属性，就像处理任何 JavaScript 对象一样：

```
employee.role = 'Manager';
```

即使您已使用`const`声明对象，此技术也适用，因为对象字面量是*引用类型*，而不是值（与其他语言中的结构体不同）。添加属性会更改对象，但不会更改引用。 （另一方面，在此示例中，将`employee`变量分配给新对象是不允许的，因为该操作将更改引用。）

## 讨论

对象字面量语法为您提供了快速创建简单对象的最清晰、最紧凑的方式。但是，它只是显式创建新`Object`实例并分配属性的快捷方式，就像这样：

```
const employee = new Object();
employee.employeeId = 402;
employee.firstName = 'Lisa';
employee.lastName = 'Stanecki';
employee.birthDate = new Date(1995, 8, 15);
```

或者您可以使用键值语法：

```
const employee = new Object();
employee['employeeId'] = 402;
employee['firstName'] = 'Lisa';
employee['lastName'] = 'Stanecki';
employee['birthDate'] = new Date(1995, 8, 15);
```

对象字面量语法最好的特性之一是它处理嵌套对象的方式，例如本例中的`birthPlace`：

```
const employee = {
  employeeId: 402,
  firstName: 'Lisa',
  lastName: 'Stanecki',
  birthPlace: {country: 'Canada', city: 'Toronto'}
};

console.log(employee.birthPlace.city);  // 'Toronto'
```

在 JavaScript 的眼中，对象字面量是基本`Object`类型的一个实例。这种简单性使得从任意的临时数据组合创建对象变得容易，但也有代价——您的对象没有有意义的*身份*。

是的，您可以测试对象是否具有某个特定属性（“检查对象是否具有属性”），或者枚举其所有属性（“遍历对象的所有属性”）。但您不能使用`instanceof`来针对自定义对象类型进行测试。换句话说，没有可编程的合同，也没有简单的方法来验证您的对象是否符合预期。如果您需要使用在代码中传递的更持久的对象、建模复杂实体并包含自己的方法，则应考虑使用正式类（“创建可重用类”）。

###### 注意

您可能会想到通过创建接受参数并构建相应对象的工厂函数来简化对象创建过程。虽然这种方法本质上没有问题，但有一个更强大和常规的替代方案。一旦您想要使用相同结构构建多个对象，请考虑使用类（“创建可重用类”）。

## 另请参阅

要查找对象字面量上的所有属性，请参见“遍历对象的所有属性”。要升级到正式类定义，请参见“创建可重用类”。

## 额外：计算属性名

正如您所知，您可以通过两种方式向任何 JavaScript 对象添加新属性。您可以使用点语法和属性名称：

```
employee.employeeId = 402;
```

或者键值语法：

```
employee['employeeId'] = 402;
```

这两种方法并不等价。当你使用键值语法时，属性名被存储为字符串，这意味着你有机会在运行时生成属性名。这被称为*计算属性名*，在某些可扩展性场景中非常重要。（例如，想象一下，如果你正在获取一些外部数据并使用它来创建匹配的对象。）

```
const dynamicProperty = 'nickname';
const dynamicPropertyValue = 'The Izz';

employee[dynamicProperty] = dynamicPropertyValue;
// Now employee.nickname = 'The Izz'

const i = 10;
employee['sequence' + i] = 1;
// Now employee.sequence10 = 1
```

计算属性名始终转换为字符串。它们支持普通变量名中不允许的字符，如空格。例如，这是可能的（尽管这是一个非常糟糕的主意）：

```
const employee = {};
const today = new Date();

employee[today] = 42;

// This reveals that 42 is stored in a property that has a long string name like
// "Tue May 04 2021 08:18:16 GMT-0400 (Eastern Daylight Time)"
console.log(employee);
```

对象字面量语法还允许你创建计算属性。但由于它不使用具有字符串键名的格式，你需要用方括号括起每个计算属性名。这是它的样子：

```
const dynamicProperty = 'nickname';
const dynamicPropertyValue = 'The Izz';
const i = 10;

const employee = {
  employeeId: 402,
  firstName: 'Lisa',
  lastName: 'Stanecki',
  [dynamicProperty]: dynamicPropertyValue,
  ['sequence' + i]: 1
};
```

###### 提示

如果你动态创建属性名，可能会遇到需要确保属性名唯一的情况。有各种自制的解决方法：检查属性并添加一个序列号直到得到唯一的内容，或者只使用 GUID（全局唯一标识符）。但 JavaScript 提供了一个内置解决方案，即`Symbol`类型，这是你最好的选择（参见“创建绝对唯一对象属性键”）。

# 检查对象是否有属性

## 问题

你想在运行时检查对象是否具有给定的属性。

## 解决方案

使用`in`运算符按名称查找属性：

```
const address = {
  country: 'Australia',
  city: 'Sydney',
  streetNum: '412',
  streetName: 'Worcestire Blvd'
};

if ('country' in address) {
  // This code runs, because there is an address.country property
}

if ('zipCode' in address) {
  // This code does not run, because there is no address.zipCode property
}
```

## 讨论

如果尝试读取一个不存在的属性，则会得到值`undefined`。你可以测试`undefined`，但这并不能完全保证该属性不存在。（技术上讲，可能存在一个属性并将其设置为`undefined`，这种情况下属性仍然存在，但你的测试会漏掉它。）查找属性的更好方法是使用`in`运算符。

`in`运算符搜索对象及其原型链。这意味着，如果你创建一个从另一个对象`Animal`派生的对象`Dog`，`in`测试会在`Dog`或`Animal`中定义了属性时返回`true`。或者，你可以使用`hasOwnProperty()`方法，它仅搜索当前对象，忽略继承的属性。

```
const address = {
  country: 'Australia',
  city: 'Sydney',
  streetNum: '412',
  streetName: 'Worcestire Blvd'
};

console.log(address.hasOwnProperty('country'));  // true
console.log(address.hasOwnProperty('zipCode'));  // false
```

关于使用继承的更多信息，请参见“从另一个类继承功能”。

## 另请参阅

“迭代对象的所有属性”展示了如何将对象的所有属性检索到一个数组中。“测试空对象”展示了如何测试对象是否为空。

# 迭代对象的所有属性

## 问题

你想检查对象中的所有属性。

## 解决方案

使用静态的`Object.keys()`方法获取包含对象属性名的数组。例如，这段代码：

```
const address = {
  country: 'Australia', city: 'Sydney', streetNum: '412',
  streetName: 'Worcestire Blvd'
};

const properties = Object.keys(address);

// Show every property and its value
for (const property of properties) {
  console.log(`Property: ${property}, Value: ${address[property]}`);
}
```

创建此控制台输出：

```
Property: country, Value: Australia
Property: city, Value: Sydney
Property: streetNum, Value: 412
Property: streetName, Value: Worcestire Blvd
```

这种技术——检查对象，找到其所有属性并显示它们——与当你向 `console.log()` 方法传递一个对象时的操作类似。

## Discussion

使用 `Object.keys()` 时，你会检索所有属性名称（也称为*键*）。但你仍需要查找对象中对应的值。你不能使用点语法来做到这一点（`object.propertyName`），因为属性是一个字符串。而是使用类似数组的索引器语法（`object['propertyName']`）。属性通常以定义的顺序出现，但 JavaScript 不保证顺序。

`Object.keys()` 方法通常用于计算对象的属性数（或*长度*）：

```
const address = {
  country: 'Australia', city: 'Sydney', streetNum: '412',
  streetName: 'Worcestire Blvd'
};

properties = Object.keys(address);
console.log(`The address object has a length of ${properties.length}`);
// (In this example, the length is 4.)
```

`Object.keys()` 方法只是反映 JavaScript 对象的许多可能解决方案之一。但它是一个很好的默认起点，因为它忽略继承的属性和不可枚举的属性，这是大多数情况下所希望的行为。

另一个选择是使用 `for...in` 循环，如下所示：

```
for (const property in address) {
  console.log(`Property: ${property}, Value: ${address[property]}`);
}
```

`for...in` 循环沿原型链遍历以查找对象继承的属性。在此示例中，使用名为 `address` 的对象字面量没有区别。然而，如果经常需要反映对象，无意中使用 `for...in` 循环而应该使用 `Object.keys()` 可能会对 `performance` 产生不利影响。

###### Note

与你可能期望的相反，`for...in` 循环与 `in` 运算符的覆盖范围略有不同。`in` 运算符检查*所有*属性，包括不可枚举的属性、符号属性和继承属性。`for...in` 循环找到继承的属性，但忽略不可枚举的属性和符号属性。

JavaScript 还有其他更专门的函数，用于查找对象的不同子集属性。例如，`getOwnPropertyNames()` 函数忽略继承的属性，而 `getOwnPropertyDescriptors()` 函数则忽略继承的属性，但还会找到不可枚举的属性和符号属性，这些通常用于可扩展性（参见 “Creating Absolutely Unique Object Property Keys”）。Table 7-1 概述了这些不同的方法。要获取更详细的信息，请参阅 Mozilla 开发者网络有关 [不同属性搜索函数](https://oreil.ly/rbd7z) 的完整信息。

表 7-1\. 查找对象属性的不同方法

| 方法 | 返回 | 获取可枚举属性 | 获取不可枚举属性 | 获取符号属性 | 包括继承属性 |
| --- | --- | --- | --- | --- | --- |
| `Object.keys()` | 一个属性名称的数组 | 是 | 否 | 否 | 否 |
| `Object.values()` | 一个属性值的数组 | 是 | 否 | 否 | 否 |
| `Object.entries()` | 一个属性数组的数组，每个数组包含一个属性名称和相应的值 | 是 | 否 | 否 | 否 |
| `Object.getOwnPropertyNames()` | 属性名数组 | 是 | 是 | 否 | 否 |
| `Object.getOwnProperty​Sym⁠bols()` | 属性名数组 | 否 | 否 | 是 | 否 |
| `Object.getOwnProperty​De⁠scriptors()` | 属性描述符对象数组，类似于使用 `defineProperty()` 时的情况（“自定义属性定义方式”） | 是 | 是 | 是 | 否 |
| `Reflect.ownKeys()` | 属性名数组 | 是 | 是 | 是 | 否 |
| `for...in` 循环 | 每个属性名 | 是 | 否 | 否 | 是 |

## 参见

“检查对象是否具有属性” 解释了如何使用 `in` 运算符来检查单个属性。

# 测试空对象

## 问题

你想确定一个对象是否为空（没有属性）。

## 解决方案

使用 `Object.keys()` 获取属性数组，并检查其 `length` 是否为 0：

```
const blankObject = {};

if (Object.keys(blankObject).length === 0) {
  // This code runs because there's nothing in this object
}

const objectWithProperty = {price: 47.99};
if (Object.keys(objectWithProperty).length === 0) {
  // This code won't run, because objectWithProperty isn't empty
}
```

## 讨论

可以使用对象字面量语法创建空对象：

```
const blankObject = {};
```

或通过使用 `new` 创建 `Object` 的实例：

```
const blankObject = new Object();
```

空对象也可以通过其他不太常见的方法产生，例如使用 `delete` 运算符删除现有对象的属性：

```
const objectWithProperty = {price: 47.99};
delete objectWithProperty.price;

if (Object.keys(objectWithProperty).length === 0) {
  // This code runs, because objectWithProperty had its only property removed
}
```

因为对象是引用类型，你不能简单地比较一个空对象和另一个空对象。例如，这个测试无法识别你的未知对象是否为空：

```
const blankObject = {};
const unknownObject = {};

if (unknownObject === blankObject) {
  // We never get here
  // Even though unknownObject is empty, like blankObject, it holds a
  // different reference to a different memory location
}
```

许多 JavaScript 库，如 Underscore 和 Lodash，提供了用于检查对象是否为空的 `isEmpty()` 方法。但是，使用 `Object.keys()` 的测试同样简单。

# 合并两个对象的属性

## 问题

你已经创建了两个带有属性的简单对象，并且想要将它们的数据合并到单个对象中。

## 解决方案

使用扩展操作符 (`...`) 扩展两个对象，并将它们赋值给一个新对象：

```
const address = {
  country: 'Australia', city: 'Sydney', streetNum: '412',
  streetName: 'Worcestire Blvd'
};

const customer = {
  firstName: 'Lisa', lastName: 'Stanecki'
};

const customerWithAddress = {...customer, ...address};
console.log(customerWithAddress);
// The customerWithAddress now has all six properties
```

## 讨论

合并两个对象是一个简单的操作，但不是没有潜在问题。如果两个对象都有相同名称的属性，第二个对象（在前面的示例中是 `address`）的属性将悄悄地覆盖第一个对象的属性。以下是演示问题的修改版本的示例：

```
const address = {
  `country``:` `'Australia'`, city: 'Sydney', streetNum: '412',
  streetName: 'Worcestire Blvd'
};

const customer = {
  firstName: 'Lisa', lastName: 'Stanecki', `country``:` `'South Korea'`
};

const customerWithAddress = {...customer, ...address};
console.log(customerWithAddress.country);  // Shows 'Australia' 
```

在这个例子中，`country` 属性出现了两次。当两个对象合并时，首先扩展 `customer` 对象，然后是 `address` 对象。因此，`address.country` 属性将覆盖 `customer.country` 属性。

# 自定义属性定义方式

## 问题

你可以很容易地给对象添加一个新属性。但有时候你需要显式地定制你的属性，以便更好地控制它的使用方式。

## 解决方案

不要通过简单赋值来创建属性，而是使用 `Object.defineProperty()` 方法来定义它。例如，考虑以下对象：

```
const data = {};
```

让我们假设你想要添加以下两个属性，并具有以下特征：

`type`

初始值设置后不能更改，不能删除或修改，但可以枚举

`id`

设置初始值，但可以更改，不能删除或修改，并且不能枚举

使用以下 JavaScript：

```
const data = {};

Object.defineProperty(data, 'type', {
  value: 'primary',
  enumerable: true
});

// Attempt to change the read-only property
console.log(data.type); // primary
data.type = 'secondary';
console.log(data.type); // nope, still primary

Object.defineProperty(data, 'id', {
  value: 1,
  writable: true
});

// Change this modifiable property
console.log(data.id); // 1
data.id = 300;
console.log(data.id); // 300

// See what properties appear during enumeration
for (prop in data) {
  console.log(prop); // only type displays
}
```

在本例中，尝试更改只读属性会静默失败。更常见的情况是，您将处于严格模式，要么因为您的代码在一个模块中（参见 “使用 ES6 模块组织您的 JavaScript 类”），要么因为您已经在 JavaScript 文件的顶部添加了 `'use strict';` 指令。在严格模式下，试图设置只读属性会中断您的代码，并显示 `TypeError`。

## 讨论

`defineProperty()` 是一种在对象上添加属性的方式，而不是直接赋值，它使您可以控制属性的行为和状态。即使您使用 `defineProperty()` 只是设置属性名称和值，它也不同于简单地设置属性。这是因为使用 `defineProperty()` 创建的属性默认情况下是只读且不可枚举的。

`defineProperty()` 方法接受三个参数：您要设置属性的对象、属性的名称以及配置属性的描述符对象。这里事情变得更有趣。实际上，您可以使用两种类型的描述符。解决方案中的示例使用了一个 *数据描述符*，它有四个可以设置的细节：

`configurable`

控制属性描述符是否可以更改。默认为 `false`。

`enumerable`

控制属性是否可枚举。默认为 `false`。

`value`

设置属性的初始值。

`writable`

控制属性值是否可以更改。默认为 `false`。

而不是使用数据描述符，您可以使用一个 *访问器描述符*，它支持一组略有不同的选项：

`configurable`

与数据描述符相同

`enumerable`

与数据描述符相同

`get`

设置一个作为属性 getter 使用的函数，它返回属性值

`set`

设置一个作为属性 setter 的函数，它应用属性值

这是一个使用带有访问器描述符的 `defineProperty()` 的示例：

```
const person = {
  firstName: 'Joe',
  lastName: 'Khan',
  dateOfBirth: new Date(1996, 6, 12)
};

Object.defineProperty(person, 'age', {
  configurable: true,
  enumerable: true,
  get: function() {
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
});

console.log(person.age);
```

这里的 `defineProperty()` 创建了一个计算属性 (`age`)，它使用另一个属性 (`birthdate`) 进行计算。（你会注意到在 setter 或 getter 中可以使用 `this` 引用其他实例属性。）此时，对象的设计变得有点过于雄心勃勃，不适合使用对象字面量语法进行即兴创建。最好使用形式化类，它有更自然的方式来暴露相同的属性 getter 和 setter 特性（“向类添加属性”）。

可以使用 `defineProperty()` 来*更改*现有属性，而不是添加新属性。实际上，语法完全相同——唯一的区别在于您指定的属性名称已存在于对象中。但是，有一个限制条件。如果将属性设置为不可配置，则在调用 `defineProperty()` 时将会得到 `TypeError`。

## 另请参阅

“向类添加属性” 解释了如何在类上设置属性，部分重叠了 `defineProperty()` 方法的方法。 “防止对象的任何更改” 涵盖了冻结对象以防止属性更改。

# 防止对象的任何更改

## 问题

您已经定义了对象，并且现在希望确保其属性不会被其他代码重新定义或编辑。

## 解决方案

使用 `Object.freeze()` 冻结对象，防止任何和所有更改：

```
const customer = {
  firstName: 'Josephine',
  lastName: 'Stanecki'
};

// freeze the object
Object.freeze(customer);

// This statement throws an error in strict mode
customer.firstName = 'Joe';

// So does an attempt to add a property
customer.middleInitial = 'P';

// Or remove one
delete customer.lastName;
```

当您尝试更改冻结对象时，会发生两种情况之一。如果打开了严格模式，则会抛出 `TypeError` 异常。如果未打开严格模式，则操作会静默失败——对象不会更改，但您的代码会继续执行。模块中始终打开严格模式（请参阅 “使用 ES6 模块组织您的 JavaScript 类”），或者在 JavaScript 文件顶部添加 `'use strict';` 指令。

## 讨论

正如您所知，对象是引用类型，JavaScript 允许您以任何方式更改它们。您可以更改属性值，添加或删除属性，即使已使用 `const` 声明了对象变量。

然而，JavaScript 还包含了一些静态方法在 `Object` 类中，您可以使用它们来锁定对象。您有三个选择，从最不限制到最严格依次列出如下：

`Object.preventExtensions()`

防止您添加新属性。但是，您仍然可以设置属性值。您还可以使用 `Object.getOwnPropertyDescriptor()` 删除属性和配置属性。

`Object.seal()`

阻止添加、删除或配置属性。但是，您仍然可以设置属性值。有时用于捕捉对不存在属性的赋值，这是一种静默的错误。

`Object.freeze()`

禁止任何方式的属性修改。您不能配置属性，添加新属性或设置属性值。对象变得不可变。

如果您使用严格模式（除了在控制台编写测试代码时），尝试更改冻结对象会抛出 `TypeError` 异常。如果没有使用严格模式，尝试更改属性会静默失败，保留原始属性值但允许代码继续执行。

您可以使用 `Object.isFrozen()` 检查对象是否已冻结，这是一个伴随方法：

```
if (Object.isFrozen(obj)) ...
```

# 拦截并更改对象上的操作

## 问题

您希望在对象发生某些操作时运行代码，但不希望将代码*放在*对象内部。

## 解决方案

`Proxy` 类允许你拦截任何对象上的各种不同操作。下面的示例使用代理在名为 `product` 的对象上执行验证。代理确保代码可以使用不存在的属性，或者使用非数值数据类型来设置数字：

```
// This is the object that we'll watch with the proxy
const product = {name: 'banana'};

// This is the handler that the proxy uses to intercept traps
const propertyChecker = {
  set: function(target, property, value) {
    if (property === 'price') {
      if (typeof value !== 'number') {
        throw new TypeError('price is not a number');
      }
      else if (value <= 0) {
        throw new RangeError('price must be greater than zero');
      }
    }
    else if (property !== 'name') {
      throw new ReferenceError(`property '${property}' not valid`);
    }
    target[property] = value;
  }
};

// Create the proxy
const proxy = new Proxy(product, propertyChecker);

// Now, modify the product object through the proxy object
proxy.name = 'apple';

// This throws a ReferenceError
proxy.type = 'red delicious';

// This throws a TypeError
proxy.price = 'three dollars';

// This throws a RangeError
proxy.price = -1.00;

// This bypasses the proxy and succeeds
product.price = -1.00;
```

###### 提示

一旦你创建了一个对单个属性有效的代理，你可以重用它来拦截其他属性或其他对象上的操作。

## 讨论

`Proxy` 对象包装一个对象，并可用于*拦截*特定操作，然后根据操作和对象在操作时的数据提供额外或替代行为。

当你创建一个`Proxy`时，需要提供两个参数：你想监视的对象和能拦截你选择的操作的处理程序。在这里展示的解决方案中，处理程序仅拦截属性设置操作。每次拦截属性设置动作时，它都会接收目标对象、正在设置的属性以及新的属性值。然后函数会检查正在设置的属性是否是`price`。如果是，则检查它是否是一个数字。如果不是，则抛出`TypeError`。如果是数字，则检查其值是否大于零。如果不是，则抛出`RangeError`。最后，处理程序检查属性是否是`name`。如果不是，则抛出最终的异常`ReferenceError`。如果没有触发任何错误条件，则像往常一样分配属性值。

`Proxy` 对象支持大量的陷阱，它们在表 7-2 中列出。该表列出了每个陷阱，随后是处理程序函数预期的参数、预期的返回值以及它是如何触发的。

表 7-2\. 代理陷阱

| 代理陷阱 | 函数参数 | 预期返回值 | 触发陷阱的方式 |
| --- | --- | --- | --- |
| `getOwnProperty​Descrip⁠tor` | 目标, 名称 | 描述符或未定义 | `Object.getOwnPropertyDescriptor(proxy,name)` |
| `getOwnPropertyNames` | 目标 | 字符串 | `Object.getOwnPropertyNames(proxy)` |
| `getPrototypeOf` | 目标 | 任意类型 | `Object.getPrototypeOf(proxy)` |
| `defineProperty` | 目标, 名称, 描述符 | 布尔值 | `Object.defineProperty(proxy,name,desc)` |
| `deleteProperty` | 目标, 名称 | 布尔值 | `Object.deleteProperty(proxy,name)` |
| `freeze` | 目标 | 布尔值 | `Object.freeze(target)` |
| `seal` | 目标 | 布尔值 | `Object.seal(target)` |
| `preventExtensions` | 目标 | 布尔值 | `Object.preventExtensions(proxy)` |
| `isFrozen` | 目标 | 布尔值 | `Object.isFrozen(proxy)` |
| `isSealed` | 目标 | 布尔值 | `Object.isSealed(proxy)` |
| `isExtensible` | 目标 | 布尔值 | `Object.isExtensible(proxy)` |
| `has` | 目标, 名称 | 布尔值 | 名称 in proxy |
| `hasOwn` | 目标, 名称 | 布尔值 | `({}).hasOwnProperty.call(proxy,name)` |
| `get` | 目标, 名称, 接收者 | 任意类型 | `receiver[name]` |
| `set` | 目标, 名称, 值, 接收者 | 布尔值 | `receiver[name] = val` |
| `enumerator` | 目标 | 迭代器 | `for` (name in proxy)（迭代器应产生所有可枚举的自有和继承属性） |
| `keys` | 目标 | 字符串 | `Object.keys(proxy)`（仅返回可枚举的自有属性的数组） |
| `apply` | 目标，thisArg，args | 任何 | `proxy(...args)` |
| `construct` | 目标，args | 任何 | `new proxy(...args)` |

代理还可以包装内置对象，例如 `Array` 或 `Date` 对象。在以下代码中，代理用于重新定义访问数组时发生的操作语义。当进行 `get` 操作时，处理程序检查给定索引处数组的值。如果它是零（0），则返回 `false` 的值；否则返回 `true` 的值：

```
const handler = {
    get: function(array, index) {
      if (array[index] === 0) {
        return false;
      }
      else {
        return true;
      }
    }
};

const numbers = [1,0,6,1,1,0];
const proxy = new Proxy(numbers, handler);

console.log(proxy[2]);  // true
console.log(proxy[0]);  // true
console.log(proxy[1]);  // false
```

在索引为 2 的数组值不为零时返回 `true`。对于索引为零的值也是如此。然而，索引为 1 的值为零，因此返回 `false`。无论何时访问此数组代理，这种行为都是成立的。

# 克隆对象

## 问题

您想创建一个自定义对象的精确副本。

## 解决方案

使用展开操作符 (`...`) 将对象展开为一组属性，并将该属性列表放入大括号 `{}` 中以构建新对象：

```
const animal = {
  name: 'Red Fox', class: 'Mammalia', order: 'Carnivora',
  family: 'Canidae', genus: 'Vulpes', species: 'Vulpes vulpes'
};

const animalCopy = {...animal};
console.log(animalCopy.species);  // 'Vulpes vulpes'
```

## 讨论

您可能期望此语句将复制一个对象：

```
const animalCopy = animal;
```

这适用于原始类型，如字符串、数字和 `BigInt`。但对象是引用类型，赋值对象会复制引用。最终您会得到两个变量（`animal` 和 `animalCopy`）指向同一个内存对象。

要正确复制自定义对象，您需要创建一个新对象，然后迭代旧对象，复制其每个属性。您可以使用 `in` 操作符（“迭代对象的所有属性”）来进行冗长的方式，但展开操作符提供了更好的方法，因为您可以将工作压缩为一行干净的代码。

当您使用展开操作符时，您会获得对象的所有 *可枚举* 属性。这包括使用对象字面量语法创建的所有属性，或者您事后分配的任何新属性。然而，您可以使用 `Object.defineProperty()` 方法具体选择创建不可枚举属性（如 “自定义属性定义方式” 所介绍）。通常，不可枚举属性是额外的一些数据，例如另一项服务作为某种可扩展性系统的一部分添加的数据片段。

###### 注意

通常，您不希望复制不可枚举属性，因此展开操作符忽略它们是有道理的。然而，也有其他方法可行。JavaScript 对象具有特殊的内置功能，如 `Object.getOwnPropertyDescriptors()` 方法，可让您找到不可枚举属性。“迭代对象的所有属性” 更详细地解释了属性枚举。

你可能还会看到一种稍旧的克隆方法，使用`Object.assign()`方法。这相当于使用展开运算符：

```
const animalCopy = Object.assign({}, animal);
```

无论哪种方式，这些操作执行的都是*浅拷贝*。如果你的对象包含数组或其他对象作为属性，则这些细节不会被复制。相反，它们将在原始对象和新对象之间*共享*。以下是该问题的演示：

```
const student = {
  firstName: 'Tazie', lastName: 'Yang',
  testScores: [78, 88, 94, 91, 88, 96]
};

const studentCopy = {...student};

// Now there are two objects sharing the same testScores array
// We can see this if we change some details.
// This affects just the copy:
studentCopy.firstName = 'Dori';
// This affects both objects:
studentCopy.testScores[0] = 56;

console.log(student);
// {firstName: "Tazie", lastName: "Yang", testScores: [56, 88, 94, 91, 88, 96]
console.log(studentCopy);
// {firstName: "Dori", lastName: "Yang", testScores: [56, 88, 94, 91, 88, 96]
```

这不一定是问题，这取决于你想要实现的目标。但是，如果你想要复制多层深度，你需要考虑一种可以创建*深拷贝*的不同克隆方法（“深拷贝对象”）。

## 参见

“深拷贝对象”展示了如何采取相同的基本数据结构（一个包含数组的学生对象）并创建其深拷贝。

# 深拷贝对象

## 问题

你想创建一个自定义对象的精确副本。你不仅想复制顶层对象，还想复制它引用的每个对象。

## 解决方案

没有单一的解决方案来深拷贝一个对象。相反，开发者使用各种技术，每种技术都有其自己的权衡。

最安全的方法是编写针对要克隆对象类型的特定克隆逻辑。以下是一个示例，演示了如何对“克隆对象”中介绍的`student`对象进行深拷贝。

```
const student = {
  firstName: 'Tazie', lastName: 'Yang',
  testScores: [78, 88, 94, 91, 88, 96]
};

function cloneStudent(student) {
  // Start with a shallow copy
  const studentCopy = {...student};

  // Now duplicate the array (by expanding it with spread)
  studentCopy.testScores = [...studentCopy.testScores];

  return studentCopy;
}

// Create a truly independent student copy
const studentCopy = cloneStudent(student);

// Verify the arrays are separate
studentCopy.testScores[0] = 56;

console.log(student.testScores[0]);      // 78
console.log(studentCopy.testScores[0]);  // 56
```

这种方法的美妙之处在于你了解对象，因此你知道应该深入到多深程度。在这个例子中，我们知道`testScores`数组保存的是数字。因此，你知道简单地使用展开运算符进行克隆就足够了。但是，如果数组包含的是对象，你需要决定是否要复制所有这些对象，这是在“克隆数组”中演示的技术。或者，如果`testScores`是其他类型的集合对象（如`Set`或`Map`），你可以适当地创建并填充一个相应类型的新集合。

如果你想要一个通用解决方案，可以深拷贝任意对象，远远最好的选择是使用一个来自著名 JavaScript 库的预构建、经过测试的例程，比如 Lodash 的`cloneDeep()`，可以通过`lodash.clonedeep`模块单独导入。

## 讨论

有关 JavaScript 未来版本中内置序列化和深拷贝支持的讨论。但目前，深克隆是一个你需要自行解决的空白。

如果你正在创建一个成熟的类（“创建可重用类”），考虑将你的自定义克隆函数作为类本身的一个方法：

```
class Student {
  constructor(firstName, lastName, testScores) {
    this.firstName = firstName;
    this.lastName = lastName;
    this.testScores = testScores;
  }

  clone() {
    return new Student(this.firstName, this.lastName,
     [...this.testScores]);
  }
}

const student = new Student('Tazie', 'Yang', [78, 88, 94, 91, 88, 96]);
const studentCopy = student.clone();

// Verif the arrays are separate
studentCopy.testScores[0] = 56;

console.log(student.testScores[0]);      // 78
console.log(studentCopy.testScores[0]);  // 56
```

此示例未使用扩展运算符。相反，它使用构造函数创建一个新的`Student`对象。如果使用扩展运算符，你的副本将是基本`Object`类的实例，而不是`Student`类的实例。你的副本仍将具有与原始对象相同的属性，但如果使用`instanceof`测试，它将不会显示为`Student`。它还将无法使用你添加到`Student`类的任何方法。为了避免这些问题，你应始终为你的副本创建正确的对象类型。

如果你想知道是否可能创建自己的通用对象复制例程，你可能会感到疑惑。这些问题比它们看起来更加困难，而且有许多在网络上推荐的反模式很可能会引起严重的头痛。

对于自引用对象链，使用递归逻辑的天真方法会导致灾难性失败（堆栈溢出）。一个简单的例子是，当一个对象引用另一个引用原始对象的对象时。然而，更微妙的版本却出奇地常见。

该问题的另一个变体是，如果一个对象有两个对同一对象的引用。例如，考虑一个`ProductCatalog`，其中包含一些对同一个`Supplier`对象的引用的`Product`对象数组。天真的方法将为每个`Product`创建多个`Supplier`副本。更复杂的实现方式，如 Lodash 的`cloneDeep()`，在进行时跟踪引用，以确保不会多次重建同一对象。（其克隆实现的[源码](https://github.com/lodash/lodash/blob/master/.internal/baseClone.js)对于考虑重复发明轮子的人来说是一个有用的解药。）

另一个常见推荐的克隆方法是使用 JSON 序列化将对象转换为字符串表示形式，然后再转换回来。这会在处理`Date`对象（变为字符串）、`Infinity`等特殊值以及包含函数的自定义对象（被丢弃）时遇到问题。最糟糕的是，你不会收到有关缺失信息的警告。

###### 注意

如果你想测试两个对象是否相等，相同的考虑因素也会发挥作用。`===`运算符只会告诉你两个变量是否指向同一个对象。如果你有包含相同数据的单独对象，它会返回`false`。你可以编写一个通用例程来查找和比较任何两个对象的所有属性。然而，相等的含义取决于你比较的数据类型，因此编写自己的`isEqual()`函数始终是最安全的方法。

# 创建绝对唯一的对象属性键

## 问题

想要给对象添加一个唯一命名的属性，并且保证它不会与任何其他属性名称冲突。

## 解决方案

使用`Symbol`类型创建一个新的属性名称。然后，使用键值语法设置该属性：

```
const newObj = {};

// Set a unique property that will never clash with anything else
const uniqueId = Symbol();
newObj[uniqueId] = 'No two alike';

// Set another one
const anotherUniqueId = Symbol();
newObj[anotherUniqueId] = 'This will not clash, either';

console.log(newObj);
```

有趣的是，您实际上从未看到`Symbol`类型使用的唯一标识符。在这个例子中，这是您在控制台中将得到的输出：

```
{Symbol(): 'No two alike', Symbol(): 'This will not clash, either'}
```

要访问使用`Symbol`创建的属性，您需要跟踪具有属性名称的变量。您可以随时使用它来检索您的值：

```
console.log(newObj[uniqueID]);  // 'No two alike'
```

## 讨论

属性名称冲突并不常见，但在 JavaScript 中比许多其他语言更常见。问题的一部分是属性始终是公共的。这意味着，如果您从另一个类继承（参见“从另一个类继承功能”），您需要注意每个继承的属性，并确保自己不使用相同的名称。但命名冲突的最常见原因是，如果您正在创建某种可扩展性系统或服务，该系统需要您向其他人的对象添加属性。在这种情况下，您不会知道您的属性是否会与已存在于该对象中的属性冲突，因为您不拥有该对象的设计。

您可以使用各种解决方法来检查属性并生成随机名称。但是，`Symbol`类型为您提供了一种快速有效的解决方案。每个`Symbol`都保证是唯一的。您通过调用`Symbol()`方法来创建它（因为`Symbol`是一个原始类型，而不是对象，所以不使用`new`调用构造函数）。

可选地，您可以为您的符号添加描述，这对调试很有用：

```
newObj = {};
const propertyName = Symbol('Log Status');
newObj[propertyName] = 'logged';
```

但是，描述并不用于创建`Symbol`。如果您使用相同的描述创建两个`Symbol`实例，将会有两个完全独立的唯一标识符，这些标识符由 JavaScript 在全局`Symbol`值注册表中内部存储。

# 使用 Symbol 创建枚举

## 问题

您希望存储一小组相关的常量，以便在代码中按名称引用它们。

## 解决方案

使用`Symbol()`为每个常量设置值：

```
// Create three constants to use as an enum
const TrafficLight = {
  Green: Symbol('green'),
  Red: Symbol('red'),
  Yellow: Symbol('yellow')
}

// This function uses the light enum
function switchLight(newLight) {
  if (newLight === TrafficLight.Green) {
    console.log('Turning light green');
  }
  else if (newLight === TrafficLight.Yellow) {
    console.log('Get ready to stop');
  }
  else {
    console.log('Turning light red');
  }
  return newLight;
}

let light = TrafficLight.Green;
light = switchLight(TrafficLight.Yellow);
light = switchLight(TrafficLight.Red);

console.log(light);   // shows "Symbol('red')"
```

## 讨论

枚举（或*枚举标识符*）是一组命名常量。枚举在任何时候都很有用，当您有一个只能取一小组允许值的变量时。通过使用枚举值，您使您的代码更清晰。您还减少了出错的机会（与使用魔术数字相比），因为您不会忘记每个数字的含义，并且您不能意外使用没有为其定义常量的数字。

###### 注意

有关常量大写的适当惯例存在一些争论。例如，`Math`类将只读属性如`Math.PI`和`Math.E`放在大写字母中。此示例中的解决方案使用了首字母大写的枚举常量和包装它们的对象，例如`TrafficLight.Red`。

常量通常使用数字值或字符串值创建。如果常量映射到其他有用的信息，例如这里显示的单位转换值，那么这是一个特别好的方法：

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
```

如果您没有自然的唯一值可用于枚举常量，请考虑使用`Symbol`。这可以避免您需要选择任意数字，并且每个`Symbol`的保证唯一性确保您无法替换任何其他值。（这还消除了当您进行更改时可能导致错误的一致性问题，比如有些地方使用硬编码的数字，其他地方使用`const`变量。）本示例中的`TrafficLight`示例使用了三个值的`Symbol`。

使用`Symbol`的缺点在于其底层值完全不透明。这就是为什么本示例中的解决方案为每个 Symbol 提供一个描述性名称，例如`Symbol('red')`。当您将`Symbol`记录到控制台或将其转换为字符串时，您将看到这个文本。如果在创建`Symbol`时没有提供描述性名称，您只会看到通用文本`"Symbol()"`。

## 参见

要查看`Symbol`数据类型的详细信息，请参见“创建绝对唯一的对象属性键”。
