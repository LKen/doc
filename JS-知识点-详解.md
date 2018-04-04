## JS 基础篇

## typeof和instanceof的区别

- `typeof` 

  - `typeof`会返回一个变量的基本类型，只有以下几种：`number`,`boolean`,`string`,`object`,[undefined](http://www.111cn.net/tags.php/undefined/),`function`

  - 如果我们想要判断一个变量是否存在，可以使用typeof：(不能使用if(a) 若a未声明，则报错)

    > ```js
    > alert(typeof(1));//number
    > alert(typeof("abc"));//string
    > alert(typeof(true));//boolean
    > alert(typeof(m));//undefined
    >
    > if(typeof a != 'undefined'){
    >     //变量存在
    > }
    > ```

- `instanceof`

  - `instanceof`   返回的是一个布尔值

    > ```js
    > var a = {};
    > alert(a instanceof Object);  //true
    > var b = [];
    > alert(b instanceof Array);  //true
    > ```

  - 需要注意的是，instanceof只能用来判断对象和函数，不能用来判断字符串和数字等

    > ```js
    > var b = '123';
    > alert(b instanceof String);  //false
    > alert(typeof b);  //string
    > var c = new String("123");
    > alert(c instanceof String);  //true
    > alert(typeof c);  //object
    > ```

  ​

  **ie下，img.src = "" , 初始加载页面的时候，会默认进入 error函数** 




## JS `removeEventListener `与 JQ  `off` 的区别，以及事件解绑的一些问题

- js `removeEventListener` 解除方法需要的第二个参数需要传入`fn`（必须），以及第三个参数`capture`（可选），第二个参数不传入是无法完成删除操作的。

> *Fn* ：这个参数很有意思，按照文档上的方法使用，这个方法须是一个**显式声明**的方法

```js
var btn = document.getElementById("myBtn");
var handler = function () {
        alert(this.id);
    };
btn.addEventListener("click", handler, false);
btn.removeEventListener("click", handler, false);  //有效！
```

> 若是**匿名函数**的话，就算你删除一万次都删除不了

```js
var btn = document.getElementById("myBtn");
btn.addEventListener("click", function () {
    alert(this.id);
}, false);
btn.removeEventListener("click", function () {  //无效！
    alert(this.id);
}, false);
```

> 有时候，你以为传入**显式声明**的方法就可以，其实不然，比如

```js
var a = function(n) {
 console.log(n)
};
var b = function(n) {
	console.log(n.type)
};
document.addEventListener('click', b.bind(a));
// VM7908:5 click 点击时候触发
document.removeEventListener('click', b)

// VM7908:5 click  仍然触发
```

> 这问题让人烦恼啊，`js removeEventListener `必须传入以前一模一样的方法，当函数使用 `bind`  动态绑定环境变量时， 方法无法再像以前那样解除绑定；
>
> 但是你以为传入绑定环境变量的方式，就可以了吗？比如这样

```js
var a = function(n) {
 console.log(n)
};
var b = function(n) {
	console.log(n.type)
};
document.addEventListener('click', b.bind(a));
document.removeEventListener('click', b.bind(a));

// VM8104:5 click  仍然触发点击
```

> 问题到这里好像就找不到问题的来源了
>
> 但是我们平常一般并不是这样来绑定方法，常见的是像上面所说，绑定匿名方法，那么怎么解决匿名方法无法解除的问题呢，提供一个解绑匿名函数的方法

```js
var dom=document.getElementById("content"),clickt=0;
dom.addEventListener("click",function(e){
	clickt++;
	alert('你摸了我'+clickt+'下了。最多摸2下哦');
	if(clickt>=2){
      	dom.removeEventListener(e.type,arguments.callee,false);
      	console.log(this);
 	 }
});
```

> 但是问题来了，这东东，必须先触发方法才能生效，而且解绑方法并不是这样，这种方法只是代表着方法的one()
>
> 那么真正解绑匿名函数的写法是？

```js
var a=[]; 
document.addEventListener('click',(function(){
	var b = function(e) {
		console.log(e)
	};
	a.push(b)
	return b;
}()))
document.removeEventListener('click', a.pop()); //这只是方法的原理
```

> 原理是：传入的方法并不是直接在回调里执行，须经过一个*处理层* ，把经过的`fn` 缓存到本地数据里面去，当需要`remove`的时候，再针对性的拿出来, 附上一个自己写的方法

```js
c._unbind('mousewheel'); // 第一层
  _unbind: function(eventType) {
    // if(typeof eventType == 'undefined') return;
    var _elem = this.canvas;

    eventUtil.removeEvent(_elem, eventType);
    eventUtil.removeEvent(document);
  }
exports.removeEvent = (function(window, undefined) {
		if(window.removeEventListener) {
			return function(el, type, fn, capture) {
				if(type === 'mousewheel' && document.mozFullScreen !== undefined) 
					type = 'DOMMouseScroll';
				var uuid = el[uuid],
					events = EVENTLIST[uuid];
				for(var i in events) {
					var list = events[i],
						type = i;
					list.forEach(function(key) {
						var fn = key.fn,
							c = key.capture;
						el.removeEventListener(type, fn, c);
					})	
				}
			}
		} else if(window.detachEvent) {
			return function(el, type, fn, capture) {
				el.detachEvent('on' + type, fn);
			}
		} else {
			return function(el, type, fn, capture) {
				el['on' + type] = null;
			}
		}
})(window);

var EVENTLIST = [];

// 添加方法
exports.addEvent = (function(window, undefined) {
		var _eventCompat = function(event) {
			var type = event.type;
			// IE11 / Chrome 两者都有，低版本IE 只有前者， FF只有后者
			if(event.srcElement && !event.target) 
				event.target = event.srcElement;
			if(!event.preventDefault && event.returnValue !== undefined) 
				event.preventDefault = function() {
					event.returnValue = false;
				}
			// 兼容滚轮事件
			if(type === 'DOMMouseScroll' || type === 'mousewheel') {
				event.delta = (event.wheelDelta) ? event.wheelDelta / 120 : -(event.detail || 0) / 3;
				event.wheelDelta = event.wheelDelta ? event.wheelDelta : (event.deltaY * (-40));
			}
			/**
			 * 其他一些兼容
			 */
			return event;
		};
		if(window.addEventListener) {
			return function(el, type, fn, capture) {
				if(type === 'mousewheel' && document.mozFullScreen !== undefined) 
					type = 'DOMMouseScroll';
				el.addEventListener(type, (function() {
					var uuid = el[uuid],
						_fn = function(event) {
							fn.call(this, _eventCompat(event));
						};
					if(!uuid)
						el[uuid] = uuid = 'eventUtil-' + 　$.uuid();
					if(!EVENTLIST[uuid])
						EVENTLIST[uuid] = {}
					if(!EVENTLIST[uuid][type])
						EVENTLIST[uuid][type] = [].concat({
							fn: _fn,
							capture: capture || false
						})
					else
						EVENTLIST[uuid][type] = EVENTLIST[uuid][type].concat({
							fn: _fn,
							capture: capture || false
						})
					return _fn;
				}()), capture || false);
			}
		} else if(window.attachEvent) {
			return function(el, type, fn, capture) {
				el.attachEvent('on' + type, function(event) {
					event = event || window.event; // low IE versions only window.event
					fn.call(this, _eventCompat(event));
				})
			}
		}
		return function() {};
	})(window);
```



- jq `off()` 

>  JQ  解除的方法 `off` 只需要传入事件名称即可，比如`click` , `dblclick` ...
>
>  参照JQ 源码，写出自己的浅薄的理解

```js
(function(doc, undefined) {
		function now() {
			return +new Date;
		};
		var win = this,
			expando = "gosunyun" + now(),
			uuid = 0,
			cache = {};
		
		win.getCache = function(){
			return cache;
		}
		
		win.data = function(elem, name, data) {
			var id = elem[expando];
			if(!id)
				id = elem[expando] = ++uuid;
			if(name && !cache[id])
				cache[id] = {};
			if(data !== undefined)
				cache[id][name] = data;
			return name ?
				cache[id][name] :
				id;
		};

		win.removeData = function(elem, name) {
			var id = elem[expando];
			if(name) {
				if(cache[id]) {
					delete cache[id][name];
					name = "";
					for(name in cache[id])
						break;
					if(!name)
						removeData(elem);
				}
			} else {
				try {
					delete elem[expando];
				} catch(e) {
					if(elem.removeAttribute)
						elem.removeAttribute(expando);
				}
				delete cache[id];
			}
		};

		win.each = function(object, callback, args) {
			var name, i = 0,
				length = object.length;
			if(args) {
				if(length === undefined) {
					for(name in object)
						if(callback.apply(object[name], args) === false)
							break;
				} else
					for(; i < length;)
						if(callback.apply(object[i++], args) === false)
							break;
			} else {
				if(length === undefined) {
					for(name in object)
						if(callback.call(object[name], name, object[name]) === false)
							break;
				} else
					for(var value = object[0]; i < length && callback.call(value, i, value) !== false; value = object[++i]) {}
			}
			return object;
		};

		win.gevent = {
			guid: 1,
			/**
			 * handler 是用户自定义的事件
			 * handle  各种行为触发后会执行的函数
			 * handlers 缓存事件数据中的对应事件列表
			 * 
			 * @param {Object} elem
			 * @param {Object} types
			 * @param {Object} handler
			 */
			add: function(elem, types, handler) {
				if(elem.nodeType == 3 || elem.nodeType == 8)
					return;
				if(elem.setInterval && elem != window)
					elem = window;
				//给函数一个唯一标识的索引  方便后面删除该事件
				if(!handler.guid)
					handler.guid = this.guid++;
				//获得该元素的events handle 下的数据
				var events = data(elem, "events") || data(elem, "events", {}),
					handle = data(elem, "handle") || data(elem, "handle", function() {
						//gevent.handle才是各种行为触发后会执行的函数
						gevent.handle.apply(arguments.callee.elem, arguments);
					});
					
				handle.elem = elem;
				//遍历事件名 因为可以是 click mouseover
				each(types.split(/\s+/), function(index, type) {
					// 事件名字有可能是 click.d
					var namespaces = type.split(".");
					//获得事件名
					type = namespaces.shift();
					//去掉点后面的东西 是个特殊的命名  在删除的时候可以指定删除他  如 click.d
					//用事件的type 记录住这个特殊的命名
					handler.type = namespaces.slice().sort().join(".");
					//获得该事件是否已经存在events 这个对象里面了
					var handlers = events[type];
					//如果不存在该事件 给元素绑定该事件
					if(!handlers) {
						handlers = events[type] = {};
						if(elem.addEventListener)
							elem.addEventListener(type, handle, false);
						else if(elem.attachEvent)
							elem.attachEvent("on" + type, handle);
					}
					//吧函数放到元素的该事件的列表里面
					handlers[handler.guid] = handler;
				});
				elem = null;
			},
			remove: function(elem, types, handler) {
				if(elem.nodeType == 3 || elem.nodeType == 8)
					return;
				//获取这个元素的所有行为列表  如 {click:{},mouseocer:{}}
				var events = data(elem, "events"),
					ret, index;
				if(events) {
					//如果没出入行为类型 则删除这个元素的所有事件
					//如果传入的是.xx这种形式的 把所有行为的包含.xx命名的全部干掉
					if(types === undefined || (typeof types === "string" && types.charAt(0) == ".")) {
						for(var type in events)
							this.remove(elem, type + (types || ""));
					} else {
						// 处理  {types: xx, handler: xxx} 这种情况
						if(types.type) {
							handler = types.handler;
							types = types.type;
						}
						//因为删除事件可以一次支持删除多个 如click mouseover  所有要遍历删除
						each(types.split(/\s+/), function(index, type) {
							var namespaces = type.split(".");
							type = namespaces.shift();
							var namespace = RegExp("(^|\\.)" + namespaces.slice().sort().join(".*\\.") + "(\\.|$)");
							if(events[type]) {
								//如果传了第3个参数 函数  则删除这个事件
								if(handler)
									delete events[type][handler.guid];
								else {
									//遍历中个这个的所有行为
									for(var handle in events[type]) {
										// Handle the removal of namespaced events
										// 删除有特殊命名的函数
										// 如果没有特殊命名 正则 则是 /(^|\.)(\.|$)/ 可以匹配空 所以也能删除掉没有特殊命名的函数
										// 个人觉得这句话，貌似没用，反正都要删除，有与没有 有差别吗
										if(namespace.test(events[type][handle].type))
											delete events[type][handle];
									}
								}
							}
							
							// 暂时无法理解 这个ret有什么用处
							for(ret in events[type]) break;
							//如果events[type]变成空的了 也就是{} 删除这个元素的的绑定事件
							if(!ret) {
								if(elem.removeEventListener)
									elem.removeEventListener(type, data(elem, "handle"), false);
								else if(elem.detachEvent)
									elem.detachEvent("on" + type, data(elem, "handle"));
								ret = null;
								delete events[type];
							}
						});
					}
					for(ret in events) break;
					//如果发现元素的整个events都是空的了
					//清空掉handle 并且清空掉他所有的引用
					if(!ret) {
						var handle = data(elem, "handle");
						if(handle) handle.elem = null;
						removeData(elem, "events");
						removeData(elem, "handle");
					}
				}
			},

			handle: function(event) {
				var all, handlers;
				//包装event
				event = arguments[0] = gevent.fix(event || window.event);
				event.currentTarget = this;
				//这里的........
				var namespaces = event.type.split(".");
				event.type = namespaces.shift();
				all = !namespaces.length;
				var namespace = RegExp("(^|\\.)" + namespaces.slice().sort().join(".*\\.") + "(\\.|$)");
				//取这个元素的该行为 的 事件列表
				handlers = (data(this, "events") || {})[event.type];
				//遍历这个事件列表 执行该执行的东西
				for(var j in handlers) {
					var handler = handlers[j];
					if(all || namespace.test(handler.type)) {
						// Pass in a reference to the handler function itself
						// So that we can later remove it
						// jq上的注释是是这么写的 把event的handler 引用这个事件 方便之后移除
						// 但是在remove里面 并没有用到event的handler  不知道这里到底有什么用  且有多个事件的时候这个事件被取代
						event.handler = handler;
						//执行事件 并且是用元素调用的事件 可以吧事件里面的this执行元素 ret为函数的返回值
						var ret = handler.apply(this, arguments);
						//如果有返回值  且返回值是false 执行阻止事件冒泡 阻止执行事件默认行为
						if(ret !== undefined) {
							event.result = ret;
							if(ret === false) {
								event.preventDefault();
								event.stopPropagation();
							}
						}
					}
				}
			},
			props: "altKey attrChange attrName bubbles button cancelable charCode clientX clientY ctrlKey currentTarget data detail eventPhase fromElement handler keyCode metaKey newValue offsetX offsetY originalTarget pageX pageY prevValue relatedNode relatedTarget screenX screenY shiftKey srcElement target toElement view wheelDelta which".split(" "),
			fix: function(event) {
				//new setEvent会给event给以个expando属性 如果有中个属性 说明已经生成了event了 不需要在次对event进行包装
				if(event[expando])
					return event;
				//保留一个原始的event
				// new一个新的event 这个与原始的event是不同的
				var originalEvent = event;
				event = new setEvent(originalEvent);
				
				/**
				 * 主要是对做一个原生event的一个copy  然后把不兼容的方法  都合成兼容的写法
				 */
				
				//获得原始event的属性值  有哪些属性值 见 this.props
				for(var i = this.props.length, prop; i;) {
					prop = this.props[--i];
					event[prop] = originalEvent[prop];
				}
				//将目标元素同一成event.target
				if(!event.target)
					event.target = event.srcElement || document; // Fixes #1925 where srcElement might not be defined either
				//如果发现是文本节点 取他的父节点
				if(event.target.nodeType == 3)
					event.target = event.target.parentNode;
				if(!event.relatedTarget && event.fromElement)
					event.relatedTarget = event.fromElement == event.target ? event.toElement : event.fromElement;
				return event;
			}
		};

		win.setEvent = function(src) {
			// Allow instantiation without the 'new' keyword
			// Event object
			if(src && src.type) {
				this.originalEvent = src;
				this.type = src.type;
				// Event type
			} else
				this.type = src;
			// timeStamp is buggy for some events on Firefox(#3843)
			// So we won't rely on the native value
			this.timeStamp = now();
			// Mark it as fixed
			this[expando] = true;
		};

		function returnFalse() {
			return false;
		}

		function returnTrue() {
			return true;
		}

		setEvent.prototype = {
			preventDefault: function() {
				var e = this.originalEvent;
				if(!e)
					return;
				// if preventDefault exists run it on the original event
				if(e.preventDefault)
					e.preventDefault();
				// otherwise set the returnValue property of the original event to false (IE)
				e.returnValue = false;
			},
			stopPropagation: function() {
				var e = this.originalEvent;
				if(!e)
					return;
				// if stopPropagation exists run it on the original event
				if(e.stopPropagation)
					e.stopPropagation();
				// otherwise set the cancelBubble property of the original event to true (IE)
				e.cancelBubble = true;
			},
			stopImmediatePropagation: function() {
				this.isImmediatePropagationStopped = returnTrue;
				this.stopPropagation();
			},
			isImmediatePropagationStopped: returnFalse
		};
	})(window);
```

```js
// cache 结构体

/*
{1: {…}}
	1:(guid)
		events:
			mousewheel:
				{1: ƒ(e)}
		handle:ƒ ()
			elem:canvas
*/
```

>理解了上面的代码，那么应该会稍微明白，JQ解绑事件的原理：
>
>为每一个节点定义一个`events` 和 `handle`
>
>events 装载着一系列的事件
>
>handle 则是每个事件的代理处理器，唯一一个，所有方法的回调都会经过它处理
>
>然后再去**调用对应的方法**；并不是为每个回调方法绑定一个方法，这种绑定事件的处理方法更优，dom节点上只有一个绑定方法，避免方法绑定冗余，删除复杂；



##  学会`try{} catch(e){}` 以及 终止 `forEach/some/map....`循环

> Error：()的使用与定义不一致
> RangeError：数值越界
> ReferenceError：非法或不能识别的引用数值
> SyntaxError：发生语法解析错误
> TypeError：操作数类型错误
> URIError：URI处理函数使用不当

- 例子：
  - 关于抛出错误终止 forEach循环


> ```js
> try {						
> BM.defaults.robotPolygonsList[i].forEach(function(key, index, arr) {
> var polygon = key.polygon;
> if(polygon.getStrokeColor() === '#5959f6') throw new Error('相同的');
> polygon.setStrokeColor('#5959f6') &  polygon.setStrokeOpacity(0.5) & polygon.setStrokeWeight(1) & polygon.setFillOpacity(0.3) & polygon.setFillColor('#ffffff');
> });
> } catch (e) {
> 	console.log(e.message); // 抛出错误终止forEach 因为forEach无法像 for 那样 使用 break； return；continue;
> }
>
> ```

## JQ  `delegate`

> 指定的元素（属于被选元素的子元素）添加一个或多个事件处理程序，并规定当这些事件发生时运行的函数。
>
> 使用 delegate() 方法的事件处理程序适用于当前或 **未来** 的元素（比如由**脚本创建的新元素**）。

###  JS获取样式的方法的差异

>1. `obj.style`：这个方法只能JS只能获取写在html标签中的写在style属性中的值（style=”…”），而无法获取定义在 <style type="text/css"> 里面的属性
>
>2. IE中使用的是`obj.currentStyle`方法，而FF是用的是`getComputedStyle` 方法 【**差异**】
>
>“DOM2级样式”增强了`document.defaultView`，提供了`getComputedStyle()`方法。这个方法接受两个参数：要取得计算样式的元素和一个伪元素字符串（例如`:after`）。如果不需要伪元素信息，第二个参数可以是null。`getComputerStyle()`方法返回一个`CSSStyleDeclaration`对象，其中包含当前元素的所有计算的样式。但是需要注意，此属性只读，**read-only**
>
>*其语法为 document.defaultView.getComputedStyle ('元素', '伪类')；*
>
>1. 需要注意的是，即使有些浏览器支持这种功能，但表示值的方式可能会有所区别。例如，Firefox和Safari会返回将所有颜色转换成RGB格式。因此，即使`getComputedStyle()`方法时，最好多在几种浏览器中测试一下

## 关于 & ，|，&&，||，‘，’ 用法的一些事

> `&` or `|` 都是运算符号，与 and 或，在js中可以用来执行符号左右两条js语句,并且返回一个值，不包括赋值
>
> ```js
> var a,b; a=2 & b=3; (error)
> var a=1,b=2; a > 2 & b>3; (ok) // 0
> ```
>
> `&&` or `||` 是判断运算符号，前者意为and ， 后者意为 or，在js中可以用来执行符号左右两条js语句,并且返回一个值，不包括赋值
>
> `&&` 只有左边成立，得出的结果为true，才会执行右边的语句，可用来做判断
>
> `||` 左右边的语句都要执行
>
> ```js
> var a=1,b=2; a > 2 && b>3; (ok)  // false
>
> // 条件判断
> k = 'focus' === a.type || 'focusin' === a.type ? r : u;
> ```
>
> `,` 逗号运算符，在js中用逗号分隔执行语句意为依次执行相关语句，会以最后语句执行返回值为结果，可用作赋值
>
> ```js
> var a = 1,b = 2; if(a==5454, b==22222222) {console.log(555)}  // undefined
> var a = 1,b = 2; if(a==5454, b==2) {console.log(555)} // 555
> var a=1,b=2; a > 2 , b>3; // (ok) false
> ```



##  关于`for in` 的一些小事

> ```js
> var a={n: 1, v: 2, b: 4};
> for(var i in a){
> 	var m = a[i];
> 	m = 0
> }
> a = 
> {n: 1, v: 2, b: 4}
> ```
>
> 先看栗子
>
> 在这里，对象引用赋值失效了
>
> ```js
> var a={n: 1, v: 2, b: 4};
> for(var i in a){
> 	var m = a[i];
> 	a[i] = 0
> }
> a = 
> {n: 0, v: 0, b: 0}
> ```
>
> 再看栗子
>
> **注意问题**： 在 `for in ` 中要使用 obj[key] 来赋值，使用 `.key` 赋值也是不行的；
>
> 在`for` / `for in`里面可以使用 `continue` ， `break`

##  关于`js` 鼠标的一些破事

> | event.button |      IE 6-8      |      IE 9+       |      Chrome      |    Firefox     |
> | :----------: | :--------------: | :--------------: | :--------------: | :------------: |
> |      左键      |        1         |        0         |        0         |       0        |
> |      中间      |        4         |        1         |        1         |       1        |
> |      右键      |        2         |        2         |        2         |       2        |
> |  滚轮 （event）  | event.wheelDelta | event.wheelDelta | event.wheelDelta |  event.detail  |
> |  滚轮 （down）   |       -120       |       -120       |       -120       |       3        |
> |   滚轮 （up）    |       120        |       120        |       120        |       -3       |
> |   `js` 事件    |    mousewheel    |    mousewheel    |    mousewheel    | DOMMouseScroll |

###  

## 尾递归优化 蹦床函数

> ```js
> function tco(f) {
>   var value;
>   var active = false;
>   var accumulated = [];
>
>   return function accumulator() {
>     accumulated.push(arguments);
>     if (!active) {
>       active = true;
>       while (accumulated.length) {
>      	 value = f.apply(this, accumulated.shift());
>       }
>       active = false;
>       return value;
>     }
>   };
> }
>
> var sum = tco(function(x, y) {
>   if (y > 0) {
>   	return sum(x + 1, y - 1)
>   }
>   else {
>     return x
>   }
> });
>
> sum(1, 100000)
> ```
>
> 上面代码中，`tco`函数是尾递归优化的实现，它的奥妙就在于状态变量`active`。默认情况下，这个变量是不激活的。一旦进入尾递归优化的过程，这个变量就激活了。然后，每一轮递归`sum`返回的都是`undefined`，所以就避免了递归执行；而`accumulated`数组存放每一轮`sum`执行的参数，总是有值的，这就保证了`accumulator`函数内部的`while`循环总是会执行。这样就很巧妙地将“递归”改成了“循环”，而后一轮的参数会取代前一轮的参数，保证了调用栈只有一层。

## 模块兼容写法

*“write once，run anywhere”*  

在浏览器端，不像Node.js内部支持CommonJS，如何进行模块化，

于是出现了 CMD 与 AMD 两种方式，其主要代表是 seajs 和 requirejs，

他们都定义了一个全局函数 define 来创建一个模块

```js
// CMD
define(function(require, exports, module) {
  var foo = require('foo');
  var out = foo.bar();
  module.exports = out;
})

//AMD
define(['foo'], function(foo) {
  var out = foo.bar();
  return out;
})
```

可以看出CMD完好的保留了CommonJS的风格，

而AMD用了一种更简洁的依赖注入和函数返回的方式实现模块化。

两者除风格不同外最大区别在于加载依赖模块的方式，

CMD是懒加载，在require时才会加载依赖，

而AMD是预加载，在定义模块时就提前加载好所有依赖。

而我们所学要的是一个模块：

- 让它既能在seajs（CMD）环境里引入，又能在requirejs（AMD）环境中引入，

- 当然也能在Node.js（CommonJS）中使用，

- 另外还可以在没有模块化的环境中用script标签全局引入，

可谓是对write once，run anywhere的向往，实际上大部分npm的前端组件包也要考虑这个

```js
// 注意：闭包前加上分号是为了给前一个模块填坑，分号多了没问题，少了则语句可能发生变化。
;(function (global, factory) { 
   if (typeof module !== 'undefined' && typeof exports === 'object') {
      // 能够满足Node.js的需求    
     module.exports = factory();   
   } else if (typeof define === 'function' && (define.cmd || define.amd)) {    
     // 当没有上述全局变量，且有define全局变量时，我们认为是AMD或CMD，可以直接将factory传入define;     
     // 注意：CMD其实也支持return返回模块接口，所以两者可以通用    
     define(factory);  
   } else {    
       // 最后是script标签全局引入
       //我们可以将模块放在window上为了模块内部在浏览器和Node.js中都能使用全局对象，我们可以做此判断：    
     global.moduleName = factory();
   }
 })(typeof window !== 'undefined' ? window : global, function() {
   	var moduleName = {};    
    return moduleName; 
});
```

**在不同环境下引入** 

```js
// Node.js
var myModule = require('moduleName');

// SeaJs
define(function(require, exports, module) {
  var myModule = require('moduleName');
});

// requireJs
define(['moduleName'], function(moduleName) {
  
});

//Browser global  <script src="moduleName.js"></script>
var _js = document.createElement('script');
_js.src = "moduleName.js";
document.body.appendChild(_js);
```

## ES6 模块与 CommonJS 模块的差异

> [ES6 模块与 CommonJS 模块的差异](http://es6.ruanyifeng.com/#docs/module-loader)
>
> **Attention** 
>
> - `export *`表示再输出模块的所有方法与属性 ，*此时* ， 这个命令会忽略模块的`default` 方法
>
> - `export *` 模块整体加载所在额那个对象，应该是可以静态分析的，所以不允许运行时改变,但是本来该模块机制输出的模块地址引用是动态的，比如：如果`a`是一个对象，改写`a`的属性是允许的，但是，脚本加载了变量`a`，对其重新赋值就会报错；
>
> ```js
> import * as circle from './circle';
>
> // 下面两行都是不允许的
> circle.foo = 'hello';
> circle.area = function () {};
>
> /********************************/
>
> import {a} from './xxx.js'
> a.foo = 'hello'; // 合法操作
>
> /* a的属性可以成功改写，并且其他模块也可以读到改写后的值。不过，这种写法很难查错，建议凡是输入的变量，都当作完全只读，轻易不要改变它的属性。*/
> ```

## ES6 Promise

​	这个es6的`Promise`，原理机制有点像`JQuery.Deferred`，除此之外，下面需要注意

​	**Attention** 

> - Promise 新建后就会立即执行，然后，`then`方法指定的回调函数，将在当前脚本所有同步任务执行完才会执行，所以`resolved`最后输出
>
> - `resolve`函数的参数除了正常的值以外，还可能是另一个 `Promise` 实例，由于`p2`返回的是另一个 `Promise`，导致`p2`自己的状态无效了，由`p1`的状态决定`p2`的状态。所以，后面的`then`语句都变成针对后者（`p1`）
>
> ```js
> const p1 = new Promise(function (resolve, reject) {
> // ...
> });
>
> const p2 = new Promise(function (resolve, reject) {
> // ...
> resolve(p1);
> })
> ```
>
> - 无论在第一行还是在最后一行改变resolved的状态，`Promise`是在本轮事件循环的末尾执行，总是晚于本轮循环的同步任务
> - `then`方法返回的是一个新的`Promise`实例（注意，不是原来那个`Promise`实例）。因此可以采用链式写法，即`then`方法后面再调用另一个`then`方法，于是可以这么玩
> - *注意： 并不是只有`reslove` 才会使`then`函数执行，只要状态已经确定，`then`函数就会执行*
>
> ```js
> getJSON("/post/1.json").then(function(post) {
> return getJSON(post.commentURL);
> }).then(function funcA(comments) {
> console.log("resolved: ", comments);
> }, function funcB(err){
> console.log("rejected: ", err);
> });
> /* 第二个then方法指定的回调函数，就会等待这个新的Promise对象状态发生变化。如果变为resolved，就调用funcA，如果状态变为rejected，就调用funcB */
> ```
>
> - 无论`reject or throw new Error()`，都相当于`reject`状态，都会进入`catch`回调，或者优先进入`then(null, reject)`的第二个参数函数，*但是* ，一般来说，不要在`then`方法里面定义 Reject 状态的回调函数（即`then`的第二个参数），总是使用`catch`方法，因为这是错误
>
> - Promise 的状态一旦改变，就永久保持该状态，不会再变了
>
> - Promise 对象的错误具有“冒泡”性质，会**一直向后** 传递，直到被捕获为止。也就是说，错误总是会被下一个`catch`语句捕获
>
> - 同一层，Promise 在`resolve`语句后面，再抛出错误，不会被捕获
>
> - 跟传统的`try/catch`代码块不同的是，如果没有使用`catch`方法指定错误处理的回调函数，Promise 对象抛出的错误不会传递到外层代码，即不会有任何反应，通俗的说法就是“Promise 会吃掉错误”
>
> - `catch`函数可以用来跳过`then`函数，达到条件选择运行的目的
>
> - ## 发现了一个有趣的东西
>
> `setTimeout(fn, 0)`在下一轮**“事件循环”** 开始时执行
>
> `Promise.resolve()`在本轮**“事件循环”** 结束时执行
>
> `console.log('one')`则是立即执行，因此**最先输出** 
>
> ```js
> setTimeout(function () {
> console.log('three');
> }, 0);
>
> Promise.resolve().then(function () {
> console.log('two');
> });
>
> console.log('one');
>
> // one
> // two
> // three
> ```

​	*Application*

> ```js
> // 图片的加载写成一个Promise，一旦加载完成，Promise的状态就发生变化
> const preloadImage = function (path) {
> return new Promise(function (resolve, reject) {
> const image = new Image();
> image.onload  = resolve;
> image.onerror = reject;
> image.src = path;
> });
> };
> ```
>
> 

## ES6 - Generator 
 	*grammar attention*
 	> - Generator 函数是一个状态机，封装了多个内部状态
 	>    >    >
 	>    >    >调用 `Generator` 函数后，该函数并不执行，返回的也不是函数运行结果，而是一个指向内部状态的指针对象
 	>    >    >
 	>    >    >位置记忆
 	>    >    >
 	>    >    >执行多次
 	>    >    >
 	>    >    >遇到`yield`，函数暂停执行，下一次再从该位置继续向后执行
 	>    >    >
 	>    >    >只能用在 Generator 函数里面
 	>    >    >
 	>    >    >紧跟在`yield`后面的那个表达式的值，作为返回的对象的`value`属性值
 	>    >    >
 	>    >    >- `yield` 
 	>    >    >
 	>    >    >`yield`表达式本身没有返回值，或者说总是返回`undefined`
 	>    >    >
 	>    >    >- `next`
 	>    >    >
 	>    >    >`next`方法可以带一个参数，该参数就会被当作上一个`yield`表达式的返回值
 	>    >    >
 	>    >    >通过`next`方法的参数，就有办法在 Generator 函数开始运行之后，继续向函数体内部注入值
 	>    >    >
 	>    >    >- `return` 
 	>    >    >
 	>    >    >Generator 函数的遍历就终止
 	>    >    >
 	>    >    >`return`方法调用时，不提供参数，则返回值的`value`属性为`undefined`
 	>    >    >
 	>    >    >`return`方法会推迟到`finally`代码块执行完再执行

​	如果将 Generator 函数当作协程，完全可以将多个需要互相协作的任务写成 Generator 函数，它们之间使用`yield`表示式交换控制权

> ```js
> Promise.resolve(step1)
> .then(step2)
> .then(step3)
> .then(step4)
> .then(function (value4) {
> // Do something with value4
> }, function (error) {
> // Handle any error from step1 through step4
> })
> .done();
> ```

​	Generator 函数可以进一步改善代码运行流程
> ```js
> function* longRunningTask(value1) {
> try {
> var value2 = yield step1(value1);
> var value3 = yield step2(value2);
> var value4 = yield step3(value3);
> var value5 = yield step4(value4);
> // Do something with value4
> } catch (e) {
> // Handle any error from step1 through step4
> }
> }
>
> scheduler(longRunningTask(initialValue));
>
> function scheduler(task) {
> var taskObj = task.next(task.value);
> // 如果Generator函数未结束，就继续调用
> if (!taskObj.done) {
> task.value = taskObj.value
> scheduler(task);
> }
> }
> ```
>

​	利用`for...of`循环会自动依次执行`yield`命令的特性，提供一种更一般的控制流管理的方法

> ```js
> let steps = [step1Func, step2Func, step3Func];
>
> function* iterateSteps(steps){
>   for (var i=0; i< steps.length; i++){
>     var step = steps[i];
>     yield step();
>   }
> }
>
> /**************/
>
> let jobs = [job1, job2, job3];
>
> function* iterateJobs(jobs){
>   for (var i=0; i< jobs.length; i++){
>     var job = jobs[i];
>     yield* iterateSteps(job.steps);
>   }
> }
> ```

​	解决异步请求，同步运行

> ```js
> function* main() {
>   var result = yield request("http://some.url");
>   var resp = JSON.parse(result);
>   console.log(resp.value);
> }
>
> function request(url) {
>   makeAjaxCall(url, function(response){
>   	it.next(response);
>   });
> }
>
> var it = main();
> it.next();
> ```
>
> 

