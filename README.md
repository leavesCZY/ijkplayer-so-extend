### 一、前言

公司的一个项目新项目涉及到音频播放的内容，音频格式从常见的 **mp3** 到冷门的**无损音乐 ape** 都有，琢磨了好久，最后选中了 B 站的开源库 **ijkplayer** ，可是 **ijkplayer** 提供的默认 so 库并不支持无损音乐，而我在网上找了许久也没有找到其他人编译的现成扩展库，就只能自己来动手了，折腾了许久总算是搞定了，此处我就写下自己的实现步骤以帮助后人，文章末尾也提供了已编译好的 so 库，需要的读者请自取

开发环境：Linux （Deepin）
NDK 版本：android-ndk-r14b
ijkplayer 版本：k0.8.8

### 二、准备工作

#### 2.1、安装 NDK

这里我选择使用的 NDK 版本是 `android-ndk-r14b-linux-x86_64` ，将它先移动到 usr 文件夹下新建的 android 文件夹内

![](https://upload-images.jianshu.io/upload_images/2552605-2a0dc0ab92fe6ffd.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

解压 ndk 压缩包到 android 目录下

![](https://upload-images.jianshu.io/upload_images/2552605-de4a1689b2800d5f.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 2.2、安装 Android SDK

我这里是选择安装 `Android Studio` 来使用其附带的 `Android SDK` ，理论上单独只安装  `Android SDK` 应该也是可以的。我使用的 `Android Studio` 版本是 **android-studio-ide-191.5791312-linux** 

将压缩包移动到 `usr/android` 文件夹内，并解压

![](https://upload-images.jianshu.io/upload_images/2552605-dd9e39d30a14c303.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

启动 Android Studio ，之后就是常规的一些配置了，将 SDK 安装在当前新建的 `android-sdk` 目录下

![](https://upload-images.jianshu.io/upload_images/2552605-025bdfc4f7cca23f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 2.3、安装 Git

安装 Git 用于下载及更新 `ijkplayer` 以及其依赖库

```shell
  sudo apt-get update //或许需要
  sudo apt-get install git
```

![](https://upload-images.jianshu.io/upload_images/2552605-4ee553a58096ab4e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 2.4、ijkplayer 

下载 **ijkplayer** 源码，并切换到当前最新的 **k0.8.8** 分支下

```shell
git clone https://github.com/Bilibili/ijkplayer.git
git checkout -B latest k0.8.8
```

### 四、开始编译

#### 4.1、配置编译脚本

ijkplayer 的 config 文件夹内的 **module-default.sh** 文件配置了编译默认支持的音频格式，enable 代表启用，disable 代表禁用，根据项目需要修改 **module-default.sh** 文件后，将之作为当前的配置脚本

```
sudo rm module.sh
ln -s module-default.sh module.sh
source module.sh
```

#### 4.2、初始化

在 ijkplayer 源代码目录下运行以下命令进行初始化以便下载一些依赖库，网速原因可能需要很久

```shell
sudo ./init-android.sh
```

![](https://upload-images.jianshu.io/upload_images/2552605-7e0c01548dc98432.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 4.3、支持 https

ijkplayer 默认不支持 https 链接，需要运行以下命令使其支持

```shell
./init-android-openssl.sh
```

#### 4.4、编译 ffmpeg

```shell
cd android/contrib
./compile-ffmpeg.sh clean
./compile-ffmpeg.sh all
```

![1566726355456](C:\Users\CZY\AppData\Roaming\Typora\typora-user-images\1566726355456.png)

如果在运行了以上命令后控制台提示报了如下提示，那么这是因为 **compile-ffmpeg.sh** 不知道当前系统安装的 NDK 路径导致的

```shell
ANDROID_NDK=
You must define ANDROID_NDK before starting.
They must point to your NDK directories.
```

可以通过在 `ijkplayer/android/contrib/tools/do-compile-ffmpeg.sh` 文件的开头定义 NDK 路径来解决

```shell
ANDROID_NDK=/usr/android/android-ndk-r14b
```

之后，重新运行命令即可

#### 4.5、编译 ijkplayer

```shell
cd ..
./compile-ijk.sh all
```

报错的话一样需要在 `compile-ijk.sh` 文件头部定义 Android SDK 和 NDK 的路径

如果编译成功，则会在 android 文件夹内生成一个 ijkplayer 工程，当中包含了各个架构的 so 库，到此那就大功告成了

![](https://upload-images.jianshu.io/upload_images/2552605-d5705a1e42a8b124.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

将各个 CPU 架构下对应的 so 库拷贝到自己的项目中，至此就获得了所需要的扩展 so 库

![](https://upload-images.jianshu.io/upload_images/2552605-10cfdaa2fd1954b6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 五、so 库下载

编译好的 so 库我已经上传到 GitHub 了，项目主页：[ijkplayer-so-extend](https://github.com/leavesC/ijkplayer-so-extend) （https://github.com/leavesC/ijkplayer-so-extend）。经测试，可以正常播放 wav 、ape 等格式的音频

本文已收录至我的学习笔记合辑：[JavaKotlinAndroidGuide](https://github.com/leavesC/JavaKotlinAndroidGuide)