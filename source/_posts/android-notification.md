---
title: Android Notification 详解
date: 2016/12/28
---

## 简单用法

**创建通知**

创建通知至少包含 **小图标、标题、内容** 才能显示

``` java 
NotificationCompat.Builder builder = new NotificationCompat.Builder(this)
        .setSmallIcon(R.drawable.notification_icon)
        .setContentTitle("My notification")
        .setContentText("Hello World!"); 
```

**发送通知**

``` java 
NotificationManager manager = (NotificationManager) mContext.getSystemService(Context.NOTIFICATION_SERVICE);
manager.notify(notifyId, builder.build());
```

**取消通知**

``` java 
NotificationManager manager = (NotificationManager) mContext.getSystemService(Context.NOTIFICATION_SERVICE);

// 取消notifyId关联的通知
manager.cancel(notifyId);

// 取消所有通知
manager.cancelAll();
```

## 基本信息

**标题/内容/小图标**

必要信息

``` java  
// 标题
builder.setContentTitle("这是通知标题");
// 内容
builder.setContentText("这是通知内容");
// 小图标
builder.setSmallIcon(R.mipmap.ic_small_icon);
```

**大图标**

大图标，未设置时使用小图标代替

``` java  
// 大图标 
Bitmap bitmap = BitmapFactory.decodeResource(context.getResources(), R.mipmap.ic_large_icon);
builder.setLargeIcon(bitmap);
```

**次要内容** 

setContentInfo 在 api 24 被废弃，不再显示，用 setSubText 代替 
setNumber 在 api 24 被废弃，不再显示    

``` java
// 次要内容
builder.setSubText("这是通知的次要内容");
// 附加文本
builder.setContentInfo("INFO");
// 附加数字，等价于 setContentInfo, 为了显示效果用一个不同的字体尺寸显示数字
builder.setNumber(123);
```

**时间** 

setShowWhen 在 api 17 被添加
setChronometerCountDown 在 api 24 添加

``` java
// 设置时间
builder.setWhen(System.currentTimeMillis());
// 设置是否显示时间
builder.setShowWhen(false);
// 设置是否显示时钟表示时间(count up)
builder.setUsesChronometer(false);
// 设置时钟是否为倒计时(count down)
builder.setChronometerCountDown(false);
```



**进度条**
 
api 24 之前，使用了 setSubText() 后会隐藏进度条   
api 24 之后，setSubText() 不再影响进度条

``` java
int max = 100; // 进度最大值
int progress = 50;  // 当前进度
int indeterminate = false; // 是否是不明确的进度条
builder.setProgress(max, progress, indeterminate);
```


**状态栏摘要(ticker)** 

在 api 21 后不再显示，仅用于辅助服务。 

``` java   
builder.setTicker("this is ticker"); 
```


## 标志符(Flags)


|Flag|描述| 
|:---|:---| 
|Notification.FLAG_SHOW_LIGHTS|是否使用呼吸灯提醒|
|Notification.FLAG_INSISTENT|持续提醒(声音/振动)直到用户响应(点击/取消)|
|Notification.FLAG_ONLY_ALERT_ONCE|提醒(铃声/震动/滚动通知摘要)只执行一次|
|Notification.FLAG_ONGOING_EVENT|正在进行中通知|
|Notification.FLAG_AUTO_CANCEL|用户点击通知后自动取消|
|Notification.FLAG_NO_CLEAR|用户无法取消|
|Notification.FLAG_FOREGROUND_SERVICE|表示正在运行的服务| 


**设置提醒只执行一次(FLAG_ONLY_ALERT_ONCE)**

设置提醒只执行一次

```java 
builder.setOnlyAlertOnce(true);
```

**设置自动取消(FLAG_AUTO_CANCEL)**

需要同时设置了 setContentIntent() 才有效

``` java 
builder.setAutoCancel(true);
builder.setContentIntent(pendingIntent);
```

**设置通知为进行中(FLAG_ONGOING_EVENT)**

通常表示一个用户积极参与的后台任务，比如电话，下载，播放音乐等   

用户不能取消，效果类似`FLAG_NO_CLEAR` 
用户点击通知且设置了自动取消时会被删除

``` java  
builder.setOngoing(true); 
```

**设置 `FLAG_INSISTENT/FLAG_NO_CLEAR`**

NotificationCompat.Builder 未提供设置方法，只能通过 Notification

``` java
Notification n = builder.build();
// 持续提醒直到用户响应
n.flags |= Notification.FLAG_INSISTENT;
// 用户无法取消
n.flags |= Notification.FLAG_NO_CLEAR;
manager.notify(notifyId, n);
```


## 优先级 

