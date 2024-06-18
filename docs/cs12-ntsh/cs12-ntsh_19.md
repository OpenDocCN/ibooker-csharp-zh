# 第十九章 动态编程

第四章 解释了 C# 语言中动态绑定的工作原理。在本章中，我们简要介绍了*动态语言运行时*（DLR），然后探讨以下动态编程模式：

+   动态成员重载解析

+   自定义绑定（实现动态对象）

+   动态语言互操作性

###### 注意

在 第二十四章 中，我们描述了 `dynamic` 如何改进 COM 互操作性。

本章中的类型位于 `System.Dynamic` 命名空间中，除了 `CallSite<>`，它位于 `System.Runtime.CompilerServices` 中。

# 动态语言运行时

C# 依赖于 DLR（动态语言运行时）来执行动态绑定。

尽管名字与 CLR 的动态版本相反，DLR 实际上是位于 CLR 之上的一个库——就像任何其他库，比如 *System.Xml.dll*。它的主要作用是提供运行时服务来*统一*静态和动态类型语言的动态编程。因此，诸如 C#、Visual Basic、IronPython 和 IronRuby 等语言都使用相同的调用函数动态协议。这使它们可以共享库并调用其他语言编写的代码。

DLR 还使得在 .NET 中编写新的动态语言相对容易。动态语言作者不需要发出中间语言（IL），而是在*表达树*的级别上工作（这些表达树与我们在 第八章 中讨论的 `System.Linq.Expressions` 中的表达树相同）。

DLR 进一步确保所有消费者都能从*调用站点缓存*中获益，这是一种优化方法，DLR 可以防止重复进行在动态绑定过程中可能昂贵的成员解析决策。

# 动态成员重载解析

使用动态类型参数调用静态已知方法会将成员重载解析从编译时推迟到运行时。这在简化某些编程任务（例如简化*访问者*设计模式）中非常有用。它还有助于解决 C# 静态类型强加的限制。

## 简化访问者模式

本质上，*访问者*模式允许您在类层次结构中“添加”方法，而无需修改现有类。虽然有用，但与大多数其他设计模式相比，其静态版本显得微妙和不直观。它还要求被访问的类通过暴露 `Accept` 方法来使其“访问者友好”，如果这些类不在您的控制之下，则可能无法实现。

借助动态绑定，您可以更轻松地实现相同的目标，而无需修改现有的类。为了说明这一点，请考虑以下类层次结构：

```cs
class Person
{
  public string FirstName { get; set; }
  public string LastName  { get; set; }

  // The Friends collection may contain Customers & Employees:
  public readonly IList<Person> Friends = new Collection<Person> ();
}

class Customer : Person { public decimal CreditLimit { get; set; } }
class Employee : Person { public decimal Salary      { get; set; } }
```

假设我们想编写一个方法，程序化地将一个`Person`的详细信息导出到 XML `XElement`中。最明显的解决方案是在`Person`类中编写一个虚方法`ToXElement()`，返回一个填充了`Person`属性的`XElement`。然后我们在`Customer`和`Employee`类中重写它，使得`XElement`还填充了`CreditLimit`和`Salary`。然而，这种模式可能存在问题，原因有两个：

+   你可能没有`Person`、`Customer`和`Employee`类的所有权，因此无法向它们添加方法。（扩展方法也无法提供多态行为。）

+   `Person`、`Customer`和`Employee`类可能已经相当庞大。一个常见的反模式是“上帝对象”，其中一个类如`Person`吸引了大量功能，使得维护变得一团糟。一个良好的解药是避免向`Person`添加不需要访问`Person`私有状态的函数。`ToXElement`方法可能是一个很好的选择。

使用动态成员重载解析，我们可以在一个单独的类中编写`ToXElement`功能，而不需要基于类型的丑陋开关：

```cs
class ToXElementPersonVisitor
{
  public XElement DynamicVisit (Person p) => Visit ((dynamic)p);

  XElement Visit (Person p)
  {
    return new XElement ("Person",
      new XAttribute ("Type", p.GetType().Name),
      new XElement ("FirstName", p.FirstName),
      new XElement ("LastName", p.LastName),
      p.Friends.Select (f => DynamicVisit (f))
    );
  }

  XElement Visit (Customer c)   // Specialized logic for customers
  {
    XElement xe = Visit ((Person)c);   // Call "base" method
    xe.Add (new XElement ("CreditLimit", c.CreditLimit));
    return xe;
  }

  XElement Visit (Employee e)   // Specialized logic for employees
  {
    XElement xe = Visit ((Person)e);   // Call "base" method
    xe.Add (new XElement ("Salary", e.Salary));
    return xe;
  }
}
```

