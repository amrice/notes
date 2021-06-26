本来想用parcel@next打包的，发现安装总是失败，用了代理也是一样，后来放弃改用vite，开发效率高，安装也快。下面用vite来构建Phaser游戏项目

## 安装vite

直接运行下面的命令，node版本要大于12

```shell
npm init @vitejs/app
```

根据提示选择`vanilla-ts`我们要用ts来写，命令执行过程中会要求你输入项目目录，执行完后进入目录安装依赖，加上Phaser。

## 准备静态资源

在项目的根目录创建一个public目录，将无需vite编译的静态资源，比如图片，xml文件等放进去。这样我们在代码中无需通过import语句来导入这些资源，可以直接引用它的地址，就像下面这样的

```typescript
this.load.bitmapFont('shortStack', '/shortStack.png', '/shortStack.xml');
```

这里的资源路径是相对于根目录的，因为vite最终会把public映射为根目录。

这里为啥不用import语句？因为当你用import时，打包工具会先处理你的资源，xml文件可能会被处理为字符串(raw)，或者xml对象，这会导致一些本来需要路径的函数执行失败。

## 游戏代码说明

游戏设置有大级和小级，大级从A开始，小级从a开始，一个大级包含4个小级，就是说当小级涨到d后，大级涨一级到B。



**游戏入口**

```typescript
const scaleConfig: ScaleConfig = {
    mode: Scale.ScaleModes.FIT, //TODO： 当要调试坐标时，改为NONE
    autoCenter: Scale.CENTER_BOTH,
    parent: 'app',
    width: 640,
    height: 960
};

const gameConfig: Types.Core.GameConfig = {
    type: Phaser.AUTO,
    scale: scaleConfig,
    scene: [PreloadAssetsScene, PlayGameScene]
};

const app = new Phaser.Game(gameConfig);
console.log(app);
```

Phaser的入口文件需要创建一个Phaser.Game实例，需要我们传入游戏配置和缩放配置相关信息。

scaleConfig中的parent需要设置为页面中的元素或其id，vite生成的项目默认是app

还有mode属性这里设置的是FIT，游戏界面会根据浏览器窗口自适应，所以显示出来的游戏对象不是原始尺寸，当你要调试的时候，可以将其设置为NONE。

游戏定义了两个场景，具体场景直接的衔接会在具体的场景实现，这里只是告诉Game类游戏有几个场景，Game类不负责场景的切换和衔接。



**游戏角色(正方形)**

```typescript
import { GameObjects } from 'phaser';
import Scene = Phaser.Scene;

export class Square extends GameObjects.Sprite {

    static INIT_VALUE:number = 97;
    static MAX_VALUE:number = 97+4;

    #successful:number = 0;

    constructor(
        scene:Scene, x:number, y:number,
        key:string, scale:number
    ) {
        super(scene, x, y, key);
        this.#successful = Square.INIT_VALUE;
        this.setScale(scale);
    }

    increaseSuccessful() {
        this.#successful++;
        if (this.#successful > Square.MAX_VALUE) {
            this.#successful = Square.INIT_VALUE;
        }
    }

    resetSuccessful() {
        this.#successful = Square.INIT_VALUE;
    }

    getSuccessful():number {
        return this.#successful;
    }

}
```

定义了一个私有属性，用于保存当前小级是多少。

**主要游戏场景**

```typescript
export class PlayGameScene extends Scene {

    stop() {
        if (this.currentMode === AppData.AppMode.GROWING) {
            this.currentMode = AppData.AppMode.IDLE;
            // 停止放大和来回转动动画
            this.growTween.stop();
            this.rotateTween.stop();
            // 将正方形角度调正
            this.rotateTween = this.tweens.add({
                targets: this.squareTweenTargets,
                angle: 0,
                duration: 300,
                ease: 'Cubic.easeOut',
                callbackScope: this,
                onComplete: () => {
                    // 播放下落动画之前判断是否掉入洞内还是可以回弹
                    const holeWidth = this.rightBaseSquare.x - this.leftBaseSquare.x;
                    const wallWidth = this.rightTopSquare.x - this.leftTopSquare.x;
                    // 掉入洞中
                    if (this.heroSquare.displayWidth <= holeWidth) {
                        this.dropToHole(this.heroSquare.displayHeight);
                    } else {
                        // 可以放入
                        if (this.heroSquare.displayWidth <= wallWidth) {
                            this.fallAndBounce(true);
                        } else {
                            // 超过了上方的两个长方形之间的距离
                            this.fallAndBounce(false);
                        }
                    }
                }
            });
        }
    }
}

// 来回转动
this.tweens.add({
    targets: this.squareTweenTargets,
    angle: 40,
    duration: 300,
    yoyo: true,
    repeat: -1
});

// 缩放动画
this.tweens.add({
    targets: this.squareTweenTargets,
    scaleX: 1,
    scaleY: 1,
    duration: AppData.AppOptions.growTime
});

// 回弹动画
this.tweens.add({
    targets: this.squareTweenTargets,
    y: destY,
    duration: 600,
    ease: 'Bounce.easeOut',
    callbackScope: this,
    onComplete: () => {}
});

// 重新开始
this.time.addEvent({
    delay: 1000,
    callbackScope: this,
    callback: () => {
        this.scene.start('playGame');
    }
});
```

