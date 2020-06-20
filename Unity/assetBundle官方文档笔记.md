
# 文档版本&地址
2019.3 [https://docs.unity3d.com/Manual/AssetBundlesIntro.html](https://docs.unity3d.com/Manual/AssetBundlesIntro.html)
# 官方工具
[Unity Addressable Asset system](https://docs.unity3d.com/Packages/com.unity.addressables@1.8/manual/index.html)
[Unity Asset Bundle Browser tool](https://github.com/Unity-Technologies/AssetBundles-Browser)
[Unity Asset Bundle Browser tool文档](https://docs.unity3d.com/Manual/AssetBundles-Browser.html)

# 基础介绍
## AssetBundle的特性
* AssetBundle能够互相依赖。例如，一个AssetBundle中的材质球能够引用另一个AssetBundle上的贴图。
* AssetBundle能够使用Unity内置的算法进行压缩，以提高传输效率。
## AssetBundle能够做什么
* DLC的更新
* 减少初始安装包大小
* 减小运行时内存压力
* 针对用户的终端（手机）加载优化资源
## AssetBundle文件和AssetBundle对象
* AssetBundle文件中包含的是打入其中的各种资源。
* AssetBundle对象中有一个Key值为路径，Value值为对应资源的字典。

# 工作流程
## 资源的设置
1. 选中要打包的资源，在检视面板的最下面可以看到AssetBundle配置选项。
2. 第一个配置：要将这个资源打进哪一个AssetBundle**待测试**
3. 第二个配置：这个资源的变体**待测试**
## 资源的打包
官网提供了示例代码**待测试**
```
using UnityEditor;
using System.IO;

public class CreateAssetBundles
{
    [MenuItem("Assets/Build AssetBundles")]
    static void BuildAllAssetBundles()
    {
        string assetBundleDirectory = "Assets/AssetBundles";
        if(!Directory.Exists(assetBundleDirectory))
        {
            Directory.CreateDirectory(assetBundleDirectory);
        }
        BuildPipeline.BuildAssetBundles(assetBundleDirectory, 
                                        BuildAssetBundleOptions.None, 
                                        BuildTarget.StandaloneWindows);
    }
}
```
## 加载AssetBundle和AssetBundle里面的资源

### 加载本地文件
官网示例代码**待测试**
```
public class LoadFromFileExample : MonoBehaviour {
    function Start() {
        var myLoadedAssetBundle 
            = AssetBundle.LoadFromFile(Path.Combine(Application.streamingAssetsPath, "myassetBundle"));
        if (myLoadedAssetBundle == null) {
            Debug.Log("Failed to load AssetBundle!");
            return;
        }
        var prefab = myLoadedAssetBundle.LoadAsset<GameObject>("MyObject");
        Instantiate(prefab);
    }
}
```
### 加载网络文件
```
IEnumerator InstantiateObject()
{
    string url = "file:///" + Application.dataPath + "/AssetBundles/" + assetBundleName;        
    UnityEngine.Networking.UnityWebRequest request 
        = UnityEngine.Networking.UnityWebRequest.GetAssetBundle(url, 0);
    yield return request.Send();
    AssetBundle bundle = DownloadHandlerAssetBundle.GetContent(request);
    GameObject cube = bundle.LoadAsset<GameObject>("Cube");
    GameObject sprite = bundle.LoadAsset<GameObject>("Sprite");
    Instantiate(cube);
    Instantiate(sprite);
}
```

# 打包策略
* 开发人员应该明确知道在什么时候什么位置使用什么资源。明确了这个问题后才能够决定使用什么打包策略。

## 依据实体打包
* 举例：
	* 将一个UIPrefab和它所用到的贴图打包在一起
	* 将一个模型和这个模型的材质球，贴图，动画文件打包在一起
	* 将一个场景和这个场景所用到的资源打包在一起
* 特点
	* 每个实体的AssetBundle是分开的，更新单独的实体时不用更新其他没有改变的资源

## 依据资源类型打包
* 特点
	* 不同平台下压缩格式相同的资源AssetBundle可以在不同平台下重用

## 选择正确的打包策略
* AssetBundle是有依赖关系的。如果你的AssetBundle打包策略不好，可能会出现想加载一个GameObject而需要加载多个AssetBundle的情况。
* 上面两种策略当然也可以混合使用。这需要根据具体的项目需求来进行选择。
### 关键点
* 将经常变化的资源和很少变化的资源分开打包。
* 理清资源依赖关系，避免复杂的相互依赖。比如将通用的依赖资源移到Common中。
* 不可能同时加载使用的资源不要打进一个AssetBundle里
* 如果一个AssetBundle中只有一部分资源被频繁的加载，那么需要把这个AssetBundle进行拆分
* 将同时频繁加载的小（资源数少于5-10）的AssetBundle合并到一起。
* 不同版本的同一物体，可以考虑使用变体 **变体具体是什么待测试**

# 打包AssetBundle

## BuildAssetBundleOptions
* 关于打AssetBundle的各种设置  **待测试**
|--|--|
|API|	描述|
|--|--|
|None|	没有特殊设置。使用LZMA进行压缩|
|UncompressedAssetBundle|	不对AssetBundle进行压缩|
|DisableWriteTypeTree|	AssetBundle中不包含类型信息|
|DeterministicAssetBundle	| 确保相同的资源打进AssetBundle里面后，哈希值不会变化 |
|ForceRebuildAssetBundle|	强制重新打AssetBundle|
|IgnoreTypeTreeChanges|	增量打包时忽略类型树变化|
|AppendHashToAssetBundleName|	将Hash值添加到AssetBundle名字之后，能够从名字直接看出AssetBundle是否有变化|
|ChunkBasedCompression|	使用LZ4格式压缩|
|StrictMode|	打包过程中报错则中断打包|
|DryRunBuild|	Do a dry run build.|
|DisableLoadAssetByFileName|	不允许使用文件名字加载资源|
|DisableLoadAssetByFileNameWithExtension|	不允许使用带后缀的文件名字加载资源|
|AssetBundleStripUnityVersion|	构建时删除Unity版本号|

## 关于压缩格式
* 默认情况下，使用LZMA格式创建，使用LZ4格式缓存

### 压缩格式类型
* LZMA：
	* 打出的AssetBundle最小，但加载时间也会更长（加载资源之前需要先将资源所在的AssetBundle整个进行解压）。
	* 资源解压后，会使用LZ4格式重新压缩。
	* 推荐在初始资源下载时使用这个格式。
	* 通过`UnityWebRequestAssetBundle`加载的LZMA格式的AssetBundle会自动解压并重新压缩为LZ4，缓存到本地。如果是使用其他方式下载的Assetbundle,则可以使用`AssetBundle.RecompressAssetBundleAsync`对其进行重新压缩
* LZ4：
	* 加载资源前不必对整个AssetBundle进行解压
	* 允许以块的形式加载资源。解压单个块，即使当前AssetBund中其他的块没有解压，也能使用当前块中的资源
	* 这种压缩模式下`AssetBundle.LoadFromFile`只在内存里加载Bundle的资源目录，而不是Bundle本身
* 完全不压缩：
	* 文件大，加载快

## 打包生成的文件
* AssetBundle文件
	* 场景Bundle和其他一般资源Bundle里面的结构略有不同。场景的Bundle有一些特殊优化。
	![一般资源的AssetBundle结构图]()

* Manifest文件
	* 包含了当前AssetBundle的资源信息以及资源的依赖信息

# AssetBundle依赖

* 一个Bundle中的资源引用另一个Bundle中的资源会产生依赖；一个Bundle中的资源引用一个不在Bundle中的资源不会产生依赖。
* 如果Bundle中资源A引用不在Bundle中的资源B，则资源B会被复制并打进A的Bundle中。
* 如果多个Bundle中的资源引用不在Bundle中的资源B，则资源B会被复制并打进每一个Bundle中。
* 如果要加载的Bundle A中的资源依赖Bundle B，那么加载这个资源前要先将Bundle B加载好。

## 跨AssetBundle的重复信息
* 如果一个Bundle中有一个预制，都引用了不在Bundle中的材质球和贴图
	* 影响最终打出来的Bundle大小增加
	* 运行时内存占用增加
	* 影响Unity自动批处理。Unity把不同Bundle中的材质球都看作是唯一的。

## 编辑器下依赖关系的查询
* `AssetDatabase`：查询依赖关系
* `AssetImporter`：查询资源分配到了哪个AssetBundle

## 几种依赖关系的处理方法
* 共同的依赖资源集中到一个Bundle里面（不适用于复杂交叉依赖的情况）
* 确保同一时间，不会有任何一个Bundle有多重被依赖。（适用于基于Level的游戏，但这样做仍然会增加AssetBundle的文件大小）
* 确保所有依赖的资源都在自己的AssetBundle里面。但这样做机制比较复杂。
	
# AssetBundle的使用

## 加载AssetBundle
* `AssetBundle.LoadFromMemoryAsync`
	* 从内存异步加载AssetBundle
	* 参数为Bundle文件的字节流
	* 使用示例：
	```
	using UnityEngine;
	using System.Collections;
	using System.IO;

	public class Example : MonoBehaviour
	{
		IEnumerator LoadFromMemoryAsync(string path)
		{
			AssetBundleCreateRequest createRequest = AssetBundle.LoadFromMemoryAsync(File.ReadAllBytes(path));
			yield return createRequest;
			AssetBundle bundle = createRequest.assetBundle;
			var prefab = bundle.LoadAsset<GameObject>("MyObject");
			Instantiate(prefab);
		}
	}
	```
* `AssetBundle.LoadFromFile`
	* 未压缩和LZ4格式压缩的AssetBundle，会直接从硬盘中加载。加载LZMA格式的AssetBundle时，会先解压，然后再加载。
	* 加载本地的未压缩资源效率很高。
	* Unity5.3以及以前版本，Android平台使用这个API加载StreamingAssets文件夹中的AssetBundle会失败。因为StreamingAssets文件夹中的内容会在一个压缩的.jar文件中。
	* Unity5.4以及更新的版本能够正常的使用这个API.
	* 使用示例：
	```
	public class LoadFromFileExample : MonoBehaviour {
		function Start() {
			var myLoadedAssetBundle 
				= AssetBundle.LoadFromFile(Path.Combine(Application.streamingAssetsPath, "myassetBundle"));
			
			if (myLoadedAssetBundle == null) {
				Debug.Log("Failed to load AssetBundle!");
				return;
			}
			var prefab = myLoadedAssetBundle.LoadAsset.<GameObject>("MyObject");
			Instantiate(prefab);
		}
	}
	```
* `UnityWebRequest`
	* 相当于原来的`WWW`类的升级版。
	* 能够方便的处理下载以及HTTP的Get/POST请求。
	* 使用示例：
	```
	IEnumerator InstantiateObject()
	{
		string uri = "file:///" + Application.dataPath + "/AssetBundles/" + assetBundleName; 
		UnityEngine.Networking.UnityWebRequest request 
			= UnityEngine.Networking.UnityWebRequest.GetAssetBundle(uri, 0);
		yield return request.Send();
		AssetBundle bundle = DownloadHandlerAssetBundle.GetContent(request);
		GameObject cube = bundle.LoadAsset<GameObject>("Cube");
		GameObject sprite = bundle.LoadAsset<GameObject>("Sprite");
		Instantiate(cube);
		Instantiate(sprite);
	}
	```
* `WWW.LoadFromCacheOrDownload`[已淘汰，直接跳过]

## 加载AssetBundle中的资源

* `T objectFromBundle = bundleObject.LoadAsset<T>(assetName);`
* `Unity.Object[] objectArray = loadedAssetBundle.LoadAllAssets();`
* 
	```
	AssetBundleRequest request = loadedAssetBundleObject.LoadAssetAsync<GameObject>(assetName);
	yield return request;
	var loadedAsset = request.asset;
	```
* 
	```
	AssetBundleRequest request = loadedAssetBundle.LoadAllAssetsAsync();
	yield return request;
	var loadedAssets = request.allAssets;
	```

## 加载Manifest文件
* 加载Manifest文件是为了解决依赖问题
* 加载Manifest文件的示例代码
	```
	AssetBundle assetBundle = AssetBundle.LoadFromFile(manifestFilePath);
	AssetBundleManifest manifest = assetBundle.LoadAsset<AssetBundleManifest>("AssetBundleManifest");
	```
* 查询并加载依赖的示例代码
	```
	AssetBundle assetBundle = AssetBundle.LoadFromFile(manifestFilePath);
	AssetBundleManifest manifest = assetBundle.LoadAsset<AssetBundleManifest>("AssetBundleManifest");
	string[] dependencies = manifest.GetAllDependencies("assetBundle"); //Pass the name of the bundle you want the dependencies for.
	foreach(string dependency in dependencies)
	{
		AssetBundle.LoadFromFile(Path.Combine(assetBundlePath, dependency));
	}
	```
	
# 管理加载的AssetBundle
	* 不正确的卸载AssetBundle可能导致复制内存中的object或者其他问题，比如贴图丢失。
## `AssetBundle.Unload(bool)`
* 卸载AssetBundle的头部信息

### 参数详细说明
* 参数为true时 也卸载AssetBundle所有的实例化对象，这个实例化对象不包含对象的复制（复制的资源不是AssetBundle里面的资源）。
* 多数情况下，会使用true来保证内存中没有重复的对象
* 如果参数必须为false的话，内存中无法访问的资源只能通过以下方式卸载
	* 手动调用`Resources.UnloadUnusedAssets`
	* 非添加方式加载场景，会自动调用`Resources.UnloadUnusedAssets`

#### 详细例子
* 从AssetBundle中加载的一个GameObject资源，true卸载时，会把这个GameObject资源卸载掉，而不会把Instantiate的GameObject也卸载掉。
* 现在从AssetBundle中加载一个材质球在活动场景中使用
	* 使用Unload(true)卸载时，材质球在场景中的所有实例一样会被卸载和销毁。**这里有个疑问待测试：Instantiate的材质球会不会也被销毁**
	* 使用Unload(false)卸载时，会打断当前材质球和AssetBundle的链接。如果AssetBundle重新加载，它并不会链接到已经存在的材质球。此时加载该材质球时，会复制一个新的。
	
## 保证资源不重复的方法
* 在恰当的时机卸载暂时加载的AssetBundle，比如Loading时
* 记录资源加载的引用计数，避免重复加载资源和异常卸载资源

# AssetBundle缓存
* Unity维护两个缓存，分别是内存缓存和硬盘缓存。
* 将AssetBundle中加载到内存中，会消耗大量的内存空间。除非有特别频繁快速访问AssetBundle中内容的需求，否则请使用磁盘缓存。
* 如果你向`UnityWebRequest`提供了一个版本参数（版本号挥着Hash），Unity会将AssetBundle数据存储到本地硬盘。如果不提供参数，Unity将会使用内存缓存。
* `Caching.compressionEnabled`设置为true时，存储到本地的AssetBundle文件会以LZ4的格式进行压缩；设置为false时，则存储到本地的文件不会压缩。
* 使用LZMA格式初始加载会耗费比较长的时间。因为它需要先解压，然后以指定格式存储到硬盘缓存。之后加载会直接使用缓存来加载。
* 推荐使用`UnityWebRequest`，因为`AssetBundle.LoadFromFile`和`AssetBundle.LoadFromFileAsync`加载LZMA后会使用内存中的缓存。如果不能使用`UnityWebRequest`，可以使用`AssetBundle.RecompressAssetBundleAsync`手动将AssetBundle数据写到硬盘缓存上。
* 内部测试表明，使用磁盘缓存和内存缓存存在着一个数量级的差异。需要根据项目的需求自己权衡。

## 缓存类型
1. 内存缓存：AssetBundle为不压缩格式
2. 硬盘缓存：在可写空间以指定格式压缩保存AssetBundle文件

# AssetBundle补丁
* 下载一个新的AssetBundle来替换现有的AssetBundle
* 使用`UnityWebRequest`API时，会根据传入的版本参数自动触发下载新的AssetBundle
* Unity使用固定方式生成AssetBundle文件，也就是说，资源不变，生成的AssetBundle也是不变的。正因为这样，可以自定义下载器来区分补丁差异。
* Unity内部并没有实现差分补丁的功能，如果有差分补丁的需求的话，需要手动实现。（差分补丁：新AssetBundle和旧AssetBundle进行比较生成补丁。客户端下载补丁并经过处理生成新AssetBundle）

## 如何确定要替换的AssetBundle
* 从服务器获取要下载的资源列表和版本信息。和本地的资源列表版本信息做对比。如果本地资源缺失或者版本号不对，则需要下载替换这个AssetBundle。
* 也可以自定义系统来检测AssetBundle变化，比如MD5，JSON等。

# 常见问题

## 自动生成的图集
* 如果一个图集的所有sprite都打进同一个Bundle，则自动生成的图集将会打进这个Bundle。
* 如果sprite被打进多个Bundle，那么图集会复制多分分别打进对应的Bundle。
* 如果sprite没有被打进Bundle，那么图集也不会被打进Bundle
* Unity5.2.2p3以及以前的版本对图集打Bundle的支持有问题。

## Android Texture
* Android设备碎片化问题严重，需要将贴图压缩成对应的格式。
* ETC1没有透明通道，但基本上所有设备都支持
* ETC2，旧设备可能不支持
* 可以使用变体，将不同变体的贴图分别打在一个AssetBundle里面，根据情况进行加载。（需要贴图设置正确）
* `SystemInfo.SupportedTextureFormat`可以使用这个API来检测设备支持什么格式的贴图
















