# 第三章\. 在 C# 中创建类型

在本章中，我们深入探讨了类型和类型成员。

# 类

类是最常见的引用类型。最简单的类声明如下：

```cs
class YourClassName
{
}
```

更复杂的类可选包括以下内容：

| 在关键字 `class` 前面 | *属性* 和 *类修饰符*。非嵌套类修饰符包括 `public`、`internal`、`abstract`、`sealed`、`static`、`unsafe` 和 `partial`。 |
| --- | --- |
| 在 `YourClassName` 之后 | *泛型类型参数* 和 *约束*，一个 *基类* 和 *接口*。 |
| 在大括号内 | *类成员*（这些是 *方法*、*属性*、*索引器*、*事件*、*字段*、*构造函数*、*重载操作符*、*嵌套类型* 和 *终结器*）。 |

本章覆盖了所有这些结构，除了属性、操作符函数和 `unsafe` 关键字，这些在 第四章 中详细讨论。以下各节列举了每个类成员。

## 字段

*字段* 是类或结构的成员变量；例如：

```cs
class Octopus
{
  string name;
  public int Age = 10;
}
```

字段允许以下修饰符：

| 静态修饰符 | `static` |
| --- | --- |
| 访问修饰符 | `public internal private protected` |
| 继承修饰符 | `new` |
| 不安全代码修饰符 | `unsafe` |
| 只读修饰符 | `readonly` |
| 线程修饰符 | `volatile` |

私有字段有两种常用的命名约定：小驼峰式（例如，`firstName`），以及带下划线的小驼峰式（`_firstName`）。后一种约定使你能够立即区分私有字段和参数以及局部变量。

### 只读修饰符

`readonly` 修饰符防止字段在构造后被修改。只读字段只能在其声明中或在封闭类型的构造函数中赋值。

### 字段初始化

字段初始化是可选的。未初始化的字段具有默认值（`0`、`'\0'`、`null`、`false`）。字段初始化器在构造函数之前运行：

```cs
public int Age = 10;
```

字段初始化器可以包含表达式并调用方法：

```cs
static readonly string TempFolder = System.IO.Path.GetTempPath();
```

### 声明多个字段一起

为了方便起见，你可以在逗号分隔的列表中声明多个相同类型的字段。这是所有字段共享相同属性和字段修饰符的便捷方式：

```cs
static readonly int legs = 8,
                    eyes = 2;
```

## 常量

*常量* 在编译时静态评估，并且编译器在使用时字面上替换它的值（有点像 C++ 中的宏）。常量可以是 `bool`、`char`、`string`，任何内置数值类型，或者枚举类型。

使用 `const` 关键字声明常量，并且必须用一个值进行初始化。例如：

```cs
public class Test
{
  public const string Message = "Hello World";
}
```

常量可以类似于 `static readonly` 字段，但限制更多——包括可以使用的类型以及字段初始化语义。常量与 `static readonly` 字段的不同之处在于常量的评估发生在编译时；因此

```cs
public static double Circumference (double radius)
{
  return 2 * System.Math.PI * radius;
}
```

编译为

```cs
public static double Circumference (double radius)
{
  return 6.2831853071795862 * radius;
}
```

将`PI`定义为常量是有意义的，因为其值在编译时确定。相比之下，`static readonly`字段的值可能每次程序运行时都有所不同：

```cs
static readonly DateTime StartupTime = DateTime.Now;
```

###### 注意

当向其他装配件暴露可能在后续版本中更改的值时，`static readonly`字段也很有优势。例如，假设装配件`X`将常量暴露如下：

```cs
public const decimal ProgramVersion = 2.3;
```

如果装配件`Y`引用`X`并使用此常量，则编译时装配件`Y`将使用值`2.3`。这意味着如果稍后使用常量重新编译`X`为 2.4，则`Y`仍将使用旧值 2.3，*直到*重新编译`Y`。使用`static readonly`字段可以避免此问题。

另一种看待这个问题的方式是，任何可能在将来改变的值从定义上来说都不是常量；因此，不应该将其表示为常量。

常量也可以声明为方法的局部变量：

```cs
void Test()
{
  const double twoPI = 2 * System.Math.PI;
  ...
}
```

非局部常量允许以下修饰符：

| 访问修饰符 | `public internal private protected` |
| --- | --- |
| 继承修饰符 | `new` |

## 方法

方法通过一系列语句执行操作。方法可以通过指定*参数*和*返回类型*从调用者那里接收*输入*数据，并将*输出*数据返回给调用者。方法可以指定`void`返回类型，表示不向其调用者返回任何值。方法还可以通过`ref`/`out`参数向调用者输出数据。

方法的*签名*在类型内必须是唯一的。方法的签名由其名称和参数类型按顺序组成（但不包括参数*名称*，也不包括返回类型）。

方法允许以下修饰符：

| 静态修饰符 | `static` |
| --- | --- |
| 访问修饰符 | `public internal private protected` |
| 继承修饰符 | `new virtual abstract override sealed` |
| 部分方法修饰符 | `partial` |
| 未管理代码修饰符 | `unsafe extern` |
| 异步代码修饰符 | `async` |

### 表达式主体方法

方法可以由单个表达式组成，例如

```cs
int Foo (int x) { return x * 2; }
```

可以更简洁地编写为*表达式主体方法*。一个箭头符号替代大括号和`return`关键字：

```cs
int Foo (int x) => x * 2;
```

表达式主体函数也可以具有`void`返回类型：

```cs
void Foo (int x) => Console.WriteLine (x);
```

### 局部方法

可在另一个方法内定义方法：

```cs
void WriteCubes()
{
  Console.WriteLine (Cube (3));
  Console.WriteLine (Cube (4));
  Console.WriteLine (Cube (5));

  int Cube (int value) => value * value * value;
}
```

局部方法（本例中的`Cube`）仅对封闭方法（`WriteCubes`）可见。这简化了包含类型，并立即向查看代码的任何人表明`Cube`没有其他用途。局部方法的另一个好处是可以访问封闭方法的局部变量和参数。我们将详细描述这些的几个后果，见“捕获外部变量”。

局部方法可以出现在其他函数类型中，例如属性访问器、构造函数等。甚至可以将局部方法放在其他局部方法和使用语句块的 lambda 表达式中（第四章）。局部方法可以是迭代器（第四章）或异步的（第十四章）。

### 静态局部方法

将`static`修饰符添加到局部方法（从 C# 8 开始）可以防止其访问封闭方法的局部变量和参数。这有助于减少耦合并防止局部方法意外引用包含方法中的变量。

### 局部方法和顶层语句

在顶层语句中声明的任何方法都视为局部方法。这意味着（除非标记为`static`），它们可以访问顶层语句中的变量：

```cs
int x = 3;
Foo();

void Foo() => Console.WriteLine (x);
```

### 方法重载

###### 警告

局部方法不能被重载。这意味着在顶层语句中声明的方法（视为局部方法）不能被重载。

类型可以*重载*方法（定义多个具有相同名称的方法），只要签名不同。例如，以下方法可以在同一类型中并存：

```cs
void Foo (int x) {...}
void Foo (double x) {...}
void Foo (int x, float y) {...}
void Foo (float x, int y) {...}
```

但是，以下方法对不能在同一类型中并存，因为返回类型和`params`修饰符不是方法签名的一部分：

```cs
void  Foo (int x) {...}
float Foo (int x) {...}           // Compile-time error

void  Goo (int[] x) {...}
void  Goo (params int[] x) {...}  // Compile-time error
```

是否参数是传值还是传引用也是签名的一部分。例如，`Foo(int)`可以与`Foo(ref int)`或`Foo(out int)`并存。但是，`Foo(ref int)`和`Foo(out int)`不能共存：

```cs
void Foo (int x) {...}
void Foo (ref int x) {...}     // OK so far
void Foo (out int x) {...}     // Compile-time error
```

## 实例构造函数

构造函数在类或结构体上运行初始化代码。构造函数的定义类似于方法，但方法名称和返回类型缩减为封闭类型的名称：

```cs
Panda p = new Panda ("Petey");   // Call constructor

public class Panda
{
  string name;                   // Define field
  public Panda (string n)        // Define constructor
  {
    name = n;                    // Initialization code (set up field)
  }
}
```

实例构造函数允许以下修饰符：

| 访问修饰符 | `public internal private protected` |
| --- | --- |
| 非托管代码修饰符 | `unsafe extern` |

单语句构造函数也可以写为表达式主体成员：

```cs
public Panda (string n) => name = n;
```

###### 注意

如果参数名称（或者任何变量名称）与字段名称冲突，您可以通过在字段前加上`this`引用来消除歧义：

```cs
public Panda (string name) => this.name = name;
```

### 构造函数重载

类或结构体可以重载构造函数。为了避免代码重复，一个构造函数可以调用另一个，使用`this`关键字：

```cs
public class Wine
{
  public decimal Price;
  public int Year;
  public Wine (decimal price) => Price = price;
  public Wine (decimal price, int year) : this (price) => Year = year;
}
```

当一个构造函数调用另一个构造函数时，*被调用的构造函数*先执行。

您可以将一个*表达式*传递到另一个构造函数中，如下所示：

```cs
public Wine (decimal price, DateTime year) : this (price, year.Year) { }
```

表达式可以访问类的静态成员，但不能访问实例成员。（这是强制执行的，因为在此阶段对象尚未通过构造函数进行初始化，因此调用它的任何方法可能会失败。）

###### 注意

这个特定的例子最好用一个具有`year`作为可选参数的单一构造函数来实现：

```cs
public Wine (decimal price, int year = 0)
{
  Price = price; Year = year;
}
```

我们将在稍后的 “对象初始化器” 中提供另一个解决方案。

### 隐式无参构造函数

对于类，只有在不定义任何构造函数时，C# 编译器才会自动生成一个无参公共构造函数。但是，一旦您定义了至少一个构造函数，无参构造函数将不再自动生成。

### 构造函数和字段初始化顺序

我们先前看到字段可以在其声明中使用默认值进行初始化：

```cs
class Player
{
  int shields = 50;   // Initialized first
  int health = 100;   // Initialized second
}
```

字段初始化发生在构造函数执行之前，并按字段的声明顺序进行。

### 非公共构造函数

构造函数不必是公共的。具有非公共构造函数的常见原因是通过静态方法调用来控制实例创建。静态方法可以用于从池中返回对象，而不是创建新对象，或者根据输入参数返回各种子类：

```cs
public class Class1
{
  Class1() {}                             // Private constructor
  public static Class1 Create (...)
  {
    // Perform custom logic here to return an instance of Class1
    ...
  }
}
```

## 解构器

解构方法（也称为*解构方法*）充当构造函数的近似对立面：构造函数通常接受一组值（作为参数）并将它们分配给字段，而解构方法则反之，将字段分配回一组变量。

解构方法必须被称为 `Deconstruct`，并且必须有一个或多个 `out` 参数，例如以下类：

```cs
class Rectangle
{
  public readonly float Width, Height;

  public Rectangle (float width, float height)
  {
    Width = width;
    Height = height;
  }

  public void Deconstruct (out float width, out float height)
  {
    width = Width;
    height = Height;
  }
}
```

以下特殊语法调用解构方法：

```cs
var rect = new Rectangle (3, 4);
(float width, float height) = rect;          // Deconstruction
Console.WriteLine (width + " " + height);    // 3 4
```

第二行是解构调用。它创建两个局部变量，然后调用 `Deconstruct` 方法。我们的解构调用等效于以下内容：

```cs
float width, height;
rect.Deconstruct (out width, out height);
```

或者：

```cs
rect.Deconstruct (out var width, out var height);
```

解构调用允许隐式类型推断，因此我们可以将我们的调用简写为这样：

```cs
(var width, var height) = rect;
```

或者简单地这样：

```cs
var (width, height) = rect;
```

###### 注意

如果您对一个或多个变量不感兴趣，可以使用 C# 的丢弃符号（`_`）：

```cs
var (_, height) = rect;
```

这比声明一个从未使用的变量更能表明您的意图。

如果您解构的变量已经定义，可以完全省略类型：

```cs
float width, height;
(width, height) = rect;
```

这称为*解构赋值*。您可以使用解构赋值来简化类的构造函数：

```cs
public Rectangle (float width, float height) =>
  (Width, Height) = (width, height);
```

通过重载 `Deconstruct` 方法，您可以为调用者提供一系列解构选项。

###### 注意

`Deconstruct` 方法可以是一个扩展方法（参见 “扩展方法”）。如果要解构您未编写的类型，这是一个有用的技巧。

从 C# 10 开始，在解构时可以混合使用现有变量和新变量：

```cs
double x1 = 0;
(x1, double y2) = rect;
```

## 对象初始化器

为了简化对象初始化，可以直接在构造之后通过*对象初始化器*设置对象的任何可访问字段或属性。例如，请考虑以下类：

```cs
public class Bunny
{
  public string Name;
  public bool LikesCarrots, LikesHumans;

  public Bunny () {}
  public Bunny (string n) => Name = n;
}
```

使用对象初始化器，您可以像以下方式实例化 `Bunny` 对象：

```cs
// Note parameterless constructors can omit empty parentheses
Bunny b1 = new Bunny { Name="Bo", LikesCarrots=true, LikesHumans=false };
Bunny b2 = new Bunny ("Bo")     { LikesCarrots=true, LikesHumans=false };
```

构造 `b1` 和 `b2` 的代码与以下完全等效：