`DynamicVisit`方法执行动态分派——根据运行时确定的最具体版本调用`Visit`。注意粗体行中的内容，在其中我们对`Friends`集合中的每个人调用`DynamicVisit`方法。这确保如果朋友是`Customer`或`Employee`，则调用正确的重载。

我们可以演示这个类，如下所示：

```cs
var cust = new Customer
{
  FirstName = "Joe", LastName = "Bloggs", CreditLimit = 123
};
cust.Friends.Add (
  new Employee { FirstName = "Sue", LastName = "Brown", Salary = 50000 }
);

Console.WriteLine (new ToXElementPersonVisitor().DynamicVisit (cust));
```

结果如下：

```cs
<Person Type="Customer">
  <FirstName>Joe</FirstName>
  <LastName>Bloggs</LastName>
  <Person Type="Employee">
    <FirstName>Sue</FirstName>
    <LastName>Brown</LastName>
    <Salary>50000</Salary>
  </Person>
  <CreditLimit>123</CreditLimit>
</Person>
```

### 变化

如果计划多个访问者类，一个有用的变化是为访问者定义一个抽象基类：

```cs
abstract class PersonVisitor<T>
{
  public T DynamicVisit (Person p) { return Visit ((dynamic)p); }

  protected abstract T Visit (Person p);
  protected virtual T Visit (Customer c) { return Visit ((Person) c); }
  protected virtual T Visit (Employee e) { return Visit ((Person) e); }
}
```

然后子类不需要定义自己的`DynamicVisit`方法：它们只需重写`Visit`的版本，以便专门化它们想要的行为。这样做的优点是集中包含`Person`层次结构的方法，并允许实现者更自然地调用基本方法：

```cs
class ToXElementPersonVisitor : PersonVisitor<XElement>
{
  protected override XElement Visit (Person p)
  {
    return new XElement ("Person",
      new XAttribute ("Type", p.GetType().Name),
      new XElement ("FirstName", p.FirstName),
      new XElement ("LastName", p.LastName),
      p.Friends.Select (f => DynamicVisit (f))
    );
  }

  protected override XElement Visit (Customer c)
  {
    XElement xe = base.Visit (c);
    xe.Add (new XElement ("CreditLimit", c.CreditLimit));
    return xe;
  }

  protected override XElement Visit (Employee e)
  {
    XElement xe = base.Visit (e);
    xe.Add (new XElement ("Salary", e.Salary));
    return xe;
  }
}
```

随后你甚至可以直接继承`ToXElementPersonVisitor`本身。

## 匿名调用泛型类型的成员

C#静态类型的严格性是一把双刃剑。一方面，它在编译时强制执行一定程度的正确性。另一方面，它偶尔会使某些类型的代码难以表达，这时你必须依赖反射。在这些情况下，动态绑定是反射的一个更清晰和更快速的替代方案。

当你需要处理类型为`G<T>`的对象时，其中`T`是未知的时候，我们可以通过定义以下类来说明这一点：

```cs
public class Foo<T> { public T Value; }
```

假设我们接着按以下方式编写一个方法：

```cs
static void Write (object obj)
{
  if (obj is Foo<>)                           // Illegal
    Console.WriteLine ((Foo<>) obj).Value);   // Illegal
}
```

这个方法不会编译通过：你不能调用*未绑定*泛型类型的成员。

动态绑定提供了两种方法来解决这个问题。第一种是动态访问`Value`成员，如下所示：

```cs
static void Write (dynamic obj)
{
  try { Console.WriteLine (obj.Value); }
  catch (Microsoft.CSharp.RuntimeBinder.RuntimeBinderException) {...}
}
```

这样做的（潜在）优势是能够与定义 `Value` 字段或属性的任何对象一起工作。然而，存在一些问题。首先，在这种方式中捕获异常有些凌乱和效率低下（并且没有办法事先询问 DLR，“这个操作会成功吗？”）。其次，如果 `Foo` 是一个接口（比如 `IFoo<T>`）并且满足以下任一条件，则会失败：

