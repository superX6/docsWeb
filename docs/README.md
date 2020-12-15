# 天翼云JS-SDK使用说明

## 概述

天翼云``JS-SDK``是天翼云应用开发平台面向应用开发者提供的基于天翼云的网页开发工具包。

通过使用该``JS-SDK``，网页开发者可以调用天翼云应用的原生能力，例如**打开新窗口**、**存储媒介资源**等手机系统能力，同时也可以**获取用户信息**、**获取公共参数**等天翼云特有的能力。

## JS-SDK使用步骤

### 步骤一：引入ctyun-jssdk

> 可以通过以下两种方式引入ctyun-jssdk

#### 通过npm安装

目前托管在小组的私有仓库中，只对内网开放，安装前请确认已经将仓库地址设置到正确的地址（需询问sdk开发者提供）

```
npm set registry http://xxx.xxx.xx.xx:xxxx
 
npm install ctyun-jssdk --save
```

#### 在页面引入JS文件

```html
<script src="https://www.app.ctyun.cn/jssdk/ctyun-jssdk.min.js"></script>
```

### 步骤二：调用ctyun-jssdk

#### 模块引入

如果使用了webpack等支持模块引入的打包工具，可以通过``import``或者``require``的方式引入。

``` javascript
// 引入全部
import ctyun from 'ctyun-jssdk'
// 或者
let ctyun = require('ctyun-jssdk')

// 局部引入
import { closeWindow } from 'ctyun-jssdk'
```

#### 使用全局变量

ctyun-jssdk定义了一个全局变量``ctyun``，可以通过如下方式使用：

```javascript
window.ctyun.onReady()
```

###  步骤三：权限验证

**远端应用**，需要通过``auth``接口注入权限验证。*本地应用可以跳过这个步骤*。

> 远端应用所有需要使用JS-SDK的页面必须先注入配置信息，否则将无法调用。
> 
> 需要在onReady回调成功后再进行权限验证配置

```javascript
ctyun.onReady(function() {
  ctyun.auth(
    {
      appId: "ctyun",
      timestamp: Date.now(),
      nonceStr: "this is nonce string",
      signature: "this is signature"
    },
    res => {
      if (!res.errCode) {
        console.log("ctyun auth validate successfully!");
      }
    }
  );
});
```

鉴权请求参数说明:
参数名 | 是否必选 | 类型 | 说明
---|---|---|---
appId | Y | String | appId，当前是ctyun
timestamp | Y | Number | 当前时间戳，Date.now()
nonceStr | Y | String | 随机字符串
signature | Y | String | 签名

#### 签名算法（该算法尚未开放，试验中）

签名生成规则如下：参与签名的字段包括应用私钥``key``，``url``（当前网页的``URL``，不包含``#``及其后面部分） ，使用``URL``键值对的格式（即``url=value1&key=value``）拼接成字符串``string1``。这里需要注意的是所有参数名均为小写字符。对``string1``作*sha1加密*，字段名和字段值都采用原始值，不进行``URL``转义。

即``signature=sha1(string1)``。 示例：

```
url=http://api.ctyun.com?params=value&key=hw3ehrP8q3KLyPb2TdOaYjwZINZ5Op
```
其中 `hw3ehrP8q3KLyPb2TdOaYjwZINZ5Op` 为该应用的私钥

对``string1``进行*sha1签名*，得到``signature``： *0f9de62fce790f9a083d5c99e95740ceb90c27ed*

#### 注意事项

- 签名用的``url``必须是调用``JS-SDK``接口页面的完整``URL``。

- 出于安全考虑，开发者必须在服务器端实现签名的逻辑。

## 接口调用说明

所有接口可以用过``ctyun``对象来调用，除了每个接口本身需要传的参数之外。还有传一个通用的回调函数callback，回调函数返回一个固定数据结构的对象：

```
{
    errorCode: 0,
    message: 'success',
    data: {} 
}
```

参数名 | 类型 | 说明
---|---|---
errorCode | Number | 错误码，0表示成功，其他失败
message | String | 提示信息，成功为success
data | String或Object | 客户端返回数据

## 预备接口

### onReady

在调用``ctyun-jssdk``其他接口之前，需要先调用``onReady``接口，这个接口主要是建立前端与客户端的桥梁。进入页面后，只需调用一次该接口。

### 调用方式

```javascript
ctyun.onReady(function() {
  console.log('成功建立连接')
  // 远端应用在此鉴权
})

// 调用一次后，可调用其他接口
ctyun.closeWindow()
```


## 权限接口

### auth

若属于远端应用，开发者在使用``ctyun-jssdk``其他接口前，需要先通过``auth``接口进行权限验证。

#### 调用方式

```javascript
ctyun.onReady(function() {
  ctyun.auth(
    {
      appId: "ctyun",
      timestamp: Date.now(),
      nonceStr: "this is nonce string",
      signature: "this is signature"
    },
    res => {
      if (!res.errCode) {
        console.log("ctyun auth validate successfully!");
      }
    }
  );
});
```

## 请求接口

### request

