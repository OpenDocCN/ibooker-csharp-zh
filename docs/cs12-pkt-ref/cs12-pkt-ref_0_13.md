# 第十三章：类

*类* 是最常见的引用类型。最简单的类声明如下：

```cs
class Foo
{
}
```

更复杂的类可选包含以下内容：

| 在关键字 `class` 前 | *特性* 和 *类修饰符*。非嵌套类修饰符包括 `public`、`internal`、`abstract`、`sealed`、`static`、`unsafe` 和 `partial`。 |
| --- | --- |
| 在 `Foo` 后面 | *泛型类型参数* 和 *约束*，一个 *基类* 和 *接口*。 |
| 在大括号内 | *类成员*（包括 *方法*、*属性*、*索引器*、*事件*、*字段*、*构造函数*、*重载运算符*、*嵌套类型* 和 *终结器*）。 |

## 字段

*字段* 是一个类或结构体的成员变量。例如：

```cs
class Octopus
{
 string name;
 public int Age = 10;
}
```

字段可以使用 `readonly` 修饰符防止其在构造后被修改。只能在声明中或封闭类型的构造函数内为只读字段赋值。

字段初始化是可选的。未初始化的字段具有默认值（`0`、`'\0'`、`null`、`false`）。字段初始化程序按其出现顺序在构造函数之前运行。

为方便起见，您可以在逗号分隔的列表中声明多个相同类型的字段。这是所有字段共享相同属性和字段修饰符的便捷方式。例如：

```cs
static readonly int legs = 8, eyes = 2;
```

## 常量

*常量*在编译时静态评估，编译器在使用时直接替换其值（类似于 C++中的宏）。常量可以是任何内置的数值类型：`bool`、`char`、`string`或枚举类型。

使用`const`关键字声明常量，并且必须用一个值进行初始化。例如：

```cs
public class Test
{
 public const string Message = "Hello World";
}
```

常量比`static readonly`字段更加受限制——无论在您可以使用的类型还是在字段初始化语义上。常量与`static readonly`字段的另一个不同之处在于常量的评估发生在编译时。常量也可以声明为方法的本地变量：

```cs
static void Main()
{
  const double twoPI = 2 * System.Math.PI;
  ...
}
```

## 方法

*方法*通过一系列语句执行操作。方法可以通过指定*参数*接收调用者的*输入*数据，并通过指定*返回类型*向调用者发送*输出*数据。方法可以指定`void`返回类型，表示它不向其调用者返回任何值。方法还可以通过`ref`和`out`参数将数据返回给调用者。

方法的*签名*必须在类型内是唯一的。方法的签名包括其名称和参数类型的顺序（但不包括参数*名称*和返回类型）。

### 表达式体方法

一个由单个表达式组成的方法，例如以下内容：

```cs
int Foo (int x) { return x * 2; }
```

可以更简洁地编写为*表达式体方法*。一个胖箭头取代了大括号和`return`关键字：

```cs
int Foo (int x) => x * 2;
```

表达式体函数也可以有`void`返回类型：

```cs
void Foo (int x) => Console.WriteLine (x);
```

### 本地方法

您可以在另一个方法内定义一个方法：

```cs
void WriteCubes()
{
  Console.WriteLine (Cube (3));

  int Cube (int value) => value * value * value;
}
```

本地方法（本例中为`Cube`）仅对封闭方法（`WriteCubes`）可见。这简化了包含类型，并立即向查看代码的任何人表明`Cube`在其他地方未被使用。本地方法可以访问封闭方法的本地变量和参数。这带来了许多后果，我们在“捕获外部变量”中描述。

本地方法可以出现在其他函数种类中，例如属性访问器、构造函数等，并且甚至可以出现在其他本地方法中。本地方法可以是迭代器或异步的。

声明在顶层语句中的方法被隐式地视为本地方法；我们可以如下演示：

```cs
int x = 3; Foo();
void Foo() => Console.WriteLine (x);  // We can access x
```

### 静态本地方法

将`static`修饰符添加到本地方法（从 C# 8 开始）可防止其访问封闭方法的本地变量和参数。这有助于减少耦合，并防止本地方法意外地引用包含方法中的变量。

### 方法重载

###### 警告

