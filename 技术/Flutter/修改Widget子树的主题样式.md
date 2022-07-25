## 修改Widget子树的主题样式

如果我们希望一个页面上的某些微件具有跟根微件不同主题的主题样式，而其他的微件的主题样式不受影响，那么我们要怎么做呢？

我们知道`MaterialApp`类中有个`theme`属性，如果我们实例化一个`ThemeData`的实例赋值给theme，那么这样会修改整个应用的主题样式。所以这个方法可能不行。

正确的做法是将要修改主题样式的微件包装到一个容器微件中，设置容器的`child/children`属性为`Theme`的实例对象，然后修改`theme`的`data`属性为`ThemeData`实例，如下所示：

```dart
Center(
    child: Theme(
        data: ThemeData(
            cardColor: Colors.deepOrangeAccent
        ),
        child: Card(
            child: Text('Unique ThemeData')
        )
    )
)
```

`Theme`的`child`属性是必填的。上面的代码会将`Center`里面所有子微件的主题样式全部重置为`ThemeData`的样子。

上面的方法虽然达到了目的，但是却将`Center`的所有主题样式都给改了，如果我们只是想改变某些想改变的样式，比如只是想改变某些字体颜色等

此时我们可以通过`Theme.of`方法来扩展父级的主题样式，然后修改想要改的即可。

```dart
Center(
    child: Theme(
        data: Theme.of(context).copyWith(cardColor: Colors.lightGreen),
        child: Card(
            child: Text('Unique ThemeData')
        )
    )
)
```

## 其他知识点

- 可以通过`theme`属性修改`MaterialApp`的主题样式，theme的值时一个`ThemeData`实例。
- `ThemeData`类提供了各种样式的设置，比如`primaryColor`，`canvasColor`等
- 主题样式会根据所运行的环境不同而表现不同，如果想在当前环境查看其他不同环境的样式表现，需要设置`ThemeData`的`platform`属性。`platform`属性取值来自`TargetPlatform`类，此类定义了`iOS`，`android`等多个不同的运行环境。
