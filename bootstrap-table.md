## Bootstrap-table 常见用法

> ```js
> var settings = {
>     method: 'post',
>     // 一般使用url访问后台获取数据
>     url: url,
>     // 可以使用自定义的数据
>     // data: someData，
>     height: height,
>     // 根据条件查询
>     queryParams: function(params) { 
>         params['deviceId'] = $("#deviceId").val();
>         return params; 
>     },
>     // 对每一行指定唯一标识符， 最好指定数据中的KEY值，可能排序的时候会用到
>     uniqueId: "actionCode",
>     // 自定义列的数据显示方式
>     columns: [{
>         field: 'createTime', // 对应的数据字段
>         align: 'center',
>         width: '40%',
>         title: '创建时间'
>     }, {
>         ......
>     },{
>         align: 'center',
>         title: '管理操作',
>         width: '10%',
>         events: _operationEvents(), // 接受一个object, 定义由_formatHandler格式化的按钮的作用
>         formatter: _formatHandler // 返回一个字符串作为html文本
>     }],
>     // 在表格加载显示之后，会调用此方法， this指向实例本身，可以另外绑定context
>     onPostBody: _eventListen.bind(that),
>     // 双击使用方法
>     onDblClickRow: function(row, $element, field) {},
>     responseHandler: function(res) {
>         // 加载服务器数据之前的处理程序，可以用来格式化数据。参数：res为从服务器请求到的数据。
>    	 	// 可以在这里改变数据结果，以符合自身需求
>         return res;
> 	}，
>     //单选
>     onCheck: function(row, $element) {},
>     //取消单选
>     onUncheck: function(row, $element) {},
>     //全部选中
>     onCheckAll: function(rows) {},
>     //全部取消选中
>     onUncheckAll: function(rows) {},
> };
> ```





## 易错误区

