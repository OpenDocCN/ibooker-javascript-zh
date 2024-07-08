# 第七章：用 Hooks 增强组件

渲染是 React 应用程序的核心。当某些东西改变（props、state），组件树重新渲染，反映最新的数据作为用户界面。到目前为止，`useState` 已经是我们描述组件应如何渲染的工具。但我们可以做得更多。有更多的 Hooks 定义了渲染何时以及为什么应该发生。还有更多的 Hooks 来增强渲染性能。总有更多的 Hooks 来帮助我们。

在上一章中，我们介绍了 `useState`、`useRef` 和 `useContext`，并看到我们可以将这些 Hooks 组合成我们自己的自定义 Hooks：`useInput` 和 `useColors`。然而，这还不是全部。React 还提供了更多的 Hooks。在本章中，我们将更详细地看看 `useEffect`、`useLayoutEffect` 和 `useReducer`。在构建应用程序时，所有这些都是至关重要的。我们还将研究 `useCallback` 和 `useMemo`，它们可以帮助优化我们的组件性能。

# 介绍 useEffect

现在我们对渲染组件发生了什么有了很好的理解。组件只是渲染用户界面的函数。当应用程序首次加载以及 props 和 state 值改变时会发生渲染。但当我们需要在渲染后执行某些操作时会发生什么？让我们仔细看一下。

考虑一个简单的组件，`Checkbox`。我们使用 `useState` 来设置 `checked` 值和一个函数来改变 `checked` 值：`setChecked`。用户可以勾选和取消勾选框，但我们如何通知用户框已被勾选？让我们尝试使用 `alert`，因为它是阻塞线程的绝佳方法：

```
import React, { useState } from "react";

function Checkbox() {
  const [checked, setChecked] = useState(false);

  alert(`checked: ${checked.toString()}`);

  return (
    <>
      <input
        type="checkbox"
        value={checked}
        onChange={() => setChecked(checked => !checked)}
      />
      {checked ? "checked" : "not checked"}
    </>
  );
};
```

我们在渲染前添加了 `alert` 来阻塞渲染。在用户点击警告框的确定按钮之前，组件不会渲染。因为警告是阻塞的，所以我们直到点击确定后才看到复选框的下一个状态被渲染。

这不是目标，所以也许我们应该在返回之后放置警报？

```
function Checkbox {
  const [checked, setChecked] = useState(false);

  return (
    <>
      <input
        type="checkbox"
        value={checked}
        onChange={() => setChecked(checked => !checked)}
      />
      {checked ? "checked" : "not checked"}
    </>
  );

  alert(`checked: ${checked.toString()}`);
};
```

刮掉。我们不能在渲染后调用 `alert`，因为代码永远不会被执行到。为了确保我们按预期看到 `alert`，我们可以使用 `useEffect`。将 `alert` 放在 `useEffect` 函数内意味着该函数将在渲染后作为副作用被调用：

```
function Checkbox {
  const [checked, setChecked] = useState(false);

  useEffect(() => {
    alert(`checked: ${checked.toString()}`);
  });

  return (
    <>
      <input
        type="checkbox"
        value={checked}
        onChange={() => setChecked(checked => !checked)}
      />
      {checked ? "checked" : "not checked"}
    </>
  );
};
```

当渲染需要引起副作用时，我们使用 `useEffect`。将副作用视为函数返回之外的事物。函数是 `Checkbox`。`Checkbox` 函数渲染 UI。但我们可能希望组件做更多事情。除了返回 UI 外，我们希望组件执行的这些事情称为*效果*。

`alert`、`console.log` 或与浏览器或本地 API 的交互不是渲染的一部分。它不是返回的一部分。在 React 应用中，渲染确实影响其中一个事件的结果。我们可以使用 `useEffect` 等待渲染，然后将值提供给 `alert` 或 `console.log`：

```
useEffect(() => {
  console.log(checked ? "Yes, checked" : "No, not checked");
});
```

类似地，在渲染时，我们可以检查 `checked` 的值，然后将其设置为 `localStorage` 中的值：

```
useEffect(() => {
  localStorage.setItem("checkbox-value", checked);
});
```

我们还可以使用 `useEffect` 来集中焦点于已添加到 DOM 的特定文本输入。React 将渲染输出，然后调用 `useEffect` 来聚焦元素：

```
useEffect(() => {
  txtInputRef.current.focus();
});
```

在渲染时，`txtInputRef` 将具有一个值。我们可以在效果中访问该值以应用焦点。每次我们渲染时，`useEffect` 都可以访问来自该渲染的最新值：属性、状态、引用等。

将 `useEffect` 视为在渲染后发生的函数。当渲染触发时，我们可以在组件内访问当前状态值，并使用它们来执行其他操作。然后，一旦我们再次渲染，整个过程重新开始。新值、新渲染、新效果。

## 依赖数组

`useEffect` 被设计用于与其他有状态 Hooks（如 `useState` 和前面未提及的 `useReducer`）协同工作，我们承诺在本章后面讨论。React 在状态改变时重新渲染组件树。正如我们所学，`useEffect` 将在这些渲染后被调用。

考虑以下情况，`App` 组件具有两个单独的状态值：

