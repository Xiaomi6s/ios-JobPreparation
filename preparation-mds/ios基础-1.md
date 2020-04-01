# 基础篇 - 语义
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
## 6.block
  block本质上是一个oc对象，它内部也有isa指针
  block是封装了函数调用以及函数调用环境的oc对象
  block是封装函数及其上下文的oc对象。
```Objective-C
int age=10;
void (^Block)(void) = ^{
    NSLog(@"age:%d",age);
};
age = 20;
Block();
```
输出：age=10，原因是创建block的时候，已经把age的值存储在里面了
```Objective-C
auto int age = 10;
static int num = 25;
void (^Block)(void) = ^{
    NSLog(@"age:%d,num:%d",age,num);
};
age = 20;
num = 11;
Block();
```
输出 age=10 num=11 auto变量block访问方式是指传递，static变量block访问是指针传递
### 为什么block对auto和static变量捕获有差异？
 auto变量可能会销毁，内存可能会消失，不采用指针访问，static变量一直保存在内存中，指针访问即可。
### block有几种类型，如何判断是那种类型？
 __NSGlobalBlock __ 在数据区
 __NSMallocBlock __ 在堆区
 __NSStackBlock __ 在栈区
 没有访问auto变量的block是_NSGlobalBlock_ 放在数据区
 访问了auto变量的block是_NSStackBlock_
 [_NSStackBlock_ copy]操作就变成了_NSMallocBlock_
 注意：NSStackBlock 只在mrc下存在，超出变量作用域，栈上的block和_block类型变量都会被销毁，NSMallocBlock在堆区域，在变量作用域结束时，变量不受影响。
## 7.protocol、category、extension
### 1.category
#### 分类的优点
 category可以把类实现到不同的文件里。这样做的好处：
  1.可以减少单个文件的体积。
  2.可以把不同的功能组织到不同的category里
  3.可以由多个开发者共同完成一个类
  4.可以按需加载想要的category
  5.声明私有方法。
#### 分类的源码
  category的源码如下：
```Objective-C
  Category 是表示一个指向分类的结构体的指针，其定义如下：
  typedef struct objc_category *Category;
  struct objc_category {
  char *category_name                          OBJC2_UNAVAILABLE; // 分类名
  char *class_name                             OBJC2_UNAVAILABLE; // 分类所属的类名
  struct objc_method_list *instance_methods    OBJC2_UNAVAILABLE; // 实例方法列表
  struct objc_method_list *class_methods       OBJC2_UNAVAILABLE; // 类方法列表
  struct objc_protocol_list *protocols         OBJC2_UNAVAILABLE; // 分类所实现的协议列表
  struct property_list_t *_classProperties;
}
```
  从源码我们可以看到category结构体主要包含了分类定义的实例方法、类方法，其中instance_methods 列表是 objc_class 中方法列表的一个子集，而class_methods列表是元类方法列表的一个子集。
#### 分类的特点
 1.category 只能给某个已有的类扩充方法，不能扩充成员变量。
 2.category中可以添加属性，只不过@property只会生成setter和getter的声明，不会生成setter和geter的实现和成员变量。
 3.如果category中的方法和类中原有方法同名，运行时会优先调用category中的方法也就是category中的方法会覆盖掉类中原有的方法。同名方法调用优先级为：分类>本来>父类
 4,如果多个分类中都有和原有类中同名的方法，那么调用顺序由编译器决定，编译器会执行最后一个参与编译的分类中的方法。
#### 分类的方法何时合并到类对象中？
 通过rutime动态将分类的方法合并到类对象、元类对象中
#### 分类的方法是如何添加到类对象方法列表中的？
 1.获取分类方法列表的count，然后原来的类方法列表内存移动count
 2.分类方法列表内容拷贝到原来类方法列表的前方。
 3.同样的方法，优先调用分类的方法
 4.分类具有同样的方法，根据编译顺序决定，取最后编译分类的方法列表。
#### Category的加载处理过程？
 1.通过runtime加载某个类所有category数据
 2.把所有Category的方法、属性、协议数据，合并到一个大数组中
 3.后面参与编译的category数据，会在数组的前面
 4.将合并后的分类数据（方法、属性、协议），插入到类原来数据的前面。
#### +load方法调用顺序？
 1.先调用类的+load方法
   1.1按照编译先后顺序调用（先编译先调用）
   1.2先调用父类的+load再调用子类的+load
 2.在调用分类的+load方法
  2.1按照编译先后顺序调用（先编译先调用）
  2.2每个类、分类的+load在程序中调用一次，只有在加装类的时候调用一次
  2.3不存在分类的+load方法会覆盖+load的方法。