```cs
Bunny *temp1* = new Bunny();    // *temp1* is a compiler-generated name
*temp1*.Name = "Bo";
*temp1*.LikesCarrots = true;
*temp1*.LikesHumans = false;
Bunny b1 = *temp1*;

Bunny *temp2* = new Bunny ("Bo");
*temp2*.LikesCarrots = true;
*temp2*.LikesHumans = false;
Bunny b2 = *temp2*;
```

临时变量的作用是确保在初始化过程中抛出异常时，不会得到一个半初始化的对象。

## this 引用

`this` 引用指的是实例本身。在以下示例中，`Marry`方法使用`this`来设置`partner`的`mate`字段：

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

`this` 引用还可以消除局部变量或参数与字段之间的歧义；例如：

```cs
public class Test
{
  string name;
  public Test (string name) => this.name = name;
}
```

`this`引用仅在类或结构的非静态成员中有效。

## 属性

从外部看，属性看起来像字段，但在内部它们包含逻辑，就像方法一样。例如，通过查看以下代码，您无法确定`CurrentPrice`是字段还是属性：

```cs
Stock msft = new Stock();
msft.CurrentPrice = 30;
msft.CurrentPrice -= 3;
Console.WriteLine (msft.CurrentPrice);
```

属性声明与字段类似，但添加了`get`/`set`块。以下是如何将`CurrentPrice`实现为属性的示例：

```cs
public class Stock
{
  decimal currentPrice;           // The private "backing" field

  public decimal CurrentPrice     // The public property
  {
    get { return currentPrice; }
    set { currentPrice = value; }
  }
}
```

`get`和`set`表示属性的*访问器*。当读取属性时，`get`访问器运行。它必须返回属性类型的值。当分配属性时，`set`访问器运行。它有一个名为`value`的隐式参数，类型为属性的类型，通常将其分配给私有字段（在本例中为`currentPrice`）。

虽然属性的访问方式与字段相同，但它们的不同之处在于，它们使实现者完全控制获取和设置其值。此控制使实现者能够选择所需的任何内部表示方式，而不会向属性的用户公开内部细节。在此示例中，如果`value`超出有效值范围，则`set`方法可能会引发异常。

###### 注意

在本书中，我们广泛使用公共字段，以保持示例的干净。在实际应用中，您通常会更倾向于使用公共属性而不是公共字段，以促进封装。

属性允许以下修饰符：

| 静态修饰符 | `static` |
| --- | --- |
| 访问修饰符 | `public internal private protected` |
| 继承修饰符 | `new virtual abstract override sealed` |
| 无管理代码修饰符 | `unsafe extern` |

### 只读和计算属性

如果属性仅指定了`get`访问器，则它是只读的；如果属性仅指定了`set`访问器，则它是只写的。很少使用只写属性。

属性通常具有专用的后备字段来存储基础数据。但是，属性也可以从其他数据计算得出：

```cs
decimal currentPrice, sharesOwned;

public decimal Worth
{
  get { return currentPrice * sharesOwned; }
}
```

### 表达式主体属性

您可以将只读属性（例如前面示例中的属性）更简洁地声明为*表达式主体属性*。一个粗箭头替换了所有大括号、`get`和`return`关键字：

```cs
public decimal Worth => currentPrice * sharesOwned;
```

加上一些额外的语法，`set`访问器也可以是表达式主体的：

```cs
public decimal Worth
{
  get => currentPrice * sharesOwned;
  set => sharesOwned = value / currentPrice;
}
```

### 自动属性

属性的最常见实现是仅读取和写入与属性相同类型的私有字段的 getter 和/或 setter。*自动属性*声明指示编译器提供此实现。我们可以通过将`CurrentPrice`声明为自动属性来改进本节中的第一个示例：

```cs
public class Stock
{
  ...
  public decimal CurrentPrice { get; set; }
}
```

编译器会自动生成一个私有后备字段，其名称为编译器生成的名称，不可引用。如果要将`set`访问器标记为`private`或`protected`，则可以将属性公开为其他类型的只读。自动属性是在 C# 3.0 中引入的。

### 属性初始化器

您可以像字段一样为自动属性添加*属性初始化器*：

```cs
public decimal CurrentPrice { get; set; } = 123;
```

这使得`CurrentPrice`的初始值为`123`。具有初始化程序的属性可以是只读的：

```cs
public int Maximum { get; } = 999;
```

与只读字段一样，只读自动属性也可以在类型的构造函数中分配。这在创建*不可变*（只读）类型时非常有用。

### 获取和设置的可访问性

`get`和`set`访问器可以具有不同的访问级别。此的典型用例是在`setter`上具有`internal`或`private`访问修饰符的`public`属性：

```cs
public class Foo
{
  private decimal x;
  public decimal X
  {
    get         { return x;  }
    private set { x = Math.Round (value, 2); }
  }
}
```

注意，您声明属性本身的访问级别更宽松（在本例中为`public`），并将修饰符添加到您希望*较不*访问的访问器。

### 仅初始化的设置器

从 C# 9 开始，可以使用`init`而不是`set`声明属性访问器：

```cs
public class Note
{
  public int Pitch    { get; init; } = 20;   // “Init-only” property
  public int Duration { get; init; } = 100;  // “Init-only” property
}
```

这些*仅初始化*属性的作用类似于只读属性，但也可以通过对象初始化器进行设置：

```cs
var note = new Note { Pitch = 50 };
```

之后，属性无法更改：

```cs
note.Pitch = 200;  // Error – init-only setter!
```

除了通过属性初始化器、构造函数或另一个仅初始化访问器，仅初始化属性甚至不能从其类内部设置。

仅初始化属性的替代方法是通过构造函数填充的只读属性：

```cs
public class Note
{
  public int Pitch    { get; }
  public int Duration { get; }

  public Note (int pitch = 20, int duration = 100)
  {
    Pitch = pitch; Duration = duration;
  }
}
```

如果类是公共库的一部分，则此方法使得后期在构造函数中添加可选参数时版本控制变得困难，因为这会破坏与消费者的二进制兼容性（而添加新的仅初始化属性则不会破坏任何内容）。

###### 注意

仅初始化属性还有另一个重要优势，即在与记录结合使用时允许非破坏性变异（参见“记录”）。

与普通`set`访问器一样，仅初始化访问器也可以提供一个实现：

```cs
public class Note
{
  readonly int _pitch;
  public int Pitch { get => _pitch; init => _pitch = value; }
  ...
```

注意，`_pitch`字段是只读的：仅初始化的设置器允许修改其自身类中的`readonly`字段。（如果没有此功能，`_pitch`将需要可写，并且类将无法内部实现不可变性。）

###### 警告

将属性的访问器从`init`更改为`set`（或反之）是*二进制破坏性更改*：任何引用您程序集的人都需要重新编译他们的程序集。

在创建完全不可变类型时，这应该不是问题，因为您的类型永远不需要具有（可写）`set`访问器的属性。

### CLR 属性实现

C# 属性访问器在内部编译为称为`get_*XXX*`和`set_*XXX*`的方法：

```cs
public decimal get_CurrentPrice {...}
public void set_CurrentPrice (decimal value) {...}
```

`init` 访问器处理类似于 `set` 访问器，但是在 `set` 访问器的“modreq”元数据中编码了一个额外的标志（请参阅 “仅初始化属性”）。

Just-In-Time (JIT) 编译器通过内联简单的非虚拟属性访问器，消除了访问属性和字段之间的任何性能差异。内联是一种优化技术，其中方法调用被该方法的主体替换。

## 索引器

Indexers 提供了一种自然的语法，用于访问封装了值列表或字典的类或结构体中的元素。Indexers 类似于属性，但通过索引参数访问，而不是属性名。`string` 类具有一个索引器，允许您通过 `int` 索引访问其每个 `char` 值：

```cs
string s = "hello";
Console.WriteLine (s[0]); // 'h'
Console.WriteLine (s[3]); // 'l'
```

使用索引器的语法类似于使用数组，不同之处在于索引参数可以是任何类型。

索引器具有与属性相同的修饰符（请参阅 “属性”）并且可以通过在方括号前插入问号来在空安全方式下调用（请参阅 “Null 操作符”）：

```cs
string s = null;
Console.WriteLine (s?[0]);  // Writes nothing; no error.
```

### 实现一个索引器

要编写一个索引器，定义一个名为 `this` 的属性，指定方括号中的参数：

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

下面是我们如何使用这个索引器的方法：

```cs
Sentence s = new Sentence();
Console.WriteLine (s[3]);       // fox
s[3] = "kangaroo";
Console.WriteLine (s[3]);       // kangaroo
```

类型可以声明多个索引器，每个索引器具有不同类型的参数。索引器还可以接受多个参数：

```cs
public string this [int arg1, string arg2]
{
  get { ... }  set { ... }
}
```

如果省略 `set` 访问器，则索引器变为只读，并且可以使用表达式主体语法来缩短其定义：

```cs
public string this [int wordNum] => words [wordNum];
```

### CLR 索引器实现

索引器在内部编译为名为 `get_Item` 和 `set_Item` 的方法，如下所示：

```cs
public string get_Item (int wordNum) {...}
public void set_Item (int wordNum, string value) {...}
```

### 使用索引和范围与索引器

您可以通过定义一个具有 `Index` 或 `Range` 参数类型的索引器来在自己的类中支持索引和范围（请参阅 “索引和范围”）。我们可以通过向 `Sentence` 类添加以下索引器来扩展我们之前的示例：

```cs
  public string this [Index index] => words [index];
  public string[] this [Range range] => words [range];
```

然后可以启用以下功能：

```cs
Sentence s = new Sentence();
Console.WriteLine (s [¹]);         // fox  
string[] firstTwoWords = s [..2];   // (The, quick)
```

## 主构造函数 (C# 12)

从 C# 12 开始，您可以在类（或结构体）声明之后直接包含一个参数列表：

```cs
class Person (string firstName, string lastName)
{
  public void Print() => Console.WriteLine (firstName + " " + lastName);
}
```

这指示编译器使用 *主构造函数参数*（`firstName` 和 `lastName`）自动构建一个 *主构造函数*，以便我们可以按以下方式实例化我们的类：

```cs
Person p = new Person ("Alice", "Jones");
p.Print();    // Alice Jones
```

主构造函数对于原型设计和其他简单场景非常有用。另一种选择是定义字段并显式编写构造函数：

```cs
class Person    // (without primary constructors)
{
  string firstName, lastName;       // Field declarations

  public Person (string firstName, string lastName)   // Constructor
  {
    this.firstName = firstName;     // Assign field
    this.lastName = lastName;       // Assign field
  }

  public void Print() => Console.WriteLine (firstName + " " + lastName);
}
```

C# 构建的构造函数被称为主构造函数，因为您选择（显式）编写的任何其他构造函数必须调用它：

```cs
class Person (string firstName, string lastName)
{
  public Person (string firstName, string lastName, int age)
    : this (firstName, lastName)   // Must call the primary constructor
  {
    ...
  }
}
```

这确保主构造函数参数始终被 *始终填充*。

###### 注意

C# 还提供了 *records*，我们在 “Records” 中介绍。记录也支持主构造函数；然而，编译器对记录采取额外步骤，并生成（默认情况下）每个主构造函数参数的公共 init-only 属性。如果需要此行为，请考虑改用记录。

主构造函数最适合简单场景，因为以下限制：

+   不能向主构造函数添加额外的初始化代码。

+   虽然很容易将主构造函数参数公开为公共属性，但除非属性是只读的，否则你无法轻松地集成验证逻辑。

主构造函数替换了 C# 否则会生成的默认无参数构造函数。

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

当此构造函数内的代码执行完毕时，参数`firstName`和`lastName`将超出作用域，并且不能随后访问。相反，主构造函数的参数不会超出作用域，并且可以在类内的任何地方访问对象的生命周期内。

###### 注意

主构造函数参数是特殊的 C# 结构，不是字段，尽管编译器在幕后确实会生成隐藏字段来存储它们的值（如果需要的话）。

### 主构造函数和字段/属性初始化器

主构造函数参数的可访问性延伸到字段和属性初始化器。在以下示例中，我们使用字段和属性初始化器将`firstName`分配给公共字段，并将`lastName`分配给公共属性：

```cs
class Person (string firstName, string lastName)
{
  public readonly string FirstName = firstName;  // Field
  public string LastName { get; } = lastName;    // Property
}
```

### 掩盖主构造函数参数

字段（或属性）可以重用主构造函数参数名称：

```cs
class Person (string firstName, string lastName)
{
  readonly string firstName = firstName;
  readonly string lastName = lastName;

  public void Print() => Console.WriteLine (firstName + " " + lastName);
}
```

在此场景中，字段或属性优先，从而掩盖主构造函数参数，*除非*在字段和属性初始化器的右侧（以粗体显示）。

###### 注意

就像普通参数一样，主构造函数参数是可写的。使用同名的`readonly`字段（如我们的示例中所示），有效地保护它们免受后续修改。

### 验证主构造函数参数

有时在字段初始化器中执行计算是有用的：

```cs
new Person ("Alice", "Jones").Print();   // Alice Jones

class Person (string firstName, string lastName)
{
  public readonly string FullName = firstName + " " + lastName;
  public void Print() => Console.WriteLine (FullName);
}
```

在下一个示例中，我们将`lastName`的大写版本保存到同名字段中（掩盖原始值）：

```cs
new Person ("Alice", "Jones").Print();   // Alice JONES

class Person (string firstName, string lastName)
{
  readonly string lastName = lastName.ToUpper();
  public void Print() => Console.WriteLine (firstName + " " + lastName);
}
```

在 “throw expressions” 中，我们描述了在遇到无效数据等情况时如何抛出异常。以下是一个预览，演示如何使用主构造函数验证`lastName`在构造时不为 null：

