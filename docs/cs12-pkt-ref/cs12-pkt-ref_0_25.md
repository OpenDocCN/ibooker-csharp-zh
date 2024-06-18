# 第二十五章：匿名方法

匿名方法是 C# 2.0 的一个特性，大多数情况下已被 Lambda 表达式取代。匿名方法类似于 Lambda 表达式，但缺少隐式类型化的参数、表达式语法（匿名方法必须始终是一个语句块）以及编译为表达式树的能力。要编写匿名方法，你需要使用 `delegate` 关键字，后面（可选）是参数声明，然后是方法体。例如：

```cs
Transformer sqr = delegate (int x) {return x * x;};
Console.WriteLine (sqr(3));         // 9

delegate int Transformer (int i);
```

第一行在语义上等同于以下 Lambda 表达式：

```cs
Transformer sqr =       (int x) => {return x * x;};
```

或者简单地：

```cs
Transformer sqr =            x  => x * x;
```

匿名方法的一个独特特性是，你可以完全省略参数声明—即使委托需要它。这在声明带有默认空处理程序的事件时非常有用：

```cs
public event EventHandler Clicked = delegate { };
```

这样可以避免在触发事件前进行空检查。以下也是合法的（注意没有参数）：

```cs
Clicked += delegate { Console.Write ("clicked"); };
```

匿名方法像 Lambda 表达式一样捕获外部变量。

