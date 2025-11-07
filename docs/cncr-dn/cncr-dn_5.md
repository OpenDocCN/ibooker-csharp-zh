# C

F# 异步工作流与 .NET Task 之间的互操作性

尽管 C# 和 F# 编程语言公开的异步编程模型之间存在相似之处，但它们的互操作性并非易事。F# 程序倾向于使用比 .NET Task 更多的异步计算表达式。这两种类型相似，但它们确实存在语义差异，如第七章和第八章所示。例如，任务在其创建后立即开始，而 F# 的 `Async` 必须显式启动。

你如何实现 F# 异步计算表达式与 .NET Task 之间的互操作性？可以使用 F# 函数，如 `Async.StartAsTask<T>` 和 `Async.AwaitTask<T>`，与返回或等待 `Task` 类型的 C# 库进行互操作。

相反，没有将 F# 的 `Async` 转换为 `Task` 类型的等效方法。在 C# 中使用内置的 F# `Async.Parallel` 计算会很有帮助。在此列表中，重复第九章的内容，F# 的 `downloadMediaAsyncParallel` 函数异步并行地从 Azure Blob 存储下载图像。

列表 C.1 `Async` 并行函数，用于从 Azure Blob 存储下载图像

```
let downloadMediaAsyncParallel containerName = async {
    let storageAccount = CloudStorageAccount.Parse(azureConnection)
    let blobClient = storageAccount.CreateCloudBlobClient()
    let container = blobClient.GetContainerReference(containerName)
    let computations =
        container.ListBlobs()
        |> Seq.map(fun blobMedia -> async {
    let blobName = blobMedia.Uri.Segments.
                         [blobMedia.Uri.Segments.Length - 1]
    let blockBlob = container.GetBlockBlobReference(blobName)
    use stream = new MemoryStream()
 do! blockBlob.DownloadToStreamAsync(stream)
    let image = System.Drawing.Bitmap.FromStream(stream)
    return image })
 return! Async.**Parallel** computations }   ①   
```

`downloadMediaAsyncParallel` 的返回类型是 `Async<Image[]>`。如前所述，F# 的 `Async` 类型通常难以互操作，并从 C# 代码中作为任务（`async/await`）执行。在下面的代码片段中，C# 代码使用 `Async.Parallel` 操作符将 F# 的 `downloadMediaAsyncParallel` 函数作为 `Task` 运行：

```
var cts = new CancellationToken();
var images = await downloadMediaAsyncParallel("MyMedia").**AsTask**(cts); 
```

在 `AsTask` 扩展方法的帮助下，代码互操作性变得毫不费力。互操作性解决方案是实现一个工具 F# 模块，该模块公开了一组扩展方法，这些方法可以被其他 .NET 语言消费。

列表 C.2 用于互操作 `Task` 和异步工作流的辅助扩展方法

```
module private AsyncInterop =
    let asTask(async: Async<'T>, token: CancellationToken option) =
 let tcs = TaskCompletionSource<'T>()   ①  
 let token = defaultArg token Async.CancellationToken   ②   
 Async.StartWithContinuations(async,   ③  
           tcs.SetResult, tcs.SetException,
           tcs.SetException, token)
 tcs.Task   ④  

    let asAsync(task: Task, token: CancellationToken option) =
 Async.FromContinuations(   ⑤  
            fun (completed, caught, canceled) ->
 let token = defaultArg token Async.CancellationToken   ②  
 task.ContinueWith(new Action<Task>(fun _ ->   ⑥  
                  if task.IsFaulted then caught(task.Exception)
                  else if task.IsCanceled then 
                     canceled(new OperationCanceledException(token)|>raise)
 else completed()), token)   ⑦  
                |> ignore)

    let asAsyncT(task: Task<'T>, token: CancellationToken option) =
 Async.FromContinuations(   ⑤  
            fun (completed, caught, canceled) ->
 let token = defaultArg token Async.CancellationToken   ②  
 task.ContinueWith(new Action<Task<'T>>(fun _ ->   ⑥  
                   if task.IsFaulted then caught(task.Exception)
                   else if task.IsCanceled then  
                       canceled(OperationCanceledException(token) |> raise)
 else completed(task.Result)), token)   ⑦  
            |> ignore)
 [<Extension>]  
type AsyncInteropExtensions =
  [<Extension>]
  static member AsAsync (task: Task<'T>) = AsyncInterop.asAsyncT 
➥
 (task, None)   ⑧  

  [<Extension>]
  static member AsAsync (task: Task<'T>, token: CancellationToken) =
 AsyncInterop.asAsyncT (task, Some token)   ⑧  

  [<Extension>]
  static member AsTask (async: Async<'T>) = AsyncInterop.asTask 
➥
 (async, None)   ⑧  

  [<Extension>]
  static member AsTask (async: Async<'T>, token: CancellationToken) =
 AsyncInterop.asTask (async, Some token)   ⑧   
```

`AsyncInterop` 模块是私有的，但允许 F# 的 `Async` 和 C# 的 `Task` 之间互操作的核心函数通过 `AsyncInteropExtensions` 类型公开。属性 `Extension` 将方法升级为扩展，使其可由其他 .NET 编程语言访问。

`asTask` 方法将 F# 的 `Async` 类型转换为 `Task` 类型，并通过 `Async.StartWithContinuations` 函数启动异步操作。内部，此函数使用 `TaskCompletionSource` 返回一个 `Task` 实例，该实例维护操作的状态。当操作完成时，返回的状态可以是取消、异常，或者如果成功，则是实际的结果。

函数 `asAsync` 的目的是将 `Task` 转换为 F# 的 `Async` 类型。此函数使用 `Async.FromContinuations` 创建异步计算，该计算提供回调，该回调将执行给定的成功、异常或取消的其中一个后续操作。

所有这些函数都将一个可选的 `CancellationToken` 作为第二个参数，该参数可用于停止当前操作。如果没有提供令牌，则默认情况下将分配上下文中的 `DefaultCancellationToken`。

这些函数提供了 .NET TPL 的基于任务的异步模式 (TAP) 与 F# 异步编程模型之间的互操作性。
