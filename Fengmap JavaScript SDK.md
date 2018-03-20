[TOC]
## Fengmap JavaScript SDK

**账号**：15521228977       **密码**：fengmaphtml



### 地图属性控制

- ##### 坐标转换      **墨卡托坐标系米制坐标**

> ```js
> //屏幕坐标转地图坐标：
> //z取值为对应楼层的高度。可用map.getGroupHeight(groupID)+map.layerLocalHeight获取。
> //若屏幕坐标超出范围，转换结果为null。
> var pt2 = map.coordScreenToMap ( x,  y,  z );
>
> //地图坐标转屏幕坐标：
> var pt1 = map.coordMapToScreen(x, y, z);
> ```

- 地图朝向  -- 您可以自定义地图的初始朝向，如正南、正北或自定义角度等
> ```js
> var map = new fengmap.FMMap({
>     container : document.getElementById("map"), //设置显示地图容器
>     appName:'fmdemo',           //开发者申请应用名称
>     key:'ac9a8a62f93648842203c09b24241425',  //开发者申请应用下web服务的key
>
>     //设置地图默认状态。defaultControlsPose值可为方向枚举、角度值、或特定角度的Json对象的任意一种。
>     //如设置二维模式下正北显示：
>     //方向枚举型。可设置正南、正北、正东、正西、东南、西南等方向值。具体可参考fengmap.ControlsPoseEnum类。
>     defaultControlsPose: fengmap.FMDirection.NORTH,
>     //defaultControlsPose: 90,     //角度值。
>
>     //如果需自定义地图角度，可使用如下JSON对象。
>     //此JSON对象用户可根据在浏览器中手动拖动的地图状态，
>     //然后在调试界面内使用fengmap.MapUtil.getControlsInfo() 获取。
>     // defaultControlsPose: {
>     //   target:"66.370,150.000,-38.260",
>     //   position:"66.370,698.433,-38.261"
>     // }
> });
> ```

- 二三维状态
> ```js
> var map = new fengmap.FMMap({
> container : document.getElementById("map"), //设置显示地图容器
> appName:'fmdemo',           //开发者申请应用名称
> key:'ac9a8a62f93648842203c09b24241425',  //开发者申请应用下web服务的key
>
> defaultViewMode:fengmap.FMViewMode.MODE_2D //初始二维还是三维状态，三维状态为MODE_3D
> }); 
> ```
> ```js
> // after map loaded , define this
> //2D、3D控件配置
> var toolControl = new fengmap.toolControl(map,{
> init2D:false,   //初始化2D模式
> groupsButtonNeeded:false,   //设置为false表示只显示2D,3D切换按钮
> //点击按钮的回调方法,返回type表示按钮类型,value表示对应的功能值
> clickCallBack:function(type,value){
> // console.log(type,value);
> }
> });
> ```


- ##### 点击高亮

> ```js
> var map = new fengmap.FMMap({
> container : document.getElementById("map"), //设置显示地图容器
> appName:'fmdemo',           //开发者申请应用名称
> key:'ac9a8a62f93648842203c09b24241425',  //开发者申请应用下web服务的key
>
> modelSelectedEffect:false          //支持单击模型高亮，false为单击时模型不高亮
> });
>
> map.selectNull(); //清空选中模型方法，把所有高亮的模型都取消高亮
> /**
> * 选中模型方法
> * @method storeSelect
> * @param {fengmap.FMModel} model 模型对象，可根据ID，Type等条件查询出模型对象
> * @param {boolean} isSelected  是否选中
> * @param {boolean} multiSelect 是否保持上一次选中模型的状态。false是不保存，即地图中只有当
> 前设置的模型高亮，其他模型不高亮。ture则不清空上次的选中状态，都高亮
> */
> map.storeSelect(model,isSelected, multiSelect);
> ```

- ##### 动画开关

