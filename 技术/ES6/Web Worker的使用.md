web worker一般用作密集型运算的应用中，比如处理较大数据，超大JSON响应，网络轮询，分析视频、音频数据，canvas中图片像素处理等等。

web worker可以分为两类，一类是专用Worker，一类是共享Worker。

## 专用Worker

专用worker就是只能被生成它的脚本所使用的worker。

### 开始使用

Worker执行的代码需要放在一个独立的文件中，Worker的构造函数接收这个文件的路径

```javascript
let worker = new Worker('./task.js');
```

上面的代码是在主线程中执行的，它会创建一个worker，此worker会在不同的线程中执行，它的全局作用域也不是window对象。

### 通信问题



### 错误处理

