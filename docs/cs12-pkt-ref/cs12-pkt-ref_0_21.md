# 第二十一章：泛型

C# 有两种分离的机制用于编写可在不同类型间重复使用的代码：*继承* 和 *泛型*。继承通过基类型表达重用性，而泛型通过包含“占位符”类型的“模板”表达重用性。与继承相比，泛型可以 *增加类型安全性* 和 *减少强制转换和装箱*。

## 泛型类型

*泛型类型* 声明 *类型参数* —— 消费泛型类型的内容提供 *类型参数*。这是一个泛型类型 `Stack<T>`，设计用于堆叠类型为 `T` 的实例。`Stack<T>` 声明了一个类型参数 `T`：

```cs
public class Stack<T>
{
  int position;
  T[] data = new T[100];
  public void Push (T obj) => data[position++] = obj;
  public T Pop()           => data[--position];
}
```

我们可以如下使用 `Stack<T>`：

```cs
var stack = new Stack<int>();
stack.Push (5);
stack.Push (10);
int x = stack.Pop();        // x is 10
int y = stack.Pop();        // y is 5
```

###### 注意

注意，在最后两行不需要向下转换，避免了运行时错误的可能性并消除了装箱/拆箱的开销。这使得我们的泛型栈优于使用 `object` 替代 `T` 的非泛型栈（参见“对象类型”的示例）。

`Stack<int>` 在类型参数 `T` 中填充了类型参数 `int`，隐式地在运行时创建了一个类型（综合在运行时发生）。`Stack<int>` 的定义如下（替换项以粗体显示，为了避免混淆，类名已经被隐藏）：

```cs
public class ###
{
  int position;
  int[] data = new int[100];
  public void Push (int obj) => data[position++] = obj;
  public int Pop()           => data[--position];
}
```

从技术上讲，我们称 `Stack<T>` 是一个 *开放类型*，而 `Stack<int>` 是一个 *闭合类型*。在运行时，所有泛型类型实例都是闭合的，占位符类型已填充。

## 泛型方法

*泛型方法* 在方法的签名中声明类型参数。通过泛型方法，可以以通用方式实现许多基本算法。这是一个交换任意类型 `T` 变量内容的泛型方法示例：

```cs
static void Swap<T> (ref T a, ref T b)
{
  T temp = a; a = b; b = temp;
}
```

你可以如下使用 `Swap<T>`：

```cs
int x = 5, y = 10;
Swap (ref x, ref y);
```

通常情况下，不需要为泛型方法提供类型参数，因为编译器可以隐式推断类型。如果存在歧义，泛型方法可以如下调用带有类型参数：

```cs
Swap<int> (ref x, ref y);
```

在泛型 *类型* 中，除非以尖括号语法 *引入* 类型参数，否则方法不被视为泛型方法。在我们的泛型栈中，`Pop` 方法仅消耗类型的现有类型参数 `T`，并不被视为泛型方法。

方法和类型是唯一可以引入类型参数的构造。属性、索引器、事件、字段、构造函数、操作符等不能声明类型参数，尽管它们可以参与其封闭类型已声明的任何类型参数。例如，在我们的通用堆栈示例中，我们可以编写一个返回通用项的索引器：

```cs
public T this [int index] { get { return data[index]; } }
```

类似地，构造函数可以参与现有的类型参数，但不能*引入*它们。

## 声明类型参数

类型参数可以在类、结构、接口、委托（参见“委托”）和方法的声明中引入。您可以通过逗号分隔它们来指定多个类型参数：

```cs
class Dictionary<TKey, TValue> {...}
```

实例化：

```cs
var myDict = new Dictionary<int,string>();
```

泛型类型名称和方法名称可以重载，只要类型参数的数量不同即可。例如，以下三个类型名称不会冲突：

```cs
class A {}
class A<T> {}
class A<T1,T2> {}
```

###### 注意

按照惯例，具有*单一*类型参数的泛型类型和方法将其参数命名为 `T`，只要参数的意图清楚即可。对于*多个*类型参数，每个参数都有一个更具描述性的名称（以 `T` 为前缀）。

