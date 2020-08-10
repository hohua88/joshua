### 安装Ruby(Mac)
 如果之前使用rvm安装的ruby，烦请卸载rvm及安装的ruby
```
$ rvm list #查看安装ruby版本
$ rvm uninstall 2.2.0 #删除ruby 2.2.0

$ rvm implode #删除rvm
```
使用HomeBrew来安装ruby：
```
$ brew install ruby
```
### 安装xcodeproj
```
$ sudo gem install -n /usr/local/bin xcodeproj
```
### 安装plist
```
$ sudo gem install -n/usr/local/bin plist
```
### 安装fir-cli
```
$ sudo gem install -n /usr/bin fir-cli
```
### 安装mysql2
```
$ sudo gem install -n /usr/local/bin mysql2  -- --with-cflags=\"-I/usr/local/opt/openssl/include\" --with-ldflags=\"-L/usr/local/opt/openssl/lib\"
```

### 安装cocoapods
```
$ sudo gem install -n /usr/local/bin cocoapods  
```
