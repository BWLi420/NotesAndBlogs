> iOS 应用内调用并发送邮件

- 目前有两种方法 mailto: 和 MFMailComposeViewController

### 1. 使用 mailto:

- 会跳转到应用外并打开系统邮件

```objective-c
NSURL *url = [NSURL URLWithString:@"mailto:yourEmail"];
if ([[UIApplication sharedApplication] canOpenURL:url]) {
    [[UIApplication sharedApplication] openURL:url options:@{} completionHandler:nil];
} else {
    NSLog(@"打开邮箱出现错误");
}
```

### 2. MFMailComposeViewController

- 该方式需要已设置好邮件账户，否则无法打开
- 可设置主题、收件人、抄送人、发送内容
- 首先需要导入 \#import <MessageUI/MFMailComposeViewController.h>

```objective-c
#pragma mark - 发送邮件
- (void)sendEmail {
    
    MFMailComposeViewController *mailVC = [MFMailComposeViewController new];
    if (!mailVC) {
        // 在设备还没有添加邮件账户的时候，为空
        NSLog(@"暂未设置邮箱账户，请先到系统设置添加账户");
        return;
    }
    
  	//代理 MFMailComposeViewControllerDelegate
    mailVC.mailComposeDelegate = self;
    //邮件主题
    [mailVC setSubject:@"反馈/建议"];
    //收件人
    [mailVC setToRecipients:@[@"yourEmail"]];
    
    [self presentViewController:mailVC animated:YES completion:nil];
}

// 实现代理方法
- (void)mailComposeController:(MFMailComposeViewController *)controller didFinishWithResult:(MFMailComposeResult)result error:(NSError *)error {
    // MFMailComposeResultCancelled
    // MFMailComposeResultSaved
    // MFMailComposeResultSent
    // MFMailComposeResultFailed
  
    if (result == MFMailComposeResultSent) {
      	NSLog(@"发送成功");
    } else if (result == MFMailComposeResultFailed) {
      	NSLog(@"发送失败");
    }
    
    [controller dismissViewControllerAnimated:YES completion:nil];
}
```

