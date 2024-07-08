# 第五章：常见问题和强大模式

现在我们更了解 React 在幕后的工作原理，让我们深入探讨它在编写 React 应用程序时的实际应用。在本章中，我们将探讨关于记忆化、惰性加载和性能的常见 React 问题的答案，以提升我们对这些概念的熟练程度。让我们开始谈论记忆化。

# 使用 React.memo 进行记忆化

记忆化是计算机科学中用于通过缓存先前计算的结果来优化函数性能的技术。简单来说，记忆化根据函数的输入存储其输出，因此如果再次使用相同的输入调用函数，则返回缓存的结果而不是重新计算输出。这显著减少了执行函数所需的时间和资源，特别是对于计算开销大或频繁调用的函数。记忆化依赖于函数的纯度，即对于给定的输入，函数可预测地返回相同的输出。一个纯函数的例子是：

```
function add(num1, num2) {
  return num1 + num2;
}
```

当给定参数 `1` 和 `2` 时，此 `add` 函数始终返回 `3`，因此可以安全地进行记忆化。如果函数依赖于像网络通信之类的副作用，则无法进行记忆化。例如考虑：

```
async function addToNumberOfTheDay(num) {
  const todaysNumber = await fetch("https://number-api.com/today")
    .then((r) => r.json())
    .then((data) => data.number);
  return num + todaysNumber;
}
```

给定输入 `2`，这个函数每天都会返回一个不同的结果，因此无法进行记忆化。也许这是一个愚蠢的例子，但通过这个例子，我们粗略理解了基本的记忆化。

在处理昂贵计算或渲染大量项目列表时，记忆化特别有用。考虑一个函数：

```
let result = null;
const doHardThing = () => {
  if (result) return result;

  // ...do hard stuff

  result = hardStuff;
  return hardStuff;
};
```

调用 `doHardThing` 一次可能需要几分钟来完成困难任务，但第二次、第三次、第四次或*n*次调用实际上不会执行困难任务，而是返回存储的结果。这就是记忆化的要点。

在 React 的上下文中，可以使用 `React.memo` 组件对功能组件应用记忆化。此函数返回一个新组件，仅当其 props 发生变化时才重新渲染。基于第四章，理想情况下现在我们知道“重新渲染”意味着重新调用函数组件。如果包装在 `React.memo` 中，该函数在协调期间不会再次调用，除非其 props 发生了变化。通过对功能组件进行记忆化，我们可以防止不必要的重新渲染，从而提高 React 应用程序的整体性能。

我们已经知道 React 组件是被调用以进行协调的函数，如第四章中讨论的那样。React 递归调用函数组件以创建 vDOM 树，然后用作协调的两个 Fiber 树的基础。有时，由于函数组件内的强烈计算或将其应用于 DOM 的放置或更新效果中的强烈计算，渲染（即调用组件函数）可能需要很长时间，如第四章中所述。这会减慢我们的应用程序并呈现迟滞的用户体验。记忆化是一种通过存储昂贵计算的结果并在传递给函数相同输入或组件相同 props 时返回它们的方式来避免这种情况。

要理解为什么`React.memo`很重要，让我们考虑一个常见的场景，我们有一个需要在组件中呈现的项目列表。例如，假设我们有一个待办事项列表，我们希望在组件中显示，如下所示：

```
function TodoList({ todos }) {
  return (
    <ul>
      {todos.map((todo) => (
        <li key={todo.id}>{todo.title}</li>
      ))}
    </ul>
  );
}
```

现在，让我们将此组件组合到另一个根据用户输入重新渲染的组件中：

```
function App() {
  const todos = Array.from({ length: 1000000 });
  const [name, setName] = useState("");

  return (
    <div>
      <input value={name} onChange={(e) => setName(e.target.value)} />
      <TodoList todos={todos} />
    </div>
  );
}
```

在我们的`App`组件中，在每次`input`字段中的键入时，`TodoList`都将重新渲染：每次在协调期间状态变化发生时，`TodoList`函数组件都将使用其 props 重新调用。这可能会导致性能问题，但这是 React 工作的核心：当组件中发生状态更改时，从该组件向下的每个函数组件都会在协调期间重新调用。

如果待办事项列表很大，并且组件频繁重新渲染，这可能导致应用程序性能瓶颈。优化此组件的一种方法是使用`React.memo`进行记忆化：

```
const MemoizedTodoList = React.memo(function TodoList({ todos }) {
  return (
    <ul>
      {todos.map((todo) => (
        <li key={todo.id}>{todo.title}</li>
      ))}
    </ul>
  );
});
```

通过使用`React.memo`包装`TodoList`组件，React 仅在其 props 更改时重新渲染组件。周围的状态变化不会影响它。这意味着如果待办事项列表保持不变，组件将不会重新渲染，并且将使用其缓存的输出。这可以节省大量资源和时间，特别是当组件复杂且待办事项列表很大时。

让我们考虑另一个例子，我们有一个包含多个昂贵渲染的嵌套组件的复杂组件：

```
function Dashboard({ data }) {
  return (
    <div>
      <h1>Dashboard</h1>
      <UserStats user={data.user} />
      <RecentActivity activity={data.activity} />
      <ImportantMessages messages={data.messages} />
    </div>
  );
}
```

如果`data`属性经常变化，这个组件可能很昂贵，特别是如果嵌套组件也很复杂。我们可以使用`React.memo`来优化这个组件，以记忆化每个嵌套组件：

```
const MemoizedUserStats = React.memo(function UserStats({ user }) {
  // ...
});

const MemoizedRecentActivity = React.memo(function RecentActivity({
  activity,
}) {
  // ...
});

const MemoizedImportantMessages = React.memo(function ImportantMessages({
  messages,
}) {
  // ...
});

function Dashboard({ data }) {
  return (
    <div>
      <h1>Dashboard</h1>
      <MemoizedUserStats user={data.user} />
      <MemoizedRecentActivity activity={data.activity} />
      <MemoizedImportantMessages messages={data.messages} />
    </div>
  );
}
```

通过记忆化每个嵌套组件，React 仅会重新渲染已更改的组件，并且缓存的输出将用于未更改的组件。这可以显著提高 `Dashboard` 组件的性能并减少不必要的重新渲染。因此，我们可以看到 `React.memo` 是优化 React 函数组件性能的重要工具。对于渲染开销大或具有复杂逻辑的组件来说，这尤其有用。

## 熟悉 `React.memo`

让我们简要介绍一下 `React.memo` 的工作原理。当 React 中发生更新时，你的组件将与其上一个渲染返回的虚拟 DOM 结果进行比较。如果这些结果不同 — 即，如果其 props 发生变化 — 调和器会在元素已存在于宿主环境（通常是浏览器 DOM）中时运行更新效果，或者在其不存在时运行放置效果。如果其 props 相同，则组件仍会重新渲染并且 DOM 仍会更新。

这就是 `React.memo` 的好处所在：在组件的 props 在渲染之间保持不变时避免不必要的重新渲染。由于我们可以在 React 中做到这一点，这引发了一个问题：我们应该对多少内容进行记忆化，以及多频繁地记忆化呢？如果我们记忆化每个组件，整体应用可能会更快，对吗？

## 记忆化的组件仍然会重新渲染

`React.memo` 执行的是所谓的 *浅* 比较，以确定 props 是否发生了更改。问题在于，虽然 JavaScript 中可以相当准确地比较标量类型，但无法对非标量类型进行比较。为了进行高质量的讨论，让我们简要地分析一下什么是标量和非标量类型，以及它们在比较操作中的行为。

### 标量（原始类型）

标量类型，也称为原始类型，是基础类型。这些类型表示单一的、不可分割的值。与数组和对象等更复杂的数据结构不同，标量没有属性或方法，并且它们天生是不可变的。这意味着一旦设置了标量值，就不能在不完全创建新值的情况下进行更改。JavaScript 有几种标量类型，包括数字、字符串、布尔值，以及像符号、BigInt、undefined 和 null 这样的其他类型。每种类型都有其独特的用途。例如，尽管数字很好理解，符号提供了一种创建唯一标识符的方式，而 undefined 和 null 允许开发人员在不同的上下文中表示值的缺失。在比较标量值时，我们通常对它们的实际内容或值感兴趣。

### 非标量（引用类型）

超越标量的简单性，我们遇到了非标量或引用类型。这些类型不存储数据，而是存储数据在内存中的引用或指针。这种区别至关重要，因为它影响了这些类型在代码中如何比较、操作和交互。在 JavaScript 中，最常见的非标量类型是对象和数组。对象允许我们使用键值对存储结构化数据，而数组提供有序的集合。在 JavaScript 中，函数也被视为引用类型。非标量的一个关键特征是多个引用可以指向相同的内存位置。这意味着通过一个引用修改数据可能会影响指向同一数据的其他引用。在比较时，非标量类型是通过它们的内存引用而不是它们的内容进行比较的。这有时会对不熟悉这种微妙差异的人产生意外结果。例如，两个具有相同内容但不同内存位置的数组在使用严格相等运算符进行比较时将被视为不相等。

考虑以下示例：

```
// Scalar types
"a" === "a"; // string; true
3 === 3; // number; true

// Non-scalar types
[1, 2, 3] === [1, 2, 3]; // array; false
{ foo: "bar"} === { foo: "bar" } // object; false
```

通过这种数组比较方式，数组、对象和其他非标量类型都是通过*引用*进行比较的：也就是说，左侧数组在计算机内存中的引用位置是否等于右侧的内存位置。这就是为什么比较返回`false`。对象也是如此。通过对象比较，我们在左侧和右侧分别在内存中创建了两个不同的对象——当然它们不相等，它们是存在于内存中两个不同位置的两个不同对象！它们只是具有相同的内容。

这就是为什么使用`React.memo`可能会很棘手。考虑一个名为`List`的函数组件，它以一个项数组作为 prop 并渲染它们：

```
const List = React.memo(function List({ items }) {
  return (
    <ul>
      {items.map((item) => (
        <li key={item}>{item}</li>
      ))}
    </ul>
  );
});
```

现在，想象一下在父组件中使用这个组件，并在每次父组件渲染时传递一个新的数组实例：

```
function ParentComponent({ allFruits }) {
  const [count, setCount] = React.useState(0);
  const favoriteFruits = allFruits.filter((fruit) => fruit.isFavorite);

  return (
    <div>
      <button onClick={() => setCount(count + 1)}>Increment</button>
      <List items={favoriteFruits} />
    </div>
  );
}
```

每当点击`Increment`按钮时，`ParentComponent`都会重新渲染。尽管传递给`List`的项值没有变化，但每次都会创建一个新的数组实例，每次都是`['apple', 'banana', 'cherry']`。由于`React.memo`对 props 执行浅比较，它会将这个新数组实例视为与上一次渲染的数组不同的 prop，导致`List`组件不必要地重新渲染。

要修复这个问题，我们可以使用`useMemo`钩子来对数组进行记忆：

```
function ParentComponent({ allFruits }) {
  const [count, setCount] = React.useState(0);
  const favoriteFruits = React.useMemo(
    () => allFruits.filter((fruit) => fruit.isFavorite),
    []
  );

  return (
    <div>
      <button onClick={() => setCount(count + 1)}>Increment</button>
      <List items={favoriteFruits} />
    </div>
  );
}
```

现在，数组只创建一次，并在重新渲染时保持相同的引用，从而防止`List`组件不必要地重新渲染。

