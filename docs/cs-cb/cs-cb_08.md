# 第八章。模式匹配

历来，开发人员用各种逻辑检查和比较实现业务规则。有时这些规则很复杂，自然而然地导致代码难以编写、阅读和维护。想想你多少次遇到多分支逻辑、多变量比较和多层嵌套的情况。

为了帮助简化这种复杂性，现代编程语言已经开始引入模式匹配——这些语言的特性通过声明性语法帮助将事实与结果匹配。在 C# 中，模式匹配体现为一个不断增长的功能列表，特别是从 C# 7 开始。

本章的主题围绕酒店调度和使用模式进行业务规则。标准通常围绕客户类型如铜牌、银牌或金牌展开，金牌客户由于更频繁的酒店住宿而获得更多积分，是最高级别。

本章讨论了属性、元组和类型的模式匹配。还有一些关于逻辑操作的部分，它们支持和简化多条件模式。令人惊讶的是，C# 从 v1.0 开始就具有某种形式的模式匹配。本章的第一部分讨论了 `is` 和 `as` 操作符，并展示了 `is` 操作符的新增强功能。

# 8.1 安全地转换实例

## 问题

您的遗留代码是弱类型的，依赖于过程式模式，并且需要重构。

## 解决方案

这里是一个接口及其实现类，用于生成我们正在寻找的结果：

```cs
public interface IRoomSchedule
{
    void ScheduleRoom();
}

public class GoldSchedule : IRoomSchedule
{
    public void ScheduleRoom() =>
        Console.WriteLine("Scheduling Gold Room");
}

public class SilverSchedule : IRoomSchedule
{
    public void ScheduleRoom() =>
        Console.WriteLine("Scheduling Silver Room");
}

public class BronzeSchedule : IRoomSchedule
{
    public void ScheduleRoom() =>
        Console.WriteLine("Scheduling Bronze Room");
}
```

这是一个代表返回的遗留非类型化实例数据的方法：

```cs
static ArrayList GetWeakTypedSchedules()
{
    var list = new ArrayList();

    list.Add(new BronzeSchedule());
    list.Add(new SilverSchedule());
    list.Add(new GoldSchedule());

    return list;
}
```

并且这段代码处理遗留集合：

```cs
static void ProcessLegacyCode()
{
    ArrayList schedules = GetWeakTypedSchedules();

    foreach (var schedule in schedules)
    {
        if (schedule is IRoomSchedule)
        {
            IRoomSchedule roomSchedule = (IRoomSchedule)schedule;
            roomSchedule.ScheduleRoom();
        }

        //
        // alternatively
        //

        IRoomSchedule asRoomSchedule = schedule as IRoomSchedule;

        if (asRoomSchedule != null)
            asRoomSchedule.ScheduleRoom();

        //
        // even better
        //

        if (schedule is IRoomSchedule isRoomSchedule)
            isRoomSchedule.ScheduleRoom();
    }
}
```

这里有更现代的代码，返回一个强类型集合：

```cs
static List<IRoomSchedule> GetStrongTypedSchedules()
{
    return new List<IRoomSchedule>
    {
        new BronzeSchedule(),
        new SilverSchedule(),
        new GoldSchedule()
    };
}
```

并且这段代码处理强类型集合：

```cs
static void ProcessModernCode()
{
    List<IRoomSchedule> schedules = GetStrongTypedSchedules();

    foreach (var schedule in schedules)
    {
        schedule.ScheduleRoom();

        if (schedule is GoldSchedule gold)
            Console.WriteLine(
                $"Extra processing for {gold.GetType()}");
    }
}
```

`Main` 方法调用旧版和现代版：

```cs
static void Main()
{
    ProcessLegacyCode();
    ProcessModernCode();
}
```

## 讨论

`as` 和 `is` 操作符在 C# 1 中就已经出现了；您可能已经知道并/或者已经使用过它们。简单回顾一下，`is` 操作符告诉您一个对象的类型是否与正在匹配的类型相同。`as` 操作符将引用类型对象转换为指定类型。如果转换后的实例不是指定的类型，则 `as` 操作符返回 `null`。这个例子还展示了最近 C# 添加的功能，允许使用 `is` 操作符进行类型检查和转换。

