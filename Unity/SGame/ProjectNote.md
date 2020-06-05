# ProjectNote
## 总结
1. 战斗时阵位pos
* 1-10<=>英雄
* 10<=> 敌人
## PageWindow 简介
1. ui的继承结构(从上到下以StoryDialogWindow为例)
(1) **FairyGUI:** IEventDispatcher->EventDispatcher->GObject->GComponent->Window->(2)
(2)**项目:** SmartResWindow->PageWindow(页窗口)->StoryDialogWindow
2. `StoryDialogWindow`的生命周期方法
OnInit()->OnShown()->InitData()
**注意：OnInit的调用顺序是:UIManager.CreatePageWindow()->Window.Init()->SotryDialogWindow.OnInit(),OnShown()同**
### PageWindow 页窗口父类
*签名:class PageWindow : SmartResWindow*
1. 成员详解
* `virtual WindowPage selfPage{getter null}` 
* `virtual InitData(WinPageArgs)` 初始化数据 因为窗口会缓存，所以务必在这里重置变量
* `virtual Refresh()` 界面后退的时候，获取页窗口缓存池的最上层窗口，调用其Refresh()方法
* `virtual void SetData(EventArgs args)` 
## EnforceWindow 简介
*签名：class EnforceWindow:SmartResWindow*
1. UI的继承结构(从上倒下 以引导泡窗口为例)
Window->SmartResWindow->EnforceWindow->GuideBubbleWindow
2. EnforceWindow的成员（虚方法以及成员）
* `EnforceWindow()`构造函数 将继承自`GObject.sortingOrder`设为1000
* `virtual EnforceWindowPage selfPage{getter null}`  当前界面的枚举类型
* `virtual bool AllowMutli=>false`
* `virtual void SetData(EventArgs args)` 传递数据 等同于`PageWindow.SetData()`
**注:PageWindow窗口和EnforceWindow的区别：前者同时刻只能有一个，后者可以有多个。后者一般对应于弹出框等**
3. 生命周期方法调用顺序
   1. UIManager.OpenEnforceWindow
   2. Window.Show()->Window.OnInit()//虚方法，如果未初始化调用，执行子类重写的方法,只调用一次
   3. Window.OnShown()//虚方法 每次打开界面都调用
   4. OpenEnforceWindow->SetData()  每次都调用

## UIManager
2. 功能函数
* PageWindow相关
    * `PageWindow OpenPageWindow(WindowPage page,WinPageArgs args=null)` 打开指定`WindowPage`枚举类型的页窗口，并将参数`WinPageArgs`传递给目标窗口
    * `PageWindow CreatePageWindow(WindowPage windowPage, WinPageArgs args = null)` 创建窗口，如果已经创建则从回收池里直接返回。page的`Show()`和`InitData(args)`依次调用
    ```     
        if (windowPage == WindowPage.Null) return null;
        var wp = PopRecylePage(windowPage) ?? NewPage(windowPage);
        wp.Show();//此时 通过基类Window的虚方法会依次调用当前PageWindow的OnInit()、OnShown()
        SetWinContentScaler(wp.contentPane, wp.curOpContentPane);
        wp.InitData(args);//通过基类PageWindow 调用当前pageWindow的InitData()
        return wp;
    ```
    * `void SetCurWindow(PageWindow winPage)` 设置当前窗口
* EnforceWindow相关
    * `OpenEnforceWindow(EnforceWindowPage page,EventArgs args=null)` 打开指定`EnforceWindowPage`枚举类型的窗口
    ```
        var enforceWin = PopRecyleEnforce(page) ?? NewEnforceWindow(page);
        if (enforceWin == null) return null;        
        enforceWindowList.AddLast(enforceWin);
        enforceWin.Show();//window.Show()
        SetWinContentScaler(enforceWin.contentPane, enforceWin.curOpContentPane);
        enforceWin.SetData(args);//调用当前窗口重写的SetData()并将窗口的事件参数传递
        enforceWin.RegisterEvent();
        if (GameEngine.Instance.loginMgr.IsState(LoginState.LoginAllSuccess))
        {
            GameEventSystem.Instance.PostEvent(MessageDefine.GuideOpenEnforceWindow, new GuideOpenEnforceWindowArgs((int)enforceWin.selfPage));
        }
        PlayMusic(page);
        return enforceWin;
    ```
