# 第九章：表达式和运算符

一个*表达式*本质上表示一个值。最简单的表达式是常量（如`123`）和变量（如`x`）。表达式可以通过运算符进行转换和组合。*运算符*接受一个或多个输入*操作数*来输出一个新的表达式：

```cs
12 * 30   // * is an operator; 12 and 30 are operands.
```

复杂的表达式可以被构建，因为操作数本身可以是一个表达式，例如下面例子中的操作数`(12 * 30)`：

```cs
1 + (12 * 30)
```

在 C#中，运算符可以分为*一元*、*二元*或*三元*，取决于它们操作的操作数数量（一个、两个或三个）。二元运算符总是使用*中缀*表示法，其中运算符被放置*在*两个操作数*之间*。

对于语言基本结构中的内置运算符，称为*主要运算符*；方法调用运算符就是一个例子。一个没有值的表达式称为*空表达式*：

```cs
Console.WriteLine (1)
```

因为空表达式没有值，所以你不能将其用作构建更复杂表达式的操作数：

```cs
1 + Console.WriteLine (1)      // Compile-time error
```

## 赋值表达式

一个*赋值表达式*使用`=`运算符将另一个表达式的结果赋给一个变量。例如：

```cs
x = x * 5
```

赋值表达式不是一个空表达式。它实际上携带了赋值的值，因此可以并入到另一个表达式中。在下面的例子中，表达式将`2`赋给了`x`，`10`赋给了`y`：

```cs
y = 5 * (x = 2)
```

这种表达方式可以用来初始化多个值：

```cs
a = b = c = d = 0
```

*复合赋值运算符*是将赋值与另一个运算符结合的语法快捷方式。例如：

```cs
x *= 2    // equivalent to x = x * 2
x <<= 1   // equivalent to x = x << 1
```

（这条规则的一个微妙的例外是*事件*，我们稍后描述：这里的`+=`和`-=`运算符被特殊对待，并映射到事件的`add`和`remove`访问器。）

## 运算符优先级和结合性

当一个表达式包含多个运算符时，*优先级*和*结合性*决定了它们的求值顺序。优先级较高的运算符比优先级较低的运算符先执行。如果运算符具有相同的优先级，则运算符的结合性决定了求值的顺序。

### 优先级

表达式 `1 + 2 * 3` 会被评估为 `1 + (2 * 3)`，因为 `*` 的优先级高于 `+`。

### 左关联运算符

二进制运算符（赋值、lambda 和 null-coalescing 运算符除外）是*左关联*的；换句话说，它们从左到右进行评估。例如，表达式 `8/4/2` 会被评估为 `(8/4)/2`。

### 右关联运算符

赋值、lambda 运算符、null-coalescing 运算符和（三元）条件运算符是*右关联*的；换句话说，它们从右到左进行评估。右关联允许多个赋值，例如 `x=y=3` 可以编译：它首先将 `3` 赋给 `y`，然后将该表达式的结果 (`3`) 赋给 `x`。

## 操作符表

下表按照 C# 操作符的优先级排序。在同一子标题下列出的操作符具有相同的优先级。我们在 “操作符重载” 部分解释用户可重载的操作符。

