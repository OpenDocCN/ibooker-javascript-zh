# 第十一章：国际化

# 简介

现代浏览器包括强大的国际化 API。这是一组围绕语言或特定区域任务的 API 集合，例如：

+   格式化日期和时间

+   格式化数字

+   货币

+   复数规则

在这个 API 出现之前，您可能不得不使用 Moment.js（用于日期和时间）或 Numeral.js（用于数字）等第三方库。然而，现代浏览器支持许多相同的用例，您可能不再需要这些库来开发应用程序。

大多数这些 API 使用*区域设置*的概念，通常是语言和地区的组合。例如，美国英语的区域设置是`en-US`，加拿大英语的区域设置是`en-CA`。您可以使用默认区域设置，即浏览器正在使用的区域设置，或者您可以指定特定的区域设置以便根据您所需的区域适当地格式化数据。

###### 注意

JavaScript 正在开发中的新的日期和时间 API 称为 Temporal。在撰写本书时，这仍然是一个 ECMAScript 提案。它可能会在不久的将来成为语言的一部分，但目前本书将介绍标准的 Date API。

# 格式化日期

## 问题

您希望以适合用户区域设置的格式显示`Date`对象。

## 解决方案

使用`Intl.DateTimeFormat`将`Date`对象格式化为字符串值。创建包含两个参数的格式对象：所需的区域设置和一个可以指定格式样式的选项对象。对于日期，支持的格式样式有（在`en-US`区域设置下显示示例）：

+   `short`: 10/16/23

+   `medium`: Oct 16, 2023

+   `long`: 2023 年 10 月 16 日

+   `full`: 2023 年 10 月 16 日星期一

要获取用户当前的区域设置，您可以检查`navigator.language`属性（参见示例 11-1）。

##### 示例 11-1\. 格式化日期

```
const formatter = new Intl.DateTimeFormat(navigator.language, { dateStyle: 'long' });
const formattedDate = formatter.format(new Date());
```

## 讨论

您还可以通过在选项对象中指定`timeStyle`属性和`dateStyle`（参见示例 11-2）来包含`Date`对象的时间信息。

##### 示例 11-2\. 格式化日期和时间

```
const formatter = new Intl.DateTimeFormat(navigator.language, {
dateStyle: 'long', timeStyle: 'long' });
const formattedDateAndTime = formatter.format(new Date());
```

# 获取格式化日期的部分

## 问题

您希望将格式化的日期拆分为令牌。例如，如果您希望不同部分的格式化日期具有不同的样式，则此操作非常有用。

## 解决方案

使用`Intl.DateTimeFormat`的`formatToParts`方法格式化日期并返回一个令牌数组（参见示例 11-3）。

##### 示例 11-3\. 获取格式化日期的部分

```
const formatter = new Intl.DateTimeFormat(navigator.language,
  { dateStyle: 'short' });
const parts = formatter.formatToParts(new Date());
```

## 讨论

对于`10/1/23`的短日期，示例 11-3 中显示的`parts`对象如示例 11-4 所示。

##### 示例 11-4\. 格式化的日期部分

```
[
  { type: 'month', value: '10' },
  { type: 'literal': value: '/' },
  { type: 'day': value: '1' },
  { type: 'literal', value: '/' },
  { type: 'year', value: '23' }
]
```

# 格式化相对日期

## 问题

您希望以近似的人类可读格式格式化给定日期与今天之间的差异。例如，您希望得到类似“2 天前”或“3 个月后”的格式化字符串。

## 解决方案

使用`Intl.RelativeTimeFormat`。它具有`format`方法，您可以用值偏移量（例如-2 代表过去，3 代表将来）和单位（如“day”、“month”等）调用它。例如，在`en-US`区域设置中调用`format(-2, *day*)`将返回字符串“2 天前”。

实际上这是一个两步过程。`Intl.RelativeTimeFormat`不直接计算两个日期之间的时间差。相反，您需要首先确定偏移量和单位，然后将它们传递给`format`方法。其思想是找到在两个日期之间存在差异的最大单位。

首先，创建一个帮助函数，该函数返回一个包含偏移量和单位的对象，如示例 11-5 所示。

##### 示例 11-5\. 查找偏移量和单位

