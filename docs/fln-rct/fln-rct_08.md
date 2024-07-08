# 第七章：并发 React  

在上一章中，我们深入探讨了使用 React 进行服务器端渲染的世界。我们探讨了服务器端渲染对于提高应用程序性能和用户体验的重要性，尤其是在现代 Web 开发背景下。我们探讨了不同的服务器渲染 API，例如 `renderToString` 和 `renderToPipeableStream`，并讨论了它们的用例和优势。我们还触及了实施服务器端渲染的挑战，以及依赖像 Next.js 和 Remix 这样的成熟框架来处理这些复杂性的重要性。  

我们讨论了水合和其在将服务器渲染的标记与客户端 React 组件连接中的重要性，从而创建无缝的用户体验。此外，我们还讨论了在服务器环境中管理多个客户端连接时可能出现的潜在安全问题和挑战，强调了使用能有效处理这些问题的框架的必要性。  

现在，随着我们迈向下一个并发 React——我们将建立在我们迄今所学的一切的基础上。我们将深入了解 Fiber 协调器，并学习 React 的并发特性，以及它如何高效地管理更新和渲染。通过研究调度、推迟更新和渲染通道，我们将深入了解 React 核心架构所可能实现的性能优化。  

###### 注  

再次强调，Fiber 本身以及我们即将讨论的事物是 React 中的实现细节，可能会发生变化，并且您并不需要深入了解它们来有效使用 React。然而，了解底层机制将帮助您更好地理解 React 的工作原理和有效使用方法，同时也会使您作为工程师更加有见识。  

有了这些，让我们踏上我们进入并发 React 的迷人世界的旅程，继续在我们的专业知识上建立，并发现使用 React 创造高性能应用的新方法。  

# 同步渲染的问题  

总结一下，同步渲染的问题在于它阻塞了主线程，这可能导致用户体验不佳。对于具有许多组件和频繁更新的复杂应用程序尤其如此。在这种情况下，UI 可能变得无响应，导致令人沮丧的用户体验。  

这个问题的典型缓解措施是将一系列更新批处理成一个更新，并尽量减少在主线程上的工作：不是对 10 个事物进行 10 次处理，而是将它们批处理并处理一次。我们在 第四章 中讨论了批处理，所以我们在这里不再详细展开，但是为了我们的讨论目的，理解批处理是解决这些问题的一种缓解措施是很重要的，但即使如此，它也有其局限性，我们将在接下来的几段中揭示。

即使通过批处理，我们所讨论的问题，由于同步渲染的设计特性，进一步加剧了。同步渲染对所有更新都没有优先级的概念。它平等地处理所有更新，无论其可见性如何。例如，使用同步渲染，您可能会阻塞主线程进行用户看不到的项目的渲染工作，比如未显示的选项卡、模态框后面的内容或加载状态中的内容。如果有 CPU 可用，仍然希望这些项目能够渲染，但希望优先渲染用户能够看到和交互的内容。在 React 拥有并发特性之前，我们经常遇到关键更新被较不重要的更新所阻塞，导致用户体验不佳。

通过并发渲染，React 可以根据更新的重要性和紧急性优先处理更新，确保关键更新不被较不重要的更新所阻塞。这使得 React 在高负载下依然能保持响应式的用户界面，从而提升用户体验。例如，当用户悬停或点击按钮时，期望立即显示相应的反馈。如果 React 正在重新渲染一长串项目列表，则悬停或活动状态的反馈会延迟到整个列表渲染完成之后才显示出来。通过并发渲染，CPU 密集型的渲染任务可以暂时让位于更重要的渲染任务，如用户交互和动画效果。

此外，有了并发渲染的能力，React 能够进行时间分片：即将渲染过程分解为较小的片段并逐步处理它们。这使得 React 能够跨多帧执行工作，并且如果需要中断工作，也可以实现。

从现在开始，我们将共同深入探讨所有这些内容。

# 重新审视 Fiber

如 第四章 所述，Fiber 协调器是 React 中实现并发渲染的核心机制。它在 React 16 中引入，代表了与之前的堆栈协调器相比的重大架构转变。Fiber 协调器的主要目标是提高大型和复杂 UI 的响应性能。

Fiber 协调器通过将渲染过程拆分为更小、更可管理的工作单元（称为 Fibers）来实现这一点。这使得 React 能够暂停、恢复和优先处理渲染任务，从而可以根据其重要性推迟或调度更新。这提高了应用的响应能力，并确保关键更新不会被较不重要的任务所阻塞。

# 调度和推迟更新

React 能够调度和推迟更新的能力对于保持应用的响应能力至关重要。Fiber 协调器通过依赖调度器和一些高效的 API 实现了这一功能。这些 API 允许 React 在空闲期间执行工作，并在最合适的时间安排更新。

我们将在接下来的章节更详细地探讨调度器，但现在可以将其视为它听起来的样子：一个接收更新并说“现在做这个”，“稍后做这个”等的系统，使用浏览器的 API，如 `setTimeout`，`MessageChannel` 等。

考虑一个实时聊天应用程序，用户可以发送和接收消息。我们将有一个聊天组件，显示消息列表，以及一个消息输入组件，用户可以输入并提交他们的消息。此外，聊天应用程序实时从服务器接收新消息。在这种情况下，我们希望优先考虑用户交互（输入和提交消息），以保持响应式体验，同时确保新消息能够高效地渲染，而不阻塞用户界面。

为了使这个例子更具体化，让我们创建一些组件。首先是消息列表：

```
const MessageList = ({ messages }) => (
  <ul>
    {messages.map((message, index) => (
      <li key={index}>{message}</li>
    ))}
  </ul>
);
```

接下来，我们有一个消息输入组件，允许用户输入和提交消息：

```
const MessageInput = ({ onSubmit }) => {
  const [message, setMessage] = useState("");

  const handleSubmit = (e) => {
    e.preventDefault();
    onSubmit(message);
    setMessage("");
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        type="text"
        value={message}
        onChange={(e) => setMessage(e.target.value)}
      />
      <button type="submit">Send</button>
    </form>
  );
};
```

最后，我们有一个聊天组件，结合了这两个组件，并处理发送和接收消息的逻辑：

```
const ChatApp = () => {
  const [messages, setMessages] = useState([]);

  useEffect(() => {
    // Connect to the server and subscribe to incoming messages
    const socket = new WebSocket("wss://your-websocket-server.com");
    socket.onmessage = (event) => {
      setMessages((prevMessages) => [...prevMessages, event.data]);
    };

    return () => {
      socket.close();
    };
  }, []);

  const sendMessage = (message) => {
    // Send the message to the server
  };

  return (
    <div>
      <MessageList messages={messages} />
      <MessageInput onSubmit={sendMessage} />
    </div>
  );
};
```

