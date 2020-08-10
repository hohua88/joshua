###1、问题描述
unity导出的iOS工程，在设置launchScreen.storyboard时，启动页会有一个竖屏的过度展示。
###2、解决办法
```
- (NSUInteger)application:(UIApplication*)application supportedInterfaceOrientationsForWindow:(UIWindow*)window{
      return UIInterfaceOrientationMaskLandscape;
}
```
