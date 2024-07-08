# 第四章：日期

JavaScript 具有出乎意料的强大日期功能，这些功能包装在有些老式的 `Date` 对象中。正如你将会看到的，`Date` 对象有其怪癖和隐藏陷阱——比如它从 0 开始计数月份，并根据当前计算机的地区设置不同解析年份信息。但一旦你学会如何应对这些障碍，你将能够完成许多常见而有用的操作，比如计算两个日期之间的天数，为显示格式化日期，以及计时事件。

# 获取当前日期和时间

## Problem

你需要获取当前日期或时间。

## Solution

JavaScript 包含一个 `Date` 对象，提供了良好的支持，用于操作日期信息（以及较为适度的日期计算支持）。当你创建一个新的 `Date` 对象时，它会自动填充当前的日期和时间，精确到最近的毫秒：

```
const today = new Date();
```

现在只需从你的 `Date` 对象中提取你想要的信息。`Date` 对象有一长串方法可以帮助你完成这个任务。表 4-1 列出了最重要的方法。请注意，不同方法使用的计数并不总是一致的。月份和星期的编号从 0 开始，而日期从 1 开始编号。

表 4-1\. 获取日期信息的日期方法

| 方法 | 获取 | 可能的值 |
| --- | --- | --- |
| `getFullYear()` | 年份 | 像 2021 这样的四位数字 |
| `getMonth()` | 月份编号 | 0 到 11，其中 0 表示一月 |
| `getDate()` | 月份中的某一天 | 1 到 31 |
| `getDay()` | 星期几 | 0 到 6，其中 0 表示星期日 |
| `getHours()` | 一天中的小时 | 0 到 23 |
| `getMinutes()` | 分钟数 | 0 到 59 |
| `getSeconds()` | 秒数 | 0 到 59 |
| `getMilliseconds()` | 毫秒数（千分之一秒） | 0 到 999 |

这是一个显示当前日期一些基本信息的示例：

```
const today = new Date();

console.log(today.getFullYear());  // example: 2021
console.log(today.getMonth());     // example: 02 (March)
console.log(today.getDay());       // example: 01 (Monday)

// Do a little extra string processing to make sure minutes are padded with
// a leading 0 if needed to make a two-digit value (like '05' in the time 4:05)
const hours = today.getHours();
const minutes = today.getMinutes().toString().padStart(2, '0');
console.log('Time ' + hours + ':' + minutes);   // example: 15:32
```

###### 注意

表 4-1 中列出的 `Date` 方法存在两个版本。表中显示的版本使用本地时间设置。第二组方法在方法名称前加上前缀 *UTC*（如 `getUTCMonth()` 和 `getUTCSeconds()`）。它们使用 *协调世界时*，全球时间标准。如果你需要比较来自不同时区的日期（或者遵循不同夏令时约定的日期），你必须使用 UTC 方法。在内部，`Date` 对象始终使用 UTC。

## 讨论

`Date()` 对象有几个构造函数。空构造函数创建一个当前日期和时间的 `Date` 对象，正如你刚刚看到的。但你也可以通过指定年、月和日来创建一个不同日期的 `Date` 对象，就像这样：

```
// February 10, 2021:
const anotherDay = new Date(2021, 1, 10);
```

再次注意，要注意不一致的计数方式（月份从 0 开始，而日期从 1 开始）。这意味着上面的 `anotherDay` 变量代表的是 2 月 10 日，而不是 1 月 10 日。

可选地，您可以为 `Date` 构造函数添加最多四个参数，分别为小时、分钟、秒钟和毫秒：

```
// February 1, 2021, at 9:30 AM:
const anotherDay = new Date(2021, 1, 1, 9, 30);
```