+   `Value` 已经显式实现

+   实现了 `IFoo<T>` 的类型是不可访问的（稍后详述）

更好的解决方案是编写一个名为 `GetFooValue` 的重载辅助方法，并使用 *动态成员重载解析* 调用它：

```cs
static void Write (dynamic obj)
{
  object result = GetFooValue (obj);
  if (result != null) Console.WriteLine (result);
}

static T GetFooValue<T> (Foo<T> foo) => foo.Value;
static object GetFooValue (object foo) => null;
```

注意，我们重载了 `GetFooValue` 来接受一个 `object` 参数，它作为任何类型的后备。在运行时，C# 的动态绑定器将在调用 `GetFooValue` 时选择最佳重载。如果涉及的对象不基于 `Foo<T>`，它将选择对象参数的重载，而不是抛出异常。

###### 注意

另一种选择是仅编写第一个 `GetFooValue` 重载，然后捕获 `RuntimeBinderException`。优点是可以区分 `foo.Value` 为 null 的情况。缺点是会产生抛出和捕获异常的性能开销。

在 第十八章 中，我们使用反射解决了使用接口的相同问题，需要更多的工作（参见 “匿名调用泛型接口的成员”）。我们使用的示例是设计一个更强大的 `ToString()` 版本，能够理解诸如 `IEnumerable` 和 `IGrouping<,>` 等对象。以下是使用动态绑定更优雅地解决相同示例：

```cs
static string GetGroupKey<TKey,TElement> (IGrouping<TKey,TElement> group)
  => "Group with key=" + group.Key + ": ";

static string GetGroupKey (object source) => null;

public static string ToStringEx (object value)
{
  if (value == null) return "<null>";
  if (value is string s) return s;
  if (value.GetType().IsPrimitive) return value.ToString();

  StringBuilder sb = new StringBuilder();

  string groupKey = GetGroupKey ((dynamic)value);   // Dynamic dispatch
  if (groupKey != null) sb.Append (groupKey);

  if (value is IEnumerable)
    foreach (object element in ((IEnumerable)value))
      sb.Append (ToStringEx (element) + " ");

  if (sb.Length == 0) sb.Append (value.ToString());

  return "\r\n" + sb.ToString();
}
```

下面是它的作用：

```cs
Console.WriteLine (ToStringEx ("xyyzzz".GroupBy (c => c) ));

*Group with key=x: x*
*Group with key=y: y y*
*Group with key=z: z z z*
```

请注意，我们使用动态 *成员重载解析* 来解决这个问题。如果我们反而做了以下操作：

```cs
dynamic d = value;
try { groupKey = d.Value); }
catch (Microsoft.CSharp.RuntimeBinder.RuntimeBinderException) {...}
```

它会因为 LINQ 的 `GroupBy` 操作符返回实现 `IGrouping<,>` 的类型，而该类型本身是内部的，因此不可访问：

```cs
internal class Grouping : IGrouping<TKey,TElement>, ...
{
  public TKey Key;
  ...
}
```

即使 `Key` 属性被声明为 `public`，它所在的类将其限制在 `internal`，因此只能通过 `IGrouping<,>` 接口访问。并且正如在 第四章 中所解释的，当动态调用 `Value` 成员时，没有办法指示 DLR 绑定到该接口。

# 实现动态对象

一个对象可以通过实现 `IDynamicMetaObjectProvider` 提供其绑定语义，或者更简单地通过子类化 `DynamicObject`，后者提供了此接口的默认实现。这在 第四章 中通过以下示例简要演示：

```cs
dynamic d = new Duck();
d.Quack();                  // Quack method was called
d.Waddle();                 // Waddle method was called

public class Duck : DynamicObject
{
  public override bool TryInvokeMember (
    InvokeMemberBinder binder, object[] args, out object result)
  {
    Console.WriteLine (binder.Name + " method was called");
    result = null;
    return true;
  }
}
```

## DynamicObject

在前面的示例中，我们重写了`TryInvokeMember`，允许使用者在动态对象上调用方法——例如`Quack`或`Waddle`。`DynamicObject`公开了其他虚拟方法，使消费者能够使用其他编程构造。以下是在 C#中具有表示的对应构造：

