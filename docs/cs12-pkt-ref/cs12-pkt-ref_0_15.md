# 第十五章：对象类型

`object`（`System.Object`）是所有类型的最终基类。任何类型都可以隐式向上转型为`object`。

为了说明这对我们有用，考虑一个通用的*栈*。栈是一种基于 LIFO 原则的数据结构——“后进先出”。栈有两个操作：*push*将对象推入栈中和*pop*从栈中弹出对象。以下是一个简单的实现，可以容纳最多 10 个对象：

```cs
public class Stack
{
  int position;
  object[] data = new object[10];
  public void Push (object o) { data[position++] = o; }
  public object Pop() { return data[--position]; }
}
```

因为`Stack`使用的是对象类型，我们可以向`Stack`中`Push`和`Pop`任何类型的实例：

```cs
Stack stack = new Stack();
stack.Push ("sausage");
string s = (string) stack.Pop();   // Downcast
Console.WriteLine (s);             // sausage
```

`object`是一个引用类型，因为它是一个类。尽管如此，值类型（如`int`）也可以被转换为`object`并从`object`转换回来。为了实现这一点，CLR 必须执行一些特殊工作来弥合值类型和引用类型之间的基本差异。这个过程称为*装箱*和*拆箱*。

###### 注意

在“泛型”中，我们描述了如何改进我们的`Stack`类以更好地处理具有相同类型元素的堆栈。

## 装箱和拆箱

装箱是将值类型实例转换为引用类型实例的过程。引用类型可以是`object`类或一个接口（见“接口”）。在这个例子中，我们将一个`int`装箱为一个对象：

```cs
int x = 9;
object obj = x;           // Box the int
```

拆箱通过将对象转换回原始值类型来逆转操作：

```cs
int y = (int)obj;         // Unbox the int
```

拆箱需要显式转换。运行时会检查声明的值类型是否与实际对象类型匹配，如果检查失败则抛出`InvalidCastException`异常。例如，以下代码因为`long`不能精确匹配`int`而抛出异常：

```cs
object obj = 9;       // 9 is inferred to be of type int
long x = (long) obj;  // InvalidCastException
```

然而以下成功：

```cs
object obj = 9;
long x = (int) obj;
```

同样适用于这里：

```cs
object obj = 3.5;      // 3.5 inferred to be type double
int x = (int) (double) obj;    // x is now 3
```

在最后一个例子中，`(double)`执行了*拆箱*操作，然后`(int)`执行了*数值转换*。

*装箱*将值类型实例复制到新对象中，而*拆箱*将对象的内容复制回值类型实例：

```cs
int i = 3;
object boxed = i;
i = 5;
Console.WriteLine (boxed);    // 3
```

## 静态和运行时类型检查

C#在静态（编译时）和运行时都检查类型。

静态类型检查使得编译器能够在运行程序之前验证其正确性。以下代码会因为编译器强制执行静态类型而失败：

```cs
int x = "5";
```

当通过引用转换或拆箱进行向下转换时，CLR 执行运行时类型检查：

```cs
object y = "5";
int z = (int) y;       // Runtime error, downcast failed
```

运行时类型检查是可能的，因为堆上的每个对象内部都存储了一个小的类型标记。通过调用`object`的`GetType`方法可以检索此标记。

## GetType 方法和 typeof 运算符

在 C#中，所有类型在运行时都由`System.Type`的实例表示。获取`System.Type`对象有两种基本方式：对实例调用`GetType`方法或者在类型名称上使用`typeof`运算符。`GetType`在运行时评估；`typeof`在编译时静态评估。

`System.Type`具有诸如类型名称、程序集、基类型等属性。例如：

```cs
int x = 3;

Console.Write (x.GetType().Name);               // Int32
Console.Write (typeof(int).Name);               // Int32
Console.Write (x.GetType().FullName);    // System.Int32
Console.Write (x.GetType() == typeof(int));     // True
```

`System.Type`还有一些方法作为运行时反射模型的入口，我们在*C# 12 in a Nutshell*中详细描述了这一点。

## 对象成员列表

这里是`object`的所有成员：

```cs
public extern Type GetType();
public virtual bool Equals (object obj);
public static bool Equals (object objA, object objB);
public static bool ReferenceEquals (object objA,
                                    object objB);
public virtual int GetHashCode();
public virtual string ToString();
protected virtual void Finalize();
protected extern object MemberwiseClone();
```

## Equals、ReferenceEquals 和 GetHashCode

`object`类中的`Equals`方法与`==`操作符类似，但`Equals`是虚拟的，而`==`是静态的。以下示例说明了它们的区别：

```cs
object x = 3;
object y = 3;
Console.WriteLine (x == y);        // False
Console.WriteLine (x.Equals (y));  // True
```

因为`x`和`y`已转换为`object`类型，编译器静态绑定到`object`的`==`操作符，它使用*引用类型*语义来比较两个实例。（并且因为`x`和`y`被装箱，它们被表示在不同的内存位置，因此是不相等的。）然而，虚拟的`Equals`方法会委托给`Int32`类型的`Equals`方法，在比较两个值时使用*值类型*语义。

静态的`object.Equals`方法简单地调用第一个参数的虚拟`Equals`方法，之前会检查参数不为 null：

```cs
object x = null, y = 3;
bool error = x.Equals (y);        // Runtime error!
bool ok = object.Equals (x, y);   // OK (false)
```

`ReferenceEquals`强制进行引用类型的相等比较（在一些重载了`==`操作符的引用类型中，偶尔会有用）。

`GetHashCode`生成适用于基于哈希表的字典（如`System.Collections.Generic.Dictionary`和`System.Collections.Hashtable`）的哈希码。

要自定义类型的相等语义，至少必须重写`Equals`和`GetHashCode`方法。通常还会重载`==`和`!=`运算符。有关如何执行这两者的示例，请参见“运算符重载”。

## ToString 方法

`ToString`方法返回类型实例的默认文本表示。所有内置类型都会重写`ToString`方法：

```cs
string s1 = 1.ToString();      // s1 is "1"
string s2 = true.ToString();   // s2 is "True"
```

您可以按如下方式重写自定义类型的`ToString`方法：

```cs
public override string ToString() => "Foo";
```

