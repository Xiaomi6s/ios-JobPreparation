# 基础篇
## 1.深浅拷贝
 浅拷贝：浅拷贝并不拷贝对象本身，只是对指向对象的指针进行拷贝（指针的拷贝）
 深拷贝：直接拷贝对象到内存中一块区域，然后把新对象的指针指向这块内存（内容的拷贝）
a.系统非容器类对象（例如string）的copy和mutablecopy
[string copy] 浅拷贝
[string mutableCopy] 深拷贝
[mutableString copy] 深拷贝
[mutableString mutableCopy] 深拷贝
mutableString copy和mutableCopy 虽然都是深拷贝但是copy相对于mutableCopy在复制对象的时候返回的是一个不可变对象，因此当调用方法改变对象的时候会崩溃
b.系统容器类对象（如 array）的copy和mutableCopy
[array copy] 浅拷贝
[array mutableCopy] 单层深拷贝（数组里面的数据依然是指针的复制）
[mutableArray copy] 单层深拷贝
[mutableArray mutableCopy] 单层深拷贝 
## 2.kvc和kvo
 kvc键-值编码
 kvo键-值监听
 kvo的实现原理：使用了isa混写（isa-swizzling）来实现kvo，当观察对象A的时候， kvo机制会动态创建一个A的新子类：NSKVONotifying_A，该类继承自对象A的本类，且KVO会为NSKVONotifying_A，重写父类的被观察属性的setter方法（插入willChangeValueForKey，didChangeValueForKey两个方法），setter方法会负责在调用原setter方法之前或之后，通知所有观察对象属性这的状态改变，在这个过程中被观察对象A的isa指针从原来的A类被kvo修改后指向了A类的子类NSKVONotifying_A。
 ## 3. 修饰词 strong weak assin