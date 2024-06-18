# 第七章：数组

*数组*表示特定类型的固定数量元素。数组中的元素始终存储在连续的内存块中，提供高效的访问。

数组在元素类型后用方括号表示。以下声明了一个包含五个字符的数组：

```cs
char[] vowels = new char[5];
```

方括号还*索引*数组，访问特定位置的元素：

```cs
vowels[0] = 'a'; vowels[1] = 'e'; vowels[2] = 'i';
vowels[3] = 'o'; vowels[4] = 'u';

Console.WriteLine (vowels [1]);      // e
```

这会打印“e”，因为数组索引从 0 开始。您可以使用`for`循环语句遍历数组中的每个元素。在这个例子中，`for`循环循环整数`i`从`0`到`4`：

```cs
for (int i = 0; i < vowels.Length; i++)
  Console.Write (vowels [i]);            // aeiou
```

数组还实现了`IEnumerable<T>`（见“枚举和迭代器”），因此您也可以使用`foreach`语句枚举成员：

```cs
foreach (char c in vowels) Console.Write (c);  // aeiou
```

运行时对所有数组索引进行边界检查。如果使用无效索引，将抛出`IndexOutOfRangeException`：

```cs
vowels[5] = 'y';   // Runtime error
```

数组的`Length`属性返回数组中的元素数。创建数组后，其长度不可更改。`System.Collection`命名空间和子命名空间提供了更高级的数据结构，如动态大小的数组和字典。

*数组初始化表达式*允许您在一步中声明并填充数组：

```cs
char[] vowels = new char[] {'a','e','i','o','u'};
```

或者简单地说：

```cs
char[] vowels = {'a','e','i','o','u'};
```

###### 注意

从 C# 12 开始，您可以使用方括号代替大括号：

```cs
char[] vowels = ['a','e','i','o','u'];
```

这被称为*集合表达式*，其优点是在调用方法时同样适用：

```cs
Foo (['a','e','i','o','u']);
void Foo (char[] letters) { ... }
```

集合表达式也适用于其他集合类型，如列表和集合——见“集合初始化器和集合表达式”。

所有数组都继承自`System.Array`类，该类为所有数组定义了常见的方法和属性。这包括实例属性如`Length`和`Rank`，以及用于执行以下操作的静态方法：

+   动态创建一个数组（`CreateInstance`）

+   获取和设置元素，无论数组类型如何（`GetValue`/`SetValue`）

+   搜索已排序的数组（`BinarySearch`）或未排序的数组（`IndexOf`，`LastIndexOf`，`Find`，`FindIndex`，`FindLastIndex`）

+   对数组进行排序（`Sort`）

+   复制数组（`Copy`）

## 默认元素初始化

创建数组总是用默认值预初始化元素。类型的默认值是内存位清零的结果。例如，考虑创建一个整数数组。因为`int`是值类型，这将在一个连续的内存块中分配 1,000 个整数。每个元素的默认值将为 0：

```cs
int[] a = new int[1000];
Console.Write (a[123]);            // 0
```

对于引用类型的元素，其默认值为`null`。

数组*本身*始终是引用类型对象，无论元素类型如何。例如，以下语句是合法的：

```cs
int[] a = null;
```

## 索引和范围

*索引和范围*（从 C# 8 开始）简化了使用数组元素或部分的操作。

###### 注意

索引和范围也适用于 CLR 类型`Span<T>`和`ReadOnlySpan<T>`，它们提供对托管或非托管内存的高效低级访问。

您还可以通过定义类型为`Index`或`Range`的索引器来使自定义类型与索引和范围一起使用（参见“索引器”）。

### 索引

索引允许您相对于数组的*末尾*引用元素，使用`^`运算符。`¹`指的是最后一个元素，`²`指的是倒数第二个元素，依此类推：

```cs
char[] vowels = new char[] {'a','e','i','o','u'};
char lastElement  = vowels[¹];   // 'u'
char secondToLast = vowels[²];   // 'o'
```

（`⁰`等于数组的长度，因此`vowels[⁰]`会生成错误。）