```
import React, { useState, useEffect } from "react";
import "./App.css";

function App() {
  const [val, set] = useState("");
  const [phrase, setPhrase] = useState("example phrase");

  const createPhrase = () => {
    setPhrase(val);
    set("");
  };

  useEffect(() => {
    console.log(`typing "${val}"`);
  });

  useEffect(() => {
    console.log(`saved phrase: "${phrase}"`);
  });

  return (
    <>
      <label>Favorite phrase:</label>
      <input
        value={val}
        placeholder={phrase}
        onChange={e => set(e.target.value)}
      />
      <button onClick={createPhrase}>send</button>
    </>
  );
}
```

`val` 是表示输入字段值的状态变量。每当输入字段值变化时，`val` 也会变化。这会导致每次用户输入新字符时组件重新渲染。当用户点击发送按钮时，文本区域的 `val` 保存为 `phrase`，并且 `val` 被重置为 `""`，这会清空文本字段。

这个方法按预期工作，但 `useEffect` 钩子被调用的次数多于应有的次数。每次渲染后，都会调用两次 `useEffect` 钩子：

```
typing ""                             // First Render
saved phrase: "example phrase"        // First Render
typing "S"                            // Second Render
saved phrase: "example phrase"        // Second Render
typing "Sh"                           // Third Render
saved phrase: "example phrase"        // Third Render
typing "Shr"                          // Fourth Render
saved phrase: "example phrase"        // Fourth Render
typing "Shre"                         // Fifth Render
saved phrase: "example phrase"        // Fifth Render
typing "Shred"                        // Sixth Render
saved phrase: "example phrase"        // Sixth Render
```

我们不希望每次渲染时都调用每个效果。我们需要将 `useEffect` 钩子与特定数据变化关联起来。为了解决这个问题，我们可以加入依赖数组。依赖数组可以用来控制何时调用效果：

```
useEffect(() => {
  console.log(`typing "${val}"`);
}, [val]);

useEffect(() => {
  console.log(`saved phrase: "${phrase}"`);
}, [phrase]);
```

我们已经向这两个效果添加了依赖数组以控制它们何时被调用。第一个效果仅在 `val` 值更改时被调用。第二个效果仅在 `phrase` 值更改时被调用。现在，当我们运行应用程序并查看控制台时，我们将看到更有效的更新发生：

```
typing ""                              // First Render
saved phrase: "example phrase"         // First Render
typing "S"                             // Second Render
typing "Sh"                            // Third Render
typing "Shr"                           // Fourth Render
typing "Shre"                          // Fifth Render
typing "Shred"                         // Sixth Render
typing ""                              // Seventh Render
saved phrase: "Shred"                  // Seventh Render
```

通过输入更改 `val` 值仅导致第一个效果触发。当我们点击按钮时，`phrase` 被保存，而 `val` 被重置为 `""`。

毕竟它是一个数组，因此可以检查依赖数组中的多个值。假设我们想在 `val` 或 `phrase` 发生变化时运行特定效果：

```
useEffect(() => {
  console.log("either val or phrase has changed");
}, [val, phrase]);
```

如果这些值中的任何一个发生变化，效果将再次被调用。也可以将空数组作为 `useEffect` 函数的第二个参数。空依赖数组只会在初始渲染后调用一次效果：

```
useEffect(() => {
  console.log("only once after initial render");
}, []);
```

因为数组中没有依赖项，所以该效果在初始渲染时被调用。没有依赖项意味着没有变化，因此该效果永远不会再次被调用。仅在首次渲染时被调用的效果对于初始化非常有用：

```
useEffect(() => {
  welcomeChime.play();
}, []);
```

如果你从效果中返回一个函数，该函数将在组件从树中移除时被调用：

```
useEffect(() => {
  welcomeChime.play();
  return () => goodbyeChime.play();
}, []);
```

这意味着您可以使用 `useEffect` 进行设置和拆卸。空数组意味着欢迎提示音将在首次渲染时仅播放一次。然后，我们将返回一个函数作为清理函数，在组件从树中移除时播放告别提示音。

在许多情况下，这种模式非常有用。也许我们会在首次渲染时订阅新闻源。然后，我们将使用清理函数取消订阅新闻源。更具体地说，我们将首先创建一个名为 `posts` 的状态值和一个名为 `setPosts` 的函数来修改该值。然后，我们将创建一个名为 `addPosts` 的函数，用于接收最新的帖子并将其添加到数组中。然后，我们可以使用 `useEffect` 订阅新闻源并播放提示音。此外，我们可以返回清理函数，用于取消订阅和播放告别提示音：

```
const [posts, setPosts] = useState([]);
const addPost = post => setPosts(allPosts => [post, ...allPosts]);

useEffect(() => {
  newsFeed.subscribe(addPost);
  welcomeChime.play();
  return () => {
    newsFeed.unsubscribe(addPost);
    goodbyeChime.play();
  };
}, []);
```

`useEffect` 中有很多内容。我们可能希望为新闻订阅事件使用一个单独的 `useEffect`，并为提示音事件使用另一个 `useEffect`：

```
useEffect(() => {
  newsFeed.subscribe(addPost);
  return () => newsFeed.unsubscribe(addPost);
}, []);

useEffect(() => {
  welcomeChime.play();
  return () => goodbyeChime.play();
}, []);
```

