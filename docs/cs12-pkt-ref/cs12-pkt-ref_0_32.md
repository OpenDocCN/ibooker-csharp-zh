# 第三十二章：元组

与匿名类型类似，*元组*（C# 7+）提供了一种简单的方法来存储一组值。元组是为了允许方法返回多个值而引入到 C# 中的，而无需使用 `out` 参数（这是匿名类型无法做到的）。然而，此后，*记录*已被引入，提供了一种简洁的类型化方法，我们将在接下来的章节中描述。

创建*元组字面量*的最简单方法是在括号中列出所需的值。这将创建一个具有*未命名*元素的元组：

```cs
var bob = ("Bob", 23);
Console.WriteLine (bob.Item1);   // Bob
Console.WriteLine (bob.Item2);   // 23
```

与匿名类型不同，`var` 是可选的，并且你可以明确指定*元组类型*：

```cs
(string,int) bob  = ("Bob", 23);
```

这意味着你可以从方法中有用地返回一个元组：

```cs
(string,int) person = GetPerson();
Console.WriteLine (person.Item1);    // Bob
Console.WriteLine (person.Item2);    // 23

(string,int) GetPerson() => ("Bob", 23);
```

元组与泛型兼容，因此以下类型都是合法的：

```cs
Task<(string,int)>
Dictionary<(string,int),Uri>
IEnumerable<(int ID, string Name)>   // See below...
```

元组是*值类型*，其元素是*可变*（读写）的。这意味着创建元组后，你可以修改`Item1`、`Item2`等元素。

## 命名元组元素

当创建元组字面量时，你可以选择为元素指定有意义的名称：

```cs
var tuple = (Name:"Bob", Age:23);
Console.WriteLine (tuple.Name);     // Bob
Console.WriteLine (tuple.Age);      // 23
```

当指定*元组类型*时，你也可以这样做：

```cs
static (string Name, int Age) GetPerson() => ("Bob",23);
```

元素名称从属性或字段名称*推断*出：

```cs
var now = DateTime.Now;
var tuple = (now.Day, now.Month, now.Year);
Console.WriteLine (tuple.Day);               // OK
```

###### 注意

元组是语法糖，用于使用名为 `ValueTuple<T1>` 和 `ValueTuple<T1,T2>` 的一组泛型结构体，这些结构体具有名为 `Item1`、`Item2` 等的字段。因此 `(string,int)` 是 `ValueTuple<string,int>` 的别名。这意味着“命名元素”仅存在于源代码中——以及编译器的想象中——并在运行时大多数时候消失。

## 解构元组

元组隐式支持解构模式（参见“解构器”），因此你可以轻松地将元组*解构*为单独的变量。考虑以下示例：

```cs
var bob = ("Bob", 23);
string name = bob.Item1;
int age = bob.Item2;
```

使用元组的解构器，你可以将代码简化为这样：

```cs
var bob = ("Bob", 23);
(string name, int age) = bob;   // Deconstruct bob into
 // name and age.
Console.WriteLine (name);
Console.WriteLine (age);
```

解构语法与声明具有命名元素的元组的语法令人困惑地相似！以下突出了它们的区别：

```cs
(string name, int age)      = bob;  // Deconstructing
(string name, int age) bob2 = bob;  // Declaring tuple
```