http请求代理，前端目前基本都是代理给app，让app请求来获取后台数据，但是涉及媒体资源的上传，是前端直接请求发送给后台的，后续可以扩展让客户端支持。

**这里需要注意，我们本地开发是无法通过APP代理请求的，只有部署到APP才可以，那么需要我们在开发时候通过axios模拟APP的代理来实现本地请求。**

#### 调用方式

```javascript
ctyun.request(config, function(res) {
    // 获取后台返回的数据
})
```
请求配置对象参数说明:
参数名 | 是否必选 | 类型 | 说明
---|---|---|---
url | Y | String | 请求url
method | Y | String | 请求方法
params | N | Object | 请求query参数
data | N | Object | 请求数据体
headers | N | Object | 请求头
timeout | N | Number | 请求超时
cache | N | Boolean | 告知是否后台从缓存读取数据，默认是false，不从缓存读取数据

#### 成功回调参数

```javascript
{
  errorCode: 0,
  data: {}, // 后台返回的数据
  message: 'success'
}
```

## 窗体接口

### closeWindow

关闭``webview``窗口

#### 调用方式

```javascript
ctyun.closeWindow()
```

### openUrl

新窗口打开网页。

#### 调用方式

```javascript
ctyun.openUrl(openUrl, title, auth = false)
```

参数名 | 是否必选 | 类型 | 说明
---|---|---|---
openUrl | Y | String | 网页url
title | N | String | 网页title，显示到客户端导航栏的标题中
auth | N | Boolean | 是否需要登录才能新窗口打开网页

### openLogin

打开登录页面。

#### 调用方式

```javascript
ctyun.openLogin(function(res) {
    // 自定义操作
})
```

### openRegister

打开注册页面。

#### 调用方式

```javascript
ctyun.openRegister(function(res) {
    // 自定义操作
})
```
### openHome

打开天翼云首页。

#### 调用方式

```javascript
ctyun.openHome(function(res) {
    // 自定义操作
})
```

### openProduct

打开产品页。

#### 调用方式

```javascript
ctyun.openProduct(function(res) {
    // 自定义操作
})
```

### openOrderList

打开订单列表页。

#### 调用方式

```javascript
ctyun.openOrderList(function(res) {
    // 自定义操作
})
```

### openPayment

打开订单支付页。

#### 调用方式

```javascript
ctyun.openPayment(orderId)
```
参数名 | 是否必选 | 类型 | 说明
---|---|---|---
orderId | Y | String | 订单id

### openOrder

打开订单页。

#### 调用方式

```javascript
ctyun.openOrder(orderId, from)
```
参数名 | 是否必选 | 类型 | 说明
---|---|---|---
orderId | Y | String | 订单id
from | N | String | 为order打开的是确认订单页面，其他值打开的是订单详情页面，默认是order

### openBalance

打开账户余额页面。

#### 调用方式

```javascript
ctyun.openBalance(function(res) {
    // 自定义操作
})
```

### openHelpCenterSearchPage

打开帮助中心页面。

#### 调用方式

```javascript
ctyun.openHelpCenterSearchPage(function(res) {
    // 自定义操作
})
```

### openAccount

打开账户管理页面。

#### 调用方式

```javascript
ctyun.openAccount(function(res) {
    // 自定义操作
})
```

### openModal

打开指定modal弹层。

#### 调用方式

```javascript
ctyun.openModal(config，function(res) {
    // 自定义操作
})
```
config对象包含的参数说明：

参数名 | 是否必选 | 类型 | 说明
---|---|---|---
type | Y | Number | 弹层类型，-998表示打开实名认证弹层，引导进入实名认证

## 导航接口

### onGoBack

用户点击导航条返回按钮时，触发此事件。

#### 调用方式

```javascript
ctyun.onGoBack(function(res) {
  // 自定义操作
})
```

### onRightTopBarClick

用户点击导航条右上角图标时，触发此事件。

#### 调用方式

```javascript
ctyun.onRightTopBarClick(function(res) {
  // 自定义操作
})
```

## 生命周期函数

### onViewWillAppear

页面显示，触发此事件。

#### 调用方式

```javascript
ctyun.onViewWillAppear(function(res) {
  // 自定义操作
})
```

### onViewWillDisappear

页面消失，触发此事件。

#### 调用方式

```javascript
ctyun.onViewWillDisappear(function(res) {
  // 自定义操作
})
```

### modifyTitle

修改导航栏的标题。

#### 调用方式

```javascript
ctyun.modifyTitle(title)
```
参数名 | 是否必选 | 类型 | 说明
---|---|---|---
title | Y | String | title名称

### modifyTitleBar

设置导航栏右侧的icon。

#### 调用方式

```javascript
ctyun.modifyTitleBar(config, function(res) {
    // 自定义操作
})
```
config对象包含的参数说明：

参数名 | 是否必选 | 类型 | 说明
---|---|---|---
type | Y | String | 导航类型，text: 文本类型，share: 分享类型，后续会越来越多的导航类型
info | Y | Object | 公共参数，value: 显示的文本，icon: 显示的icon，tips: 提示语  
params | Y | Object | 特殊参数，可以根据不同类型接口定制不同参数

