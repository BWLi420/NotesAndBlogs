关于 iOS 多线程中 GCD 的基础知识已在上一篇文章中详细说明，本文主要对 GCD 中的几种常用函数做进一步概述。

在 GCD 中的常用函数主要有**栅栏函数、一次性函数、延迟执行函数、快速迭代函数**，最后包括 *`队列组的使用`以及`线程间的通信`*。

# 一、栅栏函数

栅栏函数的作用是用来控制任务的执行顺序，必须上面的任务 1 执行完毕才执行当前栅栏任务，必须当前栅栏任务执行完毕才执行下面的任务 2。

```objective-c
//1.获取并发队列 
dispatch_queue_t queue = dispatch_queue_create("barrier", DISPATCH_QUEUE_CONCURRENT);
    //2.异步函数
dispatch_async(queue, ^{
    NSLog(@"1 --- %@", [NSThread currentThread]);
});
    //3.设置栅栏
dispatch_barrier_async(queue, ^{
    NSLog(@" ++++++++++++++ ");
});
dispatch_async(queue, ^{
    NSLog(@"2 --- %@", [NSThread currentThread]);
});
```

**注意：**`栅栏函数不能使用全局并发队列，会丧失拦截功能`

# 二、一次性函数

一次性函数顾名思义就是在程序的运行过程中，一次性函数中的代码只会执行一次，一次性函数的内部原理就是开始时 onceToken == 0，如果为 0 则执行，执行之后值为 -1，就不执行了。一次性函数在单例中会经常使用。

```objective-c
static dispatch_once_t onceToken;
dispatch_once(&onceToken, ^{
    NSLog(@"只执行一次，默认是线程安全的");
});
```

**注意：**`一次性函数不能放入懒加载中`

# 三、延迟执行函数

在以往的学习中，我们已经知道了两种实现延时执行的方法：

```objective-c
// 2秒后再调用self的run方法
[self performSelector:@selector(run) withObject:nil afterDelay:2.0];
```
```objective-c
// 2秒后再调用self的run方法
[NSTimer scheduledTimerWithTimeInterval:2.0 target:self selector:@selector(run) userInfo:nil repeats:NO];
```

在 GCD 中利用 dispatch_after 也可以实现延迟执行的效果：

```objective-c
dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(2.0 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
    NSLog(@"延时 2.0 秒执行");
});
```

在延迟执行中，dispatch_after 函数本身是一个异步函数，其中的队列也可以使用自定义的队列，它的实质是延迟 2 秒再将任务提交到队列中，从而达到延迟执行的效果。

# 四、快速迭代函数（遍历）

在以往我们知道可以使用 for 循环和 block 进行遍历，在 GCD 中我们也可以使用 dispatch_apply 函数进行快速迭代。

```objective-c
dispatch_queue_t queue = dispatch_get_global_queue(0, 0);  
dispatch_apply(10, queue, ^(size_t i) {
    NSLog(@"%zd --- %@", i, [NSThread currentThread]);
});
```

在 dispatch_apply 函数中的三个参数分别代表着需要遍历的次数，存放任务的队列以及索引（就相当于 for 循环中的  i ）。

**注意：**`1. 快速迭代中的队列只能使用并发队列，不要使用串行队列或主队列，使用串行队列时就相当于 for 循环，使用主队列会造成死锁`。

`2. 在快速迭代中，会开启子线程与主线程一起并发执行任务，执行顺序是不固定的`。

# 五、队列组的基本使用

需求：同时下载图片一和图片二，当两张图片均下载完毕后合成使之成为一张图片。

在以前我们可以利用 GCD 中的栅栏函数解决这个问题，具体的思路为：使用异步函数 + 并发队列开启子线程来同时下载图片，然后添加一个栅栏函数进行拦截，最后在栅栏函数之后进行合成图片的任务，这里需要注意的是在合成图片的子线程中需要回到主线程（异步函数 + 主队列）进行展示图片。

其实除此之外，我们也可以利用队列组来解决这个问题，当队列组中所有的任务都执行完毕，就会执行 dispatch_group_notify 函数。

```objective-c
//创建队列组
dispatch_group_t group = dispatch_group_create();
//创建并发队列
dispatch_queue_t queue = dispatch_queue_create("queue", DISPATCH_QUEUE_CONCURRENT);
//将任务添加到队列组中
dispatch_group_async(group, queue, ^{
    NSLog(@"1 --- %@", [NSThread currentThread]);
});
//该函数本身是异步函数
dispatch_group_notify(group, queue, ^{
    NSLog(@"--- end --- %@", [NSThread currentThread]);
});
```

# 六、线程间的通信

在子线程中下载图片回到主线程设置并显示：

```objective-c
//1.获取队列（串行|并行） 0 就是异步函数
dispatch_queue_t queue = dispatch_queue_create(0, 0);
//2.异步函数封装任务提交到队列
dispatch_async(queue, ^{       
    //3.获取 URL
    NSURL *url = [NSURL URLWithString:@"http://c.hiphotos.baidu.com/image/pic/item/3b292df5e0fe99254674c15036a85edf8db171b2.jpg"];
    //4.下载二进制数据
    NSData *imageData = [NSData dataWithContentsOfURL:url];
    //5.将二进制数据转换为图片
    UIImage *image = [UIImage imageWithData:imageData];
    //6.回到主线程设置图片（异步函数 + 主队列）
    dispatch_async(dispatch_get_main_queue(), ^{
        self.imageView.image = image;
        NSLog(@"%@", [NSThread currentThread]);
    });
});
```
