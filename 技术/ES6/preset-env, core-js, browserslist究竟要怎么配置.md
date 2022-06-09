如题这三个东西，相信很多人都见过，也都会用会配置，但要是让你说出个所以然来，恐怕还不是那么容易。

曾经的我，一边google一边配置，只要能用就万事大吉了，且每次google出来的似乎还跟上次的都有所不同，所以每次配置项目时，有差异的。然而却很少去深究为啥？为啥这样配也可以，那样配也可以，那到底怎样配置才是最佳的？

下面就来简单说说我的理解，不对的地方欢迎指正。

### 名称解释

##### `preset-env`

从名称看有个env，这里就是包含“环境”之意。

在preset-env出来之前，我们需要自己知道要用什么es6的特性，然后自己去babel的配置文件中加进去，如：

```json
{
  "plugins": [
      "@babel/plugin-transform-arrow-functions",
      "@babel/plugin-transform-classes",
      "@babel/plugin-transform-spread",
      ...
  ]
}
```

这样做非常繁琐，开发体验也不好。再后来，后来就出现了@babel/preset-env。

@babel/preset-env可以通过target属性配置一个目标环境，babel会根据环境来转换那些它不支持的语法，这样就不需要我们一个一个的去自己加入所要支持的es6特性。如

```json
{
    "presets": [
        ["@babel/preset-env", {
            "target": {
                "browsers": ["last 2 versions", "ie >= 7"]
            }
        }]
    ]
}
```

所以，@babel/preset-env的作用就是将常用的ES6特性放到一起了，然后添加一个可以配置的目标环境，它自己决定要转换那些ES6特性，这样开发体验就好很多。这样虽然不需要我们配置ES6特性，但需要我们自己配置目标环境，且这个环境只是babel自己知道，如果还有其他应用，如ESLint，TS等等，其他应用也需要读取目标环境来决定行为，还得配置……所以browserslist出现了。

##### `browserslist`

browserslist提供了一种项目共享的目标环境配置，整个项目的babel、eslint，ts等都可以读取到。如：

```shell
# Browsers that we support

[modern]
Firefox >= 53
Edge >= 15
Chrome >= 58
iOS >= 10.1

[legacy]
> 1%
```

它有自己的配置语法，一看就会，它有多种具体文档：https://github.com/browserslist/browserslist

有了browserslist的配置，我们就可以不用配置@babel/preset-env的target了。browserslist的配置可以写在package.json里面也可以用独立的.browserslistrc文件。

##### `core-js`

相信很多人一开始并不清楚core-js是干嘛的，因为我们已经有babel了，为何还要core-js呢？如果你这么想，那就是有个关键的概念没有搞清楚。一般babel只是用来将ES6+的语法转为ES5，它并不处理ES6新增的API，如

```javascript
const t = [1,2,3];
console.log(...t);
const x = t.includes(2);
console.log(x);
```

转换后

```javascript
"use strict";
var _console;
var t = [1, 2, 3];
(_console = console).log.apply(_console, t);
var x = t.includes(2);
console.log(x);
```

上面的扩展运算符是属于语法的范畴，而数组的includes方法是属于ES6新增的API，所以babel只是转换了扩展运算符，而并没有处理includes方法。所以当我们在比较老旧的浏览器中运行时会报错，如何让这些老旧的浏览器也能认识ES6新增的这些API，这就是core-js要做的事情了。

### 一起使用

core-js提供了各种“老旧浏览器”需要的ES6新API的实现，browserslist提供了指定哪些“老旧浏览器”的功能，而@babel/preset-env提供了针对browserslist指定的老旧浏览器来转换ES6到ES5，同时还能根据browserslist指定的环境从core-js中提取需要的ES6新增API的实现。简单说@babel/preset-env使用core-js、browserslist通过适当的配置来支持各种老旧的环境。

下面用一些简单的代码来演示如何一起使用，假设我们有如下`index.js`文件

```javascript{codeTitle="index.j"}
init();

function init() {
    const p = new Map();
    const t = Object.entries({a:1,b:2})
    const y = [1,2,3].findIndex(x => x === 3);

    const t1 = new Promise((resolve) => {
        setTimeout(resolve, 1000, 3,2);
    });

    const obj = {
        maps: [p],
        printArr() {
            console.log(...t);
        },
        async p() {
            return t1;
        }
    };

    obj.printArr();
    obj.p().then(s => {console.log(s)});
}
```

`.browserslistrc`只指定ie11

```shell{codeTitle=".browserslistrc"}
ie 11
```

`babel.config.json`，指定usage方式引入core-js，以及指定使用的core-js版本

```json{codeTitle="babel.config.json"}
{
    "presets": [
        ["@babel/preset-env", {
            "useBuiltIns": "usage",
            "corejs": "3"
        }]
    ]
}
```

然后通过babel命令行转译

