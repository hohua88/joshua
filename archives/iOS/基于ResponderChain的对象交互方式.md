#概述
  
感谢casa大神的分享：[一种基于ResponderChain的对象交互方式](https://casatwy.com/responder_chain_communication.html),这里也只是作为笔记来记录。闲话少叙，进入正文。

#前言

众所周知，传统iOS的对象间交互模式就那么几种：直接property传值、delegate、KVO、block、protocol、多态、Target-Action。本文主要介绍基于ResponderChain来实现对象间交互。

这种方式通过在UIResponder上挂一个category，使得事件和参数可以沿着responder chain逐步传递。

这相当于借用responder chain实现了一个自己的事件传递链。这在事件需要层层传递的时候特别好用，然而这种对象交互方式的有效场景仅限于在responder chain上的UIResponder对象上。

#实现

仅需要一个category即可实现基于ResponderChain的对象间交互。

.h文件

```
#import <UIKit/UIKit.h>

@interface UIResponder (Router)

- (void)routerEventWithName:(NSString *)eventName userInfo:(NSDictionary *)userInfo;

@end
```

.m文件

```
#import "UIResponder+Router.h"

@implementation UIResponder (Router)


- (void)routerEventWithName:(NSString *)eventName userInfo:(NSDictionary *)userInfo {
    [[self nextResponder] routerEventWithName:eventName userInfo:userInfo];
}

@end
```

发送事件时
```
[self routerEventWithName:kTHCellClickedEvent userInfo:@{@"key":@"clicked",@"value":@"2"}];
```

相应事件时
```
#pragma mark - event response
- (void)routerEventWithName:(NSString *)eventName userInfo:(NSDictionary *)userInfo
{

    /*
        do things you want
    */

    // 如果需要让事件继续往上传递，则调用下面的语句
    // [super routerEventWithName:eventName userInfo:userInfo];
}
```
结合[策略模式](http://www.runoob.com/design-pattern/strategy-pattern.html)进行更好的处理

在上面的Demo中，如果事件来源有多个，那就无法避免需要if-else语句来针对具体事件作相应的处理。这种情况下，会导致if-else语句极多。所以，可以考虑采用strategy模式来消除if-else语句。

```
#pragma mark - event response
- (void)routerEventWithName:(NSString *)eventName userInfo:(NSDictionary *)userInfo
{

    NSInvocation *invocation = self.eventStrategy[eventName];
    [invocation setArgument:&userInfo atIndex:2];
    [invocation invoke];

    // 如果需要让事件继续往上传递，则调用下面的语句
    // [super routerEventWithName:eventName userInfo:userInfo];
}
```

self.eventStrategy是一个字典，这个字典以eventName作key，对应的处理逻辑封装成[NSInvocation](http://www.jianshu.com/p/03e7279a9916)来做value。

```
- (NSDictionary <NSString *, NSInvocation *> *)eventStrategy
{
    if (_eventStrategy == nil) {
        _eventStrategy = @{
                               kTHCellClickedEvent: [self createInvocationWithSelector:@selector(ticketEvent:)]
                               };
    }
    return _eventStrategy;
}
```
博客中casa大神没有给出createInvocationWithSelector的实现，我在这里给出我的实现：
```
#import "NSObject+Invotation.h"

@implementation NSObject (Invotation)

- (NSInvocation *)createInvocationWithSelector:(SEL)aSelector {
    NSMethodSignature *signature = [[self class] instanceMethodSignatureForSelector:aSelector];
    
    if (!signature) {
        NSString *info = [NSString stringWithFormat:@"-[%@ %@]:unrecognized selector sent to instance", [self class], NSStringFromSelector(aSelector)];
        @throw [[NSException alloc] initWithName:@"remind:" reason:info userInfo:nil];
        return nil;
    }
    
    NSInvocation *invocation = [NSInvocation invocationWithMethodSignature:signature];
    invocation.target = self;
    invocation.selector = aSelector;
    
    return  invocation;
}

@end
```
在这种场合下使用Strategy模式，即可避免多事件处理场景下导致的冗长if-else语句。

结合Decorator模式

在事件层层向上传递的时候，每一层都可以往UserInfo这个dictionary中添加数据。那么到了最终事件处理的时候，就能收集到各层综合得到的数据，从而完成最终的事件处理。

```
#pragma mark - event response
- (void)routerEventWithName:(NSString *)eventName userInfo:(NSDictionary *)userInfo
{
    NSMutableDictionary *decoratedUserInfo = [[NSMutableDictionary alloc] initWithDictionary:userInfo];
    decoratedUserInfo[@"newParam"] = @"new param"; // 添加数据
    [super routerEventWithName:eventName userInfo:decoratedUserInfo]; // 往上继续传递
}

```
分析基于ReponderChain的对象交互方式

这种对象交互方式的缺点显而易见，它只能对存在于Reponder Chain上的UIResponder对象起作用。

优点倒是也有蛮多：

以前靠delegate层层传递的方案，可以改为这种基于Responder Chain的方式来传递。在复杂UI层级的页面中，这种方式可以避免无谓的delegate声明。
由于众多自定义事件都通过这种方式做了传递，就使得事件处理的逻辑得到归拢。在这个方法里面下断点就能够管理所有的事件处理。
使用Strategy模式优化之后，UIViewController/UIView的事件响应逻辑得到了很好的管理，响应逻辑不会零散到各处地方。
在此基础上使用Decorator模式，能够更加有序地收集、归拢数据，降低了工程的维护成本。


基于ResponderChain的对象交互方式的适用场景首先要求事件的产生和处理的对象都必须在Responder Chain上，这一点前面已经说过，我就不再赘述了。



它的适用场景还有一个值得说的地方，就是它可以无视命名域的存在。如果采用传统的delegate层层传递的方式，由于delegate需要protocol的声明，因此就无法做到命名域隔离。但如果走Responder Chain，即使是另一个UI组件产生了事件，这个事件就可以被传递到其他组件的UI上。

举个例子：XXXViewController属于A组件，这个UIViewController的view上添加了B组件的某个YYView。那么YYView的事件在通过Responder Chain被XXXViewController处理的时候，就可以不必依赖B组件的YYView了。当然，如果事件本身传递了只有B组件才有的对象，无视命名域这个优点就没有了，不过这种场景在实际业务中其实也不多。

在这种场景下就做到了UI展示与事件处理的分离，事件处理的代码就可以归拢到另外一个对象中去了，使得C的代码量得以减少。

或许可以算是一种新的架构模式：MVCE（Modle View Controller Event）？其实名字、模式什么的都已经不重要了，毕竟所有的架构模式都是脱胎于MVC的，作不同程度的拆解罢了。

