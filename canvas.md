## canvas 笔录

- moveTo 与 lineTo 的区别

  1. `moveTo` 是断点操作，`lineTo` 是连续操作
  2. `beginPath` 清空之前的状态
  3. `beginPath` + `lineTo` = `moveTo`


- beginPath and closePath

  1. 构建封闭的多边形（往往最后一个点不用写，自动处理封闭）

  ​

- difference between fill()  and  stroke()

  1. `fill()` 在 `stroke()`后面使用，会有一个效果，`border`的一半宽度会被填充颜色覆盖


- 刷新canvas 方法

  > ctx.canvas.height = ctx.canvas.height;


- 获取**点击点**距离**目标四周** 的位置 (通用方法)

  > var x = event.clientX - canvas.getBoundingClientRect().left;
  >
  > var y = event.clientY - canvas.getBoundingClinetRect().top;

### 剖析canvas 画布 与 screem 的区别

>很多童鞋一直以为显示所画东西的改屏幕就是Canvas，其实这是一个非常错误的理解
>
>由于屏幕显示与Canvas根本不是一个概念！Canvas是一个很虚幻的概念，相当于一个透明图层（用过PS的同学应该都知道），每次Canvas画图时（即调用Draw系列函数），都会产生一个透明图层，然后在这个图层上画图，画完之后覆盖在屏幕上显示。所以上面的两个结果是由下面几个步骤形成的：
>
>- 调用canvas.drawRect(rect1, paint_green);时，产生一个Canvas透明图层，由于当时还没有对坐标系平移，所以坐标原点是（0，0）；再在系统在Canvas上画好之后，覆盖到屏幕上显示出来，过程如下图：
>
>![](./Snipaste_2018-01-17_19-54-23.png)
>
>- 然后再第二次调用canvas.drawRect(rect1, paint_red);时，又会重新产生一个全新的Canvas画布，但此时画布坐标已经改变了，即向右和向下分别移动了100像素，所以此时的绘图方式为：（合成视图，从上往下看的合成方式）
>
>![](./Snipaste_2018-01-17_19-57-16.png)



> ### 总结
> 1. 每次调用canvas.drawXXXX系列函数来绘图进，都会产生一个全新的Canvas画布。
> 2. 如果在DrawXXX前，调用平移、旋转等函数来对Canvas进行了操作，那么这个操作是不可逆的！每次产生的画布的最新位置都是这些操作后的位置。Save()、Restore()的画布可逆
> 3. 在Canvas与屏幕合成时，超出屏幕范围的图像是不会显示出来的
> 4. 关于rotate, scale，其实就是把`canvas`画布的坐标系旋转 或者 缩放
>
> ```js
> 			ctx.save();
> 			ctx.beginPath();
> 			ctx.fillStyle = 'red';
> 			ctx.rect(0,0,150,100);
> 			ctx.fill();
> 			
> 			/*ctx.translate(100, 100);
> 			ctx.beginPath();
> 			ctx.fillStyle = 'green';
> 			ctx.globalAlpha = 0.7;
> 			ctx.rect(20,20,150,100);
> 			ctx.fill();*/
> 			
> 			ctx.translate(200, 120);
> 			ctx.beginPath();
> 			ctx.fillStyle = '#000080';
> 			ctx.globalAlpha = 0.5;
> 			ctx.rect(0,0,150,100);
> 			ctx.fill();
> 			
> 			ctx.rotate(Math.PI / 2);
> 			ctx.beginPath();
> 			ctx.fillStyle = '#20B2AA';
> 			ctx.globalAlpha = 0.3;
> 			ctx.rect(0,0,150,100);
> 			ctx.fill();
> 			
> 			ctx.save();
> 			ctx.translate(0, 120);
> 			ctx.beginPath();
> 			ctx.fillStyle = '#D58512';
> 			ctx.globalAlpha = 0.2;
> 			ctx.rect(0,0,150,100);
> 			ctx.fill();
> 			ctx.restore();
> 			
> 			ctx.save();
> 			ctx.translate(120, 0);
> 			ctx.beginPath();
> 			ctx.fillStyle = '#D58512';
> 			ctx.globalAlpha = 0.2;
> 			ctx.rect(0,0,150,100);
> 			ctx.fill();
> 			ctx.restore();
> ```
> *图示*
> ![](Snipaste_2018-01-17_20-06-17.png)

### 注意 `canva` `transform` 与 `div transform` 的区别