这里针对特殊参数params说明一下，目前只有导航类型为share配置了特殊参数：
- share
    - shareTitle：分享的标题；
    - shareDes：分享的描述；
    - shareUrl：分享的url。


## 设备接口

### getPublicParams

获取设备相关信息，同时也是获取公共参数，公共参数需要在每次请求都带上的，否则请求会失败。

#### 调用方式

```javascript
ctyun.getPublicParams(function(res) {
    // 获取公共参数
})
```

#### 成功回调参数

```javascript
{
  errorCode: 0,
  data: {
    osType: '', 
    mainVersion: '',
    ......
  },
  message: 'success'
}
```

参数名 | 类型 | 说明
---|---|---
device | String | 设备类型
deviceId | String | 设备id
loginToken | String | 用户登录的token
mainVersion | Number | 客户端版本号
osType | String | 操作系统类型
osVersion | String | 操作系统版本
producer | String | 生产商
comParam_curTime | String | 当前时间戳
comParam_seqCode | String | 随机码
comParam_signature | String | 签名

### screenCapture

截屏生成图片。

#### 调用方式

```javascript
ctyun.screenCapture(config, function(res) {
    // 自定义操作
})
```
config对象包含的参数说明:

参数名 | 是否必选 | 类型 | 说明
---|---|---|---
type | Y | String | 标识截屏类型，目前只有一种类型，就是workorder，因为只有在工单页面中使用


## 用户相关接口

### getToken

获取用户token，以此来判断用户是否登录。

#### 调用方式

```javascript
ctyun.getToken(function(res) {
    // 获取token
})
```

#### 成功回调参数

```javascript
{
  errorCode: 0,
  data: '', // token
  message: 'success'
}
```

### getUserInfo

获取用户信息。

#### 调用方式

```javascript
ctyun.getUserInfo(function(res) {
    // 获取用户信息
})
```

#### 成功回调参数

```javascript
{
  errorCode: 0,
  data: {}, // 用户信息
  message: 'success'
}
```

**需要注意，data字段会随着业务增长出现扩增，最新字段可以以移动端登录过后返回的字段为准。**

data字段参数说明：

参数名 | 类型 | 说明
---|---|---
token | String | 用户token
userId | String | 用户id
accountId | String | 账户id
loginEmail | String | 登录邮箱
requestDate | Number | 请求时间戳
accountType | String | 账户类型，个人用户：1，企业用户：2
productNbr | String | 
proUserInnerId | String | 
isHavePrivilege | Boolean | 是否有待同意的协议
expires | Number | token过期时间戳
accountMd | String | 账户md5加密串
refer | String | 来源，目前有wap、wechat、app
innerUserId | String | 内部用户id
rootUserId | String | 
isRoot | Number | 
userInfo | Object | 用户信息

userInfo字段参数说明：

参数名 | 类型 | 说明
---|---|---
accountType | String | 账户类型，个人用户：1，企业用户：2
userId | String | 用户id
accountId | String | 账户id
loginEmail | String | 登录邮箱
auditStatus | String | 实名认证状态，未认证：0，认证中：1，认证失败：2，一认证：3，人工审核：4
auditMsg | String | 实名认证提示信息
telephone | String | 固定电话
address | String | 地址
postNo | String | 邮政编码
mobilephone | String | 手机
email | String | 邮箱
name | String | 姓名
subType | String | 
channel | String | 
productNbr | Number | 
isPostpaid | Boolean | 
province | String | 省
city | String | 市
county | String | 县区
sex | String | 性别，女：0，男：1
cardType | String | 证件类型，个人用户--身份证：1，护照：2，军官证：3，台胞证：4；企业用户--工商营业执照：1，组织机构代码：2，事业法人：3，社团法人：4，军队代号：5
cardNo | String | 证件号码
idCardPathBean | String | 

### getPrivateData

获取业务数据，主要用来获取订单支付后的订单id，根据订单id查询支付情况。

#### 调用方式

```javascript
ctyun.getPrivateData(function(res) {
    // 获取订单id
})
```

#### 成功回调参数

```javascript
{
  errorCode: 0,
  data: {
      orderId: '' 
  }, 
  message: 'success'
}
```
参数名 | 类型 | 说明
---|---|---
orderId | String | 订单id



## 媒介接口

### saveMedia

保存媒介资源到本地，图片、音频、视频等。

#### 调用方式

```javascript
ctyun.saveMedia(config, function(res) {
    // 自定义操作
})
```
config对象包含的参数说明：


参数名 | 是否必选 | 类型 | 说明
---|---|---|---
type | Y | String | 媒介资源类型，image | audio | video
url | Y | String | 媒介资源地址


## 开发/测试环境说明
- 构建模拟协议：因为协议需要依赖APP的宿主环境，只有部署到APP才可以调用相关协议，本地开发就需要我们构建模拟的协议进行，类似于请求后台接口，后台接口还没给到前端，前端需要mock数据一样；
- 构建代理请求：我们大部分http请求都是代理到APP去完成的，本地开发不存在APP的宿主环境，也需要我们去构建模拟的代理请求，可以通过axios去构建。