C#借助`Index`类型实现索引，因此您也可以执行以下操作：

```cs
Index first = 0;
Index last = ¹;
char firstElement = vowels [first];   // 'a'
char lastElement = vowels [last];     // 'u'
```

### 范围

范围允许您使用`..`运算符“切片”数组：

```cs
char[] firstTwo =  vowels [..2];    // 'a', 'e'
char[] lastThree = vowels [2..];    // 'i', 'o', 'u'
char[] middleOne = vowels [2..3];   // 'i'
```

范围中的第二个数字是*排除的*，因此`..2`返回`vowels[2]`*之前*的元素。

您还可以在范围中使用`^`符号。以下返回最后两个字符：

```cs
char[] lastTwo = vowels [²..⁰];    // 'o', 'u'
```

（这里`⁰`有效，因为范围中的第二个数字是*排除的*。）

C#借助`Range`类型实现范围，因此您也可以执行以下操作：

```cs
Range firstTwoRange = 0..2;
char[] firstTwo = vowels [firstTwoRange];   // 'a', 'e'
```

## 多维数组

多维数组有两种类型：*矩形*和*锯齿*。矩形数组表示一个*n*维内存块，而锯齿数组则是数组的数组。

### 矩形数组

要声明矩形数组，请使用逗号分隔每个维度。以下声明一个矩形二维数组，其维度为 3 × 3：

```cs
int[,] matrix = new int [3, 3];
```

数组的`GetLength`方法返回给定维度的长度（从 0 开始）：

```cs
for (int i = 0; i < matrix.GetLength(0); i++)
  for (int j = 0; j < matrix.GetLength(1); j++)
    matrix [i, j] = i * 3 + j;
```

可以通过以下方式初始化矩形数组（创建与前面示例相同的数组）：

```cs
int[,] matrix = new int[,]
{
  {0,1,2},
  {3,4,5},
  {6,7,8}
};
```

（在此类声明语句中可以省略粗体显示的代码。）

### 锯齿数组

要声明锯齿数组，请为每个维度使用连续的方括号对。以下是声明锯齿二维数组的示例，其中最外层维度为 3：

```cs
int[][] matrix = new int[3][];
```

在声明中未指定内部维度，因为与矩形数组不同，每个内部数组的长度都可以是任意的。每个内部数组隐式初始化为 null 而不是空数组。每个内部数组必须手动创建：

```cs
for (int i = 0; i < matrix.Length; i++)
{
  matrix[i] = new int [3];       // Create inner array
  for (int j = 0; j < matrix[i].Length; j++)
    matrix[i][j] = i * 3 + j;
}
```

可以通过以下方式初始化锯齿数组（创建与前面示例相同的数组，但在末尾增加一个元素）：

```cs
int[][] matrix = new int[][]
{
  new int[] {0,1,2},
  new int[] {3,4,5},
  new int[] {6,7,8,9}
};
```

（在此类声明语句中可以省略粗体显示的代码。）

## 简化的数组初始化表达式

我们已经看到如何通过省略`new`关键字和类型声明来简化数组初始化表达式：

```cs
char[] vowels = new char[] {'a','e','i','o','u'};
char[] vowels =            {'a','e','i','o','u'};
char[] vowels =            ['a','e','i','o','u'];
```

另一种方法是使用`var`关键字，该关键字指示编译器隐式地为局部变量确定类型。以下是一些简单的示例：

```cs
var i = 3;           // i is implicitly of type int
var s = "sausage";   // s is implicitly of type string
```

相同的原理可以应用于数组，但可以进一步进行。通过在`new`关键字后省略类型限定符，编译器推断数组类型：

```cs
// Compiler infers char[]
var vowels = new[] {'a','e','i','o','u'};
```

下面我们来看如何将其应用于多维数组：

```cs
var rectMatrix = new[,]
{
  {0,1,2},
  {3,4,5},
  {6,7,8}
};
var jaggedMat = new int[][]
{
  new[] {0,1,2},
  new[] {3,4,5},
  new[] {6,7,8,9}
};
```