将功能拆分为多个 `useEffect` 调用通常是一个好主意。但让我们进一步增强它。我们要创建的功能是订阅新闻源并为订阅、取消订阅以及每当有新帖子时播放不同的时髦音效。每个人都喜欢很多大声音乐，对吧？这是一个自定义钩子的情况。也许我们应该称其为 `useJazzyNews`：

```
const useJazzyNews = () => {
  const [posts, setPosts] = useState([]);
  const addPost = post => setPosts(allPosts => [post, ...allPosts]);

  useEffect(() => {
    newsFeed.subscribe(addPost);
    return () => newsFeed.unsubscribe(addPost);
  }, []);

  useEffect(() => {
    welcomeChime.play();
    return () => goodbyeChime.play();
  }, []);

  return posts;
};
```

我们的自定义钩子包含处理时髦新闻源的所有功能，这意味着我们可以轻松地与组件共享此功能。在一个名为 `NewsFeed` 的新组件中，我们将使用自定义钩子：

```
function NewsFeed({ url }) {
  const posts = useJazzyNews();

  return (
    <>
      <h1>{posts.length} articles</h1>
      {posts.map(post => (
        <Post key={post.id} {...post} />
      ))}
    </>
  );
}
```

## 深度检查依赖项

到目前为止，我们在数组中添加的依赖项都是字符串。JavaScript 的原始类型如字符串、布尔值、数字等是可比较的。字符串将如预期般相等：

```
if ("gnar" === "gnar") {
  console.log("gnarly!!");
}
```

然而，当我们开始比较对象、数组和函数时，比较就不同了。例如，如果我们比较两个数组：

```
if ([1, 2, 3] !== [1, 2, 3]) {
  console.log("but they are the same");
}
```

这些数组 `[1,2,3]` 和 `[1,2,3]` 不相等，即使它们在长度和条目上看起来相同。这是因为它们是两个不同的类似数组实例。如果我们创建一个变量来保存此数组值，然后进行比较，我们将看到预期的输出：

```
const array = [1, 2, 3];
if (array === array) {
  console.log("because it's the exact same instance");
}
```

在 JavaScript 中，只有当数组、对象和函数完全相同时，它们才相同。那么这如何与 `useEffect` 的依赖数组相关联呢？为了证明这一点，我们需要一个我们可以随意强制重新渲染的组件。让我们构建一个钩子，每当按键时都会导致组件重新渲染：

```
const useAnyKeyToRender = () => {
  const [, forceRender] = useState();

  useEffect(() => {
    window.addEventListener("keydown", forceRender);
    return () => window.removeEventListener("keydown", forceRender);
  }, []);
};
```

至少，我们只需调用状态更改函数即可强制重新渲染。我们不关心状态值。我们只需要状态函数：`forceRender`。（这就是我们使用数组解构添加逗号的原因。记住，来自第二章？）组件首次渲染时，我们将监听 keydown 事件。按键时，我们通过调用 `forceRender` 强制组件重新渲染。如以前所做的那样，我们将返回一个清理函数，停止监听 keydown 事件。通过将此钩子添加到组件中，我们只需按键即可强制重新渲染它。

使用自定义钩子构建后，我们可以在 `App` 组件中（以及任何其他组件！Hooks 很酷。）使用它：

```
function App() {
  useAnyKeyToRender();

  useEffect(() => {
    console.log("fresh render");
  });

  return <h1>Open the console</h1>;
}
```

每次按键时，都会重新渲染 `App` 组件。`useEffect` 通过在每次 `App` 渲染时将 “fresh render” 记录到控制台来演示这一点。让我们调整 `App` 组件中的 `useEffect`，以引用 `word` 值。如果 `word` 改变，我们将重新渲染：

```
const word = "gnar";
useEffect(() => {
  console.log("fresh render");
}, [word]);
```

不要在每个 keydown 事件上调用 `useEffect`，我们只在第一次渲染后和 `word` 值变化时调用它。它不会改变，所以不会发生后续的重新渲染。在依赖数组中添加一个原始值或数字的效果与预期相同。该效果仅调用一次。

如果我们使用单词数组而不是单个单词会发生什么？

```
const words = ["sick", "powder", "day"];
useEffect(() => {
  console.log("fresh render");
}, [words]);
```

变量 `words` 是一个数组。因为每次渲染时都声明了一个新数组，JavaScript 认为 `words` 已经改变，从而触发“fresh render”效果。数组每次都是新实例，这会注册为应触发重新渲染的更新。

在 `App` 的作用域之外声明 `words` 将解决问题：

```
const words = ["sick", "powder", "day"];

function App() {
  useAnyKeyToRender();
  useEffect(() => {
    console.log("fresh render");
  }, [words]);

  return <h1>component</h1>;
}
```

在这种情况下，依赖数组指的是在函数外部声明的 `words` 的一个实例。“fresh render”效果在第一次渲染后不会再次被调用，因为 `words` 与上次渲染的实例相同。对于这个例子，这是一个很好的解决方案，但并不总是可能（或建议）在函数作用域外定义变量。有时传递给依赖数组的值需要函数作用域内的变量。例如，我们可能需要从类似 `children` 的 React 属性创建 `words` 数组：