```cs
new Person ("Alice", null);   // throws ArgumentNullException

class Person (string firstName, string lastName)
{
  readonly string lastName = (lastName == null)
     ? throw new ArgumentNullException ("lastName")
     : lastName;
}
```

（请记住，在对象构造时执行字段或属性初始化器中的代码，而不是在访问字段或属性时执行。）在下一个示例中，我们将主构造函数参数公开为读/写属性：

```cs
class Person (string firstName, string lastName)
{
  public string LastName { get; set; } = lastName;
}
```

在这个例子中添加验证并不简单，因为你必须在两个地方验证：在（手动实现的）属性 `set` 访问器中和属性初始化器中。 （如果属性定义为仅初始化，则存在相同的问题。）在这一点上，放弃主要构造函数的捷径并显式定义构造函数和后备字段更容易。

## 静态构造函数

静态构造函数每个*类型*只执行一次，而不是每个*实例*。类型只能定义一个静态构造函数，它必须是无参数的，并且与类型同名：

```cs
class Test
{
  static Test() { Console.WriteLine ("Type Initialized"); }
}
```

运行时会在类型被使用之前自动调用静态构造函数。两件事会触发这一点：

+   实例化类型

+   访问类型中的静态成员

静态构造函数允许的唯一修饰符是 `unsafe` 和 `extern`。

###### 警告

如果静态构造函数抛出未处理的异常（第四章），该类型将在应用程序的整个生命周期中变得*不可用*。

###### 注意

从 C# 9 开始，您还可以定义*模块初始化器*，这些初始化器在程序集每次加载时执行一次。要定义模块初始化器，请编写一个静态 `void` 方法，然后将 `[ModuleInitializer]` 属性应用于该方法：

```cs
[System.Runtime.CompilerServices.ModuleInitializer]
internal static void InitAssembly()
{
  ...
}
```

### 静态构造函数和字段初始化顺序

静态字段初始化器在调用静态构造函数之前运行。如果类型没有静态构造函数，则静态字段初始化器将在类型被使用之前执行——或者在运行时任意更早的时间执行。

静态字段初始化器按照字段声明的顺序运行。以下示例说明了这一点。`X` 被初始化为 `0`，而 `Y` 被初始化为 `3`：

```cs
class Foo
{
  public static int X = Y;    // 0
  public static int Y = 3;    // 3
}
```

如果我们调换两个字段初始化器的顺序，两个字段都将初始化为 3。下一个示例将打印出 0，然后是 3，因为在将 `X` 初始化为 `3` 之前实例化 `Foo` 的字段初始化器先执行：

```cs
Console.WriteLine (Foo.X);    // 3

class Foo
{
  public static Foo Instance = new Foo();
  public static int X = 3;

  Foo() => Console.WriteLine (X);    // 0
}
```

如果我们交换粗体部分的两行，则示例将打印出 3 后面跟着 3。

## 静态类

标记为`static`的类不能实例化或派生，并且必须仅由静态成员组成。 `System.Console` 和 `System.Math` 类是静态类的良好示例。

## 终结器

终结器是仅限于类的方法，在垃圾收集器回收未引用对象的内存之前执行。终结器的语法是类名前缀为 `~` 符号：

```cs
class Class1
{
  ~Class1()
  {
    ...
  }
}
```

这实际上是 C# 中覆盖 `Object` 的 `Finalize` 方法的语法，编译器将其扩展为以下方法声明：

```cs
protected override void Finalize()
{
  ...
  base.Finalize();
}
```

我们在第十二章中全面讨论了垃圾收集和终结器。

您可以使用表达式体语法编写单语句终结器：

```cs
~Class1() => Console.WriteLine ("Finalizing");
```

## 部分类型和方法

部分类型允许将类型定义分割-通常跨多个文件。常见的情况是从其他来源（如 Visual Studio 模板或设计者）自动生成部分类，并且该类用额外的手动编写的方法增强：

```cs
// PaymentFormGen.cs - auto-generated
partial class PaymentForm { ... }

// PaymentForm.cs - hand-authored
partial class PaymentForm { ... }
```

每个参与者必须有`partial`声明；以下内容是非法的：

```cs
partial class PaymentForm {}
class PaymentForm {}
```

参与者不能有冲突的成员。例如，具有相同参数的构造函数不能重复。部分类型完全由编译器解析，这意味着每个参与者在编译时必须可用，并且必须驻留在同一个程序集中。

您可以在一个或多个部分类声明上指定一个基类，只要基类（如果指定）相同即可。此外，每个参与者可以独立指定要实现的接口。我们在“继承”和“接口”中介绍基类和接口。

编译器不保证部分类型声明之间的字段初始化顺序。

### 部分方法

部分类型可以包含*部分方法*。这些方法允许自动生成的部分类型为手动编写提供可定制的钩子；例如：

```cs
partial class PaymentForm    // In auto-generated file
{
  ...
  partial void ValidatePayment (decimal amount);
}

partial class PaymentForm    // In hand-authored file
{
  ...
  partial void ValidatePayment (decimal amount)
  {
    if (amount > 100)
      ...
  }
}
```

部分方法由两部分组成：*定义*和*实现*。定义通常由代码生成器编写，实现通常由手动编写。如果未提供实现，则部分方法的定义将被编译器删除（调用它的代码也会被删除）。这使得自动生成的代码可以自由地提供钩子，而不必担心膨胀。部分方法必须是`void`，并且隐式为`private`。它们不能包含`out`参数。

### 扩展的部分方法

*扩展的部分方法*（来自 C# 9）旨在用于反向代码生成场景，其中程序员定义代码生成器实现的钩子。这种情况的示例是*源生成器*，这是 Roslyn 的一个功能，允许您向编译器提供一个自动生成代码部分的程序集。

如果部分方法声明以访问修饰符开头，则称为*扩展*部分方法。

```cs
public partial class Test
{
  public partial void M1();    // Extended partial method
  private partial void M2();   // Extended partial method
}
```

访问修饰符的存在不仅影响可访问性：它告诉编译器以不同的方式处理声明。

扩展的部分方法*必须*有实现；如果未实现，它们不会消失。在这个例子中，`M1`和`M2`必须都有实现，因为它们各自指定了访问修饰符（`public`和`private`）。

由于它们不会消失，扩展的部分方法可以返回任何类型，并且可以包含`out`参数：

```cs
public partial class Test
{
  public partial bool IsValid (string identifier);
  internal partial bool TryParse (string number, out int result);
}
```

## nameof 运算符

`nameof`运算符返回任何符号（类型、成员、变量等）的名称作为字符串：

```cs
int count = 123;
string name = nameof (count);       // name is "count"
```

与仅指定字符串相比，它的优势在于静态类型检查。像 Visual Studio 这样的工具可以理解符号引用，因此如果您重命名该符号，所有引用也将被重命名。

要指定类型成员（如字段或属性）的名称，请同时包含类型。这适用于静态和实例成员：

```cs
string name = nameof (StringBuilder.Length);
```

这个求值结果是`Length`。要返回`StringBuilder.Length`，你可以这样做：

```cs
nameof (StringBuilder) + "." + nameof (StringBuilder.Length);
```

# 继承

一个类可以从另一个类*继承*，以扩展或定制原始类。从类继承可以重用该类中的功能，而不是从头开始构建。一个类只能从一个类继承，但可以被多个类继承，从而形成一个类层次结构。在这个例子中，我们首先定义了一个名为`Asset`的类：

```cs
public class Asset
{
  public string Name;
}
```

接下来，我们定义了名为`Stock`和`House`的类，它们将从`Asset`继承。`Stock`和`House`获得`Asset`拥有的一切，以及它们自己定义的任何额外成员：

```cs
public class Stock : Asset   // inherits from Asset
{
  public long SharesOwned;
}

public class House : Asset   // inherits from Asset
{
  public decimal Mortgage;
}
```

下面是如何使用这些类的示例：

```cs
Stock msft = new Stock { Name="MSFT",
                         SharesOwned=1000 };

Console.WriteLine (msft.Name);         // MSFT
Console.WriteLine (msft.SharesOwned);  // 1000

House mansion = new House { Name="Mansion",
                            Mortgage=250000 };

Console.WriteLine (mansion.Name);      // Mansion
Console.WriteLine (mansion.Mortgage);  // 250000
```

*派生类*`Stock`和`House`从*基类*`Asset`继承了`Name`字段。

###### 注意

派生类也称为*子类*。

基类也称为*超类*。

## 多态性

引用是*多态*的。这意味着类型为*x*的变量可以引用一个子类*x*的对象。例如，考虑以下方法：

```cs
public static void Display (Asset asset)
{
  System.Console.WriteLine (asset.Name);
}
```

此方法可以显示`Stock`和`House`，因为它们都是`Asset`：

```cs
Stock msft    = new Stock ... ;
House mansion = new House ... ;

Display (msft);
Display (mansion);
```

多态性基于子类（`Stock`和`House`）拥有其基类（`Asset`）的所有特征。反之则不然。如果修改`Display`以接受`House`，则无法传递`Asset`：

```cs
Display (new Asset());     // Compile-time error

public static void Display (House house)         // Will not accept Asset
{
  System.Console.WriteLine (house.Mortgage);
}
```

## 引用类型转换和引用转换

对象引用可以是：

+   隐式地向上转型到基类引用

+   显式地向下转型到子类引用

在兼容的引用类型之间进行向上转型和向下转型执行*引用转换*：一个新的引用（逻辑上）被创建，指向*同一个*对象。向上转型总是成功的；向下转型仅在对象适当类型化时成功。

### 向上转型

向上转型操作会从子类引用创建一个基类引用：

```cs
Stock msft = new Stock();
Asset a = msft;              // Upcast
```

在向上转型后，变量`a`仍然引用与变量`msft`相同的`Stock`对象。被引用的对象本身不会被改变或转换：

```cs
Console.WriteLine (a == msft);        // True
```

尽管`a`和`msft`引用相同的对象，但`a`对该对象有更严格的视图：

```cs
Console.WriteLine (a.Name);           // OK
Console.WriteLine (a.SharesOwned);    // Compile-time error
```

最后一行生成了编译时错误，因为变量`a`的类型是`Asset`，即使它引用了类型为`Stock`的对象。要访问其`SharesOwned`字段，必须将`Asset`向下转型为`Stock`。

### 向下转型

向下转型操作会从基类引用创建一个子类引用：

```cs
Stock msft = new Stock();
Asset a = msft;                      // Upcast
Stock s = (Stock)a;                  // Downcast
Console.WriteLine (s.SharesOwned);   // <No error>
Console.WriteLine (s == a);          // True
Console.WriteLine (s == msft);       // True
```

与向上转型一样，只影响引用，而不是底层对象。向下转型需要显式转换，因为它在运行时可能失败：

```cs
House h = new House();
Asset a = h;               // Upcast always succeeds
Stock s = (Stock)a;        // Downcast fails: a is not a Stock
```

如果向下转换失败，将抛出`InvalidCastException`。这是*运行时类型检查*的一个例子（我们在“静态和运行时类型检查”中详细阐述此概念）。

### as 运算符

`as`运算符执行一个向下转换，如果转换失败则返回`null`（而不是抛出异常）：

```cs
Asset a = new Asset();
Stock s = a as Stock;       // s is null; no exception thrown
```

当你随后要测试结果是否为`null`时，这非常有用：

```cs
if (s != null) Console.WriteLine (s.SharesOwned);
```

###### 注意

没有这样的测试，强制转换更有优势，因为如果失败，会抛出一个更有帮助的异常。我们可以通过比较以下两行代码来说明：

```cs
long shares = ((Stock)a).SharesOwned;    // Approach #1
long shares = (a as Stock).SharesOwned;  // Approach #2
```

如果`a`不是`Stock`，第一行将抛出`InvalidCastException`，这准确描述了出错的原因。第二行抛出`NullReferenceException`，这是含糊不清的。`a`不是`Stock`，还是`a`是空的？

另一种看待它的方式是，使用转换运算符时，你在告诉编译器：“我*确定*这个值的类型；如果我错了，那么我的代码有 bug，所以抛出异常！”而使用`as`运算符时，你不确定它的类型，并希望根据运行时的结果进行分支。

`as`运算符无法执行*自定义转换*（参见“运算符重载”），也不能执行数值转换：

```cs
long x = 3 as long;    // Compile-time error
```

###### 注意

`as`和转换运算符也会执行向上转换，尽管这并不是非常有用，因为隐式转换可以完成工作。

### is 运算符

`is`运算符测试变量是否匹配*模式*。C#支持多种模式，最重要的是*类型模式*，其中类型名跟在`is`关键字后面。

在这个上下文中，`is`运算符测试引用转换是否会成功——换句话说，对象是否从指定的类（或实现接口）派生。通常用于在进行向下转换之前进行测试：

```cs
if (a is Stock)
  Console.WriteLine (((Stock)a).SharesOwned);
```

`is`运算符还在*拆箱转换*成功时返回 true（参见“对象类型”）。但它不考虑自定义或数值转换。

###### 注意

`is`运算符与 C#最近版本引入的许多其他模式一起使用。有关详细讨论，请参见“模式”。

### 引入模式变量

当你使用`is`运算符时，可以引入一个变量：

```cs
if (a is Stock s)
  Console.WriteLine (s.SharesOwned);
```

这等效于以下内容：

```cs
Stock s;
if (a is Stock)
{
  s = (Stock) a;
  Console.WriteLine (s.SharesOwned);
}
```

引入的变量可以“立即”使用，因此以下是合法的：

```cs
if (a is Stock s && s.SharesOwned > 100000)
  Console.WriteLine ("Wealthy");
```

并且它在`is`表达式之外仍然处于作用域中，允许这样：

