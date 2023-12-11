# iOS多线程详解

![2969810-f242dbdb90cf7e25.png](https://upload-images.jianshu.io/upload_images/2969810-f242dbdb90cf7e25.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# \- iOS多线程详解：『概念篇』

- ## CPU
- ## 进程
- ## 线程
- ## 同异步
- ## 队列
- ## 多线程
- # iOS多线程详解：『pthread、NSThread』
- # iOS多线程详解：『GCD』
- ### GCD 简介
- ### GCD 任务和队列
- ### GCD 的使用步骤
- ### GCD 的基本使用（6种不同组合区别）
- ### GCD 线程间的通信
- ### GCD 的其他方法（栅栏方法：dispatch_barrier_async、延时执行方法：dispatch_after、一次性代码（只执行一次）：dispatch_once、快速迭代方法：dispatch_apply、队列组：dispatch_group、信号量：dispatch_semaphore）
- # iOS多线程详解：『NSOperation、NSOperationQueue』

# 一、iOS多线程详解：『概念篇』

## CPU

## CPU是什么

> ### 引自维基百科CPU中央处理器 （英语：Central Processing Unit，缩写：CPU），是计算机的主要设备之一，功能主要是解释计算机指令以及处理计算机软件中的数据。

> ### 计算机的可编程性主要是指对中央处理器的编程。

> ### 中央处理器、内部存储器和输入／输出设备是现代电脑的三大核心部件。

> ### 1970年代以前，中央处理器由多个独立单元构成，后来发展出由集成电路制造的中央处理器，这些高度收缩的组件就是所谓的微处理器，其中分出的中央处理器最为复杂的电路可以做成单一微小功能强大的单元。 **CPU**主要由运算器、控制器、寄存器三部分组成，从字面意思看就是运算就是起着运算的作用，控制器就是负责发出 **CPU**每条指令所需要的信息，寄存器就是保存运算或者指令的一些临时文件，这样可以保证更高的速度。 **CPU**有着处理指令、执行操作、控制时间、处理数据四大作用，打个比喻来说， **CPU**就像我们的大脑，帮我们完成各种各样的生理活动。因此如果没有 **CPU**，那么电脑就是一堆废物，无法工作。

## 多核CPU与多个CPU多核

> ### 来，简单举个例子。假设现在我们要设计一台计算机的处理器部分的架构。现在摆在我们面前的有两种选择，多个单核CPU和单个多核CPU。如果我们选择多个单核CPU，那么每一个CPU都需要有较为独立的电路支持，有自己的Cache，而他们之间通过板上的总线进行通信。假如在这样的架构上，我们要跑一个多线程的程序（常见典型情况），不考虑超线程，那么每一个线程就要跑在一个独立的CPU上，线程间的所有协作都要走总线，而共享的数据更是有可能要在好几个Cache里同时存在。这样的话，总线开销相比较而言是很大的，怎么办？那么多Cache，即使我们不心疼存储能力的浪费，一致性怎么保证？如果真正做出来，还要在主板上占多块地盘，给布局布线带来更大的挑战，怎么搞定？如果我们选择多核单CPU，那么我们只需要一套芯片组，一套存储，多核之间通过芯片内部总线进行通信，共享使用内存。

> ### 在这样的架构上，如果我们跑一个多线程的程序，那么线程间通信将比上一种情形更快。如果最终实现出来，对板上空间的占用较小，布局布线的压力也较小。看起来，多核单CPU完胜嘛。可是，如果需要同时跑多个大程序怎么办？假设俩大程序，每一个程序都好多线程还几乎用满cache，它们分时使用CPU，那在程序间切换的时候，光指令和数据的替换就要费多大事情啊！所以呢，大部分一般咱们使用的电脑，都是单CPU多核的，比如我们配的Dell T3600，有一颗Intel Xeon E5-1650，6核，虚拟为12个逻辑核心。少部分高端人士需要更强的多任务并发能力，就会搞一个多颗多核CPU的机子，Mac Pro就可以有两颗。 ::**一个核心同时只能处理一个线程，单核CPU只能实现并发，而不是并行**。如果有2个线程，双核 CPU，那这两个线程是并行的，如果有三个线程，那么就还是并发的。下面会讲到并发与并行的区别。::

## 进程

## 进程是什么

> ### 引自维基百科进程（英语：process），是计算机中已运行程序的实体。进程为曾经是分时系统的基本运作单位。在面向进程设计的系统（如早期的UNIX，Linux 2.4及更早的版本）中，进程是程序的基本执行实体；在面向线程设计的系统（如当代多数操作系统、Linux 2.6及更新的版本）中，进程本身不是基本运行单位，而是线程的容器。
程序本身只是指令、数据及其组织形式的描述，进程才是程序（那些指令和数据）的真正运行实例。
若干进程有可能与同一个程序相关系，且每个进程皆可以同步（循序）或异步（平行）的方式独立运行。现代计算机系统可在同一段时间内以进程的形式将多个程序加载到内存中，并借由时间共享（或称时分复用），以在一个处理器上表现出同时（平行性）运行的感觉。
同样的，使用多线程技术（多线程即每一个线程都代表一个进程内的一个独立执行上下文）的操作系统或计算机架构，同样程序的平行线程，可在多CPU主机或网络上真正同时运行（在不同的CPU上）。 **::在 iOS系统中，一个 APP的运行实体代表一个进程。一个进程有独立的内存空间、系统资源、端口等。在进程中可以生成多个线程、这些线程可以共享进程中的资源。
打个比方， CPU好比是一个工厂，进程是一个车间，线程是车间里面的工人。车间的空间是工人们共享的，比如许多房间是每个工人都可以进出的。这象征一个进程的内存空间是共享的，每个线程都可以使用这些共享内存。::**

## 进程间通信

### 搜集了一下资料， iOS大概有8种进程间的通信方式，可能不全，后续补充。

### iOS系统是相对封闭的系统， App各自在各自的沙盒（sandbox）中运行，每个 App都只能读取 iPhone上 iOS系统为该应用程序程序创建的文件夹 AppData下的内容，不能随意跨越自己的沙盒去访问别的 App沙盒中的内容。 所以 iOS的系统中进行 App间通信的方式也比较固定，常见的 App间通信方式以及使用场景总结如下。

## 1、Port (local socket)

- ### 上层封装为 NSMachPort : Foundation层
- ### 中层封装为 CFMachPort ： CoreFoundation层
- ### 下层封装为 MachPorts : Mach内核层（线程、进程都可使用它进行通信）

### 一个 App1在本地的端口 port1234进行 TCP的 bind和 listen，另外一个 App2在同一个端口 port1234发起 TCP的 connect连接，这样就可以建立正常的 TCP连接，进行 TCP通信了，那么就想传什么数据就可以传什么数据了。但是有一个限制，就是要求两个 App进程都在活跃状态，而没有被后台杀死。尴尬的一点是 iOS系统会给每个 TCP在后台 600秒的网络通信时间， 600秒后 APP会进入休眠状态。

## 2、URL Scheme

### 这个是 iOS App通信最常用到的通信方式， App1通过 openURL的方法跳转到 App2，并且在 URL中带上想要的参数，有点类似 http的 get请求那样进行参数传递。这种方式是使用最多的最常见的，使用方法也很简单只需要源 App1在 info.plist中配置 LSApplicationQueriesSchemes，指定目标 App2的 scheme；然后在目标 App2的 info.plist中配置好 URL types，表示该 App接受何种 URLScheme的唤起。

![2969810-8f6853760db624b6.png](https://upload-images.jianshu.io/upload_images/2969810-8f6853760db624b6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 典型的使用场景就是各开放平台 SDK的分享功能，如分享到微信朋友圈微博等，或者是支付场景。比如从滴滴打车结束行程跳转到微信进行支付。

## 3、Keychain

### iOS系统的 Keychain是一个安全的存储容器，它本质上就是一个 sqllite数据库，它的位置存储在 /private/var/Keychains/keychain-2.db，不过它所保存的所有数据都是经过加密的，可以用来为不同的 App保存敏感信息，比如用户名，密码等。 iOS系统自己也用 Keychain来保存 VPN凭证和 Wi-Fi密码。它是独立于每个 App的沙盒之外的，所以即使 App被删除之后， Keychain里面的信息依然存在。

### 基于安全和独立于 App沙盒的两个特性， Keychain主要用于给 App保存登录和身份凭证等敏感信息，这样只要用户登录过，即使用户删除了 App重新安装也不需要重新登录。

### 那 Keychain用于 App间通信的一个典型场景也和 App的登录相关，就是统一账户登录平台。使用同一个账号平台的多个 App，只要其中一个 App用户进行了登录，其他app就可以实现自动登录不需要用户多次输入账号和密码。一般开放平台都会提供登录 SDK，在这个 SDK内部就可以把登录相关的信息都写到 Keychain中，这样如果多个 App都集成了这个 SDK，那么就可以实现统一账户登录了。

### Keychain的使用比较简单，使用 iOS系统提供的类 KeychainItemWrapper，并通过 Keychainaccess groups就可以在应用之间共享 Keychain中的数据的数据了。

## 4、UIPasteboard

### 顾名思义， UIPasteboard是剪切板功能，因为 iOS的原生控件 UITextView， UITextField 、 UIWebView，我们在使用时如果长按，就会出现复制、剪切、选中、全选、粘贴等功能，这个就是利用了系统剪切板功能来实现的。而每一个App都可以去访问系统剪切板，所以就能够通过系统剪贴板进行 App间的数据传输了。

### UIPasteboard典型的使用场景就是淘宝跟微信/QQ的链接分享。由于腾讯和阿里的公司战略，腾讯在微信和QQ中都屏蔽了淘宝的链接。那如果淘宝用户想通过QQ或者微信跟好友分享某个淘宝商品，怎么办呢？ 阿里的工程师就巧妙的利用剪贴板实现了这个功能。首先淘宝 App中将链接自定义成淘口令，引导用户进行复制，并去QQ好友对话中粘贴。然后QQ好友收到消息后再打开自己的淘宝 App，淘宝 App每次从后台切到前台时，就会检查系统剪切板中是否有淘口令，如果有淘口令就进行解析并跳转到对于的商品页面。微信好友把淘口令复制到淘宝中，就可以打开好友分享的淘宝链接了。

## 5、UIDocumentInteractionController

### UIDocumentInteractionController主要是用来实现同设备上 App之间的共享文档，以及文档预览、打印、发邮件和复制等功能。它的使用非常简单.

### 首先通过调用它唯一的类方法 interactionControllerWithURL:，并传入一个 URL(NSURL)，为你想要共享的文件来初始化一个实例对象。然后 UIDocumentInteractionControllerDelegate，然后显示菜单和预览窗口。

## 6、AirDrop

### 引自维基百科AirDrop是苹果公司的MacOS和iOS操作系统中的一个随建即连网络，自Mac OS X Lion（Mac OS X 10.7）和iOS 7起引入，允许在支持的麦金塔计算机和iOS设备上传输文件，无需透过邮件或大容量存储设备。

### 在OS X Yosemite（OS X 10.10）之前，OS X 中的隔空投送协议不同于iOS的隔空投送协议，因此不能互相传输[2]。但是，OS X Yosemite或更新版本支持iOS的隔空投送协议（使用Wi-Fi和蓝牙），这适用于一台Mac与一台iOS设备以及两台2012年或更新版本的Mac计算机之间的传输。[3][4]使用旧隔空投送协议（只使用Wi-Fi）的旧模式在两台2012年或更早的Mac计算机之间传输也是可行的。[4]

### 隔空投送所容纳的文件大小没有限制。苹果用户报告称隔空投送能传输小于10GB的视频文件。

### iOS并没有直接提供 AirDrop的实现接口，但是使用 UIActivityViewController的方法唤起 AirDrop，进行数据交互。

## 7、UIActivityViewController

### UIActivityViewController类是一个标准的 ViewController，提供了几项标准的服务，比如复制项目至剪贴板，把内容分享至社交网站，以及通过 Messages发送数据等等。在 iOS7SDK中， UIActivityViewController类提供了内置的 AirDrop功能。

### 如果你有一些数据一批对象需要通过 AirDrop进行分享，你所需要的是通过对象数组初始化 UIActivityViewController，并展示在屏幕上。

## 8、App Groups

### AppGroup用于同一个开发团队开发的 App之间，包括 App和 Extension之间共享同一份读写空间，进行数据共享。同一个团队开发的多个应用之间如果能直接数据共享，大大提高用户体验。

### 实现细节参考App之间的数据共享—— AppGroup的配置

## 线程

## 线程是什么

> ### 引自维基百科线程（英语：thread）是操作系统能够进行运算调度的最小单位。它被包含在进程之中，是进程中的实际运作单位。
一条线程指的是进程中一个单一顺序的控制流，一个进程中可以并发多个线程，每条线程并行执行不同的任务。在Unix System V及SunOS中也被称为轻量进程（lightweight processes），但轻量进程更多指内核线程（kernel thread），而把用户线程（user thread）称为线程。
讲线程就不能不提任务，任务是什么的，通俗的说任务就是就一件事情或一段代码，线程其实就是去执行这件事情。

> ### 线程（thread），指的是一个独立的代码执行路径，也就是说线程是代码执行路径的最小分支。在 iOS 中，线程的底层实现是基于 POSIX threads API 的，也就是我们常说的 pthreads ；

## 进程和线程的比较

### 1.线程是CPU调用(执行任务)的最小单位。

### 2.进程是CPU分配资源的最小单位。

### 3.一个进程中至少要有一个线程。

### 4.同一个进程内的线程共享进程的资源。

## 超线程技术

> ### 引自维基百科超线程（HT, Hyper-Threading）超线程技术就是利用特殊的硬件指令，把一个物理内核模拟成两个逻辑内核，让单个处理器都能使用线程级并行计算，进而兼容多线程操作系统和软件，减少了CPU的闲置时间，提高了CPU的运行速度。 采用超线程即是可在同一时间里，应用程序可以使用芯片的不同部分。

> ### 引自知乎超线程这个东西并不是开了就一定比不开的好。因为每个CPU核心里ALU，FPU这些运算单元的数量是有限的，而超线程的目的之一就是在一个线程用运算单元少的情况下，让另外一个线程跑起来，不让运算单元闲着。但是如果当一个线程整数，浮点运算各种多，当前核心运算单元没多少空闲了，这时候你再塞进了一个线程，这下子资源就紧张了。两线程就会互相抢资源，拖慢对方速度。至于，超线程可以解决一个线程cache miss，另外一个可以顶上，但是如果两个线程都miss了，那就只有都在等了。这个还是没有GPU里一个SM里很多warp，超多线程同时跑来得有效果。所以，如果你的程序是单线程，关了超线程，免得别人抢你资源，如果是多线程，每个线程运算不大，超线程比较有用。

## 线程池

> ### 线程池（英语：thread pool）：一种线程使用模式。 线程过多会带来调度开销，进而影响缓存局部性和整体性能。 而线程池维护着多个线程，等待着监督管理者分配可并发执行的任务。 这避免了在处理短时间任务时创建与销毁线程的代价。
线程池的执行流程如：
首先，启动若干数量的线程，并让这些线程处于睡眠状态
其次，当客户端有新的请求时，线程池会唤醒某一个睡眠线程，让它来处理客户端的请求
最后，当请求处理完毕，线程又处于睡眠状态 **所以在并发的时候，同时能有多少线程在跑是有线程池的线程缓存数量决定的。**

## 同异步和队列

## 同异步是什么

> ### 同步和异步操作的主要区别在于是否等待操作执行完成，即是否阻塞当前线程。同步操作会等待操作执行完成后再继续执行接下来的代码，而异步操作则恰好相反，它会在调用后立即返回，不会等待操作的执行结果。
| | 同步 | 异步 |
|:----:|:----------|:--------|
| 是否阻塞当前线程 | 是 | 否 |
| 是否等待任务执行完成 | 是 | 否 |

## 队列是什么

## 队列含义

![2969810-cf2f45d24af1ee2c.png](https://upload-images.jianshu.io/upload_images/2969810-cf2f45d24af1ee2c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> ### 引自维基百科队列，又称为伫列（queue），是先进先出（FIFO, First-In-First-Out）的线性表。在具体应用中通常用链表或者数组来实现。队列只允许在后端（称为rear）进行插入操作，在前端（称为front）进行删除操作。 队列的操作方式和堆栈类似，唯一的区别在于队列只允许新数据在后端进行添加。

## \- 串行队列和并发队列

### 串行队列一次只能执行一个任务，而并发队列则可以允许多个任务同时执行。 iOS系统就是使用这些队列来进行任务调度的，它会根据调度任务的需要和系统当前的负载情况动态地创建和销毁线程，而不需要我们手动地管理。

### **注意这个并发多个任务同时执行的 同时在二字指的是同一时间内(由于 CPU执行很快，感觉几乎同时)**

### 这里会有一个经常性的疑问，串行队列一次执行一个任务，任务按顺序执行，先进先出，这个好理解。**那并发几个任务同时执行也是先进先出，这个怎么理解呢。因为并发执行任务，先进去的任务并不一定先执行完，但是即使后面的任务先执行完，也是要等前面的任务退出。这是由队列的性质决定的。**

![2969810-468988dae64eb045.png](https://upload-images.jianshu.io/upload_images/2969810-468988dae64eb045.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![2969810-fe524b111cbd3040.png](https://upload-images.jianshu.io/upload_images/2969810-fe524b111cbd3040.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![2969810-70cdea705419a99c.png](https://upload-images.jianshu.io/upload_images/2969810-70cdea705419a99c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 注意这里说的是队列执行时间，并不是先进先出的示例。

## \- 并发队列和并行队列

> ### **并发的来历** 在过去单CPU时代，单任务在一个时间点只能执行单一程序。之后发展到多任务阶段，计算机能在同一时间点并行执行多任务或多进程。虽然并不是真正意义上的“同一时间点”，而是多个任务或进程共享一个CPU，并交由操作系统来完成多任务间对CPU的运行切换，以使得每个任务都有机会获得一定的时间片运行。

> ### **并行的来历** 多线程比多任务更加有挑战。多线程是在同一个程序内部并行执行，因此会对相同的内存空间进行并发读写操作。这可能是在单线程程序中从来不会遇到的问题。其中的一些错误也未必会在单CPU机器上出现，因为两个线程从来不会得到真正的并行执行。然而，更现代的计算机伴随着多核CPU的出现，也就意味着不同的线程能被不同的CPU核得到真正意义的并行执行。
并发的关键是你有处理多个任务的能力，不一定要同时。 并行的关键是你有同时处理多个任务的能力。**并发是一种能力，处理多个任务的能力。并行是状态，多个任务同时执行的状态。**可以看到并发和并行并不是同一类概念，所以不具有比较性，并发包含并行，就比如水果是包含西瓜一样。并发的不一定是并行，并行的一定是并发。

![2969810-c59dd1ec3c1c5066.png](https://upload-images.jianshu.io/upload_images/2969810-c59dd1ec3c1c5066.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![2969810-99d1f61364f8a8b8.png](https://upload-images.jianshu.io/upload_images/2969810-99d1f61364f8a8b8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> ### 一个CPU的 核心同时只能处理一个线程。 单核CPU一个线程，当前是并行(两个以上才叫并行，为了理解，暂且这样叫吧)。单核CPU两个线程，当前是并发。双核CPU两个线程，当前是并行。双核CPU四个线程，当前是并发。

## 多线程

### **1个进程中可以开启多条线程，每条线程可以并行（同时）执行不同的任务

### 多线程技术可以提高程序的执行效率**

## 多线程原理

### 同一时间，CPU只能处理1条线程，只有1条线程在工作（执行），多线程并发（同时）执行，其实是CPU快速地在多条线程之间调度（切换），如果CPU调度线程的时间足够快，就造成了多线程并发执行的假象。

### 那么如果线程非常非常多，会发生什么情况？

### CPU会在N多线程之间调度，CPU会累死，消耗大量的CPU资源，同时每条线程被调度执行的频次也会会降低（线程的执行效率降低）。

### 因此我们一般只开3-5条线程。

## 多线程优缺点

## \- 多线程的优点

- ### 能适当提高程序的执行效率
- ### 能适当提高资源利用率（CPU、内存利用率）

## \- 多线程的缺点

- ### 创建线程是有开销的，iOS下主要成本包括：内核数据结构（大约1KB）、栈空间（子线程512KB、主线程1MB，也可以使用-setStackSize:设置，但必须是4K的倍数，而且最小是16K），创建线程大约需要90毫秒的创建时间
- ### 如果开启大量的线程，会降低程序的性能，线程越多，CPU在调度线程上的开销就越大。
- ### 程序设计更加复杂：比如线程之间的通信、多线程的数据共享等问题。

## 多线程的应用

## \- 主线程的主要作用

- ### 显示\刷新UI界面
- ### 处理UI事件（比如点击事件、滚动事件、拖拽事件等）

## \- 主线程的使用注意

- ### 别将比较耗时的操作放到主线程中
- ### 耗时操作会卡住主线程，严重影响UI的流畅度，给用户一种“卡”的坏体验
- ### 将耗时操作放在子线程中执行，提高程序的执行效率

## 多线程实现方案

![2969810-85ad9baf139d62c3.png](https://upload-images.jianshu.io/upload_images/2969810-85ad9baf139d62c3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# iOS多线程总结：『pthread、NSThread』

> ### 本文用来介绍 iOS 多线程中，**pthread、NSThread**的使用方法及实现。
第一部分：pthread 的使用、其他相关方法。
第二部分：NSThread 的使用、线程相关用法、线程状态控制方法、线程之间的通信、线程安全和线程同步，以及线程的状态转换相关知识。
文中 Demo 我已放在了 Github 上，Demo 链接：[传送门](https://link.juejin.im/?target=https%3A%2F%2Fgithub.com%2Fbujige%2FYSC-pthread-NSThread-demo)

## 1. pthread

## 1.1 pthread 简介

### pthread 是一套通用的多线程的 API，可以在Unix / Linux / Windows 等系统跨平台使用，使用 C 语言编写，需要程序员自己管理线程的生命周期，使用难度较大，我们在 iOS 开发中几乎不使用 pthread，但是还是来可以了解一下的。

> ### 引自 [百度百科](https://link.juejin.im/?target=http%3A%2F%2Fbaike.baidu.com%2Fitem%2FPthread)POSIX 线程（POSIX threads），简称 Pthreads，是线程的 POSIX 标准。该标准定义了创建和操纵线程的一整套 API。在类Unix操作系统（Unix、Linux、Mac OS X等）中，都使用 Pthreads 作为操作系统的线程。Windows 操作系统也有其移植版 pthreads-win32。

> ### 引自 [维基百科](https://link.juejin.im/?target=https%3A%2F%2Fzh.wikipedia.org%2Fwiki%2FPOSIX%25E7%25BA%25BF%25E7%25A8%258B)POSIX 线程（英语：POSIX Threads，常被缩写 为 Pthreads）是 POSIX 的线程标准，定义了创建和操纵线程的一套 API。 实现 POSIX 线程标准的库常被称作 Pthreads，一般用于 Unix-like POSIX 系统，如 Linux、Solaris。但是 Microsoft Windows 上的实现也存在，例如直接使用 Windows API 实现的第三方库 pthreads-w32；而利用 Windows 的 SFU/SUA 子系统，则可以使用微软提供的一部分原生 POSIX API。

## 1.2 pthread 使用方法

1. ### 首先要包含头文件`#import <pthread.h>`
2. ### 其次要创建线程，并开启线程执行任务

```other
// 1. 创建线程: 定义一个pthread_t类型变量
pthread_t thread;
// 2. 开启线程: 执行任务
pthread_create(&thread, NULL, run, NULL);
// 3. 设置子线程的状态设置为 detached，该线程运行结束后会自动释放所有资源
pthread_detach(thread);
void * run(void *param) // 新线程调用方法，里边为需要执行的任务
{
NSLog(@"%@", [NSThread currentThread]);
return NULL;
}
```

## \- `pthread_create(&thread, NULL, run, NULL)`; 中各项参数含义：

- ### 第一个参数`&thread`是线程对象，指向线程标识符的指针
- ### 第二个是线程属性，可赋值NULL
- ### 第三个run表示指向函数的指针(run对应函数里是需要在新线程中执行的任务)
- ### 第四个是运行函数的参数，可赋值NULL

## 1.3 pthread 其他相关方法

- ### `pthread_create()` 创建一个线程
- ### `pthread_exit()` 终止当前线程
- ### `pthread_cancel()`中断另外一个线程的运行
- ### `pthread_join()` 阻塞当前的线程，直到另外一个线程运行结束
- ### `pthread_attr_init()` 初始化线程的属性
- ### `pthread_attr_setdetachstate()` 设置脱离状态的属性（决定这个线程在终止时是否可以被结合）
- ### `pthread_attr_getdetachstate()` 获取脱离状态的属性
- ### `pthread_attr_destroy()` 删除线程的属性
- ### `pthread_kill()` 向线程发送一个信号

## 2. NSThread

### NSThread 是苹果官方提供的，使用起来比 pthread 更加面向对象，简单易用，可以直接操作线程对象。不过也需要需要程序员自己管理线程的生命周期(主要是创建)，我们在开发的过程中偶尔使用 NSThread。比如我们会经常调用[NSThread currentThread]来显示当前的进程信息。

### 下边我们说说 NSThread 如何使用。

## 2.1 创建、启动线程

- ### 先创建线程，再启动线程

```other
// 1. 创建线程
NSThread *thread = [[NSThread alloc] initWithTarget:self selector:@selector(run) object:nil];
// 2. 启动线程
[thread start]; // 线程一启动，就会在线程thread中执行self的run方法
// 新线程调用方法，里边为需要执行的任务
- (void)run {
NSLog(@"%@", [NSThread currentThread]);
}
```

- ### 创建线程后自动启动线程

```other
// 1. 创建线程后自动启动线程
[NSThread detachNewThreadSelector:@selector(run) toTarget:self withObject:nil];
// 新线程调用方法，里边为需要执行的任务
- (void)run {
NSLog(@"%@", [NSThread currentThread]);
}
```

- ### 隐式创建并启动线程

```other
// 1. 隐式创建并启动线程
[self performSelectorInBackground:@selector(run) withObject:nil];
// 新线程调用方法，里边为需要执行的任务
- (void)run {
NSLog(@"%@", [NSThread currentThread]);
}
```

## 2.2 线程相关用法

```other
// 获得主线程
+ (NSThread *)mainThread;
// 判断是否为主线程(对象方法)
- (BOOL)isMainThread;
// 判断是否为主线程(类方法)
+ (BOOL)isMainThread;
// 获得当前线程
NSThread *current = [NSThread currentThread];
// 线程的名字——setter方法
- (void)setName:(NSString *)n;
// 线程的名字——getter方法
- (NSString *)name;
```

## 2.3 线程状态控制方法

- ### 启动线程方法

```other
- (void)start;
// 线程进入就绪状态 -> 运行状态。当线程任务执行完毕，自动进入死亡状态
```

- ### 阻塞（暂停）线程方法

```other
+ (void)sleepUntilDate:(NSDate *)date;
+ (void)sleepForTimeInterval:(NSTimeInterval)ti;
// 线程进入阻塞状态
```

- ### 强制停止线程

```other
+ (void)exit;
// 线程进入死亡状态
```

## 2.4 线程之间的通信

### 在开发中，我们经常会在子线程进行耗时操作，操作结束后再回到主线程去刷新 UI。这就涉及到了子线程和主线程之间的通信。我们先来了解一下官方关于 NSThread 的线程间通信的方法。

```other
// 在主线程上执行操作
- (void)performSelectorOnMainThread:(SEL)aSelector withObject:(id)arg waitUntilDone:(BOOL)wait;
- (void)performSelectorOnMainThread:(SEL)aSelector withObject:(id)arg waitUntilDone:(BOOL)wait modes:(NSArray<NSString *> *)array;
// equivalent to the first method with kCFRunLoopCommonModes
// 在指定线程上执行操作
- (void)performSelector:(SEL)aSelector onThread:(NSThread *)thr withObject:(id)arg waitUntilDone:(BOOL)wait modes:(NSArray *)array NS_AVAILABLE(10_5, 2_0);
- (void)performSelector:(SEL)aSelector onThread:(NSThread *)thr withObject:(id)arg waitUntilDone:(BOOL)wait NS_AVAILABLE(10_5, 2_0);
// 在当前线程上执行操作，调用 NSObject 的 performSelector:相关方法
- (id)performSelector:(SEL)aSelector;
- (id)performSelector:(SEL)aSelector withObject:(id)object;
- (id)performSelector:(SEL)aSelector withObject:(id)object1 withObject:(id)object2;
```

### 下面通过一个经典的下载图片 DEMO 来展示线程之间的通信。具体步骤如下：

### 开启一个子线程，在子线程中下载图片。

### 回到主线程刷新 UI，将图片展示在 UIImageView 中。

### DEMO 代码如下：

```other
/**
* 创建一个线程下载图片
*/
- (void)downloadImageOnSubThread {
// 在创建的子线程中调用downloadImage下载图片
[NSThread detachNewThreadSelector:@selector(downloadImage) toTarget:self withObject:nil];
}
/**
* 下载图片，下载完之后回到主线程进行 UI 刷新
*/
- (void)downloadImage {
NSLog(@"current thread -- %@", [NSThread currentThread]);
// 1. 获取图片 imageUrl
NSURL *imageUrl = [NSURL URLWithString:@"https://ysc-demo-1254961422.file.myqcloud.com/YSC-phread-NSThread-demo-icon.jpg"];
// 2. 从 imageUrl 中读取数据(下载图片) -- 耗时操作
NSData *imageData = [NSData dataWithContentsOfURL:imageUrl];
// 通过二进制 data 创建 image
UIImage *image = [UIImage imageWithData:imageData];
// 3. 回到主线程进行图片赋值和界面刷新
[self performSelectorOnMainThread:@selector(refreshOnMainThread:) withObject:image waitUntilDone:YES];
}
/**
* 回到主线程进行图片赋值和界面刷新
*/
- (void)refreshOnMainThread:(UIImage *)image {
NSLog(@"current thread -- %@", [NSThread currentThread]);
// 赋值图片到imageview
self.imageView.image = image;
}
```

## 2.5 NSThread 线程安全和线程同步

### **线程安全**：

### 如果你的代码所在的进程中有多个线程在同时运行，而这些线程可能会同时运行这段代码。如果每次运行结果和单线程运行的结果是一样的，而且其他的变量的值也和预期的是一样的，就是线程安全的。

### 若每个线程中对全局变量、静态变量只有读操作，而无写操作，一般来说，这个全局变量是线程安全的；若有多个线程同时执行写操作（更改变量），一般都需要考虑线程同步，否则的话就可能影响线程安全。

### **线程同步**：

### 可理解为线程 A 和 线程 B 一块配合，A 执行到一定程度时要依靠线程 B 的某个结果，于是停下来，示意 B 运行；B 依言执行，再将结果给 A；A 再继续操作。

### 举个简单例子就是：两个人在一起聊天。两个人不能同时说话，避免听不清(操作冲突)。等一个人说完(一个线程结束操作)，另一个再说(另一个线程再开始操作)。

### 下面，我们模拟火车票售卖的方式，实现 NSThread 线程安全和解决线程同步问题。

### 场景：总共有50张火车票，有两个售卖火车票的窗口，一个是北京火车票售卖窗口，另一个是上海火车票售卖窗口。两个窗口同时售卖火车票，卖完为止。

## 2.5.1 NSThread 非线程安全

### 先来看看不考虑线程安全的代码：

```other
/**
* 初始化火车票数量、卖票窗口(非线程安全)、并开始卖票
*/
- (void)initTicketStatusNotSave {
// 1. 设置剩余火车票为 50
self.ticketSurplusCount = 50;
// 2. 设置北京火车票售卖窗口的线程
self.ticketSaleWindow1 = [[NSThread alloc]initWithTarget:self selector:@selector(saleTicketNotSafe) object:nil];
self.ticketSaleWindow1.name = @"北京火车票售票窗口";
// 3. 设置上海火车票售卖窗口的线程
self.ticketSaleWindow2 = [[NSThread alloc]initWithTarget:self selector:@selector(saleTicketNotSafe) object:nil];
self.ticketSaleWindow2.name = @"上海火车票售票窗口";
// 4. 开始售卖火车票
[self.ticketSaleWindow1 start];
[self.ticketSaleWindow2 start];
}
/**
* 售卖火车票(非线程安全)
*/
- (void)saleTicketNotSafe {
while (1) {
//如果还有票，继续售卖
if (self.ticketSurplusCount > 0) {
self.ticketSurplusCount --;
NSLog(@"%@", [NSString stringWithFormat:@"剩余票数：%ld 窗口：%@", self.ticketSurplusCount, [NSThread currentThread].name]);
[NSThread sleepForTimeInterval:0.2];
}
//如果已卖完，关闭售票窗口
else {
NSLog(@"所有火车票均已售完");
break;
}
}
}
```

### 运行后部分结果为：

![2969810-750b020f9151959a.png](https://upload-images.jianshu.io/upload_images/2969810-750b020f9151959a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 可以看到在不考虑线程安全的情况下，得到票数是错乱的，这样显然不符合我们的需求，所以我们需要考虑线程安全问题。

## 2.5.2 NSThread 线程安全

### **线程安全解决方案**：

### 可以给线程加锁，在一个线程执行该操作的时候，不允许其他线程进行操作。iOS 实现线程加锁有很多种方式。@synchronized、 NSLock、NSRecursiveLock、NSCondition、NSConditionLock、pthread_mutex、dispatch_semaphore、OSSpinLock、atomic(property) set/ge等等各种方式。为了简单起见，这里不对各种锁的解决方案和性能做分析，只用最简单的@synchronized来保证线程安全，从而解决线程同步问题。

### 考虑线程安全的代码：

```other
/**
* 初始化火车票数量、卖票窗口(线程安全)、并开始卖票
*/
- (void)initTicketStatusSave {
// 1. 设置剩余火车票为 50
self.ticketSurplusCount = 50;
// 2. 设置北京火车票售卖窗口的线程
self.ticketSaleWindow1 = [[NSThread alloc]initWithTarget:self selector:@selector(saleTicketSafe) object:nil];
self.ticketSaleWindow1.name = @"北京火车票售票窗口";
// 3. 设置上海火车票售卖窗口的线程
self.ticketSaleWindow2 = [[NSThread alloc]initWithTarget:self selector:@selector(saleTicketSafe) object:nil];
self.ticketSaleWindow2.name = @"上海火车票售票窗口";
// 4. 开始售卖火车票
[self.ticketSaleWindow1 start];
[self.ticketSaleWindow2 start];
}
/**
* 售卖火车票(线程安全)
*/
- (void)saleTicketSafe {
while (1) {
// 互斥锁
@synchronized (self) {
//如果还有票，继续售卖
if (self.ticketSurplusCount > 0) {
self.ticketSurplusCount --;
NSLog(@"%@", [NSString stringWithFormat:@"剩余票数：%ld 窗口：%@", self.ticketSurplusCount, [NSThread currentThread].name]);
[NSThread sleepForTimeInterval:0.2];
}
//如果已卖完，关闭售票窗口
else {
NSLog(@"所有火车票均已售完");
break;
}
}
}
}
```

### 运行后结果为：

![2969810-4e0fa43c929be287.png](https://upload-images.jianshu.io/upload_images/2969810-4e0fa43c929be287.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![2969810-a3574e29ea3a92e6.png](https://upload-images.jianshu.io/upload_images/2969810-a3574e29ea3a92e6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 可以看出，在考虑了线程安全的情况下，加锁之后，得到的票数是正确的，没有出现混乱的情况。我们也就解决了多个线程同步的问题。

## 2.6 线程的状态转换

### 当我们新建一条线程`NSThread *thread = [[NSThread alloc] initWithTarget:self selector:@selector(run) object:nil];`，在内存中的表现为：

![2969810-9d79be4f898bc376.png](https://upload-images.jianshu.io/upload_images/2969810-9d79be4f898bc376.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 当调用`[thread start];`后，系统把线程对象放入可调度线程池中，线程对象进入就绪状态，如下图所示。

![2969810-7f5a0e3cd1163781.png](https://upload-images.jianshu.io/upload_images/2969810-7f5a0e3cd1163781.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 当然，可调度线程池中，会有其他的线程对象，如下图所示。在这里我们只关心左边的线程对象。

![2969810-068f65dd333994a5.png](https://upload-images.jianshu.io/upload_images/2969810-068f65dd333994a5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### **下边我们来看看当前线程的状态转换。**

- ### 如果CPU现在调度当前线程对象，则当前线程对象进入运行状态，如果CPU调度其他线程对象，则当前线程对象回到就绪状态。
- ### 如果CPU在运行当前线程对象的时候调用了sleep方法\等待同步锁，则当前线程对象就进入了阻塞状态，等到sleep到时\得到同步锁，则回到就绪状态。
- ### 如果CPU在运行当前线程对象的时候线程任务执行完毕\异常强制退出，则当前线程对象进入死亡状态。

### 只看文字可能不太好理解，具体当前线程对象的状态变化如下图所示。

![2969810-7bcff53de39984f4.png](https://upload-images.jianshu.io/upload_images/2969810-7bcff53de39984f4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- # iOS多线程详解：『GCD』
- ### GCD 简介
- ### GCD 任务和队列
- ### GCD 的使用步骤
- ### GCD 的基本使用（6种不同组合区别）
- ### GCD 线程间的通信
- ### GCD 的其他方法（栅栏方法：dispatch_barrier_async、延时执行方法：dispatch_after、一次性代码（只执行一次）：dispatch_once、快速迭代方法：dispatch_apply、队列组：dispatch_group、信号量：dispatch_semaphore）

# 1. GCD 简介

### 什么是 GCD 呢？我们先来看看百度百科的解释简单了解下概念。

> ### 引自[百度百科](https://link.juejin.im/?target=http%3A%2F%2Fbaike.baidu.com%2Fitem%2FGCD)

> ### **Grand Central Dispatch(GCD)**是 Apple 开发的一个多核编程的较新的解决方法。它主要用于优化应用程序以支持多核处理器以及其他对称多处理系统。它是一个在线程池模式的基础上执行的并发任务。在 Mac OS X 10.6 雪豹中首次推出，也可在 iOS 4 及以上版本使用。

## 为什么要用 GCD 呢？

### 因为 GCD 有很多好处啊，具体如下：

- ### GCD 可用于多核的并行运算
- ### GCD 会自动利用更多的 CPU 内核（比如双核、四核）
- ### GCD 会自动管理线程的生命周期（创建线程、调度任务、销毁线程）
- ### 程序员只需要告诉 GCD 想要执行什么任务，不需要编写任何线程管理代码

### 既然 GCD 有这么多的好处，那么下面我们就来系统的学习一下 GCD 的使用方法。

# 2. GCD 任务和队列

### 学习 GCD 之前，先来了解 GCD 中两个核心概念：**任务和队列**。

### **任务：**就是执行操作的意思，换句话说就是你在线程中执行的那段代码。在 GCD 中是放在 block 中的。执行任务有两种方式：**同步执行（sync）和异步执行（async）。**两者的主要区别是：**是否等待队列的任务执行结束，以及是否具备开启新线程的能力**。

- ### **同步执行（sync）**：
   - ### 同步添加任务到指定的队列中，在添加的任务执行结束之前，会一直等待，直到队列里面的任务完成之后再继续执行。
   - ### 只能在当前线程中执行任务，不具备开启新线程的能力。
- ### **异步执行（async）**：
   - ### 异步添加任务到指定的队列中，它不会做任何等待，可以继续执行任务。
   - ### 可以在新的线程中执行任务，具备开启新线程的能力。

### 举个简单例子：你要打电话给小明和小白。

### 同步执行就是，你打电话给小明的时候，不能同时打给小白，等到给小明打完了，才能打给小白（等待任务执行结束）。而且只能用当前的电话（不具备开启新线程的能力）。

### 而异步执行就是，你打电话给小明的时候，不等和小明通话结束，还能直接给小白打电话，不用等着和小明通话结束再打（不用等待任务执行结束）。除了当前电话，你还可以使用其他所能使用的电话（具备开启新线程的能力）。

![2969810-ce5000daaec0a8d9.png](https://upload-images.jianshu.io/upload_images/2969810-ce5000daaec0a8d9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> ### 注意： 异步执行（async） 虽然具有开启新线程的能力，但是并不一定开启新线程。这跟任务所指定的队列类型有关（下面会讲）。 **队列（Dispatch Queue）**：这里的队列指执行任务的等待队列，即用来存放任务的队列。队列是一种特殊的线性表，采用 FIFO（先进先出）的原则，即新任务总是被插入到队列的末尾，而读取任务的时候总是从队列的头部开始读取。每读取一个任务，则从队列中释放一个任务。队列的结构可参考下图：

在 GCD 中有两种队列：**串行队列和并发队列**。两者都符合 FIFO（先进先出）的原则。两者的主要区别是：**执行顺序不同，以及开启线程数不同**。

- ### **串行队列（Serial Dispatch Queue）**：
   - ### 每次只有一个任务被执行。让任务一个接着一个地执行。（只开启一个线程，一个任务执行完毕后，再执行下一个任务）
- ### **并发队列（Concurrent Dispatch Queue）**：
   - ### 可以让多个任务并发（同时）执行。（可以开启多个线程，并且同时执行任务）

![2969810-36ec6e54050765a4.png](https://upload-images.jianshu.io/upload_images/2969810-36ec6e54050765a4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![2969810-2dcf2e2c12211a91.png](https://upload-images.jianshu.io/upload_images/2969810-2dcf2e2c12211a91.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> ### 注意：并发队列 的并发功能只有在异步（dispatch_async）函数下才有效
两者具体区别如下两图所示。

# 3. GCD 的使用步骤

### GCD 的使用步骤其实很简单，只有两步。

1. ### 创建一个队列（串行队列或并发队列）
2. ### 将任务追加到任务的等待队列中，然后系统就会根据任务类型执行任务（同步执行或异步执行）

### 下边来看看 队列的**创建方法/获取方法**，以及 **任务的创建方法**。

## 3.1 队列的创建方法/获取方法

- ### 可以使用`dispatch_queue_create`来创建队列，需要传入两个参数，第一个参数表示队列的唯一标识符，用于 DEBUG，可为空，Dispatch Queue 的名称推荐使用应用程序 ID 这种逆序全程域名；第二个参数用来识别是串行队列还是并发队列。`DISPATCH_QUEUE_SERIAL` 表示串行队列，`DISPATCH_QUEUE_CONCURRENT` 表示并发队列。

```other
// 串行队列的创建方法
dispatch_queue_t queue = dispatch_queue_create("net.bujige.testQueue", DISPATCH_QUEUE_SERIAL);
// 并发队列的创建方法
dispatch_queue_t queue = dispatch_queue_create("net.bujige.testQueue", DISPATCH_QUEUE_CONCURRENT);
```

- ### 对于串行队列，GCD 提供了的一种特殊的串行队列：**主队列（Main Dispatch Queue）**。
   - ### 所有放在主队列中的任务，都会放到主线程中执行。
   - ### 可使用`dispatch_get_main_queue()`获得主队列。

```other
// 主队列的获取方法
dispatch_queue_t queue = dispatch_get_main_queue();
```

- ### 对于并发队列，GCD 默认提供了全局并发队列（Global Dispatch Queue）。
   - ### 可以使用`dispatch_get_global_queue`来获取。需要传入两个参数。第一个参数表示队列优先级，一般用`DISPATCH_QUEUE_PRIORITY_DEFAULT`。第二个参数暂时没用，用0即可。

```other
// 全局并发队列的获取方法
dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
```

## 3.2 任务的创建方法

### GCD 提供了同步执行任务的创建方法`dispatch_sync`和异步执行任务创建方法`dispatch_async`。

```other
// 同步执行任务创建方法
dispatch_sync(queue, ^{
// 这里放同步执行任务代码
});
// 异步执行任务创建方法
dispatch_async(queue, ^{
// 这里放异步执行任务代码
});
```

### 虽然使用 GCD 只需两步，但是既然我们有两种队列（串行队列/并发队列），两种任务执行方式（同步执行/异步执行），那么我们就有了四种不同的组合方式。这四种不同的组合方式是：

1. > ### 同步执行 + 并发队列
2. > ### 异步执行 + 并发队列
3. > ### 同步执行 + 串行队列
4. > ### 异步执行 + 串行队列
实际上，刚才还说了两种特殊队列：**全局并发队列、主队列**。全局并发队列可以作为普通并发队列来使用。但是主队列因为有点特殊，所以我们就又多了两种组合方式。这样就有六种不同的组合方式了。
5. > ### 同步执行 + 主队列
6. > ### 异步执行 + 主队列
那么这几种不同组合方式各有什么区别呢，这里为了方便，先上结果，再来讲解。你可以直接查看表格结果，然后跳过**4. GCD 的基本使用** 。

下边我们来分别讲讲这几种不同的组合方式的使用方法。

![2969810-58d09f1ee192a882.png](https://upload-images.jianshu.io/upload_images/2969810-58d09f1ee192a882.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 4. GCD 的基本使用

### **先来讲讲并发队列的两种执行方式。**

## 4.1 同步执行 + 并发队列

- ### 在当前线程中执行任务，不会开启新线程，执行完一个任务，再执行下一个任务。

```other
/**
* 同步执行 + 并发队列
* 特点：在当前线程中执行任务，不会开启新线程，执行完一个任务，再执行下一个任务。
*/
- (void)syncConcurrent {
NSLog(@"currentThread---%@",[NSThread currentThread]); // 打印当前线程
NSLog(@"syncConcurrent---begin");
dispatch_queue_t queue = dispatch_queue_create("net.bujige.testQueue", DISPATCH_QUEUE_CONCURRENT);
dispatch_sync(queue, ^{
// 追加任务1
for (int i = 0; i < 2; ++i) {
[NSThread sleepForTimeInterval:2]; // 模拟耗时操作
NSLog(@"1---%@",[NSThread currentThread]); // 打印当前线程
}
});
dispatch_sync(queue, ^{
// 追加任务2
for (int i = 0; i < 2; ++i) {
[NSThread sleepForTimeInterval:2]; // 模拟耗时操作
NSLog(@"2---%@",[NSThread currentThread]); // 打印当前线程
}
});
dispatch_sync(queue, ^{
// 追加任务3
for (int i = 0; i < 2; ++i) {
[NSThread sleepForTimeInterval:2]; // 模拟耗时操作
NSLog(@"3---%@",[NSThread currentThread]); // 打印当前线程
}
});
NSLog(@"syncConcurrent---end");
}
```

### 输出结果：

```other
2018-02-23 20:34:55.095932+0800 YSC-GCD-demo[19892:4996930] currentThread---<NSThread: 0x60400006bbc0>{number = 1, name = main}
2018-02-23 20:34:55.096086+0800 YSC-GCD-demo[19892:4996930] syncConcurrent---begin
2018-02-23 20:34:57.097589+0800 YSC-GCD-demo[19892:4996930] 1---<NSThread: 0x60400006bbc0>{number = 1, name = main}
2018-02-23 20:34:59.099100+0800 YSC-GCD-demo[19892:4996930] 1---<NSThread: 0x60400006bbc0>{number = 1, name = main}
2018-02-23 20:35:01.099843+0800 YSC-GCD-demo[19892:4996930] 2---<NSThread: 0x60400006bbc0>{number = 1, name = main}
2018-02-23 20:35:03.101171+0800 YSC-GCD-demo[19892:4996930] 2---<NSThread: 0x60400006bbc0>{number = 1, name = main}
2018-02-23 20:35:05.101750+0800 YSC-GCD-demo[19892:4996930] 3---<NSThread: 0x60400006bbc0>{number = 1, name = main}
2018-02-23 20:35:07.102414+0800 YSC-GCD-demo[19892:4996930] 3---<NSThread: 0x60400006bbc0>{number = 1, name = main}
2018-02-23 20:35:07.102575+0800 YSC-GCD-demo[19892:4996930] syncConcurrent---end
```

### **从`同步执行 + 并发队列`中可看到：**

- ### 所有任务都是在当前线程（主线程）中执行的，没有开启新的线程（`同步执行`不具备开启新线程的能力）。
- ### 所有任务都在打印的`syncConcurrent---begin`和`syncConcurrent---end`之间执行的（`同步任务`需要等待队列的任务执行结束）。
- ### 任务按顺序执行的。按顺序执行的原因：虽然`并发队列`可以开启多个线程，并且同时执行多个任务。但是因为本身不能创建新线程，只有当前线程这一个线程（`同步任务`不具备开启新线程的能力），所以也就不存在并发。而且当前线程只有等待当前队列中正在执行的任务执行完毕之后，才能继续接着执行下面的操作（`同步任务`需要等待队列的任务执行结束）。所以任务只能一个接一个按顺序执行，不能同时被执行。

## 4.2 异步执行 + 并发队列

- ### 可以开启多个线程，任务交替（同时）执行。

```other
/**
* 异步执行 + 并发队列
* 特点：可以开启多个线程，任务交替（同时）执行。
*/
- (void)asyncConcurrent {
NSLog(@"currentThread---%@",[NSThread currentThread]); // 打印当前线程
NSLog(@"asyncConcurrent---begin");
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
dispatch_async(queue, ^{
// 追加任务3
for (int i = 0; i < 2; ++i) {
[NSThread sleepForTimeInterval:2]; // 模拟耗时操作
NSLog(@"3---%@",[NSThread currentThread]); // 打印当前线程
}
});
NSLog(@"asyncConcurrent---end");
}
```

### 输出结果：

```other
2018-02-23 20:36:41.769269+0800 YSC-GCD-demo[19929:5005237] currentThread---<NSThread: 0x604000062d80>{number = 1, name = main}
2018-02-23 20:36:41.769496+0800 YSC-GCD-demo[19929:5005237] asyncConcurrent---begin
2018-02-23 20:36:41.769725+0800 YSC-GCD-demo[19929:5005237] asyncConcurrent---end
2018-02-23 20:36:43.774442+0800 YSC-GCD-demo[19929:5005566] 2---<NSThread: 0x604000266f00>{number = 5, name = (null)}
2018-02-23 20:36:43.774440+0800 YSC-GCD-demo[19929:5005567] 3---<NSThread: 0x60000026f200>{number = 4, name = (null)}
2018-02-23 20:36:43.774440+0800 YSC-GCD-demo[19929:5005565] 1---<NSThread: 0x600000264800>{number = 3, name = (null)}
2018-02-23 20:36:45.779286+0800 YSC-GCD-demo[19929:5005567] 3---<NSThread: 0x60000026f200>{number = 4, name = (null)}
2018-02-23 20:36:45.779302+0800 YSC-GCD-demo[19929:5005565] 1---<NSThread: 0x600000264800>{number = 3, name = (null)}
2018-02-23 20:36:45.779286+0800 YSC-GCD-demo[19929:5005566] 2---<NSThread: 0x604000266f00>{number = 5, name = (null)}
```

### **在`异步执行 + 并发队列`中可以看出：**

- ### 除了当前线程（主线程），系统又开启了3个线程，并且任务是交替/同时执行的。（`异步执行`具备开启新线程的能力。且`并发队列`可开启多个线程，同时执行多个任务）。
- ### 所有任务是在打印的`syncConcurrent---begin`和`syncConcurrent---end`之后才执行的。说明当前线程没有等待，而是直接开启了新线程，在新线程中执行任务（`异步执行`不做等待，可以继续执行任务）。

### **接下来再来讲讲串行队列的两种执行方式。**

## 4.3 同步执行 + 串行队列

- ### 不会开启新线程，在当前线程执行任务。任务是串行的，执行完一个任务，再执行下一个任务。

```other
/**
* 同步执行 + 串行队列
* 特点：不会开启新线程，在当前线程执行任务。任务是串行的，执行完一个任务，再执行下一个任务。
*/
- (void)syncSerial {
NSLog(@"currentThread---%@",[NSThread currentThread]); // 打印当前线程
NSLog(@"syncSerial---begin");
dispatch_queue_t queue = dispatch_queue_create("net.bujige.testQueue", DISPATCH_QUEUE_SERIAL);
dispatch_sync(queue, ^{
// 追加任务1
for (int i = 0; i < 2; ++i) {
[NSThread sleepForTimeInterval:2]; // 模拟耗时操作
NSLog(@"1---%@",[NSThread currentThread]); // 打印当前线程
}
});
dispatch_sync(queue, ^{
// 追加任务2
for (int i = 0; i < 2; ++i) {
[NSThread sleepForTimeInterval:2]; // 模拟耗时操作
NSLog(@"2---%@",[NSThread currentThread]); // 打印当前线程
}
});
dispatch_sync(queue, ^{
// 追加任务3
for (int i = 0; i < 2; ++i) {
[NSThread sleepForTimeInterval:2]; // 模拟耗时操作
NSLog(@"3---%@",[NSThread currentThread]); // 打印当前线程
}
});
NSLog(@"syncSerial---end");
}
```

### 输出结果：

```other
2018-02-23 20:39:37.876811+0800 YSC-GCD-demo[19975:5017162] currentThread---<NSThread: 0x604000079400>{number = 1, name = main}
2018-02-23 20:39:37.876998+0800 YSC-GCD-demo[19975:5017162] syncSerial---begin
2018-02-23 20:39:39.878316+0800 YSC-GCD-demo[19975:5017162] 1---<NSThread: 0x604000079400>{number = 1, name = main}
2018-02-23 20:39:41.879829+0800 YSC-GCD-demo[19975:5017162] 1---<NSThread: 0x604000079400>{number = 1, name = main}
2018-02-23 20:39:43.880660+0800 YSC-GCD-demo[19975:5017162] 2---<NSThread: 0x604000079400>{number = 1, name = main}
2018-02-23 20:39:45.881265+0800 YSC-GCD-demo[19975:5017162] 2---<NSThread: 0x604000079400>{number = 1, name = main}
2018-02-23 20:39:47.882257+0800 YSC-GCD-demo[19975:5017162] 3---<NSThread: 0x604000079400>{number = 1, name = main}
2018-02-23 20:39:49.883008+0800 YSC-GCD-demo[19975:5017162] 3---<NSThread: 0x604000079400>{number = 1, name = main}
2018-02-23 20:39:49.883253+0800 YSC-GCD-demo[19975:5017162] syncSerial---end
```

### **在`同步执行 + 串行队列`可以看到：**

- ### 所有任务都是在当前线程（主线程）中执行的，并没有开启新的线程（`同步执行`不具备开启新线程的能力）。
- ### 所有任务都在打印的`syncConcurrent---begin`和`syncConcurrent---end`之间执行（`同步任务`需要等待队列的任务执行结束）。
- ### 任务是按顺序执行的（`串行队列`每次只有一个任务被执行，任务一个接一个按顺序执行）。

## 4.4 异步执行 + 串行队列

- ### 会开启新线程，但是因为任务是串行的，执行完一个任务，再执行下一个任务。

```other
/**
* 异步执行 + 串行队列
* 特点：会开启新线程，但是因为任务是串行的，执行完一个任务，再执行下一个任务。
*/
- (void)asyncSerial {
NSLog(@"currentThread---%@",[NSThread currentThread]); // 打印当前线程
NSLog(@"asyncSerial---begin");
dispatch_queue_t queue = dispatch_queue_create("net.bujige.testQueue", DISPATCH_QUEUE_SERIAL);
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
dispatch_async(queue, ^{
// 追加任务3
for (int i = 0; i < 2; ++i) {
[NSThread sleepForTimeInterval:2]; // 模拟耗时操作
NSLog(@"3---%@",[NSThread currentThread]); // 打印当前线程
}
});
NSLog(@"asyncSerial---end");
}
```

### 输出结果：

```other
2018-02-23 20:41:17.029999+0800 YSC-GCD-demo[20008:5024757] currentThread---<NSThread: 0x604000070440>{number = 1, name = main}
2018-02-23 20:41:17.030212+0800 YSC-GCD-demo[20008:5024757] asyncSerial---begin
2018-02-23 20:41:17.030364+0800 YSC-GCD-demo[20008:5024757] asyncSerial---end
2018-02-23 20:41:19.035379+0800 YSC-GCD-demo[20008:5024950] 1---<NSThread: 0x60000026e100>{number = 3, name = (null)}
2018-02-23 20:41:21.037140+0800 YSC-GCD-demo[20008:5024950] 1---<NSThread: 0x60000026e100>{number = 3, name = (null)}
2018-02-23 20:41:23.042220+0800 YSC-GCD-demo[20008:5024950] 2---<NSThread: 0x60000026e100>{number = 3, name = (null)}
2018-02-23 20:41:25.042971+0800 YSC-GCD-demo[20008:5024950] 2---<NSThread: 0x60000026e100>{number = 3, name = (null)}
2018-02-23 20:41:27.047690+0800 YSC-GCD-demo[20008:5024950] 3---<NSThread: 0x60000026e100>{number = 3, name = (null)}
2018-02-23 20:41:29.052327+0800 YSC-GCD-demo[20008:5024950] 3---<NSThread: 0x60000026e100>{number = 3, name = (null)}
```

### **在`异步执行 + 串行队列`可以看到：**

- ### 开启了一条新线程（`异步执行`具备开启新线程的能力，`串行队列`只开启一个线程）。
- ### 所有任务是在打印的`syncConcurrent---begin`和`syncConcurrent---end`之后才开始执行的（`异步执行`不会做任何等待，可以继续执行任务）。
- ### 任务是按顺序执行的（`串行队列`每次只有一个任务被执行，任务一个接一个按顺序执行）。

### 下边讲讲刚才我们提到过的特殊队列：**主队列**。

- ### **主队列**：GCD自带的一种特殊的串行队列
- ### 所有放在主队列中的任务，都会放到主线程中执行
- ### 可使用`dispatch_get_main_queue()`获得主队列

### **我们再来看看主队列的两种组合方式**。

## 4.5 同步执行 + 主队列

### `同步执行 + 主队列`在不同线程中调用结果也是不一样，在主线程中调用会出现死锁，而在其他线程中则不会。

## 4.5.1 在主线程中调用同步执行 + 主队列

- ### 互相等待卡住不可行

```other
/**
* 同步执行 + 主队列
* 特点(主线程调用)：互等卡主不执行。
* 特点(其他线程调用)：不会开启新线程，执行完一个任务，再执行下一个任务。
*/
- (void)syncMain {
NSLog(@"currentThread---%@",[NSThread currentThread]); // 打印当前线程
NSLog(@"syncMain---begin");
dispatch_queue_t queue = dispatch_get_main_queue();
dispatch_sync(queue, ^{
// 追加任务1
for (int i = 0; i < 2; ++i) {
[NSThread sleepForTimeInterval:2]; // 模拟耗时操作
NSLog(@"1---%@",[NSThread currentThread]); // 打印当前线程
}
});
dispatch_sync(queue, ^{
// 追加任务2
for (int i = 0; i < 2; ++i) {
[NSThread sleepForTimeInterval:2]; // 模拟耗时操作
NSLog(@"2---%@",[NSThread currentThread]); // 打印当前线程
}
});
dispatch_sync(queue, ^{
// 追加任务3
for (int i = 0; i < 2; ++i) {
[NSThread sleepForTimeInterval:2]; // 模拟耗时操作
NSLog(@"3---%@",[NSThread currentThread]); // 打印当前线程
}
});
NSLog(@"syncMain---end");
}
```

### 输出结果：

```other
2018-02-23 20:42:36.842892+0800 YSC-GCD-demo[20041:5030982] currentThread---<NSThread: 0x600000078a00>{number = 1, name = main}
2018-02-23 20:42:36.843050+0800 YSC-GCD-demo[20041:5030982] syncMain---begin
(lldb)
```

### **在`同步执行 + 主队列`可以惊奇的发现：**

- ### 在主线程中使用`同步执行 + 主队列`，追加到主线程的任务1、任务2、任务3都不再执行了，而且`syncMain---end`也没有打印，在XCode 9上还会报崩溃。这是为什么呢？

### 这是因为我们在主线程中执行syncMain方法，相当于把syncMain任务放到了主线程的队列中。而同步执行会等待当前队列中的任务执行完毕，才会接着执行。那么当我们把任务1追加到主队列中，任务1就在等待主线程处理完syncMain任务。而syncMain任务需要等待任务1执行完毕，才能接着执行。

### 那么，现在的情况就是syncMain任务和任务1都在等对方执行完毕。这样大家互相等待，所以就卡住了，所以我们的任务执行不了，而且syncMain---end也没有打印。

### **要是如果不在主线程中调用，而在其他线程中调用会如何呢？**

## 4.5.2 在其他线程中调用同步执行 + 主队列

- ### 不会开启新线程，执行完一个任务，再执行下一个任务

```other
// 使用 NSThread 的 detachNewThreadSelector 方法会创建线程，并自动启动线程执行
selector 任务
[NSThread detachNewThreadSelector:@selector(syncMain) toTarget:self withObject:nil];
```

### 输出结果：

```other
2018-02-23 20:44:19.377321+0800 YSC-GCD-demo[20083:5040347] currentThread---<NSThread: 0x600000272fc0>{number = 3, name = (null)}
2018-02-23 20:44:19.377494+0800 YSC-GCD-demo[20083:5040347] syncMain---begin
2018-02-23 20:44:21.384716+0800 YSC-GCD-demo[20083:5040132] 1---<NSThread: 0x60000006c900>{number = 1, name = main}
2018-02-23 20:44:23.386091+0800 YSC-GCD-demo[20083:5040132] 1---<NSThread: 0x60000006c900>{number = 1, name = main}
2018-02-23 20:44:25.387687+0800 YSC-GCD-demo[20083:5040132] 2---<NSThread: 0x60000006c900>{number = 1, name = main}
2018-02-23 20:44:27.388648+0800 YSC-GCD-demo[20083:5040132] 2---<NSThread: 0x60000006c900>{number = 1, name = main}
2018-02-23 20:44:29.390459+0800 YSC-GCD-demo[20083:5040132] 3---<NSThread: 0x60000006c900>{number = 1, name = main}
2018-02-23 20:44:31.391965+0800 YSC-GCD-demo[20083:5040132] 3---<NSThread: 0x60000006c900>{number = 1, name = main}
2018-02-23 20:44:31.392513+0800 YSC-GCD-demo[20083:5040347] syncMain---end
```

### **在其他线程中使用`同步执行 + 主队列`可看到：**

- ### 所有任务都是在主线程（非当前线程）中执行的，没有开启新的线程（所有放在`主队列`中的任务，都会放到主线程中执行）。
- ### 所有任务都在打印的`syncConcurrent---begin`和`syncConcurrent---end`之间执行（`同步任务`需要等待队列的任务执行结束）。
- ### 任务是按顺序执行的（主队列是`串行队列`，每次只有一个任务被执行，任务一个接一个按顺序执行）。

### 为什么现在就不会卡住了呢？

### 因为`syncMain` 任务放到了其他线程里，而`任务1`、`任务2`、`任务3`都在追加到主队列中，这三个任务都会在主线程中执行。`syncMain` 任务在其他线程中执行到追加`任务1`到主队列中，因为主队列现在没有正在执行的任务，所以，会直接执行主队列的`任务1`，等`任务1`执行完毕，再接着执行`任务2`、`任务3`。所以这里不会卡住线程。

## 4.6 异步执行 + 主队列

- ### 只在主线程中执行任务，执行完一个任务，再执行下一个任务。

```other
/**
* 异步执行 + 主队列
* 特点：只在主线程中执行任务，执行完一个任务，再执行下一个任务
*/
- (void)asyncMain {
NSLog(@"currentThread---%@",[NSThread currentThread]); // 打印当前线程
NSLog(@"asyncMain---begin");
dispatch_queue_t queue = dispatch_get_main_queue();
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
dispatch_async(queue, ^{
// 追加任务3
for (int i = 0; i < 2; ++i) {
[NSThread sleepForTimeInterval:2]; // 模拟耗时操作
NSLog(@"3---%@",[NSThread currentThread]); // 打印当前线程
}
});
NSLog(@"asyncMain---end");
}
```

### 输出结果：

```other
2018-02-23 20:45:49.981505+0800 YSC-GCD-demo[20111:5046708] currentThread---<NSThread: 0x60000006d440>{number = 1, name = main}
2018-02-23 20:45:49.981935+0800 YSC-GCD-demo[20111:5046708] asyncMain---begin
2018-02-23 20:45:49.982352+0800 YSC-GCD-demo[20111:5046708] asyncMain---end
2018-02-23 20:45:51.991096+0800 YSC-GCD-demo[20111:5046708] 1---<NSThread: 0x60000006d440>{number = 1, name = main}
2018-02-23 20:45:53.991959+0800 YSC-GCD-demo[20111:5046708] 1---<NSThread: 0x60000006d440>{number = 1, name = main}
2018-02-23 20:45:55.992937+0800 YSC-GCD-demo[20111:5046708] 2---<NSThread: 0x60000006d440>{number = 1, name = main}
2018-02-23 20:45:57.993649+0800 YSC-GCD-demo[20111:5046708] 2---<NSThread: 0x60000006d440>{number = 1, name = main}
2018-02-23 20:45:59.994928+0800 YSC-GCD-demo[20111:5046708] 3---<NSThread: 0x60000006d440>{number = 1, name = main}
2018-02-23 20:46:01.995589+0800 YSC-GCD-demo[20111:5046708] 3---<NSThread: 0x60000006d440>{number = 1, name = main}
```

### **在`异步执行 + 主队列`可以看到：**

- ### 所有任务都是在当前线程（主线程）中执行的，并没有开启新的线程（虽然`异步执行`具备开启线程的能力，但因为是主队列，所以所有任务都在主线程中）。
- ### 所有任务是在打印的syncConcurrent---begin和syncConcurrent---end之后才开始执行的（异步执行不会做任何等待，可以继续执行任务）。
- ### 任务是按顺序执行的（因为主队列是`串行队列`，每次只有一个任务被执行，任务一个接一个按顺序执行）。

### 弄懂了难理解、绕来绕去的**队列+任务**之后，我们来学习一个简单的东西：**5. GCD 线程间的通信**。

# 5. GCD 线程间的通信

### 在 iOS 开发过程中，我们一般在主线程里边进行 UI 刷新，例如：点击、滚动、拖拽等事件。我们通常把一些耗时的操作放在其他线程，比如说图片下载、文件上传等耗时操作。而当我们有时候在其他线程完成了耗时操作时，需要回到主线程，那么就用到了线程之间的通讯。

```other
/**
* 线程间通信
*/
- (void)communication {
// 获取全局并发队列
dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
// 获取主队列
dispatch_queue_t mainQueue = dispatch_get_main_queue();
dispatch_async(queue, ^{
// 异步追加任务
for (int i = 0; i < 2; ++i) {
[NSThread sleepForTimeInterval:2]; // 模拟耗时操作
NSLog(@"1---%@",[NSThread currentThread]); // 打印当前线程
}
// 回到主线程
dispatch_async(mainQueue, ^{
// 追加在主线程中执行的任务
[NSThread sleepForTimeInterval:2]; // 模拟耗时操作
NSLog(@"2---%@",[NSThread currentThread]); // 打印当前线程
});
});
}
```

### 输出结果：

```other
2018-02-23 20:47:03.462394+0800 YSC-GCD-demo[20154:5053282] 1---<NSThread: 0x600000271940>{number = 3, name = (null)}
2018-02-23 20:47:05.465912+0800 YSC-GCD-demo[20154:5053282] 1---<NSThread: 0x600000271940>{number = 3, name = (null)}
2018-02-23 20:47:07.466657+0800 YSC-GCD-demo[20154:5052953] 2---<NSThread: 0x60000007bf80>{number = 1, name = main}
```

- ### 可以看到在其他线程中先执行任务，执行完了之后回到主线程执行主线程的相应操作。

# 6. GCD 的其他方法

## 6.1 GCD 栅栏方法：dispatch_barrier_async

### 我们有时需要异步执行两组操作，而且第一组操作执行完之后，才能开始执行第二组操作。这样我们就需要一个相当于 **栅栏** 一样的一个方法将两组异步执行的操作组给分割起来，当然这里的操作组里可以包含一个或多个任务。这就需要用到`dispatch_barrier_async`方法在两个操作组间形成栅栏。

### `dispatch_barrier_async`函数会等待前边追加到并发队列中的任务全部执行完毕之后，再将指定的任务追加到该异步队列中。然后在`dispatch_barrier_async`函数追加的任务执行完毕之后，异步队列才恢复为一般动作，接着追加任务到该异步队列并开始执行。具体如下图所示：

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

### 输出结果：

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

### **在dispatch_barrier_async相关代码执行结果中可以看出：**

- ### 在执行完栅栏前面的操作之后，才执行栅栏操作，最后再执行栅栏后边的操作。

## 6.2 GCD 延时执行方法：dispatch_after

### 我们经常会遇到这样的需求：在指定时间（例如3秒）之后执行某个任务。可以用 GCD 的dispatch_after函数来实现。

### 需要注意的是：dispatch_after函数并不是在指定时间之后才开始执行处理，而是在指定时间之后将任务追加到主队列中。严格来说，这个时间并不是绝对准确的，但想要大致延迟执行任务，dispatch_after函数是很有效的。

```other
/**
* 延时执行方法 dispatch_after
*/
- (void)after {
NSLog(@"currentThread---%@",[NSThread currentThread]); // 打印当前线程
NSLog(@"asyncMain---begin");
dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(2.0 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
// 2.0秒后异步追加任务代码到主队列，并开始执行
NSLog(@"after---%@",[NSThread currentThread]); // 打印当前线程
});
}
```

### 输出结果：

```other
2018-02-23 20:53:08.713784+0800 YSC-GCD-demo[20282:5080295] currentThread---<NSThread: 0x60000006ee00>{number = 1, name = main}
2018-02-23 20:53:08.713962+0800 YSC-GCD-demo[20282:5080295] asyncMain---begin
2018-02-23 20:53:10.714283+0800 YSC-GCD-demo[20282:5080295] after---<NSThread: 0x60000006ee00>{number = 1, name = main}
```

### 在`dispatch_after`相关代码执行结果中可以看出：在打印 `asyncMain---begin` 之后大约 2.0 秒的时间，打印了 `after---<NSThread: 0x60000006ee00>{number = 1, name = main}`

## 6.3 GCD 一次性代码（只执行一次）：dispatch_once

- ### 我们在创建单例、或者有整个程序运行过程中只执行一次的代码时，我们就用到了 GCD 的 `dispatch_once` 函数。使用

### `dispatch_once` 函数能保证某段代码在程序运行过程中只被执行1次，并且即使在多线程的环境下，`dispatch_once`也可以保证线程安全。

```other
/**
* 一次性代码（只执行一次）dispatch_once
*/
- (void)once {
static dispatch_once_t onceToken;
dispatch_once(&onceToken, ^{
// 只执行1次的代码(这里面默认是线程安全的)
});
}
```

## 6.4 GCD 快速迭代方法：dispatch_apply

- ### 通常我们会用 for 循环遍历，但是 GCD 给我们提供了快速迭代的函数`dispatch_apply`。`dispatch_apply`按照指定的次数将指定的任务追加到指定的队列中，并等待全部队列执行结束。

### 我们可以利用异步队列同时遍历。比如说遍历 0~5 这6个数字，for 循环的做法是每次取出一个元素，逐个遍历。`dispatch_apply`可以同时遍历多个数字。

```other
/**
* 快速迭代方法 dispatch_apply
*/
- (void)apply {
dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
NSLog(@"apply---begin");
dispatch_apply(6, queue, ^(size_t index) {
NSLog(@"%zd---%@",index, [NSThread currentThread]);
});
NSLog(@"apply---end");
}
```

### 输出结果：

```other
2018-02-23 22:03:18.475499+0800 YSC-GCD-demo[20470:5176805] apply---begin
2018-02-23 22:03:18.476672+0800 YSC-GCD-demo[20470:5177035] 1---<NSThread: 0x60000027b8c0>{number = 3, name = (null)}
2018-02-23 22:03:18.476693+0800 YSC-GCD-demo[20470:5176805] 0---<NSThread: 0x604000070640>{number = 1, name = main}
2018-02-23 22:03:18.476704+0800 YSC-GCD-demo[20470:5177037] 2---<NSThread: 0x604000276800>{number = 4, name = (null)}
2018-02-23 22:03:18.476735+0800 YSC-GCD-demo[20470:5177036] 3---<NSThread: 0x60000027b800>{number = 5, name = (null)}
2018-02-23 22:03:18.476867+0800 YSC-GCD-demo[20470:5177035] 4---<NSThread: 0x60000027b8c0>{number = 3, name = (null)}
2018-02-23 22:03:18.476867+0800 YSC-GCD-demo[20470:5176805] 5---<NSThread: 0x604000070640>{number = 1, name = main}
2018-02-23 22:03:18.477038+0800 YSC-GCD-demo[20470:5176805] apply---end
```

### 从`dispatch_apply`相关代码执行结果中可以看出：

- ### 0~5 打印顺序不定，最后打印了 `apply---end`。

### 因为是在并发队列中异步队执行任务，所以各个任务的执行时间长短不定，最后结束顺序也不定。但是`apply---end`一定在最后执行。这是因为`dispatch_apply`函数会等待全部任务执行完毕。

## 6.5 GCD 的队列组：dispatch_group

### 有时候我们会有这样的需求：分别异步执行2个耗时任务，然后当2个耗时任务都执行完毕后再回到主线程执行任务。这时候我们可以用到 GCD 的队列组。

- ### 调用队列组的 `dispatch_group_async` 先把任务放到队列中，然后将队列放入队列组中。或者使用队列组的 `dispatch_group_enter、dispatch_group_leave` 组合 来实现`dispatch_group_async`。
- ### 调用队列组的 `dispatch_group_notify` 回到指定线程执行任务。或者使用 `dispatch_group_wait` 回到当前线程继续向下执行（会阻塞当前线程）。

## 6.5.1 dispatch_group_notify

- ### 监听 group 中任务的完成状态，当所有的任务都执行完成后，追加任务到 group 中，并执行任务

```other
/**
* 队列组 dispatch_group_notify
*/
- (void)groupNotify {
NSLog(@"currentThread---%@",[NSThread currentThread]); // 打印当前线程
NSLog(@"group---begin");
dispatch_group_t group = dispatch_group_create();
dispatch_group_async(group, dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
// 追加任务1
for (int i = 0; i < 2; ++i) {
[NSThread sleepForTimeInterval:2]; // 模拟耗时操作
NSLog(@"1---%@",[NSThread currentThread]); // 打印当前线程
}
});
dispatch_group_async(group, dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
// 追加任务2
for (int i = 0; i < 2; ++i) {
[NSThread sleepForTimeInterval:2]; // 模拟耗时操作
NSLog(@"2---%@",[NSThread currentThread]); // 打印当前线程
}
});
dispatch_group_notify(group, dispatch_get_main_queue(), ^{
// 等前面的异步任务1、任务2都执行完毕后，回到主线程执行下边任务
for (int i = 0; i < 2; ++i) {
[NSThread sleepForTimeInterval:2]; // 模拟耗时操作
NSLog(@"3---%@",[NSThread currentThread]); // 打印当前线程
}
NSLog(@"group---end");
});
}
```

### 输出结果：

```other
2018-02-23 22:05:03.790035+0800 YSC-GCD-demo[20494:5183349] currentThread---<NSThread: 0x604000072040>{number = 1, name = main}
2018-02-23 22:05:03.790237+0800 YSC-GCD-demo[20494:5183349] group---begin
2018-02-23 22:05:05.792721+0800 YSC-GCD-demo[20494:5183654] 1---<NSThread: 0x60000026f280>{number = 4, name = (null)}
2018-02-23 22:05:05.792725+0800 YSC-GCD-demo[20494:5183656] 2---<NSThread: 0x60000026f240>{number = 3, name = (null)}
2018-02-23 22:05:07.797408+0800 YSC-GCD-demo[20494:5183656] 2---<NSThread: 0x60000026f240>{number = 3, name = (null)}
2018-02-23 22:05:07.797408+0800 YSC-GCD-demo[20494:5183654] 1---<NSThread: 0x60000026f280>{number = 4, name = (null)}
2018-02-23 22:05:09.798717+0800 YSC-GCD-demo[20494:5183349] 3---<NSThread: 0x604000072040>{number = 1, name = main}
2018-02-23 22:05:11.799827+0800 YSC-GCD-demo[20494:5183349] 3---<NSThread: 0x604000072040>{number = 1, name = main}
2018-02-23 22:05:11.799977+0800 YSC-GCD-demo[20494:5183349] group---end
```

### **从`dispatch_group_notify`相关代码运行输出结果可以看出： 当所有任务都执行完成之后，才执行`dispatch_group_notify block` 中的任务。**

## 6.5.2 dispatch_group_wait

- ### 暂停当前线程（阻塞当前线程），等待指定的 group 中的任务执行完成后，才会往下继续执行。

```other
/**
* 队列组 dispatch_group_wait
*/
- (void)groupWait {
NSLog(@"currentThread---%@",[NSThread currentThread]); // 打印当前线程
NSLog(@"group---begin");
dispatch_group_t group = dispatch_group_create();
dispatch_group_async(group, dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
// 追加任务1
for (int i = 0; i < 2; ++i) {
[NSThread sleepForTimeInterval:2]; // 模拟耗时操作
NSLog(@"1---%@",[NSThread currentThread]); // 打印当前线程
}
});
dispatch_group_async(group, dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
// 追加任务2
for (int i = 0; i < 2; ++i) {
[NSThread sleepForTimeInterval:2]; // 模拟耗时操作
NSLog(@"2---%@",[NSThread currentThread]); // 打印当前线程
}
});
// 等待上面的任务全部完成后，会往下继续执行（会阻塞当前线程）
dispatch_group_wait(group, DISPATCH_TIME_FOREVER);
NSLog(@"group---end");
}
```

### 输出结果：

```other
2018-02-23 22:10:16.939258+0800 YSC-GCD-demo[20538:5198871] currentThread---<NSThread: 0x600000066780>{number = 1, name = main}
2018-02-23 22:10:16.939455+0800 YSC-GCD-demo[20538:5198871] group---begin
2018-02-23 22:10:18.943862+0800 YSC-GCD-demo[20538:5199137] 2---<NSThread: 0x600000464b80>{number = 4, name = (null)}
2018-02-23 22:10:18.943861+0800 YSC-GCD-demo[20538:5199138] 1---<NSThread: 0x604000076640>{number = 3, name = (null)}
2018-02-23 22:10:20.947787+0800 YSC-GCD-demo[20538:5199137] 2---<NSThread: 0x600000464b80>{number = 4, name = (null)}
2018-02-23 22:10:20.947790+0800 YSC-GCD-demo[20538:5199138] 1---<NSThread: 0x604000076640>{number = 3, name = (null)}
2018-02-23 22:10:20.948134+0800 YSC-GCD-demo[20538:5198871] group---end
```

### 从`dispatch_group_wait`相关代码运行输出结果可以看出：

### 当所有任务执行完成之后，才执行 `dispatch_group_wait` 之后的操作。但是，使用`dispatch_group_wait` 会阻塞当前线程。

## 6.5.3 dispatch_group_enter、dispatch_group_leave

- ### `dispatch_group_enter` 标志着一个任务追加到 group，执行一次，相当于 group 中未执行完毕任务数+1
- ### `dispatch_group_leave`标志着一个任务离开了 group，执行一次，相当于 group 中未执行完毕任务数-1。
- ### 当 group 中未执行完毕任务数为0的时候，才会使`dispatch_group_wait`解除阻塞，以及执行追加到`dispatch_group_notify`中的任务。

```other
/**
* 队列组 dispatch_group_enter、dispatch_group_leave
*/
- (void)groupEnterAndLeave
{
NSLog(@"currentThread---%@",[NSThread currentThread]); // 打印当前线程
NSLog(@"group---begin");
dispatch_group_t group = dispatch_group_create();
dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
dispatch_group_enter(group);
dispatch_async(queue, ^{
// 追加任务1
for (int i = 0; i < 2; ++i) {
[NSThread sleepForTimeInterval:2]; // 模拟耗时操作
NSLog(@"1---%@",[NSThread currentThread]); // 打印当前线程
}
dispatch_group_leave(group);
});
dispatch_group_enter(group);
dispatch_async(queue, ^{
// 追加任务2
for (int i = 0; i < 2; ++i) {
[NSThread sleepForTimeInterval:2]; // 模拟耗时操作
NSLog(@"2---%@",[NSThread currentThread]); // 打印当前线程
}
dispatch_group_leave(group);
});
dispatch_group_notify(group, dispatch_get_main_queue(), ^{
// 等前面的异步操作都执行完毕后，回到主线程.
for (int i = 0; i < 2; ++i) {
[NSThread sleepForTimeInterval:2]; // 模拟耗时操作
NSLog(@"3---%@",[NSThread currentThread]); // 打印当前线程
}
NSLog(@"group---end");
});
// // 等待上面的任务全部完成后，会往下继续执行（会阻塞当前线程）
// dispatch_group_wait(group, DISPATCH_TIME_FOREVER);
//
// NSLog(@"group---end");
}
```

### 输出结果：

```other
2018-02-23 22:14:17.997667+0800 YSC-GCD-demo[20592:5214830] currentThread---<NSThread: 0x604000066600>{number = 1, name = main}
2018-02-23 22:14:17.997839+0800 YSC-GCD-demo[20592:5214830] group---begin
2018-02-23 22:14:20.000298+0800 YSC-GCD-demo[20592:5215094] 1---<NSThread: 0x600000277c80>{number = 4, name = (null)}
2018-02-23 22:14:20.000305+0800 YSC-GCD-demo[20592:5215095] 2---<NSThread: 0x600000277c40>{number = 3, name = (null)}
2018-02-23 22:14:22.001323+0800 YSC-GCD-demo[20592:5215094] 1---<NSThread: 0x600000277c80>{number = 4, name = (null)}
2018-02-23 22:14:22.001339+0800 YSC-GCD-demo[20592:5215095] 2---<NSThread: 0x600000277c40>{number = 3, name = (null)}
2018-02-23 22:14:24.002321+0800 YSC-GCD-demo[20592:5214830] 3---<NSThread: 0x604000066600>{number = 1, name = main}
2018-02-23 22:14:26.002852+0800 YSC-GCD-demo[20592:5214830] 3---<NSThread: 0x604000066600>{number = 1, name = main}
2018-02-23 22:14:26.003116+0800 YSC-GCD-demo[20592:5214830] group---end
```

### 从`dispatch_group_enter、dispatch_group_leave`相关代码运行结果中可以看出：当所有任务执行完成之后，才执行`dispatch_group_notify`中的任务。这里的`dispatch_group_enter、dispatch_group_leave`组合，其实等同于`dispatch_group_async`。

## 6.6 GCD 信号量：dispatch_semaphore

### GCD 中的信号量是指 Dispatch Semaphore，是持有计数的信号。类似于过高速路收费站的栏杆。可以通过时，打开栏杆，不可以通过时，关闭栏杆。在 Dispatch Semaphore 中，使用计数来完成这个功能，计数为0时等待，不可通过。计数为1或大于1时，计数减1且不等待，可通过。

### Dispatch Semaphore 提供了三个函数。

- ### `dispatch_semaphore_create：`创建一个Semaphore并初始化信号的总量
- ### `dispatch_semaphore_signal：`发送一个信号，让信号总量加1
- ### `dispatch_semaphore_wait：`可以使总信号量减1，当信号总量为0时就会一直等待（阻塞所在线程），否则就可以正常执行。

> ### 注意：信号量的使用前提是：想清楚你需要处理哪个线程等待（阻塞），又要哪个线程继续执行，然后使用信号量。

### **Dispatch Semaphore 在实际开发中主要用于：**

- ### 保持线程同步，将异步执行任务转换为同步执行任务
- ### 保证线程安全，为线程加锁

## 6.6.1 Dispatch Semaphore 线程同步

### 我们在开发中，会遇到这样的需求：异步执行耗时任务，并使用异步执行的结果进行一些额外的操作。换句话说，相当于，将将异步执行任务转换为同步执行任务。比如说：AFNetworking 中 AFURLSessionManager.m 里面的 tasksForKeyPath: 方法。通过引入信号量的方式，等待异步执行任务结果，获取到 tasks，然后再返回该 tasks。

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

### 复制代码下面，我们来利用 Dispatch Semaphore 实现线程同步，将异步执行任务转换为同步执行任务。

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

### 输出结果：

```other
2018-02-23 22:22:26.521665+0800 YSC-GCD-demo[20642:5246341] currentThread---<NSThread: 0x60400006bc80>{number = 1, name = main}
2018-02-23 22:22:26.521869+0800 YSC-GCD-demo[20642:5246341] semaphore---begin
2018-02-23 22:22:28.526841+0800 YSC-GCD-demo[20642:5246638] 1---<NSThread: 0x600000272300>{number = 3, name = (null)}
2018-02-23 22:22:28.527030+0800 YSC-GCD-demo[20642:5246341] semaphore---end,number = 100
```

### **从 Dispatch Semaphore 实现线程同步的代码可以看到：**

### `semaphore---end` 是在执行完 number = 100; 之后才打印的。而且输出结果 number 为 100。

### 这是因为异步执行不会做任何等待，可以继续执行任务。异步执行将任务1追加到队列之后，不做等待，接着执行`dispatch_semaphore_wait`方法。此时 semaphore  ::0，当前线程进入等待状态。然后，异步任务1开始执行。任务1执行到`dispatch_semaphore_signal`之后，总信号量，此时 semaphore::  1，`dispatch_semaphore_wait`方法使总信号量减1，正在被阻塞的线程（主线程）恢复继续执行。最后打印`semaphore---end`,number = 100。这样就实现了线程同步，将异步执行任务转换为同步执行任务。

## 6.6.2 Dispatch Semaphore 线程安全和线程同步（为线程加锁）

### **线程安全**：如果你的代码所在的进程中有多个线程在同时运行，而这些线程可能会同时运行这段代码。如果每次运行结果和单线程运行的结果是一样的，而且其他的变量的值也和预期的是一样的，就是线程安全的。

### 若每个线程中对全局变量、静态变量只有读操作，而无写操作，一般来说，这个全局变量是线程安全的；若有多个线程同时执行写操作（更改变量），一般都需要考虑线程同步，否则的话就可能影响线程安全。

### **线程同步**：可理解为线程 A 和 线程 B 一块配合，A 执行到一定程度时要依靠线程 B 的某个结果，于是停下来，示意 B 运行；B 依言执行，再将结果给 A；A 再继续操作。

### 举个简单例子就是：两个人在一起聊天。两个人不能同时说话，避免听不清(操作冲突)。等一个人说完(一个线程结束操作)，另一个再说(另一个线程再开始操作)。

### 下面，我们模拟火车票售卖的方式，实现 NSThread 线程安全和解决线程同步问题。

### 场景：总共有50张火车票，有两个售卖火车票的窗口，一个是北京火车票售卖窗口，另一个是上海火车票售卖窗口。两个窗口同时售卖火车票，卖完为止。

## 6.6.2.1 非线程安全（不使用 semaphore）

### 先来看看不考虑线程安全的代码：

```other
/**
* 非线程安全：不使用 semaphore
* 初始化火车票数量、卖票窗口(非线程安全)、并开始卖票
*/
- (void)initTicketStatusNotSave {
NSLog(@"currentThread---%@",[NSThread currentThread]); // 打印当前线程
NSLog(@"semaphore---begin");
self.ticketSurplusCount = 50;
// queue1 代表北京火车票售卖窗口
dispatch_queue_t queue1 = dispatch_queue_create("net.bujige.testQueue1", DISPATCH_QUEUE_SERIAL);
// queue2 代表上海火车票售卖窗口
dispatch_queue_t queue2 = dispatch_queue_create("net.bujige.testQueue2", DISPATCH_QUEUE_SERIAL);
__weak typeof(self) weakSelf = self;
dispatch_async(queue1, ^{
[weakSelf saleTicketNotSafe];
});
dispatch_async(queue2, ^{
[weakSelf saleTicketNotSafe];
});
}
/**
* 售卖火车票(非线程安全)
*/
- (void)saleTicketNotSafe {
while (1) {
if (self.ticketSurplusCount > 0) { //如果还有票，继续售卖
self.ticketSurplusCount--;
NSLog(@"%@", [NSString stringWithFormat:@"剩余票数：%d 窗口：%@", self.ticketSurplusCount, [NSThread currentThread]]);
[NSThread sleepForTimeInterval:0.2];
} else { //如果已卖完，关闭售票窗口
NSLog(@"所有火车票均已售完");
break;
}
}
}
```

### 输出结果（部分）：

```other
2018-02-23 22:25:35.789072+0800 YSC-GCD-demo[20712:5258914] currentThread---<NSThread: 0x604000068880>{number = 1, name = main}
2018-02-23 22:25:35.789260+0800 YSC-GCD-demo[20712:5258914] semaphore---begin
2018-02-23 22:25:35.789641+0800 YSC-GCD-demo[20712:5259176] 剩余票数：48 窗口：<NSThread: 0x60000027db80>{number = 3, name = (null)}
2018-02-23 22:25:35.789646+0800 YSC-GCD-demo[20712:5259175] 剩余票数：49 窗口：<NSThread: 0x60000027e740>{number = 4, name = (null)}
2018-02-23 22:25:35.994113+0800 YSC-GCD-demo[20712:5259175] 剩余票数：47 窗口：<NSThread: 0x60000027e740>{number = 4, name = (null)}
2018-02-23 22:25:35.994129+0800 YSC-GCD-demo[20712:5259176] 剩余票数：46 窗口：<NSThread: 0x60000027db80>{number = 3, name = (null)}
2018-02-23 22:25:36.198993+0800 YSC-GCD-demo[20712:5259176] 剩余票数：45 窗口：<NSThread: 0x60000027db80>{number = 3, name = (null)}
......
```

### 可以看到在不考虑线程安全，不使用 semaphore 的情况下，得到票数是错乱的，这样显然不符合我们的需求，所以我们需要考虑线程安全问题。

## 6.6.2.2 线程安全（使用 semaphore 加锁）

### 考虑线程安全的代码：

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

### 输出结果为：

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

### 可以看出，在考虑了线程安全的情况下，使用 `dispatch_semaphore`

### 机制之后，得到的票数是正确的，没有出现混乱的情况。我们也就解决了多个线程同步的问题。

# iOS多线程：『NSOperation、NSOperationQueue』详尽总结

### 本文用来介绍 iOS 多线程中 NSOperation、NSOperationQueue 的相关知识以及使用方法。

### 通过本文，您将了解到：

- ### **NSOperation、NSOperationQueue 简介**
- ### **操作和操作队列**
- ### **使用步骤和基本使用方法**
- ### **控制串行/并发执行**
- ### **NSOperation 操作依赖和优先级**
- ### **线程间的通信**
- ### **线程同步和线程安全**
- ### **NSOperation、NSOperationQueue 常用属性和方法归纳**

### 文中 Demo 我已放在了 Github 上，Demo 链接：[传送门](https://link.juejin.im/?target=https%3A%2F%2Fgithub.com%2Fbujige%2FYSC-NSOperation-demo)

# 1. NSOperation、NSOperationQueue 简介

### NSOperation、NSOperationQueue 是苹果提供给我们的一套多线程解决方案。实际上 NSOperation、NSOperationQueue 是基于 GCD 更高一层的封装，完全面向对象。但是比 GCD 更简单易用、代码可读性也更高。

### **为什么要使用 NSOperation、NSOperationQueue？**

- ### 可添加完成的代码块，在操作完成后执行。
- ### 添加操作之间的依赖关系，方便的控制执行顺序。
- ### 设定操作执行的优先级。
- ### 可以很方便的取消一个操作的执行。
- ### 使用 KVO 观察对操作执行状态的更改：isExecuteing、isFinished、isCancelled。

# 2. NSOperation、NSOperationQueue 操作和操作队列

### 既然是基于 GCD 的更高一层的封装。那么，GCD 中的一些概念同样适用于 NSOperation、NSOperationQueue。在 NSOperation、NSOperationQueue 中也有类似的**任务（操作）** 和 **队列（操作队列）** 的概念。

- ### **操作（Operation）**：
   - ### 执行操作的意思，换句话说就是你在线程中执行的那段代码。
   - ### 在 GCD 中是放在 block 中的。在 NSOperation 中，我们使用 NSOperation 子类 **NSInvocationOperation、NSBlockOperation**，或者自定义子类来封装操作。
- ### **操作队列（Operation Queues）**：
   - ### 这里的队列指操作队列，即用来存放操作的队列。不同于 GCD 中的调度队列 FIFO（先进先出）的原则。NSOperationQueue 对于添加到队列中的操作，首先进入准备就绪的状态（就绪状态取决于操作之间的依赖关系），然后进入就绪状态的操作的开始执行顺序（非结束执行顺序）由操作之间相对的优先级决定（优先级是操作对象自身的属性）。
   - ### 操作队列通过设置 **最大并发操作数（maxConcurrentOperationCount）** 来控制并发、串行。
   - ### NSOperationQueue 为我们提供了两种不同类型的队列：主队列和自定义队列。主队列运行在主线程之上，而自定义队列在后台执行。

# 3. NSOperation、NSOperationQueue 使用步骤

### NSOperation 需要配合 NSOperationQueue 来实现多线程。因为默认情况下，NSOperation 单独使用时系统同步执行操作，配合 NSOperationQueue 我们能更好的实现异步执行。

### **NSOperation 实现多线程的使用步骤分为三步：**

- ### 创建操作：先将需要执行的操作封装到一个 NSOperation 对象中。
- ### 创建队列：创建 NSOperationQueue 对象。
- ### 将操作加入到队列中：将 NSOperation 对象添加到 NSOperationQueue 对象中。

### 之后呢，系统就会自动将 NSOperationQueue 中的 NSOperation 取出来，在新线程中执行操作。

### 下面我们来学习下 NSOperation 和 NSOperationQueue 的基本使用。

# 4. NSOperation 和 NSOperationQueue 基本使用

## 4.1 创建操作

### NSOperation 是个抽象类，不能用来封装操作。我们只有使用它的子类来封装操作。我们有三种方式来封装操作。

- ### 使用子类 NSInvocationOperation
- ### 使用子类 NSBlockOperation
- ### 自定义继承自 NSOperation 的子类，通过实现内部相应的方法来封装操作。

### 在不使用 NSOperationQueue，单独使用 NSOperation 的情况下系统同步执行操作，下面我们学习以下操作的三种创建方式。

## 4.1.1 使用子类 NSInvocationOperation

```other
/**
* 使用子类 NSInvocationOperation
*/
- (void)useInvocationOperation {
// 1.创建 NSInvocationOperation 对象
NSInvocationOperation *op = [[NSInvocationOperation alloc] initWithTarget:self selector:@selector(task1) object:nil];
// 2.调用 start 方法开始执行操作
[op start];
}
/**
* 任务1
*/
- (void)task1 {
for (int i = 0; i < 2; i++) {
[NSThread sleepForTimeInterval:2]; // 模拟耗时操作
NSLog(@"1---%@", [NSThread currentThread]); // 打印当前线程
}
}
```

![2969810-5c0b46b62a847e25.png](https://upload-images.jianshu.io/upload_images/2969810-5c0b46b62a847e25.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- ### 可以看到：在没有使用 NSOperationQueue、在主线程中单独使用使用子类 NSInvocationOperation 执行一个操作的情况下，操作是在当前线程执行的，并没有开启新线程。

### 如果在其他线程中执行操作，则打印结果为其他线程。

```other
// 在其他线程使用子类 NSInvocationOperation
[NSThread detachNewThreadSelector:@selector(useInvocationOperation) toTarget:self withObject:nil];
```

### 下边再来看看 NSBlockOperation。

## 4.1.2 使用子类 NSBlockOperation

```other
/**
* 使用子类 NSBlockOperation
*/
- (void)useBlockOperation {
// 1.创建 NSBlockOperation 对象
NSBlockOperation *op = [NSBlockOperation blockOperationWithBlock:^{
for (int i = 0; i < 2; i++) {
[NSThread sleepForTimeInterval:2]; // 模拟耗时操作
NSLog(@"1---%@", [NSThread currentThread]); // 打印当前线程
}
}];
// 2.调用 start 方法开始执行操作
[op start];
}
```

![2969810-7884f8d94910e48b.png](https://upload-images.jianshu.io/upload_images/2969810-7884f8d94910e48b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- ### 可以看到：在没有使用 NSOperationQueue、在主线程中单独使用 NSBlockOperation 执行一个操作的情况下，操作是在当前线程执行的，并没有开启新线程。

> ### 注意：和上边 NSInvocationOperation 使用一样。因为代码是在主线程中调用的，所以打印结果为主线程。如果在其他线程中执行操作，则打印结果为其他线程。
但是，NSBlockOperation 还提供了一个方法 `addExecutionBlock`:，通过 `addExecutionBlock:` 就可以为 NSBlockOperation 添加额外的操作。这些操作（包括 `blockOperationWithBlock` 中的操作）可以在不同的线程中同时（并发）执行。只有当所有相关的操作已经完成执行时，才视为完成。
如果添加的操作多的话，`blockOperationWithBlock:` 中的操作也可能会在其他线程（非当前线程）中执行，这是由系统决定的，并不是说添加到 `blockOperationWithBlock:` 中的操作一定会在当前线程中执行。（可以使用 `addExecutionBlock:` 多添加几个操作试试）。

```other
/**
* 使用子类 NSBlockOperation
* 调用方法 AddExecutionBlock:
*/
- (void)useBlockOperationAddExecutionBlock {
// 1.创建 NSBlockOperation 对象
NSBlockOperation *op = [NSBlockOperation blockOperationWithBlock:^{
for (int i = 0; i < 2; i++) {
[NSThread sleepForTimeInterval:2]; // 模拟耗时操作
NSLog(@"1---%@", [NSThread currentThread]); // 打印当前线程
}
}];
// 2.添加额外的操作
[op addExecutionBlock:^{
for (int i = 0; i < 2; i++) {
[NSThread sleepForTimeInterval:2]; // 模拟耗时操作
NSLog(@"2---%@", [NSThread currentThread]); // 打印当前线程
}
}];
[op addExecutionBlock:^{
for (int i = 0; i < 2; i++) {
[NSThread sleepForTimeInterval:2]; // 模拟耗时操作
NSLog(@"3---%@", [NSThread currentThread]); // 打印当前线程
}
}];
[op addExecutionBlock:^{
for (int i = 0; i < 2; i++) {
[NSThread sleepForTimeInterval:2]; // 模拟耗时操作
NSLog(@"4---%@", [NSThread currentThread]); // 打印当前线程
}
}];
[op addExecutionBlock:^{
for (int i = 0; i < 2; i++) {
[NSThread sleepForTimeInterval:2]; // 模拟耗时操作
NSLog(@"5---%@", [NSThread currentThread]); // 打印当前线程
}
}];
[op addExecutionBlock:^{
for (int i = 0; i < 2; i++) {
[NSThread sleepForTimeInterval:2]; // 模拟耗时操作
NSLog(@"6---%@", [NSThread currentThread]); // 打印当前线程
}
}];
[op addExecutionBlock:^{
for (int i = 0; i < 2; i++) {
[NSThread sleepForTimeInterval:2]; // 模拟耗时操作
NSLog(@"7---%@", [NSThread currentThread]); // 打印当前线程
}
}];
[op addExecutionBlock:^{
for (int i = 0; i < 2; i++) {
[NSThread sleepForTimeInterval:2]; // 模拟耗时操作
NSLog(@"8---%@", [NSThread currentThread]); // 打印当前线程
}
}];
// 3.调用 start 方法开始执行操作
[op start];
}
```

![2969810-7877f098b0be908b.png](https://upload-images.jianshu.io/upload_images/2969810-7877f098b0be908b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- ### 可以看出：使用子类 NSBlockOperation，并调用方法 AddExecutionBlock: 的情况下，`blockOperationWithBlock:`方法中的操作 和 `addExecutionBlock:` 中的操作是在不同的线程中异步执行的。而且，这次执行结果中 `blockOperationWithBlock:`方法中的操作也不是在当前线程（主线程）中执行的。从而印证了`blockOperationWithBlock:` 中的操作也可能会在其他线程（非当前线程）中执行。

### 一般情况下，如果一个 NSBlockOperation 对象封装了多个操作。NSBlockOperation 是否开启新线程，取决于操作的个数。如果添加的操作的个数多，就会自动开启新线程。当然开启的线程数是由系统来决定的。

## 4.1.3 使用自定义继承自 NSOperation 的子类

### 如果使用子类 `NSInvocationOperation、NSBlockOperation` 不能满足日常需求，我们可以使用自定义继承自 NSOperation 的子类。可以通过重写 `main` 或者 `start` 方法 来定义自己的 NSOperation 对象。重写main方法比较简单，我们不需要管理操作的状态属性 `isExecuting` 和 `isFinished`。当 `main` 执行完返回的时候，这个操作就结束了。

### 先定义一个继承自 NSOperation 的子类，重写main方法。

```other
// YSCOperation.h 文件
#import <Foundation/Foundation.h>
@interface YSCOperation : NSOperation
@end
// YSCOperation.m 文件
#import "YSCOperation.h"
@implementation YSCOperation
- (void)main {
if (!self.isCancelled) {
for (int i = 0; i < 2; i++) {
[NSThread sleepForTimeInterval:2];
NSLog(@"1---%@", [NSThread currentThread]);
}
}
}
@end
```

### 然后使用的时候导入头文件YSCOperation.h。

```other
/**
* 使用自定义继承自 NSOperation 的子类
*/
- (void)useCustomOperation {
// 1.创建 YSCOperation 对象
YSCOperation *op = [[YSCOperation alloc] init];
// 2.调用 start 方法开始执行操作
[op start];
}
```

![2969810-f8d8e1a6dfab8d01.png](https://upload-images.jianshu.io/upload_images/2969810-f8d8e1a6dfab8d01.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- ### 可以看出：在没有使用 NSOperationQueue、在主线程单独使用自定义继承自 NSOperation 的子类的情况下，是在主线程执行操作，并没有开启新线程。

### 下边我们来讲讲 NSOperationQueue 的创建。

## 4.2 创建队列

### NSOperationQueue 一共有两种队列：主队列、自定义队列。其中自定义队列同时包含了串行、并发功能。下边是主队列、自定义队列的基本创建方法和特点。

- ### 主队列
- ### 凡是添加到主队列中的操作，都会放到主线程中执行。

```other
// 主队列获取方法
NSOperationQueue *queue = [NSOperationQueue mainQueue];
```

- ### 自定义队列（非主队列）
- ### 添加到这种队列中的操作，就会自动放到子线程中执行。
- ### 同时包含了：串行、并发功能。

```other
// 自定义队列创建方法
NSOperationQueue *queue = [[NSOperationQueue alloc] init];
```

## 4.3 将操作加入到队列中

### 上边我们说到 NSOperation 需要配合 NSOperationQueue 来实现多线程。

### 那么我们需要将创建好的操作加入到队列中去。总共有两种方法：

1. ### `-(void)addOperation:(NSOperation *)op;`
- ### 需要先创建操作，再将创建好的操作加入到创建好的队列中去。

```other
/**
* 使用 addOperation: 将操作加入到操作队列中
*/
- (void)addOperationToQueue {
// 1.创建队列
NSOperationQueue *queue = [[NSOperationQueue alloc] init];
// 2.创建操作
// 使用 NSInvocationOperation 创建操作1
NSInvocationOperation *op1 = [[NSInvocationOperation alloc] initWithTarget:self selector:@selector(task1) object:nil];
// 使用 NSInvocationOperation 创建操作2
NSInvocationOperation *op2 = [[NSInvocationOperation alloc] initWithTarget:self selector:@selector(task2) object:nil];
// 使用 NSBlockOperation 创建操作3
NSBlockOperation *op3 = [NSBlockOperation blockOperationWithBlock:^{
for (int i = 0; i < 2; i++) {
[NSThread sleepForTimeInterval:2]; // 模拟耗时操作
NSLog(@"3---%@", [NSThread currentThread]); // 打印当前线程
}
}];
[op3 addExecutionBlock:^{
for (int i = 0; i < 2; i++) {
[NSThread sleepForTimeInterval:2]; // 模拟耗时操作
NSLog(@"4---%@", [NSThread currentThread]); // 打印当前线程
}
}];
// 3.使用 addOperation: 添加所有操作到队列中
[queue addOperation:op1]; // [op1 start]
[queue addOperation:op2]; // [op2 start]
[queue addOperation:op3]; // [op3 start]
}
```

![2969810-e08a2a5e3c2ce574.png](https://upload-images.jianshu.io/upload_images/2969810-e08a2a5e3c2ce574.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- ### 可以看出：使用 NSOperation 子类创建操作，并使用 addOperation: 将操作加入到操作队列后能够开启新线程，进行并发执行。
2. ### `-(void)addOperationWithBlock:(void (^)(void))block;`
- ### 无需先创建操作，在 block 中添加操作，直接将包含操作的 block 加入到队列中。

```other
/**
* 使用 addOperationWithBlock: 将操作加入到操作队列中
*/
- (void)addOperationWithBlockToQueue {
// 1.创建队列
NSOperationQueue *queue = [[NSOperationQueue alloc] init];
// 2.使用 addOperationWithBlock: 添加操作到队列中
[queue addOperationWithBlock:^{
for (int i = 0; i < 2; i++) {
[NSThread sleepForTimeInterval:2]; // 模拟耗时操作
NSLog(@"1---%@", [NSThread currentThread]); // 打印当前线程
}
}];
[queue addOperationWithBlock:^{
for (int i = 0; i < 2; i++) {
[NSThread sleepForTimeInterval:2]; // 模拟耗时操作
NSLog(@"2---%@", [NSThread currentThread]); // 打印当前线程
}
}];
[queue addOperationWithBlock:^{
for (int i = 0; i < 2; i++) {
[NSThread sleepForTimeInterval:2]; // 模拟耗时操作
NSLog(@"3---%@", [NSThread currentThread]); // 打印当前线程
}
}];
}
```

![2969810-34f25d809fc3c1d4.png](https://upload-images.jianshu.io/upload_images/2969810-34f25d809fc3c1d4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- ### 可以看出：使用 addOperationWithBlock: 将操作加入到操作队列后能够开启新线程，进行并发执行。

## 5. NSOperationQueue 控制串行执行、并发执行

### 之前我们说过，NSOperationQueue 创建的自定义队列同时具有串行、并发功能，上边我们演示了并发功能，那么他的串行功能是如何实现的？

### 这里有个关键属性 `maxConcurrentOperationCount`，叫做最大并发操作数。用来控制一个特定队列中可以有多少个操作同时参与并发执行。

> ### 注意：这里 maxConcurrentOperationCount 控制的不是并发线程的数量，而是一个队列中同时能并发执行的最大操作数。而且一个操作也并非只能在一个线程中运行。

- ### 最大并发操作数：`maxConcurrentOperationCount`
- ### `maxConcurrentOperationCount` 默认情况下为-1，表示不进行限制，可进行并发执行。
- ### `maxConcurrentOperationCount` 为1时，队列为串行队列。只能串行执行。
- ### `maxConcurrentOperationCount` 大于1时，队列为并发队列。操作并发执行，当然这个值不应超过系统限制，即使自己设置一个很大的值，系统也会自动调整为 min{自己设定的值，系统设定的默认最大值}。

```other
/**
* 设置 MaxConcurrentOperationCount（最大并发操作数）
*/
- (void)setMaxConcurrentOperationCount {
// 1.创建队列
NSOperationQueue *queue = [[NSOperationQueue alloc] init];
// 2.设置最大并发操作数
queue.maxConcurrentOperationCount = 1; // 串行队列
// queue.maxConcurrentOperationCount = 2; // 并发队列
// queue.maxConcurrentOperationCount = 8; // 并发队列
// 3.添加操作
[queue addOperationWithBlock:^{
for (int i = 0; i < 2; i++) {
[NSThread sleepForTimeInterval:2]; // 模拟耗时操作
NSLog(@"1---%@", [NSThread currentThread]); // 打印当前线程
}
}];
[queue addOperationWithBlock:^{
for (int i = 0; i < 2; i++) {
[NSThread sleepForTimeInterval:2]; // 模拟耗时操作
NSLog(@"2---%@", [NSThread currentThread]); // 打印当前线程
}
}];
[queue addOperationWithBlock:^{
for (int i = 0; i < 2; i++) {
[NSThread sleepForTimeInterval:2]; // 模拟耗时操作
NSLog(@"3---%@", [NSThread currentThread]); // 打印当前线程
}
}];
[queue addOperationWithBlock:^{
for (int i = 0; i < 2; i++) {
[NSThread sleepForTimeInterval:2]; // 模拟耗时操作
NSLog(@"4---%@", [NSThread currentThread]); // 打印当前线程
}
}];
}
```

![2969810-1f0eb26f7bb94227.png](https://upload-images.jianshu.io/upload_images/2969810-1f0eb26f7bb94227.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![2969810-eaf96d149e1a49e8.png](https://upload-images.jianshu.io/upload_images/2969810-eaf96d149e1a49e8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- ### 可以看出：当最大并发操作数为1时，操作是按顺序串行执行的，并且一个操作完成之后，下一个操作才开始执行。当最大操作并发数为2时，操作是并发执行的，可以同时执行两个操作。而开启线程数量是由系统决定的，不需要我们来管理。

### 这样看来，是不是比 GCD 还要简单了许多？

## 6. NSOperation 操作依赖

### NSOperation、NSOperationQueue 最吸引人的地方是它能添加操作之间的依赖关系。通过操作依赖，我们可以很方便的控制操作之间的执行先后顺序。NSOperation 提供了3个接口供我们管理和查看依赖。

- ### `-(void)addDependency:(NSOperation *)op;` 添加依赖，使当前操作依赖于操作 op 的完成。
- ### `-(void)removeDependency:(NSOperation *)op;` 移除依赖，取消当前操作对操作 op 的依赖。
- ### `@property (readonly, copy) NSArray<NSOperation *> *dependencies;` 在当前操作开始执行之前完成执行的所有操作对象数组。

### 当然，我们经常用到的还是添加依赖操作。现在考虑这样的需求，比如说有 A、B 两个操作，其中 A 执行完操作，B 才能执行操作。

### 如果使用依赖来处理的话，那么就需要让操作 B 依赖于操作 A。具体代码如下：

```other
/**
* 操作依赖
* 使用方法：addDependency:
*/
- (void)addDependency {
// 1.创建队列
NSOperationQueue *queue = [[NSOperationQueue alloc] init];
// 2.创建操作
NSBlockOperation *op1 = [NSBlockOperation blockOperationWithBlock:^{
for (int i = 0; i < 2; i++) {
[NSThread sleepForTimeInterval:2]; // 模拟耗时操作
NSLog(@"1---%@", [NSThread currentThread]); // 打印当前线程
}
}];
NSBlockOperation *op2 = [NSBlockOperation blockOperationWithBlock:^{
for (int i = 0; i < 2; i++) {
[NSThread sleepForTimeInterval:2]; // 模拟耗时操作
NSLog(@"2---%@", [NSThread currentThread]); // 打印当前线程
}
}];
// 3.添加依赖
[op2 addDependency:op1]; // 让op2 依赖于 op1，则先执行op1，在执行op2
// 4.添加操作到队列中
[queue addOperation:op1];
[queue addOperation:op2];
}
```

![2969810-9f78f7ec87bcb11a.png](https://upload-images.jianshu.io/upload_images/2969810-9f78f7ec87bcb11a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- ### 可以看到：通过添加操作依赖，无论运行几次，其结果都是 op1 先执行，op2 后执行。

## 7. NSOperation 优先级

### NSOperation 提供了`queuePriority`（优先级）属性，`queuePriority`属性适用于同一操作队列中的操作，不适用于不同操作队列中的操作。默认情况下，所有新创建的操作对象优先级都是`NSOperationQueuePriorityNormal`。但是我们可以通过`setQueuePriority:`方法来改变当前操作在同一队列中的执行优先级。

```other
// 优先级的取值
typedef NS_ENUM(NSInteger, NSOperationQueuePriority) {
NSOperationQueuePriorityVeryLow = -8L,
NSOperationQueuePriorityLow = -4L,
NSOperationQueuePriorityNormal = 0,
NSOperationQueuePriorityHigh = 4,
NSOperationQueuePriorityVeryHigh = 8
};
```

### 上边我们说过：对于添加到队列中的操作，首先进入准备就绪的状态（就绪状态取决于操作之间的依赖关系），然后进入就绪状态的操作的**开始执行顺序**（非结束执行顺序）由操作之间相对的优先级决定（优先级是操作对象自身的属性）。

### **那么，什么样的操作才是进入就绪状态的操作呢？**

- ### 当一个操作的所有依赖都已经完成时，操作对象通常会进入准备就绪状态，等待执行。

### 举个例子，现在有4个优先级都是 `NSOperationQueuePriorityNormal`（默认级别）的操作：op1，op2，op3，op4。其中 op3 依赖于 op2，op2 依赖于 op1，即 op3 -> op2 -> op1。现在将这4个操作添加到队列中并发执行。

- ### 因为 op1 和 op4 都没有需要依赖的操作，所以在 op1，op4 执行之前，就是处于准备就绪状态的操作。
- ### 而 op3 和 op2 都有依赖的操作（op3 依赖于 op2，op2 依赖于 op1），所以 op3 和 op2 都不是准备就绪状态下的操作。

### 理解了进入就绪状态的操作，那么我们就理解了`queuePriority` 属性的作用对象。

- ### `queuePriority` 属性决定了进入准备就绪状态下的操作之间的开始执行顺序。并且，优先级不能取代依赖关系。
- ### 如果一个队列中既包含高优先级操作，又包含低优先级操作，并且两个操作都已经准备就绪，那么队列先执行高优先级操作。比如上例中，如果 op1 和 op4 是不同优先级的操作，那么就会先执行优先级高的操作。
- ### 如果，一个队列中既包含了准备就绪状态的操作，又包含了未准备就绪的操作，未准备就绪的操作优先级比准备就绪的操作优先级高。那么，虽然准备就绪的操作优先级低，也会优先执行。优先级不能取代依赖关系。如果要控制操作间的启动顺序，则必须使用依赖关系.

## 8. NSOperation、NSOperationQueue 线程间的通信

### 在 iOS 开发过程中，我们一般在主线程里边进行 UI 刷新，例如：点击、滚动、拖拽等事件。我们通常把一些耗时的操作放在其他线程，比如说图片下载、文件上传等耗时操作。而当我们有时候在其他线程完成了耗时操作时，需要回到主线程，那么就用到了线程之间的通讯。

```other
/**
* 线程间通信
*/
- (void)communication {
// 1.创建队列
NSOperationQueue *queue = [[NSOperationQueue alloc]init];
// 2.添加操作
[queue addOperationWithBlock:^{
// 异步进行耗时操作
for (int i = 0; i < 2; i++) {
[NSThread sleepForTimeInterval:2]; // 模拟耗时操作
NSLog(@"1---%@", [NSThread currentThread]); // 打印当前线程
}
// 回到主线程
[[NSOperationQueue mainQueue] addOperationWithBlock:^{
// 进行一些 UI 刷新等操作
for (int i = 0; i < 2; i++) {
[NSThread sleepForTimeInterval:2]; // 模拟耗时操作
NSLog(@"2---%@", [NSThread currentThread]); // 打印当前线程
}
}];
}];
}
```

![2969810-e2c274dc3d274fd1.png](https://upload-images.jianshu.io/upload_images/2969810-e2c274dc3d274fd1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- ### 可以看到：通过线程间的通信，先在其他线程中执行操作，等操作执行完了之后再回到主线程执行主线程的相应操作。

## 9. NSOperation、NSOperationQueue 线程同步和线程安全

- ### **线程安全**：如果你的代码所在的进程中有多个线程在同时运行，而这些线程可能会同时运行这段代码。如果每次运行结果和单线程运行的结果是一样的，而且其他的变量的值也和预期的是一样的，就是线程安全的。

### 若每个线程中对全局变量、静态变量只有读操作，而无写操作，一般来说，这个全局变量是线程安全的；若有多个线程同时执行写操作（更改变量），一般都需要考虑线程同步，否则的话就可能影响线程安全。

- ### **线程同步**：可理解为线程 A 和 线程 B 一块配合，A 执行到一定程度时要依靠线程 B 的某个结果，于是停下来，示意 B 运行；B 依言执行，再将结果给 A；A 再继续操作。

### 举个简单例子就是：两个人在一起聊天。两个人不能同时说话，避免听不清(操作冲突)。等一个人说完(一个线程结束操作)，另一个再说(另一个线程再开始操作)。

### 下面，我们模拟火车票售卖的方式，实现 NSOperation 线程安全和解决线程同步问题。

### 场景：总共有50张火车票，有两个售卖火车票的窗口，一个是北京火车票售卖窗口，另一个是上海火车票售卖窗口。两个窗口同时售卖火车票，卖完为止。

## 9.1 NSOperation、NSOperationQueue 非线程安全

### 先来看看不考虑线程安全的代码：

```other
/**
* 非线程安全：不使用 NSLock
* 初始化火车票数量、卖票窗口(非线程安全)、并开始卖票
*/
- (void)initTicketStatusNotSave {
NSLog(@"currentThread---%@",[NSThread currentThread]); // 打印当前线程
self.ticketSurplusCount = 50;
// 1.创建 queue1,queue1 代表北京火车票售卖窗口
NSOperationQueue *queue1 = [[NSOperationQueue alloc] init];
queue1.maxConcurrentOperationCount = 1;
// 2.创建 queue2,queue2 代表上海火车票售卖窗口
NSOperationQueue *queue2 = [[NSOperationQueue alloc] init];
queue2.maxConcurrentOperationCount = 1;
// 3.创建卖票操作 op1
__weak typeof(self) weakSelf = self;
NSBlockOperation *op1 = [NSBlockOperation blockOperationWithBlock:^{
[weakSelf saleTicketNotSafe];
}];
// 4.创建卖票操作 op2
NSBlockOperation *op2 = [NSBlockOperation blockOperationWithBlock:^{
[weakSelf saleTicketNotSafe];
}];
// 5.添加操作，开始卖票
[queue1 addOperation:op1];
[queue2 addOperation:op2];
}
/**
* 售卖火车票(非线程安全)
*/
- (void)saleTicketNotSafe {
while (1) {
if (self.ticketSurplusCount > 0) {
//如果还有票，继续售卖
self.ticketSurplusCount--;
NSLog(@"%@", [NSString stringWithFormat:@"剩余票数:%d 窗口:%@", self.ticketSurplusCount, [NSThread currentThread]]);
[NSThread sleepForTimeInterval:0.2];
} else {
NSLog(@"所有火车票均已售完");
break;
}
}
}
```

![2969810-1026614565231db1.png](https://upload-images.jianshu.io/upload_images/2969810-1026614565231db1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![2969810-000c2b576b8fd97d.png](https://upload-images.jianshu.io/upload_images/2969810-000c2b576b8fd97d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- ### 可以看到：在不考虑线程安全，不使用 NSLock 情况下，得到票数是错乱的，这样显然不符合我们的需求，所以我们需要考虑线程安全问题。

## 9.2 NSOperation、NSOperationQueue 非线程安全

### 线程安全解决方案：可以给线程加锁，在一个线程执行该操作的时候，不允许其他线程进行操作。iOS 实现线程加锁有很多种方式。@synchronized、 NSLock、NSRecursiveLock、NSCondition、NSConditionLock、pthread_mutex、dispatch_semaphore、OSSpinLock、atomic(property) set/ge等等各种方式。这里我们使用 NSLock 对象来解决线程同步问题。NSLock 对象可以通过进入锁时调用 lock 方法，解锁时调用 unlock 方法来保证线程安全。

### 考虑线程安全的代码：

```other
/**
* 线程安全：使用 NSLock 加锁
* 初始化火车票数量、卖票窗口(线程安全)、并开始卖票
*/
- (void)initTicketStatusSave {
NSLog(@"currentThread---%@",[NSThread currentThread]); // 打印当前线程
self.ticketSurplusCount = 50;
self.lock = [[NSLock alloc] init]; // 初始化 NSLock 对象
// 1.创建 queue1,queue1 代表北京火车票售卖窗口
NSOperationQueue *queue1 = [[NSOperationQueue alloc] init];
queue1.maxConcurrentOperationCount = 1;
// 2.创建 queue2,queue2 代表上海火车票售卖窗口
NSOperationQueue *queue2 = [[NSOperationQueue alloc] init];
queue2.maxConcurrentOperationCount = 1;
// 3.创建卖票操作 op1
__weak typeof(self) weakSelf = self;
NSBlockOperation *op1 = [NSBlockOperation blockOperationWithBlock:^{
[weakSelf saleTicketSafe];
}];
// 4.创建卖票操作 op2
NSBlockOperation *op2 = [NSBlockOperation blockOperationWithBlock:^{
[weakSelf saleTicketSafe];
}];
// 5.添加操作，开始卖票
[queue1 addOperation:op1];
[queue2 addOperation:op2];
}
/**
* 售卖火车票(线程安全)
*/
- (void)saleTicketSafe {
while (1) {
// 加锁
[self.lock lock];
if (self.ticketSurplusCount > 0) {
//如果还有票，继续售卖
self.ticketSurplusCount--;
NSLog(@"%@", [NSString stringWithFormat:@"剩余票数:%d 窗口:%@", self.ticketSurplusCount, [NSThread currentThread]]);
[NSThread sleepForTimeInterval:0.2];
}
// 解锁
[self.lock unlock];
if (self.ticketSurplusCount <= 0) {
NSLog(@"所有火车票均已售完");
break;
}
}
}
```

![2969810-258a7cc0a4a02436.png](https://upload-images.jianshu.io/upload_images/2969810-258a7cc0a4a02436.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- ### 可以看出：在考虑了线程安全，使用 NSLock 加锁、解锁机制的情况下，得到的票数是正确的，没有出现混乱的情况。我们也就解决了多个线程同步的问题。

## 10. NSOperation、NSOperationQueue 常用属性和方法归纳

## 10.1 NSOperation 常用属性和方法

1. ### 取消操作方法
- ### `-(void)cancel;` 可取消操作，实质是标记 isCancelled 状态。
2. ### 判断操作状态方法
- ### `-(BOOL)isFinished;` 判断操作是否已经结束。
- ### `-(BOOL)isCancelled;` 判断操作是否已经标记为取消。
- ### `-(BOOL)isExecuting;` 判断操作是否正在在运行。
- ### `-(BOOL)isReady;` 判断操作是否处于准备就绪状态，这个值和操作的依赖关系相关。
3. ### 操作同步
- ### `-(void)waitUntilFinished;` 阻塞当前线程，直到该操作结束。可用于线程执行顺序的同步。
- ### `-(void)setCompletionBlock:(void (^)(void))block;` completionBlock 会在当前操作执行完毕时执行 completionBlock。
- ### `-(void)addDependency:(NSOperation *)op;` 添加依赖，使当前操作依赖于操作 op 的完成。
- ### `-(void)removeDependency:(NSOperation *)op;` 移除依赖，取消当前操作对操作 op 的依赖。
- ### `@property (readonly, copy) NSArray<NSOperation *> *dependencies;` 在当前操作开始执行之前完成执行的所有操作对象数组。

## 10.2 NSOperationQueue 常用属性和方法

1. ### 取消/暂停/恢复操作
- ### `-(void)cancelAllOperations;` 可以取消队列的所有操作。
- ### `-(BOOL)isSuspended;` 判断队列是否处于暂停状态。 YES 为暂停状态，NO 为恢复状态。
- ### `-(void)setSuspended:(BOOL)b;` 可设置操作的暂停和恢复，YES 代表暂停队列，NO 代表恢复队列。
2. ### 操作同步
- ### `-(void)waitUntilAllOperationsAreFinished;` 阻塞当前线程，直到队列中的操作全部执行完毕。
3. ### 添加/获取操作
- ### `-(void)addOperationWithBlock:(void (^)(void))block;` 向队列中添加一个 NSBlockOperation 类型操作对象。
- ### `-(void)addOperations:(NSArray *)ops waitUntilFinished:(BOOL)wait;` 向队列中添加操作数组，wait 标志是否阻塞当前线程直到所有操作结束
- ### `-(NSArray *)operations;` 当前在队列中的操作数组（某个操作执行结束后会自动从这个数组清除）。
- ### `-(NSUInteger)operationCount;` 当前队列中的操作数。
4. ### 获取队列
+ ### `-(id)currentQueue;` 获取当前队列，如果当前线程不是在 NSOperationQueue 上运行则返回 nil。
+ ### `-(id)mainQueue;` 获取主队列。

> ### 注意：
这里的暂停和取消（包括操作的取消和队列的取消）并不代表可以将当前的操作立即取消，而是当当前的操作执行完毕之后不再执行新的操作。
暂停和取消的区别就在于：暂停操作之后还可以恢复操作，继续向下执行；而取消操作之后，所有的操作就清空了，无法再接着执行剩下的操作。