## `typeof` 和未绑定的泛型类型

开放的泛型类型在运行时不存在：开放的泛型类型作为编译的一部分被关闭。然而，未绑定的泛型类型可以在运行时存在——纯粹作为一个 `Type` 对象。在 C# 中指定未绑定的泛型类型的唯一方法是使用 `typeof` 运算符：

```cs
class A<T> {}
class A<T1,T2> {}
...

Type a1 = typeof (A<>);   // *Unbound* type
Type a2 = typeof (A<,>);  // Indicates 2 type args
Console.Write (a2.GetGenericArguments().Count());  // 2
```

您还可以使用 `typeof` 运算符来指定封闭类型：

```cs
Type a3 = typeof (A<int,int>);
```

它还可以指定一个开放类型（在运行时关闭）：

```cs
class B<T> { void X() { Type t = typeof (T); } }
```

## 默认通用值

您可以使用 `default` 关键字获取泛型类型参数的默认值。引用类型的默认值为 `null`，值类型的默认值是对类型字段进行按位零操作的结果：

```cs
static void Zap<T> (T[] array)
{
  for (int i = 0; i < array.Length; i++)
 array[i] = default(T);
}
```

从 C# 7.1 开始，可以省略类型参数，编译器能够推断出的情况下：

```cs
 array[i] = default;
```

## 泛型约束

默认情况下，类型参数可以用任何类型替换。*约束* 可以应用于类型参数，以要求更具体的类型参数。约束有八种类型：

```cs
where *T* : *base-class*  // Base class constraint
where *T* : *interface*   // Interface constraint
where *T* : class       // Reference type constraint
where *T* : class?      // (See "Nullable Reference Types")
where *T* : struct      // Value type constraint
where *T* : unmanaged   // Unmanaged constraint
where *T* : new()       // Parameterless constructor
                      // constraint
where *U* : *T*           // Naked type constraint
where *T* : notnull     // Non-nullable value type
                      // or non-nullable reference type
```

在下面的示例中，`GenericClass<T,U>` 要求 `T` 派生自（或与）`SomeClass` 相同，并实现 `Interface1`，并要求 `U` 提供一个无参数的构造函数：

```cs
class     SomeClass {}
interface Interface1 {}

class GenericClass<T,U> where T : SomeClass, Interface1
                        where U : new()
{ ... }
```

约束可以应用于类型参数定义的任何地方，无论是在方法中还是在类型定义中。

###### 注意

*约束* 是一种*限制*；然而，类型参数约束的主要目的是允许禁止的事情。

例如，约束 `T:Foo` 允许您将 `T` 的实例视为 `Foo`，约束 `T:new()` 允许您构造 `T` 的新实例。

*基类约束*指定类型参数必须是某个类的子类（或匹配）；*接口约束*指定类型参数必须实现该接口。这些约束允许类型参数的实例被隐式转换为该类或接口。

*类约束*和*结构约束*指定`T`必须是引用类型或（非空）值类型。未管理的约束是结构约束的更强版本：`T`必须是简单值类型或递归地不含任何引用类型的结构体。*无参构造约束*要求`T`必须有一个公共的无参构造函数，并允许你在`T`上调用`new()`：

```cs
static void Initialize<T> (T[] array) where T : new()
{
  for (int i = 0; i < array.Length; i++)
    array[i] = new T();
}
```

*裸类型约束*要求一个类型参数派生自（或匹配）另一个类型参数。

## 泛型类型的子类化

泛型类可以像非泛型类一样被子类化。子类可以将基类的类型参数保持开放，如以下示例：

```cs
class Stack<T>                   {...}
class SpecialStack<T> : Stack<T> {...}
```

或子类可以用具体类型关闭泛型类型参数：

```cs
class IntStack : Stack<int>  {...}
```

子类型也可以引入新的类型参数：

```cs
class List<T>                     {...}
class KeyedList<T,TKey> : List<T> {...}
```