这个示例强调了在使用`React.memo`和非标量 props 时理解引用比较的重要性。如果不小心使用，我们可能会在优化之外无意中引入性能问题。

`React.memo`也经常被另一种非标量类型——函数所绕过。考虑以下情况：

```
<MemoizedAvatar
  name="Tejas"
  url="https://github.com/tejasq.png"
  onChange={() => save()}
/>
```

尽管 props 看起来没有改变或依赖于封闭状态，但如果我们比较 props，我们会看到以下内容：

```
"Tejas" === "Tejas"; // <- `name` prop; true
"https://github.com/tejasq.png" === "https://github.com/tejasq.png";

(() => save()) === (() => save()); // <- `onChange` prop; false
```

再次强调，这是因为我们通过引用比较函数。请记住，只要 props 不同，我们的组件就不会被记忆化。我们可以通过在 `MemoizedAvatar` 的父组件中使用 `useCallback` 钩子来解决这个问题：

```
const Parent = ({ currentUser }) => {
  const onAvatarChange = useCallback(
    (newAvatarUrl) => {
      updateUserModel({ avatarUrl: newAvatarUrl, id: currentUser.id });
    },
    [currentUser]
  );

  return (
    <MemoizedAvatar
      name="Tejas"
      url="https://github.com/tejasq.png"
      onChange={onAvatarChange}
    />
  );
};
```

现在我们可以确信 `onAvatarChange` 只有在其依赖数组中的某个东西（第二个参数）更改时才会改变。有了这个，我们的记忆化完全完成并可靠。这是记忆化具有函数作为 props 的组件的推荐方式。

很好！现在这意味着我们的记忆化组件将不会不必要地重新渲染。对吗？错了！我们还需要注意一件事。

## 这是一个指导方针，而不是一个规则

React 使用 `React.memo` 作为向其调和器的提示，表明如果它们的 props 保持不变，我们不希望我们的组件重新渲染。这个函数只是给 React 一个提示。最终，React 的操作由 React 决定。`React.memo` 一致地避免从父级开始的重新渲染级联，这是它的唯一目的。它不能保证组件永远不会重新渲染。回想一下本书开头的话，React 旨在成为我们用户界面的声明性抽象，在这里我们描述我们想要的*是什么*，React 找到最佳实现*如何*。`React.memo` 是这个过程的一部分。

`React.memo` 不能保证始终避免重新渲染，因为 React 可能会因为各种原因重新渲染记忆化组件，比如组件树的更改或应用程序的全局状态的更改。

要更好地理解这一点，让我们看一些来自 React 源代码的代码片段。

首先，让我们看一下 `React.memo` 的实现：

```
function memo(type, compare) {
  return {
    $$typeof: REACT_MEMO_TYPE,
    type,
    compare: compare === undefined ? null : compare,
  };
}
```

在这个实现中，`React.memo` 返回一个新对象，表示记忆化组件。该对象具有一个 `$$typeof` 属性，用于标识它为记忆化组件，一个 `type` 属性，引用原始组件，以及一个 `compare` 属性，指定用于记忆化的比较函数。

接下来，让我们看一下调和器中如何使用 `React.memo`：

```
function updateMemoComponent(
  current: Fiber | null,
  workInProgress: Fiber,
  Component: any,
  nextProps: any,
  renderLanes: Lanes
): null | Fiber {
  if (current === null) {
    const type = Component.type;
    if (
      isSimpleFunctionComponent(type) &&
      Component.compare === null &&
      // SimpleMemoComponent codepath doesn't resolve outer props either.
      Component.defaultProps === undefined
    ) {
      let resolvedType = type;
      if (__DEV__) {
        resolvedType = resolveFunctionForHotReloading(type);
      }
      // If this is a plain function component without default props,
      // and with only the default shallow comparison, we upgrade it
      // to a SimpleMemoComponent to allow fast path updates.
      workInProgress.tag = SimpleMemoComponent;
      workInProgress.type = resolvedType;
      if (__DEV__) {
        validateFunctionComponentInDev(workInProgress, type);
      }
      return updateSimpleMemoComponent(
        current,
        workInProgress,
        resolvedType,
        nextProps,
        renderLanes
      );
    }
    if (__DEV__) {
      const innerPropTypes = type.propTypes;
      if (innerPropTypes) {
        // Inner memo component props aren't currently validated in createElement
        // We could move it there, but we'd still need this for lazy code path.
        checkPropTypes(
          innerPropTypes,
          nextProps, // Resolved props
          "prop",
          getComponentNameFromType(type)
        );
      }
      if (Component.defaultProps !== undefined) {
        const componentName = getComponentNameFromType(type) || "Unknown";
        if (!didWarnAboutDefaultPropsOnFunctionComponent[componentName]) {
          console.error(
            "%s: Support for defaultProps will be removed from components " +
              "in a future release. Use JavaScript default parameters instead.",
            componentName
          );
          didWarnAboutDefaultPropsOnFunctionComponent[componentName] = true;
        }
      }
    }
    const child = createFiberFromTypeAndProps(
      Component.type,
      null,
      nextProps,
      null,
      workInProgress,
      workInProgress.mode,
      renderLanes
    );
    child.ref = workInProgress.ref;
    child.return = workInProgress;
    workInProgress.child = child;
    return child;
  }
  if (__DEV__) {
    const type = Component.type;
    const innerPropTypes = type.propTypes;
    if (innerPropTypes) {
      // Inner memo component props aren't currently validated in createElement
      // We could move it there, but we'd still need this for lazy code path.
      checkPropTypes(
        innerPropTypes,
        nextProps, // Resolved props
        "prop",
        getComponentNameFromType(type)
      );
    }
  }
  // This is always exactly one child
  const currentChild = ((current.child: any): Fiber);
  const hasScheduledUpdateOrContext = checkScheduledUpdateOrContext(
    current,
    renderLanes
  );
  if (!hasScheduledUpdateOrContext) {
    // This will be the props with resolved defaultProps,
    // unlike current.memoizedProps which will be the unresolved ones.
    const prevProps = currentChild.memoizedProps;
    // Default to shallow comparison
    let compare = Component.compare;
    compare = compare !== null ? compare : shallowEqual;
    if (compare(prevProps, nextProps) && current.ref === workInProgress.ref) {
      return bailoutOnAlreadyFinishedWork(current, workInProgress, renderLanes);
    }
  }
  // React DevTools reads this flag.
  workInProgress.flags |= PerformedWork;
  const newChild = createWorkInProgress(currentChild, nextProps);
  newChild.ref = workInProgress.ref;
  newChild.return = workInProgress;
  workInProgress.child = newChild;
  return newChild;
}
```

这里是正在发生的事情的详细解释：

1. 初始检查

函数 `updateMemoComponent` 接受几个参数，包括当前和工作中的 Fiber，组件，新 props，以及渲染 lanes（如前所述，指示更新的优先级和时机）。初始检查（`if (current === null)`）确定这是否是组件的初始渲染。如果 `current` 是 `null`，则表示组件是首次挂载。

2\. 类型和快速路径优化

然后，它检查组件是否是一个简单的函数组件，并且是否符合快速更新路径的条件，通过检查`Component.compare`和`Component.defaultProps`。如果满足这些条件，它将工作中的 Fiber 的标签设置为`SimpleMemoComponent`，表示这是一种更简单的组件类型，可以更高效地更新。

3\. 开发模式检查

在开发模式下（`__DEV__`），该函数执行额外的检查，如验证属性类型和警告有关函数组件中已弃用功能（如`defaultProps`）的内容。

4\. 创建新的 Fiber

如果是初始渲染，将使用`createFiberFromTypeAndProps`创建一个新的 Fiber。这个 Fiber 表示 React 渲染器的工作单元。它设置引用并返回子 Fiber（新的 Fiber）。

5\. 更新现有的 Fiber

如果组件正在更新（`current !== null`），它执行类似的开发模式检查。然后，它通过浅比较（`shallowEqual`）或自定义比较函数（如果提供了）来比较旧的属性和新的属性，以判断组件是否需要更新。

6\. 提前退出更新

如果属性相等且引用未更改，则可以通过`bailoutOnAlreadyFinishedWork`提前退出更新，这意味着此组件无需进一步的渲染工作。

7\. 更新工作中的 Fiber

如果需要更新，函数将使用`PerformedWork`标记工作中的 Fiber，并基于当前子 Fiber 创建一个新的工作中子 Fiber，但带有新的 props。

总结一下，这个函数负责确定 memoized 组件（使用`React.memo`包装的组件）是否需要更新，或者是否可以跳过更新以优化性能。它处理初始渲染和更新，根据是否创建新的 Fiber 或更新现有的 Fiber 执行不同的操作。

以下部分告诉我们有关`React.memo`组件何时重新渲染或不重新渲染的条件：

没有先前的渲染（初始挂载）

如果`current === null`，则该组件是首次挂载，因此无法跳过渲染。将创建一个新的 Fiber 并返回，用于组件的渲染。

简单函数组件优化

如果组件是一个简单的函数组件（没有默认属性且没有自定义比较函数），React 将优化它为`SimpleMemoComponent`。这使得 React 可以使用快速更新路径，因为它可以假定组件仅依赖于其属性，而浅比较就足以确定是否应该更新。

比较函数

如果有先前的渲染，仅当比较函数返回 `false` 时，组件才会更新。如果提供了比较函数，则此比较函数可以自定义，否则默认为浅比较（`shallowEqual`）。如果比较函数确定新的 props 等于先前的 props，并且 `ref` 也相同，则组件不会重新渲染，函数将退出渲染过程。

开发中的默认 props 和 prop 类型

在开发模式（`__DEV__`）下，会对 `defaultProps` 和 `propTypes` 进行检查。在函数组件中使用 `defaultProps` 会在开发模式下触发警告，因为 React 的未来版本计划废弃函数组件上的 `defaultProps`。Prop 类型会被检查以进行验证。

退出条件

如果没有计划的更新或上下文变化（`hasScheduledUpdateOr​Con⁠text` 为 `false`），比较函数认为旧 props 和新 props 相等，并且 `ref` 没有改变，那么函数将返回 `bailoutOnAlreadyFinishedWork` 的结果，从而有效地跳过重新渲染。

然而，如果有计划的上下文更新，组件将重新渲染，即使其 props 没有变化。这是因为上下文更新被认为是超出组件 props 范围之外的内容。状态变化、上下文变化和计划更新也可以触发重新渲染。

执行的工作标志

如果需要更新，则在 `workInProgress` Fiber 上设置 `PerformedWork` 标志，表示此 Fiber 在当前渲染期间已执行工作。

因此，如果使用 `React.memo` 组件，在旧 props 和新 props 之间的比较（使用自定义的比较函数或默认的浅比较）确定 props 相等，并且没有由于状态或上下文变化而计划更新，那么组件将不会重新渲染。如果 props 被确定为不同，或者存在状态或上下文变化，组件将重新渲染。

# 使用 `useMemo` 进行记忆化

`React.memo` 和 `useMemo` 钩子都是记忆化的工具，但目的完全不同。`React.memo` 记忆化整个组件以防止其重新渲染。`useMemo` 则记忆化组件内的特定计算，以避免昂贵的重新计算并保持结果的一致引用。

让我们简要地深入了解 `useMemo`。考虑一个组件：