我们今天大部分编写的代码使用泛型集合，越来越不需要使用弱类型集合。我敢说，你可能会采纳一个经验法则，以泛型集合作为默认选择，除非无法避免使用弱类型集合。必须使用弱类型集合的一个重要情况是维护已经使用它们的遗留代码。泛型直到 C# 2 才添加，因此你可能会遇到一些使用弱类型集合的旧代码。另一个例子是当你需要或希望使用使用弱类型集合的库时。实际上，你可能不想重写该代码，因为需要的时间和资源——特别是如果它已经经过测试且运行良好。

在解决方案中，`GetWeakTypedSchedules` 方法返回一个 `ArrayList`，这是一个弱类型集合，因为它仅对 `Object` 类型的实例进行操作。`ProcessLegacyCode` 方法调用 `GetWeakTypedSchedules` 并展示了如何使用 `as` 和 `is` 运算符。

第一个 `foreach` 循环中的第一个 `if` 语句使用 `is` 运算符来确定对象是否为 `IRoomSchedule`。如果是，它使用转型运算符获取 `IRoomSchedule` 实例并调用 `GetSchedule`。如果我们已经知道集合包含 `IRoomSchedule` 实例，为什么还需要 `is` 运算符呢？为什么不直接进行转换？问题在于，并不保证集合中的类型是什么。如果开发人员意外加载了一个不是 `IRoomSchedule` 的对象进入集合怎么办？`is` 运算符提高了代码的可靠性。

与 `is` 运算符相对的是 `as` 运算符。在解决方案中，`schedule as IRoomSchedule` 执行转换。如果结果不是 `null`，则对象是 `IRoomSchedule`。这种方法可能性能更好，因为 `is` 操作既检查类型又需要转换，而 `as` 运算符只需要转换和 `null` 检查。

最后一个 `if` 语句演示了更新的 `is` 运算符语法。它既进行类型检查又进行转换，并将结果赋给 `isRoomSchedule` 变量。如果 `schedule` 不是 `IRoomSchedule`，则 `isRoomSchedule` 变量为 `null`，但由于 `is` 运算符返回了一个 `bool` 结果，我们不需要额外的 `null` 检查。

`GetStrongTypedSchedules` 和 `ProcessModernCode` 展示了今天你可能想要编写的代码。请注意，由于强类型化，它具有更少的仪式感。每个类都实现了相同的接口，集合就是该接口，允许你编写有效地操作每个对象的代码。

这个例子还展示了新的`is`运算符在当前代码中可以很有用（不仅限于旧代码）。在`ProcessModernCode`中，即使所有对象都实现了`IRoomSchedule`，`is`运算符也让我们能够检查`GoldSchedule`并进行一些额外处理。

# 8.2 捕获筛选后的异常

## 问题

你需要处理相同异常类型的不同条件逻辑。

## 解决方案

这是一个演示抛出异常的演示类：

```cs
public class Scheduler
{
    public void ScheduleRoom(string arg1, string arg2)
    {
        _ = arg1 ?? throw new ArgumentNullException(nameof(arg1));
        _ = arg2 ?? throw new ArgumentNullException(nameof(arg2));
    }
}
```

这个`Main`方法使用异常过滤器来进行清晰的处理：

```cs
static void Main()
{
    try
    {
        Console.Write("Choose (1) arg1 or (2) arg2? ");
        string arg = Console.ReadLine();

        var scheduler = new Scheduler();

        if (arg == "1")
            scheduler.ScheduleRoom(null, "arg2");
        else
            scheduler.ScheduleRoom("arg1", null);
    }
    catch (ArgumentNullException ex1)
        when (ex1.ParamName == "arg1")
    {
        Console.WriteLine("Invalid arg1");
    }
    catch (ArgumentNullException ex2)
        when (ex2.ParamName == "arg2")
    {
        Console.WriteLine("Invalid arg2");
    }
}
```

## 讨论

C#中一个有趣的补充，与模式匹配相关，是异常过滤器。如你所知，`catch`块根据抛出的异常类型进行操作。然而，当同一类型的异常由于不同原因可能被抛出时，有时能够区分每个原因的处理方式会很有用。虽然你可以在`catch`块中添加`if`或`switch`语句，但过滤器提供了一种清晰分离和简化不同逻辑的方式。

在解决方案中，我们希望根据参数是`null`来过滤`ArgumentNullException`。`ScheduleRoom`方法检查每个参数，如果任一参数为`null`，则抛出`ArgumentNullException`。

