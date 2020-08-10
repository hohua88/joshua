## 前言
从入职就一直想实现全自动操作，缘由是我们SDK构成复杂，包括依赖的c++库，接入需要配置很多项，最开始要求是导入依赖的库实现脚本配置，（最开始没有用CocoaPods,需要手动添加依赖），简单做了了解之后，没有发现好的方式。就一直搁浅了，最近重构支持pod方式，解决了依赖库的添加问题，但还是手动添加我们自己库的配置。在做了一番调研后，发现Xcodeproj这个gem可以完美解决我们的问题，遂引入工程。在做完自动配置后，就想把所有的步骤都脚本化，这才有了这篇记录。

## 1、添加依赖的gem
脚本化所依赖的gem，fir-cli 是用来上传[官方工具 fir-cli 使用说明](http://blog.betaqr.com/fir_cli/),[Xcodeproj](https://github.com/CocoaPods/Xcodeproj)主要用来修改target配置,[plist](https://github.com/patsplat/plist)用来修改info.plist, [mysql2](https://github.com/brianmario/mysql2)用来连接数据库，获取数据。
```
sudo gem install fir-cli  
sudo gem install xcodeproj
sudo gem install plist
sudo gem install mysql2 -- --with-cflags=\"-I/usr/local/opt/openssl/include\" --with-ldflags=\"-L/usr/local/opt/openssl/lib\"
```
## 2、clone Vigame模块
我已把所有的脚本及SDK放到Vigame

*注：需cd到工程目录,记录ruby执行shell命令*
>cmd = "git clone https://github.com/hohua88/Vigame.git"
system (cmd)
```
cd /Users/xxxx/xxx
git clone https://github.com/hohua88/Vigame.git

```

## 3、修改工程配置
使用Xcodeproj实现的，参考我自己写的文档[Xcodeproj工程配置脚本化](https://www.jianshu.com/p/67ab2522daa7)

## 根据服务器数据修改plist及生成Podfile
具体实现参见我的文章[修改plist](https://www.jianshu.com/p/5ce0df6fa302)
生成Podfile根据数据库的配置信息，生成对应的命令，我的根据服务器返回写了一个hash，具体如下

```
as_hash = {"headlineId" => "pod 'KTMSDK/Analysis/ByteDance',sdkVersion\n", "ReYunAppKey" => "pod 'KTMSDK/Analysis/TrackingIO',sdkVersion\n", "umengId" => "pod 'KTMSDK/Analysis/Umeng',sdkVersion\n"}
#生成Podfile
def create_podfile (var)
    var.concat(["end"])
    puts var
    aFile=File.new('Podfile','w')
    var.each do |i|
        aFile.write(i)
    end
    aFile.close
end
```
生成Podfile之后可以直接执行
```
pod install
```

## 4、 检测使用SDK版本
```
require 'Plist'
require 'net/https'
require 'uri'
require 'json'

#获取本地配置
path = File.join(File.dirname(__FILE__), "VigameLibrary.plist")
result = Plist.parse_xml(path)
version = result["KTMSDK_Version"]
puts version
#读取服务器配置
url = "http://gui.vigame.cn/plugin/files/Versions_iOS.json"
uri = URI.parse(url)
res = Net::HTTP.get_response(uri)

resbody = JSON.parse(res.body)

sever_version = resbody["KTMSDK_Version"]["version"]

if sever_version == version
    print ( "已是最新版本 #{version}\n")

else
    raise "请升级到最新版本 #{sever_version}"
    cmd = "exit 1"
    system (cmd)
end
```

## 5、添加p12及mobileprovision
```
#添加证书和描述文件
security import "./Vigame/证书-密码123456/dis.p12" -k ~/Library/Keychains/login.keychain-db -P "123456" -A
security import "./Vigame/证书-密码123456/dev.p12" -k ~/Library/Keychains/login.keychain-db -P "123456" -A
open ./Vigame/证书-密码123456/gzsj2_20191102_adhoc.mobileprovision
open ./Vigame/证书-密码123456/gzsj2_20191102_dev.mobileprovision

```
## 6、打包脚本
如果需要上传服务器/提审可以扩展实现
```
xcodebuild clean -workspace Unity-iPhone.xcworkspace -scheme Unity-iPhone -configuration enterprise
xcodebuild archive -workspace Unity-iPhone.xcworkspace -scheme Unity-iPhone -archivePath Unity-iPhone.xcarchive
xcodebuild -exportArchive -archivePath "Unity-iPhone.xcarchive" -exportPath ipa -exportOptionsPlist "Vigame/ExportOptions.plist"
```
## 7、上传fir.im
登录fir.im，在个人账号下API Token中获取自己的api_token

```
echo "遍历文件，找到xxxx.ipa"
for file in $(ls *)
do
    strB=".ipa"
    if [[ $file == *$strB* ]] ; then
        fir publish ./ipa/${file} -T “your-apitoken” -Q
    fi
done
```

## 8、打开打包文件夹
里面有下载二维码图片，可以直接把图片给到测试进行下载测试，如果要打上传appstore的包，可以自己手动打包。
```
open ./ipa
```
