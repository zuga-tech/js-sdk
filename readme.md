# Bainu JS SDK
## 概述
Bainu js-sdk 是面向蒙古文网页开发者提供的基于bainu的网页开发工具包。
由于蒙古文的书写方向、输入法，对很多移动网站应用来说解决显示与输入成了较大的难题，现在开发者们可以使用Bainu js sdk来快速实现移动网站应用，让移动网站应用具有更好的用户体验。
bainu js sdk 提供了蒙古文的弹出框，确认框，输入框，输入法等基础组件功能外，还有选图、位置等功能。让开发者的工作变得更为高效。为bainu用户提供了更优质的网页体验。

## JSSDK使用步骤
### 1. 开发者认证
   目前Bainu开放平台暂未开启公开注册，所以请将你的应用信息及开发者信息发送到 business@zuga-tech.com ，我们审核通过后会联系你。
   需要提供的信息：
   - **应用名称**
   - **蒙文名称**
   - **LOGO** （640*640）
   - **官方网站**
   - **授权域名**
   - **蒙文介绍**
   - **团队或个人介绍**
   - **联系方式**

   邮件申请通过后会你将获得：
   - **appId**
   - **jsapi_ticket** （用于生成JS-SDK权限验证的签名，由于目前没有公开开放，所提供的jsapi_ticket是固定的，请妥善管理。）
   - **可使用的api列表**

   签名生成规则如下：参与签名的字段包括noncestr（随机字符串）, 有效的jsapi_ticket, timestamp（时间戳）, url（当前网页的URL，不包含#及其后面部分） 。对所有待签名参数按照字段名的ASCII 码从小到大排序（字典序）后，使用URL键值对的格式（即key1=value1&key2=value2…）拼接成字符串string1。这里需要注意的是所有参数名均为小写字符。对string1作sha1加密，字段名和字段值都采用原始值，不进行URL 转义。
   即signature=sha1(string1)。 示例：
   ```
   noncestr=3361d794c73b5d27
   jsapi_ticket=a9471f601efd5440d6871147b8dd989b5a95abd588e4d37b72bef38c08491d90
   timestamp=1498473262
   url=http://zuga-tech.net?params=value
   ```

   步骤1. 对所有待签名参数按照字段名的ASCII 码从小到大排序（字典序）后，使用URL键值对的格式（即key1=value1&key2=value2…）拼接成字符串string1：
   ```
   jsapi_ticket=a9471f601efd5440d6871147b8dd989b5a95abd588e4d37b72bef38c08491d90&noncestr=3361d794c73b5d27&timestamp=1498473262&url=http://zuga-tech.net?params=value
   ```

   步骤2. 对string1进行sha1签名，得到signature：
   ```
   704a330431353f544d79582bddf3faa5e862a488
   ```

   注意事项
   - 签名用的noncestr和timestamp必须与bainu.config中的nonceStr和timestamp相同。
   - 签名用的url必须是调用JS接口页面的完整URL。
   - 出于安全考虑，开发者必须在服务器端实现签名的逻辑。


### 2. 引入js文件
   在需要使用Bainu js-sdk的页面引入如下js文件。`http://static.zuga-tech.com/bainu-js-sdk-1.0.js`
### 3. 通过config接口注入权限验证配置
   所有需要使用JS-SDK的页面必须先注入配置信息，否则将无法调用（同一个url仅需调用一次，对于变化url的SPA的web app可在每次url变化时进行调用）。
```javascript
bainu.config({
    debug: true, // 开启调试模式,调用的所有api的返回值会在客户端alert出来，若要查看传入的参数，可以在pc端打开，参数信息会通过log打出，仅在pc端时才会打印。
    appId: '', // 必填，公众号的唯一标识
    timestamp: '', // 必填，生成签名的时间戳
    nonceStr: '', // 必填，生成签名的随机串
    signature: '',// 必填，权限验证的签名
    apiList: [] // 必填，需要使用的JS接口列表
});
```
### 4. 通过ready接口处理成功验证

```javascript
bainu.ready(function() {
    // config信息验证后会执行ready方法，所有接口调用都必须在config接口获得结果之后，
    // config是一个客户端的异步操作，所以如果需要在页面加载时就调用相关接口，则须把相关接口放在ready函数中调用来确保正确执行。
    // 对于用户触发时才调用的接口，则可以直接调用，不需要放在ready函数中。
});
```

