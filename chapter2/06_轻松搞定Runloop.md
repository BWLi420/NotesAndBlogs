# Runloop

## 认识 Runloop

- Runloop 就是运行循环，如果没有 Runloop，程序一运行就会退出，有 Runloop 就相当于在程序内部开了一个死循环
- 在 iOS 开发中，有两套 API 可以访问 Runloop：**`NSRunloop`** 和 **`CFRunloopRef`**，它们是等价的，可以相互转换
- NSRunloop 是基于 CFRunloopRef 的 OC 包装
- 参考资料：[苹果官方文档](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/Multithreading/RunLoopManagement/RunLoopManagement.html)、[CFRunloopRef 源码](http://opensource.apple.com/source/CF/CF-1151.16/)

## Runloop 的本质

- Mach 是 XNU 的内核，进程、线程和虚拟内存等对象通过端口发消息进行通信，Runloop 通过 mach_msg() 函数发送消息
- 如果没有 port 消息，内核会将线程置于等待状态 mach_msg_trap() 
- 如果有消息，判断消息类型处理事件，并通过 modeItem 的 callback 进行回调

## Runloop 的作用

- 保证程序的持续运行
- 处理 APP 中的各类事件
- 节省 CPU 资源，提高程序性能（有事情就做，没事情就休息）

## Runloop 与线程的关系

- NSRUnloop 与线程一一对应
- 主线程中的 runloop 在程序运行时已经创建并启动了
- 子线程中的 runloop 需要我们手动创建并开启
- runloop 在线程结束时，也会销毁

## 获取 Runloop 对象

- 获取主线程 runloop 对象

  ```objective-c
  NSRunLoop *mainRL = [NSRunLoop mainRunLoop];
  CFRunLoopRef mainRLRef = CFRunLoopGetMain();
  ```
- 获取当前线程 runloop 对象

  ```objective-c
  NSRunLoop *currentRL = [NSRunLoop currentRunLoop];
  CFRunLoopRef currentRlRef = CFRunLoopGetCurrent();
  ```
- 通过子线程创建 runloop

  ```objective-c
  NSRunLoop *curRunloop = [NSRunLoop currentRunLoop];
  ```
  这个方法本身是懒加载的，如果是第一次调用该方法，那么就创建子线程对应的 runloop。
- *补充：*runloop 对象是利用字典进行存储的，key 值对应线程对象，value 值对应该线程的 runloop，在子线程中runloop 不会自动创建。

## Runloop 的相关类

与 runloop 相关的共有五个类：CFRunLoopRef、CFRunLoopModeRef、CFRunLoopTimerRef、CFRunLoopSourceRef、CFRunLoopObserverRef
![Runloop 五个相关类之间的关系](https://tva1.sinaimg.cn/large/006y8mN6gy1g8qtvie170j308o06l3z2.jpg)

1. **CFRunLoopRef**（ Runloop 对象）

   -  Runloop 对象就是Runloop 本身
2. **CFRunLoopModeRef**（ Runloop 的运行模式）
   - 一个 Runloop 包含若干个 Mode，而每个 Mode 又包含若干个 Source/Timer/Observer，每次 RunLoop 启动时，只能指定其中一个 Mode，这个 Mode 被称作 CurrentMode。如果需要切换 Mode，只能退出 Loop，再重新指定一个 Mode 进入，这样就可以分隔开不同组的 Source/Timer/Observer，让其互不影响。
   - Mode 的分类，系统默认注册了 5 个 Mode：
     - *kCFRunLoopDefaultMode*：App的默认Mode，通常主线程是在这个 Mode 下运行
     - *UITrackingRunLoopMode*：界面跟踪 Mode，用于 ScrollView 追踪触摸滑动，保证界面滑动时不受其他 Mode 影响
     - *UIInitializationRunLoopMode*: 在刚启动 App  时进入的第一个 Mode，启动完成后就不再使用
     - *GSEventReceiveRunLoopMode*: 接受系统事件的内部 Mode，通常用不到
     - *kCFRunLoopCommonModes*: 占位用的 Mode，不是一种真正的 Mode，就相当于  KCFRunLoopDefaultMode 和 UITrackingRunLoopMode的合体
3. **CFRunLoopTimerRef**（ Timer 事件）
   - 基于时间的触发器，基本等同于 NSTimer
   - NSTimer 在各种模式下的运行效果

     ```objective-c
     [NSTimer scheduledTimerWithTimeInterval:2.0 target:self selector:@selector(run) userInfo:nil repeats:YES];
     ```
     以上 scheduledTimerWithTimeInterval 方法内部默认把创建的定时器对象添加到当前的 Runloop 中，并且指定运行模式为 NSDefaultRunLoopMode

     ```objective-c
     NSTimer *timer = [NSTimer timerWithTimeInterval:2.0 target:self selector:@selector(run) userInfo:nil repeats:YES];
     [[NSRunLoop currentRunLoop] addTimer:timer forMode:NSRunLoopCommonModes];
     ```
     以上 timerWithTimeInterval 方法创建定时器，如果想要定时器工作，还需要添加到 Runloop 中，并指定运行模式
   - **注意：**当 Runloop 切换到非指定模式，定时器就会停止工作
4. **CFRunLoopSourceRef**（ Runloop 要处理的事件源）
   - 事件源也就是输入源，只需要对它的分类有所了解就可以了
   - 以前的分法（根据官方文档）
     - Port-Based Sources
     - Custom Input Sources
     - Cocoa Perform Selector Sources
   - 现在的分法（基于函数的调用栈）
     - Source0：非基于 Port 的
     - Source1：基于 Port 的
5. **CFRunLoopObserverRef**（ Runloop 的监听者）
   - 主要用于监听 Runloop 的状态
   - Runloop 的状态主要有：

     ```objective-c
     typedef CF_OPTIONS(CFOptionFlags, CFRunLoopActivity) {
       kCFRunLoopEntry = (1UL << 0),   //即将进入 Runloop
       kCFRunLoopBeforeTimers = (1UL << 1),    //即将处理 NSTimer
       kCFRunLoopBeforeSources = (1UL << 2),   //即将处理 Sources
       kCFRunLoopBeforeWaiting = (1UL << 5),   //即将进入休眠
       kCFRunLoopAfterWaiting = (1UL << 6),    //刚从休眠中唤醒
       kCFRunLoopExit = (1UL << 7),            //即将退出 Runloop
       kCFRunLoopAllActivities = 0x0FFFFFFFU   //所有状态改变
     };
     ```
   - 实现 Runloop 的监听

     ```objective-c
     //创建监听对象，当 Runloop 的状态改变时就会调用该方法
     CFRunLoopObserverRef observer = CFRunLoopObserverCreateWithHandler(CFAllocatorGetDefault(), kCFRunLoopAllActivities, YES, 0, ^(CFRunLoopObserverRef observer, CFRunLoopActivity activity) {
       switch (activity) {
            case kCFRunLoopEntry:
                NSLog(@"进入runloop");
                break;
            case kCFRunLoopBeforeTimers:
                NSLog(@"即将处理time事件");
                break;
            case kCFRunLoopBeforeSources:
                NSLog(@"即将处理source事件");
                break;
            case kCFRunLoopBeforeWaiting:
                NSLog(@"即将休眠");
                break;
            case kCFRunLoopAfterWaiting:
                NSLog(@"runloop被唤醒");
                break;
            case kCFRunLoopExit:
                NSLog(@"runloop退出");
                break;
                
            default:
                break;
        }
     });
     //设置监听
     CFRunLoopAddObserver(CFRunLoopGetCurrent(), observer, kCFRunLoopDefaultMode);
     ```

## Runloop 的启动

1. 选择一个运行模式，且只能选择一个
2. 判断当前选择的运行模式是否为空
3. 检查当前运行模式里面是否有 Source 或 Timer，如果都没有，则 Runloop 立即退出
4. 至少有 Source 或 Timer 中的任意一个，则 Runloop 开启
5. 在检查的时候不会检查 Observer

## Runloop 的运行处理逻辑

![Runloop 的运行处理逻辑](https://tva1.sinaimg.cn/large/006y8mN6gy1g8qtvjc842j30f20bjq6o.jpg)

- 在运行 Runloop 时，RUnloop 会自动处理之前未处理的消息，并通知相关的监听者，具体的处理逻辑如下：
  1. 通知观察者 Runloop 已经启动
  - 通知观察者即将要开始的定时器
  - 通知观察者即将启动的非基于端口的源
  - 启动已经准备好的非基于端口的源
  - 如果有基于端口的源并处于等待状态，立即启动，跳到第九步
  - 通知观察者 Runloop 进入休眠
  - Runloop 进入休眠，等待发生以下事件时唤醒
    - 有事件到达基于端口的源
    - 定时器启动
    - Runloop 超时
    - Runloop 被外界手动唤醒
  - 通知观察者，线程刚被唤醒
  - 处理唤醒时收到的消息，之后跳到第二步
    - 如果有用户定义的定时器启动，处理定时器事件并重启 Runloop
    - 如果输入源启动，传递相应的信息
    - 如果 Runloop 被外界手动唤醒且未超时，重启 Runloop
  - 通知观察者，Runloop 即将结束

## Runloop 的应用

- 常驻线程
  - 在子线程中创建一个 Runloop
  - 需要至少指定 Runloop 的 Source 或者 Timer 中的任意一个（一般情况下指定 Source，比较简单）
  - 需要指定 Runloop 的运行模式（保证 Runloop 不退出）
  - 需要手动开启 Runloop

    ```objective-c
    //线程常驻
    //创建子线程
    -(IBAction)createThread:(id)sender {
      NSThread *thread = [[NSThread alloc] initWithTarget:self selector:@selector(run) object:nil];
      [thread start];
      self.thread = thread;
    }
    //继续执行任务
    -(IBAction)goOn:(id)sender {
      NSLog(@"继续执行任务");
      //线程间的通信
      [self performSelector:@selector(goOnRun) onThread:self.thread withObject:nil waitUntilDone:YES];
    }
    -(void)run {
      NSLog(@"run --- %@", [NSThread currentThread]);
      NSRunLoop *curRunloop = [NSRunLoop currentRunLoop];
      [curRunloop addPort:[NSPort port] forMode:NSDefaultRunLoopMode];
      [curRunloop run];
    }
    -(void)goOnRun {
      NSLog(@"goOnRun --- %@", [NSThread currentThread]);
    }
    ```

- imageView 的显示
  - 控制方法在特定模式下可用

    ```objective-c
    [self.imageView performSelector:@selector(setImage:) withObject:[UIImage imageNamed:@"Snip2016"] afterDelay:3.0];
    ```
    以上方法默认添加到当前的 Runloop 中，并且指定运行模式为默认 KCFRunLoopDefaultMode，如果 Runloop 切换运行模式，则图片不会加载到 imageView 上。

    ```objective-c
    [self.imageView performSelector:@selector(setImage:) withObject:[UIImage imageNamed:@"Snip2016"] afterDelay:3.0 inModes:@[NSRunLoopCommonModes]];
    ```
    以上方法添加到当前 Runloop 中，且指定运行模式为 NSRunLoopCommonModes
- 自动释放池
  - 进入 Runloop 的时候第一次创建
  - 退出 Runloop 的时候最后一次释放（超时或线程销毁）
  - 其他时候的创建与释放
    - 当 Runloop 即将休眠的时候会把之前的自动释放池释放，再重新创建一个新的自动释放池

  ```objective-c
  void msg(int n)
  {
    NSLog(@"runloop被唤醒");
    NSLog(@"runloop处理事件---%zd",n);
  }
  int main(int argc, const char * argv[]) {
    @autoreleasepool {
       NSLog(@"runloop启动了");
        
       do {
            NSLog(@"runloop询问，还有事情需要我处理吗？");
            NSLog(@"没有事情的话，我就睡觉了");
            NSLog(@"runloop进入到休眠");
          
            int number = 0;
            scanf("%zd",&number);
            msg(number);
            
        } while (1);
    }
    return 0;
  }
  ```
