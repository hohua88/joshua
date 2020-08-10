[转自李峰峰博客](http://www.imlifengfeng.com/blog/?p=457)
#一、概述
闭包 = 一个函数「或指向函数的指针」+ 该函数执行的外部的上下文变量「也就是自由变量」；Block 是 Objective-C 对于闭包的实现。
其中，Block：

>可以嵌套定义，定义 Block 方法和定义函数方法相似
Block 可以定义在方法内部或外部
只有调用 Block 时候，才会执行其{}体内的代码
本质是对象，使代码高聚合
 

使用 clang 将 OC 代码转换为 C++ 文件查看 block 的方法：
+ 在命令行输入代码 clang -rewrite-objc 需要编译的OC文件.m
+ 这时查看当前的文件夹里 多了一个相同的名称的 .cpp 文件，在命令行输入 open main.cpp 查看文件

#二、Block的定义与使用
1、无参数无返回值
> //1，无参数，无返回值，声明和定义
void(^MyBlockOne)(void) = ^(void){
NSLog(@"无参数，无返回值");  
};  
MyBlockOne();//block的调用

2、有参数无返回值
>//2，有参数，无返回值，声明和定义
void(^MyblockTwo)(int a) = ^(int a){
NSLog(@"@ = %d我就是block，有参数，无返回值",a);
  };  
MyblockTwo(100);

3、有参数有返回值

>//3，有参数，有返回值
int(^MyBlockThree)(int,int) = ^(int a,int b){    
  NSLog(@"%d我就是block，有参数，有返回值",a + b);returna + b; 
 };  
MyBlockThree(12,56);

4、实际开发中常用typedef 定义Block

例如，用typedef定义一个block：
>typedef int (^MyBlock)(int , int);

这时，MyBlock就成为了一种Block类型
在定义类的属性时可以这样：

>@property (nonatomic,copy) MyBlock myBlockOne;

使用时：

>self.myBlockOne = ^int (int ,int){
    //TODO
}

#三、Block与外界变量

1、截获自动变量（局部变量）值
（1）默认情况

对于 block 外的变量引用，block 默认是将其复制到其数据结构中来实现访问的。也就是说block的自动变量截获只针对block内部使用的自动变量, 不使用则不截获, 因为截获的自动变量会存储于block的结构体内部, 会导致block体积变大。特别要注意的是默认情况下block只能访问不能修改局部变量的值。

![](http://upload-images.jianshu.io/upload_images/1648908-8c32ee3ca19f8687.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

>int age = 10;
myBlock block = ^{
    NSLog(@"age = %d", age);
};
age = 18;
block();

输出结果：
	
>age = 10

（2） __block 修饰的外部变量

对于用 __block 修饰的外部变量引用，block 是复制其引用地址来实现访问的。block可以修改__block 修饰的外部变量的值。
![](http://upload-images.jianshu.io/upload_images/1648908-371df0a06fe65229.jpg)
>__block int age = 10;
myBlock block = ^{
    NSLog(@"age = %d", age);
};
age = 18;
block();

输出为：

>age = 18

为什么使用__block 修饰的外部变量的值就可以被block修改呢？

我们使用 clang 将 OC 代码转换为 C++ 文件：
>clang -rewrite-objc 源代码文件名

便可揭开其真正面纱：

>__block int val = 10;
转换成
__Block_byref_val_0 val = {
    0,
    &val,
    0,
    sizeof(__Block_byref_val_0),
    10
};

会发现一个局部变量加上__block修饰符后竟然跟block一样变成了一个__Block_byref_val_0结构体类型的自动变量实例！！！！

此时我们在block内部访问val变量则需要通过一个叫__forwarding的成员变量来间接访问val变量(下面会对__forwarding进行详解)

#四、Block的copy操作
1、Block的存储域及copy操作

在开始研究Block的copy操作之前，先来思考一下：Block是存储在栈上还是堆上呢？

我们先来看看一个由C/C++/OBJC编译的程序占用内存分布的结构：
![](http://upload-images.jianshu.io/upload_images/1648908-a08227546d1cf255.jpg)
其实，block有三种类型：
+ 全局块(_NSConcreteGlobalBlock)
+ 栈块(_NSConcreteStackBlock)
+ 堆块(_NSConcreteMallocBlock)

这三种block各自的存储域如下图：
![](http://upload-images.jianshu.io/upload_images/1648908-afba6aaea73a35e7.jpg)

+ 全局块存在于全局内存中, 相当于单例.
+ 栈块存在于栈内存中, 超出其作用域则马上被销毁
+ 堆块存在于堆内存中, 是一个带引用计数的对象, 需要自行管理其内存

简而言之，存储在栈中的Block就是栈块、存储在堆中的就是堆块、既不在栈中也不在堆中的块就是全局块。

 

遇到一个Block，我们怎么这个Block的存储位置呢？

（1）Block不访问外界变量（包括栈中和堆中的变量）

Block 既不在栈又不在堆中，在代码段中，ARC和MRC下都是如此。此时为全局块。

 

（2）Block访问外界变量

MRC 环境下：访问外界变量的 Block 默认存储栈中。
ARC 环境下：访问外界变量的 Block 默认存储在堆中（实际是放在栈区，然后ARC情况下自动又拷贝到堆区），自动释放。

ARC下，访问外界变量的 Block为什么要自动从栈区拷贝到堆区呢？

栈上的Block，如果其所属的变量作用域结束，该Block就被废弃，如同一般的自动变量。当然，Block中的__block变量也同时被废弃。如下图：
![](http://upload-images.jianshu.io/upload_images/1648908-fa88626136d7ddba.png)

为了解决栈块在其变量作用域结束之后被废弃（释放）的问题，我们需要把Block复制到堆中，延长其生命周期。开启ARC时，大多数情况下编译器会恰当地进行判断是否有需要将Block从栈复制到堆，如果有，自动生成将Block从栈上复制到堆上的代码。Block的复制操作执行的是copy实例方法。Block只要调用了copy方法，栈块就会变成堆块。

如下图：
![](http://upload-images.jianshu.io/upload_images/1648908-0e6e1229c626ed9b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

例如下面一个返回值为Block类型的函数：

>typedef int (^blk_t)(int);
blk_t func(int rate) {
    return ^(int count) { return rate * count; };
}

分析可知：上面的函数返回的Block是配置在栈上的，所以返回函数调用方时，Block变量作用域就结束了，Block会被废弃。但在ARC有效，这种情况编译器会自动完成复制。

在非ARC情况下则需要开发者调用copy方法手动复制，由于开发中几乎都是ARC模式，所以手动复制内容不再过多研究。

将Block从栈上复制到堆上相当消耗CPU，所以当Block设置在栈上也能够使用时，就不要复制了，因为此时的复制只是在浪费CPU资源。

 

Block的复制操作执行的是copy实例方法。不同类型的Block使用copy方法的效果如下表：
![](http://upload-images.jianshu.io/upload_images/1648908-27fba051f832e782.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
根据表得知，Block在堆中copy会造成引用计数增加，这与其他Objective-C对象是一样的。虽然Block在栈中也是以对象的身份存在，但是栈块没有引用计数，因为不需要，我们都知道栈区的内存由编译器自动分配释放。关于堆区和栈区详细内容可以参考下峰哥之前的文章：《[总结：堆、栈、队列](http://www.imlifengfeng.com/blog/?p=147)》

不管Block存储域在何处，用copy方法复制都不会引起任何问题。在不确定时调用copy方法即可。

在ARC有效时，多次调用copy方法完全没有问题：
>blk = [[[[blk copy] copy] copy] copy];
// 经过多次复制，变量blk仍然持有Block的强引用，该Block不会被废弃。

2、__block变量与__forwarding

在copy操作之后，既然__block变量也被copy到堆上去了, 那么访问该变量是访问栈上的还是堆上的呢?__forwarding 终于要闪亮登场了，如下图：
![](http://upload-images.jianshu.io/upload_images/1648908-999ec1b3d7b1c740.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

通过__forwarding, 无论是在block中还是 block外访问__block变量, 也不管该变量在栈上或堆上, 都能顺利地访问同一个__block变量。
 

#五、防止 Block 循环引用

Block 循环引用的情况：
某个类将 block 作为自己的属性变量，然后该类在 block 的方法体里面又使用了该类本身，如下：
>self.someBlock = ^(Type var){
    [self dosomething];
};

解决办法：

（1）ARC 下：使用 __weak

>__weak typeof(self) weakSelf = self;
self.someBlock = ^(Type var){
   [weakSelf dosomething];
};

（2）MRC 下：使用 __block
>__block typeof(self) blockSelf = self;
self.someBlock = ^(Type var){
   [blockSelf dosomething];
};


值得注意的是，在ARC下，使用 __block 也有可能带来的循环引用，如下：
>// 循环引用 self -> _attributBlock -> tmp -> self
typedef void (^Block)();
@interface TestObj : NSObject
{
    Block _attributBlock;
}
@end
 
>@implementation TestObj
- (id)init {
    self = [super init];
    __block id tmp = self;
    self.attributBlock = ^{
        NSLog(@"Self = %@",tmp);
        tmp = nil;
   };
}
 
>- (void)execBlock {
    self.attributBlock();
}
@end
 
>// 使用类
id obj = [[TestObj alloc] init];
[obj execBlock]; // 如果不调用此方法，tmp 永远不会置 nil，内存泄露会一直在

#六、Block的使用示例

1、Block作为变量（Xcode快捷键：inlineBlock）

>int (^sum) (int, int); // 定义一个 Block 变量 sum
// 给 Block 变量赋值
// 一般 返回值省略：sum = ^(int a,int b)…
sum = ^int (int a,int b){  
    return a+b;
}; // 赋值语句最后有 分号
int a = sum(10,20); // 调用 Block 变量
 

2、Block作为属性（Xcode 快捷键：typedefBlock）

>// 1. 给  Calculate 类型 sum变量 赋值「下定义」
typedef int (^Calculate)(int, int); // calculate就是类型名
Calculate sum = ^(int a,int b){ 
    return a+b;
};
int a = sum(10,20); // 调用 sum变量
 
>// 2. 作为对象的属性声明，copy 后 block 会转移到堆中和对象一起
@property (nonatomic, copy) Calculate sum;    // 使用   typedef
@property (nonatomic, copy) int (^sum)(int, int); // 不使用 typedef
 
>// 声明，类外
self.sum = ^(int a,int b){
    return a+b;
};
// 调用，类内
int a = self.sum(10,20);
 

3、作为 OC 中的方法参数
>// ---- 无参数传递的 Block ---------------------------
// 实现
- (CGFloat)testTimeConsume:(void(^)())middleBlock {
    // 执行前记录下当前的时间
    CFTimeInterval startTime = CACurrentMediaTime();
    middleBlock();
    // 执行后记录下当前的时间
    CFTimeInterval endTime = CACurrentMediaTime();
    return endTime - startTime;
}
 
>// 调用
[self testTimeConsume:^{
       // 放入 block 中的代码 
 
>}];
 
>// ---- 有参数传递的 Block ---------------------------
// 实现
- (CGFloat)testTimeConsume:(void(^)(NSString * name))middleBlock {
    // 执行前记录下当前的时间
    CFTimeInterval startTime = CACurrentMediaTime();
    NSString *name = @"有参数";
    middleBlock(name);
    // 执行后记录下当前的时间
    CFTimeInterval endTime = CACurrentMediaTime();
    return endTime - startTime;
}
 
>// 调用
[self testTimeConsume:^(NSString *name) {
   // 放入 block 中的代码，可以使用参数 name
   // 参数 name 是实现代码中传入的，在调用时只能使用，不能传值    
 
>}];

4、Block回调

Block回调是关于Block最常用的内容，比如网络下载，我们可以用Block实现下载成功与失败的反馈。开发者在block没发布前，实现回调基本都是通过代理的方式进行的，比如负责网络请求的原生类NSURLConnection类，通过多个协议方法实现请求中的事件处理。而在最新的环境下，使用的NSURLSession已经采用block的方式处理任务请求了。各种第三方网络请求框架也都在使用block进行回调处理。这种转变很大一部分原因在于block使用简单，逻辑清晰，灵活等原因。

如下：
>//DownloadManager.h
#######import <Foundation/Foundation.h>
 
>@interface DownloadManager : NSObject <NSURLSessionDownloadDelegate>
 
>// block 重命名
typedef void (^DownloadHandler)(NSData * receiveData, NSError * error);
 
>- (void)downloadWithURL:(NSString *)URL parameters:(NSDictionary *)parameters handler:(DownloadHandler)handler ;
 
>@end

>//DownloadManager.m
#import "DownloadManager.h"
 
>@implementation DownloadManager
 
> -(void)downloadWithURL:(NSString *)URL parameters:(NSDictionary *)parameters handler:(DownloadHandler)handler
{
    NSURLRequest * request = [NSURLRequest requestWithURL:[NSURL URLWithString:URL]];
    NSURLSession * session = [NSURLSession sharedSession];
 
  >  //执行请求任务
    NSURLSessionDataTask * task = [session dataTaskWithRequest:request completionHandler:^(NSData * _Nullable data, NSURLResponse * _Nullable response, NSError * _Nullable error) {
        if (handler) {
            dispatch_async(dispatch_get_main_queue(), ^{
                handler(data,error);
            });
        }
    }];
    [task resume];
}

上面通过封装NSURLSession的请求，传入一个处理请求结果的block对象，就会自动将请求任务放到工作线程中执行实现，我们在网络请求逻辑的代码中调用如下：

>-(IBAction)buttonClicked:(id)sender {
    define DOWNLOADURL @"https://codeload.github.com/AFNetworking/AFNetworking/zip/master"
    //下载类
    DownloadManager * downloadManager = [[DownloadManager alloc] init];
    [downloadManager downloadWithURL: DOWNLOADURL parameters:nil handler:^(NSData *receiveData, NSError *error) {
        if (error) {
            NSLog(@"下载失败：%@",error);
        }else {
            NSLog(@"下载成功，%@",receiveData);
        }
    }];
}


为了加深理解，再来一个简单的小例子：

A，B两个界面，A界面中有一个label，一个buttonA。点击buttonA进入B界面，B界面中有一个UITextfield和一个buttonB，点击buttonB退出B界面并将B界面中UITextfield的值传到A界面中的label。

A界面中，也就是ViewController类中：

>//关键demo：
-(IBAction)buttonAction {  
    MyFirstViewController *myVC = [[MyFirstViewController alloc] init];
    [self presentViewController:myVC animated:YES completion:^{    
    }];
    __weak typeof(self) weakSelf = self;//防止循环引用
//用属性定义的注意：这里属性是不会自动补全的，方法就会自动补全
    [myVC setBlock:^(NSString *string){
        weakSelf.labelA.text = string;
    }];
}

B界面中，也就是MyFirstViewController类中.m文件:

>-(IBAction)buttonBAction {
    [self dismissViewControllerAnimated:YES completion:^{
    }];
      self.block(_myTextfielf.text);
}

.h文件：
>####### import <UIKit/UIKit.h>
//typedef定义一下block，为了更好用
typedef void(^MyBlock)(NSString *string); 
@interface MyFirstViewController : UIViewController
@property (nonatomic, copy) MyBlock block;
@end

看了以上两个Block回调示例，是不是感觉比delegate清爽了不少？
