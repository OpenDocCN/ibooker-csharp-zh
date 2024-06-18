# Chapter 3\. Migrating a Legacy .NET Framework Application to AWS

In the previous chapters, we have seen some of the exciting tools and services that AWS gives us as developers. We are next going to take a look at what we can do with some of our legacy .NET applications and explore what is made possible by moving them to the cloud.

Software development is not, as we’re sure you are uncomfortably aware, a pursuit solely of greenfield projects, clean repos, tidy backlogs, and the latest toolsets. Organizations of all sizes can have legacy code, some of which may still be running on premises, internal tools, APIs, workflows, and applications that are actively used but not actively maintained. Migrating these to the cloud can provide your organization with cost savings, increased performance, and a drastically improved ability to scale.

In this chapter, you will learn how to choose, plan, and execute the migration of a web application running on IIS and built on either .NET Framework or .NET Core/6+.

###### Note

With the release of .NET 5 in November 2020, Microsoft has renamed .NET Core to simply “.NET”. In these next chapters, we will refer to .NET Core and all future versions as .NET and the previous, legacy version of the framework as .NET Framework.

# Choosing a Migration Path

Every .NET application will have a different path to the cloud and, while we cannot create a one-size-fits-all framework for migrating a legacy .NET application, we *can* learn from the migrations of those that came before us. In 2011, the technology research company Gartner identified [five migration strategies for migrating on-premises software to the cloud](https://oreil.ly/KFvfw). These were known as “The 5 Rs” and over the years have been refined, adapted, and expanded as new experiences emerged, growing to encompass all the challenges you might face migrating and modernizing a legacy web application.

For migrating some of your code to AWS we now have 6 Rs, any of which could be applied to a legacy .NET web application running on IIS and built in either .NET Framework or the more recent incarnation .NET:

*   Rehosting

*   Replatforming

*   Repurchasing

*   Rearchitecting

*   Rebuilding

*   Retaining

The first five of these strategies have increasing levels of effort and complexity; this, however, is rewarded with increasing value and ability to iterate going forward. We will delve deeper into some of these approaches later in this chapter.

## Rehosting

Rehosting is the process of moving an application from one host to another. It could be moving an application from running in a server room on a company’s premises to a virtual machine in the cloud, or it could be moving from one cloud provider to another. As a strategy for migration, rehosting does not change (or even require access to) the source code. It is a process of moving assets in their final built or compiled state. In the .NET world, this means *.dll* files, *.config* files, *.cshtml* views, static assets, and anything else required to serve your application. It is for this reason that rehosting is sometimes called the “lift and shift” approach to migration. Your entire application is lifted out as-is and shifted to a new host.

The advantages of rehosting your application include being able to take advantage of the cost savings and performance improvements possible on a cloud-hosted virtual machine. It can also make it easier to manage your infrastructure if you can rehost your lesser maintained or legacy applications alongside your more actively developed code on AWS.

For an overview of some of the tools and resources available, should you choose to follow this migration path, see [“Rehosting on AWS”](#3-rehosting).

## Replatforming

The replatforming approach goes one step further than simply rehosting and changes not just *where* but also *how* your application is hosted. Unlike rehosting, replatforming *could* involve changes to your code, although these changes should be kept to a minimum to keep the strategy viable.

There are many definitions of what constitutes a “platform,” but one platform we as .NET developers are all aware of is Internet Information Services (IIS) running on Windows Server. Replatforming would be the process of migrating your application *away* from IIS and onto a more cloud native hosting environment such as Kubernetes. Later in this chapter, we will explore one type of replatforming in more detail: [“Replatforming via Containerization”](#3-containerization).

## Repurchasing

This strategy is relevant when your application depends on a licensed third-party service or application that cannot run on a cloud infrastructure. Perhaps you use a self-hosted product for customer relationship management (CRM) or content management system (CMS) functionality in your application that cannot be migrated to the cloud. Repurchasing is a migration strategy for applications that rely on these products and involves ending your existing, self-hosted license, and purchasing a new license for a cloud-based replacement. This can either be a cloud-based version of a similar product (for example, Umbraco CMS to Umbraco Cloud), or a replacement product on the AWS Marketplace.

## Rearchitecting

As the name implies, rearchitecting deals with the overall architecture of your application and asks you to think about how you can make changes to facilitate its move to the cloud.^([1](ch03.xhtml#idm45599657227728)) For a legacy .NET Framework application, this will almost certainly mean moving to .NET. Microsoft in 2019 announced that version 4.8 will be the last major release of .NET Framework and, while it will continue to be supported and distributed with future releases of Windows, it will not be actively developed by Microsoft.^([2](ch03.xhtml#idm45599657227072))

History has sculpted out a fairly linear journey for rearchitecting a monolithic web application, as shown in [Figure 3-1](#Figure-3-1).

![doac 0301](assets/doac_0301.png)

###### Figure 3-1\. Evolution of monolith web applications

We will take a deeper dive into porting .NET Framework to .NET in the section [“Rearchitecting: Moving to .NET (Core)”](#3-move-to-core).

## Rebuilding

Sometimes your legacy codebase fails to pass the effort versus value benchmark for migration and you have no choice but to rebuild from scratch. You are not, of course, starting entirely from scratch. You will be able to migrate some of your business logic; all those solved problems your legacy codebase has taken years to navigate through can be re-created on a new codebase. However, the code itself—the architecture, the libraries, databases, API schemas, and documentation^([3](ch03.xhtml#idm45599657216256))—will not be coming with you.

## Retaining

The final migration strategy on this list is really just *none of the above*. Perhaps your legacy application has some special requirements, cannot be connected to the internet, will have to go through a prohibitively lengthy recertification process. There are many unique and often unforeseen reasons why some legacy codebases cannot be migrated to the cloud, or cannot be migrated at the present moment in time. You should only migrate applications for which a viable business case can be made, and if that is not possible, then selecting *none of the above* is sometimes your best option.

## Choosing a Strategy

The migration strategy you choose will depend upon the current architecture of your application, where you want to get to, and how much you are willing to change in order to get there. The chart in [Figure 3-2](#Figure-3-2) summarizes the decisions you can make in order to choose the migration path most appropriate for your individual use case.

![doac 0302](assets/doac_0302.png)

###### Figure 3-2\. Choosing a strategy

## AWS Migration Hub Strategy Recommendations

For further assistance in choosing a migration strategy, AWS offers a tool called [AWS Migration Hub Strategy Recommendations](https://oreil.ly/56iyO). This service gathers data from your existing servers, augments it with analysis of your source code and SQL schemas, then recommends a migration strategy from those we have covered previously.

###### Tip

Strategy Recommendations is part of the AWS Migration Hub: a set of tools for analyzing an infrastructure planning for and then tracking a migration to AWS. The Migration Hub is available at no additional charge; you only pay the cost of any tools you use and any AWS resources consumed in the process

To get started with Strategy Recommendations, we first need to give it as much data about our existing infrastructure as possible. We can do this with the [AWS Application Discovery Service](https://aws.amazon.com/application-discovery), another service accessible from the AWS Migration Hub. To start application discovery, navigate to [discovery tools in the AWS Management Console](https://oreil.ly/Vkp65) for your home region^([4](ch03.xhtml#idm45599657199024)) and choose from one of three methods to collect the data shown in [Figure 3-3](#Figure-3-3).

![doac 0303](assets/doac_0303.png)

###### Figure 3-3\. Application Discovery Service collection methods

Let’s walk through a quick setup using the discovery agent. Before we start, ensure you have the AWS CLI installed and you have an IAM user access key (and secret). You can use the following commands to save these values in your AWS configuration files. These settings will then be used by all the tools in this chapter.

[PRE0]

Next, open up a PowerShell terminal on one of the Windows servers you want to begin collecting data for, then download the agent installer:

[PRE1]

Next set your home region for the AWS Migration Hub, your access key ID, and secret.

And run the discovery agent installer on this server:

[PRE2]

This will install the agent into the new folder you created, *C:\ADSAgent*. Back in the Migration Hub section of the AWS Management Console, you can navigate to Discover → Data Collectors → Agents and, if all has gone well, the agent you installed should appear in the list. Select the agent and click Start Data Collection to allow ADS to begin collecting data about your server.

###### Warning

If your agent does not show up, ensure your server is allowing the agent process to send data over TCP port 443 to *https://arsenal-discovery.<your-home-region>.amazonaws.com:443*.

The discovery agent will poll its host server approximately every 15 minutes and report data including CPU usage, free RAM, operating system properties, and process IDs of running processes that were discovered. You will be able to see your servers in the Migration Hub by navigating to Discover → Servers on the dashboard. Once you have all your servers added to ADS, you are ready to begin collating the data necessary for strategy recommendations.

The Strategy Recommendations service has an automated, agentless data collector you can use to analyze your .NET applications running on the servers you now have in ADS. To get started navigate to Strategy → Get Started in the Migration Hub console and follow the wizard to download the data collector as an Open Virtual Appliance (OVA). This can then be deployed to your VMware vCenter Server. Full instructions for setting up the data collector can be found in the [AWS documentation](https://oreil.ly/LRIeN).

Once your data collector is set up, you can move to the next page of the wizard and select your priorities for migration. [Figure 3-4](#Figure-3-4) shows the priorities selection screen in Strategy Recommendations. This will allow AWS to recommend a migration strategy that best fits your business needs and plans for the future. Select the options that most align with your reasons for migrating.

![doac 0304](assets/doac_0304.png)

###### Figure 3-4\. Strategy Recommendations service goals

After running data analysis on your servers, the service will give you recommendations for each application, including a link to any relevant AWS tools to assist you with that type of migration. In [Figure 3-5](#Figure-3-5), you can see the tool is recommending we use rehosting as a migration strategy onto EC2 using the Application Migration Service, which we will look at next.

![doac 0305](assets/doac_0305.png)

###### Figure 3-5\. AWS recommending the rehost strategy

# Rehosting on AWS

The approach for rehosting your legacy .NET application onto AWS will vary depending on how your application is currently hosted. For web apps deployed to a single server running IIS, you can easily replicate this environment on an Amazon EC2 instance on AWS. If your .NET application is currently deployed to a managed environment (an “app service” as other cloud providers might call it), then the equivalent on AWS is Elastic Beanstalk, and you should find the experience of working with Elastic Beanstalk familiar. We will cover migrating managed hosting to Elastic Beanstalk later on in the section [“Elastic Beanstalk”](#3-elastic-beanstalk), but first, let’s take a look at the case of rehosting a virtual machine running IIS over to EC2.

## Application Migration Service (MGN)

The latest AWS offering for performing a lift-and-shift migration to EC2 is called the *Application Migration Service*, or MGN.^([5](ch03.xhtml#idm45599657010544)) This service evolved from a product called CloudEndure that AWS acquired in 2018\. CloudEndure is a disaster recovery solution that works by creating and maintaining replicas of your production servers on AWS EC2 and Elastic Block Store (EBS). This replication concept can be repurposed for the sake of performing a lift-and-shift rehosting. You simply set up replication to AWS then, when you are ready, switch over to running your application exclusively from your AWS replicas, allowing you to decommission the original server. An overview of how the Application Migration Service replicates your servers is shown in [Figure 3-6](#Figure-3-6).

![doac 0306](assets/doac_0306.png)

###### Figure 3-6\. Overview of the Application Migration Service

The Application Migration Service is accessed via the AWS Management Console; type “MGN” into the search or find it in the menu of the Migration Hub as in [Figure 3-7](#Figure-3-7).

![doac 0307](assets/doac_0307.png)

###### Figure 3-7\. Access MGN through the AWS Management Console

Set up of the Application Migration Service begins by installing the replication agent onto your servers, similar to how we installed the discovery agent for the Application Discovery Service. First log onto your Windows server and open up a PowerShell window as Administrator to download the agent installer for Windows. Replace `<region>` with the AWS region you would like to migrate your servers into:

[PRE3]

Next put your region, access key ID, and secret into variables if you haven’t already and execute the installer:

[PRE4]

The agent installer will ask you which disks on this server you want to replicate and then will get to work syncing your disks to AWS. You can see the status of the agent installer operations in the console window as shown in [Figure 3-8](#Figure-3-8).

![doac 0308](assets/doac_0308.png)

###### Figure 3-8\. MGN replication agent console window

If you head back over to the MGN Management Console, you will be able to see your server under Source Servers in the menu. Click on your server name and you can see the status of the replication for this server as shown in [Figure 3-9](#Figure-3-9). It can take a while to get through all the stages, but once complete, the status in the console will change to “Ready For Testing.” One nice feature of the Application Migration Service is the ability to spin up an instance of a server from a replica and test that everything is as you expect it to be, without interrupting or otherwise interfering with the replication itself.

![doac 0309](assets/doac_0309.png)

###### Figure 3-9\. MGN replication status in the Management Console

To test a server, select “Launch test instances” from the “Test and Cutover” action menu of your source server. This will launch a new EC2 instance that should mirror the original Windows server you replicated, with transfer of licenses handled automatically by the Application Migration Service. You can connect to the EC2 instance with Remote Desktop (RDP) by selecting it in the list of EC2 instances from the Management Console once it becomes ready. When you are happy that the test instance is working the way you expect it to, you can execute the final stage of the migration: cutover.

If you refer back to the stages of application migration via replication in [Figure 3-6](#Figure-3-6), you can see the final stage being Execute Cutover. This is where we create the resources for all our source servers (that is: spin up a new EC2 instance for each server), stop replication of our old servers, and allow us to decommission the original servers we installed the replication agents onto.

So now we have all our servers running like-for-like on EC2 and we have performed the lift-and-shift rehosting, what’s next? Staying inside the realm of *rehosting*, we can go one step further and take advantage of AWS’s managed environment for running a web application: Elastic Beanstalk.

## Elastic Beanstalk

Elastic Beanstalk is a *managed* hosting environment for your web application. It supports a variety of backend stacks such as Java, Ruby, or PHP, but it is the .NET Framework support that we are most interested in here. The difference between hosting our app on a Windows server on EC2, as we did in the previous section, and using Elastic Beanstalk can be distilled down to one key discrepancy.

With an unmanaged server, you upload your compiled and packaged website files to a server and then tweak the settings on that server in order to handle web traffic. With a managed service, you upload your package to the cloud, and the service will take care of the rest for you, setting up load balancers and dynamically scaling virtual machines horizontally. Managed services are the real draw for deploying your applications to the cloud and the more you lean into having AWS manage your infrastructure for you, the more you can concentrate on just writing code and solving problems for your business. Later in this book, we will cover serverless programming and how you can architect your .NET applications to be more serverless, a concept rooted in managed services as much as possible. You can think of Elastic Beanstalk as an equivalent to “App Service,” which you may be familiar with from a slightly *bluer* cloud provider.

So let’s get started with our first managed service on Elastic Beanstalk. The simplest way to deploy code is by using the AWS Toolkit for Visual Studio. With the AWS Toolkit installed, deploying to Elastic Beanstalk really is as simple as right-clicking your solution and selecting “Publish to Elastic Beanstalk.” [Figure 3-10](#Figure-3-10) shows the toolkit in use in the Solution Explorer of Visual Studio.

![doac 0310](assets/doac_0310.png)

###### Figure 3-10\. Publishing to Elastic Beanstalk directly from Visual Studio

There is of course also a CLI tool you can use in your CI pipeline. To get started with the Elastic Beanstalk CLI, see the [EB CLI Installer](https://oreil.ly/8aKRX) on GitHub. The CLI tool allows you to publish your code directly to Elastic Beanstalk from AWS CodeBuild or GitHub.

Lastly, there is one final tool worth mentioning if you are thinking of trying out Elastic Beanstalk, and that is the [Windows Web App Migration Assistant (WWMA)](https://oreil.ly/8e4Q2). This is a PowerShell script that you can run on any Windows Server with an IIS hosted web application to automatically migrate the application onto Elastic Beanstalk. This is useful if you have a legacy .NET application that is no longer maintained and you want to take advantage of the benefits of Elastic Beanstalk but you no longer perform releases for this app. It is a true *rehosting* tool that simply moves the compiled website assets from your *C:\inetpub* folder on the server into EC2 instance(s) managed and scaled by Elastic Beanstalk.

# Replatforming via Containerization

As a migration strategy, replatforming is concerned with looking at the *platform* on which our .NET application is running and exploring moving somewhere else. In the case of a .NET Framework web application, that *platform* will be some version of Windows Server running IIS. While previously we looked at Elastic Beanstalk as a way of getting more out of this infrastructure, Elastic Beanstalk still hosts your application on IIS on Windows Server, albeit in a much more scalable and efficient way. If we want to really push the envelope for scalability and performance, while still keeping away from the original source code (these are “legacy” applications, after all), then we need to move away from IIS and onto something else. This is where containerization comes in.

We’re going to skip over exactly what containerization is and why it matters as we have [Chapter 5](ch05.xhtml#Chapter5) later in this book dedicated to containerization of .NET. Suffice to say moving your legacy .NET application from Windows Server web hosting to containers unlocks both performance and cost benefits to your organization, all without having to touch any of that legacy code.

App2Container is a command-line tool from AWS that runs against your .NET application running on IIS on a Windows Server. It analyzes your application and dependencies, then creates Docker container images that can be deployed to an orchestration service in the cloud such as Elastic Container Service (ECS) or Amazon Elastic Kubernetes Services (EKS). Because App2Container runs on an application that is already deployed to a server, it doesn’t need access to your source code and sits right at the end of a deployment pipeline. For this reason, App2Container is perfect for quickly replatforming an old application that is not being actively developed and you don’t want to be rebuilding; simply skip right to the last two steps of the pipeline shown in [Figure 3-11](#Figure-3-11) and containerize the production files.

![doac 0311](assets/doac_0311.png)

###### Figure 3-11\. Deployment pipeline of .NET Framework application using App2Container

To containerize a .NET application, you first need to download and install App2Container onto the server running your application. You can find the installation package on [AWS’s website](https://oreil.ly/UIZzD). Download, unzip, and run *.\install.ps1* from an administrator PowerShell terminal on the application server. This will install the `app2container` command-line utility. If you haven’t already, make sure your application server has the AWS Tools for Windows PowerShell installed and you have a default profile configured that allows you access to manage AWS resources from your application server. If your server is running on an EC2 instance (for example, if you rehosted it using the Application Migration Service, see [“Rehosting on AWS”](#3-rehosting)), then these tools will already be installed as they are included on the Windows-based machine images used in EC2\. Once you have confirmed you have an AWS profile with IAM permissions to manage AWS resources, you can initialize App2Container by running:

[PRE5]

The tool will ask about collecting usage metrics and ask you for an S3 bucket to upload artifacts, but this is entirely optional and you can skip through these options. Once initialized, you are ready to begin analyzing your application server for running .NET applications that can be containerized. Run the `app2container inventory` command to get a list of running applications in JSON format, and then pass in the JSON key as the `--applcation-id` of the app you want to containerize, as shown in [Figure 3-12](#Figure-3-12):

[PRE6]

![doac 0312](assets/doac_0312.png)

###### Figure 3-12\. Listing the IIS sites capable of being containerized

We are encouraged to take a look at the *analysis.json* file that is generated for us, and we echo that sentiment. A full list of the fields that appear in *analysis.json* can be found in the [App2Container User Guide](https://oreil.ly/HCIa6), but it is worth spending the time exploring the analysis output as these settings will be used to configure our container. You can edit the `containerParameters` section of *analysis.json* before containerizing if required. It is also worth opening up *report.txt* in the same folder as this is where any connection strings will be added by the analyze command. When you are ready, run the `containerize` and `generate` commands to build a docker image then generate all the artifacts you need to deploy it to either ECS or EKS:^([6](ch03.xhtml#idm45599656749792))

[PRE7]

The second command here (`generate app-deployment`) will upload your containers to Elastic Container Registry (ECR) and create a CloudFormation template you can use to deploy your app to (in this case) ECS. The tool will show you the output destination of this CloudFormation template (see [Figure 3-13](#Figure-3-13)) and give you the command you need to deploy it to AWS.

![doac 0313](assets/doac_0313.png)

###### Figure 3-13\. Results of App2Container deployment generation

This has been a brief overview of App2Container from AWS, but much more is possible with this tool than we have covered here. Instead of performing the containerization on the application server itself, App2Container also lets you deploy a worker machine either to EC2 or your local virtualization environment. This would be useful if you wanted to protect the application server, which could be serving a web application in production, from having to spend resources executing a containerization process. Since App2Container is a CLI tool, it would also be simple to integrate into a full deployment pipeline for code you are still actively working on and releasing changes to. If you refer back to [Figure 3-11](#Figure-3-11), you can see how App2Container can be used to *extend* an existing .NET deployment pipeline into containerization without touching anything further upstream, including your code.

One final note on App2Container is framework version support, which has been expanding with new releases of the tool. You can use App2Container on both .NET Framework and .NET 6+ applications running on both Windows and, more recently, Linux. For .NET Framework, the minimum supported version is .NET 3.5 running in IIS 7.5\. Java applications can also be containerized with App2Container, in a similar way to that we have explored here, so it’s not just us C# developers who can benefit.

# Rearchitecting: Moving to .NET (Core)

So far we have looked at migration approaches for our .NET applications that do not involve making changes to the code, but what can we do if changing the code is acceptable? In the next chapter we will be looking at modernizing .NET applications; however, if your application is still built on .NET Framework, the first step down every modernization path will almost certainly be a migration to .NET 6+. The road ahead for .NET Framework applications is not a long one and, aside from the rehosting and replatforming approaches we have covered in this book, you will eventually be approaching the topic of migrating framework versions. .NET is, after all, a complete rewrite of Microsoft’s framework, and feature parity was not an aim. APIs have changed, namespaces like `System.Web.Services` are no longer present, and some third-party libraries that you rely on may not have been migrated, forcing you to replace them with an alternative. For these reasons, it is vital to do as much investigation as possible in order to assess the lift required in migrating your legacy .NET Framework application to modern .NET.

While there is no such thing as a tool that will automatically refactor your entire solution and convert your .NET Framework monolith to .NET 6+, what does exist are a handful of extremely useful tools to analyze your project, perform small refactoring tasks, and give you an insight into where you will find compatibility problems. I’m going to give you a brief insight into two of these tools: the .NET Upgrade Assistant from Microsoft, and the Porting Assistant from AWS.

Before you start, however, it is worth becoming familiar with which .NET Framework technologies are [unavailable on .NET 6+](https://oreil.ly/1JaqV). These include almost everything that used the Component Object Model (COM, COM+, DCOM) such as .NET Remoting and Windows Workflow Foundation. For applications that rely heavily on these Windows-only frameworks, one of the migration strategies we discussed earlier in this chapter may be more appropriate. Applications that use Windows Communication Foundation (WCF) can take advantage of the [CoreWCF project](https://github.com/CoreWCF/CoreWCF) in order to continue using WCF features on modern .NET.

## Microsoft .NET Upgrade Assistant

With the releases of .NET 5 and 6, Microsoft cemented its vision for a unified single framework going forward, with .NET 6 being the long-term support (LTS) release of the platform. In order to assist migration of .NET Framework applications to this new, unified version of the framework, Microsoft has been developing a command-line tool called the *.NET Upgrade Assistant*. The Upgrade Assistant is intended to be a single entry point to guide you through the migration journey and wraps within it the more longstanding .NET Framework conversion tool try-convert. It is a good idea to use try-convert from within the context of the Upgrade Assistant as you will get more analysis and guidance toward the strategies most applicable to your project.

The types of .NET Framework applications that this tool can be used with at time of writing are:

*   .NET class libraries

*   Console Apps

*   Windows forms

*   Windows Presentation Foundation (WPF)

*   ASP.NET MVC web applications

The .NET Upgrade Assistant has an extensible architecture that encourages the community to contribute extensions and analyzers/code fixers. You can even write your own analyzers to perform automatic code refactoring based on rules you define. The Upgrade Assistant comes with a set of default analyzers that look for common incompatibilities with your code and offer a solution. For example, the `HttpContextCurrentAnalyzer` looks for calls to the static `System.Web.HttpContext.Current`, a pattern often employed in controller actions of .NET Framework applications that will need to be refactored since `HttpContext.Current` was removed in .NET Core. In [Figure 3-14](#Figure-3-14), you can see an example of the message this analyzer emits when `HttpContext.Current` is found in your code.

So let’s get going with an upgrade. For this example, we have created a very simple ASP.NET MVC web application in Visual Studio 2019 using .NET Framework version 4.7.2\. There is a controller action that takes the query string from the web request, adds it to the `ViewBag`, then displays it on the *Index.cshtml* Razor view. The output of this website can be seen in [Figure 3-14](#Figure-3-14).

![doac 0314](assets/doac_0314.png)

###### Figure 3-14\. Example ASP.NET MVC website running in a browser

We have purposefully added a couple of things to this example that we *know* are not compatible with .NET Core and later versions of the framework. First, as introduced earlier, we have a call to `HttpContext.Current` on line 9 of *HomeController.cs* ([Example 3-1](#ex_3-1)). This will need to be replaced with a call to an equivalent HTTP context property in .NET 6\. We also have a Razor helper in *Index.cshtml*, the `@helper` syntax for which is not present in later versions of .NET ([Example 3-2](#ex_3-2)). We have this code checked into a Git repository with a clean working tree; this will help view the changes to the code that the .NET Upgrade Assistant will make by using a Git diff tool.

##### Example 3-1\. HomeController.cs

[PRE8]

##### Example 3-2\. Index.cshtml

[PRE9]

To get started with the .NET Upgrade Assistant, first install it as a .NET CLI tool:

[PRE10]

Next, in the directory that contains your solution file, run the upgrade assistant and follow the instructions to select the project to use as your entry point:

[PRE11]

Depending on the type of .NET Framework project you have (WebForms, WPF etc.), the upgrade assistant will give you a different set of steps. We have an ASP.NET MVC Web Application, so we are offered a ten-step process for the upgrade, as follows:

[PRE12]

Most of the steps are self explanatory; step 2 in my example is “Convert project to SDK style”. This means reformatting the *.csproj* file to the newer .NET format that starts with `<Project Sdk="Microsoft.NET.Sdk">`. As you go through these steps, use a source control diff tool (e.g., KDiff, VS Code, or the Git Changes window in Visual Studio) to see the changes being made to your code and project files.

The logs from the upgrade assistant are stored in Compact Log Event Format (CLEF) inside the directory in which you ran the tool. It will also create a backup of your project; however, this is not particularly useful if you have everything checked into source control (you *do* have everything checked into source control, right?).

You can see from the preceding list that step 9c was `Do not use HttpContext.Current`. This is coming from the `HttpContextCurrentAnalyzer` we introduced earlier, and the fix from this analyzer will change all usage of `HttpContext.Current` in your code to `HttpContextHelper.Current`. We do still get a warning about `HttpContextHelper.Current` being obsolete and to use dependency injection instead; however, this doesn’t prevent my upgraded code from compiling. The upgrade assistant also refactored my `@helper` syntax in the Razor view (step 8b) and replaced it with a .NET 6 compatible helper method. The code after running the upgrade assistant looks like this (Examples [3-3](#ex_3-3) and [3-4](#ex_3-4)):

##### Example 3-3\. HomeController.cs

[PRE13]

##### Example 3-4\. Index.cshtml

[PRE14]

## AWS Porting Assistant

Another tool you can use to aid your migration from .NET Framework is the porting assistant provided by AWS themselves. This is a Windows application that you download to the machine on which you have your .NET Framework solution. Although the porting assistant runs locally on your code, it will need to connect to AWS in order to retrieve NuGet package upgrade information from an S3 bucket; it is for this reason that you need to set it up with a local AWS profile as shown in [Figure 3-15](#Figure-3-16). No resources will be created on your AWS profile. The porting assistant can be downloaded from the [AWS Porting Assistant page](https://oreil.ly/31UMk).

![doac 0315](assets/doac_0315.png)

###### Figure 3-15\. Running the .NET Porting Assistant from AWS

If we step through the wizard and use the same ASP.NET MVC web application we used for the Microsoft Upgrade Assistant, we can see that it has correctly identified the solution is targeting .NET Framework 4.7.5, and we have 7 incompatible NuGet packages and 15 incompatible APIs. This is the *15 of 17* values highlighted in [Figure 3-16](#Figure-3-17). You can see more information about these in the relevant tabs of the porting assistant including links to the code that has been identified as incompatible.

![doac 0316](assets/doac_0316.png)

###### Figure 3-16\. AWS Porting Assistant analysis results

When you are ready, click Port Solution to begin making the changes to your project files. The application will ask you where you would like to save the ported solution to; having your code in source control means you can choose “Modify source in place”. Unlike Microsoft’s tool, the AWS Porting does actually let you fine-tune which versions you would like to upgrade your NuGet packages to in the UI. For this example, we have simply left the defaults in place for all packages and stepped through the assistant. You can see now from [Figure 3-17](#Figure-3-18) that the project files have been upgraded and the project is now showing in the porting assistant as .NET 6.

![doac 0317](assets/doac_0317.png)

###### Figure 3-17\. AWS Porting Assistant complete

If you click the link to open in Visual Studio, you will see the solution loads and your NuGet packages will have been upgraded to versions compatible with .NET Core/6+. Where the porting assistant’s help ends, however, is by refactoring all the unsupported code. When we try to build this, we will still get errors that `System.Web.HttpContext` does not exist and “the helper directive is not supported,” so it may be worth trying out all tools available and comparing your results. Overall, the .NET Porting Assistant from AWS does provide a very quick and accessible UI for visualizing and assessing the effort involved in refactoring your .NET Framework code to work with modern .NET.

# Conclusion

The strategies and tools we have covered in this chapter will help you move an existing application running on IIS to AWS. From the basic [“Rehosting on AWS”](#3-rehosting) of an application without touching the original source, to [“Rearchitecting: Moving to .NET (Core)”](#3-move-to-core) and making inroads into modernization of your codebase. Whichever strategy you choose, you will benefit from at least some of the advantages of running .NET in the AWS cloud; however, it is worth considering the next steps for your application. Some of the approaches we have covered here will leave your code in the exact same state as it was before you migrated. If you were not actively developing your codebase before the migration, then you will not be in a much better position to do so after choosing one of these paths. For a codebase you intend to continue to develop, to iterate on, and to add functionality to, you should to modernize your legacy application. Modernization will involve not just moving to .NET 6+ (although that will be a prerequisite); it will also involve replacing third-party dependencies, replacing external services, and refactoring the architecture of your application to use patterns that better exploit the cloud environment you are now running in. All these will be covered in the next chapter, in which we will cover the term *serverless* and how it can apply to you as a .NET developer.

# Critical Thinking Discussion Questions

*   Which is the easiest for a small start-up of the six Rs in a migration strategy?

*   Which is the easiest for a large enterprise of the six Rs in a migration strategy?

*   What are strategies to programmatically convert thousands of Windows services to containers using App2Container?

*   What is a robust enterprise use case for the Porting Assistant for .NET?

*   What are the advantages of Elastic Beanstalk applications in migration from on-premise to the AWS Cloud in an enterprise setting?

# Exercises

*   Build a hello world Blazor app on a Windows Server, then use App2Container to convert it to a Docker container image.

*   Deploy the Strategy Recommendations collector on an Amazon EC2 instance.

*   Run the [AWS Migration Hub Orchestrator](https://oreil.ly/bRnkz).

*   Build a hello world Blazor app on a Windows Server, then use App2Container to convert it to a Docker container image and deploy it to AWS App Runner.

*   Build a hello world Blazor app on a Windows Server, then use App2Container to convert it to a Docker container image and deploy it to Fargate using [AWS Copilot](https://oreil.ly/6Zzes).

^([1](ch03.xhtml#idm45599657227728-marker)) This migration strategy is sometimes called “refactoring”; however, this can be a chameleon of a term, so I’ll be sticking to “rearchitecting” for this book.

^([2](ch03.xhtml#idm45599657227072-marker)) You can find the .NET Framework support policy [here](https://oreil.ly/FJImk).

^([3](ch03.xhtml#idm45599657216256-marker)) I will admit to looking up from my screen and taking a long sip of coffee before adding “documentation” to this list. My cat gave me the same knowing stare that you are.

^([4](ch03.xhtml#idm45599657199024-marker)) Your home region in the AWS Migration Hub is the region in which migration data is stored for discovery, planning, and migration tracking. You can set a home region from the Migration Hub Settings page.

^([5](ch03.xhtml#idm45599657010544-marker)) Interestingly, the three letter abbreviation *MGN* for Application Migration Service is a contraction and not an initialism. Perhaps *AMS* was too similar to *AWS*.

^([6](ch03.xhtml#idm45599656749792-marker)) We will revisit these two container orchestration services later in this book, but in a nutshell ECS is a simpler and more “managed” service for running containers without the added complexity of dealing with Kubernetes.