在这个例子中，React 的并发渲染能力发挥作用，通过高效地管理消息列表的更新和用户与消息输入的交互。当用户输入或提交消息时，React 优先处理文本输入更新，以确保流畅的用户体验。

当从服务器接收到新消息并需要渲染时，它们会在默认/未知的渲染通道中渲染，这会同步且立即地阻塞 DOM：这会延迟任何用户输入。如果我们希望渲染新的消息列表的优先级较低，我们可以将相应的状态更新包装在 `useTransition` 钩子的 `startTransition` 函数中，如下所示：

```
const ChatApp = () => {
  const [messages, setMessages] = useState([]);
  const [isPending, startTransition] = useTransition();

  useEffect(() => {
    // Connect to the server and subscribe to incoming messages
    const socket = new WebSocket("wss://your-websocket-server.com");
    socket.onmessage = (event) => {
      startTransition(() => {
        setMessages((prevMessages) => [...prevMessages, event.data]);
      });
    };

    return () => {
      socket.close();
    };
  }, []);

  const sendMessage = (message) => {
    // Send the message to the server
  };

  return (
    <div>
      <MessageList messages={messages} />
      <MessageInput onSubmit={sendMessage} />
    </div>
  );
};
```

通过这种方式，我们向 React 发出信号，以较低的优先级安排消息列表更新，并在不阻塞用户界面的情况下渲染它们，使聊天应用程序能够在高负载下高效运行。因此，用户输入不会被打断，而新消息的渲染优先级低于用户交互，因为它们对用户体验的重要性较小。

这个例子展示了如何利用 React 的并发渲染能力来构建响应式应用程序，处理复杂的交互和频繁的更新，同时不影响性能或用户体验。我们将在本章节后面深入讨论 `useTransition`。现在，让我们稍微深入了解一下，React 如何精确地安排更新。

# 深入探讨

在 React 中，调度、优先级和延迟更新的过程对于维护响应式用户界面至关重要。该过程确保高优先级任务得到及时处理，而低优先级任务可以延迟处理，从而使 UI 在高负载下保持流畅。为了深入探讨这个主题，我们将研究几个核心概念：调度器、任务的优先级以及推迟更新的机制。

###### 注意

在我们继续之前——让我们再次提醒自己，这里涵盖的信息包括实现细节，*并非使用 React 的必要条件*。然而，理解这些概念将帮助您更好地理解 React 的工作原理以及如何有效使用它，同时教授您可以应用于其他工程任务中的基础机制，提升整体技能水平。有了这个想法，让我们继续。

## 调度器

在 React 架构的核心，调度器是一个独立的包，提供与时间相关的实用程序，与 Fiber 协调器无关。React 在协调器内部使用这个调度器。通过使用渲染通道，调度器和协调器使任务能够通过优先级和组织方式进行协作，根据它们的紧迫性。我们将很快深入研究渲染通道。调度器在今天的 React 中的主要作用是管理主线程的让出，主要通过调度微任务来确保平滑执行。

为了更详细地了解这一点，让我们看一下 React 源代码的部分内容：

```
export function ensureRootIsScheduled(root: FiberRoot): void {
  // This function is called whenever a root receives an update. It does two
  // things 1) it ensures the root is in the root schedule, and 2) it ensures
  // there's a pending microtask to process the root schedule.
  //
  // Most of the actual scheduling logic does not happen until
  // `scheduleTaskForRootDuringMicrotask` runs.

  // Add the root to the schedule
  if (root === lastScheduledRoot || root.next !== null) {
    // Fast path. This root is already scheduled.
  } else {
    if (lastScheduledRoot === null) {
      firstScheduledRoot = lastScheduledRoot = root;
    } else {
      lastScheduledRoot.next = root;
      lastScheduledRoot = root;
    }
  }

  // Any time a root received an update, we set this to true until the next time
  // we process the schedule. If it's false, then we can quickly exit flushSync
  // without consulting the schedule.
  mightHavePendingSyncWork = true;

  // At the end of the current event, go through each of the roots and ensure
  // there's a task scheduled for each one at the correct priority.
  if (__DEV__ && ReactCurrentActQueue.current !== null) {
    // We're inside an `act` scope.
    if (!didScheduleMicrotask_act) {
      didScheduleMicrotask_act = true;
      scheduleImmediateTask(processRootScheduleInMicrotask);
    }
  } else {
    if (!didScheduleMicrotask) {
      didScheduleMicrotask = true;
      scheduleImmediateTask(processRootScheduleInMicrotask);
    }
  }

  if (!enableDeferRootSchedulingToMicrotask) {
    // While this flag is disabled, we schedule the render task immediately
    // instead of waiting for a microtask.
    // TODO: We need to land enableDeferRootSchedulingToMicrotask ASAP to
    // unblock additional features we have planned.
    scheduleTaskForRootDuringMicrotask(root, now());
  }

  if (
    __DEV__ &&
    ReactCurrentActQueue.isBatchingLegacy &&
    root.tag === LegacyRoot
  ) {
    // Special `act` case: Record whenever a legacy update is scheduled.
    ReactCurrentActQueue.didScheduleLegacyUpdate = true;
  }
}
```

在 React 代码库中，`ensureRootIsScheduled` 函数在管理渲染过程中扮演着至关重要的角色。当 React 根据 `root: FiberRoot` 接收到更新时，调用此函数执行两个关键操作。请回忆第四章：React 根是在提交阶段发生的最后一次“交换”，以完成更新。

当调用 `ensureRootIsScheduled` 时，它确认将根包含在根调度列表中：跟踪需要处理的根。其次，它确保存在一个专用于处理此根调度的待处理微任务。

微任务是 JavaScript 事件循环管理中的一个概念，表示由微任务队列管理的一种任务类型。要理解微任务，首先需要基本了解 JavaScript 事件循环及其相关的任务队列：

事件循环

JavaScript 引擎使用事件循环来管理异步操作。事件循环不断检查是否有工作（如执行回调）需要完成。它操作两种任务队列：任务队列（宏任务队列）和微任务队列。

任务队列（宏任务队列）

该队列包含处理事件、执行`setTimeout`和`setInterval`回调以及执行 I/O 操作等任务。这些任务逐个处理，只有在当前任务完成后才会继续下一个任务。

微任务队列

微任务是一种更小、更即时的任务。微任务源自诸如 promises、`Object.observe`和`MutationObserver`等操作。它们存储在微任务队列中，这与常规任务队列不同。

执行

微任务在当前任务结束之前处理，然后 JavaScript 引擎从任务队列中获取下一个（宏）任务。执行完任务后，引擎会检查微任务队列中是否有任何微任务，并在继续之前执行它们。这确保微任务快速且有序地处理，就在当前脚本执行之后，但在其他任务（如渲染或处理事件）之前。