```cs
if (a is Stock s && s.SharesOwned > 100000)
  Console.WriteLine ("Wealthy");
else
  s = new Stock();   // s is in scope

Console.WriteLine (s.SharesOwned);  // Still in scope
```

## 虚函数成员

标记为`virtual`的函数可以被子类重写，以提供专门的实现。方法、属性、索引器和事件都可以声明为`virtual`：

```cs
public class Asset
{
  public string Name;
  public virtual decimal Liability => 0;   // Expression-bodied property
}
```

（`Liability => 0`是`{ get { return 0; } }`的一种快捷方式。有关此语法的详细信息，请参见“表达式体属性”。）

子类通过应用`override`修饰符来覆盖虚方法：

```cs
public class Stock : Asset
{
  public long SharesOwned;
}

public class House : Asset
{
  public decimal Mortgage;
  public override decimal Liability => Mortgage;
}
```

默认情况下，`Asset`的`Liability`为`0`。`Stock`不需要专门化此行为。然而，`House`专门化`Liability`属性以返回`Mortgage`的值：

```cs
House mansion = new House { Name="McMansion", Mortgage=250000 };
Asset a = mansion;
Console.WriteLine (mansion.Liability);  // 250000
Console.WriteLine (a.Liability);        // 250000
```

虚方法和重写方法的签名、返回类型和可访问性必须相同。重写方法可以通过`base`关键字调用其基类实现（我们在“基类关键字”中讨论此问题）。

###### 警告

从构造函数调用虚方法可能是危险的，因为子类的作者在覆盖方法时可能不知道它们正在处理的是部分初始化的对象。换句话说，覆盖方法可能会访问尚未由构造函数初始化的字段的方法或属性。

### 协变返回类型

从 C# 9 开始，您可以覆盖一个方法（或属性的`get`访问器），使其返回一个*更具体*（子类化）的类型。例如：

```cs
public class Asset
{
  public string Name;
  public virtual Asset Clone() => new Asset { Name = Name };
}

public class House : Asset
{
  public decimal Mortgage;
  public override House Clone() => new House
                                   { Name = Name, Mortgage = Mortgage };
}
```

这是允许的，因为它不违反`Clone`必须返回`Asset`的约定：它返回一个`House`，*是*一个`Asset`（更多）。

在 C# 9 之前，您必须重写具有相同返回类型的方法：

```cs
public override Asset Clone() => new House { ... }
```

这依然有效，因为重写的`Clone`方法实例化了一个`House`而不是`Asset`。然而，要将返回的对象视为`House`，您必须执行向下转换：

```cs
House mansion1 = new House { Name="McMansion", Mortgage=250000 };
House mansion2 = (House) mansion1.Clone();
```

## 抽象类和抽象成员

声明为*抽象*的类永远不能被实例化。相反，只能实例化它的具体*子类*。

抽象类能够定义*抽象成员*。抽象成员类似于虚成员，但它们不提供默认实现。除非该子类也声明为抽象，否则该实现必须由子类提供：

```cs
public abstract class Asset
{
  // Note empty implementation
  public abstract decimal NetValue { get; }
}

public class Stock : Asset
{
  public long SharesOwned;
  public decimal CurrentPrice;

  // Override like a virtual method.
  public override decimal NetValue => CurrentPrice * SharesOwned;
}
```

## 隐藏继承成员

基类和子类可以定义相同的成员。例如：

```cs
public class A      { public int Counter = 1; }
public class B : A  { public int Counter = 2; }
```

类`B`中的`Counter`字段被称为*隐藏*类`A`中的`Counter`字段。通常情况下，当一个成员被添加到基类型之后，意外地在子类型中添加了相同的成员时，就会发生这种情况。因此，编译器会生成警告，然后按以下方式解决歧义：

+   对`A`的引用（在编译时）绑定到`A.Counter`。

+   对`B`的引用（在编译时）绑定到`B.Counter`。

偶尔，您希望有意隐藏一个成员，在这种情况下，您可以在子类中对成员应用`new`修饰符。`new`修饰符*仅仅是为了抑制编译器警告*：

```cs
public class A     { public     int Counter = 1; }
public class B : A { public new int Counter = 2; }
```

`new`修饰符向编译器和其他程序员传达了您的意图，即重复成员不是偶然。

###### 注意

C#在不同上下文中重载`new`关键字以获得独立的含义。具体来说，`new` *运算符*与`new` *成员修饰符*是不同的。

### new 与 override

考虑以下类层次结构：

```cs
public class BaseClass
{
  public virtual void Foo()  { Console.WriteLine ("BaseClass.Foo"); }
}

public class Overrider : BaseClass
{
  public override void Foo() { Console.WriteLine ("Overrider.Foo"); }
}

public class Hider : BaseClass
{
  public new void Foo()      { Console.WriteLine ("Hider.Foo"); }
}
```

`Overrider`和`Hider`之间的行为差异在以下代码中有所体现：

```cs
Overrider over = new Overrider();
BaseClass b1 = over;
over.Foo();                         // Overrider.Foo
b1.Foo();                           // Overrider.Foo

Hider h = new Hider();
BaseClass b2 = h;
h.Foo();                           // Hider.Foo
b2.Foo();                          // BaseClass.Foo
```

## 封闭函数和类

重写的函数成员可以使用`sealed`关键字封闭其实现，防止进一步的子类重写它。在我们早期的虚函数成员示例中，我们可以封闭`House`对`Liability`的实现，防止从`House`派生的类重写`Liability`，如下所示：

```cs
public sealed override decimal Liability { get { return Mortgage; } }
```

您还可以将`sealed`修饰符应用于类本身，以防止其被子类化。封闭类比封闭函数成员更常见。

尽管您可以防止重写函数成员，但不能防止成员被*隐藏*。

## `base`关键字

`base`关键字类似于`this`关键字。它有两个基本用途：

+   从子类访问重写的函数成员

+   调用基类构造函数（请参见下一节）

在这个例子中，`House`使用`base`关键字访问`Asset`的`Liability`实现：

```cs
public class House : Asset
{
  ...
  public override decimal Liability => base.Liability + Mortgage;
}
```

使用`base`关键字，我们可以*非虚拟地*访问`Asset`类的`Liability`属性。这意味着我们总是访问`Asset`类的这个属性版本，而不管实例的实际运行时类型如何。

如果`Liability`被*隐藏*而不是*重写*，同样的方法也适用。（您还可以通过在调用函数之前将类型转换为基类来访问隐藏成员。）

## 构造函数与继承

子类必须声明自己的构造函数。基类的构造函数对派生类是*可访问*的，但从不会自动*继承*。例如，如果我们定义`Baseclass`和`Subclass`如下：

```cs
public class Baseclass
{
  public int X;
  public Baseclass () { }
  public Baseclass (int x) => X = x;
}

public class Subclass : Baseclass { }
```

下面的做法是非法的：

```cs
Subclass s = new Subclass (123);
```

因此，`Subclass`必须“重新定义”想要公开的任何构造函数。不过，在这样做时，它可以通过`base`关键字调用任何基类的构造函数：

```cs
public class Subclass : Baseclass
{
  public Subclass (int x) : base (x) { }
}
```

`base`关键字的工作方式类似于`this`关键字，不同之处在于它调用基类中的构造函数。

基类构造函数始终先执行；这确保了*基本*初始化发生在*专业化*初始化之前。

### 隐式调用无参数基类构造函数

如果子类的构造函数省略了`base`关键字，则隐式调用基类型的*无参数*构造函数：

```cs
public class Baseclass
{
  public int X;
  public Baseclass() { X = 1; }
}

public class Subclass : Baseclass
{
  public Subclass() { Console.WriteLine (X); }  // 1
}
```

如果基类没有可访问的无参数构造函数，则子类被迫在其构造函数中使用`base`关键字。这意味着基类只有一个多参数构造函数时，子类会负担调用它的责任：

```cs
class Baseclass
{
   public Baseclass (int x, int y, int z, string s, DateTime d) { ... }
}

public class Subclass : Baseclass
{
  public Subclass (int x, int y, int z, string s, DateTime d)
    : base (x, y, z, s, d) { ... }
}
```

### 必需成员（C# 11）

如果在大型类层次结构中存在许多具有许多参数的构造函数，则要求子类调用基类中的构造函数可能会变得繁琐。有时，最好的解决方案是完全避免构造函数，并仅依赖对象初始化器在构造期间设置字段或属性。为了帮助解决这个问题，您可以将字段或属性标记为`required`（来自 C# 11）：

```cs
public class Asset
{
  public required string Name;
}
```

*必需成员*在构造时必须通过对象初始化器填充：

```cs
Asset a1 = new Asset { Name="House" };  // OK
Asset a2 = new Asset();                 // Error: will not compile!
```

如果你希望也写一个构造器，你可以应用`[SetsRequired​Mem⁠bers]`属性，以绕过该构造器的必需成员限制：

```cs
public class Asset
{
  public required string Name;

  public Asset() { }

  [System.Diagnostics.CodeAnalysis.SetsRequiredMembers]
  public Asset (string n) => Name = n;
}
```

消费者现在可以享受到那个构造器的便利，而没有任何的权衡：

```cs
Asset a1 = new Asset { Name = "House" };  // OK
Asset a2 = new Asset ("House");           // OK
Asset a3 = new Asset();                   // Error!
```

请注意，我们还定义了一个无参数构造函数（用于对象初始化器）。其存在还确保了子类不需要重现任何构造函数。在以下示例中，`House`类选择不实现便利构造函数：

```cs
public class House : Asset { }            // No constructor, no worries!

House h1 = new House { Name = "House" };  // OK
House h2 = new House();                   // Error!
```

### 构造函数和字段初始化顺序

当对象被实例化时，按以下顺序进行初始化：

1.  从子类到基类：

    1.  字段被初始化。

    1.  调用基类构造函数的参数被评估。

1.  从基类到子类：

    1.  构造函数体执行。

例如：

```cs
public class B
{
  int x = 1;         // Executes 3rd
  public B (int x)
  {
    ...              // Executes 4th
  }
}
public class D : B
{
  int y = 1;         // Executes 1st
  public D (int x)
    : base (x + 1)   // Executes 2nd
  {
     ...             // Executes 5th
  }
}
```

### 使用主构造函数的继承

具有主构造函数的类可以使用以下语法进行子类化：

```cs
public class Baseclass (int x) { ... }

public class Subclass (int x, int y) : Baseclass (x) { ... }
```

在以下示例中，对`Baseclass(x)`的调用等同于调用`base(x)`：

```cs
public class Subclass : Baseclass
{
  public Subclass (int x, int y) : base (x) { ... }
}
```

## 重载与解析

继承对方法重载有着有趣的影响。考虑以下两个重载：

```cs
static void Foo (Asset a) { }
static void Foo (House h) { }
```

当调用重载时，最具体的类型优先：

```cs
House h = new House (...);
Foo(h);                      // Calls Foo(House)
```

调用具体的重载是在静态（编译时）而不是在运行时确定的。以下代码调用`Foo(Asset)`，即使`a`的运行时类型是`House`：

```cs
Asset a = new House (...);
Foo(a);                      // Calls Foo(Asset)
```

###### 注意

如果你将`Asset`转换为`dynamic`（第四章），决定调用哪个重载将被推迟到运行时，并且基于对象的实际类型进行：

```cs
Asset a = new House (...);
Foo ((dynamic)a);   // Calls Foo(House)
```

# 对象类型

`object`（`System.Object`）是所有类型的最终基类。任何类型都可以向上转型为`object`。

为了说明这是如何有用的，考虑一个通用的*堆栈*。堆栈是一种基于“后进先出（LIFO）”原则的数据结构。堆栈有两个操作：*push*（将对象推入堆栈）和*pop*（从堆栈弹出对象）。以下是一个可以容纳最多 10 个对象的简单实现：

```cs
public class Stack
{
  int position;
  object[] data = new object[10];
  public void Push (object obj)   { data[position++] = obj;  }
  public object Pop()             { return data[--position]; }
}
```

因为`Stack`与对象类型一起工作，我们可以向`Stack`推送和弹出*任何类型*的实例：

```cs
Stack stack = new Stack();
stack.Push ("sausage");
string s = (string) stack.Pop();   // Downcast, so explicit cast is needed

Console.WriteLine (s);             // sausage
```

`object`是引用类型，因为它是一个类。尽管如此，值类型，如`int`，也可以被转换为`object`并且被添加到我们的堆栈中。C#的这一特性称为*类型统一*，这里进行了演示：

```cs
stack.Push (3);
int three = (int) stack.Pop();
```

当你在值类型和`object`之间进行类型转换时，CLR 必须执行一些特殊工作以弥合值类型和引用类型之间的语义差异。这个过程称为*装箱*和*拆箱*。

###### 注意

在“泛型”中，我们描述了如何改进我们的`Stack`类以更好地处理具有相同类型元素的堆栈。

## 装箱与拆箱

装箱是将值类型实例转换为引用类型实例的过程。引用类型可以是 `object` 类或接口（稍后在本章讨论）。¹ 在此示例中，我们将 `int` 装箱为对象：

```cs
int x = 9;
object obj = x;           // Box the int
```

拆箱通过将对象强制转换回原始值类型来反转操作：

```cs
int y = (int)obj;         // Unbox the int
```

拆箱需要显式转换。运行时检查所述值类型是否与实际对象类型匹配，如果检查失败，则抛出 `InvalidCastException` 异常。例如，以下代码因 `long` 与 `int` 不完全匹配而引发异常：

```cs
object obj = 9;           // 9 is inferred to be of type int
long x = (long) obj;      // InvalidCastException
```