`Main`方法在`try`/`catch`块中包装对`ScheduleRoom`的调用。这个例子有两个`catch`块，每个均为`ArgumentNullException`类型。两者之间的区别在于过滤器，由`when`子句指定。`when`子句的参数是一个布尔表达式。在解决方案中，表达式比较了`ParamName`与其处理的参数名。

# 8.3 简化`switch`分配

## 问题

你想根据某些条件返回一个值，但不想从每一个`switch`分支中返回。

## 解决方案

这里是一个接口及其实现类，是我们寻找的结果：

```cs
public interface IRoomSchedule
{
    void ScheduleRoom();
}

public class GoldSchedule : IRoomSchedule
{
    public void ScheduleRoom() =>
        Console.WriteLine("Scheduling Gold Room");
}

public class SilverSchedule : IRoomSchedule
{
    public void ScheduleRoom() =>
        Console.WriteLine("Scheduling Silver Room");
}

public class BronzeSchedule : IRoomSchedule
{
    public void ScheduleRoom() =>
        Console.WriteLine("Scheduling Bronze Room");
}
```

这个枚举将在即将进行的逻辑中使用：

```cs
public enum ScheduleType
{
    None,
    Bronze,
    Silver,
    Gold
}
```

这个类展示了`switch`语句和新的`switch`表达式：

```cs
public class Scheduler
{
    public IRoomSchedule CreateStatement(
        ScheduleType scheduleType)
    {
        switch (scheduleType)
        {
            case ScheduleType.Gold:
                return new GoldSchedule();
            case ScheduleType.Silver:
                return new SilverSchedule();
            case ScheduleType.Bronze:
            default:
                return new BronzeSchedule();
        }
    }

    public IRoomSchedule CreateExpression(
        ScheduleType scheduleType) =>
            scheduleType switch
            {
                ScheduleType.Gold => new GoldSchedule(),
                ScheduleType.Silver => new SilverSchedule(),
                ScheduleType.Bronze => new BronzeSchedule(),
                _ => new BronzeSchedule()
            };
}
```

`Main`方法测试代码：

```cs
static void Main()
{
    Console.Write(
        "Choose (1) Bronze, (2) Silver, or (3) Gold: ");
    string choice = Console.ReadLine();

    Enum.TryParse(choice, out ScheduleType scheduleType);

    var scheduler = new Scheduler();

    IRoomSchedule scheduleStatement =
        scheduler.CreateStatement(scheduleType);
    scheduleStatement.ScheduleRoom();

    IRoomSchedule scheduleExpression =
        scheduler.CreateExpression(scheduleType);
    scheduleExpression.ScheduleRoom();
}
```

## 讨论

`switch`语句从 C# 1 开始存在，而最近的增加是`switch`表达式。`switch`表达式的主要语法特性是一种简写表示法和将结果分配给变量的能力。如果你思考过你使用`switch`语句的所有情况，你可能会注意到产生值或新实例是一个常见主题。`switch`表达式简化了这个主题，并通过模式匹配进一步改进它。

解决方案有两个例子：一个是`switch`语句，一个是`switch`表达式。两者都依赖于`ScheduleType`枚举来进行条件判断，并根据此条件生成一个`IRoomSchedule`类型的结果。

`CreateStatement`方法使用了`switch`语句，其中有针对`ScheduleType`枚举的每个成员的`case`分支。注意它如何从方法中返回值，以及它需要普通的块体语法（带有花括号）。

`CreateExpression`方法使用了新的`switch`表达式。请注意，该方法可以是命令体（带箭头），返回表达式。与`switch`关键字后面的括号中的参数不同，参数位于`switch`关键字之前。此外，与`case`子句不同，案例模式匹配出现在箭头之前，箭头后的结果表达式。默认情况是丢弃模式`_`。

每当参数与案例模式匹配时，`switch`表达式返回结果。在解决方案中，模式是`ScheduleType`枚举的值。`switch`表达式的结果是方法的结果，因为方法的命令语法指定了`switch`表达式。

如果您有一个用例，其中逻辑需要处理每个情况，但不需要返回值，那么经典的`switch`语句可能更合适。但是，如果您可以使用模式匹配并需要返回值，那么`switch`表达式可能是一个很好的选择。

# 8.4 切换属性值

## 问题

您需要基于强类型类属性的业务规则。

## 解决方案

这是一个具有我们需要评估的属性的类：

```cs
public class Room
{
    public int Number { get; set; }
    public string RoomType { get; set; }
    public string BedSize { get; set; }
}
```

这个枚举是评估的结果：