|优先级|描述| 
|:---|:---| 
|Notification.PRIORITY_MAX|重要而紧急的通知，通知用户这个事件是时间上紧迫的或者需要立即处理的。|
|Notification.PRIORITY_HIGH|高优先级用于重要的通信内容，例如短消息或者聊天，这些都是对用户来说比较有兴趣的|
|Notification.PRIORITY_DEFAULT|默认优先级用于没有特殊优先级分类的通知|
|Notification.PRIORITY_LOW|低优先级可以通知用户但又不是很紧急的事件。只显示状态栏图标|
|Notification.PRIORITY_MIN|用于后台消息 (例如天气或者位置信息)。只有用户下拉通知抽屉才能看到内容|

设置优先级

``` java 
builder.setPriority(Notification.PRIORITY_HIGH);
```


## 提醒通知到达

提供了 **铃声/振动/呼吸灯** 三种提醒方式，可以使用一种或同时使用多种

**使用默认提醒**

|FLAG|描述| 
|:---|:---| 
|Notification.DEFAULT_SOUND|添加默认声音提醒|
|Notification.DEFAULT_VIBRATE|添加默认震动提醒|
|Notification.DEFAULT_LIGHTS|添加默认呼吸灯提醒|
|Notification.DEFAULT_ALL|同时添加以上三种默认提醒|


``` java 
// 添加默认声音提醒
builder.setDefaults(Notification.DEFAULT_SOUND);

// 添加默认呼吸灯提醒，自动添加FLAG_SHOW_LIGHTS
builder.setDefaults(Notification.DEFAULT_LIGHTS);
```

**添加自定义提醒**

``` java 
// 添加自定义声音提醒
builder.setSound(Uri.parse("path/to/sound"));

// 添加自定义震动提醒
// 延迟200ms后震动300ms，再延迟400ms后震动500ms
long[] pattern = new long[]{200,300,400,500}; 
builder.setVibrate(pattern);

// 添加自定义呼吸灯提醒，自动添加FLAG_SHOW_LIGHTS
int argb = 0xffff0000;  // led灯光颜色
int onMs = 300;         // led亮灯持续时间
int offMs = 100;        // led熄灯持续时间
builder.setLights(argb, onMs, offMs);
```

## 事件

**点击内容事件**

``` java  
int flags = PendingIntent.FLAG_UPDATE_CURRENT;
Intent intent = new Intent(this, ResultActivity.class);
PendingIntent pi = PendingIntent.getActivity(this, 0, intent, flags);
builder.setContentIntent(pi);
```

**取消通知事件**

通知被用户取消时发送(清除所有，右滑删除)  
“自动取消(`FLAG_AUTO_CANCEL`)”不会产生该事件

``` java    
Intent intent = new Intent(ACTION);
intent.putExtra("op", op);
PendingIntent pi = PendingIntent.getBroadcast(this, 0, intent, 0);
builder.setDeleteIntent(pi);
```

**全屏通知事件**

响应紧急事件(比如来电)

``` java    
Intent intent = new Intent(ACTION);
intent.putExtra("op", op);
PendingIntent pi = PendingIntent.getBroadcast(this, 0, intent, 0);
builder.setFullScreenIntent(pi, true);
```
 

## 浮动通知

Android 5.0(API 21)开始支持浮动通知

设备处于**活动状态(设备未锁定且其屏幕已打开)**时，可显示浮动通知

满足下列条件之一可触发浮动通知：

- 用户的 Activity 处于全屏模式中（应用使用 fullScreenIntent）
- 通知具有**较高的优先级**(PRIORITY_MAX 或 PRIORITY_HIGH)并**使用铃声或振动**

注：国内各种ROM可能由于各种原因导致浮动通知不能显示


## 锁定屏幕通知

Android 5.0(API 21)开始，通知可显示在锁定屏幕上。   
可使用此功能提供媒体播放控件以及其他常用操作。      
用户可以通过“设置”选择是否将通知显示在锁定屏幕上。  

**设置可见性**

- `VISIBILITY_PUBLIC` 显示通知的完整内容。
- `VISIBILITY_SECRET` 不会在锁定屏幕上显示此通知的任何部分。
- `VISIBILITY_PRIVATE` 显示通知图标和内容标题等基本信息，但是隐藏通知的完整内容。

设置 `VISIBILITY_PRIVATE` 后，还可以通过 `setPublicVersion() ` 提供其中隐藏了某些详细信息的替换版本通知内容。


## 扩展布局

Android 4.1(API 16) 开始支持扩展布局，下拉抽屉中最顶部的一条通知的扩展布局自动展开  
Android 7.0(API 24) 开始每条通知都可以单独展开

### 操作按钮 
 
api 19 开始支持添加操作按钮，每个展开的通知可包含最多3个操作按钮

``` java  
// 添加操作按钮
builder.addAction(icon1, title1, pendingIntent1);
builder.addAction(icon2, title2, pendingIntent2);
```

### 样式 

使用`Builder.setStyle()`设置扩展布局样式

**多行文本通知**

``` java
Notification notif = new Notification.Builder(mContext)
     .setContentTitle("New mail from " + sender.toString())
     .setContentText(subject)
     .setSmallIcon(R.drawable.new_mail)
     .setLargeIcon(aBitmap)
     .setStyle(new Notification.BigTextStyle().bigText(aVeryLongString))
     .build();
```

