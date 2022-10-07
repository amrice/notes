### ESM与CMJ的几个不同之处

1 ES模块是静态的，也就是说引入这种模块的那些语句要置于最顶部，且不能放置在流控制语句之内。另外受引用的模块只能用常量字符串，不能用表达式在运行时动态求值。但当通过`import(x)`动态导入模块时，此时是可以用表达式的。

2 ESM的静态引入机制，让我们有机会对模块间的依赖关系执行静态分析，继而做出必要的优化。

3 ESM使用的是`export`，单数形式，而CMJ的`exports`与`module.exports`是复数形式的。

4 ESM引入模块所在的文件时，一定要写出完整的扩展名，而CMJ只需要文件名即可。

5 ESM导入绝对路径的模块时，必须是`file:///xxx/yyy/zzz.js`这样的，而CMJ是以`/`或`//`开头的

6 模块的引入顺序在CMJ中可能会影响最终的执行结果，但ESM中不会。

7 ESM的export是引用模块内容，CMJ是对模块内容的浅拷贝。所以对于简单类型，如字符串、数字、布尔等类型在模块外引用的时候会有本质的不同。

8 CMJ中可以访问`__filename`找到当前文件(模块)的绝对路径；ESM中可以通过`import.meta.url`来获取，它是形如：`file:///path/to/current_module.js`

```javascript
import { fileURLToPath } from 'url';
import { dirname } from 'path';

// ES模块中获取当前模块的__dirname与__filename
const __filename = fileURLToPath(import.meta.url);
const __dirname = dirname(__filename);

// ES模块中获取CMJ中的require函数
import { createRequire } from 'module';
const require = createRequire(import.meta.url);
// 1 在ES模块中通过require函数加载CMJ模块
const xxx = require('xxx');

// 2 直接用import来导入CMJ模块,只能导出默认的东西，命名无法导出
import xx from 'xxx'; // ok
import { a } from 'xxx'; // not ok
```

9 ESM中this是未定义的`undefined`，而CMJ中的this是`exports`/`module.exports`

10 ESM中可以通过require、import导入CMJ模块，但反过来不行。

11 ESM无法直接导入json文件，但CMJ可以直接导入json文件。
