## Iframe

### 获取iframe的几种方法

- window.frames[0].frameElement 
- 通过jQuery 选择器 id or name 来获取

### 父子页面通讯 -- 解决跨域

- postMessage 最流行的使用方法

  > ####父页面
   ```js
   window.addEventListener('message', function(e) {
       var data = e.data || '';
       if(!data) return;
       // balabala 通过判断e.origin 确定是否是正确的子页面发过来的信息
    }, false);

    $(window).on('message', function(e) { 
        var event = e.originalEvent
        var data = event.data
        var origin = event.origin
    })
   ```
    > #### 子页面

  ```js
  window.parent.postMessage(type,'*'); // 因为没有固定origin，所以采用*
  ```

- window.name 跨域实现原理及实例
  > 当在浏览器中打开一个页面，或者在页面中添加一个iframe时即会创建一个对应的window对象，当页面加载另一个新的页面时，window的name属性是不会变的。这样就可以利用在页面动态添加一个iframe然后src加载数据页面，在数据页面将需要的数据赋值给window.name。然而此时承载iframe的parent页面还是不能直接访问，不在同一域下iframe的name属性，这时只需要将iframe再加载一个与承载页面同域的空白页面，即可对window.name进行数据读取

  ```js
  function getData ( url, fn ) {
      var oIframe = document.createElement('iframe'),
          firstBtn = true,
          loadFn = function () {
              if ( firstBtn ) {
                  //导航回同一域下的air.html，以便获取到name值
                  oIframe.contentWindow.location = 'http://www.a.com/air.html';
                  firstBtn = false;
              } else {
                  fn( oIframe.contentWindow.name );
                  oIframe.contentWindow.document.write('');
                  oIframe.contentWindow.close();
                  document.body.removeChild(oIframe);
                  oIframe.src = '';
                  oIframe = null;
              }
          };

      oIframe.src = url;

      //1.第一次iframe加载完毕触发事件，执行loadFn函数，会将iframe导航回air.html
      //2.air.html加载完毕后又会触发事件，再次执行loadFn函数，此时会走else
      if ( oIframe.attachEvent ) {
          oIframe.attachEvent( 'onload', loadFn );
      } else {
          oIframe.onload = loadFn;
      }

      document.body.appendChild(oIframe);
  }


  //页面加载完毕后调用getData函数去获取数据
  window.onload = function () {
      getData( 'http://www.b.com/target.html', function ( data ) {
          console.log( data );
      } );
  }

  ```




- document.domain  跨域实现原理及实例

  > #### 首先用document.domain来指定域，是可以的，但是有局限性，也就是一级域名一致才可以。也就是说适合子页面间的信息交流，你如下：
  >
  > `www.sojson.com`  下指到`sojson.com` 是可以的。
  >
  > `icp.sojson.com`  下指到 `sojson.com` 是可以的。
  >
  > 像上面是可以的，因为[ 一级域名  ](http://www.sojson.com/tag_yjym.html)都是 `sojson.com` 。
  >
  > `www.sojson.com`  下指到 `www.baidu.com`  是不行的。
  >
  > `sojson.com`  指到 `baidu.com`  还是不行的