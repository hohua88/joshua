##背景介绍
苹果在审核拒约时给出了以下信息：
ITMS-90809: Deprecated API Usage – Apple will stop accepting submissions of apps that use UIWebView APIs . See https://developer.apple.com/documentation/uikit/uiwebview
for more information.

但是项目内并未使用 UIWebView API，尝试使用 Unity 构建一个空工程上传到 TestFlight 或者 QuickSDK
进行预先检查，发现依然存在 UIWebView API 调用。

搜索后发现 Unity 已在 2017.4, 2018.4, 2019.2, 2019.3, 2020.1 中修复，但是旧版本如 Unity 5.6 就不管了：

[Unity Issue Tracker – [iOS] Apple throws Deprecated API Usage warning for using UIWebView when submitting Builds to the App Store Connect](https://www.colabug.com/goto/aHR0cHM6Ly9pc3N1ZXRyYWNrZXIudW5pdHkzZC5jb20vaXNzdWVzL2lvcy1hcHBsZS10aHJvd3MtZGVwcmVjYXRlZC1hcGktdXNhZ2Utd2FybmluZy1mb3ItdXNpbmctdWl3ZWJ2aWV3LXdoZW4tc3VibWl0dGluZy1idWlsZHMtdG8tdGhlLWFwcC1zdG9yZS1jb25uZWN0)

##解决方案
1、首先将以下内容保存为 URLUtility.mm
```
#include <iostream>
#import <UIKit/UIKit.h>
using namespace std;
namespace core {
template <class type>
class StringStorageDefault {};
template <class type,class type2>
class basic_string {
public:
char *c_str(void);
};
}
void OpenURLInGame(core::basic_string< char,core::StringStorageDefault<char> > const&arg){}
void OpenURL(core::basic_string<char,core::StringStorageDefault<char> >const&arg){
const void *arg2= &arg;
UIApplication *app = [UIApplication sharedApplication];
NSString *urlStr = [NSString stringWithUTF8String:(char *)arg2];
NSURL *url = [NSURL URLWithString:urlStr];
[app openURL:url];
}
void OpenURL(std::string const&arg){
UIApplication *app = [UIApplication sharedApplication];
NSString *urlStr = [NSString stringWithUTF8String:arg.c_str()];
NSURL *url = [NSURL URLWithString:urlStr];
[app openURL:url];
}
```
2、查看libiPhone-lib.a支持那些架构
```
file libiPhone-lib.a
libiPhone-lib.a: Mach-O universal binary with 3 architectures: [arm_v7:current ar archive] [arm64] [arm_v7s]
libiPhone-lib.a (for architecture armv7):	current ar archive
libiPhone-lib.a (for architecture arm64):	current ar archive
libiPhone-lib.a (for architecture armv7s):	current ar archive
```
3、编译对应架构的URLUtility.o
```
//arm64
clang -c URLUtility.mm -arch arm64 -isysroot /Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/Developer/SDKs/iPhoneOS.sdk
//armv7s
clang -c URLUtility.mm  -arch armv7s -isysroot /Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/Developer/SDKs/iPhoneOS.sdk
//armv7
clang -c URLUtility.mm  -arch armv7 -isysroot /Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/Developer/SDKs/iPhoneOS.sdk
```
⚠️不能同时编译3种架构，fouz不能移除
```
ar -d /Users/eddy/Desktop/lib/libiPhone-lib.a URLUtility.o
ar: /Users/eddy/Desktop/lib/libiPhone-lib.a is a fat file (use libtool(1) or lipo(1) and ar(1) on it)
ar: /Users/eddy/Desktop/lib/libiPhone-lib.a: Inappropriate file type or format
```
注意： -isysroot
指定的 SDK 路径一定是当前工程使用的 Xcode 版本，特别是当机器上存在多个 Xcode 版本时要注意。

4、分离lipo libiPhone-lib.a中每个架构的.a
```
 lipo libiPhone-lib.a -thin armv7 -output libiPhone-lib_armv7.a
```
5、替换对应分离出来架构的.o
```
//删除原有的 URLUtility.o
ar -d libiPhone-lib_armv7.a URLUtility.o
//在文件最后增加 URLUtility.o
ar -q libiPhone-lib_armv7.a URLUtility.o
```
6、合并替换后的.a
```
//合并armv7和armv7s
ipo -create libiPhone-lib_arm64.a libiPhone-lib_armv7.a libiPhone-lib_armv7s.a -output libiPhone-lib_new.a 
//合并arm64和上面的合并
lipo -create libiPhone-lib_new.a  libiPhone-lib_arm64.a  -output libiPhone-lib_w.a 
```
7、用合并后的替换之前的.a,解决问题

*   [libiPhone-lib.a去掉WebViewController – 简书](https://www.colabug.com/goto/aHR0cHM6Ly93d3cuamlhbnNodS5jb20vcC9hZDM1NGYxMWQ4NTI=)
*   [Unity iOS 删除 UIWebView](https://www.colabug.com/2020/0113/6839873/amp/)
