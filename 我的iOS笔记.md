#我的iOS笔记
###[1.语法](#syntax)
###[2.集合](#set)
###[3.UI](#ui)
####[3.1 手动编写UI](#manualui)
####[3.2 autolayout](#autolayout)
###[4.内存管理](#memorymanage)
####[4.1 手动管理](#manual)
####[4.2 Autorelease](#autorelease)
####[4.3 ARC](#arc)
###[5.线程](#thread)
####[5.1 GCD](#gcd)
####[5.2 NSThread](#nsthread)
###[6.并发](#concurrency)
###[7.数据处理](#data)
###[8.其它](#other)
 
###<span id="syntax">1.基本语法</span>
- 与其它编程语言一样有基本类型，比如int，bool，double，float等等。为了64位的问题需要使用OC中定义的类型，例如NSInteger

```
#if __LP64__ || (TARGET_OS_EMBEDDED && !TARGET_OS_IPHONE) || TARGET_OS_WIN32 || NS_BUILD_32_LIKE_64
typedef long NSInteger;
typedef unsigned long NSUInteger;
#else
typedef int NSInteger;
typedef unsigned int NSUInteger;
#endif
```

- 在类的定义上分为Interface和Implement两部分，这个与C的头文件和实现文件是一个概念，只不过写法上还是有区别的。Interface作为头文件的基本写法就是这样的，注意到在@Interface上有一句变量的定义，这个是全局静态变量其用法和C中定义静态变量是一样的，在@Interface的花括号中还定义了两变量，其实@private可有可无，写法与C++是一样的。而property则写在中间，在@Interface花括号区域定义的变量目前发现不能使用类似retain这样的写法。

```
static NSInteger staticIntInFirstViewController;
@interface FirstViewController : UIViewController {
@private
    NSInteger privateInt;
@public
    NSInteger publicInt;
}
- (void)foo:(NSInteger)arg1;
- (void)foo:(BOOL)arg1 withArg2:(NSInteger)arg2;
+ (void)foo;

- (void)printRetain;
@property (nonatomic, retain) NSArray* arr;
@end
```
字段和属性的使用方法如下：

```
FirstViewController* f = [[FirstViewController alloc] init];
staticIntInFirstViewController = 1; // 只要引用了头文件，直接使用即可
f->publicInt = 2; // C++的用法
f.arr;
```



- 而Implement则是实现部分，Implement中可以添加一些只用于.m文件中的变量。

```
@implementation FirstViewController {
    NSInteger I;
}
```

- 类文件的生成：
  
  UI类：右键→New File→选定平台(iOS/OSX等)→Cocoa Class
  分类(category)：右键→New File→选定平台(iOS/OSX等)→Objective-C File→输入名称、选择类型为category，选择基类。<font size=4 color=green>（在这里还可以创建协议(protocol)和类扩展(extension)）</font>

- 类的扩展，类的扩展分为分类(category)和使用()的形式来扩展类。其区别在于category类似一个完整的类文件，有.h和.m。它的文件的名称通常为`原类名+分类名`，比如
`FirstViewController+Test`,其Interface和Implement分别为

```
#import "FirstViewController.h"

@interface FirstViewController (Test)

@end
```
```
#import "FirstViewController+Test.h"

@implementation FirstViewController (Test)

@end
```

而使用()来做的扩展类只是个头文件，形式如下，注意到类名后面有个`()`，这个里面定义的方法还是需要在主类的.m文件中实现。

```
#import "FirstViewController.h"

@interface FirstViewController ()

@end
```

- 函数的定义,"-"为实例方法，"+"类方法（静态方法），语法上基本没有什么特别注意的，只是在函数和变量的明面上应该遵循oc命名的习惯

```
@interface FirstViewController : UIViewController
- (void)foo:(NSInteger)arg1;
- (void)foo:(BOOL)arg1 withArg2:(NSInteger)arg2;
+ (void)foo;
@end
```
- 函数的调用`[xxxInstance foo]`或者实例函数`[xxxClass foo]`

