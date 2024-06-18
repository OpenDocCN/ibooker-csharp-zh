# Chapter 8\. Developing with AWS C# SDK

Throughout this book we have utilized many ways to interact with AWS from your development machine. We have been using the [AWS Management Console](https://aws.amazon.com/console), and we have been using the AWS CLI both through Cloud9 and locally by installing the CLI locally to our machine. In this last chapter, we will take a look at some tools and integrations that offer a third way to integrate with and control your AWS services, through integrated development environments (IDEs) you will most likely already be familiar with: Visual Studio and Visual Studio Code. Both of these IDEs have extensions for AWS that allow you to interrogate your AWS resources, and we will be covering both in this chapter.

After looking at some of the features of the UI toolkits for AWS, we will also be taking a closer look at the AWS C# SDK. We have been using the SDK extensively throughout this book, so you should be familiar with the basics, but there is a lot more this collection of NuGet packages has to offer.

Lastly, we are going to round off this book by exploring some of the artificial intelligence (AI) services in AWS and how we can utilize these in .NET. So let’s jump straight in and talk about the AWS Toolkit for Visual Studio and how you can set up your Visual Studio workflow so that deployment to AWS feels seamless.

# Using AWS Toolkit for Visual Studio In Depth

The AWS Toolkit is an extension that supplements the Visual Studio interface with various AWS-related functionality. The aim of the toolkit is to make Visual Studio feel as natively integrated into AWS as it does out of the box with Microsoft’s own cloud services. With the toolkit installed you will be able to quickly deploy your code to a multitude of AWS services with the same “publish” workflow you are probably already familiar with.

The toolkit adds menu options, context menu options, project templates, and an AWS Explorer window, all of which we will cover in this chapter.

## Configuring Visual Studio for AWS Toolkit

We briefly covered installation of the toolkit in [“Using Visual Studio with AWS and AWS Toolkit for Visual Studio”](ch01.xhtml#one-aws-tookit-visual-studio), so just to recap: you can find the toolkit extension in the [Visual Studio Marketplace](https://oreil.ly/CuWwr), or through AWS’s web page for [AWS Toolkit for Visual Studio](https://aws.amazon.com/visualstudio), the latter of which also contains links to guides and documentation that can help you in getting the most out of AWS in Visual Studio.

There are three versions of the toolkit that span five versions of Visual Studio, so be sure to download the correct *.msi* installer file for your version of Visual Studio:

*   AWS Toolkit for Visual Studio 2013–2015

*   AWS Toolkit for Visual Studio 2017–2019

*   AWS Toolkit for Visual Studio 2022

The examples in this book will be using AWS Toolkit for Visual Studio 2022; however, screens for earlier versions of Visual Studio are not radically dissimilar.

Once you have installed the extension and opened up Visual Studio, you will be presented with the Getting Started page as shown in [Figure 8-1](#Figure-8-1).

Before we can connect to AWS from Visual Studio, we need to configure the toolkit with some AWS credentials so it can make API calls to AWS services on our behalf. The toolkit uses the access key and secret key(s) from your AWS credentials file, stored against a profile. You can find your AWS credentials files under *~/.aws/credentials* (Linux or macOS) or *%USERPROFILE%\.aws\credentials* (Windows), with the profile name in square brackets. For example:

[PRE0]

![doac 0801](assets/doac_0801.png)

###### Figure 8-1\. Getting started with the AWS Toolkit

If you already have credentials stored on your machine, then you can skip this step as the toolkit will pick them up inside Visual Studio. If you do not have credentials stored, then you can create an IAM user and download credentials, then import into the Credential Setup section in [Figure 8-1](#Figure-8-1).

Once you have credentials stored on your machine, you can begin exploring the AWS Toolkit features inside Visual Studio, and what better place to start than the AWS Explorer, accessible from the new Tools → AWS Explorer menu option ([Figure 8-2](#Figure-8-2)).

![doac 0802](assets/doac_0802.png)

###### Figure 8-2\. AWS Explorer menu option

We introduced the AWS Explorer at the start of this book; you can see an example of the Explorer window in [Figure 1-14](ch01.xhtml#Figure-1-7) under [“Using Visual Studio with AWS and AWS Toolkit for Visual Studio”](ch01.xhtml#one-aws-tookit-visual-studio). Alongside AWS Explorer, the toolkit installs a handful of AWS-specific project templates and blueprints. These serve as an excellent way to both accelerate the creation of a new project and as a means of experimentation with different AWS deployment models for your code.

## Project Templates from AWS

Select from one of the newly installed project templates by choosing “AWS” in the *Project type* filter, as shown in [Figure 8-3](#Figure-8-3). The project templates are for applications built on AWS Lambda, like many of the examples we have covered in this book.

![doac 0803](assets/doac_0803.png)

###### Figure 8-3\. Project templates in Visual Studio

If we select this template, “AWS Serverless Application with Tests (.NET Core - C#),” then we will be presented with another selection where we can choose from one of what AWS calls “blueprints” on the next screen. These blueprints are ways to further configure your template project to best reflect the project you are creating. In this case ([Figure 8-4](#Figure-8-4)), we selected Serverless Application in the preceding step, so we are offered several different blueprints for a serverless application.

![doac 0804](assets/doac_0804.png)

###### Figure 8-4\. AWS project blueprints

Choose the ASP.NET Core Web API blueprint and click “Finish” to create the project we will use in the next example.

## Publishing to AWS from Visual Studio

With our serverless ASP.NET Core Web API project created from the blueprint, let’s open up the Solution Explorer and visit some of the files that have been created:

serverless.template

This file is the Serverless Application Model (SAM) template for our resources. This file allows us to configure our infrastructure from one file, which can be checked into version control. We covered SAM in [“Serverless Application Model (SAM)”](ch04.xhtml#4-serverless-application-model).

AWSServerless1.Tests

This test project has been created, including a single unit test in *ValuesControllerTests.cs* to serve as a starting point for writing unit tests for our API. The test project included in this AWS blueprint uses [xUnit](https://xunit.net), which has been brought in as a NuGet dependency.

aws-lambda-tools-defaults.json

This stores the publish settings for our project. Settings added in the next step will be saved to this file as defaults, so you do not need to re-enter them each time.

The new project that was created from this blueprint is ready to publish immediately—we don’t need to make any changes. We can use the AWS Toolkit to publish this serverless project directly from Visual Studio. To do this, open the context menu by right-clicking on the project name and selecting one of the “Publish to AWS…” options. As you can see in [Figure 8-5](#Figure-8-5), the AWS Toolkit has added two new menu items you can use to publish your project.

![doac 0805](assets/doac_0805.png)

###### Figure 8-5\. New publish options added by the toolkit

However you choose to publish your application, the toolkit will use the *serverless.template* file described previously to create a CloudFormation stack for your project and then build and push your code to AWS.

## AWS Toolkit for Visual Studio Code

The rise of Visual Studio Code has been nothing short of meteoric. From public release in 2016 to being used by over [70% of professional software developers](https://oreil.ly/KJeur) in 2022, it is clear that Visual Studio Code has supplanted other IDEs for development across the industry. C# developers often turn to Visual Studio Code as a lightweight alternative to Visual Studio, and so in 2019 AWS announced the release of their [AWS Toolkit for Visual Studio Code](https://oreil.ly/spYbP).

The VS Code toolkit includes a version of the AWS Explorer found in Visual Studio, allowing you to browse and interact with your AWS services from a new “AWS” menu option under the side bar. [Figure 8-6](#Figure-8-6) shows how the AWS Explorer in VSCode allows you to browse and interact with your resources.

![doac 0806](assets/doac_0806.png)

###### Figure 8-6\. AWS Explorer in Visual Studio Code

Installing the toolkit is as simple as searching “AWS Toolkit” in the extensions marketplace from within Visual Studio Code (Ctrl+Shift+X on Windows, or ⌘-Shift-X on macOS).

As with the AWS Toolkit for Visual Studio, the VS Code extension uses your AWS credential profiles stored under *~/.aws/credentials* (Linux or macOS) or *%USERPROFILE%\.aws\credentials* (Windows). You can select which profile to use by opening the Command Palette (Ctrl+Shift+P on Windows, or ⌘-Shift-P on macOS) and selecting AWS: Choose AWS Profile. This will bring up the option to select a profile or, if you don’t have any profiles saved, set one up from within Visual Studio Code.

Alongside the AWS Explorer ([Figure 8-6](#Figure-8-6)), the AWS Toolkit for Visual Studio Code also provides an array of AWS commands you can invoke from the Command Palette. Simply search for “AWS:” to bring up all the commands with this prefix. A few of these are shown in [Figure 8-7](#Figure-8-7).

![doac 0807](assets/doac_0807.png)

###### Figure 8-7\. AWS commands in VS Code

This brings us to the end of our overview of the AWS Toolkits for both Visual Studio and Visual Studio Code. Next, let’s take a quick look at the AWS Toolkit for Rider.

## AWS Toolkit for Rider

The last AWS Toolkit extension to be aware of is that provided for JetBrains Rider. The [AWS Toolkit for Rider](https://aws.amazon.com/rider) provides similar functionality to the toolkits for the other IDEs we have seen here, including an AWS Lambda project template that integrates into Rider’s New Project dialog and creates a simple `Hello World` project and test project.

The AWS Toolkit for Rider also supports running an AWS Lambda locally inside a Docker container, allowing you to debug AWS Lambda from within Rider. We also have a new Deploy Serverless Application context menu item on the *template.yaml* file in your solution that allows you to deploy your Lambda function from within Rider.

Next in this chapter, we are going to take a closer look at the AWS SDK for .NET, and jump back into some C#.

# Key SDK Features

The [AWS SDK for .NET](https://aws.amazon.com/sdk-for-net) is a collection of over 300 NuGet packages that make it easy to call AWS services from applications running on .NET. The SDK libraries can be imported into any .NET project and used from your code to make authenticated calls to the various AWS APIs.

###### Tip

You can explore more tools, libraries, and other resources on the [.NET on AWS on GitHub](https://github.com/aws/dotnet) page, which contains links to GitHub repositories for all the open source tools maintained by AWS including the AWS SDK for .NET

All the SDK libraries follow a common pattern of service clients that wrap API calls to AWS. These are implemented in the SDK using strongly typed request and response classes available for use in your C# code. An example of one of these client classes is the `AmazonS3Client` from the [AWSSDK.S3](https://oreil.ly/xuwoj) package:

[PRE1]

Here we are invoking the `copy-object` on an AWS Simple Storage Service (S3) bucket to copy one key to another key. We start by creating a new instance of the service client (in this case `AmazonS3Client`) with some configuration values (`AmazonS3Config`). Then, with the instance of the service client, we invoke the asynchronous version of the method `CopyObjectAsync()`. The preceding code snippet is the equivalent of calling the following from the AWS command line:

[PRE2]

This pattern is followed for the majority of AWS services, as all AWS service clients inherit from the base class [`Amazon.Runtime.AmazonServiceClient`](https://oreil.ly/5b17a). This gives all AWS service client objects access to common functionality such as credentials management, logging, metrics, retries, and timeouts, some of which we will cover later in this chapter.

## Authentication in the AWS .NET SDK

In order to make calls to AWS APIs, you need to pass credentials to the AWS .NET SDK. There are several methods to do this, and different methods will be applicable in different scenarios. For example, if you are running your code locally for development purposes, you may want to authenticate with the access key and secret stored in one of your AWS profiles. When you deploy your code to AWS, however, you can use the IAM user configured for the specific EC2 instance, App Runner, or AWS Lambda function that is executing your code.

When you use a service client, the AWS SDK for .NET will look for credentials in the following places, in order:

1.  Credentials passed to the constructor of a service client object, for example `new AmazonS3Client(awsAccessKeyId, awsSecretAccessKey)`.

2.  A named credentials profile on the local machine, as covered previously in [“Configuring Visual Studio for AWS Toolkit”](#VS-AWS-Toolkit), with the profile name coming from *appsettings.{env}.json*.

3.  A named credentials profile on the local machine, with the profile name stored in an environment variable called `AWS_PROFILE`.

4.  A named credentials profile with the name `[default]`, if it exists.

5.  Access key, secret, and session tokens stored in the environment variables `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, and `AWS_SESSION_TOKEN`, respectively.

6.  Access key and secret only, in the environment variables `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY`.

7.  IAM role native to the EC2 task, EC2 instance, or other execution environment your .NET code is running on.

The easiest and most flexible way to configure AWS credentials for the SDK, and the way that allows you to best manage multiple credentials for different environments, is to use the *appsettings.{env}.json* files, and load them in using `AWSSDK.Extensions.NETCore.Setup`, which we will look at next.

## Dependency Injection with the AWS SDK

Dependency injection is an extremely common pattern in .NET applications that allows you to configure all your dependencies in one place and inject them into your .NET controllers or other services, achieving inversion of control (IoC). If you are not familiar with dependency injection and IoC, a brief overview can be found on [YouTube](https://oreil.ly/rk9Ql).

Versions of .NET from .NET Core onward support dependency injection as part of the framework, using the classes found in `Microsoft.Extensions.DependencyInjection`. Services (dependencies) are registered with the container in the *Program.cs* or *Startup.cs* files of your project. Here, an implementation of `MyDependency` is being registered against the interface `IMyDependency` in our .NET 6 *Program.cs* file:

[PRE3]

When we use the AWS SDK for .NET, we can add AWS service clients into the .NET dependency injection container using similar syntax, with the help of the NuGet package AWSSDK.Extensions.NETCore.Setup. This package allows us to do two important things: register AWS service clients as dependencies, and use an *appsettings.json* file to store our AWS configuration, as previously mentioned.

To load the configuration settings, add the AWSSDK.Extensions.NETCore.Setup NuGet package to your .NET Core / 6+ project and then add these two lines into your service registration code:

[PRE4]

Alternatively, if you are using a `Startup` class, then the code will look like this:

[PRE5]

[![1](assets/1.png)](#co_developing_with_aws_c__sdk_CO1-1)

Add the configuration settings in for use in all resolved service clients.

You can use your *appsettings.<env>.json* settings file(s) reference the name of your AWS credentials profile, among other settings. This allows you to have different AWS credentials for each environment, connecting to AWS under a different IAM user with roles and permissions bespoke to that environment. An example of *appsettings.Development.json* using a local profile called “my-profile-name” would look like this:

[PRE6]

The settings under the AWS node map to the class `Amazon.AWSConfigs` and a full list of available properties can be found in the [documentation for Amazon.AWSConfigs](https://oreil.ly/38DIi).

Once you have default AWS options added to your dependency injection container, you can go ahead and start to register AWS service clients as dependencies for use in your application. Service clients from the AWS SDK will have a concrete class and an interface you can register it against. This allows you to easily mock the interface for unit testing or to otherwise modify the behavior of your application without modifying any of the calling logic. Here is an example using the `AmazonLambdaClient` service client from `AWSSDK.Lambda`. This client allows us to invoke a function on AWS Lambda from our C# code:

[PRE7]

[![1](assets/1.png)](#co_developing_with_aws_c__sdk_CO2-1)

Register the AWS Lambda service client using the interface. At runtime this will be injected as an instance of `AmazonLambdaClient`.

With this service registered, you can inject it into a .NET controller through the constructor:

[PRE8]

Notice how we are only referencing the abstraction `IAmazonLambda` in our code and calling `IAmazonLambda.InvokeAsync(...)` to invoke the AWS Lambda function. This allows us to unit test our `DoSomething()` method above by mocking out `IAmazonLambda`, either by implementing a mock version of this interface or using a tool such as [Moq](https://nugetmusthaves.com/Package/Moq) to automate the implementation.

The credentials to make this call to AWS Lambda and invoke our function will be retrieved via the use of credentials referenced by the profile in *appsettings.{env}.json* file, allowing us to configure separate credentials for each environment and/or local development user.

## Retries and Timeouts

Since the methods in the AWS SDK make HTTP calls over the network to AWS APIs, there is always the chance that something can go wrong and a call can fail. The base `AmazonServiceClient` class that all the SDK service clients inherit from includes functionality to manage retries. Retry behavior can be configured by setting two properties on the service client configuration:

RetryMode

Set to one of three values from the `Amazon.Runtime.RequestRetryMode` enum. These are `Legacy`, `Standard`, or `Adaptive`.

MaxErrorRetry

The number of times to retry a failing call from the SDK service clients before throwing an exception in your code.

You can set these values individually when creating a new instance of any AWS service client, for example:

To follow on from the previous example of dependency injection, however, you may want to set them globally for all service clients used in your application. You can do this by either setting the environment variables `AWS_RETRY_MODE` and `AWS_MAX_ATTEMPTS`, or by adding configuration keys into your AWS config file like this:

[PRE9]

###### Note

The AWS config file is located alongside your credentials file under *~/.aws/credentials* (Linux or macOS) or *%USERPROFILE%\.aws\credentials* (Windows).

These values will be loaded into the `AWSOptions.DefaultClientConfig` property of your AWS options object:

[PRE10]

From here we can use or modify the values in our code to fine-tune how retries are performed in the AWS SDK.

## Paginators

As well as retries and timeouts, the SDK service clients also include pagination functionality for services that may return large arrays of data. For services that support it, pagination is a really nice feature that replaces the continuation token approach with a new object-based approach that returns an async enumerable.

First, let’s look at how we can paginate large results set *without* paginators, by using the `request.ContinuationToken` property. We are calling `s3Client.ListObjectsV2Async(...)` inside a `do...while` loop while there is a continuation token in the response:

[PRE11]

This is fine; however, despite the advantages we get from returning an `IAsyncEnumerable<T>`, this is still quite a lot of code to do something as simple as chaining the results from multiple pages. Since version 3.5 of the AWS SDK for .NET, we can now have access to the `Paginators` property on this `IAmazonS3` interface.^([1](ch08.xhtml#idm45599648642496)) Here is the same method but using paginators. Notice how the call to `Paginators.ListObjectsV2` is *synchronous* whereas the call to `listObjectsV2Paginator.S3Objects` is asynchronous. The first call encapsulates our request object but does not actually call off to the API until we iterate over the `S3Objects` property on the paginator:

[PRE12]

The paginator in this example returns `IPaginatedEnumerable<S3Object>`, which inherits `IAsyncEnumerable<S3Object>`, allowing us to `await foreach` over it.

This concludes our look at the AWS SDK for .NET and how you can get the most out of the common features that AWS has included in its service client classes. Next we are going to round out this chapter—and this book—with a look at some of the ways AWS can bring AI into your C# codebase, through their various Artificial Intelligence as a service (AIaaS) products.

# Using AWS AI Services

A natural next step in using AWS is to dive into high-level AI services. One of the enormous advantages of using the AWS platform is the high-level AI and ML services available. These services allow the developer to build solutions quickly in services ranging from computer vision to natural language processing. Here is a complete list of [AI services on AWS](https://oreil.ly/oxoiD). In this chapter, we are going to explore two of these AI services: Amazon Comprehend and Amazon Rekognition. These services offer analysis for text and images, respectively, using pretrained machine learning models. Between Comprehend and Rekognition we can cover many of the most common usages for AI that a typical .NET application may require.

## AWS Comprehend

Let’s get started exploring these AI services by using Amazon Comprehend. Amazon Comprehend is a service that uses natural language processing (NLP) to find critical insights about the content of documents. This capability could be essential for a company looking to detect customer sentiment in reports. You can work with one or multiple documents at the same time. The services available include the following items:

Entities

Amazon Comprehend returns document entities, including nouns like people, places, and locations.

Key phrases

Amazon Comprehend extracts vital phrases from a critical document to explain what is in the document.

PII

This function detects the presence of [personally identifiable information (PII)](https://oreil.ly/GjsXC).

Language

This function can successfully classify up to 100 languages as the primary language in the document.

Sentiment

This function detects a document’s emotional sentiment, including positive, neutral, negative, or mixed.

Syntax

This function extracts the parts of speech in a document, finding everything from adjectives to nouns.

One of the best ways to start any AI service on AWS is with the command line using AWS CloudShell. Let’s start with a snippet of code that you can paste into a CloudShell Bash terminal. The command uses the following style of `aws` followed by the name of the service `comprehend`, then the feature of the service `detect-sentiment`:

[PRE13]

The output shown in [Figure 8-8](#Figure-1-10) returns a JSON payload that includes the `SentimentScore`, i.e., the emotion of the text.

![doac 0808](assets/doac_0808.png)

###### Figure 8-8\. Async listing buckets

Another way to explore this API is by dynamically piping text from a website into our CLI. This step works through the use of `lynx`. Let’s run this to help us determine the sentiment around Albert Einstein on Wikipedia:

1.  First, install `lynx`:

    [PRE14]

2.  Next, dump the page for Albert Einstein and pipe it into less to explore it:

    [PRE15]

3.  By using `wc -l`, we get a count of the lines:

    [PRE16]

4.  To get the number of bytes, we can use `wc --bytes`:

    [PRE17]

    The result shows:

    [PRE18]

If you run `aws comprehend detect-sentiment help`, it can only process 5000 bytes. Because of this, we need to truncate the output. The truncation is done via `head` and then assigned to a Bash `TEXT` variable:

[PRE19]

Next, the command’s output using the `$TEXT` shows that Wikipedia content around Albert Einstein is generally `NEUTRAL` with a significant `POSITIVE` percentage of 34%:

[PRE20]

The Bash command line also makes building one solution and pivoting into another common. In the following example, we switch to detecting entities from the same blob of Wikipedia text. Notice that we use `--output text` to use the power of Bash to filter the output:

[PRE21]

The output of the command shows the following entities:

[PRE22]

We could go even further and count the unique entities found using Bash. The following Bash pipeline converts entity column to lowercase then formats to easily count unusual occurrences. It isn’t perfect, but it gets us a close enough proof of concept to know how to prototype APIs with Bash before moving on to C#:

[PRE23]

The output of the Bash pipeline shows us that several entities are worth exploring more, at least from the initial part of the text extracted:

[PRE24]

###### Note

You can also watch a walk-through of using Bash to extract entities from scratch on [O’Reilly](https://oreil.ly/NzKhQ) or [YouTube](https://oreil.ly/DwlYG).

With those explorations out of the way, let’s move to code up a solution in C#.

Again, select a Visual Studio Console App and install Comprehend via NuGet as shown in [Figure 8-9](#Figure-1-11).

![doac 0809](assets/doac_0809.png)

###### Figure 8-9\. NuGet Comprehend install

Here is the sentiment detection done using the `DetectSentimentAsync()` method of the service class found in the AWSSDK.Comprehend package:

[PRE25]

Running the Console Application and entering some text, as illustrated in [Figure 8-10](#Figure-1-12), shows our statement is a `MIXED` sentiment.

![doac 0810](assets/doac_0810.png)

###### Figure 8-10\. Output of Comprehend Console App

###### Note

You can also watch a walk-through of using C# with AWS Comprehend on [O’Reilly](https://oreil.ly/bNhVs) or [YouTube](https://youtu.be/zhiNMmg8FxA).

This example has used `DetectSentimentAsync()` to detect the *sentiment* (overall mood) of the text; however, we can just as easily perform entity detection on our text by calling `client.DetectEntities()` and inspecting the list of entities AWS detected in the text. Another service that offers entity detection, but this time in an image, is AWS Rekognition.

## AWS Rekognition

Rekognition is AWS’s computer vision service for both images and video. It provides a number of pretrained machine learning models to fit a range of different image recognition needs, all in an easy-to-use API available through the .NET SDK and priced using a pay-as-you-go model. You can try the service out with your own image by searching for “Rekognition” in the AWS Management Console and clicking the Try Demo button. From here you can upload an image from your machine (or choose a sample image provided by AWS) and select from one of the pretrained models. [Figure 8-11](#Figure-8-X) shows the results of the *label detection* algorithm, as you can see AWS Rekognition has identified this image as 98.1% likely to contain a cat.^([2](ch08.xhtml#idm45599648023152))

![doac 0811](assets/doac_0811.png)

###### Figure 8-11\. Testing out Rekognition through the Management Console

Calling AWS Rekognition from your C# code is as simple as installing the AWSSDK.Rekognition package from NuGet and utilizing the `Amazon.Rekognition.AmazonRekognitionClient` service class to call the API. You can inject or instantiate this service class in any of the ways we have seen earlier in this chapter; in this example, we have registered it with `services.AddAWSService<IAmazonRekognition>();`:

[PRE26]

In this example, we are calling `IAmazonRekognitionDetectLabelsAsync(...)` and passing a reference to the image in an S3 bucket. The Rekognition API works by loading the image data from an S3 bucket over accepting image data in the HTTP call to invoke the function. Saving large data files like images to S3 first, and then calling services such as AWS Rekognition is a good pattern to follow in a cloud native AWS application as it allows you to architect an event-driven system. If you need to upload the image first, you could utilize an S3 trigger much like we did in [“C# Résumé Uploader example: event-driven”](ch04.xhtml#3-resume-example-event-driven).

# Critical Thinking Discussion Questions

*   What are a few real-world consequences of using timeouts in communicating the AWS SDK in production code?

*   What are a few real-world consequences of using retries in communicating the AWS SDK in production code?

*   What new workflows are enabled by using async communication using the AWS SDK?

*   What architecture patterns are most cost-effective to avoid duplicate API calls in production systems?

*   What unique advantages does .NET coupled with AWS give you?

# Exercises

*   Set up the AWS Toolkit for Visual Studio if you haven’t already and use it to create a new S3 bucket, create a folder inside the bucket, and finally upload an image or file from your desktop.

*   Expand on the AWS Rekognition example and convert it to a web service deployed on AWS App Runner.

*   Expand on the AWS Comprehend example and convert it to a web service deployed on AWS App Runner.

*   Create a portfolio project highlighting a complete master of .NET on AWS showing end-to-end development skills, including IaC and frontend.

# Conclusion

We have used the term cloud native several times in this book, and you have no doubt encountered it many times in the wild, but let’s think about what it really means and why it matters. Cloud native code runs either exclusively in, or at least is optimized for, the cloud. By writing our code with the intention of running it exclusively on AWS, we can adopt a much deeper level of integration with AWS services such as message queues, databases, logging, and reporting. If our C# code is born in the cloud (the dictionary definition of the word *native*), we can write it to best take advantage of the services around it. Sure, there are trade-offs. Local development is more difficult if you have architected your application to use exclusively AWS Lambda functions. Complexity can increase exponentially if you increasingly integrate AWS X-Ray tracing for performance monitoring into every execution path. There are trade-offs; however, the benefits you gain in performance, flexibility, and often cost, will often vastly outweigh these drawbacks.

Remember, whether you prefer to develop in Visual Studio, VS Code, or Cloud9, there is a C# AWS workflow to suit you. The AWS Toolkit for Visual Studio has been around in one form or another since Visual Studio 2008 and is regularly updated to keep pace with additions to both AWS and Visual Studio itself. The latest version of the toolkit is even rolling out a new Publish to AWS experience that aims to make publishing your code to the cloud even simpler, allowing you to rapidly test your code in the environment it will be native to: the cloud. For us, we love Cloud9 for its seamless integration into AWS services and ability to access it from anywhere you have a web browser. Web-based IDEs may not be for everyone, but if you regularly change development machines or use portable or loaner devices for development, having your code accessible from anywhere in the cloud can be a game changer.

Amazon Web Services is the most broadly adopted cloud platform in the world and has been supporting .NET in the cloud since 2008\. With over 200 services offering everything from a simple Windows virtual machine you can spin up for any task imaginable, to machine learning services like those we have visited in this chapter, AWS supports .NET on 485 instance types, 255 different AMIs for Windows workloads, and 40 different Linux AMIs with .NET or SQL Server preconfigured. This all means deploying to and integrating with AWS from your C# codebase has never been easier.

In this book we have covered many of the ways you can leverage AWS in your application, but there are many more we have not had time to cover. There are open source .NET tools, Windows samples for AWS CodeBuild, an MQTT client for leveraging Internet of Things (IoT) services, and over 20 AWS services for machine learning alone. Among these, AWS SageMaker is a service that allows you to train and deploy your own ML models, a topic we could fill an entire book with by itself.

What we have covered, however, should give you an entry point into Amazon’s cloud services and help you plan the next stage of development for your .NET codebase(s). Whether that be a migration, containerization, or rewrite from the ground up, we hope the topics on these pages have given you enough food for thought.

^([1](ch08.xhtml#idm45599648642496-marker)) The example here is just using the Simple Storage Service (S3) client; however, this same `Paginators` property is available on many different service clients for AWS services that have methods potentially returning a large number of items.

^([2](ch08.xhtml#idm45599648023152-marker)) We almost got to the end of this book without including a picture of James’s cat.