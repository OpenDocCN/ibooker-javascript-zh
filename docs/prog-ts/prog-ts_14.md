# 附录 A. 类型操作符

TypeScript 支持丰富的类型操作符，用于处理类型。想要了解更多关于操作符的信息，请使用表格 A-1 作为方便的参考。

表格 A-1. 类型操作符

| 类型操作符 | 语法 | 在哪里使用 | 了解更多 |
| --- | --- | --- | --- |
| 类型查询 | `typeof`, `instanceof` | 任何类型 | “细化”, “类声明值和类型” |
| 键 | `keyof` | 对象类型 | “keyof 操作符” |
| 属性查找 | `O[K]` | 对象类型 | “键入操作符” |
| 映射类型 | `[K in O]` | 对象类型 | “映射类型” |
| 修饰符加法 | `+` | 对象类型 | “映射类型” |
| 修饰符减法 | `-` | 对象类型 | “映射类型” |
| 只读修饰符 | `readonly` | 对象类型，数组类型，元组类型 | “对象”, “类和继承”, “只读数组和元组” |
| 可选修饰符 | `?` | 对象类型，元组类型，函数参数类型 | “对象”, “元组”, “可选和默认参数” |
| 条件类型 | `?` | 泛型类型，类型别名，函数参数类型 | “条件类型” |
| 非`null`断言 | `!` | 可空类型 | “非空断言”, “明确赋值断言” |
| 泛型类型参数默认值 | `=` | 泛型类型 | “泛型类型默认值” |
| 类型断言 | `as`, `<>` | 任何类型 | “类型断言”, “const 类型” |
| 类型守卫 | `is` | 函数返回类型 | “用户定义的类型守卫” |
