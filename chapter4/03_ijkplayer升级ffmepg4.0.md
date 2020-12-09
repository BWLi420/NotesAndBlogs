## ijkplayer升级ffmepg4.0及相关设置

> 编译及https支持不再赘述，可以查看之前的文章 [ijkPlayer编译、打包、多格式及Https支持](https://www.jianshu.com/p/4419111de807)

### 1. 升级ffmepg4.0

- 找到 **init-ios.sh** 文件，打开并修改 **IJK_FFMPEG_COMMIT** 选项，可以到 [https://github.com/bilibili/FFmpeg/releases](https://github.com/bilibili/FFmpeg/releases) 查看最新tag

- 比如我这里使用了 ff4.0--ijk0.8.8--20201130--001

    ![image-20201207175520799](https://tva1.sinaimg.cn/large/0081Kckwly1glffyojhrkj30fa02o3yt.jpg)

### 2. 升级ffmepg4.0的注意事项

- 修改自己使用的 **module.sh** 文件
    - 注释 export COMMON_FF_CFG_FLAGS="$COMMON_FF_CFG_FLAGS --disable-vda"
    - 注释 export COMMON_FF_CFG_FLAGS="$COMMON_FF_CFG_FLAGS --disable-ffserver"
    - 添加 export COMMON_FF_CFG_FLAGS="$COMMON_FF_CFG_FLAGS --disable-bsf=eac3_core"

### 3. 更改openssl版本

- 找到 **init-ios-openssl.sh** 文件，打开并修改 **IJK_OPENSSL_COMMIT** 选项，可以到 [https://github.com/bilibili/openssl/releases](https://github.com/bilibili/openssl/releases) 查看最新tag

- 比如我这里使用了 OpenSSL_1_0_2u

    ![image-20201207175840424](https://tva1.sinaimg.cn/large/0081Kckwly1glfg23bzqgj30es02vjro.jpg)

### 4. 配置最低编译版本

- 找到 **ios/tools/do-compile-ffmpeg.sh** 文件，打开并修改 **FF_XCRUN_OSVERSION** 选项，可根据需求修改不同架构下的最低版本

- 比如我这里将 arm64 下最低版本设为 iOS9.0

    ![image-20201207180800236](https://tva1.sinaimg.cn/large/0081Kckwly1glfgbsj9j6j30ga04p752.jpg)

- openssl 修改同理，在 **do-compile-openssl.sh** 文件中更改

### 5. 设置编译平台

- 找到 **ios/compile-ffmpeg.sh** 文件，打开并修改 **FF_ALL_ARCHS_IOS8_SDK** 选项，目前iOS已经不支持armv7，可直接删除，其他可根据需求设置需要的平台

- 比如我这里只编译了 arm64

    ![image-20201207182025620](https://tva1.sinaimg.cn/large/0081Kckwly1glfgopz24rj30ev03omxj.jpg)

- openssl 修改同理，找到 **compile-openssl.sh** 文件并更改对应的 **FF_ALL_ARCHS_IOS8_SDK** 选项

