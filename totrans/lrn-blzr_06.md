# Chapter 5\. Localizing the App

In this chapter, I’m going to show you how to localize Blazor WebAssembly apps. Using the Learning Blazor App as an example, I’ll show you how an app can be automatically localized into dozens of languages. You’ll see how Blazor WebAssembly recognizes static resource files for the client browser’s corresponding language. You will also learn how to consume the framework-provided `IStringLocalizer<T>` interface type. Additionally, I’ll show you one possible way to machine translate static files at rest with a GitHub Action using the [Azure Cognitive Services Translator](https://oreil.ly/4RTN0).

We live in a global society, and an application that speaks to one group of people is a disappointment. Not only will this dramatically affect the UX for those who do not speak the app’s language, but if the app contributes to an online shopping experience, for example, it will have a detrimental effect on sales as well. This is where localization comes in.

# What Is Localization?

*Localization* is the act of translating static resources, such as those found in resource files, into a specific language that an app plans to support. When your app supports many languages, it will have various resource files for each supported locale. In .NET, localization maintains locale-specific resource files in an XML format with the *.resx* file extension.

###### Note

Localization is not the same thing as globalization. Globalization is when you code your app in a way that makes it easy to localize. For an overview of globalization, see Microsoft’s [“Globalization” .NET documentation](https://oreil.ly/cRRVw).

The Learning Blazor app supports roughly 40 languages. Supporting these languages is possible with the help of AI. As an English-speaking developer, I write my resource files in English. This means the resources filenames end with *.en.resx* and the other supported locales are machine-translated as an automated pull request. You’ll learn how you can use this functionality in your apps later in this chapter.

As part of .NET, Blazor WebAssembly can dynamically determine which translated version of the file to pull resources from. The browser will determine the language it is using, and this information is available on the Web.Client app. Using the appropriate resource file, the app will render the correct content, following various numerical and date formatting rules. To support an app’s many languages is to *localize* it. For more information about localization in .NET, see Microsoft’s [“Localization in .NET” documentation](https://oreil.ly/Nm2vS).

###### Warning

Localizing an app using only machine-translated text is not ideal. Instead, developers should hire professional translators who can help maintain post-machine-translated files. This approach yields more reliable translations. They’re not free, but you get what you pay for. Machine translations are not always accurate, but they strive to be natural sounding and can accommodate simple user needs for limited text.

Localization is largely accomplished by using the app’s resource files. Resource (*.resx*) files have their language encoded as a subextension `.{lang-id}.resx`, where the `{lang-id}` placeholder is the browser’s specified language. The app exposes the language configuration through the `LanguageSelectionComponent`, which uses the `ModalComponent` to prompt the user to select from a list of languages that the app supports. These languages are accessible to the app through an `"api/cultures/all"` endpoint.

# The Localization Process

Let’s prepare to localize our Learning Blazor sample app. To localize any Blazor WebAssembly app, you’ll need the following:

*   A client reference to the `Microsoft.Extensions.Localization` NuGet package

*   The ability to call `AddLocalization()` when registering services for DI

*   The ability to update culture based on user preferences and at app startup, as shown in [“Detecting Client Culture at Startup”](ch02.html#detect-client-culture)

*   Resource files available to the Web.Client project

*   `IStringLocalizer<T>` instances injected into components where localization is used

*   An opportunity to call upon the localizer instances through their indexer method APIs

Blazor relies on `CultureInfo.DefaultThreadCurrentCulture` and `Cultur⁠eInfo​.DefaultThreadCurrentUICulture` values to determine which resource file to use.

Let’s take a moment to understand how this process comes together. The Blazor app needs to register the localization services. When the Web.Client project starts, all services it relies on are registered as being discoverable through the framework-provided DI service provider. Each client app instance makes use of the internal HTTP clients and business logic services, with one, in particular, coming from the [`Microsoft.Extensions.Localization` NuGet package](https://oreil.ly/4olfW). This package contains the services required to use localization. Recall from [“The Web.Client ConfigureServices Functionality”](ch04.html#web-client-configure-services) that when setting up `IServiceCollection`, we made a call to `AddLocalization()`. This method, from the Localization NuGet package, adds the `IStringLocalizer<T>` service types to the client app’s DI container.

With the `IStringLocalizer<T>` type, a component can use the resources within the translation files. Each Blazor component potentially has many resource files that correspond to it. An instance of `IStringLocalizer<T>` corresponds to a single type of `T`, where `T` is any type that might have resources.

You can use a shared object (`SharedResource`) that contains resources with these common values. When you’re using `IStringLocalizer<T>` and `IStringLocal⁠izer​<SharedResource>`, it becomes redundant to inject both of these types over and over again. To solve this redundancy, a custom `CoalescingStringLocalizer<T>` service exists to coalesce these multiple localizer types, favoring the type of `T` and coalescing to the `SharedResource` type when a value is not found. Examples of common text would be the text of command-centric buttons on the UI, like “Okay” or “Cancel.” This approach could be used in other Blazor apps, or any localized .NET app, for that matter. Consider the following *CoalescingStringLocalizer.cs* C# file:

[PRE0]

[![1](assets/1.png)](#co_localizing_the_app_CO1-1)

The `CoalescingStringLocalizer<T>` object relies on two fields:

*   `_localizer`: The localizer for the `T` type, where `T` is a component

*   `_sharedLocalizer`: The localizer for the `SharedResource` type

[![2](assets/2.png)](#co_localizing_the_app_CO1-2)

The constructor requires both localizer instances, and they’re assigned to the class-scoped fields.

[![3](assets/3.png)](#co_localizing_the_app_CO1-3)

The first of two indexers accepts the `name` of the resource and coalesces on both localizer instances. When not found, the given `name` is returned.

[![4](assets/4.png)](#co_localizing_the_app_CO1-4)

The second indexer accepts the `name` of the resource and the `arguments`. It too coalesces on both localizer instances and returns the given `name` when no resource is found.

`CoalescingStringLocalizer<T>` is used throughout the Web.Client project of our Learning Blazor app and is injected into `LocalizableComponentBase<T>`. Components that inherit from the `LocalizableComponentBase<T>` type will have access to the `Localizer` property. `LocalizableComponentBase<T>` is a descendant of the framework-provided `ComponentBase` class. `LanguageSelectionComponent<T>` provides a great example for binding to `Localizer`, and this component is responsible for exposing the client language configuration. In the next section, we’ll explore how this component binds localized content and lets the user choose the app’s language.

# The Language Selection Component

While exposing the ability for the user to select the app’s language isn’t specifically part of the *localization process*, it’s an important feature to provide. When localizing apps, you should be mindful to include such functionality.

The language selection component prompts the user for their desired language when the user selects the Language top-level navigation button. Its markup introduces a new framework-provided component used for handling errors, the `ErrorBoundary` component. Whenever you write code that doesn’t handle errors, for example, potentially errant code that’s not wrapped in a `try`/`catch` block, that code has the potential to negatively impact the component’s ability to render properly. As such, as an alternative to writing `try`/`catch`, you could handle errors by displaying error-specific markup. The `ErrorBoundary` component allows consumers to template both `ChildContent` for successful logic and `ErrorContent` when an error is thrown. This is useful for conditionally rendering content even if the component encounters an error. For example, if the endpoint that serves the app’s supported languages is unavailable, the `ErrorBoundary` component can render a disabled button.

Assuming no errors are present, the modal dialog acts as a user prompt. When `LanguageSelectionComponent` is displayed, clicking its `button` will show the modal dialog that renders similar to [Figure 5-1](#language-selection-component).

![](assets/lblz_0501.png)

###### Figure 5-1\. An example `LanguageSelectionComponent` rendering with the modal shown

Now, let’s look at the following *LanguageSelectionComponent.razor* markup file, which is responsible for rendering the modal dialog:

[PRE1]

[![1](assets/1.png)](#co_localizing_the_app_CO2-1)

An `ErrorBoundary` component is used to wrap the potentially errant component.

[![2](assets/2.png)](#co_localizing_the_app_CO2-2)

`ModalComponent` is used to render the modal dialog.

[![3](assets/3.png)](#co_localizing_the_app_CO2-3)

The body is an HTML `form` element.

[![4](assets/4.png)](#co_localizing_the_app_CO2-4)

`ButtonContent` renders both cancel and confirm buttons.

The `LanguageSelectionComponent` markup file starts with an `ErrorBoundary` component. Its `ChildContent` renders a `button` that binds its `onclick` event handler to the `ShowAsync` method. `ErrorContent` renders a disabled `button`. Both render fragments use the same syntax to call into the `LocalizableComponentBase.Localizer` instance. The `@Localizer["Language"]` invocation asks the localizer to fetch the corresponding value for the `"Language"` key. This returns a framework-provided `LocalizedString` type that represents a locale-specific `string`. The `LocalizedString` type defines an implicit operator as a `string`.

The localization services understand that for `IStringLocalizer<LanguageSelectionComponent>`, they should look for resources that match by naming convention. For example, the *LanguageSelectionComponent.razor* and *LanguageSelectionCompo⁠nent​.razor.cs* files are related, as they’re two `partial class` definitions for the same object. The same relationship exists for this component’s resource files. I defined a single *LanguageSelectionComponent.razor.en.resx* resource file for this, and that is shown later in [Example 5-1](#language-selection-component-en-resx).

`ModalComponent` is captured as a reference and assigned to the `_modal` field using the `@ref="_modal"` syntax. `BodyContent` contains a native HTML `form` element, and it binds to a native HTML `selection` element. Each `option` node is bound from the current `culture` in the iteration to the `value` attribute. It’s `selected` when the current culture’s Language Code Identifier (or `LCID`) matches the one being iterated over. A `ToDisplayName` helper method is used to convert the `culture` and `azureCulture` objects into their text representation.

`ButtonContent` defines two buttons. The first `button` is the `"Okay"` button that calls `ConfirmAsync` when clicked. The other `button` is the `"Cancel"` button, and when it’s clicked, it will call `_modal.CancelAsync()`.

When the user expands all of the supported cultures, the dialog will render similar to that shown in [Figure 5-2](#language-selection-component-expanded).

![](assets/lblz_0502.png)

###### Figure 5-2\. An example `LanguageSelectionComponent` rendering with an open modal dialog and culture selection expanded

###### Tip

At the time of writing this book, there was [a bug](https://oreil.ly/Gwu5T) concerning ASP.NET Core’s ability to locate resources when the component used a file-scoped namespace. As such, components that do not display any text or user inputs do not need to be localized. So they’re free to use file-scoped namespaces. You will see both namespace formats in the code, so don’t be alarmed.

The corresponding component partial code is reflected in the *LanguageSelection​Com⁠ponent.razor.cs* C# file. Let’s look at that next:

[PRE2]

[![1](assets/1.png)](#co_localizing_the_app_CO3-1)

Component state is managed by private fields.

[![2](assets/2.png)](#co_localizing_the_app_CO3-2)

The `OnInitializedAsync` method is used to fetch the supported cultures from the server.

[![3](assets/3.png)](#co_localizing_the_app_CO3-3)

The `ToDisplayName` helper method is used to convert the `culture` and `azure​Cul⁠ture` objects into their text representation.

[![4](assets/4.png)](#co_localizing_the_app_CO3-4)

Several methods expose `_modal` functionality to the component.

`LanguageSelectionComponent` defines a few fields and a few injected properties:

`_supportedCultures`

An `IDictionary<CultureInfo, AzureCulture>` field that represents the supported cultures. The field’s keys are the framework-provided `CultureInfo`, and their value is a custom `AzureCulture` positional `record class`.

`_selectedCulture`

This value is bound in the Razor markup to the `select` element and corresponds to what the user has selected as their desired culture.

`_modal`

The reference to `ModalComponent`. With this reference, we will call `ShowAsync` and `ConfirmAsync` to show and confirm the modal accordingly.

`Http`

A framework-provided `HttpClient` instance used to fetch the supported cultures.

`Navigation`

A framework-provided `NavigationManager` used to force reloading of the current page. When changing the culture, this is required to reload the entire app.

When the component is initialized (`OnInitializedAsync`), the `"api/cultures/all"` server endpoint is called. The `_supportedCultures` map is assigned from the returned values and a calculation of the intersecting supported cultures. These values reflect the set of overlapping client cultures and the server’s supported set, as shown in the example Venn diagram in [Figure 5-3](#supported-cultures-venn-diagram), where each small circle represents a two-letter language identifier.

![](assets/lblz_0503.png)

###### Figure 5-3\. Supported cultures are the intersection of the client and server cultures

The remaining methods rely on the `_modal` instance:

`ShowAsync`

Delegate to `_modal.ShowAsync()`.

`ConfirmAsync`

If the user has selected a different culture, a reload is forced, and the new value is persisted to local storage. The modal is closed by the call to `_modal.Confirm​A⁠sync()`.

`LanguageSelectionComponent` supports 41 languages. From the earlier markup shown in the *LanguageSelectionComponent.razor* file, you may have noticed that `@Localizer` has its indexer called with the given arguments:

`"Language"`

Bound in the `<button title=@Localizer["Language"]></button>` markup

`"ChangeLanguage"`

Bound to the `TitleContent` markup

`"Okay"`

Bound to the `ButtonContent` confirm button text

`"Cancel"`

Bound to the `ButtonContent` cancel button text

Each of these keys (or names) corresponds to the resource files of `Lan⁠guage​Selec⁠tionComponent`. Consider the *LanguageSelectionComponent.razor.en.resx* resource file, shown in [Example 5-1](#language-selection-component-en-resx).

##### Example 5-1\. Resource files for the `LanguageSelectionComponent`

[PRE3]

Each `data` node has a `name` attribute. This `name` matches the name you use when asking an `IStringLocalizer<T>` for a corresponding value. The `value` returned corresponds to the English version of the resource. Consider the *LanguageSelection​Component.razor.es.resx* resource file, shown in [Example 5-2](#language-selection-component-es-resx).

##### Example 5-2\. Web.Client/Components/LanguageSelectionComponent.razor.es.resx

[PRE4]

This resource file has a subextension of *.es.resx* instead of *.en.resx*, and each `value` is in Spanish. These resource files contain only two `data` nodes. There were two additional names referenced in the markup, and that’s where `CoalescingString​Lo⁠calizer<T>` comes in. The `"Okay"` and `"Cancel"` resources are part of the `SharedResource` object resource files. This approach of coalescing does incur a minor performance implication, but saying it’s *minor* is an overstatement. It’s proven unmeasurable with all of my testing.

###### Warning

This code is fully functional and readable. While it might seem advantageous to spend time trying to optimize it, you should heed the famous words of Professor Donald Knuth. He warns developers that “premature optimization is the root of all evil” in programming.^([1](ch05.html#idm46365018501872))

# Automating Translations with GitHub Actions

If it’s right for your app, you may want to have it support as many languages as possible. This can be done manually by creating a static resource file for each language your app supports, or you could consider a more automated approach. How might you manage the creation and maintenance of many resource files? If you’re to do this manually, when a single translation file changes, you’d have to update each corresponding supported language translation file by hand. Many larger apps will have teams assigned to translation tasks, monitoring changes to translation files and creating pull requests to make the appropriate changes, and this can become expensive. As an alternative, you can automate this.

You can create your own GitHub Action to automate translation, or you can use an existing GitHub Action that’s available on the GitHub Action Marketplace that does the same thing. If this is new to you, I suggest using an existing GitHub Action, such as the one I made for this book, called [Machine Translator](https://oreil.ly/fFmmQ). It relies on [Azure’s Cognitive Services Text Translator service](https://oreil.ly/KjO9d), and it’s written in TypeScript. The Machine Translator workflow in the Learning Blazor’s repo requires my Azure’s encrypted subscription key so that it can access a cloud-based neural machine translation technology. This allows for source-to-text translation, taking the static *.resx* resource files as input and writing out translated text for non-English languages. Within a GitHub repo, as an admin you can access the *Settings > Secrets* page, where you’ll add several repository secrets that the action will rely on as it runs.

If you’re following along in your clone of the Learning Blazor App repo, see Microsoft’s [“Quickstart: Azure Cognitive Services Translator” documentation](https://oreil.ly/82O5w). With an Azure Translator subscription key, you can run the action and see the results in GitHub Action’s output. You need to set the `AZURE_TRANSLATOR_SUBSCRIPTION_KEY`, `AZURE_TRANSLATOR_ENDPOINT`, and `AZURE_TRANSLATOR_REGION` secrets.

To automate the translation of the Learning Blazor app, we start with the following [*machine-translation.yml* workflow file](https://oreil.ly/Otzm9):

[PRE5]

[![1](assets/1.png)](#co_localizing_the_app_CO4-1)

The *machine-translation.yml* workflow is named `Azure Translation`.

[![2](assets/2.png)](#co_localizing_the_app_CO4-2)

The primary step in this workflow is to run the `IEvangelist/resource-translator@main` GitHub Action.

[![3](assets/3.png)](#co_localizing_the_app_CO4-3)

The `create-pull-request` step is run only if the `translator` step outputs changes.

The GitHub Action workflow file describes the `name` as `"Azure Translation"`, which is used later by the GitHub Action real-time status screen. The `on` syntax is used to describe when the action will run; this action runs when any *.en.resx* files are updated and `pushed` to the `main` branch. The hosting environment maps the `secrets` context object’s GitHub token value as `GITHUB_TOKEN`. The workflow defines a single job in the `jobs` node, where named `translate` operation `runs-on: ubuntu-latest` (the latest supported version of Ubuntu). Like most other GitHub Action workflow files, it needs to check out the repo’s source code using the `action/checkout@v2` action.

The second step of the `steps` node describes my [`IEvangelist/resource-translator@main` GitHub Action](https://oreil.ly/0m9jO). This reference is identified as `translator`, which later allows the workflow to reference it by name (or `id`) through expressions. The `with` syntax allows this step to provide the required GitHub Action input. The keys listed in the `with` node map directly to the names the GitHub Action publishes as input:

`subscriptionKey`

A string value from the repo’s `secrets` context named `AZURE_TRANSLATOR_SUBSCRIPTION_KEY` using expression syntax. This value should come from the Azure Translator resource’s Keys and Endpoint page, and either KEY 1 or KEY 2 is valid.

`endpoint`

A string value from the repo’s `secrets` context named `AZURE_TRANSLATOR_ENDPOINT` using expression syntax. This value should come from the Azure Translator resource’s Keys and Endpoint page, and either KEY 1 or KEY 2 is valid.

`region`

A string value from the repo’s `secrets` context named `AZURE_TRANSLA⁠TOR​_REGION` using expression syntax.

`sourceLocale`

A literal value that equals the `'en'` string.

`toLocales`

A string array of values for the locals to translate to using literal syntax.

Now, we need an action to conditionally run. We can use another action that’s available in the Github Action Marketplace. GitHub user and community member Peter Evans has a `create-pull-request` action that we can use. The `Create pull request` step will run only when changes to resource files have occurred. This occurs when the `translator` step has an output that indicates that new translations were created. The pull requests are automated and appear as requests from the `github-actions` bot. The pull requests’ description (`title`) and `body` are dynamically determined from the output of the previous step. If you’re curious what an actual pull request from a GitHub Action bot looks like, see [automated pull request #13](https://oreil.ly/8bp3v) in the Learning Blazor sample app’s GitHub repo.

Now that we’ve covered how resource files are used and their translation files are generated, we can move on to exploring various localization formatting examples.

# Localization in Action

Thus far, we’ve scrutinized the XML resource files, and we saw a mechanism for accessing the data in these files with the framework-provided `IStringLocalizer<T>` abstraction. In this section, you’ll learn how the “Have I Been Pwned” (HIBP) service (see [“Leveraging “Pwned” Functionality”](ch03.html#have-i-been-pwned)) of the Learning Blazor sample app works and how its content is affected by localization. You’ll also learn the role of the `LocalizableComponentBase<T>​.Local⁠izer` property. As an example, this functionality pairs nicely with both localized and nonlocalized content, as you will see. As we look through this, you’ll learn a bit more about how the app uses the HIBP services. The site has a *Pwned?!* top-level navigation, and clicking this link navigates the user to the `https://webassemblyof.net/pwned` route, as depicted in [Figure 5-4](#pwned-sub-routes).

![](assets/lblz_0504.png)

###### Figure 5-4\. Pwned page rendering with Breaches and Passwords subroutes

The `/pwned` route renders a page with two buttons, each with a link to its corresponding subroute. The *Breaches* button routes to `/pwned/breaches`, and the *Passwords* button routes to `/pwned/passwords`.

The markup for the *Pwned.razor* page is as follows:

[PRE6]

[![1](assets/1.png)](#co_localizing_the_app_CO5-1)

The page uses the framework-provided `PageTitle` component. This sets the browser tab title to `Pwned`.

[![2](assets/2.png)](#co_localizing_the_app_CO5-2)

The button text is localized using the `Localizer` instance and the `"Breaches"` resource.

[![3](assets/3.png)](#co_localizing_the_app_CO5-3)

The button text is localized using the `Localizer` instance and the `"Passwords"` resource.

This is the first time you’re seeing the `@attribute` directive in this book. This directive lets you add any valid class-scoped attribute to the page. In this case, the `Authorize` attribute is added to the page. This attribute is used by the framework to determine whether the user is logged in. If the user is not logged in, they are redirected to the login page. Next, let’s look at the component shadow. Consider the *Pwned.razor.cs* C# file:

[PRE7]

[![1](assets/1.png)](#co_localizing_the_app_CO6-1)

The `Pwned` page depends on the injected `NavigationManager` instance, using its navigation functionality.

[![2](assets/2.png)](#co_localizing_the_app_CO6-2)

The page has two navigation methods that navigate to the Breaches and Passwords subroutes when called.

The `Pwned` page has the following English *Pwned.razor.en.resx* resource file:

[PRE8]

[![1](assets/1.png)](#co_localizing_the_app_CO7-1)

The first `data` node is named `"Breaches"` and has a child `value` node of `Breaches`.

[![2](assets/2.png)](#co_localizing_the_app_CO7-2)

The last `data` node is named `"Passwords"` and has a child `value` node of `Passwords`.

You might be wondering why we aren’t using only the `name` attribute. That’s because, when localized, `name` isn’t translated, only the `value`. This is based on the schema of the resource file XML and is universal to all .NET apps.

The `Breaches` page lets the user freely enter any email address and check if it has been part of a data breach. This page renders as shown in [Figure 5-5](#breaches-empty).

![](assets/lblz_0505.png)

###### Figure 5-5\. The `Breaches` page rendering

When the language of the app is set to (`es-ES`), the page renders as shown in [Figure 5-6](#breaches-empty-es).

![](assets/lblz_0506.png)

###### Figure 5-6\. The `Breaches` page rendering in Spanish

Before entering an email address, there are several textual values drawn on the screen, as shown in [Figure 5-5](#breaches-empty):

`';--have i been pwned?`

This value is not translated and is hardcoded in the markup because it’s a name and shouldn’t be translated.

`pwned`

Likewise, this value isn’t translated either because it’s a term that’s well known on the internet and doesn’t need to be translated.

`Email address`

This value is translated, and its name is `"EmailAddress"` in `Localizer`.

`Breaches`

This value is translated, and its name is `"Breaches"` in `Localizer`.

`Apply filter`

This value is translated, and its name is `"ApplyFilter"` in `Localizer`.

Rather than showing the entire markup file, I’m going to focus on specific parts of the markup as it relates to localization. Consider the following snippet from the *Breaches.razor* markup file, which focuses on the email address input field:

[PRE9]

This is the markup for the email address input. The framework-provided `InputText` is used to render the text input for the email address. Its `placeholder` displays a hint for the user, expressing what the expected value is for a given HTML `input` element. In this case, a localized string of `"Email address"` is rendered.

Imagine that the user starts searching for data breaches. When an email isn’t found in any data-breach records (such is the case with *fake-email@not-real.com*), the results are formatted using the `IStringLocalizer<T>` indexer with parameter overload. Consider the following snippet from the *Breaches.razor* markup file:

[PRE10]

In this scenario, the `Localizer` instance calls its indexer and passes the `"NoBreachesFormat"` resource name and the model’s `EmailAddress`. This renders as shown in [Figure 5-7](#breaches-no-results).

The lack of a data breach is certainly a relief; however, it’s not entirely realistic. Chances are your email address has been compromised in a data breach. As an example, when the user searches for `test@user.org`, the `Breaches` page queries the Web.Api service’s `/api/pwned/breaches` endpoint. When the results are returned, the component updates to show a list of data breaches. To verify that the breaches page is capable of successfully communicating with the Web.PwnedApi project’s endpoints, we can use a test user email address that is known to have been breached seven times. If you visit the Learning Blazor sample app’s `Breaches` page and [enter the “test@user.org” email](https://oreil.ly/MimnM), you’ll see that it has, indeed, been *pwned* seven times, as shown in [Figure 5-8](#breaches). The `Breaches` page makes use of the custom-shared `ModalComponent` and displays the details of each breach when the result row is clicked.

![](assets/lblz_0507.png)

###### Figure 5-7\. The `Breaches` page rendering when no results are found

![](assets/lblz_0508.png)

###### Figure 5-8\. The `Breaches` page rendering for test@user.org

Let’s say you’re interested in learning more about the Dropbox data breach. You can click on the breach to learn more information. This action displays the modal and passes the selected data breach record as a component parameter, as shown in [Figure 5-9](#dropbox-breach).

![](assets/lblz_0509.png)

###### Figure 5-9\. The Dropbox data breach modal

To help further understand how localization works, we’ll look at the translation resource file of the *Breaches.razor.en.resx* XML:

[PRE11]

There are several name-value pairs with English values in this resource file. Other languages will have their translated values. Most components inherit either from the custom `LocalizableComponentBase` or the framework-provided `IStringLocalizer`. Then each component defines resource files and uses the localizer instance to retrieve the resources at runtime.

Next, let’s look at the `Passwords` page and a select few segments from its *Passwords.razor* markup:

[PRE12]

[![1](assets/1.png)](#co_localizing_the_app_CO8-1)

The password `InputText` component has its `placeholder` and `DisplayName` attributes assigned from the localizer `"Password"` resource.

When the user first lands on this page, the results are empty, but the heading text and the message prompt are both localized resources. These are rendered as shown in [Figure 5-10](#passwords-empty).

![](assets/lblz_0510.png)

###### Figure 5-10\. The `Passwords` page

Now we’ll see the following segment from the *Passwords.razor* markup, which is responsible for rendering the results content:

[PRE13]

[![1](assets/1.png)](#co_localizing_the_app_CO9-1)

The localizer gets the resource value matching the `"Results"` name and plots it into the `article` element heading.

[![2](assets/2.png)](#co_localizing_the_app_CO9-2)

Using a control structure, when the component’s `_pwnedPassword` object is not `null` and has a `IsPwned` value of `true`, two bits of information are added.

[![3](assets/3.png)](#co_localizing_the_app_CO9-3)

The number of times that the given password has been pwned is formatted as a string using the standard C# number formatting and the current culture.

Imagine that a user types `"password"` into the input field and searches to see if it has ever been pwned. It’s easy to imagine that this password has been used many times, and you’re not wrong. See [Figure 5-11](#passwords-password) for an example rendering of how many times `"password"` has been pwned. Yikes!

![](assets/lblz_0511.png)

###### Figure 5-11\. The `Passwords` page with a pwned password

There are a few additional control structures within the `Passwords` page. Consider the remaining *Passwords.razor* markup:

[PRE14]

[![1](assets/1.png)](#co_localizing_the_app_CO10-1)

If the password has been pwned, the `OhNoFormat` resource is used to format the localized message.

[![2](assets/2.png)](#co_localizing_the_app_CO10-2)

A message is displayed indicating that the password has not been compromised.

[![3](assets/3.png)](#co_localizing_the_app_CO10-3)

Otherwise, a prompt localizer message is displayed.

For relying on whether the `_pwnedPassword` object is `null` and when it has an `IsPwned` value of `true`, there is conditional rendering. This will show the exclamation icon with a formatted resource matching the `"OhNoFormat"` name and given the number of times the password has been pwned. This relies on the `Localizer` indexer overload that accepts `params object[] arguments`. When the `_state` object is set as loaded, but the `_pwnedPassword` object is either `null` or has a nonpwned result, the `"NotPwned"` resource is rendered. When the page is first rendered, neither the `_pwnedPassword` object nor the `_state` object is set; in this case, the `"EnterPassword"` resource is rendered. This prompts the user to enter a password.

Notice in the following XML resource that each `data` node has a `name` attribute and a single `value` subnode. Consider the *Passwords.razor.en.resx* file:

[PRE15]

# Summary

In this chapter, I showed you how to localize Blazor WebAssembly apps. You learned what *localization* means as it pertains to .NET apps and what it means to *localize* an app. I showed you how to localize apps into dozens of languages using a GitHub Action that relies on Azure Cognitive Services. I explained how Blazor WebAssembly recognizes resource files using a familiar resource manager. I also covered how to consume the `IStringLocalizer<T>` interface.

In the next chapter, you’ll learn how to use ASP.NET Core SignalR with Blazor WebAssembly. You’ll learn a pattern for using real-time web functionality throughout the app, along with a custom notification system, messaging page, and live tweet streaming page.

^([1](ch05.html#idm46365018501872-marker)) Donald Knuth, “Structured Programming with go to Statements,” *ACM Computing Surveys* 6, no. 4 (Dec. 1974): 261–301, [*https://doi.org/10.1145/356635.356640*](https://doi.org/10.1145/356635.356640).