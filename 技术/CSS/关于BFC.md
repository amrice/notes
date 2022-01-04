BFC(块级格式化上下文)是block formatting context的简写，BFC是网页的一个区域，元素可以基于此块区域布局，它将其内部的内容与外部的内容隔离开。换句话说就是，BFC里的内容不会跟外面的元素重叠或相互影响。创建了BFC的元素，会有下面3个特点：

-   包含了内部所有元素的上下文外边距，他们不会跟外部元素的外边距折叠。
-   包含了内部所有浮动的元素。
-   不会跟BFC外部的浮动元素重叠。

### 如何创建一个BFC

给元素添加以下的任意属性值都会创建BFC

-   float：left或right，不为none即可。
-   overflow：hidden、auto或scroll，不为visible即可。
-   display：inline-block、table-cell、table-caption、flex、inline-flex、grid或inline-grid。
-   position：absolute或fixed

### 例子

html

```html
<div class="media">
	<img class="media-image" src="https://bkssl.bdimg.com/cms/rc/r/image/2014-08-30/15362fa4188d2cc42f1476e2245f165e_360_212.png">
	<div class="media-body">
		<h4>What can i do for you</h4>
		<p>
		 	Don't run the same every time you hit the road. Vary your pace, and vary the distance of you runs.
		</p>
	</div>
</div>
```

css

```css
.media {
	width: 50%;
	font-size: 1rem;
	background-color: #eee;
	border-radius: 0.5em;
	padding: 0.5em;
}

.media h4 {
	margin: .1em 0;
	margin-top: 0;
}

.media-image {
	float: left;
	width: 150px;
	height: 100px;
	margin-right: 1em;
}

.media-body {
    /** 使media-body成为一个BFC */
    /** 致使图片右侧的文字不会因为图片左浮动而环绕 */
	overflow: auto;
	margin-top: 0;
}
```

### 效果

![image-20220104212641354](https://cdn.jsdelivr.net/gh/ywxgod/image_source/imgs20220104214025.png)