```cs
public enum ScheduleType
{
    None,
    Bronze,
    Silver,
    Gold
}
```

该方法获取我们需要的数据：

```cs
static List<Room> GetRooms()
{
    return new List<Room>
    {
        new Room
        {
            Number = 333,
            BedSize = "King",
            RoomType = "Suite"
        },
        new Room
        {
            Number = 222,
            BedSize = "King",
            RoomType = "Regular"
        },
        new Room
        {
            Number = 111,
            BedSize = "Queen",
            RoomType = "Regular"
        },
    };
}
```

该方法使用该数据并根据匹配模式返回枚举：

```cs
const int RoomNotAvailable = -1;

static int AssignRoom(ScheduleType scheduleType)
{
    foreach (var room in GetRooms())
    {
        ScheduleType roomType = room switch
        {
            { BedSize: "King", RoomType: "Suite" }
                => ScheduleType.Gold,
            { BedSize: "King", RoomType: "Regular" }
                => ScheduleType.Silver,
            { BedSize: "Queen", RoomType: "Regular" }
                => ScheduleType.Bronze,
            _ => ScheduleType.Bronze
        };

        if (roomType == scheduleType)
            return room.Number;
    }

    return RoomNotAvailable;
}
```

`Main`方法驱动程序：

```cs
static void Main()
{
    Console.Write(
        "Choose (1) Bronze, (2) Silver, or (3) Gold: ");
    string choice = Console.ReadLine();

    Enum.TryParse(choice, out ScheduleType scheduleType);

    int roomNumber = AssignRoom(scheduleType);

    if (roomNumber == RoomNotAvailable)
        Console.WriteLine("Room not available.");
    else
        Console.WriteLine($"The room number is {roomNumber}.");
}
```

## 讨论

以前，`switch`语句根据单个参数的值匹配情况。现在，您可以根据对象属性的值进行参数匹配。

解决方案使用`Room`类的实例作为`AssignRoom`方法中`switch`表达式的参数。模式是具有参数属性和匹配值的对象。返回的结果基于参数属性匹配哪种模式而确定。

该程序的目标是为客户找到一个可用的房间。`AssignRoom`的目的是返回与特定调度类型关联的第一个房间。这就是为什么`AssignRoom`比较`roomType`和`scheduleType`，如果它们匹配，则返回的原因。

属性模式匹配是一种很好的方法，因为它易于阅读。这可能会转化为更可维护的代码。一个权衡是，如果要匹配许多属性，则可能会很冗长。下一个配方提供了更短的语法。

## 参见

配方 8.5，“切换元组”

# 8.5 切换元组

## 问题

您需要基于业务规则，并且更喜欢更短的语法。

## 解决方案

这个类很有趣，因为它有一个析构函数：

```cs
public class Room
{
    public int Number { get; set; }
    public string RoomType { get; set; }
    public string BedSize { get; set; }

    public void Deconstruct(out string size, out string type)
    {
        size = BedSize;
        type = RoomType;
    }
}
```

这是程序将生成的枚举：

```cs
public enum ScheduleType
{
    None,
    Bronze,
    Silver,
    Gold
}
```

这是程序将使用的数据：

```cs
static List<Room> GetRooms()
{
    return new List<Room>
    {
        new Room
        {
            Number = 333,
            BedSize = "King",
            RoomType = "Suite"
        },
        new Room
        {
            Number = 222,
            BedSize = "King",
            RoomType = "Regular"
        },
        new Room
        {
            Number = 111,
            BedSize = "Queen",
            RoomType = "Regular"
        },
    };
 }
```

并且这个方法使用从类析构函数返回的元组来确定要返回的枚举：

```cs
static int AssignRoom(ScheduleType scheduleType)
{
    foreach (var room in GetRooms())
    {
        ScheduleType roomType = room switch
        {
            ("King", "Suite") => ScheduleType.Gold,
            ("King", "Regular") => ScheduleType.Silver,
            ("Queen", "Regular") => ScheduleType.Bronze,
            _ => ScheduleType.Bronze
        };

        if (roomType == scheduleType)
            return room.Number;
    }

    return RoomNotAvailable;
}
```

`Main`方法驱动程序：

```cs
static void Main()
{
    Console.Write(
        "Choose (1) Bronze, (2) Silver, or (3) Gold: ");
    string choice = Console.ReadLine();

    Enum.TryParse(choice, out ScheduleType scheduleType);

    int roomNumber = AssignRoom(scheduleType);

    if (roomNumber == RoomNotAvailable)
        Console.WriteLine("Room not available.");
    else
        Console.WriteLine($"The room number is {roomNumber}.");
}
```