> ```js
> var map = new fengmap.FMMap({
> container : document.getElementById("map"), //设置显示地图容器
> appName:'fmdemo',           //开发者申请应用名称
> key:'ac9a8a62f93648842203c09b24241425',  //开发者申请应用下web服务的key
>
> focusAnimateMode: false,            //开启聚焦层切换的动画显示
> viewModeAnimateMode:false,          //开启2维，3维切换的动画显示
> moveToAnimateMode:false,           //地图定位跳转动画设置
> naviLineAnimation:false            //路径线流动动画开关。false关闭
> });
> ```

- 设置中心点

> ```js
> //中心点
> var pnt = {
> x:11100000.000,
> y:481929.000,
> groupID:1,      //目标层GroupID
> callback:function(){
> //跳转中心点完成
> }
> };
> //跳转
> map.moveTo(pnt);
> ```

- 手势控制

> ```js
> //地图手势对象
> var controller = map.gestureEnableController;
> controller.enableMapPan = true; // 可移动
> controller.enableMapRotate = true; // 可旋转
> controller.enableMapIncline = true; //可倾斜
> controller.enableMapPinch = true;   //可缩放
> controller.enableMapSingleTap = false; //不可单击
> ```

- 坐标转换

> ````js
> //屏幕坐标转地图坐标：
> //z取值为对应楼层的高度。可用map.getGroupHeight(groupID)+map.layerLocalHeight获取。
> //若屏幕坐标超出范围，转换结果为null。
> var pt2 = map.coordScreenToMap ( x,  y,  z );
> //地图坐标转屏幕坐标：
> var pt1 = map.coordMapToScreen(x, y, z);
> ````

- ##### 气泡信息框

> ```js
> //信息框控件大小配置
> var ctlOpt = new fengmap.controlOptions({
> mapCoord: {
> //设置弹框的x轴
> x: event.target.x,
> //设置弹框的y轴
> y: event.target.y,
> //设置弹框位于的楼层
> groupID: 1 
> },
> //设置弹框的宽度
> width: 200,
> //设置弹框的高度
> height: 100,
> marginTop: 10,
> //设置弹框的内容
> content: '这是一个信息框' 
> });
>
> //添加弹框到地图上
> var popMarker = new fengmap.FMPopInfoWindow(map, ctlOpt);
>
> // 关闭气泡信息框。
> popMarker.close();
> ```
>

### 地图事件

- 点击事件
> ```js
> //点击地图事件
> map.on('mapClickNode', function(event) {
> console.log('mapClickNode', event);
> switch(event.nodeType) {
> case fengmap.FMNodeType.MODEL:
> //返回模型信息
> break;
> case fengmap.FMNodeType.TEXT_MARKER:
> //返回用户自定义标注信息
> break;
> case fengmap.FMNodeType.FACILITY:
> //返回公共设施层标注信息
> break;
> case fengmap.FMNodeType.IMAGE_MARKER:
> //返回用户自定义图片标注
> break;
> case fengmap.FMNodeType.LOCATION_MARKER:
> //返回定位标注
> break;
> }
> });
> ```

- ##### 地图状态更新事件

> ```js
> var callback = function() {
> //刷新地图后操作
> };
> // 监听地图状态更新事件
> map.on('update', callback);
>
> // 移除监听地图状态更新事件
> map.off('update', callback);
> ```

  ### 点标注

  ​Fengmap JavaScript SDK 提供根据地图坐标点添加标注的功能。点标注可分为文字标注（TextMarker）、图片标注（ImageMarker）、定位标注（LocationMarker）。

- 添加文字标注前需要先获取指定楼层。

> ```js
> map.getFMGroup(groupID);
> ```

- 获取markers的详细信息

> ```js
> // 获取 markers 的详细信息
> map.getFMGroup(map.focusGroupID).getOrCreateLayer('imageMarker').markers
> map.getFMGroup(map.focusGroupID).getOrCreateLayer('textMarker').markers
> ```
>

- ​