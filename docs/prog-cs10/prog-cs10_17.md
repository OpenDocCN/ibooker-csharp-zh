# 第十七章：异步语言特性

C# 提供了语言级别的支持来使用和实现异步方法。使用异步 API 通常是使用某些服务的最有效方式。例如，大多数 I/O 在操作系统内核中是异步处理的，因为大多数外设（如磁盘控制器或网络适配器）能够自主完成大部分工作。它们只需要 CPU 在每个操作的开始和结束时介入。

尽管操作系统提供的许多服务本质上是异步的，开发者通常选择通过同步 API 使用它们（即在工作完成前不返回）。这可能会浪费资源，因为它们会阻塞线程直到 I/O 完成。线程会带来额外开销，如果你的目标是在高并发应用程序（例如为大量用户提供服务的 Web 应用）中获得最佳性能，通常最好只使用相对较少的 OS 线程。理想情况下，你的应用程序 OS 线程数量不应该超过硬件线程数量，但只有在确保线程只在没有未完成工作时阻塞时才是最佳的。《第十六章》描述了操作系统线程和硬件线程之间的区别。在性能敏感的代码中，异步 API 很有用，因为它们不会浪费资源，不会强制线程等待 I/O 完成，而是可以在此期间启动其他有用的工作。

异步 API 的问题在于，它们使用起来可能比同步 API 复杂得多，特别是如果你需要协调多个相关操作并处理错误的话。这也是为什么在主流编程语言提供内置支持之前，开发者通常选择效率较低的同步替代方案的原因。2012 年，C# 和 Visual Basic 将这些特性从研究实验室带到了实际应用中，此后许多其他流行的语言也添加了类似的特性（尤其是 JavaScript，在 2016 年也采用了非常相似的语法）。C# 中的异步特性使得我们可以编写使用高效异步 API 的代码，同时保留了使用简单同步 API 时的大部分简洁性。

这些语言特性在一些场景中也非常有用，其中最大化吞吐量并非主要性能目标。使用客户端代码时，避免阻塞 UI 线程以保持响应性是很重要的，而异步 API 提供了一种实现方式。语言对异步代码的支持可以处理线程亲和性问题，大大简化了编写高度响应式 UI 代码的工作。

# 异步关键字：async 和 await

C# 通过两个关键字来支持异步代码：`async` 和 `await`。前者不能单独使用。你需要将 `async` 关键字放在方法的声明中，这告诉编译器你打算在方法中使用异步特性。如果没有这个关键字，你就不能使用 `await` 关键字。

这可能会显得多余——如果试图在不使用 `async` 的情况下使用 `await`，编译器会产生错误。它知道方法体何时尝试使用异步特性，为什么我们还需要显式告诉它呢？有两个原因。首先，正如你将看到的，这些特性显著改变了编译器生成的代码行为，因此对于阅读代码的人来说，清楚地指示方法异步行为是有用的。其次，`await` 在 C# 中并不总是关键字，因此开发人员曾可以自由将其用作标识符。或许微软可以设计 `await` 的语法，使其仅在非常特定的上下文中充当关键字，从而使你能够继续在所有其他情况下将其用作标识符，但是 C# 团队决定采取稍微粗粒度的方法：你不能在 `async` 方法内部将 `await` 用作标识符，但在其他任何地方它都是有效的标识符。

###### 注意

`async` 关键字不会改变方法的签名。它决定了方法如何编译，而不是如何使用。

程序的入口点是一个有趣的情况。通常，`Main` 方法要么返回 `void`，要么返回 `int`，但你也可以返回 `Task` 或 `Task<int>`。.NET 运行时不支持异步入口点，因此如果你使用这些任务返回类型之一，C# 编译器会生成一个隐藏方法作为真正的入口点，该方法调用你的异步 `Main` 方法，然后阻塞直到返回的任务完成。这使得可以将 C# 程序的 `Main` 方法设为 `async`（即使在使用这些返回类型时，编译器也会生成包装器，即使你没有将方法设为 `async`）。如果你使用 C# 10.0 的顶级语句来避免显式声明 `Main`，那么就没有地方放置 `async` 关键字或返回类型，因此这是唯一一种情况，编译器根据你是否使用 `await` 推断方法是否异步。它基于程序入口点的返回类型来确定你是否返回任何内容。

因此，`async`关键字只是声明你打算使用`await`关键字。尽管不能在不使用`async`的情况下使用`await`，但将`async`关键字应用于不使用`await`的方法不会报错。然而，这样做没有任何意义，所以如果你这样做，编译器会生成警告。示例 17-1 显示了一个相当典型的例子。它使用`HttpClient`类仅请求特定资源的头部（使用 HTTP 协议为此目的定义的标准`HEAD`动词）。然后将结果显示在 UI 控件中——这种方法是 UI 的代码后端的一部分，其中包括一个名为`headerListTextBox`的`TextBox`。

##### 示例 17-1\. 使用`async`和`await`来获取 HTTP 头

```cs
// Note: as you'll see later, async methods usually should not be void private async void FetchAndShowHeaders(string url, IHttpClientFactory cf)
{
    using (HttpClient w = cf.CreateClient())
    {
        var req = new HttpRequestMessage(HttpMethod.Head, url);
        HttpResponseMessage response =
            `await` `w``.``SendAsync``(``req``,` `HttpCompletionOption``.``ResponseHeadersRead``)``;`

        headerListTextBox.Text = response.Headers.ToString();
    }
}
```

此代码包含一个粗体显示的单个`await`表达式。你可以在可能需要一些时间来产生结果的表达式中使用`await`关键字，它表示在该操作完成之前，方法的其余部分不应执行。这听起来很像阻塞同步 API 所做的事情，但不同之处在于`await`表达式不会阻塞线程——这段代码并不完全是看上去的样子。

`HttpClient`类的`SendAsync`方法返回一个`Task<HttpResponseMessage>`，你可能会想为什么我们不直接使用其`Result`属性。正如你在第十六章中看到的，如果任务未完成，此属性将阻塞线程，直到结果可用（或任务失败，这种情况下它会抛出异常）。然而，在 UI 应用程序中这样做是很危险的：如果你尝试读取不完整任务的`Result`来阻塞 UI 线程，那么将阻止任何需要在该线程上运行的操作的进展。由于 UI 应用程序需要在 UI 线程上执行大量工作，以这种方式阻塞该线程几乎可以保证迟早会导致死锁，从而导致应用程序冻结。所以不要这样做！

尽管示例 17-1 中的`await`表达式在逻辑上类似于读取`Result`，但其工作方式截然不同。如果任务的结果不立即可用，`await`关键字并不会使线程等待，尽管其名称暗示了这一点。相反，它会导致包含的方法立即返回。你可以使用调试器来验证`FetchAndShowHeaders`立即返回。例如，如果我从示例 17-2 中显示的按钮点击事件处理程序中调用该方法，我可以在该处理程序中的`Debug.WriteLine`调用上设置断点，并在示例 17-1 中更新`headerListTextBox.Text`属性的代码处设置另一个断点。

##### 示例 17-2\. 调用异步方法

```cs
private void fetchHeadersButton_Click(object sender, RoutedEventArgs e)
{
    FetchAndShowHeaders("https://endjin.com/", this.clientFactory);
    Debug.WriteLine("Method returned");
}
```