canvas 一切transform 基于原点， 只是画布改变， 本身canvas dom 节点没有改变，没有实际的origin，都是基于矩形变换的

div 随着transform 而改变自身， 不是基于原点，比如缩放会基于`transform.origin`



## 我们会经常遇到的一些操作

​	面对业务，我们需要基础的图形变换功能，比如说缩放，旋转，拖动，通过上面的图形介绍，我们大概了解transfrom的变换原理，接下来我会详细记录下实际的操作

#### 移动

​	通常我们需要以下几种相关参数

```js
var that = this,
	elem = eventElem,
    moveParams = {
        left: 0, // 移动左边距
        top: 0, // 移动上边距
        currentX: 0, // 当前移动时鼠标的X轴位置
        currentY: 0, // 当前移动时鼠标的Y轴位置
        moveFlag: false // 这是一个标记，通常canvas有许多交叉的事件触发，我们需要一个控制点
    };
```

​	添加以下事件

```js
eventUtil.addEvent(elem, 'contextmenu.translate', function(e) {
    e.preventDefault();
});
eventUtil.addEvent(elem, 'selectstart.translate', function(e) {
    e.preventDefault();
});
eventUtil.addEvent(elem, 'mousedown.translate', function(e) {
    if(e.stopPropagation)
        e.stopPropagation();

    if(e.button === 0) { // 左键
        var p = that.params,
            offsetLeft = p.offsetLeft,
            offsetTop = p.offsetTop;

        moveParams.moveFlag = true;
        moveParams.left = offsetLeft;
        moveParams.top = offsetTop;
        moveParams.currentX = e.offsetX;
        moveParams.currentY = e.offsetY;
    }

}, false);

eventUtil.addEvent(elem, 'mousemove.translate', function(e) {
    // moveEvent Operation
    if(moveParams.moveFlag) {

        elem.style.cursor = 'move';

        var p = that.params;

        var f = e.offsetX,
            d = e.offsetY;
        var h = f - moveParams.currentX,
            g = d - moveParams.currentY;

        p.offsetLeft = parseInt(moveParams.left) + h;
        p.offsetTop = parseInt(moveParams.top) + g;

        if(typeof fb == 'function') {
            var p = that.params;
            event.offsetLeft = p.offsetLeft;
            event.offsetTop = p.offsetTop;
            fb.call(this, event);
        }

        if(e.preventDefault) {
            e.preventDefault()
        }
        return false
    }
}, false);

eventUtil.addEvent(document, 'mouseup.translate', function(e) {
    if(e.button === 0) {
        var p = that.params;

        moveParams.moveFlag = false;
        moveParams.left = p.offsetLeft;
        moveParams.top = p.offsetTop;
    }

    elem.style.cursor = 'default';
}, false);
```

#### 缩放

缩放，我们关注的是鼠标滚动的兼容性

```js
// IE11 / Chrome 两者都有，低版本IE 只有前者， FF只有后者
if(event.srcElement && !event.target) {
    event.target = event.srcElement;
}
if(!event.preventDefault && event.returnValue !== undefined) {
    event.preventDefault = function() {
        event.returnValue = false;
    }
}
// 兼容滚轮事件
if(type === 'DOMMouseScroll' || type === 'mousewheel') {
    event.delta = (event.wheelDelta) ? event.wheelDelta / 120 : -(event.detail || 0) / 3;
    event.wheelDelta = event.wheelDelta ? event.wheelDelta : (event.deltaY * (-40));
}
```

关于怎么操作事件嘛，且看一下代码

```js
var that = this,
    elem = eventElem;
eventUtil.addEvent(elem, "mousewheel.zoom", function(e) {
    e = e.originalEvent
    var delta = (function(event) {
        var type = event.type;
        if(type === 'DOMMouseScroll' || type === 'wheel' || type === 'mousewheel') {
            event.delta = (event.wheelDelta) ? event.wheelDelta / 120 : -(event.detail || 0) / 3;
            event.wheelDelta = event.wheelDelta ? event.wheelDelta : (event.deltaY * (-40));
        }
        return event.delta
    }(e))
	// 缩放操作
    delta < 0 ? _action.bind(that)(1, e.offsetX, e.offsetY) : _action.bind(that)(-1, e.offsetX, e.offsetY); 
	// 下面为回调函数传入参数而已
    var ret;
    if(typeof fb == 'function') {
        var p = that.params;

        e.originalScale = p.originalScale;
        e.offsetLeft = p.offsetLeft;
        e.offsetTop = p.offsetTop;
        ret = fb.call(window, e);
    }
    e.preventDefault();
    return ret;
}, false);

function _action(bool, offsetX, offsetY) {
    var p = that.params,
        os = p.originalScale,
        ol = p.offsetLeft,
        ot = p.offsetTop,
        zr = p.zoomRatio;

    // 触发点到触发源左上角的宽高比例，决定offset的增减比例 这是因为定点缩放的原因
    ol += bool * (zr) * (offsetX - ol);
    ot += bool * (zr) * (offsetY - ot);
    p.offsetLeft = ol;
    p.offsetTop = ot;

    // 增减比例
    os *= 1 + (-zr * bool);
    p.originalScale = os;
};
```