```
const People = ({ unsortedPeople }) => {
  const [name, setName] = useState("");
  const sortedPeople = unsortedPeople.sort((a, b) => b.age - a.age);

  // ... rest of the component
};
```

由于其排序操作，此组件可能会潜在地减慢我们的应用程序。排序操作的时间复杂度通常为平均情况和最坏情况下的 O(n log n)。例如，如果我们的列表有一百万人，每次渲染可能会涉及显著的计算开销。在计算机科学术语中，排序操作的效率很大程度上取决于项数 n，因此时间复杂度为 O(n log n)。

为了优化此过程，我们将使用 `useMemo` 钩子，避免在每次渲染时对 people 数组进行排序，尤其是在 `unsortedPeople` 数组未更改时。

组件的当前实现存在显著的性能问题。每次状态更新时（即在输入字段内每次按键时），组件都会重新渲染。如果输入一个包含 5 个字符的名称，而我们的列表包含 100 万人，组件将重新渲染 5 次。每次渲染时，它将对列表进行排序，这涉及到大约 100 万 × log(100 万)的操作，由于排序的时间复杂度。这相当于仅仅输入一个五字符的名称就需要执行许多百万次操作！幸运的是，可以使用`useMemo`钩子来减轻这种效率低下，确保只在`unsortedPeople`数组变化时执行排序操作。

让我们稍微改写一下这段代码片段：

```
const People = ({ unsortedPeople }) => {
  const [name, setName] = useState("");
  const sortedPeople = useMemo(
    // Spreading so we don't mutate the original array
    () => [...unsortedPeople].sort((a, b) => b.age - a.age),
    [unsortedPeople]
  );

  return (
    <div>
      <div>
        Enter your name:{" "}
        <input
          type="text"
          placeholder="Obinna Ekwuno"
          onChange={(e) => setName(e.target.value)}
        />
      </div>
      <h1>Hi, {name}! Here's a list of people sorted by age!</h1>
      <ul>
        {sortedPeople.map((p) => (
          <li key={p.id}>
            {p.name}, age {p.age}
          </li>
        ))}
      </ul>
    </div>
  );
};
```

好多了！我们将`sortedPeople`的值包装在传递给`useMemo`第一个参数的函数中。我们传递给`useMemo`的第二个参数表示一个值数组，如果变化，则重新排序数组。由于数组只包含`unsortedPeople`，因此它只会在人员列表发生更改时重新排序数组，而不是每当有人在姓名输入字段中键入时。这是如何使用`useMemo`避免不必要重新渲染的绝佳示例。

## [Memo 化的考虑](https://wiki.example.org/memoization_considered_harmful)

尽管将所有变量声明包装在`useMemo`组件内部可能很诱人，但这并不总是有利的。`useMemo`特别适用于记忆化计算密集型操作或保持对象和数组的稳定引用。对于标量值（如字符串、数字或布尔值），通常不需要使用`useMemo`。这是因为这些标量值在 JavaScript 中是通过它们的实际值传递和比较的，而不是通过引用。因此，每次设置或比较标量值时，你都在处理的是实际值，而不是可能会变化的内存位置的引用。

在这些情况下，加载和执行`useMemo`函数可能比其试图优化的实际操作更昂贵。例如，考虑以下示例：

```
const MyComponent = () => {
  const [count, setCount] = useState(0);
  const doubledCount = useMemo(() => count * 2, [count]);

  return (
    <div>
      <p>Count: {count}</p>
      <p>Doubled count: {doubledCount}</p>
      <button onClick={() => setCount((oldCount) => oldCount + 1)}>
        Increment
      </button>
    </div>
  );
};
```

在这个例子中，`doubledCount`变量使用`useMemo`钩子进行了记忆化。然而，由于`count`是一个标量值，没有必要进行记忆化。相反，我们可以直接在 JSX 中计算加倍后的计数：

```
const MyComponent = () => {
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>Count: {count}</p>
      <p>Doubled count: {count * 2}</p>
      <button onClick={() => setCount((oldCount) => oldCount + 1)}>
        Increment
      </button>
    </div>
  );
};
```

现在，`doubledCount`不再是记忆化的，但组件仍然执行相同的计算，内存消耗更少，开销更小，因为我们没有导入和调用`useMemo`。这是一个很好的例子，说明在不必要时如何避免使用`useMemo`。

然而，可能会出现额外的性能问题，因为我们在每次渲染时重新创建按钮上的`onClick`处理程序，因为它是通过内存引用传递的。但这真的是个问题吗？让我们仔细看看。

有人建议我们应该使用`useCallback`来记忆化`onClick`处理程序：

```
const MyComponent = () => {
  const [count, setCount] = useState(0);
  const doubledCount = useMemo(() => count * 2, [count]);
  const increment = useCallback(
    () => setCount((oldCount) => oldCount + 1),
    [setCount]
  );

  return (
    <div>
      <p>Count: {count}</p>
      <p>Doubled count: {doubledCount}</p>
      <button onClick={increment}>Increment</button>
    </div>
  );
};
```

应该这样做吗？答案是否定的。在这里将增量函数进行记忆化并没有好处，因为`<button>`是一个浏览器原生元素，而不是可以调用的 React 函数组件。此外，它下面也没有更多的组件可以继续渲染。

此外，在 React 中，内置或“主机”组件（如`div`，`button`，`input`等）在处理 props 时与自定义组件略有不同，包括函数 props。

这是在内置组件上处理函数 props 时发生的情况：

直接传递

当您将函数 prop（如`onClick`处理程序）传递给内置组件时，React 会将其直接传递给实际的 DOM 元素。它不会为这些函数创建任何包装器或执行任何额外的工作。

然而，在`onClick`和基于事件的 props 的情况下，React 使用事件委托来处理事件，而不是直接将事件处理程序附加到 DOM 元素上。这意味着当你为像`<button>`这样的内置 React 元素提供一个`onClick`处理程序时，React 并不直接将`onClick`处理程序附加到按钮的 DOM 节点上。相反，React 在顶层使用单个事件侦听器监听所有事件。这个侦听器附加到文档的根部（或 React 应用程序的根部），并依赖事件冒泡来捕获来自各个元素的事件。这种方法是高效的，因为它减少了事件处理程序的内存占用和初始设置时间。React 不必为每个元素上每个事件实例附加和管理单独的处理程序，而是可以使用单个真实事件侦听器处理特定类型的所有事件（例如点击）。当事件发生时，React 会将其映射到适当的组件，并按照预期的传播路径调用您定义的处理程序。因此，即使事件是在顶层捕获的，它们的行为也会表现得好像它们是直接附加到特定元素上的。在编写 React 应用程序时，这种事件委托系统通常是透明的；您定义`onClick`处理程序的方式与直接附加处理程序时相同。然而，在幕后，React 正在为您优化事件处理。

重新渲染行为

由于函数 props 的变化不会导致内置组件重新渲染，除非它们是属于已经重新渲染的更高级组件的一部分。例如，如果父组件重新渲染并为内置组件提供一个新的函数作为 props，那么内置组件将重新渲染，因为其 props 已经改变。然而，这种重新渲染通常是快速的，通常不需要优化，除非分析显示它是一个问题。

没有函数的虚拟 DOM 比较

对于内置组件，虚拟 DOM 的比较是基于函数属性的标识。如果传递一个内联函数（例如 `onClick={() => do​Something()}`），每次组件渲染时都会是一个新函数，但 React 不会对函数进行深度比较以检测更改。新函数简单地替换旧函数在 DOM 元素上，因此我们在内置组件中获得了性能上的节省。

事件池化

React 使用事件池化来减少事件处理程序的内存开销。传递给事件处理程序的事件对象是一个合成事件，它是池化的，意味着它被重用于不同的事件，以减少垃圾回收的开销。

这与自定义组件形成了鲜明的对比。对于自定义组件，如果将一个新的函数作为属性传递，子组件可能会重新渲染，如果它是一个纯组件或者应用了记忆化（如`React.memo`），因为它检测到了属性的变化。但是对于宿主组件，React 不提供这种内置的记忆化，因为这样做在大多数情况下没有好处。React 输出的实际 DOM 元素没有记忆化的概念；它们只是在属性改变时更新为新的函数引用。

实际应用中，这意味着虽然您应该谨慎地将新的函数实例传递给可能昂贵重新渲染的自定义组件，但对于内置组件来说，这并不是那么值得关注。然而，始终注意您创建新函数和传递它们的频率，因为不必要的函数创建可能会导致垃圾回收的频繁触发，在非常高频率的更新场景中可能会成为性能问题。

因此，在这里 `useCallback` 根本没有帮助，实际上比没有使用还要糟糕：它不仅不提供任何价值，而且还给我们的应用程序增加了额外的开销。这是因为 `useCallback` 必须被导入、调用并传递依赖项，然后它必须比较依赖项，以确定是否应该重新计算函数。所有这些都具有运行时复杂性，可能对我们的应用程序造成更多的伤害而不是帮助。

`useCallback` 的一个很好的例子是什么？当你有一个组件可能经常重新渲染并且将回调传递给子组件时，特别是如果该子组件使用了`React.memo`或`shouldComponentUpdate`进行了优化时，`useCallback`特别有用。回调函数的记忆化确保了当父组件重新渲染时，子组件不会不必要地重新渲染。

下面是 `useCallback` 有益的一个例子：

```
import React, { useState, useCallback } from "react";

const ExpensiveComponent = React.memo(({ onButtonClick }) => {
  // This component is expensive to render and we want
  // to avoid unnecessary renders
  // We're just simulating something expensive here
  const now = performance.now();
  while (performance.now() - now < 1000) {
    // Artificial delay -- block for 1000ms
  }

  return <button onClick={onButtonClick}>Click Me</button>;
});

const MyComponent = () => {
  const [count, setCount] = useState(0);
  const [otherState, setOtherState] = useState(0);

  // This callback is memoized and will only change if count changes
  const incrementCount = useCallback(() => {
    setCount((prevCount) => prevCount + 1);
  }, []); // Dependency array

  // This state update will cause MyComponent to rerender
  const doSomethingElse = () => {
    setOtherState((s) => s + 1);
  };

  return (
    <div>
      <p>Count: {count}</p>
      <ExpensiveComponent onButtonClick={incrementCount} />
      <button onClick={doSomethingElse}>Do Something Else</button>
    </div>
  );
};
```

在这个例子中：

+   `ExpensiveComponent` 是一个包裹在 `React.memo` 中的子组件，这意味着它仅在其 props 发生变化时重新渲染。这是一个你希望避免在每次渲染时传递新函数实例的情况。

+   `MyComponent` 有两个状态：`count` 和 `otherState`。

+   `incrementCount`是一个更新`count`的回调函数。它通过`use​Call⁠back`进行了记忆化，这意味着当`MyComponent`由于`otherState`的更改而重新渲染时，`ExpensiveComponent`不会重新渲染。

+   `doSomethingElse`函数改变了`otherState`，但不需要使用`useCallback`进行记忆化，因为它不会传递给`Expensive​Compo⁠nent`或任何其他子组件。

通过使用`useCallback`，我们确保`ExpensiveComponent`在`MyComponent`重新渲染时不会不必要地重新渲染，原因与`count`无关。在子组件的渲染是一个耗时操作且你希望通过减少渲染次数来优化性能的情况下，这是有益的。

