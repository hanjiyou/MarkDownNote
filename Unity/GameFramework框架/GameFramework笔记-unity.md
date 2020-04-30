# 1. GameFramework笔记-unity

## 1.1. EventSystem

### 1.1.1. UGUI的EventSystem中
 有很多事件接口，其中最常用的是IPointerClickHandler, IPointerDownHandler,IPointerUpHandler，其中Up重写的方法必须同时又Click时，才会执行且先执行；Down重写的方法可自己直接执行
### 1.1.2. IsPointerOverGameObject() 防止点击穿透
判断点击是否在UI对象上，是则返回true,`if(EventSystem.current.IsPointerOverGameObject()){return;} ` 
- **pc端**:无参时判断对象为鼠标左键(pointerId=-1)
- **安卓端(该方法无效，建议使用1.2中的方法)**:有参时判断对象为手指Touch点击 参数应该为:Input.GetTouch(0).fingerId

## 1.2. 发射射线的3种方法
### 1.2.1. 与3D模型交互（模型需要碰撞器）
```
Ray r=camera.ScreenPointToRay(Input.mousePosition)
RaycastHit rh,
if(Physics.Raycast(ray, out raycastHit)){
    rh.transform
}
```
### 1.2.2. UI交互-PointerEventData与EventSystem
```
PointerEventData ped=new PointerEventData(EventSystem.current);
ped.position=Input.mousePosition;
List<RaycastResult> results=new List<RaycastResult>();
EventSystem.current.RaycastAll(ped, results);
```
*注意：点击的UI对象必须得开启Raycast Target，否则RaycastResult检测不到*
### 1.2.3. UI交互-PointerEventData与GraphicRaycaster
```
PointerEventData ped=new PointerEventData(EventSystem.current);
ped.position=Input.mousePosition;
List<RaycastResult> results=new List<RaycastResult>();
GraphicRaycaster raycaster=XXX;
raycaster.Raycast(ped,results);
```
*注意：点击的UI对象必须得开启Raycast Target，否则RaycastResult检测不到*
## 1.3. Editor总结
1. 普通的Editor直接使用[MenuItem]特性指定静态方法，会在菜单栏生成自定义的选项
2. 控制Inspector监视面板的Editor:
* 步骤:
     * Editor类引用特性[CustomEditor(typeof(TargetClass))]
     * OnEnable() 目标类显示在监视面板上会马上执行一次
     * OnInspectorGUI() 监视面板刚显示或者发生交互时会调用
     * SerializedProperty 重新定义所有想要显示在监视面板的MB中的属性
* 各种单独的组件(EditorGUILayout):
     * 浮点型的滑动条:EditorGUILayout.Slider(sp,leftNum,rightNum)
     * 加黑的标签: EditogGUILayout.LabelField(labelText,EditorStyles.boldLabel)
     * 带标签的输入框: EditorGUILayout.TextField(label,inputStringField)
     * 下拉框 : newIndex=EditorGUILayout.Popup("列表标签",selectIndex,string[])
     * 复选框:  newSelectStatus=Toggle/ToggleLeft(label,isSelected)
     * 分割: EditorGUILayout.Separator()
     * 根据序列化属性来自动生成对应不同类型的组件:EditorGUILayout.PropertyField(sp)
* 布局组件:
     * 开启一个矩形垂直的布局组件:
 EditorGUILayout.BeginVertical("box"){布局组件的具体内容} EGL.EndVertical()
     * 根据输入条件控制是否能操作:如果正在运行 组件就不可操作
 EditorGUI.BeginDisabledGroup(EditorApplication.isPalying){} EG.End();
     * 设置滑动区域 EGL.BeginScrollView(){}  end
* 序列化字段SP的使用
     * 在MB中对应字段如果是私有的，需要加上[SerializeField]特性
     * 序列化属性在Editor中通过serializedObject.FindProperty()对象获取
     * 如果是数组，则可以:sOP.arrySize获取长度、ClearArray()清空数组、InsertArrayElementAtIndex(i)插入格子、GetArrayElementAtIndex(i).stringValue为格子的string赋值

## 1.4. 脚本编译顺序
1. 我们自己定义的普通cs文件会被打入Assembly-CSharp.dll中,放在Plugins下的cs文件会被打入Assembly-CSharp-firstpass.dll中
3. unity编译顺序如下①Plugins ②Plugins/Editor ③其他不在Plugins ④其他Editor

## 1.5. www与UnityWebRequest(两者都是用于http请求)
如果都是用协程等待下面两种方法的下载
1. WWW属于老方法，没有timeout字段，无法设置超时时间，协程会一直等待
2. UnityWebRequest([文档](https://blog.csdn.net/qwe25878/article/details/85051911)):  
* unity5.4版本出的用于替换WWW，可设置timeout，如果超时会返回，且IsNetworkError为true
* 判断错误不应用webRequest.erro!=null,这是错误的；应该用.isHttpError||.isNetworkError来判断是否出错
* SendWebRequest()开始与远程服务器通信，返回WebRequestAsyncOperation对象，协程内产生该对象将导致协程暂停，直到UnityWebRequest遇到错误或完成通信。
* Abort() 尽快结束联网过程，可以随时调用此方法,UnityWebRequests被认为遇到了系统错误。 isNetworkError或isHttpError属性将返回true，error属性将为“User Aborted”
* EscapeURL()些文本字符在URL中存在时具有特殊含义。 如果需要在URL参数中包含这些字符，则必须使用转义序列表示它们。

## 1.6. 宏定义
UNITY_ANDROID宏定义运行情况包含两种①android端 ②unity切换到android平台

## 1.7. AssetBundle加载([文档](https://www.cnblogs.com/MeowChocola/p/9022839.html))

# Unity相关
## 默认编码格式
1. 在.net 3.x中 `Encoding.Default=Encoding.GetEncoding("GB2312")`
2. 在.net 4.x中 `Encoding.Default= Encoding.GetEncoding("UTF-8")`