#### +load方法和其他的类方法调用方式不同？
 其他分类的方法是通过消息转发机制调用的，通过isa和superClass来寻找的，而+load是通过函数指针指向函数，拿到函数地址，分开直接调用的，直接通过内存地址查找调用的。
#### Category中有load方法吗？load方法是什么时候调用的？load 方法能继承吗？
 有load方法
 在runtime加载类和分类的时候调用
 可以继承，但是一般不主动调用load方法，由系统自动调用。
#### +initialize方法是怎么调用的？
 +initialize是在类第一次接收到消息时调用的，消息转发机制调用（objc_send）
#### +initialize方法调用顺序？
 1.先调用父类的+initalize，再调用子类的+initialize；（先初始化父类，再初始化子类，每个类之后初始化一次）子类内部+initialize会主动调用父类的++initialize。
 2.如果子类没有实现+initialize会调用父类的+initialize（所以父类的+initialize可能会调用多次）
 3.如果该类的分类中也实现了initialize那么会覆盖原本类中的+initialize，如果多个分类同时实现了最后会调用编译器最后一个参与编译的分类中的+initialize。
#### +initialize和+load的区别
  1.调用方式
   load是通过函数指针指向函数，拿到函数地址分开调用的，直接通过内存地址查找调用的。
   initialize是通过objc_msgSend调用的。
  2.调用时刻
   load是runtime加载类和分类的时候调用（只调用一次）
   initialize是类第一次接收到消息时候调用。每一个类之后initialize一次（如果子类没有实现initialize父类的initialize可能会多次调用）。
  3.调用顺序
   load先调用类的load（a 先编译的类，先调用 b.在调用子类的load之前会先调用父类的load），再调用分类的load（先编译先调用）
   initialize先初始化父类，再初始化子类（可能最终调用的是父类的initialize）。
#### Category能否添加成员变量？为什么？
   由于分类底层结构限制，不能添加成员变量到分类中，但是可以通过关联对象实现添加属性。
   原因：category是通过runtime动态地把category中的方法添加到类中（苹果在实现的过程中并没有将属性添加到类中，所以属性仅仅是声明了setter和getter方法并为实现）在Objective-C提供的runtime函数中确实有一个class_addIvar()函数用于给类添加成员变量，但是这个函数只能在构建类的过程中被调用，一旦完成类定义就不能添加成员变量，经过编译的类在程序启动后被runtime加载，没有机会调用addIvar，程序在运行时动态创建的类需要在调用class_registerClassPair之后才可以被使用，同样没有机会在添加成员变量。
   方法和属性并不属性类实例，而成员变量属于类实例，我们所说的类实例实际是指一块内存区域，包含了isa指针和所有成员变量，所以假如允许修改类成员变量布局，已经创建出的类实例就不符合类定义了，变成了无效对象，单方法定义是在objc_class中关联的不管如何增删类方法都不影响类实例的内存布局。（因为在运行期，对象的内存布局已经确定，如果添加成员变量就会破坏类内存布局这个是不能添加成员变量的根本原因）
### 2.extension 以及和 category的区别
   extension延展、扩展，匿名分类。extension看起来像匿名的category，但是两者几乎完全是两个东西，和category不同的是，extension不但可以声明方法，还可以声明属性、成员变量。extension一般用于私有方法、私有属性、私有成员变量的声明。

   category是拥有.h和.m文件的，extension只有一个.h文件或者在一个类的.m文件中。extension在编译器决议，它是类的一部分，在编译器和头文件里的@intenface以及实现文件里的@implement一起形成一个完整的类，它伴随着类的产生而产生，也一起消亡，必须拥有一个类的源码才能为一个类添加extension，所以无法为系统类添加extension。

   但是category 它是运行期决议的，所以就可以得出extension和category一个明显的区别，extension可以添加实例变量，而category不能添加实例变量（因为运行期对象的内存布局已经确定，如果添加实例变量就会破坏类的内部布局，这对编译型语言是灾难性的）。
### 3.Protocol 以及和category的区别
   Protocol声明了任何类都能够选择实现的程序接口。协议能够使两个不同的继承树上的类相互交流并完成特定的目的。因此它提供了除继承外的另一种选择。任何能够为其他类提供有用行为的类都能够声明接口来匿名的传达这个行为。任何其他类都能够选择遵守这个协议并实现其中的一个或多个方法，从而利用这个行为。如果协议遵守者实现了协议中的方法，那么声明协议的类就能够通过遵守者调用协议中的方法。

   协议能声明方法和属性，但是声明的属性没有实现getter和setter方法也不会生成成员变量，这就要求遵守者必须自己实现setter和getter方法，但是有一种情况不需要，那就是遵守者本来就有这个属性，此时系统会为这个属性自动生成setter和getter方法，既然已经实现了，那么遵守者就没必要去实现协议中的这个属性了。