以下操作成功：

```cs
object obj = 9;
long x = (int) obj;
```

如下所示：

```cs
object obj = 3.5;              // 3.5 is inferred to be of type double
int x = (int) (double) obj;    // x is now 3
```

在最后一个示例中，`(double)` 执行*拆箱*操作，然后 `(int)` 执行*数值转换*。

###### 注意

装箱转换在提供统一类型系统方面至关重要。然而，该系统并非完美：我们将在“泛型”中看到，数组和泛型的协变仅支持*引用转换*，而不支持*装箱转换*：

```cs
object[] a1 = new string[3];   // Legal
object[] a2 = new int[3];      // Error
```

### 装箱和拆箱的复制语义

装箱会将值类型实例*复制*到新对象中，而拆箱会将对象内容*复制*回值类型实例。在以下示例中，更改 `i` 的值不会更改其先前装箱的副本：

```cs
int i = 3;
object boxed = i;
i = 5;
Console.WriteLine (boxed);    // 3
```

## 静态和运行时类型检查

C# 程序在静态（编译时）和运行时（由 CLR 进行）都进行类型检查。

静态类型检查使编译器能够在不运行程序的情况下验证其正确性。以下代码将因编译器强制执行静态类型而失败：

```cs
int x = "5";
```

CLR 在通过引用转换或拆箱向下转换时执行运行时类型检查：

```cs
object y = "5";
int z = (int) y;          // Runtime error, downcast failed
```

运行时类型检查之所以可行，是因为堆上的每个对象内部都存储有一个小的类型标记。通过调用 `object` 的 `GetType` 方法可以检索此标记。

## `GetType` 方法和 `typeof` 运算符

在运行时，所有 C#类型都用 `System.Type` 的实例表示。获取 `System.Type` 对象有两种基本方式：

+   在实例上调用 `GetType`

+   在编译时使用 `typeof` 运算符对类型名进行操作

`GetType` 在运行时评估；`typeof` 在编译时静态评估（涉及泛型类型参数时，由 JIT 编译器解析）。

`System.Type` 具有诸如类型名称、程序集、基类型等属性：

```cs
Point p = new Point();
Console.WriteLine (p.GetType().Name);             // Point
Console.WriteLine (typeof (Point).Name);          // Point
Console.WriteLine (p.GetType() == typeof(Point)); // True
Console.WriteLine (p.X.GetType().Name);           // Int32
Console.WriteLine (p.Y.GetType().FullName);       // System.Int32

public class Point { public int X, Y; }
```

`System.Type` 还具有作为运行时反射模型入口的方法，详见第十八章。

## ToString 方法

`ToString` 方法返回类型实例的默认文本表示。所有内置类型都重写了此方法。以下是使用 `int` 类型的 `ToString` 方法的示例：

```cs
int x = 1;
string s = x.ToString();     // s is "1"
```

可以重写自定义类型的 `ToString` 方法如下：

```cs
Panda p = new Panda { Name = "Petey" };
Console.WriteLine (p);   // Petey

public class Panda
{
  public string Name;
  public override string ToString() => Name;
}
```

如果不重写 `ToString`，该方法将返回类型名称。

###### 注意

当你直接调用值类型上的*重写*的`object`成员，比如`ToString`时，不会发生装箱。只有在你进行强制类型转换时才会发生装箱：

```cs
int x = 1;
string s1 = x.ToString();    // Calling on nonboxed value
object box = x;
string s2 = box.ToString();  // Calling on boxed value
```

## 对象成员列表

下面是所有`object`的成员：

```cs
public class Object
{
  public Object();

  public extern Type GetType();

  public virtual bool Equals (object obj);
  public static bool Equals  (object objA, object objB);
  public static bool ReferenceEquals (object objA, object objB);

  public virtual int GetHashCode();

  public virtual string ToString();

  protected virtual void Finalize();
  protected extern object MemberwiseClone();
}
```

我们在“相等比较”中描述了`Equals`、`ReferenceEquals`和`GetHashCode`方法。

# 结构体

*结构体*与类似，但以下是其关键区别：

+   结构体是值类型，而类是引用类型。

+   结构体不支持继承（除了隐式从`object`派生，或更确切地说，`System.ValueType`）。

一个结构体可以拥有所有类的成员，除了析构函数。并且因为它不能被子类化，成员不能标记为虚拟的、抽象的或受保护的。

###### 警告

在 C# 10 之前，结构体进一步禁止定义字段初始化器和无参数构造函数。尽管这个禁令现在已经放宽，主要是为了支持记录结构体（见“记录”），但在定义这些结构之前，仍值得仔细考虑，因为它们可能导致混乱的行为，我们将在“结构体构造语义”中描述。

当需要值类型语义时，结构体是合适的。结构体的良好示例是数值类型，在这些类型中，赋值复制值而不是引用更为自然。因为结构体是值类型，每个实例不需要在堆上实例化对象；这在创建多个类型实例时可以节省内存。例如，创建值类型元素的数组只需要单个堆分配。

因为结构体是值类型，实例不能为 null。结构的默认值是一个空实例，所有字段都为空（设置为它们的默认值）。

## 结构体构造语义

###### 注意

在 C# 11 之前，结构体中的每个字段都必须在构造函数（或字段初始化器）中显式赋值。这个限制现在已经放宽。

### 默认构造函数

除了你定义的任何构造函数外，结构体始终有一个隐式的无参数构造函数，执行其字段的比特零化（将它们设置为它们的默认值）：

```cs
Point p = new Point();        // p.x and p.y will be 0
struct Point { int x, y; }
```

即使你定义了自己的无参数构造函数，隐式的无参数构造函数仍然存在，并且可以通过`default`关键字访问：

```cs
Point p1 = new Point();       // p1.x and p1.y will be 1
Point p2 = default;           // p2.x and p2.y will be 0

struct Point
{
  int x = 1;
  int y;
  public Point() => y = 1;
}
```

在这个例子中，我们通过字段初始化器将`x`初始化为 1，通过无参数构造函数将`y`初始化为 1。然而，使用`default`关键字，我们仍然能够创建一个绕过这两个初始化的`Point`。默认构造函数也可以通过其他方式访问，正如以下例子所示：

```cs
var points = new Point[10];   // Each point in the array will be (0,0)
var test = new Test();        // test.p will be (0,0)

class Test { Point p; }
```

###### 注意

拥有两个无参数构造函数可能会导致混淆，这可能是避免在结构体中定义字段初始化器和显式的无参数构造函数的一个好理由。

结构体的一个好策略是设计它们的 `default` 值是一个有效状态，从而使初始化变得多余。例如，而不是初始化属性如下：

```cs
public string Protocol { get; set; } = "https";
```

考虑以下内容：

```cs
struct WebOptions
{
  string protocol;
  public string Protocol { get => protocol ?? "https";
                           set => protocol = value;    }
}
```

## 只读结构体和函数

您可以将 `readonly` 修饰符应用于结构体，以强制所有字段都是 `readonly` 的；这有助于声明意图，并为编译器提供更多的优化自由度：

```cs
readonly struct Point
{
  public readonly int X, Y;   // X and Y must be readonly
}
```

如果需要在更细粒度的级别应用 `readonly`，可以将 `readonly` 修饰符（从 C# 8 开始）应用于结构体的 *函数*。这确保如果函数尝试修改任何字段，则会生成编译时错误：

```cs
struct Point
{
  public int X, Y;
  public readonly void ResetX() => X = 0;  // Error!
}
```

如果一个 `readonly` 函数调用一个非 `readonly` 函数，编译器会生成警告（并为了避免可能的突变而进行防御性地复制结构体）。

## Ref 结构体

###### 注意

在 C# 7.2 中引入了 ref 结构体作为一个主要为了 `Span<T>` 和 `ReadOnlySpan<T>` 结构体的利益而引入的特性。我们在 第二十三章 中描述了这些结构体（以及高度优化的 `Utf8JsonReader` 在 第十一章 中的描述）。这些结构体有助于一种微优化技术，旨在减少内存分配。

不像引用类型，其实例总是存在于堆上，值类型则在 *原地*（变量声明的地方）生存。如果值类型出现为参数或局部变量，它将驻留在堆栈上：

```cs
void SomeMethod()
{
  Point p;   // p will reside on the stack
}
struct Point { public int X, Y; }
```

但是，如果一个值类型作为类的字段出现，它将驻留在堆上：

```cs
class MyClass
{
  Point p;   // Lives on heap, because MyClass instances live on the heap
}
```

类似地，结构体数组存在于堆上，而装箱一个结构体会将其发送到堆上。

向结构体的声明添加 `ref` 修饰符确保它只能驻留在堆栈上。试图使用 *ref 结构体* 的方式使其可能驻留在堆上会生成编译时错误：

```cs
var points = new Point [100];           // Error: will not compile!

ref struct Point { public int X, Y; }
class MyClass    { Point P;         }   // Error: will not compile!
```

Ref 结构体主要为了 `Span<T>` 和 `ReadOnlySpan<T>` 结构体的利益而引入。因为 `Span<T>` 和 `ReadOnlySpan<T>` 实例只能存在于堆栈上，所以它们可以安全地包装堆栈分配的内存。

Ref 结构体不能参与任何直接或间接导致存在于堆上的 C# 功能。这包括我们在 第四章 中描述的许多高级 C# 功能，即 lambda 表达式、迭代器和异步函数（因为在幕后，这些功能都会创建带有字段的隐藏类）。此外，ref 结构体不能出现在非 ref 结构体内部，也不能实现接口（因为这可能导致装箱）。

# 访问修饰符

为了促进封装，类型或类型成员可以通过在声明中添加 *访问修饰符* 来限制其对其他类型和其他程序集的 *可访问性*：

`public`

完全可访问。这是枚举或接口成员的隐式可访问性。

`internal`

仅在包含程序集或友元程序集中可访问。这是非嵌套类型的默认可访问性。

`private`

仅在包含类型内部可访问。这是类或结构的成员的默认可访问性。

`protected`

仅在包含类型或子类中可访问。

`protected internal`

`protected` 和 `internal` 可访问性的*并集*。具有 `protected internal` 访问性的成员可以通过两种方式访问。

`private protected`

`protected` 和 `internal` 可访问性的*交集*。具有 `private protected` 访问性的成员仅在包含类型内部可访问，或者来自*同一程序集中的子类*（使其比单独的 `protected` 或 `internal` 更不可访问）。

`file`（来自 C# 11）

仅在同一文件内部可访问。用于*源生成器*的使用（请参阅“扩展部分方法”）。此修饰符仅适用于类型声明。

## 示例

`Class2` 可从其程序集外部访问；`Class1` 则不行：

```cs
class Class1 {}                  // Class1 is internal (default)
public class Class2 {}
```

`ClassB` 将字段 `x` 公开给同一程序集中的其他类型；`ClassA` 则不公开：

```cs
class ClassA { int x;          } // x is private (default)
class ClassB { internal int x; }
```

`Subclass` 中的函数可以调用 `Bar`，但不能调用 `Foo`：

```cs
class BaseClass
{
  void Foo()           {}        // Foo is private (default)
  protected void Bar() {}
}

class Subclass : BaseClass
{
  void Test1() { Foo(); }       // Error - cannot access Foo
  void Test2() { Bar(); }       // OK
}
```

## 友元程序集

您可以通过添加`System.Runtime.CompilerServices.InternalsVisibleTo`程序集属性，指定友元程序集的名称，来向其他*友元*程序集公开`internal`成员，如下所示：

```cs
[assembly: InternalsVisibleTo ("Friend")]
```

如果友元程序集具有强名称（请参阅第十七章），则必须指定其完整的 160 字节公钥：

```cs
[assembly: InternalsVisibleTo ("StrongFriend, PublicKey=0024f000048c...")]
```

您可以使用 LINQ 查询从强命名程序集中提取完整的公钥（我们在第八章详细解释 LINQ）：

```cs
string key = string.Join ("",
  Assembly.GetExecutingAssembly().GetName().GetPublicKey()
    .Select (b => b.ToString ("x2")));
```

###### 注意

在 LINQPad 中的伴侣示例邀请您浏览程序集，然后将程序集的完整公钥复制到剪贴板。

## 访问性限制

类型限制其声明成员的可访问性。最常见的限制示例是具有 `public` 成员的 `internal` 类型。例如，请考虑以下情况：

```cs
class C { public void Foo() {} }
```

`C` 的（默认）`internal` 可访问性限制了 `Foo` 的访问性，实际上使 `Foo` 变为 `internal`。将 `Foo` 标记为 `public` 的常见原因是，如果以后将 `C` 更改为 `public`，则易于重构。

## 访问修饰符的限制

当重写基类函数时，重写函数的可访问性必须与被重写函数相同；例如：

```cs
class BaseClass             { protected virtual  void Foo() {} }
class Subclass1 : BaseClass { protected override void Foo() {} }  // OK
class Subclass2 : BaseClass { public    override void Foo() {} }  // Error
```

（例外情况是在另一个程序集中重写 `protected internal` 方法时，此时重写必须简单地是 `protected`。）

编译器阻止任何不一致使用访问修饰符的情况。例如，子类本身可以比基类更不可访问，但不能更多：

```cs
internal class A {}
public class B : A {}          // Error
```

# 接口

接口类似于类，但仅*指定行为*，而不保存状态（数据）。因此：

+   接口只能定义函数而不能定义字段。

+   接口成员是*隐式抽象*的。（有一些例外情况，我们将在“默认接口成员”和“静态接口成员”中描述。）

+   一个类（或结构）可以实现多个接口。相比之下，一个类只能继承一个类，而结构体则根本不能继承（除了从`System.ValueType`派生）。

