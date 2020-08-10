```
IconContents(){
cat <<EOF >./AppIcon.appiconset/Contents.json

    {
        "images": [
            {
                "size": "20x20",
                "idiom": "iphone",
                "filename": "icon-20@2x.png",
                "scale": "2x"
            },
            {
                "size": "20x20",
                "idiom": "iphone",
                "filename": "icon-20@3x.png",
                "scale": "3x"
            },
            {
                "size": "29x29",
                "idiom": "iphone",
                "filename": "icon-29.png",
                "scale": "1x"
            },
            {
                "size": "29x29",
                "idiom": "iphone",
                "filename": "icon-29@2x.png",
                "scale": "2x"
            },
            {
                "size": "29x29",
                "idiom": "iphone",
                "filename": "icon-29@3x.png",
                "scale": "3x"
            },
            {
                "size": "40x40",
                "idiom": "iphone",
                "filename": "icon-40@2x.png",
                "scale": "2x"
            },
            {
                "size": "40x40",
                "idiom": "iphone",
                "filename": "icon-40@3x.png",
                "scale": "3x"
            },
            {
                "size": "60x60",
                "idiom": "iphone",
                "filename": "icon-60@2x.png",
                "scale": "2x"
            },
            {
                "size": "60x60",
                "idiom": "iphone",
                "filename": "icon-60@3x.png",
                "scale": "3x"
            },
            {
                "size": "20x20",
                "idiom": "ipad",
                "filename": "icon-20-ipad.png",
                "scale": "1x"
            },
            {
                "size": "20x20",
                "idiom": "ipad",
                "filename": "icon-20@2x-ipad.png",
                "scale": "2x"
            },
            {
                "size": "29x29",
                "idiom": "ipad",
                "filename": "icon-29-ipad.png",
                "scale": "1x"
            },
            {
                "size": "29x29",
                "idiom": "ipad",
                "filename": "icon-29@2x-ipad.png",
                "scale": "2x"
            },
            {
                "size": "40x40",
                "idiom": "ipad",
                "filename": "icon-40.png",
                "scale": "1x"
            },
            {
                "size": "40x40",
                "idiom": "ipad",
                "filename": "icon-40@2x.png",
                "scale": "2x"
            },
            {
                "size": "76x76",
                "idiom": "ipad",
                "filename": "icon-76.png",
                "scale": "1x"
            },
            {
                "size": "76x76",
                "idiom": "ipad",
                "filename": "icon-76@2x.png",
                "scale": "2x"
            },
            {
                "size": "83.5x83.5",
                "idiom": "ipad",
                "filename": "icon-83.5@2x.png",
                "scale": "2x"
            },
            {
                "size": "1024x1024",
                "idiom": "ios-marketing",
                "filename": "icon-1024.png",
                "scale": "1x"
            }
        ],
        "info": {
            "version": 1,
            "author": "eddy"
        }
    }
EOF
}

setIconImage(){
    echo "20pt图标生成······"
    sips -z 20 20 $iconfileName --out ./AppIcon.appiconset/icon-20-ipad.png
    sips -z 40 40 $iconfileName --out ./AppIcon.appiconset/icon-20@2x-ipad.png
    sips -z 40 40 $iconfileName --out ./AppIcon.appiconset/icon-20@2x.png
    sips -z 60 60 $iconfileName --out ./AppIcon.appiconset/icon-20@3x.png
    echo "29pt图标生成······"
    sips -z 29 29 $iconfileName --out ./AppIcon.appiconset/icon-29-ipad.png
    sips -z 29 29 $iconfileName --out ./AppIcon.appiconset/icon-29.png
    sips -z 58 58 $iconfileName --out ./AppIcon.appiconset/icon-29@2x-ipad.png
    sips -z 58 58 $iconfileName --out ./AppIcon.appiconset/icon-29@2x.png
    sips -z 87 87 $iconfileName --out ./AppIcon.appiconset/icon-29@3x.png
    echo "40pt图标生成······"
    sips -z 40 40 $iconfileName --out ./AppIcon.appiconset/icon-40.png
    sips -z 80 80 $iconfileName --out ./AppIcon.appiconset/icon-40@2x.png
    sips -z 120 120 $iconfileName --out ./AppIcon.appiconset/icon-40@3x.png
    echo "60pt图标生成······"
    sips -z 120 120 $iconfileName --out ./AppIcon.appiconset/icon-60@2x.png
    sips -z 180 180 $iconfileName --out ./AppIcon.appiconset/icon-60@3x.png
    echo "76pt图标生成······"
    sips -z 76 76 $iconfileName --out ./AppIcon.appiconset/icon-76.png
    sips -z 152 152 $iconfileName --out ./AppIcon.appiconset/icon-76@2x.png

    echo "83.5pt图标生成······"
    sips -z 167 167 $iconfileName --out ./AppIcon.appiconset/icon-83.5@2x.png

    echo "1024pt图标生成······"
    sips -z 1024 1024 $iconfileName --out ./AppIcon.appiconset/icon-1024.png
}
mkdir AppIcon.appiconset
if [ -n "$1" ] ; then
   iconfileName=$1
else
    echo "icon不能为空"
   exit 1
fi
IconContents
setIconImage
```
mv -f AppIcon.appiconset  "$x"
