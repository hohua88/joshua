最近有需求用脚本实现工程的配置，现记录一下，实用的是Xcodeproj来实现修改配置。我们的库是c++写的，依赖了openssl等库，需要添加路径依赖

##1.安装xcodeproj
```
[sudo] gem install xcodeproj
```
##2.xcodeproj在工程添加group并引入xx.h和xx.m文件
```
#添加group
require 'xcodeproj'
 
#获取target名字
$target_name
file_path=Dir::pwd
aDir=Dir::entries(file_path)
aDir.each do |dir_file|
    if dir_file.include?('xcodeproj')
        $target_name= dir_file.split('.')[0]
    end
end
#工程路径
$path = File.join(File.dirname(__FILE__), "#{$target_name}.xcodeproj")
puts "获得当前文件的目录 = #{$path} "
#获取project
$project = Xcodeproj::Project.open($path)
puts "当前project = #{$project} "

#获取target
$target = $project.targets.first
puts "当前target = #{$target} "
$group = $project.main_group.find_subpath(File.join('Vigame'), true)
$group.clear
puts "当前group = #{$group} "
$group.set_source_tree('SOURCE_ROOT')

$project.save
```
##3.添加HEADER_SEARCH_PATHS设置

```
$target.build_configurations.each do |config|
   nums = Array["\"$(SRCROOT)/Vigame/include\"","\"$(SRCROOT)/Vigame/deps/curl/include/ios\"", "\"$(SRCROOT)/Vigame/deps/openssl/include/ios\"", "\"$(SRCROOT)/Vigame/deps/zlib/include/linux\"", "\"$(SRCROOT)/Vigame/deps/boost/include\""]
   paths = config.build_settings['HEADER_SEARCH_PATHS']
 paths.concat(nums)
 config.build_settings['HEADER_SEARCH_PATHS'] = paths.uniq
 puts config.build_settings['HEADER_SEARCH_PATHS']
 end

```
##4.添加LIBRARY_SEARCH_PATHS设置
```
$target.build_configurations.each do |config|
   nums = Array["\"$(SRCROOT)/Vigame\""]
   paths = config.build_settings['LIBRARY_SEARCH_PATHS']

   paths.concat(nums)
   
   config.build_settings['LIBRARY_SEARCH_PATHS'] = paths.uniq
   
   puts "LIBRARY_SEARCH_PATHS设置成功 "
   
end
```
#5、添加其他设置
```
$target.build_configurations.each do |config|

   config.build_settings['ENABLE_BITCODE'] = 'NO'
   config.build_settings['ENABLE_BITCODE'] = 'NO'
   config.build_settings['GCC_C_LANGUAGE_STANDARD'] = 'gnu11'
   config.build_settings['CLANG_CXX_LANGUAGE_STANDARD'] = 'gnu++14'
   config.build_settings['GCC_ENABLE_CPP_RTTI'] = 'YES'
   config.build_settings['GCC_ENABLE_OBJC_EXCEPTIONS'] = 'YES'
   config.build_settings['CODE_SIGN_STYLE'] = 'Manual'
   config.build_settings['DEVELOPMENT_TEAM'] = 'xxx'
   config.build_settings['PRODUCT_BUNDLE_IDENTIFIER'] = 'com.xx.xx.xx'
   if config.name == "Release"
       config.build_settings['PROVISIONING_PROFILE_SPECIFIER'] = 'xxx'
        config.build_settings['CODE_SIGN_IDENTITY'] = 'iPhone Distribution'
       config.build_settings['CODE_SIGN_IDENTITY[sdk=iphoneos*]'] = 'iPhone Distribution'
   else
       config.build_settings['PROVISIONING_PROFILE_SPECIFIER'] = 'xxx'
       config.build_settings['CODE_SIGN_IDENTITY'] = 'iPhone Developer'
       config.build_settings['CODE_SIGN_IDENTITY[sdk=iphoneos*]'] = 'iPhone Developer'
   end
end
puts "其他设置成功 "
```
##6.添加.h,m文件
```
$file_refs = []
$file_path3 = File.join(File.dirname(__FILE__), "xxx.h")
$file_path4 = File.join(File.dirname(__FILE__), "xxx.mm")
if $group.find_file_by_path($file_path3)
    puts "xx.h已添加"
else
    $file_refs << $group.new_reference($file_path3)
end
if $group.find_file_by_path($file_path4)
    puts "xx.mm已添加"
else
    $file_refs << $group.new_reference($file_path4)
end

$target.add_file_references($file_refs)
$target.source_build_phase.remove_file_reference($file_refs)
$target.headers_build_phase.remove_file_reference($file_refs)

$file_path7 = File.join(File.dirname(__FILE__), "xx.plist")
if $group.find_file_by_path($file_path7)
    puts "xxx.plist已添加"
else
    $target.resources_build_phase.add_file_reference($group.new_reference($file_path7))
end

end
```
##6.添加xx.a的引用（bundle，framework）
```
$file_path2 = File.join(File.dirname(__FILE__), "xxx/xxx.a")
    
#先移除.a
$file_ref = $project.frameworks_group.new_file($file_path2)
$target.frameworks_build_phases.remove_file_reference($file_ref)
$project.frameworks_group.remove_reference($file_ref)
#再添加.a
$target.frameworks_build_phases.add_file_reference($file_ref)

#添加xx.framework的引用
#file_ref = project.frameworks_group.new_file('xxPath/xx.framework')
#target.frameworks_build_phases.add_file_reference(file_ref)

#添加xx.bundle的引用
#file_ref = project.frameworks_group.new_file('xxPath/xx.bundle')
#target.resources_build_phase.add_file_reference(file_ref)

 #添加.plist
$target.resources_build_phase.add_file_reference($group.new_reference($file_path8))
```
##7.通过xcodeproj设置证书

```
$target.build_configurations.each do |config|
  if config.name == 'Debug'
    config.build_settings["PROVISIONING_PROFILE_SPECIFIER"] = "xxProfileName"
    config.build_settings["DEVELOPMENT_TEAM"] = "xxTeamName"
    config.build_settings["CODE_SIGN_IDENTITY"] = "xxIdentityName"
    config.build_settings["CODE_SIGN_IDENTITY[sdk=iphoneos*]"] = "iPhone Developer"
  end
end
```
##8.添加系统库(framework、tbd)
Xcodeprj提供的添加方式有问题：target.add_system_framework(),问题是路径有问题导致的
```
sys_path = "Platforms/iPhoneOS.platform/Developer/SDKs/iPhoneOS.sdk/System/Library/Frameworks/Metal.framework"
sys_group = $project.frameworks_group()
unless ref = sys_group.find_file_by_path(sys_path)
  ref = sys_group.new_file(sys_path, :developer_dir)
end
$target.frameworks_build_phase.add_file_reference(ref, true)
ref

$target.add_system_library_tbd("sqlite3.0")
```
##9.参考文档
[Xcodeproj](https://github.com/CocoaPods/Xcodeproj.git)
[项目工程的配置库xcodeproj](https://blog.csdn.net/allanGold/article/details/99624099)
[iOS自动打包之xcodeproj](https://www.jianshu.com/p/76ab1d57ad73)
[http://www.rubydoc.info/gems/xcodeproj/Xcodeproj](https://link.jianshu.com/?t=http://www.rubydoc.info/gems/xcodeproj/Xcodeproj)
