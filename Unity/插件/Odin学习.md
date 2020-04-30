# Essentials 基础特性

1. `Assets Only Attribute` 指定GameObject对象可赋值的范围

* `AssetOnly` 仅资产特性，点击打开选择弹窗时，只有Project中的资源文件，不会出现Hierachy（场景）的资源

* `SceneObjectOnly` 仅场景对象特性，点击打开选择窗口时，只显示Hierachy中的场景对象，不会出现Project中的资源

  *注意：不加任何特性的公有GameObject类型的变量，可以拖拽Asset+SceneObject*

2. `CustomValueDrawer` 自定义监视面板属性绘制方法的特性（必须是公有的字段，可以是数组）

   * `[CustomValueDrawer(MethodName)]` :参数即指定字段绘制方法名，且格式为`MethodName(float,GUIContent label)`

   *注意:指定数组字段时，调用EditorGUILayout绘制时，label对象为空*

3. ` Delayed Property Attribute ` 延迟序列化操作

   * `[OnValueChanged(CallBackMethodName)]`   当值改变时，调用指定的回调方法
   * `[ShowInInSpector]` 可以修饰字段或属性，私有变量使用该特性后，也会显示在监视面板上

4. `EnableGUIAttribute` 激活指定成员

   * 如果是灰色不可操作的（如引用ShowInInSpector特性的只有Get的属性），引用该特性后，会被激活，但是修改操作无效。

5. `GUIColorAttribute` GUI颜色特性，可以直接用来修饰任何公有成员

   * `[ButtonGroup(groupName)]` 修饰函数成员的按钮组特性， 参数为按钮组名，同一组名在同一行显示，点击按钮后执行方法。
   * `[GUIColor(1, 0.6f, 0.4f)]` 用法1，直接指定成员颜色
   * `[Button(name,ButtonSize)]`  用于指定给方法的按钮，点击该按钮，会执行引用该特性的方法。**可以传参，需要在监视面板中指定，但是该参数并没有被序列化**

6. `[PropertyOrder(index)]` 对属性进行排序 index数值小的在上面

7. `[PropertySpace(spaceNum)]` 字段或属性之间留空白

8. `[TypeFilter(methodName)]` 类型过滤，一般用于为序列化的多态类过滤可显示的实例

9. `[ValidateInput(methodName),InfoMessageType=null]` 监视字段是否满足指定的函数，不满足则显示不同类型的警告盒，切换变为显示状态时会震动一下

   * `[InfoBox(content),InfoMessageType]` 信息盒子：图标+content
   * `methodName`格式为:bool methodName(GameObject,ref str,ref Info)
   
10. `[ValueDropdown]` 下拉框

    * `[ValueDropdown(memberName)]` memberName：指定当前字段对应的下拉框数据集合。两种形式,一种是普通数组，一种是key-value
    * `SortDropdownItems` 属性，是否根据key正序排序，**true为正序排序**，false默认不排序
    * `FlattenTreeView` 属性，**默认false显示树形结构，设为true关闭树形结构显示平铺**
    * `ExpandAllMenuItems` 下拉条条目是否全部展开
    * `IsUniqueList ` **显示勾选框**，可以一次性勾选多个item并添加

11. `[AssetList]` 资产列表，可以过滤显示不同的类型的资源，也可以自定义过滤条件

12. `[AssetSelector]` 资源选择器，在对象字段旁边添加一个小按钮，该按钮将向用户显示资产下拉列表(或者弹框)，以便从属性中进行选择。

13. `[ChildGameObjectsOnly]`  显示**子物体对象下拉列表**

14. `[ColorPaltte]`   显示调色板

15. `[DisplayAsString]` 将属性/字段以字符串的形式展示

16. `[FilePath/FolderPath]` 只用于字符串类型的字段，在Inspector面板对应的属性值旁绘制一个文件夹/文件按钮，便于快读定位文件，并提供文件路径的接口。

    * 默认输入框内显示相对于Unity的路径
    * `ParentFolder` 自定义父路径
    * `Extensions="cs,lua"` 指定单个/多个扩展名筛选文件
    * `AbsolutePath` 显示绝对路径

17. `[ShowInInlineEditors/HideInInlineEditors]` 前者在脚本监视面板显示；后者在资源监视面板显示

18. `[TableList]` 以表格的形式展示List对应类中的成员

    * `[HideInTables]`  指定隐藏的成员

19. `[HideReferenceObjectPicker]` 隐藏声明时直接**实例化类的多态选择器对象**

20. `[PreviewField]` 绘制一个方形ObjectField，可以选择预览对象

21. `[Button]` 按钮特性。对应方法的参数，不被序列化

    * `ButtonSizes` 指定枚举类型值，设置按钮大小。
    * `ButtonStyle` 指定不同的按钮风格。 `FoldoutButton` 折叠按钮，将对应的方法的参数作为被折叠item；`Box`折叠参数+按钮；`CompactBox`右上角Invoke。方法最后一个参数被out和ref修饰，代表按钮点击后的结果

22. `[EnumPaging]` 绘制枚举下拉框，**循环访问**枚举属性的可用值

23. `[EnumToggleButtons]` 将枚举类型显示为Toggle按钮。

    * 普通枚举：引用该特性后，显示为单选Toggle
    * Flag枚举：如果是多个左移项的或运算，则会选中多个Toggle

24. `[InlineButton(methodName)]`  将按钮添加到属性末尾

# Groups 组特性

1. `[BoxGroup(groupName)]` 盒子组，同一组名的成员，属于同一盒子。盒子名称包含'/'表示分层
2. `[TitleGroup(name1,name2),TitleAlignments]` 标题组。参数(主标题，副标题，对齐方式)。
   * `horizontalLine: true` 是否显示横线
   * `indent:true` 是否有缩进
3. `[FoldoutGroup(groupName)]`  添加折叠组，多个成员可指定同一个折叠组
4. `[HorizontalGroup(groupName)]` 制作一个水平分组。进行折页(名称含有'/')的情况下`FoldoutGroup`>`HorizontalGroup`>`BoxGroup`
5. 

