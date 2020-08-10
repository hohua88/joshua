创建两个启动的.storyboard分别命名为：LaunchScreen-English和LaunchScreen-Chinese（当然这里你也可以只创建一个storyboard，然后在启动的时候对storyboard加载的图片的名字国家化，就不具体写了）

"UILaunchStoryboardName" = "LaunchScreen-Chinese";

应用名称本地化/国际化
新建Strings File，文件名字命名为InfoPlist，且必须是这个名字（因每个人电脑设置差异，此处本人电脑没有显示strings后缀名）:
（1）在InfoPlist.strings(english)中加入如下代码：

// Localizable App Name是App在英语环境环境下显示的名称
CFBundleDisplayName = "Localizable App Name";