在调试器中运行时，我发现代码在示例 17-2 的最后语句上的断点命中，然后才命中示例 17-1 最终语句上的断点。换句话说，示例 17-1 中跟随`await`表达式的部分在方法已返回给其调用者后运行。显然，编译器以某种方式安排方法的剩余部分通过回调运行，一旦异步操作完成。

###### 注意

Visual Studio 的调试器在调试异步方法时会进行一些技巧，使您能够像调试普通方法一样逐步执行它们。这通常很有帮助，但有时会掩盖执行的真实本质。我刚描述的调试步骤是人为设计的，旨在打败 Visual Studio 的聪明尝试，而是揭示实际发生的情况。

注意，示例 17-1 中的代码期望在 UI 线程上运行，因为它朝着结束修改了文本框的`Text`属性。异步 API 不一定保证在您启动工作的同一线程上通知您完成，事实上，大多数情况下都不会。尽管如此，示例 17-1 按预期工作，因此除了将方法的一半转换为回调外，`await`关键字还为我们处理了线程关联性问题。

显然，每次使用`await`关键字时，C#编译器都会对您的代码进行一些重大的修改。在较早的 C#版本中，如果您想使用此异步 API 然后更新 UI，您需要编写类似于示例 17-3 的内容。这使用了我在第十六章中展示的技术：它为`SendAsync`返回的任务设置了一个继续项，使用`TaskScheduler`确保继续项的主体在 UI 线程上运行。

##### 示例 17-3\. 手动异步编码

```cs
private void OldSchoolFetchHeaders(string url, IHttpClientFactory cf)
{
    HttpClient w = cf.CreateClient();
    var req = new HttpRequestMessage(HttpMethod.Head, url);

    var uiScheduler = TaskScheduler.FromCurrentSynchronizationContext();
    w.SendAsync(req, HttpCompletionOption.ResponseHeadersRead)
        .ContinueWith(sendTask =>
        {
            try
            {
                HttpResponseMessage response = sendTask.Result;
                headerListTextBox.Text = response.Headers.ToString();
            }
            finally
            {
                w.Dispose();
            }
        },
        uiScheduler);
}
```

这是直接使用 TPL 的一个合理方式，并且与示例 17-1 有类似的效果，尽管它不完全代表 C#编译器如何转换代码。正如我稍后将展示的，`await`使用的模式是由`Task`或`Task<T>`支持的，但不是必需的。它还生成处理早期完成（即任务在您准备等待它之前已经完成）的代码，比示例 17-3 要高效得多。但在展示编译器的具体操作之前，我想说明它为您解决的一些问题，最好的方法是展示在此语言功能出现之前可能编写的代码类型。

我当前的示例相当简单，因为它仅涉及一个异步操作，但除了我已经讨论过的两个步骤——设置某种完成回调并确保它在正确的线程上运行之外——我还不得不处理位于示例 17-1 中的`using`语句。示例 17-3 不能使用`using`关键字，因为我们希望在完成后才处理`HttpClient`对象的释放。¹ 在外部方法返回之前不久调用`Dispose`是行不通的，因为我们需要在继续运行时使用该对象，而这通常会晚一些。因此，我需要在一个方法（外部方法）中创建对象，然后在另一个方法（嵌套方法）中处理释放。而且因为我手动调用`Dispose`，所以现在需要处理异常，因此我不得不用`try`块包装我移入回调的所有代码，并在`finally`块中调用`Dispose`。（事实上，我甚至没有做完整的工作——如果`HttpRequestMessage`构造函数或检索任务调度程序的调用抛出异常，`HttpClient`将不会被处理。我只处理了 HTTP 操作本身失败的情况。）

示例 17-3 已经使用了任务调度程序来安排继续通过启动时的`SynchronizationContext`运行。这确保了回调在正确的线程上发生以更新 UI。`await`关键字可以替我们处理这些。

## 执行和同步上下文

当程序的执行达到一个`await`表达式时，如果这个操作不会立即完成，为该`await`生成的代码将确保当前的执行上下文已被捕获。（如果这不是在该方法中第一个阻塞的`await`，并且如果自上次捕获以来上下文未更改，则已经捕获。）当异步操作完成时，方法的剩余部分将通过执行上下文执行。²

正如我在第十六章中描述的那样，执行上下文处理某些需要在一个方法调用另一个方法时流动的上下文信息（即使间接调用也是如此）。但在写 UI 代码时，还有另一种上下文可能会引起我们的兴趣：同步上下文（也在第十六章中有描述）。

尽管所有的 `await` 表达式都会捕获执行上下文，但是否流动同步上下文取决于被等待的类型。如果你等待一个 `Task`，同步上下文默认也会被捕获。任务并不是你可以 `await` 的唯一东西，我将在 “等待模式” 小节中描述类型如何支持 `await`。

有时候，你可能希望避免涉及同步上下文。如果你希望从 UI 线程开始执行异步工作，但又没有特别需要保持在该线程上，通过同步上下文调度每一个继续操作就是不必要的开销。如果异步操作是一个 `Task` 或 `Task<T>`（或者等效的值类型，如 `ValueTask` 或 `ValueTask<T>`），你可以通过调用 `ConfigureAwait` 方法并传递 `false` 来声明不需要这样做。这会返回异步操作的另一种表示，如果你 `await` 这个新的表示而不是原始任务，它将忽略当前的 `Syn⁠chr⁠oni⁠zat⁠ion​Con⁠text`（如果有的话）。（没有相应的机制来选择退出执行上下文。）示例 17-4 展示了如何使用它。

##### 示例 17-4\. `ConfigureAwait`

```cs
private async void OnFetchButtonClick(object sender, RoutedEventArgs e)
{
    using (HttpClient w = this.clientFactory.CreateClient())
    using (Stream f = File.Create(fileTextBox.Text))
    {
        Task<Stream> getStreamTask = w.GetStreamAsync(urlTextBox.Text);
        `Stream` `getStream` `=` `await` `getStreamTask``.``ConfigureAwait``(``false``)``;`

        Task copyTask = getStream.CopyToAsync(f);
        `await` `copyTask``.``ConfigureAwait``(``false``)``;`
    }
}

```

这段代码是按钮的点击处理程序，因此最初在 UI 线程上运行。它从两个文本框中检索 `Text` 属性。然后启动一些异步工作——获取 URL 的内容并将数据复制到文件中。在获取这两个 `Text` 属性之后，它不再使用任何 UI 元素，因此在每个 `await` 之后不必要地涉及 UI 线程无关紧要。通过向 `ConfigureAwait` 传递 `false` 并等待它返回的值，我们告诉 TPL 可以使用任何方便的线程来通知我们完成，这在本例中很可能是线程池线程。这将使工作完成更有效率和更快速，因为它避免了在每个 `await` 之后不必要地涉及 UI 线程。

