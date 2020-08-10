

ERROR: While executing gem ... (Gem::FilePermissionError)

You don't have write permissions for the /usr/bin directory.

解决方案 有人说 前面加sudo 明明已经加了 是无写入到/usr/bin directory 权限

执行此命令即可

sudo gem install cocoapods -n /usr/local/bin
