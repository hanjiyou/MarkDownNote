# FGUISummary

## FGUI自定义组件绑定组件类

1. 将FGUI组件和类进行绑定

   * ```c#
     UIObjectFactory.SetPackageItemExtension("ui://ZodiacPlayMenuWindow/StarAnimationGroup", typeof(ChapterItemComponent));
     ```

*注意:指定FGUI组件路径时，格式为："ui://包名//组件名"即可，不需要包含文件夹，且该组件必须设置为导出。*

## FGUI基础属性

1. 坐标
   * `xy` 二维坐标系
   * `position` 三维坐标系

2. 物体名称
   * `gameObjectName` 是当前对象的组件名(即xml名)，在FGUI显示列表里即为{}中的名字
   * `name` 当前对象所在组件中，手动指定的命名
   * unity层级面板显示名称的都是gameObjectName(即如果是组件就是xml名；如果是基础物体入loader/image则直接是Gloader/Image)
3.  父子物体
   * 设置父/子物体:`parent.AddChild(son)`
   * 获取子物体的数量:`numChildren` 列表也可用
   * 指定子物体数量:`numItem` 仅列表可用，直接指定Item的数量

4. FGUI坐标与u3d监视面板坐标的关系
   * u3d显示坐标的原点是屏幕左上角（无论FGUI锚点在哪）
   * FGUI锚点在左上角时，X检=x,Y检=-y
   * FGUI锚点在中心时，X检=-((2W/x)-x),Y检=(H/2  -y)
   * FGUI锚点在右下时，X检=-(W-x),Y检=H-y

## 启动流程(场景只存在StageCamera时)

1. StageCamera.OnScreenSizeChanged()函数中调用 `Stage.inst.HandleScreenSizeChanged()`
   * `Stage.inst` 则调用了 `Stage.Instantiate` 实例化方法,用于创建 `Stage`容器、`GRoot`组件(对象名称为"GRoot"),并添加GRoot的游戏对象作为 `Stage`游戏对象的子物体

2. `Stage`构造中，会创建 `StageEngine`(继承了MN)对象，并且命名为"Stage")

## 常用功能介绍

### 滚动容器

1. 对组件或者列表**溢出处理**设置为"XX滚动"后，组件或者列表即成为了滚动容器。点击齿轮按钮即可设置详细的滚动相关属性。如果溢出处理设置不符合，则obj.scrollPane为空。
2. 滚动相关属性
   * "滚动条"：为滚动容器设置水平/竖直滚动条
   * "刷新组件"：指定下拉/上拉刷新时需要显示的组件
   * "滚动位置自动贴近元件"：滚动结束后，保证位置刚好处于任意元件的上/左边缘
   * "页面模式":以适口大小为页面大小，每次滚动距离为一页
3. `ScrollPane`
   * `view/contentWidth` 视口/内容宽度
   * `ScrollLeft/Right/Up/Down` 向指定方向滚动N*`scrollStep`个像素
   * `ScrollToView` 调整位置，使指定元件出现在视口内
   * `onScroll` 滚动监听
4. `GList`
   * 虚拟列表(有限个item对应无限个数据对象)
     * `itemRenderer` 渲染item的函数，会频繁调用，**不应该有new等产生GC的操作**。如果在此处对按钮等进行事件监听，**绝不可以使用临时函数或lambda表达式，因为会造成添加多次回调**，直接①EventCallback0 callback=OnBtnClick()  ②btn.onClick.Add/Set(callback) *注意:直接穿方法名会产生几十B的GC。*
     * Item索引与显示对象索引的区别
       * item的数量可以通过`numItems`获得，显示对象的数量通过 `numChildren`获得
       * `selecedIndex`获得的是item的索引，`ScrollToView(index)`、`AddSelection`同样是item索引
       * 前者是数据的下标（当前显示对象显示的数据下标）；后者是显示对象下标
       * 将item索引转换成ui对象索引：`ItemIndexToChildIndex`；将ui对象索引转换成item索引：`ChildIndexToItemIndex`
     * 只能通过`numitems`来设置或清空列表，不允许使用 `AddChild/RemoveChild`
     * 可以高度不同

## 常见问题

1. 在sgame项目的window中，为了优化DrawCall，一般的窗口都默认开启fgui封装的深度调整 `fairyBatching`，但是带来的后遗症是：造成物体层级错乱。

   * 原因：开启了 `fairyBatching`的组件，当开发者自己调用 `SetPosition`等API改变子元件位置、大小、缩放时，不会自动触发深度调整，如果使用 `Tween`将它从原来的位置移到另一个位置，这个图片就有可能被其他物体遮挡。

   * 解决办法：这就需要开发者**手动触发深度调整**：`aObject.InvalidateBatchingState();`。对动效强制每帧执行深度调整：`aTransition.invalidateBatchingEveryFrame = true;`

   * *注意：不要在GRoot上开启 `fairyBatching`*


2. 按钮即使**透明度为0**，也是可以**响应点击事件**的