### UIManager_Window 界面注册类
*签名:public partial class UIManager*
1. 外部成员
* `enum WindowPage` 定义所有的Page窗口对应的枚举
* `enum EnforceWindowPage` 定义所有的EnforceWindow弹出框对应的枚举类型
* `enum WindowPanel` 
* `class WinPageArgs` 书页窗口参数基类
2. 字段
* `delegate PageWindow PageFactoryNew()` PageWindow类型的委托
* `Dictionary<WindowPage, PageFactoryNew> pageWindowFactory` 存储所有的Page窗口.key:窗口对应的枚举类型;value:PageWindow委托,本质是每个PageWindow子类的无参构造委托,如`() => new ZodiacLoginWindow()`.
3. 方法
* `void RegisterPageWindow()`  注册Page窗口 保存每个窗口枚举对应的无参构造委托
``` 
    void RegisterPageWindow(WindowPage window, PageFactoryNew factory)
    {
        pageWindowFactory[window] = factory;
    }
```
*注意:修改fgui当前包体显示不同的Component面板的方法:`contentPane = obj.asCom;`*
## Asset加载
### ConfigLoader 配置加载器
1. 作用：用于加载每个表格专程的asset文件，加载完成后将asset文件序列化的数据保存到相应到集合种
2. 成员方法详解
* `Dictionary<string, CallbackScriptableObject> dictData` 保存所有加载过的asset。key:是资产文件名，value：所有asset绑定类的基类`CallbackScriptableObject`。
* `Init()` 初始化单例，被调用在**GameEngine.InitOther()**中
* `CallbackScriptableObject GetData(string assetFileName)` 加载指定名称的asset文件，保存到dictData字典中，并调用`CallbackScriptableObject.OnLoadFinished()`，最后返回该cso对象


## 其他界面
### Guide 引导模块

#### GuideDataSet引导数据集

1. 数据来源表格
* `List<GuideData> guideList` 对应表格**Guide**。数据类`GuideData`，相比表格新增了两个字段`List<GuideCondition> conditionList`、`List<GuideContent> contList` 保存引导条件和内容列表
* `List<GuideContentData> guideContentList` 对应表格**GuideContent**。数据类`GuideContentData`，其中contentType字段对应每个GuideContent类，在asset加载完成时，被绑定到`contentFactory`中。
2. 数据字典
* `Dictionary<int, GuideData> guideDict` 引导数据字典。key保存GuideData的id；value保存自身。其中，每一项的conditionList集合保存的是根据GuideData对应的GuideCondition字段创建的GuideCondition对象，constList保存的是根据`GuideData.contentList`字段指定的id获取到的GuideContent对象。
* `Dictionary<int, GuideContent> contentDict` 引导内容字典。key保存GuideContent的id；value保存自身。 
3. 成员详解
* `GuideData.conditionList` 当前引导触发条件的列表`GuideCondition类型`，通过判断引导的3个condition是否存在，存在则根据condition和后面对应的两个参数传递给新建的GuideCondition子类的`param()`中
* `GuideData.contList` 当前引导的内容列表，根据表格里的`contentList`列表获取到每个对应的`GuideContent` 添加到`contList中`
### GuideManager引导管理器
1. 方法
* `CheckGuide(GuideConditionEnum cond, params int[] param)` 从第一个引导开始检查，如果第一个执行则获取下一个引导。遍历每个引导的触发条件`GuideCondition`列表，如果满足触发条件，则执行该引导的`GuideContent`.
**注意：GudieData数据类里的List<GuideCondition> conditionList，List<GuideContent> contList**
2. 打开扭蛋引导的流程
GuideManager.CheckGuide()//获取当前的扭蛋引导->GuideGoWindow.Do()//打开扭蛋界面

