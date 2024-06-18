# Chapter 7\. Using Source Generators

In this chapter, we’ll explore how the .NET developer platform enables you to use C# source generators for your Blazor apps. This is a compelling feature because it makes for a great developer experience and alleviates the concerns of writing repetitive code, allowing you to focus on more interesting problems. In fact, you can use a source generator to take advantage of JavaScript APIs without needing to write any JavaScript interop code yourself. We’ll cover how a well-defined JavaScript API can be used to generate code using an example source generator.

# What Are Source Generators?

A *source generator* is a component that C# developers can write that allows you to do two things:

1.  Retrieve a compilation object that represents all user code that is being compiled

2.  Generate C# source files that can be added to a compilation object during compilation

Essentially, you can write source generator code so it generates more code. Why would you do this? As a developer, you might notice that you’re writing the same code over and over again. Or, maybe you write a lot of boilerplate code or repetitive programming idioms. When this happens, it’s time to consider automation and using source generators to write code on your behalf. Not only will it make your work easier, but it will help reduce human errors in your code.

This is where a C# source generator comes in. A C# source generator hooks into the C# compilation context as an analyzer and optionally emits source code that compiles within the same context. The resulting code is both a combination of user-written code and code that was automatically generated.

Let’s consider the code for JavaScript interop. Every time I have to write JavaScript interop code, I have to take the following steps:

1.  Use an API reference document to observe the target JavaScript API and infer the correct consumption of the JavaScript API

2.  Create an extension method that extends the `IJSRuntime` or `IJSInProcessJRuntime` interface to expose the JavaScript API

3.  Delegate out the interop call to the interface I’m extending, mapping parameters and return values from the JavaScript API to the C# method

4.  Use the extension method to call the JavaScript interop functionality

This gets repetitive and is thus a good candidate for writing and using a source generator. With Blazor WebAssembly, the framework-provided `IJSRuntime` is also an implementation of the `IJSInProcessRuntime` type. This interface exposes synchronous JavaScript interop methods. This is because WebAssembly has its corresponding JavaScript implementation in the same process, so things can happen synchronously. This has less overhead than using the `async ValueTask` alternatives, and it’s considered an optimization for Blazor WebAssembly apps over the Blazor Server hosting model.

