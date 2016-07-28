---
layout: page
title: Frequently Asked Questions
---
{% include JB/setup %}


## Microsoft支持Orleans吗？
<!--## Does Microsoft support Orleans?-->

<!--Source code of Orleans has been released under an MIT license on [GitHub](https://github.com/dotnet/orleans). Microsoft continues to invest in Orleans and accepts community contributions to the codebase.-->
Orleans的源码已经在MIT协议下在[GitHub](https://github.com/dotnet/orleans)上开源。Microsoft支持资助Orleans和接受社区的贡献。

## 我能获取一个“可上线”授权码？
<!--## Can I get a "Go Live" License?-->

<!--The source code has been releases under the [MIT license](https://github.com/dotnet/orleans/blob/master/LICENSE).-->
源码已经在[MIT license](https://github.com/dotnet/orleans/blob/master/LICENSE)下开源。

## Orleans是否已经可以用于生产环境？
<!--## When will Orleans be production ready?-->

<!--Orleans has been production ready and used in production for several years. -->
Orlean已经可以用于生产环境，并且已经用于生产环境多年。
 
## 什么时候我应该使用grain什么时候我应该使用以前普通的对象？
<!--## When should I use a grain and when should I use a plain old object?-->

<!--There are two ways to answer this, from a runtime behavior perspective, and from a modeling perspective.-->
有两个角度来回答这个问题，从运行时行为的角度和从建模的角度。

<!--The **runtime behavior perspective** is that an object can only be created within a grain and is not remotely accessible. Grains are accessible from anywhere in the system and are location-transparent, so that they can be automatically placed on any server in the deployment, and they survive failures or reboots of servers.-->
**运行时行为的角度**:一个对象只有在grain中创建并且不会被远程访问。Grain在系统的任何地方都可以访问并且是透明寻址的，所以开发的时候他们可以自动分配在任何服务器上，并且不受服务器出错或者重启的影响。

<!--The **modeling perspective**: there are four kinds of "things" in your Orleans-based application: communication interfaces, message payloads, grains, and data held privately by grains. Objects should be used for payloads and data held by grains; communication interfaces are regular interfaces with some minor restrictions. The question that remains, then, is what entities in a given system should be modeled as grains?-->
**建模的角度**：在你的和基于Orleans的应用中有四个“东西”：通信接口、消息、grain和grain自己持有的数据。对象可以被用作消息负载和grain自己持有的数据；通信接口是有一些小约束的正常接口。那么什么样的实体应该被建模成grain呢？

<!--Generally, you should use a grain to model an independent entity which has a publicly exposed communication interface with other components of the system and that has a life of its own – that is, it can exist independently from other components. For example, a user in social network is a grain, while it's name is not. A user’s news wall may be a grain, while the list of the messages it received is not (since the wall is accessible by other users, while the list of messages is a private data to that user only). Hopefully, the samples on this site will help you identify some of the patterns and see parallels to your own scenarios.-->
一般，你应该使用grain来建模一个有公开暴露的与其他系统组建通信的接口的独立实体并且他有自己的生命——它可以自己独立于其他组建存在。例如，一个社交网络中的用户是一个grain，然而他的名字不是。一个用户新创建一个主页（原文the wall是twitter上类似微博分组微博的东西）可以是一个grain，然后收到的信息列表不是（因为分类主页是其他用户可以访问，但是消息列表是用户自己私有的数据）。希望这些例子可能帮助你分辨出一些模式并且找到与你的业务场景的相似之处。

## 我如何避免单点grain过热？
<!--## How should you avoid grain hot spots?-->

<!--The throughput of a grain is limited by a single thread that its activation can execute on. Therefore, it is advisable to avoid designs where a single grain receives a disproportionate share of requests. There are various patterns that help prevent overloading of a single grain even when logically it is a central point of communication.-->
一个grain的吞吐量受限于一个单线程的激活能执行多少。因此，建议不要升级成单个grain收到不成比例的请求。有许多模式可以帮助避免单个grain的过载，甚至逻辑上这个grain是通信的中心节点的时候。

<!--For example, if a grain is an aggregator of some counters or statistics that are reported by a large number of grains on a regular basis, one proven approach is to add a controlled number of intermediate aggregator grains and assign each of the reporting grains (using a modulo on a key or a hash) to an intermediate aggregator, so that the load is more or less evenly distributed across all intermediate aggregator grains that in their turn periodically report partial aggregates to the central aggregator grain.-->
例如，一个grain是大量周期性的计数器grain和统计grain的聚合器g，一个行之有效的方式的方法是添加一定数量的中间聚合器grain，并且让这上报数据的grain（使用对一个键取模胡总和哈希的方法）报告给中间聚合器，这样负载或多或少的分布到了所有的中间聚合器grain上，并且在中间聚合器的turn中定期地发送部分聚合过的数据给中央聚合grain。

## 我如何销毁一个grain？
<!--## How do I tear down a grain?-->

<!--In general there is no need for application logic to force deactivation of a grain, as the Orleans runtime automatically detects and deactivates idle activations of a grain to reclaim system resources. In rare cases when you think you do need to expedite deactivation of a grain, the grain can do that by calling the `base.DeactivateOnIdle()` method. -->
 一般没有必要在应用逻辑中强制注销一个grain，因为Orleans运行时自动检测和注销一个闲置的grain激活来回收系统资源。在极少的情况下你认为你需要加快一个grain的注销，grain可以通过调用`base.DeactivateOnIdle()`方法来实现。

## 我能否告诉Orleans在哪里激活一个grain吗？
<!--## Can I tell Orleans where to activate a grain?-->

<!--It is possible to do so using restrictive placement strategies, but we generally consider this an anti-pattern, so it is not recommended. If you find yourself needing to specify a specific silo for grain activation, you are likely not modeling your system to take full advantage of Orleans.-->
这可以通过使用限定的布局策略，但是我们一般认为这是一种反模式，所以不建议这么做。如果你发现你需要为grain激活指定一个特定的silo，你可能不是按照发挥Orleans的全部优势来设计你的系统的。

<!--By doing what the question suggests, the application would take on the burden of resource management without necessarily having enough information about the global state of the system to do so well. This is especially counter-productive in cases of silo restarts, which in cloud environments may happen on a regular basis for OS patching. Thus, specific placement may have a negative impact on your application's scalability as well as resilience to system failure.-->
按照上面的问题的建议，在没有足够的关于系统的全局状态的信息的情况应用也可以很好的完成系统资源管理的任务。这样在silo重启的情况下特别不利于生产，在云环境中这可能因为系统打补丁而定期发生。因此指定布局可能可能对你的应用的可扩展性和对系统错误的适应性有不利的影响。

## 一个部署可以在不同数据中心之间运行吗？
<!--## Can a single deployment run across multiple data centers?-->

<!--Orleans deployments are currently limited to a single data center per deployment.-->
一个Orleans部署现在只能在一个数据中心内。

## 我能热部署grain吗？添加和更新他们。
<!--## Can I hot deploy grains either adding or updating them?-->

<!--No, not currently.-->
不，现在不能。

## 怎么管理grain的版本？
<!--## How do you version grains?-->

<!--Orleans currently does not support updating grain code on the fly. To upgrade to a new version of grain code, you need to provision and start a new deployment, switch incoming traffic to it, and then shut down the old deployment.-->
Orleans现在不支持在线更新grain的代码。要升级新版的grain代码，你需要准备和启用一个新的部署，切换流量到新部署上，然后关掉旧的不熟。

## 我可以把一个grain的状态持久化到Azure cache service吗？
<!--## Can I persist a grain’s state to the Azure cache service?-->

<!--This can be done though a storage provider for Azure Cache. We don’t have one but you can easily build your own.-->
这可以通过一个storage provider for Azure Cache实现。我不提供，但是你可以很容易地自己写一个你自己的。

## 我能从公网连接到Orleans silo吗？
<!--## Can I Connect to Orleans silos from the public internet?-->

<!--Orleans is designed to be hosted as the back-end part of a service and you are suposed to create a front-end in your servers which externa clients connect to. It can be an http based Web API project, a socket server, a SignalR server or anything else which you require. You can actually connect to Orleans from the internet, but it is not a good practice from the security point of view.-->
Orleans被设计成服务的后端宿主部分并且假设你在你的服务器上创建一个外部客户端连接的前端部分。可以是一个基于http的Web API工程，一个socket放服务器，一个SignalR服务器或者其他你需要的。你的确可以通过互联网连接到Orleans，但是从安全的角度看这不是一个好的实践。

## 如果一个silo在我的grain调用返回一个响应之前出错会怎么样？
<!--## What happens if a silo fails before my grain call returns a response for my call?-->

<!--You'll receive an exception which you can catch and retry or do anything else which makes sense in your application logic. The Orleans runtime does not immediately recreate grains from a failed silo because many of them may not be needed immediately or at all. Instead, the runtime recreates such grains individually and only when a new request arrives for a particular grain. For each grain it picks one of the available silos as a new host. The benefit of this approach is that the recovery process is performed only for grains that are actually being used and it is spread in time and across all available silos, which improves the responsiveness of the system and the speed of recovery.-->
你将会收到一个你能够捕获的一场，并且重试或者其他你的应用逻辑中合理的处理。Orleans运行时不会直接从一个出错的silo上重新创建grain，因为他们中的许多或者全部并不需要立即重新创建。相反的，运行时仅在新的请求到达特定的grain时，个别地重建这样的grain。对于这样的grain，运行时选择一个可用的silo作为新的宿主。这样做的好处是，回复过程只有对于真正在使用的grain进行，并且不是在同一时间，并且发生在所有可用的silo上，这提高了系统的灵敏和恢复速度。
<!--Note also that there is a delay between the time when a silo fails and when the Orleans cluster detects the failure. The delay is a configurable tradeoff between the speed of detection and the probability of false positives. During this transition period all calls to the grain will fail, but after the detection of the failure the grain will be created, upon a new call to it, on another silo, so it will be eventually available. More information can be found [here](Runtime-Implementation-Details/Cluster-Management).-->
注意在一个silo出错不可用和Orleans集群检测到出错之间是有延迟的。这个延迟可配置的，要平衡检测的速度和误报的可能性在这期间对其grain的所有调用都会失败，但是在失败检测后，有对grain的一个新的调用，grain将会在其他silo上被创建，所以它最终是可用的。更多的信息可以在 [这里](Runtime-Implementation-Details/Cluster-Management)找到。

## 对一个grain的调用执行了特别长的时间将会发生什么。
<!--## What happens if a grain call takes too much time to execute?-->

<!--Since Orleans uses a cooperative multi-tasking model, it will not preempt the execution of a grain automatically but Orleans generates warnings for long executing grain calls so you can detect them. Cooperative multi-tasking has a much better throughput compared to preemptive multi-tasking. You should keep in mind that grain calls should not execute any long running tasks like IO synchronously and should not block on other tasks to complete. All waiting should be done asynchronously using the `await` keyword or other asynchronous waiting mechanisms. Grains should return as soon as possible to let other grains execute for maximum throughput.-->
由于Orleans使用协调式多工型，它不会自动抢占一个grain的执行，但是Orleans保证对长时间执行的grain调用发出警告，这样开发者能发现他们。协调式多工模型与抢占式多工模型相比有更好的吞吐量。你需要注意grain调用不应该执行任何长时间的任务，比如IO同步并且不要阻塞其他任务的完成。所有等待操作完成都应该使用`await`关键字或者其他的一部机制。grain应该尽量早的返回好让其他的grain执行来达到最大的吞吐量。

## 在什么情况下split brain（同一个grain在多个不同的silo活动）发生？
<!--## In what cases can a split brain (same grain activated in multiple silos at the same time) happen?-->

<!--This can never happen during normal operations and each grain will have one and only one instance per ID.-->
这在正常操作的时候绝不会发生并且每个ID每个grain将有切仅有一个实例。
<!--The only time this can occur is when a silo crashes or if it's killed without being allowed to properly shutdown.-->
唯一能发生这种情况的时候是一个silo崩溃了或者他没有被正常关闭而杀死。
<!--In that case there is a 30-60 seconds window (based on configuration) where a grain can exist in multiple silos before one is removed from the system.-->
在这种情况下，有30-60秒的窗口期（可以设置）一个grain可以存在在不同silo上知道一个被从系统中删除。
<!--The mapping from grain IDs to their activation (instance) addresses is stored in a distributed directory (implemented with DHT) and each silo owns a partition of this directory. When membership views of two silos differ, both can request creation of an instance and as a result. two instances of the grain may co-exist. Once the cluster silos reach an agreement on membership, one of the instances will be deactivated and only one activation will survive.-->
从grain Id到他们的激活（实例）地址的映射被存储在一个分布式目录中（用DHT实现）并且每一个silo拥有这个目录的一个分区。当成员观察到两个silo有差异，两个silo都可以请求创建一个实例。结果就是两个grain的实例共存。一旦集群中的silo对成员达成一致，其中一个实例将会被注销并且仅有一个激活存活。
<!--You can find out more about how Orleans manages the clusters at [Cluster Management](Runtime-Implementation-Details/Cluster-Management) page.-->
你可以在[集群管理](Runtime-Implementation-Details/Cluster-Management)页面找到更多关于Orleans如何管理集群的内容。
<!--Also you can take a look at Orleans's [paper](http://research.microsoft.com/pubs/210931/Orleans-MSR-TR-2014-41.pdf) for a more detailed information, however you don't need to understand it fully to be able to write your application code.-->
你也可以可以看一下Orleans的[论文](http://research.microsoft.com/pubs/210931/Orleans-MSR-TR-2014-41.pdf)了解更多详细信息，然而你不必完全理解就可以写你的应用代码。
<!--You just need to consider the rare possibility of having two instances of an actor while writing your application.-->
你仅仅需要在你写你的应用的时候考虑一个actor有两个实例微小可能性。
