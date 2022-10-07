搞工程化，不是为了提升技术而搞，最根本目的是服务于业务，不能促进业务发展得更好，啥技术都是扯淡。但要将工程化的一些工作直接跟业务扯上关系，是比较难的，最多就跟老板说说能提高效率，优化代码质量之类的事儿，可能还比不上一个运营活动拉新拉活拉收入来得直接。举个跟业务结合非常紧密的工程例子吧，那边是a/b test，或者叫做[灰度系统](https://www.zhihu.com/search?q=灰度系统&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"answer"%2C"sourceId"%3A407728192})，这个系统可以直接帮业务验证新的设计或者功能到底市场是否接受。抽象工作流的话，其实也不用多说，就是看看团队工作的流程，业务有哪些需要解决的痛点，通通列出来即可。

结合用户数据系统，发布系统，数据上报检测系统

编码前：编码规范、最佳实践、登录鉴权

编码中：脚手架、构建器、代码检查工具

编码后：发布工具、脚本、CI/CD

编码规范、公共库、工具使用文档等 - 申请一台虚拟机/服务器做一个专门的前端文档站点。可以考虑vuepress来实现，能编译markdown又能运行vue文件。

最佳实践包括两个部分：

1、最佳的文件目录结构，目录命名，项目初始化后最基本的目录配置等。

2、框架层面的底座代码，包括跟后台交互的数据格式，前端登录鉴权等。

脚手架-整合最佳实践：最佳目录结构，公共库的引入

- `domLoading`: this is the starting timestamp of the entire process, the browser is about to start parsing the first received bytes of the HTML document.
  
  开始解析接收到的html页面的第一个字节的时间戳

- `domInteractive`: marks the point when the browser has finished parsing all of the HTML and DOM construction is complete.
  
  解析完所有HTML文件内容，已经构建DOM完成。-- DOM准备完成

- `domContentLoaded`: marks the point when both the DOM is ready and there are no stylesheets that are blocking JavaScript execution - meaning we can now (potentially) construct the render tree.
  
  DOM构建完成，且CSS也加载完成并CSSDOM也构建完成，此时可以构建渲染树。如果没有CSS，那么此事件会接着domInteractive触发。-- DOM与CSSDOM准备完成

- `domComplete`: as the name implies, all of the processing is complete and all of the resources on the page (images, etc.) have finished downloading - in other words, the loading spinner has stopped spinning.
  
  所有资源下载完成，包括图片。-- 页面所有资源下载完成

- `loadEvent`: as a final step in every page load the browser fires an `onload` event which can trigger additional application logic.
  
  onload事件触发时间，此时可以执行js的业务逻辑。

```javascript
function measureCRP() {
        var t = window.performance.timing,
          interactive = t.domInteractive - t.domLoading,
          dcl = t.domContentLoadedEventStart - t.domLoading,
          complete = t.domComplete - t.domLoading;
        var stats = document.createElement('p');
        stats.textContent =
          'interactive: ' +
          interactive +
          'ms, ' +
          'dcl: ' +
          dcl +
          'ms, complete: ' +
          complete +
          'ms';
        document.body.appendChild(stats);
      }
```

1. 异步加载非首屏用到的资源
   
   ```javascript
   form.addEventListener("submit", e => {
     e.preventDefault();
     import('library.moduleA')
       .then(module => module.default) // using the default export
       .then(someFunction())
       .catch(handleError());
   });
   ```

2. 预加载关键路径上的js和css中引入的字体文件
   
   ```html
   <head>
     <link rel="preload" as="script" href="critical.js">
     <link rel="preload" as="style" href="css/style.css">
     <link rel="preload" href="ComicSans.woff2" as="font" type="font/woff2" crossorigin>
   </head>
   ```
   
   通过webpack预加载
   
   ```javascript
   import(_/* webpackPreload: true */_ "CriticalChunk");
   ```

3. 可以将关键路径上的css分成两部分，一部分是首屏渲染需要用到的，一部分是非首屏用到的，对于首屏用到的我们可以通过预加载处理，另一部分可以使用defer异步加载。

4. xxxx

5. xxxx

6. xxxx

性能：

首屏时间：

有图：