## 总结

上面只留了一些关键的逻辑代码，以作备用。因为使用的字体中没有数字，所以这里的级别改用了字母A和a。



1、我们可以通过场景Scene的load对象来加载资源，包括图片，字体文件等

```typescript
// 加载图片
this.load.image('base', '/base.png');
// 加载位图字体
this.load.bitmapFont('shortStack', '/shortStack.png', '/shortStack.xml');
```

2、设置BitmapText的文本颜色是调用setTint(0x000000)方法。

3、场景之间的衔接，我们可以在前一个创建的create函数中的最后调用

```typescript
// key是定义场景类时需要传入构造函数的字符串，唯一标识该场景
this.scene.start('下一场景的key'); 

// super({key: 'preloadAssets'});
```

4、setOrigin(x,y)方法可以将游戏对象的坐标原点设置到相对于其本身的某个点上，这样在计算其坐标的时候，有时可以省去高度宽度的掺和。

5、想获取游戏界面(canvas)的宽度和高度，我们可以从游戏配置中获取

```typescript
this.game.config.width
```

6、要设置游戏界面的背景色，我们可以从摄像头来设置

```typescript
this.cameras.main.setBackgroundColor(0x000000);
```

7、要添加鼠标的点击和释放事件监听，可以通过input对象

```typescript
this.input.on('pointerdown', this.grow, this);
this.input.on('pointerup', this.stop, this);
```

8、displayWidth与width的区别在于前者会将缩放旋转等因素考虑进去。当游戏对象经过缩放后要获取其尺寸就可以用display相关属性。

9、这里需要注意的是各种动画的实现，都是用的tweens这个对象，来回转动可以设置yoyo=true来实现，回弹可以设置ease= 'Bounce.easeOut'来实现。

10、在每个动画的onComplete回调中这里都用了箭头函数，如果是普通函数你可能需要处理ts的类型错误，还有这里设置为箭头函数，属性callbackScope应该是可以不用设置了。

11、tweens动画需要我们设置的是目标状态，比如目标位置，目标角度等，所以动画的倒序播放其实只要你把目标位置设置为反方向即可。

12、还有一个延迟的实现是用了time对象，通过addEvent函数来配置延迟时间即可，比如设置delay为1000，即延迟1s，addEvent也有回调callback和回调作用域callbackScope的设置，这里我们一样用箭头函数处理。

13、Utils.Array.GetRandom可以从一个数组中随机获取一个元素。



**运行效果**

![image-20210626124612742](https://cdn.jsdelivr.net/gh/ywxgod/image_source/imgs20210626124612.png)

## 打包问题

用vite构建生产包的时候，可能会出现下面的ts错误：

```shell
> build
> tsc && vite build

node_modules/phaser/types/phaser.d.ts:9452:54 - error TS2304: Cannot find name 'ActiveXObject'.

9452         function ParseXML(data: string): DOMParser | ActiveXObject;
                                                          ~~~~~~~~~~~~~


Found 1 error.

```

我们需要修改vite默认生成的tsconfig.json中的lib，即增加`ScriptHost`

```json
"lib": ["ESNext", "DOM", "ScriptHost"],
```

ts还可能报下面的错误

![image-20210626124959859](https://cdn.jsdelivr.net/gh/ywxgod/image_source/imgs20210626124959.png)

我们可以将saveData声明为

```typescript
saveData: SaveData|null = null;
```

但这感觉太麻烦，因为太多这样的变量了。后来折中处理，设置strictNullChecks为false，但strict还是为true

```json
{
  "compilerOptions": {
    "target": "ESNext",
    "module": "ESNext",
    "lib": ["ESNext", "DOM", "ScriptHost"],
    "moduleResolution": "Node",
    "strict": true,
    "strictNullChecks": false,
    "sourceMap": true,
    "resolveJsonModule": true,
    "esModuleInterop": true,
    "noEmit": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noImplicitReturns": true
  },
  "include": ["./src"]
}
```

## 参考

https://www.emanueleferonato.com/2021/06/16/html5-prototype-of-ios-game-perfect-square-built-with-phaser-using-typescript-build-a-physics-game-using-only-tweens/