Later in this chapter, you’ll learn about the [*blazorators* library](https://oreil.ly/wEeFJ), which provides a source generator that can be used to generate JavaScript interop code for Blazor apps. It also produces libraries that are the result of source generation. This source generator relies on the APIs of the C# compiler platform (Roslyn). It has a generator that implements the `Microsoft.CodeAnalysis.ISourceGenerator` interface. This interface is used by the compiler to generate source code, and we’re free to implement that how you see fit. In the next section, you’ll see an example JavaScript API that is source generated into a reusable class library.

# Building a Case for Source Generators

Many apps require some sort of persistence for the user-state. Luckily all modern browsers support storage, which is a means for persisting user-state directly in the browser. The `Blazor.LocalStorage.WebAssembly` NuGet package is created by the *blazorators* source generator. It’s a class library that exposes a set of powerful APIs, and it relies on JavaScript but doesn’t contain any JavaScript itself. It simply delegates to the browser’s `localStorage` API.

The ECMAScript standard specifies many well-known and supported Web APIs as well as DOM and Browser Object Model (BOM) APIs.

###### Tip

Blazor is responsible for exclusively managing the DOM, so it’s recommended to avoid source-generating DOM-specific JavaScript APIs. This is an important detail because having more than one bit of code working on the same API can be conflictual and lead to unusual behavior.

Let’s focus on the Web APIs, which are the APIs that are exposed to JavaScript. The term *Web API* here is not to be confused with HTTP Web APIs but instead refers to APIs that are native to JavaScript. One such API is that of [`window.localStorage`](https://oreil.ly/fJ8m0). This is one implementation of the `Storage` API. Local storage allows a website to persist data across browser sessions, and it is great for user preferences and things of that nature. The `localStorage` API doesn’t require a secure context, and the content is stored on the client browser and is visible to the user through the browser’s developer tools.

The API surface area of `window.localStorage` is described in [Table 7-1](#window_local_storage_api).

Table 7-1\. Local storage API table

| Method name | Parameters | Return type |
| --- | --- | --- |
| `clear` | none | `void` |
| `removeItem` | `DOMString keyName` | `void` |
| `getItem` | `DOMString keyName` | `DOMString &#124; null` |
| `setItem` | `DOMString keyName, DOMString keyValue` | `void` |
| `key` | `number index` | `DOMString &#124; null` |
| `length` | none | `number` |

Blazor JavaScript interop has a canonical example in the `localStorage` JavaScript API. It’s not uncommon to see various implementations of this in Blazor apps. This code becomes repetitive and can be tedious, time-consuming, and error-prone to maintain. In the next section, we’ll discuss how the *blazorators* source generator can create the appropriate JavaScript interop code for the `localStorage` API using TypeScript declarations. To expose this JavaScript API to the Razor component library or a Blazor WebAssembly app, you need a reference to the `IJSRuntime` or `IJSInProcessRuntime` implementations and delegate JavaScript interop calls on the native `localStorage` API to provide its functionality.

As explained in [“Single-Page Applications, Redefined”](ch01.html#single-page-apps-redefined), TypeScript provides a static type system for JavaScript. Types can be defined in a type declaration file. The *blazorators* source generator relies on TypeScript type declarations. For common JavaScript APIs, type declaration information is publicly available on the TypeScript GitHub repository. The type declarations are fetched and read by the source generator. The source generator parses the types from the [*lib.dom.d.ts* file](https://oreil.ly/KFsKq) and uses the custom `JSAutoInterop` attribute to generate the corresponding JavaScript code. The types in the *lib.dom.d.ts* file are stable, as changes are infrequent. The source generator is capable of converting the types from JavaScript into their corresponding C# shapes.

To help visualize this process, consider [Figure 7-1](#source-generator).

![](assets/lblz_0701.png)

###### Figure 7-1\. Source generator block diagram

The type declarations are requested from an HTTP GET call where the source generator determines what C# code to output. The `Storage` interface from the *lib.dom.d.ts* file resembles the following TypeScript code and is used to generate the corresponding C# code:

[PRE0]

The implementations of this interface will provide a read-only `length` property that returns the number of items in `Storage`. Implementations will also provide the common functionality of `clear`, `getItem`, `key`, `removeItem`, and `setItem`. The source generator parses this interface into a C# object that describes the interface. The source generator dynamically creates the `JSAutoGenericInterop` attribute. The attribute is discovered by the source generator, and it’s converted into generator options using the metadata from the attribute’s values. The source generator will recognize the desired `TypeName` and corresponding implementation from the `Implementation` value.

At compile time, when the source generator detects the `JSAutoGenericInterop` attribute, it will look up the `TypeName` and `Implementation` values. The source generator will then generate the JavaScript interop code for the `Storage` interface. The source generator parses the TypeScript declarations and has the logic to convert these methods into JavaScript interop extension methods. In the next section, I’ll show you how to implement the `localStorage` API as a reusable class library.

# C# Source Generators in Action

Now that you know how C# source generators work, I’ll show you how you can use them in your Blazor app development. While we were building a use case for source generators, we saw that TypeScript’s type declarations define APIs, and the source generator could use this information to generate the appropriate JavaScript interop code. You could choose to write your own source generator, or you could use the *blazorators* source generator.

## Source Generating the localStorage API

What if I told you that a C# source generator could be used to generate an entire library with corresponding JavaScript interop code—would you believe me? It’s true! As an example, I’ve created the [`Blazor.SourceGenerator` project](https://oreil.ly/Wymlh), which does just this. It’s a C# source generator that can be used to generate JavaScript interop code based on well-known APIs.

The [`Blazor.LocalStorage.WebAssembly` NuGet package](https://oreil.ly/Lo5vG) contains only the following code, defined in the *ILocalStorageService.cs* C# file:

[PRE1]

The `Blazor.SourceGenerator` project source generates a ton of code on your behalf. The only handwritten code in this project is the preceding 14 lines. This code designates itself into the `Microsoft.JSInterop` namespace, making all of the source-generated functionality available to any consumer who uses types from this namespace. The interface is `partial` as it keeps the user-defined code separate from the source-generated code. It uses `JSAutoGenericInteropAttribute` to specify the following metadata:

`TypeName = "Storage"`

This sets the target type name to [`Storage`](https://oreil.ly/pz6H1).

`Implementation = "window.localStorage"`

This expresses how to locate the implementation of the specified type from the globally scoped `window` object; this is the [`localStorage`](https://oreil.ly/sWMpG) implementation.

`Url`

This sets the URL for the implementation; it’s used by the source generator to automatically create code comments for the APIs it generates.

`GenericMethodDescriptors`

These descriptors are used to reason about which methods should be source generated using generic return types or generic parameters. By specifying the `"getItem"` method, its return type will be a generic `TValue` type. Likewise, specifying `"setItem:value"` will instruct the parameter with a name of `value` as a generic `TValue` type.

There is a lot of descriptive metadata that can be inferred from this decorative attribute. When compiled, the `Blazor.SourceGenerators` project will recognize this file and source generate the corresponding `localStorage` JavaScript interop extension methods on `ILocalStorageService`. The file needs to also be a `public partial interface`.

The resulting generated C# code now appears in the *ILocalStorageService.g.cs* C# file:

[PRE2]

[![1](assets/1.png)](#co_using_source_generators_CO1-1)

The `Length` method returns the `length` of the underlying array in the `local​Stor⁠age` implementation.

[![2](assets/2.png)](#co_using_source_generators_CO1-2)

The `Clear` method clears `localStorage`.

[![3](assets/3.png)](#co_using_source_generators_CO1-3)

The `GetItem` method returns the item for the corresponding `key` in the generic shape it’s expecting.

[![4](assets/4.png)](#co_using_source_generators_CO1-4)

The `Key` method returns the `key` at the corresponding `index` in `localStorage`.

[![5](assets/5.png)](#co_using_source_generators_CO1-5)

The `RemoveItem` method removes the item for the corresponding `key` in `localStorage`.

[![6](assets/6.png)](#co_using_source_generators_CO1-6)

The `SetItem` method sets the item for the corresponding `key` in `localStorage`.

Since this is a `partial interface`, the source generator will generate `ILocalStorageService`. The corresponding implementation is also source generated. Consumers of the generated code use the methods created on the `ILocalStorageService` type to access the `localStorage` API. This code is the synchronous alternative to the asynchronous code generated by the [`Blazor.LocalStorage.Server` NuGet package](https://oreil.ly/bADBx). The `Blazor.LocalStorage.WebAssembly` NuGet package is a class library that relies on the `Blazor.SourceGenerators` project. The advantages of the source generating this code are immense. With a bit of declarative handwritten C#, entire libraries can be source generated, and these libraries can be used by any Razor project or Blazor WebAssembly project.

`ILocalStorageService` will be exposed through the framework’s DI system. This interface is generated using the knowledge of the `TypeName` and `Implementation` properties. `TypeName` is the name of the type that will be exposed to the consumer of the generated code. `Implementation` is the name of the JavaScript type that will be used to implement the `ILocalStorageService` interface. This is based on the `localStorage` Web API. Here is the source-generated `LocalStorage` implementation, defined in the source-generated *LocalStorageService.g.cs* C# file:

[PRE3]

[![1](assets/1.png)](#co_using_source_generators_CO2-1)

The `Length` property returns the number of items in `localStorage`.

[![2](assets/2.png)](#co_using_source_generators_CO2-2)

The `LocalStorage` constructor takes `IJSInProcessRuntime` as a parameter.

[![3](assets/3.png)](#co_using_source_generators_CO2-3)

The `Clear` method clears `localStorage` by calling the `clear` JavaScript method.

[![4](assets/4.png)](#co_using_source_generators_CO2-4)

The `GetItem` method returns the item for the corresponding `key` in `local​Stor⁠age`.

[![5](assets/5.png)](#co_using_source_generators_CO2-5)

The `Key` method returns the `key` at the given `index` in `localStorage`.

[![6](assets/6.png)](#co_using_source_generators_CO2-6)

The `RemoveItem` method removes the item for the corresponding `key` in `localStorage`.

[![7](assets/7.png)](#co_using_source_generators_CO2-7)

The `SetItem` method sets the item for the corresponding `key` in `localStorage`.

The interface supports both generics and customizable serialization with `Json​Seria⁠lizerOptions`. `JsonSerializerOptions` are used to control how the type of `TValue` in the `GetItem` method is serialized. The `options` are optional and if not provided, the default serialization will be used.

It’s important to note that this is an `internal sealed class` and that it is an explicit implementation of the `ILocalStorageService` interface. This is done to ensure that the `LocalStorageService` implementation is not directly exposed to the consumer of the generated code but instead only to the abstraction. The functionality will be shared with the consumer through the native .NET DI mechanism, and that code is also source generated.

The implementation relies on the `IJSInProcessRuntime` type to perform JavaScript interop. From the given `TypeName` and corresponding `Implementation`, the following code is also generated:

ILocalStorageService.g.cs

The `partial interface` for the corresponding `Storage` Web API surface area

LocalStorageService.g.cs

The `internal sealed` implementation of the `ILocalStorageService` interface

LocalStorageServiceCollectionExtensions.g.cs

Extension methods to add the `ILocalStorageService` service to the DI `IServiceCollection`

The following is a source-generated *LocalStorageServiceCollectionExtensions.g.cs* C# file:

[PRE4]

[![1](assets/1.png)](#co_using_source_generators_CO3-1)

The `AddLocalStorageServices` method takes `IServiceCollection` as a parameter.

[![2](assets/2.png)](#co_using_source_generators_CO3-2)

The `AddLocalStorageServices` method returns `IServiceCollection` with the `ILocalStorageService` service added and the dependent framework-provided `IJSInProcessRuntime` as well.

This is called in the Web.Client’s `WebAssemblyHostBuilderExtensions` class to register the `ILocalStorageService` service with the DI `IServiceCollection`. Putting this all together, the `Blazor.LocalStorage.WebAssembly` NuGet package is less than 15 lines of handwritten code and the rest is generated, providing a fully functioning JavaScript interop implementation that is a DI-ready service. The service is registered as a singleton, and the `ILocalStorageService` interface is exposed to the consumer of the generated code. In the next section, I’ll explain how the source generator can be used to create an entirely different library for the `Geolocation` JavaScript API.

## Source Generating the Geolocation API

Geolocation information can be immensely useful, and it can enhance the UX of your app. For example, you could use it to tell the user the location of the nearest store, or you could use it to give the weather for the user’s location. It’s handy, but you need to ask the user for permission to share their geolocation with your app. The source generator project I introduced to you in the previous section also generates the `Blazor.Geolocation.WebAssembly` NuGet package. This package is used to access the `Geolocation` API in the browser. This API is a bit different from the `localStorage` API as it doesn’t require generics or custom serialization, but it does require bidirectional JavaScript interop, which is a great example to learn from.

The JavaScript API for the `Geolocation` API is exposed through the `window​.nav⁠iga⁠tor.geolocation` JavaScript object. The `Geolocation` API requires a secure context, meaning that the browser will natively prompt the user for permission to use the location services. The user has a choice, and if they choose “no,” this functionality cannot be used. If the user selects “allow,” the browser will then enable the use of this feature. In a secure context, the browser is required to use the HTTPS protocol. The API is defined as follows according to the TypeScript interface declaration, again found in the *lib.dom.d.ts* file:

[PRE5]

All of the types can be found in the *lib.dom.d.ts* file. The `Geolocation` definition is where things get a bit interesting. Sure, the source generator can generate this API much like was done with the local storage bits, but this time the generator needs to do a bit more work. The following types need to also be evaluated and potentially generated:

*   `PositionCallback`

*   `PositionErrorCallback`

*   `PositionOptions`

Let’s start first with the two callbacks. `PositionCallback` is a callback that is called when the `getCurrentPosition` or `watchPosition` methods are called. The callbacks are defined in TypeScript as follows:

[PRE6]

Each callback is an `interface` that defines a delegate or the method signature of the callback. The source generator also has to then understand and source generate the `GeolocationPosition` and `GeolocationPositionError` types. These types are defined in TypeScript as follows:

[PRE7]

The `GeolocationPosition` type has two properties, `coords` and `timestamp`. The `coords` property is an `interface` that defines the `GeolocationCoordinates` type. The `timestamp` property is a `DOMTimeStamp` type. The `DOMTimeStamp` type is a `number` type, and its value is the number of milliseconds elapsed since the Unix Epoch (January 1, 1970) as Coordinated Universal Time (UTC). The source generator will generate `readonly` properties for `DOMTimeStamp` types that expose a .NET `DateTime` with the UTC conversion as a convenience. The `GeolocationCoordinates` type is defined as follows:

[PRE8]

Finally, the source generator will recognize the `PositionOptions` type, which is defined in TypeScript as follows:

[PRE9]

The source generator has a lot of code to generate. Let’s look at how this is achieved. The `Blazor.Geolocation.WebAssembly` NuGet package contains two handwritten files. The first is the *IGeolocationService.cs* C# file that we’ll look at now, and the second is a JavaScript file, which we’ll see a bit later:

[PRE10]

Again, the library defines a `partial interface`. `TypeName` is set to `"Geolocation"`, which is the name of the JavaScript API. `Implementation` is set to `"window​.nav⁠iga⁠tor.geolocation"`, which is the JavaScript API that the library exposes. The `Url` is set to the URL of the JavaScript API documentation. The source generator will generate the following *IGeolocationService.g.cs* C# interface:

[PRE11]

[![1](assets/1.png)](#co_using_source_generators_CO4-1)

The `ClearWatch` method accepts a `double watchId` value, which is the value returned by the `WatchPosition` method.

[![2](assets/2.png)](#co_using_source_generators_CO4-2)

The `GetCurrentPosition` method accepts a `TComponent` component, which is the calling Razor (or Blazor) component.

[![3](assets/3.png)](#co_using_source_generators_CO4-3)

The `WatchPosition` method accepts a `TComponent` component, which is the calling Razor (or Blazor) component.

The `TComponent` parameters are used to call the `onSuccessCallbackMethodName` and `onErrorCallbackMethodName` methods. These method names need to be methods that are attributed with the `JSInvokableAttribute` attribute. The method signatures are detailed in the generated triple-slash comments. This is great for consuming these APIs, as the source generator will generate the appropriate C# method signature details based on the types it parsed from the corresponding TypeScript declaration.

The implementation of this interface is generated in the *GeolocationServices.g.cs* C# file:

[PRE12]

[![1](assets/1.png)](#co_using_source_generators_CO5-1)

The `GeolocationService` constructor accepts an `IJSInProcessRuntime` JavaScript, which is the JavaScript runtime specific to the Blazor WebAssembly execution model.

[![2](assets/2.png)](#co_using_source_generators_CO5-2)

The `IGeolocationService.ClearWatch` method accepts a `double` watchId and delegates to the `"window.navigator.geolocation.clearWatch"` JavaScript method.

[![3](assets/3.png)](#co_using_source_generators_CO5-3)

The `IGeolocationService.GetCurrentPosition` method delegates to the `"blazorators.getCurrentPosition"` JavaScript method.

[![4](assets/4.png)](#co_using_source_generators_CO5-4)

The `IGeolocationService.WatchPosition` method delegates to the `"blaz⁠ora⁠tors​.watchPosition"` JavaScript method.

The framework-provided `DotNetObjectReference` is used to create a reference to the component, which is used to invoke the callback methods. For the `GetCurrentPosition` and `WatchPosition` methods, the callback arguments are used internally within the delegated JavaScript along with the created component reference. At the time of writing, the *blazorators* source generator was not capable of generating the JavaScript code for the `"blazorators"` object. This should hypothetically be possible, but it would require more time to develop. Instead, the second handwritten file is a JavaScript file that contains a bit of corresponding functionality. Consider the *blazorators.geolocation.js* JavaScript file:

[PRE13]

[![1](assets/1.png)](#co_using_source_generators_CO6-1)

The `onSuccess` callback method is a helper method. It’s called by the `getCurrentPosition` success callback.

[![2](assets/2.png)](#co_using_source_generators_CO6-2)

The `onError` callback method is a helper method. It’s called by the `watch​Posi⁠tion` error callback.

[![3](assets/3.png)](#co_using_source_generators_CO6-3)

The `getCurrentPosition` method accepts a `DotNetObjectReference` dotnetObj, which is the reference to the component, and a `string successMethodName`, which is the name of the method to invoke on the component. The `options` parameter is a `PositionOptions` object, which contains the options for the current position request.

[![4](assets/4.png)](#co_using_source_generators_CO6-4)

The `watchPosition` method accepts a `DotNetObjectReference` dotnetObj, which is the reference to the component, and a `string successMethodName`, which is the name of the method to invoke on the component. The `options` parameter is a `PositionOptions` object, which contains the options for the current position request.

[![5](assets/5.png)](#co_using_source_generators_CO6-5)

The `blazorators` object is used to invoke the `getCurrentPosition` and `watch​Po⁠sition` methods.

The following types are all generated by the source generator:

*   `GeolocationPosition`

*   `GeolocationPositionError`

*   `GeolocationCoordinates`

*   `PositionOptions`

This means that as a developer, you’d consume the `Blazor.Geolocation.Web​Assem⁠bly` NuGet package, call the `AddGeolocationServices` extension method, and then use `IGeolocationService`. The types of these callbacks are also available. This is a huge win, and it provides a great example of bindings between JavaScript and the .NET world.

You may recall that in the `WeatherComponent` discussion in [Chapter 3](ch03.html#chapter-three) we discussed a manual JavaScript interop implementation of `geolocation`. While this is intentional for education, you could refactor the manual implementation out and instead use the `Blazor.Geolocation.WebAssembly` library.

In the next section, we’ll look at how to use the `Blazor.LocalStorage.WebAssembly` NuGet package to access the `localStorage` API in the application code.

## Example Usage of the ILocalStorageService

The `ILocalStorageService` type has its implementation source generated, so let’s see it in use. The model app for this book provides several bits of functionality that rely on the ability of the app state to be persisted beyond the user’s session—for example, the user’s preferred language and audio description settings, such as voice speed and speech synthesis voice. These values are persisted in the `localStorage` and are restored when the user revisits the site.

In [Chapter 4](ch04.html#chapter-four), we discussed `AudioDescriptionComponent` in passing. `Audio​Descrip⁠tionComponent` is a component that allows the user to configure the speech synthesis settings. When the user configures the audio description settings, `Audio​DescriptionComponent` is relying on the `AppInMemoryState` class. `AppInMemoryState` is used as a service and was discussed in [Chapter 2](ch02.html#chapter-two). It exposes a `ClientVoice​Pre⁠fer⁠ence` property that is used to persist the user’s preferred voice settings, as shown in [Figure 7-2](#audio-description-component-modal).

![](assets/lblz_0702.png)

###### Figure 7-2\. Audio description component modal

Consider the following *ClientVoicePreference.cs* record class:

[PRE14]

The `ClientVoicePreference` record has two properties, `Voice` and `VoiceSpeed`. The `Voice` property is the name of the voice that the user has selected. The `VoiceSpeed` property is the speed at which the voice is spoken. The value of this client preference is persisted in `localStorage` as a JSON `string`. For example, the following JSON `string` would represent the user’s preferred voice settings:

[PRE15]

When this value is present in `localStorage`, `AudioDescriptionComponent` will use it to initialize the `ClientVoicePreference` property of `AppInMemoryState`. Consider a trimmed-down version of the *AppInMemoryState.cs* class, focusing on the `Client​Voi⁠cePreference` property:

[PRE16]

[![1](assets/1.png)](#co_using_source_generators_CO7-1)

The `ILocalStorageService` type is injected into the `AppInMemoryState` class.

[![2](assets/2.png)](#co_using_source_generators_CO7-2)

The `ClientVoicePreference` property is read from `_localStorage` if it’s not already present in the `AppInMemoryState` instance.

The class exposes a `ClientVoicePreference` property that is used to persist the user’s preferred voice settings. The `ClientVoicePreference` property is read from `AudioDescriptionComponent` to initialize itself.

With knowledge of user-persisted preferences, let’s look now at `AudioDescriptionComponent`, which allows the user to configure the speech synthesis settings. Consider the following *AudioDescriptionComponent.cs* C# class:

[PRE17]

[![1](assets/1.png)](#co_using_source_generators_CO8-1)

The `_voiceSpeeds` property is an array of doubles that is used to populate the Voice Speed slider.

[![2](assets/2.png)](#co_using_source_generators_CO8-2)

The `_voice` and `_voiceSpeed` fields are assigned from `AppState.ClientVoicePreference`, which comes from `localStorage`.

[![3](assets/3.png)](#co_using_source_generators_CO8-3)

The available voices are retrieved from the callback registered in the `JavaScript.GetClientVoicesAsync` call.

[![4](assets/4.png)](#co_using_source_generators_CO8-4)

The `ClientVoicePreference` property is written to `localStorage` when it’s changed.

[![5](assets/5.png)](#co_using_source_generators_CO8-5)

The `AudioDescriptionDetails` struct is a `readonly record` type that is used to initialize the `AudioDescriptionComponent`’s `_details` field.

`AudioDescriptionComponent` represents various bits of functionality that rely on the ability of the app state to be persisted beyond the user’s session. This is an important detail, as it differs from session-based storage. There are two implementations of the JavaScript `Storage` interface: `localStorage` and `sessionStorage`. The session storage implementation is for only a single tab life. When the tab is closed, the session’s storage is gone forever, including the user’s preferred language and audio description settings, such as voice speed and speech synthesis voice. These values are persisted in `localStorage` and are restored when the user revisits the site. Let’s look at the markup of the *AudioDescriptionComponent.razor* file:

[PRE18]

[![1](assets/1.png)](#co_using_source_generators_CO9-1)

`AudioDescriptionComponent` uses the `LocalizableComponentBase` class to provide localization support.

[![2](assets/2.png)](#co_using_source_generators_CO9-2)

The majority of the markup is the button within the navigation bar.

[![3](assets/3.png)](#co_using_source_generators_CO9-3)

`AudioDescriptionModalComponent` is the modal that is displayed when the user clicks the audio description button.

When the user clicks the audio description button, `ShowAsync` is called and `AudioDescriptionModalComponent` is displayed. `AudioDescriptionModalComponent` is a simple modal that allows the user to configure the speech synthesis settings. A reference to `AudioDescriptionModalComponent` is stored in the `_modal` field using the `@ref` attribute. The `_details` field is initialized with the current values from `AppState.ClientVoicePreference` and passed to `AudioDescriptionModalComponent`. `AudioDescriptionModalComponent` exposes an `OnDetailsSaved` event that is handled by the `AudioDescriptionComponent`’s `OnDetailsSaved` method.

Let’s now look at the *AudioDescriptionModalComponent.cs* C# class:

[PRE19]

[![1](assets/1.png)](#co_using_source_generators_CO10-1)

The `Details` property is a lightweight `readonly record struct` type.

[![2](assets/2.png)](#co_using_source_generators_CO10-2)

The `OnDetailsSaved` event is an `EventCallback` that is invoked when the user clicks the `Confirm` button.

[![3](assets/3.png)](#co_using_source_generators_CO10-3)

The `_voice` field is assigned from the `Details` property when the component’s parameters are set.

[![4](assets/4.png)](#co_using_source_generators_CO10-4)

The `VoiceSpeed` property is updated when the user changes the value in the slider.

[![5](assets/5.png)](#co_using_source_generators_CO10-5)

The `ConfirmAsync` method is invoked when the user clicks the `Confirm` button.

`AudioDescriptionModalComponent` depends on the user’s preferred `ClientVoice​Pre⁠ference` to be persisted. This is a very important detail because it differs from session-based storage. There are two implementations of the JavaScript `Storage` interface: `localStorage` and `sessionStorage`. The app is concerned only with `localStorage` data persistence. Finally, we’re looking at the `AudioDescriptionModal​Compo⁠nent` Razor markup defined in the *AudioDescriptionModal​Compo⁠nent.razor* file:

[PRE20]

[![1](assets/1.png)](#co_using_source_generators_CO11-1)

`ModalComponent` is a reusable component that is used to display a modal.

[![2](assets/2.png)](#co_using_source_generators_CO11-2)

The `form` element is used to provide a form with a slider and a dropdown. The slider is used to control the voice speed. The dropdown is used to select the voice.

[![3](assets/3.png)](#co_using_source_generators_CO11-3)

The `input` is a `range` type element used to control the voice speed.

[![4](assets/4.png)](#co_using_source_generators_CO11-4)

The `datalist` element is used to provide a list of voice speeds.

[![5](assets/5.png)](#co_using_source_generators_CO11-5)

The `select` element is used to select the voice.

[![6](assets/6.png)](#co_using_source_generators_CO11-6)

The `option` element is used to provide a list of voices from all the `Audio​Descrip⁠tionDetails.Voices` available.

[![7](assets/7.png)](#co_using_source_generators_CO11-7)

The `"Okay"` `button` element will call `ConfirmAsync` when the user clicks it.

This `form` is an example of how to use Blazor for two-way binding without using `EditContext`. The `@bind` attribute is used to bind the `_voice` field to the `Details` property. The `@onchange` attribute is used to update the `Details` property when the user changes the value in the slider or when the user changes the value in the dropdown. When the user alters these values and closes `_modal`, the `ILocalStorageService` implementation will be used to persist the user’s preferred `ClientVoicePreference` value. In the next chapter, we’re going to cover advanced form techniques that use `EditContext` to provide two-way binding.

# Summary

In this chapter, you learned why source generators are so useful when developing Blazor apps. Source generators save you development time and can help to reduce human error that is inherent with handwritten code. You were introduced to the possibilities of the source generating entire consumable libraries of JavaScript interop functionality. Using the *blazorators* source generator project as an example, I showed you how to consume the `Blazor.LocalStorage.WebAssembly` NuGet package.

In the next chapter, I’m going to teach you how to do Blazor forms. I’ll demonstrate to you how to validate user input and how to customize the UX. You’ll learn how to use the framework-provided `EditForm` component.