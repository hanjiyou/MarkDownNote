# Knowledge
## 基础部分
### Coroutine 协程
*为了符合规范要求且便于控制，启动协程时，建议保存改协程的引用*

1. 协程启动三种方法
*  `_coroutine=StartCoroutine(Coroutine1())` 
* `_coroutine1=StartCoroutine("Coroutine1")`
* `yield return Coroutine2()` 这个方法是在协程Cor1中启动协程Cor2并等待其结束。**使用该方法启动的协程，①在协程链外部，停止头  头尾都会停止 ②尾部协程 停止头  头停止 尾部执行**
2. 协程结束的方法
* `StopCoroutine(_coroutine1)` 
* `StopCoroutine("Coroutine1")`
3. 返回对象的时机
`_cor1=StartCoroutine(Coroutine1())` 返回对象的时机是协程遇到yield  return 的时候，即在yield return之前，_cor1都是空。
4. 协程的几种类型
* 同步等待 如：`yield return new WaitForSecondes()`
* 异步协程 如: 在现有协程中直接开启一个新的协程`StartCoroutine()`
* 同步协程 如：现有协程A中，通过`yield return StartCoroutine(B())` 开启一个协程，A会等待B完成后，才继续向下执行
* 并行协程 如：现有协程A中，通过`Coroutine b = StartCoroutine(B())` 开启异步协程B，A和B会同时执行，但是又通过`yield return b` 阻塞A，直到B完成A才继续向下执行
5. 协程链
* 协程链的形成：现有协程中，通过`yield return B()` 直接进入协程B，B会作为A的子协程运行。
* 协程链的停止
    * 在A的非子协程中，停止协程A，A和A的所有子协程都会停止
    * 在A的子协程中停止协程A，只会停止协程A
### 字符串格式化
1. `ToString()`
* `ToString("#.###")` 如果数值>1 则返回正确如3.345 如果<1 则返回.345
2. `string.Format(string,3.5456)` 格式自定字符忽略大小写
* `{0:C3}` 货币 ￥3.545
* `{0:F3}` 浮点数三位小数 3.545
* `{0:p3}` 百分比3位小数 54.536% 默认保留两位小数（会四舍五入）

### 浮点数格式化

1. 保留指定小数位:`Math.Round(value,length)`
2. float转int
   * 强转(int):该方法不是四舍五入，而是把小数全部舍弃
   * Convert.ToInt16()：四舍五入

## 继承与多态

一、继承

1. 创建子类对象，会默认调用父类的**无参构造**，有参数时不调用无参构造。
2. 子类构造后默认且省略了`base()`，子类通过`base(args)`来指定调用父类相应的构造，此时也会使无参父构造不执行。
3. 初始化顺序：子类成员字段 、父类成员 、父类构造、子类构造
4. 方法的重写：子类实例只调用子类方法，父类同。子类调用重写方法时，可通过`base.MethodName()` 

二、多态

1. 父类代码

```c#
public class ParentDemo1{    
    private int a = 5;    
    int Name{get {        Debug.Log("hhh");        return 1;}}    
    public ParentDemo1(){        
        Debug.Log("parent构造");        
        PrintFields();    
    }    
    public ParentDemo1(int a)    {        Debug.Log("parent构造"+a);    }    
    public virtual void PrintFields()    {        Debug.Log("parent PrintFields");    }
}
```

2. 子类

```c#
public class SonDemo1:ParentDemo1
{
    private int x = 2;
    private int y;
    public SonDemo1()
    {
        Debug.Log("son构造");
    }
    public SonDemo1(int a)
    {
        y = -1;
        Debug.Log("son构造"+a);
    }
    public override void PrintFields()
    {
//        base.PrintFields();
        Debug.Log($"son PrintFields x={x},y={y}");
    }
}
```

3. 执行

   ```c#
   SonDemo1 sonDemo1=new SonDemo1(2);
   ```

4. 结果：父类构造、子类.`PrintFields`（未执行父类该方法，原因:被重写，未调用`base.PrintFileds()`）、

## 游戏运行质量设置问题

1. 分辨率：`Screen.SetResolution()` 修改分辨率，编辑器下不起作用，真机有效。
2. 帧率：`Application.targetFrameRate = ` 修改帧率，需要关闭垂直同步（目前编辑器可行，模拟器未起作用）

## 适配方案(以UGUI为例)

### 理论

1. 宽度适配:宽度适配即水平UI不会变化，只会改变高度。即在适配的过程中，拉伸的是高度，此时需要考虑UI的竖直方向的适配（如设置UI的锚点在竖直方向），防止在调整分辨率的时候显示错误。

2. 高度适配:即竖直UI不会变化，只会拉伸宽度，此时应该重点考虑水平方向的适配。

   *注意:本项目是高度不变，宽度拉伸，即高度适配*

### Screen类

1. `currentResolution` 当前分辨率

2. `Screen.height/width`  屏幕窗口的当前宽度/高度 以像素为单位(安卓环境时和1值始终相等，pc环境时，是Game窗口选择的分辨率)

## U3d特性

1. `InitializeOnLoad` Unity加载时初始化编辑器

* 静态构造函数的运行时机:①u3d启动时；②按下Run时

* 静态构造中绑定要监听的u3d事件，如:`EditorApplication.update`

* `~Constructor` 析构函数中解除绑定

  

##  项目插件

### Spine搭配FGUI

1. Spine的加载
   * ResourceManger加载SkeletonDataAsset
   * SkeletonDataAsset实例化对象SkeletonAnimation
   * 使用GoWrapper(skeleAnim.gameObject)包装spine对象
   * 为挂点GGraph设置包裹对象为gowrapper

*注：1.GoWrapper的gameObject是GoWrapper本身。wrapTarget是其子物体，即spine对象本身*

*注：2.GGraph设置包裹对象以后，graph.displayObject就是GoWrapper,gameObjectName也是GoWrapper*

2. Spine的销毁
   * 