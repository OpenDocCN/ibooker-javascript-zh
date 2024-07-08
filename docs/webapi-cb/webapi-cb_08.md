# 第八章：Web 动画 API

# 简介

现代 Web 浏览器中有几种不同的元素动画方式。第一章 中有一个使用 `requestAnimationFrame` API 手动为元素创建动画的示例（参见 “使用 requestAnimationFrame 进行元素动画”）。这样做可以提供很多控制，但代价高昂。它需要跟踪时间戳以计算帧率，并且必须在 JavaScript 中计算每个增量动画变化。

## 基于关键帧的动画

CSS3 引入了关键帧动画。您可以在 CSS 规则中指定起始样式、结束样式和持续时间。浏览器会自动插值或填充动画的中间帧。使用 `@keyframes` 规则定义动画，并通过 `animation` 属性使用。示例 8-1 定义了一个淡入动画。

##### 示例 8-1。使用 CSS 关键帧动画

```
@keyframes fade {
  from {
    opacity: 0;
  }

  to {
    opacity: 1;
  }
}

.some-element {
  animation: fade 250ms;
}
```

渐变淡入动画从不透明度 0 开始，到不透明度 1 结束。当动画运行时，浏览器在 250 毫秒内计算中间样式帧。动画在元素进入 DOM 或应用 `some-element` 类时开始。

## 使用 JavaScript 进行关键帧动画

Web 动画 API 允许您在 JavaScript 代码中使用关键帧动画。`Element` 接口具有 `animate` 方法，您可以在其中定义动画的关键帧和其他选项。示例 8-2 展示了使用 Web 动画 API 从 示例 8-1 转换的相同动画。

##### 示例 8-2。使用 Web 动画 API 渐变淡入

```
const element = document.querySelector('.some-element');
element.animate([
  { opacity: 0 },
  { opacity: 1 }
], {
  // Animate for 250 milliseconds
  duration: 250
});
```

结果是相同的。元素在 250 毫秒内淡入。在这种情况下，动画由 `element.animate` 调用触发。

## 动画对象

当您调用 `element.animate` 时，将返回一个 `Animation` 对象。这允许您暂停、恢复、取消或反转动画。它还提供了一个 `Promise`，您可以使用它来等待动画完成。

要注意动画的属性。某些属性，如 `height` 或 `padding`，会影响页面的布局；对它们进行动画处理可能会导致性能问题，并且动画通常不够流畅。最佳的动画属性是 `opacity` 和 `transform`，因为它们不会影响页面布局，甚至可以由系统的 GPU 加速。

# 点击时应用“波纹”效果

## 问题

当点击按钮时，您希望在按钮内从用户点击的位置开始显示“波纹”动画。

## 解决方案

当点击按钮时，为“波纹”创建一个临时子元素。这个元素将被动画化。

首先，为波纹元素创建一些样式。按钮还需要应用一些样式（参见 示例 8-3）。

##### 示例 8-3。按钮和波纹元素的样式

```
.ripple-button {
  position: relative;
  overflow: hidden;
}

.ripple {
  background: white;
  pointer-events: none;
  transform-origin: center;
  opacity: 0;
  position: absolute;
  border-radius: 50%;
  width: 150px;
  height: 150px;
}
```

在按钮的点击处理程序中，动态创建一个新的涟漪元素并将其添加到按钮，然后更新其位置并执行动画（参见 Example 8-4）。

##### 示例 8-4\. 执行涟漪动画