```
function WordCount({ children = "" }) {
  useAnyKeyToRender();

  const words = children.split(" ");

  useEffect(() => {
    console.log("fresh render");
  }, [words]);

  return (
    <>
      <p>{children}</p>
      <p>
        <strong>{words.length} - words</strong>
      </p>
    </>
  );
}

function App() {
  return <WordCount>You are not going to believe this but...</WordCount>;
}
```

`App` 组件包含一些作为 `WordCount` 组件子节点的单词。`WordCount` 组件将 `children` 作为属性输入。然后我们在组件中将 `words` 设置为调用 `.split` 后的单词数组。我们希望组件仅在 `words` 改变时重新渲染，但是一旦按键，我们就会看到控制台中出现可怕的“fresh render”单词。

让我们用平静的感觉代替恐惧感，因为 React 团队已经为我们提供了一种避免这些额外渲染的方法。他们不会像那样把我们搭在半空中。解决此问题的方法正如你所期望的另一个钩子：`useMemo`。

`useMemo` 调用一个函数来计算一个记忆化的值。在计算机科学中，记忆化是一种用来提高性能的技术。在一个记忆化的函数中，函数调用的结果被保存和缓存。然后，当再次使用相同输入调用函数时，返回缓存的值。在 React 中，`useMemo` 允许我们将缓存的值与其自身进行比较，以查看它是否实际上已经改变。

`useMemo` 的工作原理是，我们传递一个函数给它，用于计算和创建一个记忆化的值。只有当依赖项之一发生变化时，`useMemo` 才会重新计算该值。首先，让我们导入 `useMemo` 钩子：

```
import React, { useEffect, useMemo } from "react";
```

然后，我们将使用该函数来设置 `words`：

```
const words = useMemo(() => {
  const words = children.split(" ");
  return words;
}, []);

useEffect(() => {
  console.log("fresh render");
}, [words]);
```

`useMemo` 调用发送给它的函数，并将 `words` 设置为该函数的返回值。与 `useEffect` 类似，`useMemo` 依赖于依赖项数组：

```
const words = useMemo(() => children.split(" "));
```

当我们不包含依赖项数组与 `useMemo` 时，单词将在每次渲染时计算。依赖项数组控制回调函数应该何时被调用。发送给 `useMemo` 函数的第二个参数是依赖项数组，应包含 `children` 值：

```
function WordCount({ children = "" }) {
  useAnyKeyToRender();

  const words = useMemo(() => children.split(" "), [children]);

  useEffect(() => {
    console.log("fresh render");
  }, [words]);

  return (...);
}
```

`words` 数组依赖于 `children` 属性。如果 `children` 发生变化，我们应该计算一个反映该变化的新值给 `words`。此时，当组件首次渲染并且 `children` 属性发生变化时，`useMemo` 将为 `words` 计算一个新值。

`useMemo` 钩子是在创建 React 应用程序时理解的一个很好的函数。

`useCallback` 可以像 `useMemo` 一样使用，但它是用来记忆化函数而不是值。例如：

```
const fn = () => {
  console.log("hello");
  console.log("world");
};

useEffect(() => {
  console.log("fresh render");
  fn();
}, [fn]);
```

`fn` 是一个函数，它依次记录 “Hello” 和 “World”。它是 `useEffect` 的一个依赖项，但与 `words` 一样，JavaScript 认为 `fn` 在每次渲染时都是不同的。因此，它会在每次渲染时触发效果。这导致了每次按键都会产生一个 “新鲜的渲染”，这并不理想。

首先通过 `useCallback` 封装该函数：

```
const fn = useCallback(() => {
  console.log("hello");
  console.log("world");
}, []);

useEffect(() => {
  console.log("fresh render");
  fn();
}, [fn]);
```

`useCallback` 为 `fn` 的函数值进行了记忆化。与 `useMemo` 和 `useEffect` 一样，它也期望一个依赖项数组作为第二个参数。在这种情况下，因为依赖项数组为空，所以我们仅创建了一次记忆化的回调函数。

现在我们对 `useMemo` 和 `useCallback` 的用途和区别有了理解，让我们改进我们的 `useJazzyNews` 钩子。每当有新的帖子时，我们将调用 `newPostChime.play()`。在这个钩子中，`posts` 是一个数组，所以我们需要使用 `useMemo` 来记忆化该值：

```
const useJazzyNews = () => {
  const [_posts, setPosts] = useState([]);
  const addPost = post => setPosts(allPosts => [post, ...allPosts]);

  const posts = useMemo(() => _posts, [_posts]);

  useEffect(() => {
    newPostChime.play();
  }, [posts]);

  useEffect(() => {
    newsFeed.subscribe(addPost);
    return () => newsFeed.unsubscribe(addPost);
  }, []);

  useEffect(() => {
    welcomeChime.play();
    return () => goodbyeChime.play();
  }, []);

  return posts;
};
```

现在，每当有新帖子时，`useJazzyNews` 钩子会播放一个提示音。我们通过对钩子进行几处更改实现了这一点。首先，`const [posts, setPosts]` 被重命名为 `const [_posts, setPosts]`。每次 `_posts` 改变时，我们将计算一个新值给 `posts`。