局部方法不能重载。这意味着在顶级语句中声明的方法（将其视为局部方法）不能重载。

类型可以重载方法（具有相同名称的多个方法），只要参数类型不同即可。例如，以下方法可以共存于同一类型中：

```cs
void Foo (int x);
void Foo (double x);
void Foo (int x, float y);
void Foo (float x, int y);
```

## 实例构造函数

构造函数在类或结构上运行初始化代码。构造函数定义类似于方法，不同之处在于方法名和返回类型被简化为封闭类型的名称：

```cs
Panda p = new Panda ("Petey");   // Call constructor

public class Panda
{
  string name;              // Define field
 public Panda (string n)   // Define constructor
 {
 name = n;               // Initialization code
 }
}
```

单语句构造函数可以编写为表达式体成员：

```cs
public Panda (string n) => name = n;
```

类或结构可以重载构造函数。一个重载可以调用另一个，使用`this`关键字：

```cs
public class Wine
{
  public Wine (decimal price) {...}

  public Wine (decimal price, int year) 
    : this (price) {...}
}
```

当一个构造函数调用另一个时，*被调用的构造函数*会首先执行。

您可以将*表达式*传递给另一个构造函数，如下所示：

```cs
  public Wine (decimal price, DateTime year)
    : this (price, year.Year) {...}
```

表达式本身不能使用`this`引用，例如调用实例方法。不过，它可以调用静态方法。

### 隐式无参数构造函数

对于类，只有当您没有定义任何构造函数时，C# 编译器才会自动生成一个无参数的公共构造函数。但是，一旦您定义了至少一个构造函数，无参数构造函数就不再自动生成。

### 非公共构造函数

构造函数不需要是公共的。有一个非公共构造函数的常见原因是通过静态方法调用控制实例创建。可以使用静态方法从池中返回对象，而不是创建新对象，或者根据输入参数返回所选的特殊子类。

## 解构方法

虽然构造函数通常接受一组值（作为参数）并将它们分配给字段，但解构函数（C# 7+）则相反，将字段分配回一组变量。解构方法必须称为`Deconstruct`并具有一个或多个`out`参数：

```cs
class Rectangle
{
  public readonly float Width, Height;

  public Rectangle (float width, float height)
  {
    Width = width; Height = height;
  }

 public void Deconstruct (out float width,
 out float height)
 {
 width = Width; height = Height;
 }
}
```

要调用解构函数，您需要使用以下特殊语法：

```cs
var rect = new Rectangle (3, 4);
(float width, float height) = rect;
Console.WriteLine (width + " " + height);    // 3 4
```

第二行是解构调用。它创建两个局部变量然后调用`Deconstruct`方法。我们的解构调用等同于以下内容：

```cs
rect.Deconstruct (out var width, out var height);
```

解构调用允许隐式类型转换，因此我们可以缩短我们的调用到：

```cs
(var width, var height) = rect;
```

或者简单点：

```cs
var (width, height) = rect;
```

如果您要解构的变量已经定义，完全可以省略类型；这称为*解构赋值*：

```cs
(width, height) = rect;
```

您可以通过重载`Deconstruct`方法为调用者提供一系列解构选项。

###### 注意

`Deconstruct`方法可以是扩展方法（参见“扩展方法”）。如果您想解构未经作者授权的类型，这是一个有用的技巧。

从 C# 10 开始，您可以在解构时混合和匹配现有变量和新变量：

```cs
double x1 = 0;
(x1, double y2) = rect;
```

## 对象初始化器

为了简化对象初始化，可以在构造后直接通过*对象初始化器*初始化对象的可访问字段或属性。例如，请考虑以下类：

```cs
public class Bunny
{
  public string Name;
  public bool LikesCarrots, LikesHumans;

  public Bunny () {}
  public Bunny (string n) => Name = n;
}
```

使用对象初始化器，你可以像下面这样实例化`Bunny`对象：

```cs
Bunny b1 = new Bunny {
                       Name="Bo",
                       LikesCarrots = true,
                       LikesHumans = false
                     };

Bunny b2 = new Bunny ("Bo") {
                              LikesCarrots = true,
                              LikesHumans = false
                            };
```

## this 引用