### 5. 通过error接口处理失败验证

```javascript
bainu.error(function(res){
    // config信息验证失败会执行error函数，如签名过期导致验证失败，具体错误信息可以打开config的debug模式查看，也可以在返回的res参数中查看。
});
```

## 接口调用说明
所有接口通过bainu对象来调用，参数是一个对象，除了每个接口本身需要传的参数之外，还有以下通用参数：
   - success：接口调用成功时执行的回调函数。
   - fail：接口调用失败时执行的回调函数。
   - complete：接口调用完成时执行的回调函数，无论成功或失败都会执行。
   - cancel：用户点击取消时的回调函数，仅部分有用户取消操作的api才会用到。
   - trigger: 监听Menu中的按钮点击时触发的方法，该方法仅支持Menu中的相关接口。

所有接口返回的数据格式为：
```
{
  code: int,   // 0=>success, 1=>cancel, 2=>error ， 每个接口必须返回
  // 如果失败则在error中返回对应的代码及错误信息
  [error]: {
    code: int
    description: string
  },
  [data]: {}// 对于某些接口没有返回值。
}
```
注意：
- `complete`回调函数接收到的参数为完全格式，
- `fail`回调函数接收到的参数为error部分，error的code请看`附1`。
- `success`与`cancel`回调函数接收到的参数为data部分。

### 基础接口

#### 弹出框`alert`接口
```javascript
bainu.alert({
  'message' : '' //提示内容
});
```

#### 确认框`confirm`接口
```javascript
bainu.confirm({
  'message': '', // 提示内容
  'confirm': '', // 确认按钮文字
  'deny' : '', // 取消按钮文字
  'success': function(res){
    // 确定
  },
  'cancel': function(){
    // 取消
  }
});
```

#### 蒙文输入框`input`接口
使用bainu自带的原生输入框及bainu输入法进行蒙文输入
```javascript
bainu.input({
  'text': '', //当前文本框已经存在的内容
  'placeholder': '', //提示文本
  'maxLength': 1000, // 长度限制 默认:1000
  'line': 0, //0: 多行, 1:单行 默认: 0
  'success': function(res){
    var text = res.text;
  },
  'cancel': function(){

  }
});
```

### 图像接口

#### 选择系统相册图片
```javascript
bainu.chooseImage({
  'count': 1, //默认9
  'crop': true, // true 剪切, false 不剪切(只在count为1时起作用)
  'success': function(res) {
    var localIds = res.localIds; //只是该图片的编号(不可直接用于img标签，如果想获取图片数据则调用getLocalImgData)。
   },
  'cancel': function(){

  }
});
```

#### 上传照片
```javascript
bainu.uploadImage({
  'localIds': '', //需要上传的图片的本地ID，由chooseImage接口获得
  'isShowProgressTips': 1, // 默认为1，显示进度提示
  'success': function(res){
    var serverIds = res.serverIds; // 返回图片的服务器端ID数组
   },
});
```

#### 图片预览
```javascript
bainu.previewImage({
  'current': '', //当前图片的url或者chooseImage获得的localId
  'urls': [''] //一组预览图片的url或者localId列表
});
```

#### 获取图片数据
```javascript
bainu.getLocalImageData({
  'localId': '', //chooseImage获得的localId
  'thumb': false, //是否返回压缩图
  'success': function(image) {
    var imageBase64 = image.base64; // 图片文件的base64，不包含类型及等数据，img标签src显示时应加 'data:image/png;base64,' + image.base64
  }
});
```

### 获取设备信息

#### 获取网络类型
```javascript
bainu.getNetworkType({
  success: function(res) {
    var networkType = res.networkType; // 返回网络类型WWAN（移动网络）, WIFI, NO（无网络）
  }
});
```

### 地理位置
#### 获取地理位置
```javascript
bainu.getLocation({
  'type': 0, //0:后台获取 或1:显示地图自己定位获取. 默认0
  'success': function(res) {
    var latitude = res.latitude,
        longitude = res.longitude;
  }
});
```
### 其它接口

#### 关闭当前窗口
```javascript
bainu.closeWindow();
```

### 附录
#### 1.错误代码
|code|desc|
|----|----|
|1   |failed|
|2   |params error|
|3   |resource not found|
|4   |api not supported|
|5   |need auth|
