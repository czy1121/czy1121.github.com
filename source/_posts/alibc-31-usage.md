---
title: 阿里百川电商SDK(3.1)接入与使用
date: 2016-12-20 
tags: android
---



## 接入

#### 接入准备

参考   
http://baichuan.taobao.com/docs/doc.htm?spm=a3c0d.7629140.0.0.L4aY7V&treeId=129&articleId=105645&docType=1

1. 申请百川无线应用
2. 下载安全图片
3. 开通业务产品的使用权限
4. 使用优惠券组件

#### 添加安全图片

将安全图片放在/res/drawable目录下，如果已经有安全图片，就替换。

#### Gradle配置

``` gradle
repositories { 
    maven { url "http://repo.baichuan-android.taobao.com/content/groups/BaichuanRepositories/" }
} 
dependencies {
    // 授权登陆 (MemberSDK)
    compile 'com.ali.auth.sdk:alibabauth_core:1.2.4@jar'
    compile 'com.ali.auth.sdk:alibabauth_ui:1.2.4@aar'
    compile 'com.ali.auth.sdk:alibabauth_ext:1.2.4@jar'
    
    // fastjson
    compile 'com.alibaba:fastjson:1.2.9'
    // 基础电商组件(AlibcTradeSDK, AlibcTrade, AlibcLogin)
    // 包含打开detail,淘客分润,jsbridge注入等功能
    compile 'com.alibaba.sdk.android:alibc_trade_sdk:3.1.1.20@aar'
    // 集成支付宝(可选)
    compile 'com.alibaba.alipay:alipaySingle:20160825@jar'
    // 组件可用性统计
    compile 'com.alibaba.mtl:app-monitor-sdk:2.5.1_for_bc'
    
    // 基础安全组件
    compile 'com.taobao.android:securityguardaar3:5.1.96@aar'
    // 网关
    compile 'com.taobao.android:mtopsdk_allinone_open:1.3.0@jar'
    // 手机淘宝与三方app之间的往返跳转
    compile 'com.taobao.android:alibc_applink:2.0.0.9@jar'
    // deviceID，主要用于计算设备的uttid， 设备唯一标识
    compile 'com.taobao.android:utdid4all:1.1.5'
}
```

#### Proguard 规则 

```
-keepattributes Signature
-keep class sun.misc.Unsafe { ; }
-keep class com.taobao.* {*;}
-keep class com.alibaba.** {*;}
-keep class com.alipay.** {*;}
-dontwarn com.taobao.**
-dontwarn com.alibaba.**
-dontwarn com.alipay.**
-keep class com.ut.** {*;}
-dontwarn com.ut.**
-keep class com.ta.** {*;}
-dontwarn com.ta.**
-keep class org.json.** {*;}
-keep class com.ali.auth.** {*;}
```

## 使用电商SDK(3.1)

#### 初始化

在应用的入口方法（比如Application的onCreate）中初始化百川SDK

``` java
AlibcTradeSDK.asyncInit(this, new AlibcTradeInitCallback() {
    @Override
    public void onSuccess() {
        // 初始化成功，设置相关的全局配置参数
        
        // 是否使用支付宝
        AlibcTradeSDK.setShouldUseAlipay(true);
        
        // 设置是否使用同步淘客打点
        AlibcTradeSDK.setSyncForTaoke(true);
        
        // 是否走强制H5的逻辑，为true时全部页面均为H5打开
        AlibcTradeSDK.setForceH5(true);
        
        // 设置全局淘客参数，方便开发者用同一个淘客参数，不需要在show接口重复传入
        AlibcTradeSDK.setTaokeParams(taokeParams)
        
        // 设置渠道信息(如果有渠道专享价，需要设置)
        AlibcTradeSDK.setChannel(typeName, channelName)

        // ...
    }

    @Override
    public void onFailure(int code, String msg) {
        //初始化失败，可以根据code和msg判断失败原因，详情参见错误说明
    }
});
```
#### 资源销毁

在使用完成后，可以调用destroy方法，释放百川相应的资源引用

``` java
AlibcTradeSDK.destroy();
```

#### 显示电商页面

实例化页面参数(必填)

``` java
// 商品详情，支持itemId和openItemId的商品，必填，不允许为null
AlibcBasePage page = new AlibcDetailPage(itemId);
 
// 店铺，店铺id，支持明文id
AlibcBasePage page = new AlibcShopPage(shopId);
 
// 添加购物车，支持itemId和openItemId的商品，必填，不允许为null；
AlibcBasePage page = new AlibcAddCartPage(itemId)
 
// 我的订单
// status   默认跳转页面(0:全部, 1:待付款, 2:待发货, 3:待收货, 4:待评价)
// allOrder 为 true 显示所有订单，为false只显示通过当前app下单的订单 
AlibcBasePage page = new AlibcMyOrdersPage(status, allOrder);
 
// 我的购物车
AlibcBasePage page = new AlibcMyCartsPage();
     
// URL
AlibcBasePage page = new AlibcPage(taokeUrl);
```

设置参数并并使用自定义webview打开页面

> [注意]：当传入webviewClient，并重载shouldOverrideUrlLoading方法时，遇到淘系链接情况下(即访问淘宝、天猫、登录、购物车等页面时)，该方法返回值要为false，否则可能会出现业务流程错误问题。