这是一个很好的例子，展示了如何使用`useCallback`来避免不必要的重新渲染，确保传递给昂贵组件的函数仅创建一次，并且在重新渲染过程中保持相同的引用。这可以防止昂贵组件的不必要重新渲染。`useCallback`本质上是函数的`useMemo`。

让我们考虑另一个例子：

```
const MyComponent = () => {
  const dateOfBirth = "1993-02-19";
  const isAdult =
    new Date().getFullYear() - new Date(dateOfBirth).getFullYear() >= 18;

  if (isAdult) {
    return <h1>You are an adult!</h1>;
  } else {
    return <h1>You are a minor!</h1>;
  }
};
```

我们这里没有使用`useMemo`，主要是因为组件是无状态的。这是好事！但是如果有一些输入会触发类似这样的重新渲染，那该怎么办呢：

```
const MyComponent = () => {
  const [birthYear, setBirthYear] = useState(1993);
  const isAdult = new Date().getFullYear() - birthYear >= 18;

  return (
    <div>
      <label>
        Birth year:
        <input
          type="number"
          value={birthYear}
          onChange={(e) => setBirthYear(e.target.value)}
        />
      </label>
      {isAdult ? <h1>You are an adult!</h1> : <h1>You are a minor!</h1>}
    </div>
  );
};
```

现在我们在每次按键时重新计算`new Date()`。让我们用`useMemo`修复这个问题：

```
const MyComponent = () => {
  const [birthYear, setBirthYear] = useState(1993);
  const today = useMemo(() => new Date(), []);
  const isAdult = today.getFullYear() - birthYear >= 18;

  return (
    <div>
      <label>
        Birth year:
        <input
          type="number"
          value={birthYear}
          onChange={(e) => setBirthYear(e.target.value)}
        />
      </label>
      {isAdult ? <h1>You are an adult!</h1> : <h1>You are a minor!</h1>}
    </div>
  );
};
```

这是好的，因为`today`将在组件重新渲染时引用相同的对象，并且我们假设组件将始终在同一天重新渲染。

###### 注意

在这里有一个小的边缘情况，如果用户在使用此组件时，他们的时钟在午夜时分归零，但这是一个我们现在可以忽略的罕见边缘情况。当然，当涉及到真实的生产代码时，我们会做得更好。

此示例引发了一个更大的问题：我们是否应该将`isAdult`的值包装在`useMemo`中？如果这样做会发生什么？答案是我们不应该，因为`isAdult`是一个标量值，除了内存分配之外不需要任何计算。我们确实多次调用`.getFullYear`，但我们信任 JavaScript 引擎和 React 运行时来处理性能问题。这是一个简单的赋值，没有进一步的计算，例如排序、过滤或映射。

在这种情况下，我们不应该使用`useMemo`，因为它更可能减慢我们应用程序的速度，而不是加快它，因为`useMemo`本身的开销，包括导入它、调用它、传递依赖项，然后比较依赖项以查看是否应重新计算值。所有这些都具有可能对我们应用程序造成更多伤害而不是帮助的运行时复杂性。相反，我们分配并信任 React 在必要时通过其自身的优化智能重新渲染我们的组件。

即使在面对重计算时，我们的应用程序现在也享受着更快的重新渲染性能好处——但我们能做得更多吗？在下一节中，让我们看看到目前为止我们已经涵盖的所有内容，可能在几年后都不会再重要，基于 React 团队正在致力于为我们自动考虑记忆化的一些激动人心的事物，使我们能够*忘记*细节，而专注于我们的应用程序。

## 忘记这一切

React Forget 是一个旨在自动化 React 应用程序中记忆化的新工具链，有可能使像 `useMemo` 和 `useCallback` 这样的钩子变得多余。通过自动处理记忆化，React Forget 帮助优化组件重新渲染，改善用户体验（UX）和开发者体验（DX）。这种自动化将 React 的重新渲染行为从对象身份更改转变为语义值更改，无需深度比较，从而提升性能。React Forget 于 2021 年的 React Conf 中推出，目前在 Meta（Facebook、Instagram 等）内部已投入使用，并且“在内部已超出预期”。

如果有足够的兴趣，我们将在未来的版本中涵盖 React Forget。请通过社交媒体（尤其是 𝕏，之前是 Twitter）发帖并标记作者 *@tejaskumar_* 来告知我们。

# 懒加载

当我们的应用程序增长时，我们积累了大量 JavaScript。然后，我们的用户下载这些庞大的 JavaScript 捆绑包——有时会达到数十兆字节——但实际上只使用其中的一小部分代码。这是一个问题，因为它减慢了用户的初始加载时间，也减慢了用户后续的页面加载时间，特别是当我们无法访问提供这些捆绑包的服务器，并且无法添加必需的缓存头等时。

运用过多 JavaScript 的一个主要问题是它会减慢页面加载时间。JavaScript 文件通常比其他类型的网页资产（如 HTML 和 CSS）要大，执行时需要更多处理时间。这可能导致页面加载时间变长，尤其是在较慢的互联网连接或旧设备上。

例如，请考虑以下加载大型 JavaScript 文件的代码片段：

```
<!DOCTYPE html>
<html>
  <head>
    <title>My Website</title>
    <script src="https://example.com/large.js"></script>
  </head>
  <body>
    <!-- Page content goes here -->
  </body>
</html>
```

在这个例子中，*large.js* 文件在页面的 `<head>` 部分加载，这意味着它将在页面上的任何其他内容之前执行。这可能导致页面加载时间变慢，尤其是在较慢的互联网连接或旧设备上。解决这个问题的常见方法是使用 `async` 属性异步加载 JavaScript 文件：

```
<!DOCTYPE html>
<html>
  <head>
    <title>My Website</title>
    <script async src="https://example.com/large.js"></script>
  </head>
  <body>
    <!-- Page content goes here -->
  </body>
</html>
```

在这个例子中，*large.js* 文件使用 `async` 属性进行异步加载。这意味着它将与页面上的其他资源并行下载，有助于提高页面加载时间。

发送太多 JavaScript 的另一个问题是可能增加数据使用量。JavaScript 捆绑包通常比其他类型的 Web 资产更大，这意味着它们需要通过网络传输更多的数据。对于拥有有限数据计划或较慢的互联网连接的用户来说，这可能是一个问题，因为它可能导致成本增加和页面加载时间变慢。

为了缓解这些问题，我们可以采取几个步骤来减少向用户传送的 JavaScript 量。一种方法是使用代码分割，只加载特定页面或功能所需的 JavaScript。这可以通过只加载必要的代码来帮助减少页面加载时间和数据使用量。

例如，考虑以下使用代码分割来加载特定页面所需的 JavaScript 的代码片段：

```
import("./large.js").then((module) => {
  // Use module here
});
```

在这个例子中，`import` 函数被用来在需要时异步加载 *large.js* 文件。这可以通过只加载必要的代码来帮助减少页面加载时间和数据使用量。

另一种方法是使用懒加载来延迟加载非关键 JavaScript，直到页面加载完成后再加载。这可以通过只在需要时加载非关键代码来帮助减少页面加载时间和数据使用量。

例如，考虑以下使用懒加载来延迟加载非关键 JavaScript 的代码片段：

```
<!DOCTYPE html>
<html>
  <head>
    <title>My Website</title>
  </head>
  <body>
    <!-- Page content goes here -->
    <button id="load-more">Load more content</button>
    <script>
      document.getElementById("load-more").addEventListener("click", () => {
        import("./non-critical.js").then((module) => {
          // Use module here
        });
      });
    </script>
  </body>
</html>
```

在这个例子中，`import` 函数被用来在“加载更多内容”按钮被点击时异步加载 *non-critical.js* 文件。这可以帮助减少页面加载时间和数据使用量，只在需要时加载非关键代码。

幸运的是，React 提供了一个解决方案，使这一切更加简单：使用 `React.lazy` 和 `Suspense` 进行懒加载。让我们看看如何使用它们来提高我们应用的性能。

懒加载是一种技术，允许我们仅在需要时加载组件，就像前面示例中的动态导入一样。对于具有许多不需要在初始渲染时加载的组件的大型应用程序非常有用。例如，如果我们有一个大型应用程序，侧边栏可以折叠，并且具有指向其他页面的链接列表，如果侧边栏在首次加载时是折叠状态，我们可能不希望加载完整的侧边栏。相反，我们可以在用户切换侧边栏时再加载它。

让我们来探索以下代码示例：

```
import Sidebar from "./Sidebar"; // 22MB to import

const MyComponent = ({ initialSidebarState }) => {
  const [showSidebar, setShowSidebar] = useState(initialSidebarState);

  return (
    <div>
      <button onClick={() => setShowSidebar(!showSidebar)}>
        Toggle sidebar
      </button>
      {showSidebar && <Sidebar />}
    </div>
  );
};
```

在这个例子中，如果 `<Sidebar />` 是 22 MB 的 JavaScript。这是一大块需要下载、解析和执行的 JavaScript，在侧边栏首次渲染时如果折叠了，这并不是必需的。相反，只有在 `showSidebar` 为 true 时，我们才能使用 `React.lazy` 来懒加载组件。也就是说，只在需要时加载：

```
import { lazy, Suspense } from "react";
import FakeSidebarShell from "./FakeSidebarShell"; // 1kB to import

const Sidebar = lazy(() => import("./Sidebar"));

const MyComponent = ({ initialSidebarState }) => {
  const [showSidebar, setShowSidebar] = useState(initialSidebarState);

  return (
    <div>
      <button onClick={() => setShowSidebar(!showSidebar)}>
        Toggle sidebar
      </button>
      <Suspense fallback={<FakeSidebarShell />}>
        {showSidebar && <Sidebar />}
      </Suspense>
    </div>
  );
};
```

而不是静态导入`./Sidebar`，我们动态导入它——也就是说，我们将一个返回导入模块的 promise 的函数传递给`lazy`。动态导入返回一个 promise，因为模块可能不会立即可用。它可能需要首先从服务器下载。React 的`lazy`函数触发导入，只有在需要渲染底层组件（在这种情况下是`Sidebar`）时才会调用。这样，我们避免在实际*渲染*`<Sidebar />`之前将 22 MB 的侧边栏传输过来。

你可能还注意到了另一个新的导入：`Suspense`。我们使用`Suspense`来包装树中的组件。`Suspense`是一个组件，允许我们在 promise 正在解析时显示一个 fallback 组件（即侧边栏正在下载时）。在片段中，我们展示了一个轻量级版本的重型侧边栏作为 fallback 组件，而重型侧边栏正在下载中。这是在侧边栏加载时为用户提供即时反馈的好方法。

现在，当用户点击按钮切换侧边栏时，他们会看到一个“骨架 UI”，可以在侧边栏加载和渲染时进行定位。

## 使用 Suspense 实现更大的 UI 控制

React Suspense 的工作方式类似于 try/catch 块。你知道你可以从代码中的任何位置`throw`一个异常，然后在其他地方（甚至是不同的模块中）使用`catch`块捕获它吗？嗯，Suspense 以类似的方式（但不完全相同）工作。你可以在组件树的任何位置放置延迟加载和异步基元，然后在树中的任何上层使用`Suspense`组件捕获它们，即使你的 Suspense 边界在完全不同的文件中。

了解了这些，我们有能力选择在哪里显示我们的 22 MB 侧边栏的加载状态。例如，我们可以在侧边栏加载时隐藏整个应用程序——这是一个相当糟糕的想法，因为我们为了一个侧边栏而阻止了整个应用程序的信息传递给用户——或者我们可以仅显示侧边栏的加载状态。让我们看看如何做前者（即使我们不应该这样做）只是为了理解`Suspense`的能力：

