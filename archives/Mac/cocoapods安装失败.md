
```
$ sudo gem install cocoapods
Building native extensions. This could take a while...
ERROR:  Error installing cocoapods:
	ERROR: Failed to build gem native extension.

    current directory: /Library/Ruby/Gems/2.3.0/gems/ffi-1.13.1/ext/ffi_c
/System/Library/Frameworks/Ruby.framework/Versions/2.3/usr/bin/ruby -I /Library/Ruby/Site/2.3.0 -r ./siteconf20200723-14718-1bs6cxc.rb extconf.rb
mkmf.rb can't find header files for ruby at /System/Library/Frameworks/Ruby.framework/Versions/2.3/usr/lib/ruby/include/ruby.h

extconf failed, exit code 1

Gem files will remain installed in /Library/Ruby/Gems/2.3.0/gems/ffi-1.13.1 for inspection.
Results logged to /Library/Ruby/Gems/2.3.0/extensions/universal-darwin-18/2.3.0/ffi-1.13.1/gem_make.out

```
更改为sudo gem install -n /usr/local/bin cocoapods 还是同样错误，原因是缺少ffi
解决方案：
```
sudo gem install -n /usr/local/bin ffi
```
成功后，再次执行sudo gem install -n /usr/local/bin cocoapods即可。
