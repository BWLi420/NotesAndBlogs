前面我们已经对 iOS 多线程中的 NSThread 和 GCD 作了初步了解与使用，在 iOS 中，使用 NSOperation 也可以实现多线程的编程。

NSOperation 是对 GCD 的一个封装，GCD 是纯 C 语言，而 NSOperation 是 OC 语言。

在 NSOperation 中有两个核心概念：**操作**与**队列**。

# 操作与队列

- **NSOperation**：用来封装操作
	- NSOperation 本身是一个抽象类，并不具备封装操作的能力，想要封装操作必须使用它的子类
	- NSOperation 的子类一共有 3 种
		- NSInvocationOperation
		- NSBlockOperation
		- 自定义子类继承 NSOperation，实现内部相应方法
		- 
- **NSOperationQueue**：用来存放任务的队列
	- 主队列：通过 mainQueue 获得，凡是放到主队列中的任务都将在主线程中执行
	- 非主队列：直接通过 alloc init 创建出来，同时具备了并发和串行的功能，默认是并发执行，可以通过设置最大并发数来实现串行执行

# 实现多线程的步骤

使用 NSOperation 和 NSOperationQueue 实现多线程非常简单，只需要两个步骤：

1. 将需要执行的操作封装到一个 NSOperation 对象中
2. 将 NSOperation 对象添加到 NSOperationQueue 中

在执行任务时，系统会自动将 NSOperationQueue 中的 NSOperation 取出来，再取出 NSOperation 封装的操作放到一条新线程中执行。

# NSOperation 的基本使用

- **NSInvocationOperation 封装操作**

	```objective-c
    //创建队列
    NSOperationQueue *queue = [[NSOperationQueue alloc]init];
    //封装任务
    NSInvocationOperation *op1 = [[NSInvocationOperation alloc] initWithTarget:self selector:@selector(run) object:nil];
    //添加到队列
    [queue addOperation:op1];
	```
	
- **NSBlockOperation 封装操作*（推荐使用）***

	```objective-c
    //创建队列
    NSOperationQueue *queue = [[NSOperationQueue alloc]init];
    //封装任务
    NSBlockOperation *op1 = [NSBlockOperation blockOperationWithBlock:^{
    NSLog(@"1 --- %@", [NSThread currentThread]);
    }];
    //添加到队列
    [queue addOperation:op1];
	```
	
	- 在使用 NSBlockOperation封装操作时，如果操作对象中封装的任务数量 > 1，就会开启子线程，和当前线程一起执行任务，如果任务数量 <= 1，就不会开启子线程。
	
		```objective-c
    	//利用 addExecutionBlock 可以追加任务，追加的任务并非一定在子线程中执行
    	[op1 addExecutionBlock:^{
              NSLog(@"1.1 --- %@", [NSThread currentThread]);
          }];
		```
		
	- 使用 addOperationWithBlock 方法系统会自动先封装操作，再将操作添加到队列中
	
		```objective-c
    	//创建队列
    	NSOperationQueue *queue = [[NSOperationQueue alloc] init];
    	//封装任务并添加到队列
    	[queue addOperationWithBlock:^{
              NSLog(@"2 --- %@", [NSThread currentThread]);
          }];
		```
	
- **自定义 NSOperation 封装操作**
	- 通过重写内部的 main 方法实现封装操作
	- 有利于代码的封装和复用
		
		```objective-c
    	-(void)main {
    	    NSLog(@"自定义 NSOperation---%@",[NSThread currentThread]);
    	}
		```

# NSOperation 的其他用法

- **设置最大并发数**
	
	- 该属性必须在任务添加到队列之前设置
	- 如果属性值 > 1，则该队列并发执行（同一时间最多执行的任务数就是设置的属性值）；如果属性值 = 1，则串行执行；如果属性值 = 0，则不执行任何任务。
- 属性值默认 = -1，表示并发执行所有任务
	
  ```objective-c
    //创建队列
    NSOperationQueue *queue = [[NSOperationQueue alloc] init];
	  queue.maxConcurrentOperationCount = 1;
	```
	
