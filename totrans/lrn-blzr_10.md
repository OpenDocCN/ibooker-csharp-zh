# Chapter 9\. Testing All the Things

In this chapter, we’re going to explore the various testing options available to you as a Blazor developer. It’s important to know what you can test and how to test it. We’ll start with the most basic testing use cases that apply to all .NET and JavaScript developers alike. I’ll provide an introduction to testing and show you how to use the xUnit, bUnit, and Playwright testing frameworks. We will then move on to more advanced testing scenarios. We’ll finish with code examples that exemplify how to automate testing with GitHub Action workflows and how to write tests, such as unit, component, and end-to-end tests.

# Why Test?

You may be asking, “What’s the point of testing if your code works anyway?” That’s a fair question. For years, I felt the same way—I disliked testing because it seemed unnecessary. After years of writing code, however, I’ve changed my mind. Testing is a great way to ensure that your code works as expected and can be refactored as needed. Testing also helps make things work right if core business rules change. Just as I once said that good code is a love letter for the next developer, testing is a show of affection as well. Let’s get started with the smallest kind of test—the *unit test*.

# Unit Testing

A *unit test* is the most basic testing strategy that exercises a small, isolated piece or unit of code. A unit test should accept only known inputs and return expected outputs—it’s best to avoid randomization in testing. By automating the unit tests and avoiding human error, you’re more likely to catch potential issues in future refactorings.

###### Note

All of the unit tests here are written in C#, but that’s not to say that you couldn’t write unit tests for the JavaScript code we used in our model app. I chose not to do this because the Learning Blazor app has very little JavaScript code and primarily wraps existing APIs, so it’s highly reliable. In other words, I’m not interested in maintaining tests that verify only framework code.

A unit test is one of the best ways to ensure code functionality, but it is not a substitute for manual functional testing because it focuses on a single unit. You can use testing frameworks, like xUnit, MSTest, and NUnit, to write unit tests for your Blazor apps. All of these frameworks are well maintained, documented, supported, and feature-rich. Pair that with a GitHub repository, and you’re off to the races. With a GitHub workflow file, you can call the `dotnet test` CLI command to run unit tests.

###### Tip

A fairly well-adopted unit testing strategy is to develop unit tests before writing the implementation of the code you’re testing. This is known as *test-driven development* (TDD). TDD has the benefit of being a bit more pragmatic in that you’re forced up front to think about how an API should be implemented before writing the code. This is a good way to ensure that you’re testing the right things.

## Defining Unit-Testable Code

One good way to do unit testing is with an extension method. I’m a big fan of extension methods. They’re so useful that they’ve become idiomatic to C# development. Extension methods are a great way to add functionality to existing classes. There was a long-standing misconception that extension methods are difficult to unit test. This is not true. This comes from the concern that an extension method cannot be mocked (its implementation cannot be controlled or customized for unit testing), and therefore other logic that relies on extended functionality cannot be controlled. It’s believed that this makes it difficult to test. However, in reality, you can still test both extension methods and consuming functionality. You do not need to mock everything to write a unit test. Again, a unit test is concerned with only a unit of work, given known inputs and expecting specific outputs.

