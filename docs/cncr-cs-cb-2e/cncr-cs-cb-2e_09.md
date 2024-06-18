# 第九章。集合

在并发应用程序中，使用适当的集合是至关重要的。我不是在谈论像`List<T>`这样的标准集合；我假设你已经了解了这些。本章的目的是介绍专门用于并发或异步使用的新集合。

*不可变集合* 是永远不会改变的集合实例。乍一看，这听起来完全没用；但实际上它们非常有用，即使在单线程、非并发应用程序中也是如此。只读操作（例如枚举）直接作用于不可变实例。写操作（例如添加项目）返回一个新的不可变实例，而不是更改现有实例。这并不像听起来的那么浪费，因为大多数情况下，不可变集合共享大部分内存。此外，不可变集合具有隐式安全的优势，可以从多个线程安全访问；因为它们不能改变，所以它们是线程安全的。

###### 小贴士

不可变集合位于[`System.Collections.Immutable`](http://bit.ly/sys-coll-imm) NuGet 包中。

不可变集合是新的，但在新开发中应考虑使用它们，除非你*需要*一个可变实例。如果你不熟悉不可变集合，我建议你从 Recipe 9.1 开始，即使你不需要栈或队列，因为我将覆盖所有不可变集合遵循的几种常见模式。

有特殊的方法可以更有效地构建具有大量现有元素的不可变集合；这些示例代码仅逐个添加元素。如果需要加快初始化速度，MSDN 文档详细介绍了如何高效构建不可变集合。

线程安全集合

这些可变集合实例可以同时被多个线程修改。线程安全的集合使用细粒度锁和无锁技术的混合方式，以确保线程被阻塞的时间最少（通常根本不会被阻塞）。对于许多线程安全集合，枚举集合会创建集合的快照，然后枚举该快照。线程安全集合的关键优势在于，它们可以安全地从多个线程访问，但操作只会在很短的时间内（如果有的话）阻塞你的代码。

生产者/消费者集合

这些可变集合实例被设计用于特定目的：允许（可能多个）生产者向集合推送项目，同时允许（可能多个）消费者从集合中取出项目。因此，它们充当生产者代码和消费者代码之间的桥梁，同时还具有限制集合中项目数量的选项。生产者/消费者集合可以具有阻塞或异步 API。例如，当集合为空时，阻塞生产者/消费者集合将阻塞调用的消费者线程，直到添加另一个项目；但异步生产者/消费者集合将允许调用的消费者线程异步等待直到添加另一个项目。

本章节中使用了许多不同的生产者/消费者集合，不同的生产者/消费者集合具有不同的优势。查看 表 9-1 可以帮助确定您应该使用哪一个。

表 9-1\. 生产者/消费者集合

| 特性 | Channels | BlockingCollection<T> | BufferBlock<T> | AsyncProducer-ConsumerQueue<T> | AsyncCollection<T> |
| --- | --- | --- | --- | --- | --- |
| 队列语义 | ✓ | ✓ | ✓ | ✓ | ✓ |
| 堆栈/袋子语义 | ✗ | ✓ | ✗ | ✗ | ✓ |
| 同步 API | ✓ | ✓ | ✓ | ✓ | ✓ |
| 异步 API | ✓ | ✗ | ✓ | ✓ | ✓ |
| 当满时丢弃项目 | ✓ | ✗ | ✗ | ✗ | ✗ |
| 由 Microsoft 测试 | ✓ | ✓ | ✓ | ✗ | ✗ |

###### 提示

Channels 可在 [`System.Threading.Channels`](http://bit.ly/sys-thrd-chanls) NuGet 包中找到，`BufferBlock<T>` 在 [`System.Threading.Tasks.Dataflow`](http://bit.ly/nuget-df) NuGet 包中，`AsyncProducerConsumerQueue<T>` 和 `AsyncCollection<T>` 在 [`Nito.AsyncEx`](http://bit.ly/nito-async) NuGet 包中。

# 9.1 不可变堆栈和队列

## 问题

您需要一个不经常更改且可以安全地被多个线程访问的堆栈或队列。

例如，队列可以用作执行操作的序列，堆栈可以用作撤销操作的序列。

## 解决方案

不可变堆栈和队列是最简单的不可变集合。它们的行为与标准`Stack<T>`和`Queue<T>`非常相似。就性能而言，不可变堆栈和队列与标准堆栈和队列具有相同的时间复杂度；然而，在简单的频繁更新集合的场景中，标准堆栈和队列更快。

堆栈是一种先进后出的数据结构。以下代码创建一个空的不可变堆栈，推送两个项目，枚举项目，然后弹出一个项目：

```cs
ImmutableStack<int> stack = ImmutableStack<int>.Empty;
stack = stack.Push(13);
stack = stack.Push(7);

// Displays "7" followed by "13".
foreach (int item in stack)
  Trace.WriteLine(item);

int lastItem;
stack = stack.Pop(out lastItem);
// lastItem == 7
```

请注意，在示例中我们不断重写本地变量 `stack`。不可变集合遵循一种模式，它们返回一个更新的集合；原始集合引用不会改变。这意味着一旦您获得对特定不可变集合实例的引用，它将永远不会改变。考虑以下示例：

```cs
ImmutableStack<int> stack = ImmutableStack<int>.Empty;
stack = stack.Push(13);
ImmutableStack<int> biggerStack = stack.Push(7);

// Displays "7" followed by "13".
foreach (int item in biggerStack)
  Trace.WriteLine(item);

// Only displays "13".
foreach (int item in stack)
  Trace.WriteLine(item);
```

在内部实现上，两个栈共享用于包含项`13`的内存。这种实现方式非常高效，同时可以轻松快速地获取当前状态的快照。每个不可变集合实例本身天然线程安全，但不可变集合也可以在单线程应用程序中使用。在我看来，不可变集合在代码更具功能性或需要存储大量快照且希望尽可能共享内存时尤为有用。

队列类似于栈，但是它们是先进先出的数据结构。以下代码创建了一个空的不可变队列，入队两个项，枚举这些项，然后出队一个项：

```cs
ImmutableQueue<int> queue = ImmutableQueue<int>.Empty;
queue = queue.Enqueue(13);
queue = queue.Enqueue(7);

// Displays "13" followed by "7".
foreach (int item in queue)
  Trace.WriteLine(item);

int nextItem;
queue = queue.Dequeue(out nextItem);
// Displays "13".
Trace.WriteLine(nextItem);
```

## 讨论

这个菜谱介绍了两种最简单的不可变集合，栈和队列。还涵盖了几个对*所有*不可变集合都适用的重要设计理念：

+   不可变集合的实例永远不会改变。

+   由于它永远不会改变，因此天然线程安全。

+   当您对不可变集合调用修改方法时，会返回一个新的修改后的集合。

###### 警告

即使不可变集合是线程安全的，对不可变集合的*引用*并不是线程安全的。指向不可变集合的变量需要与其他变量一样的同步保护（参见第十二章）。

不可变集合非常适合共享状态。然而，它们不适合作为通信通道。特别是不要使用不可变队列在线程间通信；生产者/消费者队列在这方面效果更佳。

###### 小贴士

`ImmutableStack<T>`和`ImmutableQueue<T>`可以在[`System.Collections.Immutable`](http://bit.ly/sys-coll-imm) NuGet 包中找到。

## 另请参阅

菜谱 9.6 介绍了线程安全（阻塞式）可变队列。

菜谱 9.7 介绍了线程安全（阻塞式）可变栈。

菜谱 9.8 介绍了支持异步的可变队列。

菜谱 9.11 介绍了支持异步的可变栈。

菜谱 9.12 介绍了阻塞/异步可变队列。

# 9.2 不可变列表

## 问题

您需要一种可以进行索引而不经常变化且可以安全地被多个线程访问的数据结构。

## 解决方案

列表是一种通用的数据结构，可以用于各种应用状态。不可变列表允许索引，但需要注意性能特征。它们并不仅仅是`List<T>`的简单替代品。

`ImmutableList<T>`支持与`List<T>`类似的方法，如以下示例所示：

```cs
ImmutableList<int> list = ImmutableList<int>.Empty;
list = list.Insert(0, 13);
list = list.Insert(0, 7);

// Displays "7" followed by "13".
foreach (int item in list)
  Trace.WriteLine(item);

list = list.RemoveAt(1);
```

不可变列表在内部组织为二叉树，以便不可变列表实例可以最大化与其他实例共享的内存量。因此，对于一些常见操作，`ImmutableList<T>`和`List<T>`之间存在性能差异（参见表 9-2）。

表 9-2\. 不可变列表的性能差异

| 操作 | List<T> | ImmutableList<T> |
| --- | --- | --- |
| 添加 | 摊销 O(1) | O(log N) |
| 插入 | O(N) | O(log N) |
| 移除在 | O(N) | O(log N) |
| 项[索引] | O(1) | O(log N) |

需要注意的是，`ImmutableList<T>`的索引操作是 O(log N)，而不是您可能期望的 O(1)。如果在现有代码中用`ImmutableList<T>`替换`List<T>`，则需要考虑如何访问集合中的项。

这意味着在可能的情况下应使用`foreach`而不是`for`。在`ImmutableList<T>`上执行的`foreach`循环的时间复杂度为 O(N)，而在相同集合上执行的`for`循环的时间复杂度为 O(N * log N)：

```cs
// The best way to iterate over an ImmutableList<T>.
foreach (var item in list)
  Trace.WriteLine(item);

// This will also work, but it will be much slower.
for (int i = 0; i != list.Count; ++i)
  Trace.WriteLine(list[i]);
```

## 讨论

`ImmutableList<T>`是一个很好的通用数据结构，但由于其性能差异，您不能盲目地用它替换所有`List<T>`的用法。`List<T>`通常是默认使用的——除非*需要*不同的集合。`ImmutableList<T>`并不是如此普遍；您需要仔细考虑其他不可变集合，并选择最适合您情况的那个。

###### 提示

`ImmutableList<T>`位于[`System.Collections.Immutable`](http://bit.ly/sys-coll-imm) NuGet 包中。

## 参见

食谱 9.1 涵盖了不可变堆栈和队列，类似于只允许访问特定元素的列表。

[MSDN 对`ImmutableList<T>.Builder`的文档](http://bit.ly/msdn-iml)介绍了一种有效的填充不可变列表的方法。

# 9.3 不可变集合

## 问题

您需要一个数据结构，不需要存储重复项，不经常更改，并且可以安全地由多个线程访问。

例如，文件中的单词索引是集合的一个好用例。

## 解决方案

有两种不可变集合类型：`ImmutableHashSet<T>`是独特项的集合，而`ImmutableSortedSet<T>`是*排序*的独特项集合。这两种类型有相似的接口：

```cs
ImmutableHashSet<int> hashSet = ImmutableHashSet<int>.Empty;
hashSet = hashSet.Add(13);
hashSet = hashSet.Add(7);

// Displays "7" and "13" in an unpredictable order.
foreach (int item in hashSet)
  Trace.WriteLine(item);

hashSet = hashSet.Remove(7);
```

只有排序集合允许像列表一样进行索引：

```cs
ImmutableSortedSet<int> sortedSet = ImmutableSortedSet<int>.Empty;
sortedSet = sortedSet.Add(13);
sortedSet = sortedSet.Add(7);

// Displays "7" followed by "13".
foreach (int item in sortedSet)
  Trace.WriteLine(item);
int smallestItem = sortedSet[0];
// smallestItem == 7

sortedSet = sortedSet.Remove(7);
```

未排序集合和排序集合具有类似的性能（参见表 9-3）。

表 9-3\. 不可变集合的性能

| 操作 | ImmutableHashSet<T> | ImmutableSortedSet<T> |
| --- | --- | --- |
| 添加 | O(log N) | O(log N) |
| 移除 | O(log N) | O(log N) |
| 项[索引] | n/a | O(log N) |

但是，我建议您使用未排序集合，除非您知道它需要排序。许多类型仅支持基本相等性而不支持完全比较，因此未排序集合可用于比排序集合更多的类型。

关于排序集的一个重要说明是，其索引是 O(log N)，而不是 O(1)，就像 `ImmutableList<T>` 一样，它在 Recipe 9.2 中讨论过。这意味着在这种情况下应该尽可能使用 `foreach` 而不是 `for` 来处理 `ImmutableSortedSet<T>`。

## 讨论

不可变集合是有用的数据结构，但是填充大型不可变集合可能会很慢。大多数不可变集合都有特殊的构建器，可以在可变方式下快速构建它们，然后将其转换为不可变集合。对许多不可变集合而言是如此，但我发现它们对于不可变集合特别有用。

###### 小贴士

`ImmutableHashSet<T>` 和 `ImmutableSortedSet<T>` 在 NuGet [`System.Collections.Immutable`](http://bit.ly/sys-coll-imm) 包中。

## 参见

Recipe 9.7 讨论了线程安全的可变背包，它们类似于集合。

Recipe 9.11 讨论了与异步兼容的可变背包。

[MSDN documentation on `ImmutableHashSet<T>.Builder`](http://bit.ly/msdn-imh) 讨论了填充不可变哈希集的高效方式。

[MSDN documentation on `ImmutableSortedSet<T>.Builder`](http://bit.ly/msdn-ims) 讨论了填充不可变排序集的高效方式。

# 9.4 不可变字典

## 问题

您需要一个不经常更改且可以安全地被多个线程访问的键/值集合。例如，您可能希望在*查找集合*中存储参考数据，这些参考数据很少更改，但应该对不同线程可用。

## 解决方案

有两种不可变字典类型：`ImmutableDictionary<TKey, TValue>` 和 `ImmutableSortedDictionary<TKey, TValue>`。从它们的名称可以猜出，`ImmutableDictionary` 中的项没有可预测的顺序，而 `ImmutableSortedDictionary` 确保其元素已排序。

这两种集合类型都有非常相似的成员：

```cs
ImmutableDictionary<int, string> dictionary =
    ImmutableDictionary<int, string>.Empty;
dictionary = dictionary.Add(10, "Ten");
dictionary = dictionary.Add(21, "Twenty-One");
dictionary = dictionary.SetItem(10, "Diez");

// Displays "10Diez" and "21Twenty-One" in an unpredictable order.
foreach (KeyValuePair<int, string> item in dictionary)
  Trace.WriteLine(item.Key + item.Value);

string ten = dictionary[10];
// ten == "Diez"

dictionary = dictionary.Remove(21);
```

注意使用 `SetItem`。在可变字典中，您可以尝试像 `dictionary[key] = item` 这样做，但是不可变字典必须返回更新后的不可变字典，因此它们使用 `SetItem` 方法代替：

```cs
ImmutableSortedDictionary<int, string> sortedDictionary =
    ImmutableSortedDictionary<int, string>.Empty;
sortedDictionary = sortedDictionary.Add(10, "Ten");
sortedDictionary = sortedDictionary.Add(21, "Twenty-One");
sortedDictionary = sortedDictionary.SetItem(10, "Diez");

// Displays "10Diez" followed by "21Twenty-One".
foreach (KeyValuePair<int, string> item in sortedDictionary)
  Trace.WriteLine(item.Key + item.Value);

string ten = sortedDictionary[10];
// ten == "Diez"

sortedDictionary = sortedDictionary.Remove(21);
```

无序字典和有序字典有类似的性能，但我建议您使用无序字典，除非您需要元素排序（参见 Table 9-4）。总体而言，无序字典可能略快。此外，无序字典可用于任何键类型，而有序字典要求它们的键类型完全可比较。

表 9-4 不可变字典的性能

| 操作 | ImmutableDictionary<TKey, TV> | ImmutableSortedDictionary<TKey, TV> |
| --- | --- | --- |
| Add | O(log N) | O(log N) |
| SetItem | O(log N) | O(log N) |
| Item[key] | O(log N) | O(log N) |
| Remove | O(log N) | O(log N) |

## 讨论

在我的经验中，字典是处理应用程序状态时常见且有用的工具。它们可用于任何类型的键/值或查找场景。

与其他不可变集合一样，不可变字典具有用于高效构建的构建器机制，如果字典包含许多元素。例如，如果在启动时加载初始引用数据，则应使用构建器机制来构建初始的不可变字典。另一方面，如果您的引用数据在应用程序执行过程中逐渐构建，则使用常规的不可变字典 `Add` 方法可能是可接受的。

###### 提示

`ImmutableDictionary<TK, TV>` 和 `ImmutableSortedDictionary<TK, TV>` 包含在 [`System.Collections.Immutable`](http://bit.ly/sys-coll-imm) NuGet 包中。

## 参见

第 9.5 节 涵盖了线程安全的可变字典。

[MSDN 关于 `ImmutableDictionary<TK,TV>.Builder`](http://bit.ly/msdn-imd) 涵盖了填充不可变字典的高效方式。

[MSDN 关于 `ImmutableSortedDictionary<TK,TV>.Builder`](http://bit.ly/msdn-isd) 涵盖了填充不可变排序字典的高效方式。

# 9.5 线程安全的字典

## 问题

您有一个键/值集合（例如内存中的缓存），需要保持同步，尽管多个线程同时读取和写入它。

## 解决方案

.NET 框架中的 `ConcurrentDictionary<TKey, TValue>` 类型是一个真正的宝藏数据结构。它是线程安全的，使用细粒度锁和无锁技术的混合确保在绝大多数场景下快速访问。

其 API 确实需要花一点时间适应。它与标准的 `Dictionary<TKey, TValue>` 类型非常不同，因为它必须处理来自多个线程的并发访问。但是一旦您在这个示例中学会了基础知识，您会发现 `ConcurrentDictionary<TKey, TValue>` 是最有用的集合类型之一。

首先，让我们学习如何向集合写入值。要设置键的值，您可以使用 `AddOrUpdate`：

```cs
var dictionary = new ConcurrentDictionary<int, string>();
string newValue = dictionary.AddOrUpdate(0,
    key => "Zero",
    (key, oldValue) => "Zero");
```

`AddOrUpdate` 有点复杂，因为它必须根据并发字典的当前内容执行几项操作。第一个方法参数是键。第二个参数是一个委托，将键（在本例中为 `0`）转换为要添加到字典中的值（在本例中为 `"Zero"`）。仅当字典中不存在该键时才会调用此委托。第三个参数是另一个委托，将键（`0`）和旧值转换为要存储在字典中的更新值（`"Zero"`）。仅当字典中存在该键时才会调用此委托。`AddOrUpdate` 返回该键的新值（由委托之一返回的相同值）。

现在让我们来看一下真正让你感到头疼的部分：为了使并发字典正常工作，`AddOrUpdate`可能必须多次调用一个或两个委托。这种情况非常罕见，但是确实可能发生。因此，你的委托应该简单快速，不应该引起任何副作用。这意味着你的委托应该只创建值；它不应该更改应用程序中的任何其他变量。对于你传递给`ConcurrentDictionary<TKey, TValue>`方法的所有委托，都应遵循相同的原则。

还有几种其他向字典添加值的方法。其中一种捷径是只使用索引语法：

```cs
// Using the same "dictionary" as above.
// Adds (or updates) key 0 to have the value "Zero".
dictionary[0] = "Zero";
```

索引语法功能较弱；它不提供根据现有值更新值的能力。然而，语法更简单，如果你已经有了要存储在字典中的值，它可以正常工作。

查看如何读取值。这可以通过`TryGetValue`轻松完成：

```cs
// Using the same "dictionary" as above.
bool keyExists = dictionary.TryGetValue(0, out string currentValue);
```

如果在字典中找到键，则`TryGetValue`将返回`true`并设置`out`值。如果未找到键，则`TryGetValue`将返回`false`。你也可以使用索引语法来读取值，但我发现这不太有用，因为如果找不到键，它会抛出异常。请记住，并发字典有多个线程同时读取、更新、添加和删除值；在许多情况下，很难知道键是否存在，直到尝试读取它为止。

删除值与读取值一样简单：

```cs
// Using the same "dictionary" as above.
bool keyExisted = dictionary.TryRemove(0, out string removedValue);
```

`TryRemove`与`TryGetValue`几乎相同（当然），只有在字典中找到键时才会删除键/值对。

## 讨论

尽管`ConcurrentDictionary<TKey, TValue>`是线程安全的，但这并不意味着它的操作是原子的。如果多个线程同时调用`AddOrUpdate`，它们可能都会检测到键不存在，并且同时执行创建新值的委托。

我认为`ConcurrentDictionary<TKey, TValue>`非常棒，主要是因为其功能强大的`AddOrUpdate`方法。然而，并不是所有情况下它都适用。`ConcurrentDictionary<TKey, TValue>`在多线程读写共享集合时表现最佳。如果更新不频繁（如果它们更为稀少），那么`ImmutableDictionary<TKey, TValue>`可能更合适。

`ConcurrentDictionary<TKey, TValue>`最适合于共享数据的情况，多个线程共享同一集合。如果一些线程仅添加元素，而其他线程仅删除元素，则生产者/消费者集合会更好地满足你的需求。

`ConcurrentDictionary<TKey, TValue>`不是唯一的线程安全集合。BCL 还提供`ConcurrentStack<T>`、`ConcurrentQueue<T>`和`ConcurrentBag<T>`。线程安全集合通常用作生产者/消费者集合，在本章的其余部分将进行介绍。

## 参见

配方 9.4 介绍了不可变字典，如果字典的内容变化非常少，则非常理想。

# 9.6 阻塞队列

## 问题

您需要一个传输介质，用于将消息或数据从一个线程传递到另一个线程。例如，一个线程可以加载数据，并在加载时将其推送到传输介质；同时，传输介质的接收端有其他线程接收并处理数据。

## 解决方案

.NET 类型`BlockingCollection<T>`设计为这种类型的传输介质。默认情况下，`BlockingCollection<T>`是一个阻塞队列，提供先进先出的行为。

阻塞队列需要被多个线程共享，并且通常被定义为私有的只读字段：

```cs
private readonly BlockingCollection<int> _blockingQueue =
    new BlockingCollection<int>();
```

通常，一个线程*要么*向集合添加项目*要么*从集合中移除项目，但不会两者兼而有之。添加项目的线程称为*生产者线程*，移除项目的线程称为*消费者线程*。

生产者线程可以通过调用`Add`添加项目，并且当生产者线程完成时（即所有项目都已添加），可以通过调用`CompleteAdding`完成集合。这会通知集合不再添加项目，并且集合可以通知其消费者没有更多项目。

这是一个简单的生产者示例，添加两个项目，然后标记集合为完成：

```cs
_blockingQueue.Add(7);
_blockingQueue.Add(13);
_blockingQueue.CompleteAdding();
```

消费者线程通常在循环中运行，等待下一个项目然后处理它。如果将生产者代码放在单独的线程中（例如通过`Task.Run`），那么可以像这样消费这些项目：

```cs
// Displays "7" followed by "13".
foreach (int item in _blockingQueue.GetConsumingEnumerable())
  Trace.WriteLine(item);
```

如果您希望有多个消费者，可以同时从多个线程调用`GetConsumingEnumerable`。然而，每个项目只会传递给这些线程中的一个。当集合完成时，可枚举对象完成。

## 讨论

前面的示例都使用了`GetConsumingEnumerable`来作为消费者线程的一种常见情况。然而，也有一个`Take`成员允许消费者只消费单个项目而不是运行循环消费所有项目。

当您使用这样的传输介质时，需要考虑如果生产者运行得比消费者快会发生什么。如果您生成的项目比您消费它们的速度快，那么可能需要限制您的队列。

当您想要异步访问传输介质时，例如 UI 线程希望充当消费者时，阻塞队列非常适合（例如线程池线程）。配方 9.8 介绍了异步队列。

###### 提示

每当您将这样的传输介质引入到您的应用程序中时，请考虑切换到 TPL Dataflow 库。大多数情况下，使用 TPL Dataflow 比构建自己的传输介质和后台线程更简单。

`BufferBlock<T>`来自 TPL Dataflow 可以像阻塞队列一样工作，TPL Dataflow 允许构建用于处理的管道或网格。然而，在许多更简单的情况下，像`BlockingCollection<T>`这样的普通阻塞队列是适当的设计选择。

您还可以使用`AsyncEx`库的`AsyncProducerConsumerQueue<T>`，它可以像阻塞队列一样工作。

## 参见

9.7 食谱 涵盖了阻塞栈和袋，如果您需要类似的传输通道而不需要先进先出语义。

9.8 食谱 涵盖了具有异步而不是阻塞 API 的队列。

9.12 食谱 涵盖了既有异步又有阻塞 API 的队列。

9.9 食谱 涵盖了限制其项目数量的队列。

# 9.7 阻塞栈和袋

## 问题

您需要一个传输通道来从一个线程传递消息或数据到另一个线程，但不希望（或不需要）这个通道具有先进先出语义。

## 解决方案

.NET 类型`BlockingCollection<T>`默认作为阻塞队列，但也可以像任何种类的生产者/消费者集合一样工作。实际上，它是围绕实现`IProducerConsumerCollection<T>`的线程安全集合的包装器。

所以，你可以创建一个具有后进先出（栈）语义或无序（袋）语义的`BlockingCollection<T>`：

```cs
BlockingCollection<int> _blockingStack = new BlockingCollection<int>(
    new ConcurrentStack<int>());
BlockingCollection<int> _blockingBag = new BlockingCollection<int>(
    new ConcurrentBag<int>());
```

重要的是要记住，现在围绕项目排序存在竞争条件。如果让相同的生产者代码在任何消费者代码之前执行，然后在生产者代码之后执行消费者代码，则项目的顺序将完全像栈一样：

```cs
// Producer code
_blockingStack.Add(7);
_blockingStack.Add(13);
_blockingStack.CompleteAdding();

// Consumer code
// Displays "13" followed by "7".
foreach (int item in _blockingStack.GetConsumingEnumerable())
  Trace.WriteLine(item);
```

当生产者代码和消费者代码在不同的线程上（这是通常情况），消费者始终获取最近添加的项目。例如，生产者可以添加`7`，消费者可以取`7`，生产者可以添加`13`，消费者可以取`13`。消费者在返回第一个项目之前*不会*等待`CompleteAdding`的调用。

## 讨论

关于阻塞队列应用于限制内存使用的节流考虑与阻塞栈和袋相同。如果您的生产者运行得比消费者快，而且您需要限制阻塞栈/袋的内存使用，可以像 9.9 食谱中所示那样使用节流。

本篇介绍使用`GetConsumingEnumerable`作为消费者代码；这是最常见的场景。还有一个`Take`成员，允许消费者只消费单个项目，而不是运行循环消费所有项目。

如果您想异步访问共享的栈或袋而不是通过阻塞（例如，让您的 UI 线程充当消费者），请参阅 9.11 食谱。

## 参见

9.6 食谱 涵盖了阻塞队列，比阻塞栈或袋更常用。

9.11 食谱 涵盖了异步栈和袋。

# 9.8 异步队列

## 问题

你需要一种传递消息或数据的通道，以先进先出的方式从代码的一部分传递到另一部分，而不阻塞线程。

例如，一个代码片段可以正在加载数据，它在加载时将数据推送到通道中；同时，UI 线程正在接收数据并显示它。

## 解决方案

你需要的是一个具有异步 API 的队列。核心 .NET 框架中没有这样的类型，但可以从 NuGet 上找到几个选项。

第一种选项是使用 Channels。Channels 是用于异步生产者/消费者集合的现代库，非常注重高性能处理高频场景。生产者通常使用`WriteAsync`向通道写入项，在它们完成所有生产后，其中一个调用`Complete`通知通道未来不会再有更多项，例如：

```cs
Channel<int> queue = Channel.CreateUnbounded<int>();

// Producer code
ChannelWriter<int> writer = queue.Writer;
await writer.WriteAsync(7);
await writer.WriteAsync(13);
writer.Complete();

// Consumer code
// Displays "7" followed by "13".
ChannelReader<int> reader = queue.Reader;
await foreach (int value in reader.ReadAllAsync())
  Trace.WriteLine(value);
```

这种更自然的消费者代码使用了异步流；更多信息请参阅第三章。截至本文撰写时，异步流仅适用于最新的 .NET 平台；旧平台可以使用以下模式：

```cs
// Consumer code (older platforms)
// Displays "7" followed by "13".
ChannelReader<int> reader = queue.Reader;
while (await reader.WaitToReadAsync())
  while (reader.TryRead(out int value))
    Trace.WriteLine(value);
```

注意旧平台消费者代码中的双重`while`循环；这是正常的。`WaitToReadAsync`会异步等待直到有可读取的项或通道已标记为完成；当有可读取的项时返回`true`。`TryRead`会尝试读取一个项（立即和同步），如果读取到项则返回`true`。如果`TryRead`返回`false`，这可能是因为当前没有可用项，或者可能是因为通道已标记为完成且以后不会再有更多项。因此，当`TryRead`返回`false`时，内部`while`循环退出，并且消费者再次调用`WaitToReadAsync`，如果通道已标记为完成，则返回`false`。

另一种生产者/消费者队列选项是使用 TPL Dataflow 库中的`BufferBlock<T>`。`BufferBlock<T>`与通道相似。以下示例显示了如何声明`BufferBlock<T>`，生产者代码的样子以及消费者代码的样子：

```cs
var _asyncQueue = new BufferBlock<int>();

// Producer code
await _asyncQueue.SendAsync(7);
await _asyncQueue.SendAsync(13);
_asyncQueue.Complete();

// Consumer code
// Displays "7" followed by "13".
while (await _asyncQueue.OutputAvailableAsync())
  Trace.WriteLine(await _asyncQueue.ReceiveAsync());
```

示例消费者代码使用了`OutputAvailableAsync`，如果只有一个消费者则确实很有用。如果有多个消费者，则可能`OutputAvailableAsync`会对多个消费者返回`true`，即使只有一个项。如果队列已完成，则`ReceiveAsync`将抛出`InvalidOperationException`。因此，如果有多个消费者，则消费者代码通常看起来更像是以下这样：

```cs
while (true)
{
  int item;
  try
  {
    item = await _asyncQueue.ReceiveAsync();
  }
  catch (InvalidOperationException)
  {
    break;
  }
  Trace.WriteLine(item);
}
```

您还可以使用`Nito.AsyncEx` NuGet 库中的`AsyncProducerConsumerQueue<T>`类型。其 API 与`BufferBlock<T>`类似但并非完全相同：

```cs
var _asyncQueue = new AsyncProducerConsumerQueue<int>();

// Producer code
await _asyncQueue.EnqueueAsync(7);
await _asyncQueue.EnqueueAsync(13);
_asyncQueue.CompleteAdding();

// Consumer code
// Displays "7" followed by "13".
while (await _asyncQueue.OutputAvailableAsync())
  Trace.WriteLine(await _asyncQueue.DequeueAsync());
```

这个消费者代码还使用了`OutputAvailableAsync`，并且和`BufferBlock<T>`一样存在相同的问题。如果有多个消费者，消费者代码通常看起来更像是以下这样：

```cs
while (true)
{
  int item;
  try
  {
    item = await _asyncQueue.DequeueAsync();
  }
  catch (InvalidOperationException)
  {
    break;
  }
  Trace.WriteLine(item);
}
```

## 讨论

我建议在可能的情况下尽量使用通道作为异步生产者/消费者队列。除了节流外，它们还具有多个抽样选项，并且高度优化。但是，如果您的应用逻辑可以表达为通过其流动数据的“管道”，那么 TPL Dataflow 可能是一个更自然的选择。最后的选择是`AsyncProducerConsumerQueue<T>`，如果您的应用已经使用了来自`AsyncEx`的其他类型，则可能会有意义。

###### 小贴士

通道可以在[`System.Threading.Channels`](http://bit.ly/sys-thrd-chanls) NuGet 包中找到。`BufferBlock<T>` 类型在[`System.Threading.Tasks.Dataflow`](http://bit.ly/nuget-df) NuGet 包中。`AsyncProducerConsumerQueue<T>` 类型在[`Nito.AsyncEx`](http://bit.ly/nito-async) NuGet 包中。

## 参见

9.6 配方涵盖了具有阻塞语义而不是异步语义的生产者/消费者队列。

9.12 配方涵盖了既有阻塞又有异步语义的生产者/消费者队列。

9.7 配方涵盖了异步堆栈和包，如果您想要一个没有先入先出语义的类似通道。

# 9.9 节流队列

## 问题

您有一个生产者/消费者队列，而且您的生产者可能比消费者运行得更快，这将导致不必要的内存使用。您还想保留所有队列项，因此需要一种方法来限制生产者的速度。

## 解决方案

当您使用生产者/消费者队列时，除非您确信消费者总是更快，否则您确实需要考虑如果您的生产者比您的消费者跑得快会发生什么。如果您生产的速度快于消费速度，则可能需要对队列进行节流。您可以通过指定最大元素数量来节流队列。当队列“满”时，它会向生产者施加背压，阻塞它们直到队列有更多空间。

通道可以通过创建有界通道而不是无界通道来进行节流。由于通道是异步的，生产者将被异步地限制：

```cs
Channel<int> queue = Channel.CreateBounded<int>(1);
ChannelWriter<int> writer = queue.Writer;

// This Write completes immediately.
await writer.WriteAsync(7);

// This Write (asynchronously) waits for the 7 to be removed
// before it enqueues the 13.
await writer.WriteAsync(13);

writer.Complete();
```

`BufferBlock<T>` 内置支持节流，详细探讨请参阅 5.4 配方。使用数据流块时，您可以设置`BoundedCapacity`选项：

```cs
var queue = new BufferBlock<int>(
    new DataflowBlockOptions { BoundedCapacity = 1 });

// This Send completes immediately.
await queue.SendAsync(7);

// This Send (asynchronously) waits for the 7 to be removed
// before it enqueues the 13.
await queue.SendAsync(13);

queue.Complete();
```

上述代码片段中的生产者使用了异步的`SendAsync` API；同样的方法适用于同步的`Post` API。

`AsyncEx` 类型 `AsyncProducerConsumerQueue<T>` 支持节流。只需用适当的值构造队列：

```cs
var queue = new AsyncProducerConsumerQueue<int>(maxCount: 1);

// This Enqueue completes immediately.
await queue.EnqueueAsync(7);

// This Enqueue (asynchronously) waits for the 7 to be removed
// before it enqueues the 13.
await queue.EnqueueAsync(13);

queue.CompleteAdding();
```

阻塞生产者/消费者队列还支持节流。您可以使用`BlockingCollection<T>`在创建时传递适当的值来限制项目的数量：

```cs
var queue = new BlockingCollection<int>(boundedCapacity: 1);

// This Add completes immediately.
queue.Add(7);

// This Add waits for the 7 to be removed before it adds the 13.
queue.Add(13);

queue.CompleteAdding();
```

## 讨论

当生产者可能比消费者运行得更快时，节流是必要的。一个您必须考虑的场景是，如果您的应用程序运行在不同于您的硬件的环境中，生产者是否可能比消费者更快。通常需要一些节流以确保您的应用程序可以在未来的硬件和/或云实例上运行，这些硬件和实例通常比开发者的机器更受限制。

节流将对生产者施加反压力，使其放慢速度以确保消费者能够处理所有项目，而不会导致不必要的内存压力。如果您不需要处理*每个*项目，您可以选择采样而不是节流。请参见食谱 9.10 了解采样生产者/消费者队列的方法。

###### 提示

通道位于[`System.Threading.Channels`](http://bit.ly/sys-thrd-chanls) NuGet 包中。`BufferBlock<T>`类型位于[`System.Threading.Tasks.Dataflow`](http://bit.ly/nuget-df) NuGet 包中。`AsyncProducerConsumerQueue<T>`类型位于[`Nito.AsyncEx`](http://bit.ly/nito-async) NuGet 包中。

## 另请参阅

食谱 9.8 介绍了基本的异步生产者/消费者队列使用方法。

食谱 9.6 介绍了基本的同步生产者/消费者队列使用方法。

食谱 9.10 介绍了采样生产者/消费者队列，作为节流的替代方法。

# 9.10 采样队列

## 问题

您有一个生产者/消费者队列，但您的生产者可能比您的消费者运行得更快，这导致了不必要的内存使用。您不需要保留所有队列项目；您需要一种方法来筛选队列项目，以便较慢的生产者只需要处理重要的项目。

## 解决方案

通道是应用输入项采样的最简单方法。一个常见的例子是始终获取最新的*n*个项目，在队列满时丢弃最旧的项目：

```cs
Channel<int> queue = Channel.CreateBounded<int>(
    new BoundedChannelOptions(1)
    {
      FullMode = BoundedChannelFullMode.DropOldest,
    });
ChannelWriter<int> writer = queue.Writer;

// This Write completes immediately.
await writer.WriteAsync(7);

// This Write also completes immediately.
// The 7 is discarded unless a consumer has already retrieved it.
await writer.WriteAsync(13);
```

这是一种简单的方法来控制输入流，防止其淹没消费者。

还有其他`BoundedChannelFullMode`选项。例如，如果您希望保留*最旧*的项目，您可以在通道满时丢弃任何新项目：

```cs
Channel<int> queue = Channel.CreateBounded<int>(
    new BoundedChannelOptions(1)
    {
      FullMode = BoundedChannelFullMode.DropWrite,
    });
ChannelWriter<int> writer = queue.Writer;

// This Write completes immediately.
await writer.WriteAsync(7);

// This Write also completes immediately.
// The 13 is discarded unless a consumer has already retrieved the 7.
await writer.WriteAsync(13);
```

## 讨论

通道非常适合进行简单的采样。在许多情况下特别有用的选项是`BoundedChannelFullMode.DropOldest`。更复杂的采样可能需要由消费者自行完成。

如果您需要进行基于时间的采样，例如“每秒只有 10 个项目”，请使用 System.Reactive。System.Reactive 具有与时间相关的自然操作符。

###### 提示

通道位于[`System.Threading.Channels`](http://bit.ly/sys-thrd-chanls) NuGet 包中。

## 另请参阅

食谱 9.9 介绍了限制通道流量的节流功能，通过阻塞生产者而不是丢弃项目来限制通道中的项目数量。

食谱 9.8 介绍了基本的通道使用，包括生产者和消费者代码。

菜谱 6.4 介绍了使用`System.Reactive`进行节流和采样，支持基于时间的采样。

# 9.11 异步堆栈和包

## 问题

您需要一个传输管道，将消息或数据从代码的一部分传递到另一部分，但您不希望（或不需要）该传输管道具有先进先出的语义。

## 解决方案

`Nito.AsyncEx`库提供了类型`AsyncCollection<T>`，默认情况下类似于异步队列，但也可以充当任何类型的生产者/消费者集合。围绕`IProducerConsumerCollection<T>`的包装器，`AsyncCollection<T>`也是.NET `BlockingCollection<T>`的`async`等效项，该项在菜谱 9.7 中有所介绍。

`AsyncCollection<T>`支持后进先出（堆栈）或无序（包）语义，取决于您传递给其构造函数的集合类型：

```cs
var _asyncStack = new AsyncCollection<int>(
    new ConcurrentStack<int>());
var _asyncBag = new AsyncCollection<int>(
    new ConcurrentBag<int>());
```

请注意，在堆栈中项目顺序方面存在竞争条件。如果所有生产者在消费者开始之前完成，则项目的顺序类似于常规堆栈：

```cs
// Producer code
await _asyncStack.AddAsync(7);
await _asyncStack.AddAsync(13);
_asyncStack.CompleteAdding();

// Consumer code
// Displays "13" followed by "7".
while (await _asyncStack.OutputAvailableAsync())
  Trace.WriteLine(await _asyncStack.TakeAsync());
```

当生产者和消费者同时执行（这是通常情况），消费者总是会获取最近添加的项。这将导致整个集合的行为不完全像一个堆栈。当然，包集合根本没有排序。

`AsyncCollection<T>`支持节流，如果生产者可能比消费者更快地向集合中添加内容，则这是必需的。只需使用适当的值构造集合即可：

```cs
var _asyncStack = new AsyncCollection<int>(
    new ConcurrentStack<int>(), maxCount: 1);
```

现在相同的生产者代码将根据需要异步等待：

```cs
// This Add completes immediately.
await _asyncStack.AddAsync(7);

// This Add (asynchronously) waits for the 7 to be removed
// before it enqueues the 13.
await _asyncStack.AddAsync(13);

_asyncStack.CompleteAdding();
```

示例消费者代码使用了`OutputAvailableAsync`，其限制与菜谱 9.8 中描述的相同。如果有多个消费者，消费者代码通常看起来更像以下内容：

```cs
while (true)
{
  int item;
  try
  {
    item = await _asyncStack.TakeAsync();
  }
  catch (InvalidOperationException)
  {
    break;
  }
  Trace.WriteLine(item);
}
```

## 讨论

`AsyncCollection<T>`只是具有略有不同 API 的`BlockingCollection<T>`的异步等效项。

###### 提示

`AsyncCollection<T>`类型位于[`Nito.AsyncEx`](http://bit.ly/nito-async) NuGet 包中。

## 参见

菜谱 9.8 介绍了异步队列，比异步堆栈或包更为常见。

菜谱 9.7 介绍了同步（阻塞）堆栈和包。

# 9.12 阻塞/异步队列

## 问题

您需要一个传输管道，以先进先出的方式将消息或数据从代码的一部分传递到另一部分，并且需要灵活性，以将生产者端或消费者端视为同步或异步。

例如，后台线程可能正在加载数据并将其推送到传输管道中，如果传输管道太满，您希望后台线程同步阻塞。同时，UI 线程正在从传输管道接收数据，您希望 UI 线程异步从传输管道中拉取数据，以保持 UI 的响应性。

## 解决方案

在查看第 9.6 节中的阻塞队列和第 9.8 节中的异步队列后，现在我们将学习一些同时支持阻塞和异步 API 的队列类型。

第一个是 TPL Dataflow NuGet 库中的`BufferBlock<T>`和`ActionBlock<T>`。`BufferBlock<T>`可以很容易地用作异步生产者/消费者队列（详见第 9.8 节）：

```cs
var queue = new BufferBlock<int>();

// Producer code
await queue.SendAsync(7);
await queue.SendAsync(13);
queue.Complete();

// Consumer code for a single consumer
while (await queue.OutputAvailableAsync())
  Trace.WriteLine(await queue.ReceiveAsync());

// Consumer code for multiple consumers
while (true)
{
  int item;
  try
  {
    item = await queue.ReceiveAsync();
  }
  catch (InvalidOperationException)
  {
    break;
  }

  Trace.WriteLine(item);
}
```

如您在以下示例中所见，`BufferBlock<T>`也支持生产者和消费者的同步 API：

```cs
var queue = new BufferBlock<int>();

// Producer code
queue.Post(7);
queue.Post(13);
queue.Complete();

// Consumer code
while (true)
{
  int item;
  try
  {
    item = queue.Receive();
  }
  catch (InvalidOperationException)
  {
    break;
  }

  Trace.WriteLine(item);
}
```

使用`BufferBlock<T>`的消费者代码相当笨拙，因为这不是编写代码的“数据流方式”。TPL Dataflow 库包括许多可以链接在一起的块，使您能够定义反应网格。在这种情况下，可以使用`ActionBlock<T>`定义完成特定操作的生产者/消费者队列：

```cs
// Consumer code is passed to queue constructor.
ActionBlock<int> queue = new ActionBlock<int>(item => Trace.WriteLine(item));

// Asynchronous producer code
await queue.SendAsync(7);
await queue.SendAsync(13);

// Synchronous producer code
queue.Post(7);
queue.Post(13);
queue.Complete();
```

如果在您期望的平台上 TPL Dataflow 库不可用，则`Nito.AsyncEx`中也有一个`AsyncProducerConsumerQueue<T>`类型，它还支持同步和异步方法：

```cs
var queue = new AsyncProducerConsumerQueue<int>();

// Asynchronous producer code
await queue.EnqueueAsync(7);
await queue.EnqueueAsync(13);

// Synchronous producer code
queue.Enqueue(7);
queue.Enqueue(13);

queue.CompleteAdding();

// Asynchronous single consumer code
while (await queue.OutputAvailableAsync())
  Trace.WriteLine(await queue.DequeueAsync());

// Asynchronous multi-consumer code
while (true)
{
  int item;
  try
  {
    item = await queue.DequeueAsync();
  }
  catch (InvalidOperationException)
  {
    break;
  }
  Trace.WriteLine(item);
}

// Synchronous consumer code
foreach (int item in queue.GetConsumingEnumerable())
  Trace.WriteLine(item);
```

## 讨论

如果可能的话，我建议使用`BufferBlock<T>`或`ActionBlock<T>`，因为 TPL Dataflow 库经过的测试比`Nito.AsyncEx`库更加全面。然而，如果您的应用程序已经使用了`AsyncEx`库的其他类型，那么`AsyncProducerConsumerQueue<T>`可能也会很有用。

也可以间接地使用`System.Threading.Channels`进行同步操作。它们的自然 API 是异步的，但由于它们是线程安全的集合，您可以通过将生产或消费代码包装在`Task.Run`中，然后阻塞`Task.Run`返回的任务来强制它们同步工作，就像这样：

```cs
Channel<int> queue = Channel.CreateBounded<int>(10);

// Producer code
ChannelWriter<int> writer = queue.Writer;
Task.Run(async () =>
{
  await writer.WriteAsync(7);
  await writer.WriteAsync(13);
  writer.Complete();
}).GetAwaiter().GetResult();

// Consumer code
ChannelReader<int> reader = queue.Reader;
Task.Run(async () =>
{
  while (await reader.WaitToReadAsync())
    while (reader.TryRead(out int value))
      Trace.WriteLine(value);
}).GetAwaiter().GetResult();
```

TPL Dataflow 块，`AsyncProducerConsumerQueue<T>`和 Channels 都支持通过在构造过程中传递选项来进行节流。当生产者推送项目比消费者消耗它们更快时，节流是必需的，这可能会导致您的应用程序占用大量内存。

###### 小贴士

`BufferBlock<T>`和`ActionBlock<T>`类型位于[`System.Threading.Tasks.Dataflow`](http://bit.ly/nuget-df) NuGet 包中。`AsyncProducerConsumerQueue<T>`类型位于[`Nito.AsyncEx`](http://bit.ly/nito-async) NuGet 包中。Channels 位于[`System.Threading.Channels`](http://bit.ly/sys-thrd-chanls) NuGet 包中。

## 另请参阅

第 9.6 节介绍了阻塞生产者/消费者队列。

第 9.8 节介绍了异步生产者/消费者队列。

第 5.4 节介绍了数据流块的节流。
