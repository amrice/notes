场景，类似于舞台剧的场景，一个舞台剧可能有多个场景，演完一场转接下一场。抽象到游戏中来是一样的道理，在游戏中也可以有多个场景，Phaser中的场景是Phaser.Scene类的实例，下面是一些使用注意点。



![image-20220422205542941](https://cdn.jsdelivr.net/gh/ywxgod/image_source/imgs20220422205606.png)

从文档截图看，scene有三个钩子函数我们可能需要关注。

init 会在preload与create之前由scene manager调用。The typical flow for a Phaser Scene is that you load assets in the Scene's `preload` method and then when the Scene's `create` method is called you are guaranteed that all of those assets are ready for use and have been loaded.