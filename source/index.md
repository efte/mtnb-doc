---
title: MTNB - MeiTuan Native Bridge

language_tabs:
  - javascript

toc_footers:
  - <a href='http://code.dianpingoa.com/ed-f2e/mtnb' target="about:_blank;">项目地址</a>
  - <a href='http://code.dianpingoa.com/ed-f2e/mtnb_h5_demo'>mtnb h5 demo</a>

search: true
---

# 概述

MTNB(meituan native bridge)，是用来在混合应用开发中打通客户端应用（美团app，开店宝，猫眼等）与网页应用信道的桥梁。MTNB也作为美团移动桥协议在webapp中的命名空间存在。
[详细文档](http://wiki.sankuai.com/display/DEVPUB/Meituan+Native+Bridge)

![MTNB接入](images/MTNB接入.png)

* mtnb-core由美团的npm模块提供，业务方不需要关心。
* mtnb和mtnb-merchant依赖于mtnb-core，并封装了BA认证过程，业务方不需要关心，直接调用即可。
* mtnb-merchant扩展了两部分功能，native支持的和js mock的接口。
* @dp/util-mtshop-ui-less提供了基础样式组件以及mtnb-merchant js相关UI接口的样式。

# 引入
使用MTNB需要通过BA认证，因此需要访问后端接口拿到authInfo，前端使用authInfo鉴权，通过鉴权才可以使用jsbridge提供的功能。

以上BA认证过程已经在mtnb以及mtnb-merchant中包装好，直接调用即可。

```javascript
var MTNB = require('mtnb');
var MTNB = require('mtnb-merchant');
```

[mtnb](http://code.dianpingoa.com/ed-f2e/mtnb)及[开店宝mtnb](http://code.dianpingoa.com/ed-f2e/mtnb-merchant)目前仅支持通过Cortex方式引入。

点评侧对mtnb-merchant做了二次封装，提供了UI及storage的一些功能。

# 初始化
```javascript
MTNB.ready(function() {
	// 业务代码
	MTNB.use(...);
});
```
<aside class="warning">MTNB中所有和native相关的接口都需要在鉴权成功(MTNB.ready)后使用。</aside>

# 基础模块-统计

## 发送统计接口
```javascript
MTNB.use('stat.log', {
    url: 'http://api.mobile.meituan.com/data/collect.json',
    type: "POST",
    data: 'type=i_stat&content={"appnm":"group","ct":"i","ua":"Mozilla/5.0 (iPhone; CPU iPhone OS 7_0 like Mac OS X; en-us) AppleWebKit/537.51.1 (KHTML, like Gecko) Version/7.0 Mobile/11A465 Safari/9537.53","app":0,"msid":"pc4pktmlm4swf9bf3vkq05b2","did":"BFCAB776BF8D7D25B53701E7EE6D93347730DA6EE5F5BB77F3DB4E0FEF21FDCB","uid":null,"uuid":"BFCAB776BF8D7D25B53701E7EE6D93347730DA6EE5F5BB77F3DB4E0FEF21FDCB","cityid":"1","ch":"touch","lch":null,"os":"iPhone","sc":"568*451","net":"none","ip":"106.120.108.34","evs":[{"cate":"truck","loadCount":10,"sumSize":33261,"duration":13,"xhrCount":0,"lsCount":10,"xhrSize":0,"lsSize":33261,"nm":"STATIC","tm":1433325192,"seq":1433325192100}]}',
    callback: function(res) {

    }
});
```
模块/app | 版本
--- | ---
mtnb | 0.3.0+
mtnb-merchant | 0.2.0+
开店宝 | 4.9.0+

<aside class="warning">发送统计接口，底层调用proxy接口实现</aside>

type | name | 描述
--- | ---- | ----
string | url | 统计的请求地址，包含?
string | type | 统计的请求方式，默认为GET
string | data | 统计的内容
function | callback | 请求回调函数

# 基础模块-定位

## 定位接口
```javascript
MTNB.use('geo.getLocation', {
    timeout: 10000
}, function (res) {
    if (res.status == 0) {
        console.log(res.data.latitude + ', ' + res.data.longitude);
    } else {
        alert(res.message);
    }
});
```
模块/app | 版本
--- | ---
mtnb | 0.3.0+
mtnb-merchant | 0.2.0+
开店宝 | 4.9.0+

### 输入：

type | name | 描述
--- | ---- | ----
int | timeout | 超时时间（毫秒），-1 默认不限制
		
### 输出：

type | name | 描述
--- | ---- | ----
int | status | 返回的状态码， 0：成功，1: 超时， 2：不可用
string | message | 错误信息
object | data | 返回数据：data.latitude为纬度，data.longitude为经度，data.radius为误差（m）

## 打开地图
```javascript
MTNB.use('geo.openMap', {
    lat: 140.23434,
    lng: 40.4335
});
```
模块/app | 版本
--- | ---
mtnb | 0.3.0+
mtnb-merchant | 0.2.0+
开店宝 | 4.9.0+

<aside class="warning">该api只提供接口描述，具体实现因为业务线使用地图sdk不同，要各自完成</aside>

type | name | 描述
--- | ---- | ----
boolean | [marker] | 是否在中心标点，默认false
boolean | [goto] | 是否去哪里，默认false，如果为true，显示用户位置到所在点的路线
float | lat | 纬度
float | lng | 经度
int | [zoom] | 缩放比例, 默认16（街区）	
				
# 基础模块-分享

## 直接调取分享
```javascript
MTNB.use('share.invoke', {
    url: location.href,
    title: document.title,
    pic: $('img').attr('src'),
    callback: function (res) {
        // res有分享渠道的信息，具体格式需要再确定
    }
}, function (res) {
    if (res.status == 0) {
        console.log('打开分享')
    } else {
        alert(res.message);
    }
});
```
模块/app | 版本
--- | ---
mtnb | 0.3.0+
mtnb-merchant | 0.2.0+
开店宝 | 4.9.0+

<aside class="warning">该api只提供接口描述，具体实现因为业务线使用地图sdk不同，要各自完成</aside>

type | name | 描述
--- | ---- | ----
string | url | 分享的地址
string | title | 分享标题
string | pic | 分享的图片
string | [content] | 分享的内容

# 基础模块-webview

## 关闭当前webview
```javascript
MTNB.use('webview.close', {
	anime: "slideleft"
});
```
模块/app | 版本
--- | ---
mtnb | 0.3.0+
mtnb-merchant | 0.2.0+
开店宝 | 4.9.0+

<aside class="warning">在MTNB.init成功回调之后才可使用</aside>
type | name | 描述
--- | ---- | ----
string | anime | 关闭时执行的动画，如果不传，按照打开的反向动画执行

## 打开新的webview
```javascript
MTNB.use('webview.open', {
	url: 'http://i.meiutan.com/',
	anime: "slideleft"
});
```
<aside class="warning">在MTNB.ready中使用</aside>

模块/app | 版本
--- | ---
mtnb | 0.3.0+
mtnb-merchant | 0.2.0+
开店宝 | 4.9.0+

type | name | 说明
--- | --- | ----
string | url	| 打开的地址
string	| anime	| 动画类型：swipeLeft, swipeRight 左右切换；fadeIn, fadeOut 淡入淡出


## 设置webview标题
```javascript
MTNB.use('webview.setTitle', {
    title: "一个很长很长的标题",
    callback: function () {
        // 标题被点击后的操作...
    }
});
```
<aside class="warning">在MTNB.ready中使用</aside>

模块/app | 版本
--- | ---
mtnb | 0.3.0+
mtnb-merchant | 0.2.0+
开店宝 | 4.9.0+

type | name | 说明
--- | --- | ----
function	|callback	| 监听标题点击的回调
string | title	| 标题名称


## 设置复杂的webview标题
```javascript
MTNB.use('webview.setHtmlTitle', {
    title: "<font size="4" face="arial" color="black">演唱会 </font><font size="2" color="black"> 北京</font><font size="1" color="black">▼</font>",
    callback: function () {
        // 标题被点击后的操作...
    }
});
```
<aside class="warning">在MTNB.ready中使用</aside>
<aside class="warning">不支持换行</aside>

模块/app | 版本
--- | ---
mtnb | 0.3.0+
mtnb-merchant | 0.2.0+
开店宝 | 4.9.0+

type | name | 说明
--- | --- | ----
string	| title | 标题名称，使用font来实现
function | callback | 监听标题点击的回调


## 设置角标
```javascript
// 分享
MTNB.use('webview.setIcon', {
    type: "share",
    channel: 385,
    url: location.href,
    title: document.title,
    pic: $('img').attr('src'),
    callback: function () {
        // 分享完成后的回调
    }
});
// 链接
MTNB.use('webview.setIcon', {
    type: "text",
    text: "退款帮助",
    url: "i.meituan.com/help/refund",
    callback: function () {
        // 点击分享的回调
    }
});
// 回调
MTNB.use('webview.setIcon', {
    type: "text",
    text: "设置提醒",
    callback: function () {
        // 点击提醒的回调
    },
});
// 设置多个icon
MTNB.use('webview.setIcon', [
    {
        type: "text",
        text: "设置提醒",
        callback: function () {
            // 交互
        }
    },
    {
        type: "text",
        text: "收藏",
        callback: function () {
            // 交互
        }
    }
]);
```
<aside class="warning">在MTNB.ready中使用</aside>

模块/app | 版本
--- | ---
mtnb | 0.3.0+
mtnb-merchant | 0.2.0+
开店宝 | 4.9.0+

type | name | 说明
--- | --- | ----
object|	data|	附加数据，其中的callback为事件触发后的回调，参数是{}
string|	type|	icon类型，“text”文字，“share”分享

## 订制左上角按钮
```javascript
	var params = {};
    if (location.search) {
        location.search.substr(1).split('&').map(function(search) {
            var pare = search.split('=');
            params[pare[0]] = decodeURIComponent(pare[1]);
        });
    }
    if (!params.reloaded) {
        params.reloaded = 1;
        var queryArray = [];
        for (var k in params) {
            queryArray.push(k + '=' + params[k]);
        }
        location.href = location.protocol + '//' + location.host + location.pathname + '?' + queryArray.join('&');
    }
```

开店宝左上角按钮，请参考[此文档](http://wiki.sankuai.com/pages/viewpage.action?pageId=372249318)

因为android没有完全实现webview相关的接口，尤其是webview.open，所以推荐使用location.href的方式进行页面跳转。

开店宝webview有一个已知的bug是：webview打开页面A跳转页面B，再点返回直接关闭webview，针对此问题的解决方法是，页面加载完自动进行一次页面跳转，示例代码见右侧：

# 基础模块-支付
<aside class="warning">没有做在1期</aside>
## 收银台
```javascript
MTNB.use('pay.go', {
    "order":23234244,
    "tradeNumber":"11231231231321",
    "payTocken":"adasreaxzca",
    callback: function (res) {
        // 支付完成的回调，res中包含有成功与否的信息
    },
    url: "....."
});
```
模块/app | 版本
--- | ---
mtnb | 0.3.0+
mtnb-merchant | 0.2.0+
开店宝 | 4.9.0+

type | name | 描述
--- | ---- | ----
int | order | 订单号
string | tradeNumber | 流水号
string | payToken | 支付token
string | url | 完成后跳转

# 基础模块-账户 

## 登录
```javascript
MTNB.use('account.login', {
    callback: function (res) {
        // 登录完成的回调，res中包含有成功与否的信息
    },
    // 登录完成后的回调页面
    url: '...'
});
```
模块/app | 版本
--- | ---
mtnb | 0.3.0+
mtnb-merchant | 0.2.0+
开店宝 | 4.9.0+

type | name | 描述
--- | ---- | ----
string | backurl | 完成后的返回地址，默认为当前页面

## 登出
```javascript
MTNB.use('account.logout', {
    callback: function (res) {
        // 登出完成的回调，res中包含有成功与否的信息
    },
    // 登出完成后的回调页面
    url: '...'
});
```
模块/app | 版本
--- | ---
mtnb | 0.3.0+
mtnb-merchant | 0.2.0+
开店宝 | 4.9.0+

### 输入：无

### 输出：
type | name | 描述
--- | ---- | ----
int | status | 返回的状态码， 0：成功
string | message | 错误数据
string | data | 返回数据

# 开店宝订制-native接口
<aside class="success">以下接口只在开店宝APP中可用，在开店宝APP中请使用mtnb-merchant。</aside>

```
var MTNB = require('mtnb-merchant');
```


## 设置 TAB 切换标题
```javascript
MTNB.use('webviewBasicOperation.setTabPageForTitleViewWithParams',
[{
    name : 'title 1',
    checked : true,
    callback : function(args){
        alert('title 1 clicked'); 
    }
}, {
    name : 'title 2',
    checked : false, 
    callback : function(args){
        alert('title 2 clicked'); 
    }
}],
function(message){
    alert(JSON.stringify(message));  
});
```
setTabPageForTitleViewWithParams：该接口设置切换的标题以及各标题在被点击后回调的方法

模块/app | 版本
--- | ---
mtnb-merchant | 0.2.0+
开店宝 | 4.9.0+

<table>
	<tr>
	    <th>type</th>
	    <th>name</th>
	    <th>说明</th>
    </tr>
	<tr>
	    <td>array</td>
	    <td>data</td>
	    <td>
			包含所有TabPage信息的对象数组, 其中每个对象的属性又分别为：
			<table border="1">
				<tr>
				    <td>string</td>
				    <td>name</td>
					<td>TabPage的名字</td>
			    </tr>
				<tr>
				    <td>bool</td>
				    <td>checked</td>
					<td>该TabPage是否被选中</td>
				</tr>
				<tr>
				    <td>function</td>
				    <td>callback</td>
					<td>当该tab被点击时，此方法会被调用</td>
				</tr>
		    </table>
		</td>
	</tr>
	<tr>
	    <td>functoin</td>
	    <td>callback</td>
		<td>设置成功后回调</td>
	</tr>
</table>

## 设置 header 右侧按钮
```javascript
MTNB.use('webviewBasicOperation.setHeaderActionForRightButtonItem',
{
    type : 'icon_button',  
    content : 'https://avatars3.githubusercontent.com/u/855749?v=3&s=460',  
    callback: function(args){
    }
},
function(message){
    alert(JSON.stringify(message));   
});
```
setHeaderActionForRightButtonItem：该接口设置header右侧按钮的种类以及被点击时所执行的相应操作

模块/app | 版本
--- | ---
mtnb-merchant | 0.2.0+
开店宝 | 4.9.0+

<table>
	<tr>
	    <th>type</th>
	    <th>name</th>
	    <th>说明</th>
    </tr>
	<tr>
	    <td>object</td>
	    <td>data</td>
	    <td>
			header右侧按钮的相关属性
			<table border="1">
				<tr>
				    <td>string</td>
				    <td>type</td>
					<td>按钮的种类，可选值： text_button（文本），icon_button（图标）</td>
			    </tr>
				<tr>
				    <td>string</td>
				    <td>content</td>
					<td>若按钮的种类为text_button（文本）时，此为纯文本；若按钮的种类为icon_button（图标）时，此为指向 icon 的 url</td>
				</tr>
				<tr>
				    <td>function</td>
				    <td>callback</td>
					<td>当 header 右侧区域被点击时，此方法会被调用</td>
				</tr>
		    </table>
		</td>
	</tr>
	<tr>
	    <td>functoin</td>
	    <td>callback</td>
		<td>设置成功后回调</td>
	</tr>
</table>

## 新开webview
```javascript
MTNB.use('webviewBasicOperation.setLoadWebViewWithParams',
{
    url : 'http://i.meituan.com',  
    callback: function(args){
    }
},
function(message){
    alert(JSON.stringify(message));  
});
```
setLoadWebViewWithParams：该接口新打开了一个webview并加载了相应的url，类似于在 webview 中实现 target="_blank" 

模块/app | 版本
--- | ---
mtnb-merchant | 0.2.0+
开店宝 | 4.9.0+


<table>
	<tr>
	    <th>type</th>
	    <th>name</th>
	    <th>说明</th>
    </tr>
	<tr>
	    <td>object</td>
	    <td>data</td>
	    <td>该对象包括了新打开的webview的各项属性：
			<table border="1">
				<tr>
				    <td>string</td>
				    <td>url</td>
					<td>新打开 webview 指向的 url</td>
			    </tr>
				<tr>
				    <td>function</td>
				    <td>callback</td>
					<td>当新打开的 webview 关闭时，此方法会被调用</td>
				</tr>
		    </table>
		</td>
	</tr>
	<tr>
	    <td>functoin</td>
	    <td>callback</td>
		<td>设置成功后回调</td>
	</tr>
</table>


## 扫二维码
```javascript
MTNB.use('webviewBasicOperation.launchNativeQRReaderPage',
{  
    imageUrl : 'https://avatars3.githubusercontent.com/u/855749?v=3&s=460',  
    callback: function(args){
        alert(JSON.stringify(args)); 
    }
},
function(message){
    alert(JSON.stringify(message));
});
```
launchNativeQRReaderPage：该接口打开并使用 native 的扫二维码功能

模块/app | 版本
--- | ---
mtnb-merchant | 0.2.0+
开店宝 | 4.9.0+

<table>
	<tr>
	    <th>type</th>
	    <th>name</th>
	    <th>说明</th>
    </tr>
	<tr>
	    <td>object</td>
	    <td>data</td>
	    <td>
			与扫二维码功能相关的各项属性：
			<table border="1">
				<tr>
				    <td>string</td>
				    <td>imageUrl</td>
					<td>在该扫码页底部显示的图片的url</td>
			    </tr>
				<tr>
				    <td>function</td>
				    <td>callback</td>
					<td>当扫码结束，获取到二维码值之后，此方法会被调用，参数为扫码得到的值</td>
				</tr>
		    </table>
		</td>
	</tr>
	<tr>
	    <td>functoin</td>
	    <td>callback</td>
		<td>设置成功后回调</td>
	</tr>
</table>

# 开店宝订制-js接口

mtnb-merchant中点评开发环境提供的js功能，通过extend方式挂在MTNB对象上。

js接口只生成dom，样式需要额外依赖或者自己实现，请使用符合开店宝视觉规范的UI组件：[开店宝基础样式组件](http://code.dianpingoa.com/ed-f2e/util-mtshop-m-less-demo/tree/master)。

<aside class="success">不需要在ready中调用。</aside>

## alert
```javascript
MTNB.alert({
    title: 'title', // 标题文字
    message: 'message', // 内容文字
    button: 'button', // 按钮文字
    success: function(){
        // 用户点击确认
    }
});
```
弹出类似于window.alert的结构。
<aside class="success">mtnb-merchant@0.2.0+</aside>
<aside class="success">不需要在ready中调用。</aside>
<aside class="warning">请使用@dp/util-mtshop-m-less</aside>

## confirm
```javascript
MTNB.confirm({
    title: 'title', // 标题文字
    message: 'message', // 内容文字
    okButton: 'OK', // 确认按钮文字
    cancelButton: 'Cancel', // 取消按钮文字
    success: function(e) {
        // 用户点击确认或取消
        if (e.ret) {} // true: 确认 false: 取消
    }
});
```
弹出类似于window.confirm的结构，有确定和取消的点击回调。
<aside class="success">mtnb-merchant@0.2.0+</aside>
<aside class="success">不需要在ready中调用。</aside>
<aside class="warning">请使用@dp/util-mtshop-m-less</aside>

## toast
```javascript
MTNB.toast({
    title: 'title', // 文字
    timeout: 2000 // 持续时间
});
```
弹出一段简短的信息，一定时间后消失。
<aside class="success">mtnb-merchant@0.2.0+</aside>
<aside class="success">不需要在ready中调用。</aside>
<aside class="warning">请使用@dp/util-mtshop-m-less</aside>

## tooltip
```javascript
MTNB.tooltip({
	targetEl: $('.level_func'),// zepto object
	placement: 'right',// top/right/bottom/left
	message: '请设置会员等级权益，否则<br>该会员卡无法正式启用', //可以传入string或者html
	timeout: 3000 // 3000ms后自动消失
});
```
带箭头的，在指定位置出现的，一定时间后自动消失的提示信息。
<aside class="success">mtnb-merchant@0.2.0+</aside>
<aside class="success">不需要在ready中调用。</aside>
<aside class="warning">请使用@dp/util-mtshop-m-less</aside>

## loading

```javascript
MTNB.loading.show();
MTNB.loading.hide();
```
## store

```javascript
MTNB.config({
	bizname: 'app-mtb-club'// 通常使用包名
});
MTNB.store({
	key: "key",
	value: "value"
	success: function(){
    	// 存值成功
	}
});
```
使用localStorage储值，value必须是字符串。
<aside class="success">mtnb-merchant@0.2.0+</aside>
<aside class="success">不需要在ready中调用。</aside>
<aside class="warning">使用前需要先使用`MTNB.config({bizname:“your-biz-name”});`进行配置</aside>
## retrieve

```javascript
MTNB.config({
	bizname: 'app-mtb-club'// 通常使用包名
});
MTNB.retrieve({
	key: 'key',
	success: function(e) {
		alert(e.value);
	}
});
```
取store的值。
<aside class="success">mtnb-merchant@0.2.0+</aside>
<aside class="success">不需要在ready中调用。</aside>
<aside class="warning">使用前需要先使用`MTNB.config({bizname:“your-biz-name”});`进行配置</aside>

## del
```javascript
MTNB.config({
	bizname: 'app-mtb-club'// 通常使用包名
});
MTNB.del({
	key: 'key',
	success: function(e) {
		// del success
	}
});
```
删除store的值。
<aside class="success">mtnb-merchant@0.2.0+</aside>
<aside class="success">不需要在ready中调用。</aside>
<aside class="warning">使用前需要先使用`MTNB.config({bizname:“your-biz-name”});`进行配置</aside>