特点和用法

微任务在任务队列中具有更高的优先级，这意味着它们在继续执行下一个宏任务之前会被执行。如果一个微任务不断向队列中添加更多微任务，可能会导致任务队列永远不会被处理。这被称为*饥饿*。

在 React 和`ensureRootIsScheduled`函数的背景下，微任务用于确保根调度的处理迅速且具有高优先级，就在当前脚本执行之后，但在浏览器执行其他任务（如渲染或处理事件）之前。这有助于在 React 框架内保持平滑的 UI 更新和高效的任务管理。

该函数首先将根节点添加到调度中。这涉及检查根节点是否是最后一个被调度的节点或已经存在于调度中。如果不存在，函数将根节点添加到调度末尾，并更新`lastScheduledRoot`指向当前根节点。如果之前没有调度过根节点（`lastScheduledRoot === null`），当前根节点将成为调度中的第一个和最后一个节点。

接下来，该函数将标志`mightHavePendingSyncWork`设置为`true`。该标志表示可能有同步工作待处理，这对于`flushSync`函数至关重要，我们将在下一节中介绍。

然后，该函数确保安排一个微任务来处理根节点的调度。这是通过调用`scheduleImmediateTask(processRootScheduleInMicrotask)`来实现的。这种调度既发生在 React 的`act`测试实用程序范围内，也发生在其范围之外，由`__DEV__`和`ReactCurrentActQueue.current`指示。

该函数的另一个重要部分是条件块，检查 `enableDeferRootSchedulingToMicrotask` 标志。如果此标志已禁用，函数将立即安排渲染任务，而不是推迟到微任务。这部分带有一个 `TODO` 注释（截至撰写时），指示未来计划启用此功能，以解锁额外的功能。

最后，函数包括一个处理 React `act` 实用程序中传统更新的条件。这是针对测试场景的特定处理，其中更新以不同方式批处理，并记录每当安排传统更新时。

长话短说，`ensureRootIsScheduled` 是一个复杂的函数，集成了 React 调度和渲染逻辑的几个方面，重点是通过策略性调度任务和微任务，有效管理对 React 根的更新，并确保平滑渲染。

从中我们可以理解 React 中调度器的角色：根据工作落入的渲染 lane 进行工作调度。在接下来的部分中，我们会深入探讨 lane，但现在只需知道 lane 表示更新的优先级即可。

如果我们用代码建模调度器的行为，会像这样：

```
if (nextLane === Sync) {
  queueMicrotask(processNextLane);
} else {
  Scheduler.scheduleCallback(callback, processNextLane);
}
```

从中我们可以看到：

+   如果下一个 lane 是`Sync`，那么调度器会将一个微任务排队，立即处理下一个 lane。理想情况下，我们现在应该理解微任务是什么及其如何适用。

+   如果下一个 lane 不是`Sync`，那么调度器会安排一个回调并处理下一个 lane。

因此，调度器确实如其名：一个根据函数 lane 调度运行函数的系统。好了，我们已经讨论了一段时间的 lane。让我们深入了解并详细理解它们！

# 渲染 Lane

渲染 lane 是 React 调度系统的重要组成部分，确保任务的高效渲染和优先级排序。一个 lane 是一个工作单元，代表一个优先级级别，并可以作为 React 渲染周期的一部分进行处理。渲染 lane 的概念是在 React 18 中引入的，取代了先前使用过期时间的调度机制。让我们深入了解渲染 lane 的细节、工作原理以及其作为位掩码的底层表示。

###### 注意

再次强调，这些是 React 中的实现细节，随时可能会更改。这里的重点是理解这些底层机制，这将有助于我们在日常工程工作中，也将帮助我们理解 React 的工作方式，并使我们能够更有效或更流畅地使用它。最好不要被细节困扰，而是坚持机制及其在实际应用中的潜力。

首先，渲染 lane 是 React 使用的轻量级抽象，用于组织和优先处理渲染过程中需要进行的更新。

例如，当你调用 `setState` 时，该更新被放入一个通道中。我们可以根据更新的上下文理解不同的优先级，如下所示：

+   如果在点击处理程序内部调用 `setState`，它将被放入同步通道（最高优先级），并在微任务中调度。

+   如果在 `startTransition` 过渡内部调用 `setState`，它将放入一个过渡通道（较低优先级）并在微任务中调度。

每个通道对应于特定的优先级级别，高优先级通道在低优先级通道之前处理。在 React 中，一些通道的示例包括：

`SyncHydrationLane`

在水合期间用户点击 React 应用时，点击事件被放入此通道。

`SyncLane`

当用户点击 React 应用时，点击事件被放入此通道。

`InputContinuousHydrationLane`

悬停事件、滚动事件以及水合期间的其他连续事件被放入此通道。

`InputContinuousLane`

与之前相同，但是在 React 应用程序水合后。

`DefaultLane`

从网络上的任何更新、像 `setTimeout` 这样的定时器，以及优先级无法推断的初始渲染都放入此通道。

`TransitionHydrationLane`

`startTransition` 期间的任何过渡都放入此通道。

`TransitionLanes`（1–15）

`startTransition` 后的任何过渡都放入这些通道。

`RetryLanes`（1–4）

任何 Suspense 重试都放入这些通道。

值得注意的是，这些通道表示了撰写时 React 的内部实现，并可能会更改。再次强调，本书的重点是理解 React 工作的*机制*，而不是过于依赖确切的实现细节，因此通道名称可能并不重要。更重要的是我们对机制的理解——即 React 如何使用这个概念，以及我们如何将其应用到工作中。

## 渲染通道的工作原理

当组件更新或新组件添加到渲染树时，React 根据其优先级使用我们之前讨论过的通道为更新分配通道。正如我们所知，优先级由更新的类型（例如，用户交互、数据获取或后台任务）和其他因素（例如组件的可见性）确定。

然后，React 使用渲染通道按以下方式安排和优先处理更新：

1. 收集更新

React 收集自上次渲染以来已安排的所有更新，并根据其优先级将它们分配到各自的通道中。

2\. 流程通道

React 按其各自的通道处理更新，从最高优先级通道开始。同一通道中的更新被批处理在一起，并在单次处理中处理。

3\. 提交阶段

处理所有更新后，React 进入提交阶段，在此阶段应用更改到 DOM，运行效果，并执行其他完成任务。

4\. 重复

每次渲染都会重复这个过程，确保更新始终按优先级顺序处理，并且高优先级的更新不会被低优先级的更新所抢占。

React 根据这些优先级将更新分配到正确的 lane 中，使得应用程序能够在不需要开发者手动干预的情况下高效运行。

当触发更新时，React 执行以下步骤来确定其优先级并将其分配到正确的 lane 中：

1. 确定更新的上下文

