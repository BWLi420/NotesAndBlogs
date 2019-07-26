GCD 的全称是 Grand Central Dispatch，可译为“牛逼的中枢调度器”，GCD 是纯 C 语言，提供了非常多且强大的函数。

- GCD 的优势
	- GCD是苹果公司为多核的并行运算提出的解决方案
	- GCD会自动利用更多的CPU内核（比如双核、四核）
	- GCD会自动管理线程的生命周期（创建线程、调度任务、销毁线程）
	- 程序员只需要告诉GCD想要执行什么任务，不需要编写任何线程管理代码

# 任务与队列

*在 GCD 中有两个核心概念，分别是任务和队列。*

- 任务：用来执行什么操作
- 队列：用来存放任务

*使用 GCD 只需要两个步骤：*

1. 封装任务：
	- 确定想要做的事情
- 将任务添加到队列中：
	- GCD会自动将队列中的任务取出，放到对应的线程中执行
	- 任务的取出遵循队列的 FIFO 原则：先进先出，后进后出，即先存进去的任务优先取出执行

# 执行任务的两种常用函数

1. 同步函数
	- 只能在当前线程中执行任务，不具备开启新线程的能力
	
	```
	dispatch_sync(dispatch_queue_t queue, dispatch_block_t block);
	```
2. 异步函数
	- 可以在新的线程中执行任务，具备开启新线程的能力
	
	```
	dispatch_async(dispatch_queue_t queue, dispatch_block_t block);
	```

# 队列的基本类型

1. 并发队列
	- 可以让多个任务并发（同时）执行（自动开启多个线程同时执行任务）
	- 并发功能只有在异步（`dispatch_async`）函数下才有效
2. 串行队列
	- 让任务一个接着一个地执行（一个任务执行完毕后，再执行下一个任务）
3. 主队列
	- 凡是添加到主队列中的任务一律放到主线程中执行
4. 全局并发队列
	- 在系统中默认就存在了四种优先级的全局并发队列
		- DISPATCH_QUEUE_PRIORITY_HIGH // 高
		- DISPATCH_QUEUE_PRIORITY_DEFAULT // 默认（中）
		- DISPATCH_QUEUE_PRIORITY_LOW // 低
		- DISPATCH_QUEUE_PRIORITY_BACKGROUND // 后台

**注意：同步和异步主要影响的是`能不能开启新的线程`，并发和串行主要影响的是`任务的执行方式`，这里千万不要混淆了。**

# GCD 的六种基本使用方式

> 在 iOS6.0 之前，在 GCD 中每当使用带 creat 单词的函数创建对象之后，都应该对其进行一次 release 操作。在 iOS6.0 之后，GCD 被纳入到了 ARC 的内存管理机制，就不再关心内存问题了。

1. **同步函数 + 串行队列**
	- 不会开线程，所有的任务在当前线程串行执行
	
	```
	/**
      "syncSerial"    队列的名称，调试的时候使用
      DISPATCH_QUEUE_CONCURRENT     并发队列
      DISPATCH_QUEUE_SERIAL         串行队列
     */
    dispatch_queue_t queue = dispatch_queue_create("syncSerial", DISPATCH_QUEUE_SERIAL);    
    dispatch_sync(queue, ^{
        NSLog(@"--- %@", [NSThread currentThread]);
    });
	```
2. **同步函数 + 并发队列**
	- 不会开线程，所有的任务在当前线程串行执行
	
	```
	//
    dispatch_queue_t queue = dispatch_queue_create("syncConcurrent", DISPATCH_QUEUE_CONCURRENT);
    dispatch_sync(queue, ^{
        NSLog(@"--- %@", [NSThread currentThread]);
    });
	```
3. **同步函数 + 主队列**
	- `产生死锁`
	- 原因：同步函数不会新开线程，必须在当前线程（主线程）执行完本次任务才能继续向下执行，但是主队列中的任务必须在主线程中执行，就会去调用主线程来执行任务，这样就会形成主线程要执行任务，任务要调用主线程的情况，形成死循环，即形成死锁。
	
	```
	//获取主队列
    dispatch_queue_t queue = dispatch_get_main_queue();
    dispatch_sync(queue, ^{
        NSLog(@"--- %@", [NSThread currentThread]);
    });
	```
4. **异步函数 + 串行队列**
	- 会开一条线程，所有的任务在当前线程串行执行
	
	```
	//
    dispatch_queue_t queue = dispatch_queue_create("asyncSerial", DISPATCH_QUEUE_SERIAL);    
    dispatch_async(queue, ^{
        NSLog(@"--- %@", [NSThread currentThread]);
    });
	```
5. **异步函数 + 并发队列**
	- 会开多条线程（至少一条，具体由系统决定，与任务的数量无关），所有的任务并发执行
	
	```
	//
    dispatch_queue_t queue = dispatch_queue_create("asyncConcurrent", DISPATCH_QUEUE_CONCURRENT);
    //将任务添加到队列中
    dispatch_async(queue, ^{
        NSLog(@"--- %@", [NSThread currentThread]);
    });
	```
6. **异步函数 + 主队列**
	- 不会开线程，所有的任务在主线程中串行执行
	
	```
	//获取主队列
    dispatch_queue_t queue = dispatch_get_main_queue();    
    dispatch_async(queue, ^{
        NSLog(@"--- %@", [NSThread currentThread]);
    });
	```
	
# 队列的执行效果

|    | 并发队列 | 手动创建的串行队列 | 主队列 |
| ---- | ---- | ---- | ---- |
| 同步( sync ) | 没有开启新线程，当前线程串行执行任务 | 没有开启新线程，当前线程串行执行任务 | 死锁 |
| 异步( async ) | 开启多条新线程，并发执行任务 | 开启一条新线程，串行执行任务 | 没有开启新线程，主线程串行执行任务 |

***注意：***使用同步函数往当前串行队列中添加任务，会卡住当前的串行队列。
