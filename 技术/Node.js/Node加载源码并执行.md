项目中遇到一个问题，就是需要远程获取一段js函数代码，然后在在本地执行。这个函数有参数，本地执行的时候，可以传入不同的参数来获取不同的返回。这个可以通过`vm`模块来实现，下面是记录实现的过程。

先在gitee上放一个scripts.js文件，比如内容如下：

```javascript
function getChar(flag) {
    return flag ? 1 : 0;
}
```

从gitee上把js文件拉取下来，这要用到gitee提供的open api，为了使用open api，你还得先申请一个token，具体需要登录gitee去设置里处理。

```javascript
async function getFileFromRemote(filename) {
    const owner = '用户名';
    const repo = '仓库名称';
    const path = filename; // 相对仓库的文件路径
    const ref = 'master'; // 文件所在分支名称
    // 获取文件sha
    let giteeApi = `${giteeBaseUrl}/${owner}/${repo}/contents/${path}`;
    giteeApi += `?access_token=${token}&ref=${ref}`;
    let { data } = await axios.get(giteeApi).catch(e => console.log(e));
    // 根据文件sha获取文件内容
    giteeApi = `${giteeBaseUrl}/${owner}/${repo}/git/blobs/${data.sha}`;
    giteeApi += `?access_token=${token}`;
    data = await axios.get(giteeApi).catch(e => console.log(e));
    // 将base64编码的文件内容转为utf8字符串
    return Buffer.from(data.data.content, 'base64').toString();
}
```

要获取gitee仓库中的某个文件，我们需要做2步，即：先拿到文件的sha，然后用这个sha去拿文件的具体内容。不过文件的内容是base64编码的，所以最好还需要处理成utf8的。

要执行js，我们得用VM模块，这里演示的是一种非常简单的情形，不考虑执行环境。更多细节可以查看文档。

```javascript
function runScript(script) {
    const scriptStr = 'x = ' + script;
    const scriptObj = new Script(scriptStr);
    const result = scriptObj.runInThisContext();
    console.log(result(1)); // 1
    console.log(this.x(1)); // 1
    console.log(global.x(1)); // 1
}
```

script参数就是从远程获取的文件内容，我们这里因为是一种非常简单的情形，函数直接附加到全局作用域中，然后调用vm中的Script类来实例化一个Script实例，这个实例提供多执行代码的方法，我们可以通过这些方法来改变函数的执行作用域，这里我们可以用`runInThisContext`方法，它返回的就是远程代码中的函数对象。

假如我们把代码改为下面这样的

```javascript
const scriptStr = 'x = {a:0}; x.b = ' + script;
```

实际是在全局作用域定义了x对象，函数被赋值给了x的b属性，那么我们可以像下面这样调用

```javascript
const scriptStr = 'x = {a:0}; x.b = ' + script;
const scriptObj = new Script(scriptStr);
const result = scriptObj.runInThisContext();
console.log(this.x.b.call(this.x, this.x.a));
```

通过call来改变函数的作用域。

上面只是一种改变作用域的手段，不过上面的Script实例提供了很多执行方法来处理函数执行时的作用域，有时间可以尝试一下。

如果想加载本地的js文件执行，也是可以的，不过我们没必要这么做。。。。因为你可以通过require，import等直接导入文件内容。

如果要在浏览器环境执行代码有办法呢？…