```
button.addEventListener('click', async event => {
  // Create the temporary element for the ripple, set its class, and
  // add it to the button.
  const ripple = document.createElement('div');
  ripple.className = 'ripple';

  // Find the largest dimension (width or height) of the button and
  // use that as the ripple's size.
  const rippleSize = Math.max(button.offsetWidth, button.offsetHeight);
  ripple.style.width = `${rippleSize}px`;
  ripple.style.height = `${rippleSize}px`;

  // Center the ripple element on the click location.
  ripple.style.top = `${event.offsetY - (rippleSize / 2)}px`;
  ripple.style.left = `${event.offsetX - (rippleSize / 2)}px`;

  button.appendChild(ripple);

  // Perform the ripple animation and wait for it to complete.
  await ripple.animate([
    { transform: 'scale(0)', opacity: 0.5 },
    { transform: 'scale(2.5)', opacity: 0 }
  ], {
    // Animate for 500 milliseconds.
    duration: 500,
    // Use the ease-in easing function.
    easing: 'ease-in'
  }).finished;

  // All done, remove the ripple element.
  ripple.remove();
});
```

## 讨论

涟漪元素是一个圆形，大小相对于按钮的大小。你通过动画其不透明度和比例变换来实现涟漪效果。

关于元素样式，这里有几个要注意的地方。首先，按钮本身的 `position` 设置为 `relative`。这样当设置涟漪的 `absolute` 位置时，它相对于按钮本身定位。

按钮还设置了 `overflow: hidden`。这会防止涟漪效果在按钮外部可见。

你可能还会注意到涟漪设置了 `pointer-events: none`。因为涟漪位于按钮内部，所以浏览器将任何点击事件委托给按钮。这意味着点击涟漪会触发新的涟漪，但位置不正确，因为它是基于涟漪内的点击位置而不是按钮内的点击位置。

解决这个问题最简单的方法是设置 `pointer-events: none`，这使得涟漪元素忽略点击事件。如果在涟漪动画正在进行时点击涟漪，点击事件会传递到按钮，这正是你希望的，以便下一个涟漪能正确定位。

接下来，涟漪代码设置了顶部和左侧位置，以便涟漪的中心位于你刚刚点击的地方。

然后涟漪被动画化。`ripple.animate` 返回的动画具有 `finished` 属性，这是一个 `Promise`，你可以等待它。一旦这个 `Promise` 解析完成，涟漪动画就完成了，你可以从 DOM 中移除元素。

如果在涟漪进行中点击按钮，将会启动另一个涟漪，并且它们将一起动画化 —— 第一个动画不会被打断。这对于常规的 CSS 动画来说更难实现。

# 启动和停止动画

## 问题

你希望能够以编程方式启动或停止动画。

## 解决方案

使用动画的 `pause` 和 `play` 函数（参见 Example 8-5）。

##### 示例 8-5\. 切换动画的播放状态

```
/**
 * Given an animation, toggles the animation state.
 * If the animation is running, it will be paused.
 * If it is paused, it will be resumed.
 */
function toggleAnimation(animation) {
  if (animation.playState === 'running') {
    animation.pause();
  } else {
    animation.play();
  }
}
```

## 讨论

从 `element.animate` 调用返回的 `Animation` 对象具有 `playState` 属性，你可以用它来确定动画当前是否正在运行。如果正在运行，它的值是字符串 `running`。其他值包括：

`paused`

动画正在运行，但在完成之前停止了。

`finished`

动画完成并停止了。

根据 `playState` 属性，`toggleAnimation` 函数调用 `pause` 或 `play` 来设置所需的动画状态。

# 动画 DOM 插入和移除

## 问题

你希望通过动画效果向 DOM 添加或删除元素。

## 解决方案

每个操作的解决方案略有不同。

对于*添加*一个元素，首先将元素添加到 DOM 中，然后立即运行动画（例如淡入效果）。因为只有在 DOM 中的元素才能被动画化，所以您需要在运行动画之前添加它（参见示例 8-6）。

##### 示例 8-6\. 使用动画显示元素

```
/**
 * Shows an element that was just added to the DOM with a fade-in animation.
 * @param element The element to show
 */
function showElement(element) {
  document.body.appendChild(element);
  element.animate([
    { opacity: 0 },
    { opacity: 1 }
  ], {
    // Animate for 250 milliseconds.
    duration: 250
  });
}
```

