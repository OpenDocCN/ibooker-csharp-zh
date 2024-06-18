# Chapter 2\. Executing the App

In this chapter, you’ll learn how a Blazor WebAssembly app starts executing—from the rendering of static HTML to the invocation of JavaScript that bootstraps Blazor, you’ll be exploring the anatomy of the app. This includes the `Program` entry point and the startup conventions. You’ll learn about the router, client-side navigation, shared components, and layouts. You’ll also learn about top-level navigation and custom components in the app. All of this will be taught using the Learning Blazor sample application’s source code.

Try to embrace the mindset that you’re onboarding as a new developer to an existing application—much like you would in the real world. Try to imagine that you’re starting a new journey, where you’re getting brought up to speed on an existing codebase. The idea is that I’ll be your mentor; I’ll meticulously walk through the code, presenting it to you and explaining exactly what it’s doing and how it’s doing it. You’ll learn why certain decisions were made and what alternative approaches should be considered. You should have a grasp of how the model app works and will be prepared to work with it in future chapters.

In the previous chapter, you learned a bit about the web app development platform, the ASP.NET Core as a framework, open source development, the programming languages of the web, and development environments Now let’s talk code. As Linus Torvalds, the creator of Linux, said, “Talk is cheap. Show me the code.” The model app is the basis for the entire book, where you’ll learn how all of the major features of Blazor work and how to use other amazing features. We’ll look at the code together, and you’ll get to read the code and let it tell you its own story. In the next few sections, you’ll learn how the Blazor framework initializes the app and how the app starts executing. I suggest you check out [*https://webassemblyof.net*](https://webassemblyof.net) to see what the final web app looks like. Feel free to click around and try out the various features to familiarize yourself with the app.

# Requesting the Initial Page

Let’s start by evaluating what happens when a client browser wants to access our application. It requests the initial page (given its URL), and HTML is returned from the server. Within the HTML itself, there are `<link>` and `<script>` elements. These define additional references to resources that our Blazor application needs to start accepting user input with components rendered from the Blazor markup. The resources include, but are not limited to, CSS, JavaScript, images, Wasm files, and .NET dynamic-link libraries (.dlls). These additional resources are requested as part of the initial page load, and while this is happening there can be no interaction with the app. Depending on the size of the peripheral resources and the connection speed of the client, the amount of time it takes for the app to become interactive will vary.

The *Time to Interactive* (TTI) is a measurement of the amount of time it takes before a website is ready to accept user input. One of the trade-offs of using Blazor WebAssembly is that the initial load time of the app is a bit longer than that of Blazor Server. The app has to be downloaded to the browser before running, whereas with Blazor Server the app is rendered dynamically on the web host. This requires the .NET runtime and a configured web server.

###### Tip

One advantage of using Blazor WebAssembly is that the app can be hosted as a static web app. Serving static files is much faster and less error-prone than serving dynamic content. But it does come at a cost. The app will be downloaded to the client browser, and the client browser will have to download the entire app. This can be a large download, and it can be a bit slower than the app running on the server.

The TTI for Blazor WebAssembly can be a bit longer than that of Blazor Server. Hypothetically, if the TTI is more than a few seconds, users will expect some sort of visual indication, such as an animated spinning gear cog to show that the app is loading.

With Blazor WebAssembly, you can *lazy load* full .NET assemblies. This is much like doing the equivalent thing in JavaScript—where various components are represented by JavaScript—but instead we get to use C#. This feature can make your application more efficient by fetching only the dependent assembly on demand and when needed. Before showing you how to lazy load assemblies, however, you’ll learn how the Blazor WebAssembly application startup loads assemblies.

Let’s start by examining the parts of the initial page’s HTML content.

# App Startup and Bootstrapping

The following HTML is served to the client, and it’s important to understand what the client browser will do when it renders it. Let’s jump in and take a look at the *wwwroot/index.html* file from the Web.Client project. I know it’s a lot, but read through it first, and we’ll go through it piece-by-piece after:

[PRE0]

Let’s break down each of the primary sections. We’ll start with reading through the `<head>` tag’s child elements:

[PRE1]

The app uses the web standard UTF-8 character set, and there’s also a `viewport` specification, both of which are very common `<meta>` tags in HTML. We set the initial `<title>` of the page to `"Learning Blazor"`. After the title is a set of `<link>` elements. If you spend time evaluating alternative options to the default Bootstrap CSS from the template, you may consider a CSS framework that takes zero JavaScript dependencies.

In this instance, Bulma was chosen as the CSS framework because it’s amazingly simple and clean. This is a perfect match for Blazor, as we can use C# instead of JavaScript to change styles at will. As described in [the Bulma documentation](https://oreil.ly/kstH3), “Bulma is a CSS library. This means it provides CSS classes to help you style your HTML code. To use Bulma, you can either use the pre-compiled *.css* file or install the *.sass* files so you can customize it to your needs.” Bulma provides everything needed to style the website; with extensibility in mind, you have modern utilities, helpers, elements, components, forms, and layout styles. Bulma also has a huge developer community following, where extensions are shared. These additional CSS packages depend on Bulma itself; they just override or extend existing class definitions. This is the same approach in any web app development and is not unique to Blazor.

When we see a `<link>` element that has a `rel` attribute set to `"preload"`, it indicates that these requests will happen asynchronously. This works by adding the `as="style" onload="this.rel='stylesheet'"` attributes. This lets the browser know that the `<link>` is for a style sheet. It will also eventually load the resource, and when it does it will set `rel` to `"stylesheet"`. Let’s think of this as the *hot-swap on load tactic*. We will pull in some additional CSS references for sliders, quick views, tooltips, and media-query-centric `@media (prefers-color-scheme: dark) { /* styles */ }` functionality. This exposes the ability to detect the client’s preferred color scheme and apply the appropriate styles. For example, an alternative color scheme to the default `white` is `dark`. These two color schemes account for the majority of all web user experiences.

We then define another `<link>` with an `href` to the */css/app.css* path to the web server.

The important styles from Bulma are not using the *hot-swap on load tactic*. While the app is loading, it’s styled appropriately to communicate that the app is working (see [Figure 2-1](#blazor_webassembly_loading)). The app also preemptively declares `<link rel="manifest" href="/​man⁠ifest.json" />` with the corresponding `<link>` icons. This is specifically to expose icons and the PWA capabilities. Per MDN’s [HTML reference guide](https://oreil.ly/X62KY), “the HTML < base> element specifies the base URL to use for all relative URLs in a document. There can be only one `<base>` element in a document.”

All applications should consider the usage of iconography where possible to make for a more accessible web experience. Icons, when done correctly, can immediately convey a message and intent, and often with little text. I proudly use Font Awesome; they have a free offering and provide seamless integration of it wherever it is needed in Blazor markup. A `<script>` points to my Font Awesome kit registered to my app. The next line, immediately following the Font Awesome source, is the application’s JavaScript bits. There are three primary areas of focus in web app development, each within the */js*, */css*, and */_content* directories. After familiarizing yourself with the child elements of the `<head>` node, we can move on. Next, we’ll take a look at the content of the `<body>` nodes:

[PRE2]

The first tag in the `<body>` element is `<div id="app">...</div>`. This is the root of the Blazor application, the true SPA. It is very important to understand that the contents of this target element will be automatically and dynamically changed to represent the Wasm application’s manipulation of the DOM. Most SPA developers settle with letting the UX be a giant white wall of `10pt` default font with black text that reads “Loading…” It’s not okay for UX. Ideally, we’d want to provide visual cues to the user that the application is responsive and loading. One approach to achieve this is to have a `<div>` initially represent a basic splash screen. In this case, the model app will include the Blazor logo image and a message that reads `"Blazor WebAssembly: Loading..."` It will also show an animated loading spinner icon.

`<section id="splash">...</section>` acts as the loading markup. It will be replaced when Blazor is ready. This markup is not Blazor but rather HTML and CSS. This markup will render similarly to that shown in [Figure 2-1](#blazor_webassembly_loading). Without this markup, the default loading experience has black text and says “Loading.” This gives you the ability to customize the splash (or loading) screen UX.

###### Tip

When writing your Blazor apps, you should consider adding a loading indicator to your application. This is a great way to give users a sense of progress and avoid a “white screen of death” when the application is first loaded.

![](assets/lblz_0201.png)

###### Figure 2-1\. An indicator lets the user know the app is loading

In the *index.html* file, following the *app* node, there lies the “blazor-error-ui” `<div>` element. This is adjusted from the template to be a bit more suited to our app’s styling. This specific element identifier will be used by Blazor when it’s bootstrapping itself into the driver seat. If there are any unrecoverable errors, it will show this element. If all goes well, you shouldn’t see this.

After the error element are a few remaining `<script>` tags. These are the JavaScript references for our referenced components, such as authentication and our Twitter Component library. The final two `<script>` tags are very important:

[PRE3]

The first `<script>` tag’s referenced JavaScript (the *blazor.webassembly.js* file) is what starts the execution of Blazor WebAssembly. Without this line, this app would not render anything besides a never-ending loading page. This JavaScript file initiates the boot subroutine from Blazor, where WebAssembly takes hold, JavaScript interop lights up, and the fun starts! The various .NET executables, namely *.dlls*, are fetched, and the Mono runtime is prepared. As part of the Blazor boot subroutine, the *app* element is discovered in the document. The entry point of the app is invoked. This is where the .NET app starts executing in the context of WebAssembly.

The second `<script>` tag registers the application’s service worker JavaScript code. This exposes our app as a PWA, which is a nice feature. It’s a way to make your app available offline and service worker functionality. For more information about PWAs, see Microsoft’s [“Overview of Progressive Web Apps (PWAs)” documentation](https://oreil.ly/5Ji8p).

# Blazor WebAssembly App Internals

Every application has a required entry point. In a web client app, that is the initial request to the web server where the app is hosted. When the *_frame⁠work/​blazor.webassembly.js* file is running, it starts requesting .dlls, and the runtime starts the Blazor application’s executable. With Blazor WebAssembly, like most other .NET apps, the *Program.cs* is the entry point. [Example 2-1](#blazor_webassembly_program) is the Web.Client project’s *Program.cs* C# file of the model app.

##### Example 2-1\. Web.Client/Program.cs

[PRE4]

Blazor relies on dependency injection as a first-class citizen of its core architecture.

###### Note

*Dependency injection* (DI) is defined as an object declaring other objects as a dependency and a mechanism in which these dependencies are injected into the dependent object. A basic example of this would be `ServiceOne` requiring `ServiceTwo`, and a service provider instantiates `ServiceOne` given `ServiceTwo` as a dependency. In this contrived example, both `ServiceOne` and `ServiceTwo` would have to be registered with the service provider. `ServiceTwo` is instantiated by the provider and passed to `ServiceOne` as a dependency whenever `ServiceOne` is used.

The entry point is succinct and makes use of C#’s top-level program syntax, which requires less boilerplate, such as omitting a `class Program` object. We create a default `WebAssemblyHostBuilder` from the app’s `args`. The `builder` instance adds two root components: first, the `App` component paired with the `#app` selector, which will resolve our `<div id="app"></div>` element from the previously discussed *index.html* file. Second, we add a `HeadOutlet` component after the `<head>` content. This `Head​Out⁠let` is provided by Blazor, and it enables the ability to dynamically append or update `<meta>` tags or related `<head>` content to the HTML document.

When the application is running in a development environment, the minimum logging level is set appropriately to debug. The `builder` invokes `ConfigureServices`, which is an extension method that encapsulates the registration of various services the client app requires. The services that are registered include the following:

`ApiAccessAuthorizationMessageHandler`

The custom handler used to authorize outbound HTTP requests using an access token

`CultureService`

An intermediary custom service used specifically to encapsulate common logic related to the client `CultureInfo`

`HttpClient`

A framework-provided HTTP client configured with the culture services’ two-letter ISO language name as a default request header

`MsalAuthentication`

The framework-provided Azure business-to-consumer (B2C) and Microsoft Authentication Library (MSAL), which is bound and configured for the app’s tenant

`SharedHubConnection`

A custom service that shares a single SignalR `HubConnection` with multiple components

`AppInMemoryState`

A custom service used to expose in-memory application state

`CoalescingStringLocalizer<T>`

A generic custom service that leverages a component-first localization attempt, falling back to a shared approach

`GeoLocationService`

The custom client service for querying geographical information given a longitude and latitude

After all the services are registered, we call `builder.Build()`, which returns a `WebAssemblyHost` object, and this type implements the `IAsyncDisposable` interface. As such, we’re mindful to properly `await using` the `host` instance. This asynchronously uses the `host` and will implicitly dispose of it when it’s no longer needed.

## Detecting Client Culture at Startup

You may have noticed that the `host` had another extension method that was used. The `host.TrySetDefaultCulture` method will attempt to set the default culture. The `culture` in this context is represented by the [`CultureInfo`](https://oreil.ly/PrM7u) .NET object and acts as the locale of the browser, such as `en-US`, for example. The extension method is defined with the *WebAssemblyHostExtensions.cs* C# file of the Web.Client project:

[PRE5]

From the `host` instance, its `Services` property is available as an `IServiceProvider` type. This is exposed as `host.Services`, and we use it to resolve services from the DI container. This is referred to as the *service locator pattern* because services are located manually from a provider.

###### Tip

You don’t need to use this pattern elsewhere because .NET handles things. In the spirit of “best practices,” you should always prefer the framework-provided DI (or third-party) container implementations. We’re using it only because we want to load the application in a specific culture, which starts early.

We don’t need to use this pattern anywhere else in the app, as the framework will automatically resolve the services we need through either constructor or property injection. We start by calling for `ILocalStorageService`, described in [Chapter 7](ch07.html#chapter-seven). We then ask for it to retrieve a `string` value that corresponds to the `StorageKeys.ClientCulture` key. `StorageKeys` is a `static class` that exposes various literals, constants, and verbatim values that the app makes use of for consistency. If the `clientCulture` value is `null`, we’ll assign a reasonable default of `"en-US"`.

Since these `culture` values come from the client, we cannot trust it—this is why we wrap the attempt to create a `CultureInfo` in a `try`/`catch` block. Finally, we run the application associated with the contextual `host` instance. From this entry point, the `App` component is the first Blazor component that starts.

## Layouts, Shared Components, and Navigation

The *App.razor* file is the first of all Blazor components, and it contains the `<Router>`, which is used to provide data that corresponds to the navigation state. Consider the following *App.razor* file markup of the Web.Client project:

[PRE6]

This is the top-level Blazor component of the app itself and is named appropriately as `App`. The `App` component is the first component that is rendered. It is the root component of the application, and all child components of the app are rendered within this component.

### Blazor navigation essentials

Let’s evaluate the `App` component markup in-depth and understand the various parts.

The `<CascadingAuthenticationState>` component is the outermost component within our application. As the name implies, it will cascade the authentication state through to interested child components.

###### Note

The approach of *cascading* state through component hierarchies has become very common due to its ease of use and similarity to related patterns, like that of CSS. The same concept is also applicable at the OS level, with systems such as lightweight directory access protocol (LDAP) permissions. Try thinking in graphs because this is a common pattern in software whenever there is a graph/tree to cascade over. That’s the idea behind cascading state.

As an example, an ancestor component can define a `<CascadingValue>` component with any value. This value can flow down the component hierarchy to any number of descendant components. Consuming components use the `CascadingParameter` attribute to receive the value from the parent. This concept is covered in greater detail as we continue to explore the app. Let’s continue descending the component hierarchy.

The first nested child is the `Error` component. It’s a custom component that is defined in the *Error.razor* file:

[PRE7]

[![1](assets/1.png)](#co_executing_the_app_CO1-1)

The `@inject` syntax is a Razor directive.

[![2](assets/2.png)](#co_executing_the_app_CO1-2)

The `Error` component makes use of *cascades*.

[![3](assets/3.png)](#co_executing_the_app_CO1-3)

An `@code` directive is a way to add C# class-scoped members to a component.

[![4](assets/4.png)](#co_executing_the_app_CO1-4)

The `ChildContent` property is a parameter.

[![5](assets/5.png)](#co_executing_the_app_CO1-5)

The `ProcessError` method is accessible to all of the consuming components.

There are several common directives that you’ll learn as part of Blazor development. This specific directive instructs the Razor view engine to *inject* the `ILogger<Error>` service from the service provider. This is how Blazor components use DI, through property injection instead of constructor injection.

The `<CascadingValue>` markup includes the template rendering of an `@Child​Con⁠tent`. The `ChildContent` is both a `[Parameter]` and a `RenderFragment`. This allows for the `Error` component to render any child content, including Blazor components. When there is a single `RenderFragment` defined as part of a templated component, its child content can be represented as pure Razor markup.

A `RenderFragment` is a void returning `delegate` type that accepts a `RenderTreeBuilder`. It represents a segment of UI content. The `RenderTreeBuilder` type is a low-level Blazor class that exposes methods for building a C# representation of the DOM.

`<CascadingValue>` is a Blazor (or framework-provided) component that provides a cascading value to all descendant components. `CascadingValue.Value` is assigned `this`, which is the `Error` component instance itself. This means that all descendant components will have access to the `ProcessError` method if they choose to consume it. Descendant components would need to define a `[CascadingParameter]` property of type `Error` for it to flow through to it.

The `Parameter` attribute is provided by Blazor as a way to denote that a property of a component is a parameter. These are available as binding targets from consuming components as attribute assignments in Razor markup.

The `ProcessError` method expects an `Exception` instance, which it uses to log as an error. The child content of the `Error` component is the `Router`. The `Router` component is what enables our SPA’s client-side routing, meaning routing occurs on the client, and the page doesn’t need to refresh.

### The Router

The `Router` is a framework-provided component that’s used in the *src/Web.Cli⁠ent/​App.razor* file. It specifies an `AppAssembly` parameter that is assigned to the `typeof(App).Assembly` by convention. Additionally, the `Context` parameter allows us to specify the name of the parameter. We assign the name of `routeData`, which overrides the default name of `context`. The `Router` also defines multiple named `RenderFragment` components; because there are multiple, we must explicitly specify child content. This is where the corresponding `RenderFragment` name comes in. For example, when the router is unable to find a matching route, we define what the page should render the `NotFound` content. Consider the following `NotFound` content section from the `Router` markup:

[PRE8]

This layout is based on the `MainLayout` component and sets its child as the `NotFoundPage` component. Assuming the user navigates to a route that doesn’t exist, they’d end up on our custom HTTP 404 page, which is localized and styled consistently with our app. We’ll handle HTTP status code 401 in the next section. However, when the router does match an expected route, the `Found` content is rendered. Consider the following `Found` content section from the `Router` markup:

[PRE9]

### Redirect to login when unauthenticated

If you recall from earlier, the `Found` content is just a `RenderFragment`. The child content, in this case, is the `AuthorizeRouteView` component. This route view is displayed only when the user is authorized to view it. It adheres to `MainLayout` as its default. The `RouteData` is assigned from the contextual `routeData`. The route data itself defines which component the router will render and the corresponding parameters from the route.

When the user is not authorized, we redirect them to the login screen using the `RedirectToLogin` component:

[PRE10]

[![1](assets/1.png)](#co_executing_the_app_CO2-1)

The `RedirectToLogin` component requires the `NavigationManager`.

[![2](assets/2.png)](#co_executing_the_app_CO2-2)

The `OnInitialized` method navigates to the authentication login page.

The `RedirectToLogin` component injects the `NavigationManager`, and when it’s initialized, it navigates to the `authentication/login` route with the escaped `returnUrl` query string. When the user is authorized, the route view renders `MainLayout`, which is a subclass of Blazor’s `LayoutComponentBase`. While the markup defines all of the layout of the app, it also splats `@Body` in the appropriate spot. This is another `RenderFragment` that is inherited from `LayoutComponentBase`. The body content is what the router ultimately controls for its client-side routing. In other words, as users navigate the site, the router dynamically updates the DOM with rendered Blazor components within the `@Body` segment.

We `override` the `OnInitialized` method. This is our first look at overriding `ComponentBase` class functionality, but this is very common in Blazor. There are several `virtual` methods (methods that can be overridden) in the `ComponentBase` class, most of which represent different points of a component’s lifecycle.

### Blazor component lifecycle

Continuing from the aforementioned *RedirectToLogin.razor.* file, there is an `override` method named `OnInitialized`. This method is one of several lifecycle methods that will occur at specific points in the life of a component. Blazor components inherit the `Microsoft.AspNetCore.Components.ComponentBase` class. Please consider [Table 2-1](#component_lifecycle_methods) for reference.

Table 2-1\. `ComponentBase` lifecycle methods

| Order | Method name(s) | Description |
| --- | --- | --- |
| 1 | `SetParametersAsync` | Sets parameters supplied by the component’s parent in the render tree |
| 2 | `OnInitialized` `OnInitializedAsync` | Method invoked when the component is ready to start, having received its initial parameters from its parent in the render tree |
| 3 | `OnParametersSet` `OnParametersSetAsync` | Method invoked when the component has received parameters from its parent in the render tree and the incoming values have been assigned to properties |
| 4 | `OnAfterRender` `OnAfterRenderAsync` | Method invoked after each time the component has been rendered |

### The MainLayout component

The *MainLayout.razor* file, as the name indicates, represents the main layout. Within this markup, the navigation bar (navbar), header, footer, and content areas are organized and structured:

[PRE11]

[![1](assets/1.png)](#co_executing_the_app_CO3-1)

The first two lines of this layout Razor file are two C# expressions, indicated by their leading `@` symbol.

[![2](assets/2.png)](#co_executing_the_app_CO3-2)

`<section>` is a native HTML element, and it’s perfectly valid with Razor syntax.

[![3](assets/3.png)](#co_executing_the_app_CO3-3)

Within the `<section>` element’s semantic header and navbar, `<NavLink>` is referenced.

[![4](assets/4.png)](#co_executing_the_app_CO3-4)

The next section of the navbar is built out, with a custom `NavBar` component.

[![5](assets/5.png)](#co_executing_the_app_CO3-5)

The `@Body` render fragment is defined within the center of the DOM.

[![6](assets/6.png)](#co_executing_the_app_CO3-6)

The native HTML `footer` element is the parent to the custom `PageFooter` component, which is responsible for rendering the very bottom of the page.

These two directives represent various behaviors required within this component. The first of the two is the `@inherits` directive, which instructs the component to inherit from the `LayoutComponentBase` class. This means that it’s a subclass of the framework’s `LayoutComponentBase` class. This layout base class is an implementation of `IComponent` and exposes a `Body` render fragment. This allows us to make the content whatever the app’s `Router` provides as output. The main layout component uses the `@inject` directive to request the service provider to resolve an `IString​Lo⁠calizer<MainLayout>`, which is assigned to a component-accessible member named `Localizer`. We’ll cover localization in [Chapter 5](ch05.html#chapter-five).

`<section>` is a native HTML element, and it’s perfectly valid Razor syntax. Notice how we can transition from C# to HTML seamlessly, in either direction. We define some standard HTML, with a bit of semantic markup. It’s known that you have familiarity with HTML and CSS, and we won’t put too much emphasis on that. Because this is such a large project, we’d likely have this HTML and CSS provided by our imaginary UX department.

Within the `<section>` element’s semantic header and navbar, `<NavLink>` is referenced. This is a framework-provided component. The `NavLink` component is used to expose the user interactive aspect of the component’s logic. It handles the routing of the Blazor application and relies on the value within the browser’s URL bar. This represents the app’s “Home” navigation route, and it’s branded with the Blazor logo.

The next section of the navigation bar is built out with a custom `NavBar` component. There is a bit of familiar protective markup, where the app specifies it’s available only when `AuthorizerView` has `Authorized` content to render in the browser. The earlier components mentioned were either left-aligned or centered, and the next components are grouped and pushed to the end or far-right-hand side of the navbar. Immediately to the right of this component grouping is a `LoginDisplay`. Let’s have a deep look into the `LoginDisplay` component (see also [“Understanding the LoginDisplay component”](#login_display)). This group of elements is theme-aware, meaning it will render in one of two ways, either the `dark` theme or the `light` theme (see [“The header and footer components”](#header-footer-components) for visual examples).

The `@Body` render fragment is defined within the center of the DOM. `@Body` is the primary aspect of the Blazor navigation and the output target for the router. In other words, as users navigate, the client-side routing renders HTML within the `@Body` placeholder.

### The custom NavBar component

Admittedly, there’s a lot to soak in from that layout component markup, but when you take the time to mentally parse each part, it will make sense. There are a few custom components, one of which is `<NavBar />`. This references the *NavBar.razor* file:

[PRE12]

[![1](assets/1.png)](#co_executing_the_app_CO4-1)

Inherits from `LocalizableComponentBase` to take advantage of the base functionality.

[![2](assets/2.png)](#co_executing_the_app_CO4-2)

The `<NavLink>` component is provided by the framework and works with the router.

[![3](assets/3.png)](#co_executing_the_app_CO4-3)

The second route is for tweets and corresponds to the `/tweets` route.

[![4](assets/4.png)](#co_executing_the_app_CO4-4)

The third route is for Pwned? and corresponds to the `/pwned` route.

Like most custom components, this too inherits from `LocalizableComponentBase` to take advantage of the base functionality. The base functionality is detailed in [Chapter 5](ch05.html#chapter-five). The framework-provided `<NavLink>` component works with the router. The first route is the chat room and corresponds to the `/chat` route. While each of the previous route names is retrieved using the `@Localizer` indexer, the “Pwned?” route is not because it’s a brand name.

### The header and footer components

The header for the app contains links to Home, Chat, Tweets, Pwned, and Contact pages. These are all navigable routes that the `Router` will recognize. The icons to the right are for Theme, Audio Descriptions, Language Selection, Task List, Notifications, and Log out. The log-out functionality does rely on the app’s navigation to navigate to routes, but the other buttons could be considered utilitarian. They open modals for global functionality and expose user preferences. The header itself supports the `dark` and `light` themes, as shown in Figures [2-2](#dark_navbar) and [2-3](#light_navbar).

![](assets/lblz_0202.png)

###### Figure 2-2\. An example navigation header with the `dark` theme

![](assets/lblz_0203.png)

###### Figure 2-3\. An example navigation header with the `light` theme

Let’s look at the `PageFooter` component first, defined in the *PageFooter.razor* file:

[PRE13]

[![1](assets/1.png)](#co_executing_the_app_CO5-1)

The component inherit from the `LocalizableComponentBase` class.

[![2](assets/2.png)](#co_executing_the_app_CO5-2)

Column one reads `"Learning Blazor by David Pine"`.

[![3](assets/3.png)](#co_executing_the_app_CO5-3)

In the second column, there are two links: one for the source codes’ MIT license and the GitHub source code link.

[![4](assets/4.png)](#co_executing_the_app_CO5-4)

The third column contains links to the *Privacy* and *Terms and Conditions* pages.

[![5](assets/5.png)](#co_executing_the_app_CO5-5)

The last column contains the .NET runtime version that the client browser is running.

We are establishing a pattern, by having custom components inherit from the `LocalizableComponentBase` common base class. The custom `PageFooter` component is written by defining a four-column layout with centered text. From left to right starting at column one, the name of the application appears and a byline, `"Learning Blazor by David Pine"`, is rendered with a nontranslatable bold phrase. The second column links to the source codes’ MIT license and the GitHub source code link. The third column contains links to the *Privacy* and *Terms and Conditions* pages, and the text is localized. Localization of Blazor apps is covered in [Chapter 5](ch05.html#chapter-five). The .NET runtime version is useful because it tells the developer immediately what version of the framework is being used.

More often than not, I prefer to have my Razor markup files accompanied by a code-behind file. In this way, the separate files help to isolate concerns where markup exists in Razor and logic exists in C#. For simple components, components that have a few parameters, and markup elements, it’s fine to just have everything in a Razor file with an `@code` directive. But when using the code-behind approach, you might think of this as *component shadowing*, as the component’s markup is shadowed by a C# file from the Visual Studio editor, as shown in [Figure 2-4](#component_shadowing).

###### Note

Component shadowing is the act of creating a C# file that matches the name of an existing Razor file but appends the *.cs* file extension. For example, the *PageFooter.razor* and *PageFooter.razor.cs* files exemplify component shadowing because they’re nested in the Visual Studio editor and together they both represent the `public partial PageFooter class`.

![](assets/lblz_0204.png)

###### Figure 2-4\. Component shadowing in Visual Studio Solution Explorer

Consider the *PageFooter.razor.cs* component shadow file:

[PRE14]

[![1](assets/1.png)](#co_executing_the_app_CO6-1)

Several constants are defined.

[![2](assets/2.png)](#co_executing_the_app_CO6-2)

The `OnInitialized` lifecycle method assigns the framework description.

There are several `const string` fields defined that contain URL literals. These are used to bind to the Razor markup. We `override` the `OnInitialized` lifecycle method and assign the `_frameDescription` value from the inherited `LocalizableComponentBase.AppState` variable.

The component is also theme-aware of the client browser preferences for either `light` or `dark`. For example, see Figures [2-5](#dark_footer) and [2-6](#light_footer).

![](assets/lblz_0205.png)

###### Figure 2-5\. An example footer with the `dark` theme

![](assets/lblz_0206.png)

###### Figure 2-6\. An example footer with the `light` theme

The footer doesn’t strive for too much. It’s intentionally simple, providing only a few links to relevant information for the app.

The `MainLayout` component is more than just Razor markup; it, too, has a shadowed component. Consider the *MainLayout.razor.cs* file:

[PRE15]

[![1](assets/1.png)](#co_executing_the_app_CO7-1)

`MainLayout` is a `sealed partial class`.

[![2](assets/2.png)](#co_executing_the_app_CO7-2)

The `AppInMemoryState` instance is injected into the component.

[![3](assets/3.png)](#co_executing_the_app_CO7-3)

The `OnInitialized` method is overridden to allow the subscription to the `AppInMemoryState.StateChanged` event.

[![4](assets/4.png)](#co_executing_the_app_CO7-4)

The `Dispose` method unsubscribes from the `AppInMemoryState.StateChanged` event.

You’ll notice that `MainLayout` is a `sealed partial class`; it’s `partial` so that it can serve as code-behind to the Razor markup, and it’s `sealed` so that it’s not inheritable by other components. It implements the `IDisposable` interface to perform necessary cleanup. Let’s ensure that we’re following the concepts of *component shadowing* and *component inheritance*.

The `AppInMemoryState` instance is injected into the component. This application state object is in-memory only; if the user refreshes the page, the state is lost.

The `OnInitialized` method is overridden from the base, and it’s used to subscribe to the `AppInMemoryState.StateChanged` event. The event handler is the framework-provided `ComponentBase.StateHasChanged` method. Eventing is a common idiom of C#, and it can be very useful. The `StateHasChanged` method notifies the component that its state has changed. When applicable, this will cause the component to be rerendered. `AppState.FrameworkDescription` is assigned from `Runtime​Informa⁠tion.FrameworkDescription`. This is the value that was displayed in the right-hand column of the footer, such as “.NET 6.”

###### Tip

The `StateHasChanged` method should be called only when required to avoid potentially unnecessarily forcing a component to rerender. When calling this method in an asynchronous context, wrap it in an `await` statement passing it into the `InvokeAsync` method. This will execute the supplied work item on the associated renderer’s synchronization context, ensuring it’s executed on the appropriate thread.

You may need to explicitly call `StateHasChanged` in the following conditions:

*   An asynchronous handler involves multiple asynchronous phases.

*   The Blazor rendering and event-handling system receives a call from something external.

*   You need to render a component outside the subtree that is rerendered by a particular event.

For more information about triggering a render, see Microsoft’s [“ASP.NET Core Razor Component Rendering” documentation](https://oreil.ly/Kt3cm).

The `Dispose` method ensures that if the `AppState` instance `is not null`, it will unsubscribe from the `AppInMemoryState.StateChanged` event. This kind of explicit cleanup helps to ensure that the component will not cause a memory leak due to event handlers not being unsubscribed.

### An in-memory app state model

Blazor apps can store their state using an in-memory methodology. In this approach, you register your app state container as a singleton service, meaning there’s only one instance for the app to share. The service itself exposes an event that subscribes to the `StateHasChanged` method, and as properties on the state object are updated, they fire the event. Consider the *AppInMemoryState.cs* C# file:

[PRE16]

[![1](assets/1.png)](#co_executing_the_app_CO8-1)

These fields and properties represent the various app states.

[![2](assets/2.png)](#co_executing_the_app_CO8-2)

We will render the `FrameworkDescription` property.

[![3](assets/3.png)](#co_executing_the_app_CO8-3)

The `AppStateChanged` method is called.

[![4](assets/4.png)](#co_executing_the_app_CO8-4)

There is an `Action` field named `StateChanged`.

[![5](assets/5.png)](#co_executing_the_app_CO8-5)

The `AppStateChanged` method invokes the `StateChanged` event.

Several backing fields are declared, which will be used to store the values of various publicly accessible properties that represent various app states.

As an example of the pattern used to communicate app state changes, consider the `FrameworkDescription` property. Its `get` accessor goes to the backing field, and the `set` accessor assigns to the backing field.

After the `value` has been assigned to the backing field, the `AppStateChanged` method is called. This pattern is followed for all properties and their corresponding backing fields.

The class exposes a nullable `Action` as an `event` named `StateChanged`. Interested parties can subscribe to this for change notifications.

The `AppStateChanged` method is expressed as the invocation of the `StateChanged` event. It’s conditionally a NOOP (or “no operation”) when the event is `null`.

This in-memory state management mechanism is used to expose client voice preferences, whether or not the client is preferring the dark theme, and the value for the framework description. To have the application state persist across browser sessions, you’d use an alternative approach, such as local storage. There are trade-offs in each approach; while using in-memory app state is less work, it will not persist beyond browser sessions. To persist beyond browser sessions, you rely on JavaScript interop to use a local storage mechanism.

###### Note

If you’re a JavaScript SPA developer, you might be familiar with the *Flux pattern*. It was introduced by Facebook to provide a clear separation of concerns. The pattern grew in popularity with the React Redux project, which is a JavaScript implementation of the Flux pattern used in React. There is an implementation of this for Blazor known as [Fluxor](https://oreil.ly/nI5v3) by Peter Morris. While it is beyond the scope of this book, it’s worth exploring as a potential in-memory state management option.

### Understanding the LoginDisplay component

The `LoginDisplay` component renders only a few things to the HTML, but there’s a bit of code to understand:

[PRE17]

[![1](assets/1.png)](#co_executing_the_app_CO9-1)

The component defines two directives.

[![2](assets/2.png)](#co_executing_the_app_CO9-2)

The component markup uses the framework-provided `AuthorizeView` component.

The component defines two directives: one to specify that it *inherits* from `LocalizableComponentBase` and one to *inject* the `SignOutSessionStateManager` service. `LocalizableComponentBase` is a custom base component, which is covered in [Chapter 5](ch05.html#chapter-five).

The component markup uses the `AuthorizeView` component and its various authorized-state-dependent templates to render content when the user is currently authorizing, already authorized, or not authorized. Each of these states has independent markup.

When authorizing, the “logging in” message is localized and rendered to the screen. When the user is authorized, the `context` exposes the `ClaimsPrincipal` object that’s assigned to the `user` variable. Consider the `Localizer` object from the previous markup. This specific type comes from the inheritance of the custom `LocalizableComponentBase<LoginDisplay>` class. This `Localizer` exposes localization functionality that is based on Microsoft’s resource (*.resx*)-driven key/value pairs (KVPs) and the frameworks’ `IStringLocalizer<T>` type. The custom *LocalizableComponent​Base.cs* class is located in the *Components* directory.

The code creates a tool tip it will render—the string concatenation of the user’s name and email address. The tool tip is bound to the button element’s `data-tooltip` attribute. This is part of the Bulma CSS framework for tool tips. Hovering over the logout button will render the message. When the user is not authorized, we render a button with a localized login message.

Next, let’s take a look at its shadowed component, the *LoginDisplay.cs* file:

[PRE18]

This component provides two functions that use the injected `Navigation` service. The `Navigation` property is assigned by the DI framework and is functionally equivalent to the component’s `@inject` directive syntax. Each method navigates to the desired authentication route. When `OnLogOut` is invoked, `SignOutManager` has its sign-out state set before navigating away. Each route is handled by the app’s corresponding authentication logic. The user will see their name next to a logout button when they’re authenticated, but if they’re not authenticated, they will see only a login button. Users can sign up with the application by providing and validating their email. This is managed by Azure Active Directory (Azure AD) business-to-consumer (B2C). As an alternative to signing up with the application, you can use one of the available third-party authentication providers, such as Google, Twitter, and GitHub.

### Native theme awareness

An app’s ability to be color-scheme aware is highly recommended for all modern web apps. From CSS, it is easy to specify style rules that are scoped to media-dependent queryable values. Consider the following CSS:

[PRE19]

The rules within this media query apply only when the browser is set to prefer the `dark` theme. These media queries can also be accessed programmatically from JavaScript. The [`window.matchMedia` method](https://oreil.ly/uFPAD) is used to detect changes to the client browser preferences. Let’s look first at the *ThemeIndicator​Com⁠ponent.razor* file:

[PRE20]

[![1](assets/1.png)](#co_executing_the_app_CO10-1)

Inherits from the generic `LocalizableComponentBase` class.

[![2](assets/2.png)](#co_executing_the_app_CO10-2)

The primary markup for `ThemeIndicatorComponent` is the button.

[![3](assets/3.png)](#co_executing_the_app_CO10-3)

`ThemeIndicatorComponent` makes use of `<HeadContent>`.

Hopefully, you’re noticing that a lot of components are inheriting from the generic `LocalizableComponentBase` class. Again, we’ll cover this in [Chapter 5](ch05.html#chapter-five). Just know that it exposes a `Localizer` member that lets us get a localized string value given a string key via a free-range indexer.

The primary markup for `ThemeIndicatorComponent` is the button. The button’s `class` attribute is mixed, with verbatim class names and Razor expressions that are evaluated at runtime. The `_buttonClass` member is a C# string field that is bound to the `"is-"` prefix. This button also has a tool tip, and its message is conditionally assigned dependent on the ternary expression from the `_isDarkTheme` boolean value. The Font Awesome class is also bound to an `_iconClass` field member.

`ThemeIndicatorComponent` makes use of `<HeadContent>`. This is a framework-provided component that allows us to dynamically update the HTML’s `<head>` content. It’s very powerful and useful for updating `<meta>` elements on the fly. When the theme is `dark`, the app specifies that the Twitter widgets should also be themed accordingly.

###### Note

While the `HeadContent` component can update `meta` tags, it’s still not ideal for search engine optimization (SEO) when using Blazor WebAssembly. This is because the `meta` tags are updated dynamically. To achieve static `meta` tag values, you’d have to use Blazor WebAssembly prerendering. For more information about component integration scenarios, see Microsoft’s [“Prerender and Integrate ASP.NET Core Razor Components” documentation](https://oreil.ly/NmB4A).

Next, let’s look at its corresponding component shadow, the C# file *ThemeIndicator​Component.razor.cs*:

[PRE21]

[![1](assets/1.png)](#co_executing_the_app_CO11-1)

The `ThemeIndicatorComponent` component shadow is defined.

[![2](assets/2.png)](#co_executing_the_app_CO11-2)

There are a few conditional CSS classes that are bound to field values.

[![3](assets/3.png)](#co_executing_the_app_CO11-3)

The component overrides `OnInitializedAsync`, where it performs a bit of app state theme logic.

[![4](assets/4.png)](#co_executing_the_app_CO11-4)

The callback method named `UpdateDarkThemePreference`.

`ThemeIndicatorComponent` is a read-only indicator of the current theme detected. There are only two types the app supports: Light and Dark. There are a few private fields, but you’ll recall that these are accessible to the markup and bound where needed. These two `string` fields are simple ternary expressions based on the `AppState.IsDarkTheme` value. The component overrides `OnInitializedAsync`, where it assigns the current state of the `AppState.IsDarkTheme` variable and calls the `Get​Cur⁠rentDarkThemePreference` method, which is an `IJSRuntime` extension method. This method requires the `ThemeIndicatorComponent` to reference itself and the callback method name. C#’s `nameof` expression produces the name of its argument, which in this case is the callback. This means that we’re registering our .NET component to receive a callback from the JavaScript side given a .NET object reference.

The callback method named `UpdateDarkThemePreference` expects the `isDarkTheme` value. The method must be decorated with the `JSInvokable` attribute for it to be callable from JavaScript. Since this callback can be invoked anytime after the component is initialized, it must use the combination of `InvokeAsync` and `StateHasChanged`:

`InvokeAsync`

Executes the supplied work item on the associated renderer’s synchronization context.

`StateHasChanged`

Notifies the component that its state has changed. When applicable, this will cause the component to be rerendered.

Let’s now consider the following *JSRuntimeExtensions.cs* C# file for the `GetCurrentDarkThemePreferenceAsync` extension method:

[PRE22]

[![1](assets/1.png)](#co_executing_the_app_CO12-1)

The `dotnetObj` parameter is generic and constrained to `class`.

[![2](assets/2.png)](#co_executing_the_app_CO12-2)

The `javaScript` runtime instance calls interop methods.

[![3](assets/3.png)](#co_executing_the_app_CO12-3)

The `"app.getClientPrefersColorScheme"` method is called.

[![4](assets/4.png)](#co_executing_the_app_CO12-4)

An argument with a value of `"dark"` is passed to the `"app.getClientPrefers​Co⁠lorScheme"` method.

[![5](assets/5.png)](#co_executing_the_app_CO12-5)

`DotNetObjectReference.Create(dotnetObj)` creates an instance of `DotNet​Ob⁠jectReference<ThemeIndicatorComponent>`.

[![6](assets/6.png)](#co_executing_the_app_CO12-6)

`callbackMethodName` is the calling method name.

The extension method defines a generic type parameter, `T`, which is constrained to a `class`. The object instance, in this case, is `ThemeIndicatorComponent`, but it could be any `class`.

The `javaScript` runtime instance is used to call a `ValueTask<bool>` returning interop function. The `"app.getClientPrefersColorScheme"` method is a JavaScript method that is accessible on the `window` scope.

The hardcoded value of `"dark"` is passed to the `app.getClientPrefersColorScheme` function as the first parameter. It’s hardcoded because we know that we’re trying to evaluate whether or not the current client browser prefers the dark theme. When they do, this will return `true`.

`DotNetObjectReference.Create(dotnetObj)` creates an instance of `DotNetObject​Re⁠ference<ThemeIndicatorComponent>`, and this is passed to the corresponding JavaScript function as the second parameter. This is used as a reference so that JavaScript can call back into the .NET component.

`callbackMethodName` is a method name from the calling `ThemeIndicatorComponent` instance that is decorated with a `JSInvokable` attribute. This method can and will be called from JavaScript when needed.

Considering this is your first look at JavaScript interop, let me anticipate and answer a few questions you may have:

Question

Where is this JavaScript coming from, and what does it look like?

Answer

This JavaScript is part of the *app.js* file that was referenced in the *index.html*. It’s served under the *wwwroot* folder. We’ll look at the source in the next section.

Question

What capabilities does it have?

Answer

That depends on what you’re looking to achieve, but really, anything you might imagine. For this specific use case, the JavaScript will expose a utilitarian helper function named `getClientPrefersColorScheme`. Internally, JavaScript relies on the `window.matchMedia` APIs. The .NET code makes an interop call into JavaScript and passes a component reference. The JavaScript code registers an event handler to monitor whether the user changes their color scheme preference. The current preference is returned immediately back to .NET from JavaScript, but the event handler is still registered. If the user preference changes, using the given component reference, the JavaScript code will make an interop call back into .NET with the new color scheme preference. This exemplifies bidirectional interop.

Question

When do I need to write JavaScript interop code?

Answer

Whenever you need finite control over a sequence of JavaScript APIs. A good example is when you need to interact with a third-party library or when calling native JavaScript APIs. You’ll see some good examples for when it’s appropriate to write JavaScript interop code throughout the book.

###### Warning

Blazor is responsible for manipulating the DOM. If Blazor doesn’t support the DOM manipulation your app requires, you might need to write JavaScript interop code to achieve the desired behavior. However, this should be rare. Ideally, you’d avoid having two different approaches to the same problem.

This specific JavaScript API uses the media query APIs, which are native to JavaScript. Consider the *app.js* JavaScript file:

[PRE23]

[![1](assets/1.png)](#co_executing_the_app_CO13-1)

Consider the `getClientPrefersColorScheme` function.

[![2](assets/2.png)](#co_executing_the_app_CO13-2)

A `media` instance is assigned from the call to `window.matchMedia`.

[![3](assets/3.png)](#co_executing_the_app_CO13-3)

The `media.onchange` event handler property is assigned to an inline function.

[![4](assets/4.png)](#co_executing_the_app_CO13-4)

When the `media` instance changes, the .NET object has its callback invoked.

[![5](assets/5.png)](#co_executing_the_app_CO13-5)

The `media.matches` value is returned.

[![6](assets/6.png)](#co_executing_the_app_CO13-6)

`getClientPrefersColorScheme` is added to the `window.app` object.

The `getClientPrefersColorScheme` function is defined as a `const` function with `color`, `dotnetObj`, and `callbackMethodName` parameters. A `media` instance is assigned from the call to `window.matchMedia`, given the media query string. The `media.onchange` event handler property is assigned to an inline function.

The event handler inline function relies on the `dotnetObj` instance, which is a reference to the calling Blazor component. This is JavaScript interop in which JavaScript calls into .NET. In other words, if the user changes their preferences, the `onchange` event is fired and the Blazor component has its `callbackMethodName` invoked.

The `media.matches` value is returned, indicating to the caller the current value of whether the media query string matches. `getClientPrefersColorScheme` is added to the `window.app` object.

Putting all of this together, you can reference `<ThemeIndicatorComponent />` in any Blazor component, and you’d have a self-contained, color scheme–aware component. As the client preferences change, the component dynamically updates its current rendered HTML representation of the color scheme. The component relies on JavaScript interop, and it’s seamless from C#.

# Summary

In this chapter, I guided you through the inner workings of how Blazor Web­Assem⁠bly starts. From the serving and processing of static HTML to the invocation of JavaScript that bootstraps Blazor, you explored the anatomy of the app. This includes the `Program` entry point and the startup conventions. You learned about the router, client-side navigation, shared components, and layouts. You also learned about top-level navigation components in the app and how content is rendered through `RenderFragment` placeholders. The model app demonstrated a native color scheme–aware component and an example of JavaScript interop. In the next chapter, you’ll see how to author custom Blazor components and how to use JavaScript interop. You’ll learn more about how Blazor uses authentication to verify a user’s identity and how to conditionally render markup. Finally, you’ll see how to use various data-binding techniques and rely on data from HTTP services.