React 评估触发更新的上下文。这个上下文可以是用户交互，由于状态或 props 变化而导致的内部更新，甚至是由服务器响应导致的更新。上下文在确定更新优先级方面起着关键作用。

2\. 根据上下文估计优先级

根据上下文，React 估计更新的优先级。例如，如果更新是用户输入的结果，则可能具有较高的优先级，而由非关键后台进程触发的更新可能具有较低的优先级。我们已经详细讨论了不同的优先级级别，所以在这里我们不会再详细讨论。

3\. 检查是否有任何优先级覆盖

在某些情况下，开发者可以使用 React 的 `useTransition` 或 `useDeferredValue` hooks 明确设置更新的优先级。如果存在这样的优先级覆盖，React 将考虑提供的优先级而不是估计的优先级。

4\. 将更新分配到正确的 lane 中

一旦确定了优先级，React 将更新分配给相应的 lane。这个过程使用我们刚刚查看的位掩码完成，它允许 React 高效地处理多个 lane，并确保更新被正确地分组和处理。

在整个过程中，React 依赖其内部启发式算法和更新发生的上下文来做出关于它们优先级的知情决策。这种动态分配优先级和 lane 的方法使得 React 能够在不需要开发者手动干预的情况下平衡响应性和性能，确保应用程序高效运行。

让我们详细看看 React 如何精确处理其各自 lane 中的更新。

## 处理 Lanes

一旦更新被分配到它们各自的 lane 中，React 将按照优先级顺序处理它们。在我们的聊天应用示例中，React 将按以下顺序处理更新：

`ImmediatePriority`

处理消息输入的更新，确保保持响应性并快速更新。

`UserBlockingPriority`

处理打字指示器的更新，为用户提供实时反馈。

`NormalPriority`

处理消息列表的更新，以合理的速度显示新消息和更新。

通过按优先级顺序处理更新，React 确保应用程序的最重要部分即使在负载较重的情况下也能保持响应性。

## 提交阶段

在处理各自车道中的所有更新后，React 进入提交阶段，在此阶段将这些更改应用于 DOM，运行效果，并执行其他的完成任务。在我们的聊天应用示例中，这可能包括更新消息输入值，显示或隐藏输入指示器，以及将新消息追加到消息列表中。React 然后继续到下一个渲染周期，重复收集更新、处理车道和提交更改的过程。

然而，这个过程比我们在本书中真正能够欣赏到的要复杂得多：有像*纠缠*这样的概念，它决定何时需要一起处理两条车道，以及像*重基*这样的进一步概念，它决定何时需要将更新重新基于已经处理过的更新之上。例如，在过渡被同步更新中断之前，需要同时运行两者时，重基就非常有用。

还有很多关于刷新效果的话题值得讨论。例如，当存在同步更新时，React 可能会在更新之前/之后刷新效果，以确保状态在同步更新之间的一致排序。

最终，这就是 React 存在的原因，以及 React 作为一个抽象层在幕后为我们处理更新问题、它们的优先级和排序问题，而我们则继续专注于我们的应用程序。

需要注意的是，虽然 React 在估算优先级方面表现良好，但并不总是完美的。作为开发者，有时您可能需要使用我们迄今为止提到的一些 API：`useTransition`和`useDeferredValue`来覆盖默认的优先级分配，以微调应用程序的性能和响应能力。让我们更详细地探讨这些 API。

# useTransition

`useTransition`是一个强大的 React Hook，允许你管理组件中状态更新的优先级，并防止由于高优先级更新而导致 UI 变得不响应。在处理可能对视觉有影响的更新，如加载新数据或在页面之间导航时，特别有用。

它基本上是将你在其返回的`startTransition`函数中包装的任何更新放入转换车道中，该车道的优先级低于之前看到的同步车道，这允许你控制更新的时机，并在其他高优先级更新争夺主线程时保持平滑的用户体验。

`useTransition`是一个钩子，意味着你只能在函数组件内部使用它。它返回一个包含两个元素的数组：

`isPending`

一个布尔值，指示是否正在进行过渡。`useTransition`的一个有趣之处在于当您调用`startTransition`时，它首先在此属性上调度同步`setState({ isPending: false })`，这意味着依赖于`isPending`的更新需要快速，否则就会失去`useTransition`的意义。

`startTransition`

你可以使用一个函数来包装那些应该延迟执行或者优先级较低的更新。

在这里值得一提的是，还有一个名为 `startTransition` 的 API，它不是作为钩子，而是作为一个普通函数可用。第二种启动非紧急转换的方法是使用直接从 React 导入的 `startTransition` 函数。这种方法不会给我们提供 `isPending` 标志的访问，但它适用于代码中无法使用钩子（如 `useTransition` ）但仍希望向 React 信号低优先级更新的场合。

## 简单示例

下面是一个简单的示例，演示了 `useTransition` 的基本用法：

```
import React, { useState, useTransition } from "react";

function App() {
  const [count, setCount] = useState(0);
  const [isPending, startTransition] = useTransition();

  const handleClick = () => {
    doSomethingImportant();
    startTransition(() => {
      setCount(count + 1);
    });
  };

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={handleClick}>Increment</button>
      {isPending && <p>Loading...</p>}
    </div>
  );
}

export default App;
```

在这个例子中，我们使用 `useTransition` 来管理一个状态更新的优先级，该更新会增加一个计数器。通过将 `setCount` 更新包装在 `startTransition` 函数内部，我们告诉 React 可以延迟这个更新，防止 UI 在同时进行其他高优先级更新时变得无响应。

## 高级示例：导航

`useTransition` 在页面导航时也很有用。通过管理与导航相关的更新的优先级，你可以确保用户体验保持流畅和响应，即使处理复杂的页面转换。

考虑这个示例，我们演示如何在单页应用程序（SPA）中使用 `useTransition` 来管理页面转换：

```
import React, { useState, useTransition } from "react";

const PageOne = () => <div>Page One</div>;
const PageTwo = () => <div>Page Two</div>;

function App() {
  const [currentPage, setCurrentPage] = useState("pageOne");
  const [isPending, startTransition] = useTransition();

  const handleNavigation = (page) => {
    startTransition(() => {
      setCurrentPage(page);
    });
  };

  const renderPage = () => {
    switch (currentPage) {
      case "pageOne":
        return <PageOne />;
      case "pageTwo":
        return <PageTwo />;
      default:
        return <div>Unknown page</div>;
    }
  };

  return (
    <div>
      <nav>
        <button onClick={() => handleNavigation("pageOne")}>Page One</button>
        <button onClick={() => handleNavigation("pageTwo")}>Page Two</button>
      </nav>
      {isPending && <p>Loading...</p>}
      {renderPage()}
    </div>
  );
}

export default App;
```

