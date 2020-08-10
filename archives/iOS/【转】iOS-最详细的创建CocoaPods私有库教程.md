转自：http://www.cnblogs.com/XYQ-208910/p/6214066.html
一、感慨
说实话，创建这个CocoaPods私有库，我愣是搞了两个星期，创建的过程中，自己的感情波动是这样的：激情四射---->有点困惑----->极度困惑----->有点失望----->非常失望----->想放弃了----->放弃了----->不甘心,一个星期后又倒腾起来了----->还是失望----->向朋友求助----->问题解决，迟来的兴奋---->成功了，急切的想给大家分享成果。可以说，这个过程真是一波三折，遇到的各种坑让我百感交集。还好，我最终坚持到底，成功了，在此感谢我的好朋友@Kakarotto-卡卡罗特，下面我就给大家分享一下教程。

 

二、说说遇到的坑：
1、本地的私有仓库验证通过，但是远程仓库上的私有仓库验证不通过，路经不对，报Error[iOS] file patterns: The `source_files` pattern did not match any错误

解决办法：重新打开xxx.podspec文件编辑一下，确定共享文件路径没有错误，然后再上传到github上验证。

source_files文件格式有几种设置方法：

s.source_files  = 'Classes/*.{h,m}'
s.source_files  = 'Classes/publicClass.{h,m}'
s.source_files  = 'Classes'
s.source_files  = 'Classes/**/*.{h,m}'
2、上传xxx.podspec到github和给xxx.podspec打tag顺序搞反了，验证不通过

解决办法：必须先将本地文件夹所有的文件上传到github上，然后再给xxx.podspec打上tag，打tag方式也有两种方法：

命令行方式：

git tag -m "注释" 1.0.0
git push --tags
直接在github上点击release进入创建tag：


