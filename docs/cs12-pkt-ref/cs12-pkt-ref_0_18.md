# 第十八章：接口

*接口*类似于类，但仅*指定行为*而不保存状态（数据）。因此：

+   接口成员都是*隐式抽象*的。（有一些例外情况，我们将在“默认接口成员”和“静态接口成员”中描述。）

+   类（或结构）可以实现*多个*接口。相比之下，类只能从*单个*类继承，结构不能继承（除了从`System.ValueType`派生）。

接口声明类似于类声明，但它（通常）不为其成员提供实现，因为其所有成员都是隐式抽象的。这些成员将由实现接口的类和结构实现。接口只能包含方法、属性、事件和索引器，这不是巧合，这些成员恰好是类的成员可以是抽象的。

这是`System.Collections`中定义的`IEnumerator`接口的稍微简化版本：

```cs
public interface IEnumerator
{
  bool MoveNext();
  object Current { get; }
  void Reset();
}
```

接口成员始终隐式公共，并且不能声明访问修饰符。实现接口意味着为其所有成员提供一个`public`实现：

```cs
internal class Countdown : IEnumerator
{
  int count = 6;
  public bool MoveNext()  => count-- > 0 ;
  public object Current   => count;
  public void Reset()     => count = 6;
}
```

您可以将对象隐式转换为其实现的任何接口：

```cs
IEnumerator e = new Countdown();
while (e.MoveNext())
  Console.Write (e.Current + " ");  // 5 4 3 2 1 0
```

## 扩展一个接口

接口可以派生自其他接口。例如：

```cs
public interface IUndoable             { void Undo(); }
public interface IRedoable : IUndoable { void Redo(); }
```

`IRedoable`“继承”了`IUndoable`的所有成员。

## 显式接口实现

实现多个接口有时会导致成员签名冲突。您可以通过*显式实现*接口成员来解决这种冲突。例如：

```cs
interface I1 { void Foo(); }
interface I2 { int Foo();  }

public class Widget : I1, I2
{
  public void Foo()   // Implicit implementation
  {
    Console.Write ("Widget's implementation of I1.Foo");
  }

  int I2.Foo()   // Explicit implementation of I2.Foo
  {
    Console.Write ("Widget's implementation of I2.Foo");
    return 42;
  }
}
```

因为`I1`和`I2`都具有冲突的`Foo`签名，所以`Widget`显式实现了`I2`的`Foo`方法。这样可以让这两个方法在同一个类中共存。调用显式实现的成员的唯一方法是将其转换为其接口：

```cs
Widget w = new Widget();
w.Foo();           // Widget's implementation of I1.Foo
((I1)w).Foo();     // Widget's implementation of I1.Foo
((I2)w).Foo();     // Widget's implementation of I2.Foo
```

显式实现接口成员的另一个原因是隐藏高度专业化且对类型正常使用案例有干扰的成员。例如，实现`ISerializable`的类型通常希望避免在未明确转换为该接口时展示其`ISerializable`成员。

## 虚拟实现接口成员

隐式实现的接口成员默认为密封的。必须在基类中将其标记为`virtual`或`abstract`以便重写：通过基类或接口调用接口成员然后调用子类的实现。

显式实现的接口成员不能标记为`virtual`，也不能以通常的方式重写。但是可以*重新实现*。

## 在子类中重新实现接口

子类可以*重新实现*任何已由基类实现的接口成员。重新实现会劫持成员实现（通过接口调用时），无论成员在基类中是否为`virtual`都有效。

在以下示例中，`TextBox`显式实现了`IUndoable.Undo`，因此不能标记为`virtual`。要“覆盖”它，`RichTextBox`必须重新实现`IUndoable`的`Undo`方法：

```cs
public interface IUndoable { void Undo(); }

public class TextBox : IUndoable
{
  void IUndoable.Undo()
    => Console.WriteLine ("TextBox.Undo");
}

public class RichTextBox : TextBox, IUndoable
{
  public new void Undo()
    => Console.WriteLine ("RichTextBox.Undo");
}
```

通过接口调用重新实现的成员会调用子类的实现：

```cs
RichTextBox r = new RichTextBox();
r.Undo();                 // RichTextBox.Undo
((IUndoable)r).Undo();    // RichTextBox.Undo
```

在这种情况下，`TextBox`显式实现了`Undo`方法。如果`TextBox`改为隐式实现`Undo`方法，则`RichTextBox`仍然可以重新实现该方法，但在通过基类调用成员时，效果不会普遍：

```cs
RichTextBox r = new RichTextBox();
((TextBox)r).Undo();      // TextBox.Undo
```

## 默认接口成员

从 C# 8 开始，您可以为接口成员添加默认实现，使其成为可选实现：

```cs
interface ILogger
{
  void Log (string text) => Console.WriteLine (text);
}
```

如果您希望在流行库中定义的接口中添加成员而不破坏（可能有数千个）实现，则这是有利的。

默认实现始终是显式的，因此如果实现`ILogger`的类未定义`Log`方法，则唯一调用它的方法是通过接口：

```cs
class Logger : ILogger { }
...
((ILogger)new Logger()).Log ("message");
```

这样可以防止多重实现继承的问题：如果将同一个默认成员添加到一个类实现的两个接口中，那么调用成员时不会存在歧义。

## 静态接口成员

接口还可以声明静态成员。有两种类型的静态接口成员：

+   静态非虚拟接口成员

+   静态虚拟/抽象接口成员

### 静态非虚拟接口成员

静态非虚拟接口成员主要用于帮助编写默认接口成员。它们不由类或结构体实现；而是直接使用。除了方法、属性、事件和索引器之外，静态非虚拟成员还允许字段，这些字段通常从默认成员实现中的代码访问：

```cs
interface ILogger
{
  void Log (string text) => 
    Console.WriteLine (Prefix + text);

  static string Prefix = ""; 
}
```

静态非虚拟接口成员默认为公共，因此可以从外部访问：

```cs
ILogger.Prefix = "File log: ";
```

您可以通过添加访问修饰符（例如`private`、`protected`或`internal`）来限制此功能。

实例字段（仍然）被禁止。这符合接口的原则，即定义*行为*而非*状态*。

### 静态虚拟/抽象接口成员

静态虚拟/抽象接口成员（来自 C# 11）标记为`static abstract`或`static virtual`，并启用*静态多态性*，这是我们将在“静态多态性”中全面讨论的高级特性：

```cs
interface ITypeDescribable
{
  static abstract string Description { get; }
  static virtual string Category => null;
}
```

实现类或结构必须实现静态抽象成员，并可选择实现静态虚拟成员：

```cs
class CustomerTest : ITypeDescribable
{
  public static string Description => "Customer tests";
  public static string Category    => "Unit testing";
}
```

除了方法、属性和事件之外，运算符和转换也是静态虚拟接口成员的合法目标（参见“运算符重载”）。静态虚拟接口成员通过约束的类型参数调用；我们将在“静态多态性”和“泛型数学”中演示这一点，之后会讨论泛型。