```
FirstViewController *f = [[FirstViewController alloc] init];
[f foo:1];
[f foo:YES withArg2:1];
[FirstViewController foo];
[f release];
```

- 协议(protocol)和代理(delegate)，两者是合起来用，<font size=4 color=green>代理本质上是一个实现了协议的对象的引用。</font>协议的形式为

```
@protocol FirstViewDelegate <NSObject>
- (void)didClickSomeUI;
@end
```
代理的形式为 `@property (nonatomic, assign) id<FirstViewDelegate> delegate;`

需要注意的是：因为是个对象的引用，因此在使用的时候可能出现互相引用的问题，这样在定义property的时候需要定义为弱引用，防止对象因为互相调用而无法被释放。

- 在流程控制上一样使用if else, switch, for, do while等

```
- (void)foo:(NSInteger)arg1 {
  if (arg1 < 1) {
    return;
  }
  NSLog(@"for start");
  for (NSInteger i = 0; i < arg1; i++) {
    if (i % 2 == 0) {
      NSLog(@"%ld", i);
    }
  }

  NSLog(@"while start");
  NSInteger m = 10;
  while (m > 0) {
    NSLog(@"%ld", m);
    m--;
  }

  NSLog(@"do while start");
  do {
    NSLog(@"%ld", m);
    m++;
  } while (m < 10);
  
  switch (arg1) {
   case 1:
       NSLog(@"");
       break;
   case 2: {
       // do some
   } break;
   default:
       break;
   }

}
```


<font color=red>- FIXME 以后想到再补充。</font>

###<span id="set">2.集合</span>
###<span id="ui">3.UI</span>
####<apan id="manualui">3.1 手动编写UI</span>

- 个人认知：手动编写UI在iOS不需要考虑手机屏幕尺寸的时代是很好用的，但是随着多种尺寸屏幕的出现，在自适应上手动编写UI代码就有了一些局限性。
- 手动编写的核心在于创建UI对象，设置frame及各种属性。
- 当然也可以使用xib来拖动UI，然后让UI与代码关联进而编写相关功能。


####<apan id="autolayout">3.2 autolayout</span>

