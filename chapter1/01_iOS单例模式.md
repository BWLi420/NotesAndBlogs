# 单例模式的作用
- 在程序运行过程中，一个类只有一个唯一的实例，而且该实例提供一个访问点，易于供外界访问
- 控制实例个数，节约系统资源

# 单例的应用场景
- 类只能有一个实例, 而且必须从一个为人熟知的访问点对其进行访问, 比如工厂方法
- 这个唯一的实例只能通过子类化进行扩展, 而且扩展的对象不会破坏客户端代码
- 在整个应用程序中，共享一份资源（这份资源只需要创建初始化一次）

# 单例模式设计要点:
1. 某个类只能有一个实例
2. 必须自行创建这个对象
3. 必须自行向整个系统提供这个实例
4. 这个方法必须是一个静态类

# 单例模式的设计
- ARC 模式下
 - .h文件中提供类方法，方便外界访问，遵守NSCopying 和 NSMutableCopying 协议

  ```obj-c
  +(instancetype)shareTool;
  ```
 - .m文件中具体实现

  ```obj-c
  // 01 提供静态变量
  static BWShareTool *_instance;
  // 02 重写allocWithZone方法,保证只分配一次存储空间
  +(instancetype)allocWithZone:(struct _NSZone *)zone {
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        _instance = [super allocWithZone:zone];
    });
    return _instance;
  }
  // 03 提供类方法
  +(instancetype)shareTool {
    return [[self alloc]init];
  }
  // 04 更严谨的做法
  -(id)copyWithZone:(NSZone *)zone {
    return _instance;
  }
  -(id)mutableCopyWithZone:(NSZone *)zone {
    return _instance;
  }
  ```
- MRC 模式下
 - 除了前面和 ARC 模式下一样，还需要在 .m 文件中增加如下方法：

  ```obj-c
  -(oneway void)release {
  }
  -(instancetype)retain {
    return _instance;
  }
  -(NSUInteger)retainCount {
    return MAXFLOAT;
  }
  ```

# 单例模式的宏抽取

以上方法定义单例模式只适用于单个对象，如果在程序中有多个对象都需要用到单例模式时，再使用上述的方法就会显得比较笨重，这时我们就可以使用宏把单例模式抽取出来，方便以后开发中的使用。

- 在抽取宏时要注意 ARC 与 MRC 模式的区别
- 具体请参考github项目地址：[https://github.com/mortal-master/BWSingletonTool](https://github.com/mortal-master/BWSingletonTool)
- 使用说明：如果现在需要定义一个单例类 Person，则只需要在 Person 类的 .h 文件中导入宏，然后调用 interfaceSingleton 方法：

```obj-c
interfaceSingleton(Person);
```
在 Person 类的 .m 文件中调用 implementationSingleton 方法即可。

```obj-c
implementationSingleton(Person);
```
通过以下测试查看结果：

```obj-c
Person *p1 = [[Person alloc]init];
Person *p2 = [Person new];
Person *p3 = [Person sharePerson];
Person *p4 = [p3 copy];
Person *p5 = [p3 mutableCopy];
```
得到打印结果为：
![宏抽取单例创建对象打印结果](http://upload-images.jianshu.io/upload_images/2997426-012ea3069e8b9a6f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
至此，说明宏抽取的单例已经可以使用。

