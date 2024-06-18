# 第二十二章：委托

*委托*将方法调用者在运行时连接到其目标方法。委托有两个方面：*类型*和*实例*。*委托类型*定义了调用者和目标将符合的*协议*，包括参数类型列表和返回类型。*委托实例*是指向符合该协议的一个（或多个）目标方法的对象。

委托实例确实充当调用者的代表：调用者调用委托，然后委托调用目标方法。这种间接性将调用者与目标方法解耦。

委托类型声明之前使用关键字`delegate`，但在其他方面类似于（抽象）方法声明。例如：

```cs
delegate int Transformer (int x);
```

要创建委托实例，可以将方法分配给委托变量：

```cs
Transformer t = Square;  // Create delegate instance
int result = t(3);       // Invoke delegate
Console.Write (result);  // 9

int Square (int x) => x * x;
```

调用委托就像调用方法一样（因为委托的目的仅是提供一级间接）：

```cs
t(3);
```

语句`Transformer t = Square`的速记法如下：

```cs
Transformer t = new Transformer (Square);
```

而`t(3)`是以下的速记法：

```cs
t.Invoke (3);
```

委托类似于*回调*，是一个通用术语，涵盖了诸如 C 函数指针之类的结构。

## 使用委托编写插件方法

委托变量在运行时分配一个方法。这对编写插件方法非常有用。在本示例中，我们有一个名为`Transform`的实用方法，它对整数数组中的每个元素应用变换。`Transform`方法具有一个委托参数，用于指定插件变换：

```cs
int[] values = { 1, 2, 3 };
Transform (values, Square);  // Hook in the Square method

foreach (int i in values)
  Console.Write (i + "  ");  // 1   4   9

void Transform (int[] values, Transformer t)
{
  for (int i = 0; i < values.Length; i++)
    values[i] = t (values[i]);
}

int Square (int x) => x * x;

delegate int Transformer (int x);
```

## 实例方法和静态方法的目标

委托的目标方法可以是本地方法、静态方法或实例方法。

当将*实例*方法分配给委托对象时，后者必须保持对方法和方法所属*实例*的引用。`System.Delegate`类的`Target`属性表示此实例（对于引用静态方法的委托，此属性将为 null）。

## 多路委托

所有委托实例都具有*多路*能力。这意味着委托实例不仅可以引用单个目标方法，还可以引用目标方法列表。`+`和`+=`运算符结合委托实例。例如：

```cs
SomeDelegate d = SomeMethod1;
d += SomeMethod2;
```

最后一行在功能上与这一行相同：

```cs
d = d + SomeMethod2;
```

调用`d`现在将调用`SomeMethod1`和`SomeMethod2`。委托按添加顺序调用。

`-`和`-=`运算符从左委托操作数中移除右委托操作数。例如：

```cs
d -= SomeMethod1;
```

调用`d`现在只会调用`SomeMethod2`。

在委托变量上调用`+`或`+=`并具有`null`值是合法的，就像在具有单个目标的委托变量上调用`-=`一样（这将导致委托实例为`null`）。

###### 注意

委托是*不可变的*，因此当您调用`+=`或`-=`时，实际上是创建一个*新*的委托实例并将其分配给现有变量。

如果多路委托具有非`void`返回类型，则调用者将接收最后一个调用的方法的返回值。之前的方法仍会被调用，但它们的返回值会被丢弃。在大多数使用多路委托的场景中，它们具有`void`返回类型，因此不会出现此微妙之处。

所有委托类型都隐式从`System.MulticastDelegate`派生，后者继承自`System.Delegate`。C#将委托上的`+`、`-`、`+=`和`-=`操作编译为`System.Delegate`类的静态`Combine`和`Remove`方法。

## 泛型委托类型

委托类型可以包含泛型类型参数。例如：

```cs
public delegate T Transformer<T> (T arg);
```

这是我们如何使用此委托类型的方式：

```cs
Transformer<double> s = Square;
Console.WriteLine (s (3.3));        // 10.89

double Square (double x) => x * x;
```

## Func 和 Action 委托

使用泛型委托，可以编写一小组非常通用的委托类型，这些委托类型可以适用于任何返回类型和任何（合理的）参数数量的方法。这些委托是在`System`命名空间中定义的`Func`和`Action`委托（`in`和`out`注释指示*变化*，我们很快将在委托上下文中涵盖它）：

```cs
delegate TResult Func <out TResult> ();
delegate TResult Func <in T, out TResult> (T arg);
delegate TResult Func <in T1, in T2, out TResult>
 (T1 arg1, T2 arg2);
*... and so on, up to T16*

delegate void Action ();
delegate void Action <in T> (T arg);
delegate void Action <in T1, in T2> (T1 arg1, T2 arg2);
*... and so on, up to T16*
```

这些委托非常通用。在我们之前的示例中，`Transformer`委托可以替换为一个接受类型为`T`的单一参数并返回相同类型值的`Func`委托：

```cs
public static void Transform<T> (
  T[] values, Func<T,T> transformer)
{
  for (int i = 0; i < values.Length; i++)
    values[i] = transformer (values[i]);
}
```

这些委托未涵盖的唯一实际场景是`ref`/`out`和指针参数。

## 委托兼容性

委托类型彼此都是不兼容的，即使它们的签名相同：

```cs
delegate void D1(); delegate void D2();
...
D1 d1 = Method1;
D2 d2 = d1;            // Compile-time error
```

然而，下面是允许的：

```cs
D2 d2 = new D2 (d1);
```

如果委托实例具有相同的类型和方法目标，则它们被视为相等。对于多播委托，方法目标的顺序是重要的。

### 返回类型变异

当您调用一个方法时，您可能会得到比您请求的更具体的类型。这是普通的多态行为。保持一致，委托目标方法可能返回比委托描述的更具体的类型。这称为*协变*：

```cs
ObjectRetriever o = new ObjectRetriever (RetrieveString);
object result = o();
Console.WriteLine (result);      // hello

string RetrieveString() => "hello";

delegate object ObjectRetriever();
```

`ObjectRetriever`期望返回一个`object`，但是`object`的*子类*也可以，因为委托的返回类型是*协变*。

### 参数变异

当您调用一个方法时，您可以提供比该方法参数更具体的参数。这是普通的多态行为。保持一致，委托目标方法的参数类型可能比委托描述的要*更少*具体。这称为*逆变*：

```cs
StringAction sa = new StringAction (ActOnObject);
sa ("hello");

void ActOnObject (object o) => Console.WriteLine (o);

delegate void StringAction (string s);
```

###### 注意

标准事件模式旨在通过其对共同`EventArgs`基类的使用帮助您利用委托参数的逆变性。例如，您可以有一个单一方法由两个不同的委托调用，一个传递`MouseEventArgs`，另一个传递`KeyEventArgs`。

### 泛型委托的类型参数变异

我们在“泛型”中看到了如何对泛型接口的类型参数进行协变和逆变。相同的能力也适用于泛型委托。如果您正在定义一个泛型委托类型，最好按照以下做法：

+   将仅用于返回值的类型参数标记为协变的（`out`）

+   将仅用于参数的类型参数标记为逆变的（`in`）

执行此操作允许通过尊重类型之间的继承关系自然进行转换。下面的委托（定义在`System`命名空间中）对于`TResult`是协变的：

```cs
delegate TResult Func<out TResult>();
```

这允许：

```cs
Func<string> x = ...;
Func<object> y = x;
```

下面的委托（定义在`System`命名空间中）对于`T:`是逆变的：

```cs
delegate void Action<in T> (T arg);
```

这允许：

```cs
Action<object> x = ...;
Action<string> y = x;
```

