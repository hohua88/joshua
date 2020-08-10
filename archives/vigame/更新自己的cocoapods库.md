##### 1.创建项目的podspec文件
pod spec create WXWCategory 

##### 2.修改XXXX.podspec文件中s.version的内容

Pod::Spec.new do |s|
s.version = "0.0.x"
s.summary = "xxxxxx"
...
end

##### 3.上传到Git

将包含配置好的 .podspec, 项目提交 Git

##### 4.打tag

因为cocoapods是依赖tag版本的,所以必须打tag,
以后再次更新只需要把你的项目打一个tag
然后修改.podspec文件中的版本接着提交到cocoapods官方就可以了

###### 提交命令

```
//为git打tag, 第一次需要在前面加一个v
git tag "v1.0.0" 
//将tag推送到远程仓库
git push --tags 

```

##### 5.验证.podspec文件

```
// --verbose 如果验证失败会报错误信息
pod lib lint xxx.podspec --allow-warnings --use-libraries

```

##### 6.发布

```
// --use-libraries --allow-warnings
pod trunk push xxxxxx.podspec --allow-warnings --use-libraries

```
 报错--template= --single-branch --depth 1 --branch 1.0.0
删除tag重新添加

后续更新版本需要执行4、5步

其他人需将私有的远程索引库copy到本地，才能使用
pod repo add IOSMavenSpec http://wy@dnsdk.vimedia.cn:8080/r/IOSMavenSpec.git
