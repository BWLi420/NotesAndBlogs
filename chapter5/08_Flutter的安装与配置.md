# Mac 下 Flutter 的安装与配置

- Homebrew 安装与更新
- Flutter 的安装与配置

## Homebrew 的安装与更新

- Mac 电脑默认已安装 Homebrew，如不确定可先卸载并重新安装

    - 打开终端
    - 卸载

    ```shell
    /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
    ```

    - 安装

    ```shell
    /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
    ```

- 更新 Homebrew

    - 由于各种原因，官方更新地址速度比较慢，这里使用国内的镜像地址
    - 我使用的是清华大学的镜像，按以下命令依次执行，替换镜像地址

    ```shell
    cd "$(brew --repo)"
    git remote set-url origin https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/brew.git
    
    cd "$(brew --repo)/Library/Taps/homebrew/homebrew-core"
    git remote set-url origin https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/homebrew-core.git
    ```

    - 更新

    ```shell
    brew update
    ```

## Flutter 的安装与配置

- 这里依然替换为国内镜像，我这里放置在系统用户根目录下，使用的是清华大学的镜像地址
- 使用 cd 进入你要放置 Flutter 的位置，依次在终端里执行以下命令

```shell
export PUB_HOSTED_URL=https://mirrors.tuna.tsinghua.edu.cn/dart-pub/
export FLUTTER_STORAGE_BASE_URL=https://mirrors.tuna.tsinghua.edu.cn/flutter
git clone -b dev https://github.com/flutter/flutter.git
```

- 配置环境变量

    - 方法一：临时配置，仅在当前终端窗口可用

    ```shell
    export PATH="$PWD/flutter/bin:$PATH"
    ```

    - 方法二：全局配置，永久生效

    ```shell
    //我的 flutter 在用户根目录下，所以此时终端 cd 调整为根目录
    //打开，如不能打开就创建
    open -e .bash_profile
    
    //在打开的文件里配置镜像地址，文件中添加以下配置
    export PUB_HOSTED_URL=https://mirrors.tuna.tsinghua.edu.cn/dart-pub/
    export FLUTTER_STORAGE_BASE_URL=https://mirrors.tuna.tsinghua.edu.cn/flutter
    export PATH=/Users/(username替换为自己的地址)/flutter/bin:$PATH
    
    //更新环境配置
    source .bash_profile
    
    //检验是否配置成功，如果看到有 flutter 那就是配置好了
    echo $PATH
    ```

- 开发二进制文件预下载（可选操作）

    flutter 命令行工具会下载不同平台的开发二进制文件，如果需要一个封闭式的构建环境， 或在网络可用性不稳定的情况下使用等情况，需要通过下面这个命令预先下载 iOS 和 Android 的开发二进制文件：

    ```shell
    flutter precache
    ```

- 检测问题

```shell
//进入自己的 flutter 目录
cd ./flutter

//通过运行以下命令来查看当前环境是否需要安装其他的依赖
$ flutter doctor
```

- 解决缺少的依赖

    通过以上命令，输出会生成一份报告，你需要仔细阅读上述命令生成的报告，看看别漏了一些需要安装的依赖， 或者需要之后执行的命令，需要执行的命令会自动加粗显示，依次按照自己的需求执行即可。

    

    **当你安装了任一缺失部分的依赖后，可以再次运行 `flutter doctor` 命令来确认是否成功安装**

![示例图](https://upload-images.jianshu.io/upload_images/2997426-3957d0c8d1992e7b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

如上图我这里解决了 iOS 和 VSCode 的依赖，没有安装 Android studio 都会有详细提示。

至此 Mac 下的配置已经全部完成。