> 本文以 Word，Excel, PPT, Pdf 为例，其他格式类似处理

### 1. 在 info.plist 中添加以下设置，表示可以接收的文件类型

![image.png](https://tva1.sinaimg.cn/large/006y8mN6gy1g8qtucqtofj30yg0cc0wk.jpg)

- 在最新版本中需额外添加 UISupportsDocumentBrowser 并设置为YES，否则上传 App Store 会有警告
- 更多类型请参考官方 UTL：[传送门](https://developer.apple.com/library/archive/documentation/Miscellaneous/Reference/UTIRef/Articles/System-DeclaredUniformTypeIdentifiers.html)
- 代码如下

```
<key>CFBundleDocumentTypes</key>
	<array>
		<dict>
			<key>CFBundleTypeName</key>
			<string>OFFICE Document</string>
			<key>LSHandlerRank</key>
			<string>Owner</string>
			<key>LSItemContentTypes</key>
			<array>
				<string>com.microsoft.word.doc</string>
				<string>org.openxmlformats.wordprocessingml.document</string>
				<string>com.microsoft.excel.xls</string>
				<string>org.openxmlformats.spreadsheetml.sheet</string>
				<string>com.microsoft.powerpoint.ppt</string>
				<string>org.openxmlformats.presentationml.presentation</string>
				<string>com.adobe.pdf</string>
			</array>
		</dict>
	</array>
	<key>UISupportsDocumentBrowser</key>
	<true/>
```

### 2. 在 AppDelegate 中实现 application:openURL:options: 方法，用以接收外界文件

```objective-c
#pragma mark - 外界文件导入
- (BOOL)application:(UIApplication *)app openURL:(NSURL *)url options:(NSDictionary<UIApplicationOpenURLOptionsKey,id> *)options {
    
    if (url == nil) {
        return NO;
    }
    NSLog(@"文件地址 - %@", url);
    //具体存储或其他处理在这里进行
}
```

### 3. 文档预览

- 目前最常用的预览方式有两种，分别是 UIDocumentInteractionController 和 QLPreviewController

#### 3.1 UIDocumentInteractionController

- 不是真正的控制器，继承自 NSObject，使用需要设置代理
- 一次只能预览一个文件

```objective-c
//创建
UIDocumentInteractionController *documentVc = [UIDocumentInteractionController interactionControllerWithURL:url];
//设置代理 UIDocumentInteractionControllerDelegate
documentVc.delegate = self;
//显示
[documentVc presentPreviewAnimated:YES];
```

- 实现代理

```objective-c
- (UIViewController *)documentInteractionControllerViewControllerForPreview:(UIDocumentInteractionController *)controller{
    return self;
}

//不实现此代理，默认系统样式，带分享功能，实现时隐藏系统分享
- (UIView *)documentInteractionControllerViewForPreview:(UIDocumentInteractionController *)controller{
    return self.view;
}

- (CGRect)documentInteractionControllerRectForPreview:(UIDocumentInteractionController *)controller{
    return self.view.bounds;
}
```

#### 3.2 QLPreviewController

- 需导入 \#import <QuickLook/QuickLook.h>
- 一次可预览多个文件，显示效果与 UIDocumentInteractionController 一样

1. 正常样式，显示系统分享按钮，继承于 QLPreviewController

    ```objective-c
    @interface PreviewViewController ()<QLPreviewControllerDataSource, QLPreviewControllerDelegate>
    
    /// 预览文档地址
    @property (nonatomic, copy) NSString *filePath;
    @end
    
    @implementation PreviewViewController
    
    - (void)viewDidLoad {
        [super viewDidLoad];
        
        self.delegate = self;
        self.dataSource = self;
    }
    
    - (NSInteger)numberOfPreviewItemsInPreviewController:(QLPreviewController *)controller {
        return 1;
    }
    
    - (id <QLPreviewItem>)previewController:(QLPreviewController *)controller previewItemAtIndex:(NSInteger)index {
        
        return [NSURL fileURLWithPath:self.filePath];
    }
    
    //不打开文档内的链接
    - (BOOL)previewController:(QLPreviewController *)controller shouldOpenURL:(NSURL *)url forPreviewItem:(id<QLPreviewItem>)item {
        return NO;
    }
    
    @end
    ```

2. 隐藏样式，隐藏系统分享按钮

    ```objective-c
    QLPreviewController *previewVC = [[QLPreviewController alloc] init];
    previewVC.dataSource = self;
    previewVC.delegate = self;
    
    // 将QLPreviewControler  添加到控制器上，隐藏系统分享按钮
    [self addChildViewController:previewVC];
    [previewVC didMoveToParentViewController:self];
    previewVC.view.frame = CGRectMake(0, 0, self.view.frame.size.width, self.view.frame.size.height);
    [self.view addSubview:previewVC.view];
    
    //其他设置代理等操作与上面一致
    ```

- QLPreviewController 其他代理不再赘述，详细可查看具体 API