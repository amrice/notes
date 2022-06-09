## 问题

将TextButton用在Wrap中的时候，尽管我们设置spacing和runSpacing为相同的值，发现垂直方向比水平方向要大出很多。如下图：

<img src="https://cdn.jsdelivr.net/gh/ywxgod/image_source/md-imgs/202205281345185.png" title="" alt="" data-align="center">

```dart
home: Scaffold(
    appBar: AppBar(title: Text('MP3Player'),),
    body: Wrap(
       spacing: 10,
       runSpacing: 10,
       children: List.generate(10, (index) {
          return Builder(builder: (context) {
              return TextButton(
                onPressed: () {print('what');},
                child: const Text('what'),
                style: TextButton.styleFrom(
                    primary: Colors.blue[index*100],
                    backgroundColor: Colors.grey[index*100]
                ),
              );
          });
        }),
    ),
)
```

通过Flutter Inspector可以看到TextButton的上下像是多出了一些间隔，现在的问题是如何将其取消。

## 解决

设置TextButton的style属性，添加`tapTargetSize`即可

```dart{6}
return Builder(builder: (context) {
     return TextButton(
          onPressed: () {print('what');},
          child: const Text('what'),
          style: TextButton.styleFrom(
               tapTargetSize: MaterialTapTargetSize.shrinkWrap,
               primary: Colors.blue[index*100],
               backgroundColor: Colors.grey[index*100]
          ),
      );
});
```