# 第十四章：React.js 应用程序结构

在构建小型爱好项目或尝试新概念或库时，开发人员可以开始向文件夹添加文件，而无需计划或组织结构。这些可能包括 CSS、辅助组件、图像和页面。随着项目的增长，单个资源文件夹变得难以管理。任何规模相当大的代码库都应根据逻辑标准组织成应用程序文件夹结构。如何结构化文件和应用程序组件的决定可能是个人/团队选择。这也通常取决于应用程序领域和使用的技术。

本章主要关注 React.js 应用程序的文件夹结构，这有助于更好地管理我们的项目随着其增长。

# 介绍

React.js 本身并没有提供有关项目结构的指导方针，但确实建议了一些常用的方法。让我们先看看这些方法，并在讨论具有复杂性和 Next.js 应用程序的项目的文件夹结构之前了解它们的优缺点。

在高层次上，您可以以[两种方式](https://oreil.ly/Tkwai)对 React 应用程序中的文件进行分组：

按功能分组

为每个应用程序模块、功能或路由创建文件夹。

按文件类型分组

为不同类型的文件创建文件夹。

让我们详细看看这种分类。

## 按模块、功能或路由分组

在这种情况下，文件结构将反映业务模型或应用程序流程。例如，如果您有一个电子商务应用程序，您将为产品、产品列表、结账等创建文件夹。专门为产品模块所需的 CSS、JSX 组件、测试、子组件或辅助库都位于产品文件夹中：

```
common/
  Avatar.js
  Avatar.css
  ErrorUtils.js
  ErrorUtils.test.js
product/
  index.js
  product.css
  price.js
  product.test.js
checkout/
  index.js
  checkout.css
  checkout.test.js
```

按功能分组文件的优势在于，如果模块发生更改，所有受影响的文件都位于同一文件夹中，更改会局限于代码的特定部分。

缺点是应定期识别跨模块使用的常见组件、逻辑或样式，以避免重复，并促进一致性和重用。

## 按文件类型分组

在这种分组方式中，您将为 CSS、组件、测试文件、图像、库等创建不同的文件夹。因此，逻辑相关的文件将根据文件类型位于不同的文件夹中：

```
css/
  global.css
  checkout.css
  product.css
lib/
  date.js
  currency.js
  gtm.js
pages/
  product.js
  productlist.js
  checkout.js
```

这种方法的优势是：

+   您有一个可以在项目中重复使用的标准结构。

+   对于对应用程序特定逻辑了解有限的新团队成员，仍然可以找到类似样式或测试文件。

+   在不同路由或模块中导入的常见组件（如日期选择器）和样式可以一次更改，以确保效果在整个应用程序中可见。

缺点是：

+   特定模块逻辑的更改可能需要跨不同文件夹的文件进行更改。

+   随着应用程序中功能的增加，不同文件夹中的文件数量会增加，这使得查找特定文件变得困难。

对于小到中型应用程序每个文件夹中的文件数目（50 到 100）较少的项目，这些方法中的任何一种都可以很容易地设置。但是，对于更大的项目，您可能需要根据应用程序的逻辑结构采取一种混合方法。让我们看看一些可能性。

## 基于领域和通用组件的混合分组

在这里，您将所有应用程序中需要的常见组件放在一个组件文件夹中，并将所有应用程序流特定的路由或功能放在[领域文件夹](https://oreil.ly/rJQaz)（名称可以是*domain*、*pages*或*routes*）。每个文件夹可以有特定组件和相关文件的子文件夹：

```
css/
  global.css
components/
  User/
    profile.js
    profile.test.js
    avatar.js
  date.js
  currency.js
  gtm.js
  errorUtils.js
domain/
  product/
    product.js
    product.css
    product.test.js
  checkout/
    checkout.js
    checkout.css
    checkout.test.js
```

因此，您可以通过将相关文件放置在一起，结合“按文件类型分组”和“按功能分组”的优势，这些文件经常一起更改，并且通用的可重用组件和样式在整个应用程序中使用。

根据应用程序的复杂性，您可以将其修改为较平的结构，没有子文件夹或更嵌套的结构：

较平的结构

以下示例说明了一个较平的结构：

```
    domain/
        product.js
        product.css
        product.test.js
        checkout.js
        checkout.css
        checkout.test.js
```

嵌套结构

以下示例显示了一个更嵌套的结构：

```
    domain/
        product/
            productType/
                features.js
                features.css
                size.js
            price/
                listprice.js
                discount.js
```

###### 注意

最好避免深层嵌套超过三到四个级别，因为在文件夹之间写相对导入或移动文件时更新这些导入会变得更加困难。

这种方法的一个变体是根据视图或路由创建文件夹，除了基于领域创建的文件夹，如此处所述 [here](https://oreil.ly/WiRca)。然后，路由组件可以根据当前路由协调要显示的视图。[Next.js](https://oreil.ly/6PwMu) 使用了类似的结构。

# 现代 React 功能的应用程序结构

现代 React 应用程序使用不同的功能，如 Redux、有状态容器、Hooks 和 Styled Components。让我们看看这些代码在上一节提出的应用程序结构中的位置。

## Redux

Redux 文档[强烈建议](https://oreil.ly/iH1aX)在一个地方协调给定特性的逻辑。在给定的特性文件夹内，该特性的 Redux 逻辑应该被编写为一个单独的“切片”文件，最好使用 Redux Toolkit 的 `createSlice` API。该文件捆绑了 {`actionTypes, actions, reducer`} 到一个独立的、隔离的模块。这也被称为[“鸭子”模式](https://oreil.ly/UOqb5)（来自 Redux）。例如，如此处所示 [here](https://oreil.ly/0gpXl):

```
/src
    index.tsx: Entry point file that renders the React component tree
    /app
        store.ts: store setup
        rootReducer.ts: root reducer (optional)
        App.tsx: root React component
    /common: hooks, generic components, utils, etc
    /features: contains all "feature folders"
    /todos: a single feature folder
        todosSlice.ts: Redux reducer logic and associated actions
        Todos.tsx: a React component
```

另一个不使用创建容器或 Hooks 的详细示例可以在这里找到 [here](https://oreil.ly/xMZiu)。

## 容器

如果你已经将代码结构化为[展示组件和有状态容器组件](https://oreil.ly/JeYgI)，你可以为容器组件创建一个单独的文件夹。容器可以让你将复杂的有状态逻辑与组件的其他方面分离开来：

```
/src
    /components
        /component1
            index.js
            styled.js

    /containers
        /container1
```

你可以在[同一篇文章](https://oreil.ly/JeYgI)中找到一个包含容器的应用的完整结构。

## Hooks

Hooks 可以像任何其他类型的代码一样适应混合结构。你可以在应用程序级别有一个用于所有 React 组件消费的常见 Hooks 的文件夹。只有一个组件使用的 React Hooks 应保留在组件的文件中或组件文件夹中的单独*hooks.js*文件中。你可以在[这里](https://oreil.ly/rtT1n)找到一个示例结构：

```
/components
    /productList
        index.js
        test.js
        style.css
        hooks.js

/hooks
    /useClickOutside
      index.js
    /useData
      index.js
```

## Styled Components

如果你使用的是 Styled Components 而不是 CSS，你可以使用*style.js*文件代替之前提到的组件级 CSS 文件。例如，如果你有一个`titlebar`组件，结构会是这样的：

```
/src/components/button/
    index.js
    style.js
```

应用级[*theme.js*文件](https://oreil.ly/OARQ8)将包含用于背景和文本颜色的值。一个[globals 组件](https://oreil.ly/LzmtQ)可以包含其他组件可以使用的常见样式元素的定义。

# 其他最佳实践

除了文件夹结构，当构建 React 应用程序时，你可以考虑一些其他最佳实践，如下所示：

+   使用[导入别名](https://oreil.ly/trM4V)来帮助处理常见导入的长相对路径。这可以通过 Babel 和[webpack](https://oreil.ly/cSkCS)配置来实现。

+   使用你的 API 包装第三方库，以便在需要时可以进行替换。

+   与组件一起使用[PropTypes](https://oreil.ly/8kL84)来确保属性值的类型检查。

构建性能取决于文件数量和依赖关系。如果你使用像 webpack 这样的捆绑器，一些建议可以帮助提高构建时间。

当使用[加载器](https://oreil.ly/zXFkv)时，只将其应用于需要转换的模块。例如：

```
const path = require('path');

module.exports = {
  //...
 module: {
   rules: [
     {
       test: /\.js$/,
       include: path.resolve(__dirname, 'src'),
       loader: 'babel-loader',
      },
    ],
  },
};
```

如果你使用混合/嵌套的文件夹结构，下面来自 webpack 的[示例](https://oreil.ly/slT4K)展示了如何在结构中包含和加载不同路径的文件：

```
const path = require('path');

module.exports = {
  //...
  module: {
    rules: [
      {
       test: /\.css$/,
       include: [
          // Include paths relative to the current directory starting with
          // `app/styles` e.g. `app/styles.css`, `app/styles/styles.css`,
          // `app/stylesheet.css`
          path.resolve(__dirname, 'app/styles'),

         // add an extra slash to only include the content of the directory
         // `vendor/styles/`
         path.join(__dirname, 'vendor/styles/'),
        ],
      },
    ],
  },
};
```

没有`import`、`require`、`define`等引用其他模块的文件不需要解析其依赖关系。你可以使用[`noParse`选项](https://oreil.ly/UjYPF)来避免解析它们。

# Next.js 应用程序结构

[Next.js](https://oreil.ly/ZeU0P)是一个可扩展的 React 应用程序的生产就绪框架。虽然你可以使用混合结构，但应用程序中的所有路由必须分组在 pages 文件夹下。（页面的 URL = 根 URL + 页面文件夹中的相对路径）。

扩展之前讨论过的结构，你可以为常见组件、样式、Hooks 和实用函数创建文件夹。在领域账户相关的代码可以被组织成功能组件，不同路由可以使用这些组件。最后，你会有一个页面文件夹用于所有路由。根据[这个指南](https://oreil.ly/AAv12)提供的示例，这里有一个例子：

```
--- public/
  Favicon.ico
  images/
--- common/
    components/
      datePicker/
        index.js
        style.js
    hooks/
    utils/
    styles/
--- modules/
    auth/
      auth.js
      auth.test.js
    product/
      product.js
      product.test.js
--- pages/
    _app.js
    _document.js
    index.js
        /products
      [id].js
```

Next.js 也为许多不同类型的应用程序提供了[示例](https://oreil.ly/Kim4W)。你可以使用`create-next-app`来创建 Next.js 提供的模板文件结构。例如，要为基本的[博客应用](https://oreil.ly/ym0kh)创建模板：

```
yarn create next-app --example blog my-blog
```

# 总结

本章讨论了多种不同的选项来组织 React 项目。根据项目的大小、类型和使用的组件，你可以选择最适合你的那种。坚持为项目结构定义一个明确定义的模式将帮助你向团队其他成员解释项目，并防止项目变得混乱和不必要复杂。

下一章是本书的结尾章节，提供了在学习 JavaScript 设计模式时可能有帮助的额外链接。
