# Chapter 8\. Accepting Form Input with Validation

In this chapter, you’ll learn how to use the framework-provided components for accepting form input to bind custom C# models to the `EditForm` component. We’ll cover native speech recognition when used in forms. You’ll also learn how to use Reactive Extensions with Rx.NET. The model app’s contact page form will demonstrate all of this.

Let’s start with how form submission is used to accept and validate user input. You’ll see how valid user input can be sent to HTTP endpoints for processing.

# The Basics of Form Submission

The core functionality of an HTML `form` element is to accept and validate user input. When a user’s input is invalid, the user should be notified. When there is valid input, submit that input to an HTTP endpoint to process. The form submission process is as follows:

1.  The `form` is presented to the user to fill out.

2.  The user fills out the `form` and attempts to submit it.

3.  The `form` is validated.

    1.  If the `form` is invalid, validation messages or errors are shown to the user.

    2.  If the `form` is valid, the input is sent off for processing.

Between these steps, the user interacts with the `form` in various ways, sometimes by typing, sometimes by clicking, sometimes by selecting a radio button, etc. When the form is invalid, the state of the `form` can display validation messages or errors to the user. A form can accept many different types of user input. We can apply dynamic CSS to desirable input elements to indicate that the user has entered invalid input. We can control which element has focus, and we can set elements as `disabled` or make them `readonly`. These styles include animations to emphasize errant conditions and draw the user’s attention to a specific area.

# Framework-Provided Components for Forms

Blazor provides many components that apply a layer atop native HTML elements. One such component is `EditForm`. The `EditForm` component is designed to be used as a wrapper around the native HTML `form` element. This is what’s used in the `Contact` form of the book’s model app. There are other framework-provided components as well. In the next section, you’ll see the various framework- provided components that can be used with `EditForm`.

