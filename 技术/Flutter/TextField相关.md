Flutter 3.3.4版本中

通过addListener监听TextField的改变时，如果TextField的autofocus初始为true，那么当TextField被第一次渲染的时候，监听函数会被执行一次，即使我们并未改变TextField的值

具体例子：

```dart
TextField(
    autofocus: true,
    controller: _controller,
    decoration: const InputDecoration(
      labelText: '姓名',
      // border: InputBorder.none, // 去掉默认的底部边框, OutlineInputBorder设置全部边框
    ),
)
```

```dart
late TextEditingController _controller;

TextFieldDemoState() {
    _controller = TextEditingController();
    _controller.text = '宋江';
}

@override
void initState() {
    // TODO: implement initState
    super.initState();
    _controller.addListener(() {
      debugPrint(_controller.text);
    });
}
```

像上面的代码，app启动后会输出一次'宋江'，因为那是初始值，如果不在TextFieldDemoState中设置初始值，则会输出空白。这往往不是我们想要的效果，我们只是像在用户改变文本值是有反应而已。此时我们可以该有onChanged属性，如下

```dart
void onTextChanged(String text) {
    debugPrint(text);
}

TextField(
    autofocus: true,
    onChanged: onTextChanged,
    controller: _controller,
    decoration: const InputDecoration(
      labelText: '姓名',
      // border: InputBorder.none, // 去掉默认的底部边框, OutlineInputBorder设置全部边框
    ),
),
```

不过在输入中文的时候，没输入一个中文字符，onChanged函数会被执行两次，这里可以通过Timer来做debunce效果

```dart
/// 定义debouce返回的函数类型 - 对应于TextField的onChanged类型
typedef OnTextChangedCb = void Function(String text);
/// 定义deounce函数
OnTextChangedCb? debounce(OnTextChangedCb callback, int delayMs) {
    Timer? timer;
    return (String text) {
      if (timer?.isActive ?? false) timer?.cancel();
      timer = Timer(Duration(milliseconds: delayMs), () {
        callback(text);
      });
    };
}

/// 应用到onChanged属性
TextField(
    autofocus: true,
    onChanged: debounce(onTextChanged, 500),
    controller: _controller,
    decoration: const InputDecoration(
      labelText: '姓名',
      // border: InputBorder.none, // 去掉默认的底部边框, OutlineInputBorder设置全部边框
    ),
),
```
