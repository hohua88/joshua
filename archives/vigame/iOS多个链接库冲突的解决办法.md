## [iOS多个链接库冲突的解决办法（重复使用相同的开源代码）](https://www.cnblogs.com/dhui69/p/7943652.html)

//查看lib库所支持的框架类型

lipo -info libiOS_common.a

//armv7

lipo -extract_family armv7 -output libiOS_common_arm.a libiOS_common.a

//查看是否是Non-fat file

lipo -info libiOS_common_arm.a

上面方法 分离出来可能包含(armv7 armv7s)

可以用下面的命令分离armv7 armv7s

//分离armv7

lipo YTFaceSDK.framework/YTFaceSDK -thin armv7 -output YTFaceSDK.framework/YTFaceSDK_armv7

//分离armv7s

lipo YTFaceSDK.framework/YTFaceSDK -thin armv7s -output YTFaceSDK.framework/YTFaceSDK_armv7s

//arm64

lipo libiOS_common.a -thin arm64 -output libiOS_common_arm64.a

//查看是否是Non-fat file

lipo -info libiOS_common_arm64.a

//分离出.o文件

ar -x libiOS_common_arm.a

ar -x libiOS_common_arm64.a

//合并成静态库

ar rcs libiOS_common_arm64.a arm64/*.o

ar rcs libiOS_common_armv7.a armv7/*.o

重新合并为fat file的.a文件（去掉重复.o的）：lipo -create libiOS_common_armv7.a libiOS_common_arm64.a -output libiOS_common_new.a
