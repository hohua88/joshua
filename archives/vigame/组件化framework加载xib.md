最近在做组件化，需要制作静态库，发现图片文件的加载没问题，但是加载xib时总是报  .bundle> (not yet loaded)，很是头疼。

先说怎么加载图片，一定要带上bundle的包名，要不然找不到文件。如下：

UIImage *key_img = [UIImage imageNamed:@"xx/ps.png"];

那么下面说下怎么解决xib问题。

1.先给静态库新建一个bundle,必要选项。
![image.png](https://upload-images.jianshu.io/upload_images/1648908-73f6456c728c5568.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

2.修改bundle为iOS
![image.png](https://upload-images.jianshu.io/upload_images/1648908-c9c2abfa85537f19.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

3、添加xib资源文件
![image.png](https://upload-images.jianshu.io/upload_images/1648908-4350df0f8ae0bbb6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

4、build后直接把framework和bundle文件导出
![image.png](https://upload-images.jianshu.io/upload_images/1648908-b5aa4de007e46030.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
