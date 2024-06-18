# 第三十七章：运算符重载

你可以重载运算符以为自定义类型提供更自然的语法。运算符重载最适合用于实现代表相当原始数据类型的自定义结构体。例如，自定义数值类型非常适合运算符重载。

你可以重载以下符号运算符：

```cs
+   -   *   /   ++   --   !   ~   %   &   |   ^
==  !=  <   <<  >>   >
```

你可以重写隐式和显式转换（使用`implicit`和`explicit`关键字），以及`true`和`false`运算符。

当你重载非复合运算符（例如`+`，`/`）时，复合赋值运算符（例如`+=`，`/=`）会自动被重载。

## 运算符函数

要重载运算符，你需要声明一个*运算符函数*。运算符函数必须是静态的，并且其中至少一个操作数必须是声明运算符函数的类型。

在下面的示例中，我们定义了一个名为`Note`的结构体，代表一个音符，并重载了`+`运算符：

```cs
public struct Note
{
  int value;

  public Note (int semitonesFromA) 
    => value = semitonesFromA;

 public static Note operator + (Note x, int semitones)
    => new Note (x.value + semitones);
}
```

此重载允许我们将一个`int`加到一个`Note`上：

```cs
Note B = new Note (2);
Note CSharp = B + 2;
```

因为我们重载了`+`，所以我们也可以使用`+=`：

```cs
CSharp += 2;
```

从 C# 11 开始，你也可以声明一个`checked`版本，它将在检查表达式或块内调用。这被称为*checked 运算符*：

```cs
public static Note operator checked + (Note x, int semis)
  => checked (new Note (x.value + semis));
```

## 重载相等性和比较运算符

当编写结构体时，经常会重写相等性和比较运算符，在类中则很少见。当重载这些运算符时，会应用特殊的规则和义务：

配对

C# 编译器强制要求逻辑对的运算符都要定义。这些运算符包括(`== !=`)，(`< >`)以及(`<= >=`)。

`Equals`和`GetHashCode`

如果你重载了`==`和`!=`，通常需要重写`object`的`Equals`和`GetHashCode`方法，以便类型可以可靠地与集合和哈希表一起工作。

`IComparable`和`IComparable<T>`

如果你重载了`<`和`>`，通常还需要实现`IComparable`和`IComparable<T>`。

扩展前面的示例，这是如何重载`Note`的相等运算符的方法：

```cs
public static bool operator == (Note n1, Note n2)
  => n1.value == n2.value;

public static bool operator != (Note n1, Note n2)
  => !(n1.value == n2.value);

public override bool Equals (object otherNote)
{
  if (!(otherNote is Note)) return false;
  return this == (Note)otherNote;
}
// value’s hashcode will work for our own hashcode:
public override int GetHashCode() => value.GetHashCode();
```

## 自定义隐式和显式转换

隐式和显式转换是可重载的运算符。通常通过重载这些转换来使得在强关联类型（如数值类型）之间转换更加简洁和自然。

正如类型讨论中所解释的，隐式转换的理念在于它们应始终成功，并且在转换过程中不丢失信息。否则，应定义显式转换。

在下面的示例中，我们定义了我们的音乐`Note`类型和`double`之间的转换（表示该音符的频率以赫兹为单位）：

```cs
...
// Convert to hertz
public static implicit operator double (Note x)
  => 440 * Math.Pow (2,(double) x.value / 12 );

// Convert from hertz (accurate to nearest semitone)
public static explicit operator Note (double x)
  => new Note ((int) (0.5 + 12 * (Math.Log(x/440)
               / Math.Log(2)) ));
...

Note n =(Note)554.37;  // explicit conversion
double x = n;          // implicit conversion
```

###### 注意

这个示例有些假设性：在实际应用中，这些转换可能更适合通过`ToFrequency`方法和（静态）`FromFrequency`方法来实现。

自定义转换被`as`和`is`运算符忽略。

