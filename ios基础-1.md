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
  strong：指持有该对象，引用计数加1，引用计数为0销毁，可以通过将变量强制赋值nil来进行销毁

  weak：指向但是不持有该对象，引用计数不加1，当被指向的对象没有strong类型指针指向时，即使有weak指针指向的对象依然会被释放weak指针并置为nil（即当weak指向的对象被销毁时weak指针会自动置nil）

  assin: 一般修饰基本数据类型 包括oc基础数据类型（NSSInteger，CGFloat）和C数据类型（int、float、double、char等）存储在栈中，assin声明的属性不会增加引用计数，assin也可以用来修饰对象，但是被assin修饰的对象在释放后指针地址还是存在的，也就是说指针并没有被置为nil，成为野指针。如果后续再分配对象到堆上的某块内存时，正好分配到这块地址程序就会崩溃。之所以可以修饰基本数据类型，因为基本数据类型一般分配在栈上，栈的内存会由系统自动处理，不会造成野指针。

  weak：修饰object类型，修饰对象在释放后，指针地址会被置为nil，是一种弱引用。
  delegate为何要使用weak修饰：为了避免循环引用，weak和strong不同的是，当一个对象不再有strong类型的指针指向他的时候，他就会释放，即使还有weak型指针指向它，那么这些weak型指针也将会被清除。

  copy：会在内存里拷贝一份对象，两个指针指向不同的内存地址。一般用来修饰nsstring等有对应可变类型的对象，因为他们有可能和对应的可变类型之间进行赋值，为确保对象中的字符串不被修改，应该在设置属性时拷贝一份，若使用strong修饰，如果对象在外部被修改了会影响到属性。

  block属性为什么要用copy来修饰？
  在mrc下，block在创建的时候，它的内存时分配在栈（stack）上的，而不是在堆（heap）上的，可能随时会被回收。它本身的作用域是属于创建时候的作用域，一旦在创建时候的作用域外面调用block将导致程序崩溃。通过copy可以把block拷贝到堆上，保证block的声明作用域外使用。在arc下写不写都行，编译器会自动对block进行copy操作。

  __block和__weak的区别：
  __block在arc和mrc下都可以使用，可以修饰对象和基本数据类型
  __weak只能在arc下使用，只能修饰对象不能修饰基本数据类型
  __block对象可以在block中被重新赋值，__weak不可以。
  __block对象在arc下可能会导致循环引用，非arc下回避免循环引用，__weak只在arc下使用，可以避免循环引用。

  总结：__block 是强引用类型， __weak是弱引用类型，两者相比，__block更加全能，因为它可以在MRC和ARC都可以使用，既能修饰对象又能修饰基本数据类型。但是它还是有缺点，缺点在于在ARC环境下，会引起循环引用。而__week则只能在ARC环境下使用，且只能修饰对象，但是它不会发生循环引用。
## 4.weak修饰的变量地址被释放后为何会被置为nil？
  在runtime中专门维护了一个用于存储weak指针变量的hash表，这个表key是weak指针指向的内存地址，value是指向这个内存地址的所有weak指针，实际是一个数组，释放时根据对象地址获取所有weak指针 地址数组，然后遍历这数组把其中的数据置为nil，最后把这个对象从weak表中删除。
## 5.atomic一定是线程安全的吗？
  atomic可以保证setter和getter存取的线程安全，并不能保证整个对象是线程安全的。比如声明一个nsmutablearray对象是原子属性的array，此时self.array和self.array = otherArray是安全的，但是使用【self.array objectAtIndex:index】任然不是线程安全的，需要用锁来保证线程安全。

  