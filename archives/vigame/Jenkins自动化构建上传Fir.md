最近项目想要升级到自动打包上传fir，方便测试童鞋测试、并解放自己的双手。下面记录详细步骤。
###Jenkins安装
1、安装homebrew
```
ruby -e "$(curl --insecure -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"  //注意 ruby -e 之间的空格
```
2、安装jenkins
```
brew install jenkins 
```
如果报错Error: An unsatisfied requirement failed this build.需要安装JDK
安装JDK报如下错误
```
curl: (56) LibreSSL SSL_read: SSL_ERROR_SYSCALL, errno 54
Error: Download failed on Cask 'adoptopenjdk8' with message: Download failed: https://github.com/AdoptOpenJDK/openjdk8-binaries/releases/download/jdk8u222-b10/OpenJDK8U-jdk_x64_mac_hotspot_8u222b10.pkg
```
3、启动jenkins
```
jenkins
```
4、在浏览器中访问
 [http://localhost:8080/](https://link.jianshu.com?t=http://localhost:8080/)

5、安装合适插件
![选择合适的插件](https://upload-images.jianshu.io/upload_images/1648908-053b79a71a124143.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