## 讨论

在整本书中，你已经看到了元组在管理一组值时有多么有用，而无需所有自定义类型的繁文缛节。元组的快速语法使它们成为简单模式匹配的理想选择。

在这个例子中，我们有一个自定义类型`Room`。注意`Room`有一个自定义的解构器，在这个解决方案中我们将使用它。`GetRooms`方法返回一个`List<Room>`。`AssignRooms`使用了那个集合。然而，由于解构器的存在，我们可以将每个房间用作`switch`表达式参数，它足够聪明以使用解构器生成用于模式匹配的元组。

除了通过解构器使用元组外，这个演示与 Recipe 8.4 相同。在这个例子中，元组提供了更简洁的语法。属性模式更冗长但更易读。一个考虑因素是，如果你匹配标量值，比如`bool`或`int`，属性模式更好地记录了文档。如果你匹配字符串或枚举，元组可能在可读性和更短语法方面提供了最佳选择。因为两种方法之间的选择是情境性的，最好在每种情况下评估权衡，看看哪种对你更有意义。

## 参见

Recipe 8.4，“基于属性值切换”

# 8.6 位置切换

## 问题

你需要基于值的业务规则，但不想创建一个新的一次性类。

## 解决方案

这个枚举是我们将要寻找的结果：

```cs
public enum ScheduleType
{
    None,
    Bronze,
    Silver,
    Gold
}
```

这是一个用于指定决策标准的类：

```cs
public class Room
{
    public int Number { get; set; }
    public string RoomType { get; set; }
    public string BedSize { get; set; }
}
```

这些方法模拟了从两个源获取数据的情况：

```cs
static List<Room> GetHotel1Rooms()
{
    return new List<Room>
    {
        new Room
        {
            Number = 333,
            BedSize = "King",
            RoomType = "Suite"
        },
        new Room
        {
            Number = 111,
            BedSize = "Queen",
            RoomType = "Regular"
        },
    };
}

static List<Room> GetHotel2Rooms()
{
    return new List<Room>
    {
        new Room
        {
            Number = 222,
            BedSize = "King",
            RoomType = "Regular"
        },
    };
}
```

这个方法将这些数据源连接起来，生成一个元组列表：

```cs
static
    List<(int no, string size, string type)>
    GetRooms()
{
    var rooms = GetHotel1Rooms().Union(GetHotel2Rooms());
    return
        (from room in rooms
         select (
            room.Number,
            room.BedSize,
            room.RoomType
         ))
        .ToList();
}
```

这个方法展示了基于位置模式匹配的业务逻辑：

```cs
static int AssignRoom(ScheduleType scheduleType)
{
    foreach (var room in GetRooms())
    {
        ScheduleType roomType = room switch
        {
            (_, "King", "Suite") => ScheduleType.Gold,
            (_, "King", "Regular") => ScheduleType.Silver,
            (_, "Queen", "Regular") => ScheduleType.Bronze,
            _ => ScheduleType.Bronze
        };

        if (roomType == scheduleType)
            return room.no;
    }

    return RoomNotAvailable;
}
```

`Main`方法驱动这个过程：

```cs
static void Main()
{
    Console.Write(
        "Choose (1) Bronze, (2) Silver, or (3) Gold: ");
    string choice = Console.ReadLine();

    Enum.TryParse(choice, out ScheduleType scheduleType);

    int roomNumber = AssignRoom(scheduleType);

    if (roomNumber == RoomNotAvailable)
        Console.WriteLine("Room not available.");
    else
        Console.WriteLine($"The room number is {roomNumber}.");
}
```

## 讨论

这里的解决方案类似于 Recipe 8.5，因为它也使用元组进行模式匹配。这个解决方案的不同之处在于它探讨了当你有两个不同数据源的情况，分别在`GetHotel1Rooms`和`GetHotel2Rooms`中展示，模拟了通常会是数据库查询的情况。这种情况可能发生在公司合并或形成合作伙伴关系时，它们的数据相似但并非完全相同。

`GetRooms`方法展示了如何使用 LINQ 的`Union`运算符来合并这两个列表。方法构建了一个元组集合，而不是为我们需要的值组合创建一个新类型。