`this`引用指向实例本身。在以下示例中，`Marry`方法使用`this`来设置`partner`的`mate`字段：

```cs
public class Panda
{
  public Panda Mate;

  public void Marry (Panda partner)
  {
    Mate = partner;
    partner.Mate = this;
  }
}
```

`this`引用还可以将局部变量或参数与字段区分开。例如：

```cs
public class Test
{
  string name;
  public Test (string name) => this.name = name;
}
```

`this`引用仅在类或结构的非静态成员中有效。

## 属性

属性看起来像外部的字段，但内部包含逻辑，就像方法一样。例如，你无法通过查看以下代码确定`CurrentPrice`是字段还是属性：

```cs
Stock msft = new Stock();
msft.CurrentPrice = 30;
msft.CurrentPrice -= 3;
Console.WriteLine (msft.CurrentPrice);
```

属性声明与字段类似，但添加了`get`/`set`块。以下是如何将`CurrentPrice`实现为属性的方法：

```cs
public class Stock
{
  decimal currentPrice;  // The private "backing" field

  public decimal CurrentPrice    // The public property
  {
     get { return currentPrice; }
     set { currentPrice = value; }
  }
}
```

`get`和`set`表示属性的*访问器*。当读取属性时，`get`访问器运行。它必须返回属性类型的值。当分配属性时，`set`访问器运行。它有一个隐含的名为`value`的参数，通常赋给一个私有字段（在本例中为`currentPrice`）。

虽然属性的访问方式与字段相同，但它们的区别在于它们允许实现者完全控制获取和设置其值。这种控制使得实现者可以选择所需的内部表示，而不向属性的用户公开内部细节。在这个例子中，如果`value`超出有效值范围，`set`方法可能会抛出异常。

###### 注意

在本书中，我们使用公共字段来保持示例不被干扰。在真实应用中，你通常会倾向于使用公共属性而不是公共字段来促进封装。

如果只指定了`get`访问器，则属性是只读的；如果只指定了`set`访问器，则属性是只写的。很少使用只写属性。

属性通常具有专用的后备字段来存储底层数据。但它不需要这样做；它可以返回从其他数据计算出的值：

```cs
decimal currentPrice, sharesOwned;

public decimal Worth
{
  get { return currentPrice * sharesOwned; }
}
```

### 表达式体属性

你可以将只读属性（如上面的示例）更简洁地声明为*表达式体属性*。一个箭头替换了所有的大括号、`get`和`return`关键字：

```cs
public decimal Worth => currentPrice * sharesOwned;
```

从 C# 7 开始，`set`访问器也可以是表达式体的：

```cs
public decimal Worth
{
  get => currentPrice * sharesOwned;
 set => sharesOwned = value / currentPrice;
}
```

### 自动属性

属性最常见的实现方式是一个简单读取和写入与属性相同类型的私有字段的 getter 和/或 setter。*自动属性*声明指示编译器提供此实现。我们可以通过将`CurrentPrice`声明为自动属性来改进本节中的第一个示例：

```cs
public class Stock
{
  public decimal CurrentPrice { get; set; }
}
```

编译器会自动生成一个不能被引用的具有编译器生成名称的私有后备字段。如果要将属性公开为其他类型的只读属性，可以将`set`访问器标记为`private`或`protected`。

### 属性初始化器

你可以像处理字段一样向自动属性添加*属性初始化器*：

```cs
public decimal CurrentPrice { get; set; } = 123;
```

这使得`CurrentPrice`有了一个初始值为 123。具有初始化器的属性可以是只读的：

```cs
public int Maximum { get; } = 999;
```

就像只读字段一样，只读自动属性也可以在类型的构造函数中分配。这在创建*不可变*（只读）类型时非常有用。

### 获取和设置的可访问性

`get`和`set`访问器可以具有不同的访问级别。这种情况的典型用法是在`public`属性上具有`internal`或`private`访问修饰符的`set`访问器：

```cs
private decimal x;
public decimal X
{
  get         { return x;  }
  private set { x = Math.Round (value, 2); }
}
```

请注意，您声明属性本身具有更宽松的访问级别（在此示例中为`public`），并在您希望*更不*可访问的访问器上添加修饰符。

### 仅初始化器

