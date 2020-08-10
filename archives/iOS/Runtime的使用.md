Runtime简介
runtime顾名思义就是运行时，OC就是运行时机制，其中最主要是消息机制。
对于C语言，函数的调用在编译的时候会决定调用那个函数。
对于OC函数，属于动态调用过程，在编译时并不能真正决定调用那个函数，只有在真正运行时才会根据函数名称找到相应的函数来调用。
Runtime作用
一.实现多继承Multiple Inheritance
根据forwardingTargetForSelector:方法可以发现，一个类可以做到继承多个类的效果，只需要在消息转发时转发给其他对象即可模拟出多继承的效果。
 
在官方文档上记录这样一个例子：

 
![1194012-d635696fe66b05b8.png](http://upload-images.jianshu.io/upload_images/1648908-41db94373205e6b8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
在OC程序中可以借用消息转发机制来实现多继承的功能。 在上图中，一个对象对一个消息做出回应，类似于另一个对象中的方法借过来或是“继承”过来一样。 在图中，warrior实例转发了一个negotiate消息到Diplomat实例中，执行Diplomat中的negotiate方法，结果看起来像是warrior实例执行了一个和Diplomat实例一样的negotiate方法，其实执行者还是Diplomat实例。
 
这使得不同继承体系分支下的两个类可以“继承”对方的方法，这样一个类可以响应自己继承分支里面的方法，同时也能响应其他不相干类发过来的消息。在上图中Warrior和Diplomat没有继承关系，但是Warrior将negotiate消息转发给了Diplomat后，就好似Diplomat是Warrior的超类一样。
 
消息转发提供了许多类似于多继承的特性，但是他们之间有一个很大的不同： 
 
多继承：合并了不同的行为特征在一个单独的对象中，会得到一个重量级多层面的对象。 
 
消息转发：将各个功能分散到不同的对象中，得到的一些轻量级的对象，这些对象通过消息通过消息转发联合起来。
二.方法交换Method Swizzling
对于runtime，很多人第一印象就是黑魔法Method Swizzling。毕竟这是runtime使用最广泛的一个方法，毕竟这是Runtime里面很强大的一部分，它可以通过Runtime的API实现更改任意的方法，理论上可以在运行时通过类名/方法名hook到任何 OC 方法，替换任何类的实现以及新增任意类。
 
1. Method Swizzling原理
 
Method Swizzing是发生在运行时的，主要用于在运行时将两个Method进行交换，我们可以将Method Swizzling代码写到任何地方，但是只有在这段Method Swilzzling代码执行完毕之后互换才起作用。而且Method Swizzling也是iOS中AOP(面相切面编程)的一种实现方式，我们可以利用苹果这一特性来实现AOP编程。
 
Method Swizzling本质上就是对IMP和SEL进行交换。
2.Method Swizzling使用
一般我们使用都是新建一个分类，在分类中进行Method Swizzling方法的交换。交换的代码模板如下：
 
#import <objc/runtime.h>
 
@implementation NSObject (Extension)
 
+ (void)originalClassSelector:(SEL)originalSelector withSwizzledClassSelector:(SEL)swizzledSelector{
 
    Method originalMethod = class_getClassMethod(self, originalSelector);
 
    Method swizzledMethod = class_getClassMethod(self, swizzledSelector);
 
    method_exchangeImplementations(originalMethod, swizzledMethod);
 
}
 
 
 
@end
 
@implementation UIFont (KCFont)
 
 
 
+ (void)load{
 
    static dispatch_once_t onceToken;
 
    dispatch_once(&onceToken, ^{
 
        [self originalClassSelector:@selector(systemFontOfSize:) withSwizzledClassSelector:@selector(kc_systemFontOfSize:)];
 
        [self originalClassSelector:@selector(boldSystemFontOfSize:) withSwizzledClassSelector:@selector(kc_boldSystemFontOfSize:)];
 
        [self originalClassSelector:@selector(italicSystemFontOfSize:) withSwizzledClassSelector:@selector(kc_italicSystemFontOfSize:)];
 
        [self originalClassSelector:@selector(fontWithName:size:) withSwizzledClassSelector:@selector(kc_fontWithName:size:)];
 
    });
 
}
 
 
 
+ (UIFont *)kc_systemFontOfSize:(CGFloat)fontSize{
 
    return [self kc_systemFontOfSize:fontSize*(SizeScale>1?SizeScale*0.9:SizeScale)];
 
}
 
 
 
+ (UIFont *)kc_boldSystemFontOfSize:(CGFloat)fontSize{
 
    return [self kc_boldSystemFontOfSize:fontSize*(SizeScale>1?SizeScale*0.9:SizeScale)];
 
}
 
+ (UIFont *)kc_italicSystemFontOfSize:(CGFloat)fontSize{
 
    return [self kc_italicSystemFontOfSize:fontSize*(SizeScale>1?SizeScale*0.9:SizeScale)];
 
}
 
 
 
+ (UIFont *)kc_fontWithName:(NSString *)fontName size:(CGFloat)fontSize{
 
    return [self kc_fontWithName:fontName size:fontSize*(SizeScale>1?SizeScale*0.9:SizeScale)];
 
}
 
3.Method Swizzling注意点
（1）.Swizzling应该总在+load中执行
Objective-C在运行时会自动调用类的两个方法+load和+initialize。+load会在类初始加载时调用， +initialize方法是以懒加载的方式被调用的，如果程序一直没有给某个类或它的子类发送消息，那么这个类的 +initialize方法是永远不会被调用的。所以Swizzling要是写在+initialize方法中，是有可能永远都不被执行。
和+initialize比较+load能保证在类的初始化过程中被加载。
关于+load和+initialize的比较可以参看这篇文章《Objective-C +load vs +initialize》
（2）.Swizzling应该总是在dispatch_once中执行
Swizzling会改变全局状态，所以在运行时采取一些预防措施，使用dispatch_once就能够确保代码不管有多少线程都只被执行一次。这将成为Method Swizzling的最佳实践。
（3）.Swizzling在+load中执行时，不要调用[super load]
 
4.Method Swizzling使用场景
 
Method Swizzling使用场景其实有很多，在一些特殊的开发需求中适时的使用黑魔法，可以做法神来之笔的效果。这里就举3种常见的场景。
 
1.实现AOP
 
AOP官方文档，很多热更新都是使用这种方法来实现的，比如JSPatch等等。（前段时间苹果限制了具有热更新功能的APP上线）
 
2.实现无埋点统计
 
如果app统计需求，并且要自己实现一套埋点逻辑，那么这里用到Swizzling是很合适的选择。看到一篇分析的挺精彩的无埋点的文章，推荐大家阅读。
iOS动态性(二)可复用而且高度解耦的用户统计埋点实现
 
3.实现异常保护
 
日常开发我们经常会遇到NSArray数组越界的情况，苹果的API也没有对异常保护，所以需要我们开发者开发时候多多留意。关于Index有好多方法，objectAtIndex，removeObjectAtIndex，replaceObjectAtIndex，exchangeObjectAtIndex等等，这些设计到Index都需要判断是否越界。
 
常见做法是给NSArray，NSMutableArray增加分类，增加这些异常保护的方法，不过如果原有工程里面已经写了大量的AtIndex系列的方法，去替换成新的分类的方法，效率会比较低。这里可以考虑用Swizzling做。
三、动态关联属性Associated Object
Associated Objects是Objective-C 2.0中Runtime的特性之一。众所周知，在 Category 中，我们无法添加@property，因为添加了@property之后并不会自动帮我们生成实例变量以及存取方法。那么，我们现在就可以通过关联对象来实现在 Category 中添加属性的功能了。
1.用法
借用环信里面关于加载框的实现方式
 
#import "UIViewController+HUD.h"
 
#import "MBProgressHUD.h"
 
#import <objc/runtime.h>
 
#define IS_IPHONE_5 ( fabs( ( double )[ [ UIScreen mainScreen ] bounds ].size.height - ( double )568 ) < DBL_EPSILON )
 
static const void *HttpRequestHUDKey = &HttpRequestHUDKey;
 
 
 
@implementation UIViewController (HUD)
 
 
 
- (MBProgressHUD *)HUD{
 
    return objc_getAssociatedObject(self, HttpRequestHUDKey);
 
}
 
 
 
- (void)setHUD:(MBProgressHUD *)HUD{
 
    objc_setAssociatedObject(self, HttpRequestHUDKey, HUD, OBJC_ASSOCIATION_RETAIN_NONATOMIC);
 
}
 
来说明一下这些参数的意义：
 
1.id object 设置关联对象的实例对象
 
2.const void *key 区分不同的关联对象的 key。
3.id value 关联的对象
4.objc_AssociationPolicy policy 关联对象的存储策略，它是一个枚举，与property的attribute 相对应。
关联对象3种使用场景
1.为现有的类添加私有变量
2.为现有的类添加公有属性
3.为KVO创建一个关联的观察者。
四、动态增加方法
在消息发送阶段，如果在父类中也没有找到相应的IMP，就会执行resolveInstanceMethod方法。在这个方法里面，我们可以动态的给类对象或者实例对象动态的增加方法。
 
//动态添加方法
 
+ (BOOL)resolveInstanceMethod:(SEL)sel{
 
    if ([NSStringFromSelector(sel) isEqualToString:@"sayHi:"]) {
 
        class_addMethod(self, sel, (IMP)abc, "v@:@");
 
    }
 
    return YES;
 
}
五、NSCoding的自动归档和自动解档
总所周知，项目开发中会需要对自定义的对象进行保存，而保存需要实现NSCoding协议的encodeWithCoder和initWithCoder方法，在项目需求变更或者属性字段很多的情况下，需要手写很多重复的代码，维护起来也很麻烦。这时自动实现就显得很有优势。
用runtime实现的思路就比较简单，快速枚举属性列表，然后利用getter和setter就可以完成encodeWithCoder和initWithCoder了。
 
#pragma mark - 归档／解归档
 
- (id)initWithCoder:(NSCoder *)aDecoder{
 
    NSArray *propertyNames = [[self class] properties];
 
    for (NSString *propertyName in propertyNames) {
 
        //首字母大写
 
        NSString *setPropertyName = propertyName.capitalizedString;
 
        //NSString* firstCharacter = [propertyName substringToIndex:1].uppercaseString;
 
        //拼接上set关键字
 
        setPropertyName = [NSString stringWithFormat:@"set%@:",setPropertyName];
 
        
 
        //获取set方法
 
        SEL setSel = NSSelectorFromString(setPropertyName);
 
        
 
        [self performSelector:setSel withObject:[aDecoder decodeObjectForKey:propertyName]];
 
    }
 
    return self;
 
}
 
 
 
- (void)encodeWithCoder:(NSCoder *)enCoder{
 
    NSArray *propertyNames = [[self class] properties];
 
    for (NSString *propertyName in propertyNames) {
 
        //获取get方法
 
        SEL getSel = NSSelectorFromString(propertyName);
 
        [enCoder encodeObject:[self performSelector:getSel] forKey:propertyName];
 
    }
 
}
五、字典和模型相互转化
 
1.字典转模型
 
1.调用 class_getProperty 方法获取当前 Model 的所有属性。
2.调用 property_copyAttributeList 获取属性列表。
3.根据属性名称生成 setter 方法。
4.使用 objc_msgSend 调用 setter 方法为 Model 的属性赋值（或者 KVC）
 
+(id)objectWithKeyValues:(NSDictionary *)aDictionary{
    id objc = [[self alloc] init];
    for (NSString *key in aDictionary.allKeys) {
        id value = aDictionary[key];

        /*判断当前属性是不是Model*/
        objc_property_t property = class_getProperty(self, key.UTF8String);
        unsigned int outCount = 0;
        objc_property_attribute_t *attributeList = property_copyAttributeList(property, &outCount);
        objc_property_attribute_t attribute = attributeList[0];
        NSString *typeString = [NSString stringWithUTF8String:attribute.value];

        if ([typeString isEqualToString:@"@\"Student\""]) {
            value = [self objectWithKeyValues:value];
        }

        //生成setter方法，并用objc_msgSend调用
        NSString *methodName = [NSString stringWithFormat:@"set%@%@:",[key substringToIndex:1].uppercaseString,[key substringFromIndex:1]];
        SEL setter = sel_registerName(methodName.UTF8String);
        if ([objc respondsToSelector:setter]) {
            ((void (*) (id,SEL,id)) objc_msgSend) (objc,setter,value);
        }
        free(attributeList);
    }
    return objc;
}
 
这段代码里面有一处判断typeString的，这里判断是防止model嵌套，比如说Student里面还有一层Student，那么这里就需要再次转换一次，当然这里有几层就需要转换几次。
 
2.模型转字典
 
这里是上一部分字典转模型的逆步骤：
 
1.调用 class_copyPropertyList 方法获取当前 Model 的所有属性。
2.调用 property_getName 获取属性名称。
3.根据属性名称生成 getter 方法。
4.使用 objc_msgSend 调用 getter 方法获取属性值（或者 KVC）
 
//模型转字典
-(NSDictionary *)keyValuesWithObject{
    unsigned int outCount = 0;
    objc_property_t *propertyList = class_copyPropertyList([self class], &outCount);
    NSMutableDictionary *dict = [NSMutableDictionary dictionary];
    for (int i = 0; i < outCount; i ++) {
        objc_property_t property = propertyList[i];

        //生成getter方法，并用objc_msgSend调用
        const char *propertyName = property_getName(property);
        SEL getter = sel_registerName(propertyName);
        if ([self respondsToSelector:getter]) {
            id value = ((id (*) (id,SEL)) objc_msgSend) (self,getter);

            /*判断当前属性是不是Model*/
            if ([value isKindOfClass:[self class]] && value) {
                value = [value keyValuesWithObject];
            }

            if (value) {
                NSString *key = [NSString stringWithUTF8String:propertyName];
                [dict setObject:value forKey:key];
            }
        }

    }
    free(propertyList);
    return dict;
}
 
中间注释那里的判断也是防止model嵌套，如果model里面还有一层model，那么model转字典的时候还需要再次转换，同样，有几层就需要转换几次。
六、实现对对象的全打印
在开发中会遇到需要打印对象的需求，而直接打印对象会打印出地址，这显然不是你想要的，你需要的是打印对象的所有属性及其值。
runtime通过重写description方法实现对对象的全打印
1、获取属性列表
+ (NSArray *)properties{
    unsigned int outcount = 0;
    Ivar *ivars = class_copyIvarList(self, &outcount);
    NSMutableArray *arrayM = [NSMutableArray array];
    for ( int i = 0; i< outcount; i ++) {
        NSString *name = @(ivar_getName(ivars[i]));
        NSString *key = [name substringFromIndex:1];
        [arrayM addObject:key];
    }
    free(ivars);
    return [arrayM copy];
}

2、重写description
- (NSString *)description{
    NSMutableString *descriptionString = [NSMutableString stringWithFormat:@"\n"];
    NSArray *properNames = [[self class] properties];
    for (NSString *propertyName in properNames) {
        SEL getSel = NSSelectorFromString(propertyName);
        NSString *propertyNameString = [NSString stringWithFormat:@"%@ - %@\n",propertyName,[self performSelector:getSel]];
        [descriptionString appendString:propertyNameString];
    }
    return descriptionString;
}
实现原理通过getter方法获取属性的值，实现一一对应的拼接。当然实现一个NSObject的category即可实现对所有继承至NSObject的对象的打印。