接口声明类似于类声明，但它（通常）不为其成员提供实现，因为其成员隐式为抽象。这些成员将由实现接口的类和结构体实现。接口只能包含函数，即方法、属性、事件和索引器（这些恰好是类的成员可以是抽象的成员）。

这是定义在`System.Collections`中的`IEnumerator`接口的定义：

```cs
public interface IEnumerator
{
  bool MoveNext();
  object Current { get; }
  void Reset();
}
```

接口成员始终是隐式公共的，不能声明访问修饰符。实现接口意味着为其所有成员提供`public`实现：

```cs
internal class Countdown : IEnumerator
{
  int count = 11;
  public bool MoveNext() => count-- > 0;
  public object Current => count;
  public void Reset() { throw new NotSupportedException(); }
}
```

您可以将对象隐式转换为其实现的任何接口：

```cs
IEnumerator e = new Countdown();
while (e.MoveNext())
  Console.Write (e.Current);      // 109876543210
```

###### 注意

即使`Countdown`是一个内部类，也可以通过将`Countdown`的实例转换为`IEnumerator`来公开调用其实现的成员。例如，如果同一程序集中的公共类型定义了以下方法：

```cs
public static class Util
{
  public static object GetCountDown() => new CountDown();
}
```

来自另一个程序集的调用者可以这样做：

```cs
IEnumerator e = (IEnumerator) Util.GetCountDown();
e.MoveNext();
```

如果`IEnumerator`本身被定义为`internal`，这是不可能的。

## 扩展一个接口

接口可以从其他接口派生；例如：

```cs
public interface IUndoable             { void Undo(); }
public interface IRedoable : IUndoable { void Redo(); }
```

`IRedoable`“继承”了`IUndoable`的所有成员。换句话说，实现`IRedoable`的类型也必须实现`IUndoable`的成员。

## 显式接口实现

实现多个接口有时会导致成员签名冲突。您可以通过*显式实现*接口成员来解决这类冲突。请考虑以下示例：

```cs
interface I1 { void Foo(); }
interface I2 { int Foo(); }

public class Widget : I1, I2
{
  public void Foo()
  {
    Console.WriteLine ("Widget's implementation of I1.Foo");
  }

  int I2.Foo()
  {
    Console.WriteLine ("Widget's implementation of I2.Foo");
    return 42;
  }
}
```

因为`I1`和`I2`具有冲突的`Foo`签名，`Widget`显式实现了`I2`的`Foo`方法。这使得两个方法可以在同一个类中共存。调用显式实现的成员的唯一方法是将其转换为其接口：

```cs
Widget w = new Widget();
w.Foo();                      // Widget's implementation of I1.Foo
((I1)w).Foo();                // Widget's implementation of I1.Foo
((I2)w).Foo();                // Widget's implementation of I2.Foo
```

另一个显式实现接口成员的原因是隐藏那些对类型的正常使用案例高度专业化和分散注意力的成员。例如，实现`ISerializable`的类型通常希望在没有显式转换为该接口的情况下避免展示其`ISerializable`成员。

## 虚拟实现接口成员

默认情况下，隐式实现的接口成员是封闭的。必须在基类中标记为`virtual`或`abstract`才能被重写：

```cs
public interface IUndoable { void Undo(); }

public class TextBox : IUndoable
{
  public virtual void Undo() => Console.WriteLine ("TextBox.Undo");
}

public class RichTextBox : TextBox
{
  public override void Undo() => Console.WriteLine ("RichTextBox.Undo");
}
```

通过基类或接口调用接口成员将调用子类的实现：

```cs
RichTextBox r = new RichTextBox();
r.Undo();                          // RichTextBox.Undo
((IUndoable)r).Undo();             // RichTextBox.Undo
((TextBox)r).Undo();               // RichTextBox.Undo
```

显式实现的接口成员不能标记为`virtual`，也不能以通常的方式被重写。但是可以*重新实现*它。

## 在子类中重新实现接口

子类可以重新实现基类已经实现的任何接口成员。当通过接口调用时，重新实现会接管成员的实现，无论基类中成员是否为`virtual`都可以工作。它还适用于成员是隐式实现还是显式实现的情况——尽管在后者的情况下效果最佳，我们将会展示。

在以下示例中，`TextBox`显式实现了`IUndoable.Undo`，因此它不能标记为`virtual`。要“覆盖”它，`RichTextBox`必须重新实现`IUndoable`的`Undo`方法：

```cs
public interface IUndoable { void Undo(); }

public class TextBox : IUndoable
{
  void IUndoable.Undo() => Console.WriteLine ("TextBox.Undo");
}

public class RichTextBox : TextBox, IUndoable
{
  public void Undo() => Console.WriteLine ("RichTextBox.Undo");
}
```

通过接口调用重新实现的成员会调用子类的实现：

```cs
RichTextBox r = new RichTextBox();
r.Undo();                 // RichTextBox.Undo      Case 1
((IUndoable)r).Undo();    // RichTextBox.Undo      Case 2
```

假设相同的`RichTextBox`定义，假设`TextBox`隐式实现了`Undo`：

```cs
public class TextBox : IUndoable
{
  public void Undo() => Console.WriteLine ("TextBox.Undo");
}
```

这将为我们提供另一种调用`Undo`的方式，这将“破坏”系统，如情况 3 所示：

```cs
RichTextBox r = new RichTextBox();
r.Undo();                 // RichTextBox.Undo      Case 1
((IUndoable)r).Undo();    // RichTextBox.Undo      Case 2
((TextBox)r).Undo();      // TextBox.Undo          Case 3
```

情况 3 表明，重新实现仅在通过接口调用成员时有效，而不是通过基类。通常情况下这是不希望出现的，因为它可能导致不一致的语义。这使得重新实现最适合作为覆盖*显式*实现的接口成员的策略。

### 接口重新实现的替代方案

即使有显式成员实现，接口的重新实现也因为几个原因而存在问题：

+   子类没有办法调用基类的方法。

+   基类作者可能没有预料到某个方法会被重新实现，也可能没有考虑到潜在的后果。

当子类化没有被预料到时，重新实现可以作为一种最后的手段。然而，更好的选择是设计一个基类，使得永远不需要重新实现。有两种方法可以实现这一点：

+   当隐式实现成员时，如果适合，将其标记为`virtual`。

+   在显式实现成员时，如果预期子类可能需要覆盖任何逻辑，请使用以下模式：

```cs
public class TextBox : IUndoable
{
  void IUndoable.Undo()         => Undo();    // Calls method below
  protected virtual void Undo() => Console.WriteLine ("TextBox.Undo");
}

public class RichTextBox : TextBox
{
  protected override void Undo() => Console.WriteLine("RichTextBox.Undo");
}
```

如果你不预期有任何子类化，可以将类标记为`sealed`以预防接口的重新实现。

## 接口和装箱

将结构转换为接口会导致装箱。在结构上调用隐式实现的成员不会导致装箱：

```cs
interface  I { void Foo();          }
struct S : I { public void Foo() {} }

...
S s = new S();
s.Foo();         // No boxing.

I i = s;         // Box occurs when casting to interface.
i.Foo();
```

## 默认接口成员

从 C# 8 开始，你可以向接口成员添加默认实现，使其成为可选实现：

```cs
interface ILogger
{
  void Log (string text) => Console.WriteLine (text);
}
```

如果你想向一个流行库中定义的接口添加成员而不破坏（可能有成千上万的）实现，这是有利的。

默认实现始终是显式的，因此如果实现`ILogger`的类未定义`Log`方法，则仅通过接口调用它的唯一方法：

```cs
class Logger : ILogger { }
...
((ILogger)new Logger()).Log ("message");
```

这可以防止多重实现继承的问题：如果一个类实现了两个接口，并且这两个接口都添加了相同的默认成员，则不会出现调用哪个成员的歧义。

## 静态接口成员

一个接口也可以声明静态成员。有两种静态接口成员：

+   静态非虚拟接口成员

+   静态虚拟/抽象接口成员

###### 注意

与*实例*成员相反，接口上的静态成员默认为非虚拟的。要使静态接口成员虚拟化，必须将其标记为`static abstract`或`static virtual`。

### 静态非虚拟接口成员

静态非虚拟接口成员主要用于帮助编写默认接口成员。它们不是由类或结构实现的；而是直接被使用。除了方法、属性、事件和索引器外，静态非虚拟成员还允许字段，这些字段通常从默认成员实现内部访问：

```cs
interface ILogger
{
  void Log (string text) => 
    Console.WriteLine (Prefix + text);

  static string Prefix = ""; 
}
```

静态非虚拟接口成员默认为公共的，因此可以从外部访问：

```cs
ILogger.Prefix = "File log: ";
```

您可以通过添加可访问性修饰符（如`private`、`protected`或`internal`）来限制此内容。

实例字段（仍然）是禁止的。这与接口的原则一致，即定义*行为*而非*状态*。

### 静态虚拟/抽象接口成员

静态虚拟/抽象接口成员（从 C# 11 起）支持*静态多态*，这是我们将在第四章中讨论的高级功能。静态虚拟接口成员标记为`static abstract`或`static virtual`：

```cs
interface ITypeDescribable
{
  static abstract string Description { get; }
  static virtual string Category => null;
}
```

实现类或结构必须实现静态抽象成员，并可以选择实现静态虚拟成员：

```cs
class CustomerTest : ITypeDescribable
{
  public static string Description => "Customer tests";  // Mandatory
  public static string Category    => "Unit testing";    // Optional
}
```

除了方法、属性和事件外，运算符和转换也是静态虚拟接口成员的合法目标（参见“运算符重载”）。静态虚拟接口成员通过约束类型参数调用；我们将在本章后面的“静态多态”和“泛型数学”中进行演示。

# 枚举

枚举是一种特殊的值类型，允许您指定一组命名的数值常量。例如：

```cs
public enum BorderSide { Left, Right, Top, Bottom }
```

我们可以如下使用此枚举类型：

```cs
BorderSide topSide = BorderSide.Top;
bool isTop = (topSide == BorderSide.Top);   // true
```

每个枚举成员都有一个基础的整数值。这些默认为：

+   基础值的类型为`int`。

+   常量`0`、`1`、`2`...会自动分配，按照枚举成员的声明顺序。

您可以按以下方式指定替代整数类型：

```cs
public enum BorderSide : byte { Left, Right, Top, Bottom }
```

您还可以为每个枚举成员指定显式基础值：

```cs
public enum BorderSide : byte { Left=1, Right=2, Top=10, Bottom=11 }
```

###### 注意

编译器还允许您显式分配*部分*枚举成员。未分配的枚举成员会从最后一个显式值递增。前面的示例等同于以下内容：

```cs
public enum BorderSide : byte
 { Left=1, Right, Top=10, Bottom }
```

## 枚举转换

您可以使用显式强制转换将枚举实例转换为其基础整数值，反之亦然：

```cs
int i = (int) BorderSide.Left;
BorderSide side = (BorderSide) i;
bool leftOrRight = (int) side <= 2;
```

您还可以将一个枚举类型显式转换为另一个枚举类型。假设 `HorizontalAlignment` 定义如下：

```cs
public enum HorizontalAlignment
{
  Left = BorderSide.Left,
  Right = BorderSide.Right,
  Center
}
```

枚举类型之间的转换使用底层整数值：

```cs
HorizontalAlignment h = (HorizontalAlignment) BorderSide.Right;
// same as:
HorizontalAlignment h = (HorizontalAlignment) (int) BorderSide.Right;
```

数字文字 `0` 在枚举表达式中由编译器特别处理，不需要显式转换：

```cs
BorderSide b = 0;    // No cast required
if (b == 0) ...
```

`0` 有两个特殊处理原因：

+   枚举的第一个成员通常用作“默认”值。

+   对于 *组合枚举* 类型，`0` 表示“无标志”。

## 标记枚举

您可以组合枚举成员。为防止歧义，组合枚举的成员需要显式分配值，通常是二的幂：

```cs
[Flags]
enum BorderSides { None=0, Left=1, Right=2, Top=4, Bottom=8 }
```

或者：

```cs
enum BorderSides { None=0, Left=1, Right=1<<1, Top=1<<2, Bottom=1<<3 }
```

要处理组合枚举值，可以使用位运算符（如 `|` 和 `&`）。这些运算符作用于底层整数值：

```cs
BorderSides leftRight = BorderSides.Left | BorderSides.Right;

if ((leftRight & BorderSides.Left) != 0)
  Console.WriteLine ("Includes Left");     // Includes Left

string formatted = leftRight.ToString();   // "Left, Right"

BorderSides s = BorderSides.Left;
s |= BorderSides.Right;
Console.WriteLine (s == leftRight);   // True

s ^= BorderSides.Right;               // Toggles BorderSides.Right
Console.WriteLine (s);                // Left
```

按照惯例，当枚举类型的成员可以组合时，应始终将 `Flags` 属性应用于枚举类型。如果在不带 `Flags` 属性的情况下声明这样的 `enum`，仍然可以组合成员，但在枚举实例上调用 `ToString` 时将输出数字而不是一系列名称。

按照惯例，可组合的枚举类型使用复数而不是单数名称。

为了方便起见，您可以在枚举声明本身中包含组合成员：

```cs
[Flags]
enum BorderSides
{
  None=0,
  Left=1, Right=1<<1, Top=1<<2, Bottom=1<<3,
  LeftRight = Left | Right, 
  TopBottom = Top  | Bottom,
  All       = LeftRight | TopBottom
}
```

## 枚举运算符

与枚举一起使用的运算符有：