在这个例子中，我们有两个简单的组件，代表 SPA 中的不同页面。我们使用 `useTransition` 来包装更改当前页面的状态更新，确保如果同时发生其他高优先级更新（如用户输入），页面转换会被延迟。

在这个例子中，你可能会想：“等等，点击用户后，页面转换不应该是即时的吗？” 是的，你是对的；然而，如果下一页需要使用 `Suspense` 获取一些数据，那么页面转换可能会延迟。这就是 `useTransition` 派上用场的地方，它允许你管理与导航相关的更新的优先级，确保用户体验保持流畅和响应，即使处理复杂的页面转换。 值得注意的是，如果下一页的数据获取发生在效果内，那么 `startTransition` 不会等待效果内的数据被获取；然而，当你在转换中挂起时，React 会将 `isPending` 状态与数据获取及数据返回时的渲染绑定在一起。

在这种情况下，当页面转换正在进行时，`isPending` 状态将为 `true`，允许我们立即在用户点击按钮时显示加载指示器。一旦转换完成，`isPending` 状态将为 `false`，并且新页面将被渲染。

## 深入了解

在了解了 React 的 Fiber 架构、React 调度器、优先级级别和渲染通道机制的背景知识后，我们现在可以深入探讨`useTransition`钩子的内部工作原理。

`useTransition`钩子通过创建一个转换并为该转换内的更新分配特定的优先级级别来工作。当一个更新被包装在转换中时，React 确保根据分配的优先级级别进行调度和渲染更新。

下面是使用`useTransition`钩子涉及的步骤概述：

1.  在函数组件内部导入并调用`useTransition`钩子。

1.  该钩子返回一个包含两个元素的数组：第一个是`isPending`状态，第二个是`startTransition`函数。

1.  使用`startTransition`函数来包装希望控制时机的任何状态更新或组件渲染。

1.  `isPending`状态指示转换是否仍在进行中或已完成。

1.  React 确保包装在转换中的更新按照适当的优先级级别处理。这通过使用调度器和渲染通道机制来分配和管理更新实现。

通过使用`useTransition`，我们可以有效地控制更新的时机，并保持流畅的用户体验，即使其他优先级更高的更新也在竞争主线程。

# useDeferredValue

`useDeferredValue`是一个 React Hook，允许将某些 UI 更新推迟到稍后，特别是在应用程序处理重负载或计算密集任务的场景中非常有用，从而帮助管理更新优先级并促进更平滑的过渡和更好的用户体验。

在初始渲染期间，返回的延迟值与提供的值相同。在后续更新中，`useDeferredValue`通过在更新至新值之前保持旧值来维持流畅的用户体验，特别是在涉及计算密集操作的情况下。这不涉及使用旧值和新值进行多次重新渲染，而是控制更新到新值。这种机制类似于`stale-while-revalidate`策略，保持过时值以保持 UI 的响应性同时等待新值。

浏览 React 的提交历史，我们可以看到`useDeferredValue`的首次实现大致如下：

```
function useDeferredValue(value) {
  const [newValue, setNewValue] = useState(value); // only stores initial value
  useEffect(() {
    // update the returned value in a transition whenever it changes,
    // "deferring" it
    startTransition(() => {
      setNewValue(value);
    });
 }, [value]);

 return newValue;
}
```

让我们简要讨论一下这段代码的作用。最初，它设置了一个带有传递给它的初始值的状态（`newValue`）。然后，该函数利用`useEffect`钩子来观察这个值的变化。当检测到变化时，会调用`startTransition`函数，这对于推迟更新至关重要。

在`startTransition`中，使用`setNewValue`将状态更新为新值。使用`startTransition`表示给 React 这个更新不是紧急的，允许 React 优先处理其他更关键的更新。这基本上就是`useDeferredValue`今天的工作方式，对我们对其的心智模型应该是有帮助的。

`useDeferredValue`是 React 并发功能的一部分，它通过允许延迟某些状态更新来实现可中断性。

当组件重新渲染具有延迟值时，React 会在一定时间内继续显示旧值，允许高优先级更新在低优先级更新之前处理。这将渲染工作分解为较小的块，可以随时间分布，提高响应性，并确保高优先级更新（如用户交互）不会被低优先级更新延迟，从而提升积极的用户体验。

## 使用`useDeferredValue`的目的

`useDeferredValue`的主要目的是允许您推迟渲染较不重要的更新。当您希望优先处理更重要的更新（例如用户交互）而不是较不重要的更新（例如显示来自服务器的更新数据）时，这将特别有用。

通过使用`useDeferredValue`，您可以提供更流畅的用户体验，并确保您的应用程序即使在负载较重或处理复杂操作时也保持响应。

要使用`useDeferredValue`，您需要从 React 包中导入它，并将一个值作为其参数传递。然后，该钩子将返回该值的延迟版本，可用于您的组件中。

这里是如何在简单应用程序中使用`useDeferredValue`的示例：

```
import React, { memo useState, useDeferredValue } from "react";

function App() {
  const [searchValue, setSearchValue] = useState("");
  const deferredSearchValue = useDeferredValue(searchValue);

  return (
    <div>
      <input
        type="text"
        value={searchValue}
        onChange={(event) => setSearchValue(event.target.value)}
      />
      <SearchResults searchValue={deferredSearchValue} />
    </div>
  );
}

const SearchResults = memo(({ searchValue }) => {
  // Perform the search and render the results
})
```

在这个示例中，我们有一个搜索输入和一个显示结果的`SearchResults`组件。我们使用`useDeferredValue`来推迟搜索结果的渲染，使应用程序能够优先处理用户输入，并在渲染结果列表昂贵时保持响应。让我们稍微详细了解一下：

1.  我们在组件上使用`memo`，以确保它不会不必要地更新，正如我们在之前的章节中讨论的那样。

1.  当更新时，会导致性能问题，因为渲染成本高昂。

1.  当我们给它一个延迟的属性，`deferredSearchValue`，因为该属性本身是在更紧急的渲染工作之后更新的，所以组件也是如此。因此，只有在没有更紧急的工作需要完成时，例如更新文本输入字段时，组件才会重新渲染。

这里可能会有人问，“为什么不只是对`searchValue`进行防抖或节流？”

很好的问题。让我们在这里进行对比：

防抖

包括在更新列表之前等待一段时间，等待用户完成输入，例如延迟一秒钟。

节流

定期更新列表，比如每秒钟不超过一次

虽然这些方法在某些情况下可能非常有效，但`useDeferredValue`作为更为精细的渲染优化解决方案显现出来，因为它可以无缝地适应用户设备的性能能力，而不是一种任意的延迟。

`useDeferredValue`的关键区别在于其动态延迟处理方法。它消除了设置固定延迟时间的需要。在高性能设备（如强大的笔记本电脑）上，重新渲染延迟几乎是不可察觉的，几乎是即时发生的。相反，在较慢的设备上，渲染延迟会相应调整，导致列表响应输入时稍有滞后，与设备速度成比例。