```javascript
"use strict";
require("core-js/modules/es.symbol.js");
require("core-js/modules/es.symbol.description.js");
require("core-js/modules/es.symbol.iterator.js");
require("core-js/modules/es.array.from.js");
require("core-js/modules/es.array.slice.js");
require("core-js/modules/es.function.name.js");
require("core-js/modules/es.regexp.exec.js");
require("regenerator-runtime/runtime.js");
require("core-js/modules/es.array.iterator.js");
require("core-js/modules/es.map.js");
require("core-js/modules/es.object.to-string.js");
require("core-js/modules/es.string.iterator.js");
require("core-js/modules/web.dom-collections.iterator.js");
require("core-js/modules/es.object.entries.js");
require("core-js/modules/es.array.find-index.js");
require("core-js/modules/es.promise.js");

function asyncGeneratorStep(gen, resolve, reject, _next, _throw, key, arg) { try { var info = gen[key](arg); var value = info.value; } catch (error) { reject(error); return; } if (info.done) { resolve(value); } else { Promise.resolve(value).then(_next, _throw); } }

function _asyncToGenerator(fn) { return function () { var self = this, args = arguments; return new Promise(function (resolve, reject) { var gen = fn.apply(self, args); function _next(value) { asyncGeneratorStep(gen, resolve, reject, _next, _throw, "next", value); } function _throw(err) { asyncGeneratorStep(gen, resolve, reject, _next, _throw, "throw", err); } _next(undefined); }); }; }

function _toConsumableArray(arr) { return _arrayWithoutHoles(arr) || _iterableToArray(arr) || _unsupportedIterableToArray(arr) || _nonIterableSpread(); }

function _nonIterableSpread() { throw new TypeError("Invalid attempt to spread non-iterable instance.\nIn order to be iterable, non-array objects must have a [Symbol.iterator]() method."); }

function _unsupportedIterableToArray(o, minLen) { if (!o) return; if (typeof o === "string") return _arrayLikeToArray(o, minLen); var n = Object.prototype.toString.call(o).slice(8, -1); if (n === "Object" && o.constructor) n = o.constructor.name; if (n === "Map" || n === "Set") return Array.from(o); if (n === "Arguments" || /^(?:Ui|I)nt(?:8|16|32)(?:Clamped)?Array$/.test(n)) return _arrayLikeToArray(o, minLen); }

function _iterableToArray(iter) { if (typeof Symbol !== "undefined" && iter[Symbol.iterator] != null || iter["@@iterator"] != null) return Array.from(iter); }

function _arrayWithoutHoles(arr) { if (Array.isArray(arr)) return _arrayLikeToArray(arr); }

function _arrayLikeToArray(arr, len) { if (len == null || len > arr.length) len = arr.length; for (var i = 0, arr2 = new Array(len); i < len; i++) { arr2[i] = arr[i]; } return arr2; }

init();

function init() {
  var p = new Map();
  var t = Object.entries({
    a: 1,
    b: 2
  });
  var y = [1, 2, 3].findIndex(function (x) {
    return x === 3;
  });
  var t1 = new Promise(function (resolve) {
    setTimeout(resolve, 1000, 3, 2);
  });
  var obj = {
    maps: [p],
    printArr: function printArr() {
      var _console;

      (_console = console).log.apply(_console, _toConsumableArray(t));
    },
    p: function p() {
      return _asyncToGenerator( /*#__PURE__*/regeneratorRuntime.mark(function _callee() {
        return regeneratorRuntime.wrap(function _callee$(_context) {
          while (1) {
            switch (_context.prev = _context.next) {
              case 0:
                return _context.abrupt("return", t1);

              case 1:
              case "end":
                return _context.stop();
            }
          }
        }, _callee);
      }))();
    }
  };
  obj.printArr();
  obj.p().then(function (s) {
    console.log(s);
  });
}
```

转换后的代码有点多，但这已经是babel根据browserslist指定环境转换的，我们可以看到babel提取了很多core-js中的实现，如：`require("core-js/modules/es.array.from.js")`等……，还有一些函数，如：`_asyncToGenerator`等我们称为babel运行时(后面会说到)。要弄清楚转译后的文件为何这样，我们得理解@babel/preset-env的配置项。

##### `useBuiltIns`

主要用于指定babel如何从core-js中提取合适的ES6新特性的实现，有两种模式：

`usage`：我们不需要在入口处导入core-js，babel会根据代码中使用的ES6 API来决定提取哪些。

`entry`：我们通过import在入口引入core-js，babel会根据引入的core-js模块来识别和拆分更细的导入，如

```javascript
// 源文件
import "core-js/es/array";
import "core-js/proposals/math-extensions";

// 输出
import "core-js/modules/es.array.unscopables.flat";
import "core-js/modules/es.array.unscopables.flat-map";
import "core-js/modules/esnext.math.clamp";
import "core-js/modules/esnext.math.deg-per-rad";
import "core-js/modules/esnext.math.degrees";
import "core-js/modules/esnext.math.fscale";
import "core-js/modules/esnext.math.rad-per-deg";
import "core-js/modules/esnext.math.radians";
import "core-js/modules/esnext.math.scale";
```

