# B

F# 概述

本附录探讨了 F# 的基本语法，F# 是一种已确立的通用目的函数式第一语言，同时支持面向对象编程（OOP）。实际上，F# 融入了 .NET 公共语言基础设施（CLI）对象模型，这允许声明接口、类和抽象类。此外，F# 是一种静态和强类型语言，这意味着编译器可以在编译时检测变量和函数的数据类型。F# 的语法与 C 风格语言（如 C#）不同，因为代码块不是用花括号来界定的。此外，空格而不是逗号和缩进对于分隔参数和界定函数体的作用域非常重要。另外，F# 是一种跨平台编程语言，可以在 .NET 生态系统内外运行。

## let 绑定

在 F# 中，`let` 是最重要的关键字之一，它将一个标识符绑定到一个值，这意味着给值命名（或，将值绑定到名称）。它定义为 `let <identifier> = <value>。`

`let` 绑定默认是不可变的。以下是一些代码示例：

```
let myInt = 42
let myFloat = 3.14
let myString = "hello functional programming" 
let myFunction = fun number -> number * number 
```

如您从最后一行所见，您可以通过将标识符 `myFunction` 绑定到 lambda 表达式 `fun number -> number * number` 来命名一个函数。

`fun` 关键字用于在语法中定义 lambda 表达式（匿名函数），形式为 `fun args -> body`。有趣的是，你不需要在代码中定义类型，因为由于它强大的内置类型推断系统，F# 编译器可以原生地理解它们。例如，在上面的代码中，编译器推断出 `myFunction` 函数的参数是一个数字，因为乘法 (`*`) 操作符的存在。

## 理解 F# 中的函数签名

在 F# 中，与大多数函数式语言一样，函数签名使用从左到右读取的箭头符号定义。函数是总是有输出的表达式，所以最右边的箭头总是指向返回类型。例如，当你看到 `typeA -> typeB` 时，你可以将其解释为一个接受 `typeA` 类型的输入值并产生 `typeB` 类型的值的函数。相同的原理也适用于接受两个以上参数的函数。当函数的签名是 `typeA -> typeB -> typeC` 时，你从左到右读取箭头，这创建了两个函数。第一个函数是 `typeA -> (typeB -> typeC)`，它接受 `typeA` 类型的输入并产生 `typeB -> typeC` 类型的函数。

这是 `add` 函数的签名：

```
val add : x:int -> y:int -> int 
```

这接受一个参数 `x:int` 并返回一个函数，该函数接受 `y:int` 作为输入并返回一个 `int` 类型的结果。箭头符号与柯里化和匿名函数内在相关。

## 创建可变类型：mutable 和 ref

FP（函数式编程）中的一个主要概念是不可变性。F#是一种以函数为主的编程语言；但显式使用`immutable`关键字可以让您创建类似变量的可变类型，如下例所示：

```
let mutable myNumber = 42 
```