### StoryDialog 剧情模块
#### StoryDataSet数据集
1. 数据来源表格
* `List<StoryData> storyList`对应表格**Story**，数据类:`StoryData`
* `List<StoryCharacter> storyCharList` 对应表格**StoryCharapater**，数据类：`StoryCharapater`
2. 数据字典
* `Dictionary<int,List<StoryData> storyDict` key保存StoryData的dialogId；value保存相同dialogId的所有StoryData集合
* `Dictionary<int, StoryCharacter> storyCharDict` key保存StoryCharacter的id；value保存StoryCharacter
3. 功能函数
* `public List<StoryData> GetStoryData(int id)` 通过dialogId获取对应的全部StoryData的集合
* `GetStoryCharacter(id)` 通过id获取StoryCharacter对象
4. 重要字段解析
* **Story表格**
    * `align` 代表了剧情对话的位置类型。3:中心独白，2：右方对话(一般敌人)，0:左方对话（一般主角）
    * `dialogId` 对应对话故事的id，一般一个**dialogId**(LevelStory的id)会对应多个Story对话项
### LevelDataSet数据集
**注意：真正保存到服务器的是LevelStory的id，LevelDataSet通过该id获取当前剧情的index，然后通过index从storyList中获取剧情数据**
1. 数据来源表格
* `List<LevelStoryData> storyList` 对应表格**LevelStory**，数据类：`LevelStoryData`
2. 数据字典
* `Dictionary<int, int> storyDict` key保存LevelStoryData的id；value保存遍历storyList时的当前索引
#### StoryDialogWindow 剧情窗口
*签名:class StoryDialogWindow::PageWindow*
1. 显示对话框的流程
 InitData()->ShowStory()->**{ShowStoryCoroutine()->ShowPicture()->ShowDialog()->NextStory()/ShowDialog() }**->OnNextDialog() 
 *注:黑体方法全是协程*
* `ShowDialog()` 该协程会判断waitTime属性，如果等待则不显示文本完成等待后调用`OnNextDialog()`，不等待则加载文本和spine
* `OnNextDialog()` 判断当前对话是否播放完，未播放完调用`ShowDialog()`，否则调用`NextStory()`
* 
2. 功能函数
* `SaveCheckpoint(levelId)` 保存当前剧情LevelStoryData的id
#### StoryDialogWindowArgs 剧情窗口参数
*签名：class StoryDialogWindowArgs : WinPageArgs* 

### ZodiacGachaMainWindow 扭蛋界面
*签名:public class ZodiacGachaMainWindow : PageWindow*
Fgui包名:"ZodiacGachaWindowV2"，对应窗口枚举类型:`WindowPage.GachaMainWindow`

### ZodiacBattleSettlementWindow 战斗结算界面
*签名：class ZodiacBattleSettlementWindow : PageWindow*
Fgui包名:"BattleUI.BattleItemSettlementWindowV2",对应窗口枚举类型：`BattleEndWindow`

1. 战斗流程
`BattleMod.OnBattleEnd(arg)`->`BattleMod_Test.SendLeveaSceneMsg()`->`UIManager.OpenPageWindow(WindowPage.BattleEndWindow,data)`->略->`ZodiacBattleSettlementWindow.OnInit()`
2.
*类名与类文件名不一致*
### HeroExpComp 英雄经验结算辅助类
### 战斗相关的数据
#### Team相关
1. **TeamInfo** 队伍信息的Proto类
* `List<long> heros` 当前队伍的所有英雄id
#### MobBattle 战斗信息基类
*签名:public abstract class ModBattle : IMod*
* `List<Avatar> avatars` 当前战斗场景中存在的骨骼(包括英雄和敌人)
* `GameBattle_BattleEnd endMsg` 战斗结束后的消息(里面包含杀死,被杀,最大血量,剩余血量等信息)
#### GameBattle_BattleEnd 战斗结束后proto数据类
* `List<BattleHeroInfo> _heros ` 获取战斗场景中所有的人物的数据`BattleHeroAttr`.`BattleHeroAttr`里包含`hp`,`maxHp`,`attack`等英雄的生命属性
* `GameBattle_Statistics _stat` 战斗统计数据字段,如`killNum`,`deadNum`等随着战斗不断变化的数据
### LoadingWindow
切换战斗场景时的云彩
### TransformWindow
切换界面时的渐变透明效果，比如点击登陆主角渐白的效果，本质就是一个白色/黑色的遮罩

## 数据管理

### 静态数据--数据表

