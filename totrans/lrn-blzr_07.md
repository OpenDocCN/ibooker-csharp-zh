# Chapter 6\. Exemplifying Real-Time Web Functionality

No web user wants to hit refresh constantly for the latest information. They want everything right now, automatically. Real-time web functionality is very common, and most modern apps require it. Many apps rely on live data to provide pertinent information to their users as soon as it becomes available. In this chapter, you’ll learn how to implement real-time web functionality using ASP.NET Core SignalR (or just SignalR). You’ll then find out how to create a server-side (`Hub`) that will expose many live data points, such as real-time alerts and notifications, a messaging system for live-user interactions, and a joinable active Twitter stream. Finally, you’ll learn how to consume this data from our Blazor WebAssembly app, which will respond to and interact with these live data points in compelling ways.

# Defining the Server-Side Events

For your Blazor app to have real-time web functionalities, you need a way for it to receive live data. That’s where SignalR comes in. Real-time browser-to-server protocols such as WebSockets or Server-Side Events can be complex to implement. SignalR provides an abstraction layer over these protocols and reduces the complexity with a succinct API. To handle the many clients to a single server, SignalR introduces the hub as a proxy between the clients and the server. In a hub, you define methods that can be called directly from clients. Likewise, the server can call methods on any of the connected clients. With a hub, you can define methods from the client to the server or server to the client—this is a two-way (duplex) communication. There is also a cloud-ready implementation of SignalR, called [Azure SignalR Service](https://oreil.ly/C8Vae). This service removes the need to manage backplanes that handle scalability and client connectivity.

The point of doing this is to allow your app to have real-time alerts, a messaging system for live-user interactions, and a joinable active Twitter stream. SignalR makes all of this possible.

This concept of one machine calling into another is known as a *remote procedure call* (RPC). All communication to the server requires an authentication token. Without a valid authentication token, the connection will not be established or maintained. With a valid token, the communication between the Web.Client app and the HTTP endpoints that it relies on is going to establish an open line where messages can be sent and received unsolicited from either process over network boundaries in real time. The optimal scenario is when both processes negotiate and agree upon the usage of WebSockets as the communication transport.

## Exposing Twitter Streams and Chat Functionality

The following examples highlight a live stream of tweets and a presence-aware chat implementation, as shown in Figures [6-1](#tweets-page) and [6-2](#chat-page-welcome).

![](assets/lblz_0601.png)

###### Figure 6-1\. `Tweets` page rendering

![](assets/lblz_0602.png)

###### Figure 6-2\. `Chat` page rendering

The Learning Blazor model app makes use of a single notification hub that manages all of the real-time functionality. In the model app, the Web.Api project contains the SignalR hub definition. It defines a single `NotificationHub` class, but there are three files in total. Each of these files represents a `partial` implementation of the `NotificationHub` object. The domain-specific segments are encapsulated within their file, for example, the *NotificationHub.Chat.cs* and *NotificationHub.Tweets.cs*. Let’s examine the *NotificationHub.cs* C# file first:

[PRE0]

[![1](assets/1.png)](#co_exemplifying_real_time_web_functionality_CO1-1)

`NotificationHub` is protected by the apps Azure AD B2C tenant.

[![2](assets/2.png)](#co_exemplifying_real_time_web_functionality_CO1-2)

The `override Task OnConnectedAsync` method is implemented as an expression that sends an event `HubServerEventNames.UserLoggedIn` to all the connected clients.

[![3](assets/3.png)](#co_exemplifying_real_time_web_functionality_CO1-3)

The `override Task OnDisconnectedAsync` method expects an error.

A valid authentication token must be provided from one of the configured third-party authentication providers, and the claims of the request must be a part of the `"User.ApiAccess"` scope. `NotificationHub` is a descendant of the framework-provided `Hub` class. This is a requirement for a SignalR server: they must expose a hub endpoint. The primary functionality in this file is the constructor (`.ctor`) definition and the overrides for handling connection and disconnection events. The other hub partials are domain-specific. This class defines several fields:

`ITwitterService _twitterService`

This service relies on the [`TweetInvi` NuGet package](https://oreil.ly/wcZVs). It manages streaming Twitter APIs and streams filtered on specific hashtags and handles.

`IStringLocalizer<Shared> _localizer`

The `Shared` class contains resources for the `NotificationHub` that are localized. Certain generic messages are translated for the alert and notification system.

`string _userName`

The hub has a single user in context. This user is the representation of the deserialized tokens from the authentication connection—in other words, the user who is currently interacting with the hub.

`string[]? _userEmail`

The hub’s user also has one or more email addresses.

The event is a `Notification<Actor>`. The generic notification object is a `record class` with a user name and an array of email addresses. These events are somewhat generic, so they can be shared by different interested parties on the client. There are some additional features the model app requires to provide a feature-rich chat room experience. You’ll learn a nice clean way to implement the “user is typing” indicator, create and share custom rooms, edit sent messages, and so on in this chapter. These same features can be reused in your Blazor apps by using similar code. Let’s explore the *NotificationHub.Chat.cs* C# file as it shows the hub’s implementation of the chat functionality:

[PRE1]

[![1](assets/1.png)](#co_exemplifying_real_time_web_functionality_CO2-1)

The `ToggleUserTyping` method alters the state of a client chat user.

[![2](assets/2.png)](#co_exemplifying_real_time_web_functionality_CO2-2)

The `PostOrUpdateMessage` method posts a message to the chat room.

[![3](assets/3.png)](#co_exemplifying_real_time_web_functionality_CO2-3)

The `JoinChat` method adds the client to the chat room.

[![4](assets/4.png)](#co_exemplifying_real_time_web_functionality_CO2-4)

The `LeaveChat` method removes the client from the chat room.

The `ToggleUserTyping` method accepts a `bool` value that indicates if the contextual user’s connection is actively typing in the chat room. This signals the `HubServer​E⁠ventNames.UserTyping` event sending out a `Notification<ActorAction>` object that represents the user and their typing status as a message.

The `PostOrUpdateMessage` method defines `room` and `message` `string` parameters and an optional `id`. If the `id` is `null`, a new globally unique identifier (GUID) is assigned to the message. The message contains the message text, the user who sent it, and whether the message is considered edited. This is used for both creating and updating user chat messages.

The `JoinChat` method requires a `room`. When called, the current connection is added to either a new or existing SignalR group with the matching room name. The method then lets the current caller know that the `HubServerEventNames.MessageReceived` event has fired, sending a welcome message to the chat room. This event sends a `Notification<ActorMessage>`. All clients have access to this custom generic notification model; it’s part of the Web.Models project. This is perfect because the clients can share these models, and serialization just works. This is far different than your typical JavaScript development, where you’d struggle to maintain the ever-changing shapes of API objects.

The `LeaveChat` method is the companion to the `JoinChat` functionality. This is intentional—you need a way to exit a `room` once you’ve joined it from the client. This happens in the `LeaveChat` method where `HubServerEventNames.MessageReceived` is sent from chat. The current contextual user’s connection to the SignalR hub instance removes them from the chat room. That specific group is sent an automated message with a bot user name and a localized message.

The chat functionality is taking shape. Imagine now that your app requires access to a live Twitter feed. The model app provides an example of how to do this too. With a requirement for Twitter-specific functionality that is communicated in real time, consider the *NotificationHub.Tweets.cs* C# file hub implementation:

[PRE2]

[![1](assets/1.png)](#co_exemplifying_real_time_web_functionality_CO3-1)

The `JoinTweets` method adds the client to the `Tweets` group.

[![2](assets/2.png)](#co_exemplifying_real_time_web_functionality_CO3-2)

The `LeaveTweets` method removes the client from the `Tweets` group.

[![3](assets/3.png)](#co_exemplifying_real_time_web_functionality_CO3-3)

The `StartTweetStream` method starts the tweet stream.

The RPCs in the tweets hub bring together the ability to join the tweet stream. When this fires, the current connection joins the `HubGroupNames.Tweets` group. The scoped `_twitterService` is asked a few questions, such as what the current streaming status is and if there are any tweets in memory:

*   When the current Twitter streaming status is not `null` and has a value, it’s assigned to `status` variables. This `status` flows to all connected clients, as they’re notified of the current Twitter `StreamingStatus`.

*   When there are tweets in memory, all connected clients are notified of the tweets as a `List<TweetContents>` collection.

The `LeaveTweets` method removes the contextual connection from the `HubGroupNames.Tweets` group. The `StartTweetStream` is idempotent as it can be called multiple times without changing the state of the first successful call to start the tweet stream. This is represented as an asynchronous operation.

You’re probably starting to wonder where the live tweets are coming from. We’ll cover that next when we look at the background service.

## Writing Contextual RPC and Inner-Process Communication

The Web.Api project of our model app is responsible for exposing an HTTP API surface area, so it’s scoped to handle requests and provide responses. We’re going to explore how to use an `IHubContext`, which allows our background service to communicate with the `NotificationHub` implementation. Beyond that, the model app shows a SignalR `/notifications` endpoint, which is handled by the collective representation of all `partial NotificationHub class` implementations. As for the live-streaming aspect of this application, we rely on a Twitter service, but we need a way to listen for events. Within an ASP.NET Core app, you can use a `Background​Ser⁠vice`, which runs in the same process but outside the request and response pipeline. SignalR provides a mechanism to access `NotificationHub` through an `IHub​Context` interface. This all comes together as shown in [Figure 6-3](#web-api-server).

![](assets/lblz_0603.png)

###### Figure 6-3\. The Web.Api server project

Let’s look at the *TwitterWorkerService.cs* C# file next:

[PRE3]

[![1](assets/1.png)](#co_exemplifying_real_time_web_functionality_CO4-1)

`TwitterWorkerService` implements `BackgroundService`.

[![2](assets/2.png)](#co_exemplifying_real_time_web_functionality_CO4-2)

The constructor takes the `ITwitterService` and `IHubContext` as parameters.

[![3](assets/3.png)](#co_exemplifying_real_time_web_functionality_CO4-3)

The `ExecuteAsync` method is the main entry point for the service.

[![4](assets/4.png)](#co_exemplifying_real_time_web_functionality_CO4-4)

The `OnStatusUpdated` method is called when `_twitterService` fires the `StatusUpdated` event.

[![5](assets/5.png)](#co_exemplifying_real_time_web_functionality_CO4-5)

`OnTweetReceived` handles the `TweetReceived` event, notifying all clients in the `HubGroupNames.Tweets` group.

`TwitterWorkerService` is a descendant of `BackgroundService`. Background services are long-lived apps that execute in a loop but with access to the notification hub’s context, and they can send messages through their connected clients. This class defines two fields:

`ITwitterService _twitterService`

This is the same service that was used in the `NotificationHub` for in-memory streaming status and tweets. It can now handle events from the underlying `TweetInvi`’s filtered streams.

`IHubContext<NotificationHub> _hubContext`

This object is used to send messages out to connected clients of the SignalR server hub.

The `TwitterWorkerService` constructor declares the values as parameters. The DI framework will provide the service and hub context objects. They’re positionally assigned using an immediate deconstruction of a tuple literal from the `.ctor` parameters. `_twitterService` has its `StatusUpdated` and `TweetReceived` event handlers assigned. The Twitter service exposes an eventing mechanism and fires an event when a tweet is received. In C# you can subscribe a delegate to events that will serve as a callback. There is no need to unsubscribe from the events because the app will not stop unless the entire app is torn down. In that case, we’re not holding on to any unsubscribed events—the entire process is being terminated.

The `ExecuteAsync` method is implemented as a signal that the app can perform its task. This just spins, listening for the stopping token’s cancellation request. It just delays and listens in an asynchronous loop.

When the `_twitterService.OnStatusUpdated` event fires, an update on the current streaming status is sent to all subscribers. All contextual clients in the `HubGroupNames.Tweets` group are sent the `HubServerEventNames.StatusUpdated` event. The notification is `StreamingStatus`.

The `_twitterService.OnTweetReceived` event is handled when a new `Tweet​Con⁠tents tweet` object is received. These `tweet` contents are sent from the `Hub​ServerEventNames.TweetReceived` event. They are also sent over to the same group named `HubGroupNames.Tweets`.

The server functionality is complete. With this, we can serve up a SignalR connection over a negotiated `/notifications` endpoint. Each client negotiates what protocol and transport they speak. A SignalR transport is the communication handler, such as WebSockets, Server-Sent Events, and Long Polling. There are various ways in which clients can talk to servers and vice versa. This usually follows a fallback convention of preferred defaults to less than preferred. The good news is that most modern browser environments support WebSockets, which are highly performant.

## Configuring the Hub Endpoint

For the functionality of the hub to be exposed as a consumable route, it has to configure how clients will communicate with it. There are a few things that need to be configured:

*   The desired message and transport protocols (may require additional NuGet packages)

*   The mapping of `NotificationHub` to the `/notifications` endpoint

*   The registration of `TwitterWorkerService` as a hosted service (`Background​Ser⁠vice`)

Since the Web.Api project targets the `net6.0` TFM and specifies `<Project Sdk="Microsoft.NET.Sdk.Web">`, SignalR is implicitly referenced as part of the SDK’s meta-package. For an overview of SDKs in .NET, see Microsoft’s [“.NET Project SDKs” documentation](https://oreil.ly/T4WRW). The default message protocol is JSON (text-based protocol), which is human-readable and convenient for debugging. However, it is far more efficient to use MessagePack, which is a binary protocol, and messages are usually half the size.

The *Web.Api.csproj* XML file includes the following among other package references:

[PRE4]

[![1](assets/1.png)](#co_exemplifying_real_time_web_functionality_CO5-1)

The `Microsoft.AspNetCore.SignalR.Protocols.MessagePack` NuGet package reference is included.

This exposes the MessagePack binary protocol. The client has to also configure MessagePack for this protocol to be used or it will fall back to the default text-based JSON protocol.

In the Web.Api project’s `Startup` class, we add SignalR and map the `NotificationHub` to the `"/notifications"` endpoint. Consider the *Startup.cs* C# file:

[PRE5]

The `Startup` class is `partial`, and it defines only the `_configuration` field and the constructor that accepts the configuration. By convention, a startup object has two methods:

`ConfigureServices(IServiceCollection services)`

This method is responsible for registering services on the service collection (commonly achieved with helper `Add{DomainService}` extension methods).

`Configure(IApplicationBuilder app, IWebHostEnvironment env)`

This method is responsible for configuring services for usage (commonly achieved with helper `Use{DomainService}` extension methods).

First, we add SignalR in the *Startup.ConfigureServices.cs* C# file:

[PRE6]

[![1](assets/1.png)](#co_exemplifying_real_time_web_functionality_CO6-1)

`IServiceCollection` has services added to it.

[![2](assets/2.png)](#co_exemplifying_real_time_web_functionality_CO6-2)

`JwtBearerOptions` are configured.

[![3](assets/3.png)](#co_exemplifying_real_time_web_functionality_CO6-3)

The `SignalR` service is configured to show detailed errors and adds `MessagePack`.

Authentication middleware is added, and this should look a bit familiar by now—it’s configured using the same Azure AD B2C tenant shown in previous chapters. It is configured to use the `"name"` as the name claim type. Since our Blazor WebAssembly app makes requests to different origins, our API needs to allow CORS.

SignalR is added, using the `.AddSignalR` extension method. Chained fluently on this call is a call to `AddMessagePackProtocol`, and as the name signifies, this will add MessagePack as the desired SignalR message protocol.

After adding these services to the startup routine, now we can configure them. Let’s have a look at the *Startup.Configure.cs* C# file:

[PRE7]

[![1](assets/1.png)](#co_exemplifying_real_time_web_functionality_CO7-1)

The `Configure` method is a convention of ASP.NET Core web apps. It configures services for DI.

[![2](assets/2.png)](#co_exemplifying_real_time_web_functionality_CO7-2)

The Web.Api project supports request localization, which is similar to localization detailed in [Chapter 5](ch05.html#chapter-five) with translation resource files and the `IString​Local⁠izer<T>` abstraction.

[![3](assets/3.png)](#co_exemplifying_real_time_web_functionality_CO7-3)

`NotificationHub` is mapped to its endpoint.

The `Configure` functionality starts by conditionally using the developer exception page when the current runtime environment is configured as `"Development"`. HTTPs redirection is used, which enforces the `https://` scheme for the API. The use of routing enables endpoint middleware services. Next, the model app’s CORS that was previously added is now being used.

In the previous chapter, we explored the concepts of localization. In the Web.Api project, we use a variation of the same approach. While all resource files use the same mechanics in this project, the concept of localization from a Web API project requires a request-specific middleware that will automatically set the appropriate `Culture` based on the HTTP request itself. The configuration routine specifies the use of several more middleware services:

`UseAuthentication`

Uses the added Azure AD B2C tenant

`UseAuthorization`

Allows APIs to be decorated with `Authorize` attributes, which require an authenticated user

`ResponseCaching`

Allows APIs to declaratively specify caching behavior

The call to `UseEndpoints` is required for SignalR, as `NotificationHub` is mapped to the `"/notifications"` endpoint. With that in place, the project is ready to serve many connected clients concurrently.

In the next section, we will examine how this data is ingested by the client app.

# Consuming Real-Time Data on the Client

Getting back to the Web.Client project, the model app for this book uses real-time data in several components and pages. To avoid opening multiple connections to the server from a single client, a shared approach for the hub connection is used. Each client will have exactly one `SharedHubConnection` instance. The `SharedHub​Con⁠nection` class has several implementations, and it’s responsible for managing a single framework-provided `HubConnection` that is shared by several components. Before we can use a `HubConnection`, we must first configure the client to support this type. The `SharedHubConnection` class shares a single `HubConnection` instance, and it’s responsible for managing the connection in a thread-safe manner.

## Configuring the Client

To configure SignalR on the client, our Web.Client project has to include two NuGet package references:

*   [`Microsoft.AspNetCore.SignalR.Client`](https://oreil.ly/P1bXb)

*   [`Microsoft.AspNetCore.SignalR.Protocols.MessagePack`](https://oreil.ly/oZLi1)

In addition to these packages, the custom `SharedHubConnection` class was registered as a singleton with the client’s service provider, enabling it as a resolvable service through DI. This was initially discussed in [“The Web.Client ConfigureServices Functionality”](ch04.html#web-client-configure-services). Only a single instance of this service will exist for the lifetime of the client app. This is an important detail as it shares a connection state with all of the consuming components and pages. Next, we’ll look at the `SharedHubConnection` implementation.

## Sharing a Hub Connection

The `SharedHubConnection` class is used by any component or page within the client app that needs to talk to the SignalR server, regardless of whether the component needs to push data to the server or whether the client subscribes to server events or both. The *SharedHubConnection.cs* C# contains the logic for sharing a single framework-provided `HubConnection`:

[PRE8]

[![1](assets/1.png)](#co_exemplifying_real_time_web_functionality_CO8-1)

`SharedHubConnection` is a `sealed partial class`.

[![2](assets/2.png)](#co_exemplifying_real_time_web_functionality_CO8-2)

`SharedHubConnection` defines several fields that are used to help manage the shared hub connection.

[![3](assets/3.png)](#co_exemplifying_real_time_web_functionality_CO8-3)

The `SharedHubConnection` constructor initializes supporting fields from the defined parameters.

[![4](assets/4.png)](#co_exemplifying_real_time_web_functionality_CO8-4)

`SharedHubConnection` explicitly implements the `IAsyncDisposable.Dispose​A⁠sync` method.

First off, notice that `SharedHubConnection` is an implementation of the `IAsync​Dis⁠posable` interface. This enables the `SharedHubConnection` class to clean up any managed resources that need to be released asynchronously.

Then the class defines several fields that are initialized during construction (or inline). They’re described as follows:

`IServiceProvider _serviceProvider`

The service provider from the client app.

`ILogger<SharedHubConnection> _logger`

A logger instance specific to `SharedHubConnection`.

`CultureService _cultureService`

Used to populate the “Accept-Language” HTTP header for requests made from the hub connection.

`HubConnection _hubConnection`

The framework-provided representation of the client’s connection to the server’s hub.

`SemaphoreSlim _lock`

An asynchronous locking mechanism used for thread-safe concurrent access. This lock is used in the shared `StartAsync` method command that is detailed later in this chapter.

The `_logger` field has access to several custom logging extension methods. These extension methods call into cached delegates created from the framework-provided `LoggerMessage.Define` factory methods. This is used as a performance optimization to avoid creating a new delegate each time a log message is logged.

The connection state is represented by the underlying `HubConnection.State` as a calculated property named `State`. When `_hubConnection` is `null`, the state is shown as `Disconnected`.

Additional states include the following:

`Connected`

The client and server are connected.

`Connecting`

The connection is being established.

`Reconnecting`

The connection is being reconnected.

Next, the `SharedHubConnection` constructor assigns several fields from the constructor’s parameters. From the client’s configured options object, the Web API server URL is used along with the `"/notifications"` route to instantiate the notification hub `Uri`. The `_hubConnection` field is instantiated using the builder pattern and the corresponding `HubConnectionBuilder` object.

The hub URL is used with the builder instance, and the hub connection has its options configured through the `WithUrl` method overload. `AccessTokenProvider` is assigned to a delegate used to get the contextual access token asynchronously. The default request HTTP headers are updated, adding the `"Accept-Language"` header with a value of the currently configured ISO two-letter language name. This ensures that the SignalR server connection knows to return the appropriately localized content to the connected client. The builder configures automatic reconnection and the MessagePack protocol just before calling `Build`.

Using the `_hubConnection` instance, the `Closed`, `Reconnected`, and `Reconnecting` events are subscribed to. The various connection states are communicated through these events. Their corresponding event handlers are all fairly similar. The app conditionally logs their occurrence.

Finally, the `DisposeAsync` functionality unsubscribes from the `_hubConnection` events and then cascades disposal of the connection and the locking mechanism used for synchronization.

### Shared hub connection authentication

The `SharedHubConnection` use is `partial`, and there are several other implementations to consider. The `GetAccessTokenValueAsync` delegate was assigned when building the `_hubConnection` instance, and that functionality is implemented in the *SharedHubConnection.Tokens.cs* C# file:

[PRE9]

The `SharedHubConnection` class was registered as a singleton, but the framework-provided `IAccessTokenProvider` is a scoped service. This is why the constructor couldn’t require `IAccessTokenProvider` directly; instead, it needs `IServiceProvider`. With the `_serviceProvider` instance, a call to `CreateScope` is used to create a scope in which we can resolve `IAccessTokenProvider`.

###### Tip

Normally, you will not need to use `IServiceProvider` directly. The `SharedHubConnection` class is a singleton, and `IAccessToken​Pro⁠vider` is a scoped service. `IServiceProvider` is used to resolve `IAccessTokenProvider` when the `SharedHubConnection` object starts communicating with a server.

With `tokenProvider`, we call `RequestAccessToken`. If the `result` has an access token, it is returned. If `GetAccessTokenValueAsync` is unable to get `accessToken`, it is logged, and `null` is returned. The access token is used to authenticate the connected Blazor client with the server hub.

### Shared hub connection initiation

Due to the shared nature of this class, the start functionality needs to be implemented in a thread-safe way. Any consumer can safely call `StartAsync` to initiate the connection from the client to the server. This happens in the *SharedHubConnection​.Com⁠mands.cs* C# file:

[PRE10]

[![1](assets/1.png)](#co_exemplifying_real_time_web_functionality_CO9-1)

The `StartAsync` method defines an optional cancellation `token`.

When a call to `StartAsync` is made, the `SemaphoreSlim _lock` variable has its `WaitAsync` method called, which completes when the semaphore is entered. This is an important detail because it alleviates the concerns of multiple components calling `StartAsync` concurrently by ensuring that all callers execute sequentially. In other words, imagine three components call `StartAsync` at the same time. This asynchronous locking mechanism ensures that the first component to enter and start `_hubConnection` is the only component that will call `_hubConnection.StartAsync`. The other two components will log that they were unable to start the connection to the server’s hub, as it was already started.

### Shared hub connection chat

Next, let’s look at how `SharedHubConnection` implements the chat functionality. You can see how this is defined in the *SharedHubConnection.Chat.cs* C# file:

[PRE11]

[![1](assets/1.png)](#co_exemplifying_real_time_web_functionality_CO10-1)

The `JoinChatAsync` method is an example of an operation that can be called from the client and invokes a method on the server.

[![2](assets/2.png)](#co_exemplifying_real_time_web_functionality_CO10-2)

The `SubscribeToUserLoggedIn` method is an example of an event that is fired from the server, and clients can listen by subscribing to them.

The chat functionality relies on two shared helper classes:

`HubClientMethodNames`

Defines method names that are invocable from a connected client on the server

`HubServerEventNames`

Defines event names (and parameter details) from the SignalR hub that a client can subscribe to

The additional functionality is implemented using these classes. Each client method delegates out to a corresponding overload of the `_hubConnection.Invoke​A⁠sync` method, passing the appropriate method name and arguments. Meanwhile, each server event is subscribed from an assigned function that acts as its callback handler. This is possible using the appropriate `_hubConnection.On` overload. These subscriptions are represented as an `IDisposable` that is returned, and it’s the caller’s responsibility to unsubscribe by calling `Dispose` on any subscriptions it may have made. Consuming components will be able to join and leave chat rooms, post and update messages in said chat rooms, and share whether they’re currently typing. Likewise, these components will be able to receive notifications when another user is typing, when a user has logged in or out, and when a message has been received.

### Shared hub connection tweets

The final bit of functionality that is implemented in this `SharedHubConnection` is tweet streaming, and it’s defined in the *SharedHubConnection.Tweets.cs* C# file:

[PRE12]

[![1](assets/1.png)](#co_exemplifying_real_time_web_functionality_CO11-1)

The *Tweet* implementation relies on `HubClientMethodNames` to invoke hub connection methods, given their name and arguments.

[![2](assets/2.png)](#co_exemplifying_real_time_web_functionality_CO11-2)

Similarly, `HubServerEventNames` are used to subscribe to named events from the server, given a handler.

By encapsulating the logic for each domain-specific feature, the corresponding `partial` implementations of `SharedHubConnection` expose more meaningful methods to the consumers. The framework-provided `HubConnection`, while used internally within this class, is abstracted away. Instead, by using `SharedHubConnection`, a consumer can call more explicitly named and meaningful methods.

## Consuming Real-Time Data in Components

The only thing that is left to do is to consume the shared hub connection where it’s needed in the consuming components. Each domain-specific feature, whether it’s a small component or a page, will rely on `SharedHubConnection` to provide the necessary functionality.

The SignalR real-time data powers three of our model app’s components: `NotificationComponent`, `Tweets`, and `Chat` pages. The notification system is capable of receiving notifications for the following events:

*   When a user logs in or out of the app

*   When there’s an important weather alert for your current location, such as a severe weather warning

*   If your email address has been part of a data breach (this refers to the “Have I Been Pwned” functionality of the app), as shown in [Figure 6-4](#pwned-notification)

![](assets/lblz_0604.png)

###### Figure 6-4\. A pwned notification

All notifications are dismissible, but only some are actionable. For example, a notification that informs you as to whether you’ve been part of a data breach provides a link. If you decide to follow the link, it will take you to the `/pwned` subroute in the app that will show you all of the data breaches your email is part of.

The app has a `Tweets` page dedicated to live Twitter content that’s streamed in real time. We’re going to focus in depth on one of the consuming components. With that knowledge, you will be able to review the others yourself. Let’s take a look at the chat functionality.

The `Chat` component defines the `@page` directive, which means it’s a *page*. It’s navigable at the `/chat` route. Consider the *Chat.razor* file:

[PRE13]

[![1](assets/1.png)](#co_exemplifying_real_time_web_functionality_CO12-1)

The `Chat` page has a route template of `"/chat/{room?}"`.

[![2](assets/2.png)](#co_exemplifying_real_time_web_functionality_CO12-2)

Each chat room has a single pair of inputs for the chat room message and a send button.

[![3](assets/3.png)](#co_exemplifying_real_time_web_functionality_CO12-3)

When there are one or more users actively typing, we display specialized messages to indicate this to participants in the chat room.

[![4](assets/4.png)](#co_exemplifying_real_time_web_functionality_CO12-4)

A collection of chat room messages are iterated over and passed to `ChatMessageComponent`.

The `Chat` page’s route template allows for an optional room parameter. This value is implicitly bound to the component’s corresponding `Room` property. Route templates are powerful, and we have a lot of flexibility. This allows users of our client app to share and bookmark rooms. They can invite their friends and interact in real time. For more information about route constraints, see Microsoft’s [“ASP.NET Core Blazor Routing and Navigation” documentation](https://oreil.ly/397Vn).

The chat room functionality enables users to edit messages they’ve sent; this is a nice feature to have. It lets the chat user fix typos or update what they’re trying to express as needed. Messages are, however, not persisted. This is intentional; every interaction is live, and if you leave, so too do the messages. It imposes an either *be in the moment or don’t bother* mentality. The progression of sending a message with a typo, from correcting it to sending it, is an interactive experience. To visualize this, see Figures [6-5](#chat-room-typo), [6-6](#chat-room-typo-edit), and [6-7](#chat-room-typo-edited).

![](assets/lblz_0605.png)

###### Figure 6-5\. Chat room message typo

![](assets/lblz_0606.png)

###### Figure 6-6\. Chat room message editing

![](assets/lblz_0607.png)

###### Figure 6-7\. Chat room message edited

Programmatically speaking, not persisting messages makes the app a bit less complex. The primary concern is the user’s ability to interact with the `Chat` room by creating or updating their chat messages. The user enters their message in `<input type="text">` and sends the message using the `<a class="button">` HTML elements. `input` has its native `spellcheck` attribute set to `true`. This enables the element to provide help to the user, ensuring the spelling accuracy of their messages. The user can send a message using the Enter key. The send button is an explicit user request, as opposed to the more passive or implicit nature of pressing the Enter key, but they’re functionally equivalent.

As part of the real-time functionality, when the users in the same chat room are typing a message, their client apps are debouncing their input.

When the user first starts typing, a notification is triggered using SignalR to let interested chat room participants know that the user is typing. Each time they type a nonterminating key, after a specific amount of time like 750 milliseconds or so, the app sends a cancellation. The UX is such that you can see not only that someone in the chat room is typing but also their names. This is depicted in [Figure 6-8](#debounce-diagram).

![](assets/lblz_0608.png)

###### Figure 6-8\. The debounce state machine diagram

The `Chat` page maintains a .NET `Dictionary<Guid, ActorMessage>` named `_messages`. This collection is tethered to the receiving of SignalR events from `NotificationHub` on the server through the generic `Notification<T> where T : notnull` and `T` represents the type for the `Payload` property. When communicated as `NotificationType.Chat`, the `T` type is `ActorMessage`. An actor message is used to represent the message from a user and their intent. These messages can reflect a message in multiple ways, whether the user is editing a message or whether the message is a general greeting. The messages are uniquely identifiable and immutable. A message has a sense of ownership in that there’s a username associated with a message. Consider the *Actors.cs* C# file:

[PRE14]

This file contains three `record class` definitions: one base `Actor` and two descendants, `ActorAction` and `ActorMessage`. Each message in the `_messages` collection is iterated over in reverse order. This displays the messages in ascending order from the time they were posted, which is common in all chat apps. The `ActorAction` class sets the user’s typing status to either `true` or `false`. These `message` objects are passed to the custom `<ChatMessageComponent>`. This component is defined in the *ChatMessageComponent.razor* file. Let’s have a look at that first:

[PRE15]

[![1](assets/1.png)](#co_exemplifying_real_time_web_functionality_CO13-1)

Each message is represented as an `<a id="@Message.Id">...</a>` anchor element.

[![2](assets/2.png)](#co_exemplifying_real_time_web_functionality_CO13-2)

The framework-provided `MarkupString` is used to render a C# `string` as HTML.

[![3](assets/3.png)](#co_exemplifying_real_time_web_functionality_CO13-3)

The component dynamically applies a style from the `_dynamicCss` calculated property.

[![4](assets/4.png)](#co_exemplifying_real_time_web_functionality_CO13-4)

The `StartEditAsync()` method is used to signal to the parent `Chat` page that this component is editing a message.

`ChatMessageComponent` is used to represent a single chat message. If the component is created with `IsEditable` set to `true`, the user can edit the message within this component. If a message has been edited before, it’s appropriately styled to indicate that to the users of the chat room. When the user is not permitted to edit a message, the `is-unselectable` style is applied.

Next, let’s explore how the `Chat` page is implemented as a few C# partial classes. Consider the *Chat.razor.cs* C# file:

[PRE16]

[![1](assets/1.png)](#co_exemplifying_real_time_web_functionality_CO14-1)

The `Chat` implementation maintains a `Stack<IDisposable>` named `_subscriptions`.

[![2](assets/2.png)](#co_exemplifying_real_time_web_functionality_CO14-2)

The `ShareHubConnection HubConnection` property is injected.

[![3](assets/3.png)](#co_exemplifying_real_time_web_functionality_CO14-3)

The class provides an `override` of the `OnInitializedAsync` method.

[![4](assets/4.png)](#co_exemplifying_real_time_web_functionality_CO14-4)

The `OnAfterRenderAsync` lifecycle event method is used to set the focus on the message input.

[![5](assets/5.png)](#co_exemplifying_real_time_web_functionality_CO14-5)

An explicit implementation of `IAsyncDisposable.DisposeAsync` performs cleanup.

The first implementation `partial` that we observe of the `Chat` component class implements `IAsyncDisposable`. The component exposes a `[Parameter] public string? Room` property. This is automatically bound (meaning its value is provided by the framework from a corresponding segment in the browser’s URL) to the navigation route. In other words, if the user visits `/chat/MyCoolChatRoom`, this `Room` property will have a value of `"MyCoolChatRoom"`. When there isn’t a room name specified, the default room name of `"public"` is used.

When the component is initialized, it subscribes to the following events:

`HubConnection.SubscribeToMessageReceived`

The `OnMessageReceivedAsync` method is the handler.

`HubConnection.SubscribeToUserTyping`

The `OnUserTypingAsync` method is the handler.

When the component is disposed of, it will leave the current chat room but issue the appropriate `HubConnection.LeaveChatAsync` method call. There’s a stack of `_subscriptions` that will be unsubscribed from as well. The next `Chat` implementation `partial` is defined in the *Chat.razor.Messages.cs* C# file:

[PRE17]

[![1](assets/1.png)](#co_exemplifying_real_time_web_functionality_CO15-1)

`_messages` are represented as a collection of key/value pairs.

[![2](assets/2.png)](#co_exemplifying_real_time_web_functionality_CO15-2)

The `OnMessageReceivedAsync` method is the event handler for when messages are received from the hub connection.

[![3](assets/3.png)](#co_exemplifying_real_time_web_functionality_CO15-3)

When the user is typing and they lift their key, the `OnKeyUpAsync` method is fired.

[![4](assets/4.png)](#co_exemplifying_real_time_web_functionality_CO15-4)

To send a message, the `SendMessageAsync` method is used.

[![5](assets/5.png)](#co_exemplifying_real_time_web_functionality_CO15-5)

When the chat room user owns a message, they can start editing the message with the `OnEditMessageAsync` method.

The `Chat/Messages` implementation is all about how messages are managed. From the collection of `_messages` to a single `_message` and `_messageId`, this class contains class-scoped fields for maintaining the state of chat messages. The `_isSending` value is used to signify that a message is being sent. `_messageInput` is a framework-provided `ElementReference`. When the component is rendered for the first time, `_messageInput` is focused on using the `FocusAsync` extension method.

The `Chat.OwnsMessage` method accepts a `user` parameter and compares it to the current user in context. This prevents anyone from editing messages that they don’t have ownership of. When a message is received, the `OnMessageReceivedAsync` method is called. Since this can happen at any time, the method needs to call `StateHasChanged`. The `_messages` collection is updated with the incoming message and a `JavaScript` call to the `ScrollIntoViewAsync` method given the `Id` of the `message.Payload`. This is a JavaScript interop call using a named extension method pattern.

As the user types their chat messages, the `OnKeyUpAsync` method is invoked. If the user is currently sending a message as determined by the `_isSending` bit, it’s a NOOP (a no operation, meaning it does nothing). However, when the user presses the Enter key, the message is sent. The `SendMessageAsync` method early exits if a message is already being sent or if there is no message at all. When there is a message to send, the `HubConnection.PostOrUpdateMessageAsync` method is called.

If the user decides to edit a message, the `OnEditMessageAsync` method first ensures that the user owns the message. `_message` and `_messageId` are assigned from the message being edited, and focus is returned to the message input. The final bit of `Chat` functionality is that of the debounce implementation. For that, take a look at the *Chat.razor.Debounce.cs* C# file:

[PRE18]

[![1](assets/1.png)](#co_exemplifying_real_time_web_functionality_CO16-1)

The debounce implementation maintains `HashSet<Actor> _usersTyping`.

[![2](assets/2.png)](#co_exemplifying_real_time_web_functionality_CO16-2)

The `Chat` constructor wires up the `_debounceTimer.Elapsed` event.

[![3](assets/3.png)](#co_exemplifying_real_time_web_functionality_CO16-3)

The `InitiateDebounceUserIsTypingAsync` method is responsible for restarting `_debounceTimer` and calling `SetIsTypingAsync` with `true`.

[![4](assets/4.png)](#co_exemplifying_real_time_web_functionality_CO16-4)

The `OnUserTypingAsync` method handles the event that is fired when people in the chat room are typing.

[![5](assets/5.png)](#co_exemplifying_real_time_web_functionality_CO16-5)

The `SetIsTypingAsync` method conditionally toggles a value representing the state of whether the user is typing.

[![6](assets/6.png)](#co_exemplifying_real_time_web_functionality_CO16-6)

The `TryGetUsersTypingText` helper method gets a message to display when users are typing.

[![7](assets/7.png)](#co_exemplifying_real_time_web_functionality_CO16-7)

After the allotted amount of debounce time, the `OnDebounceElapsed` method is called, thus clearing the typing status.

The `Chat/Debounce` implementation manages the collection of `_usersTyping`, `_debounceTimer` (which comes from the `System.Timers.Timer` namespace), and a value indicating whether or not the user `_isTyping`.

When the `OnUserTypingAsync` method is called, the `Notification<ActorAction>` parameter provides a value as to whether the user is typing. The user is either added or removed from the `_usersTyping` collection. The `TryGetUsersTypingText` helper message relies on the current state of the `_usersTyping` collection and `Localizer` to format messages. For example, if my friends Carol and Chad were both typing a message, the UI would look similar to [Figure 6-9](#chat-room).

![](assets/lblz_0609.png)

###### Figure 6-9\. The chat room with multiple people typing

# Summary

In this chapter, you learned how to implement real-time web functionality using ASP.NET Core SignalR. You saw how to cleanly separate domain responsibilities making extensive use of C# partial classes. We walked through the source code for a feature-rich server-side SignalR implementation, complete with `Hub` and `IHub​Con⁠text<T>` within a `BackgroundService`. You learned possible ways to create real-time alerts and notifications, a messaging system for live-user interactions, and a joinable active Twitter stream. Finally, you learned how to consume this data from our Blazor WebAssembly app while focusing on a feature-rich chat app.

In the next chapter, you’ll learn a valid use case for C# source generators. You’ll see how well-known JavaScript Web APIs can be used to source generate extension method implementations, fulfilling JavaScript interop functionality. You’ll learn how this specifically applies to the `localStorage` JavaScript API.