要*移除*一个元素，您需要*先*运行动画（例如淡出）。一旦动画完成，立即从 DOM 中移除元素（参见示例 8-7）。

##### 示例 8-7\. 使用动画移除元素

```
/**
 * Removes an element from the DOM after performing a fade-out animation.
 * @param element The element to remove
 */
async function removeElement(element) {
  // First, perform the animation and make the element disappear from view.
  // The resulting animation's 'finished' property is a Promise.
  await element.animate([
    { opacity: 1 },
    { opacity: 0 }
  ], {
    // Animate for 250 milliseconds.
    duration: 250
  }).finished;

  // Animation is done, now remove the element from the DOM.
  element.remove();
}
```

## 讨论

当您在添加元素的同时运行动画时，它会从零不透明度开始动画，然后再开始渲染。这会产生您想要的效果——一个隐藏的元素淡入视图。

当您移除元素时，可以使用动画的`finished` `Promise`等待动画完成。在动画完全完成之前，不要将元素从 DOM 中移除，否则效果可能只运行部分并且元素会消失。

# 反向动画

## 问题

您想要取消正在进行的动画，例如悬停效果，并平稳地恢复到初始状态。

## 解决方案

使用`Animation`对象的`reverse`方法以反向播放。

您可以通过变量跟踪正在进行的动画。当您更改所需的动画状态，并且此变量具有值时，意味着另一个动画已在进行中，浏览器应该将其反转。

在悬停效果的示例中（参见示例 8-8），您可以在鼠标悬停在元素上时启动动画。

##### 示例 8-8\. 悬停效果

```
element.addEventListener('mouseover', async () => {
  if (animation) {
    // There was already an animation in progress. Instead of starting a new
    // animation, reverse the current one.
    animation.reverse();
  } else {
    // Nothing is in progress, so start a new animation.
    animation = element.animate([
      { transform: 'scale(1)' },
      { transform: 'scale(2)' }
    ], {
      // Animate for 1 second.
      duration: 1000,
      // Apply the initial and end styles.
      fill: 'both'
    });

    // Once the animation finishes, set the current animation to null.
    await animation.finished;
    animation = null;
  }
});
```

当鼠标移开时，同样的逻辑也适用（参见示例 8-9）。

##### 示例 8-9\. 移除悬停效果

```
button.addEventListener('mouseout', async () => {
  if (animation) {
    // There was already an animation in progress. Instead of starting a new
    // animation, reverse the current one.
    animation.reverse();
  } else {
    // Nothing is in progress, so start a new animation.
    animation = button.animate([
      { transform: 'scale(2)' },
      { transform: 'scale(1)' }
    ], {
      // Animate for 1 second.
      duration: 1000,
      // Apply the initial and end styles.
      fill: 'both'
    });

    // Once the animation finishes, set the current animation to null.
    await animation.finished;
    animation = null;
  }
});
```

## 讨论

由于每种情况下关键帧相同（它们仅按其顺序不同），因此您可以拥有一个单一的动画函数来设置动画的`direction`属性。当鼠标悬停在元素上时，您希望以`forward`或正常方向运行元素。当鼠标离开时，您将运行相同的动画，但方向设置为`reverse`（参见示例 8-10）。

##### 示例 8-10\. 单一动画函数

```
async function animate(element, direction) {
  if (animation) {
    animation.reverse();
  } else {
    animation = element.animate([
      { transform: 'scale(1)' },
      { transform: 'scale(2)' }
    ], {
      // Animate for 1 second.
      duration: 1000,
      // Apply the end style after the animation is done.
      fill: 'forward',
      // Run the animation forward (normal) or backward (reverse)
      // depending on the direction argument.
      direction
    });

    // Once the animation finishes, set the variable to
    // null to signal that there is no animation in progress.
    await animation.finished;
    animation = null;
  }
}

element.addEventListener('mouseover', () => {
  animate(element, 'normal');
});

element.addEventListener('mouseout', () => {
  animate(element, 'reverse');
});
```

