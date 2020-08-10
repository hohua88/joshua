# 一、写在前面的话：

> 发现网上Unity Hub破解的教程大多针对于Windows版本的，几乎没有发现有关于Mac版本的破解教程。所以在这边给介绍下我的破解方法，其实原理和win版类似，只是可能有些命令不同而已。

# 二、安装node.js

不会的童鞋请移步菜鸟教程的[传送门](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.runoob.com%2Fnodejs%2Fnodejs-install-setup.html)，自行选择下载安装包或者使用命令行进行安装，这里不做过多累述。

# 三、终端操作

1、按下`Command` + `Space`，键入终端，打开mac的终端，然后输入以下命令

```
sudo npm install -g asar
```
2、打开`访达`，在`应用程序`中找到`Unity Hub`，然后打开`显示包内容`，然后依次进入`Contents ——> Resources`，不会的小伙伴请参照下图。

![image](//upload-images.jianshu.io/upload_images/9879225-1a1a4916897d9e9d.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/1200)

3、 在Resources文件夹上打开`新建位于文件夹位置的终端窗口`

![image](//upload-images.jianshu.io/upload_images/9879225-19663a31b760c275.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/1148)
4、 接着在打开的终端命令窗口上，输入

```
asar extract app.asar app

```

5、修改Resources文件下的 `app/src/services/licenseService/licenseClient.js`路径下js脚本

![image](//upload-images.jianshu.io/upload_images/9879225-22f9dbe82e4fbb91.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/1200)

6、修改Resources文件下的 `app/src/services/licenseService/licenseCore.js`路径下js脚本

![image](//upload-images.jianshu.io/upload_images/9879225-0254f10a91fd206a.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/1200)

# 四、大功告成

![image](//upload-images.jianshu.io/upload_images/9879225-d1fae259db057e13.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/1200)

五、写在最后的话

Unity是一款强大的3D游戏引擎，如果条件允许请支持正版，我本身也是一名开发者，破解只是为了私下试用，在公司的话请一定支持正版。谢谢合作。