```cs
=   ==   !=   <   >   <=   >=   +   -   ^  &  |   ˜
+=   -=   ++  --   sizeof
```

位运算、算术和比较运算符返回处理底层整数值的结果。可以在枚举和整数类型之间进行加法运算，但不能在两个枚举之间进行加法运算。

## 类型安全问题

考虑以下枚举：

```cs
public enum BorderSide { Left, Right, Top, Bottom }
```

因为枚举可以在其基础整数类型之间进行强制转换，所以它可以具有的实际值可能超出合法枚举成员的范围：

```cs
BorderSide b = (BorderSide) 12345;
Console.WriteLine (b);                // 12345
```

位运算和算术运算符可能产生类似的无效值：

```cs
BorderSide b = BorderSide.Bottom;
b++;                                  // No errors
```

无效的 `BorderSide` 将破坏以下代码：

```cs
void Draw (BorderSide side)
{
  if      (side == BorderSide.Left)  {...}
  else if (side == BorderSide.Right) {...}
  else if (side == BorderSide.Top)   {...}
  else                               {...} // Assume BorderSide.Bottom
}
```

一个解决方法是添加另一个 `else` 子句：

```cs
  ...
  else if (side == BorderSide.Bottom) ...
  else throw new ArgumentException ("Invalid BorderSide: " + side, "side");
```

另一个解决方法是显式检查枚举值的有效性。静态的 `Enum.IsDefined` 方法完成这项工作：

```cs
BorderSide side = (BorderSide) 12345;
Console.WriteLine (Enum.IsDefined (typeof (BorderSide), side));   // False
```

不幸的是，`Enum.IsDefined` 对于标记的枚举不起作用。但是，以下辅助方法（依赖于 `Enum.ToString()` 的行为）如果给定的标记枚举有效则返回 `true`：

```cs
for (int i = 0; i <= 16; i++)
{
  BorderSides side = (BorderSides)i;
  Console.WriteLine (IsFlagDefined (side) + " " + side);
}

bool IsFlagDefined (Enum e)
{
  decimal d;
  return !decimal.TryParse(e.ToString(), out d);
}

[Flags]
public enum BorderSides { Left=1, Right=2, Top=4, Bottom=8 }
```

# 嵌套类型

*嵌套类型* 是在另一个类型的范围内声明的：

```cs
public class TopLevel
{
  public class Nested { }               // Nested class
  public enum Color { Red, Blue, Tan }  // Nested enum
}
```

嵌套类型具有以下特点：

+   它可以访问封闭类型的私有成员以及封闭类型可以访问的其他所有内容。

+   您可以使用全范围的访问修饰符声明它，而不仅仅是 `public` 和 `internal`。

+   嵌套类型的默认访问权限是 `private` 而不是 `internal`。

+   访问封闭类型外部的嵌套类型需要使用封闭类型的名称进行限定（类似访问静态成员时的方式）。

例如，要从外部访问我们的 `TopLevel` 类中的 `Color.Red`，我们需要这样做：

```cs
TopLevel.Color color = TopLevel.Color.Red;
```

所有类型（类、结构体、接口、委托和枚举）都可以嵌套在类或结构体内部。

这里是从嵌套类型访问类型的私有成员的示例：

```cs
public class TopLevel
{
  static int x;
  class Nested
  {
    static void Foo() { Console.WriteLine (TopLevel.x); }
  }
}
```

下面是将`protected`访问修饰符应用于嵌套类型的示例：

```cs
public class TopLevel
{
  protected class Nested { }
}

public class SubTopLevel : TopLevel
{
  static void Foo() { new TopLevel.Nested(); }
}
```

这里是从封闭类型外部引用嵌套类型的示例：

```cs
public class TopLevel
{
  public class Nested { }
}

class Test
{
  TopLevel.Nested n;
}
```

编译器本身在生成私有类时广泛使用嵌套类型，这些类为迭代器和匿名方法等构造捕获状态。

###### 注意

如果使用嵌套类型的唯一原因是避免在命名空间中拥有太多类型，考虑使用嵌套命名空间。应当使用嵌套类型是因为它更强的访问控制限制，或者当嵌套类必须访问包含类的私有成员时。

# 泛型

C# 有两种分离的机制用于编写可在不同类型之间重复使用的代码：*继承* 和 *泛型*。继承通过基类型表达可重用性，而泛型通过包含“占位符”类型的“模板”表达可重用性。与继承相比，泛型可以*增加类型安全性* 和 *减少强制转换和装箱*。

###### 注意

C# 的泛型和 C++ 的模板是类似的概念，但它们的工作方式不同。我们在 “C# Generics Versus C++ Templates” 中解释了这种区别。

## 泛型类型

泛型类型声明*类型参数* ——由泛型类型的使用者提供*类型参数*填充的占位类型。这里是一个泛型类型`Stack<T>`的例子，设计用于堆叠`T`类型的实例。`Stack<T>`声明了一个单一的类型参数`T`：

```cs
public class Stack<T>
{
  int position;
  T[] data = new T[100];
  public void Push (T obj)  => data[position++] = obj;
  public T Pop()            => data[--position];
}
```

我们可以像这样使用`Stack<T>`：

```cs
var stack = new Stack<int>();
stack.Push (5);
stack.Push (10);
int x = stack.Pop();        // x is 10
int y = stack.Pop();        // y is 5
```

`Stack<int>`用类型参数`T`隐式地创建了一个类型（合成发生在运行时）。然而，试图将字符串推入我们的`Stack<int>`将会产生编译时错误。`Stack<int>`实际上有以下定义（为避免混淆，类名以粗体标出）：

```cs
public class ###
{
  int position;
  int[] data = new int[100];
  public void Push (int obj)  => data[position++] = obj;
  public int Pop()            => data[--position];
}
```

技术上，我们称`Stack<T>`为*开放类型*，而`Stack<int>`为*闭合类型*。在运行时，所有泛型类型实例都是闭合的，即用具体的类型参数填充了占位符类型。这意味着以下语句是非法的：

```cs
var stack = new Stack<T>();   // Illegal: What is T?
```

然而，如果它在一个自身定义了`T`作为类型参数的类或方法内，这是合法的：

```cs
public class Stack<T>
{
  ...
  public Stack<T> Clone()
  {
    Stack<T> clone = new Stack<T>();   // Legal
    ...
  } 
}
```

## 泛型的存在原因

泛型存在的目的是编写可在不同类型之间重复使用的代码。假设我们需要一个整数堆栈，但没有泛型类型。一种解决方案是为每种所需的元素类型硬编码一个单独的版本（例如，`IntStack`、`StringStack`等）。显然，这将导致大量的代码重复。另一种解决方案是编写一个使用`object`作为元素类型泛化的堆栈：

```cs
public class ObjectStack
{
  int position;
  object[] data = new object[10];
  public void Push (object obj) => data[position++] = obj;
  public object Pop()           => data[--position];
}
```

然而，`ObjectStack` 并不像硬编码的 `IntStack` 那样适用于专门堆叠整数。`ObjectStack` 需要装箱和向下转型，这些在编译时无法检查：

```cs
// Suppose we just want to store integers here:
ObjectStack stack = new ObjectStack();

stack.Push ("s");          // Wrong type, but no error!
int i = (int)stack.Pop();  // Downcast - runtime error
```

我们需要的是一个通用的堆栈实现，可以适用于所有元素类型，并且可以轻松专门化到特定的元素类型以提高类型安全性并减少转型和装箱。通用类型通过允许我们参数化元素类型来实现这一点。`Stack<T>` 同时具有 `ObjectStack` 和 `IntStack` 的优点。像 `ObjectStack` 一样，`Stack<T>` 一次编写即可*普遍*适用于所有类型。像 `IntStack` 一样，`Stack<T>` *专门化*于特定类型——其美妙之处在于这种类型是 `T`，我们可以动态替换。

###### 注意

`ObjectStack` 在功能上等同于 `Stack<object>`。

## 通用方法

通用方法在方法的签名内声明类型参数。

使用通用方法，可以以通用方式实现许多基本算法。以下是一个通用方法示例，用于交换任意类型 `T` 的两个变量的内容：

```cs
static void Swap<T> (ref T a, ref T b)
{
  T temp = a;
  a = b;
  b = temp;
}
```

调用 `Swap<T>` 如下所示：

```cs
int x = 5;
int y = 10;
Swap (ref x, ref y);
```

通常情况下，不需要为通用方法提供类型参数，因为编译器可以隐式推断类型。如果存在歧义，可以如下调用通用方法带有类型参数：

```cs
Swap<int> (ref x, ref y);
```

在通用*类型*中，除非它*引入*类型参数（使用尖括号语法），否则方法不被归类为通用方法。我们的通用堆栈中的 `Pop` 方法仅使用类型的现有类型参数 `T`，不被归类为通用方法。

方法和类型是唯一可以引入类型参数的结构。属性、索引器、事件、字段、构造函数、运算符等无法声明类型参数，尽管它们可以参与其封闭类型已声明的任何类型参数。例如，在我们的通用堆栈示例中，我们可以编写一个索引器来返回通用项：

```cs
public T this [int index] => data [index];
```

类似地，构造函数可以参与现有的类型参数，但不能*引入*它们：

```cs
public Stack<T>() { }   // Illegal
```

## 声明类型参数

类型参数可以在类、结构、接口、委托（详见第四章）和方法的声明中引入。其他构造，如属性，不能*引入*类型参数，但可以*使用*类型参数。例如，属性 `Value` 使用 `T`：

```cs
public struct Nullable<T>
{
  public T Value { get; }
}
```

通用类型或方法可以具有多个参数：

```cs
class Dictionary<TKey, TValue> {...}
```

实例化：

```cs
Dictionary<int,string> myDict = new Dictionary<int,string>();
```

或者：

```cs
var myDict = new Dictionary<int,string>();
```

通用类型名称和方法名称可以重载，只要类型参数的数量不同。例如，以下三个类型名称不会冲突：

```cs
class A        {}
class A<T>     {}
class A<T1,T2> {}
```

###### 注意

按照惯例，具有*单一*类型参数的通用类型和方法通常将其参数命名为 `T`，只要参数的意图清晰即可。在使用*多个*类型参数时，每个参数都以 `T` 为前缀，但具有更具描述性的名称。

## typeof 和未绑定的通用类型

开放泛型类型在运行时不存在：它们作为编译的一部分关闭。然而，未绑定的泛型类型可以在运行时存在，纯粹作为`Type`对象。在 C# 中指定未绑定的泛型类型的唯一方法是通过`typeof`运算符：

```cs
class A<T> {}
class A<T1,T2> {}
...

Type a1 = typeof (A<>);   // *Unbound* type (notice no type arguments).
Type a2 = typeof (A<,>);  // Use commas to indicate multiple type args.
```

开放泛型类型与反射 API（第十八章）结合使用。

您还可以使用`typeof`运算符指定一个封闭类型：

```cs
Type a3 = typeof (A<int,int>);
```

或者，您可以指定一个在运行时关闭的开放类型：

```cs
class B<T> { void X() { Type t = typeof (T); } }
```

## 默认通用值

您可以使用`default`关键字获取泛型类型参数的默认值。引用类型的默认值为`null`，值类型的默认值是对值类型字段进行按位零处理的结果：

```cs
static void Zap<T> (T[] array)
{
  for (int i = 0; i < array.Length; i++)
    array[i] = default(T);
}
```

从 C# 7.1 开始，可以省略类型参数，这是编译器能够推断的情况。我们可以用以下代码替换最后一行：

```cs
    array[i] = default;
```

## 泛型约束

默认情况下，可以用任何类型替换类型参数。*约束*可以应用于类型参数，以要求更具体的类型参数。这些是可能的约束：

```cs
where *T* : *base-class*   // Base-class constraint
where *T* : *interface*    // Interface constraint
where *T* : class        // Reference-type constraint
where *T* : class?       // (See "Nullable Reference Types" in Chapter 4)
where *T* : struct       // Value-type constraint (excludes Nullable types)
where *T* : unmanaged    // Unmanaged constraint
where *T* : new()        // Parameterless constructor constraint
where *U* : *T*            // Naked type constraint
where *T* : notnull      // Non-nullable value type, or (from C# 8)
                       // a non-nullable reference type
```

在以下示例中，`GenericClass<T,U>`要求`T`派生自（或等同于）`SomeClass`并实现`Interface1`，并要求`U`提供无参数构造函数：

```cs
class     SomeClass {}
interface Interface1 {}

class GenericClass<T,U> where T : SomeClass, Interface1
                        where U : new()
{...}
```

您可以在定义类型参数的地方应用约束，无论是在方法还是类型定义中。

###### 注意

*约束*是*限制*；然而，类型参数约束的主要目的是启用其他情况下会被禁止的功能。

例如，约束`T:Foo`允许您将`T`的实例视为`Foo`，约束`T:new()`允许您构造`T`的新实例。

*基类约束*指定类型参数必须是某个类的子类（或匹配）；*接口约束*指定类型参数必须实现该接口。这些约束允许将类型参数的实例隐式转换为该类或接口。例如，假设我们想要编写一个通用的`Max`方法，它返回两个值中的最大值。我们可以利用在`System`命名空间中定义的通用接口：

```cs
public interface IComparable<T>   // Simplified version of interface
{
  int CompareTo (T other);
}
```

`CompareTo`如果`this`大于`other`，则返回一个正数。通过此接口作为约束，我们可以编写`Max`方法如下（为避免分散注意力，省略了空值检查）：

```cs
static T Max <T> (T a, T b) where T : IComparable<T>
{
  return a.CompareTo (b) > 0 ? a : b;
}
```

`Max`方法可以接受实现`IComparable<T>`的任何类型的参数（包括大多数内置类型如`int`和`string`）：

