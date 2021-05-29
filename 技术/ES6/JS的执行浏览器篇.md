# JS的执行浏览器篇

ECMAScript标准只告诉了浏览器，如何让一段相同的JS执行后能产生相同的输出，即定义了JS的语法，关键字等，各个浏览器需要能解释这些JS语法，之后能执行它。

但它并没有规定每个浏览器内部应该怎么做，才能去解释这些JS语法和执行JS。浏览器内部的实现是每个浏览器厂商自家的实现。

## JS引擎从头说起

每一个浏览器都有一个内置的JS引擎，用于执行JS代码。最初的网景浏览器使用的JS引擎叫SpiderMonkey，它只是一个非常简陋的JS解释器，没有做任何优化，虽然处理JS比较慢，但却可以工作。

![](https://cdn.jsdelivr.net/gh/ywxgod/image_source/imgs20210528220406.png)

从上图我们知道，JS引擎的第一个任务是读取JS源代码，然后编译成CPU能理解的二进制指令(机器码)。JS引擎包含了一个基线编译器，它负责将JS源码转为中间过程的字节码，接下来由解释器将字节码转为二进制指令输入到CPU。

基线编译器以尽可能快的速度将源码转为字节码，这些字节码没有经过优化，然后就传给了解释器，这会导致解释器变慢，从而影响到整个JS的执行速度。

>   现在的SpiderMonkey已经不再像以前，它已经对产出的字节码做了高度的优化，并且目前在FireFox浏览器中使用。

随着网络应用的大规模交互与动态化，上面的这种慢的JS执行解释模式，已经影响到用户体验了。当年谷歌浏览器在运行谷歌地图应用的时候就遇到这样的体验问题，所以为了提升JS的性能，以及用户体验，他们不得不想出一个更好的方法。

谷歌浏览器在初期使用的JS引擎叫V8，为了提升JS性能，他们给V8引擎的执行流程增加了2个东西，如下图

![](https://cdn.jsdelivr.net/gh/ywxgod/image_source/imgs20210528223029.png)

在2010年版的V8中，主要有两个东西负责重活：full-codegen(基线编译器)与crankshaft(负责优化的编译器)。前者主要负责尽可能快的产出未优化的二级制指令，以加快应用程序的启动；应用启动后，crankshaft编译器会启动来优化源代码，并替换full-codegen来产生优化过的二进制指令，以提升JS的运行性能。

然而上面的优化会消耗大量的CPU时间和内存，所以V8必须提出另外一种JS的执行和解析模式，它就是JIT模式。从上图中我们可以看出解释器已经不存在了，就是说JS源码直接变成了CPU能识别的二进制指令，并且是在边处理源码边生成二进制指令的，所以这加快了JS的执行效率。

上面是从流程上减少了解释器部分来优化，那从JS的执行来看又做了哪些优化呢？

### JS优化

JS在进入基线编译器之前，它会被解析为AST，通过AST我们可以对源码进行优化，优化主要有以下两点：

-   优化JS代码中没必要的复杂逻辑。
-   基于已定义的变量值来猜测其出类型，以便于产出更加优化的机器码。

在优化JS的执行的同时，V8还会收集和分析JS的执行过程，找出其运行缓慢的地方。这些运行缓慢的代码会被进一步优化，以输出更加优化的机器码。

基于上面的分析和考虑，V8团队在2017年又发布了一个新版本。

![](https://cdn.jsdelivr.net/gh/ywxgod/image_source/imgs20210528234657.png)



从上图可以看出，这一版的V8引入了Ignition的概念，它实际上是一个基线编译器加一个解释器组成的，跟之前一样基线编译器用于将源码转为字节码，而解释器用于将字节码转为二进制的指令。

另外一个就是TurboFan，也是一个字节码优化编译器。它会用独立的线程来对字节码进行优化处理。TurboFan的另外一个作用是接收V8运行JS所产生的分析数据，并找出运行缓慢且耗时的代码，然后通过基本的优化手段(数据类型)对代码作出优化或去优化。

## JS运行时

JS是通过单线程运行的，这就是说同一时间只有一段代码在被执行，且它们是按照顺序(序列)执行的。所以，如果某段代码执行时间过长，就会阻塞后面的代码执行。就像下面这样的情况：

![](https://cdn.jsdelivr.net/gh/ywxgod/image_source/imgs20210529085945.png)

当JS执行被阻塞后，浏览器会停止响应当前页面，比如下面的代码：

```javascript
while(true) {}
```

现代浏览器通过多tab/域名形式来处理这种可能的阻塞，让其只会发生在当前tab标签中，而不会影响其他tab。但如果是不同的tab但是打开的相同域名的页面，如果一个tab阻塞，其他tab也会阻塞，因为谷歌浏览器会用相同的线程处理相同的域名下的页面。

### JS运行机制

先可以看看Philip_Roberts的经典演讲：[What the heck is the event loop anyway? | Philip Roberts | JSConf EU](https://youtu.be/8aGhZQkoFbQ)，还可以看看他还专门做了一个在线JS运行可视化应用：[JS运行时可视化](http://latentflip.com/loupe/?code=JC5vbignYnV0dG9uJywgJ2NsaWNrJywgZnVuY3Rpb24gb25DbGljaygpIHsKICAgIHNldFRpbWVvdXQoZnVuY3Rpb24gdGltZXIoKSB7CiAgICAgICAgY29uc29sZS5sb2coJ1lvdSBjbGlja2VkIHRoZSBidXR0b24hJyk7ICAgIAogICAgfSwgMjAwMCk7Cn0pOwoKY29uc29sZS5sb2coIkhpISIpOwoKc2V0VGltZW91dChmdW5jdGlvbiB0aW1lb3V0KCkgewogICAgY29uc29sZS5sb2coIkNsaWNrIHRoZSBidXR0b24hIik7Cn0sIDUwMDApOwoKY29uc29sZS5sb2coIldlbGNvbWUgdG8gbG91cGUuIik7!!!PGJ1dHRvbj5DbGljayBtZSE8L2J1dHRvbj4%3D)，让你对JS的运行机制能有更深的理解。

JS的运行时基于事件循环的，那么什么是事件循环？往后面看。

我们知道JS是单线程执行，按顺序来的，那它是怎么做到异步的呢？其实JS的任务分为同步和异步，同步任务是直接在JS的主线程中被执行，而异步任务需要放到异步队列中排队等候执行。

简单来说JS的执行过程中有以下组成：

-   主线程调用栈
-   异步队列

执行过程可以简单概括：

1.  所有同步任务都在主线程的调用栈上被执行。
2.  如果某个异步任务有了结果，就被加入到异步队列中。
3.  一旦主线程调用栈任务执行完毕，就会去读取异步队列中的异步任务，将其加入到主线程调用栈，由主线程来执行。

上面的一整个过程，包括检查调用栈是否为空，如何确定将那个异步任务加入到调用栈等，这就是事件循环，它是JS实现异步的核心。

### 宏任务(macro task)和微任务(micro task)

macro task与micro task是针对异步任务来划分的，就是那些会被放到异步队列中的任务。

macro task：包含执行整体的js代码，事件回调，ajax回调，定时器(setTimeout,setInterval,setImmediate)，IO操作，UI渲染等

micro task：更新应用程序状态的任务，包括promise回调，process.nextTick，Object.observe，MutationObserver等

![](https://cdn.jsdelivr.net/gh/ywxgod/image_source/imgs20210529093836.jpg)

上图就是macro task与micro task的执行过程，转为代码就是：

```javascript
for (macroTask of macroTaskQueue) {
    // 1. Handle current MACRO-TASK
    handleMacroTask();
      
    // 2. Handle all MICRO-TASK
    for (microTask of microTaskQueue) {
        handleMicroTask(microTask);
    }
    
    // 3. render ui
    handleUIRenderer();
}
```

### ui渲染的时机

从上图我们可以看出，ui渲染发生在每轮的macro task的最后面，即前面的所有micro task执行完之后。

我们暂将一轮事件循环叫做上面的外层for的一个循环，即包含：单个的macro task，所有micro task，ui渲染三部分。

浏览器通常以60fps的速率刷新页面，也就是16.7ms一帧。所以如果不影响浏览器的ui渲染，执行单个macro task与其后的所有micro task的耗时不应超过16.7ms。

但并不是每轮事件循环浏览器都会执行ui渲染，它有自己的优化策略，例如把几次更新累积到一起来渲染，渲染之前会通知requestAnimationFrame执行回调函数，也就是说在进行渲染之前，一定会执行requestAnimationFrame的回调函数。但回调函数却并不是每轮的事件循环都执行，它的执行时机可能在多轮事件循环之后再执行。

## 参考

>   https://medium.com/jspoint/how-javascript-works-in-browser-and-node-ab7d0d09ac2f
>
>   http://lynnelv.github.io/js-event-loop-browser
>
>   https://zhuanlan.zhihu.com/p/41496446
>
>   https://mp.weixin.qq.com/s?__biz=MzI3ODU4MzQ1MA==&mid=2247491880&idx=2&sn=d913172be896b665b2386b9b2ebcb648&chksm=eb5660dddc21e9cb2604e2c1ee164843b0796abe7e69c9f596ecc4ab3b677de8b4a00f837e3d&mpshare=1&scene=24&srcid=0528wYTdu8xuWjkvk0ptfjx7&sharer_sharetime=1622176686338&sharer_shareid=ab55c2f1f985b6f43c607e6fd7131d43#rd

