## 修改UnityAppController脚本
```
using UnityEngine;
using System.Collections;
using System.Collections.Generic;
using System.IO;

namespace UnityEditor.XCodeEditor
{
    public partial class XClass : System.IDisposable
    {

        private string filePath;

        public XClass(string fPath)
        {
            filePath = fPath;
            if (!System.IO.File.Exists(filePath))
            {
                Debug.LogError(filePath + "路径下文件不存在");
                return;
            }
        }

        public void WriteBelow(string below, string text)
        {
            StreamReader streamReader = new StreamReader(filePath);
            string text_all = streamReader.ReadToEnd();
            streamReader.Close();

            int beginIndex = text_all.IndexOf(below);
            if (beginIndex == -1)
            {
                Debug.LogError(filePath + "中没有找到标致" + below);
                return;
            }

            int endIndex = text_all.LastIndexOf("\n", beginIndex + below.Length);

            text_all = text_all.Substring(0, endIndex) + "\n" + text + "\n" + text_all.Substring(endIndex);

            StreamWriter streamWriter = new StreamWriter(filePath);
            streamWriter.Write(text_all);
            streamWriter.Close();
        }

        public void Replace(string below, string newText)
        {
            StreamReader streamReader = new StreamReader(filePath);
            string text_all = streamReader.ReadToEnd();
            streamReader.Close();

            int beginIndex = text_all.IndexOf(below);
            if (beginIndex == -1)
            {
                Debug.LogError(filePath + "中没有找到标致" + below);
                return;
            }

            text_all = text_all.Replace(below, newText);
            StreamWriter streamWriter = new StreamWriter(filePath);
            streamWriter.Write(text_all);
            streamWriter.Close();

        }

        public void Dispose()
        {

        }
    }
}
```

## 导出添加接入代码
```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEditor;
using UnityEditor.Callbacks;
using UnityEditor.iOS.Xcode;
using System.IO;
using System;
using UnityEditor.XCodeEditor;

public class ZPXCodePostProcess
{
#if UNITY_EDITOR
    [PostProcessBuild(1000)]
    public static void OnPostProcessBuild(BuildTarget target, string pathToBuiltProject)
    {
        if (target != BuildTarget.iOS) return;
        //得到xcode工程的路径
        string path = Path.GetFullPath (pathToBuiltProject);
        //编辑代码文件
        EditorCode(path);
    }
    private static void EditorCode(string filePath)
    {
		//读取UnityAppController.mm文件
        XClass UnityAppController = new XClass(filePath + "/Classes/UnityAppController.mm");
 
        //在指定代码后面增加一行代码
        UnityAppController.WriteBelow("#include \"PluginBase/AppDelegateListener.h\"","#import \"IOSLoader.h\"");

        //在指定代码后面增加一行代码
        UnityAppController.WriteBelow("AppController_SendNotificationWithArg(kUnityWillFinishLaunchingWithOptions, launchOptions);","\r\t [IOSLoader splashReport];");
        UnityAppController.WriteBelow("AppController_SendNotificationWithArg(kUnityOnOpenURL, notifData);","\r\t return [IOSLoader application:app openURL:url options:options];");
 
        UnityAppController.WriteBelow("UnityInitApplicationNoGraphics([[[NSBundle mainBundle] bundlePath] UTF8String]);","\r\t [IOSLoader application:application DidFinishLaunchingWithOptions:launchOptions];");

        UnityAppController.WriteBelow("- (void)applicationDidEnterBackground:(UIApplication*)application\n{","\r\t [IOSLoader applicationDidEnterBackground:application];");
        UnityAppController.WriteBelow("- (void)applicationWillEnterForeground:(UIApplication*)application\n{","\r\t [IOSLoader applicationWillEnterForeground:application];");
        UnityAppController.WriteBelow("- (void)applicationDidBecomeActive:(UIApplication*)application\n{","\r\t [IOSLoader applicationDidBecomeActive:application];");
        
        UnityAppController.WriteBelow(" SensorsCleanup();\n}","- (BOOL)application:(UIApplication *)application continueUserActivity:(NSUserActivity *)userActivity restorationHandler:(void (^)(NSArray<id<UIUserActivityRestoring>> * _Nullable))restorationHandler \r{\r\t  return [IOSLoader application:application continueUserActivity:userActivity restorationHandler:restorationHandler];\r}");
 
    }
#endif

}
```

## 参考
[Unity3D研究院之IOS全自动编辑framework、plist、oc代码（六十七）](http://www.xuanyusong.com/archives/2720)
