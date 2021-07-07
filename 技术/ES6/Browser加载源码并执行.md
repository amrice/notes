前面看过在node中如何加载远程代码执行，这里是针对浏览器环境的执行方法。想了解node环境的处理方法，可以连接：xxxxxx。

## 本地加载代码

因为现在的项目都是通过各种打包工具来构建编译的，要从本地加载js代码我们可以直接用import语句，不过各个工具都有所不同。

vite中可以在路径后面加查询参数，使之变为原始字符串

```javascript
import scripts from './scripts?raw';
```

上面这样加载的就是代码的字符串了，而不是可执行代码。webpack也有相同功能的插件来处理这种加载。

## 源码从远程加载

```typescript
async function getFileFromRemote(filename) {
	// 前面的步骤跟node环境一样，只是最后一句不一样
    ......
    // 将base64编码的文件内容转为utf8字符串
    return decodeURIComponent(escape(window.atob(data.data.content)));
}
```

如果你的代码里有中文，那么需要用上面的`decodeURLComponent(escape(atob(….)))`才可以显示正常，否则只需要用`atob(….)`即可。

## 执行代码

### eval

浏览器中执行JS，最先想到的就是eval，虽然eval可以执行JS代码，但不安全，我们可以从MDN中看到下面的话

>   ## [永远不要使用 `eval`！](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/eval#don.27t_use_eval.21)
>
>   `eval()` 是一个危险的函数， 它使用与调用者相同的权限执行代码。如果你用 `eval()` 运行的字符串代码被恶意方（不怀好意的人）修改，您最终可能会在您的网页/扩展程序的权限下，在用户计算机上运行恶意代码。更重要的是，第三方代码可以看到某一个 `eval()` 被调用时的作用域，这也有可能导致一些不同方式的攻击。

简单说，eval函数所执行的代码也是能访问调用eval的函数所在作用域，假设eval要执行的代码中的变量名跟调用eval的作用域中的某些变量名重名，那么这些变量将会被传入所执行的代码中。所以MDN推荐我们使用Function来执行函数。

### new Function

Function与eval不同的地方在于，它只能访问全局作用域，看下面的代码

```javascript
function test(a = 1, b = 5) {
    return eval('a+b');
}

function test2(a,b) {
    a = 3;
    b = 4;
    return new Function('return a+b');
}

a = 1;
b = 2;
c = test(); // 6
d = test(a,b); // 3
e = test2()(); // 3
console.log(c,d,e,f,window.a,window.b);
```

13行调用test函数，没有传参数，得c的值为6，因为eval能访问test函数的作用域中a=1，b=5；

14行调用test时传入了a=1，b=2，eval直接从test作用域中拿到a，b的值然后执行代码；

15行调用Function没有传入参数，但它也没有从test2的作用域中拿到a=3，b=4，而是从全局作用域中拿到a=1，b=2，所以e的值是3；

### with

eval会访问到其调用者所在的作用域，还有全局作用域；Function缩小了影响范围，它只能访问到全局作用域；而with就可以访问我们指定的作用域，不过正常情况下with也是不推荐使用的。

```javascript
function f(x, o) {
  with (o)
    print(x);
}
```

上面的代码打印x，如果有值，x可能是来自o的属性，也可能来自参数x，所以我们很难知道x来自哪里？所以要用with来指定作用域的话，上面的例子如果x有值，那么它一定只能来自o，而不是参数x，那如何做到呢？

### Proxy

为了阻止x的值从x参数获取，我们得保证o中存在x，所以得用Proxy对o做代理。

```javascript
const handler = {
    get: function(obj, prop, receiver) {
        return prop in obj ? obj[prop] : 37;
        // return Reflect.get(obj, prop, receiver);
    },
    has: function() { return true; }
};
const o = {}；
const p = new Proxy(o, handler);
console.log('c' in p, p.c);
```

上面的has始终返回true，这可以保证所有in操作的结果为true，即任何属性都会存在于o中。这里的第三行要注意的是，in操作判断的是原对象，所以`prop in obj`还是false。除非改成`prop in receiver`，或者取消第四行的注释。



经过上面的准备工作，我们终于可以来实现无污染的代码执行了。

```typescript
function runScript(scripts) {
    scripts = 'with(context) { ' + scripts + '}';
    let fun = new Function('context', scripts);
    return context => {
        const proxy = new Proxy(context, {
            has() { return true; },
            get(target, p, receiver) {
                return Reflect.get(target, p, receiver);
            }
        });
        return fun(proxy);
    };
}

async function start() {
    let s = await getFileFromRemote('scripts.js');
    s = `return ${s};`;
    console.log(s);
    s = runScript(s)({});
    console.log(s(1));
}
```

放入with中的代码我们需要具体情况做具体分析，如果是有返回值的函数你可能要在外面增加return语句才能拿到值，如果是一般的JS语句的执行，则需要考虑返回执行的结果。

## 总结

上面的代码还是比较粗糙的，不过基本思路应该是这样的，权作学习用。