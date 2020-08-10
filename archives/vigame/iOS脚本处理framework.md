## 背景
SDK支持window、iOS、Android等，内里有很多文件，每次手动生成.a及头文件，由于这个库很多人维护，不清楚具体的修改情况，会只生成.a文件，没有重新生成头文件，导致有些接口返回异常或不能使用，最近就因为这个原因导致上线的一个游戏用户串号，给公司造成不小的损失。本着一次解决问题的想法，引入脚本处理人工步骤。

## 解决方案
出新版本SDK时，用脚本同步生成头文件和静态库.a，避免人为因素导致的问题。

1、新建一个target
![Aggregate.png](https://upload-images.jianshu.io/upload_images/1648908-1cef1eee5da57de0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

2、编写处理脚本
脚本部分包含源代码文件夹，生成头文件需要copy一份源码，删除非.h文件，重命名为include文件夹，放到Vigame文件夹下,xxx.a也放到此目录下。
*注：此部分为公司特有，且不支持模拟器及arm64以下架构，有需要可以自行裁剪*
```
if [ "${ACTION}" = "build" ]
then

#要build的target名
target_Name=${PROJECT_NAME}
    echo "target_Name=${target_Name}"

#build之后的文件夹路径
build_DIR=${SRCROOT}/build
    echo "build_DIR=${build_DIR}"

#vigame源文件夹路径

SOURCE_DIR=$(dirname $(dirname $(dirname "${SRCROOT}")))

VIGAME_PATH="${SOURCE_DIR}/source/vigame"
#真机build生成的头文件的文件夹路径
DEVICE_DIR_INCLUDE=${build_DIR}/Release-iphoneos/${PROJECT_NAME}
    echo "DEVICE_DIR_INCLUDE=${DEVICE_DIR_INCLUDE}"

#真机build生成的.a文件路径
DEVICE_DIR_A=${build_DIR}/Release-iphoneos/lib${PROJECT_NAME}.a
    echo "DEVICE_DIR_A=${DEVICE_DIR_A}"
    
#目标文件夹路径
INSTALL_DIR=${SRCROOT}/Products/${PROJECT_NAME}
    echo "INSTALL_DIR=${INSTALL_DIR}"

#目标头文件文件夹路径
INSTALL_DIR_INCLUDE=${SRCROOT}/Products/${PROJECT_NAME}/include
    echo "INSTALL_DIR_Headers=${INSTALL_DIR_Headers}"

#目标.a路径
INSTALL_DIR_A=${SRCROOT}/Products/${PROJECT_NAME}/lib${PROJECT_NAME}.a
    echo "INSTALL_DIR_A=${INSTALL_DIR_A}"

#判断build文件夹是否存在，存在则删除
if [ -d "${build_DIR}" ]
then
rm -rf "${build_DIR}"
fi

#判断目标文件夹是否存在，存在则删除该文件夹
if [ -d "${INSTALL_DIR}" ]
then
rm -rf "${INSTALL_DIR}"
fi
#创建目标文件夹
mkdir -p "${INSTALL_DIR}"

#build之前clean一下
xcodebuild -target ${target_Name} clean

#真机build
xcodebuild -target ${target_Name} -configuration Release -sdk iphoneos

#复制.a/.h文件到目标文件夹

cp -R "${DEVICE_DIR_A}"  "${INSTALL_DIR_A}"

cp -R "${VIGAME_PATH}" "${INSTALL_DIR_INCLUDE}"

rm -rf ${build_DIR}

cd  "${INSTALL_DIR_INCLUDE}"

#删除非.h文件
find ./ -name *.cpp | xargs rm -f;
find ./ -name *.m | xargs rm -f;
find ./ -name *.mm | xargs rm -f;
find ./ -name *.hpp | xargs rm -f;
find ./ -name *.a | xargs rm -f;
 
#打开目标文件夹
open "${INSTALL_DIR}"

fi
```

每次重新导出新版本SDK时，只需要执行aggregate，达到事半功倍的效果。
