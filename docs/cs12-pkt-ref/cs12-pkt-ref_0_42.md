# 第四十二章：不安全代码和指针

C# 支持在标记为`unsafe`的代码块内通过指针进行直接内存操作。指针类型对于与本机 API 进行交互、访问托管堆之外的内存以及在性能关键的热点中实现微优化非常有用。

包含不安全代码的项目必须在项目文件中指定`<AllowUnsafeBlocks>true</AllowUnsafeBlocks>`。

## 指针基础知识

对于每种值类型或引用类型*V*，都有相应的指针类型*V**。指针实例保存变量的地址。指针类型可以（不安全地）转换为任何其他指针类型。以下是主要的指针运算符：

| 运算符 | 含义 |
| --- | --- |
| `&` | *取地址*运算符返回指向变量地址的指针。 |
| `*` | *解引用*运算符返回指针地址处的变量。 |
| `->` | *成员指针*运算符是一种语法快捷方式，其中`x->y`等同于`(*x).y`。 |

## 不安全代码

通过使用`unsafe`关键字标记类型、类型成员或语句块，您被允许在该作用域内使用指针类型，并执行 C++ 风格的指针操作。以下是使用指针快速处理位图的示例：

```cs
unsafe void BlueFilter (int[,] bitmap)
{
  int length = bitmap.Length;
  fixed (int* b = bitmap)
  {
    int* p = b;
    for (int i = 0; i < length; i++)
      *p++ &= 0xFF;
  }
}
```

不安全代码可能比相应的安全实现运行更快。在这种情况下，代码需要具有带有数组索引和边界检查的嵌套循环。不安全的 C# 方法也可能比调用外部 C 函数更快，因为没有离开托管执行环境的开销。

## fixed 语句

`fixed`语句用于固定管理对象，例如前面示例中的位图。在程序执行期间，许多对象从堆中分配和释放。为了避免内存的不必要浪费或碎片化，垃圾回收器会移动对象。如果在引用时对象的地址可能会更改，则指向对象是徒劳的，因此`fixed`语句指示垃圾回收器“固定”对象并防止其移动。这可能会影响运行时的效率，因此您应该仅在短时间内使用固定块，并避免在固定块内分配堆内存。

在`fixed`语句中，您可以获取值类型、值类型数组或字符串的指针。对于数组和字符串，指针实际上将指向第一个元素，该元素是值类型。

在引用类型内部声明的值类型需要引用类型被固定，如下所示：

```cs
Test test = new Test();
unsafe
{
  fixed (int* p = &test.X)   // Pins test
  {
    *p = 9;
  }
  Console.WriteLine (test.X);
}

class Test { public int X; }
```

## 成员指针运算符

除了`&`和`*`运算符外，C#还提供了 C++风格的`->`运算符，您可以在结构体上使用它：

```cs
Test test = new Test();
unsafe
{
  Test* p = &test;
  p->X = 9;
  System.Console.WriteLine (test.X);
}

struct Test { public int X; }
```

## `stackalloc`关键字

您可以使用`stackalloc`关键字显式在堆栈上分配内存块。因为它是在堆栈上分配的，所以它的生存期仅限于方法的执行，就像任何其他局部变量一样。该块可以使用`[]`运算符索引到内存中：

```cs
int* a = stackalloc int [10];
for (int i = 0; i < 10; ++i)
  Console.WriteLine (a[i]);   // Print raw memory
```

## 固定大小的缓冲区

要在结构体中分配一块内存块，请使用`fixed`关键字：

```cs
unsafe struct UnsafeUnicodeString
{
  public short Length;
  public fixed byte Buffer[30];
}

unsafe class UnsafeClass
{
  UnsafeUnicodeString uus;

  public UnsafeClass (string s)
  {
    uus.Length = (short)s.Length;
    fixed (byte* p = uus.Buffer)
      for (int i = 0; i < s.Length; i++)
        p[i] = (byte) s[i];
  }
}
```

固定大小的缓冲区不是数组：如果`Buffer`是数组，它将由存储在结构体内部的对象引用组成，而不是结构体本身的 30 个字节。

此示例中还使用了`fixed`关键字来固定包含缓冲区的堆上对象（这将是`UnsafeClass`的实例）。

## void*

*void 指针*（`void*`）对底层数据类型不做任何假设，并且对处理原始内存的函数非常有用。任何指针类型都可以隐式转换为`void*`。`void*`无法被解引用，并且不能对 void 指针执行算术运算。例如：

```cs
short[] a = {1,1,2,3,5,8,13,21,34,55};
fixed (short* p = a)
{
  //sizeof returns size of value-type in bytes
  Zap (p, a.Length * sizeof (short));
}
foreach (short x in a)
  Console.WriteLine (x);  // Prints all zeros

unsafe void Zap (void* memory, int byteCount)
{
  byte* b = (byte*) memory;
    for (int i = 0; i < byteCount; i++)
      *b++ = 0;
}
```

## 函数指针

*函数指针*（来自 C# 9）类似于委托，但没有委托实例的间接性；而是直接指向方法。函数指针仅指向静态方法，缺乏多播功能，并且需要`unsafe`上下文（因为它绕过运行时类型安全性）。其主要目的是简化和优化与不受管 API 的互操作（我们在*C# 12 in a Nutshell*中介绍了互操作）。

函数指针类型声明如下（返回类型最后出现）：

```cs
delegate*<int, char, string, void>
```

这与具有此签名的函数匹配：

```cs
void SomeFunction (int x, char y, string z)
```

`&`运算符从方法组创建函数指针。这里有一个完整的示例：

```cs
unsafe
{
 delegate*<string, int> functionPointer = &GetLength;
  int length = functionPointer ("Hello, world");

  static int GetLength (string s) => s.Length;
}
```

在此示例中，`functionPointer` 不是您可以调用 `Invoke` 方法的 *对象*。而是一个直接指向目标方法内存地址的变量：

```cs
Console.WriteLine ((IntPtr)functionPointer);
```

