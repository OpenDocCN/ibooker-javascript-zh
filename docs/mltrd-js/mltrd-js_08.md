# 第七章：WebAssembly

尽管这本书的标题是 *Multithreaded JavaScript*，现代 JavaScript 运行时也支持 WebAssembly。对于不了解的人来说，WebAssembly（通常缩写为 WASM）是一种二进制编码的指令格式，运行在基于堆栈的虚拟机上。它设计时考虑了安全性，并在仅能访问内存和主机环境提供的函数的沙箱中运行。在浏览器和其他 JavaScript 运行时中运行的程序部分，它比 JavaScript 可以更快地执行。另一个目标是为通常编译的语言（如 C、C++ 和 Rust）提供一个编译目标。这为这些语言的开发者开发 Web 提供了机会。

通常，WebAssembly 模块使用的内存由 `ArrayBuffers` 表示，但也可以由 `SharedArrayBuffers` 表示。此外，还有用于原子操作的 WebAssembly 指令，类似于我们在 JavaScript 中使用的 `Atomics` 对象。通过 `SharedArrayBuffers`、原子操作和 Web Workers（或在 Node.js 中的 `worker_threads`），我们可以完成使用 WebAssembly 进行多线程编程的全套任务。

在我们深入讨论多线程 WebAssembly 之前，让我们构建一个“Hello, World!” 的例子并执行它，以找出 WebAssembly 的优势和限制。

# 你的第一个 WebAssembly

虽然 WebAssembly 是一个二进制格式，但存在一个纯文本格式来以人类可读的形式表示它。这类似于如何将机器码表示为人类可读的汇编语言。这种 WebAssembly 文本格式的语言简称为 WAT，但通常使用的文件扩展名是 *.wat*。它使用 *S 表达式* 作为其主要的语法分隔符，这对于解析和可读性都很有帮助。S 表达式主要来自 Lisp 系列语言，是由括号括起的嵌套列表，列表中的每个项之间有空白。

要感受这种格式，让我们在 WAT 中实现一个简单的加法函数。创建一个名为 *ch7-wasm-add/add.wat* 的文件，并添加 示例 7-1 的内容。

##### 示例 7-1\. *ch7-wasm-add/add.wat*

```
(module ![1](img/1.png)
  (func $add (param $a i32) (param $b i32) (result i32) ![2](img/2.png)
    local.get $a ![3](img/3.png)
    local.get $b
    i32.add)
  (export "add" (func $add)) ![4](img/4.png)
)
```

