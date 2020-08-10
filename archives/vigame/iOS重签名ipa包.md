##背景
我们上包流程是在测试完成后，再出正式包上传,每次都需要重新替换包名、参数等信息，及其麻烦、很容易没有修改完完，导致重出。引出了重签名的需求。在本文中，我将说明如何重新签署iOS应用程序，以便我们生成可部署到测试设备的IPA。
##代码签名
作为一项安全措施，Apple要求其信任的开发人员对在其设备上运行的所有代码进行数字签名。签名的工作方式类似于SSL证书在网站上的工作方式。我将使用这种类比，因为大多数人已经习惯了Web，并且这些概念（即使他们不知道其名称）也很有意义。

SSL证书和CA
要在您的网站上使用HTTPS，您需要具有数字证书。该证书包含有关您网站的信息，任何人都可以验证。您可以在网站上使用该证书来加密用户与服务器之间的通信。但是您的用户需要能够检查您所使用的证书的有效性。用户如何验证它？

首先，应该为您获得的证书由受信任的实体签名，该实体会验证您的身份。否则，想象一下，任何人都可以生成数字证书并假装成您的银行。为此，您的浏览器已预装了许多受信任的证书颁发机构（CA），并且您的浏览器将仅接受由任何这些CA签名（生成）的证书。您的浏览器通过查询CA来验证您网站上的证书。

要获得证书，首先需要生成一个CSR（证书签名请求），在其中指定所需的证书的详细信息。此CSR发送到CA。CA会验证您提交的数据是正确的，他们可能会要求您提供域所有权的证明，以及任何其他可以证明您身份的文档。他们验证了所有内容后，就会生成证书并将其发送回给您。您可以在您的网站上使用该证书，并且每个人都感到高兴和安全（高估）。

这是有关如何为您的网站获取SSL证书的一般想法，并且相同的概念也适用于iOS应用程序。
## 苹果是iOS应用程序的唯一有效CA

因此，如您现在可能想象的那样，或者如果您已经使用Apple创建了Developer帐户，则过程几乎是相同的。发送CSR后，您会从有效的证书颁发机构（在我们的情况下仅是Apple）获得证书。

有了证书后，就可以开始数字签名代码了。使用证书对数字进行代码签名将识别您创建的任何应用程序。Apple相信这是您的代码，因为您是唯一有权访问该证书的私钥的人。

希望您可以更好地了解证书的工作方式。我不想做太多细节，因为那里有很多好的资源可以解释这一点，而这篇文章不是要了解代码签名的工作原理。如果概念仍然不清楚，您可以查看[Apple的代码签名支持页面](https://developer.apple.com/support/code-signing/)或[代码签名指南](https://developer.apple.com/library/archive/documentation/Security/Conceptual/CodeSigningGuide/Introduction/Introduction.html)。

好的，让我们开始退出iOS应用程序。为此，我们需要一个IPA文件。
# 从IPA中提取应用程序捆绑包

我假设您拥有一个`.ipa`文件，无论您使用的是[frida-ios-dump](https://github.com/AloneMonkey/frida-ios-dump)还是任何其他工具，[都由](https://github.com/AloneMonkey/frida-ios-dump)您自己决定。但是您应该具有可以使用的IPA文件。如果您想继续学习，我们将使用[OWASP iGoat-Swift](https://github.com/owasp/igoat-swift)。您可以直接下载 [iGoat-Swift_v1.0.ipa ](https://github.com/OWASP/iGoat-Swift/blob/master/iGoat-Swift_v1.0.ipa)，也可以克隆项目并提取IPA文件。

我们将[`ios-deploy`](https://github.com/ios-control/ios-deploy)用来在应用程序的侧面加载应用程序。当前，如果您尝试按原样旁载IPA文件，则该文件将失败，因为配置文件未将您的设备作为运行应用程序的有效设备包括在内。这就是我们重新签署该应用程序的原因。现在让我们回到提取IPA。

可以解压缩IPA文件，将其视为ZIP文件。文件扩展名没有什么意义，但是为了清楚起见，让我们更改扩展名。

```
mv ios_resign.ipa ios_resign.zip
unzip -q ios_resign.zip -d ios_resign
```
##获取正确的配置文件
把需要重签名的mobileprovision文件改名为embedded.mobileprovision，保存到并ios_resign目录下，等下重签时替换需要使用；并使用PlistBuddy获取签名文件new_entitlements.plist

```
cp xxxxxx.mobileprovision  embedded.mobileprovision
security cms -D -i embedded.mobileprovision > new_provision.plist
/usr/libexec/PlistBuddy -x -c 'Print :Entitlements' new_provision.plist | tee new_entitlements.plist
```
##删除原来签名
需要把Payload/xxx.app下签名文件_CodeSignature以及embedded.mobileprovision删除，把上一步重命名embedded.mobileprovision放到Payload/xxx.app目录下。
```
rm -rf Payload/xxx.app/_CodeSignature
rm -rf Payload/xxx.app/embedded.mobileprovision
mv embedded.mobileprovision  Payload/xxx.app/
```
##修改bundleid
```
/usr/libexec/PlistBuddy -c 'Set :CFBundleIdentifier 'com.xx.xx.xx'' Payload/xxx.ipa/info.plist
```
##重签名
```
/usr/bin/codesign -fs "iPhone Distribution: xxxxxx" --entitlements new_entitlements.plist Payload/$app_name
/usr/bin/codesign -fs "iPhone Distribution: xxxxxx" --entitlements new_entitlements.plist Payload/xx.app/Frameworks/*
```
##重新生成ipa
```
zip -qr new.ipa  Payload/
```

##最后
由于很多工程不同，导致xxx.app名称不同，可以写个获取名称的脚本；证书名称也是可以实现脚本获取。



