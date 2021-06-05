![](https://cdn.jsdelivr.net/gh/ywxgod/image_source/imgs20210605160641.jpg)

我们在构建flutter应用的时候，可能会遇到Widget树变得越来越庞大的情况，最后就变得很难维护。作为一个前端，不管是`vue`还是`react`我们都是将其尽量分解为单个的组件文件来处理，这是最佳实践。同理，flutter也是一样的。

先展示一下需要分解的代码，庞大的widget树：

```dart
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Home'),
      ),
      body: SafeArea(
        child: SingleChildScrollView(
          child: Padding(
            padding: EdgeInsets.all(16.0),
            child: Column(
              children: <Widget>[
                Row(
                  children: <Widget>[
                    Container(
                      color: Colors.yellow,
                      width: 40,
                      height: 40,
                    ),
                    Padding(padding: EdgeInsets.all(16.0)),
                    Expanded(
                        child: Container(
                          width: 40,
                          color: Colors.amber,
                          height: 40,
                        )
                    ),
                    Padding(padding: EdgeInsets.all(16.0)),
                    Container(
                      color: Colors.brown,
                      width: 40,
                      height: 40,
                    ),
                  ],
                ),
                Padding(padding: EdgeInsets.all(16)),
                Row(
                  children: <Widget>[
                    Column(
                      crossAxisAlignment: CrossAxisAlignment.start,
                      mainAxisSize: MainAxisSize.max,
                      children: <Widget>[
                        Container(
                          color: Colors.yellow,
                          width: 60,
                          height: 60,
                        ),
                        Padding(padding: EdgeInsets.all(16),),
                        Container(
                          color: Colors.amber,
                          width: 40,
                          height: 40,
                        ),
                        Padding(padding: EdgeInsets.all(16),),
                        Container(
                          color: Colors.brown,
                          width: 20,
                          height: 20,
                        ),
                        Divider(),
                        Row(
                          children: [
                            CircleAvatar(
                              backgroundColor: Colors.lightGreen,
                              radius: 100,
                              child: Stack(
                                children: [
                                  Container(
                                    width: 100,
                                    height: 100,
                                    color: Colors.yellow
                                  ),
                                  Container(
                                    height: 60,
                                    width: 60,
                                    color: Colors.amber,
                                  ),
                                  Container(
                                    height: 40,
                                    width: 40,
                                    color: Colors.brown
                                  )
                                ],
                              ),
                            ),
                          ],
                        ),
                        Divider(),
                        Text('End of the Line')
                      ],
                    )
                  ],
                ),
                Row(
                  children: [
                    Container(
                      color: Colors.lightBlue,
                      width: MediaQuery.of(context).size.width - 32,
                      height: 40,
                    )
                  ],
                )
              ],
            ),
          )
        ),
      ),
    );
  }
```

上面是《Flutter入门经典》一书中的一个例子，我们可以通过一下三种方法来分解这个庞大的widget树。

## 通过常量

将树中的某些widget提出来赋值给一个常量，下面这样的

```dart
final container = Container(
    color: Colors.yellow,
    height: 40,
    width: 40
);
```

需要注意的是这里要用final，当通过常量初始widget时，它会依赖父级的`BuildContext`实例对象，也就是说每次重绘父Widget时，所有常量对应的Widget也会被重绘，这是有损性能。

## 通过函数

将树中的某些独立的widget提出来放进一个函数里，父级通过调用这个函数返回对应的widget来进行渲染。

```dart
Widget _buildContainer(double width, double height, Color color) {
    return Container(
        color: color,
        height: height,
        width: width
    );
}
```

这种方法创建的widget还是依赖父级的`BuildContext`对象，还有上面函数的返回值，我们可以用Container，也可以用Widget。一个是具体的Widget类，一个是表示更加通用的场景。

跟通过常量创建Widget一样，这种方法中也会有性能损耗，并不是最好的。

### 界面操作

Android Studio中提供了界面操作来提取widget树中的某段代码。我们只需要将鼠标放到你需要提取的widget的代码上面，点击右键

![image-20210605152200622](https://cdn.jsdelivr.net/gh/ywxgod/image_source/imgs20210605152728.png)

点击**Extract Method…**后，会出现一个弹出框，输入对于的函数名，Android Studio会自动创建对于的函数，且将树中原代码替换成函数调用。

在**Refactor**的子菜单中，我们还可以发现很多其他的功能，比如：**Extract Flutter Widget…** 这个就是我们下面一种方法的界面操作了。

## 通过Widget类

这个类似于`Vue`中的单文件组件，原理是一样的。

将树中的代码提取到类中，首先我们要决定这个类是继承`StatelessWidget`还是`StatefulWidget`，如果你的Widget类需要一个State实例，有自己的状态需要处理，那么就用`StatefulWidget`，否则用`StatelessWidget`。

为了满足widget的缓存性能要求，我们需要在提取的Widget类的构造函数前添加`const`修饰，`const`会告诉Flutter这个组件需要缓存和重用。

下面是提取了其中的一个Row出来的代码：

```dart
class RowWidget extends StatelessWidget {
  const RowWidget({
    Key key,
  }) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return Row(
      children: <Widget>[
        Container(
          color: Colors.yellow,
          width: 40,
          height: 40,
        ),
        Padding(padding: EdgeInsets.all(16.0)),
        Expanded(
            child: Container(
              width: 40,
              color: Colors.amber,
              height: 40,
            )
        ),
        Padding(padding: EdgeInsets.all(16.0)),
        Container(
          color: Colors.brown,
          width: 40,
          height: 40,
        ),
      ],
    );
  }
}
```

调用的地方我们可以改成

```dart
const RowWidget()
```

注意上面的类构造函数前面的`const`关键字，还有调用的时候前面也加了`const`关键字。

当然在实际的应用中，我们可能需要通过构造函数传入各种参数来实例化我们的Widget，这个根据需要在处理了。

至于分解的粒度怎么控制，这个可以根据具体的功能来决定。

## 总结

出于性能考虑，建议使用第三种方式来分解Widget树。不过要注意的是，在创建Widget类的时候，构造函数和调用构造函数时都需要增加`const`关键字来保证Widget的缓存和重用。