接下来，我们添加了一个效果，每次 `post` 数组改变时播放提示音。我们在新闻源上监听新的帖子。当添加新帖子时，这个钩子被重新调用，`_posts` 反映了新帖子。然后，因为 `_posts` 已经改变，`post` 的新值被记忆化。然后，由于这个效果依赖于 `posts`，所以提示音会播放。它只在帖子改变时播放，而帖子列表只在添加新帖子时改变。

在本章后面，我们将讨论 React Profiler，这是一个用于测试 React 组件性能和渲染的浏览器扩展。在那里，我们将更详细地讨论何时使用 `useMemo` 和 `useCallback`。（剧透警告：节俭使用！）

## 何时使用 useLayoutEffect

我们知道渲染总是在 `useEffect` 之前发生。首先发生渲染，然后所有效果按顺序运行，并完全访问来自渲染的所有值。快速查看 React 文档将指出还有另一种类型的效果钩子：`useLayoutEffect`。

useLayoutEffect 在渲染周期中的特定时刻被调用。事件序列如下：

1.  渲染

1.  `useLayoutEffect` 被调用

1.  浏览器绘制：组件元素实际添加到 DOM 的时间

1.  `useEffect` 被调用

可以通过添加一些简单的控制台消息来观察这一点：

```
import React, { useEffect, useLayoutEffect } from "react";

function App() {
  useEffect(() => console.log("useEffect"));
  useLayoutEffect(() => console.log("useLayoutEffect"));
  return <div>ready</div>;
}
```

在 `App` 组件中，`useEffect` 是第一个 Hook，紧随其后的是 `useLayoutEffect`。我们看到 `useLayoutEffect` 在 `useEffect` 之前被调用：

```
useLayoutEffect
useEffect
```

`useLayoutEffect` 在渲染之后但在浏览器绘制更改之前被调用。在大多数情况下，`useEffect` 是合适的工具，但如果你的效果对于浏览器绘制（UI 元素在屏幕上的出现或位置）很重要，你可能需要使用 `useLayoutEffect`。例如，当窗口调整大小时，你可能想要获取元素的宽度和高度：

```
function useWindowSize {
  const [width, setWidth] = useState(0);
  const [height, setHeight] = useState(0);

  const resize = () => {
    setWidth(window.innerWidth);
    setHeight(window.innerHeight);
  };

  useLayoutEffect(() => {
    window.addEventListener("resize", resize);
    resize();
    return () => window.removeEventListener("resize", resize);
  }, []);

  return [width, height];
};
```

窗口的 `width` 和 `height` 是组件可能在浏览器绘制之前需要的信息。`useLayoutEffect` 用于在绘制之前计算窗口的 `width` 和 `height`。另一个需要使用 `useLayoutEffect` 的例子是跟踪鼠标位置：

```
function useMousePosition {
  const [x, setX] = useState(0);
  const [y, setY] = useState(0);

  const setPosition = ({ x, y }) => {
    setX(x);
    setY(y);
  };

  useLayoutEffect(() => {
    window.addEventListener("mousemove", setPosition);
    return () => window.removeEventListener("mousemove", setPosition);
  }, []);

  return [x, y];
};
```

很可能在绘制屏幕时会使用鼠标的 `x` 和 `y` 位置。`useLayoutEffect` 可用于在绘制之前准确计算这些位置。

## 使用 Hooks 的规则

在使用 Hooks 时，有一些指南需要牢记，这可以帮助避免错误和异常行为：

Hooks 只在组件的作用域内运行

Hooks 应该只从 React 组件中调用。它们也可以添加到自定义 Hooks 中，最终添加到组件中。Hooks 不是常规 JavaScript —— 它们是 React 的一种模式，但它们开始被建模并纳入其他库中。

将功能分解成多个 Hooks 是个好主意

在我们之前的例子中，与 Jazzy News 组件相关的一切都被分成一个效果，与声音效果相关的一切被分成另一个效果。这立即使代码更易读，但这样做还有另一个好处。由于 Hooks 按顺序调用，保持它们小是个好主意。一旦调用，React 就会将 Hooks 的值保存在数组中，以便跟踪这些值。考虑以下组件：

```
function Counter() {
  const [count, setCount] = useState(0);
  const [checked, toggle] = useState(false);

  useEffect(() => {
    ...
  }, [checked]);

  useEffect(() => {
    ...
  }, []);

  useEffect(() => {
    ...
  }, [count]);

  return ( ... )
}
```

每次渲染时，Hook 调用的顺序都是相同的：

```
[count, checked, DependencyArray, DependencyArray, DependencyArray]
```

Hooks 应该只在顶层调用

Hooks 应该在 React 函数的顶层使用。它们不能放置在条件语句，循环或嵌套函数中。让我们调整计数器：

```
function Counter() {
  const [count, setCount] = useState(0);

  if (count > 5) {
    const [checked, toggle] = useState(false);
  }

  useEffect(() => {
    ...
  });

  if (count > 5) {
    useEffect(() => {
      ...
    });
  }

  useEffect(() => {
    ...
  });

  return ( ... )
}
```

当我们在`if`语句中使用`useState`时，我们的意思是只有当`count`值大于 5 时才调用该钩子。这会使数组的值混乱。有时数组会是：`[count, checked, DependencyArray, 0, DependencyArray]`。其他时候是：`[count, DependencyArray, 1]`。在该数组中，效果的索引对 React 很重要。这是值保存的方式。

