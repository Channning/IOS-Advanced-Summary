# (五)   关联对象

## 前言

我们都很清楚本身分类使用@Property添加属性只会有setter和getter方法的声明没有对应的实现，也不会有对应的带下划线的成员变量存在，目前我们添加属性都是通过关联对象的方式即运用了运行时相关的API来实现的，下面我们来分析下它的实现原理。

## 关联对象添加

我们常写的分类添加关联对象的代码如下：

```other
- (void)setCate_name:(NSString *)cate_name{
    /**
     1: 对象
     2: 标识符
     3: value
     4: 策略
     */
    objc_setAssociatedObject(self, "cate_name", cate_name, OBJC_ASSOCIATION_COPY_NONATOMIC);

}

- (NSString *)cate_name{
    return  objc_getAssociatedObject(self, "cate_name");
}
```

我们进入`objc_setAssociatedObject()`方法内部，看下它的实现：

![f3f4b8e1150b41f28c373b46dd288b2b~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f3f4b8e1150b41f28c373b46dd288b2b~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp)

进入`_object_set_associative_reference`方法：

```other
void _object_set_associative_reference(id object, const void *key, id value, uintptr_t policy)

{

    // This code used to work when nil was passed for object and key. Some code
    // probably relies on that to not crash. Check and handle it explicitly.
    // rdar://problem/44094390

    if (!object && !value) return;

    if (object->getIsa()->forbidsAssociatedObjects())

        _objc_fatal("objc_setAssociatedObject called on instance (%p) of class %s which does not allow associated objects", object, object_getClassName(object));
	//对object指针进行数据封装以方便作为统一的格式处理
    DisguisedPtr<objc_object> disguised{(objc_object *)object};
	//对存储策略和value进行相关赋值存储在ObjcAssociation对象中
    ObjcAssociation association{policy, value};

    // retain the new value (if any) outside the lock.

    association.acquireValue();

    bool isFirstAssociation = false;

    {
		//构造函数 AssociationsManager()   { AssociationsManagerLock.lock(); }
		//析构函数 ~AssociationsManager()  { AssociationsManagerLock.unlock();}
		//下面这行代码调用AssociationsManager构造函数
        AssociationsManager manager;
		//这里AssociationsHashMap是个全局静态变量，相当于单例，获取全局唯一的哈希表
        AssociationsHashMap &associations(manager.get());
		//value存在
        if (value) {

            auto refs_result = associations.try_emplace(disguised, ObjectAssociationMap{});

            if (refs_result.second) {

                /* it's the first association we make */
                isFirstAssociation = true;

            }

            /* establish or replace the association */

            auto &refs = refs_result.first->second;
            auto result = refs.try_emplace(key, std::move(association));

            if (!result.second) {
                association.swap(result.first->second);
            }

        } else {
			//若value不存在，移除关联对象
            auto refs_it = associations.find(disguised);
            if (refs_it != associations.end()) {

                auto &refs = refs_it->second;
                auto it = refs.find(key);

                if (it != refs.end()) {
                    association.swap(it->second);
                    refs.erase(it);

                    if (refs.size() == 0) {
                        associations.erase(refs_it);
                    }
                }
            }
        }
    }

    // Call setHasAssociatedObjects outside the lock, since this
    // will call the object's _noteAssociatedObjects method if it
    // has one, and this may trigger +initialize which might do
    // arbitrary stuff, including setting more associated objects.

    if (isFirstAssociation)

        object->setHasAssociatedObjects();

    // release the old value (outside of the lock).

    association.releaseHeldValue();
}
```

简单的地方我们直接就上面代码里注释说明，这里有涉及哈希表(`AssociationsHashMap`)存储查询的地方`associations-(AssociationsHashMap).try_emplace(disguised, ObjectAssociationMap{})`，我们看下：

![aeda4216f6224615a434776fed8cbdf1~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/aeda4216f6224615a434776fed8cbdf1~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp)

这里我们首先看到`BucketT *TheBucket`创建了一个空的桶子，接着调用`LookupBucketFor(Key, TheBucket)`判断根据`Key`查询对应的`BucketT`是否存在，若存在，调用`make_pair`返回当前`BucketT`并返回`false`，表示未添加新的`BucketT`，这里的`Key`是外面传入的`object指针`包装的`DisguisedPtr<objc_object> disguised`，`Value`是外部传入的`ObjectAssociationMap`，我们看下`LookupBucketFor`方法：

![151f01cbeff64cd8a972801127993073~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/151f01cbeff64cd8a972801127993073~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp)

这里`LookupBucketFor`调用了另一个方法`LookupBucketFor(const LookupKeyT &Val, const BucketT *&FoundBucket)`：

