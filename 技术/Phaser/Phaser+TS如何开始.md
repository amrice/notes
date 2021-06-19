目前Phaser4发布了beta版，它用TS重写了Phaser3的实现，所以我们又得研究一下如何用TS来写Phaser了，官网已经开始推荐我们用TS了。下面看看Windows上的TS+Phaser该如何开始。

## 初始化项目

先安装稳定版的Node，如Node 12/14都可以，然后初始化项目。在一个空目录下面执行下面的命令

```shell
npm init -y
```

这会在目录中创建我们的package.json，接着安装phaser

```shell
npm install phaser -S
```

全局安装typescript，parcel

```shell
npm install typescript parcel-bundler -g
```

## 创建文件

在项目目录中创建一个game目录，然后依次创建三个文件：Game.ts，GameScene.ts，index.html。

Game.ts是游戏逻辑入口，index.html是运行页面，GameScene.ts是游戏的一个场景。

```typescript
// GameScene.ts
export class GameScene extends Scene {

    text:GameObjects.Text;
    radian:number = 0.1;
    a:number = 0.01;
    v:number = 0;

    constructor() {
        super({key: 'game_scene'});
        this.text = null;
    }

    create() {
        const width: number = +this.game.config.width;
        const height: number = +this.game.config.height;
        this.text = this.add.text(10,10, 'Hello, World! TS + Phaser');
        const px = width/2;
        const py = height/2;
        this.text.setPosition(px,py);
        this.text.setOrigin(0.5, 0.5);
    }

    update() {
        this.v += this.a;
        this.radian += this.v;
        this.text.setRotation(this.radian);
    }

}
```

在update函数中我们让文本旋转起来了。

```typescript
// Game.ts
import { Game, Types } from 'phaser';
import { GameScene } from "./GameScene";

const config: Types.Core.GameConfig = {
    type: Phaser.AUTO,
    scene: GameScene,
    width: 640,
    height: 320
};

new Game(config);
```

Game.ts负责实例化一个Game实例

```typescript
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Phaser Started</title>
</head>
<body>
    <script src="Game.ts"></script>
</body>
</html>
```

index.html负责加载Game.js到浏览器执行

## 运行

修改package.json中的scripts，添加下面的命令

```json
"scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "dev": "parcel game/index.html"
},
```

然后执行`npm run dev`，就会看到下面的效果



![双击填充](https://cdn.jsdelivr.net/gh/ywxgod/image_source/imgs20210619221439.gif)