``` java

// 页面打开方式
AlibcShowParams params = new AlibcShowParams(OpenType.Native, isNeedPush);
// 淘宝客参数
AlibcTaokeParams taoke = new AlibcTaokeParams(pid, unionId, subId);
// 提供给三方传递配置参数
Map<String, String> extras = new HashMap<>(); 

// activity, page, callback 为必填
AlibcTrade.show(activity, webView, webViewClient, webChromeClient, page, params, taoke, extras, new AlibcTradeCallback() {
    @Override
    public void onTradeSuccess(TradeResult tradeResult) {
        //打开电商组件，用户操作中成功信息回调。tradeResult：成功信息（结果类型：加购，支付；支付结果）
    }
 
    @Override
    public void onFailure(int code, String msg) {
        //打开电商组件，用户操作中错误信息回调。code：错误码；msg：错误信息
    }
}); 
```

使用默认webview打开页面

``` java
AlibcTrade.show(context, page, params, taoke, extras, callback);
```

## 登陆授权(1.2.4)

电商SDK已经集成登陆授权  
在电商SDK初始化时，会自动初始化登陆授权 

#### 自动触发登陆

在电商SDK打开的页面中，在需要时会自动触发调用登陆授权SDK  

#### 手动触发登陆

通过 `AlibcLogin.showLogin` 方法可以手动调起登陆

``` java
AlibcLogin.getInstance().showLogin(activity, new AlibcLoginCallback() {
    @Override
    public void onSuccess() { 
        // 
    }
    @Override
    public void onFailure(int code, String message) {

    }
});
```

#### 接收登陆/登出结果

为了正常接收登陆/登出的结果，需要重写传入的 activity 的 onActivityResult 方法

``` java
@Override
protected void onActivityResult(int requestCode, int resultCode, Intent data) {   
    CallbackContext.onActivityResult(this, requestCode, resultCode, data);
}
```

#### 设置全局登陆回调

目前电商SDK里的 AlibcLogin 还未提供该功能    
只能通过 MemberSDK 获取 LoginService 可以设置全局登陆回调

``` java
LoginService service = (LoginService)MemberSDK.getService(LoginService.class)
service.setLoginCallback(new LoginCallback(){
    @Override
    public void onSuccess(Session session) { 
        // 
    }
    @Override
    public void onFailure(int code, String message) {

    }
})
```

#### 错误码
 

``` java
public class KernelMessageConstants {
    public static final int GENERIC_SYSTEM_ERROR = 10010;
    public static final int SERVER_REQUEST_FAILED = 15;
    public static final int SERVICE_NOT_AVAILABLE_ERROR = 17; 
}

public class SystemMessageConstants extends KernelMessageConstants {
    public static final int JS_BRIDGE_MODULE_NOT_FOUND = 10000;
    public static final int USER_CANCEL_CODE = 10003;
    public static final int H5_LOGIN_FAILURE = 10101;
    public static final int TAOBAO_CANCEL_CODE = 10004;
    public static final int TAOBAO_ERROR_CODE = 10005;
    public static final int JS_BRIDGE_METHOD_NOT_FOUND = 951;
    public static final int JS_BRIDGE_ANNOTATION_NOT_PRESENT = 952;
    public static final int NET_WORK_ERROR = 10099;
    public static final int NPE_ERROR = 10098; 
}

```

## 一些坑

#### 在Dialog中打开页面

由于 dialog 会给 context 包上一层 ContextThemeWrapper, 而与 dialog 交互时 sdk 可能会从其中的 view 获取 context 然后转成 activity 这时就会出现异常导致崩溃。  

一个解决办法是使用 `LayoutInflater.from(activity)` 来 inflate 对话框的布局

#### 【bug】在授权或支付页面返回(failure)时会把传入的activity关闭(finish)

打开页面时必需传入一个activity，传入的activity被关闭，这通常不是期望的效果

通过重载传入的activity.finish方法可以临时解决

目前官方文档上使用的sdk版本 3.1.0.7 有这个问题，更新到 3.1.1.20 已经没有此问题
 

#### 【bug】AlibcTrade 的默认 WebViewClient.shouldInterceptRequest 方法的逻辑错误

alibc_trade_sdk 包的 `com.alibaba.baichuan.android.trade.c.b.b` 类 
它的 shouldInterceptRequest 方法如下

打开页面时我们传入的WebViewClient会被这个类代理

``` java
public WebResourceResponse shouldInterceptRequest(WebView var1, String var2) {
  return VERSION.SDK_INT > 23 && this.a != null && this.a.get() != null?((WebViewClient)this.a.get()).shouldInterceptRequest(var1, var2):super.shouldInterceptRequest(var1, var2);
}
```

这个方法是用于API LEVEL < 21 的，然而 VERSION.SDK_INT > 23 这个错误的判断导致原本应该被执行的代码未执行

4.x 的设备都会因此bug产生各种问题


## 参考

阿里百川 SDK 官方文档
http://baichuan.taobao.com/docs/doc.htm?spm=a3c0d.7629140.0.0.fZiJXS&treeId=129&articleId=105645&docType=1

3.1SDK 常见问题补充
https://baichuan.bbs.taobao.com/detail.html?spm=a3c0d.7971500.0.0.OkQc3n&postId=7215938

常见错误码
http://baichuan.taobao.com/docs/doc.htm?spm=a3c0d.7629140.0.0.SjH1Xf&treeId=129&articleId=104308&docType=1

SG error:XXX错误码
http://baichuan.taobao.com/docs/doc.htm?spm=a3c0d.7629140.0.0.AjAmY2&treeId=129&articleId=103222&docType=1

客户端SDK常见问题
http://baichuan.taobao.com/docs/doc.htm?spm=a3c0d.7629140.0.0.LQayyc&treeId=129&articleId=102553&docType=1