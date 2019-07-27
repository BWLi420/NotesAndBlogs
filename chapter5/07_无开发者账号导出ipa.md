# iOS 无付费账号导出 ipa 包

## 一、手动生成 ipa

1. Xcode 编译 --> Products --> your.app --> Show in Finder 找到你编译的App

![Show in Finder](https://upload-images.jianshu.io/upload_images/2997426-4c8f8accc8089563.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

2. 在桌面创建 Payload 文件夹（***注意：名称一定不能错***）
3. 将 your.app 拷贝到 Payload 文件夹下
4. 压缩 Payload 文件夹为 zip
5. 将 .zip 后缀更改为 .ipa，这样 ipa 包就生成好了

## 二、iTunes 导出

>  iTunes 从 12.7 开始已经移除了 “应用” 选项，后续无法通过 iTunes 管理手机 App，因此，我们需要将 iTunes 进行降级

1.  iTunes 作为 macOS 的内置应用，我们需要先将其删除

    - 关闭 System Integrity Protection（SIP）

    ```
    1、重启Mac；
    
    2、开机，同时按住 command+R ，进入Recovery模式；
    
    3、在实用工具中选择 终端，输入 csrutil disable
    
    4、重启Mac，打开终端，输入 csrutil status，显示 System Integrity Protection status：disabled ，则 SIP 已关闭。
    
    //
    删除 iTunes 后，如需重新开启 SIP，在第三步执行 csrutil enable 重启 Mac 即可
    ```

    - 删除 iTunes

    ```shell
    // 打开终端，执行以下命令，删除iTunes
    // 期间需要输入您的密码
    
    sudo rm -rf /Applications/iTunes.app
    ```

2. 下载 iTunes 12.6.2，官网下载地址：[iTunes 12.6.2](https://support.apple.com/zh_CN/downloads/itunes)

3. 解压安装并打开

```shell
// 1. 如果安装后打开提示 iTunes Library.itl 由高版本创建，资料库提示不能打开
// 解决方法：
// 在 /Users/../Music/iTunes 中搜索 iTunes Library.itl 并删除

// 2. 如果打开提示 “iTunes.app” 的这个版本不能与此版本的 macOS 配合使用
// 解决方法
// 在 Applications 中找到 iTunes 右键显示包内容，找到 Info.plist 和 version.plist 复制到桌面，打开找到版本号 12.6.2 修改为高版本并保存，将修改后的两个文件替换掉原有文件
```

4. 关闭 iTunes 的自动更新

```shell
// 终端下执行

sudo softwareupdate --ignore iTunesXPatch
```

5. 打开 iTunes ，将编译的 .app 文件拖入 iTunes，再拖出到桌面即可

![选择应用页.png](https://upload-images.jianshu.io/upload_images/2997426-2868a8484227068c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 三、Archive 导出

- Archive 导出 ipa 与前两种方式一样

Show in Finder --> 显示包内容 --> Products --> Applications 找到 .app 文件，后续与前两种方式相同。



## 注意

**这样导出的 ipa 包是不能直接安装到手机上的，主要适用于后期企业签名使用。**