结果与以前相同。当您悬停在元素上时，由于`scale(2)`变换，它开始增大。如果您移开鼠标，则通过反转动画的方向开始缩小。

区别在于事件处理程序。它们都调用一个单一函数，使用不同的值作为动画的`direction`选项。

示例 8-8 将动画的*fill*模式设置为`both`。动画的填充模式决定了动画前后元素的样式。默认情况下，填充模式为`none`。这意味着当动画完成时，元素的样式会跳回到动画之前的状态。

在实践中，这意味着当您悬停在元素上时，它开始增长直到达到最终大小，但由于未设置填充模式，它立即跳回到原始大小。

除了 `none` 外，填充模式还有三个选项：

`backward`

在动画开始之前，元素的样式被设置为动画的起始关键帧。通常仅在使用动画延迟时适用，因为它定义了元素在延迟期间的样式。

`forward`

动画完成后，结束关键帧样式仍然被应用。

`both`

应用了 `backward` 和 `forward` 的规则。

在 Example 8-10 中的动画没有延迟，因此使用 `forward` 选项保留动画结束后的样式。

# 显示滚动进度指示器

## 问题

您想要在页面顶部显示一个随滚动移动的条形条。随着向下滚动，该条形条向右移动。

## 解决方案

通过创建一个 `ScrollTimeline` 并将其传递给元素的 `animate` 方法来使用与滚动链接的动画。为了使元素从左向右增长，您可以将 `transition` 属性从 `scaleX(0)` 动画到 `scaleX(1)`。

###### 注意