[Table 8-1](#built-in-components) shows the various framework-provided components that can be used with the `EditForm` component.^([1](ch08.html#idm46365007034848))

Table 8-1\. Framework-provided form components

| Blazor component | HTML element wrapped | Purpose of component |
| --- | --- | --- |
| `EditForm` | `<form>` | Provides a wrapper around the native HTML `form` element |
| `InputCheckbox` | `<input type="checkbox">` | Accepts user input for either `true` or `false` |
| `Input⁠Date​<TValue>` | `<input type="date">` | Accepts a `DateTime` value as user input |
| `InputFile` | `<input type="file">` | Accepts a file upload |
| `InputNum⁠ber​<TValue>` | `<input type="number">` | Accepts a numeric value as user input |
| `InputRa⁠dio​<TValue>` | `<input type="radio">` | Accepts a mutually exclusive set of values representing a single choice |
| `InputRa⁠dioGroup​<TValue>` | Parent of one or more `Input​Ra⁠dio<TValue>` components | Semantically wraps the `InputRa⁠dio​<TValue>` components together such that they’re mutually exclusive |
| `InputSe⁠lect​<TValue>` | `<select>` | Accepts a `TValue` value as user input from a list of custom options |
| `InputText` | `<input type="text">` | Accepts a string value as user input |
| `InputTextArea` | `<textarea>` | Accepts a string value as user input but traditionally displays and expects larger values than the `InputText` component |

Using these aforementioned components, you can build out a form that is as rich and as complex as your app needs.

In the next section, I’ll show you how to build a model that will represent the state of the form and the user interacting with it. This model will be decorated with metadata that will power the validation of the form it binds to.

# Models and Data Annotations

One of the common use cases for forms is to give the end user a way to contact someone from within the app for various reasons. The [`Contact` form](https://oreil.ly/LZzCM) of the Learning Blazor app does exactly that. The user can fill out the form and send me, the owner of the app, a message. After they hit send and confirm that they’re human, the message is sent to me as an email. We’ll go over how this works throughout the chapter.

Let’s start by going through the form’s user inputs:

1.  The user’s email address (current user of the app, which is prefilled if the user is logged in).

2.  The user’s first and last name, as a pair.

3.  The subject of the message, or the reason they’re contacting through the app.

4.  The message input, which uses a `TextArea` component and some interesting JavaScript interop. The message input exposes a microphone button that toggles speech recognition.

As a visual point of reference, consider [Figure 8-1](#contact-page).

![](assets/lblz_0801.png)

###### Figure 8-1\. An example rendering of the `Contact` page

## Defining Component Models

As part of the form submission process, `EditForm` will validate the user’s input. `EditForm` will also display validation messages and errors. This is all based on either `EditContext` or a model. A model is a C# class used to bind to properties and represent relevant values. In the case of the `Contact` page, it’s using `EditContext` to manage the state of the form. And `EditContext` relies on a corresponding model. Let’s take a look at `ContactComponentModel` in the *ContactComponentModel.cs* C# file, which is responsible for representing the state of the form:

[PRE0]

[![1](assets/1.png)](#co_accepting_form_input_with_validation_CO1-1)

The model is a `record` type.

[![2](assets/2.png)](#co_accepting_form_input_with_validation_CO1-2)

The `FirstName` and `LastName` properties are required, per the `Required` attribute.

[![3](assets/3.png)](#co_accepting_form_input_with_validation_CO1-3)

The `EmailAddress` property is required, and it must be a valid email address.

[![4](assets/4.png)](#co_accepting_form_input_with_validation_CO1-4)

The `AgreesToTerms` property is required as `true`.

[![5](assets/5.png)](#co_accepting_form_input_with_validation_CO1-5)

The `NotRobot` property is a `readonly` property that is calculated from the `AreYouHumanMath` class.

[![6](assets/6.png)](#co_accepting_form_input_with_validation_CO1-6)

The `record` defines an operator to convert to a `ContactRequest`.

This model exposes the values that the user is expected to provide. The user’s first and last name is required, as well as a valid email address. The `Required` attribute is a framework-provided data annotation attribute that is used to indicate that the user must provide a value for the property. If the user doesn’t provide a value, and they attempt to either submit the form or navigate away from the underlying HTML element, `EditForm` will display an error message. C# attributes are used to provide additional information about the thing they’re applied to.

## Defining and Consuming Validation Attributes

The `RegexEmailAddress` attribute is a custom attribute that is used to indicate that the user must provide a valid email address. When decorating a model property, this attribute will validate it as an email address. The `RequiredAcceptance` attribute is a custom attribute that is used to indicate that the user must accept the terms and conditions. You can use all sorts of attributes to define objects. The `Message` property is required, as is the `Subject` property.

Let’s take a look at the `RegexEmailAddress` attribute implementation in the *Regex​E⁠mailAddressAttribute.cs* C# file:

[PRE1]

[![1](assets/1.png)](#co_accepting_form_input_with_validation_CO2-1)

The `AttributeUsage` decorator specifies the usage of another attribute class, in this case, `RegexEmailAddressAttribute`, which applies only to properties, fields, and parameters.

[![2](assets/2.png)](#co_accepting_form_input_with_validation_CO2-2)

`EmailExpression` is a `readonly Regex` instance that is used to validate the email address.

[![3](assets/3.png)](#co_accepting_form_input_with_validation_CO2-3)

The `IsRequired` property allows the developer to determine whether an email address is required at all.

[![4](assets/4.png)](#co_accepting_form_input_with_validation_CO2-4)

The constructor calls its base constructor with the `DataType.EmailAddress` value.

[![5](assets/5.png)](#co_accepting_form_input_with_validation_CO2-5)

The `IsValid` method is used to validate the email address, which is passed as a nullable `object?`.

Blazor developers can author a custom `DataTypeAttribute`. If the user enters an email address that doesn’t match the regular expression, `EditForm` will display an error message. If the value is `null` and the `IsRequired` property is `true`, `EditForm` will display an error message. The other custom attribute is `RequireAcceptance​At⁠tribute`. This attribute is used to indicate that the user must accept the terms and conditions.

Next, let’s look at `RequiredAcceptanceAttribute`, which is defined in the *Require⁠d​AcceptanceAttribute.cs* C# file:

[PRE2]

[![1](assets/1.png)](#co_accepting_form_input_with_validation_CO3-1)

`RequiredAcceptanceAttribute` is similar to `RegexEmailAddressAttribute`.

[![2](assets/2.png)](#co_accepting_form_input_with_validation_CO3-2)

The constructor calls the `DataTypeAttribute` base constructor with the `DataType.Custom` value.

[![3](assets/3.png)](#co_accepting_form_input_with_validation_CO3-3)

The `IsValid` method is used to validate the acceptance of the terms and conditions.

If the user doesn’t accept the terms and conditions, `EditForm` will display an error message. When the object that represents `value` is `null`, or `value` is `false`, the error condition is triggered. You’re free to create any custom business logic rules that you may require. Whenever you need to accept user input, you’ll start with modeling an object that represents your needs. You’ll attribute the model’s properties with either custom or framework-provided data annotations. In the next section, we’ll put this into practice and see how a model is bound to the form components.

# Implementing a Contact Form

The markup for the `Contact` page is a bit lengthy, but it contains a fair number of user inputs with various functionality and validation requirements. To animate the controls and provide the appropriate styles when user input is in a state of error, the form needs a bit more markup than a semantic form. The page markup is contained in the *Contact.cshtml* Razor file:

[PRE3]

[![1](assets/1.png)](#co_accepting_form_input_with_validation_CO4-1)

The `Contact` page allows anonymous users to contact the site owner.

[![2](assets/2.png)](#co_accepting_form_input_with_validation_CO4-2)

`EditForm` is a framework-provided component that is used to render a `form`.

[![3](assets/3.png)](#co_accepting_form_input_with_validation_CO4-3)

The page model accepts an `EmailAddress` property and renders an `<InputText>` element.

[![4](assets/4.png)](#co_accepting_form_input_with_validation_CO4-4)

The page model accepts `FirstName` and `LastName` properties and renders two `<input type="text">` elements.

[![5](assets/5.png)](#co_accepting_form_input_with_validation_CO4-5)

The page model accepts a `Subject` property and renders an `<InputText>` element.

[![6](assets/6.png)](#co_accepting_form_input_with_validation_CO4-6)

The page model accepts a `Message` property and renders an `<InputTextArea>` element. This is where the additive speech recognition component is rendered, and that’s detailed later in this chapter.

[![7](assets/7.png)](#co_accepting_form_input_with_validation_CO4-7)

The page model accepts an `AgreesToTerms` property and renders an `<Input​Check⁠box>` element.

[![8](assets/8.png)](#co_accepting_form_input_with_validation_CO4-8)

The page model accepts a `Send` button and renders a `<button>` element.

[![9](assets/9.png)](#co_accepting_form_input_with_validation_CO4-9)

The page references `VerificationModalComponent` for a spam filter.

The page that displays when the `/contact` route is requested renders as shown in [Figure 8-2](#contact-page-rendered).

![](assets/lblz_0802.png)

###### Figure 8-2\. A blank `Contact` page form with only the email address prefilled

Let’s summarize what’s going on here. The `Contact` page is a form with a few fields. The page model is a class that contains the properties that are bound to the form elements. The `EditForm` component is a framework-provided component that renders an HTML `form` element. It requires either `EditContext` or `Model`, but not both. In this case, `EditContext` wraps `ContactComponentModel`. The model used in `Edit​Con⁠text` can be any `object`. `EditContext` holds metadata related to a data editing process, such as flags to indicate which fields have been modified and the current set of validation messages. The `EditContext.Model` will be used by `EditForm` to render the form. `EditContext.OnValidSubmit` event handler is used to handle the form submission. When the form is both valid and submitted, the `Contact.OnValidSubmitAsync` event handler is called. The `DataAnno⁠tations​Val⁠ida⁠tor` framework-provided component is used to add validation `Data​Annota⁠tions` attribute support that informs the `EditContext` instance with metadata about the model.

The fields in the form are as follows:

Email

A `FieldInput` custom component bound to the model’s `EmailAddress` property.

From

Two horizontal fields presented as framework-provided `InputText` components, bound to the model’s `FirstName` and `LastName` properties. These values are both required and can alter the state of the validation for a shared validation icon.

Subject

A `FieldInput` custom component bound to the model’s `Subject` property.

Message

A `FieldInput` custom component bound to the model’s `Message` property but relying on `AdditiveSpeechRecognitionComponent` to add speech recognition support that is tied to an `InputTextArea` component. `AdditiveSpeech​Rec⁠ogni⁠tionComponent` renders an overlay toggle `<button>` in the upper-righthand corner of its parent HTML element.

Whether the user agrees to the terms

A `FieldInput` custom component bound to the model’s `AgreesToTerms` property, and the framework-provided `InputCheckbox` component that is used to render a checkbox.

Submit form button

A send `<button type="submit">` element at the end of the `EditForm` markup. When the user clicks this button, the `EditContext.OnValidSubmit` event handler is called, if the form is valid.

Modal dialog

A dialog rendered by `VerificationModalComponent` is shown when the user clicks the `Send` button. The dialog serves as a spam filter, as it requires the user who submitted the form to answer a math question in string form.

The shadow component does this because there’s a fair bit of Razor markup. It’s used to manage the framework-provided `EditContext`, `_model`, `_emailInput`, `_firstNameInput`, `_modalComponent`, and two `bool` values for whether the email or message input elements should be `readonly`. These are detailed in the coming sections. Since the contact page is attributed with `AllowAnonymous`, it can be accessed by nonauthenticated users; this is intentional to allow potential users of the app to reach out with questions.

It is common for Razor components to use `Expression<Func<T>>` semantics (or expression trees) when evaluating model properties. An expression tree represents code as a data structure, where each node is an expression. Expressions look like functions but are not evaluated. Instead, an expression is parsed. For example, when we pass in `_model.EmailAddress`, the Blazor library calls `FieldCssClass`. The expression is then parsed extracting how to evaluate both our model and its corresponding property value.

As a convenience for determining which CSS classes are applicable given the state of a specific model’s property expression, the `GetValidityCss` extension method calculates the appropriate CSS classes for the property. Consider the *EditContext​Ex⁠tensions.cs* C# file:

[PRE4]

[![1](assets/1.png)](#co_accepting_form_input_with_validation_CO5-1)

The `Expression<Func<T>>` parameter is used to access the model’s property.

[![2](assets/2.png)](#co_accepting_form_input_with_validation_CO5-2)

The `Expression<Func<TOne>>` and `Expression<Func<TTwo>>` parameters are used to access the model’s property.

[![3](assets/3.png)](#co_accepting_form_input_with_validation_CO5-3)

The `bool` parameters are used to determine the CSS class to return.

[![4](assets/4.png)](#co_accepting_form_input_with_validation_CO5-4)

The `IsValid` method is used to determine if the property is valid.

[![5](assets/5.png)](#co_accepting_form_input_with_validation_CO5-5)

The `IsInvalid` method is used to determine if the property is invalid.

[![6](assets/6.png)](#co_accepting_form_input_with_validation_CO5-6)

The `IsModified` method is used to determine if the property is modified.

The `EditContextExtensions` class contains some extension methods that are used to determine the CSS class to return based on the state of the model’s property. The `GetValidityCss` method and its overloads are used to determine the CSS class to return based on the state of the model’s property. Using the framework-provided `EditContextFieldClassExtensions.FieldCssClass` extension method, we can evaluate the current CSS classes given the state of the corresponding expression. The `Get​Vali⁠dityCss` method is used throughout the markup.

Next, let’s have a look at the *Contact.razor.cs* C# file:

[PRE5]

[![1](assets/1.png)](#co_accepting_form_input_with_validation_CO6-1)

The `EditContext` instance wraps `ContactComponentModel`.

[![2](assets/2.png)](#co_accepting_form_input_with_validation_CO6-2)

The `OnInitializedAsync` method calls the `base` implementation, which initializes the `User` instance and immediately calls `InitializeModelAndContext`.

[![3](assets/3.png)](#co_accepting_form_input_with_validation_CO6-3)

The `InitializeModelAndContext` method initializes the `_model` and `_edit​Con⁠text` properties from the `User` instance.

[![4](assets/4.png)](#co_accepting_form_input_with_validation_CO6-4)

The `OnAfterRenderAsync` method determines which input element should have focus when the page is rendered.

[![5](assets/5.png)](#co_accepting_form_input_with_validation_CO6-5)

The `OnRecognitionStarted` method sets the `_isMessageReadonly` property to `true`.

[![6](assets/6.png)](#co_accepting_form_input_with_validation_CO6-6)

The `OnSpeechRecognized` method updates the `_model.Message` property with the transcript and notifies the `_editContext` instance that the `Message` property has changed.

[![7](assets/7.png)](#co_accepting_form_input_with_validation_CO6-7)

The `OnValidSubmitAsync` method is called when the user clicks the `Send` button.

[![8](assets/8.png)](#co_accepting_form_input_with_validation_CO6-8)

The `OnVerificationAttempted` method throws a `ContactRequest` at the Web.Api project’s `[HttpPost("api/contact")]` endpoint.

When the `Contact` page is initialized, the `base.User` instance is initialized as well. If there is an authenticated user, the email address is set as `readonly` and the user’s email is used. If there is no authenticated user, the `_model` instance is initialized with an empty `ContactComponentModel` instance. When the page first is rendered, either the `_emailInput` or `_firstNameInput` element is focused.

There are two methods responsible for managing whether the `_messageInput` element is `readonly`. The `OnRecognitionStarted` method sets the `_isMessageReadonly` property to `true`; `OnRecognitionStopped` sets it to `false`. When speech is recognized, the `_model.Message` property is updated with the transcript, and the `_edit​Con⁠text` instance is notified that the `Message` property has changed.

When the user supplies all of the required inputs, the form is considered “valid.” At this point, the user is free to submit the form. When the form is submitted, the `_modalComponent` instance is shown, which prompts the user to answer one question. If they’re able to do so, the form information is sent to the Web.Api project’s `[HttpPost("api/contact")]` endpoint for processing.

To help encapsulate a bit of common code for various field inputs, I wrote a `Field​In⁠put` form component. This component is used throughout the `Contact` page. Let’s take a look at the *FieldInput.razor* Razor markup file:

[PRE6]

[![1](assets/1.png)](#co_accepting_form_input_with_validation_CO7-1)

The `FieldLabelContent` property is used to render `label` for the field.

[![2](assets/2.png)](#co_accepting_form_input_with_validation_CO7-2)

The `FieldControlContent` property is used to render `input` for the field.

[![3](assets/3.png)](#co_accepting_form_input_with_validation_CO7-3)

The component accepts several optional and required parameters.

Since the `label` and `input` elements are rendered as a `RenderFragment`, the consumer is free to render whatever they want. In the `Contact` page markup, you can see examples of `FieldInput` components with the following components:

*   Single framework-provided `InputText` component

*   Multiple framework-provided `InputText` components

*   A custom `AdditiveSpeechRecognitionComponent` component

*   Single framework-provided `InputCheckbox` component

Let’s explore a few more states that the form can be rendered as.

In addition to `label`, icons are used to help deliver even more clarity to validation errors. Imagine the user enters the first name, forgets to enter the last name, and then provides a subject. They’re free to attempt clicking the `Send` button, but the `_lastNameInput` element will be outlined with a red border and its validity icon will change to a red cross. The `_subjectInput` element will have its validity icon change from the question mark to a green check mark, but the `_messageInput` element will not be highlighted, as shown in [Figure 8-3](#contact-page-invalid-close-up).

![](assets/lblz_0803.png)

###### Figure 8-3\. An example close-up rendering of an invalid `Contact` page

The user could provide a value for the last name and a message, thus clearing the validation errors, as shown in [Figure 8-4](#contact-page-valid-close-up).

![](assets/lblz_0804.png)

###### Figure 8-4\. An example close-up rendering of a valid `Contact` page

In Figures [8-3](#contact-page-invalid-close-up) and [8-4](#contact-page-valid-close-up), you may have noticed a microphone. The message input element has a button rendered in the upper-righthand corner of its bounding box. When the user clicks the button, the `_messageInput` element is temporarily disabled. This element accepts speech recognition as a form of input. The next section will show you how to incorporate speech recognition input into your form.

# Implementing Speech Recognition as User Input

Speech recognition is a commonly used input mechanism in modern apps, both for accessibility and overall convenience. More than 90% of web browsers support the speech recognition API, according to the [“Can I Use *Speech Recognition*?” web page](https://oreil.ly/qhqjt). The speech recognition API allows web developers to acquire a transcript from a recognition session from the user’s voice as input. The API is supported by all modern browsers, including Chrome, Firefox, Safari, and Edge.

To make it so that the user can use speech recognition to input text in the message field of a form, you need to rely on the browser’s native speech recognition API. This requires JavaScript interop. To use this API, you could either write your own implementation to interface with the native API or use a library that contains this logic. I maintain a Razor class library that provides an `ISpeech​Recog⁠nitionService` implementation that’s published on NuGet as [`Blazor.Speech​Recog⁠nition.WebAssem⁠bly`](https://oreil.ly/UpI60). This library exposes this type through DI, allowing consumers to call `.AddSpeechRecognitionServices` on the `IServiceCollection` type. Once the services are registered, you can consume this interface. This is an abstraction over the speech recognition API, and it uses Blazor JavaScript interop. It’s a good example of how you can create a reusable Razor class library.

Blazor class libraries let you write components, effectively sharing common markup, logic, and even static assets. Static assets are typically in the *wwwroot* folder in ASP.NET Core apps. The `Blazor.SpeechRecognition.WebAssembly` library defines a bit of JavaScript code in the *wwwroot*. Let’s take a look at the *blazorators.speech​Recog⁠nition.js* JavaScript file that exposes the `speechSynthesis` functionality:

[PRE7]

[![1](assets/1.png)](#co_accepting_form_input_with_validation_CO8-1)

The `_recognition` variable is used to store the current `SpeechRecognition` instance globally.

[![2](assets/2.png)](#co_accepting_form_input_with_validation_CO8-2)

The `cancelSpeechRecognition` method is used to cancel the current speech recognition session.

[![3](assets/3.png)](#co_accepting_form_input_with_validation_CO8-3)

The `recognizeSpeech` method is used to start the speech recognition session.

[![4](assets/4.png)](#co_accepting_form_input_with_validation_CO8-4)

The `_recognition` instance has several callbacks, each of which is registered.

[![5](assets/5.png)](#co_accepting_form_input_with_validation_CO8-5)

The `_recognition.onresult` callback is used to send the results back to the client.

[![6](assets/6.png)](#co_accepting_form_input_with_validation_CO8-6)

The `window.addEventListener` method aborts any active speech recognition session.

Although we’ve used JavaScript for other functionality in this book, this one’s different because the functions defined here use the `export` keyword. The `export` JavaScript keyword allows you to export a function or variable as an `import`-able code from another module. This is a very common JavaScript feature, and it’s used to make your code more sharable and readable and easier to maintain. Blazor can `import` these functions into .NET via JavaScript interop calls to `import` and a path to a JavaScript module. Modules simply `export` their desired functionality, and other modules consume it. In .NET, this module is represented as the framework-provided `IJSInProcessObjectReference` type. For more information about JavaScript isolation, see Microsoft’s [“Call JavaScript Functions from .NET Methods in ASP.NET Core Blazor” documentation](https://oreil.ly/0nf9D).

The two functions of this JavaScript file are `cancelSpeechRecognition` and `recognizeSpeech`. The primary function is `recognizeSpeech` as it conditionally registers all of the provided callbacks when they’re able to be handled. It’s responsible for instantiating a `SpeechRecognition` instance and assigning it to the global `_recognition` variable of the JavaScript code. Next, we’ll look at the `ISpeech​Rec⁠ogni⁠tionService` interface. It’s defined in the *ISpeechRecognitionService.cs* C# file:

[PRE8]

[![1](assets/1.png)](#co_accepting_form_input_with_validation_CO9-1)

The interface declares itself in the `Microsoft.JSInterop` namespace as a convenience.

[![2](assets/2.png)](#co_accepting_form_input_with_validation_CO9-2)

The `ISpeechRecognitionService` interface is used to define the public `Speech​Re⁠cognition` API.

[![3](assets/3.png)](#co_accepting_form_input_with_validation_CO9-3)

The `InitializeModuleAsync` method is used to initialize the speech recognition module.

[![4](assets/4.png)](#co_accepting_form_input_with_validation_CO9-4)

The `CancelSpeechRecognition` method is used to cancel the speech recognition session.

[![5](assets/5.png)](#co_accepting_form_input_with_validation_CO9-5)

The `RecognizeSpeech` method is used to start the speech recognition session.

###### Warning

Declaring a type in someone else’s namespace (such as `Microsoft.JSInterop`) should not be overused. This practice is typically not publicly recommended, but it’s used here to make the library more accessible to developers. In this way, as developers opt in to using this NuGet package, where their apps are already making use of `Microsoft.JSInterop`, they can also use the `SpeechRecognition` API.

This interface inherits `IAsyncDisposable`, and its `DisposeAsync` call will perform the necessary cleanup of the captured module reference. The `ISpeechRecognitionService` interface is small, so it’s a good candidate for simple unit testing, which is discussed in [Chapter 9](ch09.html#chapter-nine). This makes it easy to perform unit tests on the logic surrounding the speech recognition module. Next, we’ll look at the `DefaultSpeech​Re⁠cognitionService` class. It’s defined in the *DefaultSpeechRecognitionService.cs* C# file:

[PRE9]

[![1](assets/1.png)](#co_accepting_form_input_with_validation_CO10-1)

The `DefaultSpeechRecognitionService` class is both `sealed` and `internal`.

[![2](assets/2.png)](#co_accepting_form_input_with_validation_CO10-2)

There are several fields required for this implementation besides the `const string` fields—two framework-provided types (`IJSInProcessRuntime` and `IJSInProcessObjectReference`) and two custom types (`SpeechRecognitionCallbackRegistry` and `SpeechRecognitionSubject`).

[![3](assets/3.png)](#co_accepting_form_input_with_validation_CO10-3)

`InitializeSpeechRecognitionSubject` creates the speech recognition subject. If it already exists, the existing speech recognition session is canceled.

[![4](assets/4.png)](#co_accepting_form_input_with_validation_CO10-4)

The `InitializeModuleAsync` method is used to initialize the speech recognition module.

[![5](assets/5.png)](#co_accepting_form_input_with_validation_CO10-5)

The `CancelSpeechRecognition` method is used to cancel the speech recognition session.

[![6](assets/6.png)](#co_accepting_form_input_with_validation_CO10-6)

The `RecognizeSpeech` method is used to start the speech recognition session.

[![7](assets/7.png)](#co_accepting_form_input_with_validation_CO10-7)

The `OnStarted` method is used to invoke the `onStarted` callback.

[![8](assets/8.png)](#co_accepting_form_input_with_validation_CO10-8)

The `OnSpeechRecognized` method is used to invoke the `onRecognized` callback.

The `InitializeModuleAsync` method is required to be called before any other call. This ensures that the `_speechRecognitionModule` field is initialized. The `Cancel​Spee⁠chRecognition` method is used to cancel the speech recognition session. The `RecognizeSpeech` method is used to start the speech recognition session. When the speech recognition session is started, the `_speechRecognition` field is initialized. An invocation key is created (`Guid.NewGuid()`), and this is passed from .NET into the JavaScript interop calls. The calling JavaScript then uses the given `key` when it invokes its callbacks. This is then used to ensure that callbacks are removed from the `_callbackRegistry` once they’re called. The `OnStarted`, `OnEnded`, and `OnRecognitionError` methods are used to invoke the corresponding callbacks. The `OnSpeechRecognized` is different, as it instead pushes the given `transcript` and `isFinal` values into the `SpeechRecognitionResult` object and calls the `Recognition​Received` method on the `_speechRecognition` field.

The `_speechRecognition` field is a `SpeechRecognitionSubject` type. This custom type wraps a bit of reactive code and provides an encapsulated observer and observable pair. In the next section, I’ll explain how ReactiveX (Reactive Extensions) are used to create the `SpeechRecognitionSubject` type.

## Reactive Programming with the Observer Pattern

Unlike the `OnStarted`, `OnEnded`, and `OnRecognitionError` events, the `OnSpeech​Recog⁠nized` event triggers many times. This is because the JavaScript speech recognition code sets the `continuous` flag to `true` when the speech recognition session is started. The JavaScript code will invoke the `onRecognized` callback multiple times, with the `isFinal` flag set to `false` for each invocation. When intermediate recognition results are available, a final recognition result is still only intermittent. When final, it’s a complete thought or sentence. The speech recognition service will continue to listen until either an error occurs or a cancellation is requested. We’ll use reactive programming, which relies on asynchronous programming logic to handle real-time updates to otherwise static content. As the speech recognition service fires, our app will observe each occurrence of the event and take appropriate action.

[ReactiveX (or Reactive Extensions)](https://reactivex.io) is an API for asynchronous programming with observable streams. ReactiveX is an implementation of the *observer pattern*.

The .NET implementation of Reactive Extensions is known as Rx.NET. Within this library, a `Subject` type represents an object that is both an observable sequence and an observer. In the case of speech recognition, the `SpeechRecognition​Sub⁠ject` type observes a stream of `SpeechRecognitionResult` objects. Consider the *Speech​Recogni⁠tionSubject.cs* C# file:

[PRE10]

[![1](assets/1.png)](#co_accepting_form_input_with_validation_CO11-1)

The `SpeechRecognitionSubject` type relies on a subject, observer, observable, and subscription.

[![2](assets/2.png)](#co_accepting_form_input_with_validation_CO11-2)

The `_observer` field is used to invoke the `onRecognized` callback, and the constructor is `private`.

[![3](assets/3.png)](#co_accepting_form_input_with_validation_CO11-3)

The `Factory` method is used to create the `SpeechRecognitionSubject` type.

[![4](assets/4.png)](#co_accepting_form_input_with_validation_CO11-4)

The `RecognitionReceived` method is used to push the given `recognition` value into the `_speechRecognitionSubject` field.

[![5](assets/5.png)](#co_accepting_form_input_with_validation_CO11-5)

The `Dispose` method is used to dispose of the `_speechRecognition​Sub⁠scrip⁠tion` field.

The `SpeechRecognitionSubject` allows the consumer to push `SpeechRecognitionResult` instances into its underlying `Subject`. The consumer also provides an `Action<string, string>` observer function, which is used within the observables subscription. When `Subject` acts as an observable, it means its stream of intermittent results can be filtered and conditionally dispatched to the consumer. When the final `transcript` is ready, the consumer is notified and provided with the `key` and `transcript` values.

The custom subject wrapper defines only a `private` constructor, which means it’s not possible to instantiate this object unless using the `static` factory method. The `Factory` functionality accepts the `observer` used to instantiate `SpeechRecognitionSubject`. The subscription instance is stored as a field so that it can be explicitly cleaned up when the subject is disposed of.

## Managing Callbacks with a Registry

Since the service exposes several callbacks, it manages the interop callbacks in a custom registry. The `SpeechRecognitionCallbackRegistry` object allows for registering a callback and the corresponding invocation of a callback given its key. Let’s look at the *SpeechRecognitionCallbackRegistry.cs* C# file:

[PRE11]

[![1](assets/1.png)](#co_accepting_form_input_with_validation_CO12-1)

The `_onResultCallbackRegister` field is used to store the callback register for the `onRecognized` callback.

[![2](assets/2.png)](#co_accepting_form_input_with_validation_CO12-2)

The `RegisterOnRecognized` method registers the `onRecognized` callback, and the `_onResultCallbackRegister` field is used to store the callback.

[![3](assets/3.png)](#co_accepting_form_input_with_validation_CO12-3)

The `RegisterOnError` method registers the `onError` callback, and the `_onErrorCallbackRegister` field is used to store the callback.

[![4](assets/4.png)](#co_accepting_form_input_with_validation_CO12-4)

The `InvokeOnRecognized` method invokes the `onRecognized` callback, and the `OnInvokeCallback` method invokes the callback.

[![5](assets/5.png)](#co_accepting_form_input_with_validation_CO12-5)

The `InvokeOnError` method invokes the `onError` callback, and the `OnInvoke​Call⁠back` method invokes the callback.

[![6](assets/6.png)](#co_accepting_form_input_with_validation_CO12-6)

The `OnInvokeCallback` method invokes the callback in the register after it’s removed.

A `ConcurrentDictionary` represents a thread-safe collection of KVPs that can be accessed by multiple threads concurrently. There are many alternative approaches to managing callbacks, but the `SpeechRecognitionCallbackRegistry` object is the simplest and most performant. It’s thread-safe and uses globally unique identifiers to manage the callbacks—which ensures that a single registration is tethered to a single invocation of a callback. One of the advantages to using C# in a browser such as this is that we’re spoiled with the native types provided by the .NET ecosystem. Having access to primitives like `ConcurrentDictionary`, `Guid`, strongly typed delegates (`Action<T>` for example), and even Rx.NET is a huge advantage.

## Applying the Speech Recognition Service to Components

Applying `SpeechRecognitionSubject` and `SpeechRecognitionCallbackRegistry` to expose the `ISpeechRecognitionService` interface, we can now create a custom component that can be added to an HTML element and surface speech recognition functionality. Let’s look at the *AdditiveSpeechRecognitionComponent.cs* C# file:

[PRE12]

[![1](assets/1.png)](#co_accepting_form_input_with_validation_CO13-1)

The `AdditiveSpeechRecognitionComponent` implements the `IAsyncDisposable` interface, which allows us to dispose of the speech recognition module when the component is removed from the DOM.

[![2](assets/2.png)](#co_accepting_form_input_with_validation_CO13-2)

The `SpeechRecognition` property is used to access the speech recognition service.

[![3](assets/3.png)](#co_accepting_form_input_with_validation_CO13-3)

The `SpeechRecognitionStarted` property is optional and is used to notify the parent component that the speech recognition has started.

[![4](assets/4.png)](#co_accepting_form_input_with_validation_CO13-4)

The `SpeechRecognitionStopped` property is also optional, and it’s signaled when speech recognition has stopped.

[![5](assets/5.png)](#co_accepting_form_input_with_validation_CO13-5)

The `SpeechRecognized` property is an `EditorRequired` parameter, and it’s called multiple times over the typical session.

[![6](assets/6.png)](#co_accepting_form_input_with_validation_CO13-6)

The `OnAfterRenderAsync` method is used to initialize the speech recognition module.

[![7](assets/7.png)](#co_accepting_form_input_with_validation_CO13-7)

The `OnRecognizeButtonClick` method is used to start or stop speech recognition.

[![8](assets/8.png)](#co_accepting_form_input_with_validation_CO13-8)

The `OnRecognized` method is used to notify the parent component that speech recognition has been completed.

When the user clicks the microphone button, the `OnRecognizeButtonClick` method is called. The consuming `Contact` page will mark the corresponding `input` element as `readonly`. This helps to ensure that the user cannot edit the text in the input field, as it is automatically updating from the speech recognition. So, you can’t talk and type at the same time. The `EventCallback` instances signal any changes to the consumer. The `TryInvokeAsync` is an extension method that conditionally calls the `InvokeAsync` method on the `EventCallback` instance if its `HasDelegate` value is `true`.

# Form Submission Validation and Verification

Putting this all together, we’ve built a custom `Contact` page that displays a beautifully styled form that boasts speech recognition functionality with the click of a button. Before a user can submit the form, all fields must be validated. As the primary function of a form is to take user input and give it to the recipient, it’s vital to validate the input to make sure the information is communicated correctly.

The form model is bound to various form fields, and the form is validated on submission. Each form field is represented by an HTML element using Blazor components. The form field components are responsible for validating the user’s input. When the framework-provided `EditForm` component is given a C# model that is invalid, it will render the form with the appropriate error messages. Only when the form submission is valid will the `EditForm` component submit the form. Meaning all of the data annotations on the model are validated, including required fields, custom regex patterns, and custom validation methods.

Once the `Contact` form is considered valid and submitted, the user is prompted by a modal that acts as a basic spam blocker. We set up this `VerificationModalComponent` in [Figure 4-3](ch04.html#are-you-human-modal) in [Chapter 4](ch04.html#chapter-four). The modal prompts the user to answer random math problems and requires a correct answer for the submission to proceed.

[Figure 8-5](#are-you-human-modal-zoom) shows an example of this modal prompt.

![](assets/lblz_0805.png)

###### Figure 8-5\. An example rendering of the `VerificationModalComponent` zoomed in

If the answer is incorrect, the modal will not allow the user’s form data to be sent to the Web.Api project’s endpoint for processing. An incorrect answer is shown in [Figure 8-6](#are-you-human-modal-zoom-wrong).

![](assets/lblz_0806.png)

###### Figure 8-6\. An example rendering of the `VerificationModalComponent` zoomed in with the wrong answer

Once the question is correctly answered, the modal is dismissed and the contact form is processed. A notification is triggered, which states that the contact attempt is successful, as shown in [Figure 8-7](#contact-page-message-sent-notification).

![](assets/lblz_0807.png)

###### Figure 8-7\. An example rendering of the confirmation notification

Because the primary function of a form is to take user input and give it to the recipient, it’s vital to validate the input to make sure the information is communicated correctly. A model is bound to various form fields, and the form is validated on submission. Each form field is represented by an HTML element using Blazor components. The form field components are responsible for validating the user’s input. When the framework-provided `EditForm` component is given a C# model that is invalid, it will render the form with the appropriate error messages. Only when the form submission is valid will the `EditForm` component submit the form, meaning all of the data annotations on the model are validated, including required fields, custom regex patterns, and custom validation methods.

# Summary

In this chapter, I showed you how to implement a form that accepts input with validation. In the process, you learned the basics of form submission, including how to bind custom C# models to `EditForm`, how to use data annotations to decorate model properties with validation attributes, and how to render a form with validation errors. I also walked you through a speech recognition library that exposes the ability to accept a user’s spoken word as input that is bound to text input.

In the next chapter, I’m going to show you how to properly test your Blazor apps. From unit tests with xUnit to component tests with bUnit, you’ll learn how to write reliable tests that can be used to verify the functionality of your app.

^([1](ch08.html#idm46365007034848-marker)) “ASP.NET Core Blazor Forms and Input Components,” Microsoft .NET Documentation, August 16, 2022, [*https://oreil.ly/3qzqQ*](https://oreil.ly/3qzqQ).