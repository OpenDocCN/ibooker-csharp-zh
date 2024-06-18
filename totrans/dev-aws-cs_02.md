# Chapter 2\. AWS Core Services

The most popular raw materials to build a house include steel, brick, stone, and wood. These raw materials each have unique properties that enable a home builder to construct a house. Similarly, AWS Core Services are the foundational materials necessary to build complex systems. From a high level, these services include computing and storage.

When home builders construct a home, they follow comprehensive building codes, typically pulled from an international standard known as the [I-Codes](https://oreil.ly/MLp72). According to the International Code Council, these modern safety codes aim to help ensure the engineering of “safe, sustainable, affordable and resilient structures.” Similarly, the [AWS Well-Architected Framework](https://oreil.ly/Nxhnd) provides “guidance to help customers apply best practices in design, delivery, and maintenance of AWS environments,” according to AWS.

These [general design principles](https://oreil.ly/IVCQk) are critical in understanding how to use AWS Core Services effectively. Let’s briefly discuss them:

Stop guessing your capacity needs

In a nutshell, it is a poor strategy to guess capacity. Instead, a system should incorporate the ability to add or remove resources dynamically.

Test systems at production scale

The cloud allows for fully automated provisioning of resources; this allows a developer to test an application in an identical environment to production. The ability to replicate a production environment solves the “it works on my machine” problem endemic to the software engineering industry.

Automate to make architectural experimentation easier

Automation results in less work over the long term, allowing the developer to audit, track changes, and revert them if needed.

Allow for evolutionary architectures

The core idea is to design a system you expect to change. When designing software, we should consider dynamic natural systems such as trees and rivers versus static systems such as bridges or roads.

Drive architectures using data

Data science is a popular discipline, but it doesn’t confine itself only to business problems. Software systems need to use data science to identify the changes necessary to keep the system performing as designed.

Improve through game days

An adequately architected system needs to have a complete simulation run regularly; it may be impossible to test all scenarios thoroughly without this.

Keep these principles in mind as we move through the chapter. Next, let’s discuss AWS storage in more detail.

# AWS Storage

Storage is an excellent example of a topic that, on the surface, is simple but can quickly explode into complexity. At the core of storage for AWS is Amazon Simple Storage Service, also known as [AWS S3](https://docs.aws.amazon.com/s3/index.xhtml). It launched in 2006 as one of the first services available to the public from AWS. As of 2021, there are 100 trillion objects stored in S3.^([1](ch02.xhtml#idm45599658074352)) If you want an example of “big data,” this is as good as it gets.

While S3 is the first storage solution launched by AWS, there are many other options. Using a .NET developer-centric view, one way to begin is to divide storage into two broad categories: core storage and database.

Let’s break down both briefly.

Core storage

Core storage refers to the low-level storage components used by all services on AWS, and examples include both object-storage Amazon S3 and Amazon Elastic Block Store (EBS). AWS core storage options include object-storage Amazon S3, [Amazon Elastic Block Store (EBS)](https://aws.amazon.com/ebs), fully managed network file storage such as [Amazon File System (EFS)](https://aws.amazon.com/efs), and [Amazon FSx for Windows File Server](https://aws.amazon.com/fsx/windows) (an example of one type of FSx option) and finally, utility storage services which include [AWS Backup and Storage Gateway](https://oreil.ly/3Zz6U).

Database

At a high level, a database is an organized collection of data accessed from a computer system. Database storage options include relational databases, including [Amazon RDS](https://aws.amazon.com/rds), key-value databases, [Amazon DynamoDB](https://aws.amazon.com/dynamodb), and special-purpose databases like [Amazon Neptune](https://oreil.ly/XdkCw) (to query graph-based data).

With this breakdown out of the way, let’s dive deeper into developing with S3 storage. S3 storage is critical to master since it offers various cost-effective tiers for dealing with object data.

## Developing with S3 Storage

One way to look at S3 is as a high-level service that stores and retrieves objects at scale. For a developer, this perspective allows you to focus on developing the business logic for an application versus managing a high-performance object storage system. Let’s first identify the key benefits of Amazon S3:

Durability

*Durability* refers to the concept of ensuring data is not lost. S3 Standard storage tier provides 99.999999999% (or eleven 9s) of [durability](https://aws.amazon.com/s3/faqs).

Availability

*Availability* refers to the concept of accessing your data when you need it. The S3 Standard storage tier has 99.99% (or four 9s) of availability.

Scalability

*Scalability* is the ability to increase capacity to meet demand. S3 offers near-infinite capacity in disk I/O and storage as a managed service. It can store single objects of 5 TB or less.

Security

S3 has both fine-grained access control and encryption in transit and rest.

Performance

The performance of the S3 filesystem supports many different access patterns, including streaming, large files, machine learning, and big data.

There are a few different ways to interact with S3 as a developer. The first option is through the .NET SDK. The example in [Chapter 1](ch01.xhtml#Chapter1) that lists the buckets in S3 is an excellent example of that process. You can also interact with S3 indirectly by using managed services that use S3\. Examples of these managed services that use S3 include AWS Athena^([2](ch02.xhtml#idm45599658047392)) and AWS SageMaker. With [Athena](https://aws.amazon.com/athena), you can query all of S3 via serverless SQL queries but don’t need to worry about the logistics of dealing with servers. Likewise, Amazon SageMaker,^([3](ch02.xhtml#idm45599658045184)) a fully managed machine learning service, heavily uses S3 to train machine learning models and store the model artifacts. A key reason for that is that S3 is serverless and scales without needing management by the user.

Another way to use S3 is by tapping into the [S3 Lifecycle configuration](https://oreil.ly/g0lpM). The Lifecycle configuration allows sophisticated automated workflows to migrate data to different storage tiers and archive data. Let’s look at these storage classes.

S3 Standard

Ideal for frequently accessed data and helpful for various use cases, including cloud native applications, content, gaming, and big data.

S3 Standard IA (infrequent access)

This storage class has the same benefits as S3 Standard but offers a different cost model, making it ideal for items like older log files since retrieval is a higher cost.

S3 One Zone-IA

The One-Zone ([storing data in a single availability zone](https://oreil.ly/suFRM)) option is helpful for scenarios where the lowest cost possible is the goal. An example use case is a secondary backup because it is cost-effective as a secondary copy of data to protect against permanent loss due to the malfunction of the primary backup system.

Amazon S3 Glacier Deep Archive

Glacier is a secure, durable, and low-cost option ideal for data archival. It works well with S3 Lifecycles as the end delivery point.

In [Figure 2-1](#Figure-2-0-1-s3-central), you can see how Amazon S3 plays a unique role as the hub of data activity beyond just storing media assets or HTML files. A challenging constraint in building global scale platforms or machine learning systems is storage capacity and disk I/O. AWS provides a core service, S3, that eliminates those constraints. As a result, the elastic nature of S3 with the near-infinite storage and disk I/O creates a new type of workflow where AWS-managed services build on top of this core service, for example, [AWS SageMaker](https://oreil.ly/x5n5M) or [AWS Glue](https://oreil.ly/XYwNg).

![doac 0201](assets/doac_0201.png)

###### Figure 2-1\. S3 object storage as a central hub

###### Note

One of Noah’s favorite use cases for Amazon S3 is to host static websites. In the O’Reilly book *Python for DevOps*, he covers a [complete walk-through](https://learning.oreilly.com/library/view/python-for-devops/9781492057680/ch06.xhtml#idm46114712215640) of how to build a [Hugo](https://gohugo.io) website that uses [AWS CodePipeline](https://aws.amazon.com/codepipeline), Amazon S3, and [Amazon CloudFront CDN](https://aws.amazon.com/cloudfront). You can also view a walk-through of the steps necessary to deploy a Hugo S3 site on AWS on [YouTube](https://oreil.ly/3pLIJ) and [O’Reilly](https://oreil.ly/qRzVl).

Another standout example of AWS building on top of S3 is the use of S3 for log files queried by [Athena](https://oreil.ly/3R0zl).

Now that you understand the different storage classes for S3 object storage, let’s discuss Elastic Block Store (EBS) storage.

## Developing with EBS Storage

EBS is high-performance, network-attached storage. [EBS storage](https://aws.amazon.com/ebs) works by mounting block storage to an individual EC2 instance. EBS volumes act like raw, [unformatted block devices](https://oreil.ly/DiuJq) (physical hard drives) but are virtualized and available as a service. Finally, EBS is block-level storage attached to an EC2 instance as opposed to S3, which stands on its own, holds data as objects, and is accessible from many instances.

Let’s discuss the key benefits and use cases of EBS.

Benefits

EBS storage for data needs to be quickly accessed yet has long-term persistence. This storage type provides a dedicated high-performance network connection and the ability to provision dedicated disk I/O. Other key features include creating a snapshot for both backups to S3 or creating custom Amazon Machine Images (AMIs).

Use cases

EBS storage is ideal for web applications and database storage. High-performance dedicated disk I/O also comes in handy for building high-performance file servers that share data via Server Message Block (SMB) or Network File System (NFS) protocol.

One notable use of EBS is that through [io2 Block Express](https://oreil.ly/K7hok) high-performance provisioned input/output operations per second (IOPs) allows for the creation of a “SAN in the Cloud” as shown in [Figure 2-2](#Figure-2-3-1-ebs-iops). Notice how EBS storage provisioned in this manner creates a high-performance Microsoft SQL server instance that serves as the focal point of a business intelligence (BI) system.

This BI web service could feature an ASP .NET web service provisioned through Elastic Beanstalk, thus allowing for rapid autoscaling as data-intensive queries peak during heavy usage. Other use cases of the “SAN in the cloud” concept include deploying high-performance NoSQL servers or specialized analytics platforms like SAP HANA or SAS Analytics.

![doac 0202](assets/doac_0202.png)

###### Figure 2-2\. EBS SAN in the cloud with SQL server

###### Note

Another type of storage is [instance storage](https://oreil.ly/tHqaY). This storage type is temporary block storage for instances and is ideal for buffers, caches, or scratch partitions. A key difference with instance storage is that you cannot detach an instance store volume from one instance and attach it to a different instance. Unlike EBS storage, instance storage terminates when the instance stops, hibernates, terminates, or fails the underlying disk drive.

There’s one use case that EBS does not address: if you have multiple instances that need to use the same storage, you need a new solution. In this case, you could use both Amazon EFS and Amazon FSx for Windows File Server. Let’s discuss this topic next.

## Using Network Storage: EFS and FSx

In the early internet era of the late 1990s and early 2000s, network storage was a large part of how large-scale Unix computer networks worked at universities and commercial organizations. NFS, or the Network File System, is a protocol developed by Sun Microsystems in 1984 and became ubiquitous in that early internet era.

One key issue NFS storage solved early on is the ability to create a portable home directory. This capability meant that users could create a terminal-based connection from any workstation and their shell configuration files and data were available. The downside of NFS storage-based systems is that the NFS file server is the central hub; as a result, it often causes a bottleneck in workflows as the system is overwhelmed with requests.

When the cloud era arrived in the early 2000s, many organizations moved to the cloud and built systems that didn’t utilize centralized network storage anymore. Instead, they moved to block storage mounted on one machine or object stored mounted via a distributed filesystem like Hadoop. With the availability of [EFS](https://aws.amazon.com/efs) (managed NFS) and [FSx](https://aws.amazon.com/fsx/windows) (managed Windows network storage), the advantages of a centralized network mount point are back without the drawback of bottlenecks from the poor performance of centralized file servers.

A great example of NFSOps is shown in [Figure 2-3](#Figure-2-0-2-nfsops). NFSOps describes using a network file system as a method of deploying software and configuration. Changes to the source code in GitHub trigger a build process through a [Jenkins](https://www.jenkins.io) deployment server. This deployment server has an EFS mount point associated with it. As a result, the build server can use Rsync to deploy the script changes to the network mount point in a few milliseconds.

With the deployment process solved, this diagram shows a real-world solution used in a computer vision system. Because this particular workload uses access to centralized storage, this workflow mounts EFS as the “source of truth,” allowing thousands of spot instances to use, read, and write data simultaneously. Further, the source code that performs the read and writes operations is stored on the EFS volume, dramatically simplifying configuration and deployment.

![doac 0203](assets/doac_0203.png)

###### Figure 2-3\. NFSOPs: using NFS storage to enhance operations

For .NET developers, there are also many exciting new workflows opened up by FSx for Windows. An excellent example in [Figure 2-4](#Figure-2-3-2-media-content) shows an FSx mount point that communicates with multiple instances of an Elastic Beanstalk–hosted .NET web service.

Another standard workflow for FSx is to develop feature films by mounting the storage endpoint for each editing workstation. In this [AWS blog post](https://oreil.ly/9pwQk), you can read about how a film company used the editing software package DaVinci Resolve on AWS to create a feature film from home editing suites. Using FSx as a central file system in an animated film pipeline for major motion pictures is becoming a popular option for media companies. Because the web service can mount the file system, it can track the location of the assets as part of an asset management workflow. When an animator needs to render the entire file sequence, they send the work to ECS for batch processing. Since ECS also has access to the [same file system](https://oreil.ly/Kf1q4), it allows for fast and seamless integration to the workflow for animators.

![doac 0204](assets/doac_0204.png)

###### Figure 2-4\. FSx for Windows in media workflow

###### Note

The media industry has a long history of high-performance file servers coupled with high-performance computing (HPC) clusters. An excellent example of a modern cloud-based version is [this blog post on virtual production](https://oreil.ly/si1VM) with Epic Games using FSx.

Using either EFS or FSx for Windows opens up new cloud-native architectures and is worth investigating further using the example tutorial on the [AWS documentation website](https://oreil.ly/NZMNd). Next, let’s discuss compute options available on AWS.

# Using AWS Compute Core Services

Amazon EC2 provides resizable compute capacity with several essential details, including the ability to provision servers in minutes, automatically scale instances up or down, and only pay for the capacity you need. A necessary aspect of this service is that elasticity is built into the foundation of both the service and the integration with other AWS services.

Use cases for EC2 instances include servers ranging from web to database, where a developer wants more fine-grained control of the deployment process. While managed services like AWS App Runner offer a huge convenience over managing servers yourself, there are scenarios where a developer needs lower-level access to the EC2 instances. This service also allows complete control of the computing resources, meaning you can spin up a Linux and a Microsoft Windows instance. Further, you can optimize costs by using different pricing plans.

Next, let’s explore these different compute options in finer-grained detail.

## Comparison of AWS Compute Core Services

Another way to think about AWS EC2 is to look at how the official documentation [compares AWS compute resources](https://aws.amazon.com/products/compute) in a more granular manner than discussed earlier in the chapter. In their official [“Overview of Amazon Web Services” whitepaper](https://oreil.ly/3HihF), AWS breaks down computing into several categories:

Instances (virtual machines)

Secure, resizable compute capacity in the cloud. EC2 instances and AWS Batch (fully managed batch processing at any scale) are two examples.

Containers

Containers provide a standard way to package your application into a single image. According to the AWS documentation, running containers on AWS “provides developers and admins a highly reliable, low-cost way to build, ship, and run distributed applications at any scale.” Examples include AWS App Runner^([4](ch02.xhtml#idm45599657961904)) and Amazon Elastic Container Service (ECS).

Serverless

This computing service allows running code without provisioning or managing infrastructure and responding to events at any scale. AWS Lambda is an example of serverless technology.

Edge and hybrid

Edge services process data close to where the data resides versus processing in a data center. This service delivers a consistent AWS experience wherever you need it by providing the ability to use the cloud, on-premise, or at the edge. AWS Snow Family is an example.^([5](ch02.xhtml#idm45599657958272))

Cost and capacity management

AWS helps you determine cost and capacity management by providing services and tools. It is up to the customer to test workloads against the recommended instance types to fine-tune price performance. Elastic Beanstalk is an example of one of the services in this category.^([6](ch02.xhtml#idm45599657955440))

Another way to reason about these choices is represented in [Figure 2-5](#Figure-2-0-compute-choices). Note that there is an inverse relationship between higher infrastructure control and faster application deployment. Fully managed services that also include additional production features like AWS Lambda and Fargate are at the far extreme, allowing for the quickest application development and deployment. This relationship is not a perfect rule in all situations, and exceptions exist, perhaps in a condition where developing an AWS Lambda microservice is harder to build and deploy than an AWS App Runner microservice for a particular domain.

![doac 0205](assets/doac_0205.png)

###### Figure 2-5\. AWS compute choices

With our knowledge of EC2 core services, let’s talk about getting started with EC2.

## Using EC2

At the surface level, EC2 “just works” by allowing you to launch a compute instance and use it for tasks. At a deeper level, EC2 has a [vast collection of features](https://oreil.ly/AiRbJ). From a developer’s perspective, let’s dive into key parts of EC2 worth noting:

Amazon Machine Images (AMIs)

AMIs are preconfigured templates that conveniently install the software necessary for a server instance. There are AWS-recommended AMIs for Linux, [Windows](https://oreil.ly/7ymoo), and even macOS. Notice in [Figure 2-6](#Figure-2-1-ami) that the AMI Catalog is accessible by navigating inside the AWS console to the EC2 dashboard and selecting images. The quickstart AMIs are the commonly used AMIs, including Amazon Linux 2 and Windows. The “My AMIs” section is where custom AMIs appear. Finally, there is both a marketplace and a community of AMIs available.

![doac 0206](assets/doac_0206.png)

###### Figure 2-6\. AMI Catalog

Instance types

In total, hundreds of [instance types](https://aws.amazon.com/ec2/instance-types), arranged by category, including Compute Optimized, Memory Optimized, Accelerated Computing (hardware accelerators), and Storage Optimized. A best practice is to assess the needs of your application and perform load testing to ensure it meets the cost and performance requirements you expect.

Firewall

EC2 has a firewall called [security groups](https://oreil.ly/05rcL) that enables granular configuration of protocols, ports, and IP ranges for inbound and outbound access to instances.

Metadata

You can create and use tags to enable resource tracking and automation of EC2 instances.

Elastic IP addresses

Static IPv4 addresses can be dynamically assigned to EC2 instances, thus enhancing elasticity and automation.

Virtual Private Clouds (VPC)

While VPC is a separate service, it has [deep integration](https://oreil.ly/7rcng) with EC2\. Virtual networks create an isolated environment with precise controls for connecting to other resources in AWS.

With the basics out of the way, let’s look at what is involved in provisioning an EC2 instance in [Figure 2-7](#Figure-2-1-2-provisioning-ec2). Notice how the instance launch can also use “User data” to assign custom commands at launch or select between EBS or instance storage. Other critical decisions include setting up a security group that opens up ports necessary for networking between services. Other configurable options include selecting the appropriate IAM role and giving the EC2 instance the ability to communicate with Amazon S3.

![doac 0207](assets/doac_0207.png)

###### Figure 2-7\. Provisioning an EC2 instance

###### Note

You can view an entire walk-through of how to provision EC2 instances from AWS CloudShell or the AWS Console in the following [YouTube Video](https://oreil.ly/4FvEP) or on [O’Reilly](https://oreil.ly/rltV3).

It is essential to mention that a regular Bash terminal works well for automating EC2\. Notice the following commands that launch an instance, describe it, then terminate it:

[PRE0]

###### Note

A recommended resource for doing a deep dive on EC2 instance types is the AWS Knowledge Center document [“How do I choose the appropriate EC2 instance type for my workload?”](https://oreil.ly/Lop74)

Now that you know how to use EC2 in more detail, let’s discuss networking with EC2.

## Networking

AWS networking consists of a global infrastructure that enables EC2 instances to work globally and is reliable and distributed. Notice in [Figure 2-8](#Figure-2-2-regions) that each region is distinct geographically and has multiple availability zones.

![doac 0208](assets/doac_0208.png)

###### Figure 2-8\. AWS regions and zones

There are more than 25 regions globally and more than 85 availability zones. This data means each region has at least 3 availability zones on average. Fortunately, it is easy to use AWS CloudShell and PowerShell to get your region’s exact number of availability zones. Launch an instance of AWS CloudShell as a user with privileges to query EC2 via an IAM role. Next, import `AWSPowerShell.NetCore` and then query the available zones available in your AWS CloudShell’s region:

[PRE1]

###### Note

Under the region level, the AZs interconnect with high-bandwidth, low-latency networking, which is also encrypted. The network performance is sufficient to accomplish synchronous replication between AZs. The AZs are physically separated to achieve a real disaster recovery solution.

You can get more fine-grained control by assigning the output to a PowerShell variable `$zones` and then using `$zones.count` to count the exact number of availability zones in your region in the status “available”:

[PRE2]

The output shows that with only a couple of lines of code, it is straightforward to script EC2 from PowerShell using the convenient AWS CloudShell as the development environment:

[PRE3]

###### Note

You can check out the [PowerShell documentation](https://oreil.ly/LNsQJ) to see all of the flags available. There is also a demo walk-through of this process on [YouTube](https://youtu.be/MeeZ3fnJsqM) and [O’Reilly](https://oreil.ly/e75S2).

Now that you understand EC2 networking in more detail, let’s talk about EC2 pricing options.

## Using EC2 Pricing Options

While purchasing options may not be top of mind to many developers, it is an important consideration. One of the five pillars of the AWS Well-Architected Framework is [cost optimization](https://oreil.ly/C170R). In particular, this framework recommends that you “adopt a consumption model,” i.e., pay only for the computing resources you require. Let’s break down the different pricing options available for EC2:

On-demand instances

These instances are ideal for spiky workloads or prototyping. You pay for computing in increments as granular as a second, and there is no long-term commitment.

Reserved instances

These instances are noteworthy for committed, steady-state workloads. For example, once a company has a good idea of the configuration and load of its architecture in the cloud, it will know how many instances to reserve. There are options for both one-year or three-year commitments, leading to a significant discount from on-demand instances.

Savings plans

This plan is suited to Amazon EC2 and AWS Fargate, as well as AWS Lambda workloads because it lets you reserve an hourly spend commitment. The discounts are the same as reserved instances with the added flexibility in exchange for a dedicated hourly spend commitment.

Spot instances

These are ideal for fault-tolerant workloads that can be both flexible and stateless. A good example would be doing a batch job that can run during the day, say, transcoding video. These instances are available at up to 90% off on-demand pricing.

Dedicated hosts

Another option for workloads requiring single tenancy or software licenses is to use a dedicated physical server.

It is helpful to understand the options for pricing on AWS because they can make or break the effectiveness of moving to the cloud. With that information covered, let’s move on to security best practices for AWS.

# Security Best Practices for AWS

Security isn’t optional for software engineering; it is a core requirement for any project. What may be surprising to newcomers to cloud computing is that moving to the cloud increases the security profile of projects. One way to explain this concept is through the unique AWS concept called the [“shared responsibility model”](https://oreil.ly/6q23h) shown in [Figure 2-9](#Figure-2-4-shared-model).

![doac 0209](assets/doac_0209.png)

###### Figure 2-9\. AWS shared responsibility model

Notice that AWS is responsible for the entire global infrastructure, including the physical security requirements. The customer then builds on top of that secure foundation and is responsible for items like customer data or firewall configuration.^([7](ch02.xhtml#idm45599657709504))

A great place to start designing a secure system on AWS is by following the guidelines available from the [AWS Well-Architected Framework website](https://oreil.ly/uaGGU). There are seven design principles. Let’s take a look:

Implement a strong identity foundation

Use the principle of least privilege (PLP) to assign just enough access to resources for a user to accomplish their assigned tasks. Use the identity and access management (IAM) system to control users, groups, and access policies while eliminating static credentials.

Enable traceability

It is critical to enable real-time monitoring, alerting, and auditing of changes to a production environment. This capability works through a comprehensive logging and metric collection system that allows actionability.

Apply security at all layers

The best defense is a layered approach where every component of cloud computing, from computing to storage to networking, adds security hardening.

Automate security best practices

Automating security best practices as code allows for an efficient and idempotent way to minimize organizational risk.

Protect data in transit and at rest

Data needs protecting both in its location and when it moves. This best practice happens through combined encryption, tokenization, and access controls.

Keep people away from data

Manual data processing is an antipattern to avoid as it introduces humans into the loop, which can inflict intentional and unintentional security holes.

Prepare for security events

A consistent theme at Amazon is “design for failure.” Similarly, it is essential to have a plan for dealing with security incidents and processes that aligns with organizational needs with security.

One of the big takeaways for AWS security is how critical automation is to implementing security. DevOps is at the heart of cloud-native automation, which is the focus of [Chapter 6](ch06.xhtml#Chapter6). With the foundational best practices of security out of the way, let’s discuss the AWS best-practice encryption.

## Encryption at Rest and Transit

Encryption is a method of transforming data that makes it unreadable without access to a secret encryption key.^([8](ch02.xhtml#idm45599657691056)) At the center of the encryption strategy in transit and rest is that data becomes exposed in an unencrypted form at no point.

###### Note

“Transit” refers to sending data from one location to another, such as from a user’s mobile phone to a banking website. *Rest* refers to the storage of the data; for example, data that resides in a database is at rest.

A good analogy would be to consider the concept of sealing perishable foods by vacuum packing the product. Vacuum packing removes oxygen, thus extending the product’s shelf life and preserving it. As soon as the seal breaks, the food item instantly degrades. Similarly, data exposed in an unencrypted form exposes itself to instant risk and breaks down best practices in storing and managing data. AWS solves this problem by providing services that allow you to encrypt your data comprehensively throughout the lifecycle of data operations on the platform.

###### Note

One essential item to consider is that for encryption to work, there is an [encryption key](https://oreil.ly/QJzoP). Privileged users can access data only if they have that key.

Now that we have covered encryption, let’s continue to drill down into security concepts on AWS by next covering the principle of least privilege.

## PLP (Principle of Least Privilege)

The mail delivery person does not have a key to your house because of the principle of least privilege (PLP). It is a security best practice for non-IT people and IT people alike that says to never give more access to resources than necessary. You can see this principle in action in [Figure 2-10](#Figure-2-5-plp). The mail delivery person has access to the mailbox but not the house. Similarly, the family has access to the house, but only the parents can access the safe. This concept means with AWS, you give users the least amount of access and responsibility necessary to complete their tasks.

![doac 0210](assets/doac_0210.png)

###### Figure 2-10\. Principle of least privilege example

PLP protects both the resource as well as the recipient of the privilege. Consider the case of the safe-in-the-house scenario. There may be something in the safe that is dangerous to the children, and it protects both the safe and the child from not having access. With AWS, this design principle is in effect: only assign IAM policies with the least privileges to get the task completed.

This approach works in a real-world microservice architecture like the one shown in [Figure 2-11](#Figure-2-0-4-lambda-plp). Notice how an AWS Lambda microservice is listening to AWS S3\. When a user uploads a new profile picture, the AWS Lambda function uses an “S3 Read Only” policy since this service only needs to accept the event payload from S3, which includes the name of the image uploaded and the S3 URI, which contains the full path to the image. The AWS Lambda microservice then writes that metadata to the DynamoDB table using a role that includes the ability to access that particular table and update it.

![doac 0211](assets/doac_0211.png)

###### Figure 2-11\. Serverless profile picture profile updater microservice

This PLP workflow prevents both security holes as well as developer errors. Limiting the scope of what a microservice does strictly by what it needs to perform its task diminishes organizational risk. This risk may not be apparent to systems developed without PLP, but it doesn’t mean the risk doesn’t exist.

With a deeper understanding of implementing security, let’s discuss a central component in any securely designed system on AWS, AWS Identity and Access Management.

## Using AWS Identity and Access Management (IAM)

At the center of securely controlling access to AWS services is [IAM](https://oreil.ly/xaPQq). With AWS IAM, you can specify who or what can access services and resources in AWS, centrally manage fine-grained permissions, and analyze access to refine permissions across AWS, as you can see in [Figure 2-12](#Figure-2-0-15-iam-how-it-works-diagram).

![doac 0212](assets/doac_0212.png)

###### Figure 2-12\. IAM high-level breakdown

To understand how IAM works in more detail, [Figure 2-13](#Figure-2-0-16-IAM-works) shows how a principal is a person or application that can request an AWS resource by authenticating. Next, a user (Account ID 0123*) must be authorized to create a request, where policies determine whether to allow or deny the request. These rules are identity-based policies, other policies, and resource-based policies that control authorization.

After the request approval, the actions available come from the service itself, i.e., `CreateDatabase`, `CreateUser`, or whatever the service supports. Finally, after the approval of the operations in the request for the service, they are performed on the resource. An example of a resource is an EC2 instance or an Amazon S3 bucket.

You can read a very detailed overview of this process by reading the [Understanding How IAM Works User Guide](https://oreil.ly/GkiUm). The IAM service has five critical components: IAM users, IAM groups, IAM roles, and IAM permissions and policies (bundled together for brevity). Let’s discuss each one:

IAM users

A user is an entity you create in AWS to represent either a person or an application. [A user](https://oreil.ly/etisJ) in AWS can have programmatic access via an access key, AWS Management Console access via a password, or both. An access key enables access to the SDK, CLI commands, and API.

IAM groups

An [IAM user group](https://oreil.ly/hawd3) is a collection of IAM users. Using groups allows you to specify permissions for multiple users.

IAM roles

An [IAM role](https://oreil.ly/hQyBG) is an IAM identity with specific permissions. A typical scenario is for a service like EC2 to have a role assigned with special permissions to call the AWS API, say, download data from S3 without needing to keep API keys on the EC2 instance.

IAM permissions and policies

You [manage access](https://oreil.ly/oi6u2) to AWS by creating or using default policies and attaching them to IAM identities like users, groups, and roles. The permissions in these policies then set what is allowed or denied.

![doac 0213](assets/doac_0213.png)

###### Figure 2-13\. How IAM works

With a deeper understanding of identity and access management on AWS out of the way, let’s switch topics to building NoSQL solutions on AWS with DynamoDB.

# Developing NoSQL Solutions with DynamoDB

The CTO of Amazon, Werner Vogel, [points out](https://oreil.ly/qSNzq) that a “one size database doesn’t fit anyone.” This problem is illustrated in [Figure 2-14](#Figure-2-6-pydo_1504), which shows that each type of database has a specific purpose. Picking the correct storage solutions, including which databases to use, is crucial for a .NET architect to ensure that a system works optimally. An excellent resource for comparing AWS Cloud databases is found on the [AWS products page](https://oreil.ly/qK0no).

![doac 0214](assets/doac_0214.png)

###### Figure 2-14\. Each type of database has a purpose

Maintenance is a consideration in designing a fully automated and efficient system. Suppose a particular technology choice is being abused, such as using a relational database for a highly available messaging queue. In that case, maintenance costs could explode, creating more automation work. So another component to consider is how much automation work it takes to maintain a solution.

At the core of [DynamoDB](https://aws.amazon.com/dynamodb) is the concept of a distributed database that is eventually consistent.^([9](ch02.xhtml#idm45599657635728)) In practice, the database can automatically scale up from [zero to millions of requests per second](https://oreil.ly/ZRZvt) while maintaining low latency.

Let’s break down the critical DynamoDB concepts via characteristics, use cases, and key features:

Characteristics

DynamoDB is a fully managed nonrelational key-value and document database that performs at any scale. It is also serverless and ideal for event-driven programming. Enterprise features of DynamoDB include encryption and backups.

Use cases

This service works well for simple high-volume data that must scale quickly and doesn’t require complex joins. It also is ideal for solutions that require high throughput and low latency.

Key features

Some key features include NoSQL tables and the ability to have items with different attributes. DynamoDB also supports caching and peaks of 20M+ requests per second.

We have covered the basics of DynamoDB; next, let’s build a simple application using DynamoDB.

# Build a Sample C# DynamoDB Console App

Let’s build a simple DynamoDB application in C# that reads data from a table. There are many ways to create a table manually, including [Figure 2-15](#Figure-2-6-1-create-table), in which you first navigate to the DynamoDB interface and then select “Create table.”

![doac 0215](assets/doac_0215.png)

###### Figure 2-15\. Create a DynamoDB table in Amazon Console

Another way to create a table and populate it with values is by using Visual Studio AWS Explorer, as shown in [Figure 2-16](#Figure-2-6-2-create-table-aws-explorer).

![doac 0216](assets/doac_0216.png)

###### Figure 2-16\. Create a DynamoDB table in Visual Studio Explorer

Yet again, we populate the table with fruit. We can do it via the .NET SDK or the console if we want to query it, as shown in [Figure 2-17](#Figure-2-6-3-query-dynamo).

![doac 0217](assets/doac_0217.png)

###### Figure 2-17\. Query table

To create an application to query DynamoDB, open up Visual Studio and create a new Console Application, then install the [DynamoDB NuGet package](https://oreil.ly/BChf7) as shown in [Figure 2-18](#Figure-2-6-4-install-dynamodb).

![doac 0218](assets/doac_0218.png)

###### Figure 2-18\. Install DynamoDB

With the installation out of the way, the process to query the table, print out the entire table, and pick a random fruit is addressed in the following C# code example:

[PRE4]

[![1](assets/1.png)](#co_aws_core_services_1)

First, create a client.

[![2](assets/2.png)](#co_aws_core_services_2)

Next, scan the table.

[![3](assets/3.png)](#co_aws_core_services_3)

Then, make a list to hold the results.

[![4](assets/4.png)](#co_aws_core_services_4)

Finally, loop through the table results, put them in the list, and randomly pick a fruit to print.

The results of the Console Application in [Figure 2-19](#Figure-6-5-table-scan) show a successful table scan and selection of random fruit.

![doac 0219](assets/doac_0219.png)

###### Figure 2-19\. Table scan Console App

Check out the [AWS docs GitHub repo](https://oreil.ly/gol15) for more ideas on building applications. It is an excellent resource for ideas on building solutions with DynamoDB.

###### Note

A CRUD application creates, reads, updates, and deletes items from a database. This [CRUD example](https://oreil.ly/1SdtM) for DynamoDB is a great resource to refer to when building this style of application on DynamoDB.

Now that you know how to build solutions with DynamoDB let’s discuss a complementary service: the Amazon Relational Database Service (RDS).

# Amazon Relational Database Service

RDS is a service that lets you point and click to set up an enterprise database in the cloud. The times before RDS were dark days for developers who have experienced running their own SQL database. The list of tasks necessary to properly administer a database is quite staggering. As a result, many positions called DBAs or database administrators filled the role of helping keep SQL systems alive like Microsoft SQL Server, MySQL, and Postgres. Now many of those tasks are features in RDS.

The features synthesize into a uniform solution that creates agility for .NET projects. The critical win of RDS is that it allows a developer to focus on building business logic, not fighting the database. This win occurs because RDS alleviates the pain of managing the database and will enable developers to focus on building the application. Let’s look at a select list of the core features that highlight the power of RDS:

Core features

RDS is easy to use. It also has automatic software patching, which reduces security risks. Best practice recommendations are baked into the product and include access to SSD storage, dramatically increasing the service’s scalability.

Reliability

The service includes the ability to create automated backups and build database snapshots. Finally, a best practice of multi-AZ deployments allows for a robust recovery option in an availability zone outage.

Security and operations

RDS includes significant encryption capabilities, including using this in both rest and transit. Further network isolation allows for increased operational security and fine-grained resource-level permissions. Also, there is extensive monitoring via CloudWatch, which increases the cost-effectiveness of the service.

With the core features of RDS out of the way, let’s discuss a fully managed serverless solution next.

# Fully Managed Databases with Amazon Aurora Serverless v2

The cloud native world of serverless is addictive to develop solutions with because of the straight line between your thoughts and their implementation as business logic. With the addition of [Amazon Aurora Serverless v2](https://oreil.ly/E5QJZ), the ability to execute quickly on business logic as code is enhanced further.

Here is a subset of the benefits of the Amazon Aurora Serverless v2:

Highly scalable

Instances can scale to hundreds of thousands of transactions in a fraction of a second.

Simple

Aurora removes the complexity of provisioning and managing database capacity.

Durable

Aurora storage is self-healing via six-way replication.

The critical use cases include the following:

Variable workloads

Running an infrequently used application that peaks 30 minutes a few times a day is a sweet spot for this service.

Unpredictable workloads

A content-heavy site that experiences heavy traffic can count on the database automatically scaling capacity and then scaling back down.

Enterprise database fleet management

Enterprises with thousands of databases can automatically scale database capacity by each application demand without managing the fleet individually.

Software as a service (SaaS) applications

SaaS vendors that operate thousands of Aurora databases can provision Aurora database clusters for each customer without needing to provision capacity. It automatically shuts down the database when not in use to reduce costs.

Scaled-out databases split across multiple servers

It is common to break high write or read requirements to numerous databases. Aurora Serverless v2’s capacity is met instantly and managed automatically, simplifying the deployment.

Putting all this together, a SaaS company could build an architecture as shown in [Figure 2-20](#Figure-2-0-4-aurora-serverless) where each client has one dedicated serverless pipeline of AWS Step Functions that orchestrate via AWS Lamba payload that proceed to Aurora Serverless v2.

![doac 0220](assets/doac_0220.png)

###### Figure 2-20\. SaaS company architecture with Aurora Serverless

The benefit of this architecture is that it is easy to debug the pipeline for each paying customer. Additionally, the complexity of managing server load for each client is alleviated since the entire pipeline uses autoscale serverless components.

###### Note

One notable example of developing a .NET Web API using Aurora Serverless is at this [blog post by AWS](https://oreil.ly/WUmb9).

This chapter covered a wide variety of essential topics for mastering AWS. Let’s wrap up the highlights.

# Conclusion

AWS Core Services are the foundation that allows a developer to build sophisticated solutions. In this chapter, we covered the core services of computing and storage. This coverage included recommendations on using service options for services like EC2 and S3 and managed services.

It is important to note that for both EC2 and S3, there is an extensive selection of pricing options. The appropriate pricing option for a given architecture is critical in building a well-architected AWS system. Fortunately, AWS provides detailed monitoring and instrumentation on pricing and valuable tools in the [AWS Cost Management Console](https://oreil.ly/8EZtn).

The chapter covered both traditional and NoSQL databases, including an example of using DynamoDB to build a simple Console Application. Take a look at the critical thinking questions and exercises to challenge yourself to apply this material to your scenarios. The next chapter covers migrating a legacy .NET application to AWS.

# Critical Thinking Discussion Questions

*   What are new workflows for architecting software made available by network filesystems like Amazon EFS and Amazon FSx?

*   What are the advantages of prototyping solutions with AWS EC2 spot instances?

*   What compute and database services should a small startup of one to three developers gravitate toward for building SaaS APIs?

*   What compute services should a company with a large data center moving to the cloud consider?

*   How could your organization translate the spirit of the blog post [“A One Size Fits All Database Doesn’t Fit Anyone”](https://oreil.ly/W4H4a) into a plan for how to use the correct types of databases for the problems it faces?

# Exercises

*   Build a CRUD (create, read, update, delete) C# Console App for AWS S3.

*   Build a CRUD (create, read, update, delete) C# Console App for AWS DynamoDB.

*   Build a CRUD (create, read, update, delete) C# Console App for AWS RDS for Aurora Serverless v2.

*   Change the DynamoDB example to select a fruit without needing to scan the table randomly.

*   Launch a Windows EC2 instance, install a custom tool or library on it, [convert it to an AMI](https://oreil.ly/fgPD9), and launch an EC2 instance with your [custom AMI](https://oreil.ly/zkG9o).

^([1](ch02.xhtml#idm45599658074352-marker)) AWS S3 also regularly gets [“peaks of tens of millions of requests per second”](https://oreil.ly/x1fhi).

^([2](ch02.xhtml#idm45599658047392-marker)) AWS Athena is an “interactive query service” that allows for easy analysis of data in S3 via SQL. You can read more about it in the [getting started guide](https://oreil.ly/N5lV8).

^([3](ch02.xhtml#idm45599658045184-marker)) You can read more about the managed service SageMaker in the [official docs](https://oreil.ly/FwqXr).

^([4](ch02.xhtml#idm45599657961904-marker)) AWS App Runner lets you package a container into a microservice that continuously deploys. You will see an example of this service in the container chapter.

^([5](ch02.xhtml#idm45599657958272-marker)) The [Snow Family](https://aws.amazon.com/snow) is a set of “Highly-secure, portable devices to collect and process data at the edge, and migrate data into and out of AWS.”

^([6](ch02.xhtml#idm45599657955440-marker)) According to AWS, [AWS Elastic Beanstalk](https://aws.amazon.com/elasticbeanstalk) “is an easy-to-use service for deploying and scaling web applications and services.”

^([7](ch02.xhtml#idm45599657709504-marker)) Further, you can find detailed information about security by visiting the [Security Pillar](https://oreil.ly/BJtas) section of the AWS Well-Architected Framework site.

^([8](ch02.xhtml#idm45599657691056-marker)) You can read more about this topic via the [AWS whitepaper “Logical Separation on AWS”](https://oreil.ly/9wDGf).

^([9](ch02.xhtml#idm45599657635728-marker)) A foundational theorem in theoretical computer science is the [CAP theorem](https://oreil.ly/aM06S), which states that there is a trade-off between consistency, availability, and partition tolerance. In practice, this means that many NoSQL databases, including DynamoDB, are “eventually consistent,” meaning they eventually converge the new writes to different nodes to create a consistent view of the data.