```
import { lazy, Suspense } from "react";

const Sidebar = lazy(() => import("./Sidebar"));

const MyComponent = () => {
  const [showSidebar, setShowSidebar] = useState(false);

  return (
    <Suspense fallback={<p>Loading...</p>}>
      <div>
        <button onClick={() => setShowSidebar(!showSidebar)}>
          Toggle sidebar
        </button>
        {showSidebar && <Sidebar />}
        <main>
          <p>Hello hello welcome, this is the app's main area</p>
        </main>
      </div>
    </Suspense>
  );
};
```

通过将整个组件包裹在`Suspense`中，我们在所有异步子组件（即 promises）解析之前渲染`fallback`。这意味着整个应用程序在侧边栏加载之前都是隐藏的。如果我们希望等到所有准备就绪后再向用户显示界面，这是很有用的，但在这种情况下可能不是最佳选择，因为用户会不知所措，无法与应用程序进行交互。

这就是为什么我们应该只使用`Suspense`来包装需要延迟加载的组件，就像这样：

```
import { lazy, Suspense } from "react";

const Sidebar = lazy(() => import("./Sidebar"));

const MyComponent = () => {
  const [showSidebar, setShowSidebar] = useState(false);

  return (
    <div>
      <button onClick={() => setShowSidebar(!showSidebar)}>
        Toggle sidebar
      </button>
      <Suspense fallback={<p>Loading...</p>}>
        {showSidebar && <Sidebar />}
      </Suspense>
      <main>
        <p>Hello hello welcome, this is the app's main area</p>
      </main>
    </div>
  );
};
```

Suspense 边界是一个非常强大的原语，可以修复布局移动，并使用户界面更加响应和直观。这是一个非常好的工具。此外，如果在`fallback`中使用高质量的骨架 UI，我们可以进一步指导用户了解正在发生的事情，并在我们的延迟加载组件加载时引导他们了解即将与之交互的界面。充分利用这一点是改善我们应用性能的好方法，并流畅地利用 React 的最佳方式。

接下来，我们将看看另一个许多 React 开发者问的有趣问题：何时应该使用`useState`而不是`useReducer`？

# useState 与 useReducer

React 暴露了两个用于管理状态的钩子：`useState`和`useReducer`。这两个钩子都用于在组件中管理状态。它们之间的区别在于，`useState`更适合管理单个状态片段，而`useReducer`更适合管理更复杂的状态。让我们看看如何在组件中使用`useState`来管理状态：

```
import { useState } from "react";

const MyComponent = () => {
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </div>
  );
};
```

在这个例子中，我们使用`useState`来管理一个单独的状态：`count`。但如果我们的状态稍微复杂呢？

```
import { useState } from "react";

const MyComponent = () => {
  const [state, setState] = useState({
    count: 0,
    name: "Tejumma",
    age: 30,
  });

  return (
    <div>
      <p>Count: {state.count}</p>
      <p>Name: {state.name}</p>
      <p>Age: {state.age}</p>
      <button onClick={() => setState({ ...state, count: state.count + 1 })}>
        Increment
      </button>
    </div>
  );
};
```

现在我们可以看到，我们的状态变得有些复杂。我们有一个`count`、一个`name`和一个`age`。通过点击按钮，我们可以增加`count`，这会将状态设置为一个*新对象*，该对象具有与上一个状态相同的属性，但`count`增加了`1`。这在 React 中非常常见。但问题是，这可能会引发 bug 的可能性。例如，如果我们没有仔细地展开旧状态，可能会意外地覆盖一些状态的属性。

有趣的事实：`useState`在内部使用`useReducer`。你可以将`useState`看作是对`useReducer`的更高级抽象。实际上，如果你愿意，你可以用`useReducer`重新实现`useState`！

说真的，你只需要这样做：

```
import { useReducer } from "react";

function useState(initialState) {
  const [state, dispatch] = useReducer(
    (state, newValue) => newValue,
    initialState
  );
  return [state, dispatch];
}
```

让我们看看相同的例子，但是使用`useReducer`实现：

```
import { useReducer } from "react";

const initialState = {
  count: 0,
  name: "Tejumma",
  age: 30,
};

const reducer = (state, action) => {
  switch (action.type) {
    case "increment":
      return { ...state, count: state.count + 1 };
    default:
      return state;
  }
};

const MyComponent = () => {
  const [state, dispatch] = useReducer(reducer, initialState);

  return (
    <div>
      <p>Count: {state.count}</p>
      <p>Name: {state.name}</p>
      <p>Age: {state.age}</p>
      <button onClick={() => dispatch({ type: "increment" })}>Increment</button>
    </div>
  );
};
```

现在，有些人会说这比`useState`多了一些冗余，许多人也会同意，但这在抽象堆栈中向下一个级别时是可以预料的：抽象级别越低，代码越冗长。毕竟，抽象的目的是在大多数情况下用语法糖替换复杂逻辑。既然我们可以用`useState`做与`useReducer`一样的事情，为什么不总是使用`useState`，因为它更简单呢？

使用`useReducer`回答这个问题有三个主要好处：

+   它将更新状态的逻辑与组件分离。它的伴随`reducer`函数可以独立测试，并且可以在其他组件中重用。这是保持我们组件简洁和清晰，同时遵循*单一责任原则*的一个好方法。

    我们可以这样测试 reducer：

    ```
    describe("reducer", () => {
      test("should increment count when given an increment action", () => {
        const initialState = {
          count: 0,
          name: "Tejumma",
          age: 30,
        };
        const action = { type: "increment" };
        const expectedState = {
          count: 1,
          name: "Tejumma",
          age: 30,
        };
        const actualState = reducer(initialState, action);
        expect(actualState).toEqual(expectedState);
      });

      test("should return the same object when given an unknown action",
        () => {
        const initialState = {
          count: 0,
          name: "Tejumma",
          age: 30,
        };
        const action = { type: "unknown" };
        const expectedState = initialState;
        const actualState = reducer(initialState, action);
        expect(actualState).toBe(expectedState);
      });
    });
    ```

    在这个例子中，我们正在测试两种情况：一种是分派增量操作到 reducer，另一种是分派未知操作。

    在第一个测试中，我们创建了一个初始状态对象，其计数值为`0`，并创建了一个增量操作对象。我们期望结果状态对象中的计数值增加到`1`。我们使用`toEqual`匹配器来比较预期和实际的状态对象。

    在第二个测试中，我们创建了一个初始状态对象，其计数值为`0`，并创建了一个未知操作对象。然后我们期望结果状态对象与初始状态对象相同。我们使用`toBe`匹配器来比较预期和实际的状态对象，因为我们测试的是引用相等性。

    通过这种方式测试我们的 reducer，我们可以确保它在给定不同输入场景时行为正确并产生预期的输出。

+   使用`useReducer`，我们的状态及其变化始终是明确的，有人会认为`useState`可能会通过多层 JSX 树来混淆组件的整体状态更新流程。

+   `useReducer`是一种*事件源模型*，意味着它可以用于建模应用程序中发生的事件，然后我们可以在某种审计日志中跟踪这些事件。这个审计日志可以用来重放应用程序中的事件以重现错误或实现*时间旅行调试*。它还能支持一些强大的模式，如撤销/重做、乐观更新和跟踪用户界面上常见用户操作的分析。

虽然`useReducer`是一个很好的工具，但并不总是必需的。事实上，对于大多数用例来说，它通常是过度复杂的。那么我们何时应该使用`useState`而不是`useReducer`？答案是取决于您的状态的复杂性。但希望通过所有这些信息，您能在应用程序中更明智地选择使用哪个。

## Immer 和人性化设计

Immer，一个流行的 React 库，在处理应用程序中复杂状态管理时特别有用。当您的状态形状是嵌套或复杂的时候，传统的状态更新方法可能会变得冗长且容易出错。Immer 通过允许您使用可变草稿状态来处理这些复杂性，同时确保生成的状态是不可变的，有助于管理这些复杂性。

在 React 应用程序中，状态管理通常使用`useState`或`useReducer`钩子处理。虽然`useState`适用于简单的状态，但`useReducer`更适用于复杂的状态管理，这正是 Immer 最擅长的地方。

当使用 `useReducer` 时，提供的 reducer 函数应当是纯的，并始终返回一个新的状态对象。当处理嵌套状态对象时，这可能导致冗长的代码。然而，通过在 `use-immer` 库中通过 `useImmerReducer` 将 Immer 集成到 `useReducer` 中，您可以编写看似直接改变状态的 reducer 函数，实际上是在 Immer 提供的草稿状态上操作。这样，您可以编写更简单、更直观的 reducer 函数：

```
import { useImmerReducer } from "use-immer";

const initialState = {
  user: {
    name: "John Doe",
    age: 28,
    address: {
      city: "New York",
      country: "USA",
    },
  },
};

const reducer = (draft, action) => {
  switch (action.type) {
    case "updateName":
      draft.user.name = action.payload;
      break;
    case "updateCity":
      draft.user.address.city = action.payload;
      break;
    // other cases...
    default:
      break;
  }
};

const MyComponent = () => {
  const [state, dispatch] = useImmerReducer(reducer, initialState);

  // ...
};
```

在此示例中，`useImmerReducer` 显著简化了 reducer 函数，允许直接赋值以更新嵌套状态属性，在传统的 reducer 中则需要使用 `spread` 或 `Object.assign` 操作。

此外，Immer 不仅仅局限于 `useReducer`。每当您拥有复杂的状态对象并希望在更新状态时确保不可变性时，您还可以在 `useState` 中使用它。Immer 提供了一个 `produce` 函数，您可以使用它根据当前状态和一组指令创建下一个状态：

```
import produce from "immer";
import { useState } from "react";

const MyComponent = () => {
  const [state, setState] = useState(initialState);

  const updateName = (newName) => {
    setState(
      produce((draft) => {
        draft.user.name = newName;
      })
    );
  };

  // ...
};
```

在 `updateName` 函数中，Immer 的 `produce` 函数接受当前的 `state` 和一个接收状态的 `draft` 的函数。在此函数内部，您可以像处理可变对象一样处理草稿，而 Immer 确保生成的状态是一个新的不可变对象。

Immer 在简化状态更新方面的能力，特别是在复杂或嵌套的状态结构中，使其成为 React 状态管理钩子的绝佳伴侣，促进更干净、更可维护和更少错误的代码。

# 强大的模式

软件设计模式是软件开发中常用的解决方案，用于解决重复出现的问题。它们提供了一种解决已被其他开发人员遇到并解决过的问题的方法，节省了软件开发过程中的时间和精力。通常以模板或指南的形式表达，用于创建可在不同情境中使用的软件。软件设计模式通常使用共同的词汇和符号描述，这使得它们更易于理解和开发人员之间的沟通。它们可以用于提高软件系统的质量、可维护性和效率。

软件设计模式之所以重要有几个原因：

可重用性

设计模式提供了解决常见问题的可重用方案，可以节省软件开发中的时间和精力。

标准化

设计模式提供了解决问题的标准方法，使开发人员更易于理解和相互沟通。

可维护性

设计模式提供了一种易于维护和修改的代码结构方式，可以提高软件系统的持久性。

效率

设计模式提供了解决常见问题的高效方案，可以提高软件系统的性能。

