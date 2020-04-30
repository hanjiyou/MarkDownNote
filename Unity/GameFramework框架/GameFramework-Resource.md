# 资源加载流程

## 读取版本配置文件（单机模式）

1. `ProcedureCheckVersion`中初始化资源 `ResourceManager.ResourceIniter.InitResource()`**读取 `verson.dat`资源配置文件**，初始化 `ResourceManager`数据成员的，如: `m_ApplicableGameVersion`、`m_InternalResourceVersion`、`Dictionary<string, AssetInfo> m_AssetInfos`普通资产信息、`Dictionary<ResourceName, ResourceInfo> m_ResourceInfos`ab资源信息(streamingAsset路径下，一个ab可能对应多个asset)、`defaultResourceGroup`默认资源组。
2. `ProcedurePreload`中预加载配置、多语言、字体、表格等资源。本质都是调用 `ResourceManager.LoadAsset()`

## LoadAsset流程

1. `ResourceManager.LoadAsset`都是调用 `ResourceManager.ResourceLoader.LoadAsset`
2. `ResourceLoader.LoadAsset` 通过传入的资产名，获取 `ResourceInfo`以及依赖来创建 `LoadAssetTask`加载资产任务，通过 `TaskPool<LoadResourceTaskBase>`任务池统一管理任务的执行。
3. `ResourceLoader.LoadResourceAgent` 加载资源代理。初始化时为代理辅助器绑定读取文件/流完成、成功、失败、更新的事件回调
4. `TaskPool`任务池，控制加载资源代理的初始化、开启、以及更新。
5. `DefaultLoadResourceAgentHelper` 默认加载资源代理辅助器