从 C# 9 开始，你可以声明一个使用`init`而不是`set`的属性访问器：

```cs
public class Note
{
  public int Pitch    { get; init; } = 20;
  public int Duration { get; init; } = 100;
}
```

这些*仅初始化*属性的行为类似于只读属性，但也可以通过对象初始化器进行设置：

```cs
var note = new Note { Pitch = 50 };
```

之后，属性将无法更改：

```cs
note.Pitch = 200;  // Error – init-only setter!
```

仅初始化属性甚至不能从其类内部进行设置，除非通过其属性初始化器、构造函数或另一个仅初始化访问器。

替代仅初始化属性的方法是通过构造函数填充只读属性：

```cs
  public Note (int pitch = 20, int duration = 100)
  {
 Pitch = pitch; Duration = duration;
  }
```

如果类是公共库的一部分，则此方法使得版本控制变得困难，因为在以后的日期向构造函数添加可选参数会破坏与使用者的二进制兼容性（而添加新的仅初始化属性不会破坏任何东西）。

###### 注意

仅初始化属性还有另一个显著的优势，即当与记录一起使用时，它们允许非破坏性变异（见“Records”）。

与普通的`set`访问器一样，初始化访问器也可以提供一个实现：

```cs
public class Point
{
  readonly int _x;
  public int X { get => _x; init => _x = value; }
  ...
```

请注意，`_x`字段是只读的：仅初始化器允许修改其自身类中的`readonly`字段。如果没有此功能，`_x`需要是可写的，而类在内部将无法完全不可变。

## 索引器

索引器为访问类或结构中封装的值列表或字典的元素提供了一种自然的语法。索引器类似于属性，但通过索引参数访问而不是属性名。`string`类具有一个索引器，它允许您通过`int`索引访问其每个`char`值：

```cs
string s = "hello";
Console.WriteLine (s[0]); // 'h'
Console.WriteLine (s[3]); // 'l'
```

使用索引器的语法与使用数组的语法类似，不同之处在于索引参数可以是任何类型。您可以在方括号之前插入问号来对索引器进行空值条件调用（参见“Null Operators”）：

```cs
string s = null;
Console.WriteLine (s?[0]);  // Writes nothing; no error.
```

### 实现索引器

要编写索引器，定义一个名为`this`的属性，并在方括号中指定参数。例如：

```cs
class Sentence
{
  string[] words = "The quick brown fox".Split();

 public string this [int wordNum]      // indexer
 { 
 get { return words [wordNum];  }
 set { words [wordNum] = value; }
 }
}
```

这是我们如何使用这个索引器的方式：

```cs
Sentence s = new Sentence();
Console.WriteLine (s[3]);       // fox
s[3] = "kangaroo";
Console.WriteLine (s[3]);       // kangaroo
```

类型可以声明多个索引器，每个索引器可以带有不同类型的参数。索引器还可以接受多个参数：

```cs
public string this [int arg1, string arg2]
{
  get { ... }  set { ... }
}
```

如果省略`set`访问器，索引器将变为只读，并且可以使用表达式主体语法来缩短其定义：

```cs
public string this [int wordNum] => words [wordNum];
```

### 使用索引和范围与索引器

您可以通过定义索引器的参数类型为`Index`或`Range`来支持自己的类中的索引和范围（见“索引和范围”）。我们可以通过向`Sentence`类添加以下索引器来扩展我们之前的示例：

```cs
  public string this [Index index] => words [index];
  public string[] this [Range range] => words [range];
```

这随后使以下内容成为可能：

```cs
Sentence s = new Sentence();
Console.WriteLine (s [¹]);         // fox  
string[] firstTwoWords = s [..2];   // (The, quick)
```

## 主构造函数（C# 12）

从 C# 12 开始，您可以直接在类（或结构）声明之后包含参数列表：

```cs
class Person (string firstName, string lastName)
{
  public void Print() =>
    Console.WriteLine (firstName + " " + lastName);
}
```

这会指示编译器使用*主构造函数参数*（`firstName`和`lastName`）自动构建一个*主构造函数*，以便我们可以按以下方式实例化我们的类：

```cs
Person p = new Person ("Alice", "Jones");
p.Print();    // Alice Jones
```

