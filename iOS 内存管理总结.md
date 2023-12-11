# iOS 内存管理总结

## 引言

本文主要针对iOS内存管理进行总结，相信看过之后一定会让你有所收获。对于内存管理，网上有很多这方面的文章，但论质量可谓良莠不齐。其中的很多总结已经过时，甚至有些内容是似懂非懂的误读。因此，本文立足严谨的治学态度，希望能够尽可能全面地让大家理解内存管理的要点。

#### 本文涵盖的知识点比较多，大家可以根据需要择章节阅读，下面是目录：

- iOS内存管理之分配与分区
- iOS内存管理之MRC与ARC
- 自动释放池Autorelease
- MRC手动管理引用计数
- ARC自动管理引用计数
- iOS内存管理之@property
- 属性修饰符
- 深浅拷贝
- iOS内存管理之循环引用

![]()

### *iOS内存分配与分区*

#### 1. RAM 与 ROM

#### RAM与ROM就是具体的存储空间，统称为存储器

- RAM(random access memory)：运行内存，CPU可以直接访问，读写速度非常快，但是不能掉电存储。它又分为：
- 动态DRAM，速度慢一点，需要定期的刷新(充电),我们常说的内存条就是指它，价格会稍低一点，手机中的运行内存也是指它
- 静态SRAM，速度快，我们常说的一级缓存，二级缓存就是指它，当然价格高一点。
- ROM(read only memory)：存储性内存，可以掉电存储，例如SD卡、Flash(机械磁盘也可以简单的理解为ROM)。用的多的：NandFlash，还有NorFlash，现在用的已经比较少了（两者主要区别是前者空间大，便宜，后者可以直接运行程序，读取速度快）

由于RAM类型不具备掉电存储能力（即一停止供电数据全没了，从新上电后全是乱码，所以需要初始化），所以app程序一般存放于ROM中。RAM的访问速度要远高于ROM，价格也要高。

#### RAM与ROM协同工作

由于RAM不能掉电存储，所以我们的APP程序，刷机包，下载的文件等等，都是在ROM里面存储的。

手机里面使用的ROM基本都是NandFlash，CPU是不能直接访问的，而是需要文件系统/驱动程序(嵌入式中的EMC)将其读到RAM里面，CPU才可以访问。另外，RAM的速度也比NandFlash快。

#### 2. App程序启动

App程序启动，系统会把开启的那个App程序从Flash或ROM里面拷贝到内存（RAM），然后从内存里面执行代码。

另一个原因是CPU不能直接从内存卡里面读取指令（需要Flash驱动等等）。

#### 3. 内存分区：

#### 说到内存分区，内存即指的是RAM

- 栈区（stack）：
- 存放的局部变量、先进后出、一旦出了作用域就会被销毁；函数跳转地址，现场保护等；
- 栈是由编译器自动分配并释放，用来存放函数括弧“{}”中定义的变量, 程序猿不需要管理栈区变量的内存；当函数被调用时，函数带有的参数也会被压入发起调用的进程栈中，待到调用结束后，函数的返回值也会被存放回栈中。由于栈的先进后出特点，所以栈特别方便用来保存/恢复调用现场。可以把栈看成一个临时数据寄存、交换的内存区
- 栈区地址从高到低分配；
- 堆区（heap）：
- 堆区的内存分配使用的是alloc；
- 需要程序猿管理内存；
- ARC的内存的管理，是编译器再编译的时候自动添加 retain、release、autorelease；
- 堆区的地址是从低到高分配
- 全局区／静态区（static）：
- 包括两个部分：未初始化过 、初始化过；
- 也就是说，（全局区／静态区）在内存中是放在一起的，初始化的全局变量和静态变量在一块区域， 未初始化的全局变量和未初始化的静态变量在相邻的另一块区域；
- eg：int a;未初始化的。int a = 10;已初始化的。
- 常量区：常量字符串就是放在这里；还有const常量
- 代码区： 存放App代码；

如下图所示：代码区存放于低地址，栈区存放于高地址。区与区之间并不是连续的。

