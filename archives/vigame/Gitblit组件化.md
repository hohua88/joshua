1.在Gitblit上创建一个自己的远程私有索引库，用来存放私有框架的podspec文件
![image.png](https://upload-images.jianshu.io/upload_images/1648908-cfadea6a41f51ab7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

2. 创建本地的私有索引库文件夹，并与远程私有索引库进行关联

2.1添加本地私有索引库并与远程私有库
```
$ pod repo add IOSMavenSpec http://wy@dnsdk.vimedia.cn:8080/r/IOSMavenSpec.git
```
![image.png](https://upload-images.jianshu.io/upload_images/1648908-cebc87b46a751874.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

2.2 再次查看本地已存在的索引库 
```
 pod repo
```
3 在Gitblit创建一个用来存放项目基础组件的仓库IOSMaven
参见第一步

4 快速创建模板工程(模板来自github)

4.1 快速创建模板测试工程 在/Users/eddy/Desktop/IOSMaven路径下
```
  $ cd /Users/eddy/

  $ pod lib create KTM-iOS-SDK
```
4.2 本地验证podspec
```
pod lib lint --allow-warnings  --use-libraries --verbose
```
验证不过的话；可以添加--verbose查看详细错误信息

4.3 提交本地代码
```
$ git status
$ git add .
$ git commit -m 'first commit'
$ git remote add origin '[http://wy@dnsdk.vimedia.cn:8080/r/KTM-iOS-SDK.git](http://wy@dnsdk.vimedia.cn:8080/r/IOSMavenSpec.git')
 (将本地库与远程代码仓库进行关联)
$ git push origin master  (提交到远程仓库)
$ git tag '0.1.0' (要与IOSMaven.podspec文件中的tag值保持一致)
$ git push --tags(将tag提交到远程)
```

4.4 验证远程是否正确
```
pod spec lint --allow-warnings --use-libraries --verbose（验证远程是否正确） 
```
5 将podspec文件提交到本地的私有索引库
```
$ pod repo push IOSMavenSpec IOSMaven.podspec --allow-warnings
```

5 组件化
5.1、抽离一个基础组件Tools
```
 spec.default_subspecs =  'Common'  #默认组件
  
  spec.subspec 'Common' do |ss|
    ss.ios.deployment_target = '9.0'
    ss.vendored_frameworks = 'Common/*.framework'
  end
```
5.2、模块组件
```

spec.subspec 'Ads' do |ss|
  ss.dependency 'KTMSDK/Common'
   ss.subspec 'ByteDance' do |hl| #头条广告
      hl.vendored_frameworks = "Ads/ByteDance/*.framework"
      hl.resource = "Ads/ByteDance/*.bundle"
     end
end
```
加载xib资源需要制作bundle [组件化framework加载xib](https://www.jianshu.com/writer#/notebooks/38500610/notes/56527770)

6 引用远程库
```
source 'http://wy@dnsdk.vimedia.cn:8080/r/IOSMavenSpec.git' #远程私有库地址
source 'https://cdn.cocoapods.org/' #公有库地址

platform :ios, '9.0'
target 'Unity-iPhone' do
  
sdkVersion='1.0.0'
####--Ads---###
pod 'KTMSDK/Ads/ByteDance',sdkVersion
pod 'Bytedance-UnionAD', '2.8.0.1'
pod 'KTMSDK/Ads/IronSource',sdkVersion
pod 'KTMSDK/Ads/Admob',sdkVersion
pod 'KTMSDK/Ads/GDT',sdkVersion
pod 'KTMSDK/Ads/Unity',sdkVersion
pod 'KTMSDK/Ads/Mintegral',sdkVersion
pod 'KTMSDK/Ads/Facebook',sdkVersion
pod 'KTMSDK/Ads/Applovin',sdkVersion
pod 'KTMSDK/Ads/KTMAd',sdkVersion
pod 'KTMSDK/Ads/Vungle',sdkVersion
####--Analysis---###
pod 'KTMSDK/Analysis/Umeng',sdkVersion
pod 'KTMSDK/Analysis/TrackingIO',sdkVersion
pod 'KTMSDK/Analysis/ByteDance',sdkVersion
pod 'KTMSDK/Analysis/DataEye',sdkVersion
pod 'KTMSDK/Analysis/Appsflyer',sdkVersion
pod 'KTMSDK/Analysis/Facebook',sdkVersion
pod 'KTMSDK/Analysis/Adjust',sdkVersion
pod 'KTMSDK/Analysis/Tenjin',sdkVersion
pod 'KTMSDK/Analysis/Google',sdkVersion
####--Social---###
pod 'KTMSDK/Social/wechat',sdkVersion
pod 'KTMSDK/Social/facebook',sdkVersion
pod 'KTMSDK/Social/apple',sdkVersion
####--Extension---###
pod 'KTMSDK/Extension/notice',sdkVersion
pod 'KTMSDK/Extension/activity',sdkVersion
pod 'KTMSDK/Extension/auth',sdkVersion
####--other---###
pod 'KTMSDK/Bugly',sdkVersion
pod 'KTMSDK/IAP',sdkVersion
```
注：ironsource聚合广告需单独导入

    ```
    #IronSource 聚合广告按需添加
    pod 'IronSourceAdMobAdapter','4.3.10.1'
    pod 'IronSourceFacebookAdapter','4.3.14.0'
    pod 'IronSourceUnityAdsAdapter','4.3.0.1'
    pod 'IronSourceTikTokAdapter','4.1.2.7'
    pod 'IronSourceAppLovinAdapter','4.3.10.1'
    pod 'IronSourceMintegralAdapter','4.3.1.0'
    ```

遇到问题：
1、不能同时存在同一个SDK的不同版本(ironsource聚合中的第三方广告单独pod)
2、依赖海外的需要放到本地（比如FBAudienceNe
twork），目前有外网是pod方式引入
cocopods搭私有库报错- ERROR | [iOS] xcodebuild: Returned an unsuccessful exit code.（库存在多个，比如irons聚合中有unity，单独的unity仅保留一个）