现在可以使用 goes-to (`<-``)`运算符来更改`myNumber`的值：

```
myNumber  <-  51 
```

定义可变类型的另一种选择是使用定义存储位置的*引用单元格*，该存储位置允许您使用引用语义创建可变值。`ref`运算符声明一个新的引用单元格，它封装了一个值，然后可以使用`:=`运算符更改该值，并使用`!`（感叹号）运算符访问该值：

```
let myRefVar = ref 42
myRefVar := 53
printfn "%d" !myRefVar 
```

第一行声明了具有值 42 的引用单元格`myRefVar`，第二行将其值更改为 53。在代码的最后一行，访问并打印了基础值。

可变变量和引用单元格可以在几乎相同的情况下使用；但首选可变类型，除非编译器不允许，并且可以使用引用单元格代替。例如，在生成需要可变状态的闭包的表达式中，编译器将报告不能使用可变变量。在这种情况下，引用单元格解决了这个问题。

## 函数作为一阶类型

在 F#中，函数是一阶数据类型；可以使用`let`关键字声明，并且可以像任何其他变量一样使用：

```
let square x = x * x 
let plusOne x = x + 1
let isEven x = x % 2 = 0 
```

函数总是返回一个值，即使没有显式的`return`关键字。函数中执行的最后一个语句的值是返回值。

## 组合：管道和组合运算符

管道（`|>`）和组合（`>>`）运算符用于链接函数和参数，以提高代码可读性。这些运算符以灵活的方式让您建立函数的*管道*。这些运算符的定义很简单：

```
let inline **(|>)** x f = f x
let inline **(>>)** f g x = g(f x) 
```

以下示例展示了如何利用这些运算符构建功能管道：

```
let squarePlusOne x =  x **|>** square **|>** plusOne
let plusOneIsEven = plusOne **>>** isEven 
```

在代码的最后一行，组合（`>>`）运算符让您无需显式定义输入参数。F#编译器理解函数`plusOneIsEven`期望一个整数作为输入。不需要参数定义的函数称为*无参数函数*。

管道（`|>`）和组合（`>>`）运算符之间的主要区别在于它们的签名和使用。管道运算符接受函数和参数，而组合将函数组合在一起。

## 委托

在.NET 中，*委托*是一个指向函数的指针；它是一个包含对具有相同公共签名的函数引用的变量。在 F#中，使用函数值代替委托；但 F#提供了与.NET API 交互的委托支持。这是 F#中定义委托的语法：

```
type delegate-typename = delegate of typeA -> typeB 
```

以下代码展示了创建具有表示加法操作的签名的委托的语法：

```
type MyDelegate = delegate of (int * int) -> int
let add (a, b) = a + b
let addDelegate = MyDelegate(add)
let result = addDelegate.Invoke(33, 9) 
```

在示例中，F# 函数 `add` 直接作为参数传递给委托构造函数 `MyDelegate`。委托可以附加到 F# 函数值、静态方法或实例方法。委托类型 `addDelegate` 上的 `Invoke` 方法调用底层函数 `add`。

## 注释

F# 中使用了三种注释类型：块注释位于 `(*` 和 `*)` 符号之间，行注释以 `//` 符号开始，并持续到行尾，XML 文档注释跟在 `///` 符号之后，允许您使用 XML 标签根据编译器生成的文件生成代码文档。以下是这些注释的示例：

```
(* This is block comment *)
// Single line comments use a double forward slash
/// This comment can be used to generate documentation. 
```

## 打开语句

您可以使用 `open` 关键字打开一个命名空间或模块，类似于 C# 中的语句。此代码打开 `System` 命名空间：`open` `System`。

## 基本数据类型

表 B.1 展示了 F# 的 *原始* 类型列表。

表 B.1 基本数据类型

| **F# 类型** | **.NET 类型** | **字节大小** | **范围** | **示例** |
| --- | --- | --- | --- | --- |
| `sbyte` | `System.SByte` | 1 | -128 to 127 | 42y |
| `byte` | `System.Byte` | 1 | 0 to 255 | 42uy |
| `int16` | `System.Int16` | 2 | -32,768 to 32,767 | 42s |
| `uint16` | `System.UInt16` | 2 | 0 to 65,535 | 42us |
| `int / int32` | `System.Int32` | 4 | -2,147,483,648 to 2,147,483,647 | 42 |
| `uint32` | `System.UInt32` | 4 | 0 to 4,294,967,295 | 42u |
| `int64` | `System.Int64` | 8 | -9,223,372,036,854,775,808 to 9,223,372,036,854,775,807 | 42L |
| `uint64` | `System.UInt64` | 8 | 0 to 18,446,744,073,709,551,615 | 42UL |
| `float32` | `System.Single` | 4 | ±1.5e-45 to ±3.4e38 | 42.0F |
| `float` | `System.Double` | 8 | ±5.0e-324 to ±1.7e308 | 42.0 |
| `decimal` | `System.Decimal` | 16 | ±1.0e-28 to ±7.9e28 | 42.0M |
| `char` | `System.Char` | 2 | U+0000 to U+ffff | 'x' |
| `string` | `System.String` | 20 + (2 * size of string) | 0 to about 2 billion characters | "Hello World" |
| `bool` | `System.Boolean` | 1 | 只有两个可能的值：true 或 false | true |

## 特殊字符串定义

在 F# 中，字符串类型是 `System.String` 类型的别名；但除了传统的 .NET 语义外，您还可以使用特殊的三个引号的方式来声明字符串。这种特殊的字符串定义允许您声明一个字符串，而无需像文本字面量那样转义特殊字符。以下示例以标准方式和 F# 三个引号方式定义了相同的字符串，以转义特殊字符：

```
let verbatimHtml = @"<input type="submit" value="Submit">"
let tripleHTML = """<input type="submit" value="Submit">""" 
```

## 元组

*元组* 是一组无名称且有序的值，可以是不同类型。元组对于创建临时数据结构很有用，并且是函数返回多个值的便捷方式。元组定义为值的逗号分隔集合。以下是构造元组的方法：

```
let tuple = (1, "Hello")
let tripleTuple = ("one", "two", "three") 
```

元组也可以被解构。在这里，元组值 `1` 和 `"Hello"` 分别绑定到标识符 `a` 和 `b`，函数 `swap` 交换给定元组 `(a, b)` 中两个值的顺序：

```
let (a, b) = tuple
let swap (a, b) = (b, a) 
```

元组通常是对象，但也可以定义为值类型`结构体`，如下所示：

```
let tupleStruct = struct (1, "Hello") 
```

注意，F#类型推断可以自动将函数泛化为一个泛型类型，这意味着元组可以与任何类型一起工作。可以使用`fst`和`snd`函数访问和获取元组的第一个和第二个元素：

```
let one = fst tuple
let hello = snd tuple 
```

## 记录类型

*记录类型*类似于元组，除了字段有名称，并定义为分号分隔的列表。虽然元组提供了一种在单个容器中存储可能异构数据的方法，但当存在多个元素时，解释元素的目的可能会变得困难。在这种情况下，记录类型通过使用名称标记其定义来帮助解释数据的目的。记录类型通过使用`type`关键字显式定义，并编译为不可变、公共和密封的 .NET 类。此外，编译器自动生成结构相等性和比较功能，并提供一个默认构造函数，该构造函数填充记录中包含的所有字段。

此示例展示了如何定义和实例化一个新的记录类型：

```
type Person = { FirstName : string; LastName : string; Age : int }
let fred = { FirstName = "Fred"; LastName = "Flintstone"; Age = 42 } 
```

记录可以通过属性和方法进行扩展：

```
type Person with
    member this.FullName = sprintf "%s %s" this.FirstName this.LastName 
```

记录是不可变类型，这意味着记录的实例不能被修改。但你可以通过使用`with`克隆语义方便地克隆记录：

```
let olderFred = { fred with Age = fred.Age + 1 } 
```

记录类型也可以使用`[<Struct>]`属性表示。这在性能至关重要的场景中很有帮助，并覆盖了引用类型的灵活性：

```
[<Struct>]
type Person = { FirstName : string; LastName : string; Age : int } 
```

## 区分联合

*区分联合*（DUs）是一种类型，它表示一组值，这些值可以是几个定义良好的情况之一，每个情况可能具有不同的值和类型。在面向对象范式中，DUs 可以被视为从同一基类继承的一组类。通常，DUs 是构建复杂数据结构、建模领域和表示递归结构（如`Tree`数据类型）的工具。

以下代码显示了扑克牌的花色和点数：

```
type Suit = Hearts | Clubs | Diamonds | Spades

type Rank = 
        | Value of int
        | Ace
        | King
        | Queen
        | Jack
       static member GetAllRanks() = 
            [ yield Ace
              for i in 2 .. 10 do yield Value i
              yield Jack
              yield Queen
              yield King ] 
```

如您所见，DUs 可以通过属性和方法进行扩展。表示牌组中所有牌的列表可以按以下方式计算：

```
 let fullDeck = 
        [ for suit in [ Hearts; Diamonds; Clubs; Spades] do
              for rank in Rank.GetAllRanks() do 
                  yield { Suit=suit; Rank=rank } ] 
```

此外，DUs 也可以通过具有`[<Struct>]`属性的`结构体`来表示。

## 模式匹配

*模式* *匹配* 是一种语言结构，它使编译器能够解释数据类型的定义并对其应用一系列条件。通过这种方式，编译器强制你编写模式匹配结构，通过覆盖所有可能的案例来匹配给定的值。这被称为 *穷举* 模式匹配。模式匹配结构用于控制流。它们在概念上类似于一系列 `if/then` 或 `case/switch` 语句，但功能更强大。它们允许你在每次匹配期间将数据结构分解为其底层组件，然后对这些值执行某些计算。在所有编程语言中，*控制流* 指的是代码中做出的决策，这些决策影响应用程序中语句执行的顺序。

通常，最常见的模式涉及代数数据类型，例如区分联合、记录类型和集合。以下代码示例展示了 Fizz-Buzz ([`en.wikipedia.org/wiki/Fizz_buzz`](https://en.wikipedia.org/wiki/Fizz_buzz)) 游戏的两个实现。第一个模式匹配结构包含一系列条件来测试函数 `divisibleBy` 的评估。如果任一条件为真或假，第二个实现使用 `when` 子句，称为 *guard*，来指定和整合必须成功的附加测试以匹配模式：

```
let fizzBuzz n = 
    let divisibleBy m = n % m = 0
    match divisibleBy 3,divisibleBy 5 with
    | true, false -> "Fizz"
    | false, true -> "Buzz"
    | true, true -> "FizzBuzz"
    | false, false -> sprintf "%d" n

let fizzBuzz n =
    match n with
    | _ when (n % 15) = 0 -> "FizzBuzz"
    | _ when (n % 3) = 0 -> "Fizz"
    | _ when (n % 5) = 0 -> "Buzz"
    | _ -> sprintf "%d" n

 [1..20] |> List.iter fizzBuzz 
```

当模式匹配结构被评估时，表达式被传递到 `match <expression>`，然后与每个模式进行测试，直到找到第一个匹配。然后评估相应的主体。下划线字符（_）被称为 *wildcard*，这是始终有正匹配的一种方式。通常，这个模式被用作通用捕获的最终子句，以应用于常见行为。

## 活跃模式

*活跃* *模式* 是扩展模式匹配功能的结构，允许对给定的数据结构进行分区和分解，从而确保通过使代码更易于阅读并使分解的结果可用于进一步的模式匹配来转换和提取底层值。

此外，活跃模式允许你将任意值包裹在 DU 数据结构中，以便于模式匹配。可以使用活跃模式包裹对象，这样你就可以像使用任何其他联合类型一样轻松地在模式匹配中使用这些对象。

有时活跃模式不会生成值；在这种情况下，它们被称为 *部分活跃模式*，并导致一个选项类型。要定义部分活跃模式，你需要在用括号和管道字符组合创建的香蕉剪辑 `(| |)` 内部的模式列表末尾使用下划线通配符字符（_）。以下是一个典型的部分活跃模式的外观：

```
let (|DivisibleBy|_|) divideBy n = 
    if n % divideBy = 0 then Some DivisibleBy else None 
```

在此部分主动模式中，如果值 `n` 能被 `divideBy` 的值整除，则返回类型为 `Some()`，这表示主动模式成功。否则，`None` 返回类型表示模式失败并移动到下一个匹配表达式。部分主动模式用于分区和匹配输入空间的一部分。以下代码演示了如何对部分主动模式进行模式匹配：

```
let fizzBuzz n = 
    match n with 
    | DivisibleBy 3 & DivisibleBy 5 -> "FizzBuzz" 
    | DivisibleBy 3 -> "Fizz" 
    | DivisibleBy 5 -> "Buzz" 
    | _ -> sprintf "%d" n

[1..20] |> List.iter fizzBuzz 
```

此函数使用部分主动模式 `(|可被|_|)` 来测试输入值 `n`。如果它能被 3 和 5 整除，则第一个情况成功。如果它只能被 3 整除，则第二个情况成功，依此类推。请注意，`&` 操作符允许你在同一个参数上运行多个模式。

另一种主动模式是 *参数化主动模式*，它与部分主动模式类似，但接受一个或多个额外的参数作为输入。

更有趣的是 *多情况主动模式*，它将整个输入空间划分为不同形状为 DU 的数据结构。以下是一个使用多情况主动模式的 `FizzBuzz` 示例：

```
let (|Fizz|Buzz|FizzBuzz|Val|) n = 
    match n % 3, n % 5 with
    | 0, 0 -> FizzBuzz
    | 0, _ -> Fizz
    | _, 0 -> Buzz
    | _ -> Val n 
```

由于主动模式可以将数据从一种类型转换为另一种类型，因此它们非常适合数据转换和验证。主动模式有四种相关的类型：单案例、部分案例、多案例和部分参数化。有关主动模式的更多详细信息，请参阅 MSDN 文档 ([`mng.bz/Itmw`](http://mng.bz/Itmw)) 和 Isaac Abraham 的 *Get Programming with F#* (Manning, 2018)。

## 集合

F# 支持标准 .NET 集合，如数组序列 (`IEnumerable`)。此外，它还提供了一套不可变的函数式集合：列表、集合和映射。

## 数组

数组是零基、可变集合，具有固定大小的相同类型元素。由于它们被编译为连续的内存块，因此它们支持快速、随机的元素访问。以下是创建、过滤和投影数组的不同方法：

```
let emptyArray= Array.empty
let emptyArray = [| |]  
let arrayOfFiveElements = [| 1; 2; 3; 4; 5 |]
let arrayFromTwoToTen= [| 2..10 |]
let appendTwoArrays = emptyArray |> Array.append arrayFromTwoToTen
let evenNumbers = arrayFromTwoToTen |> Array.filter(fun n -> n % 2 = 0)
let squareNumbers = evenNumbers |> Array.map(fun n -> n * n) 
```

可以通过使用点操作符 (`.`) 和方括号 `[ ]` 来访问和更新数组的元素：

```
let arr = Array.init 10 (fun i -> i * i)
arr.[1] <- 42
arr.[7] <- 91 
```

数组也可以使用 `Array` 模块中的函数以各种其他语法创建：

```
let arrOfBytes = Array.create 42 0uy
let arrOfSquare = Array.init 42 (fun i -> i * i)
let arrOfIntegers = Array.zeroCreate<int> 42 
```

## 序列（seq）

*序列* 是同一类型元素的一系列。与 `List` 类型不同，序列是延迟评估的，这意味着元素可以在需要时计算（仅当需要时）。在不需要所有元素的情况下，这比列表提供了更好的性能。以下是创建、过滤和投影序列的另一种方法：

```
let emptySeq = Seq.empty
let seqFromTwoToFive = seq { yield 2; yield 3; yield 4; yield 5 }
let seqOfFiveElements = seq { 1 .. 5 }
let concatenateTwoSeqs = emptySeq |> Seq.append seqOfFiveElements
let oddNumbers = seqFromTwoToFive |> Seq.filter(fun n -> n % 2 <> 0)
let doubleNumbers = oddNumbers |> Seq.map(fun n -> n + n) 
```

序列可以使用 `yield` 关键字来延迟返回成为序列一部分的值。

## 列表

在 F#中，`List`集合是一个不可变的、元素类型相同的单链表。一般来说，列表是枚举的好选择，但在性能关键时，不建议用于随机访问和连接。列表使用`[ ... ]`语法定义。以下是一些创建、过滤和映射列表的示例：

```
let emptyList = List.empty
let emptyList = [ ]  
let listOfFiveElements = [ 1; 2; 3; 4; 5 ]
let listFromTwoToTen = [ 2..10 ]
let appendOneToEmptyList = 1::emptyList
let concatenateTwoLists = listOfFiveElements @ listFromTwoToTen
let evenNumbers = listOfFiveElements |> List.filter(fun n -> n % 2 = 0)
let squareNumbers = evenNumbers |> List.map(fun n -> n * n) 
```

列表使用括号（`[ ]`）和分号（`;`）分隔符来向列表中添加多个项目，使用符号`::`来添加一个项目，并使用 at 符号运算符（`@`）来连接两个给定的列表。

## 集合

一个*集合*是基于二叉树的集合，其中元素类型相同。使用集合时，不保留插入顺序，也不允许重复。集合是不可变的，并且更新其元素的操作都会创建一个新的集合。以下是一些创建集合的不同方法：

```
let emptySet = Set.empty<int>
let setWithOneItem = emptySet.Add 8
let setFromList = [ 1..10 ] |> Set.ofList 
```

## 映射

一个*映射*是一个不可变的、具有相同类型的元素集合的键值对。这个集合将值与键关联起来，并且它的行为类似于不允许重复或不尊重插入顺序的`Set`类型。以下示例展示了如何以不同的方式实例化映射：

```
let emptyMap = Map.empty<int, string>
let mapWithOneItem = emptyMap.Add(42, "the answer to the meaning of life")
let mapFromList = [ (1, "Hello"), (2, "World") ] |> Map.ofSeq 
```

## 循环

F#支持循环结构来遍历如列表、数组、序列、映射等可枚举集合。`while...do`表达式在指定的条件为真时执行迭代：

```
let mutable a = 10
while (a < 20) do
   printfn "value of a: %d" a
   a <- a + 1 
```

`for...to`表达式在循环中遍历一个循环变量的值集合：

```
 for i = 1 to 10 do
    printf "%d " i 
```

`for...in`表达式在循环中遍历值集合中的每个元素：

```
for i in [1..10] do
   printfn "%d" i 
```

## 类和继承

如前所述，F#支持像其他.NET 编程语言一样的 OOP 结构。实际上，可以定义类对象来模拟现实世界的领域。在 F#中用于声明类的`type`关键字可以公开属性、方法和字段。以下代码展示了从`Person`类继承的子类`Student`的定义：

```
type Person(firstName, lastName, age) =
    member this.FirstName = firstName
    member this.LastName = lastName
    member this.Age = age

    member this.UpdateAge(n:int) =
        Person(firstName, lastName, age + n)  

    override this.ToString() = 
        sprintf "%s %s" firstName lastName

type Student(firstName, lastName, age, grade) =
    inherit Person(firstName, lastName, age)

    member this.Grade = grade 
```

属性`FirstName`、`LastName`和`Age`作为字段公开；方法`UpdateAge`返回一个新的`Person`对象，其`Age`已修改。可以使用`override`关键字更改从基类继承的方法的默认行为。在示例中，`ToString`基方法被重写以返回全名。

对象`Student`是通过`inherit`关键字定义的子类，它从基类`Person`继承其成员，并添加了自己的成员`Grade`。

## 抽象类和继承

一个*抽象类*是一个提供定义类的模板的对象。通常它暴露一个或多个不完整的方法或属性实现，并要求你创建子类来填充这些实现。但是可以定义默认行为，这可以被覆盖。在下面的例子中，抽象类`Shape`定义了`Rectangle`和`Circle`类：

```
[<AbstractClass>]
type Shape(weight :float, height :float) =
    member this.Weight = weight
    member this.Height = height

 abstract member Area : unit -> float
    default this.Area() = weight * height

type Rectangle(weight :float, height :float) =
    inherit Shape(weight, height)

type Circle(radius :float) =
    inherit Shape(radius, radius)
    override this.Area() = radius * radius * Math.PI 
```

`AbstractClass`属性通知编译器该类有抽象成员。`Rectangle`类使用`Area`方法的默认实现，而`Circle`类则使用自定义行为覆盖它。

## **接口**

*接口*代表了一个定义类实现细节的合约。但在接口声明中，成员没有被实现。接口提供了一种抽象的方式来引用它公开的公共成员和函数。在 F#中，要定义一个接口，使用`abstract`关键字声明成员，后跟它们的类型签名：

```
type IPerson =
   abstract FirstName : string
   abstract LastName : string
   abstract FullName : unit -> string 
```

类实现的接口方法是通过接口定义而不是通过类的实例来访问的。因此，要调用接口方法，需要对类应用一个`:>`（向上转换）操作符：

```
type Person(firstName : string, lastName : string) =
    interface IPerson with
        member this.FirstName = firstName
        member this.LastName = lastName
        member this.FullName() = sprintf "%s %s" firstName lastName

let fred = Person("Fred", "Flintstone")

(fred :> IPerson).FullName() 
```

## **对象表达式**

接口代表了可以在程序的其他部分之间共享的有用代码实现。但可能需要繁琐的工作来定义通过创建新类实现的特定接口。一种解决方案是使用*对象表达式*，它允许您通过使用匿名类即时实现接口。以下是一个创建新对象以实现`IDisposable`接口并将颜色应用到控制台然后恢复原始状态的示例：

```
let print color =
    let current = Console.ForegroundColor
    Console.ForegroundColor <- color
 **{**   **new** IDisposable **with**
            member x.Dispose() =                
                Console.ForegroundColor <- current
   ** }**

using(print ConsoleColor.Red) (fun _ -> printf "Hello in red!!")
using(print ConsoleColor.Blue) (fun _ -> printf "Hello in blue!!") 
```

## **类型转换**

将原始值转换为对象类型的过程称为*装箱*，它通过`box`函数应用。这个函数将任何类型向上转换为.NET 的`System.Object`类型，在 F#中用名称 obj 缩写。

*向上转换*函数应用于类和接口层次结构，它从类向上到继承的类。语法是`expr :> type`。转换的成功在编译时进行检查。

*向下转换*函数用于应用一个“向下”类或接口层次结构的转换：例如，从一个接口到一个实现类。语法是`expr :?> type`，其中操作符内的问号表示该操作可能因`InvalidCastException`而失败。在应用向下转换之前，安全地比较和测试类型。这可以通过类型测试操作符`:?`来实现，它在 C#中相当于`is`操作符。*匹配表达式*如果值匹配给定的类型则返回 true；否则返回 false：

```
let testPersonType (o:obj) =
    match o with
    | o :? IPerson -> printfn "this object is an IPerson"
    | _ -> printfn "this is not an IPerson" 
```

## **度量单位**

*单位* *度量*（UoM）是 F#类型系统的独特特性，它提供了定义上下文和为静态类型单位元数据注释的能力，并将其应用于数值字面量。这是一种方便的方式来操作表示特定度量单位（如米、秒、磅等）的数字。F#类型系统首先检查 UoM 是否正确使用，从而消除运行时错误。例如，如果 F#编译器期望一个`float<mil>`，而使用了`float<m/sec>`，则会抛出错误。此外，可以将特定函数与定义的 UoM 关联起来，该函数在单位上执行工作而不是在数值字面量上。在此，代码展示了如何定义米（m）和秒（sec）UoM，然后执行一个计算速度的操作：

```
[<Measure>]
type m 

[<Measure>]
type sec 

let distance = 25.0<m>
let time = 10.0<sec>
let speed = distance / time 
```

## 事件模块 API 参考

*事件模块*提供了管理事件流的函数。表 B.1 列出了来自在线 MSDN 文档的 API 参考([`mng.bz/a0hG`](http://mng.bz/a0hG))。

表 B.2 API 参考

| **函数** | **描述** |
| --- | --- |

| add :

```
`('T -> unit) -> Event<'Del,'T> -> unit` 
```

| 每次事件触发时运行函数。 |
| --- |

| choose :

```
`('T -> 'U option) -> IEvent<'Del,'T> -> IEvent<'U>` 
```

| 返回一个新事件，该事件在原始事件的消息选择上触发。选择函数将原始消息转换为可选的新消息。 |
| --- |

| filter :

```
`('T -> bool) -> IEvent<'Del,'T> -> IEvent<'T>` 
```

| 返回一个新事件，该事件监听原始事件，并且仅在事件传递给给定函数的参数通过时触发结果事件。 |
| --- |

| map :

```
`('T -> 'U) -> IEvent<'Del, 'T> -> IEvent<'U>` 
```

| 返回一个新事件，该事件通过给定函数转换值。 |
| --- |

| merge :

```
`IEvent<'Del1,'T> -> IEvent<'Del2,'T> -> IEvent<'T>` 
```

| 当任一输入事件触发时，触发输出事件。 |
| --- |
| pairwise :`IEvent<'Del,'T> -> IEvent<'T * 'T>` | 返回一个新事件，该事件在输入事件的第二次和后续触发时触发。输入事件的第 N 次触发将传递来自第 N-1 次和第 N 次触发的参数作为一对。传递给第 N-1 次触发的参数在隐藏的内部状态中保持，直到第 N 次触发发生。 |

| partition :

```
`('T -> bool) -> IEvent<'Del,'T> -> IEvent<'T> * IEvent<'T>` 
```

| 返回一对事件，它们监听原始事件。当原始事件触发时，根据谓词的结果，触发这对中的第一个或第二个事件。 |
| --- |

| scan :

```
`('U -> 'T -> 'U) -> 'U -> IEvent<'Del,'T> -> IEvent<'U>` 
```  | 返回一个新事件，该事件由将给定累积函数应用于输入事件上连续触发的值的结果组成。内部状态项记录状态参数的当前值。在累积函数执行期间，内部状态不会被锁定，因此应小心，确保输入`IEvent`不会被多个线程同时触发。 |

| split :

```
`('T -> Choice<'U1,'U2>) -> IEvent<'Del,'T> -> IEvent<'U1> * IEvent<'U2>` 
```

| 返回一个新事件，该事件监听原始事件，并在将函数应用于事件参数返回`Choice1Of2`时触发第一个结果事件，如果返回`Choice2Of2`则触发第二个事件。 |
| --- |

## **了解更多**

关于学习 F# 的更多信息，我推荐 Isaac Abraham 的 *《用 F# 编程：.NET 开发者指南》*（Manning，2018，[www.manning.com/books/get-programming-with-f-sharp](http://www.manning.com/books/get-programming-with-f-sharp)）。