等等，我们是在说在 React 应用程序中我们再也不能使用条件逻辑了吗？当然不是！我们只是要以不同的方式组织这些条件。我们可以在钩子内嵌套`if`语句，循环和其他条件：

```
function Counter() {
  const [count, setCount] = useState(0);
  const [checked, toggle] =
  useState(
  count => (count < 5)
  ? undefined
  : !c,
  (count < 5) ? undefined
  );

  useEffect(() => {
    ...
  });

  useEffect(() => {
    if (count < 5) return;
    ...
  });

  useEffect(() => {
    ...
  });

  return ( ... )
}
```

在这里，`checked`的值是基于`count`大于 5 的条件。当`count`小于 5 时，`checked`的值是`undefined`。将此条件嵌套在钩子内意味着钩子保持在顶层，但结果类似。第二个效果强制执行相同的规则。如果`count`小于 5，则返回语句将阻止效果继续执行。这样可以保持钩子值数组完整：`[countValue, checkedValue, DependencyArray, DependencyArray, DependencyArray]`。

与条件逻辑一样，您需要将异步行为嵌套在钩子内部。`useEffect`以函数作为第一个参数，而不是一个 Promise。因此，您不能将异步函数作为第一个参数使用：`useEffect(async () => {})`。但是，您可以在嵌套函数内创建异步函数，就像这样：

```
useEffect(() => {
  const fn = async () => {
    await SomePromise();
  };
  fn();
});
```

我们创建了一个变量`fn`来处理异步/等待，然后我们调用函数作为返回值。你可以给这个函数起一个名字，或者你可以将异步效果作为匿名函数使用：

```
useEffect(() => {
  (async () => {
    await SomePromise();
  })();
});
```

如果遵循这些规则，可以避免 React Hooks 的一些常见陷阱。如果您正在使用 Create React App，那么其中包含的 ESLint 插件称为 eslint-plugin-react-hooks 将提供警告提示，如果您违反了这些规则。

## 改善使用`useReducer`的代码

考虑 `Checkbox` 组件。这个组件是一个简单状态的完美例子。方框要么被选中，要么未选中。`checked` 是状态值，`setChecked` 是一个用于改变状态的函数。当组件首次渲染时，`checked` 的值将为 `false`：

```
function Checkbox() {
  const [checked, setChecked] = useState(false);

  return (
    <>
      <input
        type="checkbox"
        value={checked}
        onChange={() => setChecked(checked => !checked)}
      />
      {checked ? "checked" : "not checked"}
    </>
  );
}
```

这个方法很有效，但这个函数的一个方面可能会引起警惕：

```
onChange={() => setChecked(checked => !checked)}
```

仔细看看。乍一看感觉还好，但我们在这里是不是在惹麻烦呢？我们发送了一个函数，它接受 `checked` 的当前值并返回相反值 `!checked`。这可能比必要的复杂。开发者可能会轻易地发送错误的信息并破坏整个功能。为什么不提供一个函数作为切换的方式，而不是这样处理它呢？

让我们添加一个名为 `toggle` 的函数，它将做同样的事情：调用 `setChecked` 并返回 `checked` 当前值的相反值：

```
function Checkbox() {
  const [checked, setChecked] = useState(false);

  function toggle() {
    setChecked(checked => !checked);
  }

  return (
    <>
      <input type="checkbox" value={checked} onChange={toggle} />
      {checked ? "checked" : "not checked"}
    </>
  );
}
```

这更好。`onChange` 设置为一个可预测的值：`toggle` 函数。我们知道每次在任何地方使用它时，它会做什么。我们仍然可以进一步进行，以在每次使用 `checkbox` 组件时产生更可预测的结果。记住我们在 `toggle` 函数中发送给 `setChecked` 的函数？

```
setChecked(checked => !checked);
```

现在我们将通过另一个名称来引用这个函数，`checked => !checked`：一个 *reducer*。一个 reducer 函数的最简单定义是它接受当前状态并返回一个新状态。如果 `checked` 是 `false`，它应该返回相反的 `true`。我们可以将这个行为抽象到一个 reducer 函数中，而不是将这个行为硬编码到 `onChange` 事件中，它将始终产生相同的结果。我们将不再在组件中使用 `useState`，而是使用 `useReducer`：

```
function Checkbox() {
  const [checked, toggle] = useReducer(checked => !checked, false);

  return (
    <>
      <input type="checkbox" value={checked} onChange={toggle} />
      {checked ? "checked" : "not checked"}
    </>
  );
}
```

`useReducer` 接受 reducer 函数和初始状态 `false`。然后，我们将 `onChange` 函数设置为 `setChecked`，这将调用 reducer 函数。

我们之前的 reducer，`checked => !checked`，就是一个很好的例子。如果给定相同的输入，应该期望得到相同的输出。这个概念源自 JavaScript 中的 `Array.reduce`。`reduce` 基本上与 reducer 做的事情一样：它接受一个函数（用于将所有值减少为单个值）和一个初始值，并返回一个值。

`Array.reduce` 接受一个 reducer 函数和一个初始值。对于 `numbers` 数组中的每个值，直到返回一个值为止，都会调用 reducer：

