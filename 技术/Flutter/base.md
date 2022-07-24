marktext 在M1上安装后，启动报错修复：xattr -cr /Applications/MarkText.app

##### **Flutter产出的App大小最少都有5M，有以下组成：**

A minimal Flutter app has to hold the core engine (circa 3 MB), framework + app code
(circa 1 MB), and other files (circa 1 MB), meaning a basic Flutter app starts at 5 MB.

##### **各种APP的底层架构图：**

<img src="https://cdn.jsdelivr.net/gh/ywxgod/image_source/md-imgs202206261102423.png" title="" alt="" width="270">  <img title="" src="https://cdn.jsdelivr.net/gh/ywxgod/image_source/md-imgs202206261103294.png" alt="" width="252">

<img src="https://cdn.jsdelivr.net/gh/ywxgod/image_source/md-imgs202206261104253.png" title="" alt="" width="271">  <img title="" src="https://cdn.jsdelivr.net/gh/ywxgod/image_source/md-imgs202206261106646.png" alt="" width="254">

#### **命令行启动模拟器：**

可以先通过android studio来安装必要的模拟器，也可以单独安装，安装后的路径：

```shell
/Users/wyin/Library/Android/sdk/emulator/
```

将其加入到环境变量中，如：.zshrc

```shell
export PATH="$PATH:/Users/wyin/Library/Android/sdk/emulator/"
```

通过source命令重新加载环境变量

```shell
source ./.zshrc
```

#### **通过命令行启动flutter：**

先选择必要的channel，默认是stable

```shell
flutter channel
flutter channel dev/beta
flutter upgrade
```

启动模拟器

```shell
# 列出已创建好的模拟器名称
emulator -list-avds 
# 启动对应的模拟器
emulator -avd <name>
# 列出flutter找到的设备，包括模拟器
flutter devices
# 将项目代码发送到对应的模拟器执行
flutter run -d <emulatorName>
```

有了上面的两个步骤，现在可以不用依赖android studio来编辑flutter项目了，用自己喜欢的IDE打开flutter代码，然后通过命令行来执行就好了。

除了直接通过emulator来启动模拟器，还可以通过`flutter emulators --launch xxxx` 来启动。

##### 利用flutter官方自带的示例代码创建项目

通过下面的命令获取到官方提供的示例代码相关信息

```shell
flutter create --list-samples=xxx.json
```

上面的命令会将官方提供的示例代码信息下载到xxx.json文件中，此文件的信息类似于下面这样的

```json
{
    "id": "material.Focus.2",
    "element": "Focus",
    "sourcePath": "lib/src/widgets/focus_scope.dart",
    "sourceLine": 111,
    "channel": "master",
    "serial": "2",
    "package": "flutter",
    "library": "material",
    "copyright": null,
    "description": "This example shows how to wrap another widget in a [Focus] widget to make it\nfocusable. It wraps a [Container], and changes the container's color when it\nis set as the [FocusManager.primaryFocus].\n\nIf you also want to handle mouse hover and/or keyboard actions on a widget,\nconsider using a [FocusableActionDetector], which combines several different\nwidgets to provide those capabilities.",
    "file": "material.Focus.2.dart"
},
{
    "id": "material.AnimatedAlign.1",
    "element": "AnimatedAlign",
    "sourcePath": "lib/src/widgets/implicit_animations.dart",
    "sourceLine": 897,
    "channel": "master",
    "serial": "1",
    "package": "flutter",
    "library": "material",
    "copyright": null,
    "description": "The following code implements the [AnimatedAlign] widget, using a [curve] of\n[Curves.fastOutSlowIn].",
    "file": "material.AnimatedAlign.1.dart"
},
```

上面每一个id即表示一个示例项目，通过下面的命令可以在本地创建对应的项目

```shell
flutter create --sample=[id] 目录名称
```

这样就会将id对应的项目下载到本地的【项目目录】中了。