1. 两种获取方法（以Description为例）
* `DataManager.Instance.descriptionDataSet.GetDescription()` 通过DataManager中的表数据集引用获取
* `GameDataCtrl.Instance.GetDescriptById(_levelId)` 通过再次封装的游戏数据控制器获取
2. 中文描述的两个表
* `Description` 服务器与客户端通用，一般通过id获取，如:`GameDataCtrl.Instance.GetDescriptById`
* `ClientDescription` 客户端专用表，比如“射手座”等星座中文，通过字符串类型的key获取，如`GameDataCtrl.Instance.GetClientDescriptByString

3.`DataManager`、`GameDataManage`、`GameDataCtrl` 三者关系

* `DataManager`  
  * 包含所有客户端表格的数据对象(继承结构:`ScriptableObject`->`CallbackScriptableObject`->数据对象)
  * 包含通过`ConfigLoader`加载表格asset数据的异步协程`LoadAssetAsyn`。
* `abstract GameDataManage`
  * `Interface ISkill` 技能接口。封装了获取Id、Name、Icon、Des、CD等属性的方法
  * 其他获取英雄、领队、玩家、队伍信息的方法

* `GameDataCtrl` 继承了抽象类 `GameDataManage`

### 动态数据--ServerDataManager

1. `ServerPlayerDataplayerData` 

### 本地数据

1. `ClientData` 客户端数据 当前设备所有用户都通用

2. `UserClientData` 用户数据，只为当前客户端当前用户使用

   *注：不能保存float、double浮点型数据*

## ResourceLoader 资源加载
### ResourceLoader_Path
*签名：public partial class ResourceLoader*

1. 字段 各种类型资源的父级目录
* `AudioPath`
* `AvatarPath`
* `SpinePath`
* `UIEffectPath`
* 略
2. 方法
* `public static AudioClip GetAudio(string name)` 获取音频资源
```
 return GetObjectDirectly<AudioClip>(string.Format(AudioPath, name), false);//调用Resource.GetObject<T>()
```
* `public static Avatar GetAvatar(string name)`
* `private T GetObject<T>(string pkgName, string resName, bool cache) where T : UnityEngine.Object` 获取指定路径的对象
```
        T reObject = default(T);
        if (cache)
        {
            // 如果包存在，那么包里的已全部导入
            if (objectMaps.ContainsKey(pkgName))
            {
                var objPool = objectMaps[pkgName];
                var obj = objPool.GetObj<T>(resName);
                if (obj != default(T))
                {
                    return obj as T;
                }
                return obj;
            }
        }
        if (bundleInitialized)
        {
            reObject = GetDirectlyFromBundle<T>(pkgName, resName, cache);
        }
        if (reObject == default(T))
		{
            reObject = GetDirectlyFromResources<T>(pkgName, resName);
        }
        return reObject;
```
### 资源使用
1. 播放特效的流程
```
        GameObject defeatEffectObj = ResourceLoader.GetUIEffect("ui_Battle_Block_R_01");
        MaterialGoWrapper defeatEffectWrapper=new MaterialGoWrapper(GameObject.Instantiate(defeatEffectObj));
        _defeatEffectPos.SetNativeObject(defeatEffectWrapper);
```
2.加载Spine的流程

```
        var skilData = ResourceLoader.GetUISpine/GetBattleSpine(heroData.animID);//加载spine资源，大英雄用前者，小英雄用后者
        var skelAni = SkeletonAnimation.NewSkeletonAnimationGameObject(skilData);//创建spine对象
        UITool.SetSpineAnim(skelAni,"start",false);//播放spine动画 一定要先设置loop 再设置名称
        _spineWrapper = new GoWrapper(_skeleAnim.gameObject);
        _spineWrapper.supportStencil = true;
        _spinePos.SetNativeObject(_spineWrapper);//设置spine对象的父物体
        _spineWrapper.gameObject.transform.localScale = new Vector3(60, 60, 1);
