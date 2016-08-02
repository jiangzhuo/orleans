---
layout: page
title: Grain Persistence
---
{% include JB/setup %}

## grain持久化的目标
<!--## Grain Persistence Goals-->

<!--1. Allow different grain types to use different types of storage providers (e.g., one uses Azure table, and one uses SQL Azure) or the same type of storage provider but with different configurations (e.g., both use Azure table, but one uses storage account #1 and one uses storage account #2)-->
<!--2. Allow configuration of a storage provider instance to be swapped (e.g., Dev-Test-Prod) with just config file changes, and no code changes required.-->
<!--3. Provide a framework to allow additional storage providers to be written later, either by the Orleans team or others.-->
<!--4. Provide a minimal set of production-grade storage providers-->
<!--5. Storage providers have complete control over how they store grain state data in persistent backing store. Corollary: Orleans is not providing a comprehensive ORM storage solution, but allows custom storage providers to support specific ORM requirements as and when required.-->
1. 允许不同的grain类型使用不同的存储（例如，一个使用Azure table，一个使用SQL Azure）或者同样类型的存储但是使用不同的配置（例如，都使用Azure table但是一个使用存储帐号＃1，一个使用存储帐号＃2）
2. 允许仅仅改变配置文件而不改变代码来实现一个存储提供者的配置转换（例如，开发－测试－生产之间转换）。
3. 提供一个框架来允许添加之后编写的额外的存储提供者，不管是Orleans团队编写的还是其他人编写的。
4. 提供一小部分生产级别的存储提供者。
5. 存储提供者对于如何在持久化后段存储存储grain的状态有完全的控制权。结论是：Orleans不提供全面的ORM解决方案，但是当有需要 时允许定制存储提供者来支持特定的ORM需求。

## grain持久化API
<!--## Grain Persistence API-->

<!--Grain types can be declared in one of two ways:-->
grain类型可以用一下其中一种方式声明：

<!--* Extend `Grain` if they do not have any persistent state, or if they will handle all persistent state themselves, or-->
<!--* Extend `Grain<T>` if they have some persistent state that they want the Orleans runtime to handle.-->
<!--Stated another way, by extending `Grain<T>` a grain type is automatically opted-in to the Orleans system managed persistence framework.-->
* 扩展`Grain`，如果他们没有任何持久化的状态或者他们自己能够处理所有持久化的状态，或者
* 扩展`Grain<T>`，如果他们有想要Orleans运行处理的持久化的状态。
换句话说，使用扩展`Grain<T>`的方式声明grain类型就是自动选择了Orleans系统管理的持久化框架。

<!--For the remainder of this section, we will only be considering Option #2 / `Grain<T>` because Option #1 grains will continue to run as now without any behavior changes.-->
这节其余的部分，我们只考虑第二种情况`Grain<T>`，因为第一种情况grain会继续运行不会有任何的行为变化。

## grain状态存储
<!--## Grain State Stores-->

<!--Grain classes that inherit from `Grain<T>` (where `T` is an application-specific state data type derived from `GrainState`) will have their state loaded automatically from a specified storage.-->
继承自`Grain<T>`（T是一个派生自`GrainState`的应用相关的状态数据）的grain类将会村特定的存储中自动地加载他们的状态。

<!--Grains will be marked with a `[StorageProvider]` attribute that specifies a named instance of a storage provider to use for reading / writing the state data for this grain.-->
grain将会被一个指定了存储提供者命名实例的`[StorageProvider]`特性所标记，用来为grain读取／写入状态数据。

``` csharp
[StorageProvider(ProviderName="store1")]
public class MyGrain<MyGrainState> ...
{
  ...
}
```

<!--The Orleans Provider Manager framework provides a mechanism to specify & register different storage providers and storage options in the silo config file.-->
Orleans提供者管理框架提供了一个指定&注册不同存储提供者的机制并且把选项存储silo的配置文件中。

```xml
<OrleansConfiguration xmlns="urn:orleans">
    <Globals>
    <StorageProviders>
        <Provider Type="Orleans.Storage.MemoryStorage" Name="DevStore" />
        <Provider Type="Orleans.Storage.AzureTableStorage" Name="store1"
            DataConnectionString="DefaultEndpointsProtocol=https;AccountName=data1;AccountKey=SOMETHING1" />
        <Provider Type="Orleans.Storage.AzureBlobStorage" Name="store2"
            DataConnectionString="DefaultEndpointsProtocol=https;AccountName=data2;AccountKey=SOMETHING2"  />
    </StorageProviders>
```

## 配置存储提供者
<!--## Configuring Storage Providers-->

### AzureTableStorage
<!--### AzureTableStorage-->

```xml
<Provider Type="Orleans.Storage.AzureTableStorage" Name="TableStore"
    DataConnectionString="UseDevelopmentStorage=true" />
```

<!--The following attributes can be added to the `<Provider />` element to configure the provider:-->
下面的特性可以被添加到`<Provider />`元素中来配置提供者：

<!--* __`DataConnectionString="..."`__ (mandatory) - The Azure storage connection string to use-->
<!--* __`TableName="OrleansGrainState"`__ (optional) - The table name to use in table storage, defaults to `OrleansGrainState`-->
<!--* __`DeleteStateOnClear="false"`__ (optional) - If true, the record will be deleted when grain state is cleared, otherwise an null record will be written, defaults to `false`-->
<!--* __`UseJsonFormat="false"`__ (optional) - If true, the json serializer will be used, otherwise the Orleans binary serializer will be used, defaults to `false`-->
<!--* __`UseFullAssemblyNames="false"`__ (optional) - (if `UseJsonFormat="true"`) Serializes types with full assembly names (true) or simple (false), defaults to `false`-->
<!--* __`IndentJSON="false"`__ (optional) - (if `UseJsonFormat="true"`) Indents the serialized json, defaults to `false`-->
* __`DataConnectionString="..."`__ （必选） － Azure storage的连接字符串
* __`TableName="OrleansGrainState"`__ （可选） － 表存储用的表名，默认是`OrleansGrainState`
* __`DeleteStateOnClear="false"`__ （可选） － 如果是true，在清除的时候记录会被删除，否则会写入一条null数据，默认是`false`
* __`UseJsonFormat="false"`__ （可选） － 如果是true，将使用json序列化，否则将会使用Orleans二进制序列化，默人是`false`
* __`UseFullAssemblyNames="false"`__ （可选） － （如果`UseJsonFormat="true"`） 序列化的类型带有完整的程序集名字（true）或者简单的名字（false）, 默认是`false`
* __`IndentJSON="false"`__ （可选） － （如果`UseJsonFormat="true"`） 缩紧序列化后的json，默人是`false`

<!--> __Note:__ state should not exceed 64KB, a limit imposed by Table Storage.-->
> __注意：__ 状态不要超出64KB，Azure Table Storage的强制限制。

### AzureBlobStorage
<!--### AzureBlobStorage-->

```xml
<Provider Type="Orleans.Storage.AzureTableStorage" Name="BlobStore"
    DataConnectionString="UseDevelopmentStorage=true" />
```

<!--The following attributes can be added to the `<Provider />` element to configure the provider:-->
下面的特性可以被添加到`<Provider />`元素中来配置提供者：

<!--* __`DataConnectionString="..."`__ (mandatory) - The Azure storage connection string to use-->
<!--* __`ContainerName="grainstate"`__ (optional) - The blob storage container to use, defaults to `grainstate`-->
<!--* __`UseFullAssemblyNames="false"`__ (optional) - Serializes types with full assembly names (true) or simple (false), defaults to `false`-->
<!--* __`IndentJSON="false"`__ (optional) - Indents the serialized json, defaults to `false`-->
* __`DataConnectionString="..."`__ (必选) - Azure storage的连接字符串
* __`ContainerName="grainstate"`__ (可选) - 使用的blob storage container，默认是`grainstate`
* __`UseFullAssemblyNames="false"`__ (可选) - 序列化的类型带有完整的程序集名字（true）或者简单的名字（false）, 默认是`false`
* __`IndentJSON="false"`__ (可选) - 序列化后的json包含锁进，默认是`false`

<!--
### SqlStorageProvider

```xml
<Provider Type="Orleans.SqlUtils.StorageProvider.SqlStorageProvider" Name="SqlStore"
    ConnectionString="..." />
```
* __`ConnectionString="..."`__ (mandatory) - The SQL connection string to use
* __`MapName=""`__ ???
* __`ShardCredentials=""`__ ???
* __`StateMapFactoryType=""`__ (optional) defaults to `Lazy`???
* __`Ignore="false"`__ (optional) - If true, disables persistence, defaults to `false`
-->

### MemoryStorage

```xml
<Provider Type="Orleans.Storage.MemoryStorage" Name="MemoryStorage"  />
```
<!--> __Note:__ This provider persists state to volatile memory which is erased at silo shut down. Use only for testing.-->
> __注意：__ 这个提供者将状态持久化到独立内存中，在silo关闭的时候将被清除。只用来测试用。

<!--* __`NumStorageGrains="10"`__ (optional) - The number of grains to use to store the state, defaults to `10`-->
* __`NumStorageGrains="10"`__ (可选) - 用来存储状态的grain的个数，默认是`10`

### ShardedStorageProvider

```xml
<Provider Type="Orleans.Storage.ShardedStorageProvider" Name="ShardedStorage">
    <Provider />
    <Provider />
    <Provider />
</Provider>
```
<!--Simple storage provider for writing grain state data shared across a number of other storage providers.-->
一个简单的写入分片到若干存储提供者的数据的简单存储提供者。
<!--A consistent hash function (default is Jenkins Hash) is used to decide which-->
<!--shard (in the order they are defined in the config file) is responsible for storing-->
<!--state data for a specified grain, then the Read / Write / Clear request-->
<!--is bridged over to the appropriate underlying provider for execution.-->
使用一个一致性哈希函数（默认是Jenkins Hash）来决定哪个分片对指定的grain的状态数据存储进行相应，然后读取/写入/清除请求路由到适当的潜在的提供者来执行。

## 关于存储提供者特别说明的
<!--## Notes on Storage Providers-->

<!--If there is no `[StorageProvider]` attribute specified for a `Grain<T>` grain class, then a provider named `Default` will be searched for instead.-->
如果没有给一个`Grain<T>`grain类指定`[StorageProvider]`特性，将会搜索使用名为`Default`的提供者。
<!--If not found then this is treated as a missing storage provider.-->
如果没有找到，就当作缺少存储提供者。

<!--If there is only one provider in the silo config file, it will be considered to be the `Default` provider for this silo.-->
如果silo配置文件中只有一个提供者，它会被当作silo的`Default`提供者。

<!--A grain that uses a storage provider which is not present and defined in the silo configuration when the silo loads will fail to load, but the rest of the grains in that silo can still load and run.-->
一个使用不存在或者没有在silo配置中定义的提供者的grain将会在加载的时候失败，但是其他的grain还是会加载并且运行。
<!--Any later calls to that grain type will fail with an `Orleans.Storage.BadProviderConfigException` error specifying that the grain type is not loaded.-->
之后任何对这个grain的调用都会失败得到一个表示那个grain类型没有被加载的`Orleans.Storage.BadProviderConfigException`错误。

<!--The storage provider instance to use for a given grain type is determined by the combination of the storage provider name defined in the `[StorageProvider]` attribute on that grain type, plus the provider type and configuration options for that provider defined in the silo config.-->
一个grain类型最终使用的存储提供者是通过`[StorageProvider]`特性中的提供者名称加上提供者的类型和silo配置中定义的提供者配置选项决定。

<!--Different grain types can use different configured storage providers, even if both are the same type: for example, two different Azure table storage provider instances, connected to different Azure storage accounts (see config file example above).-->
不同的grain类型使用不同配置的存储提供者，即使是同一个类型：例如，两个不同的Azure table存储提供者实例，连接到不同的Azure storge account（参考上面的配置文件例子）。

<!--All configuration details for storage providers is defined statically in the silo configuration that is read at silo startup.-->
所有的存储提供者的配置细节静态地卸载silo配置中在silo启动时读取。
<!--There are _no_ mechanisms provided at this time to dynamically update or change the list of storage providers used by a silo.-->
现在 _没有_ 动态地更新或者改变silo的使用的存储提供者的机制。
<!--However, this is a prioritization / workload constraint rather than a fundamental design constraint.-->
然而，这是一个优先级/工作量限制，而不是一个基本的设计约束。

## 状态存储API
<!--## State Storage APIs-->

<!--There are two main parts to the grain state / persistence APIs: Grain-to-Runtime and Runtime-to-Storage-Provider.-->
grain状态/持久化API有两个部分：Grain到与形时和运行时到存储提供者。

## grain状态存储API
<!--## Grain State Storage API-->

<!--The grain state storage functionality in the Orleans Runtime will provide read and write operations to automatically populate / save the `GrainState` data object for that grain.-->
Orleans运行时的状态存储功能提供读取和写入操作来自动的填充/保存grain的`GrainState`数据对象。
<!--Under the covers, these functions will be connected (within the code generated by Orleans client-gen tool) through to the appropriate persistence provider configured for that grain.-->
这些功能将会通过为grain配置的恰当的持久化提供者来默默地完成。

## grain状态读写函数
<!--## Grain State Read / Write Functions-->

Grain state will automatically be read when the grain is activated, but grains are responsible for explicitly triggering the write for any changed grain state as and when necessary.
See the [Failure Modes](#FailureModes) section below for details of error handling mechanisms.

`GrainState` will be read automatically (using the equivalent of `base.ReadStateAsync()`) _before_ the `OnActivateAsync()` method is called for that activation.
`GrainState` will not be refreshed before any method calls to that grain, unless the grain was activated for this call.

During any grain method call, a grain can request the Orleans runtime to write the current grain state data for that activation to the designated storage provider by calling `base.WriteStateAsync()`.
The grain is responsible for explicitly performing write operations when they make significant updates to their state data.
Most commonly, the grain method will return the `base.WriteStateAsync()` `Task` as the final result `Task` returned from that grain method, but it is not required to follow this pattern.
The runtime will not automatically update stored grain state after any grain methods.

During any grain method or timer callback handler in the grain, the grain can request the Orleans runtime to re-read the current grain state data for that activation from the designated storage provider by calling `base.ReadStateAsync()`.
This will completely overwrite any current state data currently stored in the grain state object with the latest values read from persistent store.

An opaque provider-specific `Etag` value (`string`) _may_ be set by a storage provider as part of the grain state metadata populated when state was read.
Some providers may choose to leave this as `null` if they do not use `Etag`s.

Conceptually, the Orleans Runtime will take a deep copy of the grain state data object for its own use during any write operations. Under the covers, the runtime _may_ use optimization rules and heuristics to avoid performing some or all of the deep copy in some circumstances, provided that the expected logical isolation semantics are preserved.

## Sample Code for Grain State Read / Write Operations

Grains must extend the `Grain<T>` class in order to participate in the Orleans grain state persistence mechanisms.
The `T` in the above definition will be replaced by an application-specific grain state class for this grain; see the example below.

The grain class should also be annotated with a `[StorageProvider]` attribute that tells the runtime which storage provider (instance) to use with grains of this type.

``` csharp
public interface MyGrainState : GrainState
{
  public int Field1 { get; set; }
  public string Field2 { get; set; }
}

[StorageProvider(ProviderName="store1")]
public class MyPersistenceGrain : Grain<MyGrainState>, IMyPersistenceGrain
{
  ...
}
```

## Grain State Read

The initial read of the grain state will occur automatically by the Orleans runtime before the grain’s `OnActivateAsync()` method is called; no application code is required to make this happen.
From that point forward, the grain’s state will be available through the `Grain<T>.State` property inside the grain class.

## Grain State Write

After making any appropriate changes to the grain’s in-memory state, the grain should call the `base.WriteStateAsync()` method to write the changes to the persistent store via the defined storage provider for this grain type.
This method is asynchronous and returns a `Task` that will typically be returned by the grain method as its own completion Task.


``` csharp
public Task DoWrite(int val)
{
  State.Field1 = val;
  return base.WriteStateAsync();
}
```

## Grain State Refresh

If a grain wishes to explicitly re-read the latest state for this grain from backing store, the grain should call the `base.ReadStateAsync()` method.
This will reload the grain state from persistent store, via the defined storage provider for this grain type, and any previous in-memory copy of the grain state will be overwritten and replaced when the `ReadStateAsync()` `Task` completes.

``` csharp
public async Task<int> DoRead()
{
  await base.ReadStateAsync();
  return State.Field1;
}
```

## Failure Modes for Grain State Persistence Operations <a name="FailureModes"></a>

### Failure Modes for Grain State Read Operations

Failures returned by the storage provider during the initial read of state data for that particular grain will result in the activate operation for that grain to be failed; in this case, there will _not_ be any call to that grain’s `OnActivateAsync()` life cycle callback method.
The original request to that grain which caused the activation will be faulted back to the caller the same way as any other failure during grain activation.
Failures encountered by the storage provider to read state data for a particular grain will result in the `ReadStateAsync()` `Task` to be faulted.
The grain can choose to handle or ignore that faulted `Task`, just like any other `Task` in Orleans.

Any attempt to send a message to a grain which failed to load at silo startup time due to a missing / bad storage provider config will return the permanent error `Orleans.BadProviderConfigException`.

### Failure Modes for Grain State Write Operations

Failures encountered by the storage provider to write state data for a particular grain will result in the `WriteStateAsync()` `Task` to be faulted.
Usually, this will mean the grain call will be faulted back to the client caller provided the `WriteStateAsync()` `Task` is correctly chained in to the final return `Task` for this grain method.
However, it will be possible for certain advanced scenarios to write grain code to specifically handle such write errors, just like they can handle any other faulted `Task`.

Grains that execute error-handling / recovery code _must_ catch exceptions / faulted `WriteStateAsync()` `Task`s and not re-throw to signify that they have successfully handled the write error.

## Storage Provider Framework

There is a service provider API for writing additional persistence providers – `IStorageProvider`.

The Persistence Provider API covers read and write operations for GrainState data.

``` csharp
public interface IStorageProvider
{
  Logger Log { get; }
  Task Init();
  Task Close();

  Task ReadStateAsync(string grainType, GrainId grainId, GrainState grainState);
  Task WriteStateAsync(string grainType, GrainId grainId, GrainState grainState);
}
```

## Storage Provider Semantics

Any attempt to perform a write operation when the storage provider detects an `Etag` constraint violation _should_ cause the write `Task` to be faulted with transient error `Orleans.InconsistentStateException` and wrapping the underlying storage exception.

``` csharp
public class InconsistentStateException : AggregateException
{
  /// <summary>The Etag value currently held in persistent storage.</summary>
  public string StoredEtag { get; private set; }
  /// <summary>The Etag value currently held in memory, and attempting to be updated.</summary>
  public string CurrentEtag { get; private set; }

  public InconsistentStateException(
    string errorMsg,
    string storedEtag,
    string currentEtag,
    Exception storageException
    ) : base(errorMsg, storageException)
  {
    this.StoredEtag = storedEtag;
    this.CurrentEtag = currentEtag;
  }

  public InconsistentStateException(string storedEtag, string currentEtag, Exception storageException)
    : this(storageException.Message, storedEtag, currentEtag, storageException)
  { }
}
```


Any other failure conditions from a write operation _should_ cause the write `Task` to be broken with an exception containing the underlying storage exception.

## Data Mapping

Individual storage providers should decide how best to store grain state – blob (various formats / serialized forms) or column-per-field are obvious choices.

The basic storage provider for Azure Table encodes state data fields into a single table column using Orleans binary serialization.
