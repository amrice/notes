本来想用parcel@next打包的，发现安装总是失败，用了代理也是一样，后来放弃改用vite，开发效率高，安装也快。下面用vite来构建Phaser游戏项目，游戏原型来自：

https://www.emanueleferonato.com/2021/06/16/html5-prototype-of-ios-game-perfect-square-built-with-phaser-using-typescript-build-a-physics-game-using-only-tweens/

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
import './style.css'

import {Types, Scale} from 'phaser';
import ScaleConfig = Phaser.Types.Core.ScaleConfig;
import {PreloadAssetsScene} from "./app/PreloadAssetsScene";
import {PlayGameScene} from "./app/PlayGameScene";

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

scaleConfig中的parent需要设置为页面中的元素或其id，vite生成的项目默认是app

还有mode属性这里设置的是FIT，游戏界面会根据浏览器窗口自适应，所以显示出来的游戏对象不是原始尺寸，当你要调试的时候，可以将其设置为NONE。

游戏定义了两个场景，具体场景直接的衔接会在具体的场景实现，这里只是告诉Game类游戏有几个场景，Game类不负责场景的切换和衔接。

**游戏配置**

```typescript
import { Utils }  from 'phaser';

type SaveData = { level: number };

export {SaveData};

export class AppData {

    static AppOptions = {
        bgColors: [0x62bd18, 0xff5300, 0xd21034, 0xff475c, 0x8f16b2, 0x588c7e, 0x8c4646],
        holeWidthRange: [80, 260],
        wallRange: [10, 50],
        growTime: 1500,
        localStorageKey: 'level'
    };

    static AppMode = {
        IDLE: 0,
        WAITING: 1,
        GROWING: 2
    };

    static AppTexts = {
        level: 'level',
        infoLines: ['tap and hold to grow', 'release to drop'],
        landHere: 'land here',
        success: 'Yeah!!!',
        failure: 'On no!!'
    };

    static getSaveData() {
        const defaultData: SaveData = { level: 65 };
        let result:SaveData;
        let dataStr:string = localStorage.getItem(this.AppOptions.localStorageKey);
        if (!dataStr) {
            return defaultData;
        }
        try {
            result = JSON.parse(dataStr) as SaveData;
        } catch (e) {
            console.log(e);
        }
        return result||defaultData;
    }

    static updateSaveData(level: number) {
        const saveData: SaveData = { level };
        localStorage.setItem(this.AppOptions.localStorageKey, JSON.stringify(saveData));
    }

    static getTintColor() {
        const PhaserArray = Utils.Array;
        return PhaserArray.GetRandom(this.AppOptions.bgColors);
    }
}
```

这里只将大级存到了本地，当刷新页面时会读取上一次的大级，小级从a开始计。

**白色的墙体**

```typescript
import { GameObjects, Scene, Math } from 'phaser';

export class Wall extends GameObjects.Sprite {

    constructor(
        scene:Scene, x:number, y:number,
        key: string, origin: Math.Vector2
    ) {
        super(scene, x, y, key);
        this.setOrigin(origin.x, origin.y);
    }

    tweenTo(x:number) {
        this.scene.tweens.add({
            targets: this,
            x,
            duration: 500,
            ease: 'Cubic.easeOut'
        });
    }

}
```

放置于左下角和右下角的四块矩形

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

**角色中的文本**

```typescript
import { GameObjects, Scene } from 'phaser';

export class SquareText extends GameObjects.BitmapText {

    constructor(
        scene:Scene, x:number, y:number,
        font:string, text:string, size:number,
        tintColor: number
    ) {
        super(scene, x, y, font, text, size);
        this.setOrigin(0.5);
        // this.setScale(0.4);
        this.setTint(tintColor);
    }

    updateText(text:string) {
        this.setText(text);
    }

}
```

接下来是定义两个游戏场景，一个负责资源加载逻辑，一个负责游戏的主逻辑

**资源加载的场景**

```typescript
import { Scene } from 'phaser';

export class PreloadAssetsScene extends Scene {

    constructor() {
        super({key: 'preloadAssets'});
    }

    preload() {
        // TODO: 这些静态资源的路径最好放在公开的静态资源目录
        // 用vite，parcel打包时phaser可能会报错误，特别是xml文件
        // bitmapFont函数需要一个路径，如果用打包工具加载就会被解析成xml或者字符串导致报错。
        this.load.image('base', '/base.png');
        this.load.image('square', '/square.png');
        this.load.image('top', '/top.png');
        this.load.bitmapFont('shortStack', '/shortStack.png', '/shortStack.xml');
    }

    create() {
        this.scene.start('playGame');
    }

}
```