```

## EventSystem 事件派发器

1. 变量

   * `Dictionary<int, List<EventSystem.EventHandler>> events` 事件字典映射
   *  `List<EventSystem.InvokeParam> waitEvent` 等待中的事件

2. 方法

   * `RegisterEvent(object evt,EventSystem.EventHandler handler)` 注册事件

     ```c#
        int key = (int) evt;
           if (!this.events.ContainsKey(key))
          this.events.Add(key, new List<EventSystem.EventHandler>());
           this.events[key].Add(handler);
     ```
   
    
   
* `UnRegisterEvent(object evt, EventSystem.EventHandler handler)` 解绑事件
  
   * `PostEvent(object evt, EventArgs args)` 抛出事件 
   
     ```c#
     this.waitEvent.Add(new EventSystem.InvokeParam(evt, (EventArgs) args));
     ```
   
* `InvokeEvent(object evt, EventArgs args)` 立刻执行事件
  
     ```c#
           List<EventSystem.EventHandler> eventHandlerList;
           if (!this.events.TryGetValue((int) evt, out eventHandlerList))
             return;
           for (int index = 0; index < eventHandlerList.Count; ++index)
             eventHandlerList[index](args);
     ```
   
   * `Update()` 遍历正在等待中的事件，依次执行

# 新总结

## 战斗相关

### 大地图战斗返回的各种情况和调用

1. 星球挑战普通关卡
   * 进入战斗 OnDispose() 
   * 胜利 OnShown()=>RecoverBackup()
* 失败 OnShown()=>RecoverBackup()（同上）
  
2. 星球挑战装备本
   * 胜利 Dispose()=>OnInit()=>OnShown()=>InitData()
   * 失败 同上
3. 星系挑战装备本
   * 胜利 同2
   * 失败 同上
4. 悬赏任务
   * 进入战斗 OnDispose() 这时候 MainMap对象里的数据和资源会被释放，想要避免，使用static
   * 胜利 OnShow()=>OnRecoveryBackUp()
   * 失败 同上

## 资源热更新流程

### 步骤总结:

* 打开ResUpdateWin
* 检测是否联网，然后根据配置是否需要从bundle加载资源，如果是bundle加载资源，走更新流程，否则走登录流程
* 如果走热更新流程，则先需要读取本地bundlerVersion，读取结束后，连接服务器
* 连接登录服务器
* 连接成功后向服务器查询版本号信息，决定是否要整包更新还是只更新资源
* 只更新资源，则走热更新流程

### 详细介绍

1. 打开ResUpdateWin
   * 打开win后，通过事件，切换到下一个流程
2. 检测是否联网
   * `if(Application.internetReachability == NetworkReachability.NotReachable)`
   * 联网则判断资源加载方式是否为bundle，是则进入热更新流程
3. 读取本地version
   * 读取沙盒路径 `Application.persistentDataPath`下的版本文件，将版本号和资源列表存在docVersion中，不存在则docVersion版本号为0。注意,是通过 **io流`StreamReader`直接读取的**
   * 读取 `Application.streamingAssetsPath`下的版本文件，缓存版本号和资源列表。备份streamVersion对象。注意，此处的读取是通过**UnityWebRequest读取**的
   * 如果docVersion<=streamVersion，则删除DocPath和TempAssetsPath下的所有文件(说明之前的临时和沙盒路径的文件是老的，需要重新下载)。否则保存到 `AssetsManifestManager`
   * 本地版本引用streamVersion和docVersion中最新的一个（版本号最大的），然后将此版本对象设置为版本版本
   * 切换到下一个状态 `UpdateConnectLoginServerNode`
4. 连接登录服务器
   * 连接登录服务器成功后，根据资源加载方式是否来自bundle，是则切换到 `UpdateCheckVersionMarkNode`状态，否则直接切换到更新完成状态
5. 向服务器查询版本号信息
   * 从cdn上下载versionMark.txt文件反序列化到VersionMark实例对象，里面记录了所有平台的版本号
   * 切换到 `UpdateDownNewVersionNode`状态
6. 获取服务器版本资源文件以及下载资源
   * 下载服务器versionFile，并解析数据（包括最新版本号，以及全部最新资源列表BundleInfo(路径|Hash值|size)）。
   * 判断是否可以断点续传。判断 `Application.temporaryCachePath`目录下是否存在versionFile文件，有则判断版本号是否与本地version相同，是则可以断点续传
   * 遍历下载列表，为每个 `UnityWebRequest.downloadHandler`设置为

### 注意

1. 读取version文件的区别
   * 在读取沙盒路径下的文件时，由于是sd卡本地文件，可以通过IO流直接读取。
   * 读取 `StreamingAssets`下的版本文件时，需要通过Unity自带的类 `WWW`或者 `UnityWebRequest`来读取，否则只能使用其他软件来查看.jar内的文件并获取。原因如下
     * 原因:安卓平台下，该文件夹下的所有文件都会被压缩到jar中(不是java类库，是一种压缩格式)，通过一般路径直接打开会有问题。 IOS上不存在该问题，但是是只读的。WebGL平台下没有文件权限访问，也可通过 `WWW`访问

## 项目战斗详细流程

### C#流程

1. 登录游戏的流程

   * `GameEngine.InitOther` 登录时进行一系列的初始化，包括 `BattleManager.Init()`

   * `BattleManager.Init()`流程

     * 首先初始化Lua接口 `BattleLuaInterface.Init()`。过程如下：注册 `BattleMsgDefine`到一个新表，使用表名保存到luaenv的全局表中(lua就可以直接使用)。也可以这样直接在C#中设置lua的全局变量 。然后加载 `BattleInterface`Lua文件，并获取到该文件中的各个关键方法(保存到LuaFunction)。

       ```c#
               LuaTable msgConst = luaenv.NewTable();
               var itr = GameEngine.Ins.DataMgr.constantDataSet.ConstanList.GetEnumerator();
               while (itr.MoveNext())
               {
                   ConstanTableData tableData = itr.Current as ConstanTableData;
                   msgConst.Set(tableData.name, tableData.value);
               }
       
               luaenv.Global.Set("ConstTableData", msgConst);//设置lua的全局Table
               luaenv.Global.Set("isUseDebug", GameEngine.Ins.luaDebug);//设置lua全局变量
               luaenv.DoString("require 'BattleLib/BattleInterface'");
               initFunc = luaenv.Global.Get<LuaFunction>("Battle_InitManager");
               createFunc = luaenv.Global.Get<LuaFunction>("Battle_CreateRoom");
       ```

     * 分别初始化 `GameBattle_InitSkill` 和 `GameBattle_InitSummoned`的数据 发送到Lua，Lua会接受保存这两条消息的数据，不会回包

2. 进入战斗的流程

* 从战斗入口，打开战队配置界面(包含战斗的基本信息：levelInfoId、战斗类型等)
* 战队配置界面选好阵容，保存到服务器和本地，发送 `enterScene`消息
* 收到 `enterSceneRet`消息进行处理：保存阵容、备份窗口（战斗退出恢复界面的时候用到）
  * `CreateBattleRoom` 调用lua的 `room.new()`创建新房间
  * 填充 `GameBattle_Init`消息的各种数据（**lua战斗计算的数据来源**），发送给lua，lua会调用 `root.init`初始化房间的数据。然后lua回包给C#
  * `BattleLuaInterface.OnMsgBack` 是C#提供的lua调用的回包函数。
* lua端处理后，会抛出 `GameBattle_Init`战斗事件，C#收到该消息，并派发 `TransmitMod`事件进入 loading逻辑
* `LoadingManager` 监听 `TransmitMod`事件，然后在 `GameEngine.TransmitLater`根据参数的type 切换到不同的 `Mod.Init()`

2. 详细步骤（暂时不写 没必要）

* 大地图。填充`levelInfoId`、`SceneType`(玩法类型)、`TeamTag`(一般是pve)，打开战队配置界面

* 战队配置界面。配置队伍后保存当前队伍(①发送给服务器 ②更新ServerTeamData ③保存到本地UserClientData)，调用 `ZodiacGoToFightLogic`进入战斗逻辑，会发送 `GameMessage_CGEnterScene`消息

* 收到 `EnterSceneRet`回包后，备份当前窗口，再次更新ServerTeamData，调用 `BattleManager.OnEnterSceneRet`

* `BattleManager`收到回包后，填充 `GameBattle_Init`，并发送给lua战斗。填充数据如下：
* 直接从LevelInfo表里填充或固定的数据：`configId` 即levelId、`battleType`战斗类型、`energyCostType`消耗能量方式、`initEnergy` 初始能量值、`initSp`初始鬼火、`addSpList`每次充能满增加鬼火、`energyType` 能量类型列表（2个，敌我？）
  
* 根据 回包的英雄数据，添加到 `initMsg.heros`中，注意Leader和Hero的区别，然后将Leader和Hero的数据统一封装到BattleHeroInfo中。



### Lua流程

