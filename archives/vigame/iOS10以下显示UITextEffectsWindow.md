在```+ (void)applicationDidBecomeActive:(UIApplication *_Nullable)application```添加代码
```
static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
         [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(keyboardWillShow:) name:UIKeyboardWillShowNotification object:nil];
               UITextField *textfield = [[UITextField alloc] init];
               [textfield becomeFirstResponder];
               textfield.tag = 100;
               [[UIApplication sharedApplication].keyWindow.rootViewController.view addSubview:textfield];
    });
```
```
+ (void)keyboardWillShow:(id)sender {
    UITextField *view = (UITextField *) [[UIApplication sharedApplication].keyWindow.rootViewController.view viewWithTag:100];
    [view resignFirstResponder];
    [view removeFromSuperview];
    
}
```

运行程序会调出UITextEffectsWindow
