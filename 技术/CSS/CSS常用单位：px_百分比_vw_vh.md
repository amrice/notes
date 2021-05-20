CSS所定义的单位有很多种，经常让人迷糊，不知道用哪个，或者不知道怎么用，经常还有新的单位出现，让人感觉学不动了，不过平时我们在80%的场景中只用到了20%的单位。下面我们就说说最常用的一些：

-   像素单位px，百分比%，视口单位vw/vh
-   基于字体的单位rem/em
-   基于flexbox和grid布局的单位fr

这次先看第一条中的三种，后面的两条再续。

```html
<body>
    <div class="px bg">75px</div>
    <div class="percent bg">50%</div>
    <div class="vw bg">100vw</div>
</body>
```

假设有上面的html结构，css如下：

```css
html,body{
    background: #eee;
    margin: 0;
    padding: 0;
}

.px { width:75px; }
.percent { width: 50% }
.vw { width: 100vw }
.bg {
	background-color: #333;
	color: white;
	margin: 10px;
}
* { box-sizing: border-box; }
```

分别设置三个DIV的宽度为75px，50%，100vw

![image-20210520233820838](https://cdn.jsdelivr.net/gh/ywxgod/image_source/imgs20210520233826.png)

然后我们将三个DIV放入一个容器中，修改css和html代码

```css
/** 增加容器样式 */
.container { width: 50%; background-color: #aaa }
```

```html
<div class="container">
	<div class="px bg">75px</div>
	<div class="percent bg">50%</div>
	<div class="vw bg">100vw</div>
</div>
```

效果

![image-20210520234406673](https://cdn.jsdelivr.net/gh/ywxgod/image_source/imgs20210520234406.png)

根据上图：

px是固定的宽度，不随容器的改变而改变

百分比是相对于父级容器的宽度的

vw是相对于视口的宽度来说的，它不会随容器的宽度改变



还有vmin和vmax两个单位，他们会根据视口的宽度和高度来选取其中一个。比如视口是800px * 600px，那么50vmin就表示50%的min(宽度，高度)，即300px。vmax同理。

### 参考

http://www.js-craft.io/blog/know-your-size-css-units-explained-part-1-pixels-percentages-and-viewpoints-units/