**大图通知** 

``` java
Notification notif = new Notification.Builder(mContext)
     .setContentTitle("New photo from " + sender.toString())
     .setContentText(subject)
     .setSmallIcon(R.drawable.new_post)
     .setLargeIcon(aBitmap)
     .setStyle(new Notification.BigPictureStyle().bigPicture(aBigBitmap))
     .build();
```
 
**收件箱通知**

最多显示5行消息

``` java
Notification notif = new Notification.Builder(mContext)
     .setContentTitle("5 New mails from " + sender.toString())
     .setContentText(subject)
     .setSmallIcon(R.drawable.new_mail)
     .setLargeIcon(aBitmap)
     .setStyle(new Notification.InboxStyle()
         .addLine(str1)
         .addLine(str2)
         .setContentTitle("")
         .setSummaryText("+3 more"))
     .build();
```

## 保留 Activity 返回栈

### 常规 Activity

默认情况下，从通知启动一个Activity，按返回键会回到主屏幕。

但某些时候有按返回键仍然留在当前应用的需求，这就要用到TaskStackBuilder了。

1、在manifest中定义Activity的关系

``` xml
<activity
    android:name=".ResultActivity"
    android:parentActivityName=".MainActivity">
</activity>
```

2、构建带返回栈的PendingIntent并发送通知

``` java
// 构建返回栈
TaskStackBuilder tsb = TaskStackBuilder.create(this);
tsb.addParentStack(ResultActivity.class); 
tsb.addNextIntent(new Intent(this, ResultActivity.class));

// 构建包含返回栈的 PendingIntent
PendingIntent pi = tsb.getPendingIntent(0, PendingIntent.FLAG_UPDATE_CURRENT);

// 构建通知
NotificationCompat.Builder builder = new NotificationCompat.Builder(this);
...
builder.setContentIntent(pi);

// 发送通知
NotificationManager manager = (NotificationManager) getSystemService(Context.NOTIFICATION_SERVICE);
manager.notify(notifyId, builder.build());
```


### 特殊Activity

默认情况下，从通知启动的Activity会在近期任务列表里出现。

如果不需要在近期任务里显示，则需要做以下操作:

1、在manifest中定义Activity

``` xml
<activity
    android:name=".ResultActivity"
    android:launchMode="singleTask"
    android:taskAffinity=""
    android:excludeFromRecents="true">
</activity>
```

2、构建PendingIntent并发送通知

``` java
// 创建 Intent
Intent intent = new Intent(this, ResultActivity.class);
intent.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK | Intent.FLAG_ACTIVITY_CLEAR_TASK);

// 创建 PendingIntent
PendingIntent pi = PendingIntent.getActivity(this, 0, intent, PendingIntent.FLAG_UPDATE_CURRENT);

// 构建通知
NotificationCompat.Builder builder = new NotificationCompat.Builder(this);
...
builder.setContentIntent(pi);

// 发送通知
NotificationManager manager = (NotificationManager) getSystemService(Context.NOTIFICATION_SERVICE);
manager.notify(notifyId, builder.build());
```  

## Demo 

https://github.com/czy1121/NotificationDemo 

![notify](https://github.com/czy1121/NotificationDemo/raw/master/screenshots/screenshot.png)
  
 
api 19 (android 4.4)

![api19_big_text](https://github.com/czy1121/NotificationDemo/raw/master/screenshots/api19_big_text.png) ![api19_big_picture](https://github.com/czy1121/NotificationDemo/raw/master/screenshots/api19_big_picture.png)
  
api 21 (android 5.0)

![api21_big_text](https://github.com/czy1121/NotificationDemo/raw/master/screenshots/api21_big_text.png) ![api19_big_picture](https://github.com/czy1121/NotificationDemo/raw/master/screenshots/api21_big_picture.png)
![api21_progress_bar](https://github.com/czy1121/NotificationDemo/raw/master/screenshots/api21_progress_bar.png) ![api21_heads_up](https://github.com/czy1121/NotificationDemo/raw/master/screenshots/api21_heads_up.png)
    
api 24 (android 7.0)

![api24_basic](https://github.com/czy1121/NotificationDemo/raw/master/screenshots/api24_basic.png) ![api24_progress_bar](https://github.com/czy1121/NotificationDemo/raw/master/screenshots/api24_progress_bar.png)
![api24_heads_up](https://github.com/czy1121/NotificationDemo/raw/master/screenshots/api24_heads_up.png)


## 参考

https://developer.android.com/reference/android/app/Notification.Builder.html
https://developer.android.com/reference/android/support/v4/app/NotificationCompat.Builder.html
https://developer.android.com/guide/topics/ui/notifiers/notifications.html
 

全面了解Android Notification 
http://www.jianshu.com/p/22e27a639787