并非所有的异步 API 都返回 `Task` 或 `Task<T>`。例如，作为 UWP 的一部分引入到 Windows 的各种异步 API 返回的是 `IAsyncOperation<T>` 而不是 `Task<T>`。这是因为 UWP 不是特定于 .NET 的，它有自己独立于运行时的异步操作表示，可以从 C++ 和 JavaScript 中使用。这个接口在概念上类似于 TPL 任务，并支持 await 模式，这意味着你可以在这些 API 上使用 `await`。然而，它不提供 `Con⁠fig⁠ure​Awa⁠it`。如果你想对其中一个这些 API 做类似于 示例 17-4 的事情，可以使用 `AsTask` 扩展方法，将 `IAsyncOperation<T>` 包装为 `Task<T>`，然后在该任务上调用 `ConfigureAwait`。

###### 提示

如果你正在编写库，那么在使用`await`的任何地方都应该调用`ConfigureAwait(false)`。这是因为通过同步上下文继续可能会很昂贵，并且在某些情况下可能会引入死锁的可能性。唯一的例外是当你做一些积极需要保留同步上下文的操作，或者你确信你的库只会在不设置同步上下文的应用程序框架中使用时。 （例如，ASP.NET Core 应用程序不使用同步上下文，因此通常无论是否在这些应用程序中调用`ConfigureAwait(false)`都无关紧要。）

示例 17-1 中仅包含一个`await`表达式，即使这个表达式在经典的 TPL 编程中也相当复杂。示例 17-4 包含两个`await`表达式，如果没有`await`关键字的帮助，要实现相同的行为就需要更多的代码，因为异常可能会在第一个`await`之前、第二个`await`之后或两者之间发生，我们需要在这些情况下调用`HttpClient`和`Stream`的`Dispose`方法（以及在没有抛出异常的情况下）。然而，一旦涉及流控制，事情可能会变得更加复杂。

## 多个操作和循环

假设我想处理响应体中的数据，而不是仅仅获取标头或将 HTTP 响应体复制到文件中。如果响应体很大，检索它可能是一个需要多个缓慢步骤的操作。示例 17-5 逐步获取一个网页。

##### 示例 17-5. 多个异步操作

```cs
private async void FetchAndShowBody(string url, IHttpClientFactory cf)
{
    using (HttpClient w = cf.CreateClient())
    {
        `Stream` `body` `=` `await` `w``.``GetStreamAsync``(``url``)``;`
        using (var bodyTextReader = new StreamReader(body))
        {
            while (!bodyTextReader.EndOfStream)
            {
                `string?` `line` `=` `await` `bodyTextReader``.``ReadLineAsync``(``)``;`
                bodyTextBox.AppendText(line);
                bodyTextBox.AppendText(Environment.NewLine);
                `await` `Task``.``Delay``(``TimeSpan``.``FromMilliseconds``(``10``)``)``;`
            }
        }
    }
}
```

现在这段代码包含了三个`await`表达式。第一个发起了一个 HTTP GET 请求，该操作将在收到响应的第一部分时完成，但响应可能尚未完全接收——可能还有几兆字节的内容未到达。此代码假定内容将是文本，因此将返回的`Stream`对象封装在`StreamReader`中，该对象将字节流展示为文本[³]。然后它使用该封装对象的异步`ReadLineAsync`方法逐行读取响应中的文本。因为数据通常是分块到达的，读取第一行可能需要一些时间，但接下来几次调用该方法可能会立即完成，因为每个网络数据包通常包含多行。但是，如果代码读取速度快于网络数据到达速度，最终它将消耗完首个数据包中的所有行，然后在下一行可用之前将需要一些时间。因此，`ReadLineAsync`的调用将返回一些慢和一些立即完成的任务。第三个异步操作是对`Task.Delay`的调用。我加入了这个操作来减慢速度，以便逐步看到数据在 UI 中到达。`Task.Delay`返回一个在指定延迟后完成的`Task`，因此它提供了`Thread.Sleep`的异步等效方式（`Thread.Sleep`会阻塞调用线程，而`await Task.Delay`引入延迟但不会阻塞线程）。

###### 注意

我已经将每个`await`表达式放在单独的语句中，但这不是必需的。写成`(await t1) + (await t2)`这种形式的表达式是完全合法的（如果你愿意，可以省略括号，因为`await`比加法运算符具有更高的优先级；但我个人喜欢在这里使用括号来提供视觉上的强调）。

我不打算展示 Example 17-5 的完整的非`async`等效版本，因为它将非常庞大，但我会描述一些问题。首先，我们有一个循环，其主体包含两个`await`块。要使用`Task`和回调构建等效的内容意味着需要构建自己的循环结构，因为循环的代码最终会分散在三个方法中：开始运行循环的那个方法（这将是作为`GetStreamAsync`的连续回调的嵌套方法）以及处理`ReadLineAsync`和`Task.Delay`完成的两个回调方法。可以通过创建一个在两个地方都调用的本地方法来解决这个问题：你希望开始循环的地方以及在`Task.Delay`的继续中再次启动下一个迭代。Example 17-6 展示了这种技术，但它只说明了我们希望编译器为我们完成的某些方面；它并非 Example 17-5 的完整替代方案。

##### 示例 17-6\. 一个不完整的手动异步循环

```cs
private void IncompleteOldSchoolFetchAndShowBody(
    string url, IHttpClientFactory cf)
{
    HttpClient w = cf.CreateClient();
    var uiScheduler = TaskScheduler.FromCurrentSynchronizationContext();
    w.GetStreamAsync(url).ContinueWith(getStreamTask =>
    {
        Stream body = getStreamTask.Result;
        var bodyTextReader = new StreamReader(body);

        StartNextIteration();

        void StartNextIteration()
        {
            if (!bodyTextReader.EndOfStream)
            {
                bodyTextReader.ReadLineAsync().ContinueWith(readLineTask =>
                    {
                        string? line = readLineTask.Result;

                        bodyTextBox.AppendText(line);
                        bodyTextBox.AppendText(Environment.NewLine);

                        Task.Delay(TimeSpan.FromMilliseconds(10))
                            .ContinueWith(
                                _ => StartNextIteration(), uiScheduler);
                    },
                uiScheduler);
            }
        };
    },
        uiScheduler);
}
```

这段代码勉强能够运行，但它甚至没有尝试释放它所使用的任何资源。存在多处可能出现故障的地方，因此我们不能只是简单地放置一个 `using` 块或 `try`/`finally` 对来清理事物。即使没有额外的复杂性，这段代码也几乎无法被识别出来 —— 它并不明显地表明这是试图执行与 示例 17-5 相同基本操作的代码。在实际应用中，也许完全采用不同的方法会更容易，比如编写一个实现状态机的类来跟踪工作进度。这可能会使得编写正确操作的代码更容易，但对于阅读你的代码的人来说，理解他们看到的内容实际上只是一个循环，这并不会变得更加容易。

众所周知，许多开发人员过去更喜欢同步 API。但是，C# 允许我们编写几乎与同步等效的异步代码结构，从而在不带来痛苦的情况下获得所有异步代码的性能和响应优势。这就是 `async` 和 `await` 的主要好处。

### 消费和生成异步序列

