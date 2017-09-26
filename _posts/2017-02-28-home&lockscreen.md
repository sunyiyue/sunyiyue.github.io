---
layout: post
title: 区分锁屏与home键
date: 2017-02-28 14:05:00.000000000 +09:00
---





### 开始
今天有被问到如何区分app是通过按了home键使程序进入后台还是因为锁屏呢
### iOS7之前
```
- (void)applicationDidEnterBackground:(UIApplication *)application
{

    UIApplicationState state = [[UIApplication sharedApplication] applicationState];
    
    if (state  == UIApplicationStateInactive) {//---说明是锁屏
      
    }else if(state  == UIApplicationStateBackground){//---说明进入后台
        NSLog(@"background");
    }
    
}

```
### iOS7之后
方法一：
当锁屏时，iOS系统会触发一个通知，即com.apple.springboard.lockcomplete，当接收到这个通知时，我们设置一个标志位，即kDisplayStatusLocked，接着在applicationDidEnterBackground方法中（以下简称后台方法），判断该标志位的值。

```
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    
    CFNotificationCenterAddObserver(CFNotificationCenterGetDarwinNotifyCenter(),
                                    NULL,
                                    displayStatusChanged,
                                    CFSTR("com.apple.springboard.lockcomplete"),
                                    NULL,
                                    CFNotificationSuspensionBehaviorDeliverImmediately);
    return YES;
}

static void displayStatusChanged(CFNotificationCenterRef center,
                                void *observer,
                                CFStringRef name,
                                const void *object,
                                CFDictionaryRef userInfo) {
    if (name ==CFSTR("com.apple.springboard.lockcomplete")) {
        [[NSUserDefaults standardUserDefaults] setBool:YES forKey:@"kDisplayStatusLocked"];
        [[NSUserDefaults standardUserDefaults] synchronize];
    }
}

- (void)applicationDidEnterBackground:(UIApplication *)application {
    UIApplicationState state = [[UIApplication sharedApplication] applicationState];
    if (state ==UIApplicationStateInactive) {
        NSLog(@"按了锁屏键");
    }
    else if (state == UIApplicationStateBackground) {
        if (![[NSUserDefaults standardUserDefaults] boolForKey:@"kDisplayStatusLocked"]) {
            NSLog(@"按了home键，或者跳转到另一个app");
        }
        else {
            NSLog(@"按了锁屏键");
        }
    }

}
                                    
```
方法二：当屏幕亮度为0时，为锁屏，否则为Home事件

```
UIApplicationState state = [[UIApplication sharedApplication] applicationState];
if (state == UIApplicationStateInactive) {
    //iOS6锁屏事件
    NSLog(@"Sent to background by locking screen");
} else if (state == UIApplicationStateBackground) {
    CGFloat screenBrightness = [[UIScreen mainScreen] brightness];
    NSLog(@"Screen brightness: %f", screenBrightness);
    if (screenBrightness > 0.0) {
        //iOS6&iOS7 Home事件
        NSLog(@"Sent to background by home button/switching to other app");
    } else {
        //iOS7锁屏事件
        NSLog(@"Sent to background by locking screen");
    }
}

```

   

  