当`AssignRooms`调用`GetRooms`时，你不需要在对象上使用解构器，因为你已经在使用元组。如果你正在使用第三方类型而无法修改其成员，这是一种有用的技术。

在`AssignRoom`内部，`switch`表达式使用元组进行匹配。在这里立即引人注目的是第一个参数，表示`Room.Number`属性 - 每个模式都有一个丢弃符号。显然，这在`GetRooms`中可以被省略，但我以这种方式编写是为了阐明几个观点：值的位置必须匹配，并且每个值都是必需的。

元组模式要求值位于正确的位置（例如，不能交换`Number`和`Size`）。每个模式值的位置必须与相应元组位置匹配。相比之下，属性模式可以任意顺序且在不同情况下不同。

对于元组，您必须包括元组每个位置的值。因此，即使在模式中不使用位置，也必须至少指定丢弃参数。属性模式没有此限制，允许您添加或忽略模式中想要的任何属性。

## 参见

[8.5 "元组切换"](https://example.org/switching_on_tuples)的食谱

# 8.7 值范围切换

## 问题

您的业务规则是连续的，而不是离散的。

## 解决方案

这是一个接口，以及实现类，我们正在寻找的结果：

```cs
public interface IRoomSchedule
{
    void ScheduleRoom();
}

public class GoldSchedule : IRoomSchedule
{
    public void ScheduleRoom() =>
        Console.WriteLine("Scheduling Gold Room");
}

public class SilverSchedule : IRoomSchedule
{
    public void ScheduleRoom() =>
        Console.WriteLine("Scheduling Silver Room");
}

public class BronzeSchedule : IRoomSchedule
{
    public void ScheduleRoom() =>
        Console.WriteLine("Scheduling Bronze Room");
}
```

此方法使用关系模式匹配来生成结果：

```cs
const int SilverPoints = 5000;
const int GoldPoints = 20000;

static IRoomSchedule GetSchedule(int points) =>
    points switch
    {
        >= GoldPoints => new GoldSchedule(),
        >= SilverPoints => new SilverSchedule(),
        < SilverPoints => new BronzeSchedule()
    };
```

`Main`方法驱动整个过程：

```cs
static void Main()
{
    Console.Write("How many points? ");
    string response = Console.ReadLine();

    if (!int.TryParse(response, out int points))
    {
        Console.WriteLine($"'{response}' is invalid!");
        return;
    }

    IRoomSchedule schedule = GetSchedule(points);

    schedule.ScheduleRoom();
}
```

## 讨论

本章的前几节探讨了基于离散值的模式匹配。模式必须精确匹配才能成功。但是，有很多情况下，值是连续的，而不是离散的。本节中的解决方案就是一个例子，酒店客户的积分范围可能各不相同。

在解决方案中，积分从 0 到 4,999 的客户为青铜。积分从 5,000 到 19,999 的客户为银。积分达到或超过 20,000 的客户为金。解决方案中的`SilverPoints`和`GoldPoints`常量定义了边界。

`Main`方法询问客户的积分数，并将该值传递给`GetSchedule`。这个值可能会变化，这取决于一个人预订房间或使用其他酒店服务的次数。因此，`GetSchedule`根据这些积分使用`switch`表达式。而不是使用离散模式进行匹配，`GetSchedule`使用关系运算符。

第一个模式询问`points`是否等于或高于`GoldPoints`。如果不是，`points`必须更少，并且代码检查是否等于或高于`SilverPoints`。由于我们已经评估了`GoldPoints`情况，所以这意味着范围在`SilverPoints`和`GoldPoints`之间。最后一个情况，低于`SilverPoints`，说明了青铜的含义，但您可以很容易地用丢弃模式替换它，因为其他两种情况处理了所有其他可能性，而青铜是唯一剩下的。

# 8.8 复杂条件切换

## 问题

您的业务规则是多条件的。

## 解决方案

这是一个接口，以及实现类，我们正在寻找的结果：

```cs
public interface IRoomSchedule
{
    void ScheduleRoom();
}

public class GoldSchedule : IRoomSchedule
{
    public void ScheduleRoom() =>
        Console.WriteLine("Scheduling Gold Room");
}

public class SilverSchedule : IRoomSchedule
{
    public void ScheduleRoom() =>
        Console.WriteLine("Scheduling Silver Room");
}

public class BronzeSchedule : IRoomSchedule
{
    public void ScheduleRoom() =>
        Console.WriteLine("Scheduling Bronze Room");
}
```

本课程描述了使用的标准：

```cs
public class Customer
{
    public int Points { get; set; }

    public bool HasFreeUpgrade { get; set; }
}
```

此方法生成具有各种值的模拟数据，以演示我们的逻辑：

```cs
static List<Customer> GetCustomers() =>
    new List<Customer>
    {
        new Customer
        {
            Points = 25000,
            HasFreeUpgrade = false
        },
        new Customer
        {
            Points = 10000,
            HasFreeUpgrade = true
        },
        new Customer
        {
            Points = 1000,
            HasFreeUpgrade = true
        },
    };
```

这是一个使用`switch`表达式中复杂逻辑的方法：

```cs
static IRoomSchedule GetSchedule(Customer customer) =>
    customer switch
    {
        Customer c
            when
                c.Points >= GoldPoints
                    ||
                (c.Points >= SilverPoints && c.HasFreeUpgrade)
            => new GoldSchedule(),

        Customer c
            when
                c.Points >= SilverPoints
                    ||
                (c.Points < SilverPoints && c.HasFreeUpgrade)
            => new SilverSchedule(),

        Customer c
            when
                c.Points < SilverPoints
            => new BronzeSchedule(),

        _ => new BronzeSchedule()
    };
```

`Main`方法遍历结果：

```cs
static void Main()
{
    foreach (var customer in GetCustomers())
    {
        IRoomSchedule schedule = GetSchedule(customer);
        schedule.ScheduleRoom();
    }
}
```

## 讨论

有时候条件非常复杂，无法通过本章前面展示的技术解决问题。例如，本节中的解决方案需要涉及多个属性的多个条件。在这里，我们使用`switch`表达式和`when`子句来指定匹配。

此场景基于`Customer`类型，指示点数数量以及客户是否有免费升级。免费升级可能来自于比赛或酒店促销活动。在安排房间时，我们希望确保客户获得与其点数水平相称的房间。此外，如果他们有免费升级选项，则会获得升级到下一个更高级别的房间。为简便起见，我们方便地忽略了金牌是否有免费升级。

`GetSchedule`方法操作`Customer`的一个实例。金牌和银牌的情况都会导致相应级别的房间。此外，`||`运算符表示，如果`customer`处于下一个较低级别，但`HasFreeUpgrade`为`true`，则结果是此较高级别的房间。

使用这样的逻辑会很快变得复杂。请注意使用换行和其他间距来增加结果的对称性和一致性以方便阅读。

当逻辑比离散模式匹配复杂时，这种技术可以帮助您。您可能希望考虑使用`if`语句的阈值作为更好的实现。一个考虑因素是维护，因为将每个逻辑片段分解出来有助于调试，而单个表达式具有多个条件可能不会立即明显。

# 8.9 使用逻辑条件

## 问题

您希望多条件逻辑更易读。

## 解决方案

这是一个作为标准使用的类：

```cs
public class Customer
{
    public int Points { get; set; }

    public int Month { get; set; }
}
```

此方法模拟数据源：

```cs
static List<Customer> GetCustomers() =>
    new List<Customer>
    {
        new Customer
        {
            Points = 25000,
            Month = 1
        },
        new Customer
        {
            Points = 10000,
            Month = 12
        },
        new Customer
        {
            Points = 10000,
            Month = 11
        },
        new Customer
        {
            Points = 1000,
            Month = 2
        },
    };
```

此方法在`switch`表达式中实现了业务规则和条件逻辑：

```cs
const int SilverPoints = 5000;
const int GoldPoints = 20000;

const int May = 5;
const int Sep = 9;
const int Dec = 12;

static decimal GetDiscount(Customer customer) =>
    (customer.Points, customer.Month) switch
    {
        (>= GoldPoints, not Dec and > Sep or < May) => 0.15m,
        (>= GoldPoints, Dec) => 0.10m,
        (>= SilverPoints, not (Dec or <= Sep and >= May)) => 0.05m,
        _ => 0.0m
    };
```

`Main`方法驱动此过程：

```cs
static void Main()
{
    foreach (var customer in GetCustomers())
    {
        decimal discount = GetDiscount(customer);
        Console.WriteLine(discount);
    }
}
```

## 讨论

Recipe 8.8 描述了如何向`switch`表达式添加复杂逻辑。在这里，我指的是涉及两个或更多属性的多个条件。这与本章先前使用的属性和元组模式进行简单模式匹配形成对比。在这些对比方法之间的某处，是一个需要逻辑隔离在各个属性内的适度方法。

此解决方案中感兴趣的属性是 `Customer` 类的 `Points` 和 `Month`。与前几节类似，`Points` 属性有助于为至少具有一定数量点数的客户预订房间。另一个条件 `Month` 是客户想要预订房间的月份。由于季节性供需，一些月份会留给酒店更多的空房。因此，此应用程序根据积分提供激励，鼓励客户在空房较多的月份预订房间。

在解决方案中，你可以看到 `GoldPoints` 和 `SilverPoints` 常量，用于确定客户的等级。同时，还有 `May`，`Sep` 和 `Dec` 这些繁忙月份的常量。逻辑是在非繁忙月份给予折扣。

`GetDiscount` 方法的 `switch` 表达式的模式匹配基于两个属性：`Points` 和 `Month`。请注意，这段代码不依赖于对象解构，而原始参数是一个类，而不是元组。`GetDiscount` 为 `switch` 表达式创建了一个内联元组。

模式本身依赖于 `Points` 的关系运算符，就像 Recipe 8.7 中一样。

`Month` 模式使用了新的 C# 9 逻辑运算符：`and`，`not` 和 `or`。第一个表达式确保客户在冬季月份（`Sep` 到 `May` 之间，除了 `Dec`）期间享受折扣。第二个模式表示金牌客户在 `Dec` 仍然享受折扣，但是折扣从 15% 变为 10%。

最后一个模式在逻辑上等同于第一个模式，并使用了德摩根定理。也就是说，它否定了整个结果，并交换了 `and` 和 `or`。因为最后一个示例将 `not` 应用于整个表达式，所以它使用了括号。而在第一个模式中，`not` 仅应用于 `Dec`。

## 参见

Recipe 8.7，“在值范围上进行切换”

Recipe 8.8，“使用复杂条件进行切换”

# 8.10 类型切换

## 问题

你需要对象的类型来做出决定。

## 解决方案

这里有一个接口，以及实现了我们需要的结果的类：

```cs
public interface IRoomSchedule
{
    void ScheduleRoom();
}

public class GoldSchedule : IRoomSchedule
{
    public void ScheduleRoom() =>
        Console.WriteLine("Scheduling Gold Room");
}

public class SilverSchedule : IRoomSchedule
{
    public void ScheduleRoom() =>
        Console.WriteLine("Scheduling Silver Room");
}

public class BronzeSchedule : IRoomSchedule
{
    public void ScheduleRoom() =>
        Console.WriteLine("Scheduling Bronze Room");
}
```

以下类型代表条件：

```cs
public class Customer {}

public class GoldCustomer : Customer {}

public class SilverCustomer : Customer {}

public class BronzeCustomer : Customer {}
```

此方法模拟了一个数据源：

```cs
static List<Customer> GetCustomers() =>
    new List<Customer>
    {
        new GoldCustomer(),
        new SilverCustomer(),
        new BronzeCustomer()
    };
```

下面是一个根据类型模式匹配实现逻辑的方法：

```cs
static IRoomSchedule GetSchedule(Customer customer) =>
    customer switch
    {
        GoldCustomer => new GoldSchedule(),
        SilverCustomer => new SilverSchedule(),
        BronzeCustomer => new BronzeSchedule(),
        _ => new BronzeSchedule()
    };
```

`Main` 方法通过数据迭代来执行模式匹配逻辑：

```cs
static void Main()
{
    foreach (var customer in GetCustomers())
    {
        IRoomSchedule schedule = GetSchedule(customer);
        schedule.ScheduleRoom();
    }
}
```

## 讨论

过去，唯一能够根据类型做出决策的方式是使用 `if` 语句或将对象类型转换为 `string`，并使用带有 `string` 情况的 `switch` 语句。多年来对 C# 的一个常见请求是允许使用类型 case 的 `switch` 语句，现在我们终于有了。

解决方案包含一组类：`GoldCustomer`，`SilverCustomer` 和 `BronzeCustomer`，它们都是从 `Customer` 派生的。我们在这个程序中的目标是根据匹配的类类型安排一个房间。

`GetSchedule` 方法通过接受一个类型为 `Customer` 的对象来进行调度，而 `switch` 表达式针对从 `Customer` 派生的每个类都有一个模式。你只需要指定每个类的名称，`switch` 表达式会根据对象类型进行匹配。