示例 17-5 展示了一个 `while` 循环，正如你所期望的那样，在 `async` 方法中你可以自由使用其他类型的循环，比如 `for` 和 `foreach`。然而，`foreach` 可能会引入一个微妙的问题：如果你遍历的集合需要执行缓慢的操作会怎么样？对于像数组或 `HashSet<T>` 这样的集合类型，所有集合项都已经在内存中，这个问题并不会出现，但是对于 `File.ReadLines` 返回的 `IEnumerable<string>` 呢？显然，这是一个适合异步操作的明显候选，但在实践中，每次需要等待更多数据从存储中到达时，它却会阻塞您的线程。这是因为 `foreach` 期望的模式不支持异步操作。问题的核心在于 `foreach` 将调用移动到下一项的方法 —— 它期望枚举器（通常，但并非总是 `IEnumerator<T>` 的实现之一）提供一个类似于 示例 17-7 中所示的 `MoveNext` 方法的签名。

##### 示例 17-7\. 不友好的非异步 `IEnumerator.MoveNext`

```cs
bool MoveNext();
```

如果有更多的项即将到来但尚未可用，集合将别无选择，只能阻塞线程，直到数据到达为止。幸运的是，C#识别了这种模式的变体。运行时库定义了一对类型，⁴在示例 17-8（首次引入于第五章）中展示，体现了这种新模式。与同步的`IEnumerable<T>`一样，`foreach`并不严格要求这些确切的类型。任何提供相同签名成员的东西都可以工作。

##### 示例 17-8\. `IAsyncEnumerable<T>` 和 `IAsyncEnumerator<T>`

```cs
public interface IAsyncEnumerable<out T>
{
    IAsyncEnumerator<T> GetAsyncEnumerator(
        CancellationToken cancellationToken = default);
}

public interface IAsyncEnumerator<out T> : IAsyncDisposable
{
    T Current { get; }

    ValueTask<bool> MoveNextAsync();
}
```

从概念上讲，这与同步模式完全相同：异步的`foreach`会向集合对象请求一个枚举器，并将重复要求其前进到下一项，每次执行循环体时使用`Current`返回的值，直到枚举器指示没有更多项为止。主要区别在于同步的`MoveNext`已被`MoveNextAsync`取代，后者返回一个可等待的`ValueTask<T>`。（`IAsyncEnumerable<T>`接口还支持传递取消令牌。异步的`foreach`不会直接使用，但可以通过`IAsyncEnumerable<T>`的`WithCancellation`扩展方法间接使用。）

要消费实现此模式的可枚举源，您必须在`foreach`前面加上`await`关键字。C#还可以帮助您实现此模式：第五章展示了如何在*迭代器*方法中使用`yield`关键字来实现`IEnumerable<T>`，但也可以返回一个`IAs⁠ync​Enu⁠mer⁠abl⁠e<T>`。示例 17-9 展示了`IAsyncEnumerable<T>`的实现和消费示例。

##### 示例 17-9\. 消费和生成异步枚举

```cs
await foreach (string line in ReadLinesAsync(args[0]))
{
    Console.WriteLine(line);
}

static async IAsyncEnumerable<string> ReadLinesAsync(string path)
{
    using (var bodyTextReader = new StreamReader(path))
    {
        while (!bodyTextReader.EndOfStream)
        {
            string? line = await bodyTextReader.ReadLineAsync();
            if (line is not null) { yield return line; }
        }
    }
}
```

由于这种语言支持使得创建和使用`IAsyncEnumerable<T>`非常类似于使用`IEnumerable<T>`，您可能会想知道是否存在异步版本的描述在第十章中描述的各种 LINQ 运算符。与 LINQ to Objects 不同，`IAsyncEnumerable<T>`的实现不在构建到.NET 或.NET Standard 的运行库的部分中，但 Microsoft 提供了一个合适的 NuGet 包。如果您引用了`System.Linq.Async`包，通常的`using System.Linq;`声明将使得所有 LINQ 运算符在`IAsyncEnumerable<T>`表达式上可用。

当我们查看广泛实现的类型的异步等效时，我们应该看看`IAsyncDisposable`。

### 异步处理

正如第七章所述，`IDisposable`接口由需要立即执行某种清理操作的类型实现，例如关闭打开的句柄，并且语言支持使用`using`语句。但是，如果清理涉及潜在的缓慢工作，例如刷新数据到磁盘？.NET Core 3.1、.NET 和.NET Standard 2.1 为这种情况提供了`IAsyncDisposable`接口。正如示例 17-10 所示，您可以在`using`语句前面放置`await`关键字以使用异步可释放资源。（您也可以在`using`声明前面放置`await`关键字。）

##### 示例 17-10\. 使用和实现`IAsyncDisposable`

```cs
await using (DiagnosticWriter w = new(@"c:\temp\log.txt"))
{
    await w.LogAsync("Test");
}

class DiagnosticWriter : IAsyncDisposable
{
    private StreamWriter? _sw;

    public DiagnosticWriter(string path)
    {
        _sw = new StreamWriter(path);
    }

    public Task LogAsync(string message)
    {
        if (_sw is null)
        { throw new ObjectDisposedException(nameof(DiagnosticWriter)); }
        return _sw.WriteLineAsync(message);
    }

    public async ValueTask DisposeAsync()
    {
        if (_sw != null)
        {
            await LogAsync("Done");
            await _sw.DisposeAsync();
            _sw = null;
        }
    }
}
```

###### 注意

尽管`await`关键字出现在`using`语句前面，但它等待的潜在缓慢操作发生在执行离开`using`语句块时。这是不可避免的，因为`using`语句和声明有效地隐藏了对`Dispose`的调用。

示例 17-10 还展示了如何实现`IAsyncDisposable`接口。与同步的`IDisposable`接口定义了单一的`Dispose`方法不同，它的异步版本定义了一个返回`ValueTask`的`DisposeAsync`方法。这使我们能够在方法上标记为`async`。使用`await using`语句将确保`DisposeAsync`返回的任务在其块的末尾完成后才继续执行。您可能已经注意到，我们对`async`方法使用了几种不同的返回类型。迭代器在同步代码中也是特例，但是对于这些返回不同任务类型的方法，又有什么不同呢？

## 返回一个 Task

任何使用`await`的方法本身可能需要一定的运行时间，因此除了能够调用异步 API 之外，通常还希望呈现一个异步的公共界面。C#编译器允许用`async`关键字标记的方法返回表示异步工作进展的对象。您可以返回`Task`，也可以返回`Task<T>`，其中`T`是任何类型。这使调用者可以发现您的方法执行的工作状态，附加连续操作，以及如果使用`Task<T>`，获取结果的方法。或者，您可以返回值类型的等价物，`ValueTask`和`ValueTask<T>`。返回任何这些类型意味着，如果您的方法从另一个`async`方法调用，它可以使用`await`等待您的方法完成，并在适用时收集其结果。

当使用 `async` 时，几乎总是比 `void` 返回类型更可取的是返回一个任务，因为对于 `void` 返回类型，调用者无法知道你的方法何时真正完成，或者发生异常时如何处理。（异步方法可以在返回后继续运行—事实上，这正是其全部意义—因此在抛出异常时，原始调用者可能已经不在堆栈上。）通过返回任务对象，你为编译器提供了一种使异常可用，并在适用时提供结果的方法。

返回任务非常简单，几乎没有理由不这样做。要修改示例 17-5 中的方法以返回任务，我只需要进行一个简单的更改。我将返回类型改为 `Task` 而不是 `void`，如示例 17-11 所示，其余代码完全可以保持不变。

