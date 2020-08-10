最近下载Xcode9-beta5来研究新功能，发现新建xib会报Safe Area Layout Guide before iOS9.0的错误。

解决方案有2个：1、勾掉Use Safe Area Layout Guides
![](http://upload-images.jianshu.io/upload_images/1648908-0d7b6092c76ef6ad.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

2、修改Build for 支持到iOS9 以上。

参考：
<https://stackoverflow.com/questions/44492404/safe-area-of-xcode-9-beta>

<https://developer.apple.com/videos/play/wwdc2017/412/>