```
function getDateDifference(fromDate) {
  const today = new Date();

  if (fromDate.getFullYear() !== today.getFullYear()) {
    return { offset: fromDate.getFullYear() - today.getFullYear(), unit: 'year' };
  } else if (fromDate.getMonth() !== today.getMonth()) {
    return { offset: fromDate.getMonth() - today.getMonth(), unit: 'month' };
  } else {
    // You could even go more granular: down to hours, minutes, or seconds!
    return { offset: fromDate.getDate() - today.getDate(), unit: 'day' };
  }
}
```

此函数返回一个包含两个属性`offset`和`unit`的对象，您可以将其传递给`Intl.RelativeTimeFormat`（参见示例 11-6）。

##### 示例 11-6\. 格式化相对日期

```
function getRelativeDate(fromDate) {
  const { offset, unit } = getDateDifference(fromDate);
  const format = new Intl.RelativeTimeFormat();
  return format.format(offset, unit);
}
```

如果您在 2023 年 10 月 7 日调用此函数，则以下是预期输出（请记住，以这种方式创建`Date`对象时，月份从 0 开始，但日期从 1 开始）：

+   2023 年 10 月 1 日：`getRelativeDate(new Date(2023, 9, 1))`：“6 天前”

+   2023 年 5 月 2 日：`getRelativeDate(new Date(2023, 4, 2))`：“5 个月前”

+   2025 年 6 月 2 日：`getRelativeDate(new Date(2025, 5, 2))`：“在 2 年内”

## 讨论

`getDateDifference`通过比较给定日期的年份、月份和日期（按顺序），与今天的日期进行比较，直到找到不匹配的日期为止。然后返回差异和单位名称，这些将传递给`Intl.RelativeTimeFormat`。

`getRelativeDate`函数并不会精确地返回月份、天数、小时、分钟和秒数的相对时间。它只是给出时间差的量级估计。

考虑将 2023 年 5 月 2 日与 2023 年 10 月 7 日进行比较。它们相差 5 个月 5 天，但`getRelativeDate`仅显示“5 个月前”作为近似。

# **格式化数字**

## 问题

您希望以区域特定的方式使用千位分隔符和小数位数格式化数字。

## 解决方案

将数字传递给`Intl.NumberFormat`的`format`方法。该方法返回一个包含格式化后数字的字符串。

默认情况下，`Intl.NumberFormat`使用默认区域设置（假设示例 11-7 中的默认区域设置是`en-US`）。

##### 示例 11-7\. 使用默认区域设置格式化数字

```
// outputs '5,200.55' for en-US
console.log(
  new Intl.NumberFormat().format(5200.55)
);
```

您还可以为`Intl.NumberFormat`构造函数指定不同的区域设置（参见示例 11-8）。

##### 示例 11-8\. 使用`de-DE`区域设置格式化数字

```
// outputs '5.200,55'
console.log(
  new Intl.NumberFormat('de-DE').format(5200.55)
);
```

## 讨论

`Intl.NumberFormat`应用区域特定的格式化规则来格式化单个数字。您还可以通过将两个值传递给`formatRange`来格式化一系列数字，如示例 11-9 所示。

##### 示例 11-9\. 格式化数字范围

```
// outputs '1,000-5,000' for en-US
console.log(
  new Intl.NumberFormat().formatRange(1000, 5000)
);
```

# **舍入小数位数**

## 问题

您想要取一个可以有许多小数位数的分数，并将其舍入到一定小数位数。

## 解决方案

使用`maximumFractionDigits`选项来指定小数点后的位数。 示例 11-10 展示了如何将数字四舍五入到最多两位小数。

##### 示例 11-10\. 四舍五入数字

```
function roundToTwoDecimalPlaces(number) {
  const format = new Intl.NumberFormat(navigator.language, {
    maximumFractionDigits: 2
  });

  return format.format(number);
}

// prints "5.49"
console.log(roundToTwoDecimalPlaces(5.49125));

// prints "5.5"
console.log(roundToTwoDecimalPlaces(5.49621));
```

# 格式化价格范围

## 问题

给定一个作为数字存储的价格数组，您希望创建一个反映数组中最低和最高价格的格式化价格范围。

## 解决方案

确定最低和最高价格，然后在创建`Intl.NumberFormat`时传递`style: *currency*`选项。使用此`Intl.NumberFormat`创建范围。还可以指定货币以获取输出中的正确符号。最后，在`Intl.NumberFormat`上调用`formatRange`，传入较低和较高的价格边界（参见示例 11-11）。