此外，`useDeferredValue`在其中断延迟重新渲染方面具有显著优势。在 React 处理大量列表的情况下，用户输入新字符时，React 可以暂停重新渲染，响应新输入，然后在后台恢复渲染过程。这与防抖和节流形成对比，后者尽管延迟更新，但仍可能导致在渲染过程中阻塞交互体验的不连贯。

尽管如此，防抖和节流在与渲染无直接关系的场景中仍然很有用。例如，它们可以有效地减少网络请求的频率。这些技术也可以与`useDeferredValue`结合使用，形成全面的优化策略。

基于这一切，我们在 React 应用中使用`useDeferredValue`看到了几个优势：

提升响应速度

在这个示例中，当用户在搜索框中输入时，输入字段会立即更新，而结果则被延迟处理。如果用户快速连续输入五个字符，输入字段会立即更新五次，而`searchResults`只有在用户停止输入后才会渲染一次。对于第 1 至第 4 个字符，`SearchResults`的渲染会被新值中断。

声明式优先级处理

`useDeferredValue`提供了一种简单而声明式的方式来管理应用中更新优先级的逻辑。通过将延迟更新的逻辑封装在钩子内部，您可以保持组件代码的清晰性，并专注于应用的核心方面。

更好的资源利用

通过延迟处理较不重要的更新，`useDeferredValue`允许应用更好地利用可用资源。这可以帮助减少性能瓶颈的可能性，提升应用的整体性能。

## 何时使用`useDeferredValue`

`useDeferredValue`在需要优先处理某些更新的情况下非常有用。一些常见的情况包括：

+   搜索或过滤大数据集

+   渲染复杂的可视化或动画

+   后台从服务器更新数据

+   处理可能影响用户交互的计算密集型操作

让我们看一个例子，`useDeferredValue` 特别有用。想象我们有一个大列表的项目，我们想根据用户输入进行过滤。过滤大列表可能计算开销大，因此使用 `useDeferredValue` 可以帮助保持应用程序的响应：

```
import React, { memo, useState, useMemo, useDeferredValue } from "react";

function App() {
  const [filter, setFilter] = useState("");
  const deferredFilter = useDeferredValue(filter);

  const items = useMemo(() => generateLargeListOfItems(), []);
  const filteredItems = useMemo(() => {
    return items.filter((item) => item.includes(deferredFilter));
  }, [items, deferredFilter]);

  return (
    <div>
      <input
        type="text"
        value={filter}
        onChange={(event) => setFilter(event.target.value)}
      />
      <ItemList items={filteredItems} />
    </div>
  );
}

const ItemList = memo(({ items }) => {
  // Render the list of items
});

function generateLargeListOfItems() {
  // Generate a large list of items for the example
}
```

在这个例子中，我们使用 `useDeferredValue` 来推迟渲染过滤后的列表。当用户在过滤输入中输入时，推迟的值更新频率较低，允许应用程序优先处理用户输入并保持响应。

`useMemo` 钩子用于记忆项目和 `filteredItems` 数组，防止不必要的重新渲染和重新计算。这进一步提升了应用程序的性能。

## 不适合使用 `useDeferredValue` 的情况

尽管 `useDeferredValue` 在某些场景下有益，但认识到其中的权衡是很重要的。通过推迟更新，显示给用户的数据可能略有过时。虽然这通常对较不重要的更新是可以接受的，但重要的是要考虑向用户显示过时数据的影响。

在决定是否使用 `useDeferredValue` 时，可以问自己一个好问题：“这个更新是否来自用户输入？”

React 之所以被称为 React，是因为它使我们的 Web 应用程序能够对事物做出反应。任何使用户期望得到反应的东西都不应被推迟。其他所有事情则应该推迟。

尽管 `useDeferredValue` 的使用可以极大地增强应用程序在负载下的响应能力，但不应视为灵丹妙药。永远记住，提高性能的最佳方式是编写高效的代码并避免不必要的计算。

# 并发渲染的问题

尽管并发渲染可以实现高性能和响应用户交互，但也给开发者带来了新的问题需要考虑。主要问题是很难理解更新处理的顺序，这可能导致意外行为和 bug。

其中一种 bug 称为 *撕裂*，其中由于更新被处理的顺序不正确，导致 UI 变得不一致。当组件依赖于在其仍在渲染时更新的某些值时，就会发生这种情况，从而导致应用程序使用不一致的数据进行渲染。让我们深入了解一下这个问题。

## 撕裂

**撕裂** 是一个 bug，在组件依赖某些在应用程序仍在渲染时更新的状态时发生。要理解这一点，让我们对比同步渲染和并发渲染。

在同步世界中，React 会遍历组件树并依次从顶部到底部渲染它们。这确保了应用程序在整个渲染过程中状态一致，因为每个组件都是使用最新的状态进行渲染的。

考虑这个例子：

```
import { useState, useSyncExternalStore, useTransition } from "react";

// External state
let count = 0;
setInterval(() => count++, 1);

export default function App() {
  const [name, setName] = useState("");
  const [isPending, startTransition] = useTransition();

  const updateName = (newVal) => {
    startTransition(() => {
      setName(newVal);
    });
  };

  return (
    <div>
      <input value={name} onChange={(e) => updateName(e.target.value)} />
      {isPending && <div>Loading...</div>}
      <ul>
        <li>
          <ExpensiveComponent />
        </li>
        <li>
          <ExpensiveComponent />
        </li>
        <li>
          <ExpensiveComponent />
        </li>
        <li>
          <ExpensiveComponent />
        </li>
        <li>
          <ExpensiveComponent />
        </li>
      </ul>
    </div>
  );
}

const ExpensiveComponent = () => {
  const now = performance.now();

  while (performance.now() - now < 100) {
    // Do nothing, just wait.
  }

  return <>Expensive count is {count}</>;
};
```

在我们应用的顶部，我们有 `count`：一个我们全局设置并通过 `setInterval` 在 React 渲染周期之外不断更新的变量，以便我们可以模拟一个撕裂错误，即在应用程序渲染时更新它。由于渲染是并发和可中断的，`ExpensiveComponent` 可能会以不同的 `count` 值被渲染，导致向用户显示不一致的数据或撕裂。

我们期望在 `ExpensiveComponent` 内部渲染的 `count` 值不一致，因为 React 在用户输入时“停止”渲染以优先处理更紧急的更新，比如更新文本输入字段，从而在 `ExpensiveComponent` 中留下了 `count` 的旧值，但并非总是如此。

