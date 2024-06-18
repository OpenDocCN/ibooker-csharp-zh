# 第二十四章：Lambda 表达式

*Lambda 表达式*是在委托实例的位置上写的无名方法。编译器立即将 lambda 表达式转换为以下之一：

+   一个委托实例。

+   *表达式树*，类型为`Expression<TDelegate>`，表示 lambda 表达式内部的代码为可遍历的对象模型。这允许稍后在运行时解释 lambda 表达式（我们在《C# 12 精要》的第八章中描述了这个过程）。

在以下示例中，`x => x * x`是一个 lambda 表达式：

```cs
Transformer sqr = x => x * x;
Console.WriteLine (sqr(3));    // 9

delegate int Transformer (int i);
```

###### 注意

在内部，编译器通过编写一个私有方法并将表达式的代码移动到该方法中来解析这种类型的 lambda 表达式。

Lambda 表达式具有以下形式：

```cs
(*parameters*) => *expression-or-statement-block*
```

为了方便起见，如果且仅当有一个可推断类型的参数时，可以省略括号。

在我们的示例中，有一个参数`x`，表达式是`x * x`：

```cs
x => x * x;
```

Lambda 表达式的每个参数对应于一个委托参数，并且表达式的类型（可以是`void`）对应于委托的返回类型。

在我们的示例中，`x`对应于参数`i`，表达式`x * x`对应于返回类型`int`，因此与`Transformer`委托兼容。

Lambda 表达式的代码可以是*语句块*而不是表达式。我们可以将我们的示例重写如下：

```cs
x => { return x * x; };
```

Lambda 表达式最常与`Func`和`Action`委托一起使用，因此你会经常看到我们之前的表达式写成如下形式：

```cs
Func<int,int> sqr = x => x * x;
```

编译器通常可以*推断*lambda 参数的类型。当情况不是这样时，可以显式指定参数类型：

```cs
Func<int,int> sqr = (int x) => x * x;
```

这是一个接受两个参数的表达式示例：

```cs
Func<string,string,int> totalLength = 
 (s1, s2) => s1.Length + s2.Length;

int total = totalLength ("hello", "world");  // total=10;
```

假设`Clicked`是`EventHandler`类型的事件，以下通过 lambda 表达式附加事件处理程序：

```cs
obj.Clicked += (sender,args) => Console.Write ("Click");
```

这是一个接受零个参数的表达式示例：

```cs
Func<string> greeter = () => "Hello, world";
```

从 C# 10 开始，编译器允许使用可以通过`Func`和`Action`委托解析的 lambda 表达式进行隐式类型推断，因此我们可以将此语句简化为：

```cs
var greeter = () => "Hello, world";
```

如果 lambda 表达式有参数，则必须指定它们的类型才能使用 `var`：

```cs
var sqr = (int x) => x * x;
```

编译器推断 `sqr` 的类型为 `Func<int,int>`。

## 默认 Lambda 参数（C# 12）

就像普通方法可以有可选参数一样：

```cs
void Print (string info = "") => Console.Write (info);
```

因此，lambda 表达式也可以：

```cs
var print = (string info = "") => Console.Write (info);

print ("Hello");
print ();
```

此功能对于像 ASP.NET Minimal API 这样的库非常有用。

## 捕获外部变量

Lambda 表达式可以引用在其定义的地方可访问的任何变量。这些称为*外部变量*，可以包括局部变量、参数和字段：

```cs
int factor = 2;
Func<int, int> multiplier = n => n * factor;
Console.WriteLine (multiplier (3));           // 6
```

被 lambda 表达式引用的外部变量称为*捕获变量*。捕获变量在实际*调用*委托时进行评估，而不是在捕获变量时：

```cs
int factor = 2;
Func<int, int> multiplier = n => n * factor;
factor = 10;
Console.WriteLine (multiplier (3));           // 30
```

Lambda 表达式本身可以更新捕获的变量：