##### 示例 17-11\. 返回 `Task`

```cs
private async Task FetchAndShowBody(string url, IHttpClientFactory cf)
// ...as before
```

编译器会自动生成所需代码来生成 `Task` 对象（或 `ValueTask`，如果你将其用作返回类型），并在方法返回或抛出异常时将其设置为已完成或故障状态。`Task` 类型的返回类型是异步版本的 `void`，因为当其完成时 `Task` 不产生任何结果（这就是为什么我们不需要在此方法中添加 `return` 语句，即使它现在的返回类型是 `Task`）。如果你想要从任务中返回结果，这也很容易。将返回类型设为 `Task<T>` 或 `ValueTask<T>`，其中 `T` 是你的结果类型，然后你可以使用 `return` 关键字，就像你的方法是正常的非异步方法一样，如示例 17-12 所示。

##### 示例 17-12\. 返回 `Task<T>`

```cs
public static async Task<string?> GetServerHeader(
    string url, IHttpClientFactory cf)
{
    using (HttpClient w = cf.CreateClient())
    {
        var request = new HttpRequestMessage(HttpMethod.Head, url);
        HttpResponseMessage response = await w.SendAsync(
            request, HttpCompletionOption.ResponseHeadersRead);

        string? result = null;
        IEnumerable<string>? values;
        if (response.Headers.TryGetValues("Server", out values))
        {
            result = values.FirstOrDefault();
        }
        `return` `result``;`
    }
}
```

此方法异步获取 HTTP 头部，方式与示例 17-1 相同，但不显示结果，而是挑选第一个 `Server:` 头部的值，并将其作为此方法返回的 `Task<string?>` 的结果。（需要是可空字符串，因为可能不存在该头部。）正如你所见，`return` 语句只是返回一个 `string?`，即使方法的返回类型是 `Task<string?>`。编译器生成的代码完成任务，并安排该字符串成为结果。无论是 `Task` 还是 `Task<T>` 返回类型，生成的代码都生成类似于使用 `TaskCompletionSource<T>` 所得到的任务，如第十六章中所述。

###### 注意

就像`await`关键字可以使用符合特定模式的任何异步方法一样（稍后描述），C#在实现异步方法时也提供了同样的灵活性。你不仅限于`Task`、`Task<T>`、`ValueTask`和`ValueTask<T>`。你可以返回任何符合两个条件的类型：它必须标注有`AsyncMethodBuilder`属性，标识编译器用于管理任务进度和完成的类，并且它还必须提供一个`GetAwaiter`方法，返回一个实现了`ICriticalNotifyCompletion`接口的类型。

返回内置任务类型几乎没有什么坏处。调用者不必对其做任何事情，因此你的方法使用起来与`void`方法一样简单，但增加了一个优势，即想要的调用者可以获得一个任务。唯一的返回`void`的理由可能是某些外部约束强制你的方法具有特定的签名。例如，大多数事件处理程序要求返回类型为`void`，这就是我之前一些示例这样做的原因。但除非你被迫使用它，否则不推荐将`void`作为异步方法的返回类型。

## 将异步应用于嵌套方法

到目前为止展示的示例中，我已经将`async`关键字应用于普通方法。你也可以将它用于匿名函数（匿名方法或 Lambda 表达式）和局部函数。例如，如果你正在编写一个以编程方式创建 UI 元素的程序，你可能会发现将写成 Lambda 的事件处理程序作为异步的会很方便，就像示例 17-13 那样。

##### 示例 17-13\. 一个异步 Lambda

```cs
okButton.Click += async (s, e) =>
{
    using (HttpClient w = this.clientFactory.CreateClient())
    {
        infoTextBlock.Text = await w.GetStringAsync(uriTextBox.Text);
    }
};
```

###### 注意

这与异步委托调用无关，这是我在第九章中提到的现在已经弃用的使用线程池的技术，在匿名方法和 TPL 提供更好替代之前曾经流行过。

# 等待模式

大多数支持`await`关键字的异步 API 将返回某种 TPL 任务。然而，C#并不绝对要求这样做。它会`await`任何实现特定模式的东西。此外，虽然`Task`支持此模式，但它的工作方式使得编译器使用任务与直接使用 TPL 时略有不同——这也是我之前说过展示基于任务的异步等价于`await`基础代码并不完全代表编译器所做的原因之一。在本节中，我将展示编译器如何使用任务和支持`await`的其他类型，以更好地说明它的实际工作方式。

我将创建一个`await`模式的自定义实现来展示 C#编译器的期望。示例 17-14 展示了一个异步方法`UseCustomAsync`，它使用了这个自定义实现。它将`await`表达式的结果赋值给一个`string`，因此明显期待异步操作以`string`形式输出。它调用了一个方法`CustomAsync`，该方法返回我们模式的实现（稍后将在示例 17-15 中展示）。如您所见，这不是一个`Task<string>`。

##### 示例 17-14\. 调用自定义等待实现

```cs
static async Task UseCustomAsync()
{
    string result = await CustomAsync();
    Console.WriteLine(result);
}

public static MyAwaitableType CustomAsync()
{
    return new MyAwaitableType();
}
```

编译器期望`await`关键字的操作数是提供名为`GetAwaiter`方法的类型。这可以是普通的实例成员或者扩展方法。（因此，可以通过定义合适的扩展方法使不本能支持`await`的类型也能够使用它。）这个方法必须返回一个对象或值，称为*等待器*，它需要执行三件事情。

首先，等待器必须提供一个名为`IsCompleted`的`bool`属性。编译器生成的`await`代码使用它来判断操作是否已经完成。在不需要执行耗时操作的情况下（例如，在`Stream`上调用`ReadAsync`时可以立即使用流中已有的缓冲区数据），设置回调将是一种浪费。因此，如果`IsCompleted`属性返回`true`，`await`将避免创建不必要的委托，直接继续执行方法的剩余部分。

编译器还需要一种在工作完成后获取结果的方式，因此等待器必须有一个`GetResult`方法。其返回类型定义了操作的结果类型——它将是`await`表达式的类型。（如果没有结果，返回类型是`void`。`GetResult`仍然需要存在，因为它负责在操作失败时抛出异常。）由于示例 17-14 将`await`的结果赋值给了类型为`string`的变量，因此`MyAwaitableType`类的`GetAwaiter`返回的等待器的`GetResult`方法必须是`string`类型（或者某种隐式转换为`string`的类型）。