| 操作符符号 | 操作符名称 | 示例 | 可重载 |  |
| --- | --- | --- | --- | --- |
| **主要（最高优先级）** |  |
| `.` | 成员访问 | `x.y` | 否 |  |
| `?.` | 空值条件 | `x?.y` | 否 |  |
| `!`（后缀） | 空值容错 | `x!.y` | 否 |  |
| `->` | 指向结构体的指针（不安全） | `x->y` | 否 |  |
| `()` | 函数调用 | `x()` | 否 |  |
| `[]` | 数组/索引 | `a[x]` | 通过索引器 |  |
| `++` | 后增量 | `x++` | 是 |  |
| `--` | 后减量 | `x--` | 是 |  |
| `new` | 创建实例 | `new Foo()` | 否 |  |
| `stackalloc` | 堆栈分配 | `stackalloc(10)` | 否 |  |
| `typeof` | 根据标识符获取类型 | `typeof(int)` | 否 |  |
| `nameof` | 获取标识符名称 | `nameof(x)` | 否 |  |
| `checked` | 整数溢出检查开启 | `checked(x)` | 否 |  |
| `unchecked` | 整数溢出检查关闭 | `unchecked(x)` | 否 |  |
| `default` | 默认值 | `default(char)` | 否 |  |
| `sizeof` | 获取结构体大小 | `sizeof(int)` | 否 |  |
| **一元** |  |
| `await` | 等待 | `await myTask` | 否 |  |
| `+` | 正值 | `+x` | 是 |  |
| `-` | 负值 | `-x` | 是 |  |
| `!` | 非 | `!x` | 是 |  |
| `~` | 按位取反 | `~x` | 是 |  |
| `++` | 前增量 | `++x` | 是 |  |
| `--` | 前减量 | `--x` | 是 |  |
| `()` | 类型转换 | `(int)x` | 否 |  |
| `^` | 从末尾索引 | `array[¹]` | 否 |  |
| `*` | 地址处的值（不安全） | `*x` | 否 |  |
| `&` | 值的地址（不安全） | `&x` | 否 |  |
| **范围** |  |
| `..` `..^` | 索引范围 | `x..y` `x..^y` | 否 |  |
| **Switch 和 with** |  |

| `switch` | Switch 表达式 | `num switch {` `1 => true,`

`_ => false`

`}` | 否 |  |

| `with` | With 表达式 | `rec with` `{ X = 123 }` | 否 |  |
| --- | --- | --- | --- | --- |
| **乘法** |  |
| `*` | 乘法 | `x * y` | 是 |  |
| `/` | 除法 | `x / y` | 是 |  |
| `%` | 取余 | `x % y` | 是 |  |
| **加法** |  |
| `+` | 加法 | `x + y` | 是 |  |
| `-` | 减法 | `x - y` | 是 |  |
| **移位** |  |
| `<<` | 左移 | `x << 1` | 是 |  |
| `>>` | 右移 | `x >> 1` | 是 |  |
| `>>>` | 无符号右移 | `x >>> 1` | 是 |  |
| **关系** |  |
| `<` | 小于 | `x < y` | 是 |  |
| `>` | 大于 | `x > y` | 是 |  |
| `<=` | 小于或等于 | `x <= y` | 是 |  |
| `>=` | 大于或等于 | `x >= y` | 是 |  |
| `is` | 类型为或类型为子类 | `x is y` | 否 |  |
| `as` | 类型转换 | `x as y` | 否 |  |
| **相等性** |  |
| `==` | 等于 | `x == y` | 是 |  |
| `!=` | 不等于 | `x != y` | 是 |  |
| **按位与** |  |
| `&` | 与 | `x & y` | 是 |  |
| **按位异或** |  |
| `^` | 异或 | `x ^ y` | 是 |  |
| **按位或** |  |
| `&#124;` | 或 | `x &#124; y` | 是 |  |
| **条件与** |  |
| `&&` | 条件与 | `x && y` | 通过 `&` |  |
| **条件或** |  |
| `&#124;&#124;` | 条件或 | `x &#124;&#124; y` | 通过 `&#124;` |  |
| **空值合并** |  |
| `??` | 空值合并 | `x ?? y` | 否 |  |
| **条件（三元）** |  |
| `? :` | 条件 | `isTrue ? thenThis : elseThis` | 否 |  |
| **赋值和 lambda（最低优先级）** |  |
| `=` | 赋值 | `x = y` | 否 |  |
| `*=` | 自身乘法 | `x *= 2` | 通过 `*` |  |
| `/=` | 自身除法 | `x /= 2` | 通过 `/` |  |
| `+=` | 自身加法 | `x += 2` | 通过 `+` |  |
| `-=` | 自身减法 | `x -= 2` | 通过 `-` |  |
| `<<=` | 自身左移 | `x <<= 2` | 通过 `<<` |  |
| `>>=` | 自身右移 | `x >>= 2` | 通过 `>>` |  |
| `>>>=` | 无符号自身右移 | `x >>>= 2` | 通过 `>>>` |
| `&=` | 自身与 | `x &= 2` | 通过 `&` |  |
| `^=` | 自身异或 | `x ^= 2` | 通过 `^` |  |
| `&#124;=` | 自身或 | `x &#124;= 2` | 通过 `&#124;` |  |
| `=>` | Lambda | `x => x + 1` | 否 |  |

