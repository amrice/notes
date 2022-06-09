在开始实例化一个Game实例的时候，有一个可选的参数就是gameConfig，它是GameConfig类的一个实例。因为是可选的，所以它的每一个属性都是有默认值的，我们可以从源码的注释中看到，下面说说创建游戏时最常用的配置。

![image-20220417095630918](https://cdn.jsdelivr.net/gh/ywxgod/image_source/imgs20220417095811.png)

### 常用配置

这里只是列举了下面的常用属性，平时开发中也基本只关注下面这几个了，其他的可以从文档中关注。

##### `width，height`

可以设置为`number|string`类型，如果不设置，则默认值分别是`1024px和768px`，就是说不管你有没有赋值，游戏的场景始终是个固定的大小，它无法根据具体的浏览器窗口适应，所以有没有将其设置为自适应的配置呢？看后面的`scale`属性。这里如果设置为字符串类型，则要求其父元素必须有一个大小。

##### `type`

默认值是`Phaser.AUTO`，意思是说如果webgl可用，优先使用webgl来渲染游戏，否则使用canvas。所以在使用的时候，我们大部分情况下无需配置它。

当type指定为`Phaser.CANVAS`时，我们需要指定`GameConfig`的canvas属性，意思是说我们用自己定义的canvas，而不是用phaser为我们自动创建这个canvas。如

```javascript
const myCustomCanvas = document.createElement('canvas');

myCustomCanvas.id = 'myCustomCanvas';
myCustomCanvas.style = 'border: 8px solid red';

document.body.appendChild(myCustomCanvas);

const config = {
    type: Phaser.CANVAS,
    parent: 'phaser-example',
    width: 800,
    height: 600,
    canvas: document.getElementById('myCustomCanvas')
};
```

当type指定为`Phaser.WEBGL`时，我们除了上面的设置外，还需要设置context属性，如

```javascript{1-11,18}
const contextCreationConfig = {
    alpha: false,
    depth: false,
    antialias: true,
    premultipliedAlpha: true,
    stencil: true,
    preserveDrawingBuffer: false,
    failIfMajorPerformanceCaveat: false,
    powerPreference: 'default'
};
const myCustomContext = myCustomCanvas.getContext('webgl', contextCreationConfig);
const config = {
    type: Phaser.CANVAS,
    parent: 'phaser-example',
    width: 800,
    height: 600,
    canvas: document.getElementById('myCustomCanvas'),
    context: myCustomContext,
};

```

##### `backgroundColor`

默认值是0x000000，即黑色。它指定了游戏场景的背景色，如果想改变背景色我们可以更改这个设置。

##### `parent`

用于指定包含canvas的元素，默认值为null，意思是说canvas会直接以body作为其父元素。如果赋值，则可以直接赋父元素(HTMLElement)，或者父元素的id(String)。

##### `scale`

用于指定场景的缩放行为，默认值为null，意思是不进行任何缩放。scale的值是`Phaser.Types.Core.ScaleConfig`的一个实例，它有多个属性来配置缩放行为，往后看。

##### `scene`

给游戏指定一个场景或多个场景，当时多个的时候需要用数组，场景对象可以是`Phaser.Scene`、`Phaser.Types.Scenes.SettingsConfig`、`Phaser.Types.Scenes.CreateSceneFromObjectConfig`三种类的任意实例。关于如何创建场景，后序关注。

##### `title，version，banner`

指定游戏的title与版本，它会在浏览器控制台中展示出来。banner可以指定控制台展示的样式，比如设置背景色，字体颜色等

### ScaleConfig的常用配置

ScaleConfig的部分属性跟GameConfig一样的，如width，height，parent这些如果在GameConfig中没有配置，我们可以在这里配置。下面看看最常用的两个配置项

##### `mode`

用于指定场景的缩放模式，默认值是`Phaser.Scale.ScaleModes.NONE`表示不进行任何缩放。如果我们要让场景随窗口自动调整大小，则需要将其设置为`Phaser.Scale.FIT`。所以这个或许就是我们最长用到的配置。还有一个值是`Phaser.Scale.ENVELOP`，它和FIT一样都是保持元素宽高比的，单前者是根据窗口的大小来缩放场景大小不要求填满窗口，它只是正好在某个方向上填满窗口，且以最小的一方为准。而后者则是以最大的一方为准，它会尽量填满窗口大小。

##### `autoCenter`

用于指定canvas是否居中放置在其父元素中。默认值是`Phaser.Scale.Center.NO_CENTER`，即不居中，左上角对其。通常我们需要将其指定为`Phaser.Scale.CENTER_BOTH`，则canvas会居中展示。

### 一个具体的例子

```javascript
const config = {
    backgroundColor: '#2d2d2d',
    parent: 'actions',
    scale: {
        mode: Phaser.Scale.FIT,
        autoCenter: Phaser.Scale.CENTER_BOTH
    },
    title: 'Shock and Awesome',
    version: '1.2b',
    scene: [ Example ]
};
```