主要游戏场景

```typescript
import { Scene, Math, GameObjects } from 'phaser';
import {AppData, SaveData} from "./AppData";
import {Wall} from "./Wall";
import {Square} from "./Square";
import {SquareText} from "./SquareText";

import BitmapText = Phaser.GameObjects.BitmapText;
import Tween = Phaser.Tweens.Tween;

export class PlayGameScene extends Scene {


    // canvas宽高
    gameWidth: number = 0;
    gameHeight: number = 0;
    // canvas背景色
    tintColor:number = 0x000000;
    // 存储级别到本地，下次从上次的级别开始
    saveData: SaveData;
    // canvas底部的四个矩形
    leftBaseSquare: Wall;
    rightBaseSquare: Wall;
    leftTopSquare: Wall;
    rightTopSquare: Wall;
    // 正中的矩形和矩形里面的文本
    heroSquare: Square;
    heroSquareText: SquareText;
    squareTweenTargets: Array<GameObjects.GameObject>;
    // 正中的显示级别的文本
    levelText: BitmapText;
    // 选择动画
    rotateTween: Tween;
    // 放大动画
    growTween: Tween;
    // 游戏状态
    currentMode: number = AppData.AppMode.IDLE;


    constructor() {
        super({key: 'playGame'});
        this.rightBaseSquare = null;
    }

    create() {
        this.gameWidth = +this.game.config.width;
        this.gameHeight = +this.game.config.height;
        this.saveData = AppData.getSaveData();

        // 设置canvas背景色
        this.tintColor = AppData.getTintColor();
        this.cameras.main.setBackgroundColor(this.tintColor);

        // 创建墙壁
        this.placeWalls();
        // 创建中间的正方形和正方形中的文本
        this.createHeroSquare();
        // 创建显示级别的文本
        this.createLevelText();
        // 从初始状态更新到游戏等待状态，播放动画，初始化结束
        this.updateGameObjets();
        // 添加交互监听
        this.input.on('pointerdown', this.grow, this);
        this.input.on('pointerup', this.stop, this);
    }

    placeWalls() {
        // 初始状态在屏幕左下角，屏幕外
        this.leftBaseSquare = new Wall(this, 0, this.gameHeight, 'base', new Math.Vector2(1,1));
        this.add.existing(this.leftBaseSquare);
        // 初始状态在屏幕右下角，屏幕外
        this.rightBaseSquare = new Wall(this, this.gameWidth, this.gameHeight, 'base', new Math.Vector2(0,1));
        this.add.existing(this.rightBaseSquare);
        // 初始状态在leftBaseSquare上方，屏幕外
        this.leftTopSquare = new Wall(this, 0, this.gameHeight - this.leftBaseSquare.height, 'top', new Math.Vector2(1,1));
        this.add.existing(this.leftTopSquare);
        // 初始状态在rightBaseSquare上方，屏幕外
        this.rightTopSquare = new Wall(this, this.gameWidth, this.gameHeight - this.rightBaseSquare.height, 'top', new Math.Vector2(0, 1));
        this.add.existing(this.rightTopSquare);
    }

    createHeroSquare() {
        // 中间的正方形，初始状态在屏幕正上方，屏幕外
        this.heroSquare = new Square(this, this.gameWidth/2, -400, 'square', 0.2);
        this.add.existing(this.heroSquare);
        // 正方形中的文本，初始化为等级数
        this.heroSquareText = new SquareText(this, this.heroSquare.x, this.heroSquare.y, 'shortStack', '', 150, this.tintColor);
        this.add.existing(this.heroSquareText);
        // 因为shortStack字体没有数字，这里将数字转为字母显示
        this.heroSquareText.setText(String.fromCharCode(this.heroSquare.getSuccessful()));
        // 组合起来，方便后面一起动画
        this.squareTweenTargets = [this.heroSquare, this.heroSquareText];
    }

    createLevelText() {
        const level = 'level ' + String.fromCharCode(this.saveData.level);
        this.levelText = this.add.bitmapText(this.gameWidth/2, 0, 'shortStack', level, 60);
        this.levelText.setOrigin(0.5);
        this.levelText.setPosition(this.gameWidth/2, 40);
    }

    updateGameObjets() {
        let [min, max] = AppData.AppOptions.holeWidthRange;
        let holeWidth = Math.Between(min, max);
        let targetLeftX = (this.gameWidth-holeWidth)/2;
        let targetRightX = (this.gameWidth+holeWidth)/2
        // 底下的矩形物就位，中间的间隔就是洞口大小
        this.leftBaseSquare.tweenTo(targetLeftX);
        this.rightBaseSquare.tweenTo(targetRightX);
        // 矩形物上层的矩形就位
        [min, max] = AppData.AppOptions.wallRange;
        let wallWidth = Math.Between(min, max);
        this.leftTopSquare.tweenTo(targetLeftX - wallWidth);
        this.rightTopSquare.tweenTo(targetRightX + wallWidth);
        // 中间的矩形和矩形中的文本就位
        this.tweens.add({
            targets: this.squareTweenTargets,
            y: 180,
            scaleX: 0.2,
            scaleY: 0.2,
            angle: 50,
            duration: 500,
            ease: 'Cubic.easeOut',
            callbackScope: this,
            onComplete: () => {
                // 来回转动
                this.rotateTween = this.tweens.add({
                    targets: this.squareTweenTargets,
                    angle: 40,
                    duration: 300,
                    yoyo: true,
                    repeat: -1
                });
                this.currentMode = AppData.AppMode.WAITING;
            }
        });

    }

    grow() {
        if (this.currentMode === AppData.AppMode.WAITING) {
            this.currentMode = AppData.AppMode.GROWING;
            this.growTween = this.tweens.add({
                targets: this.squareTweenTargets,
                scaleX: 1,
                scaleY: 1,
                duration: AppData.AppOptions.growTime
            });
        }
    }

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

    // 掉入洞中动画
    dropToHole(heroDisplayHeight: number) {
        this.tweens.add({
            targets: this.squareTweenTargets,
            y: this.gameHeight + heroDisplayHeight,
            duration: 600,
            ease: 'Cubic.easeIn',
            callbackScope: this,
            onComplete: () => {
                this.levelText.setText('Oh no!!!!');
                this.gameOver();
            }
        });
    }

    fallAndBounce(flag:boolean) {
        // 计算成功后回弹后的最终y坐标
        let destY = this.gameHeight - this.heroSquare.displayHeight/2 - this.leftBaseSquare.displayHeight;
        let msg = flag ? AppData.AppTexts.success : AppData.AppTexts.failure;
        if (flag) {
            // 成功一次，小级别加一
            this.heroSquare.increaseSuccessful();
        } else {
            // 计算失败后的回弹位置
            destY = destY - this.leftTopSquare.displayHeight;
        }
        this.tweens.add({
            targets: this.squareTweenTargets,
            y: destY,
            duration: 600,
            ease: 'Bounce.easeOut',
            callbackScope: this,
            onComplete: () => {
                // 显示结果，成功还是失败
                this.levelText.setText(msg);
                if (!flag) {
                    this.gameOver();
                    return;
                }
                // 1s 后决定是重新开始游戏(进大级)还是继续下一级(进小级)
                this.time.addEvent({
                    delay: 1000,
                    callbackScope: this,
                    callback: () => {
                        if (this.heroSquare.getSuccessful() === Square.MAX_VALUE) {
                            // 升一大级
                            this.saveData.level ++;
                            AppData.updateSaveData(this.saveData.level);
                            this.scene.start('playGame');
                        } else {
                            // 更新小级文本
                            this.heroSquareText.setText(String.fromCharCode(this.heroSquare.getSuccessful()));
                            // 更新大级文本
                            this.levelText.setText('level ' + String.fromCharCode(this.saveData.level));
                            this.updateGameObjets();
                        }
                    }
                });
            }
        });
    }

    gameOver() {
        // 重新开始
        this.time.addEvent({
            delay: 1000,
            callbackScope: this,
            callback: () => {
                this.scene.start('playGame');
            }
        });
    }

}
```

因为使用的字体中没有数字，所以这里的级别改用了字母A和a。

这里需要注意的是各种动画的实现，都是用的tweens这个对象，来回转动可以设置yoyo=true来实现，回弹可以设置ease= 'Bounce.easeOut'来实现。

在每个动画的onComplete回调中这里都用了箭头函数，如果是普通函数你可能需要处理ts的类型错误，还有这里设置为箭头函数，属性callbackScope应该是可以不用设置了。

tweens动画需要我们设置的是目标状态，比如目标位置，目标角度等，所以动画的倒序播放其实只要你把目标位置设置为反方向即可。

还有一个延迟的实现是用了time对象，通过addEvent函数来配置延迟时间即可，比如设置delay为1000，即延迟1s，addEvent也有回调callback和回调作用域callbackScope的设置，这里我们一样用箭头函数处理。

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