我们的示例渲染了一个文本输入字段和五个 `ExpensiveComponent` 的列表。这些组件故意没有进行记忆化，以说明一个问题，因为它们会导致性能问题，我们需要这些性能问题来识别撕裂以便理解。在实际应用中，你会想要在 `ExpensiveComponent` 中使用 `React.memo` 进行包装。在这里，我们故意避免这样做来演示撕裂——你将希望在你的应用程序中避免这种情况。

`ExpensiveComponent` 需要花费很长时间来渲染，模拟计算密集型操作。`ExpensiveComponent` 还显示了 `count` 变量的当前值，该值每毫秒递增并从外部存储中读取，在本例中是全局命名空间。

如果我们运行这个例子，我们会看到我们渲染的五个 `ExpensiveComponent` 实例，在输入几个按键后，这些 `ExpensiveComponent` 将以不同的 `count` 值进行渲染。

这是因为 `ExpensiveComponent` 被渲染了五次，每次它被渲染时，`count` 的值都不同。由于 React 同时渲染组件，`ExpensiveComponent` 可能会以不同的 `count` 值被渲染，导致向用户显示不一致的数据。

这被称为撕裂，这是一种错误，可能会在应用程序仍在渲染时发生，当组件依赖于某些在更新的状态时。在这种情况下，`ExpensiveComponent` 依赖于 `count` 变量，而该变量在组件仍在渲染时更新，导致应用程序以不一致的数据进行渲染。对于 `ExpensiveComponent` 的五个实例，我们看到以下输出：

+   `昂贵的计数为 568`

+   `昂贵的计数为 568`

+   `昂贵的计数为 569`

+   `昂贵的计数为 569`

+   `昂贵的计数为 570`

这是有道理的，因为早期的组件实例被渲染时，更新的 `count` 值被刷新/提交到 DOM，较低的实例继续被渲染和产生（刷新，更新）新的 `count` 值。

这并不是一个大问题，因为 React 最终会渲染出一致的状态。更大的问题是当你遇到这样的情况时：

```
<UserDetails id={user.id} />
```

使用这段代码，如果用户在渲染之间从全局存储中删除，则会抛出突然的错误，这可能会让用户感到意外。这就是撕裂的问题所在。

要解决这个撕裂问题，React 提供了一个名为`useSyncExternalStore`的钩子。让我们深入了解一下这个钩子。

### useSyncExternalStore

`useSyncExternalStore`是一个 React Hook，允许您将外部状态与应用程序的内部状态同步。在处理可能导致撕裂的计算密集操作时特别有用，如果不正确处理可能会导致撕裂。`useSyncExternalStore`中的“sync”有双重含义。它是“同步”，但也是“同步的”：它在存储更改时强制同步更新。

`useSyncExternalStore`钩子的签名如下：

```
const value = useSyncExternalStore(store.subscribe, store.getSnapshot);
```

`store.subscribe`

一个接收回调函数作为其第一个且唯一参数的函数。在此函数内部，您可以订阅外部存储的更改，并在存储更改时调用回调函数。回调可以被视为调用以提示 React 使用新值重新渲染组件的调用。此函数的预期返回是一个清除函数，用于取消订阅存储。

典型的`subscribe`函数看起来像这样：

```
const store = {
  subscribe(rerender) {
    const newData = getNewData().then(rerender);
    return () => {
      // unsubscribe somehow
    };
  },
};
```

一个简单的用例是订阅浏览器事件，比如`resize`或者`scroll`事件，在这些事件发生时更新组件，就像这样：

```
const store = {
  subscribe(rerenderImmediately) {
    window.addEventListener("resize", rerenderImmediately);
    return () => {
      window.removeEventListener("resize", rerenderImmediately);
    };
  },
};
```

现在，我们的 React 组件将在浏览器窗口大小调整时重新渲染。但是，它如何获取新值？这就是`useSyncExternalStore`的第二个参数发挥作用的地方。

`store.getSnapshot`

一个返回外部存储当前值的函数。每当组件渲染时调用此函数，并使用返回的值来更新组件的内部状态。此函数是同步调用的，因此不应执行任何异步操作或具有任何副作用。此外，此函数确保在渲染时状态在组件的多个实例之间保持一致。

要跟随我们的窗口大小调整示例，这是我们如何获取当前窗口大小的方法：

```
const store = {
  subscribe(immediatelyRerenderSynchronously) {
    window.addEventListener("resize",
      immediatelyRerenderSynchronously);
    return () => {
      window.removeEventListener("resize",
        immediatelyRerenderSynchronously);
    };
  },
  getSnapshot() {
    return {
      width: window.innerWidth,
      height: window.innerHeight,
    };
  },
};
```

具有`{ width, height }`的对象是窗口当前状态的快照，这是`useSyncExternalStore`将返回的内容。然后，我们可以放心地在组件中使用此对象，确保其状态始终在并发渲染之间保持一致。

我们如何能够有这种信心呢？这是因为`immediatelyRerenderSynchronously`函数强制同步重新渲染，并且不允许 React 推迟它。这是解决撕裂问题的关键。

现在，让我们看看如何使用`useSyncExternalStore`来解决我们上一个示例中的撕裂问题。如果我们回想一下，由于撕裂，我们看到了一个`ExpensiveComponent`列表，其渲染的`count`具有不同的值。让我们看看如何使用`useSyncExternalStore`来修复这个问题。

首先，我们不想在订阅商店并且当更新发生时重新渲染 React；相反，我们希望当用户输入导致重新渲染时，有一致的状态。因此我们的`subscribe`函数将是空的，但是为了得到一致的状态，我们将使用`getSnapshot`函数来获取`count`的当前值并返回它：

```
const store = {
  subscribe() {},
  getSnapshot() {
    return count;
  },
};
```

这是我们之前示例中使用`useSyncExternalStore`的效果：

```
import { useState, useSyncExternalStore, useTransition } from "react";

let count = 0;
setInterval(() => count++, 1);

export default function App() {
  const [name, setName] = useState("");
  const [, startTransition] = useTransition();

  const updateName = (newVal) => {
    startTransition(() => {
      setName(newVal);
    });
  };

  return (
    <div>
      <input value={name} onChange={(e) => updateName(e.target.value)} />
      <ul>
        <li>
          <ExpensiveComponent />
        </li>
        <li>
          <ExpensiveComponent />
        </li>
        <li>
          <ExpensiveComponent />
        </li>
        <li>
          <ExpensiveComponent />
        </li>
        <li>
          <ExpensiveComponent />
        </li>
      </ul>
    </div>
  );
}

const ExpensiveComponent = () => {
  // Instead of reading count globally,
  // we'll use the hook to ensure consistent state
  const consistentCount = useSyncExternalStore(
    () => {},
    () => count
  );

  const now = performance.now();
  while (performance.now() - now < 100) {
    // Do nothing
  }

  return <>Expensive count is {consistentCount}</>;
};
```

