### 模块查找

当你在用require导入模块的时候，require函数会调用`resovle`方法找出模块的唯一全局路径，一般会经历下面的步骤：

```javascript
require.resolve(moduleName)
```

##### 一、判断相对或绝对路径

1、确定`moduleName`是否以`/`或`./`开头，如果是`/`，则认为是绝对路径，直接读取；如果是`./`则认为是相对路径（相对于载入该模块的这个目录），找出其绝对路径，然后进入`2`;如果既不是相对路径也不是绝对路径，则进入`3`。

2、进一步判断`moduleName`是文件路径还是目录路径。如果是目录，则看该目录下面是否有`index.js`，如果有则加载；没有，则看该目录下是否有`package.json`，且`package.json`中的`main`属性有一个合法的文件地址，然后加载`main`指定的文件。如果不是目录，则直接加载`moduleName.js`，如果`2`中的所有尝试都失败，则报错终止。

##### 二、判断是否是内建模块

3、查找`Node.js`的内建模块中是否有名称为`moduleName`的，如果有，则加载；如果没有则进入`4`。

##### 三、判断是否是依赖包中的模块

4、从载入`moduleName`的模块所在的目录开始查找，逐层查找是否存在`node_modules`目录，如果有，则查看`node_modules`中是否有名称为`moduleName`的模块，有则进入`2`；如果没有，则查找到根目录，直到报错终止。

### 模块缓存

require通过resolve查找到模块的全局唯一路径，将其作为键，模块内容作为值，存在`require.cache`中。当我们多次加载同一个模块时，获取到的是同一个实例。

### 模块加载

下面看看`require(moduleName)`时，发生了什么。

1、调用`require.resolve`找出`moduleName`的全局唯一路径，将其保存在变量id中。

2、将id去`require.cache`中检查，看是否有加载过，有的话直接返回缓存中的模块实例。

3、如果缓存中没有id对应的模块，则生成一个module对象，如下

```javascript
const module = {
    exports: {},
    id
};
```

此对象包含一个exports字面量空对象和id属性，id为`2`中的id值

4、将`3`中的module对象加入缓存

```javascript
require.cache[id] = module;
```

5、加载模块内容，这里单独写入一个`loadModule`函数中，大概思路如下

```javascript
function loadModule(id, module, require) {
    const wrappedSrc = 
          `(function(module, exports, require){
          ${fs.readFileSync(id, 'utf8')}
          })(module, module.exports, require)`;
    eval(wrappedSrc);
}
```

`loadModule`是通过同步方式加载`js`模块的，同时传入id-文件路径，module对象，exports对象-module对象中的exports，全局require方法。

6、因为我们定义模块时，会填充`module.exports`对象，所以在模块经过加载后，`module.exports`对象会有该模块导出的`api`，所以我们只需要将其作为require函数的返回值即可。

##### 大概的实现代码如下

```javascript
require.resolve = function(moduleName){ ... };
require.cache = {};

function require(moduleName){
    // 查找全局唯一路径
    const id = require.resolve(moduleName);
    // 是否已经缓存过
    if (require.cache[id]) {
        return require.cache[id].exports;
    }
    // 构建module对象
    const module = {
        exports: {},
        id
    };
    // 将module对象写入缓存
    require.cache[id] = module;
    // 加载模块
    loadModule(id, module, require);
    // 返回填充后的exports对象
    return module.exports
}
```

### 循环依赖

循环依赖的一个场景就是，你在入口文件中加载`a.js`，然后`a.js`中加载了`b.js`，而`b.js`中又加载了`a.js` …… 这就形成了循环加载，循环依赖。

通过上面require函数，我们知道模块在加载之前就已经被缓存了，它加载的作用只是填充了`module.exports`对象。所以当`loadModule`在加载`a.js`的时候，如果遇再到加载`a.js`，其实是缓存中的同一个`module.exports`对象，至于对象中有那些数据，就要看循环加载`a.js`时，`loadModule`中将`module.exports`对象填充了多少。

### 参考

文章内容参考自：《Node.js设计模式第三版》的第二章