#### 旋转

​	这是我认为比较复杂的操作了，因为原生的canvas没有中心轴这一概念的问题，导致操作旋转会相对男理解一些

首先，我们得控制和获取参数

```js
var that = this,
    elem = eventElem,
    rotateParams = {
        rotationAngle: 0, // 初始得旋转角度
        originX: 0, // 用作判断矢量方向
        originY: 0, // 用作判断矢量方向
        prevX: 0, // 前一个移动点的X轴坐标
        prevY: 0, // 前一个移动点的Y轴坐标
        rotateFlag: false, // 这是一个标记，通常canvas有许多交叉的事件触发，我们需要一个控制点
        direction: 0, // 移动方向判断 1 正向， 0反向
    };
```

事件绑定以及处理

```JS
eventUtil.addEvent(elem, 'contextmenu.rotate', function(e) {
    e.preventDefault();
});
eventUtil.addEvent(elem, 'selectstart.rotate', function(e) {
    e.preventDefault();
});
eventUtil.addEvent(elem, 'mousedown.rotate', function(e) {
    if(e.stopPropagation)
        e.stopPropagation();
    if(e.button === 2) { // 右键
        var p = that.params;
        rotateParams.rotateFlag = true;
        rotateParams.direction = 0;
        rotateParams.rotationAngle = p.rotationAngle;
        rotateParams.originX = rotateParams.prevX = e.offsetX;
        rotateParams.originY = rotateParams.prevY = e.offsetY;
    }
}, false);
eventUtil.addEvent(elem, 'mousemove.rotate', function(e) {
    // sacleEvent Operation
    if(rotateParams.rotateFlag) {
        elem.style.cursor = "e-resize";
        var p = that.params,
            l = e.offsetX,
            n = l - rotateParams.originX, // 矢量【方向 大小】
            o = Math.abs(n) + 1,
            ra = p.rotationAngle,
            rr = p.rotateRatio;
        ra += -n * rr;
        rotateParams.originX = rotateParams.prevX = e.offsetX; // 更新每次移动过的位置
        rotateParams.originY = rotateParams.prevY = e.offsetY;
        p.rotationAngle = ra;
        if(typeof fb == 'function') {
            event.rotationAngle = p.rotationAngle;
            fb.call(this, event);
        }
        return false;
    }
}, false);
eventUtil.addEvent(document, 'mouseup.rotate', function(e) {
    if(e.button === 2) {
        var p = that.params;
        rotateParams.rotateFlag = false;
        rotateParams.direction = 0; // 重置方向
        rotateParams.rotationAngle = p.rotationAngle;
    }
    elem.style.cursor = 'default';
}, false);
```

### 我们关心的重点来了

​	通常，我们会觉得即使获取的相关的变换参数，但是canvas怎么去操作呢，一般的操作会先旋转>移动>缩放，按顺序`先旋转在移动`，否则位置发生错乱改变，`移动>缩放`是固定顺序，要是先缩放再移动， 移动距离变短了

```JS
var that = this,
    canvas = this.canvas,
    ctx = this.ctx,
    p = that.params,
    /**
     * 这里可能存在一些难理解，通常我们操作画布旋转的时候，会相对于container的中心去旋转
     * 但是偶尔我们需要对某个单独的图形进行旋转，所以你要移动到对应图型的中点去
     */
    width = p.imgWidth || canvas.width, 
    height = p.imgHeight || canvas.height,
    ol = p.offsetLeft,
    ot = p.offsetTop,
    os = p.originalScale,
    ra = p.rotationAngle,
    newTransform = matrix.create(),
    centerX = (width / 2) * 1,
    centerY = (height / 2) * 1;

ctx.translate(ol + (width * os / 2), ot + (height * os / 2)); // 先移动到中心 旋转， 在移动回原点
ctx.rotate(ra);
ctx.translate(-(ol + (width * os / 2)), -(ot + (height * os / 2))); // 先移动到中心 旋转， 在移动回原点  这步骤很重要
ctx.translate(ol, ot);
ctx.scale(os, os);
```