最后，编译器需要能够提供回调。如果`IsCompleted`返回`false`，表示操作尚未完成，`await`表达式生成的代码将创建一个委托，该委托将运行方法的其余部分。它需要能够将该委托传递给等待器。（这类似于将委托传递给任务的`ContinueWith`方法。）为此，编译器不仅需要一个方法，还需要一个接口。您需要实现`INotifyCompletion`，并且建议您在可能的情况下也实现一个可选接口，称为`ICriticalNotifyCompletion`。这两者功能类似：每个定义一个接受单个`Action`委托的方法（分别是`OnCompleted`和`UnsafeOnCompleted`），等待器在操作完成后必须调用此委托。这两个接口及其相应方法的区别在于，前者要求等待器将当前执行上下文流到目标方法，而后者则不需要。C#编译器用于构建异步方法的.NET 运行时库总是为您流动执行上下文，因此生成的代码通常在可能时调用`UnsafeOnCompleted`，以避免重复流动。 （如果编译器使用`OnCompleted`，等待器将再次流动上下文。）然而，在.NET Framework 上，您会发现安全约束可能会阻止使用`UnsafeOnCompleted`。（.NET Framework 有一个*不受信任代码*的概念。来自可能不可信来源的代码—例如从互联网下载的代码—将受到各种约束。这个概念在.NET Core 中被放弃，但各种遗留物仍然存在，例如这种异步操作的设计细节。）因为`UnsafeOnCompleted`不流动执行上下文，不受信任的代码不应允许调用它，因为这将提供一种绕过某些安全机制的方式。各种任务类型的.NET Framework 实现中提供的`UnsafeOnCompleted`标有`SecurityCriticalAttribute`，这意味着只有完全信任的代码才能调用它。我们需要`OnCompleted`，以便部分受信任的代码能够使用等待器。

示例 17-15 展示了等待器模式的最小可行实现。这个示例过于简化，因为它总是同步完成，所以它的`OnCompleted`方法什么也不做。如果你在`My​Awa⁠ita⁠ble⁠Type`的实例上使用`await`关键字，由 C#编译器生成的代码永远不会调用`OnCompleted`。`await`模式要求只有在`IsCompleted`返回`false`时才调用`OnCompleted`，而在这个例子中，`IsCompleted`始终返回`true`。这就是为什么我让`OnCompleted`抛出异常。然而，尽管这个例子过于简单，它将说明`await`的作用。

##### Example 17-15\. 过于简单的 `await` 模式实现

```cs
public class MyAwaitableType
{
    public MinimalAwaiter GetAwaiter()
    {
        return new MinimalAwaiter();
    }

    public class MinimalAwaiter : INotifyCompletion
    {
        public bool IsCompleted => true;

        public string GetResult() => "This is a result";

        public void OnCompleted(Action continuation)
        {
            throw new NotImplementedException();
        }
    }
}
```

有了这段代码，我们可以看到 示例 17-14 将会做什么。它会在 `CustomAsync` 方法返回的 `MyAwaitableType` 实例上调用 `Get​Awai⁠ter`。然后它将测试 awaiter 的 `IsCompleted` 属性，如果为 `true`（在这种情况下确实是），将立即运行方法的其余部分。编译器不知道在这种情况下 `IsCompleted` 总是 `true`，因此它生成代码来处理 `false` 的情况。这将创建一个委托，当调用时，将运行方法的其余部分，并将该委托传递给 waiter 的 `OnCompleted` 方法。（我这里没有提供 `UnsafeOnCompleted`，所以它被强制使用 `OnCompleted`。）示例 17-16 展示了完成所有这些操作的代码。

##### Example 17-16\. `await` 大致的粗略近似

```cs
static void ManualUseCustomAsync()
{
    var awaiter = CustomAsync().GetAwaiter();
    if (awaiter.IsCompleted)
    {
        TheRest(awaiter);
    }
    else
    {
        awaiter.OnCompleted(() => TheRest(awaiter));
    }
}

private static void TheRest(MyAwaitableType.MinimalAwaiter awaiter)
{
    string result = awaiter.GetResult();
    Console.WriteLine(result);
}
```

我将方法分成两部分，因为 C# 编译器避免在 `IsCompleted` 为 `true` 的情况下创建委托，而我也想做同样的事情。然而，这并不完全是 C# 编译器所做的——它还设法避免为每个 `await` 语句创建额外的方法，但这意味着它必须创建更复杂的代码。事实上，对于仅包含单个 `await` 的方法，它引入的开销比 示例 17-16 要大得多。然而，一旦 `await` 表达式的数量开始增加，复杂性就会得到回报，因为编译器不需要添加任何进一步的方法。示例 17-17 展示了接近编译器所做的事情。

##### Example 17-17\. 更接近 `await` 工作方式的稍微近似

```cs
private class ManualUseCustomAsyncState
{
    private int state;
    private MyAwaitableType.MinimalAwaiter? awaiter;

    public void MoveNext()
    {
        if (state == 0)
        {
            awaiter = CustomAsync().GetAwaiter();
            if (!awaiter.IsCompleted)
            {
                state = 1;
                awaiter.OnCompleted(MoveNext);
                return;
            }
        }
        string result = awaiter!.GetResult();
        Console.WriteLine(result);
    }
}

static void ManualUseCustomAsync()
{
    var s = new ManualUseCustomAsyncState();
    s.MoveNext();
}
```

这仍然比实际代码简单，但它展示了基本策略：编译器生成一个充当状态机的嵌套类型。它有一个字段 (`state`) 来跟踪方法到目前为止的位置，并且还包含与方法的局部变量对应的字段（在本例中仅有 `awaiter` 变量）。当异步操作不阻塞时（即其 `IsCompleted` 立即返回 `true`），方法可以继续到下一部分，但一旦遇到需要一些时间的操作，它会更新 `state` 变量以记住当前位置，然后使用相关的 awaiter 的 `OnCompleted` 方法。注意它请求在完成时调用的方法是已经运行的相同方法：`MoveNext`。无论需要执行多少次 `await` 阻塞，每次完成回调都会调用同一个方法；该类只需记住它已经进行到哪里，方法就会从那里继续。这样，即使 `await` 阻塞多少次，也永远不需要创建超过一个委托。

我不会展示真正生成的代码。它几乎无法阅读，因为包含了许多*难以言说*的标识符。（从第 3 章记得，当 C#编译器需要生成带有标识符的项目，这些标识符不能与或直接可见于我们的代码，它会创建一个在运行时被认为是合法的但在 C#中不合法的名称；这被称为*难以言说*的名称。）此外，编译器生成的代码使用了来自`System.Runtime.CompilerServices`命名空间的各种辅助类，这些类仅用于异步方法以管理诸如确定等待者支持哪些完成接口以及处理相关执行上下文流的事务。此外，如果方法返回一个任务，那么还有额外的辅助程序来创建和更新它。但是，当涉及到理解可等待类型与编译器为`await`表达式生成的代码之间关系的本质时，示例 17-17 提供了一个公正的印象。

# 错误处理

`await`关键字处理异常的方式正如你希望的那样：如果异步操作失败，异常会从消耗该操作的`await`表达式中抛出。在异常面前，异步代码可以像普通同步代码一样结构化，编译器会进行必要的工作来实现这一点。

示例 17-18 包含两个异步操作，其中一个在循环中发生。这类似于示例 17-5。它对获取的内容进行了一些不同的处理，但更重要的是，它返回了一个任务。这提供了一个错误的去处，如果其中任何操作失败。

##### 示例 17-18\. 多个潜在的故障点

```cs
private static async Task<string> FindLongestLineAsync(
    string url, IHttpClientFactory cf)
{
    using (HttpClient w = cf.CreateClient())
    {
        Stream body = await w.GetStreamAsync(url);
        using (var bodyTextReader = new StreamReader(body))
        {
            string longestLine = string.Empty;
            while (!bodyTextReader.EndOfStream)
            {
                string? line = await bodyTextReader.ReadLineAsync();
                if (line is not null && longestLine.Length > line.Length)
                {
                    longestLine = line;
                }
            }
            return longestLine;
        }
    }
}
```