In this section, we’re going to work through the Web.Extensions.Tests project of the model app that uses the common Arrange-Act-Assert testing pattern. In this pattern, we’ll arrange our inputs, act on the system under test, and assert the expected outputs are accurate. For more information about this pattern, see Microsoft’s [“Unit Testing Best Practices with .NET Core and .NET Standard” documentation](https://oreil.ly/WCx2o). Web​.Exten⁠sions.Tests is an xUnit test project that relies on `Microsoft.NET.Sdk`, and test projects like this can be created using the .NET CLI: `dotnet new xunit` command. The `xunit` template has all the dependencies specified and is ready to run tests. For more information, see the [xUnit website](https://xunit.net).

Throughout the development and the discussion of the model app in this book, you’ve seen the `User` property wherever our authenticated user flows through the system. This property is a `ClaimsPrincipal` instance, and it serves as a good example of how to unit test an extension method. You may recall seeing that the `User.Get​FirstEmailAddress()` method is called (in the `Contact` page) from [Chapter 8](ch08.html#chapter-eight). This method is an extension method that returns the first email address from the user’s “emails” claim. Let’s look at the extension method functionality first to understand how it should function and consider the *ClaimsPrincipalExtensions.cs* file in the Web.Extensions class library project:

[PRE0]

[![1](assets/1.png)](#co_testing_all_the_things_CO1-1)

The `GetFirstEmailAddress` method gets the first email address from the call to `GetEmailAddresses`.

[![2](assets/2.png)](#co_testing_all_the_things_CO1-2)

The `GetEmailAddresses` method gets all email addresses for a given user’s “emails” claim.

The `ClaimsPrincipalExtensions` class could benefit from some unit tests as the functionality has several different logical branches. The logic is to return `null` when there is not an “emails” claim value. When there is an “emails” claim value, we want to return an array of email addresses from `GetEmailAddresses`. This method normalizes the claim value, effectively parsing whether the `string` value starts as an array, in which case it would deserialize it as JSON to a `string[]`. Otherwise, it’s treated as a single-length array with the sole email address. In other words, if there is only one email address, we want to return an array with one element. When there is more than one, we care only about the first.

## Writing an Extension Method Unit Test

To unit test the `ClaimsPrincipal` extension method, we’ll need to be able to create an instance with known claims. Consider an internal helper class that’s used to build a custom `ClaimsPrincipal` instance, as in the *ClaimsPrincipalExtensionsTests​.Inter⁠nal.cs* C# file:

[PRE1]

[![1](assets/1.png)](#co_testing_all_the_things_CO2-1)

`ClaimsPrincipalBuilder` is a helper class internal to `ClaimsPrincipal​Exten⁠sionsTests`.

[![2](assets/2.png)](#co_testing_all_the_things_CO2-2)

The `WithClaim` method adds a claim type and value to the builder instance.

[![3](assets/3.png)](#co_testing_all_the_things_CO2-3)

The `Build` method returns a `ClaimsPrincipal` instance, creating an identity with the claims in the builder.

The builder pattern (as described in [“Builder Pattern”](ch06.html#builder-pattern)) is useful for this helper. Because we’re creating the `ClaimsPrincipal` type specific to the test, the framework will not provide the `User` instance. Instead, we’ll use the `WithClaim` method on the builder to add claims and then use the `Build` method to create a `Claim⁠s​Principal` instance. Each test can create its own instance (with known inputs). Let’s see this helper/builder in action by looking at the *ClaimsPrincipal​Exten⁠sionsTests.cs* file from the Web.Extensions.Tests project:

[PRE2]

[![1](assets/1.png)](#co_testing_all_the_things_CO3-1)

`GetFirstEmailAddressNull` verifies that given no “emails” claim value, the method returns `null`.

[![2](assets/2.png)](#co_testing_all_the_things_CO3-2)

`GetFirstEmailAddressKeyMismatch` verifies that given a claim type mismatch (there is no “emails” claim, instead “email”), the method returns `null`.

[![3](assets/3.png)](#co_testing_all_the_things_CO3-3)

`GetFirstEmailAddressArrayString` verifies that given an array of “emails” in the claim value, the first email address is returned.

[![4](assets/4.png)](#co_testing_all_the_things_CO3-4)

`GetFirstEmailAddressGetSimpleString` verifies that given there’s a single “email,” it’s returned.

[![5](assets/5.png)](#co_testing_all_the_things_CO3-5)

`GetEmailAddressesCorrectlyGetsEmails` verifies when given claim type and value pair, the expected email addresses are returned.

The first four tests are decorated using the `Fact` attribute. This signals to xUnit’s discoverability mechanism that these methods represent a single unit test. Likewise, the last test is decorated with `Theory` and the `InlineData` attribute. This signals to xUnit that the test is a parameterized test. The `InlineData` attribute takes a string array of email addresses and the expected result. Unit tests decorated with `Theory` are run multiple times, once for each `InlineData` or against various data sets through other attributes.

###### Tip

When writing `Theory` tests, it’s important to note that there are several types of data set attributes that can be used. You can do some powerful things with xUnit. I prefer it over the other options because it comes with analyzers that help ensure your tests are written correctly. For more information about xUnit analyzers, see my article [“xUnit Roslyn Analyzers”](https://oreil.ly/TP1pG).

The `ClaimsPrincipalExtensionsTests` test class is a single set of eight unit tests. Some advantages to unit testing are that the tests usually run fast and they have good readability. At the time of writing, the Web.Extensions.Tests project had 31 tests, and it took 30 milliseconds to run all the tests.

# Component Testing

Component testing focuses on a single component of functionality. Component tests have to deal with a bit more overhead than unit tests. This is because components often reference multiple other components, take on external dependencies, and manage the component’s state, among other reasons. With this added complexity comes a need for a test framework that can help you test your components.

Blazor components are unable to render themselves. This is where [bUnit](https://bunit.dev), a testing library for Blazor components, comes in. With bUnit, you can do the following:

*   Set up and define components under tests using C# or Razor syntax

*   Verify outcomes using semantic HTML comparer

*   Interact with and inspect components as well as trigger event handlers

*   Pass parameters, cascade values, and inject services into components under test

*   Mock `IJSRuntime`, Blazor authentication and authorization, and others

To demonstrate component testing, we’re going to look at the Web.Client.Tests project in the model app. The Web.Client.Tests project was created using the same template as the xUnit test project that we did in the previous section. To simplify the passing of parameters to components and the verifying of markup, bUnit allows the test project to target the `Microsoft.NET.Sdk.Razor` SDK. This makes it a Razor project, so it can render Razor markup. The project also defines a `<Package​Refer⁠ence Include="bunit" Version="1.6.4" />` element, which tells the project to use the bUnit package. Like other test projects, we add a `<Project​Reference>` to the project that we’re going to write tests against. The `Web.Client​.Tests` project references the `Web.Client` project.

In this test, we’ll define some inputs and see how to write a test that arranges a component under test, acts on it, and then asserts that it renders correctly. Let’s jump right into a component test. Consider the *ChatMessageComponentTests.razor* Razor test file:

[PRE3]

[![1](assets/1.png)](#co_testing_all_the_things_CO4-1)

The class inherits from the bUnit `TestContext` class.

[![2](assets/2.png)](#co_testing_all_the_things_CO4-2)

Several test inputs are defined in the `ChatMessageInput` property.

[![3](assets/3.png)](#co_testing_all_the_things_CO4-3)

The test method is a theory, which means that it will be run multiple times, once for each element in the `ChatMessageInput` property.

[![4](assets/4.png)](#co_testing_all_the_things_CO4-4)

`ActorMessage` is arranged with inputs from the test method parameters.

[![5](assets/5.png)](#co_testing_all_the_things_CO4-5)

`ChatMessageComponent` is acted upon by rendering it given its required parameters.

[![6](assets/6.png)](#co_testing_all_the_things_CO4-6)

The test asserts that the markup matches the expected markup.

The `ActorMessage` type is a `record` from the model app’s Web.Models project. The test framework provides `TestContext`, which is used to render the component under test (or `cut`). The `Render` method returns `IRenderedFragment`. The `MarkupMatches` method is one of many extension methods from bUnit that verifies that the rendered markup from the markup fragment matches the expected markup fragment.

To run these tests, you can use the `dotnet test` command or your favorite .NET IDE. When running these tests in Visual Studio, you can see the unique parameters for each test in the test summary details, as shown in [Figure 9-1](#component-test-results).

![](assets/lblz_0901.png)

###### Figure 9-1\. Visual Studio: Test Explorer—test detail summary for the `ChatMessageComponentTests`

Now that you’ve seen both unit tests and component tests, I’m going to show you how to achieve end-to-end testing. In the next section, I’ll introduce you to end-to-end testing with Playwright by Microsoft.

# End-to-End Testing with Playwright

End-to-end testing is a way to test an entire scenario. It tests much more than the integration of a few parts of an app; instead, it exercises a full app scenario from beginning to end. [Playwright](https://playwright.dev) is a browser automation library that enables reliable end-to-end testing for modern web apps. It’s similar to Selenium, but in my professional experience, it is far more reliable and has an easier API surface area from the standpoint of ease of use. We can use Playwright to test our model app with multiple browsers, such as Chrome and Firefox.

To demonstrate end-to-end testing with Playwright, let’s look at a login test for our model app’s Web.Client project. As you may have realized, I enjoy writing `partial` classes and separating each `partial` into a separate file with shared common concepts. There’s a bit of utilitarian code in the *LoginTests.Utilities.cs* C# file in the Web.Client.EndToEndTests project:

[PRE4]

[![1](assets/1.png)](#co_testing_all_the_things_CO5-1)

The class declares two constant `string` values, which are the live app URL for the Learning Blazor site and the authentication B2C site.

[![2](assets/2.png)](#co_testing_all_the_things_CO5-2)

The `ToBrowser` method returns an `IBrowserType` instance, which is a wrapper around the Playwright browser type.

[![3](assets/3.png)](#co_testing_all_the_things_CO5-3)

The `GetTestCredentials` method returns a `Credentials` object, which is a `readonly record struct` type that contains the username and password for the test.

[![4](assets/4.png)](#co_testing_all_the_things_CO5-4)

`Credentials` is an immutable object with two `readonly string?` values representing a username and password pair.

[![5](assets/5.png)](#co_testing_all_the_things_CO5-5)

`BrowserType` is an enumeration of the supported browsers.

These utilities will be used in the Playwright test itself.

###### Warning

The `Credentials` type is populated using environment variables. This is a *secure alternative* to hardcoding these values for the test. The environment variables are used for testing. The `TEST_USERNAME` and `TEST_PASSWORD` environment variables need to also exist in the continuous delivery pipeline. Luckily, if you’re using a GitHub repo, it’ll use GitHub Action workflows to consume encrypted secrets and run all the tests. This is good because it’s a secure alternative to hardcoding these values for the test, and the tests run automatically in the CI/CD pipeline.

The end-to-end tests run in both Chromium-based browsers (Chrome and Edge) and Firefox. Because these tests run in multiple browsers, you’ll need to specify input for each browser type. Let’s first look at the test input for Chromium by considering the following *LoginTests.Chromium.cs* file:

[PRE5]

The xUnit testing framework allows for parameterization of test inputs. The `ChromiumLoginInputs` property is a collection of `object[]` objects, each of which contains the browser type, latitude, longitude, and the calculated location. There is an optional argument for `CultureInfo` to use for each test. The test input for Firefox is similar but with a different browser type. Consider the *LoginTests.Firefox.cs* file:

[PRE6]

The only difference between the two is the browser type. Next, let’s consider the *LoginTests.cs* file:

[PRE7]

[![1](assets/1.png)](#co_testing_all_the_things_CO6-1)

The `IsHeadless` property is used when launching the test browser, and it determines whether the browser is launched in *headless* mode.

[![2](assets/2.png)](#co_testing_all_the_things_CO6-2)

`CanLoginWithVerifiedCredentials` is a `Theory` test method that runs on both Chromium and Firefox browsers.

[![3](assets/3.png)](#co_testing_all_the_things_CO6-3)

The `playwright` object is initialized and creates a `browser` instance.

[![4](assets/4.png)](#co_testing_all_the_things_CO6-4)

The `browser` configures `geolocation` permissions and sets `latitude` and `longitude` to test values.

[![5](assets/5.png)](#co_testing_all_the_things_CO6-5)

The `context` from the configured `browser` creates a new page named `loginPage`.

[![6](assets/6.png)](#co_testing_all_the_things_CO6-6)

The `loginPage` fills in the username and password and then clicks the “Sign in” button.

[![7](assets/7.png)](#co_testing_all_the_things_CO6-7)

The `text` from the `#weather-city-state` element is retrieved.

The `CanLoginWithVerifiedCredentials` test is a good example of how to use Playwright. In this case, the test is considered a `Theory` test, and a set of parameters are passed in as arguments to the test. When using a `Theory` attribute, the test is run for each occurrence of test input collection—in this case, on both `Chromium` and `Firefox` browsers. The `GetTestCredentials` method is used to get the test credentials stored as environment variables. If they’re not present, the test will fail. The test browser instance is created and launched using the `ToBrowser` method. The `browser` object is configured with the `geolocation` permission, and `latitude` and `longitude` are set to test values. The `context` object is created, and `loginPage` is created from `context`. This is a result of awaiting the call to `NewPageAsync`. This method creates a new page in the browser context.

We’re validating that a verified and registered user can log in to the Learning Blazor site. We instruct the `context` to run and wait for `loginPage` to navigate to the Learning Blazor site. As part of this operation, we conditionally add an initialization script that will set the client culture with the given `locale`. This is extremely powerful, as it allows for the testing of translations. It tests the following:

*   The user can log in with the correct credentials.

*   Given a user’s locale, the weather data is displayed in the correct language.

With `loginPage`, we then wait for the browser’s URL to match the login site URL. As the URL changes, the code will wait for the page to render its HTML, the document to be fully loaded, and the network to be idle. If this doesn’t happen within a configurable amount of time, the test will fail. Once this condition is met, we fill in the username and password and click the “Sign in” button.

If we’re unable to interact with the login page or cannot find any of these specific elements or attributes, the test will fail. The test submits the login test credentials and given their `geolocation` permissions, the browser will be able to determine the current location. The test ends by verifying that the `#weather-city-state` element contains the correct text. The test `latitude` and `longitude` are set to the test’s theory values as parameters. The correct string is matched against the known formatted City, State, and Country values.

All of this functionality is possible only when an authenticated user is logged in and their location is known. This end-to-end test runs on two browsers, and it is triggered whenever you `push` code to `main` on the app’s GitHub repository. This automated testing functionality pairs perfectly with the other tests in the model app! All of these tests are run in an automated fashion, and the results are automatically published to the CI pipeline. Let’s take a look at that next.

# Automating Test Execution

A wise person once told me, “Automate yourself out of a job,”^([1](ch09.html#idm46364998674128)) and I’m happy to tell you that this philosophy pays dividends. As a developer, your goal is to be lazy, in a way. Whenever you catch yourself doing the same thing repeatedly, it’s time to automate. One way to do this is to use GitHub Actions. I love GitHub Actions! It’s a powerful and straightforward tool that you can use to automate the testing of your code as the code changes. I’m very excited to use GitHub Actions to automate the testing of my code. I believe it’s straightforward to automate the deployment of my code. In my opinion, GitHub has perfected the art of automation. With just a few lines of code, you can create a fully automated CI/CD pipeline.

In this section, I’ll show you how to automate a test with GitHub Action workflows, using the Learning Blazor app as an example. To start, all recognizable GitHub Action workflow files should reside in the *.github/workflows* directory of your project’s GitHub repository. For example, in the Learning Blazor repository, there’s a `.github/workflows/build-validation.yml` file used to build and run unit tests. The build will fail if any of the numerous tests fail. Let’s look at the *build-validation.yml* YAML file:

[PRE8]

[![1](assets/1.png)](#co_testing_all_the_things_CO7-1)

The `name` is what displays on the GitHub *README.md* file status badges, along with the status of the latest run.

[![2](assets/2.png)](#co_testing_all_the_things_CO7-2)

Using the `on` attribute, we’re telling the GitHub Action to run on a specific event.

[![3](assets/3.png)](#co_testing_all_the_things_CO7-3)

The `env` attribute is used to set environment variables.

[![4](assets/4.png)](#co_testing_all_the_things_CO7-4)

Every workflow has a `jobs` attribute, and a job has multiple `steps`.

[![5](assets/5.png)](#co_testing_all_the_things_CO7-5)

Prepare the .NET CLI in the build environment, or install build dependencies.

[![6](assets/6.png)](#co_testing_all_the_things_CO7-6)

Playwright requires NodeJS and its package manager, the Node Package Manager (NPM).

[![7](assets/7.png)](#co_testing_all_the_things_CO7-7)

Finally, `dotnet test` is called, running all three sets of tests.

In this case, we’re telling the workflow to run on a `push` event, and we’re running on only the `main` branch. We can specify additional logic to function on the `pull_request` event or even run it manually from the GitHub UI with the `workflow_dispatch` event. With this automated workflow, GitHub Actions will automatically run tests as your code changes so you don’t have to.

# Summary

In this chapter, you learned why it’s important to test your code. You saw three distinct ways you can test your Blazor applications. You can use unit testing to make sure the tiniest pieces of your app are on point, component testing to make sure groups of things are going smoothly, and end-to-end testing to ensure that everything works together. You saw how to automate testing through GitHub Actions.

We’ve covered a lot of ground throughout this entire book. I’ve shared with you some of the most important moving parts of an enterprise-scale app containing more than 90,000 lines of code.

To continue learning about Blazor, I encourage you to check out the following resources:

*   The [official Microsoft Docs](https://oreil.ly/eiO9A) by the amazing ASP.NET Core team

*   The [Awesome Blazor GitHub repository](https://oreil.ly/icivq), which acts as a collection of awesome Blazor resources

*   [Blazor University (by Peter Morris)](https://oreil.ly/02X21), a free online Blazor site packed with educational content

*   [On-demand Blazor content](https://oreil.ly/9UAnV) from the .NET Community on YouTube

I hope that you’ve enjoyed this book, and I hope that you’ll continue to learn and grow as a developer. All in all, Blazor is a very well-suited web application framework. To me, .NET has a huge advantage over other programming languages and platforms, as I have shown throughout this book. I hope that my love for this stack will shine through. Thank you for giving me this opportunity to walk you through *Learning Blazor*!

^([1](ch09.html#idm46364998674128-marker)) Scott Hanselman of Microsoft.