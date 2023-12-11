# 2、KVO实现原理

`KVO`的全称是`Key-value observing(键值观察)`，它提供了一种机制，允许对象在`其他对象的特定属性发生变化`时收到通知。下面我们先从`API`中去分析他的用法，然后分析他的底层原理。

## API分析

我们经常使用`KVO`方式如下：

> `// Wushuang.h

> @interface Wushuang : NSObject

> @property (nonatomic, strong) NSString *name;

> @property (nonatomic, strong) NSString *nickName;

> @end

> // VC

> @interface SecondViewController ()

> @property (nonatomic, strong) Wushuang *ws;

> @end

> [self.ws](http://self.ws) = [Wushuang alloc];

> [self.ws.name](http://self.ws.name) = @"Dianji";

> self.ws.nickName = @"Iron Man";

> // 添加观察

> [[self.ws](http://self.ws) addObserver:self forKeyPath:@"name" options:(NSKeyValueObservingOptionNew) context:NULL];

> // 观察回调

- > (void)observeValueForKeyPath:(NSString *)keyPath ofObject:(id)object change:(NSDictionary<NSKeyValueChangeKey,id> *)change context:(void *)context {

> NSLog(@" 🎈 change : %@ 🎈 ", change);

> }`

`addObserver`时，末尾的`context`参数往往被忽视了，那么这个参数有什么作用呢？下面我们去分析

### 1. context的作用

