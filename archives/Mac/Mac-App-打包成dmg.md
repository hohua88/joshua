##先把开发的Mac App打包成xxx.app
![打包截图.png](https://upload-images.jianshu.io/upload_images/1648908-d444ccf71daba7ff.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![导出.app截图.png](https://upload-images.jianshu.io/upload_images/1648908-b9c872b517e9831f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##安装create-dmg
```
brew install create-dmg
```
安装成功后，可以用下面指令查看详细信息
>create-dmg --help

##生成dmg
```
create-dmg  <output_name.dmg>  <source_folder>
```

##参考
[create-dmg](https://github.com/create-dmg)