主构造函数非常适用于原型设计和其他简单场景。另一种方法是定义字段来存储`firstName`和`lastName`，然后编写一个构造函数来填充它们。

C#生成的构造函数之所以称为主构造函数，是因为您选择（显式地）编写的任何额外构造函数都必须调用它：

```cs
class Person (string firstName, string lastName)
{
  public Person (string first, string last, int age)
    : this (first, last)  // Must call primary constructor
 { ... }
}
```

这确保了主构造函数参数*始终被填充*。

###### 注意

C#还提供了*记录*，我们在“记录”中介绍。记录也支持主构造函数；但是，编译器会额外处理记录，并默认为每个主构造函数参数生成一个公共的 init-only 属性。

主构造函数最适用于简单的场景，因为以下限制：

+   您不能向主构造函数添加额外的初始化代码。

+   尽管很容易将主构造函数参数公开为公共属性，但除非属性是只读的，否则不能轻松地整合验证逻辑。

主构造函数取代了 C#否则会生成的默认无参数构造函数。

### 主构造函数语义

要理解主构造函数的工作原理，请考虑普通构造函数的行为：

```cs
class Person
{
  public Person (string firstName, string lastName)
  {
 *   ... do something with firstName, lastName*
  }
}
```

当此构造函数内部的代码执行完毕后，参数`firstName`和`lastName`将退出作用域，不能随后访问。相比之下，主构造函数的参数不会退出作用域，并且在类的任何地方都可以随后访问，对象的生命周期内有效。

###### 注意

主构造函数参数是特殊的 C#构造，不是*字段*，尽管编译器最终会在幕后生成隐藏字段来存储它们的值（如果需要）。

### 主构造函数和字段/属性初始化器

主构造函数参数的可访问性延伸到字段和属性的初始化器。在以下示例中，我们使用字段和属性的初始化器将`firstName`分配给公共字段，将`lastName`分配给公共属性：

```cs
class Person (string firstName, string lastName)
{
  public readonly string FirstName = firstName;
  public string LastName { get; } = lastName;
}
```

### 遮蔽主构造函数参数

字段（或属性）可以重用主构造函数参数名称：

```cs
class Person (string firstName, string lastName)
{
  readonly string firstName = firstName;
  readonly string lastName = lastName;
}
```

在这种情况下，字段或属性具有优先权，因此在字段和属性初始化器的右侧（用粗体显示）*以外*，主构造函数参数会被屏蔽。

###### 注意

与普通参数一样，主构造函数参数是可写的。使用同名的`readonly`字段屏蔽它们（就像我们的示例中那样），有效地保护它们免受后续修改。

### 验证主构造函数参数

在“抛出表达式”中，我们将描述在遇到无效数据等场景时如何抛出异常。以下是一个预览，说明了如何在主构造函数中使用这个技术来验证`lastName`，确保其不为 null：

```cs
new Person ("Alice", null);   // throws exception

class Person (string firstName, string lastName)
{
  readonly string lastName = (lastName == null)
 ? throw new ArgumentNullException ("lastName")
 : lastName;
}
```

（请记住，在字段或属性初始化器中的代码是在对象构造时执行的，而不是在访问字段或属性时执行。）同样的技术也可以将`lastName`公开为一个只读属性：

```cs
public string LastName { get; } = (lastName == null)
  ? throw new ArgumentNullException ("lastName")
  : lastName;
```

## 静态构造函数

静态构造函数每个*类型*只执行一次，而不是每个*实例*执行一次。类型可以定义一个静态构造函数，它必须是无参数的，并且与类型同名：

```cs
class Test
{
  static Test() { Console.Write ("Type Initialized"); }
}
```

在类型被使用之前，运行时会自动调用静态构造函数。这有两个触发条件：实例化该类型和访问该类型中的静态成员。

###### 警告

如果静态构造函数抛出未处理的异常，该类型在应用程序的生命周期内将变得*不可用*。

###### 注意

从 C# 9 开始，您还可以定义*模块初始化器*，它在程序集加载时执行一次（当程序集首次加载时）。要定义模块初始化器，编写一个静态 void 方法，然后将`[ModuleInitializer]`属性应用于该方法：