##### `corejs`

用于指定core-js的版本，因为core-js有2和3的版本，这里babel默认会使用2的版本，这里建议用3的版本。babel还建议指定core-js的minor版本，这样能将最新实现的API包含进来。默认情况下，babel只会提取稳定的API实现，如果你想将还在提案阶段的API也包含进来，可以这样配置：

```json
{
    "corejs": { version: "3.8", proposals: true }
}
```

总的来说，`useBuiltIns`配置为`usage`，`corejs`配置为`{ version: "3.8", proposals: true }`会是大部分场景的选择，不过后面还有更多的东西。。。

### babel运行时

为什么会有babel运行时，从上面的转译文件中我们可以看到有一些独立的函数，如：`asyncGeneratorStep`，`_iterableToArray`等。当有多个文件被转译的时候，babel会在每一个转译后的文件中根据需要生成这些函数，那么问题就是为什么每个转译后的文件不是共享一份这样的函数代码呢？所以这个时候@babel/runtime出现了，即所谓的babel运行时，跟webpack运行时一个道理。

@babel/runtime提供了独立的运行时代码，那babel是如何在转译过程中根据需要引入这些独立代码的呢？这个时候就要用到@babel/plugin-transform-runtime了。@babel/plugin-transform-runtime有个运行时依赖就是@babel/runtime，所以你在安装@babel/plugin-transform-runtime的时候，需要安装@babel/runtime或者@babel/runtime-corejs3或者@babel/runtime-corejs2作为运行时依赖，注意这里不是开发依赖。

##### `@babel/plugin-transform-runtime`

尽管@babel/plugin-transform-runtime的主要作用是处理运行时，但如果你将它的配置项：`corejs`配置为2或者3，那么它将会跟preset-env一样从core-js中提取对于的ES6新API的实现，只是它不会考虑目标环境也没有可选的core-js引入模式。所以它的`corejs`配置项默认为`false`表示不提取core-js中的ES6 API实现，在某些场景下这个提取工作默认是留给preset-env去做的。

还有重要的一点就是@babel/preset-env提取的core-js是直接注入到全局环境里的，即它会污染全局环境。而@babel/runtime提取的是core-js-pure，它不会污染全局环境，所以更适合用于独立的JS库。

### 最终配置

针对两种情况来配置，一种是通过打包工具创建APP，一种是通过打包工具创建自己的JS库。

上面说的，通过@babel/preset-env来提取指定环境需要的core-js包，但每个转译后的文件会出现重复的运行时代码，这时我们可以添加@babel/plugin-transform-runtime，并设置其corejs配置为false来统一处理运行时代码。

针对APP，可以做下面的配置：

```json
{
    "presets": [
        ["@babel/preset-env", {
            "useBuiltIns": "usage",
            "corejs": "3.22"
        }]
    ],
    "plugins": [
        ["@babel/plugin-transform-runtime"]
    ]
}
```

如果你在入口文件引入了core-js，那么useBuiltIns需要设置为entry，否则就用usage。这里@babel/plugin-transform-runtime的corejs配置项默认为false，所以babel会默认使用@babel/runtime来统一引入运行时代码。这里不要忘记安装运行时依赖@babel/runtime。

针对JS库，其实还可以细分两种情况，即我们的JS库要不要自带core-js，即所谓的polyfill。

如果不需要自带core-js，那我们可以只需要下面的配置：

```json
{
    "presets": [
        ["@babel/preset-env"]
    ]
}
```

@babel/preset-env只负责转换ES6语法，并不提取core-js的ES6 API实现。

如果你的JS库需要自带polyfill(core-js)，那我们选择用@babel/plugin-transform-runtime来提取core-js，因为preset-env无法统一处理运行时，虽然@babel/plugin-transform-runtime不考虑目标，但@babel/preset-env会从browserslist中读取目标环境，且@babel/plugin-transform-runtime不污染全局环境，所以可以用下面的配置：

```json
{
    "presets": [
        ["@babel/preset-env"]
    ],
    "plugins": [
        ["@babel/plugin-transform-runtime", {"corejs": 3}]
    ]
}
```

### 参考

https://babeljs.io/docs/en/babel-preset-env#targets

https://blog.jakoblind.no/babel-preset-env/

https://www.valentinog.com/blog/preset-env/

https://stackoverflow.com/questions/63231564/what-is-best-practice-for-babel-preset-env-usebuiltins-babel-runtime

https://www.jmarkoski.com/understanding-babel-preset-env-and-transform-runtime

https://github.com/babel/babel-polyfills/blob/main/docs/migration.md
