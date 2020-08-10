# 1. 将相关的文件添加到工程中

![](https://upload-images.jianshu.io/upload_images/1648908-6548391a1a2796ba.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


#### (备注: 将deps文件删除引用。)

# 2. 添加所有的 .framework .a 文件 路径和头文件链接

(target->build setting -> search path ->Header Search Paths 中添加)
特殊添加一项目录 路径
`"$(SRCROOT)/Vigame/include"`
`"$(SRCROOT)/Vigame/tools"`
`"$(SRCROOT)/Vigame/deps/boost/include"`
`"$(SRCROOT)/Vigame/deps/curl/include"`
`"$(SRCROOT)/Vigame/deps/openssl/include"`
`"$(SRCROOT)/Vigame/deps/zlib/include"`

![添加sdk引用文件路径](https://upload-images.jianshu.io/upload_images/1648908-f0a533025fd7e71f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 3. 添加必要配置

1. 打开 Capabilities-> Keychain Sharing 获取设备唯一标识
2. 如果使用到谷歌广告 需要在info.plist 中添加广告广告key
    GADApplicationIdentifier : ca-app-pub-xxxxxxxxxxxxxxxxxxx
3. 如果项目中使用到Applovin广告在info.plist添加
    AppLovinSdkKey：Occxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
4. 在VigameLibrary.plist 中检测 company_appid 、company_prijid.
    以及相关的统计参数
6. 如果出海外包带有Facebook广告 需要在info.plist文件中添加Facebook 中相关的配置如下：

    ![fb配置](https://upload-images.jianshu.io/upload_images/2183351-96f3333dbc663e72.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/692)

7. 苹果新出的规定无论有没有使用到相机相册都得申请 权限
8. 游戏需要访问网络 需要有网络权限
9. 游戏第三方可能会用到定位，所有游戏info.plist添加NSLocationWhenInUseUsageDescription

# 4.添加系统支持库文件

target->build phases -> Link Binary With Libraries
`OpenGLES.framework`
`OpenAL.framework`
`iAd.framework`

`WebKit.framework`
`AVFoundation.framework`
`Accelerate.framework`
`MobileCoreServices.framework`
`CoreMotion.framework`

`CoreLocation.framework`
`CoreTelephony.framework`
`QuartzCore.framework`
`StoreKit.framework`
`AdSupport.framework`

`UIKit.framework`
`CoreFoundation.framework`
`CoreGraphics.framework`
`CoreMedia.framework`

`CoreBluetooth.framework`
`CoreText.framework`
`Security.framework`
`MediaPlayer.framework`
`CFNetwork.framework`
`libresolv.9.tbd`

`SystemConfiguration.framework`
`MessageUI.framework`
`JavaScriptCore.framework`
`AudioToolBox.framework`
`GLKit.framework`

`libz.tbd`
`libsqlite3.tbd`
`libiconv.tbd`
`libxml2.tbd`

`libc++.tbd`
`libz.1.1.3.tbd`
`libresolv.tbd`
`libsqlite3.0.tbd`

`//接入tapjoy需要下面库`
`PassKit.framework`
`MapKit.framework`
`ImageIO.framework`
`CoreData.framework`
# 5. 设备编译选项

1. `Other Linker Flags添加-ObjC`

![添加-ObjC](https://upload-images.jianshu.io/upload_images/2183351-f13ed84628186502.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/802)

2. Enable Bitcode 设置为NO
![设置Bitcode](https://upload-images.jianshu.io/upload_images/1648908-a8b9998bf49b9737.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


# 6 常见配置错误及解决方法
1. 在info.plist添加Google广告配置
    GADIsAdManagerApp：YES
不配置会出现崩溃
![未配置Google广告ID](https://upload-images.jianshu.io/upload_images/1648908-10b02a368fa1b206.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

添加方式如下：
![添加Google广告配置方式](https://upload-images.jianshu.io/upload_images/1648908-89539912206f3690.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

2. Google广告由于用xib自动布局，需要最低iOS9版本会报如下错误
![系统版本太低报错](https://upload-images.jianshu.io/upload_images/1648908-60441f51b86e81a6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

3. 如果报这个错误，修改游戏支持iOS版本最低为iOS9，参考下图
![设置iOS最低支持9](https://upload-images.jianshu.io/upload_images/1648908-e94d66e37fb2142e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

4. 缺少include文件配置报错
![缺少配置错误](https://upload-images.jianshu.io/upload_images/1648908-cf447bd6d14c7a26.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

解决如下：
![添加include引用](https://upload-images.jianshu.io/upload_images/1648908-933ec652fe73bbd7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

5. 缺少include中boost文件引入错误
![缺少boost引入错误](https://upload-images.jianshu.io/upload_images/1648908-5f38236e86b5d8e8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

解决如下：
![添加boost引用](https://upload-images.jianshu.io/upload_images/1648908-a4fce67fdbbba4ef.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



6. __weak typeof(self)wSelf = self报错：- A parameter list without types is only allowed in a function definition. A corresponding warning tells me that __weak only applies to Objective-C object or block pointer types;type here is 'int'

解决方案: Xcode－> Build Settings-> C Language Dialect修改配置，C99改为GNU99，C99不包含typeof

# 7. 确认c++ 编译项

![确认C++编译设置](https://upload-images.jianshu.io/upload_images/2183351-cb8d2dc8d55a615c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/854)

报错
![D970B31A66A9D0CC47F120ECE3C4F362.jpg](https://upload-images.jianshu.io/upload_images/1648908-dc7dca47a865b797.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)




# 8设置项目为自动内存管理

![设置ARC](https://upload-images.jianshu.io/upload_images/2183351-ff19eccac2bf6326.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/747)
![设置.png](https://upload-images.jianshu.io/upload_images/1648908-2b663a3c58c6a41b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
# 9接入微信配置（不接入忽略）
在info.plist文件添加
![添加微信白名单](https://upload-images.jianshu.io/upload_images/1648908-3b9d2adf2506a9a7.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
添加微信登陆URL Type
![添加微信URL Type](https://upload-images.jianshu.io/upload_images/1648908-7aa347ae8a163c04.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 10 SDK初始化工作

##### 1 导入头文件

在appDelegate文件中引入头文件`#import "IOSLoader.h"`

##### 2 调用初始化入口文件

```
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    [IOSLoader startLoaderLibrary];//初始化
KTMBannerViewController *bannerVC = [[KTMBannerViewController alloc] init];
    [_window.rootViewController presentViewController:bannerVC animated:NO completion:nil];//解决头条插屏关闭引起banner关闭
    return YES;
}
- (BOOL)application:(UIApplication *)application continueUserActivity:(NSUserActivity *)userActivity restorationHandler:(void (^)(NSArray<id<UIUserActivityRestoring>> * _Nullable))restorationHandler {
    
    if (userActivity.activityType == NSUserActivityTypeBrowsingWeb) {
        [IOSLoader application:application continueUserActivity:userActivity];
        return YES;
    }
    return NO;
}
- (void)applicationDidEnterBackground:(UIApplication *)application {
    [IOSLoader is_Active:false];//更新状态
    // Use this method to release shared resources, save user data, invalidate timers, and store enough application state information to restore your application to its current state in case it is terminated later.
    // If your application supports background execution, this method is called instead of applicationWillTerminate: when the user quits.
}
- (void)applicationWillEnterForeground:(UIApplication *)application {
   [IOSLoader isAwaken];
}

- (void)applicationDidBecomeActive:(UIApplication *)application {
    // Restart any tasks that were paused (or not yet started) while the application was inactive. If the application was previously in the background, optionally refresh the user interface.
    [IOSLoader is_Active:true]; //更新状态、开屏广告
     [IOSLoader openAwakenAd];
}
-(BOOL)application:(UIApplication *)app openURL:(NSURL *)url options:(NSDictionary<UIApplicationOpenURLOptionsKey,id> *)options {
    [IOSLoader isOpenURL];//解决唤醒广告在微信登录、充值频繁问题
    return YES;
}
```

# 11 代码调用

###### 1.通过广告位名称打开一个广告

```
#include "vigame_ad.h"

```

```
 // 打开一个横幅广告
  vigame::ad::ADManager::openAd("banner");

```

```
 // 打开关闭横幅广告
  vigame::ad::ADManager::closeAd("banner");

```

```
  // 打开一个插屏广告
  vigame::ad::ADManager::openAd("pause");

```

```
  // 打开一个开屏广告
  vigame::ad::ADManager::openAd("splash");

```

```
 /*监听 某个视频广告位是否加载成功*/
    vigame::ad::ADManager::addAdReadyChangedCallback("level_fail_mfzs", [=](bool isReady){
        if (isReady) {
            NSLog(@"level_fail_mfzs视频广告位 加载成功");
        }
    });
    //手动检测广告是否加载成功
    vigame::ad::ADManager::isAdReady("level_fail_mfzs");

```

```
// 打开一个视频广告 && 监听是否视频播放成功
  vigame::ad::ADManager::openAd("level_fail_mfzs",[=](vigame::ad::ADSourceItem* adSourceItem, int result){
      if (1 == result) {/*打开视频失败*/ }
      else if( 0 == result){/*打开视频成功*/ }
   });

```


###### 2.添加自定义统计事件

```
//统计
#import "vigame_tj.h"

```

```
      //方式1
      vigame::tj::DataTJManager::event("eventName");
     //方式2
      vigame::tj::DataTJManager::event("eventName","value");
        //方式3
        std::unordered_map<std::string, std::string> map;
        map.insert(std::make_pair("key1", "value1"));
        map.insert(std::make_pair("key2", "value2"));
        vigame::tj::DataTJManager::event("eventName", map);

```

```
// 导入支付头文件 发起支付
#import "pay/PayManager.h"
vigame::pay::PayManager::orderPay(154);

```


###### 3.微信登陆

```
// 在AppDelegate.m导入头文件
#import "WXSocialAgent.h"

 - (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
  [WXSocialAgent application:application DidFinishLaunchingWithOptions:launchOptions];
    
    return YES;
}

-(BOOL)application:(UIApplication *)app openURL:(NSURL *)url options:(NSDictionary<UIApplicationOpenURLOptionsKey,id> *)options {
     [[[WXSocialAgent alloc] init] application:app handleOpenURL:url];
    return YES;
}

```
```
// 导入头文件
#import "IOSLoader.h"
*注本接口已整合微信登录逻辑（已登录不再跳转到登录，游戏方不需再判断是否登录的情况）
[IOSLoader wxLogin:^(KTMLoginState code, NSString *returnMsg) {
            if (code == KTMLoginStateSuccess) {
                //调用获取用户信息接口
                [IOSLoader getWXUserInfo:^(NSDictionary *userInfo) {
                    NSLog(@"userInfo == %@",userInfo);
                }];
            }
        }];
```
```
userInfo数据格式如下，按需获取头像地址、openid、nickname等信息
{
    city = "";
    country = "";
    headimgurl = "http://thirdwx.qlogo.cn/mmopen/vi_32/DYAIOgq83epXrSKiaXoSxs38WdicmRuwQQjzk5Xnia9N3OAfGCGdIjRCWXic5mbm2vMDPkx96tLKAHVHKjdjcWYDgA/132";
    language = "zh_CN";
    nickname = "\U5149\U5934\U5f3a2\U53f7";
    openid = "oN702waxaalZ-1ycPtpfj0ALpbeg";
    privilege =     (
    );
    province = "";
    sex = 0;
    unionid = ofp95s1JJofVChMLEULM0H40iaEs;
}
```
* * *

## 交互流程

1. 我们提供一个测试包名和证书（已上线的项目#为了能出广告#）
2. 使用测试包名和证书出一个测试包--然后测试
3. 测试完成后 换正式包名和证书出正式包上传苹果商店