这个 API 可能尚未被所有浏览器支持。请查看[CanIUse](https://oreil.ly/l-hvN)获取最新的兼容性数据。

首先为进度条元素设置一些样式，如 Example 8-11 所示。

##### 示例 8-11\. 滚动进度条样式

```
.scroll-progress {
  height: 8px;
  transform-origin: left;
  position: sticky;
  top: 0;
  transform: scaleX(0);
  background: blue;
}
```

`position: sticky` 属性确保元素在向下滚动页面时保持可见。此外，其初始样式设置为 `scaleX(0)`，有效地将其隐藏。没有这个设置，条形条会在瞬间出现全宽然后消失。这确保了直到滚动时才会看到条形条。

接下来，创建一个 `ScrollTimeline` 对象，并将其作为动画的 `timeline` 选项传递，如 Example 8-12 所示。

##### 示例 8-12\. 创建滚动时间线

```
const progress = document.querySelector('.scroll-progress');

// Create a timeline that's linked to the document's
// scroll position.
const timeline = new ScrollTimeline({
  source: document.documentElement
});

// Start the animation, passing the timeline you just created.
progress.animate(
  [
    { transform: 'scaleX(0)' },
    { transform: 'scaleX(1)' }
  ],
  { timeline });
```

现在您有了一个与滚动链接的动画。

## 讨论

动画的 *时间线* 是一个实现 `AnimationTimeline` 接口的对象。默认情况下，动画使用文档的默认时间线，即 `DocumentTimeline` 对象。这是一个与时钟上经过时间相关联的时间线。当您使用默认时间线启动动画时，它从初始关键帧开始向前运行，直到达到结束（或手动停止）。因为这种类型的时间线与经过的时间相关联，它具有定义的起始值，并且随着时间的推移不断增加。

然而，*与滚动链接的* 动画提供了一个与滚动位置相关联的时间线。当您滚动到顶部时，滚动位置为 0，动画保持在其初始状态。随着向下滚动，位置增加，动画前进。一旦您滚动到底部，动画达到结束。如果向上滚动，则动画反向运行。

`ScrollTimeline`被赋予一个源元素。在示例 8-12 中，源是文档元素（`body`标签）。您可以将任何可滚动元素作为源传递，并且`ScrollTimeline`使用该元素的滚动位置来确定当前进度。

在撰写本文时，`DocumentTimeline`在所有现代浏览器中都受支持，但`ScrollTimeline`则不受支持。在使用`ScrollTimeline`之前，请务必检查浏览器支持情况。

# 使元素弹跳

## 问题

您想要对一个元素应用短暂的弹跳效果。

## 解决方法

应用一系列动画，依次执行。使用动画的`finished` `Promise`来等待其完成，然后再运行下一个动画。

元素上下移动三次。每次通过`translateY`变换将元素向页面上移动，然后回到原始位置。第一次移动使元素反弹 40 像素，第二次移动使元素反弹 20 像素，第三次移动使元素反弹 10 像素。这样看起来像是重力每次减慢反弹速度。这可以通过`for`-`of`循环来实现（见示例 8-13）。

##### 示例 8-13\. 串行应用弹跳动画

```
async function animateBounce(element) {
  const distances = [ '40px', '20px', '10px' ];
  for (let distance of distances) {
    // Wait for this animation to complete before continuing.
    await element.animate([
      // Start at the bottom.
      { transform: 'translateY(0)' },

      // Move up by the current distance.
      { transform: `translateY(-${distance})`, offset: 0.5 },

      // Back to the bottom
      { transform: 'translateY(0)' }
    ], {
      // Animate for 250 milliseconds.
      duration: 250,

      // Use a more fluid easing function than linear
      // (the default).
      easing: 'ease-in-out'
    }).finished;
  }
}
```

## 讨论

本示例演示了 Web 动画 API 的一个优势：动态关键帧值。每次循环迭代都使用关键帧效果内部的不同`distance`值。

`for`-`of`循环遍历三个距离值（40px，20px 和 10px），依次对它们进行动画处理。在每次迭代中，它将元素向上移动给定的距离，然后再向下移动。关键在于最后一行，它引用了动画的`finished`属性。这确保了在当前动画完成之前不会开始下一个循环迭代。结果是动画依次串行运行，提供了弹跳效果。

您可能会想为什么本例中使用`for`-`of`循环而不是数组的`forEach()`方法。在诸如`forEach`之类的数组方法中使用`await`不会按预期工作。这些方法不是为异步使用而设计的。如果使用`forEach`调用，`element.animate`调用将会立即依次执行，结果只有最后一个动画会播放。使用`for`-`of`循环（普通的`for`循环也可以）与`async`/`await`一起按预期工作，并得到所需的结果。

# 同时运行多个动画

## 问题

您想要使用多个动画对一个元素应用多个变换。

## 解决方法

对元素多次调用`animate`，使用不同的变换关键帧。你还必须指定`composite`属性来组合这些变换，如示例 8-14 所示。

##### 示例 8-14\. 结合两个变换动画

```
// The first animation will move the element back and forth on the x-axis.
element.animate([
  { transform: 'translateX(0)' },
  { transform: 'translateX(250px)' }
], {
  // Animate for 5 seconds.
  duration: 5000,
  // Run the animation forward, then run it in reverse.
  direction: 'alternate',
  // Repeat the animation forever.
  iterations: Infinity,
  // Slow to start, fast in the middle, slow at the end.
  easing: 'ease-in-out'
});

// The second animation rotates the element.
element.animate([
  { transform: 'rotate(0deg)' },
  { transform: 'rotate(360deg)' }
], {
  // Animate for 3 seconds.
  duration: 3000,
  // Repeat the animation forever.
  iterations: Infinity,
  // Combine the effects with other running animations.
  composite: 'add'
});
```

`alternate`方向意味着动画正向运行完成，然后反向运行完成。因为`iterations`设置为`Infinity`，所以动画会无限循环运行。

## 讨论

此效果的关键在于第二个动画中添加的`composite`属性。如果不指定`composite: add`，则*仅*会看到`rotate`变换，因为它会覆盖`translateX`变换。元素会旋转但不会水平移动。

实际上，这将两个变换组合成单个变换。但请注意，这些变换发生的速率不同。旋转持续三秒，而平移持续五秒。动画还使用不同的缓动函数。尽管有不同的选项，浏览器仍然平滑地组合这些动画。

# 显示加载动画

## 问题

在等待网络请求完成时，您希望向用户显示加载指示器。

## 解决方案

创建并设计加载指示器，然后对其应用无限旋转动画，直到由`fetch`返回的`Promise`解析。

为了产生流畅的效果，您可以首先应用一个淡入动画。一旦`Promise`解析完成，您可以将其淡出。

首先，创建一个加载器元素并定义一些样式，如示例 8-15 所示。

##### 示例 8-15\. 加载器元素

```
<style>
  #loader {
    width: 64px;
    height: 64px;

    /* Make a circle shape */
    border-radius: 50%;
    border-width: 10px;
    border-style: solid;
    border-color: skyblue blue skyblue blue;

    /* Set the initial opacity so the animation that appears is smooth */
    opacity: 0;
  }
</style>

<div id="loader"></div>
```

加载器是一个具有交替边框颜色的环，如图 8-1 所示。

![风格化加载器](img/wacb_0801.png)

###### 图 8-1\. 风格化加载器

接下来，定义一个启动动画并等待`Promise`的函数，如示例 8-16 所示。

##### 示例 8-16\. 加载器动画

```
async function showLoader(promise) {
  const loader = document.querySelector('#loader');

  // Start the spin animation before fading in.
  const spin = loader.animate([
    { transform: 'rotate(0deg)' },
    { transform: 'rotate(360deg)' }
  ], { duration: 1000, iterations: Infinity });

  // Since the opacity is 0, the loader isn't visible yet.
  // Show it with a fade-in animation.
  // The loader will continue spinning as it fades in.
  loader.animate([
    { opacity: 0 },
    { opacity: 1 }
  ], { duration: 500, fill: 'both' });

  // Wait for the Promise to resolve.
  await promise;

  // The Promise is done. Now fade the loader out.
  // Don't stop the spin animation until the fade out is complete.
  // You can wait by awaiting the 'finished' Promise.
  await loader.animate([
    { opacity: 1 },
    { opacity: 0 }
  ], { duration: 500, fill: 'both' }).finished;

  // Finally, stop the spin animation.
  spin.cancel();

  // Return the original Promise to allow chaining.
  return promise;
}
```

现在可以将您的`fetch`调用作为参数传递给`showLoader`，如示例 8-17 所示。

##### 示例 8-17\. 使用加载器

```
showLoader(
  fetch('https://example.com/api/users')
    .then(response => response.json())
);
```

## 讨论

您不一定需要 Web 动画 API 来创建动画加载器——您可以使用纯 CSS 来实现。但正如这个示例所示，Web 动画 API 允许您结合多个动画。无限旋转动画继续运行，而淡入动画也在运行。这在常规 CSS 动画中有些棘手。

# 尊重用户的动画偏好

## 问题

如果用户已经配置操作系统减少动画，您希望减弱或禁用动画。

## 解决方案

使用`window.matchMedia`来检查`prefers-reduced-motion`媒体查询（参见示例 8-18）。

##### 示例 8-18\. 使用 prefers-reduced-motion 媒体查询

```
if (!window.matchMedia('(prefers-reduced-motion: reduce)').matches) {
  // Reduced motion is not enabled, so animate normally.
} else {
  // Skip this animation or run a less intense one.
}
```

## 讨论

这对可访问性非常重要。癫痫或前庭障碍的用户可能会因大型或快速移动的动画而引发癫痫、偏头痛或其他不良反应。

您不一定需要完全禁用动画；您可以选择使用更为微妙的动画效果。假设您正在显示一个具有很好视觉效果的弹跳效果元素，但对某些用户可能会造成迷惑。如果用户启用了减少动画选项，您可以改为提供简单的淡入动画。
