# 第五章：现代 JavaScript 语法和特性

JavaScript 现在已经存在了多个十年，并经历了多次修订。本书探讨了现代 JavaScript 背景下的设计模式，并对所有讨论的示例使用了现代的 ES2015+语法。本章讨论了 ES2015+ JavaScript 的特性和语法，这些对于进一步讨论当前 JavaScript 背景下的设计模式至关重要。

###### 注意

ES2015 引入了一些对我们讨论的模式特别重要的 JavaScript 语法的基本更改。这些在[BabelJS ES2015 指南](https://oreil.ly/V09r_)中有很好的介绍。

本书依赖于现代 JavaScript 语法。您可能也对 TypeScript 感兴趣。TypeScript 是 JavaScript 的静态类型超集，提供了几个 JavaScript 不具备的语言特性。这些特性包括强类型、接口、枚举和高级类型推断，还可以影响设计模式。要了解有关 TypeScript 及其优势的更多信息，请考虑查阅 O'Reilly 书籍，如[*Programming TypeScript*](https://learning.oreilly.com/library/view/programming-typescript/9781492037644/)，作者 Boris Cherny。

# 解耦应用程序的重要性

模块化 JavaScript 允许您将应用程序逻辑上分割成称为模块的小块。一个模块可以被其他模块导入，这些模块又可以被更多的模块导入。因此，应用程序可以由许多嵌套模块组成。

在可伸缩的 JavaScript 世界中，当我们说一个应用程序是*模块化*时，通常意味着它由一组高度解耦的功能模块组成。松散耦合通过尽可能消除*依赖关系*来更轻松地维护应用程序。如果实现得当，它允许您看到对系统的一个部分进行更改可能如何影响另一个部分。

不像一些更传统的编程语言，旧版本的 JavaScript 直到 ES5（[ECMA-262 第 5.1 版](https://oreil.ly/w2WxN)）没有为开发人员提供清晰地组织和导入代码模块的手段。这是在更近年才显得需要更有组织的 JavaScript 应用程序规范时的一个问题。[AMD（异步模块定义）](https://oreil.ly/W5XPd)和[CommonJS](https://oreil.ly/lgw0w)模块是在 JavaScript 初始版本中解耦应用程序最流行的模式之一。

解决这些问题的本地解决方案随着[ES6 或 ES2015](https://oreil.ly/rPxFL)的到来而出现。[TC39](https://oreil.ly/GJduA)，负责定义 ECMAScript 及其未来版本的语法和语义的标准机构，一直密切关注 JavaScript 在大规模开发中的使用演变，并深刻意识到编写更模块化 JS 的需要。

使用 ECMAScript 2015 发布的 ECMAScript 模块的语法已经开发并标准化。今天，所有主要浏览器都支持 JavaScript 模块。它们已成为在 JavaScript 中实现现代模块化编程的事实标准。在本节中，我们将使用 ES2015+ 中的模块语法来探索代码示例。

# 具有导入和导出的模块

模块允许我们将应用程序代码分隔为独立单元，每个单元包含一个功能方面的代码。模块还鼓励代码的可重用性，并公开可集成到不同应用程序中的功能。

一种语言应该具有功能，允许您 `import` 模块依赖项并 `export` 模块接口（我们允许其他模块使用的公共 API/变量）以支持模块化编程。对于 [JavaScript 模块（也称为 ES 模块）](https://oreil.ly/kd-pu)，在 ES2015 中引入了对 JavaScript 的支持，允许您使用 `import` 关键字指定模块依赖项。同样，您可以使用 `export` 关键字从模块内部导出几乎任何内容：

+   `import` 声明将模块的导出绑定为本地变量，并且可以重命名以避免名称冲突。

+   `export` 声明声明模块的本地绑定是外部可见的，以便其他模块可以读取导出但不能修改它们。有趣的是，模块可以导出子模块，但不能导出在其他地方定义的模块。我们还可以重命名导出，使其外部名称与本地名称不同。

###### 注意

*.mjs* 是用于 JavaScript 模块的扩展，帮助我们区分模块文件和经典脚本（*.js*）。*.mjs* 扩展确保相应的文件被运行时和构建工具（例如，[Node.js](https://oreil.ly/E9oRS)、[Babel](https://oreil.ly/fkQAL)）解析为模块。

以下示例展示了三个模块，分别是面包店员工、他们在烘焙时执行的功能以及面包店本身。我们看到一个模块导出的功能如何被另一个模块导入并使用：

```
// Filename: staff.mjs
// =========================================
// specify (public) exports that can be consumed by other modules
export const baker = {
   bake(item) {
      console.log( `Woo! I just baked ${item}` );
    }
};

// Filename: cakeFactory.mjs
// =========================================
// specify dependencies
import baker from "/modules/staff.mjs";

export const oven = {
    makeCupcake(toppings) {
       baker.bake( "cupcake", toppings );
    },
    makeMuffin(mSize) {
        baker.bake( "muffin", size );
    }
}

// Filename: bakery.mjs
// =========================================
import {cakeFactory} from "/modules/cakeFactory.mjs";
cakeFactory.oven.makeCupcake( "sprinkles" );
cakeFactory.oven.makeMuffin( "large" );
```

通常，一个模块文件包含几个相关的函数、常量和变量。您可以在文件末尾使用单个导出语句，后跟一个逗号分隔的模块资源列表，将这些资源集体导出：

```
// Filename: staff.mjs
// =========================================
const baker = {
  //baker functions
};
const pastryChef = {
  //pastry chef functions
};
const assistant = {
  //assistant functions
};

export { baker, pastryChef, assistant };
```

同样，您可以只导入您需要的函数：

```
import {baker, assistant} from "/modules/staff.mjs";
```

您可以通过指定值为 `module` 的 `type` 属性，告诉浏览器接受包含 JavaScript 模块的 `<script>` 标签：

```
<script type="module" src="main.mjs"></script>
<script nomodule src="fallback.js"></script>
```

`nomodule`属性告诉现代浏览器不要将经典脚本作为模块加载。这对于不使用模块语法的回退脚本非常有用。它允许你在 HTML 中使用模块语法，并使其在不支持该语法的浏览器中正常工作。这在多个方面都很有用，包括性能。现代浏览器不需要为现代特性提供 polyfill，可以单独为传统浏览器提供更大的转码代码。

# 模块对象

一种更清晰的导入和使用模块资源的方法是将模块作为对象导入。这使得所有导出都作为对象的成员可用：

```
// Filename: cakeFactory.mjs

import * as Staff from "/modules/staff.mjs";

export const oven = {
    makeCupcake(toppings) {
       Staff.baker.bake( "cupcake", toppings );
    },
    makePastry(mSize) {
        Staff.pastryChef.make( "pastry", type );
    }
}
```

# 从远程源加载的模块

ES2015+还支持远程模块（例如第三方库），使得从外部位置加载模块变得简单。这里有一个示例，展示了如何拉取我们之前定义的模块并利用它：

```
import {cakeFactory} from "https://example.com/modules/cakeFactory.mjs";
// eagerly loaded static import

cakeFactory.oven.makeCupcake( "sprinkles" );
cakeFactory.oven.makeMuffin( "large" );
```

# 静态导入

刚刚讨论的导入类型称为静态导入。在主代码运行之前，需要使用静态导入下载和执行模块图形。有时这会导致在初始页面加载时前端加载大量代码，这可能是昂贵的并延迟关键功能的提前可用性：

```
import {cakeFactory} from "/modules/cakeFactory.mjs";
// eagerly loaded static import

cakeFactory.oven.makeCupcake( "sprinkles" );
cakeFactory.oven.makeMuffin( "large" );
```

# 动态导入

有时候，你不想预加载一个模块，而是在需要时按需加载。延迟加载模块允许你在需要时加载所需内容，例如，当用户点击链接或按钮时。这提高了初始加载性能。[动态导入](https://oreil.ly/fqR6v)的引入使这成为可能。

动态导入引入了一种类似函数的导入形式。`import(url)`返回请求的模块命名空间对象的 promise，该对象在获取、实例化和评估模块及其所有依赖项后创建。这里有一个示例展示了对`cakeFactory`模块的动态导入：

```
form.addEventListener("submit", e => {
  e.preventDefault();
  import("/modules/cakeFactory.js")
    .then((module) => {
      // Do something with the module.
      module.oven.makeCupcake("sprinkles");
      module.oven.makeMuffin("large");
    });
});
```

动态导入也可以使用`await`关键字支持：

```
let module = await import("/modules/cakeFactory.js");
```

使用动态导入时，只有在模块被使用时才会下载和评估模块图形。

流行的模式如交互式导入和可见性导入可以在原生 JavaScript 中利用动态导入功能轻松实现。

## 交互式导入

一些库可能仅在用户开始与网页上的特定功能进行交互时才需要。典型的例子包括聊天窗口、复杂的对话框或视频嵌入。这些功能的库不需要在页面加载时导入，而是可以在用户与它们进行交互时加载（例如通过点击组件外观或占位符）。操作可以触发相应库的动态导入，然后调用函数以激活所需的功能。

例如，你可以使用外部的[`lodash.sortby`模块](https://oreil.ly/VUgnM)实现屏幕上的排序功能，该模块是动态加载的：

```
const btn = document.querySelector('button');

btn.addEventListener('click', e => {
  e.preventDefault();
  import('lodash.sortby')
    .then(module => module.default)
    .then(sortInput()) // use the imported dependency
    .catch(err => { console.log(err) });
});
```

## 可见性导入

许多组件在初始页面加载时不可见，但随着用户向下滚动，它们变得可见。由于用户可能不会总是向下滚动，因此当这些组件变得可见时，相应的模块可以进行延迟加载。[IntersectionObserver API](https://oreil.ly/wXwgi) 可以检测组件占位符即将变得可见的时候，动态导入可以加载相应的模块。

# 服务器端的模块

[Node](https://oreil.ly/4Bh_O) 15.3.0 以后支持 JavaScript 模块。它们无需实验标志，并且与 npm 软件包生态系统的其余部分兼容。Node 处理以 *.mjs* 和 *.js* 结尾的文件，并将顶级 type 字段值设置为 module 作为 JavaScript 模块：

```
{
  "name": "js-modules",
  "version": "1.0.0",
  "description": "A package using JS Modules",
  "main": "index.js",
  "type": "module",
  "author": "",
  "license": "MIT"
}
```

# 使用模块的优点

模块化编程和使用模块提供了几个独特的优势。以下是其中一些：

模块脚本仅评估一次。

浏览器只会对模块脚本进行一次评估，而经典脚本会根据其添加到 DOM 的次数进行评估。这意味着，如果您有一个依赖模块层次结构，那么依赖于最内层模块的模块将首先进行评估。这是一件好事，因为这意味着最内层模块将首先进行评估，并且可以访问依赖于它的模块的导出内容。

模块自动推迟加载。

与其他脚本文件不同，如果不希望立即加载它们，您无需包含 `defer` 属性，浏览器会自动推迟模块的加载。

模块易于维护和重用。

模块促进解耦能够独立维护的代码片段，而不需要对其他模块进行重大更改。它们还允许在多个不同功能中重复使用相同的代码。

模块提供命名空间。

模块为相关变量和常量创建了一个私有空间，以便可以通过模块引用它们而不污染全局命名空间。

模块实现了死代码消除。

在引入模块之前，必须手动从项目中删除未使用的代码文件。使用模块导入后，像 [webpack](https://oreil.ly/37e9F) 和 [Rollup](https://oreil.ly/rUWiB) 这样的打包工具可以自动识别未使用的模块并将其消除。在添加到捆绑包之前可能会删除死代码。这称为摇树。

所有现代浏览器都支持模块的 [import](https://oreil.ly/IauTK) 和 [export](https://oreil.ly/NubAY)，您可以在没有任何回退的情况下使用它们。

# 带有构造函数、getter 和 setter 的类

除了模块外，ES2015+ 还允许使用构造函数和一些私密性概念来定义类。JavaScript 类是用 `class` 关键字定义的。在下面的示例中，我们定义了一个 `Cake` 类，包含一个构造函数和两个 getter 和 setter：

```
class Cake{

    // We can define the body of a class constructor
    // function by using the keyword constructor
    // with a list of class variables.
    constructor( name, toppings, price, cakeSize ){
        this.name = name;
        this.cakeSize = cakeSize;
        this.toppings = toppings;
        this.price = price;
    }

    // As a part of ES2015+ efforts to decrease the unnecessary
    // use of function for everything, you will notice that it is
    // dropped for cases such as the following. Here an identifier
    // followed by an argument list and a body defines a new method.

    addTopping( topping ){
        this.toppings.push( topping );
    }

    // Getters can be defined by declaring get before
    // an identifier/method name and a curly body.
    get allToppings(){
        return this.toppings;
    }

    get qualifiesForDiscount(){
        return this.price > 5;
    }

    // Similar to getters, setters can be defined by using
    // the set keyword before an identifier
    set size( size ){
        if ( size < 0){
            throw new Error( "Cake must be a valid size: " +
                                    "either small, medium or large");
        }
       this.cakeSize = size;
    }
}

// Usage
let cake = new Cake( "chocolate", ["chocolate chips"], 5, "large" );
```

JavaScript 类是基于原型的，并且是 JavaScript 函数的一个特殊类别，需要在引用之前定义。

您还可以使用`extends`关键字指示一个类继承自另一个类：

```
class BirthdayCake extends Cake {
  surprise() {
    console.log(`Happy Birthday!`);
  }
}

let birthdayCake = new BirthdayCake( "chocolate", ["chocolate chips"], 5,
  "large" );
birthdayCake.surprise();
```

所有现代浏览器和 Node 都支持 ES2015 类。它们也兼容 ES6 中引入的[new-style class syntax](https://oreil.ly/9c9jm)。

JavaScript 模块和类的区别在于模块是[导入](https://oreil.ly/IauTK)和[导出](https://oreil.ly/NubAY)，而类则是用`class`关键字定义的。

阅读下去，你可能还会注意到前面的例子中缺少了`function`一词。这不是打字错误：TC39 有意减少我们对`function`关键字的滥用，希望能简化我们编写代码的方式。

JavaScript 类还支持`super`关键字，它允许您调用父类的构造函数。这对实现自身继承模式非常有用。您可以使用`super`来调用超类的方法：

```
class Cookie {
  constructor(flavor) {
    this.flavor = flavor;
  }

  showTitle() {
    console.log(`The flavor of this cookie is ${this.flavor}.`);
  }
}

class FavoriteCookie extends Cookie {
  showTitle() {
    super.showTitle();
    console.log(`${this.flavor} is amazing.`);
  }
}

let myCookie = new FavoriteCookie('chocolate');
myCookie.showTitle();
// The flavor of this cookie is chocolate.
// chocolate is amazing.
```

现代 JavaScript 支持公共和私有类成员。公共类成员可以被其他类访问。私有类成员只能在定义它们的类中访问。类字段默认为公共。通过使用`#`（哈希）前缀可以创建[私有类字段](https://oreil.ly/SXsES)：

```
class CookieWithPrivateField {
  #privateField;
}

class CookieWithPrivateMethod {
  #privateMethod() {
    return 'delicious cookies';
  }
}
```

JavaScript 类支持使用`static`关键字的静态方法和属性。静态成员可以在不实例化类的情况下引用。您可以使用静态方法创建实用函数，并使用静态属性来保存配置或缓存数据：

```
class Cookie {
  constructor(flavor) {
    this.flavor = flavor;
  }
  static brandName = "Best Bakes";
  static discountPercent = 5;
}
console.log(Cookie.brandName); //output = "Best Bakes"
```

# JavaScript 框架中的类

在过去几年中，一些现代 JavaScript 库和框架——特别是 React——引入了替代类的方法。React Hooks 使得在不使用 ES2015 类组件的情况下使用 React 状态和生命周期方法成为可能。在 Hooks 出现之前，React 开发者必须将功能组件重构为类组件，以便处理状态和生命周期方法。这通常很棘手，并且需要理解 ES2015 类的工作原理。React Hooks 是函数，允许您管理组件的状态和生命周期方法，而无需依赖于类。

请注意，构建 Web 的其他几种方法，例如[Web Components](https://oreil.ly/ndfeb)社区，继续以类作为组件开发的基础。

# 总结

本章介绍了模块和类的 JavaScript 语言语法。这些特性使我们能够在遵循面向对象设计和模块化编程原则的同时编写代码。我们还将使用这些概念来对不同的设计模式进行分类和描述。下一章将讨论设计模式的不同类别。

# 相关阅读

+   [v8 上的 JavaScript 模块](https://oreil.ly/IEuAq)

+   [MDN 上的 JavaScript 模块](https://oreil.ly/OAL9O)