| Method | 编程构造 |
| --- | --- |
| `TryInvokeMember` | 方法 |
| `TryGetMember`, `TrySetMember` | 属性或字段 |
| `TryGetIndex`, `TrySetIndex` | 索引器 |
| `TryUnaryOperation` | 例如 `!` 的一元操作符 |
| `TryBinaryOperation` | 例如 `==` 的二元操作符 |
| `TryConvert` | 转换（类型转换）为另一种类型 |
| `TryInvoke` | 在对象本身上的调用——例如`d("foo")` |

如果这些方法成功返回`true`，则它们应返回`true`。如果它们返回`false`，DLR 将回退到语言绑定程序，查找`Dynamic​Ob⁠ject`（子类）本身的匹配成员。如果失败，将抛出`RuntimeBinderException`。

我们可以通过一个类来说明`TryGetMember`和`TrySetMember`，该类允许我们动态访问`XElement`中的属性（`System.Xml.Linq`）：

```cs
static class XExtensions
{
  public static dynamic DynamicAttributes (this XElement e)
    => new XWrapper (e);

  class XWrapper : DynamicObject
  {
    XElement _element;
    public XWrapper (XElement e) { _element = e; }

    public override bool TryGetMember (GetMemberBinder binder,
                                       out object result)
    {
      result = _element.Attribute (binder.Name).Value;
      return true;
    }

    public override bool TrySetMember (SetMemberBinder binder,
                                       object value)
    {
      _element.SetAttributeValue (binder.Name, value);
      return true;
    }
  }
}
```

这里是如何使用它的：

```cs
XElement x = XElement.Parse (@"<Label Text=""Hello"" Id=""5""/>");
dynamic da = x.DynamicAttributes();
Console.WriteLine (da.Id);           // 5
da.Text = "Foo";
Console.WriteLine (x.ToString());    // <Label Text="Foo" Id="5" />
```

以下对`System.Data.IDataRecord`做了类似的事情，使得使用数据读取器更加简单：

```cs
public class DynamicReader : DynamicObject
{
  readonly IDataRecord _dataRecord;
  public DynamicReader (IDataRecord dr) { _dataRecord = dr; }

  public override bool TryGetMember (GetMemberBinder binder,
                                     out object result)
  {
    result = _dataRecord [binder.Name];
    return true;
  }
}
...
using (IDataReader reader = *someDbCommand*.ExecuteReader())
{
  dynamic dr = new DynamicReader (reader);
  while (reader.Read())
  {
    int id = dr.ID;
    string firstName = dr.FirstName;
    DateTime dob = dr.DateOfBirth;
    ...
  }
}
```

以下展示了`TryBinaryOperation`和`TryInvoke`的示例：

```cs
dynamic d = new Duck();
Console.WriteLine (d + d);          // foo
Console.WriteLine (d (78, 'x'));    // 123

public class Duck : DynamicObject
{
  public override bool TryBinaryOperation (BinaryOperationBinder binder,
                                           object arg, out object result)
  {
    Console.WriteLine (binder.Operation);   // Add
    result = "foo";
    return true;
  }

  public override bool TryInvoke (InvokeBinder binder,
                                  object[] args, out object result)
  {
    Console.WriteLine (args[0]);    // 78
    result = 123;
    return true;
  }
}
```

`DynamicObject`还公开了一些为动态语言提供的虚拟方法。特别是，重写`GetDynamicMemberNames`允许您返回您的动态对象提供的所有成员名称列表。

###### 注意

另一个实现`GetDynamicMemberNames`的原因是 Visual Studio 的调试器利用此方法显示动态对象的视图。

## ExpandoObject

`DynamicObject`的另一个简单应用是编写一个动态类，该类通过字符串键存储和检索对象。然而，通过`ExpandoObject`类已提供了这种功能：

```cs
dynamic x = new ExpandoObject();
x.FavoriteColor = ConsoleColor.Green;
x.FavoriteNumber = 7;
Console.WriteLine (x.FavoriteColor);    // Green
Console.WriteLine (x.FavoriteNumber);   // 7
```

`ExpandoObject`实现了`IDictionary<string,object>`，因此我们可以继续我们的示例并执行以下操作：

```cs
var dict = (IDictionary<string,object>) x;
Console.WriteLine (dict ["FavoriteColor"]);    // Green
Console.WriteLine (dict ["FavoriteNumber"]);   // 7
Console.WriteLine (dict.Count);                // 2
```