![22641a599dac4fbdb4fbf626bc93d104~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/22641a599dac4fbdb4fbf626bc93d104~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp)

这里其实就是哈希表的一个查询过程，查询返回结果后，就接着根据返回的结果来继续执行后续代码，我们第一次进来调用`LookupBucketFor`的时候，`bucket`肯定是空的，所以接着执行`InsertIntoBucket(TheBucket, Key, std::forward<Ts>(Args)...)`，把当前创建的空桶子存入`AssociationsHashMap`中，此时注意`BucketT`是空的，我们调试下：

![05056d0bb8ea4d1696abb2e75837aa3b~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/05056d0bb8ea4d1696abb2e75837aa3b~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp)

我们看下`InsertIntoBucket`实现：

![66f2d7b75eda400288102082404038eb~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/66f2d7b75eda400288102082404038eb~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp)

我们看到这里一个关键函数`InsertIntoBucketImpl(Key, Key, TheBucket)`：

![57e769ace21041cda1a52eaf80391e88~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/57e769ace21041cda1a52eaf80391e88~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp)

这里插入完成后，我们在`InsertIntoBucket`返回的`TheBucket`就看到它就赋值完成：

![6d52e4c409fd4e1a8ef7f4a8efe703bc~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6d52e4c409fd4e1a8ef7f4a8efe703bc~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp)

此时我们第一次调用`associations.try_emplace`调用结束，我们看`refs_result.second`为`true`，此时`isFirstAssociation`也为`true`，此时我们的`policy`和`value`还没有存进去，他们目前存在`assocition`对象中，我们看下`refs`这里，此时都还是空：

![7579da628c854e15af58cf78615fde56~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7579da628c854e15af58cf78615fde56~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp)

下一步`refs.try_emplace(key,std::move(association))`存入`association`，这里刚好也解释了第一次`associations-(AssociationsHashMap).try_emplace(disguised, ObjectAssociationMap{})`中`bucket`最终插入后并不包含相关的`Key`和`Value`，因为此时`association`并没有传入，我们看`refs.try_emplace`这里，此时传入的`Key`就是我们外部传入的`Key`（这里是`const void *`类型值为`cate_name`），而`Key`存`bucket`的`first`，`value`存`second`：

![4427bad0dae542fd9d7413ac7e8eb0fc~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4427bad0dae542fd9d7413ac7e8eb0fc~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp)

我们看`refs.try_emplace`调用结束后，发现`bucket`中`Key`和`Value`都已存入：

![5578489c659a45f8a0899fec31cf4864~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5578489c659a45f8a0899fec31cf4864~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp)

到这里，我们基本已经清楚了它的存储结构，大致是这样的，我们看张图：

![6a2e8a7a53df4e92ba485417d428967f~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6a2e8a7a53df4e92ba485417d428967f~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp)

## **::关联对象设值流程::**

1. 创建一个 AssociationsManager 管理类
2. 获取唯一的全局静态哈希Map
3. 判断是否插入的关联值是否存在:

   3.1 存在走第4步

   3.2 不存在：关联对象插入空流程

4. 创建一个空的 ObjectAssociationMap 去取查询的键值对
5. 如果发现没有这个key就插入一个空的BucketT进去，然后返回
6. 标记对象存在关联对象
7. 用当前修饰策略(policy)和值(value)组成了一个ObjcAssociation替换原来BucketT中的空
8. 标记一下 ObjectAssociationMap的第一次为false

### 关联对象取值分析

我们上面分析的是`objc_setAssociatedObject`，接下来看下`objc_getAssociatedObject(self, "cate_name")`，我们进入底层源码看下它的具体实现：

```other
_object_get_associative_reference(id object, const void *key)
{
    ObjcAssociation association{};

    {
		//创建一个 AssociationsManager管理类
        AssociationsManager manager;
		//获取唯一的全局静态AssociationsHashMap
        AssociationsHashMap &associations(manager.get());
		//根据 DisguisedPtr找到AssociationsHashMap中的iterator迭代查询器
        AssociationsHashMap::iterator i = associations.find((objc_object *)object);
		//如果这个迭代查询器不是最后一个获取 :ObjectAssociationMap (这里有策略和value)
        if (i != associations.end()) {

            ObjectAssociationMap &refs = i->second;
            ObjectAssociationMap::iterator j = refs.find(key);
			//找到ObjectAssociationMap的迭代查询器获取一个经过属性修饰符修饰的value
            if (j != refs.end()) {

                association = j->second;
                association.retainReturnedValue();
            }
        }
    }
	//返回value
    return association.autoreleaseReturnedValue();

}
```

### **::关联对象取值流程::**

