# 编译环境

  macOS + Xcode

# 准备工作

  ```shell
  # 安装 homebrew、 git、 yasm (如果已经安装好可以跳过, 不清楚的可以再来一遍)
  # 依次执行以下命令
  ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
  brew install git
  brew install yasm
  ```

# 获取 ijkPlayer 源码

  - 桌面新建一个文件夹 ijkplayer，位置自行决定，我这里文件放到桌面了

  - 打开终端、依次输入以下指令

    ```shell
    # 进入新建的文件夹内
    cd ~/Desktop/ijkplayer/
    
    # 获取 ijkplayer 源码
    git clone https://github.com/Bilibili/ijkplayer.git ijkplayer-ios
    
    # 进入源码目录
    cd ijkplayer-ios
    
    # 切换分支 (目前为k0.8.8, 可以自行去GitHub查看最新版本号)
    git checkout -B latest k0.8.8
    ```

# 配置编解码器多格式支持

  - 默认编解码为 module-lite.sh，格式最少，如果足够使用，可以跳过这一步

  - 配置文件在 config 目录下
    - **module-default.sh** 最多的编解码器/格式（打包时体积最大）
    - **module-lite-hevc.sh** 较少的编解码器/格式(包括hevc)
    - **module-lite.sh** 较少的编解码器/格式(默认情况)

  - 依次执行以下命令，将默认编译的 module-lite.sh 更改为 module-default.sh

    ```shell
    # 进入 config 目录
    cd config
    
    # 删除当前的 module.sh 文件
    rm module.sh
    
    # 可根据需要替换为 `module-default.sh`, `module-lite-hevc.sh`, `module-lite.sh`
    # 创建软链接 module.sh 指向 module-default.sh
    ln -s module-default.sh module.sh
    
    cd ..
    cd ios
    sh compile-ffmpeg.sh clean
    ```

# 获取 ffmpeg

  - 依次执行以下命令

  - 速度比较慢，耐心等待（推荐一个梯子，GitHub 搜索 xx-net，你懂的）

    ```shell
    # 回到根目录
    cd ..
    
    # 初始化 ffmpeg
    ./init-ios.sh
    ```

# 添加 Https 支持

  - 依次执行以下命令

    ```shell
    # 获取 openssl 并初始化
    ./init-ios-openssl.sh
    
    cd ios
    
    # 在模块文件 module.sh 中添加配置 以启用 openssl 组件
    echo 'export COMMON_FF_CFG_FLAGS="$COMMON_FF_CFG_FLAGS --enable-openssl"' >> ../config/module.sh
    
    ./compile-ffmpeg.sh clean
    ```

# 编译 openssl 和 ffmpeg

  - 编译 openssl，如果不需要 https 可以跳过这一步

    ```shell
    ./compile-openssl.sh all
    ```

    > 如果编译 openssl 时报错：`xcrun: error: SDK "iphoneos" cannot be located`，先执行以下命令，再编译 openssl

    ```sh
    # 指定 Xcode 路径，如果有多个 Xcode 可以改成自己需要的版本路径，这里我用的是 Xcode9.1
    sudo xcode-select --switch /Applications/Xcode9.1.app/Contents/Developer/
    ```

  - 编译 ffmpeg

    ```shell
    ./compile-ffmpeg.sh all
    ```

    > 如果编译 ffmpeg 时报错：`error: unknown directive .arch armv7-a`，这是因为最新的 Xcode9.3 弱化了对 32 位的支持，解决方式有两种：
    >
    > 1. 可以在 `compile-ffmpeg.sh` 中删除 armv7，修改为 `FF_ALL_ARCHS_IOS8_SDK="arm64 i386 x86_64"` ；
    > 2. 使用之前的 Xcode 版本，这里我用的是 9.1 可以顺利编译，[点这里下载](https://developer.apple.com/download/more/) ，别忘了指定 Xcode 路径

# 打开 IJKMediaPlayer 项目

  - 手动打开 ios 目录下 IJKMediaPlayer 或者使用命令

    ```shell
    open IJKMediaPlayer/IJKMediaPlayer.xcodeproj
    ```

  - 添加 openssl 相关包以支持 Https，若不需要 Https 则忽略此步

    ![openssl 相关包路径](https://upload-images.jianshu.io/upload_images/2997426-2f7b85c38646b5c5.jpg)

    - 推荐使用 IJKMediaFramework 这个 target

    ![导入 openssl 相关包](https://upload-images.jianshu.io/upload_images/2997426-43b73ac575dafaa7.png)

# 打包 Framework

  - 这里以 Release 模式为例，需要 Debug 版操作同理

    - 配置 Release 模式

      ![配置 Release 模式](https://upload-images.jianshu.io/upload_images/2997426-c9a08581a196827b.png)

      ![配置 Release 模式](https://upload-images.jianshu.io/upload_images/2997426-d62b195054e6fcf0.png)

    - 打包模拟器 framework

      ![打包模拟器 framework](https://upload-images.jianshu.io/upload_images/2997426-b384ffb04731c1e5.png)

    - 打包真机 framework

      ![打包真机 framework](https://upload-images.jianshu.io/upload_images/2997426-e77c90a10f4a0e79.png)

      > 如果之前的步骤删除了 `compile-ffmpeg.sh` 中 `armv7`, 可能会报错，直接把报错的 armv7 注释掉就好

    - 合并 framework，方便模拟器和真机都可以使用

      - 打开 framework 文件夹

        ![打开 framework 文件夹](https://upload-images.jianshu.io/upload_images/2997426-bf091a49508ede88.png)

        ![有真机和模拟器两个包](https://upload-images.jianshu.io/upload_images/2997426-acbd0a3023defdb4.png)

      - 合并操作

        1. 打开终端 cd 到 Products 目录下

        2. 执行以下命令完成合并

           ```shell
           lipo -create Release-iphoneos/IJKMediaFramework.framework/IJKMediaFramework Release-iphonesimulator/IJKMediaFramework.framework/IJKMediaFramework -output IJKMediaFramework
           ```

        3. 替换真机 IJKMediaFramework 得到最终需要的包

           ![替换包](https://upload-images.jianshu.io/upload_images/2997426-2fccb9979d87ba73.png)

# 导入 framework 到项目中

  - 将得到的包拖到项目中，注意勾选 `Copy items if needed` 和 对应的 `target`

  - 添加以下依赖

    ```shell
    libc++.tbd
    libz.tbd
    libbz2.tbd
    AudioToolbox.framework
    CoreGraphics.framework
    AVFoundation.framework
    CoreMedia.framework
    CoreVideo.framework
    MediaPlayer.framework
    MobileCoreServices.framework
    OpenGLES.framework
    QuartzCore.framework
    VideoToolbox.framework
    ```

       ![添加依赖库](https://upload-images.jianshu.io/upload_images/2997426-70f9d2b0ed9e1266.png)

  - 在合适的位置导入头文件

    ```shell
    #import <IJKMediaFramework/IJKMediaPlayer.h>
    ```

  - command + b 编译成功，即集成成功。

# 附上我打包的文件，需要的话请自行下载

[百度云下载](https://pan.baidu.com/s/1HZMJlzfw6roP6hnjGpehjA) 密码：k5wl

