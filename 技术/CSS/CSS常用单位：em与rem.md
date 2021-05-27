关于CSS常用单位的另外一篇请链接：[CSS单位之：px、百分比、vw、vmin](https://caoos.gitee.io/blog/CSS/CSS%E5%8D%95%E4%BD%8D%E4%B9%8Bpx%E3%80%81%E7%99%BE%E5%88%86%E6%AF%94%E3%80%81vw%E3%80%81vmin/)

# em与rem

css中1em相当于当前元素的字号大小，如：

```css
.padded {
	font-size: 16px;
	padding: 1em; /** 设置padding值为16px */
}
```

如果有另外的选择器也命中了相同的元素，且修改了字号的大小，则em的相对大小也会随之改变，上面的padding值会是最后的font-size大小来决定。

我们一般用em来设置如下属性：padding，height，width，border-radius等，因为当字体大小发生改变时，这些属性会跟随改变。

```css
.box {
	padding: 1em;
	border-radius: 1em;
	background-color: lightgray;
}

.box-small {
	font-size: 12px;
}

.box-large {
	font-size: 18px;
}
```

## 用em定义字号

前面说过，当前元素的字号决定了em值，如果字号也被定义为em，那结果如何？如下代码：

```css
.slogan {
	font-size: 1.2em;
	padding: 1.2em;
	background-color: #ccc;
}
```

padding值该如何计算？

首先我们需要算出font-size的值，实际上font-size定位为相对大小是，它的值时根据继承的值来计算的。假如上面的继承值是16px(可能来自父元素，也可能需要往上查找直到body，html)，我们先得到字号是16*1.2=19.2px，然后计算padding为19.2*1.2=23.04。所以这里就出现一种情况：尽管padding，font-size申明的值一样，但是计算值却不一样。

## 字体缩小的问题

在嵌套的列表中，如果使用em设置font-size，会使字体越来越小，如下代码：

```html
<ul>
	<li>Top Level</li>
	<ul>
		<li>Second Level</li>
		<ul>
			<li>Third Level</li>
			<ul>
				<li>Fourth Level</li>
			</ul>
		</ul>
	</ul>
</ul>
```

样式如下：

```css
body {
	font-size: 16px;
}

ul {
	font-size: .8em;
}
```

根据em的计算规则，每一级ul的字体大小是前一级的0.8，所以会出现字体越来越小的效果。我们只要重新设置回ul的字体大小即可解决问题。如下：

```css
ul {
	font-size: .8em;
}

ul ul {
	font-size: 1em;
}
```

## 使用rem设置字体

rem是root em的缩写。rem不是相对当前元素，而是相对于根元素的单位。意思就是说不管在css中什么地方使用rem作为单位，相同的rem值最终计算出来的计算值一定相同。如：

```css
:root { /** :root伪类等价于html选择器 */
	font-size: 1em; /** 使用浏览器默认字号：16px */
}

ul {
	font-size: .8rem; /** 相对于根元素，即：16*0.8px */
}
```

## 建议

- 用rem设置字号
- 用px设置边框
- 用em设置其他大部分属性，尤其是内边距，外边距，圆角等