![2969810-02ab9257745f7503.png](https://upload-images.jianshu.io/upload_images/2969810-02ab9257745f7503.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 4.代码示例：

```other
import "JKViewController.h"
@interface JKViewController ()
@end
@implementation JKViewController
int num = 1;//数据区（全局区/静态区）
NSString str;//BSS区（全局区/静态区）
static NSString str2 = @"string";//静态区（静态初始化区/全局区）
(void)viewDidLoad
{
[super viewDidLoad];
int age;//栈
NSString name = @"xiaoming";//栈
NSString number = @"123";//123在常量区，number在栈上
//分配而来的8字节的区域就在堆中(相当于alloc分配内存)，array在栈中，指向堆区的地址
NSMutableArray *array = [NSMutableArray arrayWithCapacity:1];
}
/ 方法中的num1和num2都在栈中，返回值num也暂存在栈中 /
(int)changenum1:(int)num1 num2:(int)num2{
int num = num1 + num2;
return num;
}
@end
```

#### 5.程序运行举例（CPU RAM ROM之间协同）

首先了解下：虚拟内存与物理内存

手机上的所有程序都是依托操作系统，运行在虚拟内存上的，每一个APP都会以为自己拥有所有的虚拟内存。比如一个手机，它是32位操作系统(一般也是32位总线)，真实的物理内存为2G：

那么他的寻址空间为4G(2的32次方),对于APP来说，它觉得自己拥有4G的内存，虽然这是不可能的(或者说同一时间是不可能的)，但是，操作系统只要保证APP当时用到的地址空间有真实的物理地址对应就可以，APP也不需要知道那对应的2G真实物理内存具体在哪里。不要求4G的虚拟内存同一时间都有真实的物理内存相对应，当然那也是不可能的，因为只有2G物理内存

![]()

在下面的举例中，只考虑虚拟内存

当我们点击手机屏幕APP的Icon启动一个APP（例如微信）时

- 操作系统会为微信开辟4G的虚拟内存空间(开辟真实的物理内存，对应一部分到4G的虚拟内存)
- 操作系统会把存储在ROM里面微信的部分代码(受空间所限，不可能全部拷贝)，拷贝到上一步开辟的4G内存空间的代码区，如上图
- 然后CPU就可以访问RAM来运行微信的程序了

假设通过微信我们下载了一个100M的视频，那么会从服务器一点一点的下载到RAM，然后再从RAM写到ROM存储。这样才能保证，我们关掉微信并再次打开时视频还在

假设隔一段时间，我们要看视频，程序会将它从ROM读到RAM然后解码播放

#### 6.注意事项

- 在iOS中，堆区的内存是应用程序共享的，堆中的内存分配是系统负责的；
- 系统使用一个链表来维护所有已经分配的内存空间（系统仅仅纪录，并不管理具体的内容）；
- 变量使用结束后，需要释放内存，OC中是根据引用计数＝＝0，就说明没有任何变量使用该空间，那么系统将直接收回；
- 当一个app启动后，代码区，常量区，全局区大小已固定，因此指向这些区的指针不会产生崩溃性的错误。而堆区和栈区是时时刻刻变化的（堆的创建销毁，栈的弹入弹出），所以当使用一个指针指向这两个区里面的内存时，一定要注意内存是否已经被释放，否则会产生程序崩溃（也即是野指针报错）。

#### 7.其它操作系统

- iOS是基于UNIX、Android是基于Linux的，在Linux和unix系统中，内存管理的方式基本相同；
- Android应用程序的内存分配也是如此。除此以外，这些应用层的程序使用的都是虚拟内存，它们都是建立在操作系统之上的，只有开发底层驱动或板级支持包时才会接触到物理内存;举例：在嵌入式Linux中，实际的物理地址只有64M甚至更小，但是虚拟内存却可以高达4G;

### *iOS之内存管理方式*

#### 首先明确一点，无论在MRC还是ARC情况下，Objective-C采用的是引用计数式的内存管理方式，这一方式的特点：

- 自己生成的对象，自己持有。例如：NSObject * __strong obj = [[NSObject alloc]init];。
- 非自己生成的对象，自己也能持有。例如：NSMutableArray * __strong array = [NSMutableArray array];。
- 不再需要自己持有对象时释放。
- 无法释放非自己持有的对象。

| **对象操作** | **Objective-C方法**             |
| -------- | ----------------------------- |
| 生成并持有对象  | alloc/new/copy/mutableCopy等方法 |
| 持有对象     | retain方法                      |
| 释放对象     | release方法                     |
| 废弃对象     | dealloc方法                     |

#### 自己生成的对象，自己持有

在iOS内存管理中有四个关键字，alloc、new、copy、mutableCopy，自身使用这些关键字产生对象,那么自身就持有了对象:

```other
// 使用了alloc分配了内存，obj指向了对象，该对象本身引用计数为1,不需要retain
id obj = [[NSObject alloc] init];
// 使用了new分配了内存,objc指向了对象，该对象本身引用计数为1，不需要retain
id obj = [NSObject new];
```

#### 非自己生成的对象，自己也能持有

```other
// NSMutableArray通过类方法array产生了对象(并没有使用alloc、new、copy、mutableCopt来产生对象),因此该对象不属于obj自身产生的
// 因此，需要使用retain方法让对象计数器+1,从而obj可以持有该对象(尽管该对象不是他产生的)
id obj = [NSMutableArray array];
[obj retain];
```

#### 不再需要自己持有对象时释放

```other
id obj = [NSMutableArray array];
[obj retain];
// 当obj不在需要持有的对象，那么，obj应该发送release消息
[obj release];
// 释放了对象还进行释放,会导致奔溃
[obj release];
```

#### 无法释放非自己持有的对象

```other
// 释放一个不属于自己的对象
id obj = [NSMutableArray array];
// obj没有进行retain操作而进行release操作，然后autoreleasePool也会对其进行一次release操作，导致奔溃。后面会讲到autorelease。
[obj release];
```

#### 针对[NSMutableArray array]方法取得的对象存在，自己却不持有对象，底层大致实现:

```other
+ (id)object
{
//自己持有对象
id obj = [[NSObject alloc]init];
[obj autorelease];
//取得的对象存在，但自己不持有对象
return obj;
}
```

使用了autorelease方法，将obj注册到autoreleasePool中，不会立即释放，当pool结束时再自动调用release。这样达到取得的对象存在，自己不持有对象。

#### 自动释放池autorelease

顾名思义：autorelease就是自动释放，它和C语言的局部变量类似，C语言局部变量在程序执行时，超出其作用域时，会被自动废弃。autorelease会像C语言局部变量那样对待对象实例，当超出作用域时，对象实例的release实例方法会被调用。我们可以设定autorelease的作用域。

#### autorelease具体使用方法:

- 生成并持有NSAutoreleasePool对象
- 调用已分配对象的autorelease实例方法
- 废弃NSAutoreleasePool对象

![2969810-cc860a0ddef12629.png](https://upload-images.jianshu.io/upload_images/2969810-cc860a0ddef12629.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

NSAutoreleasePool对象的生命周期就是一个作用域，对于所有调用过autorelease实例方法的对象，在废弃NSAutoreleasePool对象时，都会调用其release实例方法。如上图所示。

用源代码表示如下：

```other
NSAutoreleasePool *pool = [[NSAutoreleasePool alloc] init];
id obj = [[NSObject alloc] init];
[obj autorelease];
[pool drain];
```

在[pool drain]调用时，NSAutoreleasePool被销毁，obj的release方法会被触发。obj被释放。

#### 对于autorelease的实现方式，书籍也对比了GNUSetp与苹果实现的方式，现在通过GNUStep源代码来理解苹果的实现。

- #### GNUStep实现

```other
id obj = [[NSObject alloc] init];
[obj autorelease];
```

```other
- (id)autorelease
{
[NSAutoreleasePool addObject:self];
}
+ (void)addObject:(id)anObject
{
NSAutoreleasePool *pool = 取得正在使用的Pool对象;
if (pool != nil) {
[pool addObject:anObject];
}else {
NSLog(@"NSAutoreleasePool非存在状态下使用Pool对象");
}
}
- (void)addObject:(id)anObject
{
[array addObject:anObject];
}
```

从上面可以看出，自动释放池就是通过数组完成的，我们在调用autorelease时最终就是将本对象添加到当前自动释放池的数组

而针对于自动释放池销毁时对数组中的进行一次release操作，见下面：

```other
NSAutoreleasePool *pool = [[NSAutoreleasePool alloc] init];
...
// 当自动释放池销毁时
[pool drain];
```

```other
- (void)drain {
[self dealloc];
}
- (void)dealloc {
[self emptyPool];
[array release];
}
- (void)emptyPool {
for (id obj in array) {
[obj release];
}
}
```

- #### autorelease苹果实现

可以通过objc4库的runtime/objc-arr.mm来确认苹果中autorelease的实现。

```other
class AutoreleasePoolPage
{
static inline void *push()
{
相当于生成或持有NSAutoreleasePool类对象
}
static inline void *pop(void *token)
{
相当于废弃NSAutoreleasePool类对象
releaseAll();
}
static inline id autorelease(id obj)
{
相当于NSAutoreleasePool类的addObject类方法
AutoreleasePoolPage *autoreleasePoolPage = 取得正在使用的AutoreleasePoolPage实例;
autoreleasePoolPage->add(obj);
}
id *add(id obj)
{
将对象追加到内部数组中
}
void releaseAll()
{
调用内部数组中对象的release实例方法
}
};
void *objc_autoreleasePoolPush(void)
{
return AutoreleasePoolPage::push();
}
void objc_autoreleasePoolPage(void *ctxt)
{
AutoreleasePoolPage::pop(ctxt);
}
id *objc_autorelease(id obj)
{
return AutoreleasePoolPage::autorelease(obj);
}
```

我们使用调速器来观察NSAutoreleasePool类方法和autorelease方法的运行过程，如下所示，这些方法调用了关联于objc4库autorelease实现的函数。

```other
//等同于objc_autoreleasePoolPush()
NSAutoreleasePool *pool = [[NSAutoreleasePool alloc] init];
id obj = [[NSObject alloc] init];
//等同于objc_autorelease(obj)
[obj autorelease];
//等同于objc_autoreleasePoolPop(pool)
[pool drain];
```

在iOS程序启动后，主线程会自动创建一个RunLoop，苹果在主线程 RunLoop 里注册了两个 Observer，其回调都是 _wrapRunLoopWithAutoreleasePoolHandler()。

第一个 Observer 监视的事件是 Entry(即将进入Loop)，其回调内会调用 _objc_autoreleasePoolPush() 创建自动释放池。其 order 是-2147483647，优先级最高，保证创建释放池发生在其他所有回调之前。

第二个 Observer 监视了两个事件： BeforeWaiting(准备进入休眠) 时调用_objc_autoreleasePoolPop() 和 _objc_autoreleasePoolPush() 释放旧的池并创建新池；Exit(即将退出Loop) 时调用 _objc_autoreleasePoolPop() 来释放自动释放池。这个 Observer 的 order 是 2147483647，优先级最低，保证其释放池子发生在其他所有回调之后。

在主线程执行的代码，通常是写在诸如事件回调、Timer回调内的。这些回调会被 RunLoop 创建好的 AutoreleasePool 环绕着，所以不会出现内存泄漏，开发者也不一定非得显示创建 Pool 了。[runloop相关](https://link.jianshu.com/?t=https%3A%2F%2Fblog.ibireme.com%2F2015%2F05%2F18%2Frunloop%2F)

尽管如此，在大量产生autorelease的对象时，只要NSAutoreleasePool没有被废弃，那么产生的对象就不能被释放，因此会产生内存不足的现象。例如：

#### 读入大量图像的同时改变其尺寸。图像文件读入到NSData对象，并从中生成UIImage对象，改变该对象尺寸后生成新的UIImage对象。这种情况会产生大量autorelease对象。

```other
for (int i = 0; i < 图片数; ++ i) {
/*
读入图片
大量产生autorelease的对象
由于没有废弃NSAutoreleasePool对象
导致产生内存峰值，可能内存不足而奔溃
*/
}
```

在这种情况下，应当在适当的位置生成、持有或废弃NSAutoreleasePool对象

```other
for (int i = 0; i < 图片数; ++ i) {
NSAutoreleasePool *pool = [[NSAutoreleasePool alloc] init];
/*
读入图片
大量产生autorelease的对象
*/
[pool drain];
/*
通过释放pool
autorelease的对象被release，就不会产生内存峰值
*/
}
```

使用容器的block版本的枚举器时，内部会自动添加一个AutoreleasePool：

```other
[array enumerateObjectsUsingBlock:^(id obj, NSUInteger idx, BOOL *stop) {
// 这里被一个局部@autoreleasepool包围着
}];
```

#### 所有权修饰符及其原理

在 ARC 特性下有 4 种与内存管理息息相关的变量所有权修饰符值得我们关注：

- __strong
- __weak
- __unsafe_unretained
- __autoreleasing

说到变量所有权修饰符，有人可能会跟属性修饰符搞混，这里做一个对照关系小结：

- assign 对应的所有权类型是 __unsafe_unretained。
- copy 对应的所有权类型是 __strong。
- retain 对应的所有权类型是 __strong。
- strong 对应的所有权类型是 __strong。
- unsafe_unretained对应的所有权类型是__unsafe_unretained。
- weak 对应的所有权类型是 __weak。

#### __strong

__strong 表示强引用，对应定义 property 时用到的 strong。当对象没有任何一个强引用指向它时，它才会被释放。如果在声明引用时不加修饰符，那么引用将默认是强引用。当需要释放强引用指向的对象时，需要保证所有指向对象强引用置为 nil。__strong 修饰符是 id 类型和对象类型默认的所有权修饰符。

#### 原理：

{

id __strong obj = [[NSObject alloc] init];

}

//编译器的模拟代码

id obj = objc_msgSend(NSObject,@selector(alloc));

objc_msgSend(obj,@selector(init));

// 出作用域的时候调用

objc_release(obj);

虽然ARC有效时不能使用release方法，但由此可知编译器自动插入了release。

对象是通过除alloc、new、copy、multyCopy外方法产生的情况：

```other
{
id __strong obj = [NSMutableArray array];
}
```

结果与之前稍有不同：

```other
//编译器的模拟代码
id obj = objc_msgSend(NSMutableArray,@selector(array));
objc_retainAutoreleasedReturnValue(obj);
objc_release(obj);
```

objc_retainAutoreleasedReturnValue函数主要用于优化程序的运行。它是用于持有(retain)对象的函数，它持有的对象应为返回注册在autoreleasePool中对象的方法，或是函数的返回值。像该源码这样，在调用array类方法之后，由编译器插入该函数。

而这种objc_retainAutoreleasedReturnValue函数是成对存在的，与之对应的函数是objc_autoreleaseReturnValue。它用于array类方法返回对象的实现上。下面看看NSMutableArray类的array方法通过编译器进行了怎样的转换：

```other
+ (id)array
{
return [[NSMutableArray alloc] init];
}
//编译器模拟代码
+ (id)array
{
id obj = objc_msgSend(NSMutableArray,@selector(alloc));
objc_msgSend(obj,@selector(init));
// 代替我们调用了autorelease方法
return objc_autoreleaseReturnValue(obj);
}
```

我们可以看见调用了objc_autoreleaseReturnValue函数且这个函数会返回注册到自动释放池的对象，但是，这个函数有个特点，它会查看调用方的命令执行列表，如果发现接下来会调用objc_retainAutoreleasedReturnValue则不会将返回的对象注册到autoreleasePool中而仅仅返回一个对象。达到了一种最优效果。如下图：

![2969810-01b23a26f092b7a6.png](https://upload-images.jianshu.io/upload_images/2969810-01b23a26f092b7a6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### __weak

__weak 表示弱引用，对应定义 property 时用到的 weak。弱引用不会影响对象的释放，而当对象被释放时，所有指向它的弱引用都会自定被置为 nil，这样可以防止野指针。使用__weak修饰的变量，即是使用注册到autoreleasePool中的对象。__weak 最常见的一个作用就是用来避免循环引用。需要注意的是，__weak 修饰符只能用于 iOS5 以上的版本，在 iOS4 及更低的版本中使用 __unsafe_unretained 修饰符来代替。

__weak 的几个使用场景：

- 在 Delegate 关系中防止循环引用。
- 在 Block 中防止循环引用。
- 用来修饰指向由 Interface Builder 创建的控件。比如：@property (weak, nonatomic) IBOutlet UIButton *testButton;。

#### 原理：

```other
{
id __weak obj = [[NSObject alloc] init];
}
```

编译器转换后的代码如下:

```other
id obj;
id tmp = objc_msgSend(NSObject,@selector(alloc));
objc_msgSend(tmp,@selector(init));
objc_initweak(&obj,tmp);
objc_release(tmp);
objc_destroyWeak(&object);
```

对于__weak内存管理也借助了类似于引用计数表的散列表，它通过对象的内存地址做为key，而对应的__weak修饰符变量的地址作为value注册到weak表中，在上述代码中objc_initweak就是完成这部分操作，而objc_destroyWeak

则是销毁该对象对应的value。当指向的对象被销毁时，会通过其内存地址，去weak表中查找对应的__weak修饰符变量，将其从weak表中删除。所以，weak在修饰只是让weak表增加了记录没有引起引用计数表的变化.

对象通过objc_release释放对象内存的动作如下:

1. objc_release
2. 因为引用计数为0所以执行dealloc
3. _objc_rootDealloc
4. objc_dispose
5. objc_destructInstance
6. objc_clear_deallocating

而在对象被废弃时最后调用了objc_clear_deallocating，该函数的动作如下:

1. 从weak表中获取已废弃对象内存地址对应的所有记录
2. 将已废弃对象内存地址对应的记录中所有以weak修饰的变量都置为nil
3. 从weak表删除已废弃对象内存地址对应的记录
4. 根据已废弃对象内存地址从引用计数表中找到对应记录删除

据此可以解释为什么对象被销毁时对应的weak指针变量全部都置为nil，同时，也看出来销毁weak步骤较多，如果大量使用weak的话会增加CPU的负荷。

还需要确认一点是：__weak修饰符的变量，即使用注册到autoreleasePool中的对象。

```other
{
id __weak obj1 = obj;
NSLog(@"obj2-%@",obj1);
}
```

编译器转换上述代码如下:

```other
id obj1;
objc_initweak(&obj1,obj);
id tmp = objc_loadWeakRetained(&obj1);
objc_autorelease(tmp);
NSLog(@"%@",tmp);
objc_destroyWeak(&obj1);
```

objc_loadWeakRetained函数获取附有__weak修饰符变量所引用的对象并retain, objc_autorelease函数将对象放入autoreleasePool中，据此当我们访问weak修饰指针指向的对象时，实际上是访问注册到自动释放池的对象。因此，如果大量使用weak的话，在我们去访问weak修饰的对象时，会有大量对象注册到自动释放池,这会影响程序的性能。

解决方案：要访问weak修饰的变量时，先将其赋给一个strong变量，然后进行访问

#### 为什么访问weak修饰的对象就会访问注册到自动释放池的对象呢?

因为weak不会引起对象的引用计数器变化，因此，该对象在运行过程中很有可能会被释放。所以，需要将对象注册到自动释放池中并在autoreleasePool销毁时释放对象占用的内存。

#### __unsafe_unretained

ARC 是在 iOS5 引入的，而 __unsafe_unretained 这个修饰符主要是为了在ARC刚发布时兼容iOS4以及版本更低的系统，因为这些版本没有弱引用机制。这个修饰符在定义property时对应的是unsafe_unretained。__unsafe_unretained 修饰的指针纯粹只是指向对象，没有任何额外的操作，不会去持有对象使得对象的 retainCount +1。而在指向的对象被释放时依然原原本本地指向原来的对象地址，不会被自动置为 nil，所以成为了野指针，非常不安全。

\######__unsafe_unretained 的应用场景：

在 ARC 环境下但是要兼容 iOS4.x 的版本，用__unsafe_unretained 替代 __weak 解决强循环引用的问题。

#### __autoreleasing

将对象赋值给附有__autoreleasing修饰符的变量等同于MRC时调用对象的autorelease方法。

```other
@autoeleasepool {
// 如果看了上面__strong的原理，就知道实际上对象已经注册到自动释放池里面了
id __autoreleasing obj = [[NSObject alloc] init];
}
```

编译器转换上述代码如下:

```other
id pool = objc_autoreleasePoolPush();
id obj = objc_msgSend(NSObject,@selector(alloc));
objc_msgSend(obj,@selector(init));
objc_autorelease(obj);
objc_autoreleasePoolPop(pool);
@autoreleasepool {
id __autoreleasing obj = [NSMutableArray array];
}
```

编译器转换上述代码如下:

```other
id pool = objc_autoreleasePoolPush();
id obj = objc_msgSend(NSMutableArray,@selector(array));
objc_retainAutoreleasedReturnValue(obj);
objc_autorelease(obk);
objc_autoreleasePoolPop(pool);
```

上面两种方式，虽然第二种持有对象的方法从alloc方法变为了objc_retainAutoreleasedReturnValue函数，都是通过objc_autorelease，注册到autoreleasePool中。

### *ARC 模式规则*

ARC 模式下，还有一些需要注意的规则：

- 不能显式使用 retain/release/retainCount/autorelease。
- 不能使用 NSAllocateObject/NSDeallocateObject。
- 需要遵守内存管理的方法命名规则。

> 在 ARC 模式和 MRC 模式下，以 alloc/new/copy/mutableCopy 开头的- 方法在返回对象时都必须返回给调用方所应当持有的对象。在 ARC 模式下，追加一条：以 init 开头的方法必须是实例方法并且必须要返回对象。返回的对象应为 id 类型或声明该方法的类的对象类型，或是该类的超类型或子类型。该返回的对象并不注册到 Autorelease Pool 中，基本上只是对 alloc 方法返回值的对象进行初始化处理并返回该对象。需要注意的是：- (void)initialize; 方法虽然是以 init 开头但是并不包含在上述规则中。

- 不要显式调用 dealloc。
- 使用 @autoreleasepool 块替代 NSAutoreleasePool。
- 不能使用区域（NSZone）。

> 区域是以前为了高效利用内存的使用率而设计的，但是，目前来说ARC下的模式已经能够有效利用内存，区域在ARC下还是非ARC下都已经被单纯的忽略

- 对象型变量不能作为 C 语言结构体（struct/union）的成员。

> OC对象型变量如果成为了C语言结构体的成员，那么，ARC不能掌握该对象的生命周期从而有效管理内存，因此，不能这样使用。

- 显式转换 id 和 void *。

> 非ARC下:
id obj = [[NSObject alloc] init];
void *p = obj;
这样的代码是可行的，id和void*可以方便得自由转化 ，但是，在ARC下是不一样的

> ARC下id和void*有三个转换的关键字 __bridge、__bridge_retained、__bridge_transfer:
id obj = [[NSObject alloc] init];
void *p = (__bridge void*)obj;
注意： __bridge不会引起对象的引用计数变化，因此，安全性不太好。相比较，__bridge_retained不仅仅实现了__bridge的功能而且能让p调用retain方法使p持有对象。另外，
__bridge_transfer也是和release方法类似，使用__bridge_transfer进行转化，既让对象p调用一次retain方法，而且原来指针obj会调用一次release方法也非常安全

#### Toll-Free Bridging

MRC 下的 Toll－Free Bridging 因为不涉及内存管理的转移，相互之间可以直接交换使用，当使用 ARC 时，由于Core Foundation 框架并不支持 ARC，此时编译器不知道该如何处理这个同时有 ObjC 指针和 CFTypeRef 指向的对象，所以除了转换类型，还需指定内存管理所有权的改变，可通过 __bridge、__bridge_retained 和 CFBridgingRetain、__bridge_transfer 和 CFBridgingRelease。

#### __bridge

只是声明类型转变，但是不做内存管理规则的转变。比如：

```other
CFStringRef s1 = (__bridge CFStringRef) [[NSString alloc] initWithFormat:@"Hello, %@!", name];
```

只是做了 NSString 到 CFStringRef 的转化，但管理规则未变，依然要用 Objective-C 类型的 ARC 来管理 s1，你不能用 CFRelease() 去释放 s1。

#### __bridge_retained or CFBridgingRetain

表示将指针类型转变的同时，将内存管理的责任由原来的 Objective-C 交给Core Foundation 来处理，也就是，将 ARC 转变为 MRC。比如，还是上面那个例子

```other
NSString *s1 = [[NSString alloc] initWithFormat:@"Hello, %@!", name];
CFStringRef s2 = (__bridge_retained CFStringRef)s1;
// or CFStringRef s2 = (CFStringRef)CFBridgingRetain(s1);
// do something with s2
//...
CFRelease(s2); // 注意要在使用结束后加这个
```

我们在第二行做了转化，这时内存管理规则由 ARC 变为了 MRC，我们需要手动的来管理 s2 的内存，而对于 s1，我们即使将其置为 nil，也不能释放内存。

#### **__bridge_transfer or CFBridgingRelease**

这个修饰符和函数的功能和上面那个 __bridge_retained 相反，它表示将管理的责任由 Core Foundation 转交给 Objective-C，即将管理方式由 MRC 转变为 ARC。比如：

```other
CFStringRef result = CFURLCreateStringByAddingPercentEscapes(. . .);
NSString *s = (__bridge_transfer NSString *)result;
//or NSString *s = (NSString *)CFBridgingRelease(result);
return s;
这里我们将 result 的管理责任交给了 ARC 来处理，我们就不需要再显式地将 CFRelease() 了。
```

#### **其他**

之前写的和内存管理有些许相关：[为什么声明NString，NSArray等需要使用copy，使用strong有什么问题，深拷贝和浅拷贝，block为什么使用copy。](https://www.jianshu.com/p/1e1a6f9c26f8)

### *iOS之@property*

#### @property介绍

相信做过iOS开发的同学都使用过@property，@property翻译过来是属性。在定义一个类时，常常会有多个@property，有了@property，我们可以用来保存类的一些信息或者状态。比如定义一个Student类：

```other
@interface Student : NSObject
@property (nonatomic, copy) NSString *name;
@property (nonatomic, copy) NSString *sex;
@end
```

Student类中有两个属性，分别是name和sex。

在程序中使用时，可以使用

```other
self.name = @"xxx";
self.sex = @"xxx";
```

那么，为什么可以这样用呢？self.name是self的name变量嘛？还是其他的什么？属性中的copy代表什么？nonatomic呢？下面来看一下这些问题的答案。

\#####@property 本质

@property到底是什么呢？实际上@property = 实例变量 + get方法 + set方法。也就是说属性

```other
@property (nonatomic, copy) NSString *name;
```

代表的有实例变量，get方法和set方法。如果大家用过Java，相信对set方法和get方法应该很熟悉，这里的set、get方法和Java里面的作用是一样的，get方法用来获取变量的值，set方法用来设置变量的值。使用@property生成的实例变量、get方法、set方法的命名有严格的规范，实例变量的名称、get方法名、set方法名稍后再介绍。

这里需要注意的是，包括实例变量、get方法和set方法，不会真的出现在我们的编辑器里面，使用属性生成的实例变量、get方法、set方法是在编译过程中生成的。下面介绍一下set方法、get方法以及自动生成的实例变量。

#### setter方法

set方法也可以称为setter方法，之后看到setter方法直接理解成set方法即可。同理，get方法也被称为getter方法。

还是以上面的属性：

```other
@property (nonatomic, copy) NSString *name;
```

为例，属性name生成的setter方法是

\- (void)setName:(NSString *)name;

该命名方法是固定的，是约定成束的。如果属性名是firstName，那么setter方法是：

```other
- (void)setFirstName:(NSString *)firstName;
```

项目中，很多时候会有重写setter方法的需求，只要重写对应的方法即可。比如说重写name属性的setter方法：

```other
- (void)setName:(NSString *)name
{
    NSLog(@"rewrite setter");
    _name = name;
}
```

关于_name是什么，后续会介绍。

#### getter方法

以属性

@property (nonatomic, copy) NSString *name;

为例，编译器自动生成的getter方法是

```other
- (NSString *)name;
```

getter方法的命名也是固定的。如果属性名是firstName，那么getter方法是：

```other
- (NSString *)firstName;
```

重写getter方法：

```other
- (NSString *)name
{
    NSLog(@"rewrite getter");
    return _name;
}
```

如果我们定义了name属性，并且按照上面所述，重写了getter方法和setter方法，Xcode会提示如下的错误：

Use of undeclared identifier '_name'; did you mean 'name'?

稍后我们再解释为何会有该错误，以及如何解决。先来看一下_name到底是什么。

#### 实例变量

既然@property = 实例变量 + getter + setter，那么属性所生成的实例变量名是什么呢？根据上面的例子，也很容易猜到，项目中也经常使用，实例变量的名称就是_name。实例变量的命名也是有固定格式的，下划线+属性名。如果属性是@property firstName，那么生成的实例变量就是_firstName。这也是为何我们在setter方法和getter方法，以及其他的方法中可以使用_name的原因。

这里再提一下，无论是实例变量，还是setter、getter方法，命名都是有严格规范的。正是因为有了这种规范，编译器才能够自动生成方法，这也要求我们在项目中，对变量的命名，方法的命名遵循一定的规范。

#### 自动合成

定义一个@property，在编译期间，编译器会生成实例变量、getter方法、setter方法，这些方法、变量是通过自动合成（autosynthesize）的方式生成并添加到类中。实际上，一个类经过编译后，会生成**变量列表**ivar_list，方法列表method_list，每添加一个属性，在**变量列表**ivar_list会添加对应的变量，如_name，方法列表method_list中会添加对应的setter方法和getter方法。

#### 动态合成

既然有自动合成，那么相对应的就要有非自动合成，非自动合成又称为动态合成。定义一个属性，默认是自动合成的，默认会生成getter方法和setter方法，这也是为何我们可以直接使用self.属性名的原因。实际上，自动合成对应的代码是：

```other
@synthesize name = _name;
```

这行代码是编译器自动生成的，无需我们来写。相应的，如果我们想要动态合成，需要自己写如下代码：

```other
@dynamic sex;
```

这样代码就告诉编译器，sex属性的变量名、getter方法、setter方法由开发者自己来添加，编译器无需处理。

那么这样写和自动合成有什么区别呢？来看下面的代码：

```other
Student *stu = [[Student alloc] init];
stu.sex = @"male";
```

编译，不会有任何问题。运行，也没问题。但是当代码执行到这一行的时候，程序崩溃了，崩溃信息是：

[Student setSex:]: unrecognized selector sent to instance 0x60000217f1a0

即：Student没有setSex方法，没有属性sex的setter方法。这就是动态合成和自动合成的区别。动态合成，需要开发者自己来写属性的setter方法和getter方法。添加上setter方法：

```other
- (void)setSex:(NSString *)sex
{
    _sex = sex;
}
```

由于使用@dynamic，编译器不会自动生成变量，因此除此之外，还需要手动定义_sex变量，如下：

```other
@interface Student : NSObject
{
    NSString *_sex;
}
@property (nonatomic, copy) NSString *name;
@property (nonatomic, copy) NSString *sex;
@end
```

现在再编译，运行，执行没有错误和崩溃。

#### 重写setter、getter方法的注意事项

上面的例子中，重写了属性name的getter方法和setter方法，如下：

```other
- (void)setName:(NSString *)name
{
    NSLog(@"rewrite setter");
    _name = name;
}
- (NSString *)name
{
    NSLog(@"rewrite getter");
    return _name;
}
```

但是编译器会提示错误，错误信息如下：

> Use of undeclared identifier '_name'; did you mean 'name'?
提示没有_name变量。为什么呢？我们没有声明@dynamic，那默认就是@autosynthesize，为何没有_name变量呢？奇怪的是，倘若我们把getter方法，或者setter方法注释掉，gettter、setter方法只留下一个，不会有错误，为什么呢？
还是编译器做了些处理。对于一个可读写的属性来说，当我们重写了其setter、getter方法时，编译器会认为开发者想手动管理@property，此时会将@property作为@dynamic来处理，因此也就不会自动生成变量。解决方法，显式的将属性和一个变量绑定：

```other
@synthesize name = _name;
```

这样就没问题了。如果一个属性是只读的，重写了其getter方法时，编译器也会认为该属性是@dynamic，关于可读写、只读，下面会介绍。这里提醒一下，当项目中重写了属性的getter方法和setter方法时，注意下是否有编译的问题。

#### 修改实例变量的名称

使用自动合成时，针对

@property (nonatomic, copy) NSString *name;

属性，生成的变量名是_name。倘若，不习惯使用下划线开头的变量名，能否指定属性对应的变量名呢？答案是可以的，使用的是上面介绍过的@synthesize关键字。如下：

```other
@synthesize name = stuName;
```

这样，name属性生成的变量名就是stuName,后续使用时需要写stuName,而不是_name。如getter、setter方法：

```other
- (void)setName:(NSString *)name
{
    NSLog(@"rewrite setter");
    stuName = name;
}
- (NSString *)name
{
    NSLog(@"rewrite getter");
    return stuName;
}
```

注意：虽然可以使用@synthesize关键字修改变量名，但是如无特殊需求，不建议这样做。因为默认情况下编译器已经为我们生成了变量名，大多数的项目、开发者也都会遵循这样的规范，既然苹果已经定义了一个好的规范，为什么不遵守呢？

#### getter方法中为何不能用self.

有经验的开发者应该都知道这一点，在getter方法中是不能使用self.的，比如：

```other
- (NSString *)name
{
    NSLog(@"rewrite getter");
    return self.name;  // 错误的写法，会造成死循环
}
```

原因代码注释中已经写了，这样会造成死循环。这里需要注意的是：self.name实际上就是执行了属性name的getter方法，[getter方法中又调用了self.name](http://xn--getterself-0t2pm7a631cfi3d9jwaio4a5g5f.name)， 会一直递归调用，直到程序崩溃。通常程序中使用：

```other
self.name = @"aaa";
```

这样的方式，setter方法会被调用。

#### @property修饰符

当我们定义一个字符串属性时，通常我们会这样写：

```other
@property (nonatomic, copy) NSString *name;
```

当我们定义一个NSMutableArray类型的属性时，通常我们会这样写：

```other
@property (nonatomic, strong) NSMutableArray *books;
```

而当我们定一个基本数据类型时，会这样写：

```other
@property (nonatomic, assign) int age;
```

定义一个属性时，nonatomic、copy、strong、assign等被称作是关键字，或者是修饰符。

#### 修饰符种类

修饰符有四种：

- **原子性**。原子性有nonatomic、atomic两个值，如果不写nonatomic,那么默认是atomic的。如果属性是atomic的，那么在访问其getter和setter方法之前，会有一些判断，大概是判断是否可以访问等，这里系统使用的是自旋锁。由于使用atomic并不能绝对保证线程安全，且会耗费一些性能，因此通常情况下都使用nonatomic。
- **读写权限**。读写权限有两个取值，readwrite和readonly。声明属性时，如果不指定读写权限，那么默认是readwrite的。如果某个属性不想让其他人来写，那么可以设置成readonly。
- **内存管理**。内存管理的取值有assign、strong、weak、copy、unsafe_unretained。
- **set、get方法名**。如果不想使用自动合成所生成的setter、getter方法，声明属性时甚至可以指定方法名。比如指定getter方法名：

```other
@property (nonatomic, assign, getter=isPass) BOOL pass;
```

属性pass的getter方法就是

```other
- (BOOL)isPass;
```

#### 默认修饰符

声明属性时，如果不显示指定修饰符，那么默认的修饰符是哪些呢？或者说未指定的修饰符，默认取值是什么呢？如果是基本数据类型，默认取值是：

> atomic,readwrite,assign
如果是Objective-C对象，默认取值是：

> atomic,readwrite,strong

#### atomic是否是线程安全的

上面提到了，声明属性时，通常使用nonatomic修饰符，原因就是因为atomic并不能保证绝对的线程安全。举例来说，假设有一个线程A在不断的读取属性name的值，同时有一个线程B修改了属性name的值，那么即使属性name是atomic，线程A读到的仍旧是修改后的值，可见不是线程安全的。如果想要实现线程安全，需要手动的实现锁。下面是一段示例代码：

声明name属性，使用atomic修饰符

```other
@property (atomic, copy) NSString *name;
```

对属性name赋值。同时，一个线程在不断的读取name的值，另一个线程在不断的设置name的值：

```other
stu.name = @"aaa";
dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
    for(int i = 0 ; i < 1000; ++i){
        NSLog(@"stu.name = %@",stu.name);
    }
});
dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
    stu.name = @"bbb";
});
```

看一下输出：

```other
2018-12-06 15:42:26.837215+0800 TestClock[15405:175815] stu.name = aaa
2018-12-06 15:42:26.837837+0800 TestClock[15405:175815] stu.name = bbb
```

证实了即使使用了atomic,也不能保证线程安全。

#### weak和assign区别

经常会有面试题问weak和assign的区别，这里介绍一下。

weak和strong是对应的，一个是强引用，一个是弱引用。weak和assign的区别主要是体现在两者修饰OC对象时的差异。上面也介绍过，assign通常用来修饰基本数据类型，如int、float、BOOL等，weak用来修饰OC对象，如UIButton、UIView等。

#### 基本数据类型用weak来修饰(X)

假设声明一个int类型的属性，但是用weak来修饰，会发生什么呢？

```other
@property (nonatomic, weak) int age;
```

Xcode会直接提示错误，错误信息如下：

> Property with 'weak' attribute must be of object type
也就是说，weak只能用来修饰对象，不能用来修饰基本数据类型，否则会发生编译错误。

#### 对象使用assign来修饰

假设声明一个UIButton类型的属性，但是用assign来修饰，会发生什么呢？

```other
@property (nonatomic, assign) UIButton *assignBtn;
```

编译，没有问题，运行也没有问题。我们再声明一个UIButton,使用weak来修饰，对比一下：

```other
@interface ViewController ()
@property (nonatomic, assign) UIButton *assignBtn;
@property (nonatomic, weak) UIButton *weakButton;
@end
```

正常初始化两个button：

```other
UIButton *btn = [[UIButton alloc] initWithFrame:CGRectMake(100,100,100,100)];
[btn setTitle:@"Test" forState:UIControlStateNormal];
btn.backgroundColor = [UIColor lightGrayColor];
self.assignBtn = btn;
self.weakButton = btn;
```

此时打印两个button，没有区别。释放button：

```other
btn = nil;
```

释放之后打印self.weakBtn和self.assignBtn

```other
NSLog(@"self.weakBtn = %@",self.weakButton);
NSLog(@"self.assignBtn = %@",self.assignBtn);
```

运行，执行到self.assignBtn的时候崩溃了，崩溃信息是

\> EXC_BAD_ACCESS (code=EXC_I386_GPFLT)

weak和assign修饰对象时的差别体现出来了。

weak修饰的对象，当对象释放之后，即引用计数为0时，对象会置为nil

> 2018-12-06 16:17:05.774298+0800 TestClock[15863:192570] self.weakBtn = (null)
而向nil发送消息是没有问题的，不会崩溃。
assign修饰的对象，当对象释放之后，即引用计数为0时，对象会变为野指针，不知道指向哪，再向该对象发消息，非常容易崩溃。
因此，当属性类型是对象时，不要使用assign，会带来一些风险。

### ::堆和栈::

上面说到，属性用assign修饰，当被释放后，容易变为野指针，容易带来崩溃问题，那么，为何基本数据类型可以用assign来修饰呢？这就涉及到堆和栈的问题。

相对来说，堆的空间大，通常是不连续的结构，使用链表结构。使用堆中的空间，需要开发者自己去释放。OC中的对象，如 UIButton 、UILabel ，[[UIButton alloc] init] 出来的，都是分配在堆空间上。

栈的空间小，约1M左右，是一段连续的结构。栈中的空间，开发者不需要管，系统会帮忙处理。iOS开发 中 int、float等变量分配内存时是在栈上。如果栈空间使用完，会发生栈溢出的错误。

由于堆、栈结构的差异，栈和堆分配空间时的寻址方式也是不一样的。因为栈是连续的控件，所以栈在分配空间时，会直接在未使用的空间中分配一段出来，供程序使用；如果剩下的空间不够大，直接栈溢出；堆是不连续的，堆寻找合适空间时，是顺着链表结点来寻找，找到第一块足够大的空间时，分配空间，返回。根据两者的数据结构，可以推断，堆空间上是存在碎片的。

回到问题，**为何assign修饰基本数据类型没有野指针的问题？**因为这些基本数据类型是分配在栈上，栈上空间的分配和回收都是系统来处理的，因此开发者无需关注，也就不会产生野指针的问题。

#### **::栈是线程安全的嘛::**

扩展一下，栈是线程安全的嘛？回答问题之前，先看一下进程和线程的关系。

#### **::进程和线程的关系::**

线程是进程的一个实体，是CPU调度和分派的基本单位。一个进程可以拥有多个线程。线程本身是不配拥有系统资源的，只拥有很少的，运行中必不可少的资源（如程序计数器、寄存器、栈）。但是线程可以与同属于一个进程的其他线程，共享进程所拥有的资源。一个进程中所有的线程共享该进程的地址空间，但是每个线程有自己独立的栈，iOS系统中，每个线程栈的大小是1M。而堆则不同。堆是进程所独有的，通常一个进程有一个堆，这个堆为本进程中的所有线程所共享。

#### **::栈的线程安全::**

其实通过上面的介绍，该问题答案已经很明显了：栈是线程安全的。

堆是多个线程所共有的空间，操作系统在对进程进行初始化的时候，会对堆进行分配；

栈是每个线程所独有的，保存线程的运行状态和局部变量。栈在线程开始的时化，每个线程的栈是互相独立的，因此栈是线程安全的。

#### **::copy、strong、mutableCopy::**

属性修饰符中，还有一个经常被问到的面试题是copy和strong。什么时候用copy，为什么？什么时候用strong,为什么？以及mutableCopy又是什么？这一节介绍一下这些内容。

#### copy和strong

首先看一下copy和strong，copy和strong的区别也是面试中出现频率最高的。之前举得例子中其实已经出现了copy和strong：

```other
@property (nonatomic, copy) NSString *sex;
@property (nonatomic, strong) NSMutableArray *books;
```

通常情况下，**不可变对象属性修饰符使用copy，可变对象属性修饰符使用strong。**

#### 可变对象和不可变对象

Objective-C中存在可变对象和不可变对象的概念。像NSArray、NSDictionary、NSString这些都是不可变对象，像NSMutableArray、NSMutableDictionary、NSMutableString这些是可变对象。可变对象和不可变对象的区别是，不可变对象的值一旦确定就不能再修改。下面看个例子来说明。

```other
- (void)testNotChange
{
    NSString *str = @"123";
    NSLog(@"str = %p",str);
    str = @"234";
    NSLog(@"after str = %p",str);
}
```

NSString是不可变对象。虽然在程序中修改了str的值，但是此处的修改实际上是系统重新分配了空间，定义了字符串，然后str重新指向了一个新的地址。这也是为何修改之后地址不一致的原因：

```other
2018-12-06 22:02:41.350812+0800 TestClock[884:17969] str = 0x106ec1290
2018-12-06 22:02:41.350919+0800 TestClock[884:17969] after str = 0x106ec12d0
```

再来看可变对象的例子：

```other
- (void)testChangeAble
{
    NSMutableString *mutStr = [NSMutableString stringWithString:@"abc"];
    NSLog(@"mutStr = %p",mutStr);
    [mutStr appendString:@"def"];
    NSLog(@"after mutStr = %p",mutStr);
}
```

NSMutableString是可变对象。程序中改变了mutStr的值，且修改前后mutStr的地址一致：

```other
2018-12-06 22:10:08.457179+0800 TestClock[1000:21900] mutStr = 0x600002100540
2018-12-06 22:10:08.457261+0800 TestClock[1000:21900] after mutStr = 0x600002100540
```

#### 不可变对象用strong

上面说了，可变对象使用strong，不可变对象使用copy。那么，如果不可变对象使用strong来修饰，会有什么问题呢？写代码测试一下：

```other
@property (nonatomic, strong) NSString *strongStr;
```

首先明确一点，既然类型是NSString，那么则代表我们不希望testStr被改变，否则直接使用可变对象NSMutableString就可以了。另外需要提醒的一点是，NSMutableString是NSString的子类，对继承了解的应该都知道，子类是可以用来初始化父类的。

介绍完之后，来看一段代码。

```other
- (void)testStrongStr
{
    NSString *tempStr = @"123";
    NSMutableString *mutString = [NSMutableString stringWithString:tempStr];
    self.strongStr = mutString;  // 子类初始化父类
    NSLog(@"self str = %p  mutStr = %p",self.strongStr,mutString);   // 两者指向的地址是一样的
    [mutString insertString:@"456" atIndex:0];
    NSLog(@"self str = %@  mutStr = %@",self.strongStr,mutString);  // 两者的值都会改变,不可变对象的值被改变
}
```

注意：**我们定义的不可变对象strongStr，在开发者无感知的情况下被篡改了。**所谓无感知，是因为开发者没有显示的修改strongStr的值，而是再修改其他变量的值时，strongStr被意外的改变。这显然不是我们想得到的，而且也是危险的。项目中出现类似的bug时，通常都很难定位。这就是不可变对象使用strong修饰所带来的风险。

#### 可变对象用copy

上面说了不可变对象使用strong的问题，那么可变对象使用copy有什么问题呢？还是写代码来验证一下：

```other
@property (nonatomic, copy) NSMutableString *mutString;
```

这里还是强调一下，既然属性类型是可变类型，说明我们期望再程序中能够改变mutString的值，否则直接使用NSString了。

看一下测试代码：

```other
- (void)testStrCopy
{
    NSString *str = @"123";
    self.mutString = [NSMutableString stringWithString:str];
    NSLog(@"str = %p self.mutString = %p",str,self.mutString); // 两者的地址不一样
    [self.mutString appendString:@"456"]; // 会崩溃，因为此时self.mutArray是NSString类型，是不可变对象
}
```

执行程序后，会崩溃，崩溃原因是：

即 self.mutString没有appendString方法。self.mutString是NSMutableString类型，为何没有appendString方法呢？这就是使用copy造成的。看一下

```other
self.mutString = [NSMutableString stringWithString:str];
```

这行代码到底发生了什么。这行代码实际上完成了两件事：

```other
// 首先声明一个临时变量
NSMutableString *tempString = [NSMutableString stringWithString:str];
// 将该临时变量copy，赋值给self.mutString
self.mutString = [tempString copy];
```

注意，通过[tempString copy]得到的self.mutString是一个不可变对象（调用 `setter`方法，由于是 `copy` 修饰，会创建一个新对象，而新对象是不可变对象），不可变对象自然没有appendString方法，这也是为何会崩溃的原因。

#### copy和mutableCopy

另外常用来做对比的是copy和mutableCopy。copy和mutableCopy之间的差异主要和深拷贝和浅拷贝有关，先看一下深拷贝、浅拷贝的概念。

#### **::深拷贝、浅拷贝::**

所谓浅拷贝，在Objective-C中可以理解为引用计数加1，并没有申请新的内存区域，只是另外一个指针指向了该区域。深拷贝正好相反，深拷贝会申请新的内存区域，原内存区域的引用计数不变。看图来说明深拷贝和浅拷贝的区别。

![2969810-b0505b656b23a869.png](https://upload-images.jianshu.io/upload_images/2969810-b0505b656b23a869.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

首先A指向一块内存区域，现在设置B = A

![2969810-8cb6d5b99b65c74a.png](https://upload-images.jianshu.io/upload_images/2969810-8cb6d5b99b65c74a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

现在B和A指向了同一块内存区域，即为浅拷贝。

再来看深考贝

![2969810-57d80995cde9a6d9.png](https://upload-images.jianshu.io/upload_images/2969810-57d80995cde9a6d9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

首先A指向一块内存区域，现在设置B = A

![2969810-575ad555a21ab900.png](https://upload-images.jianshu.io/upload_images/2969810-575ad555a21ab900.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

A和B指向的不是同一块内存区域，只是这两块内存区域中的内容是一样的，即为深拷贝。

#### 可变对象的copy、mutableCopy

可变对象的copy和mutableCopy都是深拷贝。以可变对象NSMutableString和NSMutableArray为例，测试代码：

```other
- (void)testMutableCopy
{
    NSMutableString *str1 = [NSMutableString stringWithString:@"abc"];
    NSString *str2 = [str1 copy];
    NSMutableString *str3 = [str1 mutableCopy];
    NSLog(@"str1 = %p str2 = %p str3 = %p",str1,str2,str3);
    NSMutableArray *array1 = [NSMutableArray arrayWithObjects:@"a",@"b", nil];
    NSArray *array2 = [array1 copy];
    NSMutableArray *array3 = [array1 mutableCopy];
    NSLog(@"array1 = %p array2 = %p array3 = %p",array1,array2,array3);
}
```

输出结果：

```other
2018-12-07 13:01:27.525064+0800 TestClock[9357:143436] str1 = 0x60000086d8f0 str2 = 0xc8c1a5736a50d5fe str3 = 0x60000086d9b0
2018-12-07 13:01:27.525198+0800 TestClock[9357:143436] array1 = 0x600000868000 array2 = 0x60000067e5a0 array3 = 0x600000868030
```

可以看到，只要是可变对象，无论是集合对象，还是非集合对象，copy和mutableCopy都是深拷贝。

#### 不可变对象的copy、mutableCopy

不可变对象的copy是浅拷贝，mutableCopy是深拷贝。以NSString和NSArray为例，测试代码如下：

```other
- (void)testCopy
{
    NSString *str1 = @"123";
    NSString *str2 = [str1 copy];
    NSMutableString *str3 = [str1 mutableCopy];
    NSLog(@"str1 = %p str2 = %p str3 = %p",str1,str2,str3);
    NSArray *array1 = @[@"1",@"2"];
    NSArray *array2 = [array1 copy];
    NSMutableArray *array3 = [array1 mutableCopy];
    NSLog(@"array1 = %p array2 = %p array3 = %p",array1,array2,array3);
}
```

输出结果：

```other
2018-12-07 13:06:29.439108+0800 TestClock[9442:147133] str1 = 0x1045612b0 str2 = 0x1045612b0 str3 = 0x6000017e4450
2018-12-07 13:06:29.439236+0800 TestClock[9442:147133] array1 = 0x6000019f5c80 array2 = 0x6000019f5c80 array3 = 0x6000017e1170
```

可以看到，只要是不可变对象，无论是集合对象，还是非集合对象，copy都是浅拷贝，mutableCopy都是深拷贝。

#### **::自定义对象如何支持copy方法::**

项目开发中经常会有自定义对象的需求，那么自定义对象是否可以copy呢？如何支持copy？

自定义对象可以支持copy方法，我们所需要做的是：自定义对象遵守NSCopying协议，且实现copyWithZone方法。NSCopying协议是系统提供的，直接使用即可。

遵守NSCopying协议：

```other
@interface Student : NSObject <NSCopying>
{
    NSString *_sex;
}
@property (atomic, copy) NSString *name;
@property (nonatomic, copy) NSString *sex;
@property (nonatomic, assign) int age;
@end
```

实现CopyWithZone方法：

```other
- (instancetype)initWithName:(NSString *)name age:(int)age sex:(NSString *)sex
{
    if(self = [super init]){
        self.name = name;
        _sex = sex;
        self.age = age;
    }
    return self;
}
- (instancetype)copyWithZone:(NSZone *)zone
{
    // 注意，copy的是自己，因此使用自己的属性
    Student *stu = [[Student allocWithZone:zone] initWithName:self.name age:self.age sex:_sex];
    return stu;
}
```

测试代码：

```other
- (void)testStudent
{
    Student *stu1 = [[Student alloc] initWithName:@"Wang" age:18 sex:@"male"];
    Student *stu2 = [stu1 copy];
    NSLog(@"stu1 = %p stu2 = %p",stu1,stu2);
}
```

输出结果：

```other
stu1 = 0x600003a41e60 stu2 = 0x600003a41fc0
```

这里是一个深拷贝，根据copyWithZone方法的实现，应该很容易明白为何是深拷贝。

除了NSCopying协议和copyWithZone方法，对应的还有NSMutableCopying协议和mutableCopyWithZone方法，实现都是类似的，不做过多介绍。

参考文献和链接：

- [iOS内存管理（MRC、ARC）深入浅出](http://www.cocoachina.com/ios/20180309/22518.html)
- [《Objective-C高级编程》](https://link.jianshu.com/?t=https%3A%2F%2Fbaike.baidu.com%2Fitem%2FObjective-C%25E9%25AB%2598%25E7%25BA%25A7%25E7%25BC%2596%25E7%25A8%258B%2F12345038%3Ffr%3Daladdin)
- [终于明白那些年知其然而不知其所以然的iOS内存管理方式](https://www.jianshu.com/p/4f49c5c81021)
- [深入理解RunLoop](https://link.jianshu.com/?t=https%3A%2F%2Fblog.ibireme.com%2F2015%2F05%2F18%2Frunloop%2F)
- [iOS之@property](https://juejin.im/post/5c105c7ce51d4562d138086f)

