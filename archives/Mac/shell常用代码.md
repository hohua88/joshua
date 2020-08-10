1、免密执行安装插件
```
echo [password] | sudo -S gem install plist
```
2、为空判断
```
if [ -n "$x" ] ; then
   command
else
   exit 1
fi
```
3、遍历文件获取文件名
文件名称不能带有空格，有空格会获取空格后的名字，现已处理支持带空格的情况。
```
for element in `ls "$1" | tr " " "\?"`
do
    element=`tr "\?" " " <<<$element`
    dir_or_file="$1"/"$element"
    if [ -d "$dir_or_file" ];then
        var=$( find "$dir_or_file" -name '*.xcodeproj' )
        var1=${var##*/}
        if [ -n "$var1" ];then
            podStr="Pods"
           if [[ $var1 == *$podStr* ]];then
            echo "Pods.xcodeproj"
           else
            #截取.之前的所有字符串
            appname=${var1%.*}
            echo $appname
           fi
        fi
       
    fi
done
```
带有空格的获取方式，🌰是获取xxx.app这个名字，改签时要用。
```
var=$( find "$path/ios_resign/Payload" -name '*.app' )
app_name=${var##*/}
```
4、打包ipa脚本
```
xcodebuild clean -workspace "${app_name}.xcworkspace" -scheme "${app_name}" -configuration enterprise
xcodebuild archive -workspace "${app_name}.xcworkspace" -scheme "${app_name}" -archivePath "${app_name}.xcarchive" -quiet
xcodebuild -exportArchive -archivePath "${app_name}.xcarchive" -exportPath ipa -exportOptionsPlist "Vigame/ExportOptions.plist"

```
5、上传fir.im
```
fir publish file -T "${token}" -Q
```
6、find命令
```
keyfile=Images.xcassets
x=$(find filepath -name $keyfile)
```

7、添加证书
```
#添加证书和描述文件
security import xx.p12 -k ~/Library/Keychains/login.keychain-db -P [password] -A
open xxx.mobileprovision
```

8、shell传递参数
如果参数超过10个，后续的参数需要加{}
```
${10}
```
