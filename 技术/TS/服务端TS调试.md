这里说的是服务端ts的调试，用了ts-node直接跑起来的，没有先用ts编译然后再用node执行。用的node.js版本为17.2.0。在开始说调试之前，先记录一下运行ts-node时的一个报错怎么处理。

## 一个报错

因为想用ESM的标准来写ts文件，所以按照ts-node文档，做了下面的配置：

>   You must set "type": "module" in package.json and "module": "ESNext" in tsconfig.json.

```json
{
  "type": "module"
}
```

```json
{
  "compilerOptions": {
    "module": "ESNext" // or ES2015, ES2020
  }
}
```

然后执行

```shell
ts-node src/main.ts
```

就会看到下面的错误信息，意思是说识别不了ts文件

```shell
TypeError [ERR_UNKNOWN_FILE_EXTENSION]: Unknown file extension ".ts" for X:\xx\xx\src\main.ts
    at new NodeError (node:internal/errors:371:5)
    at Object.file: (node:internal/modules/esm/get_format:72:15)
    at defaultGetFormat (node:internal/modules/esm/get_format:85:38)
    at defaultLoad (node:internal/modules/esm/load:22:14)
    at ESMLoader.load (node:internal/modules/esm/loader:353:26)
    at ESMLoader.moduleProvider (node:internal/modules/esm/loader:274:58)
    at new ModuleJob (node:internal/modules/esm/module_job:66:26)
    at ESMLoader.#createModuleJob (node:internal/modules/esm/loader:291:17)
    at ESMLoader.getModuleJob (node:internal/modules/esm/loader:255:34)
    at async Promise.all (index 0) {
  code: 'ERR_UNKNOWN_FILE_EXTENSION'
}
```

## 尝试解决

从错误堆栈看，是node内置的esm/loader识别不了ts文件，所以尝试将ts扩展名改为mjs，在ts文件中没有类型声明的时候，这个是可以的。但这没意义了，因为这个项目就是要用ts的，所以不行；尝试去掉package.json中的`"type": "module"`，然后在ts文件中用CMD的方式引入模块，发现也是可以的，但这也没意义，因为这个项目需要用ESM的标准来引入模块的。

## 最终的方法

```shell
node --loader ts-node/esm src/main.ts
```

通过node的--loader参数将其默认的模块加载器(esm/loader)替换为ts-node/esm，不过控制台会出现下面的警告，因为参数目前是实验性的。

```shell
(node:5088) ExperimentalWarning: --experimental-loader is an experimental feature. This feature could change at any time
(Use `node --trace-warnings ...` to show where the warning was created)
```

更多信息可以看这个

https://nodejs.medium.com/announcing-a-new-experimental-modules-1be8d2d6c2ff

## Webstorm上的调试

虽然有警告，但上面的代码总是跑起来了，我用的webstorm版本是2020.3.3，先看看JB上是如何告诉我们来调试ts-node+Typescript的

![image-20220123114712974](https://cdn.jsdelivr.net/gh/ywxgod/image_source/imgs20220123114713.png)

安装上面的步骤，得出最终结果

![image-20220123115621694](https://cdn.jsdelivr.net/gh/ywxgod/image_source/imgs20220123115621.png)


这里有node的参数比JB文档说的多了2个。第一个`--loader`参数是前面那个错误导致我们必须要加上的，而第二个参数`--inspect`是为了启动node的调试，正常情况下我想这个参数应该可以不用加的，因为我们通过webstorm的debug按钮来启动程序的，它会自动处理。但此处如果不加`–-inspect`，你又会看到如下报错

```shell
"C:\Program Files\nodejs\node.exe" --require ts-node/register --loader ts-node/esm X:\xx\xx\src\main.ts
Debugger listening on ws://127.0.0.1:4445/7ce8b9ce-647f-4662-8199-8ac7cc55fede
For help, see: https://nodejs.org/en/docs/inspector
Error in debuggerConnector: connect ECONNREFUSED ::1:4442
Error: connect ECONNREFUSED ::1:4442
```

## 在Chrome中调试

上面的步骤只是在Webstorm中将调试程序跑起来了，具体调试我们需要在Chrome中进行。打开一下网址：chrome://inspect，一段时间你会看到我们刚刚跑起来的调试程序。

![image-20220123120232710](https://cdn.jsdelivr.net/gh/ywxgod/image_source/imgs20220123120232.png)

点击inspect按钮，会打开一个新的只有devtools的chrome窗口

![image-20220123120439294](https://cdn.jsdelivr.net/gh/ywxgod/image_source/imgs20220123120439.png)

到此Webstorm相关的开发调试就可以正常进行了。下面再看看VSCode上面是怎么调试的

## VSCode上的调试

我用的版本是1.63.2，先用VSCode打开项目目录，然后创建一个如下内容的launch.json

```json
{
    // Use IntelliSense to learn about possible attributes.
    // Hover to view descriptions of existing attributes.
    // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
        {
            "type": "node",
            "request": "launch",
            "name": "ts-node debug",
            "skipFiles": [
                "<node_internals>/**"
            ],
            "runtimeArgs": ["-r", "ts-node/register", "--loader", "ts-node/esm"],
            "program": "${workspaceFolder}\\src\\main.ts"
        }
    ]
}
```

这里除了runtimeArgs属性，其他的都是默认生成的，runtimeArgs的作用就是给node设置运行时参数，上面的-r，--loader两个参数的意思跟在Webstorm里面设置是一样的意思。然后直接可以在ts文件上设置断点(如下图中的红点）,点击左侧的绿色三角形按钮开始调试。

![image-20220123121751580](https://cdn.jsdelivr.net/gh/ywxgod/image_source/imgs20220123121751.png)

这里不需要打开Chrome进行调试，比Webstorm方便许多。所以写ts的还是用VSCode比较方便吧

## 参考

https://nodejs.medium.com/announcing-a-new-experimental-modules-1be8d2d6c2ff

https://www.jetbrains.com/help/webstorm/running-and-debugging-typescript.html#ws_ts_run_debug_server_side

https://github.com/TypeStrong/ts-node#commonjs-vs-native-ecmascript-modules

https://blog.appsignal.com/2022/01/19/how-to-set-up-a-nodejs-project-with-typescript.html

https://nodejs.org/en/docs/guides/debugging-getting-started/