## 自引用泛型声明

类型可以在关闭类型参数时命名*自身*作为具体类型：

```cs
public interface IEquatable<T> { bool Equals (T obj); }

public class Balloon : IEquatable<Balloon>
{
  public bool Equals (Balloon b) { ... }
}
```

以下也是合法的：

```cs
class Foo<T> where T : IComparable<T> { ... }
class Bar<T> where T : Bar<T> { ... }
```

## 静态数据

静态数据对于每个封闭类型都是唯一的：

```cs
Console.WriteLine (++Bob<int>.Count);     // 1
Console.WriteLine (++Bob<int>.Count);     // 2
Console.WriteLine (++Bob<string>.Count);  // 1
Console.WriteLine (++Bob<object>.Count);  // 1

class Bob<T> { public static int Count; }
```

## 协变性

###### 注意

协变性和逆变性是高级概念。它们引入到 C#中的动机是允许泛型接口和泛型（特别是.NET 中定义的那些，如`IEnumerable<T>`）工作得更像你期望的那样。你可以在不理解协变性和逆变性背后的细节的情况下受益于此。

假设`A`可转换为`B`，如果`X<A>`可转换为`X<B>`，则`X`具有协变类型参数。

（根据 C#的协变性概念，*可转换*意味着通过*隐式引用转换*可转换，比如`A`是`B`的子类，或`A`实现了`B`。数值转换、装箱转换和自定义转换不包括在内。）

例如，类型`IFoo<T>`如果以下内容合法，则具有协变`T`：

```cs
IFoo<string> s = ...;
IFoo<object> b = s;
```

接口（和委托）允许协变类型参数。举例来说，假设我们在本节开始编写的`Stack<T>`类实现了以下接口：

```cs
public interface IPoppable<out T> { T Pop(); }
```

对`T`上的`out`修饰符表示`T`仅在*输出位置*（例如方法的返回类型）中使用，并标志类型参数为*协变*，允许以下代码：

```cs
// Assuming that Bear subclasses Animal:
var bears = new Stack<Bear>();
bears.Push (new Bear());

// Because bears implements IPoppable<Bear>,
// we can convert it to IPoppable<Animal>:
IPoppable<Animal> animals = bears;   // Legal
Animal a = animals.Pop();
```

编译器允许从`bears`到`animals`的转换—这是通过接口的类型参数是协变而允许的。

###### 注意

接口`IEnumerator<T>`和`IEnumerable<T>`（见“枚举和迭代器”）标记为协变`T`。这允许你将`IEnumerable<string>`转换为`IEnumerable<object>`，例如。

如果在*输入*位置使用协变类型参数（例如方法的参数或可写属性），编译器将生成错误。此限制的目的是确保编译时类型安全性。例如，它阻止我们向接口添加`Push(T)`方法，消费者可能会误用，试图将骆驼推入`IPoppable<Animal>`（请记住，我们示例中的底层类型是一堆熊）。要定义`Push(T)`方法，`T`必须实际上是*逆变的*。

###### 注意

C#仅支持*引用转换*的元素的协变性（和逆变性）—不支持*装箱转换*。因此，如果您编写了一个接受`IPoppable<object>`类型参数的方法，可以使用`IPoppable<string>`但不能使用`IPoppable<int>`调用它。

## 逆变性

我们之前看到，假设`A`允许将隐式引用转换为`B`，则类型`X`具有协变类型参数，如果`X<A>`允许将引用转换为`X<B>`。当类型参数仅出现在*输入*位置时，带有`in`修饰符。扩展我们之前的示例，如果`Stack<T>`类实现以下接口：

```cs
public interface IPushable<in T> { void Push (T obj); }
```

我们可以合法地执行这样的操作：

```cs
IPushable<Animal> animals = new Stack<Animal>();
IPushable<Bear> bears = animals;    // Legal
bears.Push (new Bear());
```

反映协变性，如果您尝试在输出位置使用逆变类型参数（例如作为返回值或可读属性），编译器将报告错误。