异步操作可能面临挑战，因为在失败发生时，最初启动工作的方法调用很可能已经返回。在此示例中，`FindLongestLineAsync`方法通常会在执行第一个`await`表达式时返回。（如果使用了 HTTP 缓存，或者`IHttpClientFactory`返回配置为从不进行任何真实请求的虚假客户端，这个操作可能会立即成功。但通常情况下，这个操作会花费一些时间，导致方法返回。）假设此操作成功并且方法的其余部分开始运行，但在检索响应体的循环的中途，计算机失去了网络连接。这将导致由`ReadLineAsync`启动的操作之一失败。

等待操作的`await`会导致异常出现。在这个方法中没有异常处理，那接下来该怎么办？通常情况下，你期望异常会沿调用堆栈向上传播，但在堆栈上方的是什么？几乎肯定不会是最初调用它的代码——记住，一旦遇到第一个`await`，方法通常会立即返回，所以在这个阶段，我们正在由`ReadLineAsync`返回的任务的等待者回调中运行。很有可能，我们将在线程池中的某个线程上运行，并且在堆栈中直接位于我们上方的代码将是任务的等待者的一部分。这段代码不知道如何处理我们的异常。

但异常不会在堆栈上传播。当`async`方法中未处理异常时，编译器生成的代码会捕获它，并将该方法返回的任务置于故障状态（这将意味着任何等待该任务的东西现在可以继续）。如果调用`FindLongestLineAsync`的代码直接与 TPL 一起工作，则可以通过检测故障状态并检索任务的`Exception`属性来看到异常。或者，它可以调用`Wait`或获取任务的`Result`属性，在任何一种情况下，任务将抛出一个包含原始异常的`AggregateException`。但如果调用`FindLongestLineAsync`的代码在我们返回的任务上使用`await`，异常将从那里重新抛出。从调用代码的角度来看，它看起来就像异常像通常一样出现了，就像示例 17-19 所示。

##### 示例 17-19\. 处理`await`中的异常

```cs
try
{
    string longest = await FindLongestLineAsync(
        "http://192.168.22.1/", this.clientFactory);
    Console.WriteLine("Longest line: " + longest);
}
catch (HttpRequestException x)
{
    Console.WriteLine("Error fetching page: " + x.Message);
}
```

这几乎是迷惑性的简单。请记住，编译器对每个`await`周围的代码执行了大量重构，看起来像是单个方法的执行在实际中可能涉及多次调用。因此，即使是像这样的简单异常处理块（或相关结构，如`using`语句）的语义保留也是非平凡的。如果你曾试图在没有编译器帮助的情况下编写异步工作的等效错误处理，你会很感激 C#在这里为你做了多少工作。

###### 注意

`await`不会重新抛出任务的`Exception`属性提供的`AggregateException`。它会重新抛出原始异常。这使得`async`方法可以像同步代码一样处理错误。

## 验证参数

C#自动通过异步方法返回的任务报告异常的方式有一个潜在令人惊讶的方面。这意味着像示例 17-20 中的代码并不会像你期望的那样运行。

##### 示例 17-20\. 可能令人惊讶的参数验证

```cs
public async Task<string> FindLongestLineAsync(string url)
{
    ArgumentNullException.ThrowIfNull(url);
    ...
```

在`async`方法内部，编译器以相同的方式处理所有异常：不允许它们像普通方法一样传播到堆栈上，并且它们总是通过使返回的任务出现故障来报告。即使是在第一个`await`之前抛出的异常也是如此。在本例中，参数验证发生在方法执行任何其他操作之前，因此在这个阶段，我们仍然在原始调用者的线程上运行。您可能认为此代码部分抛出的参数异常会直接传播回调用者。实际上，调用者将看到一个非异常返回，生成一个处于故障状态的任务。

如果调用方法立即在返回任务上调用`await`，那么这不会有太大影响——它无论如何都会看到异常。但某些代码可能选择不立即等待，在这种情况下，它直到后来才会看到参数异常。对于简单的参数验证异常，调用者显然出现编程错误时，您可能期望代码立即抛出异常，但这段代码并没有这样做。

###### 注意

如果不能确定某个特定参数是否有效而又不进行缓慢的工作，如果您想要一个真正的异步方法，您将无法立即抛出异常。在这种情况下，您需要决定是阻塞方法直到能够验证所有参数，还是让参数异常通过返回的任务报告，而不是立即抛出。

大多数`async`方法都是这样工作的，但假设您想立即抛出这种异常（例如，因为它被调用的代码不会立即`await`结果，而您希望尽快发现问题）。通常的技术是编写一个普通方法，在调用执行实际工作的`async`方法之前验证参数，并将第二个方法设置为私有或局部方法。（顺便说一下，要执行迭代器的立即参数验证也需要类似的操作。迭代器在第五章中有描述。）示例 17-21 展示了这样一个公共包装方法以及调用实际工作方法的开头。

##### 示例 17-21\. 针对`async`方法的参数验证

```cs
public static Task<string> FindLongestLineAsync(string url)
{
    ArgumentNullException.ThrowIfNull(url);
    return FindLongestLineCore(url);

    static async Task<string> FindLongestLineCore(string url)
    {
        ...
    }
}
```

因为公共方法未标记为`async`，所以它抛出的任何异常将直接传播给调用者。但是在本地方法进行实际工作时发生的任何故障都将通过任务报告。

我选择将 `url` 参数传递给本地方法。虽然不必如此，因为本地方法可以访问其包含方法的变量。但是，依赖于这一点会导致编译器创建一个类型来保存这些局部变量，以便在方法之间共享它们。在可能的情况下，编译器会将其创建为值类型，并通过引用传递给内部类型，但是如果内部方法的作用域可能超出外部方法，则无法这样做。由于这里的本地方法是 `async` 的，所以它很可能在外部方法的栈帧不再存在后继续运行，因此这将导致编译器创建一个引用类型仅用于保存该 `url` 参数。通过传递参数，我们避免了这种情况（我已将该方法标记为 `static`，以指示这是我的意图——这意味着如果我无意中在本地方法中使用外部方法的任何内容，编译器将生成错误）。编译器可能仍然必须生成代码来创建一个对象，以在异步执行期间保存内部方法的局部变量，但至少我们避免了创建多余的对象。

## 单个和多个异常

正如第十六章所示，TPL 定义了一种报告多个错误的模型——任务的 `Exception` 属性返回一个 `AggregateException`。即使只有一个失败，你仍然必须从其包含的 `AggregateException` 中提取它。然而，如果你使用 `await` 关键字，它会为你完成这一切——正如你在示例 17-19 中看到的那样，它会从 `InnerExceptions` 中检索第一个异常并重新抛出。

当操作只能产生单个失败时，这是很方便的——它使你无需编写额外的代码来处理聚合异常并挖掘内容。（如果你正在使用由 `async` 方法返回的任务，它永远不会包含多个异常。）但是，如果你正在处理可能同时以多种方式失败的复合任务，这就会带来问题。例如，`Task.WhenAll` 接受一组任务并返回一个单个任务，该任务仅在其所有组成任务完成时才完成。如果其中一些通过失败完成，你将得到一个包含多个错误的 `AggregateException`。如果你在这样的操作中使用 `await`，它只会将第一个异常抛回给你。