##### 示例 11-11\. 格式化价格范围

```
function formatPriceRange(prices) {
  const format = new Intl.NumberFormat(navigator.language, {
    style: 'currency'.

    // The currency code is required when using style: 'currency'.
    currency: 'USD'
  });
  return format.formatRange(
    // Find the lowest price in the array.
    Math.min(...prices),

    // Find the highest price in the array.
    Math.max(...prices)
  );
}

// outputs '$1.75—$11.00'
console.log(
  formatPriceRange([5.5, 3, 1.75, 11, 9.5])
);
```

## 讨论

`Math.max`和`Math.min`函数接受多个参数，并从整组参数中返回最大值或最小值。 示例 11-11 使用数组展开语法将`prices`数组中的所有元素传递给`Math.max`和`Math.min`。

# 格式化测量单位

## 问题

您想要格式化一个带有测量单位的数字。

## 解决方案

在创建`Intl.NumberFormat`对象时使用`unit`样式，并指定目标单位。 示例 11-12 展示了如何格式化千兆字节的数字。

##### 示例 11-12\. 格式化千兆字节

```
const format = new Intl.NumberFormat(navigator.language, {
  style: 'unit',
  unit: 'gigabyte'
});

// prints "1,000 GB"
console.log(format.format(1000));
```

## 讨论

您还可以通过为`NumberFormat`指定`unitDisplay`选项来自定义单位标签。可能的值有：

`short`

显示缩写单位，用空格分隔：`1,000 GB`

`narrow`

显示缩写单位，无空格：`1,000GB`

`long`

显示完整的单位名称：`1,000 gigabytes`

# 应用复数规则

## 问题

您希望在引用不同数量的项目时使用正确的术语。例如，考虑用户列表。在英语中，您会说“一个用户”（单数），但“三个用户”（复数）。其他语言有更复杂的规则，您希望确保涵盖这些规则。

## 解决方案

使用`Intl.PluralRules`来选择正确的复数形式字符串。

首先，使用所需的区域设置构建一个`Intl.PluralRules`对象，并调用其`select`方法来确定用户数量（参见示例 11-13）。

##### 示例 11-13\. 确定复数形式

```
// An array containing the users
const users = getUsers();

const rules = new Intl.PluralRules('en-US');
const form = rules.select(users.length);
```

`select`方法根据要使用的复数形式和指定的区域设置返回一个字符串。对于`en-US`区域设置，它返回“one”（当用户计数为一时）或“other”（当用户计数不为一时）。您可以使用这些值作为键来定义消息，如示例 11-14 所示。

##### 示例 11-14\. 完整的复数规则解决方案

```
function formatUserCount(users) {
  // The variations of the message, depending
  // on the count
  const messages = {
    one: 'There is 1 user.',
    other: `There are ${users.length} users.`
  };

  // Use Intl.PluralRules to determine which message
  // should be displayed.
  const rules = new Intl.PluralRules('en-US');
  return messages[rules.select(users.length)];
}
```

## 讨论

此解决方案需要预先了解不同形式，以便定义正确的消息。

`Intl.PluralRules` 还支持一种 `ordinal` 模式，其工作方式略有不同。你可以使用此模式来格式化 *序数* 值，如 “1st,” “2nd,” “3rd,” 等。格式化规则因语言而异，你可以将它们映射到附加到数字的后缀。

例如，在 `en-US` 区域设置中，序数 `Intl.PluralRules` 返回如下值：

+   对于以 1 结尾的数字使用 `one` —“1st,” “21st,” 等。

+   对于以 2 结尾的数字使用 `two` —“2nd, 42nd,” 等。

+   对于以 3 结尾的数字使用 `few` —“3rd, “33rd,” 等。

+   对于其他数字使用 `other` —“5th,” “47th,” 等。

# 计算字符、单词和句子的数量

## 问题

你想使用特定于区域设置的规则计算字符串的字符数、单词数和句子数。

## 解决方案

使用 `Intl.Segmenter` 来分割字符串并计算出现次数。

你可以创建一个以字形（单个字符）、单词或句子为粒度的分段器。粒度决定了段的边界。每个分段器只能有一种粒度，所以你需要三个分段器（参见 示例 11-15）。

##### 示例 11-15\. 获取字符串的字符数、单词数和句子数