正如本章所述，JavaScript 内置的 `Date` 对象有一些众所周知的局限性和一些怪异之处。如果您需要在代码中执行广泛的日期操作，例如计算日期范围、解析不同类型的日期字符串或在时区之间移动日期，则最佳实践是使用经过测试的第三方日期库，例如 [*day.js*](https://github.com/iamkun/dayjs) 或 [*date-fns*](https://date-fns.org)。

## 另请参阅

一旦您有了日期，您可能希望在日期计算中使用它，如 “比较日期和测试日期是否相等” 中所述。您可能还有兴趣将日期转换为格式化的字符串（“将日期值格式化为字符串”）或将包含日期的字符串转换为正确的 `Date` 对象（“将字符串转换为日期”）。

# 将字符串转换为日期

## 问题

您有一个日期信息的字符串，但您希望将其转换为 `Date` 对象，以便在代码中操纵它或执行日期计算。

## 解决方案

如果您很幸运，您的日期字符串符合 ISO 8601 标准时间戳格式（例如 “2021-12-17T03:24:00Z”），您可以直接将其传递给 `Date` 构造函数：

```
const eventDate = new Date('2021-12-17T03:24:00Z');
```

此字符串中的 *T* 将日期与时间分隔开，字符串末尾的 *Z* 表示它是使用 UTC 时区的通用时间，这是确保在不同计算机上保持一致性的最佳方式。

`Date` 构造函数（以及 `Date.parse()` 方法）可能识别其他格式。但是，现在强烈建议避免使用这些格式，因为它们在不同浏览器中的实现不一致。它们在测试示例中可能运行正常，但是在不同浏览器应用不同的区域设置（例如夏令时）时可能会遇到问题。

如果您的日期不是 ISO 8601 格式，您需要采取手动方法。从字符串中提取不同的日期组件，然后与 `Date` 构造函数一起使用。您可以充分利用像 `split()`、`slice()` 和 `indexOf()` 这样的 `String` 方法，这些方法在 第二章 的示例中有更详细的讨论。

例如，如果您有一个格式为 *mm/dd/yyyy* 的日期字符串，您可以使用以下代码：

```
const stringDate = '12/30/2021';

// Split on the slashes
const dateArray = stringDate.split('/');

// Find the individual date ingredients
const year = dateArray[2];
const month = dateArray[0];
const day = dateArray[1];

// Apply the correction for 0-based month numbering
const eventDate = new Date(year, month-1, day);
```

## 讨论

`Date` 对象构造函数并不执行太多的验证。在创建 `Date` 对象之前，请检查您的输入，因为 `Date` 对象可能接受您不希望的值。例如，它允许日期数字滚动（换句话说，如果您将 40 设置为您的日期数字，JavaScript 将会将您的日期移动到下个月）。`Date` 构造函数还会接受可能在不同计算机上解析不一致的字符串。

如果你尝试用非数字字符串创建一个`Date`对象，你将收到一个“Invalid Date”对象。你可以使用`isNaN()`来测试这种情况：

```
const badDate = '12 bananas';

const convertedDate = new Date(badDate);

if (Number.isNaN(convertedDate)) {
  // We end up here, because the date object was not created successfully
} else {
  // For a valid Data instance, we end up here
}
```

这种技术有效，因为`Date`对象实际上是幕后的*数字*，这一点在“比较日期和测试日期的相等性”中有所探讨。

## 另请参阅

“将日期值格式化为字符串”解释了相反的操作——将`Date`对象转换为字符串。

# 给日期添加天数

## 问题

你想找到在另一个日期之前或之后特定天数的日期。

## 解决方案

使用`Date.getDate()`找到当前日期号码，然后用`Date.setDate()`更改它。`Date`对象足够智能，可以根据需要滚动到下一个月或年。

```
const today = new Date();
const currentDay = today.getDate();

// Where will be three weeks in the future?
today.setDate(currentDay + 21);
console.log(`Three weeks from today is ${today}`);
```

## 讨论

`setDate()`方法不仅限于正整数。你可以使用负数来向后移动日期。你可能想使用其他*setXxx()*方法来修改日期，比如`setMonths()`每次向前或向后移动一个月，`setHours()`每小时移动，等等。所有这些方法都会像`setDate()`一样滚动，因此添加 48 小时将使日期向前移动两天。

`Date`对象是*可变的*，这使得其行为看起来相当老式。在更现代的 JavaScript 库中，你期望像`setDate()`这样的方法返回一个新的`Date`对象。但它实际上改变的是*当前*的`Date`对象。即使你用`const`声明了一个日期，这种情况仍然会发生。（`const`阻止你将变量指向不同的`Date`对象，但不能阻止你修改当前引用的`Date`对象。）为了安全地避免潜在的问题，你可以在操作日期之前克隆它。只需使用`Date.getTime()`获取表示日期的毫秒数，并用它创建一个新对象：

```
const originalDate = new Date();

// Clone the date
const futureDate = new Date(originalDate.getTime());

// Change the cloned date
futureDate.setDate(originalDate.getDate()+21);
console.log(`Three weeks from ${originalDate} is ${futureDate}`);
```

## 另请参阅

“计算两个日期之间经过的时间”展示了如何计算两个日期之间的时间段。

# 比较日期和测试日期的相等性

## 问题

你需要查看两个`Date`对象是否表示相同的日历日期，或者确定一个日期是否在另一个日期之前。

## 解决方案

你可以像比较数字一样比较`Date`对象，使用`<`和`>`运算符：

```
const oldDay = new Date(1999, 10, 20);
const newerDay = new Date(2021, 1, 1);

if (newerDay > oldDay) {
  // This is true, because newerDay falls after oldDay.
}
```

在内部，日期被存储为数字。当你使用`<`或`>`运算符时，它们会自动转换为数字并进行比较。当你运行此代码时，你正在比较`oldDay`（943,074,000,000）的毫秒值与`newerDay`（1,612,155,600,000）的毫秒值。

等号运算符（`=`）的工作方式不同。它测试对象的引用，而不是对象的*内容*。（换句话说，只有当你比较指向同一实例的两个变量时，两个`Date`对象才相等。）

如果您希望测试两个`Date`对象是否表示同一时刻，您需要自己将它们转换为数字。最清晰的方法是调用`Date.getTime()`，它返回日期的毫秒数：

```
const date1 = new Date(2021, 1, 1);
const date2 = new Date(2021, 1, 1);

// This is false, because they are different objects
console.log(date1 === date2);

// This is true, because they have the same date
console.log(date1.getTime() === date2.getTime());
```

###### 注意

尽管其名称为`getTime()`，但它并不仅仅返回时间。它返回的是毫秒数，准确表示该`Date`对象的日期和时间。

## 讨论

在内部，`Date`对象只是一个整数。具体来说，它是自 1970 年 1 月 1 日以来经过的毫秒数。毫秒数可以是负数或正数，这意味着`Date`对象可以表示从遥远的过去（大约公元前 271,821 年）到遥远的未来（公元 275,760 年）的日期。您可以通过调用`Date.getTime()`获取毫秒数。

两个`Date`对象仅在精确匹配（毫秒级别）时才相同。即使两个`Date`对象表示同一日期但具有不同的时间组件，它们也不会匹配。这可能是个问题，因为您可能没有意识到您的`Date`对象包含时间信息。这在为当前日期创建`Date`对象时是一个常见问题（“获取当前日期和时间”）。

要避免这个问题，您可以使用`Date.setHours()`来移除时间信息。尽管它的名称是`setHours()`，但该方法最多接受四个参数，允许您设置小时、分钟、秒和毫秒。为了创建一个仅包含日期的`Date`对象，将所有这些组件都设置为 0：

```
const today = new Date();

// Create another copy of the current date
// The day hasn't changed, but the time may have already ticked on
// to the next millisecond
const todayDifferent = new Date();

// This could be true or false, depending on timing factors beyond your control
console.log(today.getTime() === todayDifferent.getTime());

// Remove all the time information
todayDifferent.setHours(0,0,0,0);
today.setHours(0,0,0,0);

// This is always true, because the time has been removed from both instances
console.log(today.getTime() === todayDifferent.getTime());
```

## 另请参见

欲了解更多关于日期的数学内容，请参见 Recipes and 。

# 计算两个日期之间的经过时间

## 问题

您需要计算两个日期之间相隔多少天、小时或分钟。

## 解决方案

因为日期是数字（以毫秒为单位，请参见“比较日期和测试日期是否相等”），所以与它们进行计算相对简单。如果从一个日期减去另一个日期，您将得到它们之间的毫秒数：

```
const oldDate = new Date(2021, 1, 1);
const newerDate = new Date(2021, 10, 1);

const differenceInMilliseconds = newerDate - oldDate;
```

除非您正在为性能测试计时短操作，否则毫秒数不是特别有用的单位。您需要将此数字除以适当的值，以将其转换为更有意义的分钟数、小时数或天数：

```
const millisecondsPerDay = 1000*60*60*24;
let differenceInDays = differenceInMilliseconds / millisecondsPerDay;

// Only count whole days
differenceInDays = Math.trunc(differenceInDays);

console.log(differenceInDays);
```

尽管此计算应该得出确切的天数（因为两个日期都没有任何时间信息），但仍需在结果上使用`Math.round()`来处理浮点数运算中的舍入误差（请参见“保留小数值的准确性”）。

## 讨论

在执行日期计算时，有两个需要注意的陷阱：

+   日期可能包含时间信息。（例如，为当前日期创建的新`Date`对象在其创建的毫秒级别上是准确的。）在计算天数之前，请使用`setHours()`来移除时间组件，正如“比较日期和测试日期是否相等”中所解释的那样。

+   仅在日期处于相同时区时，两个日期的计算才有意义。理想情况下，这意味着您比较两个本地日期或两个 UTC 标准日期。从一个时区转换日期到另一个时区可能看起来很简单，但通常会出现关于夏令时的意外边缘情况。

对于老旧的`Date`对象，有一个暂定的替代方案。[`Temporal`对象](https://oreil.ly/BAbB2)旨在改进使用本地日期和不同时区的计算。与此同时，如果您的日期需求超出了`Date`对象，可以尝试使用第三方库来操作日期。[*day.js*](https://github.com/iamkun/dayjs)和[*date-fns*](https://date-fns.org)都是流行的选择。

如果您想要使用微小的时间计算来分析性能，请注意`Date`对象不是最佳选择。相反，请使用`Performance`对象，在浏览器环境中通过内置的`window.performance`属性可用。它允许您捕获高分辨率时间戳，如果系统支持，精度可以达到*毫秒的分数*。以下是一个示例：

```
// Get a DOMHighResTimeStamp object that represents the start time
const startTime = window.performance.now();

// (Do a time consuming task here.)

// Get a DOMHighResTimeStamp object that represents the end time
const endTime = window.performance.now();

// Find the elapsed time in milliseconds
const elapsedMilliseconds = endTime - startTime;
```

结果（`elapsedMilliseconds`）不是最近的整毫秒，而是当前硬件支持的最精确的分数毫秒计数。

###### 注意

虽然 Node 不提供`Performance`对象，但它具有自己的机制来检索高分辨率时间信息。您可以使用其全局`process`对象，该对象提供`process.hrtime.bigint()`方法。它返回以*纳秒*或十亿分之一秒为单位的时间读数。只需从另一个`process.hrtime.bigint()`读数中减去一个读数，即可找到纳秒中的时间差异。（每毫秒为 1,000,000 纳秒。）

因为纳秒计数显然会是一个非常大的数字，您需要使用`BigInt`数据类型来保存它，如“使用 BigInt 操作非常大的数字”中所述。

## 另请参阅

“添加日期”显示如何通过增加或减少日期来向前或向后移动日期。

# 将日期值格式化为字符串

## 问题

您想基于`Date`对象创建格式化的字符串。

## 解决方案

如果使用`console.log()`打印日期，您将获得日期的格式化字符串表示，例如“Wed Oct 21 2020 22:17:03 GMT-0400 (Eastern Daylight Time)”。此表示是由`DateTime.toString()`方法创建的标准化、非特定于区域设置的日期字符串，定义在[JavaScript 标准](https://oreil.ly/S0lMb)中。

###### 注意

在内部，`Date`对象将其时间信息存储为 UTC 时间，没有附加的时区信息。当您将`Date`转换为字符串时，该 UTC 时间将转换为当前计算机或设备上设置的特定于区域设置的时间。

如果您希望日期字符串格式不同，可以调用此处演示的其他预定义`Date`方法之一：

```
const date = new Date(2021, 0, 1, 10, 30);

let dateString;
dateString = date.toString();
 // 'Fri Jan 01 2021 10:30:00 GMT-0500 (Eastern Standard Time)'

dateString = date.toTimeString();
 // '10:30:00 GMT-0500 (Eastern Standard Time)'

dateString = date.toUTCString();
 // 'Fri, 01 Jan 2021 15:30:00 GMT'

dateString = date.toDateString();
 // 'Fri Jan 01 2021'

dateString = date.toISOString();
 // '2021-01-01T15:30:00.000Z'

dateString = date.toLocaledateString();
 // '1/1/2021, 10:30:00 AM'

dateString = date.toLocaleTimeString();
// '10:30:00 AM'
```

请记住，如果使用`toLocaleString()`或`toLocaleTime()`，您的字符串表示基于浏览器实现和当前计算机的设置。不要假设一致性！

## 讨论

将日期信息转换为字符串有许多可能的方法。对于显示目的，*toXxxString()*方法效果很好。但是，如果您需要更具体或更精细调整的内容，可能需要自己控制`Date`对象。

如果您希望超越标准格式化方法，有两种方法可以尝试。您可以使用“获取当前日期和时间”中描述的*getXxx()*方法从日期中提取单独的时间组件，然后将其连接成您需要的精确字符串。以下是一个示例：

```
const date = new Date(2021, 10, 1);

// Ensure date numbers less than 10 are padded with an initial 0.
const day = date.getDate().toString().padStart(2, '0');

// Ensure months are 0-padded and add 1 to convert the month from its
// 0-based JavaScript representation
const month = (date.getMonth()+1).toString().padStart(2, '0');

// The year is always 4-digit
const year = date.getFullYear();

const customDateString = `${year}.${month}.${day}`;
// now customDateString = '2021.11.01'
```

这种方法非常灵活，但它强制您编写自己的日期样板，这并不理想，因为它增加了复杂性并为新错误创造了空间。

如果您想为特定区域设置使用标准格式，那么情况会变得简单一些。您可以使用`Intl.DateTimeFormat`对象进行转换。以下是三个示例，使用了美国、英国和日本的区域字符串：

```
const date = new Date(2020, 11, 20, 3, 0, 0);

// Use the standard US date format
console.log(new Intl.DateTimeFormat('en-US').format(date));  // '12/20/2020'

// Use the standard UK date format
console.log(new Intl.DateTimeFormat('en-GB').format(date));  // '20/12/2020'

// Use the standard Japanese date format
console.log(new Intl.DateTimeFormat('ja-JP').format(date));  // '2020/12/20'
```

所有这些都是仅包含日期的字符串，但是在创建`Intl.DateTimeFormat()`对象时，您可以设置许多其他选项。以下是一个示例，将星期几和月份添加到德语字符串中：

```
const date = new Date(2020, 11, 20);

const formatter = new Intl.DateTimeFormat('de-DE',
 { weekday: 'long', year: 'numeric', month: 'long', day: 'numeric' });

const dateString = formatter.format(date);
// now dateString = 'Sonntag, 20\. Dezember 2020'
```

这些选项还允许您通过`hour`、`minute`和`second`属性向字符串添加时间信息，可以设置为：

```
const date = new Date(2022, 11, 20, 9, 30);

const formatter = new Intl.DateTimeFormat('en-US',
 { year: 'numeric', month: 'numeric', day: 'numeric',
   hour: 'numeric', minute: 'numeric' });

const dateString = formatter.format(date);
// now dateString = '12/20/2022, 9:30 AM'
```

## 参见

“将数值转换为格式化字符串”介绍了`Intl`对象和区域字符串的概念，这些字符串标识不同的地理和文化区域。要详细了解`Intl.DateTimeFormat`对象支持的 21 个选项，请参阅[MDN 参考文档](https://oreil.ly/at36f)。值得注意的是，其中一些细节取决于实现，并非所有浏览器都支持（例如`timeStyle`、`dateStyle`和`timeZone`属性，在此处我们没有讨论）。对于复杂的日期操作，始终考虑使用第三方库。
