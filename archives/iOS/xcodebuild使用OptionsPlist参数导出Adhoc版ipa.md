1.  [[ -s "$HOME/.rvm/scripts/rvm" ]] && source "$HOME/.rvm/scripts/rvm" && rvm use system  

在Xcode 7.0和8.0之后。使用xcodebuild -exportArchive -exportFormat ipa 导出ipa会有个警告，这个警告不会影响导出。只是指出一种新的导出方式。

首先看如下的命令行

xcodebuild 

> > -exportArchive
> 
> > -archivePath ./build/app.xcarchive 
> > 
> > -exportPath ./out
> > 
> > -OptionsPlist ./com.options.plist 
> > 

这里对参数做几个说明

1\. -exportPath 参数是输出ipa的文件夹，如果不存在会新建。ipa的名称，使用这种导出方式自动使用scheme或者build setting名称作为ipa文件名。

2\. -OptionsPlist ./com.options.plist 是新增参数，plist文件可以有ad-hoc或者appstore两种配置。这里列出ad-hoc的配置
```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
        <key>method</key>//app-store/enterprise/ad-hoc/development
        <string>enterprise</string>
</dict></plist>
```

3\. 调用这个命令行导出时，要首先执行如下语句，切换使用系统的rvm

[[ -s "$HOME/.rvm/scripts/rvm" ]] && source "$HOME/.rvm/scripts/rvm" && rvm use system

error: exportArchive: No applicable devices found.

如果不调用这一行，会报出一个错误
