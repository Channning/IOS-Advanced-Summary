# 1、KVC实现原理

## 前言

在日常的开发中，我们通常会用到`KVC`进行赋值，或者访问一些私有`属性`。那么`KVC`是什么，它的原理又是怎样的？接下来一起去探究分析下。

## KVC简介

- `Key-value coding(键值编码)`是一种由`NSKeyValueCoding非正式协议`启用的机制，对象采用它来提供对其属性的间接访问。**当一个对象符合键值编码时，它的属性可以通过一个简洁、统一的消息传递接口通过字符串参数来寻址**。这种间接访问机制补充了实例变量及其关联的访问方法所提供的直接访问。

![a466e4309f304c7d9819177409431ade~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a466e4309f304c7d9819177409431ade~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp)

- 查看`setValue:forKey:`源码，最终到`Foundatin`框架的`NSKeyValueCoding`文件：

但`Foundation`是不开源的，所以只能查看`KVC`的官方文档：[Key-Value Coding Programming Guide](https://link.juejin.cn/?target=https%3A%2F%2Fdeveloper.apple.com%2Flibrary%2Farchive%2Fdocumentation%2FCocoa%2FConceptual%2FKeyValueCoding%2Findex.html%23%2F%2Fapple_ref%2Fdoc%2Fuid%2F10000107-SW1)

## KVC的赋值和取值

![25e1189687ee461bb2b1e8b3f6214789~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/25e1189687ee461bb2b1e8b3f6214789~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp)

先确定要看的位置，根据文档找到了`Accessor Search Patterns`：

- 先来看`setter`方法

### Basic Setter

- 文档上说调用`setValue:forKey:`对属性进行赋值时，
   1. 会按顺序去找`set<Key>`或者`_set<Key>`的方法，如果找到，就对它赋值，
   2. 如果没找到，如果类方法`accessinstancevariablesdirect`返回值为`YES`，就会按顺序去找实例变量`_<key>`, `_is<Key>`, `<key>`, 或者 `is<Key>`，找到后并对它赋值
   3. 如果没有找到就会走`setValue:forUndefinedKey:`方法

#### 代码验证

   1. 先定义一个`LGPerson`类，并设置实例变量，实现`setName`和`_setName`方法，最后在`ViewController`中调用`setValue:forKey:`方法：

> `// .h

> @interface LGPerson : NSObject {

> @public

> NSString *_isName;

> NSString *name;

> NSString *isName;

> NSString *_name;

> }

> // .m

- > (void)setName:(NSString *)name{

> NSLog(@"%s - %@",**func**,name);

> }

- > (void)_setName:(NSString *)name{

> NSLog(@"%s - %@",**func**,name);

> }

> // ViewController.m

> LGPerson *person = [[LGPerson alloc] init];

> [person setValue:@"wushuang" forKey:@"name"];`

打印结果如下：

![9779b258ab5a4ae3976903a6f3ffd6da~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9779b258ab5a4ae3976903a6f3ffd6da~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp)

说明调用`setValue:forKey:`方法会先走`setName`方法，将`setName`方法注释掉，留下`_setName`，再运行：

![8dfea720df2c42e6aec5b5ffcc89a089~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8dfea720df2c42e6aec5b5ffcc89a089~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp)

这次走了`_setName`的方法，也就是说调用`setValue:forKey:`后，会先找`setName`方法，没找到再找`_setName`。

- 当`setName`和`_setName`都不实现时会去找实例变量`_name`, `_isName`, `name`, 或者 `isName`

> 猜想：此时多了`isName`和`_isName`，那么`setIsName`和`_setIsName`方法会不会走呢？接下来去验证下：

- 先注释前面的两个`set`方法，然后添加`setIsName`和`_setIsName`方法：

> `- (void)setIsName:(NSString *)name{

> NSLog(@"%s - %@",**func**,name);

> }

- > (void)_setIsName:(NSString *)name{

> NSLog(@"%s - %@",**func**,name);

> }`

- 运行结果发现走了`setIsName`方法，并没有走`_setIsName`，也就得到了新流程：

> 新流程：在调用`setValue:forKey:`后查找顺序为  `setName` -> `_setName` -> `setIsName`

- 当`set`相关方法没找到就会去找相关的实例变量，先注释掉`set`相关方法，然后先实现类方法`accessInstanceVariablesDirectly`，返回值为`YES`，再打印实例变量的值：

> `// LGPerson.m

+ > (BOOL)accessInstanceVariablesDirectly{

> return YES;

> }

> // ViewController.m

> NSLog(@"%@-%@-%@-%@",person->_name,person->_isName,person->name,person->isName);`

运行结果：

![8d6f399597c249f88eb0cb937937b711~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8d6f399597c249f88eb0cb937937b711~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp)

- 会先查找`_name`，注释掉最先打印的实例变量然后打印，最终得到实例变量的打印顺序：`_name` -> `_isName` -> `name` -> `isName`
- 注释掉4个实例变量，然后实现方法`setValue:forUndefinedKey`在打印：

> `- (void)setValue:(id)value forUndefinedKey:(NSString *)key {

> NSLog(@"%s–%@ --- %@", **func**, value, key);

> }`

结果如下：

![4fdb4ba3b4ad4d9782eeb5e395db9b01~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4fdb4ba3b4ad4d9782eeb5e395db9b01~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp)

- 证明了当`set方法`和`实例变量`都没有时，就会走`setValue:forUndefinedKey`方法。

#### 流程图

![a4b174f79e0645b2938f2d492797efa0~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a4b174f79e0645b2938f2d492797efa0~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp)

- `Basic Setter`流程如下：

### Basic Getter

再来看看取值`Search Pattern for the Basic Getter`：

![7f13adb61fac48238cc496315fc1f030~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7f13adb61fac48238cc496315fc1f030~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp)

- 也就是实现`valueForKey:`方法后，系统会走以下几个步骤(暂不考虑集合类型)：
   1. 在实例方法中搜索第一个名称为`getName`、`name`、`isName`或`_name`的访问器方法。如果找到了，就调用它
   2. 如果没找到(除去集合类型)，先检验类方法`accessInstanceVariablesDirectly`实现，并返回`YES`，然后依次搜索实例变量`_name`， `_isName`， `name`，或`isName`，如果找到就直接获取实例变量的值并返回
   3. 如果没有找到就会走`valueForUndefinedKey:`方法

#### 代码验证

先将`set`相关代码都注释，然后实现第一个步骤：

> `// LGPerson.m

- > (NSString *)getName{

> return NSStringFromSelector(_cmd);

> }

- > (NSString *)name{

> return NSStringFromSelector(_cmd);

> }

- > (NSString *)isName{

> return NSStringFromSelector(_cmd);

> }

- > (NSString *)_name{

> return NSStringFromSelector(_cmd);

> }

> // ViewController.m

> NSLog(@"取值: %@",[person valueForKey:@"name"]);`

- 然后打印出一个后，注释掉打印的再运行，得到第一步实例方法的取值顺序：`getName` -> `name` -> `isName` -> `_name`

然后注释掉实例方法，再将实例变量赋值：

> `// ViewController.m

> person->_name = @"_name";

> person->_isName = @"_isName";

> person->name = @"name";

> person->isName = @"isName";

> NSLog(@"取值: %@",[person valueForKey:@"name"]);`

- 然后打印出来一个，`注释`掉一个`实例变量的赋值`和`实例变量`，再打印，得到第二步实例变量的取值顺序：`_name`-> `_isName` -> `name` -> `isName`
- 再注释掉实例变量和实例变量的赋值，然后在`LGPerson.m`实现`valueForUndefinedKey:`方法：

> `- (id)valueForUndefinedKey:(NSString *)key {

> NSLog(@"%s --- %@", **func**, key);

> return NSStringFromSelector(_cmd);

> }

> // ViewController.m

> NSLog(@"取值: %@",[person valueForKey:@"name"]);`

- 运行结果如下：

![96672c3c0b9c40e3998f4caa4e698428~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/96672c3c0b9c40e3998f4caa4e698428~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp)

至此，三步的取值流程都得以验证

#### 流程图

![bc02c8b85a0d46858be2a476e49e6ff5~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bc02c8b85a0d46858be2a476e49e6ff5~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp)

- `Basic Setter`查找流程如下：

## 自定义KVC

- 了解了`KVC`的`setter`和`getter`特性后，那我们自己能定义`KVC`吗？答案是肯定的。
- 根据`NSKeyValueCoding`中的`setter`和`getter`方法，在定义的`NSOjbect`分类中定义新的`setter`和`getter`方法，然后定义`WSPerson`，里面有`set`实例方法和实例变量：

> `@interface NSObject (WSKVC)

- > (void)ws_setValue:(nullable id)value forKey:(NSString *)key;
- > (nullable id)ws_valueForKey:(NSString *)key;

> @end

> // WSPerson.h

> @interface WSPerson : NSObject {

> @public

> NSString *_name;

> NSString *_isName;

> NSString *name;

> NSString *isName;

> }

> @end

> // WSPerson.m

> @implementation WSPerson

+ > (BOOL)accessInstanceVariablesDirectly {

> return true;

> }

- > (void)setName:(NSString *)name {

> NSLog(@"%s --- %@", **func** ,name);

> }

- > (void)_setName:(NSString *)name {

> NSLog(@"%s --- %@", **func** ,name);

> }

- > (void)setIsName:(NSString *)name {

> NSLog(@"%s --- %@", **func** ,name);

> }

> @end`

- 接下来根据前面分析的取值和赋值流程对自定义方法进行相关处理

### 赋值`ws_setValue:forKey:`

代码如下：

> `- (void)ws_setValue:(nullable id)value forKey:(NSString *)key {

> if (key  ::nil || key.length::  0) {

> return;

> }

> NSString *Key = key.capitalizedString; //首字母大写

> // 拼接相关的方法名

> NSString *setKey = [NSString stringWithFormat:@"set%@:",Key];

> NSString *_setKey = [NSString stringWithFormat:@"_set%@:",Key];

> NSString *setIsKey = [NSString stringWithFormat:@"setIs%@:",Key];

> // 按顺序判断是否实现三个实例方法

> if ([self ws_performSelectorWithMethodName:setKey value:value]) {

> NSLog(@"___ %@ ___",setKey);

> return;

> } else if ([self ws_performSelectorWithMethodName:_setKey value:value]) {

> NSLog(@"___ %@ ___",_setKey);

> return;

> } else if ([self ws_performSelectorWithMethodName:setIsKey value:value]) {

> NSLog(@"___ %@ ___",setIsKey);

> return;

> }

> if (![self.class accessInstanceVariablesDirectly] ) {

> @throw [NSException exceptionWithName:@"WS_UnknownKeyException" reason:[NSString stringWithFormat:@"**[%@ valueForUndefinedKey:]: this class is not key value coding-compliant for the key name.**",self] userInfo:nil];

> }

> // 获取所有实例变量的名字

> NSMutableArray *mArray = [self getIvarListName];

> // 拼接出需要的实例变量名字

> NSString **key = [NSString stringWithFormat:@"*%@",key];

> NSString *_isKey = [NSString stringWithFormat:@"_is%@",Key];

> NSString *isKey = [NSString stringWithFormat:@"is%@",Key];

> // 依次判断拼接的实例变量名字是否在实例变量数组中，存在则获取实例变量并赋值

> if ([mArray containsObject:_key]) {

> Ivar ivar = class_getInstanceVariable([self class], _key.UTF8String);

> object_setIvar(self , ivar, value);

> return;

> } else if ([mArray containsObject:_isKey]) {

> Ivar ivar = class_getInstanceVariable([self class], _isKey.UTF8String);

> object_setIvar(self , ivar, value);

> return;

> } else if ([mArray containsObject:key]) {

> Ivar ivar = class_getInstanceVariable([self class], key.UTF8String);

> object_setIvar(self , ivar, value);

> return;

> } else if ([mArray containsObject:isKey]) {

> Ivar ivar = class_getInstanceVariable([self class], isKey.UTF8String);

> object_setIvar(self , ivar, value);

> return;

> }

> // 如果找不到相关实例变量，就抛出异常

> @throw [NSException exceptionWithName:@"WS_UnknownKeyException" reason:[NSString stringWithFormat:@"**[%@ %@]: this class is not key value coding-compliant for the key name.**",self, NSStringFromSelector(_cmd)] userInfo:nil];

> }

- > (BOOL)ws_performSelectorWithMethodName:(NSString *)methodName value:(id)value{

> // 判断方法是否能响应，

> if ([self respondsToSelector:NSSelectorFromString(methodName)]) {

> \#pragma clang diagnostic push

> \#pragma clang diagnostic ignored "-Warc-performSelector-leaks"

> // 能响应则调用

> [self performSelector:NSSelectorFromString(methodName) withObject:value];

> \#pragma clang diagnostic pop

> return YES;

> }

> return NO;

> }

> // 获取实例变量的名字

- > (NSMutableArray *)getIvarListName{

> NSMutableArray *mArray = [NSMutableArray arrayWithCapacity:1];

> unsigned int count = 0;

> Ivar *ivars = class_copyIvarList([self class], &count);

> for (int i = 0; i<count; i++) {

> Ivar ivar = ivars[i];

> const char *ivarNameChar = ivar_getName(ivar);

> NSString *ivarName = [NSString stringWithUTF8String:ivarNameChar];

> NSLog(@"ivarName == %@",ivarName);

> [mArray addObject:ivarName];

> }

> free(ivars);

> return mArray;

> }`

   1. 先判断`key`是否存在
   2. 再按顺序判断相关的`set`方法：`setName:` -> `_setName` -> `setIsName`
   - 先拼接出三个方法的名字，再依次调用`respondsToSelector`方法判断能不能响应，如果能响应就用`performSelector`方法执行调用
   3. 如果类方法`accessInstanceVariablesDirectly`返回值为`NO`，则抛出异常
   4. 如果返回`YES`，则先获取类的实例变量名字数组，然后根据相关实例方法的名字`按顺序`判断是否在数组中，如果存在就获取实例变量`ivar`，然后对它赋值
   5. 当实例方法也找不到值时，再抛出异常

### 取值`ws_valueForKey:`

代码如下：

> `- (nullable id)ws_valueForKey:(NSString *)key {

> if (key  ::nil  || key.length::  0) {

> return nil;

> }

> // 首字母大写

> NSString *Key = key.capitalizedString;

> // 拼接 方法名字，部分和实例变量名字相同

> NSString *getKey = [NSString stringWithFormat:@"get%@",Key];

> NSString *isKey = [NSString stringWithFormat:@"is%@",Key];

> NSString **key = [NSString stringWithFormat:@"*%@",key];

> NSString *countOfKey = [NSString stringWithFormat:@"countOf%@",Key];

> NSString *objectInKeyAtIndex = [NSString stringWithFormat:@"objectIn%@AtIndex:",Key];

> \#pragma clang diagnostic push

> \#pragma clang diagnostic ignored "-Warc-performSelector-leaks"

> if ([self respondsToSelector:NSSelectorFromString(getKey)]) {

> return [self performSelector:NSSelectorFromString(getKey)];

> } else if ([self respondsToSelector:NSSelectorFromString(key)]){

> return [self performSelector:NSSelectorFromString(key)];

> } else if ([self respondsToSelector:NSSelectorFromString(isKey)]) {

> return [self performSelector:NSSelectorFromString(isKey)];

> } else if ([self respondsToSelector:NSSelectorFromString(_key)]) {

> return [self performSelector:NSSelectorFromString(_key)];

> }

> // 集合类型处理

> else if ([self respondsToSelector:NSSelectorFromString(countOfKey)]){

> if ([self respondsToSelector:NSSelectorFromString(objectInKeyAtIndex)]) {

> int num = (int)[self performSelector:NSSelectorFromString(countOfKey)];

> NSMutableArray *mArray = [NSMutableArray arrayWithCapacity:1];

> for (int i = 0; i<num-1; i++) {

> num = (int)[self performSelector:NSSelectorFromString(countOfKey)];

> }

> for (int j = 0; j<num; j++) {

> id objc = [self performSelector:NSSelectorFromString(objectInKeyAtIndex) withObject:@(num)];

> [mArray addObject:objc];

> }

> return mArray;

> }

> }

> \#pragma clang diagnostic pop

> // 判断是否能够直接赋值实例变量

> if (![self.class accessInstanceVariablesDirectly] ) {

> @throw [NSException exceptionWithName:@"WS_UnknownKeyException" reason:[NSString stringWithFormat:@"**[%@ valueForUndefinedKey:]: this class is not key value coding-compliant for the key name.**",self] userInfo:nil];

> }

> // 获取实例变量名字

> NSMutableArray *mArray = [self getIvarListName];

> NSString *_isKey = [NSString stringWithFormat:@"_is%@",Key];

> // 依次判断拼接的实例变量名字是否在实例变量数组中，存在则获取实例变量并取值

> if ([mArray containsObject:_key]) {

> Ivar ivar = class_getInstanceVariable([self class], _key.UTF8String);

> return object_getIvar(self, ivar);;

> } else if ([mArray containsObject:_isKey]) {

> Ivar ivar = class_getInstanceVariable([self class], _isKey.UTF8String);

> return object_getIvar(self, ivar);;

> } else if ([mArray containsObject:key]) {

> Ivar ivar = class_getInstanceVariable([self class], key.UTF8String);

> return object_getIvar(self, ivar);;

> } else if ([mArray containsObject:isKey]) {

> Ivar ivar = class_getInstanceVariable([self class], isKey.UTF8String);

> return object_getIvar(self, ivar);;

> }

> @throw [NSException exceptionWithName:@"WS_UnknownKeyException" reason:[NSString stringWithFormat:@"**[%@ %@]: this class is not key value coding-compliant for the key name.**",self, NSStringFromSelector(_cmd)] userInfo:nil];

> return @"";

> }`

   1. 先判断`key`是否存在，不存在或者为空时，返回`nil`
   2. 再按顺序判断相关的`get`实例方法：`getName:` -> `name` -> `isName` -> `_name`
   - 先拼接出这几个方法的名字，再依次调用`respondsToSelector`方法判断能不能响应，如果能响应就用`performSelector`方法执行调用
   3. 如果类方法`accessInstanceVariablesDirectly`返回值为`NO`，则抛出异常
   4. 如果返回`YES`，则先获取类的实例变量名字数组，然后根据相关实例方法的名字`按顺序`判断是否在数组中，如果存在就获取实例变量`ivar`，然后取值
   5. 当实例方法也找不到值时，再抛出异常并返回空

