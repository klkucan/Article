#我的iOS笔记
###[1.语法](#syntax)
###[2.集合](#set)
###[3.UI](#ui)
###[4.内存管理](#memorymanage)
####[4.1 手动管理](#manual)
####[4.2 ARC](#arc)
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

- 在类的定义上分为Interface和Implement两部分，这个与C的头文件和实现文件是一个概念，只不过写法上还是有区别的。Interface作为头文件的基本写法就是这样

```
@interface FirstViewController : UIViewController
- (void)foo:(NSInteger)arg1;
- (void)foo:(BOOL)arg1 withArg2:(NSInteger)arg2;
+ (void)foo;

- (void)printRetain;
@property (nonatomic, retain) NSArray* arr;
@end
```
而Implement则是实现部分，Implement中可以添加一些只用于.m文件中的变量。

```
@implementation FirstViewController {
    NSInteger I;
}
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

####<apan id="arc">4.2 ARC</span>
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