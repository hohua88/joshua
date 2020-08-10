 在做完Xcode工程脚本化构建后[Xcodeproj工程配置脚本化](https://www.jianshu.com/u/59f75869cc90)，领导提出直接在unity工程配置，导出就可以直接运行，不必再接入Vigame的SDK。

在一番调研后，发现Unity自带的Xcode插件就可以实现这个需求。现记录如下

## 入口

PostProcessBuildAttribute (在Unity编译的最后加入了一个脚本调用的命令，会自动搜索Editor文件夹下的PostprocessBuildPlayer，并进行调用)。

```
[PostProcessBuildAttribute]
public static void OnPostProcessBuild(BuildTarget target, string pathToBuildProject)
{

}
```

## 修改Xcode工程

###1. 打开Xcode工程
```
string pbxProjectPath = PBXProject.GetPBXProjectPath( pathToBuildProject );
	// 修改工程设置
	PBXProject pbxProject = new PBXProject();
	pbxProject.ReadFromFile( pbxProjectPath );
```
###2. 修改工程配置
```
string target = project.TargetGuidByName(PBXProject.GetUnityTargetName());

        //project.SetBuildProperty(target, "CODE_SIGN_IDENTITY", "********");
        //project.SetBuildProperty(target, "PROVISIONING_PROFILE_SPECIFIER", "******");
        //project.SetBuildProperty(target, "DEVELOPMENT_TEAM", "******");

        //project.SetBuildProperty(target, "CODE_SIGNING_STYLE", "MANUAL");


        project.SetBuildProperty(target, "ENABLE_BITCODE", "NO");
        // 添加flag
        project.AddBuildProperty(target, "OTHER_LDFLAGS", "-ObjC");
```
###3. 添加系统库
```
string[] frameworks =
        {
            "libz.dylib",
            "libz.1.1.3.dylib",
            "libsqlite3.dylib",
            "libxml2.dylib",
            "libc++.dylib",
            "libstdc++.dylib",

            "CoreTelephony.framework",
            "SystemConfiguration.framework",
            "UIKit.framework",
            "Foundation.framework",
            "CoreGraphics.framework",
            "MobileCoreServices.framework",
            "StoreKit.framework",
            "CFNetwork.framework",
            "CoreData.framework",
            "Security.framework",
            "CoreLocation.framework",
            "ImageIO.framework",
            "CoreText.framework",
            "QuartzCore.framework",
            "AdSupport.framework",
            "Accelerate.framework",
            "CoreImage.framework",
            "SafariServices.framework",
            "Social.framework",
            "JavaScriptCore.framework",
            "AssetsLibrary.framework",
            "MediaPlayer.framework",
            "AddressBook.framework",
            "CoreMotion.framework",
            "AVFoundation.framework",
            "MessageUI.framework",
            "EventKit.framework",
            "EventKitUI.framework",
            "CoreAudio.framework",
        };

        foreach (var framework in frameworks)
        {
            project.AddFrameworkToProject(target, framework, false);
        }
```

###4. 添加三方库
以我们自己的SDK为例
```
 // Vigame
        project.AddBuildProperty(target, "LIBRARY_SEARCH_PATHS", "$(PROJECT_DIR)/Libraries/Wb/Vigame");
        project.AddBuildProperty(target, "HEADER_SEARCH_PATHS", "$(SRCROOT)/Libraries/Wb/Vigame/deps/curl/include/ios");
        project.AddBuildProperty(target, "HEADER_SEARCH_PATHS", "$(SRCROOT)/Libraries/Wb/Vigame/deps/openssl/include/ios");
        project.AddBuildProperty(target, "HEADER_SEARCH_PATHS", "$(SRCROOT)/Libraries/Wb/Vigame/deps/boost/include");
        project.AddBuildProperty(target, "HEADER_SEARCH_PATHS", "$(SRCROOT)/Libraries/Wb/Vigame/deps/zlib/include/linux");
        project.AddBuildProperty(target, "HEADER_SEARCH_PATHS", "$(SRCROOT)/Libraries/Wb/Vigame/include");
```

###4. 修改plist文件
```
 PlistElementArray urlArray = null;
        if (!rootDict.values.ContainsKey("CFBundleURLTypes"))
            urlArray = rootDict.CreateArray("CFBundleURLTypes");
        else
            urlArray = rootDict.values["CFBundleURLTypes"].AsArray();
        var urlTypeDict = urlArray.AddDict();
        urlTypeDict.SetString("CFBundleTypeRole", "Editor");
        urlTypeDict.SetString("CFBundleURLName", "weixin");
        var urlScheme = urlTypeDict.CreateArray("CFBundleURLSchemes");
        urlScheme.AddString("wx12333444");

        if (!rootDict.values.ContainsKey("LSApplicationQueriesSchemes"))
            urlArray = rootDict.CreateArray("LSApplicationQueriesSchemes");
        else
            urlArray = rootDict.values["LSApplicationQueriesSchemes"].AsArray();
      
        urlArray.AddString("weixin");
        urlArray.AddString("wechat");

        rootDict.SetString("NSCameraUsageDescription", "");
        rootDict.SetString("NSPhotoLibraryUsageDescription", "");


        if (rootDict.values.ContainsKey("NSAppTransportSecurity"))
            rootDict.values.Remove("NSAppTransportSecurity");
        PlistElementDict urlDict = rootDict.CreateDict("NSAppTransportSecurity");
        urlDict.SetBoolean("NSAllowsArbitraryLoads", true);
```

最后由于我们c++库中有.hpp文件，导出工程时会丢失，导致工程运行报错，故暂时未用此方式。
```
'boost/property_tree/xml_parser.hpp' file not found
```
后续重构SDK时，不依赖c++的化，这种方式也是极简的。
参考：

[配置Xcode工程](https://blog.csdn.net/blog_lee/article/details/52400535)

[PBXProject.AddFile Code Examples](https://csharp.hotexamples.com/examples/UnityEditor.iOS.Xcode/PBXProject/AddFile/php-pbxproject-addfile-method-examples.html)

[配置Xcode工程](https://blog.csdn.net/zp288105109a/article/details/86628379)
