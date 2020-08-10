
Mac版本的Python默认是2.7，安装高版本后需要修改为你安装的版本。

1，首先打开终端

    open ~/.bash_profile

   打开配置文件

 2\. 写入python的外部环境变量

 export PATH=${PATH}:/Library/Frameworks/Python.framework/Versions/3.8/bin

3.重命名python

alias python="/Library/Frameworks/Python.framework/Versions/3.8/bin/python3.8"

（这步很重要，直接关系到默认启动的python版本是否修改）

4.关闭文件后，在终端调用 source ~/.bash_profile

5.在终端调用 python，查看是否修改成功