```
const numbers = [28, 34, 67, 68];

numbers.reduce((number, nextNumber) => number + nextNumber, 0); // 197
```

发送给 `Array.reduce` 的 reducer 接受两个参数。你也可以向 reducer 函数发送多个参数：

```
function Numbers() {
  const [number, setNumber] = useReducer(
    (number, newNumber) => number + newNumber,
    0
  );

  return <h1 onClick={() => setNumber(30)}>{number}</h1>;
}
```

每次点击 `h1` 时，我们将总数加 30。

## 使用 `useReducer` 处理复杂状态

当状态变得更加复杂时，`useReducer` 可以帮助我们更可预测地处理状态更新。考虑一个包含用户数据的对象：

```
const firstUser = {
  id: "0391-3233-3201",
  firstName: "Bill",
  lastName: "Wilson",
  city: "Missoula",
  state: "Montana",
  email: "bwilson@mtnwilsons.com",
  admin: false
};
```

然后我们有一个名为 `User` 的组件，将 `firstUser` 设置为初始状态，并且组件显示相应的数据：

```
function User() {
  const [user, setUser] = useState(firstUser);

  return (
    <div>
      <h1>
        {user.firstName} {user.lastName} - {user.admin ? "Admin" : "User"}
      </h1>
      <p>Email: {user.email}</p>
      <p>
        Location: {user.city}, {user.state}
      </p>
      <button>Make Admin</button>
    </div>
  );
}
```

在管理状态时常见的错误是覆盖状态：

```
<button
  onClick={() => {
    setUser({ admin: true });
  }}
>
  Make Admin
</button>
```

这样做将覆盖`firstUser`的状态，并用我们传递给`setUser`函数的内容替换：`{admin: true}`。 可以通过从用户当前值中扩展当前值，然后覆盖`admin`值来解决这个问题：

```
<button
  onClick={() => {
    setUser({ ...user, admin: true });
  }}
>
  Make Admin
</button>
```

这将获取初始状态并推入新的键/值：`{admin: true}`。 我们需要在每次`onClick`时重新编写此逻辑，这样容易出错（明天再回到应用程序时可能会忘记这样做）：

```
function User() {
  const [user, setUser] = useReducer(
    (user, newDetails) => ({ ...user, ...newDetails }),
    firstUser
  );
  ...
}
```

然后，我们将新的状态值`newDetails`发送到 reducer，并将其推送到对象中：

```
<button
  onClick={() => {
    setUser({ admin: true });
  }}
>
  Make Admin
</button>
```

当状态具有多个子值或下一个状态取决于上一个状态时，此模式非常有用。 教大家如何传播，他们会在一天内传播。 教大家使用`useReducer`，他们会终身传播。

## 改进组件性能

在 React 应用程序中，组件通常会被渲染很多次。 改进性能包括防止不必要的渲染并减少渲染传播所需的时间。 React 提供了工具来帮助我们防止不必要的渲染：`memo`，`useMemo`和`useCallback`。 我们之前在章节中看过`useMemo`和`useCallback`，但在本节中，我们将更详细地讨论如何使用这些 Hooks 来提升网站性能。

`memo`函数用于创建纯组件。 如在第三章中讨论的那样，我们知道，给定相同的参数，纯函数总是返回相同的结果。 纯组件也是如此。 在 React 中，纯组件是一个在给定相同属性时始终呈现相同输出的组件。

让我们创建一个名为`Cat`的组件：

```
const Cat = ({ name }) => {
  console.log(`rendering ${name}`);
  return <p>{name}</p>;
};
```

`Cat`是一个纯组件。 输出始终是显示名称属性的段落。 如果提供的名称作为属性相同，则输出将相同：

```
function App() {
  const [cats, setCats] = useState(["Biscuit", "Jungle", "Outlaw"]);
  return (
    <>
      {cats.map((name, i) => (
        <Cat key={i} name={name} />
      ))}
      <button onClick={() => setCats([...cats, prompt("Name a cat")])}>
        Add a Cat
      </button>
    </>
  );
}
```

此应用程序使用`Cat`组件。 初始渲染后，控制台显示：

```
rendering Biscuit
rendering Jungle
rendering Outlaw
```

单击“添加猫”按钮后，提示用户添加一只猫。

如果我们添加一个名为“Ripple”的猫，我们会看到所有`Cat`组件都重新渲染：

```
rendering Biscuit
rendering Jungle
rendering Outlaw
rendering Ripple
```

###### 警告

此代码之所以有效，是因为`prompt`是阻塞的。 这只是一个例子。 在真实应用中不要使用`prompt`。

每次添加猫时，每个`Cat`组件都会被重新渲染，但`Cat`组件是一个纯组件。 给定相同的属性，输出不会改变，因此不应为每个属性重新渲染。 `memo`函数可用于创建仅在其属性更改时才会重新渲染的组件。 首先从 React 库导入它，然后将其用于包装当前的`Cat`组件：

```
import React, { useState, memo } from "react";

const Cat = ({ name }) => {
  console.log(`rendering ${name}`);
  return <p>{name}</p>;
};

const PureCat = memo(Cat);
```

