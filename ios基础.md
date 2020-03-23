1.语法语义
1.深浅拷贝
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