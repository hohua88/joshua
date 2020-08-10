##1.安装mysql
```
gem install mysql2 -- --with-cflags=\"-I/usr/local/opt/openssl/include\" --with-ldflags=\"-L/usr/local/opt/openssl/lib\"
```
##2.访问数据库
```
require 'mysql2'
client = Mysql2::Client.new(
:host     => 'localhost', # 主机
:username => 'root',      # 用户名
:password => '123456',    # 密码
:database => 'test',      # 数据库
:encoding => 'utf8'       # 编码
)

results = client.query("SELECT * FROM `table_name` WHERE `pid` LIKE '%123456%' LIMIT 0, 1000")
results.each do |row|
    row['pid'] 
#   puts row
end
```
##2.安装plist
```
[sudo] gem install plist
```
##3.修改plist
```
require 'Plist'

def modify_config (var1, var2)
    path = File.join(File.dirname(__FILE__), "VigameLibrary.plist")
    result = Plist.parse_xml(path)
    result[var1] = var2
    Plist::Emit.save_plist(result, path)
    puts result
end
```
遇到问题：
在执行脚本时报错：
`require': cannot load such file -- mysql (LoadError)

解决方式：
```
gem uninstall mysql2
gem install mysql2 -- --with-cflags=\"-I/usr/local/opt/openssl/include\" --with-ldflags=\"-L/usr/local/opt/openssl/lib\"
```
##参考

[https://github.com/patsplat/plist](https://github.com/patsplat/plist)
[https://www.rubydoc.info/gems/plist](https://www.rubydoc.info/gems/plist)