- 目前在Xcode中可以很好的时候用约束(constraints)来实现UI的自适应。
- constraints的使用心得：
 1. 按住ctrl+左键来创建约束
 2. 可以在Constraints下找到具体的约束，进而进行修改。
 
 ![](http://i.imgur.com/Zg0x473.png)
 
 3. 其基本的使用可以对比NGUI，都是距离superview上下左右的距离，与其它View的间距、size等约束。
 4. 需要注意，当存在多层关系时，需要逐层设置，否则效果可能有问题。
 5. 对于constraint不但可以通过视图来设置，还可以编写代码来设置或修改。
 

###<apan id="memorymanage">4.内存管理</span>
####<apan id="manual">4.1 手动管理</span>

- 自己生成的对象自己持有，可以调用`release`方法减少retain数量。
- 非自己生成的对象也可以持有，通过调用`retain`方法可以持有对象，引用数+1。
- 不再需要自己持有的对象要及时释放，注意类中的property要在`- (void)dealloc
`方法中赋值nil，这样写相当于release了。

```
- (void)dealloc
{
    self.arr = nil;
    [super dealloc];
}
```
- 无法释放非自己持有的对象，注意当一个变量持有一次对象后，只能释放一次。也就是说retainCount必须+1和-1对称。

- alloc的实现。其实就是调用calloc方法申请内存和C语言的差不多，只不过对象的头部位有个地址用于存储引用数。而retain、release就是对引用数加减，dealloc则是free掉对象。

####<apan id="autorelease">4.2 Autorelease </span>
- autorelease这个玩意本质上是将对象加入到最近的一个NSAutoreleasePool中，当NSAutoreleasePool销毁时会将对象release。因此这里就有个坑了，如果这个pool很久都不销毁，里面的对象就始终存在，有可能会造成内存不足。

```
NSAutoreleasePool* pool = [[NSAutoreleasePool alloc] init];

// 当调用autorelease方法时，其实是将对象obj放到了pool的一个对象列表中
NSObject* obj = [[[NSObject alloc] init] autorelease];

[pool drain]; // obj会被调用release方法

```

- 注意到main.m中有这样的代码，在最外层就有个autoreleasepool了。

```
int main(int argc, char * argv[]) {
    @autoreleasepool {
        return UIApplicationMain(argc, argv, nil, NSStringFromClass([AppDelegate class]));
    }
}
```


####<apan id="arc">4.3 ARC</span>
- 我的理解
> 所谓ARC是通过编译器和运行时的协作来实现自动管理引用计数，编译器在ARC有效的代码中加入额外的代码来加减引用计数。

- 标识
	1. __strong：默认就是
	2. __weak：需显式使用
	3. __autoreleasing：这个标识一些情况下是不需要显式使用，一些情况下是不需要的，最为致命。
	4. __unsafe\_unretained：个人感觉已经被弃用了，在没有weak的时代使用的东西。

- __strong
> 这个修饰符具有持有对象的功能，与retain类似。它的使用分为如下几种情况：
> 1、`id __strong obj = [[NSObject alloc] init];` 这行代码可以理解为如下代码，可见所谓自动管理，就是在编译出来的代码中对强引用变量调用release方法。
> 
> ```
> id obj = objc_msgSend(NSObject, @selector(alloc));
  objc_msgSend(obj, @selector(init));
  objc_release(obj);
> ```
>
> 2、`id __strong obj = [NSMutableArray array];` 这行代码可以理解为以下代码，很有意思的代码。objc_retainAutoreleasedReturnValue是持有(retain)了一个在autoreleasepool中的对象,而这个对象就是array方法的返回值。
> 
> ```
> id obj = objc_msgSend(NSMutableArray, @selector(array));
  objc_retainAutoreleasedReturnValue(obj);
  objc_release(obj); // 离开作用域后自动释放
> ```
> 
> 与`objc_retainAutoreleasedReturnValue`成对出现的是这个方法`objc_autoreleasedReturnValue`方法，对于NSMutableArray类的array方法可能是这样实现的
> ```
> + (id)array
{
    return [[NSMutableArray alloc] init];
}
> ```
> 
> 它可以理解为如下代码
> 
> ```
> + (id)array
{ 
       id obj = objc_msgSend(NSMutableArray, @selector(alloc));
       objc_msgSend(obj, @selector(init));
       return objc_autoreleasedReturnValue(obj);
}
> ```
> 
> <font color=green>对于外界调用这个方法赋值的变量来说，只是在使用一个autoreleasepool中的对象。在结合`objc_retainAutoreleasedReturnValue`使用时，其实生成的对象并没有进入autoreleasepool，而是直接传递给了使用`objc_retainAutoreleasedReturnValue`方法赋值的变量。</font>

- __weak

	> 1、被__weak修饰的变量的地址会被放入到weak表中，这个表是个k-v形式的，key是对象的地址，value是所有引用了这个对象的变量的地址。
	> 
	> 2、一个对象被释放的过程是个复杂的过程，
	> 
	> ```
	> objc_release -> dealloc(如果引用计数为0) -> _objc_rootDealloc 
	> -> object_dispose -> objc_destructInstance 
	> -> objc_clear_deallocating
	> {
	>   1、用对象地址找到weak表中的value
	>   2、所有变量(weak表中记录了地址)赋值nil
	>   3、从weak表删除记录
	>   4、从引用计数表删除废弃对象的地址为键值的记录
	> } 
	  
	>```
	> 从以上的过程可以看出当对象被销毁后所有引用它的__weak变量都会被赋值为nil，这个过程是比较消耗CPU的，少用。
	> 
	> 3、<font color=green>**使用被__weak修饰的变量就是使用注册到autoreleasepool中的对象**</font>，从以下代码来进行理解这句话，

	> ```
 	  id __weak obj1 = obj;
      NSLog(@"%@", obj1);
	> ```
	> 这句话会大致被编译器翻译成这样
	
	> ```
	id obj；
    objc_initWeak(&obj1, obj);
    id tmp = objc_loadWeakRetaind(&obj1);
    objc_autorelease(tmp);
    NSLog(@"%@", tmp);
    objc_destroyWeak(&obj1);
	> ```
	
	>可以看到为了能够NSLog执行时obj1引用的对象不被销毁，需要将它赋值给一个strong(默认)修饰的临时变量，而这个临时变量需要放到autoreleasepool中，因此存在一个问题，当你多次在一个作用域中多次使用weak修饰的变量，会导致很多临时变量产生而且会放到autoreleasepool中，作用域结束后autoreleasepool有很多工作要做。所以少用weak，一般就是避免循环引用。

- __autoreleasing，核心就是把修饰的变量放入到autoreleasepool中，没啥多说的。

- 注意事项：
  1. 不能使用retain、release、retainCount、autorelease这样方法
  2. 不能使用NSAllocateObject和NSDeallocateObject方法，实际上我根本没用过。
  3. 需要在函数命名时遵守规则，比如alloc\new\copy\mutableCopy必须给与调用者对象持有权限。
  4. 不能使用NSAutoreleasePool，可以用@autoreleasepool替换。
  5. dealloc方法不能显示调用，很明显的例子就是在MRC中写dealloc方法时一定要调用super的dealloc方法，但是在ARC中不行了，不过notificationCenter的删除等处理还是要写在dealloc方法中的，会自动调用。
 
 

###<apan id="thread">5.线程</span>
####<apan id="gcd">5.1 GCD</span>
- GCD是一套多线程库，可以有效的替换NSThread或者NSOperation。它的基本结构是`dispatch_async(queue, block);`参数中的queue可以通过`dispatch_queue_create`或者系统提供的标准dispatch queue。

```
    // 生成一个serial dispatch queue
    dispatch_queue_t serialQueue = dispatch_queue_create("com.demo.sai", NULL);
    // 生成一个concurrent dispatch queue
    dispatch_queue_t concurrentQueue = dispatch_queue_create("com.demo.sai", DISPATCH_QUEUE_CONCURRENT);

    // 生成的dispatch queue需要手动release，注意ARC不会释放dispatch_queue_t类型的变量
    dispatch_release(serialQueue);
    dispatch_release(concurrentQueue);

    // 使用系统已经提供的方法来create queue
    dispatch_queue_t serialQueue2 = dispatch_get_main_queue();
    dispatch_queue_t concurrentQueue2 = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);

    // 因此真正在代码中经常是这样写的
    dispatch_async(dispatch_get_main_queue(), ^{
        NSLog(@"execute in main thread");
    });

    dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
        NSLog(@"execute in a concurrent thread");
    });

```
- GCD中线程分为Serial Dispatch Queue和Concurrent Dispatch Queue，分别为顺序执行和并发执行。在使用dispatch_get_main_queue时获得的是主线程queue，因此它一定是顺序执行的。使用dispatch_get_global_queue获得的queue所能并行的线程数量由系统来确定，并且可以甚至优先级，然后由于XNU内核用于Global Dispatch Queue的线程不保证实时性，因此执行优先级只是大致的判断（说白了就是并不是严格执行的）。

- `dispatch_set_target_queue(dispatch_object_t object, dispatch_queue_t queue);`可以用来改变queue的优先度

```
// 变更queue的priority
dispatch_set_target_queue(concurrentQueue, serialQueue);
```
- 延迟将block添加到queue中，dispatch_after

```
 dispatch_after(dispatch_time(DISPATCH_TIME_NOW, 3 * NSEC_PER_SEC), dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
        NSLog(@"dispatch_after");
    });
```
<font size=3 color=red>**注意这个方法可以用来做指定一个时间后执行某段代码但是这个时间参数并不准确,因为这个只是在一段时间后将block加入到queue中，但是并不意味着马上执行。**</font>

PS:关于用到的时间的宏的说明
>NSEC：纳秒。
>
>USEC：微妙。
>
>SEC：秒
>
>PER：每

>所以：

>NSEC_PER_SEC，每秒有多少纳秒。

>USEC_PER_SEC，每秒有多少毫秒。（注意是指在纳秒的基础上）

>NSEC_PER_USEC，每毫秒有多少纳秒。

>所以，延时1秒可以写成如下几种：

>dispatch_time(DISPATCH_TIME_NOW, 1 * NSEC_PER_SEC);

>dispatch_time(DISPATCH_TIME_NOW, 1000 * USEC_PER_SEC);

>dispatch_time(DISPATCH_TIME_NOW, USEC_PER_SEC * NSEC_PER_USEC);

>最后一个“USEC_PER_SEC * NSEC_PER_USEC”，翻译过来就是“每秒的毫秒数乘以每毫秒的纳秒数”，也就是“每秒的纳秒数”

- `dispatch_group`，当在并发完成一些处理后，可能需要一个节点来完成某个操作，这个操作必须等到之前的处理全部完成，那么就需要用到dispatch_group来实现。配合`dispatch_group`使用的有两个，`dispatch_group_notify `、`dispatch_group_wait`,前者是当处理全部完成后执行，后者是等待一段时间后，用返回值判断是否所有的任务都完成了。当等待时间为`DISPATCH_TIME_FOREVER`时返回值一定会是0。另外，`dispatch_group_wait`会阻塞调用的线程。

```
       dispatch_queue_t groupQueue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
    dispatch_group_t group = dispatch_group_create();

    // 第一个处理
    dispatch_group_async(group, groupQueue, ^{
        NSLog(@"first");
    });

    // 第二个处理
    dispatch_group_async(group, groupQueue, ^{
        NSLog(@"secend");
    });

    // 完成后最终的处理
    dispatch_group_notify(group, groupQueue, ^{
        NSLog(@"done");
    });

    long result = dispatch_group_wait(group, DISPATCH_TIME_FOREVER);
    if (result == 0) {
        NSLog(@"done");
    }
    else {
        NSLog(@"not done");
    }
    // 需要release
    dispatch_release(group);
```
-`dispatch_barrier_async`,在代码中会等待之前加入某个queue的block全部执行完，然后使得concurrent queue变为一个serial queue，只能执行用`dispatch_barrier_async`添加的block，等完成后，queue变回并发。典型应用场景DB read的操作使用`dispatch_async`，当需要write DB时使用`dispatch_barrier_async`，可以保证数据的一致性，也相当于给写操作加了锁（好吧这样说不够严谨）。

-`dispatch_sync`，同步的将block放到某个queue中，执行这个函数的线程会阻塞等待block执行完成。很容易导致死锁，目前看最好别用。

>关于dispatch_sync导致死锁的问题：
>`dispatch_sync(dispatch_get_main_queue, ...)`这样写一定会死锁，dispatch_(a)sync这两个函数本质上是将block放到一个queue中，只不过一个会阻塞当前调用函数的线程，一个不会。
>
>`dispatch_async(dispatch_get_main_queue, ...)`假设是main thread执行这个函数，那么线程不会等待block执行，虽然这个block是在main thread中执行的，最有可能的是在下一个loop中才会执行block。
>
>`dispatch_async(dispatch_get_global_queue(...), ...)`假设是main thread执行这个函数，那么线程不会等待block执行，而是由系统分配另一个线程完成block，这个方式就是典型的多线程。
>
>`dispatch_sync(dispatch_get_main_queue, ...)`假设是main thread执行这个函数，那么线程会等待block执行，但是这个block又是在main thread执行的，导致死锁。
>
>`dispatch_sync(dispatch_get_global_queue(...), ...)`假设是main thread执行这个函数，那么线程会等待block执行，由系统分配另一个线程完成block，这个方式可用，但是最好不用main thread，而是自己创建一个serial queue。

- 最后看一组代码的结果

```
  // 使用系统已经提供的方法来create queue
    dispatch_queue_t serialQueue2 = dispatch_get_main_queue();
    dispatch_queue_t concurrentQueue2 = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);

    // 因此真正在代码中经常是这样写的
    dispatch_async(dispatch_get_main_queue(), ^{
        NSLog(@"execute in main thread");
    });

    dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
        NSLog(@"execute in a concurrent thread");
    });

    // 变更queue的priority
    //    dispatch_set_target_queue(concurrentQueue, serialQueue);

    dispatch_after(dispatch_time(DISPATCH_TIME_NOW, 3ull * NSEC_PER_SEC), dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
        NSLog(@"dispatch_after");
    });

    dispatch_queue_t groupQueue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
    dispatch_group_t group = dispatch_group_create();

    // 第一个处理
    dispatch_group_async(group, groupQueue, ^{
        NSLog(@"first");
    });

    // 第二个处理
    dispatch_group_async(group, groupQueue, ^{
        NSLog(@"secend");
    });

    // 完成后最终的处理
    dispatch_group_notify(group, groupQueue, ^{
        NSLog(@"done");
    });

    long result = dispatch_group_wait(group, DISPATCH_TIME_FOREVER);
    if (result == 0) {
        NSLog(@"result done");
    }
    else {
        NSLog(@"result not done");
    }
    // 需要release
    dispatch_release(group);

    dispatch_async(concurrentQueue2, ^{
        NSLog(@"read1");
    });
    dispatch_async(concurrentQueue2, ^{
        NSLog(@"read2");
    });

    dispatch_barrier_async(concurrentQueue2, ^{
        NSLog(@"write");
    });

    dispatch_async(concurrentQueue2, ^{
        NSLog(@"read3");
    });
    dispatch_async(concurrentQueue2, ^{
        NSLog(@"read4");
    });

```
```
2016-07-09 14:46:28.500 CRHelper[3027:231293] execute in a concurrent thread
2016-07-09 14:46:28.500 CRHelper[3027:231294] first
2016-07-09 14:46:28.500 CRHelper[3027:231297] secend
2016-07-09 14:46:28.502 CRHelper[3027:231297] done
2016-07-09 14:46:28.502 CRHelper[3027:231205] result done
2016-07-09 14:46:28.507 CRHelper[3027:231297] read1
2016-07-09 14:46:28.507 CRHelper[3027:231293] read2
2016-07-09 14:46:28.507 CRHelper[3027:231390] write
2016-07-09 14:46:28.507 CRHelper[3027:231294] read3
2016-07-09 14:46:28.507 CRHelper[3027:231297] read4
2016-07-09 14:46:28.546 CRHelper[3027:231205] execute in main thread
2016-07-09 14:46:31.782 CRHelper[3027:231294] dispatch_after
```
"execute in main thread"这句话在代码顺序中是第一个但是却在倒数第二个输出，因为是在main thread执行的，可以很明确的看出来这个代码是在下一个loop中执行的，还有就是dispatch_after也并不是严格的按照3秒后执行的.

- `dipatch_apply`的应用场合主要是循环在一个queue中调用某个block，可以用于处理集合。这个函数会想wait一样阻塞线程，因此在非主线程中用比较好。

```
    NSArray* arr = [NSArray arrayWithObjects:@1, @2, nil];
    for (NSInteger i = 0; i < [arr count]; i++) {
        NSLog(@"%@", [arr objectAtIndex:i]);
    }
    dispatch_async(concurrentQueue2, ^{
        dispatch_apply([arr count], concurrentQueue2, ^(size_t i) {
            NSLog(@"%@", [arr objectAtIndex:i]);
        });
    });
```

- 拾遗：`dipatch_semaphore`、`dipatch_suspend`、`dipatch_resume`等。基本没用过，懒得写了。
 
####<apan id="nsthread">5.2 NSThread</span>
###<apan id="concurrency">6.并发</span>
###<apan id="data">7.数据处理</span>
###<apan id="other">8.其它</span>