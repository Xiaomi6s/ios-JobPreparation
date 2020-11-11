# didAddSubview:、willRemoveSubview:、willMoveToSuperview:、didMoveToSuperview、willMoveToWindow 何时被调用
```Objective-C
// 当视图添加子视图时调用
- (void)didAddSubview:(UIView *)subview;


// 当子视图从本视图移除时调用

- (void)willRemoveSubview:(UIView *)subview;


// 当视图即将加入父视图时 / 当视图即将从父视图移除时调用

- (void)willMoveToSuperview:(nullable UIView *)newSuperview;


// 当试图加入父视图时 / 当视图从父视图移除时调用

- (void)didMoveToSuperview;


// 当视图即将加入父视图时 / 当视图即将从父视图移除时调用

- (void)willMoveToWindow:(nullable UIWindow *)newWindow;


// 当视图加入父视图时 / 当视图从父视图移除时调用

- (void)didMoveToWindow;

```
