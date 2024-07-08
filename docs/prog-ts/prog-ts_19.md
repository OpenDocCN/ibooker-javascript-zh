# 附录 F. TSC 编译器安全标志

###### Tip

要获取完整的编译器标志列表，请访问 [TypeScript Handbook website](http://bit.ly/2JWfsgY)。

每个 TypeScript 发布版本都引入了新的检查，您可以启用这些检查以使代码更加安全。其中一些标志以`strict`为前缀，作为`strict`标志的一部分包含在内；或者，您可以逐个选择加入`strict`标志。表格 F-1 列出了与安全性相关的编译器标志（截至撰写本文时）。

表 F-1\. TSC 安全标志

| Flag | Description |
| --- | --- |
| `alwaysStrict` | 发出 `'use strict'`。 |
| `noEmitOnError` | 当代码存在类型错误时不生成 JavaScript。 |
| `noFallthroughCasesInSwitch` | 确保每个`switch`分支要么返回一个值，要么中断。 |
| `noImplicitAny` | 当变量类型被推断为`any`时报错。 |
| `noImplicitReturns` | 确保每个函数中的每条代码路径都显式返回。参见 “Totality”。 |
| `noImplicitThis` | 在函数中使用`this`而未显式注释`this`类型时报错。参见 “Typing this”。 |
| `noUnusedLocals` | 警告未使用的局部变量。 |
| `noUnusedParameters` | 警告未使用的函数参数。使用 `_` 前缀忽略此错误。 |
| `strictBindCallApply` | 强制执行`bind`、`call`和`apply`的类型安全性。参见 “call, apply, and bind”。 |
| `strictFunctionTypes` | 强制函数在其参数和`this`类型上是逆变的。参见 “Function variance”。 |
| `strictNullChecks` | 将`null`提升为一种类型。参见 “null, undefined, void, and never”。 |
| `strictPropertyInitialization` | 强制类属性要么是可空的，要么被初始化。参见 第五章。 |