现在，如果我们运行此示例，我们将看到`ExpensiveComponent`使用相同的`count`值重新渲染，从而防止撕裂的发生。这是因为`useSyncExternalStore`钩子确保在渲染时状态跨组件的多个实例之间保持一致。

我们不使用`subscribe`函数，因为它的目的是告诉 React 何时使用最新状态重新渲染，但在我们的情况下，我们只想要在渲染之间保持状态一致。我们使用`getSnapshot`函数获取`count`的当前值并返回它，确保在渲染时状态在组件的多个实例之间保持一致。

这是我们如何在之前示例中使用`useSyncExternalStore`解决撕裂问题的方式，确保在渲染时状态跨组件的多个实例之间保持一致。

这确保了当文本输入变化并且`ExpensiveComponent`重新渲染时，它将具有与`ExpensiveComponent`的其他实例相同的`count`值，从而防止撕裂。但是，如果我们想要在`ExpensiveComponent`内部以与我们在`ExpensiveComponent`外部更新`count`的相同间隔更新`count`，那该怎么办呢？

我们只需为此创建一个遵循相同更新规则的存储：

```
import { useState, useSyncExternalStore, useTransition } from "react";

let count = 0;
setInterval(() => count++, 1);

const store = {
  subscribe(forceSyncRerender) {
    // Whenever count changes,
    forceSyncRerender();
  },
  getSnapshot() {
    return count;
  },
};

export default function App() {
  const [name, setName] = useState("");
  const [, startTransition] = useTransition();

  const updateName = (newVal) => {
    startTransition(() => {
      setName(newVal);
    });
  };

  return (
    <div>
      <input value={name} onChange={(e) => updateName(e.target.value)} />
      <ul>
        <li>
          <ExpensiveComponent />
        </li>
        <li>
          <ExpensiveComponent />
        </li>
        <li>
          <ExpensiveComponent />
        </li>
        <li>
          <ExpensiveComponent />
        </li>
        <li>
          <ExpensiveComponent />
        </li>
      </ul>
    </div>
  );
}

const ExpensiveComponent = () => {
  // Instead of reading count globally,
  // we'll use the hook to ensure consistent state
  const consistentCount = useSyncExternalStore(
    store.subscribe,
    store.getSnapshot
  );

  const now = performance.now();
  while (performance.now() - now < 100) {
    // Do nothing
  }

  return <>Expensive count is {consistentCount}</>;
};
```

现在，每当`count`变化时，`ExpensiveComponent`将会重新渲染，并且我们会看到所有`ExpensiveComponent`实例中的`count`都显示相同的新值。变更检测逻辑本身可以简单也可以复杂，但关键是我们理解了`useSyncExternalStore`如何保证其主要功能的机制，即：

+   确保并发渲染时状态保持一致

+   当存储更改时强制同步重新渲染

现在我们理解了`useSyncExternalStore`的工作原理和如何解决撕裂问题，不仅对 React 中的并发渲染有了牢固的掌握，还知道如何解决一些相关问题。这对于作为 React 开发者而言是一个很好的起点。

这是一次相当深入的探索，但我们快要完成了。让我们来回顾一下。

# 章节回顾

这场全面的对话聚焦于并发 React 的深度探索，涉及多个方面，包括 Fiber 调和器、调度、推迟更新、渲染通道以及新的钩子，如`useTransition`和`useDeferredValue`。

我们首先讨论了 Fiber 协调器，这是 React 并发渲染引擎的核心。它是框架能够将工作分解为更小块并管理执行优先级的算法，使得 React 可以“中断”并支持并发渲染。这显著促进了 React 处理复杂、高性能应用程序的能力，确保用户交互在重计算过程中仍能保持响应。

然后我们转向了调度和推迟更新的概念，这本质上允许 React 优先处理某些状态更新而不是其他。React 可以推迟低优先级更新以支持高优先级更新，从而在重负载下保持流畅的用户体验。一个例子展示了聊天应用程序中如何智能调度和渲染传入消息更新，而不阻塞用户界面。

讨论随后转向了渲染通道，这是 React 并发特性的核心概念。渲染通道是 React 使用的一种机制，用于为更新分配优先级并有效管理它们的执行。它是 React 如何决定哪些更新是紧急且需要立即处理，哪些可以推迟到稍后的秘密。详细解释提到这些渲染通道如何使用位掩码来高效处理多个优先级。

然后，我们深入介绍了 React 中为并发操作引入的新钩子 `useTransition` 和 `useDeferredValue`。这些钩子旨在处理过渡并提供更流畅的用户体验，特别是对于需要较长时间操作的情况。

首先讨论了 `useTransition` 钩子，它允许 React 在确保响应用户界面的同时过渡到不同状态，即使新状态需要一段时间准备。换句话说，它允许将更新延迟到下一个渲染周期，如果组件当前正在渲染。

我们还讨论了 `useDeferredValue` 钩子，它推迟了组件较不重要部分的更新，从而避免了用户体验不佳。它基本上允许 React “保持”先前的值更长时间，如果新值花费太多时间。

最后，我们深入探讨了并发中的问题，包括 tearing，并探讨了 `useSyncExternalStore` 如何帮助保持跨多个并发渲染一致的状态。

在整个对话中，重复出现的主题是理解 React 处理复杂、动态应用程序和重计算时的策略的“什么”和“为什么”，以及开发者如何利用这些策略来提供流畅、响应迅速的用户体验。

# 复习问题

让我们问自己几个问题，以测试我们对本章概念的理解：

1.  Fiber 协调器在 React 中的作用是什么，它如何有助于处理复杂的高性能应用程序？

1.  解释 React 中调度和推迟更新的概念。它如何帮助在高负载下保持流畅的用户体验？

1.  什么是 React 中的渲染通道，它们如何管理更新的执行？您能描述渲染通道如何使用位掩码处理多个优先级吗？

1.  `useTransition`和`useDeferredValue`钩子在 React 中的目的是什么？描述每个钩子有益的情境。

1.  何时使用`useDeferredValue`可能不合适？使用这些钩子涉及哪些权衡？

# 接下来

现在您深刻理解了 React 的并发特性及其内部工作原理，可以充分利用它在构建高性能应用程序方面的潜力。在第八章，我们将探讨建立在 React 之上的各种流行框架，如 Next.js 和 Remix，它们通过提供最佳实践、约定和额外功能进一步简化开发过程。

这些框架旨在帮助您轻松构建复杂的应用程序，处理许多常见问题，如服务器渲染、路由和代码拆分。通过利用这些框架的力量，您可以专注于构建应用程序的功能和功能，同时确保优化性能和用户体验。

敬请期待对这些强大框架的深入探讨，了解如何利用 React 及其生态系统构建可扩展、高性能和功能丰富的应用程序。
