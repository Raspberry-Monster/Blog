---
title: "MAUI NotificationListenerService 实现记录"
description: 咕咕咕
date: 2026-02-17T22:30:00Z
image: 
math: 
license: CC BY-NC-SA 4.0
hidden: false
comments: true
draft: false
categories:
    - Work
---
## 写在前面
这个项目是我高三时期写的一个小玩意，一直没有动力写Blog。但是这次[Spotify更新API规则](https://developer.spotify.com/blog/2026-02-06-update-on-developer-access-and-platform-security)极大地影响了Lyricify, 同时激起了我寻找不需要Spotify API获取目前正在播放的曲目的好奇心。便将当时的一些心得记录一下。
## 获取MediaSession
[NotificationListenerService](https://developer.android.com/reference/android/service/notification/NotificationListenerService)是Android于 API level 18 时期添加的Service。通过NotificationListenerService 可以获取软件发布的通知。包括我们要讲的Spotify的MediaSession通知。具体实现可参考仓库中的[文件](https://github.com/Storyteller-Studios/NotificationListenerServiceDemo-dotNet/blob/main/NotificationListener-MAUI/NotificationListenerService.cs)。

需要注意的是：在Android 12及以下的系统中，我们使用`notification?.Extras?.GetParcelable(Notification.ExtraMediaSession);`。在Android 13 及以上的系统中，此方法被弃用, 需要使用`Class.FromType(typeof(MediaSession.Token));`获取MediaSession.Token的Java Class, 并且在GetParcelable中传入，如下: `(IParcelable?)notification?.Extras?.GetParcelable(Notification.ExtraMediaSession, MediaSessionClass);`

这个Service需要获取通知访问权，可以通过`NotificationManagerCompat.GetEnabledListenerPackages(this).Contains(PackageName!);`获取是否已经取得权限。如果没有取得权限，可以使用`Intent("android.settings.ACTION_NOTIFICATION_LISTENER_SETTINGS");`跳转至通知授权界面。同时，这个Service会被系统自动拉起，所以在部分陆版UI(如ColorOS， HyperOS)上需要允许App自启动。

这个方法有一个小小的缺陷，就是假如通知在授权前就已经被发布，在下一次通知被发布之前，你是无法从通知渠道获取MediaSession的。不过假如你已经获取了通知权限，你可以通过MediaSessionService直接获取当前活动的MediaSession。代码如下：
```
var manager = (MediaSessionManager)GetSystemService(MediaSessionService)!;
var activeSpotifySession = manager.GetActiveSessions(NotificationListenerServiceComponentName).Where(t => t.PackageName == "com.spotify.music").FirstOrDefault();
```

获取了MediaSession实际上就已经成功75%，接下来就是耳熟能详的RegisterCallback和获取TransportControls。可以参考[Demo](https://github.com/Storyteller-Studios/NotificationListenerServiceDemo-dotNet/blob/main/NotificationListener-MAUI/MainActivity.cs)中的 OnMediaSessionCreated 方法。

## 接收来自Spotify App的Broadcast
这个操作需要先去Spotify的设置-播放-设备广播状态将开关打开。这样之后才可以收到Spotify的Broadcast。

大部分操作可以参考[Spotify提供的示例](https://developer.spotify.com/documentation/android/tutorials/android-media-notifications)。MAUI版本已经在[此处](https://github.com/Storyteller-Studios/NotificationListenerServiceDemo-dotNet/blob/main/NotificationListener-MAUI/BroadcastReceiver.cs)提供。别忘记RegisterReceiver就可以了。

## 为什么我重启App之后没办法使用NotificationListener接收消息?
在Android 7及以上系统，可以通过`NotificationListenerService.RequestRebind(NotificationListenerServiceComponentName);`重新绑定 NotificationListenerService

而在那之下的系统，可以通过
```
PackageManager!.SetComponentEnabledSetting(NotificationListenerServiceComponentName!, Android.Content.PM.ComponentEnabledState.Disabled, Android.Content.PM.ComponentEnableOption.DontKillApp);
PackageManager.SetComponentEnabledSetting(NotificationListenerServiceComponentName!, Android.Content.PM.ComponentEnabledState.Enabled, Android.Content.PM.ComponentEnableOption.DontKillApp);
```
进行重新绑定

不过我得吐槽两句，这个RequestRebind方法为啥在很多中文文章里都没提到？我看到很多文章都是用PackageManager这个魔法操作进行Rebind。还有个解答在OnListenerDisconnected中用RequestRebind，OnListenerDisconnected方法是你撤销了通知使用权的时候才会被调用。结果你在这个方法里调用需要通知使用权的方法，您没事吧？
## 写在后面
这是我在很久没更新Blog之后第一次写Blog，有什么不好的地方麻烦多多包涵。See you next time!