```
function getCounts(text) {
  const characters = new Intl.Segmenter(
    navigator.language,
    { granularity: 'grapheme' }
  );

  const words = new Intl.Segmenter(
    navigator.language,
    { granularity: 'word' }
  );

  const sentences = new Intl.Segmenter(
    navigator.language,
    { granularity: 'sentence' }
  );

  // Convert each segment to an array, then get its length.
  return {
    characters: [...characters.segment(text)].length,
    words: [...words.segment(text)].length,
    sentences: [...sentences.segment(text)].length
  };
}
```

###### 注意

此 API 可能尚未得到所有浏览器的支持。请查看 [CanIUse](https://oreil.ly/OL9G0) 获取最新的兼容性数据。

## 讨论

当你在一段文本上调用分段器的 `segment` 方法时，它会返回一个包含所有段的可迭代对象。有几种方法可以获取此可迭代对象中项目的长度，但是此示例使用了数组展开语法，它创建一个包含所有项的数组。然后你只需要获取每个数组的长度。

你可能过去通过使用字符串的 `split` 方法来解决这个问题。例如，你可以在空格上分割以获取单词数组并获取单词计数。这种方法在你的语言中可能有效，但是使用 `Intl.Segmenter` 的优势在于它采用给定区域设置的单词和句子分割规则。

# 格式化列表

## 问题

你有一个项目数组，想要以逗号分隔的列表形式显示。例如，一个用户数组显示为 “user1, user2, 和 user3。”

## 解决方案

使用 `Intl.ListFormat` 根据给定区域设置的规则将项目合并成列表。示例 11-16 使用用户数组，每个用户具有 `username` 属性。

##### 示例 11-16\. 格式化用户对象列表

```
function getUserListString(users, locale = 'en-US') {
  // The locale of the ListFormat is configurable.
  const listFormat = new Intl.ListFormat(locale);
  return listFormat.format(users.map(user => user.username));
}
```

## 讨论

`Intl.ListFormat` 根据需要添加单词和标点。例如，在 `en-US` 区域设置中，你会得到以下结果：

+   1 位用户：“user1”

+   2 位用户：“user1 和 user2”

+   3 位用户：“user1, user2, 和 user3”

下面是另一个使用 `de-DE` 区域设置的示例：

+   1 位用户：“user1”

+   2 位用户：“user1 und user2”

+   3 位用户：“user1, user2 und user3”

注意在第三种情况下使用 “und” 而不是 “and”，并且还要注意在 `en-US` 中没有在 user2 后面使用逗号的情况。这是因为德语语法不使用这个逗号（称为“牛津逗号”）。

正如你所看到的，使用`Intl.ListFormat`比使用数组的`join`方法更加健壮，后者当然没有考虑区域设置规则。

# 排序一个名字数组

## 问题

你有一个名字数组，想要使用特定于区域的排序规则进行排序。

## 解决方案

创建一个`Intl.Collator`提供比较逻辑，然后使用它的`compare`函数传递给`Array.prototype.sort`（参见示例 11-17）。此函数比较两个字符串。如果第一个字符串在第二个之前，则返回负值，如果相等则返回零，如果第一个字符串在第二个之后则返回正值。

##### 示例 11-17\. 使用`Intl.Collator`排序一个名字数组

```
const names = [
  'Elena',
  'Mário',
  'André',
  'Renée',
  'Léo',
  'Olga',
  'Héctor',
]

const collator = new Intl.Collator();
names.sort(collator.compare);
```

###### 注意

一个`Collator`可以返回任何负值或正值，不一定只是–1 或 1。

## 讨论

这是一种对字符串数组进行排序的简洁方式。在`Intl.Collator`之前，你可能会做类似于示例 11-18 的事情。

##### 示例 11-18\. 直接排序一个字符串数组

```
names.sort((a, b) => a.localeCompare(b));
```

这种方法运行良好，但一个主要的区别在于，当比较字符串时，你无法指定要应用的区域设置的排序规则。另一个`Intl.Collator`的好处是它的灵活性。你可以微调它用来比较字符串的逻辑。

例如，考虑数组`[1, 2, 20, 3]`。使用默认排序器，这将是排序后的顺序，因为它使用字符串比较逻辑。你可以传递`numeric: true`选项给`Intl.Collator`，然后排序后的数组变成`[1, 2, 3, 20]`。