- 在官方文档 [Key-Value Observing Programming Guide](https://link.juejin.cn/?target=https%3A%2F%2Fdeveloper.apple.com%2Flibrary%2Farchive%2Fdocumentation%2FCocoa%2FConceptual%2FKeyValueObserving%2FArticles%2FKVOBasics.html%23%2F%2Fapple_ref%2Fdoc%2Fuid%2F20002252-SW4) 中，是这样描述`context`的：

![de9388b90eb24c1bb404e145248dd260~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/de9388b90eb24c1bb404e145248dd260~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp)

   - 大致意思是`context`可以用来判断观察属性的路径，这样更安全，可拓展行也更强。
   - 如果观察一个对象中的两个不同属性，我们可以用`keyPath`来判断，但观察多个不同对象的属性时，就需要先判断`object`再判断`keyPath`，这样就比较繁琐，此时`context`的优势就体现出来了，用法如下：

   > `static void *wushuangNameContext = &wushuangNameContext;

   > static void *teacherNameContext = &teacherNameContext;

   > [[self.ws](http://self.ws) addObserver:self forKeyPath:@"name" options:(NSKeyValueObservingOptionNew) context:wushuangNameContext];

   > [self.teacher addObserver:self forKeyPath:@"name" options:(NSKeyValueObservingOptionNew) context:teacherNameContext];

   - > (void)observeValueForKeyPath:(NSString *)keyPath ofObject:(id)object change:(NSDictionary<NSKeyValueChangeKey,id> *)change context:(void *)context {

   > if (context == wushuangNameContext) {

   > NSLog(@"1. keyPath == %@  object = %@", keyPath, object);

   > } else if (context == teacherNameContext) {

   > NSLog(@"2. keyPath == %@  object = %@", keyPath, object);

   > }

   > NSLog(@" 🎈 change : %@ 🎈 ", change);

   > }`

   `context`和属性一一对应，这样就更简洁也更安全，也提高了判断的效率

### 2. 移除观察者

- 我们知道在`iOS9`之后`KVO`就不需要手动移除，所以往往不需要手动处理移除。但在官方文档中有这样一句话介绍`KVO`移除：

![c4d949dda084445086eb2d9bd682700f~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c4d949dda084445086eb2d9bd682700f~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp)

   - 观察者在释放时不会自动移除自己，如果`继续发送消息`就会产生`空指针异常`。平时不移除观察者，是因为一般观察者在销毁时，被观察者也不存在了，所以不会出现异常
   - 也就是说如果被观察者是一个`单例`，在观察者释放后，再`重新添加该类的对象为观察者`时就会产生异常。如图所示：

![bc00fe51888142ff84e312e5cb52b104~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bc00fe51888142ff84e312e5cb52b104~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp)

   - 所以在使用时还是尽量手动移除观察者。

### 3. 手动触发KVO

- 我们常用的`KVO`是自动触发回调通知，也就是观察的属性值改变后，就会收到回调通知。在 [KVO Compliance](https://link.juejin.cn/?target=https%3A%2F%2Fdeveloper.apple.com%2Flibrary%2Farchive%2Fdocumentation%2FCocoa%2FConceptual%2FKeyValueObserving%2FArticles%2FKVOCompliance.html%23%2F%2Fapple_ref%2Fdoc%2Fuid%2F20002178-SW1) 中有介绍，触发回调是由`automaticallyNotifiesObserversForKey`方法确定，因为不去实现默认值是`YES`，也就是`自动触发回调通知`，如果设置成`NO`则就需要手动触发，需要在赋值前后设置相应的`willChangeValueForKey:`和`didChangeValueForKey:`方法。
- 下面进行案例验证案例验证：

> // Wushuang.m

+ > (BOOL)automaticallyNotifiesObserversForKey:(NSString *)key {

   if ([key isEqualToString:@"name"]) {
return YES;

   } else if ([key isEqualToString:@"nickName"]) {
return NO;

   }

   return YES;

> }

- > (void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event {

   > [self.ws.name](http://self.ws.name) = [NSString stringWithFormat:@"%@ +", [self.ws.name](http://self.ws.name)];

   > self.ws.nickName = [NSString stringWithFormat:@"%@ !", self.ws.nickName];

> }

- `vc`中观察了`Wushuang`的`name`和`nickName`属性，其中`name`设置成自动通知，`nickName`设置成手动通知，点击赋值结果如下：

![38fdb179056248a78303b41fbde40999~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/38fdb179056248a78303b41fbde40999~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp)

   结果只有`name`值的改变，回调中收到了通知。

- 在`Wushuang.m`中添加一些代码再尝试

> (void)setNickName:(NSString *)nickName {

   > [self willChangeValueForKey:@"nickName"];

   > _nickName = nickName;

   > [self didChangeValueForKey:@"nickName"];

> }

- 再次修改属性的值，就会收到`nickName`值改变的通知了：

![50b777a466ee4c4db65038e62029bbf5~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/50b777a466ee4c4db65038e62029bbf5~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp)

### 4. 依赖多个Keys

- 在很多情况下，一个属性的值取决于另一个对象中的一个或多个其他属性的值。这个时候就需要用`keyPathsForValuesAffectingValueForKey`来设置依赖关系：

> // Wushuang.h

> @interface Wushuang : NSObject

> @property (nonatomic, strong) NSString *food;

> @property (nonatomic, strong) NSString *meat;

> @property (nonatomic, strong) NSString *vegetable;

> @end

> // Wushuang.m

+ > (NSSet<NSString *> *)keyPathsForValuesAffectingValueForKey:(NSString *)key {

   NSSet *keyPaths = [super keyPathsForValuesAffectingValueForKey:key];

   if ([key isEqualToString:@"food"]) {
NSArray *affectingKeys = @[@"meat", @"vegetable"];
keyPaths = [keyPaths setByAddingObjectsFromArray:affectingKeys];

   }

   return keyPaths;

> }

- > (NSString *)food {

   > return [NSString stringWithFormat:@"%@ and %@", self.meat, self.vegetable];

> }

> // VC代码

> [[self.ws](http://self.ws) addObserver:self forKeyPath:@"food" options:NSKeyValueObservingOptionNew context:NULL];

- > (void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event {

   > self.ws.meat = @"beef";

   > self.ws.vegetable = @"radish";

> }

- 改变值输出结果如下：

![88b556b51c9a4905a81e6b3d974e385e~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/88b556b51c9a4905a81e6b3d974e385e~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp)

### 5. 集合类型

- 对于集合类型的属性的观察就有些不一样了，在`KVC`的 [Accessing Collection Properties](https://link.juejin.cn/?target=https%3A%2F%2Fdeveloper.apple.com%2Flibrary%2Farchive%2Fdocumentation%2FCocoa%2FConceptual%2FKeyValueCoding%2FAccessingCollectionProperties.html%23%2F%2Fapple_ref%2Fdoc%2Fuid%2F10000107i-CH4-SW1) 中有如下介绍：

![3a245cb344fc4b5698624ebff529fc63~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3a245cb344fc4b5698624ebff529fc63~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp)

   对于集合对象的访问需要使用代理的相关方法，这样会相应的修改底层属性，进而触发`KVO`的回调。

- 以数组为例子，具体实现如下：

> // Wushuang.m

> @property (nonatomic, strong) NSMutableArray *dataArray;

> // 添加监听

> [[self.ws](http://self.ws) addObserver:self forKeyPath:@"dataArray" options:NSKeyValueObservingOptionNew context:NULL];

- > (void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event {

> //    [self.ws.dataArray addObject:@"Tony Stark"]; // 没有作用

> [[[self.ws](http://self.ws) mutableArrayValueForKey:@"dataArray"] addObject:@"Steve Rogers"];

> }

- 在想数组里添加`object`时，以常规的方式访问数组是不会触发`KVO`通知的。需要使用`mutableArrayValueForKey`协议方法的方式访问到`dataArray`，再进行添加`object`。

## 原理分析

- 在`KVO`的文档最后对它的原理有一些介绍：

![4565545f76654d9ba544517c819d2398~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4565545f76654d9ba544517c819d2398~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp)

   - 我们知道对象`isa`指向类，但当注册观察者时，被观察对象的`isa`指向了中间类，我们可以打印被观察对象在观察前后的`isa`来对比差异

### 产生中间类

- 打印在观察前后`ws`对象的内容：

![09b57849533246309b7d1b61c4d9b1b2~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/09b57849533246309b7d1b61c4d9b1b2~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp)

   - 在观察前对象的`isa`指向`Wushuang`，但观察后`isa`就指向了`NSKVONotifying_Wushuang`，这个中间类名字上也带有`Wushuang`，那么它和原来的类有什么关系呢？中间类又有哪些内容呢？对象的值改变后为什么会影响原类呢？下面我们去挨个分析

### 原类和中间类的关系

- 我们可以使用`Runtime`获取所以注册类的信息，然后筛选出类`Wushuang`的子类：

> // 遍历类及其子类

- > (void)printClasses:(Class)class {

   > // 获取所以注册类数量

   > int count = objc_getClassList(NULL, 0);

   > // 创建一个数组， 其中包含给定对象

   > NSMutableArray *mutArray = [NSMutableArray arrayWithObject:class];

   > // 获取所有已注册的类

   > Class *classes = (Class *)malloc(sizeof(class) * count);

   > objc_getClassList(classes, count);

      > for (int i = 0; i < count; i++) {
if (class == class_getSuperclass(classes[i])) {
[mutArray addObject:classes[i]];
}

   > }

   > free(classes);

   > NSLog(@" classes = %@ ", mutArray);

> }

- 然后在添加观察前后查看与`Wushuang`类相匹配的信息：

![c01fe275ecf6475b82a55418267de03f~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c01fe275ecf6475b82a55418267de03f~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp)

   - 其中的`WSTeacher`是`Wushuang`的子类，所以可以得出结论：
      1. `中间类是被观察类的子类`
      2. `中间类没有其他子类`

### 中间类有什么

- 现在知道了中间类的名字，那么我们可以使用`Runtime`方法打印出它的方法列表

   > `- (void)printMethodesInClass: (Class)class {

   > unsigned int count = 0;

   > Method *methodList = class_copyMethodList(class, &count);

   > for (int i = 0; i < count; i++) {

   > Method method = methodList[i];

   > SEL sel = method_getName(method);

   > IMP imp = method_getImplementation(method);

   > NSLog(@" 🎉 : %@ --- %p ", NSStringFromSelector(sel), imp);

   > }

   > free(methodList);

   > }`

   分别打印原类和中间类，输出如下：

![fa26723241944eff8bb42d408d008355~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fa26723241944eff8bb42d408d008355~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp)

   - 原类里打印的是一些属性对应的`setter`和`getter`方法
   - 中间类打印的是观察属性的`setter`方法，`class`，`dealloc`以及`_isKVOA`这四个：
      - `_isKVOA`：是一个辨识码，来判断这个类是不是因为`KVO`产生的动态子类
      - `dealloc`：判断它是否进行释放
      - `class`：是类的信息
      - `setName`：是要变化的属性的`setter`方法
   - 实际上这四个方法都是重写父类的方法。

### `isa`什么时候指回来

- 中间类中重写了`dealloc`，可以在移除观察者前后来观察`isa`信息：

![8b14fa4d34464efe877b3e0dac32fd6d~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8b14fa4d34464efe877b3e0dac32fd6d~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp)

   - 从打印结果可以发现，在`dealloc`中移除观察者后，对象的`isa`就指回来了。

### 中间类什么时候销毁

- 那么当前页面销毁时，这个中间类也会销毁吗？下面来验证下：

![5c5a3e053abd41f1b0aac4af8673d1cb~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5c5a3e053abd41f1b0aac4af8673d1cb~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp)

   - 通过几次打印对比发现：
      1. `vc`对象出栈后，因监听而产生的`中间类`依然在注册类中，并不会因为页面销毁而销毁
      2. 再次监听相同类时，并不会产生新的中间类

### setter方法做了什么

- 我们知道中间类中有`setter`方法，那么它做了什么？监听的属性还是成员变量？可我们可以在类中定义一个成员变量来查看改变值后是有回调信息：

   > `// Wushuang.h

   > @interface Wushuang : NSObject {

   > @public

   > NSString *hobby;

   > }

   > @property (nonatomic, strong) NSString *name;

   > @end

   > // VC

   > [[self.ws](http://self.ws) addObserver:self forKeyPath:@"name" options:(NSKeyValueObservingOptionNew) context:NULL];

   > [[self.ws](http://self.ws) addObserver:self forKeyPath:@"hobby" options:NSKeyValueObservingOptionNew context:NULL];

   - > (void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event {

   > [self.ws.name](http://self.ws.name) = [NSString stringWithFormat:@"%@ +", [self.ws.name](http://self.ws.name)];

   > self.ws->hobby = @"Swimming";

   > }`

   - 打印结果如下：

![e69631072efe4cf088792b8f4a49f0e5~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e69631072efe4cf088792b8f4a49f0e5~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp)

   - 结果改变成员变量并不会触发监听通知，所以`KVO`在底层是通过`setter`方法来监听属性，要分析这个过程就需要借助`LLDB`的指令`watchpoint set variable +变量`，在添加观察前来设置内存断点，当变量的值发生改变就会触发断点，如下图：

![240245695f024f49a10e758c1caf3853~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/240245695f024f49a10e758c1caf3853~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp)

   堆栈信息反应了在属性值变化后，底层调用了下面几个函数：

      1. `Foundation _NSSetObjectValueAndNotify`
      - 结合堆栈和汇编可以得出该方法里会调用`willChangeValueForKey`和`didChangeValueForKey`方法：

![d8019545df2341e0ba8ef90b635e2099~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d8019545df2341e0ba8ef90b635e2099~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp)

   2. `Foundation -[NSObject(NSKeyValueObservingPrivate) _changeValueForKey:key:key:usingBlock:]`
   3. `Foundation -[NSObject(NSKeyValueObservingPrivate) _changeValueForKeys:count:maybeOldValuesDict:maybeNewValuesDict:usingBlock:]`

   这三个函数执行完后最终就会执行`-(void)observeValueForKeyPath:ofObject:change:context:`方法

## 流程总结

- 在添加观察时，`runtime`会产生一个中间类：
   1. 中间类`继承`于原类
   2. 中间类会`重写`被观察`key`的`setter`方法，
   3. 对象的`isa`从指向元类，变成`指向中间类`
- 当对属性赋值时,对象会根据`isa`找到`中间类`对应的`setter`方法，然后在`willChangeValueForKey`和`didChangeValueForKey`方法之间进行赋值，进而触发`-(void)observeValueForKeyPath:ofObject:change:context:`方法。
- 当在`dealloc`中移除通知后，`isa`会重新`指向原来的类`，相关实例变量的值不变。`dealloc`后中间类并不会释放，依然在注册类中。

观察前后，以及赋值流程图如下：

![101a30dee1444ed882170ca7b6686753~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/101a30dee1444ed882170ca7b6686753~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp)

**面试题**

讲了这些，我们来讨论面试题吧

**1、iOS用什么方式实现对一个对象的KVO？(KVO的本质是什么？)**

- 1、利用RuntimeAPI动态生成一个子类NSKVONotifying_XXX，并且让instance对象的isa指向这个全新的子类NSKVONotifying_XXX
- 2、当修改对象的属性时，会在子类NSKVONotifying_XXX调用Foundation的_NSSetXXXValueAndNotify函数
- 3、在_NSSetXXXValueAndNotify函数中依次调用 - 1、willChangeValueForKey - 2、父类原来的setter - 3、didChangeValueForKey，didChangeValueForKey:内部会触发监听器（Oberser）的监听方法( observeValueForKeyPath:ofObject:change:context:）

**2、如何手动触发KVO方法**

手动调用willChangeValueForKey和didChangeValueForKey方法

键值观察通知依赖于 NSObject 的两个方法: willChangeValueForKey: 和 didChangeValueForKey。在一个被观察属性发生改变之前， willChangeValueForKey: 一定会被调用，这就 会记录旧的值。而当改变发生后， didChangeValueForKey 会被调用，继而 observeValueForKey:ofObject:change:context: 也会被调用。如果可以手动实现这些调用，就可以实现“手动触发”了

有人可能会问只调用didChangeValueForKey方法可以触发KVO方法，其实是不能的，因为willChangeValueForKey: 记录旧的值，如果不记录旧的值，那就没有改变一说了

**3、直接修改成员变量会触发KVO吗**

不会触发KVO，因为KVO的本质就是监听对象有没有调用被监听属性对应的setter方法，直接修改成员变量，是在内存中修改的，不走set方法

**4、不移除KVO监听，会发生什么**

- 不移除会造成内存泄漏
- 但是多次重复移除会崩溃。系统为了实现KVO，为NSObject添加了一个名为NSKeyValueObserverRegistration的Category，KVO的add和remove的实现都在里面。在移除的时候，系统会判断当前KVO的key是否已经被移除，如果已经被移除，则主动抛出一个NSException的异常