# 与动态语言的互操作性

尽管 C#通过`dynamic`关键字支持动态绑定，但它并不允许您在运行时执行描述在字符串中的表达式这么远：

```cs
string expr = "2 * 3";
// We can’t "execute" expr
```

这是因为将字符串转换为表达式树的代码需要词法和语义分析器。这些功能内置于 C#编译器中，不作为运行时服务提供。在运行时，C#仅提供一个*binder*，它指示 DLR 如何解释已构建的表达式树。

真正的动态语言，如 IronPython 和 IronRuby，确实允许您执行任意字符串，这在诸如脚本编写、编写动态配置系统和实现动态规则引擎等任务中非常有用。因此，尽管您可以在 C#中编写大部分应用程序，但在这些任务中调用动态语言可能很有用。此外，您可能希望使用用动态语言编写的 API，在.NET 库中没有等效功能的情况下使用。

###### 注意

Roslyn 脚本化 NuGet 包*Microsoft.CodeAnalysis.CSharp.Scripting*提供了一个 API，让您可以执行 C#字符串，尽管它是通过将您的代码编译成程序来实现的。编译开销使其比 Python 互操作更慢，除非您打算重复执行相同的表达式。

在以下示例中，我们使用 IronPython 来评估在 C#中动态创建的表达式。您可以使用以下脚本编写计算器。

###### 注意

要运行此代码，请将 NuGet 包*Dynamic​LanguageRuntime*（不要与*System.Dynamic.Runtime*包混淆）和*IronPython*添加到您的应用程序中。

```cs
using System;
using IronPython.Hosting;
using Microsoft.Scripting;
using Microsoft.Scripting.Hosting;

int result = (int) Calculate ("2 * 3");
Console.WriteLine (result);              // 6

object Calculate (string expression)
{
  ScriptEngine engine = Python.CreateEngine();
  return engine.Execute (expression);
}
```

因为我们将一个字符串传递给 Python，所以表达式将按照 Python 的规则进行评估，而不是 C#的规则。这也意味着我们可以使用 Python 的语言特性，比如列表：

```cs
var list = (IEnumerable) Calculate ("[1, 2, 3] + [4, 5]");
foreach (int n in list) Console.Write (n);  // 12345
```

## 在 C#和脚本之间传递状态

要将变量从 C#传递到 Python，需要进行几个额外的步骤。以下示例说明了这些步骤，并可以作为规则引擎的基础：

```cs
// The following string could come from a file or database:
string auditRule = "taxPaidLastYear / taxPaidThisYear > 2";

ScriptEngine engine = Python.CreateEngine ();    

ScriptScope scope = engine.CreateScope ();       
scope.SetVariable ("taxPaidLastYear", 20000m);
scope.SetVariable ("taxPaidThisYear", 8000m);

ScriptSource source = engine.CreateScriptSourceFromString (
                      auditRule, SourceCodeKind.Expression);

bool auditRequired = (bool) source.Execute (scope);
Console.WriteLine (auditRequired);   // True
```

通过调用`GetVariable`也可以获取变量：

```cs
string code = "result = input * 3";

ScriptEngine engine = Python.CreateEngine();

ScriptScope scope = engine.CreateScope();
scope.SetVariable ("input", 2);

ScriptSource source = engine.CreateScriptSourceFromString (code,
                                  SourceCodeKind.SingleStatement);
source.Execute (scope);
Console.WriteLine (scope.GetVariable ("result"));   // 6
```

请注意，在第二个示例中，我们指定了`SourceCodeKind.SingleStatement`（而不是`Expression`），以通知引擎我们要执行一个语句。

类型在.NET 和 Python 世界之间自动驱动。您甚至可以从脚本侧访问.NET 对象的成员：

```cs
string code = @"sb.Append (""World"")";

ScriptEngine engine = Python.CreateEngine ();

ScriptScope scope = engine.CreateScope ();
var sb = new StringBuilder ("Hello");
scope.SetVariable ("sb", sb);

ScriptSource source = engine.CreateScriptSourceFromString (
                      code, SourceCodeKind.SingleStatement);
source.Execute (scope);
Console.WriteLine (sb.ToString());   // HelloWorld
```