```cs
int seed = 0;
Func<int> natural = () => seed++;
Console.WriteLine (natural());           // 0
Console.WriteLine (natural());           // 1
Console.WriteLine (seed);                // 2
```

捕获变量的生命周期延长到委托的生命周期。在以下示例中，局部变量 `seed` 在 `Natural` 执行完成时通常会从作用域中消失。但因为 `seed` 被*捕获*，其生命周期延长到捕获委托 `natural` 的生命周期：

```cs
Func<int> natural = Natural();
Console.WriteLine (natural());      // 0
Console.WriteLine (natural());      // 1

static Func<int> Natural()
{
  int seed = 0;
  return () => seed++;      // Returns a *closure*
}
```

变量也可以被匿名方法和本地方法捕获。在这些情况下，捕获变量的规则是相同的。

### 静态 lambda

从 C# 9 开始，您可以通过应用 `static` 关键字来确保 lambda 表达式、本地方法或匿名方法不会捕获状态。这在微优化场景中很有用，以防止（可能是无意的）闭包内存分配和清理。例如，我们可以将 static 修饰符应用于 lambda 表达式如下：

```cs
Func<int, int> multiplier = static n => n * 2;
```

如果后来试图修改 lambda 表达式，以便捕获一个局部变量，编译器将会生成一个错误。这个特性在本地方法中更有用（因为 lambda 表达式本身会引起内存分配）。在以下示例中，`Multiply` 方法无法访问 `factor` 变量：

```cs
void Foo()
{
  int factor = 123;
 static int Multiply (int x) => x * 2;
}
```

在此应用 `static` 也可以作为文档工具，指示减少耦合的程度。静态 lambda 仍然可以访问静态变量和常量（因为这些不需要闭包）。

###### 注意

`static` 关键字仅作为*检查*；它不影响编译器生成的 IL。如果没有 `static` 关键字，编译器不会生成闭包，除非需要（即使如此，它也有技巧来减少成本）。

### 捕获迭代变量

当在 `for` 循环中捕获迭代变量时，C# 将迭代变量视为在循环*外部*声明。这意味着每次迭代中都会捕获*同一个*变量。以下程序输出 `333` 而不是 `012`：

```cs
Action[] actions = new Action[3];

for (int i = 0; i < 3; i++)
  actions [i] = () => Console.Write (i);

foreach (Action a in actions) a();     // 333
```

每个闭包（以粗体显示）捕获同一个变量 `i`。（这实际上在考虑到 `i` 是一个在循环迭代之间保持其值的变量时是有道理的；你甚至可以在循环体内显式更改 `i` 的值。）其结果是，当稍后调用委托时，每个委托都会看到调用时 `i` 的值—这是 3\. 如果我们想写 `012`，解决方法是将迭代变量分配给循环内部范围的本地变量：

```cs
Action[] actions = new Action[3];
for (int i = 0; i < 3; i++)
{
  int loopScopedi = i;
  actions [i] = () => Console.Write (loopScopedi);
}
foreach (Action a in actions) a();     // 012
```

这会导致闭包在每次迭代时捕获*不同*的变量。

请注意（从 C# 5 开始），`foreach` 循环中的迭代变量是隐式局部的，因此可以安全地在其上进行闭包，而无需临时变量。

## Lambda 表达式与本地方法对比

本地方法（参见“本地方法”）的功能与 Lambda 表达式重叠。本地方法的优点在于允许递归并避免指定委托的混乱。避免委托的间接引用也使它们稍微更有效，并且它们可以访问包含方法的局部变量，而无需编译器将捕获的变量提升到隐藏类中。

然而，在许多情况下，你*需要*一个委托，最常见的情况是调用高阶函数（即，带有委托类型参数的方法）：

```cs
public void Foo (Func<int,bool> predicate) { ... }
```

在这种情况下，你无论如何都需要一个委托，而恰恰在这些情况下，Lambda 表达式通常更简洁、更清晰。