```cs
int z = Max (5, 10);               // 10
string last = Max ("ant", "zoo");  // zoo
```

###### 注意

从 C# 11 开始，接口约束还允许你在该接口上调用静态虚拟/抽象成员（参见“静态虚拟/抽象接口成员”）。例如，如果接口`IFoo`定义了一个名为`Bar`的静态抽象方法，则`T:IFoo`约束使得调用`T.Bar()`合法。我们在“静态多态性”中再次提到这个话题。

*类约束* 和 *结构约束* 指定`T`必须是引用类型或（非空）值类型。结构约束的一个很好的例子是`System.Nullable<T>`结构（我们在“可空值类型”中深入讨论这个类）：

```cs
struct Nullable<T> where T : struct {...}
```

*未管理约束*（在 C# 7.3 中引入）是结构约束的一个更强版本：`T`必须是简单值类型或递归地没有任何引用类型的结构。

*无参构造函数约束* 要求`T`具有公共的无参构造函数。如果定义了这个约束，你可以在`T`上调用`new()`：

```cs
static void Initialize<T> (T[] array) where T : new()
{
  for (int i = 0; i < array.Length; i++)
    array[i] = new T();
}
```

*裸类型约束* 要求一个类型参数从另一个类型参数派生（或匹配）。在这个例子中，方法`FilteredStack`返回另一个`Stack`，其中只包含类型参数`U`是类型参数`T`的子集的元素：

```cs
class Stack<T>
{
  Stack<U> FilteredStack<U>() where U : T {...}
}
```

## 泛型类型的子类化

泛型类可以像非泛型类一样被子类化。子类可以保留基类的类型参数不变，如下面的例子：

```cs
class Stack<T>                   {...}
class SpecialStack<T> : Stack<T> {...}
```

或者，子类可以用具体类型关闭泛型类型参数：

```cs
class IntStack : Stack<int>  {...}
```

子类型也可以引入新鲜的类型参数：

```cs
class List<T>                     {...}
class KeyedList<T,TKey> : List<T> {...}
```

###### 注意

从技术上讲，子类型上的所有类型参数都是新鲜的：你可以说子类型关闭然后重新打开基类型参数。这意味着子类可以为重新打开的类型参数提供新的（可能更有意义的）名称：

```cs
class List<T> {...}
class KeyedList<TElement,TKey> : List<TElement> {...}
```

## 自引用泛型声明

一个类型可以在关闭类型参数时将*自己*命名为具体类型：

```cs
public interface IEquatable<T> { bool Equals (T obj); }

public class Balloon : IEquatable<Balloon>
{
  public string Color { get; set; }
  public int CC { get; set; }

  public bool Equals (Balloon b)
  {
    if (b == null) return false;
    return b.Color == Color && b.CC == CC;
  }
}
```

以下也是合法的：

```cs
class Foo<T> where T : IComparable<T> { ... }
class Bar<T> where T : Bar<T> { ... }
```

## 静态数据

静态数据对于每个关闭类型是唯一的：

```cs
Console.WriteLine (++Bob<int>.Count);     // 1
Console.WriteLine (++Bob<int>.Count);     // 2
Console.WriteLine (++Bob<string>.Count);  // 1
Console.WriteLine (++Bob<object>.Count);  // 1

class Bob<T> { public static int Count; }
```

## 类型参数和转换

C#的转型运算符可以执行多种类型的转换，包括以下内容：

+   数字转换

+   引用转换

+   装箱/拆箱转换

+   自定义转换（通过运算符重载；参见第四章）

决定将发生哪种类型的转换是在*编译时*进行的，根据操作数的已知类型。这在泛型类型参数方面创建了一个有趣的场景，因为编译时未知精确的操作数类型。如果这导致歧义，编译器会生成错误。

最常见的情况是当你想要执行引用转换时：

```cs
StringBuilder Foo<T> (T arg)
{
  if (arg is StringBuilder)
    return (StringBuilder) arg;   // Will not compile
  ...
}
```

在不知道 `T` 的实际类型的情况下，编译器担心您可能打算进行一个 *自定义转换*。最简单的解决方案是改用 `as` 运算符，因为它是明确的，因为它不能执行自定义转换：

```cs
StringBuilder Foo<T> (T arg)
{
  StringBuilder sb = arg as StringBuilder;
  if (sb != null) return sb;
  ...
}
```

更一般的解决方案是首先转换为 `object`。这是因为到/从 `object` 的转换被假定不是自定义转换，而是引用或装箱/拆箱转换。在这种情况下，`StringBuilder` 是一个引用类型，所以必须是一个引用转换：

```cs
  return (StringBuilder) (object) arg;
```

拆箱转换也可能引入歧义。以下内容可能是一个拆箱、数值或自定义转换：

```cs
int Foo<T> (T x) => (int) x;     // Compile-time error
```

解决方案再次是先转换为 `object`，然后再转换为 `int`（这在这种情况下明确地表示一个拆箱转换）：

```cs
int Foo<T> (T x) => (int) (object) x;
```

## 协变

假设 `A` 可转换为 `B`，则如果 `X<A>` 可转换为 `X<B>`，则 `X` 具有协变类型参数。

###### 注意

在 C# 的协变（和逆变）概念中，“可转换”意味着可以通过 *隐式引用转换* 进行转换，例如 `A` *子类化* `B`，或者 `A` *实现* `B`。数值转换、装箱转换和自定义转换不包括在内。

例如，类型 `IFoo<T>` 如果以下内容合法，则具有协变的 `T`：

```cs
IFoo<string> s = ...;
IFoo<object> b = s;
```

接口允许协变类型参数（委托也允许；参见 第四章），但类不允许。数组也允许协变（如果 `A` 对 `B` 有隐式引用转换，则 `A[]` 可以转换为 `B[]`），并且在此处进行了比较讨论。

###### 注意

协变和逆变（或简称“变异”）是高级概念。引入和增强 C# 中协变和逆变的动机是为了使泛型接口和泛型类型（特别是在 .NET 中定义的那些，比如 `IEnumerable<T>`）能更符合您的预期工作。您可以在不理解协变和逆变的详细信息的情况下从中受益。

### 协变不是自动的

为了确保静态类型安全，类型参数不会自动变异。考虑以下情况：

```cs
class Animal {}
class Bear : Animal {}
class Camel : Animal {}

public class Stack<T>   // A simple Stack implementation
{
  int position;
  T[] data = new T[100];
  public void Push (T obj)  => data[position++] = obj;
  public T Pop()            => data[--position]; 
}
```

以下内容无法编译通过：

```cs
Stack<Bear> bears = new Stack<Bear>();
Stack<Animal> animals = bears;            // Compile-time error
```

该限制阻止了以下代码可能导致运行时失败的可能性：

```cs
animals.Push (new Camel());      // Trying to add Camel to bears
```

然而，缺乏协变可能会阻碍可重用性。例如，假设我们想编写一个方法来 `Wash` 一堆动物：

```cs
public class ZooCleaner
{
  public static void Wash (Stack<Animal> animals) {...}
}
```

使用一堆熊调用 `Wash` 将会生成一个编译时错误。一种解决方法是重新定义带有约束条件的 `Wash` 方法：

```cs
class ZooCleaner
{
  public static void Wash<T> (Stack<T> animals) where T : Animal { ... }
}
```

现在我们可以按以下方式调用 `Wash`：

```cs
Stack<Bear> bears = new Stack<Bear>();
ZooCleaner.Wash (bears);
```

另一种解决方案是让 `Stack<T>` 实现一个具有协变类型参数的接口，您很快将会看到。

### 数组

基于历史原因，数组类型支持协变。这意味着如果 `B` 是 `A` 的子类（且两者都是引用类型），则 `B[]` 可以转换为 `A[]`：

```cs
Bear[] bears = new Bear[3];
Animal[] animals = bears;     // OK
```

这种可重用性的缺点是元素赋值可能在运行时失败：

```cs
animals[0] = new Camel();     // Runtime error
```

### 声明一个协变类型参数

接口和委托上的类型参数可以通过标记它们为`out`来声明为协变。该修饰符确保与数组不同，协变类型参数是完全类型安全的。

我们可以通过我们的`Stack<T>`类来说明这一点，通过实现以下接口：

```cs
public interface IPoppable<out T> { T Pop(); }
```

对于`T`上的`out`修饰符表明，`T`仅在*输出*位置（例如，方法的返回类型）中使用。`out`修饰符标志着类型参数为*协变*，并允许我们这样做：

```cs
var bears = new Stack<Bear>();
bears.Push (new Bear());
// Bears implements IPoppable<Bear>. We can convert to IPoppable<Animal>:
IPoppable<Animal> animals = bears;   // Legal
Animal a = animals.Pop();
```

编译器允许从`bears`到`animals`的转换，因为类型参数是协变的。这是类型安全的，因为编译器试图避免的情况——将`Camel`推入堆栈——不可能发生，因为没有方法将`Camel`*输入*到仅能在*输出*位置中出现`T`的接口中。

###### 注意

接口中的协变（和逆变）通常是*消费*的概念：相比之下，您很少需要*编写*变体接口。

###### 警告

有趣的是，由于 CLR 的限制，标记为`out`的方法参数不符合协变条件。

我们可以利用协变转型来解决前面描述的可重用性问题：

```cs
public class ZooCleaner
{
  public static void Wash (IPoppable<Animal> animals) { ... }
}
```

###### 注意

在第七章中描述的`IEnumerator<T>`和`IEnumerable<T>`接口具有协变的`T`。这允许您将`IEnumerable<string>`转换为`IEnumerable<object>`，例如。

如果您在*输入*位置使用协变类型参数（例如，方法的参数或可写属性），编译器将生成错误。

###### 注意

协变（和逆变）仅适用于*引用转换*的元素——而非*装箱转换*（这既适用于类型参数的方差，也适用于数组的方差）。因此，如果您编写了一个接受类型为`IPoppable<object>`的参数的方法，则可以使用`IPoppable<string>`但不能使用`IPoppable<int>`来调用它。

## 逆变

我们之前看到，假设`A`允许隐式引用转换到`B`，如果类型`X<A>`允许将引用转换到`X<B>`，则类型`X`具有协变类型参数。*逆变*是指可以反向转换——从`X<B>`到`X<A>`。如果类型参数仅在*输入*位置中出现并带有`in`修饰符，则支持此操作。扩展我们之前的示例，假设`Stack<T>`类实现了以下接口：

```cs
public interface IPushable<in T> { void Push (T obj); }
```

现在我们可以合法地执行此操作：

```cs
IPushable<Animal> animals = new Stack<Animal>();
IPushable<Bear> bears = animals;    // Legal
bears.Push (new Bear());
```

`IPushable`中没有任何成员*输出*`T`，因此我们无法通过该接口将`animals`转换为`bears`（例如，无法通过该接口进行`Pop`操作）。

###### 注意

我们的`Stack<T>`类可以同时实现`IPushable<T>`和`IPoppable<T>`—尽管`T`在这两个接口中具有相反的变异注释！这是因为你必须通过接口来实现变异，而不是类；因此，在执行变体转换之前，你必须承诺使用`IPoppable`或`IPushable`的视角。然后，这个视角限制了你只能在适当的变异规则下执行合法的操作。

这也说明了为什么*类*不允许变异类型参数：具体的实现通常要求数据在两个方向上流动。

再举一个例子，考虑在`System`命名空间中定义的以下接口：

```cs
public interface IComparer<in T>
{
  // Returns a value indicating the relative ordering of a and b
  int Compare (T a, T b);
}
```

因为接口具有逆变`T`，我们可以使用`IComparer<**object**>`来比较两个*字符串*：

```cs
var objectComparer = Comparer<object>.Default;
// objectComparer implements IComparer<object>
IComparer<string> stringComparer = objectComparer;
int result = stringComparer.Compare ("Brett", "Jemaine");
```

与协变相反，如果您尝试在输出位置（例如作为返回值或可读属性中）使用逆变类型参数，编译器将报告错误。

## C#泛型与 C++模板对比

C#泛型与 C++模板在应用上相似，但它们的工作方式大不相同。在两种情况下，都需要在生产者和消费者之间进行综合，其中生产者的占位类型由消费者填充。然而，对于 C#泛型，生产者类型（例如`List<T>`这样的开放类型）可以编译为库（例如*mscorlib.dll*）。这是因为生产者和消费者之间在运行时实际上并不进行生成闭合类型的综合。而对于 C++模板，这种综合是在编译时执行的。这意味着在 C++中，你不会将模板库部署为*.dll*文件——它们仅存在于源代码中。这也使得动态检查甚至创建参数化类型变得困难。

要深入探讨这种情况背后的原因，再次考虑 C#中的`Max`方法：

```cs
static T Max <T> (T a, T b) where T : IComparable<T>
  => a.CompareTo (b) > 0 ? a : b;
```

为什么我们不能这样实现呢？

```cs
static T Max <T> (T a, T b)
  => (a > b ? a : b);             // Compile error
```

原因是`Max`需要编译一次，并且适用于`T`的所有可能值。编译无法成功，因为对于所有`T`的值来说，`>`都没有单一的含义——事实上，并不是每个`T`都有`>`运算符。相比之下，以下代码展示了用 C++模板编写的相同`Max`方法。这段代码将为每个`T`的值单独编译，承担特定`T`所具有的`>`运算符的任何语义，并在特定`T`不支持`>`运算符时无法编译通过：

```cs
template <class T> T Max (T a, T b)
{
  return a > b ? a : b;
}
```

¹ 参考类型也可以是`System.ValueType`或`System.Enum`（第六章）。