![1](img/#co_webassembly_CO1-1)

第一行声明了一个模块。每个 WAT 文件都以这个开始。

![2](img/#co_webassembly_CO1-2)

我们声明了一个名为 `$add` 的函数，接受两个 32 位整数并返回另一个 32 位整数。

![3](img/#co_webassembly_CO1-3)

这是函数体的开始部分，在其中我们有三条语句。前两条语句获取函数参数并将它们依次放入堆栈中。请记住，WebAssembly 是基于堆栈的。这意味着许多操作将在堆栈的第一个（如果是一元操作）或前两个（如果是二元操作）项上进行。第三条语句是对 i32 值进行二元“add”操作，因此它从堆栈中获取顶部两个值并将它们相加，将结果放在堆栈顶部。函数的返回值是堆栈在完成时的顶部值。

![4](img/#co_webassembly_CO1-4)

为了在主机环境中使用模块外的函数，它需要被导出。在这里，我们导出了`$add`函数，并为其指定了外部名称`add`。

我们可以使用 WebAssembly 二进制工具包（WABT）中的`wat2wasm`工具将此 WAT 文件转换为 WebAssembly 二进制文件。这可以通过*ch7-wasm-add*目录中的以下一行命令完成。

```
$ npx -p wabt wat2wasm add.wat -o add.wasm
```

现在我们有了我们的第一个 WebAssembly 文件！这些文件在主机环境外并不实用，所以让我们编写一点 JavaScript 来加载 WebAssembly 并测试`add`函数。将示例 7-2 的内容添加到*ch7-wasm-add/add.js*中。

##### 示例 7-2\. *ch7-wasm-add/add.js*

```
const fs = require('fs/promises'); // Needs Node.js v14 or higher.

(async () => {
  const wasm = await fs.readFile('./add.wasm');
  const { instance: { exports: { add } } } = await WebAssembly.instantiate(wasm);
  console.log(add(2, 3));
})();
```

如果您已经使用前面的`wat2wasm`命令创建了*.wasm*文件，那么您应该可以在*ch7-wasm-add*目录中运行它。

```
$ node add.js
```

您可以从输出中验证，我们确实通过我们的 WebAssembly 模块进行了加法操作。

堆栈上的简单数学运算不会使用线性内存或 WebAssembly 中没有意义的概念，比如字符串。考虑 C 语言中的字符串。实际上，它们只不过是指向以空字节结尾的字节数组的指针。我们不能通过值传递整个数组到 WebAssembly 函数或返回它们，但我们可以通过引用传递它们。这意味着要将字符串作为参数传递，我们需要首先在线性内存中分配字节并写入它们，然后将第一个字节的索引传递给 WebAssembly 函数。由于我们需要管理线性内存中的可用空间，这可能会变得更复杂。基本上，我们需要在线性内存上运行的`malloc()`和`free()`实现。^(1)

在 WAT 中手写 WebAssembly 虽然显然可行，但通常不是提高效率和性能的最简单途径。它被设计为更高级语言的编译目标，这也是它真正闪耀的地方。“使用 Emscripten 将 C 程序编译为 WebAssembly”更详细地探讨了这一点。

# WebAssembly 中的原子操作

虽然在这本书中详细介绍每个[WebAssembly 指令](https://oreil.ly/PfxJq)并不合适，但值得指出的是与共享内存上的原子操作相关的指令，因为它们对于多线程的 WebAssembly 代码是关键的，无论是从其他语言编译还是手写 WAT。

WebAssembly 指令通常以类型开头。在原子操作的情况下，类型总是`i32`或`i64`，分别对应 32 位和 64 位整数。所有原子操作在指令名称中紧跟`.atomic.`。之后，你会找到具体的指令名称。

让我们来看一些原子操作指令。我们不会详细介绍语法，但这应该让你对指令级别的操作类型有所了解：

`[i32|i64].atomic.[load|load8_u|load16_u|load32_u]`

`load` 指令系列相当于 JavaScript 中的 `Atomics.load()`。使用其中一个带后缀的指令允许您加载更小的位数，并使用零扩展结果。

`[i32|i64].atomic.[store|store8|store16|store32]`

`store` 指令系列相当于 JavaScript 中的 `Atomics.store()`。使用其中一个带后缀的指令将输入值包装到该位数，并将其存储在索引位置。

`[i32|i64].atomic.[rmw|rmw8|rmw16|rmw32].[add|sub|and|or|xor|xchg|cmpxchg][|_u]`

`rmw` 指令系列都执行读-修改-写操作，分别相当于 JavaScript 中 `Atomics` 对象的 `add()`、`sub()`、`and()`、`or()`、`xor()`、`exchange()` 和 `compareExchange()`。当它们进行零扩展时，操作后缀为 `_u`，并且 `rmw` 可以有与待读取位数相对应的后缀。

下面的两个操作有略微不同的命名约定：

`memory.atomic.[wait32|wait64]`

这些相当于 JavaScript 中的 `Atomics.wait()`，根据它们操作的位数后缀不同。

`memory.atomic.notify`

这相当于 JavaScript 中的 `Atomics.notify()`。

这些指令足以在 WebAssembly 中执行与 JavaScript 中相同的原子操作，但 JavaScript 中没有的附加操作是：

`atomic.fence`

此指令不接受任何参数，也不返回任何内容。它旨在供具有保证非原子访问顺序方式的高级语言使用。

所有这些操作都与给定的 WebAssembly 模块的*线性内存*一起使用，这是一个允许读取和写入值的沙盒。当从 JavaScript 初始化 WebAssembly 模块时，可以选择使用线性内存进行初始化。这可以由`SharedArrayBuffer`支持，以便跨线程使用。

虽然在 WebAssembly 中使用这些指令是完全可能的，但它们遭受与 WebAssembly 其余部分相同的缺点：编写起来非常乏味和费力。幸运的是，我们可以将高级语言编译成 WebAssembly。

# 使用 Emscripten 将 C 程序编译为 WebAssembly

自 WebAssembly 诞生以来，[Emscripten](https://emscripten.org) 一直是将 C 和 C++ 程序编译为 JavaScript 环境可用的首选方式。如今，它支持在浏览器中使用 web workers 和在 Node.js 中使用 `worker_threads` 来实现多线程的 C 和 C++ 代码。

实际上，在野外存在大量现有的多线程代码可以无缝地使用 Emscripten 编译，没有问题。在 Node.js 和浏览器中，Emscripten 模拟了编译为 WebAssembly 的本地代码使用的系统调用，以便以编译语言编写的程序可以在不进行太多更改的情况下运行。

的确，我们在 第一章 中编写的 C 代码可以毫无修改地编译！现在让我们试试。我们将使用 Docker 镜像来简化使用 Emscripten。对于其他编译器工具链，我们需要确保工具链与系统对齐，但由于 WebAssembly 和 JavaScript 都是跨平台的，我们可以在支持 Docker 的任何地方使用 Docker 镜像。

首先，请确保已安装 [Docker](https://docker.com)。然后，在您的 *ch1-c-threads* 目录中，运行以下命令：

```
$ docker run --rm -v $(pwd):/src -u $(id -u):$(id -g) \
  emscripten/emsdk emcc happycoin-threads.c -pthread \
  -s PTHREAD_POOL_SIZE=4 -o happycoin-threads.js
```

关于这个命令有几点需要讨论。我们正在运行 `emscripten/emsdk` 镜像，当前目录已挂载，并以当前用户身份运行。`emcc` 以及后续的内容是我们在容器内运行的命令。在大多数情况下，这看起来很像使用 `cc` 编译 C 程序时会做的事情。主要区别在于输出文件是 JavaScript 文件而不是可执行二进制文件。不用担心！还会生成一个 *.wasm* 文件。JS 文件被用作与必要系统调用的桥梁，并设置线程，因为这些无法仅通过 WebAssembly 实例化。

另一个额外的参数是 `-s PTHREAD_POOL_SIZE=4`。因为 `happycoin-threads.c` 使用了三个线程，我们在此提前分配它们。在 Emscripten 中处理线程创建有几种方法，主要是由于不会阻塞主浏览器线程。在这里预分配是最简单的，因为我们知道需要多少个线程。

现在我们可以运行多线程 Happycoin 的 WebAssembly 版本。我们将使用 Node.js 运行 JavaScript 文件。在撰写本文时，这要求 Node.js 版本为 v16 或更高，因为这是 Emscripten 输出支持的版本。

```
$ node happycoin-threads.js
```

输出结果应该类似于以下内容：

```
120190845798210000 ... [ 106 more entries ] ... 14356375476580480000
count 108
Pthread 0x9017f8 exited.
Pthread 0x701500 exited.
Pthread 0xd01e08 exited.
Pthread 0xb01b10 exited.
```

输出看起来与我们之前章节中的其他 Happycoin 示例相同，但 Emscripten 提供的包装还会告知我们线程何时退出。你还需要按 Ctrl+C 退出程序。为了额外的乐趣，看看你能否找出需要更改的内容，以使进程在完成时退出，并避免那些 `Pthread` 消息。

与 Happycoin 的原生或 JavaScript 版本进行比较时，你可能会注意到的一件事是时间。它显然比多线程 JavaScript 版本快，但比本机多线程 C 版本稍慢。总是很重要的是，通过测量你的应用程序，确保你获得了正确的利益与权衡。

虽然 Happycoin 示例不使用任何原子操作，但 Emscripten 支持完整的 POSIX 线程功能和 GNU 编译器集合（GCC）内置的原子操作函数。这意味着许多 C 和 C++ 程序可以使用 Emscripten 编译为 WebAssembly。

# 其他 WebAssembly 编译器

Emscripten 并不是将代码编译为 WebAssembly 的唯一方式。事实上，WebAssembly 主要设计为编译目标，而不是作为自身的通用语言。有许多工具可以将众所周知的语言编译为 WebAssembly，甚至有些语言是以 WebAssembly 为主要目标构建的，而不是机器码。这里列出了一些，但这并不是 [详尽无遗](https://oreil.ly/wKfBe)。在这里你会注意到许多“在撰写时”，因为这个领域相对较新，创建多线程 WebAssembly 代码的最佳方法仍在开发中！至少，在撰写时是这样的。

Clang/Clang++

LLVM 的 C 家族编译器可以通过 `-target wasm32-unknown-unknown` 或 `-target wasm64-unknown-unknown` 选项目标 WebAssembly。实际上，Emscripten 现在就是基于此的，其中 POSIX 线程和原子操作按预期工作。在撰写时，这是对多线程 WebAssembly 的一些最佳支持。虽然 `clang` 和 `clang++` 支持 WebAssembly 输出，但推荐的方法是使用 Emscripten，在浏览器和 Node.js 中获得完整的平台支持套件。

Rust

Rust 编程语言的编译器 `rustc` 支持生成 WebAssembly 输出。Rust 网站是使用 `rustc` 的 [很好的起点](https://oreil.ly/ibOs3)。要使用线程，可以使用 [`wasm-bindgen-rayon` crate](https://oreil.ly/Pyuv4)，它提供了使用 web workers 实现的并行 API。在撰写时，Rust 标准库的线程支持无法工作。

AssemblyScript

AssemblyScript 编译器以 TypeScript 的子集作为输入，然后生成 WebAssembly 输出。虽然它不支持生成线程，但它支持原子操作和使用 `SharedArrayBuffers`，因此只要你通过 web workers 或 `worker_threads` 在 JavaScript 侧处理线程本身，就可以在 AssemblyScript 中充分利用多线程编程。我们将在下一节详细介绍它。

当然，还有很多其他选项，新的选项也在不断出现。值得在网上看看，你选择的编译语言是否可以针对 WebAssembly，以及它是否支持在 WebAssembly 中的原子操作。

# AssemblyScript

[AssemblyScript](https://assemblyscript.org) 是 [TypeScript](https://typescriptlang.org) 的一个子集，编译成 WebAssembly。与编译现有语言并提供现有系统 API 实现的方法不同，AssemblyScript 设计为一种以比 WAT 更熟悉的语法生成 WebAssembly 代码的方式。AssemblyScript 的一个主要卖点是许多项目已经使用 TypeScript，因此添加一些 AssemblyScript 代码以利用 WebAssembly 不需要进行太多的上下文切换，甚至学习完全不同的编程语言。

一个 AssemblyScript 模块看起来很像一个 TypeScript 模块。如果你不熟悉 TypeScript，可以将其视为普通的 JavaScript，但是在每个函数参数后面加上`: number`来指示类型信息。以下是执行加法的基本 TypeScript 模块：

```
export function add(a: number, b: number): number {
  return a + b
}
```

你会注意到，这几乎与普通的 ECMAScript 模块完全相同，唯一的区别是在每个函数参数后面以`: number`形式表示类型信息，并标识返回值的类型。TypeScript 编译器可以使用这些类型来检查调用此函数的任何代码是否传入了正确的类型，并且假设返回值的正确类型。

AssemblyScript 看起来基本相同，但不是使用 JavaScript 的 `number` 类型，而是为 WebAssembly 中的每种类型提供了内置类型。如果我们想在 TypeScript 中编写相同的加法模块，并假设在整个类型中都使用 32 位整数，它看起来会像是 示例 7-3。继续在名为 *ch7-wasm-add/add.ts* 的文件中添加它。

##### 示例 7-3\. *ch7-wasm-add/add.ts*

```
export function add(a: i32, b: i32): i32 {
  return a + b
}
```

由于 AssemblyScript 文件只是 TypeScript 文件，它们使用与原来相同的 *.ts* 扩展名。要将给定的 AssemblyScript 文件编译为 WebAssembly，我们可以使用 `assemblyscript` 模块中的 `asc` 命令。尝试在 *ch7-wasm-add* 目录中运行以下命令：

```
$ npx -p assemblyscript asc add.ts --binaryFile add.wasm
```

你可以尝试使用与 示例 7-2 中相同的 *add.js* 文件运行 WebAssembly 代码。输出应该与代码相同，因为代码是一样的。

如果省略 `--binaryFile add.wasm`，则会得到转换为 WAT 的模块，如 示例 7-4 所示。你会看到它与 示例 7-1 几乎相同。

##### 示例 7-4\. AssemblyScript 中 `add` 函数的 WAT 表示

```
(module
 (type $i32_i32_=>_i32 (func (param i32 i32) (result i32)))
 (memory $0 0)
 (export "add" (func $add/add))
 (export "memory" (memory $0))
 (func $add/add (param $0 i32) (param $1 i32) (result i32)
  local.get $0
  local.get $1
  i32.add
 )
)
```

AssemblyScript 不提供生成线程的能力，但可以在 JavaScript 环境中生成线程，并且 `SharedArrayBuffers` 可用于 WebAssembly 内存。最重要的是，它通过全局的 `atomics` 对象支持原子操作，这与常规 JavaScript 的 `Atomics` 并没有特别不同。主要区别在于，这些函数不是在 `TypedArray` 上操作，而是在 WebAssembly 模块的线性内存中进行操作，带有指针和可选偏移量。详情请参阅 [AssemblyScript 文档](https://oreil.ly/LhTkW)。

要查看其运行效果，让我们创建一个新的 Happycoin 示例实现，这是我们自第 第一章 以来不断迭代的一个例子。

# AssemblyScript 幸福币

与我们之前的 Happycoin 示例版本类似，这种方法将数字的处理多路复用到多个线程，并将结果返回。这是多线程 AssemblyScript 工作方式的一个示例。在实际应用程序中，您可能希望利用共享内存和原子操作，但为了保持简单，我们将仅仅把工作分配到线程中。

让我们首先创建一个名为 *ch7-happycoin-as* 的目录，并切换到该目录。我们将初始化一个新项目，并按以下步骤添加一些必要的依赖项：

```
$ npm init -y
$ npm install assemblyscript
$ npm install @assemblyscript/loader
```

`assemblyscript` 包包含 AssemblyScript 编译器，而 `assemblyscript/loader` 包为我们提供了方便的工具，用于与构建的模块进行交互。

在新创建的 *package.json* 的 `scripts` 对象中，我们将添加 `"build"` 和 `"start"` 属性，以简化程序的编译和运行：

```
"build": "asc happycoin.ts --binaryFile happycoin.wasm --exportRuntime",
"start": "node --no-warnings --experimental-wasi-unstable-preview1 happycoin.mjs"
```

额外的 `--exportRuntime` 参数为我们提供了一些与 AssemblyScript 值交互的高级工具。稍后我们会详细介绍这一点。

在调用 Node.js 的 `"start"` 脚本时，我们传递了实验性的 WASI 标志。这启用了 [WebAssembly 系统接口 (WASI)](https://wasi.dev) 接口，使 WebAssembly 能够访问通常无法访问的系统级功能。我们将从 AssemblyScript 中使用它来生成随机数。由于写作时它仍处于实验阶段，我们将添加 `--no-warnings` 标志^(2) 来抑制因使用 WASI 而产生的警告。实验性状态还意味着该标志可能会在未来更改，因此请务必查阅运行的 Node.js 版本的 Node.js 文档。

现在，让我们来编写一些 AssemblyScript 吧！示例 7-5 包含了 Happycoin 算法的 AssemblyScript 版本。请继续将其添加到名为 *happycoin.ts* 的文件中。

##### 示例 7-5\. *ch7-happycoin-as/happycoin.ts*

```
import 'wasi'; ![1](img/1.png)

const randArr64 = new Uint64Array(1);
const randArr8 = Uint8Array.wrap(randArr64.buffer, 0, 8); ![2](img/2.png)
function random64(): u64 {
  crypto.getRandomValues(randArr8); ![3](img/3.png)
  return randArr64[0];
}

function sumDigitsSquared(num: u64): u64 {
  let total: u64 = 0;
  while (num > 0) {
    const numModBase = num % 10;
    total += numModBase ** 2;
    num = num / 10;
  }
  return total;
}

function isHappy(num: u64): boolean {
  while (num != 1 && num != 4) {
    num = sumDigitsSquared(num);
  }
  return num === 1;
}

function isHappycoin(num: u64): boolean {
  return isHappy(num) && num % 10000 === 0;
}

export function getHappycoins(num: u32): Array<u64> {
  const result = new Array<u64>();
  for (let i: u32 = 1; i < num; i++) {
    const randomNum = random64();
    if (isHappycoin(randomNum)) {
      result.push(randomNum);
    }
  }
  return result;
}
```

![1](img/#co_webassembly_CO2-1)

在这里导入了`wasi`模块，以确保加载适当的支持 WASI 的全局变量。

![2](img/#co_webassembly_CO2-2)

我们为随机数初始化了一个`Uint64Array`，但`crypto.getRandomValues()`只能使用`Uint8Array`，因此我们将在同一缓冲区上创建一个视图。此外，AssemblyScript 中的`TypedArray`构造函数没有重载，因此可以使用静态的`wrap()`方法从`ArrayBuffer`实例构造新的`TypedArray`实例。

![3](img/#co_webassembly_CO2-3)

这种方法是我们为 WASI 启用的方法。

如果你熟悉 TypeScript，你可能会认为这个文件看起来非常接近于“Happycoin: Revisited”的 TypeScript 移植版本。你是对的！这是 AssemblyScript 的主要优势之一。我们不是在一个全新的语言中编写代码，但我们编写的代码非常接近于 WebAssembly。请注意，导出函数的返回值类型为`Array<u64>`。WebAssembly 中的导出函数不能返回任何类型的数组，但可以返回模块内存的索引（实际上是一个指针），这正是这里发生的事情。我们可以手动处理这个问题，但正如我们将看到的，AssemblyScript 加载器使这一切变得更加容易。

当然，由于 AssemblyScript 本身不提供线程生成的方法，我们需要从 JavaScript 中进行操作。在这个示例中，我们将利用顶层`await`的 ECMAScript 模块，将示例 7-6 的内容放入名为*happycoin.mjs*的文件中。

##### 示例 7-6\. *ch7-happycoin-as/happycoin.mjs*

```
import { WASI } from 'wasi'; ![1](img/1.png)
import fs from 'fs/promises';
import loader from '@assemblyscript/loader';
import { Worker, isMainThread, parentPort } from 'worker_threads';

const THREAD_COUNT = 4;

if (isMainThread) {
  let inFlight = THREAD_COUNT;
  let count = 0;
  for (let i = 0; i < THREAD_COUNT; i++) {
    const worker = new Worker(new URL(import.meta.url)); ![2](img/2.png)
    worker.on('message', msg => {
      count += msg.length;
      process.stdout.write(msg.join(' ') + ' ');
      if (--inFlight === 0) {
        process.stdout.write('\ncount ' + count + '\n');
      }
    });
  }
} else {
  const wasi = new WASI();
  const importObject = { wasi_snapshot_preview1: wasi.wasiImport };
  const wasmFile = await fs.readFile('./happycoin.wasm');
  const happycoinModule = await loader.instantiate(wasmFile, importObject);
  wasi.start(happycoinModule);

  const happycoinsWasmArray =
    happycoinModule.exports.getHappycoins(10_000_000/THREAD_COUNT);
  const happycoins = happycoinModule.exports.__getArray(happycoinsWasmArray);
  parentPort.postMessage(happycoins);
}
```

![1](img/#manual_co_webassembly_CO3-1)

这需要使用`--experimental-wasi-unstable-preview1`标志才能完成。

![2](img/#manual_co_webassembly_CO3-2)

如果你对 ESM 不熟悉，这可能看起来有些奇怪。我们无法像在 CommonJS 模块中那样使用`__filename`变量。相反，`import.meta.url`属性以文件 URL 字符串的形式给出了完整路径。我们需要将其传递给`URL`构造函数，以便在`Worker`构造函数中使用。

改编自“Happycoin: Revisited”，我们再次检查是否在主线程中，并从主线程生成四个工作线程。在主线程中，我们只期望在默认`MessagePort`上收到一个消息，其中包含一个找到的 Happycoins 数组。我们简单地记录这些消息及其总数，一旦所有工作线程都发送了消息。

在`else`侧，在工作线程中，我们初始化了一个 WASI 实例以传递给 WebAssembly 模块，然后使用`@assemblyscript/loader`实例化模块，从而得到处理从`getHappycoins`函数返回的数组返回值所需的内容。我们调用模块导出的`getHappycoins()`方法，这为我们提供了指向 WebAssembly 线性内存中数组的指针。加载器提供的`__getArray`函数将该指针转换为 JavaScript 数组，然后我们可以像往常一样使用它。我们将其传递给主线程进行输出。

要运行此示例，请执行以下两个命令。第一个将将 AssemblyScript 编译为 WebAssembly，第二个将通过刚刚组装的 JavaScript 运行它：

```
$ npm run build
$ npm start
```

输出结果与先前的 Happycoin 示例大致相同。以下是来自一次本地运行的输出：

```
7641056713284760000 ... [ 134 more entries ] ... 10495060512882410000
count 136
```

和所有这些解决方案一样，评估通过适当的基准测试所做的权衡是很重要的。作为练习，请尝试计时本书中其他 Happycoin 实现与此示例之间的差异。它更快还是更慢？你能找出原因吗？可以做出哪些改进？

^(1) 在 C 和其他没有自动内存管理的语言中，必须为使用类似`malloc()`的分配函数分配内存，然后使用类似`free()`的函数释放以供稍后分配使用。像垃圾收集这样的内存管理技术使得在高级语言（如 JavaScript）中编写程序变得更加容易，但它们并非 WebAssembly 的内置特性。

^(2) 通常情况下，这不是您想在生产应用程序中启用的标志。希望在您阅读本文时，WASI 支持不再处于实验阶段。如果是这种情况，请相应调整这些参数。
