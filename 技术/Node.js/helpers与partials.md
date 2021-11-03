在express项目中用了handlebars模板引擎，其中helpers与partial可以让模板变得非常灵活。下面记录一下helpers与partials的常用场景。

handlebars模板引擎将模板分为两类，一类是layouts（布局），一类是views（视图），布局的默认存放位置是views/layouts/main.handlebars，views的默认存放目录是views/*.handlebars。

## 修改默认布局文件

下面看看如何配置express使用默认的布局和视图

```javascript
// 配置模板引擎
app.engine('handlebars', expressHandlebars());
app.set('view engine', 'handlebars');
```

如果我们的布局不用默认的main.handlebars而改为myMain.handlebars，我们可以像下面这样配置

```javascript
app.engine('handlebars', expressHandlebars({
    defaultLayout: 'myMain'
}));
```

此时我们需要在views/layouts下放置main.handlebars文件。但是不管怎样这里我们只能配置一个布局文件，即限制了我们所有的页面都用一个布局文件。

## 使用不同的布局文件

上面的配置只能使用一个布局文件，那我们如何使用不同的布局来渲染不同的页面呢？这就要用到res.render方法。

下面是使用默认布局，渲染views/foo.handlebars视图。

```javascript
app.get('/foo', (req, res) => res.render('foo'));
```

下面是不使用任何布局，不过此时foo视图需要包含完整的页面html

```javascript
app.get('/foo', (req, res) => res.render('foo', { layout: null }));
```

下面是使用指定的布局来渲染视图

```javascript
app.get('/foo', (req, res) => res.render('foo', { layout: 'myMain' }));
```

## helpers

目前看到的是视图会被渲染到布局文件中的一个固定位置，但如何将一个视图文件的不同部分渲染到布局的不同位置呢？这里就要用到helpers了。

先定义一个helpers对象

```javascript
const helpers = {
    section(name, options) {
        if(!this._sections) this._sections = {};
        this._sections[name] = options.fn(this);
        return null;
    }
};

exports.helpers = helpers;
```

修改引擎配置

```javascript
app.engine('handlebars', expressHandlebars({
    defaultLayout: 'main', // 模板
    helpers: handlebarHelpers.helpers
}));
```

假设我们有一个about视图：views/about.handlebars

```handlebars
<h1>About Meadowlark Travel</h1>
{{#if fortune}}
    <p>Your fortune for the day:</p>
    <blockquote>{{fortune}}</blockquote>
{{/if}}
{{#section 'first'}}
    <p>first</p>
{{/section}}
{{#section 'second'}}
    <p>second</p>
{{/section}}
```

在视图中我们使用了helpers中的section函数两次，分别传入了first和second，然后我们只需要在使用的布局中放置即可将其渲染的布局的不同位置，下面是views/layouts/main.handlebars的内容

```handlebars
<!doctype html>
<html>
    <head>
        <title>Meadowlark Travel</title>
    </head>
    <body>
        {{{body}}}
        {{{_sections.first}}}
        <p>=============</p>
        {{{_sections.second}}}
    </body>
</html>
```

如果在布局中没有放置{{{_sections.first}}}和{{{_sections.second}}}，但视图中又使用了，那如何？当然什么也不会渲染出来。因为在section函数中，最后返回的是null。

```javascript
section(name, options) {
    if(!this._sections) this._sections = {};
    // 直接返回内容
    return this._sections[name] = options.fn(this);
}
```

如果我们将section函数改为上面的样子，即直接返回fn的执行结果，那么视图的所有内容会被渲染到main.handlebars的body中。first部分会返回视图中first定义的内容，second同理。

## partials

将同一个视图的不同部分渲染到同一布局文件的不同部分，或者将同一个视图文件渲染到不同的布局文件的不同位置，又或者不使用布局而仅仅通过视图来定义完整的页面…

这些我们已经都可以做到了，下面的问题是我们如何在视图中重用一些html组件，比如在A视图中有一个list块，B视图中也用了list块，如何做到一个list重用到不同的视图中，此时我们可以定义partials。

partials的默认路径是views/partials/*.handlebars，假设我们定义一个myList.handlebars

```handlebars
<div class="my-list">
	<ol>
	{{#each partials.listData}}
		<li>
			<span>{{label}}</span> / 
			<span>{{icon}}</span>
		</li>
	{{/each}}
	</ol>
</div>
```

因为myList.handlebars可能会用在任何一个视图中，那each函数中的partials数据需要在渲染每一个视图的时候传入？这是可能的，但不好，我们应该将其放入res.locals中，res.locals是每一个视图渲染时的默认上下文环境。如果我们在res.render方法中传入了locals参数，locals将会覆盖res.locals对象，但res.locals中没有被覆盖的部分仍然可用。

```javascript
// 定义list数据
const listData = [
    { label: 'a', icon: 'search' },
    { label: 'b', icon: 'edit' },
    { label: 'c', icon: 'lang' }
];

// 通过中间件修改res.locals
function addPartialsContext(req, res, next) {
    if (!res.locals.partials) {
        res.locals.partials = {};
    }
    res.locals.partials.listData = listData;
    next();
}

app.use(addPartialsContext);
```

上面定义了list的数据，然后定义了中间件来将listData附加到res.locals中，下面将partials.handlebars用到视图中。

```handlebars
<h1>About Meadowlark Travel</h1>
{{#if fortune}}
    <p>Your fortune for the day:</p>
    <blockquote>{{fortune}}</blockquote>
{{/if}}
{{#section 'first'}}
    <p>first</p>
{{/section}}
{{#section 'second'}}
    <p>second</p>
{{/section}}
<p>partials======</p>
{{!这里插入list.handlebars}}
{{> list}}
```

{{> partial_name}} 这个语法实现在视图中使用某个名称为partial_name的partials，handlebars会默认去views/partials下查找【partial_name.handlebars】文件。









