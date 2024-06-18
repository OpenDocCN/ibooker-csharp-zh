# Appendix A. Legacy Platform Support

Many of the technologies discussed in this book have some support for older platforms as well. If you’re in the unfortunate situation where you need to support these platforms, the information in this appendix may help you determine which technologies are available. Using these technologies on older platforms isn’t ideal; and even if you get it working, bear in mind that the only long-term solution is to update the platform target for your code. This appendix is intended mainly as a historical reference and not as a recommendation; that said, maintainers of old code may find it useful.

[Table A-1](#legacy_support_platforms) summarizes the support of legacy platforms for different techniques.

Table A-1\. Legacy platform support

| Platform | async | Parallel | Reactive | Dataflow | Concurrent collections | Immutable collections |
| --- | --- | --- | --- | --- | --- | --- |
| .NET 4.5 | ✓ | ✓ | NuGet | NuGet | ✓ | NuGet |
| .NET 4.0 | NuGet | ✓ | NuGet | ✗ | ✓ | ✗ |
| Windows Phone Apps 8.1 | ✓ | ✓ | NuGet | NuGet | ✓ | NuGet |
| Windows Phone SL 8.0 | ✓ | ✗ | NuGet | NuGet | ✗ | NuGet |
| Windows Phone SL 7.1 | NuGet | ✗ | NuGet | ✗ | ✗ | ✗ |
| Silverlight 5 | NuGet | ✗ | NuGet | ✗ | ✗ | ✗ |

# Legacy Platform Support for Async

If you need `async` support on older legacy platforms, install the NuGet package for [`Microsoft.Bcl.Async`](http://bit.ly/micro-async).

###### Warning

Do not use `Microsoft.Bcl.Async` to enable `async` code on ASP.NET running on .NET 4.0! The ASP.NET pipeline was updated in .NET 4.5 to be `async`-aware, and you must use .NET 4.5 or newer for `async` ASP.NET projects. `Microsoft.Bcl.Async` is only for non-ASP.NET applications.

Table A-2\. Legacy platform support for async

| Platform | Async support |
| --- | --- |
| .NET 4.5 | ✓ |
| .NET 4.0 | NuGet: `Microsoft.Bcl.Async` |
| Windows Phone Apps 8.1 | ✓ |
| Windows Phone SL 8.0 | ✓ |
| Windows Phone 7.1 | NuGet: `Microsoft.Bcl.Async` |
| Silverlight 5 | NuGet: `Microsoft.Bcl.Async` |

When using `Microsoft.Bcl.Async`, many of the members on the modern `Task` type are on the `TaskEx` type, including `Delay`, `FromResult`, `WhenAll`, and `WhenAny`.

# Legacy Platform Support for Dataflow

To use TPL Dataflow, install the NuGet package [`System.Threading.Tasks.Dataflow`](http://bit.ly/nuget-df) into your application. The TPL Dataflow library has limited platform support for older platforms ([Table A-3](#legacy_platform_support_tpl_data)).

###### Warning

Do not use the old `Microsoft.Tpl.Dataflow` package. It is no longer maintained.

Table A-3\. Legacy platform support for TPL Dataflow

| Platform | Dataflow support |
| --- | --- |
| .NET 4.5 | NuGet: `System.Threading.Tasks.Dataflow` |
| .NET 4.0 | ✗ |
| Windows Phone Apps 8.1 | NuGet: `System.Threading.Tasks.Dataflow` |
| Windows Phone SL 8.0 | NuGet: `System.Threading.Tasks.Dataflow` |
| Windows Phone SL 7.1 | ✗ |
| Silverlight 5 | ✗ |

# Legacy Platform Support for System.Reactive

To use System.Reactive, install the NuGet package [`System.Reactive`](http://bit.ly/sys-reactive) into your application. System.Reactive historically has had wide platform support ([Table A-4](#legacy_platform_support_rx)); however, most of the legacy platforms are no longer supported:

Table A-4\. Legacy platform support for System.Reactive

| Platform | Reactive support |
| --- | --- |
| .NET 4.7.2 | NuGet: `System.Reactive` |
| .NET 4.5 | NuGet: `System.Reactive v3.x` |
| .NET 4.0 | NuGet: `Rx.Main` |
| Windows Phone Apps 8.1 | NuGet: `System.Reactive v3.x` |
| Windows Phone SL 8.0 | NuGet: `System.Reactive v3.x` |
| Windows Phone SL 7.1 | NuGet: `Rx.Main` |
| Silverlight 5 | NuGet: `Rx.Main` |

###### Warning

The old `Rx.Main` package is no longer maintained.