# IOS 多线程十问

![2969810-f4730965d0802151.jpg](https://upload-images.jianshu.io/upload_images/2969810-f4730965d0802151.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

本文是iOS多线程知识小结，通过本文，你可以了解到：

- 进程和线程的区别？同步异步的区别？并行和并发的区别？
- 多线程的原理、多线程优缺点、多线程的应用？
- NSOperation 和 GCD 的区别？以及各自使用场景？
- 并发编程中有哪些常见问题？
- 说说你对线程安全的理解？
- 自旋锁与互斥锁有什么区别，各自使用场景？
- Dispatch Semaphore具体使用场景？
- dispatch_barrier_async具体使用场景？
- 使用 GCD 如何实现这个需求：A、B、C 三个任务并发，完成后执行任务 D。
- 什么是线程死锁？死锁如何产生？如何避免线程死锁？

## **::一问：进程和线程的区别？同步异步的区别？并行和并发的区别？::**

\####进程和线程的区别

- 进程是资源的分配和调度的一个独立单元，而线程是CPU调度的基本单元
- 同一个进程中可以包括多个线程，并且线程共享整个进程的资源（寄存器、堆栈、上下文），一个进行至少包括一个线程。
- 进程的创建调用fork或者vfork，而线程的创建调用pthread_create，进程结束后它拥有的所有线程都将销毁，而线程的结束不会影响同个进程中的其他线程的结束
- 线程是轻两级的进程，它的创建和销毁所需要的时间比进程小很多，所有操作系统中的执行功能都是创建线程去完成的
- 线程中执行时一般都要进行同步和互斥，因为他们共享同一进程的所有资源
- 线程有自己的私有属性TCB，线程id，寄存器、硬件上下文，而进程也有自己的私有属性进程控制块PCB，这些私有属性是不被共享的，用来标示一个进程或一个线程的标志

\####同步和异步的区别

- 同步(synchronous)：进程之间的关系不是相互排斥临界资源的关系，而是相互依赖的关系。进一步的说明：就是前一个进程的输出作为后一个进程的输入，当第一个进程没有输出时第二个进程必须等待。具有同步关系的一组并发进程相互发送的信息称为消息或事件。
- 异步(asynchronous)：异步和同步是相对的，同步就是顺序执行，执行完一个再执行下一个，需要等待、协调运行。异步就是彼此独立,在等待某事件的过程中继续做自己的事，不需要等待这一事件完成后再工作。线程就是实现异步的一个方式。异步是让调用方法的主线程不需要同步等待另一线程的完成，从而可以让主线程干其它的事情。

\####并行和并发的区别

- 并发：在操作系统中，是指一个时间段中有几个程序都处于已启动运行到运行完毕之间，且这个几个程序都是在同一个处理机上运行。其中两种并发关系分别是同步和互斥。
- 互斥：进程间相互排斥的使用临界资源的现象
- 同步：进程之间的关系不是相互排斥临界资源的关系，而是相互依赖的关系。进一步说明就是前一个进程的输出作为后一个进程的输入，当第一个进程没有输出时第二个进程必须等待。具有同步关系的一组并发进程相互发送的信息称为消息或事件。

其中并发又有伪并发和真并发，伪并发是指单核处理器的并发，真并发是指多核处理器的并发。

- 并行(parallelism)：在单处理器中多道程序设计系统中，进程被交替执行，表现出一种并发的外部特种；在多处理器系统中，进程不仅可以交替执行，而且可以重叠执行。在多处理器上的程序才可实现并行处理。从而可知，并行是针对多处理器而言的。并行是同时发生的多个并发事件，具有并发的含义，但并发不一定并行，也亦是说并发事件之间不一定要同一时刻发生。

## **::二问：多线程的原理、多线程优缺点、多线程的应用？::**

\####多线程的原理

- 同一时间，CPU只能处理1条线程，只有1条线程在工作（执行）
- 多线程并发（同时）执行，其实是CPU快速地在多条线程之间调度（切换）
- 如果CPU调度线程的时间足够快，就造成了多线程并发执行的假象

\####多线程的优点

- 能适当提高程序的执行效率
- 能适当提高资源利用率（CPU、内存利用率）

\####多线程的缺点

- 开启线程需要占用一定的内存空间（默认情况下，主线程占用1M，子线程占用512KB），如果开启大量的线程，会占用大量的内存空间，降低程序的性能
- 线程越多，CPU在调度线程上的开销就越大
- 程序设计更加复杂：比如线程之间的通信、多线程的数据共享

\####多线程的应用

#### 一、共享资源

共享资源 : 就是内存中的一块资源同时被多个进程所访问，而每个进程可能会对该资源的数据进行修改

问题 : 如果 线程A 访问了某块资源 C，并且修改了其中的数据，此时 线程B 也访问了 资源C，并且也对 C 中的数据进行了修改；那么等到 线程A 和 线程B 执行结束后，此时，资源C 中的数据就并不是最初的设置了

此时，如果用户想获取 被 线程A 修改的 资源C 的数据，但是 资源C 的数据也被 线程B 修改了，所以获得的是错误的数据；如果用户想获取 被 线程B 修改的 资源C 的数据，但是 资源C 的数据也被 线程A 修改了，所以获得的是错误的数据

下面看代码，以出售火车票为例

建立火车票数据模型

```other
// LHTicketModal.h
#import <Foundation/Foundation.h>
@interface LHTicketModal : NSObject
@property (nonatomic) NSUInteger ticketCount; // 剩余票数
@property (nonatomic) NSUInteger ticketSaleCount; // 卖出数量
@end
// LHTicketModal.m
#import "LHTicketModal.h"
static const NSUInteger kTicketTotalCount = 50;
@implementation LHTicketModal
- (instancetype)init {
  self = [super init];
  if (self) {
  // 初始化剩余票数应该为总数
  _ticketCount = kTicketTotalCount;
  // 初始化卖出票数应该为 0
  _ticketSaleCount = 0;
  }
  return self;
}
@end
```

UIViewController 代码，声明两个线程，分别代表两个地点

```other
// LHSharedViewController.h
#import "LHSharedViewController.h"
#import "LHTicketModal.h"
@interface LHSharedViewController ()
// 车票模型
@property (nonatomic, strong) LHTicketModal * ticket;
// 线程对象
@property (nonatomic, strong) NSThread * threadBJ;
@property (nonatomic, strong) NSThread * threadSH;
@end
// LHSharedViewController.m
@implementation LHSharedViewController
- (void)viewDidLoad {
  [super viewDidLoad];
  // 1\. 初始化 票数模型对象
  _ticket = [[LHTicketModal alloc] init];
  // 2\. 初始化 线程对象 并设置线程名
  _threadBJ = [[NSThread alloc] initWithTarget:self selector:@selector(sellTickets:) object:@"北京卖出"];
  _threadBJ.name = @"北京";
  _threadSH = [[NSThread alloc] initWithTarget:self selector:@selector(sellTickets:) object:@"上海卖出"];
  _threadSH.name = @"上海";
}
// 线程执行的方法
- (void)sellTickets:(NSString *)str {
  while (YES) {
  // 判断剩余量是否大于 0，大于 0 才可以卖出
  if (_ticket.ticketCount > 0) {
  // 暂停一段时间
  [NSThread sleepForTimeInterval:0.2];
  // 卖出一张票，剩余票数减 1，卖出票数加 1
  _ticket.ticketCount--;
  _ticket.ticketSaleCount++;
  // 获取当前进程
  NSThread * currentThread = [NSThread currentThread];
  NSLog(@"%@ 卖了一张票，还剩 %lu 张票", currentThread.name, _ticket.ticketCount);
  }
  }
}
// 按钮的动作方法
- (IBAction)startSaleBtnClick:(id)sender {
  // 开启线程
  [_threadBJ start];
  [_threadSH start];
}
```

点击按钮后，线程开始运行，结果如图

![2969810-1c6d59700dbe073b.png](https://upload-images.jianshu.io/upload_images/2969810-1c6d59700dbe073b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

看一看出，共享资源同时被多个线程访问并修改，导致用户取得了错误的数据

\#####那么解决这种情况的办法就是 加锁

加锁是指对一段代码进行加锁，加锁之后，若一个线程已经在对共享资源的数据修改，此时就不会再有其他线程来访问该资源进行修改，直至当前线程已经结束修改资源的代码时，其他进程才可以对其进行修改

可以把共享资源看做一个房间，线程看做人；当一个人进入房间之后就会锁门（加锁），对房间进行各种布置，此时其他人是进不来的，因为没有钥匙；直至当前的人出房间，其他的人才可以进房间进行布置

加锁的方式有多种，这里介绍两种：

**方式一 : 使用  @synchronized (加锁对象) {} 关键字**

```other
只需修改上述代码的  sellTickets 方法，其余不变，这里将其方法名改为  sellTickets_v2
- (void)sellTickets_v2:(NSString *)str {
  while (YES) {
  // 对当前对象加锁
  @synchronized (self) {
  // 判断剩余量是否大于 0，大于 0 才可以卖出
  if (_ticket.ticketCount > 0) {
  // 暂停一段时间
  [NSThread sleepForTimeInterval:0.2];
  // 卖出一张票，剩余票数减 1，卖出票数加 1
  _ticket.ticketCount--;
  _ticket.ticketSaleCount++;
  // 获取当前进程
  NSThread * currentThread = [NSThread currentThread];
  NSLog(@"%@ 卖了一张票，还剩 %lu 张票", currentThread.name, _ticket.ticketCount);
  }
  }
  }
}
```

再次运行程序，点击按钮。结果如图

![2969810-920c39810389317d.png](https://upload-images.jianshu.io/upload_images/2969810-920c39810389317d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**方式二 : 使用 NSCondition 类**

在 类扩展中声明并在 viewDidLoad 方法中初始化

```other
- (void)sellTickets_v3:(NSString *)str {
  while (YES) {
  // 使用 NSCondition 加锁
  [_ticketCondition lock];
  // 判断剩余量是否大于 0，大于 0 才可以卖出
  if (_ticket.ticketCount > 0) {
  // 暂停一段时间
  [NSThread sleepForTimeInterval:0.2];
  // 卖出一张票，剩余票数减 1，卖出票数加 1
  _ticket.ticketCount--;
  _ticket.ticketSaleCount++;
  // 获取当前进程
  NSThread * currentThread = [NSThread currentThread];
  NSLog(@"%@ 卖了一张票，还剩 %lu 张票", currentThread.name, _ticket.ticketCount);
  }
  // 使用 NSCondition 解锁
  [_ticketCondition unlock];
  }
}
```

运行结果和上述一样

使用 NSCondition 时，将加锁的代码放在 loac 和 unlock 中间

总结 :

1. 互斥锁的优缺点

优点 : 有效防止因多线程抢夺资源造成的数据安全问题

缺点 : 需要消耗大量 CPU 资源

2. 互斥锁使用的前提

- 多个线程抢夺同一资源
- 线程同步，互斥锁就是使用了线程同步

#### 二、线程通信

通常， 一个线程不应该单独存在，应该和其他线程之间有关系

例如 : 一个线程完成了自己的任务后需要切换到另一个线程完成某个任务；或者 一个线程将数据传递给另一个线程

看代码，现在 xib 中拖入一个 按钮，并设置其动作方法，如下:

```other
- (IBAction)btn1Click:(id)sender {
  // 1\. 获取主线程
  NSLog(@"mainThread --- %@", [NSThread mainThread]);
  // 2\. 创建子线程并完成指定的任务
  [self performSelectorInBackground:@selector(run1:) withObject:@"子线程中执行方法"];
}
- (void)run1:(NSString *)str {
  // 1\. 获取当前线程
  NSThread * currentThread = [NSThread currentThread];
  NSLog(@"currentThread --- %@ --- %@", currentThread, str);
  // 2\. 回到主线程中执行指定的方法
  [self performSelectorOnMainThread:@selector(backMainThread:) withObject:@"回到主线程" waitUntilDone:YES];
}
- (void)backMainThread:(NSString *)str {
  // 1\. 获取当前线程（主线程）
  NSThread * currentThread = [NSThread currentThread];
  NSLog(@"currentThread --- %@ --- %@", currentThread, str);
}
```

这里通过 performSelectorInBackground: withObject: 方法创建一个子线程并完成指定的任务（run1:），然后在子线程中又通过

performSelectorOnMainThread: withObject: waitUntilDone: 方法回到主线程中完成指定的任务（backMainThread:）

点击按钮，运行结果如下:

![2969810-ff416e3b3d3d6b3e.png](https://upload-images.jianshu.io/upload_images/2969810-ff416e3b3d3d6b3e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 三、线程的状态

1. 当一个线程对象创建并开启后，它就会被放到线程调度池中，等待系统调度；如图

![2969810-82e57fd1a8c6ca50.png](https://upload-images.jianshu.io/upload_images/2969810-82e57fd1a8c6ca50.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

2. 当正在运行的线程被阻塞时，就会被移出 可调度线程池，此时不可再调度它

![2969810-bc359e2c55cc571b.png](https://upload-images.jianshu.io/upload_images/2969810-bc359e2c55cc571b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

3. 当线程正常结束，异常退出，强制退出时都会导致该线程死亡，死亡的线程会从内存中移除，无法调度

![2969810-074d2fad16b551c0.png](https://upload-images.jianshu.io/upload_images/2969810-074d2fad16b551c0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```other
1 #import "LHThreadStateViewController.h"
 2
 3 @interface LHThreadStateViewController ()
 4
 5 @property (nonatomic, strong) NSThread * thread;
 6
 7 @end
 8
 9 @implementation LHThreadStateViewController
10
11 - (void)viewDidLoad {
12
13     [super viewDidLoad];
14
15     // 1. 初始化线程对象
16     _thread = [[NSThread alloc] initWithTarget:self selector:@selector(runThread:) object:@"子线程执行任务"];
17
18     _thread.name = @"线程A";
19
20 }
21
22 - (IBAction)startThreadClick:(id)sender {
23
24     // 1. 获取主线程
25     NSLog(@"mainThread --- %@", [NSThread mainThread]);
26
27     // 线程开始
28     [_thread start];
29
30 }
31
32 - (void)runThread:(NSString *)str {
33
34     // 1. 获取当前线程
35     NSThread * currentThread = [NSThread currentThread];
36
37     NSLog(@"currentThread --- %@", currentThread);
38
39     // 2. 阻塞 2s
40     NSLog(@"线程开始阻塞 2s");
41
42     [NSThread sleepForTimeInterval:2.0];
43
44     NSLog(@"线程结束阻塞 2s");
45
46     // 3. 阻塞 4s
47     NSLog(@"线程开始阻塞 4s");
48
49     // 创建 NSDate 对象，并从当前开始推后 4s
50     NSDate * date = [NSDate dateWithTimeIntervalSinceNow:4.0];
51
52     [NSThread sleepUntilDate:date];
53
54     NSLog(@"线程结束阻塞 4s");
55
56     // 4. 退出线程
57     for (int i = 0; i < 10; ++i) {
58
59         if (i == 7) {
60
61             // 结束当前线程，之后的语句不会再执行
62             [NSThread exit];
63
64             NSLog(@"线程结束");
65
66         }
67
68         NSLog(@"%i --- %@", i, currentThread);
69
70     }
71
72 }
```

42 行使用 sleepForTimeInterval: 方法使得当前线程阻塞指定的时间

52 行使用 sleepUntilDate: 方法使得当前线程阻塞 参数中设定的时间；该参数必须是 NSDate 对象，并且以当前时间作为基准

62 行使用 exit 方法退出当前线程，并且退出后，该方法之后的语句不再会执行，上述中即 64 行代码不会执行

点击按钮，运行结果如下:

![2969810-8309ca4989db533d.png](https://upload-images.jianshu.io/upload_images/2969810-8309ca4989db533d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

注意 : 当 执行 exit 方法后，表示当前的线程已经死亡，即从内存中移除了，若此时再点击按钮（该线程在内存中已经没有空间了），程序会崩溃，并报以下的错误:

![2969810-7ba6ff8107377705.png](https://upload-images.jianshu.io/upload_images/2969810-7ba6ff8107377705.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## ::三问：NSOperation 和 GCD 的区别？以及各自使用场景？::

\####NSOperation 和 GCD 的区别

1. GCD 的核心是 C 语言写的系统服务，执行和操作简单高效，因此 NSOperation 底层也通过 GCD 实现，换个说法就是 NSOperation 是对 GCD 更高层次的抽象，这是他们之间最本质的区别。因此如果希望自定义任务，建议使用 NSOperation；
2. 依赖关系，NSOperation 可以设置两个 NSOperation 之间的依赖，第二个任务依赖于第一个任务完成执行，GCD 无法设置依赖关系，不过可以通过dispatch_barrier_async来实现这种效果；
3. KVO(键值对观察)，NSOperation 容易判断 Operation 当前的状态(是否执行，是否取消)，对此 GCD 无法通过 KVO 进行判断；
4. 优先级，NSOperation 可以设置自身的优先级，但是优先级高的不一定先执行，GCD 只能设置队列的优先级，无法在执行的 block 设置优先级；
5. 继承，NSOperation 是一个抽象类，实际开发中常用的两个类是 NSInvocationOperation 和 NSBlockOperation ，同样我们可以自定义 NSOperation，GCD 执行任务可以自由组装，没有继承那么高的代码复用度；
6. 效率，直接使用 GCD 效率确实会更高效，NSOperation 会多一点开销，但是通过 NSOperation 可以获得依赖，优先级，继承，键值对观察这些优势，相对于多的那么一点开销确实很划算，鱼和熊掌不可得兼，取舍在于开发者自己；

\####使用场景

- 使用NSOperation的情况：各个操作之间有依赖关系、操作需要取消暂停、并发管理、控制操作之间优先级，限制同时能执行的线程数量.让线程在某时刻停止/继续等。
- 使用GCD的情况：一般的需求很简单的多线程操作，用GCD都可以了，简单高效。
- 从编程原则来说，一般我们需要尽可能的使用高等级、封装完美的API，在必须时才使用底层API。
- 当需求简单，简洁的GCD或许是个更好的选择，而Operation queue 为我们提供能更多的选择。

## **::四问：并发编程中有哪些常见问题？::**

在并发编程中，一般会面对这样的三个问题：竞态条件、优先倒置、死锁问题。针对 iOS 开发，它们的具体定义为：

- **竞态条件（Race Condition）**。指两个或两个以上线程对共享的数据进行读写操作时，最终的数据结果不确定的情况。例如以下代码：

```other
var num = 0
DispatchQueue.global().async {
for _ in 1…10000 {
num += 1
}
}
for _ in 1…10000 {
num += 1
}
```

最后的计算结果 num 很有可能小于 20000，因为其操作为非原子操作。在上述两个线程对num进行读写时其值会随着进程执行顺序的不同而产生不同结果。

竞态条件一般发生在多个线程对同一个资源进行读写时。解决方法有两个，第一是串行队列加同步操作，无论读写，指定时间只能优先做当前唯一操作，这样就保证了读写的安全。其缺点是速度慢，尤其在大量读写操作发生时，每次只能做单个读或写操作的效率实在太低。另一个方法是，用并发队列和 barrier flag，这样保证此时所有并发队列只进行当前唯一的写操作（类似将并发队列暂时转为串行队列），而无视其他操作。

- **优先倒置（Priority Inverstion）**。指低优先级的任务会因为各种原因先于高优先级任务执行。例如以下代码：

```other
var highPriorityQueue = DispatchQueue.global(qos: .userInitiated)
var lowPriorityQueue = DispatchQueue.global(qos: .utility)
let semaphore = DispatchSemaphore(value: 1)
lowPriorityQueue.async {
semaphore.wait()
for i in 0...10 {
print(i)
}
semaphore.signal()
}
highPriorityQueue.async {
semaphore.wait()
for i in 11...20 {
print(i)
}
semaphore.signal()
}
```

上述代码如果没有 semaphore，高优先权的 highPriorityQueue 会优先执行，所以程序会优先打印完 11 到 20。而加了 semaphore 之后，低优先权的 lowPriorityQueue 会先挂起 semaphore，高优先权的highPriorityQueue 就只有等 semaphore 被释放才能再执行打印。

也就是说，低优先权的线程可以锁上某种高优先权线程需要的资源，从而优于迫使高优先权的线程等待低优先权的线程，这就叫做优先倒置。其对应的解决方法是，对同一个资源不同队列的操作，我们应该用同一个QoS指定其优先级。

- **死锁问题（Dead Lock）**。指两个或两个以上的线程，它们之间互相等待彼此停止执行，以获得某种资源，但是没有一方会提前退出的情况。iOS 中有个经典的例子就是两个 Operation 互相依赖：

```other
let operationA = Operation()
let operationB = Operation()
operationA.addDependency(operationB)
operationB.addDependency(operationA)
```

还有一种经典的情况，就是在对同一个串行队列中进行异步、同步嵌套：

```other
serialQueue.async {
serialQueue.sync {
}
}
```

因为串行队列一次只能执行一个任务，所以首先它会把异步 block 中的任务派发执行，当进入到 block 中时，同步操作意味着阻塞当前队列 。而此时外部 block 正在等待内部 block 操作完成，而内部block 又阻塞其操作完成，即内部 block 在等待外部 block 操作完成。所以串行队列自己等待自己释放资源，构成死锁。

对于死锁问题的解决方法是，注意Operation的依赖添加，以及谨慎使用同步操作。其实聪明的读者应该已经发现，在主线程使用同步操作是一定会构成死锁的，所以我个人建议在串行队列中不要使用同步操作。

尽管我们已经知道了并发编程中的问题，以及其对应方法。但是日常开发中，我们怎样及时发现这些问题呢？其实 Xcode 提供了一个非常便利的工具 —— Thread Sanitizer (TSan)。在Schemes中勾选之后，TSan就会将所有的并发问题在 Runtime 中显示出来，如下图：

![]()

这里我们有7个线程问题，TSan清晰地告诉了我们这是读写问题，展开之后会告诉我们具体触发代码，十分方便。16年的WWDC上，苹果也郑重向大家宣告，如果有并发问题，请记得用 TSan。

## **::五问：说说你对线程安全的理解？::**

**线程安全** 就是多线程访问时，采用了加锁机制，当一个线程访问该类的某个数据时，进行保护，其他线程不能进行访问直到该线程读取完，其他线程才可使用。不会出现数据不一致或者数据污染。

**线程不安全** 就是不提供数据访问保护，有可能出现多个线程先后更改数据造成所得到的数据是脏数据

如果你的代码所在的进程中有多个线程在同时运行，而这些线程可能会同时运行这段代码。如果每次运行结果和单线程运行的结果是一样的，而且其他的[变量](http://baike.baidu.com/view/296689.htm)的值也和预期的是一样的，就是线程安全的。

线程安全问题都是由全局变量及静态变量引起的。

若每个线程中对全局变量、静态变量只有读操作，而无写操作，一般来说，这个全局变量是线程安全的；若有多个线程同时执行写操作，一般都需要考虑线程同步，否则的话就可能影响线程安全。

## **::六问：自旋锁与互斥锁有什么区别，各自使用场景？::**

\####互斥锁

在编程中，引入对象互斥锁的概念，来保证共享数据操作的完整性。每个对象都对应于一个可称为“互斥锁”的标记，这个标记用来保证在任一时刻，只能有一个线程访问对象。

\####自旋锁

何谓自旋锁？它是为实现保护共享资源而提出一种锁机制。其实，自旋锁与互斥锁比较类似，它们都是为了解决对某项资源的互斥使用。无论是互斥锁，还是自旋锁，在任何时刻，最多只能有一个保持者，也就说，在任何时刻最多只能有一个执行单元获得锁。但是两者在调度机制上略有不同。对于互斥锁，如果资源已经被占用，资源申请者只能进入睡眠状态。但是自旋锁不会引起调用者睡眠，如果自旋锁已经被别的执行单元保持，调用者就一直循环在那里看是否该自旋锁的保持者已经释放了锁，"自旋"一词就是因此而得名。

\####相同点：

- 自旋锁和互斥锁是多线程程序中的重要概念。 它们被用来锁住一些共享资源， 以防止并发访问这些共享数据时可能导致的数据不一致问题。

\####不同点：

1. 自旋锁不会睡眠，互斥锁会有睡眠，因此自旋锁效率高于互斥锁。
2. 由于一直查询，所以自旋锁一直占用cpu，互斥锁不会，自旋锁导致cpu使用效率低。
3. 自旋锁容易造成死锁，当递归调用时有可能造成死锁，调用有些其他函数也可能造成死锁。

\####自旋锁应用场景

- 在单核/单CPU系统上使用自旋锁是没用的， 因为当线程尝试获取自旋锁不成功的时候会一直尝试， 这会一直占用CPU， 其它线程不可能运行， 因为其他线程不能运行， 这个锁也就不会被解锁。 换句话说， 在单核/单CPU的系统上，自旋锁除了浪费时间没有一点好处。 这时如果这个线程(记为A)可以休眠， 其它线程可以立即运行， 因为其它有可能解锁， 那么线程A可能在唤醒后继续执行。
- 在多核/多CPU的系统上， 特别是大量的线程只会短时间的持有锁的时候， 在使线程睡眠和唤醒线程上浪费大量的时间， 也许会显著降低程序的运行性能。 使用自旋锁， 线程可以充分利用调度程序分配的时间片(经常阻塞很短的时间， 不用休眠， 然后马上继续它们的工作了)， 以达到更高的处理能力和吞吐量。

### **::七问：Dispatch Semaphore具体使用场景？::**

GCD 中的信号量是指 Dispatch Semaphore，是持有计数的信号。类似于过高速路收费站的栏杆。可以通过时，打开栏杆，不可以通过时，关闭栏杆。在 Dispatch Semaphore 中，使用计数来完成这个功能，计数为0时等待，不可通过。计数为1或大于1时，计数减1且不等待，可通过。

Dispatch Semaphore 提供了三个函数。

- `dispatch_semaphore_create：`创建一个Semaphore并初始化信号的总量
- `dispatch_semaphore_signal：`发送一个信号，让信号总量加1
- `dispatch_semaphore_wait：`可以使总信号量减1，当信号总量为0时就会一直等待（阻塞所在线程），否则就可以正常执行。

> 注意：信号量的使用前提是：想清楚你需要处理哪个线程等待（阻塞），又要哪个线程继续执行，然后使用信号量。 **Dispatch Semaphore 在实际开发中主要用于：**

- 保持线程同步，将异步执行任务转换为同步执行任务
- 保证线程安全，为线程加锁

#### Dispatch Semaphore线程同步

我们在开发中，会遇到这样的需求：异步执行耗时任务，并使用异步执行的结果进行一些额外的操作。换句话说，相当于，将将异步执行任务转换为同步执行任务。比如说：AFNetworking 中 AFURLSessionManager.m 里面的 tasksForKeyPath: 方法。通过引入信号量的方式，等待异步执行任务结果，获取到 tasks，然后再返回该 tasks。

```other
- (NSArray *)tasksForKeyPath:(NSString *)keyPath {
__block NSArray *tasks = nil;
dispatch_semaphore_t semaphore = dispatch_semaphore_create(0);
[self.session getTasksWithCompletionHandler:^(NSArray *dataTasks, NSArray *uploadTasks, NSArray *downloadTasks) {
if ([keyPath isEqualToString:NSStringFromSelector(@selector(dataTasks))]) {
tasks = dataTasks;
} else if ([keyPath isEqualToString:NSStringFromSelector(@selector(uploadTasks))]) {
tasks = uploadTasks;
} else if ([keyPath isEqualToString:NSStringFromSelector(@selector(downloadTasks))]) {
tasks = downloadTasks;
} else if ([keyPath isEqualToString:NSStringFromSelector(@selector(tasks))]) {
tasks = [@[dataTasks, uploadTasks, downloadTasks] valueForKeyPath:@"@unionOfArrays.self"];
}
dispatch_semaphore_signal(semaphore);
}];
dispatch_semaphore_wait(semaphore, DISPATCH_TIME_FOREVER);
return tasks;
}
```

下面，我们来利用 Dispatch Semaphore 实现线程同步，将异步执行任务转换为同步执行任务。

```other
/**
* semaphore 线程同步
*/
- (void)semaphoreSync {
NSLog(@"currentThread---%@",[NSThread currentThread]); // 打印当前线程
NSLog(@"semaphore---begin");
dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
dispatch_semaphore_t semaphore = dispatch_semaphore_create(0);
__block int number = 0;
dispatch_async(queue, ^{
// 追加任务1
[NSThread sleepForTimeInterval:2]; // 模拟耗时操作
NSLog(@"1---%@",[NSThread currentThread]); // 打印当前线程
number = 100;
dispatch_semaphore_signal(semaphore);
});
dispatch_semaphore_wait(semaphore, DISPATCH_TIME_FOREVER);
NSLog(@"semaphore---end,number = %zd",number);
}
```

输出结果：

```other
2018-02-23 22:22:26.521665+0800 YSC-GCD-demo[20642:5246341] currentThread---<NSThread: 0x60400006bc80>{number = 1, name = main}
2018-02-23 22:22:26.521869+0800 YSC-GCD-demo[20642:5246341] semaphore---begin
2018-02-23 22:22:28.526841+0800 YSC-GCD-demo[20642:5246638] 1---<NSThread: 0x600000272300>{number = 3, name = (null)}
2018-02-23 22:22:28.527030+0800 YSC-GCD-demo[20642:5246341] semaphore---end,number = 100
```

**从 Dispatch Semaphore 实现线程同步的代码可以看到：**

`semaphore---end` 是在执行完 number = 100; 之后才打印的。而且输出结果 number 为 100。

这是因为异步执行不会做任何等待，可以继续执行任务。异步执行将任务1追加到队列之后，不做等待，接着执行`dispatch_semaphore_wait`方法。此时 semaphore  ::0，当前线程进入等待状态。然后，异步任务1开始执行。任务1执行到`dispatch_semaphore_signal`之后，总信号量，此时 semaphore::  1，`dispatch_semaphore_wait`方法使总信号量减1，正在被阻塞的线程（主线程）恢复继续执行。最后打印`semaphore---end`,number = 100。这样就实现了线程同步，将异步执行任务转换为同步执行任务。

### **::使用信号量进行并发控制（GCD实现最大并发数实践）::**

```other
dispatch_queue_t concurrentQueue = dispatch_queue_create("concurrentQueue", DISPATCH_QUEUE_CONCURRENT);
dispatch_queue_t serialQueue = dispatch_queue_create("serialQueue",DISPATCH_QUEUE_SERIAL);
dispatch_semaphore_t semaphore = dispatch_semaphore_create(4);
for (NSInteger i = 0; i < 15; i++) {
dispatch_async(serialQueue, ^{
dispatch_semaphore_wait(semaphore, DISPATCH_TIME_FOREVER);
dispatch_async(concurrentQueue, ^{
NSLog(@"thread:%@开始执行任务%d",[NSThread currentThread],(int)i);
sleep(1);
NSLog(@"thread:%@结束执行任务%d",[NSThread currentThread],(int)i);
dispatch_semaphore_signal(semaphore);});
});
}
NSLog(@"主线程...!");
```

结果

![](https://upload-images.jianshu.io/upload_images/2969810-a9cd9c9838630a56?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### Dispatch Semaphore 线程安全和线程同步（为线程加锁）

考虑线程安全的代码：

```other
/**
* 线程安全：使用 semaphore 加锁
* 初始化火车票数量、卖票窗口(线程安全)、并开始卖票
*/
- (void)initTicketStatusSave {
NSLog(@"currentThread---%@",[NSThread currentThread]); // 打印当前线程
NSLog(@"semaphore---begin");
semaphoreLock = dispatch_semaphore_create(1);
self.ticketSurplusCount = 50;
// queue1 代表北京火车票售卖窗口
dispatch_queue_t queue1 = dispatch_queue_create("net.bujige.testQueue1", DISPATCH_QUEUE_SERIAL);
// queue2 代表上海火车票售卖窗口
dispatch_queue_t queue2 = dispatch_queue_create("net.bujige.testQueue2", DISPATCH_QUEUE_SERIAL);
__weak typeof(self) weakSelf = self;
dispatch_async(queue1, ^{
[weakSelf saleTicketSafe];
});
dispatch_async(queue2, ^{
[weakSelf saleTicketSafe];
});
}
/**
* 售卖火车票(线程安全)
*/
- (void)saleTicketSafe {
while (1) {
// 相当于加锁
dispatch_semaphore_wait(semaphoreLock, DISPATCH_TIME_FOREVER);
if (self.ticketSurplusCount > 0) { //如果还有票，继续售卖
self.ticketSurplusCount--;
NSLog(@"%@", [NSString stringWithFormat:@"剩余票数：%d 窗口：%@", self.ticketSurplusCount, [NSThread currentThread]]);
[NSThread sleepForTimeInterval:0.2];
} else { //如果已卖完，关闭售票窗口
NSLog(@"所有火车票均已售完");
// 相当于解锁
dispatch_semaphore_signal(semaphoreLock);
break;
}
// 相当于解锁
dispatch_semaphore_signal(semaphoreLock);
}
}
```

输出结果为：

```other
2018-02-23 22:32:19.814232+0800 YSC-GCD-demo[20862:5290531] currentThread---<NSThread: 0x6000000783c0>{number = 1, name = main}
2018-02-23 22:32:19.814412+0800 YSC-GCD-demo[20862:5290531] semaphore---begin
2018-02-23 22:32:19.814837+0800 YSC-GCD-demo[20862:5290687] 剩余票数：49 窗口：<NSThread: 0x6040002709c0>{number = 3, name = (null)}
2018-02-23 22:32:20.017745+0800 YSC-GCD-demo[20862:5290689] 剩余票数：48 窗口：<NSThread: 0x60000046c640>{number = 4, name = (null)}
2018-02-23 22:32:20.222039+0800 YSC-GCD-demo[20862:5290687] 剩余票数：47 窗口：<NSThread: 0x6040002709c0>{number = 3, name = (null)}
......
2018-02-23 22:32:29.024817+0800 YSC-GCD-demo[20862:5290689] 剩余票数：4 窗口：<NSThread: 0x60000046c640>{number = 4, name = (null)}
2018-02-23 22:32:29.230110+0800 YSC-GCD-demo[20862:5290687] 剩余票数：3 窗口：<NSThread: 0x6040002709c0>{number = 3, name = (null)}
2018-02-23 22:32:29.433615+0800 YSC-GCD-demo[20862:5290689] 剩余票数：2 窗口：<NSThread: 0x60000046c640>{number = 4, name = (null)}
2018-02-23 22:32:29.637572+0800 YSC-GCD-demo[20862:5290687] 剩余票数：1 窗口：<NSThread: 0x6040002709c0>{number = 3, name = (null)}
2018-02-23 22:32:29.840234+0800 YSC-GCD-demo[20862:5290689] 剩余票数：0 窗口：<NSThread: 0x60000046c640>{number = 4, name = (null)}
2018-02-23 22:32:30.044960+0800 YSC-GCD-demo[20862:5290687] 所有火车票均已售完
2018-02-23 22:32:30.045260+0800 YSC-GCD-demo[20862:5290689] 所有火车票均已售完
```

可以看出，在考虑了线程安全的情况下，使用 `dispatch_semaphore`

机制之后，得到的票数是正确的，没有出现混乱的情况。我们也就解决了多个线程同步的问题。

### **::八问：dispatch_barrier_async具体使用场景？::**

<一>什么是dispatch_barrier_async函数

毫无疑问,dispatch_barrier_async函数的作用与barrier的意思相同,在进程管理中起到一个栅栏的作用,它等待所有位于barrier函数之前的操作执行完毕后执行,并且在barrier函数执行之后,barrier函数之后的操作才会得到执行,该函数需要同dispatch_queue_create函数生成的concurrent Dispatch Queue队列一起使用

![2969810-1c75369a61c000e9.png](https://upload-images.jianshu.io/upload_images/2969810-1c75369a61c000e9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

<二>dispatch_barrier_async函数的作用

1.实现高效率的数据库访问和文件访问

2.避免数据竞争

<三>dispatch_barrier_async实例

```other
/**
* 栅栏方法 dispatch_barrier_async
*/
- (void)barrier {
dispatch_queue_t queue = dispatch_queue_create("net.bujige.testQueue", DISPATCH_QUEUE_CONCURRENT);
dispatch_async(queue, ^{
// 追加任务1
for (int i = 0; i < 2; ++i) {
[NSThread sleepForTimeInterval:2]; // 模拟耗时操作
NSLog(@"1---%@",[NSThread currentThread]); // 打印当前线程
}
});
dispatch_async(queue, ^{
// 追加任务2
for (int i = 0; i < 2; ++i) {
[NSThread sleepForTimeInterval:2]; // 模拟耗时操作
NSLog(@"2---%@",[NSThread currentThread]); // 打印当前线程
}
});
dispatch_barrier_async(queue, ^{
// 追加任务 barrier
for (int i = 0; i < 2; ++i) {
[NSThread sleepForTimeInterval:2]; // 模拟耗时操作
NSLog(@"barrier---%@",[NSThread currentThread]);// 打印当前线程
}
});
dispatch_async(queue, ^{
// 追加任务3
for (int i = 0; i < 2; ++i) {
[NSThread sleepForTimeInterval:2]; // 模拟耗时操作
NSLog(@"3---%@",[NSThread currentThread]); // 打印当前线程
}
});
dispatch_async(queue, ^{
// 追加任务4
for (int i = 0; i < 2; ++i) {
[NSThread sleepForTimeInterval:2]; // 模拟耗时操作
NSLog(@"4---%@",[NSThread currentThread]); // 打印当前线程
}
});
}
```

输出结果：

```other
2018-02-23 20:48:18.297745+0800 YSC-GCD-demo[20188:5059274] 1---<NSThread: 0x600000079d80>{number = 4, name = (null)}
2018-02-23 20:48:18.297745+0800 YSC-GCD-demo[20188:5059273] 2---<NSThread: 0x600000079e00>{number = 3, name = (null)}
2018-02-23 20:48:20.301139+0800 YSC-GCD-demo[20188:5059274] 1---<NSThread: 0x600000079d80>{number = 4, name = (null)}
2018-02-23 20:48:20.301139+0800 YSC-GCD-demo[20188:5059273] 2---<NSThread: 0x600000079e00>{number = 3, name = (null)}
2018-02-23 20:48:22.306290+0800 YSC-GCD-demo[20188:5059274] barrier---<NSThread: 0x600000079d80>{number = 4, name = (null)}
2018-02-23 20:48:24.311655+0800 YSC-GCD-demo[20188:5059274] barrier---<NSThread: 0x600000079d80>{number = 4, name = (null)}
2018-02-23 20:48:26.316943+0800 YSC-GCD-demo[20188:5059273] 4---<NSThread: 0x600000079e00>{number = 3, name = (null)}
2018-02-23 20:48:26.316956+0800 YSC-GCD-demo[20188:5059274] 3---<NSThread: 0x600000079d80>{number = 4, name = (null)}
2018-02-23 20:48:28.320660+0800 YSC-GCD-demo[20188:5059273] 4---<NSThread: 0x600000079e00>{number = 3, name = (null)}
2018-02-23 20:48:28.320649+0800 YSC-GCD-demo[20188:5059274] 3---<NSThread: 0x600000079d80>{number = 4, name = (null)}
```

**在dispatch_barrier_async相关代码执行结果中可以看出：**

- 在执行完栅栏前面的操作之后，才执行栅栏操作，最后再执行栅栏后边的操作。

### **::九问：使用 GCD 如何实现这个需求：A、B、C 三个任务并发，完成后执行任务 D。::**

需要解决这个首先就需要了解dispatch_group_enter 和 dispatch_group_leave。

- dispatch_group_enter 标志着一个任务追加到 group，执行一次，相当于 group 中未执行完毕任务数+1
- dispatch_group_leave 标志着一个任务离开了 group，执行一次，相当于 group 中未执行完毕任务数-1。
- 当 group 中未执行完毕任务数为0的时候，才会使dispatch_group_wait解除阻塞，以及执行追加到dispatch_group_notify中的任务。

```other
dispatch_queue_t globalQueue = dispatch_get_global_queue(0, 0);
dispatch_group_t group = dispatch_group_create();
dispatch_group_enter(group);
[self requestA:^{
NSLog(@"---执行A任务结束---");
dispatch_group_leave(group);
}];
dispatch_group_enter(group);
[self requestB:^{
NSLog(@"---执行B任务结束---");
dispatch_group_leave(group);
}];
dispatch_group_enter(group);
[self requestC:^{
NSLog(@"---执行C任务结束---");
dispatch_group_leave(group);
}];
dispatch_group_notify(group, globalQueue, ^{
[self requestD:^{
NSLog(@"---执行D任务结束---");
}];
});
- (void)requestA:(void(^)(void))block{
NSLog(@"---执行A任务开始---");
dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(1 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
block();
});
}
- (void)requestB:(void(^)(void))block{
NSLog(@"---执行B任务开始---");
dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(2 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
block();
});
}
- (void)requestC:(void(^)(void))block{
NSLog(@"---执行C任务开始---");
dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(3 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
block();
});
}
- (void)requestD:(void(^)(void))block{
NSLog(@"---执行D任务开始---");
dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(4 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
block();
});
}
```

### **::十问：什么是线程死锁？死锁如何产生？如何避免线程死锁？::**

#### 什么是线程死锁

![Image.tiff](https://res.craft.do/user/full/90948131-1a40-e8ea-369f-21447fdfb1dc/doc/63423857-2F70-456B-98F3-D86315748229/D60707D2-475B-4177-BCFE-88508BD9DCD3_2/nXCLNYinxCA16fhczxc2k85KnUJD92qLXcpV3MyfzqEz/Image.tiff)

线程死锁是指由于两个或者多个线程互相持有对方所需要的资源，导致这些线程处于等待状态，无法前往执行。当线程互相持有对方所需要的资源时，会互相等待对方释放资源，如果线程都不主动释放所占有的资源，将产生死锁。

#### 死锁如何产生

#### 场景一

星期日早上十点半，你在公路上开车，这是一条窄路，只能容纳一辆车。这时，迎面又驶来一辆车，你们都走到一半，谁也不想倒回去，于是各不相让，陷入无尽的等待。

#### 场景二

你和她吵架了，谁也不理谁，甚至晚饭时间都各自煮饭。你在炒京酱肉丝，她在做葱烤鲫鱼。炒到一半你发现小葱被她全部拿走了，于是你默默等待她做好菜后再去拿，殊不知她也在等待你炒完菜后来拿酱油。

#### 场景三

你和四个好朋友坐在圆形餐桌旁，你们只做两件事情：吃饭，或者思考。吃饭的时候，你们就停止思考。思考的时候，也停止吃饭。每个人面前有一碗兰州拉面，并且每个人左右两边各有一根筷子。你们必须要拿到两根筷子才能开始吃面。吃完后再放下筷子，让别人可以使用。吃了一会之后，每个人都拿起了自己左手边的筷子，导致每个人都只有一根筷子，并且等待别人吃完放下筷子给自己。可惜，没有人吃到面，所以没有人会放下筷子。（著名的哲学家就餐问题）

#### 场景四

你有两个线程 A 和 B ，各自在加锁的状态下运行。A 持有一部分资源，并且等待 B 线程中的资源以完成自己的工作，而此时 B 线程也在等待 A 中的资源以完成自己的工作。由于他们都是锁定状态，所以他们必须完成了自己的工作后，自己持有的资源才能释放。于是线程无休止地等待，导致死锁。

这些都是程序员在工作或生活中会遇到的问题，人生就像是一个进程，时间是我们的主线程，期间做的每一件事都是开启的一个子线程。当多件事冲突时，并发问题就产生了。上述问题都指向同一类并发问题：死锁。

产生死锁的的四个条件如下：

1、互斥条件：一个资源每次只能被一个进程使用；

2、请求与保持条件：一个进程因请求资源而阻塞时，对已获得的资源保持不放；

3、不剥夺条件：进程已获得的资源，在没使用完之前，不能强行剥夺；

4、循环等待条件：多个进程之间形成一种互相循环等待资源的关系。

并发带来压力，有的人或有的程序，会因为承受不住压力而崩溃，情绪崩溃和程序崩溃没什么两样。当然，不论是做人还是写程序，面对问题时，正确的做法都应是采取策略，解除死锁。

#### 如何避免线程死锁

#### 方案一

你想起书中所言：退一步海阔天空。但你也深知公平好过忍让。正值周赛时间，你摇下车窗，对对面的兄弟喊道：咱来比赛一场力扣周赛，谁输了谁倒出去让另一个人过吧！于是你们打开力扣，开始答题。半小时后，你凭借高超的代码水平 AC 了全部题目。对面司机对你拱手道：技不如人，甘拜下风。于是他倒了回去，让出了自己的一半路。最终你们都得以顺利通行。

#### 方案二

你在炒菜时发现没有小葱，于是你换位思考，想到她会不会也缺少自己用着的材料。虽然她还在和你冷战，但你劝解自己一个大老爷们不应该和女孩子置气，于是你主动把自己用着的所有材料拿给了她。她感受到你设身处地为她着想，大为感动，你们和好如初。之后她为你们两个人一起炒了京酱肉丝和葱烤鲫鱼。

#### 方案三

你和你的朋友们决定给筷子编上号：1~5。规定每个人拿筷子时必须先拿到自己两边的筷子中号码小的那一根，再去拿号码大的那一根。如果小的那一根没有拿到，不能先拿大的。当你们开始吃饭时，由于数字 5 不可能被一个人单独拿到。因为他旁边的另一根筷子编号必定比 5 小，所以不会再出现每个人都拿着一根的无限等待情形。

#### 方案四

你在运行两个线程前，预先将线程 A 和 B 中的资源拷贝一份，让他们不需互相等待对方的资源，于是两个线程都得以顺利运行。

这四种方案分别破坏了上述四个条件之一。

这也是解决死锁问题的四种方法：

1、破坏不剥夺条件：让对面的司机放弃了自己已有的资源。

2、破坏请求与保持条件：在自己需要的材料缺少时，主动放弃自己持有的资源，防止出现互相等待。

3、破坏循环等待条件：由于筷子指定了编号和获取规则，所以每个锁定状态都将按照顺序执行，于是便杜绝了环路等待条件。

4、破坏互斥条件：由于每次使用时都拷贝一份，所以一个资源可以被多个进程使用。

面试中对死锁的考察无外乎以上三种类型。死锁的概念和死锁的四个产生条件是固定的，上文都已经讲到。但解决方案并不止以上列出的四种。

事实上，使用预先拷贝资源解决死锁问题的方案一般并不常用。这是由于拷贝的成本往往很大，并且影响效率。实际工作中较常采用的是第三种方案，通过控制加锁顺序解决死锁：

**加锁顺序**：当多个线程需要相同的一些锁，但是按照不同的顺序加锁，死锁就很容易发生。如果能确保所有的线程都是按照相同的顺序获得锁，那么死锁就不会发生。当然这种方式需要你事先知道所有可能会用到的锁，然而总有些时候是无法预知的。

除此之外，我们还可以通过设置加锁时限或添加死锁检测避免死锁：

**加锁时限**：加上一个超时时间，若一个线程没有在给定的时限内成功获得所有需要的锁，则会进行回退并释放所有已经获得的锁，然后等待一段随机的时间再重试。但是如果有非常多的线程同一时间去竞争同一批资源，就算有超时和回退机制，还是可能会导致这些线程重复地尝试但却始终得不到锁。

**死锁检测**：死锁检测即每当一个线程获得了锁，会在线程和锁相关的数据结构中（ map 、 graph 等）将其记下。除此之外，每当有线程请求锁，也需要记录在这个数据结构中。死锁检测是一个更好的死锁预防机制，它主要是针对那些不可能实现按序加锁并且锁超时也不可行的场景。

其中，死锁检测最出名的算法是由艾兹格·迪杰斯特拉在 1965 年设计的银行家算法，通过记录系统中的资源向量、最大需求矩阵、分配矩阵、需求矩阵，以保证系统只在安全状态下进行资源分配，由此来避免死锁，对于面算法岗的同学一定要对其有所了解。

参考资料：

- 书籍：『Objective-C 高级编程 iOS 与 OS X 多线程和内存管理』
- 博文：[iOS GCD 之 dispatch_semaphore（信号量）](https://link.juejin.im/?target=http%3A%2F%2Fblog.csdn.net%2Fliuyang11908%2Farticle%2Fdetails%2F70757534)
- [iOS多线程：『GCD』详尽总结](https://juejin.im/post/5a90de68f265da4e9b592b40)

