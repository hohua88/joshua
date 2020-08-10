1、@property本质是什么？
  @property = ivar + getter + setter;
 成员变量+get方法+set方法,也就是说使用@property 系统会自动生成setter和getter方法;
2、@synthesize和@dynamic的区别
  iOS4.5之前写法是，在@property声明属性后，可以有@synthesize或者@dynamic两种实现方式的选择。
使用@property+@synthesize方式，能够让编译器在编译期间自动生成setter/getter这两个方法，配合属性声明使用。
使用@property+@dynamic方式，在编译期间没有生成setter/getter这两个方法，你想要使用这个属性需要自己重新写一遍setter/getter方法，不写也不产生编译警告。
3、属性默认关键字是readwrite,atomic, assign,strong。
4、NSString何时用copy和strong？
注意：上面的情况是针对于当把NSMutableString赋值给NSString的时候，才会有不同，如果是赋值是NSString对象，那么使用copy还是strong，结果都是一样的，因为NSString对象根本就不能改变自身的值，他是不可变的。

把一个对象赋值给一个属性变量，当这个对象变化了，如果希望属性变量变化就使用strong属性，如果希望属性变量不跟着变化，就是用copy属性。
5、若想令自己所写的对象具有拷贝功能，则需实现 NSCopying 协议。如果自定义的对象分为可变版本与不可变版本，那么就要同时实现 NSCopying与 NSMutableCopying协议。

具体步骤：

需声明该类遵从 NSCopying 协议
实现 NSCopying 协议。该协议只有一个方法:
- (id)copyWithZone:(NSZone *)zone;

至于如何重写带 copy 关键字的 setter这个问题，
如果抛开本例来回答的话，如下：

- (void)setName:(NSString *)name {
    //[_name release];
    _name = [name copy];
}
6、Outlet属性是有view来进行强引用的，我们在viewController里面仅仅是对其使用，并没有必要拥有它，所以是weak的。
7、atomic线程安全吗？
当线程A进行写操作，这时其他线程的读或者写操作会因为该操作而等待。当A线程的写操作结束后，B线程进行写操作，然后当A线程需要读操作时，却获得了在B线程中的值，这就破坏了线程安全，如果有线程C在A线程读操作前release了该属性，那么还会导致程序崩溃。所以仅仅使用atomic并不会使得线程安全，我们还要为线程添加lock来确保线程的安全。
8、NSCache优于NSDictionary的地方？
1NSCache类结合了各种自动删除策略，以确保不会占用过多的系统内存。如果其它应用需要内存时，系统自动执行这些策略。当调用这些策略时，会从缓存中删除一些对象，以最大限度减少内存的占用。
2NSCache是线程安全的，我们可以在不同的线程中添加、删除和查询缓存中的对象，而不需要锁定缓存区域。
3不像NSMutableDictionary对象，一个缓存对象不会拷贝key对象。

Designated Initializer 注意事项：
1. 每个类的正确初始化过程应当是按照从子类到父类的顺序，依次调用每个类的Designated Initializer。并且用父类的Designated Initializer初始化一个子类对象，也需要遵从这个过程。
 
2. 如果子类指定了新的初始化器，那么在这个初始化器内部必须调用父类的Designated Initializer。并且需要重写父类的Designated Initializer，将其指向子类新的初始化器。
 
3. 你可以不自定义Designated Initializer，也可以重写父类的Designated Initializer，但需要调用直接父类的Designated Initializer。
 
4. 如果有多个Secondary initializers(次要初始化器)，它们之间可以任意调用，但最后必须指向Designated Initializer。在Secondary initializers内不能直接调用父类的初始化器。
 
5. 如果有多个不同数据源的Designated Initializer，那么不同数据源下的Designated Initializer应该调用相应的[super (designated initializer)]。如果父类没有实现相应的方法，则需要根据实际情况来决定是给父类补充一个新的方法还是调用父类其他数据源的Designated Initializer。比如UIView的initWithCoder调用的是NSObject的init。
 
6.需要注意不同数据源下添加额外初始化动作的时机。

进程和线程的区别？
（1）   进程：是一个程序在其自身的地址空间中的一次执行活动。

进程是资源申请、调度和独立运行的单位，因此，它使用系统中的运行资源； 而程序不能申请系统资源，不能被系统调度，也不能作为独立运行的单位，因此，它不占用系统的运行资源。
（2）   线程：是进程中的一个单一的连续控制流程。一个进程可以拥有多个线程。

线程又称为轻量级进程，它和进程一样拥有独立的执行控制，由操作系统负责调度，区别在于线程没有独立的存储空间，而是和所属进程中的其它线程共享一个存储空间，这使得线程间的通信远较进程简单。
（3）   一个运行的程序就是一个进程或者叫做一个任务

（4）   一个进程至少包含一个线程，线程是程序的执行流

（5）   iOS程序启动时，在创建一个进程的同时， 会开始运行一个线程，该线程被称为主线程

（6）   主线程是其他线程最终的父线程，所有界面的显示操作必须在主线程进行！！！

（7）   后台线程无法更新UI界面和响应用户点击事件

（8）   系统中的每一个进程都有自己独立的虚拟内存空间，而同一个进程中的多个线程则共用进程的内存空间

（9）   每创建一个新的线程，都会消耗一定内存和CPU时间

（10）    当多个线程对同一个资源出现争夺的时候需要注意线程安全问题

线程间通信：1全局变量（用volatile来定义，以防编译器对此变量进行优化）2Message消息机制（PostThreadMessage）3、CEvent对象，可以通过对CEvent的触发状态进行改变，从而实现线程间的通信和同步。

UICollectView 自定义layout实现步骤：
1覆写prepareLayout方法，并在里面事先就计算好必要的布局信息并存储起来。
2基于prepareLayout方法中的布局信息，使用collectionViewContentSize方法返回UICollectionView的内容尺寸。
3使用layoutAttributesForElementsInRect:方法返回指定区域cell、Supplementary View和Decoration View的布局属性。

__block修饰的内容可以修改：Block不允许修改外部变量的值，这里所说的外部变量的值，指的是栈中指针的内存地址。__block 所起到的作用就是只要观察到该变量被 block 所持有，就将“外部变量”在栈中的内存地址放到了堆中。进而在block内部也可以修改外部变量的值。

LLDB : [http://ios.jobbole.com/83393/](http://ios.jobbole.com/83393/)
