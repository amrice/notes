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
