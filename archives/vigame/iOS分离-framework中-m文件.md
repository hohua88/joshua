1、进入.framework目录
```
cd /Users/xx/Downloads/xxx.framework 
```
2、新建文件夹-用来保存分离出的文件
```
mkdir arm_64
```
3、分离出arm64的静态库文件
```
lipo ./xxx -thin arm64 -output ./arm_64/aa_arm64
```
4、进入arm_64目录
```
cd arm-64
```
5、抽离.framework文件的object文件.o
```
ar -x aa_arm64
```
6、将.o转换成.m
```
nm xxx.o > xxx.m
```