- **依赖与监听**

	在之前想要实现*必须完成某些任务后再执行特定的任务*这样的需求时，我们可以使用 GCD 中的栅栏函数或者队列组来解决问题，同样的，在 NSOperation 中可以通过设置依赖来解决。
	
	```objective-c
    //1.创建队列
    NSOperationQueue *queue = [[NSOperationQueue alloc] init];
    //2.封装任务
    NSBlockOperation *op1 = [NSBlockOperation blockOperationWithBlock:^{
        NSLog(@"1 --- %@", [NSThread currentThread]);
    }];
    NSBlockOperation *op2 = [NSBlockOperation blockOperationWithBlock:^{
        NSLog(@"2 --- %@", [NSThread currentThread]);
    }];
    NSBlockOperation *op3 = [NSBlockOperation blockOperationWithBlock:^{
        NSLog(@"3 --- %@", [NSThread currentThread]);
    }];
    //3.设置依赖
    [op2 addDependency:op3];
    [op1 addDependency:op2];
    //4.将操作添加到队列
    [queue addOperation:op1];
    [queue addOperation:op2];
    [queue addOperation:op3];
	```
	
	- 如上设置依赖，则必须 op3 执行完毕才能执行 op2，op2 执行完毕才能执行 op1
	- **注意：依赖不能相互设置，且必须在添加到队列之前设置，可以对不同队列中的操作设置依赖**
	- 
	![](https://tva1.sinaimg.cn/large/006y8mN6gy1g8qtvkbpqcj30a905oq3o.jpg)
	
	- *补充：当一个任务执行完毕后，会在子线程中执行 `completionBlock` 中的代码块*
	
		```objective-c
    	op2.completionBlock = ^{
              NSLog(@"op2 已经执行完毕 --- %@", [NSThread currentThread]);
          };
		```
	
- **队列的状态（暂停、恢复与取消）**
	- 暂停（suspended 属性设置为 YES）
		- self.queue.suspended = YES;
		- 暂停队列只能暂停下一个操作，当前正在执行的操作必须要执行完毕
	- 恢复（suspended 属性设置为 NO）
		- [self.queue setSuspended:NO];
		- 继续执行当前队列中未执行的操作
	- 取消（cancelAllOperations）
		- [self.queue cancelAllOperations];
		- 取消队列中的所有任务，当前正在执行的任务要等到执行完毕之后才能取消
		- 取消操作之后是不能再恢复的，就好像所有的操作都被移除了
	- 自定义 NSOperation 的取消操作
		- 如果想要实现随时可以取消操作，可以在耗时操作内部进行判断，但是这样会消耗大量性能，不建议这样做
		
			```objective-c
    		for (int i = 0; i < 3000; i++) {
    	    	NSLog(@"1 --- %i", i);
           		if (self.isCancelled) {
           		    return;
           		}
           	}
			```
			
		- 苹果官方建议：每执行完一段耗时操作，就判断一下当前操作是否被取消，如果被取消则退出，提高程序的性能
		
			```objective-c
    		for (int i = 0; i < 3000; i++) {
    		    NSLog(@"1 --- %i", i);
    		}
    		//判断操作是否被取消
    		if (self.isCancelled) {
    		    return;
    		}
    		for (int i = 0; i < 3000; i++) {
    		    NSLog(@"2 --- %i", i);
    		}
			```

# **NSOperation 线程间的通信**

```objective-c
//创建队列
NSOperationQueue *queue = [[NSOperationQueue alloc] init];
//封装任务
NSBlockOperation *download = [NSBlockOperation blockOperationWithBlock:^{
    NSURL *url = [NSURL URLWithString:@"http://a.hiphotos.baidu.com/image/pic/item/7acb0a46f21fbe091bd6251369600c338744ad29.jpg"];
    NSData *data = [NSData dataWithContentsOfURL:url];
    UIImage *image = [UIImage imageWithData:data];
    //回到主线程设置图片     
    [[NSOperationQueue mainQueue] addOperationWithBlock:^{
        self.imageView.image = image;
    }];
}];
//将操作添加到队列
[queue addOperation:download];
```

# 总结（子线程回到主线程的四种方法）

```objective-c
[self performSelectorOnMainThread:@selector(run) withObject:nil waitUntilDone:YES];
```

```objective-c
[self performSelector:@selector(run) onThread:[NSThread mainThread] withObject:nil waitUntilDone:YES];
```

```objective-c
dispatch_async(dispatch_get_main_queue(), ^{   });
```

```objective-c
[[NSOperationQueue mainQueue] addOperationWithBlock:^{ }];
```

