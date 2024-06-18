# Chapter 4\. Modernizing .NET Applications to Serverless

The term *serverless* can be a source of confusion in software engineering circles. How can you run a web application without servers? Is it really just a marketing phrase? In a way, yes it is. Serverless computing is the broad term for backend systems that are architected to use managed services instead of manually configured servers. Serverless applications are “serverless” in the same way that a wireless charger for your smartphone is “wireless.” A wireless charger still has a wire; it comes out of the charger and plugs into the mains outlet, transferring energy from your wall to your desk. The *wireless* part is only the interface between the charger and your phone. The word “serverless” in serverless computing is used in the same way. Yes, there are servers, but they are all managed behind the scenes by AWS. The servers are the mains cable plugging your otherwise wireless charger into the wall outlet. The other side, the part visible to us as developers, the part that really matters when deploying and running code, that part does not involve any configuration or management of servers and is therefore *serverless*.

You are most likely already using serverless computing for some tasks. Consider three popular types of server:

*   Web servers

*   Email servers

*   File servers

Of these three, the second two are servers that we have been replacing with managed serverless solutions for a long time now. If you have ever sent an email via an API call to Mailchimp, Mailgun, SendGrid, SparkPost, or indeed Amazon’s Simple Email Service (SES), then you have used a serverless emailing solution. The days of running an SMTP server either in-house or in the cloud are, for a lot of organizations, already firmly in the past. File servers too are steadily going out of fashion and being replaced by serverless file storage solutions. Many modern web applications rely entirely on cloud native services such as Amazon Simple Storage Service (S3) as their primary storage mechanism for files. The last stone to fall from that list is the web server. In this chapter, we will show you how to replace your .NET web server with a serverless, cloud native implementation and what that means for the way you write your code.

# A Serverless Web Server

A web server is a computer, either physical or virtual, that is permanently connected to the internet and is ready to respond to HTTP(S) requests 24 hours a day. The lifecycle of an HTTP request inside a web server involves a lot of steps and executes code written by many different parties in order to read, transform, and execute each request. Right in the middle of that journey is the code written by you, the application developer.