在这里，我们创建了一个名为`PureCat`的新组件。 `PureCat`仅在属性更改时才会导致`Cat`重新渲染。 然后，我们可以在`App`组件中用`PureCat`替换`Cat`组件：

```
cats.map((name, i) => <PureCat key={i} name={name} />);
```

现在，每当我们添加一个新的猫名字，比如“Pancake”，我们在控制台中只看到一次渲染：

```
rendering Pancake
```

因为其他猫的名字没有改变，我们不会渲染那些`Cat`组件。这对于`name`属性效果很好，但如果我们向`Cat`组件引入一个函数属性会怎么样？

```
const Cat = memo(({ name, meow = f => f }) => {
  console.log(`rendering ${name}`);
  return <p onClick={() => meow(name)}>{name}</p>;
});
```

每当点击猫时，我们可以使用此属性将`meow`记录到控制台中：

```
<PureCat key={i} name={name} meow={name => console.log(`${name} has meowed`)} />
```

当我们添加了这个更改后，`PureCat`不再按预期工作。它始终渲染每个`Cat`组件，即使`name`属性保持不变也是如此。这是因为增加了`meow`属性。不幸的是，每次我们将`meow`属性定义为一个函数时，它总是一个新函数。对于 React 来说，`meow`属性已经改变，因此组件会重新渲染。

`memo`函数将允许我们在何时重新渲染此组件周围定义更具体的规则：

```
const RenderCatOnce = memo(Cat, () => true);
const AlwaysRenderCat = memo(Cat, () => false);
```

第二个参数传递给`memo`函数的是一个*predicate*。谓词是一个仅返回`true`或`false`的函数。此函数决定是否重新渲染猫。当它返回`false`时，`Cat`会重新渲染。当此函数返回`true`时，`Cat`将不会重新渲染。无论如何，`Cat`至少渲染一次。这就是为什么在`RenderCatOnce`中，它会渲染一次，然后再也不会。通常，此函数用于检查实际值：

```
const PureCat = memo(
  Cat,
  (prevProps, nextProps) => prevProps.name === nextProps.name
);
```

我们可以使用第二个参数来比较属性并决定是否应重新渲染`Cat`。谓词接收先前的属性和下一个属性。这些对象用于比较`name`属性。如果`name`发生变化，组件将重新渲染。如果`name`相同，那么不管 React 如何看待`meow`属性，它都将重新渲染。

## shouldComponentUpdate 和 PureComponent

我们讨论的概念对 React 并不新鲜。`memo`函数是解决一个常见问题的新方法。在 React 的早期版本中，有一种称为`shouldComponentUpdate`的方法。如果存在于组件中，它将告诉 React 在哪些情况下组件应该更新。`shouldComponentUpdate`描述了哪些 props 或 state 需要更改以便重新渲染组件。一旦`shouldComponentUpdate`成为 React 库的一部分，许多人都认为它是一个有用的特性。如此有用，以至于 React 团队决定创建一个创建类组件的替代方法。类组件看起来像这样：

```
class Cat extends React.Component {
  render() {
    return (
      {name} is a good cat!
    )
  }
}
```

一个`PureComponent`看起来像这样：

```
class Cat extends React.PureComponent {
  render() {
    return (
      {name} is a good cat!
    )
  }
}
```

`PureComponent`与`React.memo`相同，但`PureComponent`仅适用于类组件；`React.memo`仅适用于函数组件。

`useCallback`和`useMemo`可用于记忆化对象和函数属性。让我们在`Cat`组件中使用`useCallback`：

```
const PureCat = memo(Cat);
function App() {
  const meow = useCallback(name => console.log(`${name} has meowed`, []);
  return <PureCat name="Biscuit" meow={meow} />
}
```

在这种情况下，我们没有为 `memo(Cat)` 提供属性检查谓词。相反，我们使用 `useCallback` 确保 `meow` 函数未发生更改。在处理组件树中过多重新渲染时，使用这些函数会很有帮助。

## 何时重构

我们讨论的最后两个 Hook，`useMemo` 和 `useCallback`，以及 `memo` 函数，通常被过度使用。React 设计用于快速渲染组件。优化性能的过程从您决定首次使用 React 开始。它很快。进一步的重构应该是最后一步。

重构存在权衡。仅仅因为觉得使用 `useCallback` 和 `useMemo` 是个好主意，实际上可能会降低应用程序的性能。您会增加代码行数和开发人员工时。在为性能重构时，设定一个目标非常重要。也许您想要防止屏幕冻结或闪烁。也许您知道某些昂贵的函数无缘无故地减慢了应用程序的速度。

React Profiler 可用于测量每个组件的性能。该分析器与您可能已经安装的 React 开发者工具一起提供（可在[Chrome](https://oreil.ly/1UNct)和[Firefox](https://oreil.ly/0NYbR)上使用）。

在重构之前，请确保您的应用程序可以正常工作，并且您对代码库感到满意。过度重构或在应用程序工作之前进行重构可能会引入难以发现的怪异 bug，而且也可能不值得花费您的时间和精力来引入这些优化。

在过去的两章中，我们介绍了许多随 React 一起提供的 Hook。您已经看到了每个 Hook 的用例，并通过组合其他 Hook 创建了自己的自定义 Hook。接下来，我们将通过整合额外的库和高级模式，进一步构建这些基础技能。
