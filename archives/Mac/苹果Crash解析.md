苹果审核给出的错误信息是xxx.txt的文件，不能直接看出具体崩溃的代码，苹果给出了symbolicatecrash这个工具来查看崩溃日志。

### 1、找到symbolicatecrash工具
打开命令行，执行脚本获取
```
find /Applications/Xcode.app -name symbolicatecrash -type f
```
输出结果：
```
/Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/iOSSupport/Library/PrivateFrameworks/DVTFoundation.framework/Versions/A/Resources/symbolicatecrash
/Applications/Xcode.app/Contents/Developer/Platforms/WatchSimulator.platform/Developer/Library/PrivateFrameworks/DVTFoundation.framework/symbolicatecrash
/Applications/Xcode.app/Contents/Developer/Platforms/AppleTVSimulator.platform/Developer/Library/PrivateFrameworks/DVTFoundation.framework/symbolicatecrash
/Applications/Xcode.app/Contents/Developer/Platforms/iPhoneSimulator.platform/Developer/Library/PrivateFrameworks/DVTFoundation.framework/symbolicatecrash
/Applications/Xcode.app/Contents/SharedFrameworks/DVTFoundation.framework/Versions/A/Resources/symbolicatecrash
```
找到我们需要的iPhoneSimulator，copy相应的路径/Applications/Xcode.app/Contents/Developer/Platforms/iPhoneSimulator.platform/Developer/Library/PrivateFrameworks/DVTFoundation.framework/symbolicatecrash，进入之后会发现
![symbolicatecrash](https://upload-images.jianshu.io/upload_images/1648908-3391f21362d99725.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

为了使用方便，在一个合适位置，新建文件夹crash，把symbolicatecrash拷贝到cash文件夹下。

查看crash.txt的uuid，打开crash.txt，搜索Binary Images:
![uuid.png](https://upload-images.jianshu.io/upload_images/1648908-e388461041a9c72b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


### 2、崩溃日志
把崩溃日志拷贝到cash目录下，可以更改崩溃日志名称为cash.txt

### 3、dSYM文件
window ->organizer找到打包文件，show in finder,再点击，查看显示包内容，这个时候，应该就找到我们所说的那个”.app.dSYM文件”，拷贝到cash目录

查看dSYM文件uuid
```
dwarfdump --uuid dSYM文件路径
```
### 4、cd 到cash目录下
```
cd /Users/xxx/Desktop/crash 
```

### 5、导出crash.log
```
./symbolicatecrash  ./crash.txt  ./xxx.app.dSYM > crash.log

```
如果报错，提示我们需要设置 "DEVELOPER_DIR" 这个环境变量
```
export DEVELOPER_DIR=/Applications/Xcode.app/Contents/Developer
```
重新执行./symbolicatecrash  ./crash.txt  ./xxx.app.dSYM > crash.log即可导出crash.log日志。
