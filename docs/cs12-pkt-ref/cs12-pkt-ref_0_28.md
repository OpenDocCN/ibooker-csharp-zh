# 第二十八章：可空值类型

引用类型可以用空引用表示不存在的值。然而，值类型通常不能表示 null 值。例如：

```cs
string s = null;   // OK - reference type
int i = null;      // Compile error - int cannot be null
```

要在值类型中表示 null，必须使用称为*可空类型*的特殊构造。可空类型用值类型后跟`?`符号表示：

```cs
int? i = null;                     // OK - nullable type
Console.WriteLine (i == null);     // True
```

## Nullable<T> 结构体

`T?`翻译成`System.Nullable<T>`。`Nullable<T>`是一个轻量级的不可变结构，只有两个字段，用于表示`Value`和`HasValue`。`System.Nullable<T>`的本质非常简单：

```cs
public struct Nullable<T> where T : struct
{
  public T Value {get;}
  public bool HasValue {get;}
  public T GetValueOrDefault();
  public T GetValueOrDefault (T defaultValue);
  ...
}
```

以下代码：

```cs
int? i = null;
Console.WriteLine (i == null);              // True
```

翻译为：

```cs
Nullable<int> i = new Nullable<int>();
Console.WriteLine (! i.HasValue);           // True
```

当`HasValue`为 false 时尝试检索`Value`会引发`InvalidOperationException`。`GetValueOrDefault()`在`HasValue`为 true 时返回`Value`；否则返回`new T()`或指定的自定义默认值。

`T?`的默认值是`null`。

## 可空类型转换

从`T`到`T?`的转换是隐式的，而从`T?`到`T`的转换是显式的。例如：

```cs
int? x = 5;        // implicit
int y = (int)x;    // explicit
```

显式转换直接等同于调用可空对象的`Value`属性。因此，如果`HasValue`为 false，则会引发`InvalidOperationException`。

## 对可空值类型进行装箱/拆箱

当`T?`被装箱时，堆上的装箱值包含`T`，而不是`T?`。这种优化是可能的，因为装箱值是一个可以表示 null 的引用类型。

C#也允许使用`as`运算符对可空类型进行拆箱。如果转换失败，则结果将为`null`：

```cs
object o = "string";
int? x = o as int?;
Console.WriteLine (x.HasValue);   // False
```

## 运算符提升

`Nullable<T>`结构体并未定义诸如`<`、`>`或甚至`==`之类的运算符。尽管如此，以下代码编译并正确执行：

```cs
int? x = 5;
int? y = 10;
bool b = x < y;      // true
```

这是因为编译器从基础值类型借用或“提升”了小于运算符。从语义上讲，它将前面的比较表达式转换为：

```cs
bool b = (x.HasValue && y.HasValue)
          ? (x.Value < y.Value)
          : false;
```

换句话说，如果`x`和`y`都有值，则通过`int`的小于运算符比较；否则返回`false`。

运算符提升意味着您可以隐式地在`T?`上使用`T`的运算符。您可以为`T?`定义运算符，以提供特定的空值行为，但在绝大多数情况下，最好依赖编译器自动为您应用系统化的可空逻辑。

编译器根据运算符的类别在空值逻辑上执行不同的操作。

### 相等运算符 (==, !=)

提升的相等运算符处理 null 值就像引用类型一样。这意味着两个 null 值是相等的：

```cs
Console.WriteLine (       null ==        null);  // True
Console.WriteLine ((bool?)null == (bool?)null);  // True
```

更进一步：

+   如果恰好一个操作数为 null，则操作数不相等。

+   如果两个操作数均非 null，则比较它们的`Value`。

### 关系运算符 (<、<=、>=、>)

关系运算符基于不能比较 null 操作数的原则。这意味着将 null 值与 null 或非 null 值比较会返回`false`：

```cs
bool b = x < y;    // Translation:

bool b = (x == null || y == null)
  ? false 
  : (x.Value < y.Value);

// b is false (assuming x is 5 and y is null)
```

### 所有其他运算符（+、−、*、/、%、&、|、^、<<、>>、+、++、--、!、~）

当任一操作数为空时，这些运算符返回空。这种模式对于 SQL 用户来说应该很熟悉：

```cs
int? c = x + y;   // Translation:

int? c = (x == null || y == null)
         ? null 
         : (int?) (x.Value + y.Value);

// c is null (assuming x is 5 and y is null)
```

例外情况是当`&`和`|`运算符应用于`bool?`时，我们稍后会讨论。

### 混合可空和非可空类型

您可以混合和匹配可空和非可空类型（这是因为存在从`T`到`T?`的隐式转换）：

```cs
int? a = null;
int b = 2;
int? c = a + b;   // c is null - equivalent to a + (int?)b
```

## `bool?`与`&`和`|`运算符

当提供`bool?`类型的操作数时，`&`和`|`运算符将`null`视为*未知值*。因此，`null | true`为 true，因为：

+   如果未知值为 false，则结果将为 true。

+   如果未知值为 true，则结果将为 true。

类似地，`null & false`为 false。这种行为对 SQL 用户来说应该很熟悉。以下示例列举了其他组合：

```cs
bool? n = null, f = false, t = true;
Console.WriteLine (n | n);    // *(null)*
Console.WriteLine (n | f);    // *(null)*
Console.WriteLine (n | t);    // True
Console.WriteLine (n & n);    // *(null)*
Console.WriteLine (n & f);    // False
Console.WriteLine (n & t);    // *(null)*
```

## 可空类型和空操作符

可空类型特别适用于`??`运算符（参见“空合并运算符”）。例如：

```cs
int? x = null;
int y = x ?? 5;        // y is 5

int? a = null, b = null, c = 123;
Console.WriteLine (a ?? b ?? c);  // 123
```

在可空值类型上使用`??`等同于使用`GetValueOrDefault`来调用显式默认值，唯一的区别是如果变量不为空，则默认值的表达式不会被评估。

可空类型也与空条件运算符配合得很好（参见“空条件运算符”）。在以下示例中，`length`评估为空：

```cs
System.Text.StringBuilder sb = null;
int? length = sb?.ToString().Length;
```

我们可以将此与空合并运算符结合使用，以评估为零而不是空：

```cs
int length = sb?.ToString().Length ?? 0;
```