常规的 TPL 机制——`Wait` 方法或 `Result` 属性——提供了完整的错误集合（通过抛出 `AggregateException` 本身而不是其第一个内部异常），但如果任务尚未完成，它们都会阻塞线程。如果你想要 `await` 的高效异步操作，它只在有事情要做时使用线程，但仍然想看到所有的错误，该怎么办？示例 17-22 展示了一种方法。

##### 示例 17-22\. 无异常抛出的等待后跟 `Wait`

```cs
static async Task CatchAll(Task[] ts)
{
    try
    {
        var t = Task.WhenAll(ts);
        await t.ContinueWith(
                    x => {},
                    TaskContinuationOptions.ExecuteSynchronously);
        t.Wait();
    }
    catch (AggregateException all)
    {
        Console.WriteLine(all);
    }
}
```

这使用`await`利用了异步 C# 方法的高效特性，但与其在复合任务本身上调用`await`不同，它设置了一个延续。一个延续可以在其前置任务完成时成功完成，无论前置任务是成功还是失败。这个延续没有任何实质内容，因此这里不会出错，这意味着`await`不会抛出异常。如果有任何失败，调用`Wait`将抛出`AggregateException`，使得`catch`块能够看到所有的异常。并且因为我们只在`await`完成后调用`Wait`，我们知道任务已经完成，所以调用不会阻塞。

其缺点之一是它设置了一个额外的任务，以便我们可以在不触发异常的情况下等待。我已配置继续同步执行，因此这将避免通过线程池调度第二个工作任务，但在资源使用上仍然存在一些不理想的浪费。更复杂但更高效的方法是通常的方式使用`await`，但编写一个异常处理程序来检查是否有其他异常，如 Example 17-23 所示。

##### Example 17-23\. 寻找额外的异常

```cs
static async Task CatchAll(Task[] ts)
{
    Task? t = null;
    try
    {
        t = Task.WhenAll(ts);
        await t;
    }
    catch (Exception first)
    {
        Console.WriteLine(first);

        if (t?.Exception?.InnerExceptions.Count > 1)
        {
            Console.WriteLine("I've found some more:");
            Console.WriteLine(t.Exception);
        }
    }
}
```

这避免了创建额外的任务，但缺点是异常处理看起来有点奇怪。

## 并发操作和未捕获的异常

使用`await`最直接的方式是按顺序执行一件事情接着一件事情，就像同步代码一样。尽管严格顺序执行工作可能不像充分利用异步代码的潜力，但它确实比同步等效方法更有效地利用了可用线程，并且在客户端 UI 代码中也很有效，在工作进行中仍然可以使 UI 线程自由响应输入。但您可能希望进一步探索。

可以同时启动多个工作任务。您可以调用异步 API，并且不立即使用`await`，而是将结果存储在变量中，然后开始另一个工作任务，然后再等待两者都完成。虽然这是一种可行的技术，可以减少操作的总执行时间，但对于不熟悉的人来说存在陷阱，如 Example 17-24 所示。

##### Example 17-24\. 如何避免运行多个并发操作

```cs
static async Task GetSeveral(IHttpClientFactory cf)
{
    using (HttpClient w = cf.CreateClient())
    {
        w.MaxResponseContentBufferSize = 2_000_000;

        Task<string> g1 = w.GetStringAsync("https://endjin.com/");
        Task<string> g2 = w.GetStringAsync("https://oreilly.com");

        // BAD!
        Console.WriteLine((await g1).Length);
        Console.WriteLine((await g2).Length);
    }
}
```

这同时从两个 URL 获取内容。启动了这两个工作后，它使用两个`await`表达式来收集每个操作的结果，并显示结果字符串的长度。如果操作成功，这将起作用，但是它对错误的处理不够完善。如果第一个操作失败，代码将永远不会执行到第二个`await`。这意味着如果第二个操作也失败，没有任何东西会查看它抛出的异常。最终，TPL 将检测到异常未被观察到，这将导致引发`UnobservedTaskException`事件。（第十六章讨论了 TPL 的未观察异常处理。）问题在于这种情况只会偶尔发生—需要两个操作快速连续失败—因此这很容易在测试中忽略掉。

你可以通过谨慎的异常处理来避免这种情况—例如，在执行第一个`await`后，你可以捕获任何出现的异常，然后再执行第二个`await`。或者，你可以使用`Task.WhenAll`来等待所有任务作为单个操作—如果有任何失败，它将产生一个带有`AggregateException`的失败任务，使你能够查看所有错误。当然，正如你在前一节中看到的那样，使用`await`处理这种多次失败是很麻烦的。但是，如果你想启动多个异步操作并同时进行，你将需要更复杂的代码来协调结果，比起顺序执行工作时所需的代码要多得多。即便如此，`await`和`async`关键字仍然使生活变得更加轻松。

# 摘要

异步操作不会阻塞调用它们的线程。这使得它们比同步 API 更高效，这一点在负载重的机器上尤为重要。它们还适用于客户端，因为它们允许你执行长时间运行的工作而不会导致 UI 失去响应。没有语言支持，异步操作可能会很难正确使用，特别是在处理多个相关操作的错误时。C#的`await`关键字使你能够以看起来像正常同步代码的风格编写异步代码。如果你想要一个单一方法来管理多个并发操作，它会变得更复杂一些，但即使你编写一个顺序执行事务的异步方法，你也将获得更有效地利用线程的好处，特别是在服务器应用程序中—它将能够支持更多同时在线的用户，因为每个单独的操作使用的资源更少—而在客户端，你将获得一个更响应的 UI 的好处。

使用`await`的方法必须标记为`async`关键字，并且通常应返回`Task`、`Task<T>`、`ValueTask`或`ValueTask<T>`之一。 (C#允许`void`返回类型，但通常仅在没有选择时才使用。) 编译器将在您的方法返回时安排此任务成功完成，或者在执行过程中任何时候失败时安排完成故障。因为`await`可以消耗任何`Task`或`Task<T>`，这使得在多个方法之间分割异步逻辑变得容易，因为高级方法可以`await`一个低级`async`方法。通常，工作最终会由某个基于任务的 API 执行，但这并非必须，因为`await`只要求一定的模式——它将接受任何表达式，您可以在其中调用`GetWaiter`方法来获取合适的类型。

¹ 这个例子有些刻意，以便我可以说明在`async`方法中如何使用`using`。通常情况下，释放从`IHttpClientFactory`获取的`HttpClient`是可选的，在直接`new`一个`HttpClient`的情况下，最好保留并重复使用它，如在“可选释放”中讨论的那样。

² 恰好，示例 17-3 也这样做了，因为 TPL 为我们捕获了执行上下文。

³ 严格来说，我应该检查 HTTP 响应头以发现编码，并使用该编码配置`StreamReader`。相反，我让它检测编码，这对于演示目的已经足够好了。

⁴ 这些在.NET Core 3.1、.NET 和.NET Standard 2.1 中可用。对于.NET Framework，您需要使用`Microsoft.Bcl.AsyncInterfaces` NuGet 包。