​	但是，canvas变换其实就是矩阵变换计算，*重要提示， 矩阵的变换算是到过来的*
​        **CTM = Mn · Mn-1 · … · M2 · M1 · M0**
	注意，后面的变换要乘在前面。矩阵的乘法不具有交换律。这意味着先旋转再平移和先平移再旋转是不一样的结果

```JS
// 中心缩放
matrix.translate(newTransform, newTransform, [-centerX, -centerY]);
matrix.scale(newTransform, newTransform, [os, os]);
matrix.translate(newTransform, newTransform, [centerX, centerY]);

matrix.translate(newTransform, newTransform, [-centerX, -centerY]);
matrix.rotate(newTransform, newTransform, -ra);
matrix.translate(newTransform, newTransform, [centerX, centerY]);

matrix.translate(newTransform, newTransform, [ol, ot]);

p.__transform = newTransform;

CanvasRenderingContext2D.prototype.setTransform.apply(ctx, newTransform);
```

### 还有更重要的东西

​	通常我们需要根据屏幕点击事件获取屏幕坐标点，那怎么才能转换成canvas中对应的坐标点呢，特别是存在缩放，移动，旋转等等

```JS
/**
	 * 计算屏幕坐标 转换为canvas坐标
	 * @param pointList {Object} 变换的坐标数组,example: [[1,2],[3,5]]
	 */
screencoordToCanvascoord = function(pointList) {
    if(!Array.isArray(pointList) || pointList.length < 1) return;
    var that = this
    return pointList.map(function(item) {
        var canvas = that.canvas,
            L = that.params,
            x = item[0] - L.offsetLeft, 
            y = item[1] - L.offsetTop,
            w = L.imgWidth || canvas.width, // 基于图片的中心，并不是画布的中心
            h = L.imgHeight || canvas.height, // 基于图片的中心，并不是画布的中心
            cx = w * L.originalScale / 2,
            cy = h * L.originalScale / 2,
            a = L.rotationAngle,
            p = that.computedRotatedCoord(x, y, cx, cy, -a);

        return [
            p.x0 / L.originalScale,
            p.y0 / L.originalScale
        ]
    })
}
/**
	 * 在平面中，一个点绕任意点旋转θ度后的点的坐标
	 * @param {Object} x
	 * @param {Object} y
	 * @param {Object} rx0
	 * @param {Object} ry0
	 * @param {Object} a
	 */
computedRotatedCoord = function(x, y, rx0, ry0, a) {
    return {
        x0: (x - rx0) * Math.cos(a) - (y - ry0) * Math.sin(a) + rx0,
        y0: (x - rx0) * Math.sin(a) + (y - ry0) * Math.cos(a) + ry0
    }
}

/**
	 * [canvas 画布坐标 转换 屏幕坐标]
	 * @return {[type]} [description]
	 */
canvascoordToScreencoord = function() {
    var L = this.params,
        canvas = this.canvas,
        pointList = L.coordsList,
        width = this.params.imgWidth || this.params.canvasWidth,
        height = this.params.imgHeight || this.params.canvasHeight,
        left = L.offsetLeft,
        top = L.offsetTop,
        scale = L.originalScale,
        rotate = L.rotationAngle,
        cx = width * L.originalScale / 2,
        cy = height * L.originalScale / 2;

    return pointList.map(function(k, i, arr) {
        var x = k[0],
            y = k[1];
        // x = x < 0 ? 0 : x > width ? width : x; // 超出范围判断
        // y = y < 0 ? 0 : y > height ? height : y;
        // 缩放前
        var sx = x * scale,
            sy = y * scale;
        // 旋转前
        var rx = (sx - cx) * Math.cos(rotate) - (sy - cy) * Math.sin(rotate) + cx,
            ry = (sx - cx) * Math.sin(rotate) + (sy - cy) * Math.cos(rotate) + cy;
        // 移动过前
        var lx = rx + left,
            ly = ry + top;
        // 屏幕点坐标
        return [
            lx,
            ly
        ]
    });
}
```

​	