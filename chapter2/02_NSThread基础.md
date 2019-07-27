# NSThread

- 特点：
  - 更加面向对象
  - 简单易用，可直接操作线程对象
  - 使用语言：OC语言
  - 使用频率：偶尔使用
  - 线程生命周期：由程序员进行管理，任务结束后自行销毁

---

## NSThread的几种创建方式

- 方式一：
  - 优点：能拿到线程对象
  
  - 缺点：需要手动启动线程
  
      ```objective-c
      NSThread *threadA = [[NSThread alloc] initWithTarget:self selector:@selector(run:) object:@"threadA"];
      [threadA start];
      ```

- 方式二：
  - 优点：自动启动线程
  
  - 缺点：不能拿到线程对象
  
      ```objective-c
      [NSThread detachNewThreadSelector:@selector(run:) toTarget:self withObject:@"threadB"];
      ```

- 方式三：（后台线程）
     - 优点：自动启动线程

     - 缺点：不能拿到线程对象

        ```objective-c
        [self performSelectorInBackground:@selector(run:) withObject:@"threadC"];
        ```

- 方式四：（自定义线程）
  - 缺点：需要自定义 NSThread 类，重写自定义类的 main 方法
  
  - 需要手动开启子线程
  
      ```objective-c
      BWThread *threadD = [[BWThread alloc] init];
      [threadD start];
      ```
  
      
  
      ```objective-c
      -(void)main {
          NSLog(@"%@", [NSThread currentThread]);
          NSLog(@"自定义线程需要重写 main 方法");
      }
      ```

## NSThread常用属性

- 设置线程的名字

```objective-c
threadA.name = @"threadA";
[threadB setName:@"threadB"];
threadC.name = @"threadC";
```

- 设置线程的优先级

```objective-c
threadA.threadPriority = 1.0;
threadC.threadPriority = 0.1;
```

 - 优先级的取值范围是 0~1.0，默认情况下是 0.5
 - 系统在执行任务时，会根据优先级的大小调整任务的执行顺序，优先级大的任务会先执行。

## NSThread线程间的通信

- 涉及到 UI 界面的操作都要放到主线程中执行

```objective-c
@interface ViewController ()
@property (weak, nonatomic) IBOutlet UIImageView *imageView;
@end
@implementation ViewController
 - (void)viewDidLoad {
    [super viewDidLoad];
}
 - (void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event {
    //创建线程
    [NSThread detachNewThreadSelector:@selector(download) toTarget:self withObject:nil];
}
 - (void)download {    
    //获取开始时间
//    NSDate *start = [NSDate date];
    CFTimeInterval start = CFAbsoluteTimeGetCurrent();    
    //1.设置URL
    NSURL *url = [NSURL URLWithString:@"http://g.hiphotos.baidu.com/image/pic/item/bd315c6034a85edf9ba34e244b540923dd54758d.jpg"];
    //2.下载图片的二进制数据到本地
    NSData *imageData = [NSData dataWithContentsOfURL:url];
    //3.把二进制数据转换成图片
    UIImage *image = [UIImage imageWithData:imageData];
    //4.回到主线程设置展示图片
    // 4.1.方式一   waitUntilDone 是否等调用的方法执行完毕再继续执行后面的任务
//    [self performSelectorOnMainThread:@selector(showImage:) withObject:image waitUntilDone:YES];
    // 4.2.方式二
//    [self performSelector:@selector(showImage:) onThread:[NSThread mainThread] withObject:image waitUntilDone:YES];
    // 4.3.方式三
    [self.imageView performSelector:@selector(setImage:) onThread:[NSThread mainThread] withObject:image waitUntilDone:YES];   
    //获取结束时间
//    NSDate *end = [NSDate date];
//    NSLog(@"用时：%f", [end timeIntervalSinceDate:start]);
    CFTimeInterval end = CFAbsoluteTimeGetCurrent();
    NSLog(@"用时：%f", end - start);   
   NSLog(@"--- END ---");
}
 - (void)showImage:(UIImage *)image {
    self.imageView.image = image;
    NSLog(@"%@", [NSThread currentThread]);
}
@end
```