- 创建一个 AssociationsManager 管理类
- 获取唯一的全局静态哈希表AssociationsHashMap
- 根据DisguisedPtr找到AssociationsHashMap中的 iterator迭代查询器
- 如果这个迭代查询器不是最后一个 获取:ObjectAssociationMap(这里有policy和value)
- 找到ObjectAssociationMap的迭代查询器获取一个经过属性修饰符修饰的value
- 返回value

### 关联对象移除

关联对象的移除在`dealloc`方法中，我们看下它调用流程：`dealloc——>_objc_rootDealloc——>rootDealloc`：

![81badb2083cb47899b6823ed572aa777~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/81badb2083cb47899b6823ed572aa777~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp)

我们接着看`object_dispose()——>objc_destructInstance`：

![6937e3f079f14f87aa2245ffbda72c7b~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6937e3f079f14f87aa2245ffbda72c7b~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp)

我们进入`_object_remove_assocations`，看下它的实现：

![597197faa71f40fea4120c26ece031ff~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/597197faa71f40fea4120c26ece031ff~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp)

移除目标对象（objct）的所有关联对象。

官方在这做了提示：这个方法的主要目的是为了让对象更容易返回到原始状态，**::调用此方法会将绑定在这个目标对象上的所有关联对象都清除::**。如果别的开发也为这个object设置了关联对象，或者你在别的模块中也为其设置了不同的关联对象，调用此方法后会被一并删除。通常清除关联对象，请通过objc_setAssociatedObject将关联对象设置为nil的方式来清除关联对象。

#### 关联对象的存储结构

看到这里可以总结一下关联关系的存储结构了。

AssociationsHashMap是管理目标对象（object）与ObjectAssociationMap的关系

ObjectAssociationMap是管理Key与ObjectAssociation（关联对象）的关系。

![1697b8a03bab8346~tplv-t2oaga2asx-jj-mark:3024:0:0:0:q75.png](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/3/14/1697b8a03bab8346~tplv-t2oaga2asx-jj-mark:3024:0:0:0:q75.png)

## 关联对象实现原理

实现关联对象技术的核心对象有

1. AssociationsManager
2. AssociationsHashMap
3. ObjectAssociationMap
4. ObjcAssociation

   其中Map同我们平时使用的字典类似。通过key-value一一对应存值。

最后我们通过一张图可以很清晰的理清楚其中的关系

![1635a628a228e349~tplv-t2oaga2asx-jj-mark:3024:0:0:0:q75.png](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2018/5/14/1635a628a228e349~tplv-t2oaga2asx-jj-mark:3024:0:0:0:q75.png)

**通过上图我们可以总结为：一个实例对象就对应一个ObjectAssociationMap，而ObjectAssociationMap中存储着多个此实例对象的关联对象的key以及ObjcAssociation，为ObjcAssociation中存储着关联对象的value和policy策略。**

**由此我们可以知道关联对象并不是放在了原来的对象里面，而是自己维护了一个全局的map用来存放每一个对象及其对应关联属性表格。**

### 总结1：

**关联对象并不是存储在被关联对象本身内存中，而是存储在全局的统一的一个AssociationsManager中，如果设置关联对象为nil，就相当于是移除关联对象。**

此时我们我们在回过头来看objc_AssociationPolicy policy 参数: 属性以什么形式保存的策略。

```other
typedef OBJC_ENUM(uintptr_t, objc_AssociationPolicy) {
	OBJC_ASSOCIATION_ASSIGN = 0,  // 指定一个弱引用相关联的对象
	OBJC_ASSOCIATION_RETAIN_NONATOMIC = 1, // 指定相关对象的强引用，非原子性
	OBJC_ASSOCIATION_COPY_NONATOMIC = 3,  // 指定相关的对象被复制，非原子性
	OBJC_ASSOCIATION_RETAIN = 01401,  // 指定相关对象的强引用，原子性
	OBJC_ASSOCIATION_COPY = 01403     // 指定相关的对象被复制，原子性   
};
```

**::我们会发现其中只有RETAIN和COPY而为什么没有weak呢？::** 总过上面对源码的分析我们知道，object经过DISGUISE函数被转化为了disguised_ptr_t类型的**disguised_object**。

> `disguised_ptr_t disguised_object = DISGUISE(object);`

而同时我们知道，weak修饰的属性，当没有拥有对象之后就会被销毁，并且指针置位nil，那么在对象销毁之后，虽然在map中依然存在值object对应的AssociationsHashMap，但是因为object地址已经被置位nil，会造成坏地址访问而无法根据object对象的地址转化为disguised_object了。

## 总结2：

- 可以看出Runtime对于关联对象的管理都是线程安全的，增删查改都是加锁的。
- 在App运行期间，由AssociationsHashMap来管理所有被添加到对象中的关联对象。
- 当NSObject dealloc时，会释放持有的关联对象。