![791499-20161223103951526-1141007884.png](http://upload-images.jianshu.io/upload_images/1648908-3cbb096223c2ca04.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

3、xxx.podsepc、LICENSE、Demo、pulcicLib它们几个没有放在同一层级上，验证不通过

解决办法：将他们放到同一个文件夹的同一个层级上，例如


![791499-20161223104637573-758494044.png](http://upload-images.jianshu.io/upload_images/1648908-5d8ba4fc74ccbc23.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

4、最后所有的验证都通过了也上传成功了，结果使用pod search仍然搜索不到，是因为search_index.json文件重复了，需要先删除再搜索

解决办法（此处是在成功安装CocoaPods，但不能pod search搜素类库的情况下探讨问题)：

     4.1 执行pod setup

其实在你安装CocoaPods执行pod install时，系统会默认操作pod setup，然而由于中国强大的墙可能会pod setup不成功。这时就需要手动执行pod setup指令，如下：
终端输入：pod setup
会出现Setting up CocoaPods master repo ，稍等几十秒，最底下会输出setup completed。说明执行pod setup成功。
如果pod search操作还是搜索失败，如下：
终端输入：pod search AFNetworking
输出：Unable to find a pod  with name,author,summary,or descriptionmastching 'AFNetworking' 这时就需要继续下面的步骤了。
     4.2 删除~/Library/Caches/CocoaPods目录下的search_index.json文件

pod setup成功后，依然不能pod search，是因为之前你执行pod search生成了search_index.json，此时需要删掉。
终端输入：rm ~/Library/Caches/CocoaPods/search_index.json
删除成功后，再执行pod search。
     4.3 执行pod search

终端输入：pod search sfneteorking (不区分大小写)
输出：Create search index for spec repo 'master'..Deno!，稍等片刻······就会出现所有带有afnetworking字段的类库。
5、验证私有仓库时如果出现gcc编译错误：[-Werror, -Wnon-modular-include-in-framework-module].造成xcode build failed

解决办法是加参数：--use-libraries 

$ pod lib lint xxx.podspec --use-libraries
$ pod spec lint xxx.podspec --use-libraries
$ pod trunk push --use-libraries
提示：最后提一个小策略，如果验证时报错了，可以在验证时加上后缀--verbose来查看错误的具体位置

 

三、简述大致流程
在github上创建项目，复制项目的链接路径，例如：https://github.com/xiayuanquan/XYQCocoaPods.git;
使用命令行或者sourceTree将项目克隆到本地新建的一个文件夹中;
cd进入本地该文件夹，将自己之前的工程文件(demo)以及共享文件(共享库Lib)拖入其中，并创建私有仓库，例如：pod spec create cocoaPodsName; 
编辑私有仓库信息(使用文本编辑器或者sublime等)
编辑结束保存，并验证本地的私有仓库是否有效（—allow-warnings可以消除警告）例如：pod lib lint cocoaPodsName.podspec  —allow-warnings；
验证有效后，然后再将本地该文件夹中所有的文件push到github上
使用git tag(此方法操作后再push上传一次)或者直接在github上点击release进入后创建release并给私有仓库打上tag
注册trunk，例如：pod trunk register 邮箱 ‘用户名’ —descripttion=‘描述’，注意：邮箱为github上的登录邮箱、用户名为github上的用户名
接收发送到邮箱的链接，点击进入后注册成功
查看注册的个人信息，例如：pod trunk me
验证上传到github上的私有仓库是否有效（—allow-warnings可以消除警告，例如：pod spec lint cocoaPodsName.podspec  —allow-warnings
将私有仓库推送到CocoaPods上，例如：pod trunk push cocoaPodsName.podspec 
使用pod search  cocoaPodsName搜索即可
 

四、详细步骤流程如下：
（1）在github上创建项目，复制项目的链接路径，例如：https://github.com/xiayuanquan/XYQCocoaPods.git;
![Uploading 791499-20161223105325245-1157266931_517404.png . . .]
（2）使用命令行或者sourceTree将项目克隆到本地新建的一个文件夹中;



![791499-20161223105453167-507650403.png](http://upload-images.jianshu.io/upload_images/1648908-81bff8a307ae7628.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![791499-20161223105533636-576132103.png](http://upload-images.jianshu.io/upload_images/1648908-8667d93a2ce19b62.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

（3）cd进入本地该文件夹，将自己之前的工程文件(demo)以及共享文件(共享库Lib)拖入其中，并创建私有仓库，例如：pod spec create cocoaPodsName; 

![791499-20161223105611589-1942059256.png](http://upload-images.jianshu.io/upload_images/1648908-42b7a25ee2d14d45.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


注意：下面说的是尽量都保持一样，其实，此处私有库名称必须和共享文件夹名称一样，而和github项目名一不一样，要求不那么严格



![791499-20161223105716964-1211886217.png](http://upload-images.jianshu.io/upload_images/1648908-297b996e521bb744.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


（4）编辑私有仓库信息(使用文本编辑器或者sublime等,注意：引号不能搞错了，是英文格式的"",不是中文格式“”)



![791499-20161223105821557-1687036411.png](http://upload-images.jianshu.io/upload_images/1648908-0c41c8ffeace8ecf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


（5）编辑结束保存，并验证本地的私有仓库是否有效（—allow-warnings可以消除警告）例如：pod lib lint cocoaPodsName.podspec  —allow-warnings；


![791499-20161223105915807-283899940.png](http://upload-images.jianshu.io/upload_images/1648908-d93579769e3be344.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

（6）验证有效后，然后再将本地该文件夹中所有的文件push到github上


![791499-20161223110305823-1118604672.png](http://upload-images.jianshu.io/upload_images/1648908-1396f7256117acea.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

（7）使用git tag(此方法操作后再push上传一次)或者直接在github上点击release进入后创建release并给私有仓库打上tag


![791499-20161223110420432-2110612782.png](http://upload-images.jianshu.io/upload_images/1648908-378f83945b15e4de.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



![791499-20161223110543995-1009159833.png](http://upload-images.jianshu.io/upload_images/1648908-ea24bcca44c7be5f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![791499-20161223110608589-316911985.png](http://upload-images.jianshu.io/upload_images/1648908-7422d7d49dbf97b6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

（8）注册trunk，例如：pod trunk register 邮箱 ‘用户名’ —descripttion=‘描述’，注意：邮箱为github上的登录邮箱、用户名为github上的用户名



![Uploading 791499-20161223110513151-1037094054-3_258257.png . . .]
（9）接收发送到邮箱的链接，点击进入后注册成功


![791499-20161223110724542-1934033210.png](http://upload-images.jianshu.io/upload_images/1648908-0054c395e8127d12.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

（10）查看注册的个人信息，例如：pod trunk me


![791499-20161223110746604-1224144306.png](http://upload-images.jianshu.io/upload_images/1648908-9b4257b55767ad37.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

（11）验证上传到github上的私有仓库是否有效（—allow-warnings可以消除警告，例如：pod spec lint cocoaPodsName.podspec  —allow-warnings


![791499-20161223110813901-1709063508.png](http://upload-images.jianshu.io/upload_images/1648908-79ef1d346518e45f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
（12）将私有仓库推送到CocoaPods上，此处时间会久一点，请耐心等待，例如：pod trunk push cocoaPodsName.podspec 

![791499-20161223110838979-2049955987.png](http://upload-images.jianshu.io/upload_images/1648908-cc571b375ad5abbe.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
（13）使用pod search  cocoaPodsName搜索即可

![791499-20161223110900557-376339341.png](http://upload-images.jianshu.io/upload_images/1648908-8a46d7d3c6131857.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
