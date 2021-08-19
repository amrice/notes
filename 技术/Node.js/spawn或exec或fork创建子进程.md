Nodejs是单线程运行的。child_process模块却允许我们创建子进程，且这些子进程之间的通讯，可以利用内建的消息机制很容易做到。

我们可以有4种方法创建子进程：spawn(), fork(), exec(), execFile()

spawn可以启动一个命令，如下：

```jsx
const { spawn } = require('child_process');
const child = spawn('ls', ['-a', '-l']);
```

spawn的第二个参数可以是数组，用于给要执行的命令传递多个参数。运行上面的代码你可能什么都看不到，因为ls的输出存在于子进程的stdout中，所以我们如何能看到输出？有三种方法：

我们可以用for await ... of ：

```jsx
(async ()=>{
    for await (const data of child.stdout) {
        console.log(`stdout from the child: ${data}`);
    }
})();
```

也可以在stdout对象上监听data事件：

```jsx
child.stdout.on('data', data => {
    console.log(data);
});

// 输出可能是这样的：
// <Buffer 74 6f 74 61 6c 20 33 32 0a 64 72 77 78 72 77 78 72 77 78 20 20 20 36 20 31 31 30 39 38 37 33 30 20 20 56 49 56 4f 5c 44 6f 6d 61 69 6e 20 55 73 65 72 ... 368 more bytes>

// 我们可以设置一下编码将Buffer转为字符串。
// child.stdout.setEncoding('utf8'); 或者
// console.log(data.toString('utf8'));

```

也可以将子进程的输出流通过管道导入到父进程的输入流，如：

```jsx
child.stdout.pipe(process.stdin);
```

如果我们用spawn在windows系统上执行dir命令，如下：

```jsx
const child = child_process.spawn('dir');
child.on('error', err => {
    console.log(err);
});
```

你会看到如下错误信息：

```
Error: spawn dir ENOENT
    at Process.ChildProcess._handle.onexit (internal/child_process.js:267:19)
    at onErrorNT (internal/child_process.js:469:16)
    at processTicksAndRejections (internal/process/task_queues.js:84:21) {
  errno: 'ENOENT',
  code: 'ENOENT',
  syscall: 'spawn dir',
  path: 'dir',
  spawnargs: []
}
```

所以怎么解决？

```jsx
const child = child_process.spawn('cmd.exe', ['/c','dir', '/w']);
child.on('error', err => {
    console.log(err);
});
child.stdout.pipe(process.stdout);
```

我们可以改为上面的方式，即执行cmd，通过cmd再传入要执行的命令即可。当然我们也可以将命令放到一个批处理文件中，如下：

```jsx
const { spawn } = require('child_process');
const bat = spawn('cmd.exe', ['/c', 'my.bat']);

bat.stdout.on('data', (data) => {
  console.log(data.toString());
});

bat.stderr.on('data', (data) => {
  console.error(data.toString());
});

bat.on('exit', (code) => {
  console.log(`Child exited with code ${code}`);
});
```

spawn返回的是ChildProcess类的一个实例，而ChildProcess是继承自EventEmitter，所以我们可以用ChildProcess实例来监听事件，比如上面的data，exit事件。除了data，exit事件，还有disconnect，error，close，message等事件。message事件主要是用于主进程和子进程间通信。当在子进程中调用process.send()方法时，message事件就会触发。

exit事件是在进程退出时触发，此时可能该进程创建的异步处理流并未处理完成。而当流结束后会触发close事件。所以close事件与exit事件是有区别的。

下面看看spawn联合处理的例子，要达到如下命令执行的效果，该如何做到？

```jsx
ls | grep "abc.js"
```

我们可以这样：

```jsx
const lsChild = spawn('ls');
const grepChild = spawn('grep', ['abc.js']);
lsChild.stdout.pipe(grepChild.stdin); // 将ls的输出接入到grep，作为输入
grepChild.stdout.pipe(process.stdin); // 将grep的输出接入到父进程的输入
```

因为spawn默认是不调用shell/cmd来执行命令的，所以上面的这种多命令处理就需要通过spawn创建多个子进程来处理。不过我们可以开启spawn的shell配置，来让spawn调用shell，如下:

```jsx
const child = spawn('ls | grep "abc.js"', {shell: true});
child.stdout.pipe(process.std);
```

这样就可以不需要多次调用spawn，只需要启动shell，一次调用spawn，传入多个关联命令即可。

而exec默认是会调用shell的，所以上面的联合命令可以直接用exec来处理，如下：

```jsx
exec('ls | grep "abc.js"', (error, stdout, stderr) => {
	if(error) return;
	console.log(stdout);
})
```

exec会将第一个参数直接传给shell/terminal，这样只要是合法的命令都会被执行。跟spawn还有一个不同的是，它用了回调函数的形式。回调函数第二个参数即为命令的输出，exec会默认缓存所有输出到buffer，然后一次输出，且输出时会用utf8将buffer进行转码。所以上面的代码中，我们只需要在回调函数中打印stdout即可。它不是流而是字符串，不过我们可以改变其默认编码，将其改为buffer类型的。

如果你已经不习惯用回调函数的形式调用exec，我们可以用promisify方法来将其处理成Promise形式的。如下：

```jsx
const { exec } = require('child_process');
const utils = require('util');
const promisifyExec = utils.promisify(exec);
(async () => {
    const { stdout, stderr } = await promisifyExec('ls | grep "001"');
    if(stderr) {
        console.log(stderr);
        return;
    }
    console.log(stdout);
})();
```

execFile跟exec一样的地方，也是用回调函数的形式来返回结果，不同的是需要用数组来传参数给命令，且它默认也是不调用shell的。execFile的第一个参数是可执行文件的文件名或者路径，而exec的第一个参数是要执行的命令名称(和参数)，如：

```jsx
const { execFile } = require('child_process');
const child = execFile('node', ['--version'], (error, stdout, stderr) => {
  if (error) {
    throw error;
  }
  console.log(stdout);
});
```

exec，execFile，spawn提供的功能都是执行各种shell命令或者操作系统提供的可执行文件，而fork提供了我们执行js文件的功能，我们可以将一些cpu计算密集型的逻辑放到单独的js文件中，通过fork来执行，这样就不影响主进程。

```jsx
// 主进程文件main.js
const http = require('http');
const { fork } = require('child_process');

const server = http.createServer();
server.listen(3000);
server.on('request', (req, res) => {
    if (req.url !== '/') return;
		// 通过fork启动子进程
    const compute = fork('./compute.js');
		// 向fork进程发送消息，通知其执行任务
    compute.send('start');
    const t = process.hrtime();
		// 监听子进程执行任务完成后发送过来的消息
    compute.on('message', result => {
        console.log(result, Number.MAX_SAFE_INTEGER, process.hrtime(t));
        res.end(JSON.stringify(result));
    });
});
```

fork函数只需要传入我们需要执行的js文件路径即可，fork进程与父进程之间的通信是通过message事件来处理的。

```jsx
// 子进程文件compute.js
const longComputation = () => {
    let sum = 0;
    let i = 0;
    for(;i<1e10;i++) {
        sum += i;
        if (sum > Number.MAX_SAFE_INTEGER) {
            sum -= i;
            break;
        }
    }
    return { sum, i };
};
// 监听父进程的消息，以便启动任务
process.on('message', message => {
    if (message !== 'start') return;
    const result = longComputation();
		// 任务执行完通过send方法发送消息给父进程
    process.send(result);
})
```