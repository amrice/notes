这是最昨天的那篇([用Vite打包Phaser项目](/blog/2021-06-26/start-phaser-use-vite-and-ts/))的补充，有什么不明白的可以先看看。



由于Phaser的一些api需要资源的路径，所以昨天我们直接将资源放到public目录了，这样直接从根目录开始引用原始路径即可。

然后今天发现vite的资源加载可以加入查询参数，如果我们只是想要资源的路径而无需vite对资源预处理，可以下面这样来加载，如：

```javascript
// 加载图片
import shortStack from '../assets/shortStack.png?url';
// 加载xml
import shortStackXMLUrl from '../assets/shortStack.xml?url';
```

在路径后面加上查询参数url，那么vite返回的就是资源路径，我们可以将预加载场景的资源加载改为下面这样的

```typescript
preload() {
    this.load.image('base', base);
    this.load.image('square', square);
    this.load.image('top', top);
    this.load.bitmapFont('shortStack', shortStack, shortStackXMLUrl);
}
```

这样处理后，我们的资源不用放到public目录也可以正常加载了。不过在build后，可能你会遇到Phaser报某个图片的格式不支持而导致无法加载

![image-20210627160752784](https://cdn.jsdelivr.net/gh/ywxgod/image_source/imgs20210627162821.png)

这是因为vite的默认配置中会把小于1k的图片转为base64，导致Phaser无法识别到图片的url而报错，所以我们只需要禁用好了

```javascript
// vite.config.js
export default {
    base: '/abc/',
    build: {
        assetsInlineLimit: 0
    }
}
```

因为phaser一般都是通过url加载图片，在phaser项目中禁用应该影响不大，这个默认设置本是为了减少图片请求而做的优化，其他项目中还是需要开启。特别是有支持base64为路径的api。