通常，软件设计模式是随着时间的推移自然而然地响应现实需求而产生的。这些模式解决了工程师遇到的具体问题，并成为“工程师工具库”中用于不同用例的工具。*一个模式并不一定比另一个更差*；每种模式都有其适用的场景。

大多数模式帮助我们确定理想的抽象级别：我们如何编写像美酒般经久的代码，而不是累积额外的状态和配置，以至于代码变得难以阅读和/或难以维护。这就是为什么在选择设计模式时常见的考虑因素是*控制*：我们将多少控制权交给用户，而我们的程序又处理了多少控制权。

接下来，让我们深入探讨一些流行的 React 模式，按照这些模式出现的大致时间顺序。

## 展示组件/容器组件

在 React 设计模式中，常见的一种模式是组合两个组件：*展示组件*和*容器组件*。展示组件负责渲染 UI，而容器组件则处理 UI 的状态。以计数器为例，实现该模式的计数器如下所示：

```
const PresentationalCounter = (props) => {
  return (
    <section>
      <button onClick={props.increment}>+</button>
      <button onClick={props.decrement}>-</button>
      <button onClick={props.reset}>Reset</button>
      <h1>Current Count: {props.count}</h1>
    </section>
  );
};

const ContainerCounter = () => {
  const [count, setCount] = useState(0);
  const increment = () => setCount(count + 1);
  const decrement = () => setCount(count - 1);
  const reset = () => setCount(0);

  return (
    <PresentationalCounter
      count={count}
      increment={increment}
      decrement={decrement}
      reset={reset}
    />
  );
};
```

在这个例子中，我们有两个组件：`PresentationalCounter`（一个展示组件）和`ContainerCounter`（一个容器组件）。展示组件负责渲染 UI，而容器组件则处理状态。

为什么要用这种模式？这种模式非常有用，因为它遵循*单一责任*原则，极大地鼓励我们在应用程序中分离关注点，使其更具扩展性，通过更模块化、可重用甚至可测试。我们将组件的外观与功能分开，结果呢？`PresentationalCounter`可以在其他有状态容器之间传递，并保持我们想要的外观，而`ContainerCounter`可以被替换为另一个有状态容器，并保留我们想要的功能。

我们也可以单独对`ContainerCounter`进行单元测试，而使用 Storybook 或类似工具对`PresentationalCounter`进行可视化测试。我们也可以将更喜欢视觉工作的工程师或工程团队分配给`PresentationalCounter`，而将更喜欢数据结构和算法的工程师分配给`ContainerCounter`。

由于这种解耦的方法，我们有更多的选择。因此，容器/展示组件模式因其灵活性而广受欢迎，并且今天仍在使用。然而，使用 hooks 的引入使得在组件中添加状态变得更加方便，而不需要容器组件来管理状态。

如今，在许多情况下，容器/展示模式可以用 hooks 替代。尽管我们仍然可以利用这种模式，即使使用 React Hooks，它在较小的应用程序中也很容易被认为是过度工程化。

## 高阶组件

