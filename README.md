# wilddog-location-weapp
----

野狗实时定位小程序SDK

## 配置
微信小程序平台不同于web平台，它是一个封闭的平台。对于所有的资源使用微信都有严格的限制。所以，在微信小程序中使用野狗之前需要做一些额外的配置，除此之外，你可以像在其他开放平台一样使用野狗。实时定位 SDK 是基于 sync 开发的 SDK ，因此需要在项目中同时引用[野狗小程序SDK](https://github.com/WildDogTeam/wilddog-weapp)和野狗实时定位小程序SDK。

#### 配置域名白名单

首先，如果想使用野狗数据同步功能和登陆认证的功能你需要在微信小程序管理后台配置域名白名单。路径是 设置>开发设置>服务器配置。由于微信给开发者设置了很多限制，每月只能修改3次，所以修改的时候一定要慎重。为了简化配置，你自需要增加以下域名到白名单：

* socket合法域名 `wss://s-dalwx-nss-1.wilddogio.com`
* request合法域名 `https://auth.wilddog.com`、`https://trace.wilddog.com`、`https://restapi.amap.com`（如不需要使用高德定位SDK，则不加该域名）


#### 配置AppId和AppSecret

如果你想使用微信的用户账号来认证(`auth.signInWeapp(opt_callback)`)，那么你还需要在野狗的管理后台配置微信的AppId 和AppSecret。
你可以在 设置>开发设置>开发者ID 中找到AppId 和AppSecret。之后在野狗后台中打开微信小程序登陆授权开关，并且将Appid 和AppSecret传入即可。

## 将野狗sdk引入到微信小程序中

1. 将wilddog-weapp-all.js 和 wilddog-location.js 直接放到微信小程序的项目中
2. 使用commonjs引入

```js
var wilddog = require('wilddog-weapp-all');
var RealtimeLocation = require('wilddog-location');

wilddog.regService('location', function(app) {
    if (app === null) {
        throw new Error('application not initialized!Please call wilddog.initializeApp first');
    };
    return new RealtimeLocation(app);
});
wilddog.Location = RealtimeLocation;

```
3. 初始化

```js
var config = {
    syncURL: 'https://<WD-APPID>.wilddogio.com',
    authDomain: '<WD-APPID>.wilddog.com'
}
wilddog.initializeApp(config)

var wildLocation = wilddog.location();
```

## API(以下API为野狗sdk在小程序平台独有的API，另外兼容web端的所有其他API)

微信小程序平台与一般的开放平台不同之一是它有默认的用户，所以我们提供了一个可以使用一个api进行auth的方法：

#### auth.signInWeapp(opt_callback)

* opt_callback: function(err,user) 可选回调函数，认证成功后被调用，如果认证过程一切正常`err`为`null`,user是一个包含用户`id`和`provider`等信息的对象。否则 `err` 是一个Error对象，user为null

return Promise 对象

```js
var config = {
    syncURL: 'https://<WD-APPID>.wilddogio.com',
    authDomain: '<WD-APPID>.wilddog.com'
}
wilddog.initializeApp(config)
wilddog.auth().signInWeapp(function(err,user){
    // do your logic
})

//或者使用Promise

wilddog.auth().signInWeapp().then(function(user){

}).catch(function(err){

})

```
#### location.AMapWXLocationProvider(type,num,AMapWX)

创建一个使用高德小程序 SDK 来获取定位信息的实例，这个实例用于 startTracingPosition() 和 starRecordingPath()。
AMapWX 是高德小程序 SDK 的实例对象。


例子

app.js

```js
var config = {
    syncURL: 'https://<WD-APPID>.wilddogio.com',
    authDomain: '<WD-APPID.wilddog.com>'
}
App({
    onLaunch:function () {
        wilddog.initializeApp(config)
        this.wildLocation = wilddog.location();
        this.AMapWX = AMapWX;
    },
    onHide:function(){
        this.todoRef.goOffline();
    },
    onShow:function(){
        this.todoRef.goOnline();
    }
})
```

** 注意： **
小程序 1.3+ 版本 onHide 触发后 ws 会自动断连，必须等到 onShow 触发时才可以重连。在此期间 wilddog 重连机制无效，因此需要用户在 onHide 时手动断连，onShow 时手动连接。

page (假设是 index)

index.js
```js
var app = getApp()
Page({
    ...
    onLoad: function () {
        var provider = app.wildLocation.AMapWXLocationProvider('timeInterval', 5000, app.AMapWX);
        app.wildLocation.startTracingPosition('key', provider);
    }
    ...
})
```

#### location.WXLocationProvider(type,num)

创建一个使用微信小程序内部方法来获取定位信息的实例，这个实例用于 startTracingPosition() 和 starRecordingPath()。
WXLocationProvider() 不需要引用其它获取定位信息的SDK，使用小程序内部自带方法获取。

例子

app.js

```js
var config = {
    syncURL: 'https://<WD-APPID>.wilddogio.com',
    authDomain: '<WD-APPID.wilddog.com>'
}
App({
    onLaunch:function () {
        wilddog.initializeApp(config)
        this.wildLocation = wilddog.location();
    },
    onHide:function(){
        this.todoRef.goOffline();
    },
    onShow:function(){
        this.todoRef.goOnline();
    }
})
```

** 注意： **
小程序 1.3+ 版本 onHide 触发后 ws 会自动断连，必须等到 onShow 触发时才可以重连。在此期间 wilddog 重连机制无效，因此需要用户在 onHide 时手动断连，onShow 时手动连接。

page (假设是 index)

index.js
```js
var app = getApp()
Page({
    ...
    onLoad: function () {
        var provider = app.wildLocation.WXLocationProvider('timeInterval', 5000);
        app.wildLocation.startTracingPosition('key', provider);
    }
    ...
})
```


完整的API请参考 [Location API](https://docs.wilddog.com/location/Web/api/AMapLocationProvider.html) 、[Sync API](https://docs.wilddog.com/api/sync/web/api.html) 和 [Auth API](https://docs.wilddog.com/api/auth/web/Auth.html)

更多信息请移步[野狗官方文档](https://docs.wilddog.com)