In order to replicate the functionality of a web server, we need a way to run that custom logic on a managed and on-demand basis. AWS offers us a few ways to do this; we will look at [“App Runner”](ch05.xhtml#five-app-runner), which is a service that allows us to deploy a fully managed containerized application without having to worry about servers. AWS Fargate is another serverless solution for running web servers on a pay-as-you-go pricing model. These services take your web services and deploy them to the cloud in a serverless way. We can, however, go one step further and break apart the service itself into individual functions that are deployed independently to the cloud. For this we will need Functions as a Service (FaaS).

FaaS solutions, such as AWS Lambda, are stateless compute containers that are triggered by events. You upload the code you want to run when an event occurs and the cloud provider will handle provisioning, executing, and then deprovisioning resources for you. This unlocks two main advantages:

*   FaaS allows your code to scale down to zero. You only pay for the resources you use *while your function is executing* and nothing in between. This means you can write applications that follow a true pay-as-you-go pricing model, paying only for each function execution and not for the unused time in between.

*   FaaS forces you to think about separation of concerns, and places restrictions on the way you write your application that will generally lead to better code.

The disadvantages are born from the extra effort involved in actually implementing a FaaS solution. Consider this quote from Rich Hickley:

> Simplicity is hard work. But, there’s a huge payoff. The person who has a genuinely simpler system—a system made out of genuinely simple parts, is going to be able to affect the greatest change with the least work. He’s going to kick your ass. He’s gonna spend more time simplifying things up front and in the long haul he’s gonna wipe the plate with you because he’ll have that ability to change things when you’re struggling to push elephants around.
> 
> Rich Hickey, creator of the Clojure programming language

Simplicity as it relates to writing your code for FaaS means thinking in terms of *pure functions* and following the Single Responsibility Principle. A function should take a value object, perform an action, and then return a value object. A good example of a pure function that follows this pattern is a `Reverse()` function for a string:

[PRE0]

This function fits the two requirements to be called a “pure function”:

*   It will always return the same output for a given input.

*   It has no side effects; it does not mutate any static variables or mutable reference arguments, or otherwise require persistence of state outside the scope of the function.

These are both characteristics that are required for writing functions for FaaS. Since FaaS functions run in response to events, you want to keep the first point true so that the sentence “When X happens, perform Y” remains true for every time event X occurs. You also need to keep your function stateless (the second point). This is both a limitation and a feature of running your function on managed FaaS. Because AWS can and will deprovision resources in between executions of your function, it is not possible to share static variables. If you want to persist data in an FaaS function, you must intentionally persist it to a shared storage area, for example by saving to a database or a Redis cache.

Architecting your code into stateless functions with no side effects like this can be a challenge that requires discipline. It forces you to think about how to write your logic and how much scope each area of your code is really allowed to control. It also forces you to think about separation of concerns.

When multiple FaaS functions are connected together using events and augmented with other serverless services (such as message queues and API gateways), it becomes possible to build a backend application that will be able to perform any function a traditional web API running on a server can. This allows you to write a completely serverless backend application hosted on AWS. And the best bit is, you can do all of this in C#! AWS offers many services you can use as the building blocks for such a serverless application. Let’s take a look at some of the components they have to offer.

# Choosing Serverless Components for .NET on AWS

We are going to introduce you to some of the most useful serverless services from AWS and how you can use the various packages of the [AWS SDK for .NET](https://aws.amazon.com/sdk-for-net) to interact with these services in your code. Let’s progressively build a serverless web application for the rest of this chapter, adding functionality with each new concept introduced.

Imagine that we run a software development consultancy and are recruiting for the best and brightest C# developers around! In order for them to apply, we want them to visit our website, where they will find an HTML form that allows them to upload their résumé as a PDF file. Submitting this form will send their PDF résumé to our API, where it will be saved to a file server and an email will be sent to our recruitment team, asking them to review it. We’ll call this the Serverless C# Résumé Uploader. [Figure 4-1](#Figure-4-1) shows a high-level overview of the architecture.

![doac 0401](assets/doac_0401.png)

###### Figure 4-1\. Serverless C# Résumé Uploader architecture

The current implementation of our backend uses a web API controller action to accept the PDF file upload, save it to cloud storage, and email our recruitment team. The code looks like this:

[PRE1]

[![1](assets/1.png)](#co_modernizing__net_applications___span_class__keep_together__to_serverless__span__CO1-1)

Read the uploaded file from the request.

[![2](assets/2.png)](#co_modernizing__net_applications___span_class__keep_together__to_serverless__span__CO1-2)

Save it to cloud storage.

[![3](assets/3.png)](#co_modernizing__net_applications___span_class__keep_together__to_serverless__span__CO1-3)

Send an email to our recruitment team with a link to the file.

The code for `IStorageService` looks like this:

[PRE2]

[![1](assets/1.png)](#co_modernizing__net_applications___span_class__keep_together__to_serverless__span__CO2-1)

Create a unique S3 key name.

[![2](assets/2.png)](#co_modernizing__net_applications___span_class__keep_together__to_serverless__span__CO2-2)

Upload the file to S3.

[![3](assets/3.png)](#co_modernizing__net_applications___span_class__keep_together__to_serverless__span__CO2-3)

Generate a presigned URL pointing to our new file.^([1](ch04.xhtml#idm45599655071344))

This uses the `AWSSDK.S3` NuGet package for saving a file to an AWS S3 bucket. In this example, the bucket is called `csharp-examples-bucket`^([2](ch04.xhtml#idm45599655069584)) and the file will be uploaded and given a unique key using `Guid.NewGuid()`.

The second service in our example is the `IEmailService`, which uses AWS SES to send an email to our recruitment team. This implementation uses another NuGet package from the AWS SDK called `Amazon.SimpleEmail`:

[PRE3]

We will be using these two implementations for the rest of the examples in this chapter. As mentioned earlier, by saving the PDF file with S3 and sending our email via SES we are already using serverless solutions for file storage and email services. So let’s get rid of this web server too and move our controller action to managed cloud functions.

## Developing with AWS Lambda and C#

AWS’s FaaS product is called Lambda. AWS Lambda was introduced in 2014 by Werner Vogels, CTO of Amazon, with the following summary that we feel nicely sums up the motivation and value behind AWS Lambda:

> The focus here is on the events. Events may be driven by web services that would trigger these events. You’ll write some code, say, in JavaScript, and this will run without any hardware that you have to provision for it.
> 
> Werner Vogels

Since 2014, AWS Lambda has grown enormously in both adoption and features. They’ve added more and more event options to trigger your functions from, cutting-edge abilities like Lambda@Edge that runs your functions on the CDN server closest to your users (a.k.a. “edge computing”), custom runtimes, shared memory layers for libraries called “Lambda layers,” and crucially for us, built-in support for .NET on both x86_64 and ARM64 CPU architectures.

There are a lot of examples provided by AWS for writing Lambda functions in C# and for refactoring existing code so it can be deployed to AWS Lambda. Here is a quick sample to create and deploy a simple Lambda function in C# by taking advantage of the templates found in the `Amazon.Lambda.Templates` NuGet package. This package includes project templates that can be used with the `dotnet new` command on the .NET Core CLI:

[PRE4]

This will create a folder called *SingleCSharpLambda* containing a source and a test project for your function. The sample function is in a file called *Function.cs*:

[PRE5]

The template creates a class with a public function `FunctionHandler(input, context)`. This is the method signature for the entry point to every C# Lambda function in AWS Lambda. The two parameters are the input, the shape of which will be determined to whatever event we hook our Lambda function up to, and the `ILambdaContext context`, which is generated by AWS on execution and contains information about the currently executing Lambda function.

We’ve also added a line to print a log message to the previous template function so we can check it executes when we run it on AWS. The static class `LambdaLogger` here is part of the `Amazon.Lambda.Core` package, which was added in with our template. You can also use `Console.WriteLine()` here; AWS will send any call that writes to `stdout` or `stderr` to the CloudWatch log stream attached to your function.^([3](ch04.xhtml#idm45599654764416))

Now we can get on and deploy our function to AWS. If you don’t already have them installed, now is a good time to get the [AWS Lambda Global Tool for .NET](https://oreil.ly/k0wcz). A Global Tool is a special kind of NuGet package that you can execute from the `dotnet` command line:

[PRE6]

Then deploy your function:

[PRE7]

If not already set in the *aws-lambda-tools-defaults.json* configuration file, you will be asked for AWS region and the ARN for an IAM role you would like the function to use when it executes. The IAM role can be created on the fly by the `Amazon.Lambda.Tools` tool including all the permissions needed. You can view and edit the permissions for this (or any other) IAM role by visiting the [IAM Dashboard](https://console.aws.amazon.com/iamv2).

[Figure 4-2](#Figure-4-2) shows our example function in the AWS Management Console for the region we deployed to. You can test the function directly in the console from the Test tab on the next screen.

![doac 0402](assets/doac_0402.png)

###### Figure 4-2\. Function list in the AWS Lambda console

This test window (shown in [Figure 4-3](#Figure-4-3)) allows you to enter some JSON to pass into your function in the `input` parameter we defined earlier. Since we declared our input value as being of type *string* in `FunctionHandler(string input, ILambdaContext context)`, we should change this to be any string and we can test the function directly in the Management Console.

![doac 0403](assets/doac_0403.png)

###### Figure 4-3\. Testing your deployed Lambda function

The results of your execution are inserted into the page, including an inline “Log output” window, which is also shown in [Figure 4-3](#Figure-4-3). This allows you to rapidly find and debug errors in your Lambda function and perform manual testing.

### C# résumé example: AWS Lambda

Now that we are familiar with AWS Lambda, let’s apply this to our serverless example. We are going to take our web API controller from the C# Résumé Uploader example and turn it into an AWS Lambda function. Any web API controller can be deployed to AWS Lambda with the help of a package from AWS called Amazon.Lambda.AspNetCoreServer.Hosting. This package makes it easy to wrap a .NET controller in a Lambda function called by API Gateway.

###### Note

API Gateway is a service from AWS that makes building an API composed of Lambda functions possible. It provides the plumbing between HTTP requests and Lambda functions allowing you to configure a set of individual Lambda functions to run on `GET`, `PUT`, `POST`, `PATCH`, and `DELETE` requests to API routes you configure in API Gateway. We are using API Gateway in this example to map the route `POST: https://our-api/Application` to a Lambda function.

To deploy an ASP .NET application to AWS Lambda using this method, install the Amazon.Lambda.AspNetCoreServer package then add a call to `AddAWSLambdaHosting()` in the services collection of the application.^([4](ch04.xhtml#idm45599654700160)) This allows API Gateway and AWS Lambda to act as the web server when running on Lambda:

[PRE8]

[![1](assets/1.png)](#co_modernizing__net_applications___span_class__keep_together__to_serverless__span__CO3-1)

This is the only line we need to add.

Just as we did in [“Developing with AWS Lambda and C#”](#4-developing-with-lambda), deploy this Lambda function to AWS using our new class as the entry point. We will have to create a JSON file for settings. If you have Visual Studio and the AWS Toolkit for Visual Studio installed, there are template projects you can try out that demonstrate how this JSON file is configured, so we’d urge you to give those a go. For our example here, however, we are just going to deploy it straight to AWS Lambda:

[PRE9]

[Figure 4-4](#Figure-4-4) shows the steps `dotnet lambda deploy-function` will take you through if you do not have all these settings configured in your JSON file. For the IAM role, we need to create a role with the policies in place to do all the things our résumé uploader function will need to do (saving to S3, sending email via SES) along with invoking a Lambda function and creating CloudWatch log groups.

![doac 0404](assets/doac_0404.png)

###### Figure 4-4\. Execution of `dotnet lambda deploy-function` in the console

Next we have to create an API Gateway service to attach our Lambda to the outside world. Later on in this chapter, we will explore the AWS Serverless Application Model (SAM)—which is a great way of creating and managing serverless resources such as API Gateway—using template files that can be checked in to source control. For now, however, we would recommend creating the API Gateway service using the AWS Management Console as shown in [Figure 4-5](#Figure-4-5).

![doac 0405](assets/doac_0405.png)

###### Figure 4-5\. Creating an API in the API Gateway console

The AWS Management Console is a great way to explore and familiarize yourself with all the different settings and options of these managed AWS services so when you do come to keep these settings in template files, you have an anchor point in your mind to how everything fits together.

[Figure 4-6](#Figure-4-6) shows how we have set up an API Gateway service with a Lambda function proxy that forwards all routes to the AWS Lambda function we just deployed, `UploadNewResume`. You can test this out either by using the `TEST` tool in the console (shown on the left in [Figure 4-6](#Figure-4-6)) or, better yet, by making an actual HTTP `POST` request with the PDF file in the body to the public URL of your API Gateway service. You can find the URL of API Gateway in the Stages setting in the Management Console. For example, my API Gateway instance has the id `xxxlrx74l3` and is in the `eu-west-2` region. To upload a résumé to our API, we can execute a `POST` request to this using `curl`:

[PRE10]

This will upload our PDF file to API Gateway, which will pass it on to our Lambda function, where our code will save it to S3 and send out an email.

![doac 0406](assets/doac_0406.png)

###### Figure 4-6\. API Gateway Lambda proxy setup

All this has been possible by wrapping our API controller in the `APIGatewayProxyFunction` class and deploying to AWS Lambda as a cloud function. Next, we are going to explore what is possible when we add multiple Lambda functions to perform more complex and granular tasks in our applications.

## Developing with AWS Step Functions

By this point, we have deployed a web API controller to a Lambda function and hooked it up to API Gateway so it can be called via HTTP requests. Next, let’s think about what we can do to really take advantage of our new function as a service execution model. Remember how we spoke about FaaS forcing you to be more intentional about separation of concerns? The function in our example is doing quite a lot. This is often the case with API controller actions in any language, not just C#—they do tend to assume multiple responsibilities at once. Our `UploadNewResume()` is doing a lot more than simply uploading a résumé; it is emailing the recruitment team, and it is creating a new file name. If we want to further unlock some of the flexibility of serverless and FaaS, we need to split these operations out into their own functions. AWS has a very neat tool for managing workflows involving multiple Lambda functions (and more), and it is called AWS Step Functions.

AWS Step Functions abstracts away the plumbing between steps in your workflow. We can take a look at the controller action from our previous example and see how much of this code is just plumbing:

[PRE11]

[![1](assets/1.png)](#co_modernizing__net_applications___span_class__keep_together__to_serverless__span__CO4-1)

Moving bits around between memory streams.

[![2](assets/2.png)](#co_modernizing__net_applications___span_class__keep_together__to_serverless__span__CO4-2)

Executing a function on the storage service.

[![3](assets/3.png)](#co_modernizing__net_applications___span_class__keep_together__to_serverless__span__CO4-3)

Executing a function on the email service.

[![4](assets/4.png)](#co_modernizing__net_applications___span_class__keep_together__to_serverless__span__CO4-4)

Creating an HTTP response with code 200.

This function is doing four things, none of which are particularly complex, however *all* of which make up the business logic (or “workflow”) of our backend API. It also does not really respect the Single Responsibility Principle. The function is called `SaveUploadedResume()`, so you could argue that its single responsibility should be inserting the résumé file into S3\. Why then, is it sending emails and constructing HTTP responses?

If you were to write unit tests for this function, it would be nice to simply cover all the cases you would expect to encounter when “saving an uploaded résumé.” Instead, you have to consider mocking out email dissemination and the boilerplate needed to execute an API controller action in a unit test. This increases the scope of the function and increases the number of things you need to test, ultimately increasing the friction involved in making changes to this piece of code.

Wouldn’t it be nice if we could reduce the scope of this function down to truly having one responsibility (saving the résumé) and split out email and HTTP concerns into something else?

With AWS Step Functions, you can move some or all of that workflow out of your code and into a configuration file. Step Functions uses a bespoke JSON object format called [Amazon States Language](https://oreil.ly/nmxp3) to build up what it calls the *state machine*. These state machines in AWS Step Functions are triggered by an event and flow through the steps configured in the definition file, executing tasks and passing data along the workflow until an end state is reached.

We are going to refactor our Serverless C# Résumé Uploader to use Step Functions in the next section but suffice to say, one of the major advantages of using Step Functions is the ability to develop the *plumbing* of your application separately from the individual functions. [Figure 4-7](#Figure-4-7) is a screenshot from the AWS Step Functions sections in the AWS Management Console with the section “Execution event history” showing the operations this execution of the state machine took to arrive at the end state.

![doac 0407](assets/doac_0407.png)

###### Figure 4-7\. Step Functions execution log in the Management Console

### C# Résumé Uploader Example: Step Functions

To demonstrate what we can do with AWS Step Functions, let’s take the AWS Lambda function we created in the previous part of our Résumé Uploader example and split it out from one, to multiple Lambda functions. The `UploadNewResume` Lambda function can have the single responsibility of uploading the file to S3, then we have a second Lambda to send the email (`EmailRecruitment`). The HTTP part can also be abstracted away entirely and handled by the API Gateway. This gives us much more flexibility to change or optimize our workflow on the backend even after these two functions have been developed, tested, and deployed:

[PRE12]

[![1](assets/1.png)](#co_modernizing__net_applications___span_class__keep_together__to_serverless__span__CO5-1)

The first Lambda function that takes a file and saves to S3.

[![2](assets/2.png)](#co_modernizing__net_applications___span_class__keep_together__to_serverless__span__CO5-2)

The second Lambda that emails our recruitment team a link to the file.

You can see the code for this is very similar to the `ApplicationController` code we had earlier in this chapter except it has been refactored into two methods on a `LambdaFunctions` class. This will be deployed as two individual AWS Lambda functions that use the same binaries, a fairly common way to deploy multiple Lambda functions that share code. The actual uploading and email sending is again performed by the `AwsS3StorageService` and `AwsSesEmailService`; however, we are not using dependency injection anymore.^([5](ch04.xhtml#idm45599654207008))

Also note that for these two functions the first parameter is now `state` and they are both of the type `StepFunctionsState`. AWS Step Functions executes your workflow by passing a state object between each task (in this case, a task is a function). The functions add to or remove properties from the state. The state for our previous functions looks like this:

[PRE13]

When this `state` input is passed to the first function, `UploadNewResume()`, it will have the `state.FileBase64` set by API Gateway from the HTTP `POST` request from our frontend. The function will save this file to S3 then set `state.StoredFileUrl` before passing the state object to the next function. It will also clear `state.FileBase64` from the state object. The state object in AWS Step Functions is passed around between each step and since this base 64 string will be quite large, we can set it to null after reading it to reduce the size of the state object that is passed on.

We can deploy these two functions to AWS Lambda as we did before; however, this time we need to specify the `--function-handler` parameter for each. The *function handler* is the C# function in our code that acts as the entry point to our Lambda, as we saw previously in [“Developing with AWS Lambda and C#”](#4-developing-with-lambda):

[PRE14]

If we go into AWS Step Functions we can create a new state machine and connect these two Lambdas together and to API Gateway. The AWS Management Console does give us a graphical user interface to build up these workflows; however, we are engineers, so we are going to write it in Amazon States Language in a JSON file. This also has the crucial benefit of utilizing a plain-text JSON file that we can check into source control, giving us a great *Infrastructure as Code (IaC)* deployment model. The JSON configuration for our simple workflow here looks like this:

[PRE15]

The Amazon Resource Name (ARN) of the Lambda function can be found in the AWS Lambda section of the Management Console, and it allows us to reference these Lambda functions from anywhere in AWS. If we copy the previous JSON into the definition of our AWS Step Functions state machine, you can see in the console that it creates a graphical representation of the workflow for us. [Figure 4-11](#Figure-4-11) later in this chapter shows what that looks like.

The last step in this example is to connect the start of our state machine to API Gateway. Whereas previously API Gateway was configured to proxy all requests to our AWS Lambda function, we now want to specify the exact route in API Gateway and forward it to our Step Functions workflow. [Figure 4-8](#Figure-4-8) shows the setup of a `POST` endpoint called `/applications` that has “Step Functions” set as the AWS Service. You can also see in [Figure 4-8](#Figure-4-8) that we have set Content Handling to “Convert to text (if needed)”. This is an option specific for our résumé uploader example here. Since we will be POSTing the PDF file to our API as raw binary, this option tells API Gateway to convert that to Base64 text for us. This can then be easily added onto our state object (represented by `StepFunctionsState.cs` shown earlier) and passed around between Lambda functions on our AWS Step Functions workflow.

![doac 0408](assets/doac_0408.png)

###### Figure 4-8\. API Gateway configuration for triggering Step Functions

Lastly, we can set up some request mapping in API Gateway to transform the request into a JSON object that tells AWS Step Functions which state machine to execute. That is shown in [Figure 4-9](#Figure-4-9) as a mapping template linked to the *application/pdf* content type header.

We now have everything in place to send a file to our API and watch the Step Functions workflow execute:

[PRE16]

![doac 0409](assets/doac_0409.png)

###### Figure 4-9\. Content type mapping for PDF file

If we navigate to the Executions tab of the AWS Step Functions state machine in the Management Console, we will be able to see this upload trigger an execution of our state machine. [Figure 4-10](#Figure-4-10) shows a visual representation of the execution as it goes through our two steps; you can see this graph by clicking on the latest execution.

![doac 0410](assets/doac_0410.png)

###### Figure 4-10\. Successful execution of our state machine

We will admit, this has seemed like a lot of work just to save a file to S3 and send an email. By this point you might be thinking what is even the point? So we can abstract our workflow into a JSON file? The point of refactoring all of this and deploying to AWS Step Functions is that now we have the starting point for something much more flexible and extensible. We can do some extremely cool things now without having to go anywhere near our two deployed Lambda functions. For example, why not take advantage of AWS Textract to read the PDF file and extract the candidate’s GitHub profile? This is what we’ll be doing in the next example.

### C# Résumé Uploader Example: AWS Textract

This is the kind of functionality that not too long ago would have been a very involved task to code, but with Step Functions and AWS Textract we can add this in very easily without even having to pull extra third-party libraries into our existing code, or even recompile it:

[PRE17]

[![1](assets/1.png)](#co_modernizing__net_applications___span_class__keep_together__to_serverless__span__CO6-1)

The AmazonTextractClient is found in the AWSSDK.Textract NuGet package and allows us to easily call AWS Textract.

###### Tip

Textract is one of many machine learning services offered by AWS. They also provide AWS Comprehend for performing natural language processing on text and Amazon Rekognition for tagging images. All of these services are priced on an accessible pay-as-you-go model and can be called from a Lambda function like we are doing here with AWS Textract.

Here we have the code for another C# Lambda function that will take in the same state object we have been using for all our functions and send the PDF file to Textract. The response from this will be an array of text blocks that were extracted from the PDF file. If any of these text blocks contains the string “github.com,” we add it into our state object for use by a later Lambda function. This allows us to include it in the email that is sent out, for example:

[PRE18]

[![1](assets/1.png)](#co_modernizing__net_applications___span_class__keep_together__to_serverless__span__CO7-1)

The stored file URL has still been left in the state object from the first function that saved it to S3.

[![2](assets/2.png)](#co_modernizing__net_applications___span_class__keep_together__to_serverless__span__CO7-2)

This new field was added by `LookForGithubProfile()` when Textract found a GitHub URL in the PDF.

We can deploy our new `LookForGithubProfile()` function as a third AWS Lambda and add it into our state machine JSON. We have also added an error handler in here that calls off to a function to notify us that there was an error uploading the résumé. There are many different steps and ways to create complex paths in your workflow using Amazon States Language JSON definitions. [Figure 4-11](#Figure-4-11) shows all of this together in the AWS Management Console with our definition JSON on the left and a visual representation of our workflow on the right.

![doac 0411](assets/doac_0411.png)

###### Figure 4-11\. Workflow definition and graphical representation in AWS Step Functions

We can also update our architecture diagram to remove the web server shown in [Figure 4-1](#Figure-4-1) and make the application completely serverless, as illustrated in [Figure 4-12](#Figure-4-12).

![doac 0412](assets/doac_0412.png)

###### Figure 4-12\. Serverless C# Résumé Uploader architecture

Now we have a truly serverless application in which the individual components can be developed, tested, and deployed independently. But what if we don’t want to rely on AWS Step Functions to bind all this logic together?

## Developing with SQS and SNS

AWS Step Functions are not the only way to communicate between AWS Lambda invocations and other serverless services. Long before AWS Step Functions existed we had message queues and the publisher/subscriber pattern. These are the two services offered by AWS:

Amazon Simple Notification Service (SNS)

SNS is a distributed implementation of the pub/sub pattern. Subscribers attach themselves to an SNS channel (called a “topic”) and when a message is published, all subscribers will instantly be notified. You can have multiple subscribers listening to published messages on any given topic in SNS and it supports several endpoints such as email and SMS, as well as triggering Lambda functions or making HTTP requests.

Amazon Simple Queue Service (SQS)

SQS is AWS’s message queue. Instead of being pushed to subscribers, messages sent to SQS are added to the queue and stored there for a duration of time (up to 14 days). Message receivers poll the queue at a rate suitable for them, reading and then deleting the messages from the queue as necessary. SQS queues make it possible to delay actions or batch up messages until the subscriber is ready to handle them.

These two services allow us to think in terms of *messages* and *events*, which are important concepts for building serverless systems. They will allow us to implement a publisher/subscriber pattern in our example system.

### C# Résumé Uploader example: SQS

Think about what we can do with our Serverless C# Résumé Uploader application to take advantage of SQS or SNS. We know users can upload their résumé to our API (via API Gateway), which will store it in S3 and then use Textract to read the candidate’s GitHub profile. How about instead of immediately emailing our recruitment team, we add a message to a queue? That way we can run a job once every morning and send *one* email for *all* the messages that have built up on the queue during the past 24 hours. This might make it easier for our recruitment team to sit down and read all of the day’s résumés at once instead of doing it piecemeal throughout the day. Can we do all of this without having to touch the rest of our code? Without having to rebuild, retest, or redeploy the entire application? Yes, and here’s how.

Since we are using AWS Step Functions, we can simply create a task in our definition JSON file that posts a message to an SQS queue. We don’t need another Lambda function to do this; instead, we can specify the message body directly in the JSON and then update our state engine definition. Here we have the task called `"QueueEmail"` that posts a message to an SQS queue we have set up called `UploadedResumeFiles`, referenced here by its ARN:

[PRE19]

Now, when file uploads trigger the state machine, the `EmailRecruitment` Lambda will not be executed; we will just get a message posted to the queue. All we need to do now is write a new AWS Lambda function to run once a day that will read all the messages from the queue and send an email containing all the file URLs. The code for such a Lambda might look like this:

[PRE20]

[![1](assets/1.png)](#co_modernizing__net_applications___span_class__keep_together__to_serverless__span__CO8-1)

Connect to the SQS queue and read a batch of messages.

[![2](assets/2.png)](#co_modernizing__net_applications___span_class__keep_together__to_serverless__span__CO8-2)

Concatenate the file URLs from the state objects we posted inside each message.

[![3](assets/3.png)](#co_modernizing__net_applications___span_class__keep_together__to_serverless__span__CO8-3)

Send one email with all links.

[![4](assets/4.png)](#co_modernizing__net_applications___span_class__keep_together__to_serverless__span__CO8-4)

Delete the messages from the queue—this does not happen automatically.

Triggering this Lambda once per day can be done using AWS EventBridge as shown in [Figure 4-13](#Figure-4-13). This “Add trigger” form is accessed from the Function Overview window of our new Lambda function in the AWS Management Console.

![doac 0413](assets/doac_0413.png)

###### Figure 4-13\. Adding a scheduled EventBridge trigger to a Lambda function

Now our system will send out an email once per day, including links to all the résumés on the queue, just as in [Figure 4-14](#Figure-4-14).

![doac 0414](assets/doac_0414.png)

###### Figure 4-14\. Email sent from our scheduled Lambda function

By adding an SQS queue, we have modified the behavior of our system at runtime and improved the service it provides by collating all the links into one daily email. All of this, however, is still kicked off by an HTTP request to our API. Next, we are going to explore some other types of events that can trigger logic and functions to run in our state engine.

## Developing Event-Driven Systems with AWS Triggers

Imagine you speak to someone who has a brilliant idea for an app and wants you to build it for them.^([6](ch04.xhtml#idm45599653147792)) You ask them to describe the functionality, and they say something like this:

“When the user uploads a photo I want to notify their followers, and when they send an email to my support mailbox, I want to open a support ticket.”

These “when X then do Y” statements are describing an *event-driven system*. All systems are event-driven under the hood; the trouble is that for a lot of backend applications, those events are just HTTP requests. Instead of, “When the user uploads a photo I want to notify their followers,” what we usually end up implementing is “When our server receives an HTTP message indicating a user has uploaded a photo, notify their followers.”

That’s understandable; up until serverless computing came along, the only way you could really implement this was to listen to an HTTP event on your API. But wouldn’t it be great if we could remove that extra step and implement our architecture to respond to the *real events as they happen* and not some intermediary implementation detail, such as an HTTP request?

This is the central tenet behind event-driven serverless systems. You execute functions and trigger workflows on the changes that really make sense to the problems you are trying to solve. Instead of writing an application that listens to an action from a human and then sends an HTTP request to an API to trigger an effect, you let AWS do all that and simply attach the event to the action directly. In the previous example, you could listen to a `s3:ObjectCreated:Post` in S3 and run your code whenever that event occurs, bypassing the API step completely.

### C# Résumé Uploader example: event-driven

Let’s visit our serverless résumé uploader for one final time. If you look back to [Figure 4-12](#Figure-4-12), you can see the PDF file goes from our website to the Step Functions tasks via API Gateway. There is no reason for API Gateway to be there; it is simply an implementation detail to allow our frontend to make an easy HTTP call when it uploads the file. Wouldn’t it be nice if we could rid ourselves of this API altogether and have the website upload the file directly to an S3 bucket? This would also allow us to throw away our `UploadNewResume` Lambda function too, and there is no more cathartic feeling in software development than deleting code we no longer need while retaining all the functionality of our system.

By removing the API step and having the frontend upload the file directly into S3, we also open up new possibilities for our system. For example, what if in addition to uploading a résumé on our *website*, we want to accept résumés emailed to *application@ouremail.com*? Using SES it is relatively simple to hook up an email forwarder that will extract an attached PDF file and save it to S3\. This would then trigger the same workflow on the backend, because we have hooked our logic up to the S3 event and not some intermediary HTTP API call. The statement “when we see a new PDF file in S3, kick off our workflow” becomes much more natural to the business problem we are solving. It doesn’t actually matter anymore how the PDF file gets in there because our system reacts in the same way.

As far as uploading the PDF file directly to S3 goes, we have options. If we already happen to be using [AWS Amplify](https://docs.amplify.aws)^([7](ch04.xhtml#idm45599653592928)) on the frontend, then we can use the storage module and the `"protected"` upload method, restricting access to a path in our S3 bucket based on the Cognito identity of the authenticated user:

[PRE21]

The frontend JavaScript used to upload a file to S3 would look like this:

[PRE22]

Even without AWS Amplify and an authenticated user on the frontend, there is still a way we can upload directly to the S3 bucket using a [presigned URL](https://oreil.ly/AXPDg). The process for this is to create a Lambda function behind an API gateway that, when executed, will call `AmazonS3Client.GetPreSignedURL()` and return that to the frontend to use to upload the file. Sure, you still have an API in this scenario, but the *function* of this API is much more in line with performing one generic task. You could, after all, use the presigned URL to upload other types of files in the frontend and hook multiple Step Function workflows to each.

Once you have the frontend uploading files directly into S3 instead of sending them via a web API, adding a trigger into the S3 bucket can be done directly in the Management Console on the Properties tab of the S3 bucket configuration, as shown in [Figure 4-15](#Figure-4-15).

![doac 0415](assets/doac_0415.png)

###### Figure 4-15\. Adding an event notification to an S3 bucket

Add the Step Functions state engine as the destination for this event and we arrive at the final form of our event-driven, serverless, C# Résumé Uploader Service! The final architecture is shown in [Figure 4-16](#Figure-4-16).

![doac 0416](assets/doac_0416.png)

###### Figure 4-16\. Final event-driven architecture of our résumé uploader example

As you can see we now have a concise, descriptive, and *event-driven* system that is dynamic enough to allow us to extend or modify parts of the workflow without risking introducing bugs or issues elsewhere. The last thing to think about is how we manage all these moving parts and easily configure them in one place.

# Serverless Application Model (SAM)

So far we have been making all our configuration changes by logging into the AWS Management Console and clicking around the UI. This is fine for experimentation, but is not a method particularly well suited to running a production system. One wrong click and you could cause an outage to part of your application. This is where Infrastructure as Code (IaC) can help.

IaC is the process of configuring your serverless infrastructure through machine readable definition files. CloudFormation is an IaC tool used across AWS that allows you to keep your entire cloud configuration in either YAML or JSON files. Virtually everything in your AWS can be modeled in CloudFormation templates, from DNS settings to S3 bucket properties to IAM roles and permissions. When a setting needs changing, you change the value in a CloudFormation template and tell AWS to apply the change to your resources. The most obvious advantage of this is that you can check your template JSON/YAML file into version control and have each change code reviewed, audited, and tested in a sandbox/staging environment, just like with any other code change.

One drawback of CloudFormation, however, is it can be quite complicated and verbose. Due to the sheer number of settings that are available for you to modify in your AWS resources, CloudFormation templates can become unwieldy when trying to configure a serverless system composed of multiple Lambda functions, message queues, and IAM roles. There are various tools that add an abstraction layer around CloudFormation and aid in configuring serverless systems. Tools such as Serverless Framework and Serverless Application Model (SAM) have been created to solve this problem.

You can think of SAM as a layer over the top of CloudFormation that brings the most pertinent settings for serverless applications to the front. You can find the full specification for SAM online at [AWS SAM Documentation](https://oreil.ly/4fEfH), but to give you an overview, here is part of the SAM YAML file for our C# Résumé Uploader system:

[PRE23]

You can see how we have defined the `SaveUploadedResumeLambda` and `LookForGithubProfileLambda` Lambda functions, indicated where in our C# code the entry point is, and configured them with memory, timeout, and permissions settings for execution.

With your infrastructure configured in SAM files like this, you can easily deploy new environments for testing or staging. You benefit from code reviews and you get the ability to create an automated deployment pipeline for your *resources* just like with your application code.

# Conclusion

Serverless computing allows you to build intricate, yet flexible and scalable solutions that operate on a pay-as-you-go pricing model and can scale down to zero. Personally, we use a serverless execution model whenever we need to build something quickly to validate an idea, solve a business problem, or where budget is a concern. Because you only pay for what you use, a serverless architecture centered on AWS Lambda can be an extremely low-cost way to build a minimum viable product (MVP) or deploy the backend for a mobile app. AWS offers a whopping 1 million Lambda executions per month for free, which can easily be enough to get a startup out of the idea phase or beta test a product. The other services we have introduced in this chapter all have extremely generous free tiers, allowing you to experiment with ideas and architectures without breaking the bank.

That being said, going serverless will not automatically cause your system to be cheaper to run. You will need to be considerate of designing applications that make unnecessarily large numbers of AWS Lambda calls. Just because two functions *can* be two separate Lambdas doesn’t always mean they *should* be from a cost and performance point of view. Experiment and measure. Functions as a Service can also be expensive at higher volumes if your application is not architected to make efficient use of each invocation. Because you pay per execution, at very high volumes you may find the costs soaring well past what it would have been to simply run a `dotnet` process on EC2 or Elastic Beanstalk. The costs of serverless should always be weighed up against the other advantages such as scalability.

Flexibility is another enormous advantage of serverless architectures, as the example in this chapter has shown. In each step we have radically changed the architecture of a part of our Serverless C# Résumé Uploader by making very small changes—most of the time without even having to redeploy the rest of the application. This separation between the moving parts of your system makes growth much easier and unlocks diversification of talent within your development team. There is nothing in a serverless architecture that specifies what version, technology, or even programming language the individual components are written in. Have you ever wanted to try your hand at F#? With a system built on AWS Lambda, there is nothing stopping you from writing the latest feature in an F# Lambda and slotting it into your serverless architecture. Need to route some HTTP calls to a third-party API? You can proxy that directly from API Gateway if you need to, without having to create the interface and the “plumbing” in your code. By adopting IaC tools such as SAM (or third-party frameworks such as Serverless Framework and Terraform), you can automate changes to your infrastructure and run code reviews, pull requests, and automate testing pipelines on the infrastructure configuration itself. A pull request for a serverless system will often be composed of two changes: a simple, easy to review AWS Lambda and an entry into the SAM/Terraform/CloudFormation template showing how that Lambda integrates with the rest of the system.

# Critical Thinking Discussion Questions

*   What could be the advantage of using a functional programming style when building FaaS services on AWS?

*   A friend tells you AWS Lambda is slightly slower than another open source framework for FaaS, so they won’t use it. What more considerable advantage on AWS could they be missing out on by not using AWS Lambda?

*   Why is the execution event history in AWS Step Functions one of the most critical features of the service?

*   Describe an architecture in which Amazon SQS is a crucial component in a global-scale platform.

*   What are the critical differences between SQS and SNS?

# Exercises

*   Build an AWS Lambda function that pulls messages from an SQS queue trigger.

*   Build an AWS Lambda function that accepts messages from an SNS topic.

*   Build an AWS Lambda function that prints the names of files placed in a bucket via an S3 trigger.

*   Build an AWS Step Function that accepts the payload “hello” in one AWS Lambda and returns the payload “goodbye” in a second AWS Lambda function.

*   Use AWS CloudShell and invoke an AWS Lambda function or AWS Step Function using the AWS CLI.

^([1](ch04.xhtml#idm45599655071344-marker)) A presigned URL is a link to an S3 resource that can be used from anywhere to download the contents anonymously (i.e., by entering it into a browser window). The URL is presigned using the authentication permissions of the IAM role that made this request. This effectively allows this you to share access to a secured S3 resource with anyone with whom this presigned URL is shared.

^([2](ch04.xhtml#idm45599655069584-marker)) S3 bucket names are unique across *all* AWS accounts in a region. So you may find the bucket name you want is unavailable as it is in use by another AWS account. You could avoid this by prefixing the bucket name with the name of your product or organization.

^([3](ch04.xhtml#idm45599654764416-marker)) CloudWatch is a service from AWS that provides logging and monitoring for your entire cloud infrastructure. AWS Lambda connects to CloudWatch and posts log messages that you can view from the CloudWatch console or integrate into another AWS service further downstream.

^([4](ch04.xhtml#idm45599654700160-marker)) This method uses the minimal APIs style of configuring and ASP .NET application that was introduced in .NET 6\. Amazon.Lambda.AspNetCoreServer does also support .NET applications built on earlier versions of .NET Core; however, the configuration is slightly different and involves implementing Amazon.Lambda.AspNetCoreServer.APIGatewayProxyFunction. More information can be found at [Amazon.Lambda.AspNetCoreServer](https://oreil.ly/gmY9K).

^([5](ch04.xhtml#idm45599654207008-marker)) It is possible to do dependency injection with AWS Lambda functions in C#, either by handcoding the setup or using third-party libraries. You may often find, however, that the functions are so simple they really do not need to be decorated with it.

^([6](ch04.xhtml#idm45599653147792-marker)) One of the drawbacks of being a software developer is this tends to happen a lot, whether you ask for it or not.

^([7](ch04.xhtml#idm45599653592928-marker)) AWS Amplify is a frontend framework for mobile and web apps that allows us to quickly build up a UI around serverless AWS services, such as S3 Simple Storage Service.