```cs
[System.Runtime.CompilerServices.ModuleInitializer]
internal static void InitAssembly()
{
  ...
}
```

静态字段初始化器在调用静态构造函数之前执行。如果类型没有静态构造函数，则静态字段初始化器将在类型被使用之前——或者在运行时的任何早期时间——执行。

## 静态类

标记为`static`的类无法实例化或继承，并且必须完全由静态成员组成。`Sys⁠tem.​Console`和`System.Math`类是静态类的良好示例。

## 终结器

*终结器*是仅在类中执行的方法，在垃圾收集器回收未引用对象的内存之前执行。终结器的语法是类名前缀加上`~`符号：

```cs
class Class1
{
  ~Class1() { ... }
}
```

C#将终结器翻译成一个方法，该方法重写`object`类中的`Finalize`方法。我们在《C# 12 入门》的第十二章中详细讨论了垃圾收集和终结器。

使用表达式体语法可以编写单语句终结器。

## 部分类型和方法

*部分类型*允许将类型定义拆分-通常跨多个文件。一个常见的场景是从其他源（例如 Visual Studio 模板）自动生成部分类，并通过额外手动编写的方法来增强该类。例如：

```cs
// PaymentFormGen.cs - autogenerated
partial class PaymentForm { ... }

// PaymentForm.cs - hand-authored
partial class PaymentForm { ... }
```

每个参与者必须有`partial`声明。

参与者不能有冲突的成员。例如，具有相同参数的构造函数不能重复。部分类型完全由编译器解析，这意味着每个参与者必须在编译时可用，并且必须驻留在同一个程序集中。

可以在单个参与者或多个参与者上指定基类（只要您指定的基类相同）。此外，每个参与者可以独立指定要实现的接口。我们在“继承”和“接口”中介绍基类和接口。

### 部分方法

部分类型可以包含*部分方法*。这些方法让自动生成的部分类型为手动编写提供可定制的钩子。例如：

```cs
partial class PaymentForm    // In autogenerated file
{
 partial void ValidatePayment (decimal amount);
}

partial class PaymentForm    // In hand-authored file
{
 partial void ValidatePayment (decimal amount)
 {
 if (amount > 100) Console.Write ("Expensive!");
 }
}
```

部分方法由两部分组成：*定义*和*实现*。定义通常由代码生成器编写，实现通常手动编写。如果未提供实现，部分方法的定义将被编译消除（调用它的代码也将被消除）。这允许自动生成的代码在提供钩子时更加自由，而无需担心膨胀。部分方法必须是`void`，并且隐式为`private`。

### 扩展的部分方法

*扩展的部分方法*（来自 C# 9）旨在用于反向代码生成场景，程序员在其中定义代码生成器实现的钩子。可能发生这种情况的一个例子是*源生成器*，这是 Roslyn 的一个功能，允许您向编译器提供一个自动生成代码部分的程序集。

如果部分方法声明以可访问性修饰符开头，则它是*扩展的*。

```cs
public partial class Test
{
  public partial void M1();   // Extended partial method
  private partial void M2();  // Extended partial method
}
```

可访问性修饰符的存在不仅影响可访问性：它告诉编译器以不同方式处理声明。

扩展的部分方法*必须*有实现；如果未实现，它们不会消失。在这个例子中，`M1`和`M2`都必须有实现，因为它们各自指定了可访问性修饰符（`public`和`private`）。

由于它们不会消失，扩展的部分方法可以返回任何类型，并且可以包含`out`参数。

## `nameof`运算符

`nameof`运算符将任何符号（类型、成员、变量等）的名称作为字符串返回：

```cs
int count = 123;
string name = nameof (count);       // name is "count"
```

它相对于简单指定字符串的优势在于静态类型检查。诸如 Visual Studio 之类的工具可以理解符号引用，因此如果您重命名所涉及的符号，所有引用也将被重命名。

要指定类型成员的名称，比如字段或属性，也要包括类型。这适用于静态成员和实例成员：

```cs
string name = nameof (StringBuilder.Length);
```

这将评估为`"Length"`。要返回`"StringBuilder.Length"`，你需要这样做：

```cs
nameof(StringBuilder)+"."+nameof(StringBuilder.Length);
```

