还有不到一周Xcode和iOS11就开放下载了，最近闲来研究一下Xcode9新特性，发现Xcode9提供了新的Build系统。新build系统默认是不可用的，但可以通过设置使它生效：在File中选择Workspace Setting ...或Project Setting ...
![](http://upload-images.jianshu.io/upload_images/1648908-d3196c9f43345e6a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
Build System 选择 New Build System （Preview）

新build系统更加严格检测循环引用，并且会给出提示，如图：

![提示截图](http://upload-images.jianshu.io/upload_images/1648908-45cc5811a35c3c9c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这对程序员很友好，可以更方便检测出循环引用，避免内存泄漏。

新build系统不支持标准的“clean”，改为支持“clean build folder”.

想了解其他更多信息请参考以下链接：
https://download.developer.apple.com/Developer_Tools/Xcode_9_GM_seed_build_9A235/Xcode_9_GM_seed_Release_Notes.pdf
