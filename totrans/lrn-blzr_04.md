# Chapter 3\. ​​Componentizing

Our app has lifted off and taken flight—hooray! We’re going to continue our adventure of learning Blazor by scrutinizing code. In this chapter, you’ll learn how to author Blazor components and various data-binding approaches. Now that you’re familiar with how the app starts, we’ll evaluate the default route of the app. This just so happens to serve the *Index.razor* file, which is the home screen for the app. You’ll learn how to limit what a user has access to by protecting components with declarative attributes and security-semantic hierarchies. You’ll see native JavaScript `geolocation` services in use with JavaScript interop. As part of this chapter, you’ll also learn about some of the peripheral services and supporting architecture that the Blazor app relies on, such as the “Have I Been Pwned” service and Open Weather Map APIs.

# Design with the User in Mind

All graphical-based applications have users, but not all applications prioritize the needs of their users. More often than not, apps use your information to drive advertisements or sell your information to other companies. These apps view *you* (the user) as a sales opportunity or a data point.

The Learning Blazor app was designed with its users in mind. As such, it authenticates the user’s identity to determine what actions the app takes (for more information, see [“Identity and Authentication”](#identity-and-authentication)).

When users log in to the app, meaning, once the web server has authenticated a user with the Azure AD B2C tenant, a JSON Web Token (JWT; or just bearer token) is returned. The app redirects to a third-party site and prompts for credentials. The UX for the model app renders as depicted in [Figure 3-1](#azure-ad-b2c-signin-screen).

![Azure Active Directory (AAD) business-to-consumer (B2C) sign-in screen](assets/lblz_0301.png)

###### Figure 3-1\. Azure AD B2C sign-in screen

The authentication token flows through peripheral services and resources as needed. For example, this token could be represented as a client browser cookie when used with the Web.Client project. Wherever this token resides, whether in the server or on the client-side app, the authenticated users’ information is represented as a collection of key/value pairs (KVPs), which are referred to as *claims*. A user is represented as a `Claim⁠s​Principal` object. `ClaimsPrincipal` has an `Identity` property, which is available at runtime with a `ClaimsIdentity` instance. When a service requires authentication and a request provides a valid authentication token, the requested claims are provided. At this time, we can demand various attributes (or claims) that a user agrees to share with our application. The log-in UX from the Blazor app is customizable, and you’ll learn more about that in [Chapter 4](ch04.html#chapter-four).

Our app uses these claims to uniquely identify an authenticated user. The claims are part of the bearer token and are passed to various services that the app relies on. The claims flow into the “Pwned” service, thus enabling an automated data-breach detection mechanism on the user’s behalf from their email.

## Leveraging “Pwned” Functionality

One of the functionalities of the Learning Blazor app is Pwned functionality, which tells the user if their email has been compromised. This functionality draws from the [“Have I Been Pwned” API](https://oreil.ly/Lzlvw) by Troy Hunt. He is one of the most renowned security experts in the world, and he’s been collecting data breaches for years. He spends time aggregating, normalizing, and persisting all of this data into a service called “Have I Been Pwned” (or HIBP for short). This service exposes the ability to check whether or not a given email address has ever existed within a data breach—at the time of writing there were nearly 11.5 billion records. This number will certainly continue to grow. The consuming components and client services of this API are detailed in [Chapter 5](ch05.html#chapter-five).

The HIBP API exposes three primary categories:

Breaches

Aggregated data breach information for security-compromised accounts

Passwords

A massive collection of hashed passwords that have appeared in data breaches, meaning they’re compromised

Pastes

Information that has been published to a publicly facing website designed to share content

The Learning Blazor application is also dependent on the [pwned-client open source project on GitHub](https://oreil.ly/KHvnn), which is a .NET HTTP client library for accessing the HIBP API programmatically with C#.

This library comes DI-ready; all the consumer needs is an API key and the [NuGet package](https://oreil.ly/X48Vq).

The pwned-client library exposes the ability for consumers to configure their API key through well-known configurations. For example, if you wanted to use an environment variable, you’d name it `HibpOptions__ApiKey`. The double underscore (`__`) is used as a cross-platform alternative to delimiting name segments with `:`, which wouldn’t work in Linux. The `HibpOptions__ApiKey` environment variable would map to the libraries’ strongly typed `HibpOptions.ApiKey` property value.

To add all of the services to the DI container (`IServiceCollection`), call one of the `AddPwnedServices` overload extension methods:

[PRE0]

This first `AddPwnedServices` overload uses an `IConfiguration _configuration` and asks for the `"HibpOptions"` section. ASP.NET Core has many configuration providers, including JSON, XML, environment variables, Azure Key Vault, and so on. The `IConfiguration` object can represent all of them. If using environment variables, for example, it would map that configuration section to the libraries’ dependent `HibpOptions`. Likewise, the JSON provider is capable of pulling in configuration from JSON files such as *appsettings.json*:

[PRE1]

[![1](assets/1.png)](#co___componentizing_CO1-1)

In this example file, the `"HibpOptions"` object would map to the `HibpOptions` type in the library.

Alternatively, you can assign the options directly with a lambda expression:

[PRE2]

This `AddPwnedServices` overload allows you to specify the API key and other options inline. After the services have been registered and the proper configurations have been set, the code can use DI for any available abstractions. There are several clients to use, each with a specific context:

`IPwnedBreachesClient`

A client to access the Breaches API

`IPwnedPasswordsClient`

A client to access the Passwords API

`IPwnedPastesClient`

A client to access the Pastes API

`IPwnedClient`

A client to access all the APIs and aggregates all other clients into a single client for convenience

If you’d like to run the sample application locally, you’ll optionally provide several API keys for various services. For example, to get the “Have I Been Pwned” API key, you can [sign up on their site](https://oreil.ly/XOKoX). This specific API key isn’t free; if you’d rather *not* sign up for the API, you can use the following API key to enable a demo mode:

[PRE3]

This could be configured in the *appsettings.json* file of the Web.Client project.

With .NET 6, minimalism-first is widely emphasized, and for good reason. The idea is to start small and allow the code to grow with your needs. Minimal APIs focus on simplicity, ease of use, extensibility, and, for lack of a better word, minimalism.

## “Have I Been Pwned” Client Services

Let’s look at the .NET 6 Minimal API project that serves as the Web.PwnedApi of the Learning Blazor app, the *Web.PwnedApi.csproj* file:

[PRE4]

[![1](assets/1.png)](#co___componentizing_CO2-1)

The project is targeting the `net.6.0` target framework moniker (TFM).

[![2](assets/2.png)](#co___componentizing_CO2-2)

There are several package references for framework-provided and third-party libraries.

[![3](assets/3.png)](#co___componentizing_CO2-3)

There are several project references for local dependencies.

The project’s root namespace is defined as `Learning.Blazor.PwnedApi`, and it targets the `net6.0` TFM. Since we’re targeting .NET 6, we can enable the `ImplicitUsings` feature; this means that by default there is a set of `usings` implicitly available in all of the project’s C# files. This is an added convenience, as these implicitly added namespaces are common. The project also defines `Nullable` as being enabled. This means that we can define nullable reference types, and the C# compiler platform (Roslyn) will provide warnings where there is the potential for `null` values, through definite assignment flow analysis.

The project adds many package references. One package of particular interest is `HaveIBeenPwned.Client`. This is the package that exposes the “Have I Been Pwned” HTTP client functionality. The project also defines authentication and identity packages, which are used to help protect the exposed APIs.

The project defines two project references, Web.Extensions and Web.Http.Extensions. These projects provide shared utilitarian functionality. The extensions project is based on the common language runtime (CLR) types, whereas the HTTP extensions project is specific to providing a shared transient fault error handling policy.

The *Program.cs* is a C# top-level program, and it looks like the following:

[PRE5]

[![1](assets/1.png)](#co___componentizing_CO3-1)

The `builder` is created and endpoints added.

[![2](assets/2.png)](#co___componentizing_CO3-2)

The `builder` is built and its endpoints are mapped, resulting in an `app` object.

[![3](assets/3.png)](#co___componentizing_CO3-3)

The `app` object is run.

The code starts by creating a `builder` instance of type `WebApplicationBuilder`, which exposes the *builder pattern* (as described in [“Builder Pattern”](ch06.html#builder-pattern)) for our web app. From the call to `CreateBuilder`, the code calls `AddPwnedEndpoints`. This is an extension method on the `WebApplicationBuilder` type that adds all the desired endpoints. `args` used to call `CreateBuilder` are implicitly available and represent the command-line args given to initiate running the application. These are available for all C# top-level programs. With the `builder`, we have access to several key members:

*   The `Services` property is our `IServiceCollection`; we can register services for DI with this.

*   The `Configuration` property is a `ConfigurationManager`, which is an implementation of `IConfiguration`.

*   The `Environment` property provides information about the hosting environment itself.

Next, `builder.Build()` is called. This returns a `WebApplication` type, and from this returned object another call is chained to `MapPwnedEndpoints`. This is yet another extension method, which encapsulates the logic for mapping the added endpoints to the `WebApplication` that it extends. The `WebApplication` type is an implementation of the `IAsyncDisposable` interface. As such, the code can asynchronously `await using` the `app` instance. This is the proper way to ensure that the `app` will be disposed of when it’s done running.

Finally, the code calls `await app.RunAsync();`. This runs the application and returns a `Task` that completes when the app is shut down.

While this Minimal API project has a `Program` file with a meager three lines of code, there is a fair amount that’s going on here. This API is exposing a very important piece of app functionality: the ability to evaluate whether a user’s email has been part of a data breach. This information is hugely helpful to users, and it needs to be properly protected. The API itself requires an authenticated user with a specific Azure AD B2C scope. Consider the *WebApplicationBuilderExtensions.cs* C# file:

[PRE6]

[![1](assets/1.png)](#co___componentizing_CO4-1)

The extension defensively checks that the `builder` is not null.

[![2](assets/2.png)](#co___componentizing_CO4-2)

The `WebClientOrigin` configuration value is extracted.

[![3](assets/3.png)](#co___componentizing_CO4-3)

`builder` is configured to use JWT bearer authentication.

[![4](assets/4.png)](#co___componentizing_CO4-4)

The JWT bearer name claim type is set to `name`.

[![5](assets/5.png)](#co___componentizing_CO4-5)

A call to `AddPwnedServices` is made, which adds the required services.

.NET 6 introduced a new API on the `ArgumentNullException` type that will `throw` if a given parameter is `null`. This API is `void` returning, so it’s not fluent, but it can still save a few lines of code.

Given the `builder.Configuration` instance, the code expects a value for the `"WebClientOrigin"` key. This is the origin of the client Blazor application, and it’s used to configure cross-origin resource sharing, or, simply, CORS. CORS is a policy that enables different origins to share resources, i.e., one origin can request resources from another. By default, browsers enforce the “same-origin” policy as a standard to ensure the browser can make API calls to a different origin. Because the Pwned API is hosted on a different origin than the Blazor client application, it must configure CORS and specify the allowable client origins.

The Azure AD B2C tenant is configured. The `"AzureAdB2C"` section from the *app​settings.json* file is bound, which sets the instance, client identifier, domain, scopes, and policy ID.

`JwtBearerOptions` is configured, specifying the `"name"` claim as the name claim type for token validation. This controls the behavior of the bearer authentication handler. The *JwtBearer* in the option’s name signifies that these options are for the JWT bearer settings. JWT stands for JSON Web Token, and these tokens represent an internet standard for authentication. ASP.NET Core uses these tokens to materialize the `ClaimsPrincipal` instance per-authenticated request.

The `AddPwnedServices` extension method is called, given the configuration’s `"Hib⁠p​Options"` section and the default HTTP retry policy. This project relies on the Web.Http.Extensions project. These extensions expose a common set of HTTP-based retry logic, relying on the Polly library. Following this pattern, the entire app shares a common transient fault handling policy to help keep everything running smoothly. Additionally, `PwnedServices` is added to DI as a singleton.

The next extension method to evaluate after `AddPwnedEndpoints` is `MapPwned​End⁠points`. This happens in the *WebApplicationExtensions.cs* C# file in the Web​.Pwne⁠dApi project:

[PRE7]

[![1](assets/1.png)](#co___componentizing_CO5-1)

After ensuring that `app` is not null, some common middleware is added.

[![2](assets/2.png)](#co___componentizing_CO5-2)

Both `Breach` and `PwnedPasswords` endpoints are mapped.

[![3](assets/3.png)](#co___componentizing_CO5-3)

Relying on the framework-provided `MapGet`, two endpoints are mapped to two handlers.

[![4](assets/4.png)](#co___componentizing_CO5-4)

Again, endpoints are mapped to handlers, this time for `PwnedPasswords`.

[![5](assets/5.png)](#co___componentizing_CO5-5)

The handler method can use framework-provided attributes and DI.

[![6](assets/6.png)](#co___componentizing_CO5-6)

Each handler is isolated and declarative.

The code uses HTTPS redirection, CORS, authentication, and authorization middleware. This middleware is commonplace with ASP.NET Core web app development and is part of the framework.

The `app` maps breach endpoints and Pwned `passwords` endpoints. These are entirely custom endpoints, defined within extension methods. After these methods are called, the `app` is returned, which fulfills a fluent API. This is what enabled the `Program` to chain calls after `builder.Build()`.

The `MapBreachEndpoints` method maps two patterns and their corresponding `Delegate handler` before returning. Each endpoint has a route pattern, which starts with `"api/pwned"`. These endpoints have placeholders for route parameters. These mapped endpoint route handlers are executed only when the framework determines the request has a matching route pattern; for example, an authenticated user could do the following:

*   Request `https://example-domain.com/api/pwned/breaches/test@email.org` and run the `GetBreachHeadersForAccountAsync` delegate

*   Request `https://example-domain.com/api/pwned/breach/linkedin` and run the `GetBreachAsync` delegate

The `MapPwnedPasswordsEndpoints` method maps the password’s endpoint to the `GetPwnedPasswordAsync` handler.

The `GetBreachHeadersForAccountAsync` method is an `async Task<IResult>` returning method. It declares an `Authorize` attribute, which protects this handler from unauthorized requests. Furthermore, it declares a `RequiredScope` of `"User.ApiAccess"`, which is the scope defined in the Azure AD B2C tenant. In other words, this handler (or API, for that matter) will be accessible only to an authenticated user from our Azure AD B2C tenant who has this specific scope. Users of the Learning Blazor application will have this scope, therefore, they can access this API. The method declares the `EnableCors` attribute, which ensures that this handler uses the configured CORS policy. Besides all of that, this method is like any other C# method. It requires a few parameters:

`[FromRoute] string email`

The `FromRoute` attribute on the parameter tells the framework that the parameter is to be provided from the `{email}` placeholder in the route pattern.

`PwnedServices pwnedServices`

The service instance is injected from DI, and the breach headers are asynchronously requested given the `email`. `breaches` are returned as JSON.

The `GetPwnedPasswordAsync` method is much like the previous, except it expects a `password` from the route and the `IPwnedPasswordsClient` instance from the DI container.

Through the lens of our application, it’s helpful to the users to make this information readily available. When the user performs their login, we’ll check the HIBP API and report back. As a user, I can trust that the app will do its intended work and I don’t have to manually check or wait for an email. As I use the app, it’s helping me by making information immediately available, which would otherwise be inconvenient to dig up. The Learning Blazor application does rely on the `HaveIBeenPwned.Client` NuGet package and exposes it through its Web Pwned API project.

## Restricting Access to Resources

If you recall, our markup thus far made use of the `Authorize` framework-provided component to protect various client rendering of custom components. We can continue to selectively use this approach to restrict access to functionality in your app. This is known as *authorization*.

In the case of the sample application, the *Index.razor* markup file uses authorization to hide the routes when the app doesn’t have an authenticated user:

[PRE8]

[![1](assets/1.png)](#co___componentizing_CO6-1)

The default page is the `Index` page, at the root of the application.

[![2](assets/2.png)](#co___componentizing_CO6-2)

The `PageTitle` component is used to display the page title.

[![3](assets/3.png)](#co___componentizing_CO6-3)

The `AuthorizeView` component is used to conditionally display the page content.

[![4](assets/4.png)](#co___componentizing_CO6-4)

`NotAuthorized` will redirect to the login page.

[![5](assets/5.png)](#co___componentizing_CO6-5)

`Authorized` will display `IntroductionComponent`, `JokeComponent`, and `WeatherComponent`.

This is the first time seeing the `@page` directive. This is how you template your apps’ navigation and client-side routing. Each component within a Blazor app that defines a page will serve as a user-navigable route. The routes are defined as a C# string. This literal is a value used to define the route templates, route parameters, and route constraints.

`PageTitle` is a framework-provided component that allows for the dynamic updating of the page’s `head > title`, its HTML DOM `<title>` element. This is the value that will display in the browser tab UI.

The `AuthorizeView` template component exposes the `NotAuthorized` and `Authorized` render fragments. These are templates specific to the state of the current user in context.

When the user is not authorized, we’ll redirect the user. We’ve already discussed the ability to redirect an unauthenticated user using the `RedirectToLogin` component. See [“Redirect to login when unauthenticated”](ch02.html#redirect-to-login).

When there is an authenticated user, they’ll see three tiles. The first tile is a simple “thank you” message for you, the user of the app and consumer of my book! It renders the custom `IntroductionComponent`. The second tile is the joke component. It’s backed by an aggregate joke service that randomly attempts to provide developer humor from multiple sources. The last tile spans the entire row under the intro and joke components, and it displays `WeatherComponent`. We’ll discuss each of these various custom Blazor component implementations and their varying degrees of data binding and event handling.

# The Introduction Component Says “Hi”

The next component of the Learning Blazor app is the `IntroductionComponent` that says “Hi” to those who visit the app, as shown in [Figure 3-2](#intro-light).

![](assets/lblz_0302.png)

###### Figure 3-2\. An example rendering of the `IntroductionComponent`

Have a look at the *Components/IntroductionComponent.razor.cs* C# file of the Web.Client project:

[PRE9]

[![1](assets/1.png)](#co___componentizing_CO7-1)

The component is `using Microsoft.Extensions.Localization`.

[![2](assets/2.png)](#co___componentizing_CO7-2)

It defines a single property.

`class` makes use of the `LocalizedString` type, which is a locale-specific `string`. This comes from the `Microsoft.Extensions.Localization` namespace.

`class` defines a single field named `_intro`, which is expressed as a call to the `Localizer` given the `"ThankYou"` key. This key identifies the resource to resolve from the localizer instance. In Blazor WebAssembly, localized resources such as those found in *.resx* files are available using the `IStringLocalizer` framework-provided type. The `Localizer` type, however, is a custom type named `CoalescingString​Local⁠izer`. This type is covered in more detail in [Chapter 5](ch05.html#chapter-five).

The `Localizer` member comes from the `LocalizableComponentBase` type. This is a subclass for a lot of our components. Now, let’s look at the *Introduction​Compo⁠nent.razor* markup file:

[PRE10]

[![1](assets/1.png)](#co___componentizing_CO8-1)

The component is a beautifully styled `<article>` element.

[![2](assets/2.png)](#co___componentizing_CO8-2)

There is a localized greeting message.

[![3](assets/3.png)](#co___componentizing_CO8-3)

`_intro` has its value bound to the `Message` property of `AdditiveSpeech​Compo⁠nent`.

[![4](assets/4.png)](#co___componentizing_CO8-4)

The `_intro` value is also rendered as text within the `<p>` element.

The HTML markup, for the most part, is pure HTML. If you look it over, you should notice only a few Blazor bits.

The Razor code context switches from raw HTML to accessing the `Localizer` instance in the `class`. I wanted to demonstrate that you can use fields in the `class`, or access other members to achieve one-way data binding. The localized message corresponding to the `"Hi"` key is bound after the waving emoji hand. The greeting message is “Hi, I’m David.”

There is a custom `AdditiveSpeechComponent` that has a `Message` parameter bound to `_intro.Value`. This component will render a button in the top-right corner of the tile. This button, when clicked, will read the given `Message` value to the user. The `AdditiveSpeechComponent` component is covered in detail in the next chapter.

The `_intro` localized resource value is splatted into the `<p>` element.

The localized resource files, by convention, have names that align with the file they’re localizing. For example, the *Introduction​Component.razor.cs* file has an *Introduction​Component.razor.en.resx* XML file. The following is a trimmed-down example of what its contents would look like:

[PRE11]

Within a top-level `root` node, there are `data` nodes. Each `data` node has a `name` attribute, and the name is the key used to retrieve the resource’s `value`. There can be any number of `data` nodes. This example file is in English, while other languages would use their specific locale identifier in the file name. For example, a French resource file would be named *IntroductionComponent.razor.fr.resx*, and it would contain the same `root > data [name]` structure, but its `value` nodes would have French translations instead. The same is true for any locale the app intends to provide resources for.

The introduction component shows one-way data binding and localized content. Let’s extend these two concepts a bit further and explore `JokeComponent`.

# The Joke Component and Services

The joke component of the Learning Blazor app displays a random joke. The joke component will render a spinner while it’s busy fetching a random joke from the endpoint. When the joke is retrieved successfully, it will render with a random joke similar to that shown in [Figure 3-3](#jokes-light).

###### Tip

I love the Internet Chuck Norris Database (icndb). I use it a lot for programming demos. Not only does it provide nerdy humor, but I like its simplicity. It makes for a compelling story. Likewise, jokes often find their way into my household. Being a father of three sons, I know that my boys love hearing “dad jokes,” and what makes them happy brings me joy.

![](assets/lblz_0303.png)

###### Figure 3-3\. An example rendering of the `JokeComponent`

This component makes an HTTP request to the `api/jokes` Web API endpoint. The joke object itself is shared with both the Web API endpoint and the client-side code. This helps to ensure that there aren’t any misalignments with the data structure, which could cause serialization errors or missing data. Consider the *Joke​Compo⁠nent.razor* markup file:

[PRE12]

[![1](assets/1.png)](#co___componentizing_CO9-1)

`IJokeFactory` is injected into the component.

[![2](assets/2.png)](#co___componentizing_CO9-2)

Like its counterpart components on the `Index` page, `JokeComponent` renders a styled `article` element.

[![3](assets/3.png)](#co___componentizing_CO9-3)

When loading, a spinner is displayed.

[![4](assets/4.png)](#co___componentizing_CO9-4)

The `@code` directive is used to specify the code block.

[![5](assets/5.png)](#co___componentizing_CO9-5)

The `RefreshJokeAsync` method is called to fetch a new joke.

The `JokeComponent` markup starts like most other components, by declaring various directives. `JokeComponent` has the framework inject an `IJokeFactory`, `ILogger<JokeComponent>`, and `IStringLocalizer<JokeComponent>`. Any service that is registered in the DI container is a valid `@inject` directive target type. This component makes use of these specific services.

The HTML markup is a bit more verbose than the introduction component. Component complexity is something you should evaluate and be aware of. It’s a good rule of thumb to limit a component to a single responsibility. The responsibility of the joke component is to render a joke in HTML. The markup is similar to the introduction component, providing an emoji and localized title, as well as an `AdditiveSpeech​Com⁠ponent` that’s bound to the `_jokeText` variable.

The content markup for this joke component is conditional, and the use of `@if, else if, else, and @switch` expressions are supported control structures. This has been a part of the Razor syntax since the beginning. When the value of `_isLoadingJoke` evaluates as `true`, a stylized `SpinnerComponent` markup is rendered. `Spinner​Compo⁠nent` is custom too, and it’s a tiny bit of common HTML. Otherwise, when `_joke​Text is not null`, the random joke text is rendered as a `blockquote`.

The joke component uses an `@code { ... }` directive rather than the shadowed component approach. It’s important to understand that as a developer, you have options. More often than not, I prefer to not use `@code` directives. To me, it’s cleaner to keep them in separate files. I like seeing a C# class, and it feels a bit more natural to me that way. But if you’re a developer coming from the JavaScript SPA world, it might feel more natural to have the files together. The point is that the only way to determine the best approach is to gain a consensus from your team, much like other stylistic developer decisions.

The `RefreshJokeAsync` method is called by the `OnInitializedAsync` lifecycle method. This means that as part of the component’s initialization, the fetching of a joke will occur asynchronously. The method starts by setting the `_isLoadingJoke` bit to `true`; this will cause the spinner markup to be rendered—but only temporarily. The method body tries to ask the `IJokeFactory` instance to get a `JokeResponse` object. When there is a valid `response`, it’s deconstructed into a tuple assignment that sets the `_jokeText` and `_sourceDetails` fields. These are then rendered as the contents of the joke itself.

The endpoints that power these jokes aggregate several third-party APIs together. The various joke endpoints have different data structures, and there are services in place to converge them into a single endpoint that our Blazor client code consumes.

## Aggregating Joke Services—Laughter Ensues

No application is useful without meaningful data. Our app will have client-specific weather, random nerdy jokes, real-time web functionality, chat, notifications, a live Twitter stream, on-demand HIBP security features, and more. This is going to be fun! But what does this mean for Blazor? Before diving into the weeds with Blazor frontend development, we should set a few more expectations about the services and data driving this application—our backend development.

Blazor apps are free to retrieve and use data from any number of other platforms, services, or web applications. Many good architectures exist, with many possible solutions for any given problem domain. After all, knowing when to use which pattern or practice is part of being successful. You should try to identify the flow of data and basic requirements, where data comes from, and how to access this data. Does this data change frequently, is the data used to calculate other points of interest, and is the data dynamic or static? These are the better questions to be asking yourself. The answer is almost always “It depends.”

Let’s take a look at how the joke service library provides random jokes:

[PRE13]

Before C# 10, `namespace` declarations wrapped their containing types in curly brackets. With C# 10, you can use file-scoped namespace, which enhances the readability by removing a level of indentation in the code. I like this feature; even though it’s a bit subtle, it does reduce noise when reading the code.

`IJokeService` is an `internal interface` type, which exposes a read-only `JokeSour⁠ce​Details` property and the ability to request a joke asynchronously. The `internal` access modifier means that the joke service is not exposed outside of the declaring assembly.

The `GetJokeAsync` method is parameterless and returns a `Task<string?>`. The `?` on the `string` type declaration identifies that the returned `string` could be `null` (the default value of the C# reference type `string`).

We have three different third-party joke web services, all of which are free. The shapes of the joke responses vary by provider, as do the URLs. We have three separate configurations, endpoints, and joke models that we have to represent.

The first `IJokeService` implementation is the `ProgrammingJokeService`:

[PRE14]

[![1](assets/1.png)](#co___componentizing_CO10-1)

The `ProgrammingJokeService` class implements the `IJokeService` interface.

[![2](assets/2.png)](#co___componentizing_CO10-2)

The `HttpClient` and `ILogger<T>` instances are injected into the constructor.

[![3](assets/3.png)](#co___componentizing_CO10-3)

The `SourceDetails` property provides information about the source of the joke.

[![4](assets/4.png)](#co___componentizing_CO10-4)

The `GetJokeAsync` method returns a `Task<string?>` that resolves to a joke or `null` if no joke could be retrieved.

This service starts with its namespace declaration followed by an `internal class` implementation of `IJokeService`.

The class requires two parameters, an `HttpClient` and an `ILogger<ProgrammingJokeService>` logger instance. These two parameters are assigned using a tuple literal and its immediate deconstruction into the field assignments. This allows for a single line and an expression-bodied constructor. This is just a boilerplate DI approach. The fields are safely typed as `private readonly` so that consumers in the `class` will not be permitted to mistakenly assign over their values. That is the responsibility of the DI container.

The programming joke service declaratively expresses its representation of the `SourceDetails` member through an implicit target-type `new` expression. We instantiate an instance of `JokeSourceDetails` given the `enum` value of the underlying API type `JokeSource.RandomProgrammingJokeApi` and the joke URL in a .NET `Uri` object.

The actual implementation of `GetJokeAsync` starts by opening with a `try` and `catch` block. `_httpClient` is used to make an HTTP GET request from the given `reques⁠t​Uri` and default JSON serialization options. In the event of an error, `Exception` details are logged and `null` is returned. When there is no error, in other words, “the happy path,” the response from the request is deserialized into a `ProgrammingJoke` array object. When there are jokes, the first joke’s text is returned. If this is `null`, that is fine too since we’ll let the consumers handle that. We’ll need to indicate it to them—again, it’s a `string?`. I call nullable types “questionable.” For example, given a `string?`, you should be asking yourself if this is `null` and should guard for that appropriately. I’ll often refer to this type of pattern as a *questionable string*.

The other two service implementations follow the same pattern, and it becomes clear that we’ll need a way to aggregate these as they represent multiple implementations of the same interface. When .NET encounters multiple services registered for the same type, they are wrapped in `IEnumerable<TService>` where `TService` is one of the given implementations.

Let’s continue by looking at the other two `IJokeService` implementations. Consider the following `DadJokeService` implementation:

[PRE15]

And the `ChuckNorrisJokeService` implementation:

[PRE16]

To handle the multiple `IJokeService` implementations, we’ll create a factory that will aggregate jokes—returning the first successful random implementation’s joke:

[PRE17]

This interface defines a single task-based async method that by its name indicates it gets a random joke. The return type is a `Task<(string, JokeSourceDetails)>`, where the generic type constraint on `Task` is a tuple of `string` and `JokeSource​De⁠tails`. `JokeSourceDetails` is shaped as follows:

[PRE18]

In C#, positional records are an amazing type. First of all, they’re immutable. Instances can be cloned using the `with` syntax, where property values are overridden into the copied object. You also get automatic equality and value-based comparison semantics. They’re declarative and succinct to write. Let’s take a look at the joke factory next:

[PRE19]

[![1](assets/1.png)](#co___componentizing_CO11-1)

The constructor accepts a collection of `IJokeService` implementations.

[![2](assets/2.png)](#co___componentizing_CO11-2)

The method body of `GetRandomJokeAsync` uses the `RandomOrder` function.

The `IJokeFactory` implementation is named appropriately as `AggregateJoke​Fac⁠tory` with its constructor (`.ctor`) accepting `IEnumerable<IJokeService>`. These are the joke services: *dad joke service*, *random programming joke API service*, and *internet Chuck Norris database service*. These values were provided by the .NET DI container.

The method body of `GetRandomJokeAsync` is leveraging an extension method named `RandomOrder` on the `IEnumerable<T>` type. This pattern relies on a fallback pattern in which services are attempted until one is capable of providing a joke. If no implementation is capable of providing a joke, the method default values, in this case, return `null`. The extension method for random is defined in the *Enumerable​Exten⁠sions.cs* C# file in the `Learning.Blazor.Extensions` namespace:

[PRE20]

[![1](assets/1.png)](#co___componentizing_CO12-1)

The framework-provided `Random` type.

[![2](assets/2.png)](#co___componentizing_CO12-2)

The algorithm for randomizing the order is O(1) time, meaning its computation time is constant regardless of the size of the collection.

The framework-provided [`Random.Shared` instance](https://oreil.ly/sYped) represents a pseudorandom number generator, which is an algorithm that produces a sequence of numbers that meet basic statistical requirements for randomness.

The random element function works on the `incoming` collection instance. From the `AggregateJokeFactory` instance we pseudorandomly determined, we’ll await its invocation of the `GetJokeAsync` method. If the joke returned is `null`, we’ll coalesce to `"There is nothing funny about this."` We then return a tuple with the `string` joke and the corresponding service’s source details.

## DI from Library Authors

The last part of the joke services library involves the fact that all of our joke services are DI-friendly, and we can add an extension method on `IServiceCollection` that registers them with the DI container. This is a common tactic that I’ll follow for all libraries that are intended for consumption. Consumers will call `AddJokeServices` to register all abstractions with DI. They can start requiring these services for `.ctor` injection in classes or with Blazor components through the property injection. The `InjectAttribute` and the `@inject` directive allow for services to be injected into components through their C# properties.

[PRE21]

[![1](assets/1.png)](#co___componentizing_CO13-1)

The class is `using` the `Learning.Blazor.Http.Extensions` namespace.

[![2](assets/2.png)](#co___componentizing_CO13-2)

All three service implementations are added to the `services` collection.

[![3](assets/3.png)](#co___componentizing_CO13-3)

Each implementation has its corresponding `HttpClient`.

[![4](assets/4.png)](#co___componentizing_CO13-4)

Collectively, each implementation is exposed through `AggregateJokeFactory`.

The `Learning.Blazor.Http.Extensions` namespace represents a shared library, which contains default, transient fault-handling policies. A reasonable set of defaults is shared throughout all projects in the solution that use an `HttpClient`. These shared fault-handling policies impose an exponential backoff pattern that helps to automatically retry intermittent HTTP request failures. They generate sleep durations that exponentially backoff, in a jittered manner, making sure to mitigate any correlations. Examples include 850ms, 1455ms, and 3060ms. This is possible using the `Polly.Contrib.WaitAndRetry` library and its `Backoff.Decorrelated​Jit⁠terBack⁠offV2` implementation.

Calling `AddJokesServices` registers all of the corresponding joke services into the DI container. Once registered in the DI container, consumers can require the `IJokeFactory` service and the implementation will be provided. All of this functionality is exposed to the Web.Client. The `JokeComponent` uses the `IJokeFactory.GetRandomJokeAsync` method. The client code will execute on the client browser, using each service to make HTTP calls to some external endpoints as needed.

We’ve covered `IntroductionComponent` and `JokeComponent`. In the next section, we’re going to look at a gradually more complex example. I’ll show you how to make a call to an Azure Function that is co-hosted with the Azure Static Web App. This Azure Function is implemented in the Web.Functions project.

###### Tip

Azure Functions are a serverless solution (similar to that of AWS Lambda). They are a great way to build scalable, reliable, and secure applications using Azure PaaS (platform as a service). For more information, see Microsoft’s [“Introduction to Azure Functions” documentation](https://oreil.ly/bJr70).

# Forecasting Local Weather

The custom components that we’ve covered thus far started a bit more basic. `IntroductionComponent` has a single localized text field that it renders. `Joke​Compo⁠nent` then demonstrated how to fetch data from an HTTP endpoint with conditional control structures and loading indicators. `WeatherComponent` is a parent component to `WeatherCurrentComponent` and `WeatherDailyComponent`. Collectively, these components display the users’ local current weather and immediate forecast for the week, as shown in [Figure 3-4](#weather-dark).

![](assets/lblz_0304.png)

###### Figure 3-4\. An example rendering of the `WeatherComponent`

All of the weather data is available for free from the [Open Weather Map API](https://oreil.ly/lg5RK). `WeatherComponent` relies on an `HttpClient` instance to retrieve weather data. In this component, we’ll also cover how to use two-way JavaScript interop. Let’s look at the *WeatherComponent.razor* markup:

[PRE22]

[![1](assets/1.png)](#co___componentizing_CO14-1)

The outermost `article` element is styled as a tile.

[![2](assets/2.png)](#co___componentizing_CO14-2)

The weather tile, like the other two tiles, also makes use of `AdditiveSpeech​Compo⁠nent`.

[![3](assets/3.png)](#co___componentizing_CO14-3)

In addition to simple `@if` control structures, you can also use `@switch` control structures.

[![4](assets/4.png)](#co___componentizing_CO14-4)

When loaded, the weather tile displays the current weather and the forecast for the week.

[![5](assets/5.png)](#co___componentizing_CO14-5)

When the component is loading, `SpinnerComponent` is shown.

[![6](assets/6.png)](#co___componentizing_CO14-6)

The `default` case renders a localized message that tells the user that weather is unavailable.

This component’s markup is similar to the other two tiles, `IntroductionComponent` and `JokeComponent`. `WeatherComponent` is a parent component of two other components: `WeatherCurrentComponent` and `WeatherDailyComponent`. Its title is “Blazor Weather,” and the word *weather* is localized.

The weather tile, like the other two tiles, also makes use of `AdditiveSpeech​Compo⁠nent`. When rendered, a speech button is visible in the top-righthand corner of its parent element. `AdditiveSpeechComponent` is covered in detail in [“Native Speech Synthesis”](ch04.html#native-speech-synthesis).

The `@switch` control structure is rather nice in markup. The weather component uses a custom component `_state` variable to help track the state of the component. The possible states are unknown, loading, loaded, or error.

When the component is loaded, the current weather (`WeatherCurrentComponent`) and daily weather forecast (`WeatherDailyComponent`) are rendered. The parent component relies on a nullable `_model` type; the `_model` is not `null` when in a loaded state, and we can tell the compiler that we’re certain of that by using the null-forgiving operator `!`. The class-scoped `_model` variable is assigned to a local-scoped `weather` variable. This variable is assigned to its child components’ `WeatherCurrentComponent` and `WeatherDailyComponent` through either helper method delegation or parameter assignment.

When the component is loading, `SpinnerComponent` is shown. The `default` case renders a localized message that tells the user that the weather is unavailable. This should happen only in the event of an error.

The weather component markup references the current weather (`WeatherCurrent​Com⁠ponent`) and daily weather forecast (`WeatherDailyComponent`) components. These two components do not make use of component shadowing and are purely for templates. Each component defines an `@code { ... }` directive with several `Parameter` properties. They do not require logic or functionality; as such, they’re just markup bound to given values. This is the *WeatherCurrentComponent.razor* markup file:

[PRE23]

`WeatherCurrentComponent` renders the image that corresponds to the current weather, such as clouds, or rain clouds, or perhaps even an image of the sun to represent a beautiful day. It also displays the temperature, high and low temperatures, a description of the weather, the wind speed and direction, as well as the city and state. For example, let’s look at the *WeatherDailyComponent.razor* markup file:

[PRE24]

`WeatherDailyComponent` uses delegates as parameters for some of its data-binding needs. It renders the day for the forecast and an icon for the forecasted weather, along with the description and highs and lows.

`WeatherComponent` relies on several services and refreshes the weather automatically using a timer, which we will look at next. This component shows a lot of powerful functionality. Now that you’ve explored the markup, consider the shadowed component C# file, *WeatherComponent.razor.cs* ([Example 3-1](#weather-component)).

##### Example 3-1\. Web.Client/Components/WeatherComponent.razor.cs

[PRE25]

[![1](assets/1.png)](#co___componentizing_CO15-1)

There are several fields and properties that `WeatherComponent` manages.

[![2](assets/2.png)](#co___componentizing_CO15-2)

When the component is initialized, a call to `TryGetClientCoordinatesAsync` is made.

[![3](assets/3.png)](#co___componentizing_CO15-3)

The `OnCoordinatesPermittedAsync` method is called when the user grants permission to geolocation.

[![4](assets/4.png)](#co___componentizing_CO15-4)

The `OnErrorRequestingCoordinatesAsync` method is called when the user does not grant permission to geolocation.

[![5](assets/5.png)](#co___componentizing_CO15-5)

The `Dispose` method performs cleanup of the `CancellationTokenSource` and `PeriodicTimer` objects.

The weather component relies on the browser’s geolocation, which is natively guarded and requires the user to grant permission. The component has several field variables used to hold this information if the user permits it. The `Coordinates` object is a C# positional record type with latitude and longitude properties. The `GeoCode` object contains the city, country, and other similar information. It is instantiated from an HTTP call to the [Big Data Cloud API](https://oreil.ly/9AtzC). This call is conditional and occurs only when the user grants access to the browser’s geolocation service. In addition to these variables, there’s a component model and state. There is also `PeriodicTimer`. `Period⁠ic​Timer` was introduced with .NET 6, and it provides a lightweight asynchronous timer. It is configured to tick every 10 minutes. The component requests that the DI container inject a formatter, HTTP client, and geolocation service.

When the component is initialized, a call to `TryGetClientCoordinatesAsync` is awaited. This method is expressed as a call to `JavaScript.GetCoordinatesAsync` given `this` and two method names. This is a JavaScript interop call from .NET, and the corresponding extension method is explained in the next section. Just know that calling `TryGetClientCoordinatesAsync` will result in one of two methods being called, either the `OnCoordinatesPermittedAsync` method or the `OnErrorRequestingCoordinatesAsync` method.

When the user grants permission to the app (or if they have already at one point in time), the `OnCoordinatesPermittedAsync` method is called and given the geo-location represented as a *latitude* and *longitude* pair. This method is invoked from JavaScript, so it needs to be decorated with the `JSInvokable` attribute. When called, the `longitude` and `latitude` values will be provided with valid values. These values are then used to instantiate the component’s `_coordinates` object. At this point, the method tries to make a series of HTTP calls, sequentially relying on the previous request. The weather service API allows for a set number of languages that it supports. We need to use the current browser’s language, which is represented by their preferred ISO 639-1 two-letter language code. With the language code, we can also now infer a default unit of measure for the temperature, either `Metric` °C (degrees Celsius) or `Imperial` °F (degrees Fahrenheit). We need to read what languages the weather API supports, so a call to the `api/weather/languages` HTTP endpoint is made. This returns a collection of `WeatherLanguage` objects. The `api/weather/latest` HTTP endpoint returns a `WeatherDetails` object, which is then used to instantiate the weather component’s `_model`. Around the same time that this is happening, the `_geoCode` object is being fetched from the `GeoLocationService.GetGeoCodeAsync`.

When there are errors, they’re logged to the browser’s console, and the `_state` is set to `Error`, causing the markup to render that the weather service is unavailable. All of these changes are then communicated back to the component by calling `StateHasChanged`. The UI will rerender when applicable. All of this code is wrapped in a `do`/`while` construct. `while` is conditional on `_timer` and `_cancella⁠tion​.Token`. This is the pattern to use when you need to periodically update values. It occurs only once from the callback; after that each invocation is controlled and protected by `PeriodicTimer`, which coalesces multiple ticks into a single tick between calls to its `WaitForNextTickAsync` method.

The `OnErrorRequestingCoordinatesAsync` method is only called when the user disables or later denies location permissions by changing the browser’s setting to blocked. When the user makes these changes, the browser will prompt the user to refresh the web app. The native browser permissions API will change the app’s ability to render weather. This callback method and the `OnCoordinatesPermittedAsync` methods are mutually exclusive and will fire only once from the client. The refresh will, however, trigger a reevaluation of the location permissions API.

The weather component demonstrates how to perform conditional rendering of various UI elements with Blazor data binding, from showing the user a `Spinner​Compo⁠nent` that indicates loading, to an error message that encourages the user to enable the location permissions, to customized weather for your shared location. All of this happens asynchronously, using DI and powerful C# 10 features on a periodic timer automatically. The periodic timer implements its `IDisposable.Dispose` functionality through the weather component, so as the component is being cleaned up, so too are the timer’s resources.

From the C# code, you will have noticed the `JavaScript.GetCoordinatesAsync` method. The arrival of these coordinates is what initiates the whole process. You will see a trend that I’m trying to convey here; specifically, I want all JavaScript interop functions to be encapsulated into extension methods. This will allow for easier unit and integration testing. For more information on testing, see [Chapter 9](ch09.html#chapter-nine). Consider the *JSRuntimeExtensions.cs* C# file:

[PRE26]

[![1](assets/1.png)](#co___componentizing_CO16-1)

The `JSRuntimeExtensions` class relies on the `Microsoft.JSInterop.IJSRuntime` type.

[![2](assets/2.png)](#co___componentizing_CO16-2)

`GetCoordinatesAsync` extends the `IJSRuntime` interface.

[![3](assets/3.png)](#co___componentizing_CO16-3)

Any component can call this extension method and pass itself as the generic-type parameter.

[![4](assets/4.png)](#co___componentizing_CO16-4)

`DotNetObjectReference` is created from the given `dotnetObj` and passed to the interop call.

`Microsoft.JSInterop` is a framework-provided namespace. There are many useful types that you should get used to using:

`DotNetObjectReference<TValue>`

Wraps a JS interop argument, indicating that the value should not be serialized as JSON but instead should be passed as a reference. This reference is then used by JavaScript to call methods on the .NET object it wraps.

`IJSRuntime`

Represents an instance of a JavaScript runtime to which calls may be dispatched. This is common to both Blazor Server and Blazor WebAssembly, and it exposes only asynchronous APIs.

`IJSInProcessRuntime`

Represents an instance of a JavaScript runtime to which calls may be dispatched. This is specific to Blazor WebAssembly because the process is shared, unlike Blazor Server. This interface inherits the `IJSRuntime` and adds a single synchronous `TResult Invoke<TResult>` method.

`IJSUnmarshalledRuntime`

Represents an instance of a JavaScript runtime to which calls may be dispatched without JSON marshaling. Currently, it is supported only on WebAssembly and for security reasons, will never be supported for .NET code that runs on the server. This is an advanced mechanism that should be used only in performance-critical scenarios.

The class extends the `IJSRuntime` type, and the `GetCoordinatesAsync` method returns `ValueTask` and accepts a single generic-type parameter `T`. The method requires the `T` instance and two method names for success and error callbacks. These method names are used from JavaScript to know what methods to invoke.

The generic type parameter `T` is constrained to a `class`; any component instance will suffice. The method body is an expression-bodied definition and lacks the `async` and `await` keywords. They are *not* necessary here because this extension method simply describes the intended asynchronous operation. Using the given `jsRuntime` instance that this method extends, it calls `InvokeVoidAsync`. This is not to be confused with “async void”; while the name is a bit confusing, it’s trying to convey that this JavaScript interop method doesn’t expect a result to be returned. The corresponding JavaScript function that is invoked is `app.getClientCoordinates`.

`DotNetObjectReference.Create(dotnetObj)` wraps `dotnetObj`, and it is what’s passed as a reference to the JavaScript call. Blazor’s JavaScript bidirectional interop support relies on `DotNetObjectReference` and maintains a special understanding of these types. `successMethodName` and `errorMethodName` are actual method names on the `dotnetObj` instance with the `JSInvokable` attribute.

After looking through the Razor markup, the shadowed component C#, and the extension method functionality, let’s follow the call through to JavaScript. Consider the *app.js* JavaScript file:

[PRE27]

[![1](assets/1.png)](#co___componentizing_CO17-1)

The `getClientCoordinates` function accepts a few parameters.

[![2](assets/2.png)](#co___componentizing_CO17-2)

If the browser supports the `geolocation` API, the `getCurrentPosition` method is called.

[![3](assets/3.png)](#co___componentizing_CO17-3)

When a `position` is available, the object reference has its success method invoked.

[![4](assets/4.png)](#co___componentizing_CO17-4)

When an error occurs, the object reference has its error method invoked.

[![5](assets/5.png)](#co___componentizing_CO17-5)

The `window.app` object is created (or updated) to include `getClient​Coordi⁠nates`.

The JavaScript file defines a `const` function named `getClientCoordinates`, which declares a method signature expecting a `dotnetObj`, `successMethodName`, and `errorMethodName`.

The function starts by asking if the browser’s `navigator` and `navigator​.geo⁠loca⁠tion` are truthy. If they are, a call to `getCurrentPosition` is invoked. This function is protected by the browser’s location permissions. If the user has not provided permission, they are prompted. If they deny this permission, the API will never call the successful callback.

When the user has already permitted access to the location services, this method will immediately call the first callback with a valid `position`. The `position` object has the `latitude` and `longitude` coordinates. From these coordinates, and the reference to `dotnetObj` with the given `successMethodName`, it calls back into the .NET code from JavaScript. This will call the `WeatherComponent.OnCoordinatesPermitte⁠d​Async` method passing the coordinates.

If there is an error for any reason, the second registered callback is invoked given the `error` object. The `error` object has an error `code` value and a `message`. The possible error `code` values are as follows:

`1`: `PERMISSION_DENIED`

When the page didn’t have permission to acquire `geolocation` information

`2`: `POSITION_UNAVAILABLE`

When an internal error occurs trying to acquire `geolocation` information

`3`: `TIMEOUT`

When the allowed time to acquire `geolocation` information was reached before acquiring it

Now that the `getClientCoordinates` function is fully defined, it’s added the `app` object on the `window` scope. If there are multiple JavaScript files defined in your apps that use the same object name on `window`, you can use the JavaScript spread operator to append the new functions into the existing object without overwriting it completely.

Assuming that you grant permissions to the app when prompted, the markup will render the component on the user’s screen.

# Summary

In this chapter, the app took flight and you learned how to put the user first by using the authenticated user information to better personalize the user’s experience with our app. When user-centric content was rendering, the user is prompted to allow `geolocation` services (native to the browser) to use their coordinates. Using this personal information, the user’s local current weather and weather forecasts are displayed. You learned how to render component variables through various control structures, such as `@if` and `@switch` component expressions. We saw how to use services within a component, such as service libraries, and how to make HTTP calls using the `HttpClient` type. You learned a pattern to periodically update values automatically using `PeriodicTimer` from .NET. In addition to all of this, you also learned how to use the browser’s native `geolocation` service from Blazor with two-way JavaScript interop. The app greets the user with a message, a bit of laughter (or eye rolls if the jokes are bad enough), and a personalized weather forecast.

In the next chapter, you’ll learn how client services are registered for DI. You’ll learn how to customize the various authorizing states through component customization and Blazor render fragmentation. I’ll take you through another JavaScript interop scenario where you’ll learn how to convince the browser to utter a custom message with native speech synthesis. In the next chapter, you also learn how components communicate with events.