### const、static、extern
  static：被static修饰的变量属于静态变量存储在静态数据区，该区域的变量在编译时被分配内存而且在app运行期间一直存在内存中，直到app停止运行，所以被static修饰的变量在内存中只存在一份，并且在整个app运行期只被运行一次，用于对变量作用域的限制，限制变量只可在本文件中使用，但是不限制变量的读写。
  const：被const修饰的右边变量为常量不可修改。
  extern：对所有文件可见，定义全局变量。
### static const 与 #define
 相同点：都不能再被修改
 不同点：static const修饰只有一份内存，宏定义只是简单的替换，每次使用都需要创建一份内存。
 使用static const修饰更加高效，在同一个文件内可以使用static const 取代#define。
### const、extern以及const、static在开发中的使用 
 想要访问全局变量可以使用extern关键字（全局变量定义不能有static修饰）。

 extern 和 const共同使用定义全局变量的文件
 一般在先建.h文件专门存放常量的引用。
 ``` Objective-c
 #import <Foundation/Foundation.h>

extern NSString *const appKey;
extern NSString *const notificationName;
 ```
 在.m文件中专门存放const修饰的变量，需要的时候导入头文件。
  ``` Objective-c
 #import "Const.h"

NSString *const appKey           = @"12666666";
NSString *const notificationName = @"notification";
 ```
static 和const组合可以定义局部作用的静态常量。

## 8.内存管理

### 内存的五大分区
 > + 栈区（stack）：由编译器自动完成分配和释放，不需要程序员手动管理，主要存储了函数的参数和局部变量等。栈区地址从高到低分配，先进后出。会存一些局部变量，函数跳转跳转时现场保护(寄存器值保存于恢复)，这些系统都会帮我们自动实现，无需我们干预。
 > + 堆区（heap）：需要程序员手动开辟并管理内存（arc模式下不需要程序员考虑释放问题，mrc下alloc申请内存，release释放内存） 创建的对象都放在这里，堆区的地址是从低到高的分配。
 > + 全局区/静态区(static)：全局变量和静态变量的存储是放在一起的，初始化的全局变量和静态变量在一块区域，未初始化的全局变量和未初始化的静态变量在相邻的另一块区域。程序结束后系统释放。
 > + 常量区：存放常量，程序结束后系统释放。
 > + 代码区： 存放app代码，app程序会拷贝到这里。

 ### 内存管理原则
 oc使用了一种引用计数的机制来管理对象，在MRC下如果对一个对象使用了alloc、copy、retain，那么你必须使用相应的release或者autorelease（即遵循谁创建谁释放，谁引用谁管理的原则）。在arc（自动引用计数）系统会在编译的时候在合适地方插入retain、release和autorelease，通过生成正确的代码区自动释放对象和保持对象。
 
### autorelease以及autoreleasePool
 autorelease实际上只是把对release的调用延后了，对于每一个autorelease，系统只是把该object放入了当前的autoreleasePool中，当pool被释放时，该pool中所有的object会被调用release。
 
 autoreleasePool的作用：autoreleasePool被称为自动释放池，在释放池中调用了autorelease方法的对象都会被压在该池的顶部（以栈的形式管理对象）。当自动释放池被销毁时，在该池中的对象会自动调用release方法释放资源销毁对象，以此来达到自动管理内存的目的。

### autoreleasePool是什么时候被创建和销毁的？
 app启动后系统在主线程的runloop里注册两个observe，回调都是_wrapRunLoopWithAutoreleasePoolHandler（）。
 第一个observe监听的事件:
  是entry（即将进入loop），其回调内会调用_objc_autoreleasePoolPush（）创建自动释放池，其优先级最高，保证创建释放池发生在其他回调之前。
 第二个observe监听了两个事件
  _BeforeWaiting（准备进入休眠）时，调用_objc_autoreleasePoolPop（）和_objc_autoreleasePoolPush（）释放旧的池，并创建新的池

  _Exit（即将退出loop）时，调用 _objc_autoreleasePoolPop（）来释放自动释放池，这个observe优先级最底，保证其释放池在其他回调之后。

  在主线程执行的代码，通常写在诸如事件回调，Timer回调内，这些回调会被runloop创建好的autoreleasePool环绕着，所以不会内存泄漏，所以开发者不必显示创建pool了。

  现在我们知道autoreleasePool是在runloop即将进入runloop和准备进入休眠这两种状态的时候被创建和销毁的。

  所以autorelease的释放有两种情况：1.autorelease对象是在当前的runloop迭代结束时释放的，而他能够释放的原因是系统在每个runloop迭代中都加入了自动释放池的push和pop。2.手动调用autoreleasePool的释放方法（drain方法）来销毁autoreleasePool。
 