根据[Wikipedia 关于高阶函数的定义](https://oreil.ly/Ywx56)

> 在数学和计算机科学中，高阶函数（HOF）是至少满足以下条件之一的函数：接受一个或多个函数作为参数（即过程参数，是一个过程的参数，本身又是一个过程），或者以函数作为其结果返回。

在 JSX 世界中，高阶组件（HOC）基本上是这样的：一个接受另一个组件作为参数并返回一个由两者组合而成的新组件。HOC 非常适合*跨组件共享行为*。

例如，许多 Web 应用程序需要异步从某些数据源请求数据。加载和错误状态通常是不可避免的，但我们有时会忘记在软件中考虑它们。如果我们手动为组件添加`loading`、`data`和`error` props，那么我们遗漏几个的机会就更高了。让我们考虑一个基本的待办事项列表应用程序：

```
const App = () => {
  const [data, setData] = useState([]);

  useEffect(() => {
    fetch("https://mytodolist.com/items")
      .then((res) => res.json())
      .then(setData);
  }, []);

  return <BasicTodoList data={data} />;
};
```

此应用程序存在一些问题。我们没有考虑加载或错误状态。让我们来解决这个问题：

```
const App = () => {
  const [isLoading, setIsLoading] = useState(true);
  const [data, setData] = useState([]);
  const [error, setError] = useState([]);

  useEffect(() => {
    fetch("https://mytodolist.com/items")
      .then((res) => res.json())
      .then((data) => {
        setIsLoading(false);
        setData(data);
      })
      .catch(setError);
  }, []);

  return isLoading ? (
    "Loading..."
  ) : error ? (
    error.message
  ) : (
    <BasicTodoList data={data} />
  );
};
```

糟糕。这很快变得非常混乱。此外，*这只解决了一个组件的问题*。我们需要为与外部数据源交互的每个组件添加这些状态（即加载、数据和错误）吗？这是一个*横切关注点*，而 HOC 正是它发挥作用的地方。

与其为每个与异步外部数据源交互的组件重复加载、错误和数据模式，我们可以使用一个 HOC 工厂来处理这些状态。让我们考虑一个解决这个问题的`withAsync` HOC 工厂：

```
const TodoList = withAsync(BasicTodoList);
```

`withAsync`将处理加载和错误状态，并在数据可用时渲染任何组件。让我们看看它的实现：

```
const withAsync = (Component) => (props) => {
  if (props.loading) {
    return "Loading...";
  }

  if (props.error) {
    return error.message;
  }

  return (
    <Component
      // Pass through whatever other props we give `Component`.
      {...props}
    />
  );
};
```

所以现在，当任何`Component`被传递到`withAsync`中时，我们得到一个新的组件，根据其 props 呈现适当的信息。这使我们的初始组件变得更加可行：

```
const TodoList = withAsync(BasicTodoList);

const App = () => {
  const [isLoading, setIsLoading] = useState(true);
  const [data, setData] = useState([]);
  const [error, setError] = useState([]);

  useEffect(() => {
    fetch("https://mytodolist.com/items")
      .then((res) => res.json())
      .then((data) => {
        setIsLoading(false);
        setData(data);
      })
      .catch(setError);
  }, []);

  return <TodoList loading={isLoading} error={error} data={data} />;
};
```

不再有嵌套的三元操作符，而`TodoList`本身可以根据其是否正在加载、是否有错误或是否有数据来显示适当的信息。由于`withAsync`的 HOC 工厂处理这种横切关注点，我们可以用它包装任何与外部数据源交互的组件，并获得一个响应`loading`和`error` props 的新组件。考虑一个博客：

```
const Post = withAsync(BasicPost);
const Comments = withAsync(BasicComments);

const Blog = ({ req }) => {
  const { loading: isPostLoading, error: postLoadError } = usePost(
    req.query.postId
  );
  const { loading: areCommentsLoading, error: commentLoadError } = useComments({
    postId: req.query.postId,
  });

  return (
    <>
      <Post
        id={req.query.postId}
        loading={isPostLoading}
        error={postLoadError}
      />
      <Comments
        postId={req.query.postId}
        loading={areCommentsLoading}
        error={commentLoadError}
      />
    </>
  );
};

export default Blog;
```

在这个示例中，`Post`和`Comments`都使用`withAsync`的高阶组件模式，分别返回更新后的`BasicPost`和`BasicComments`版本，现在响应`loading`和`error`属性。这种横切关注点的行为在`withAsync`的实现中进行了集中管理，因此我们在这里使用 HOC 模式时可以“免费”处理加载和错误状态。

然而，与展示性组件和容器组件类似，由于钩子提供了额外的便利性，HOCs 经常被抛弃。

### 组合 HOCs

将多个 HOC 组合在一起是 React 中的一种常见模式，它允许开发人员跨组件混合和匹配功能和行为。以下是一个示例，展示了如何组合多个 HOC：

假设你有两个 HOC，`withLogging` 和 `withUser`：

```
// withLogging.js
const withLogging = (WrappedComponent) => {
  return (props) => {
    console.log("Rendered with props:", props);
    return <WrappedComponent {...props} />;
  };
};

// withUser.js
const withUser = (WrappedComponent) => {
  const user = { name: "John Doe" }; // Assume this comes from some data source
  return (props) => <WrappedComponent {...props} user={user} />;
};
```

现在，假设你想要将这两个 HOC 组合在一起。一种方法是嵌套它们：

```
const EnhancedComponent = withLogging(withUser(MyComponent));
```

然而，嵌套的高阶组件（HOC）调用可能难以阅读和维护，特别是随着 HOC 数量的增加。想象一下这在你的应用程序中随着时间的推移会是怎样的情况：

```
const EnhancedComponent = withErrorHandler(
  withLoadingSpinner(
    withAuthentication(
      withAuthorization(
        withPagination(
          withDataFetching(
            withLogging(withUser(withTheme(withIntl(withRouting(MyComponent)))))
          )
        )
      )
    )
  )
);
```

哎呀！更好的方法是创建一个实用函数，将多个 HOC 组合成一个单一的 HOC。这样的实用函数可能看起来像这样：

```
// compose.js
const compose =
  (...hocs) =>
  (WrappedComponent) =>
    hocs.reduceRight((acc, hoc) => hoc(acc), WrappedComponent);

// Usage:
const EnhancedComponent = compose(withLogging, withUser)(MyComponent);
```

在这个`compose`函数中，使用`reduceRight`从右到左应用每个 HOC 到`WrappedComponent`上。这样一来，你可以将你的 HOC 列在一个平面列表中，这样更容易阅读和维护。`compose`函数是函数式编程中常见的实用工具，像 Redux 这样的库也提供了它们自己的`compose`实用函数用于此目的。

要重访我们之前的丑陋示例，使用新的`compose`实用程序后，它看起来会更像这样：

```
const EnhancedComponent = compose(
  withErrorHandler,
  withLoadingSpinner,
  withAuthentication,
  withAuthorization,
  withPagination,
  withDataFetching,
  withLogging,
  withUser,
  withTheme,
  withIntl,
  withRouting
)(MyComponent);
```

是不是更好了？缩减了缩进，增强了可读性，更易于维护。链中的每个 HOC 都包装了前一个 HOC 生成的组件，并为其添加了自己的行为。这样一来，你可以从简单组件和 HOC 构建复杂组件，每个组件都专注于一个单一关注点。这使得你的代码更模块化、更易于理解和测试。

### HOC 与 hooks

自从引入 hooks 以来，HOC 已经变得不那么流行了。Hooks 提供了一种更方便的方法来向组件添加功能，并解决了一些 HOC 存在的问题。例如，HOC 可能会在错误使用时导致 ref 转发问题和不必要的重新渲染。Table 5-1 展示了两者之间的详细比较。

表格 5-1\. HOC 与 hooks 的比较

| 功能 | HOCs | Hooks |
| --- | --- | --- |
| **代码重用** | 用于在多个组件之间共享逻辑非常出色。 | 用于从组件内提取和共享逻辑，或在相似组件之间共享逻辑理想。 |
| **渲染逻辑** | 可以控制包装组件的渲染。 | 不会直接影响渲染，但可以在函数组件中使用以管理与渲染相关的副作用。 |
| **属性操作** | 可以注入和操作属性，提供额外的数据或函数。 | 不能直接注入或操作属性。 |
| **状态管理** | 可以在包装组件之外管理和操作状态。 | 设计用于在函数组件内管理局部状态。 |
| **生命周期方法** | 可以封装与包装组件相关的生命周期逻辑。 | `useEffect` 和其他 hooks 可以处理函数组件内的生命周期事件。 |
| **组合的便利性** | 可以一起组合，但如果管理不当，可能导致“包装地狱”。 | 易于组合，可以与其他钩子同时使用，而不增加组件的层次。 |
| **测试的便利性** | 由于需要额外的包装组件，测试可能会更复杂。 | 通常比高阶组件（HOCs）更容易测试，因为它们可以更容易地被隔离。 |
| **类型安全性** | 在 TypeScript 中，正确地进行类型定义可能会比较棘手，特别是在深度嵌套的高阶组件（HOCs）中。 | 更好的类型推断和在 TypeScript 中更容易进行类型定义。 |

表格 5-1 提供了高阶组件（HOCs）和钩子（hooks）的并排比较，展示了它们各自的优势和使用案例。虽然高阶组件（HOCs）仍然是一个有用的模式，但由于其简单性和易用性，钩子（hooks）通常在大多数使用案例中更受青睐。

从这个表格中，我们可以看出，在 React 中，高阶组件（HOCs）和钩子（hooks）对于在组件之间共享逻辑非常关键，然而它们针对的使用案例略有不同。高阶组件（HOCs）在跨多个组件共享逻辑方面表现出色，特别擅长控制包装组件的渲染和操作属性，提供额外的数据或函数给组件。它们可以管理包装组件之外的状态，并封装与包装组件相关的生命周期逻辑。然而，如果管理不当，特别是当许多高阶组件（HOCs）嵌套在一起时，它们可能导致“包装地狱”。这种嵌套也可能使测试变得更加复杂，而在 TypeScript 中的类型安全性可能会变得棘手，特别是在深度嵌套的高阶组件（HOCs）中。

另一方面，钩子（hooks）非常适合在组件内或类似组件之间提取和共享逻辑，而不会增加额外的组件层次，因此避免了“包装地狱”的场景。与高阶组件（HOCs）不同，钩子（hooks）不会直接影响渲染，并且不能直接注入或操作属性。它们设计用于在函数组件中管理局部状态，并使用 `useEffect` Hook 等处理生命周期事件。钩子（hooks）促进了组合的便利性，并且通常比高阶组件（HOCs）更容易测试，因为它们可以更容易地被隔离。此外，与 TypeScript 结合使用时，钩子（hooks）提供了更好的类型推断和更容易的类型定义，因此可以减少与类型错误相关的 bug。

尽管高阶组件（HOCs）和钩子（hooks）都提供了重用逻辑的机制，但钩子（hooks）在管理状态、生命周期事件和其他 React 特性方面提供了更直接、不那么复杂的方法。另一方面，高阶组件（HOCs）提供了一种更结构化的方式将行为注入组件中，在较大的代码库或尚未采用钩子的代码库中可能非常有益。每种方式都有其自身的优势，选择使用高阶组件（HOCs）还是钩子（hooks）将主要取决于项目的具体需求以及团队对这些模式的熟悉程度。

我们可以考虑一些我们经常使用的 React 高阶组件吗？是的，我们可以！`React.memo`就是我们在本章中刚介绍过的一个高阶组件！让我们再看一个例子：`React.forwardRef`。这是一个将引用转发给子组件的高阶组件。让我们看一个例子：

```
const FancyInput = React.forwardRef((props, ref) => (
  <input type="text" ref={ref} {...props} />
));

const App = () => {
  const inputRef = useRef(null);

  useEffect(() => {
    inputRef.current.focus();
  }, []);

  return (
    <div>
      <FancyInput ref={inputRef} />
    </div>
  );
};
```

在这个例子中，我们使用`React.forwardRef`来将引用转发给`FancyInput`组件。这允许我们在父组件中访问输入元素的`focus`方法。这是 React 中的一个常见模式，也是如何使用高阶组件来解决那些难以用常规组件解决的问题的一个很好的例子。

## 渲染属性

由于我们已经讨论了 JSX 表达式，一个常见的模式是拥有那些作为函数的属性，这些函数接收组件范围的状态作为参数以促进代码重用。这里有一个简单的例子：

```
<WindowSize
  render={({ width, height }) => (
    <div>
      Your window is {width}x{height}px
    </div>
  )}
/>
```

注意到有一个名为`render`的属性，它接收一个函数作为值。这个属性甚至输出一些实际渲染的 JSX 标记。但为什么呢？原来`WindowSize`在内部做了一些魔法来计算用户窗口的大小，然后调用`props.render`来返回我们声明的结构，利用封闭状态来渲染窗口大小。

让我们来看一下`WindowSize`，以更深入地理解这一点：

```
const WindowSize = (props) => {
  const [size, setSize] = useState({ width: -1, height: -1 });

  useEffect(() => {
    const handleResize = () => {
      setSize({ width: window.innerWidth, height: window.innerHeight });
    };
    window.addEventListener("resize", handleResize);
    return () => window.removeEventListener("resize", handleResize);
  }, []);

  return props.render(size);
};
```

从这个例子中，我们可以看到`WindowSize`使用事件侦听器在每次调整大小时将一些内容存储在状态中，但组件本身是无头的：它对要呈现的 UI 没有任何意见。相反，它将控制权委托给渲染它的父组件，并调用提供的*渲染属性*，有效地将控制反转给其父组件来完成渲染工作。

这有助于依赖窗口大小进行渲染的组件在接收此信息时不重复使用`useEffect`块，并使我们的代码更加 DRY（不要重复自己）。这种模式不再那么流行，已经被 React Hooks 有效地取代。

### 作为函数的子组件

由于`children`是一个属性，一些人更喜欢完全放弃`render`属性名称，而是只使用`children`。这将使`WindowSize`的使用看起来像这样：

```
<WindowSize>
  {({ width, height }) => (
    <div>
      Your window is {width}x{height}px
    </div>
  )}
</WindowSize>
```

一些 React 作者更喜欢这样做，因为这更符合代码的意图：在这种情况下，`WindowSize`看起来有点像一个 React 上下文，我们显示的内容似乎就像是消费这个上下文的子元素。不过，React Hooks 消除了对这种模式的需求，所以可能要谨慎使用。

## 控制属性

React 中的**控制属性模式**是一种策略性的状态管理方法，扩展了受控组件的概念。它提供了一种灵活的机制来确定组件内部状态的管理方式。为了理解这一点，让我们首先了解受控组件。

受控组件是不维护自身内部状态的组件。相反，它们从父组件作为 prop 接收其当前值，这是它们状态的唯一真相来源。当状态应该更改时，受控组件使用回调函数（通常是 `onChange`）通知父组件。因此，父组件负责管理状态并更新受控组件的值。

例如，受控 `<input>` 元素如下所示：

```
function Form() {
  const [inputValue, setInputValue] = React.useState("");

  function handleChange(event) {
    setInputValue(event.target.value);
  }

  return <input type="text" value={inputValue} onChange={handleChange} />;
}
```

控制属性模式进一步扩展了受控组件的原则，允许组件可以通过 props 外部控制或者在内部管理自己的状态，提供可选的外部控制。遵循控制属性模式的组件接受状态值和更新该状态的函数作为 props。这种双重能力使得父组件可以选择性地控制子组件的状态，但也允许子组件在未受控时独立操作。

控制属性模式的一个示例是一个切换按钮，可以由其父组件控制或管理其自身状态：

```
function Toggle({ on, onToggle }) {
  const [isOn, setIsOn] = React.useState(false);

  const handleToggle = () => {
    const nextState = on === undefined ? !isOn : on;
    if (on === undefined) {
      setIsOn(nextState);
    }
    if (onToggle) {
      onToggle(nextState);
    }
  };

  return (
    <button onClick={handleToggle}>
      {on !== undefined ? on : isOn ? "On" : "Off"}
    </button>
  );
}
```

在 `Toggle` 组件中，`isOn` 表示内部状态，而 `on` 是外部控制属性。如果父组件提供了 `on` prop，则组件可以以受控模式运行。如果没有，则退回到其内部状态 `isOn`。`onToggle` prop 是一个回调，允许父组件响应状态变化，提供父组件与 `Toggle` 组件状态同步的机会。

此模式增强了组件的灵活性，提供了受控和非受控两种操作模式。它允许父组件在必要时接管控制，同时在未明确控制时让组件保持对其自身状态的自治。

## 属性集合

我们经常需要打包一整套属性在一起。例如，在创建拖放用户界面时，有很多属性需要管理：

`onDragStart`

告知浏览器在用户开始拖动元素时应执行的操作

`onDragOver`

识别一个放置区域

`onDrop`

当元素被拖放到此元素上时执行一些代码

`onDragEnd`

当元素拖动完成时，告知浏览器应执行的操作

此外，默认情况下，数据/元素不能被放置在其他元素中。要允许元素被放置到另一个元素上，我们必须阻止元素的默认处理。这可以通过在可能的放置区域的 `onDragOver` 事件上调用 `event.preventDefault` 方法来实现。

由于这些属性通常一起使用，并且 `onDragOver` 通常默认为 `event => { event.preventDefault(); moreStuff(); }`，我们可以将这些属性集合在一起，并在各种组件中重复使用，如下所示：

```
export const droppableProps = {
  onDragOver: (event) => {
    event.preventDefault();
  },
  onDrop: (event) => {},
};

export const draggableProps = {
  onDragStart: (event) => {},
  onDragEnd: (event) => {},
};
```

现在，如果我们有一个期望行为类似放置区域的 React 组件，我们可以像这样在其上使用属性集合：

```
<Dropzone {...droppableProps} />
```

这是属性集合模式，它使许多属性可重复使用。在可访问组件中广泛使用，包括许多`aria-*`属性。然而，仍然存在一个问题，即如果我们编写一个自定义的`onDragOver`属性并覆盖该集合，我们将失去使用集合时的`event.preventDefault`调用。

这可能会导致意外行为，从而无法将组件放置在`Dropzone`上：

```
<Dropzone
  {...droppableProps}
  onDragOver={() => {
    alert("Dragged!");
  }}
/>
```

幸运的是，我们可以使用属性获取器来解决这个问题。

### 属性获取器

属性获取器本质上是将属性集合与自定义属性合并。从我们的示例中，我们希望在`droppableProps`集合的`onDragOver`处理程序中保留`event.preventDefault`调用，并同时添加自定义的`alert("Dragged!");`调用。我们可以使用属性获取器来实现这一点。

首先，我们将把`droppableProps`集合改为属性获取器：

```
export const getDroppableProps = () => {
  return {
    onDragOver: (event) => {
      event.preventDefault();
    },
    onDrop: (event) => {},
  };
};
```

到目前为止，除了我们之前导出属性集合的位置，我们现在导出一个返回属性集合的函数。这就是属性获取器。由于这是一个函数，它可以接收参数，比如自定义的`onDragOver`。我们可以像这样将自定义的`onDragOver`与默认的组合起来：

```
const compose =
  (...functions) =>
  (...args) =>
    functions.forEach((fn) => fn?.(...args));

export const getDroppableProps = ({
  onDragOver: replacementOnDragOver,
  ...replacementProps
}) => {
  const defaultOnDragOver = (event) => {
    event.preventDefault();
  };

  return {
    onDragOver: compose(replacementOnDragOver, defaultOnDragOver),
    onDrop: (event) => {},
    ...replacementProps,
  };
};
```

现在，我们可以像这样使用属性获取器：

```
<Dropzone
  {...getDroppableProps({
    onDragOver: () => {
      alert("Dragged!");
    },
  })}
/>
```

这个自定义的`onDragOver`将与我们的默认`onDragOver`组合在一起，两件事都会发生：`event.preventDefault()`和`alert("Dragged!")`。这就是属性获取器模式。

## 复合组件

有时，我们会有类似这样的手风琴组件：

```
<Accordion
  items={[
    { label: "One", content: "lorem ipsum for more, see https://one.com" },
    { label: "Two", content: "lorem ipsum for more, see https://two.com" },
    { label: "Three", content: "lorem ipsum for more, see https://three.com" },
  ]}
/>
```

这个组件的目的是渲染类似于此的列表，只是在任何给定时间*只能打开一个*项目：

+   `One`

+   `Two`

+   `Three`

此组件的内部工作将如下所示：

```
export const Accordion = ({ items }) => {
  const [activeItemIndex, setActiveItemIndex] = useState(0);

  return (
    <ul>
      {items.map((item, index) => (
        <li onClick={() => setActiveItemIndex(index)} key={item.id}>
          <strong>{item.label}</strong>
          {index === activeItemIndex && i.content}
        </li>
      ))}
    </ul>
  );
};
```

但如果我们想在项目`Two`和`Three`之间添加一个自定义分隔符怎么办？如果我们想让第三个链接变成红色或其他颜色？我们可能会诉诸某种类型的 hack，比如这样：

```
<Accordion
  items={[
    { label: "One", content: "lorem ipsum for more, see https://one.com" },
    { label: "Two", content: "lorem ipsum for more, see https://two.com" },
    { label: "---" },
    { label: "Three", content: "lorem ipsum for more, see https://three.com" },
  ]}
/>
```

但那看起来不符合我们的期望。所以我们可能会在当前的 hack 基础上做更多的 hack：

```
export const Accordion = ({ items }) => {
  const [activeItemIndex, setActiveItemIndex] = useState(0);

  return (
    <ul>
      {items.map((item, index) =>
        item === "---" ? (
          <hr />
        ) : (
          <li onClick={() => setActiveItemIndex(index)} key={item.id}>
            <strong>{item.label}</strong>
            {index === activeItemIndex && i.content}
          </li>
        )
      )}
    </ul>
  );
};
```

现在这段代码能让我们自豪吗？我不确定。这就是为什么我们需要*复合组件*：它们允许我们组合一组相互连接但具有独立状态的组件，但它们可以被原子化地呈现，从而使我们能够更好地控制元素树。

使用复合组件模式表达的手风琴将如下所示：

```
<Accordion>
  <AccordionItem item={{ label: "One" }} />
  <AccordionItem item={{ label: "Two" }} />
  <AccordionItem item={{ label: "Three" }} />
</Accordion>
```

如果我们决定探索如何在 React 中实现这种模式，我们可能会考虑两种方式：

+   使用`React.cloneElement`处理子组件

+   使用 React 上下文

`React.cloneElement`被视为遗留 API，因此让我们尝试使用 React 上下文来实现这一点。首先，我们将从每个手风琴部分都可以读取的上下文开始：

```
const AccordionContext = createContext({
  activeItemIndex: 0,
  setActiveItemIndex: () => 0,
});
```

然后，我们的`Accordion`组件将仅向其子组件提供上下文：

```
export const Accordion = ({ items }) => {
  const [activeItemIndex, setActiveItemIndex] = useState(0);

  return (
    <AccordionContext.Provider value={{ activeItemIndex, setActiveItemIndex }}>
      <ul>{children}</ul>
    </AccordionContext.Provider>
  );
};
```

现在，让我们创建离散的`AccordionItem`组件，以便作为上下文的消费者并对其做出响应：

```
export const AccordionItem = ({ item, index }) => {
  // Note we're using the context here, not state!
  const { activeItemIndex, setActiveItemIndex } = useContext(AccordionContext);

  return (
    <li onClick={() => setActiveItemIndex(index)} key={item.id}>
      <strong>{item.label}</strong>
      {index === activeItemIndex && i.content}
    </li>
  );
};
```

现在我们有了多个部分组成的 `Accordion`，使其成为一个复合组件，我们对 `Accordion` 的使用从这里开始：

```
<Accordion
  items={[
    { label: "One", content: "lorem ipsum for more, see https://one.com" },
    { label: "Two", content: "lorem ipsum for more, see https://two.com" },
    { label: "Three", content: "lorem ipsum for more, see https://three.com" },
  ]}
/>
```

到这里：

```
<Accordion>
  {items.map((item, index) => (
    <AccordionItem key={item.id} item={item} index={index} />
  ))}
</Accordion>
```

这样做的好处是，我们有了更多的控制权，同时每个 `AccordionItem` 都知道 `Accordion` 的更大状态。因此，如果我们想在 `Two` 和 `Three` 之间包含一条水平线，我们可以在 `map` 中跳出并手动操作：

```
<Accordion>
  <AccordionItem key={items[0].id} item={items[0]} index={0} />
  <AccordionItem key={items[1].id} item={items[1]} index={1} />
  <hr />
  <AccordionItem key={items[2].id} item={items[2]} index={2} />
</Accordion>
```

或者，我们可以做一些更混合的东西，比如：

```
<Accordion>
  {items.slice(0, 2).map((item, index) => (
    <AccordionItem key={item.id} item={item} index={index} />
  ))}
  <hr />
  {items.slice(2).map((item, index) => (
    <AccordionItem key={item.id} item={item} index={index} />
  ))}
</Accordion>
```

这就是复合组件的好处：它们将渲染的控制权反转给父组件，同时在子组件之间保留了上下文状态感知。同样的方法可以用于标签 UI，其中标签知道当前标签状态，同时具有不同层次的元素嵌套。

另一个好处是，这种模式促进了关注点分离，有助于应用程序随着时间的推移显著扩展。

## 状态减少器

React 中的状态减少器模式是由 Kent C. Dodds（*@kentcdodds*）发明和推广的，他是 React 领域中最杰出和熟练的工程师和教育者之一，是该领域真正享誉世界的专家。这种模式提供了一个强大的方式来创建灵活和可定制的组件。让我们用一个现实世界的例子来说明这个概念：一个切换按钮组件。这个例子将演示如何增强基本的切换组件，使消费者能够定制其状态逻辑，在某些业务原因下禁用特定日期的切换按钮。

首先，我们使用 `useReducer` 钩子创建一个基本的切换组件。该组件维护其自己的状态，确定切换是否处于 `On` 或 `Off` 位置。初始状态设置为 `false`，表示处于 `Off` 状态：

```
import React, { useReducer } from "react";

function toggleReducer(state, action) {
  switch (action.type) {
    case "TOGGLE":
      return { on: !state.on };
    default:
      throw new Error(`Unhandled action type: ${action.type}`);
  }
}

function Toggle() {
  const [state, dispatch] = useReducer(toggleReducer, { on: false });

  return (
    <button onClick={() => dispatch({ type: "TOGGLE" })}>
      {state.on ? "On" : "Off"}
    </button>
  );
}
```

要实现状态减少器模式，`Toggle` 组件被修改为接受一个 `stateReducer` 属性。这个属性允许定制或扩展组件的内部状态逻辑。组件的 `internalDispatch` 函数将内部减少器逻辑与 `stateReducer` 属性提供的外部减少器结合起来：

```
function Toggle({ stateReducer }) {
  const [state, dispatch] = useReducer(
    (state, action) => {
      const nextState = toggleReducer(state, action);
      return stateReducer(state, { ...action, changes: nextState });
    },
    { on: false }
  );

  return (
    <button onClick={() => internalDispatch({ type: "TOGGLE" })}>
      {state.on ? "On" : "Off"}
    </button>
  );
}

Toggle.defaultProps = {
  stateReducer: (state, action) => state, // Default reducer does nothing special
};
```

从这段代码片段中，我们可以看到 `stateReducer` 属性用于定制组件的内部状态逻辑。`stateReducer` 函数被调用时传入当前状态和动作对象；然而，我们在动作对象中添加了一个额外的元数据属性：`changes`。这个 `changes` 属性包含了组件的下一个状态，该状态是由内部减少器计算得出的。这允许外部减少器访问组件的下一个状态，并基于此做出决策。

让我们看看`Toggle`组件如何利用基于这种模式的自定义行为。在下面的例子中，`App`组件使用了`Toggle`，但提供了一个自定义的`stateReducer`。这个 reducer 包含逻辑，防止在周三将开关关闭，因为在这个应用程序的位置，周三是一个普遍的“不能关”的日子。这说明了状态 reducer 模式如何允许在不改变组件本身的情况下灵活修改组件行为：

```
function App() {
  const customReducer = (state, action) => {
    // Custom logic: prevent toggle off on Wednesdays
    if (new Date().getDay() === 3 && !changes.on) {
      return state;
    }
    return action.changes;
  };

  return <Toggle stateReducer={customReducer} />;
}
```

通过这个例子，我们看到了状态 reducer 模式在创建高度灵活和可重用组件中的威力。通过允许外部逻辑与组件的内部状态管理集成，我们可以满足各种行为和用例的需求，增强组件的实用性和多功能性。

哎呀！这是一章啊！让我们总结一下我们学到了什么。

# 章节复习

在本章中，我们讨论了 React 的各个方面，包括记忆化、延迟加载、reducers 和状态管理。我们探讨了不同方法在这些主题上的优势和潜在缺点，以及它们如何影响 React 应用程序的性能和可维护性。

我们首先讨论了 React 中的记忆化及其优化组件渲染的好处。我们看了看`React.memo`函数及其如何用于防止组件不必要的重新渲染。我们还检查了一些记忆化可能遇到的问题，例如陈旧状态和需谨慎管理依赖关系的需要。

接下来，我们谈到了 React 中的延迟加载及其如何延迟加载某些组件或资源直到它们真正需要的时候。我们看了看`React.lazy`和`Suspense`组件及其如何在 React 应用程序中实现延迟加载。我们还讨论了延迟加载的权衡，例如增加的复杂性和潜在的性能问题。

接着，我们转向 reducers 及其在 React 中用于状态管理的使用。我们探讨了`useState`和`useReducer`之间的区别，并讨论了使用集中式 reducer 函数来管理状态更新的优势。

在我们的讨论中，我们使用了来自我们自己实现的代码示例来说明我们讨论的概念。我们探讨了这些示例在内部运行的方式以及它们如何影响 React 应用程序的性能和可维护性。

通过使用代码示例和深入的解释，我们深入了解了这些主题及其在实际 React 应用程序中的应用。

# 复习问题

让我们自问一些问题，以测试我们对本章学习的概念的理解：

1.  什么是 React 中的记忆化，它如何用于优化组件渲染？

1.  使用`useReducer`进行 React 状态管理有什么优势，它与`useState`有什么不同？

1.  如何利用 `React.lazy` 和 `Suspense` 组件在 React 应用中实现懒加载？

1.  使用 memoization 在 React 中可能出现的一些潜在问题是什么，以及如何减轻这些问题？

1.  `useCallback` 钩子如何用于在 React 组件中将函数作为 props 进行记忆化？

# 接下来

在下一章中，我们将探讨 React 在服务器端的应用——深入研究服务器端渲染、其优势和权衡、水合、框架等等。到时见！
