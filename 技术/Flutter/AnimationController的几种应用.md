`AnimationController`的大概作用是在一定的时间内，按照屏幕刷新率（帧频）在指定的范围内按照顺序（顺序、倒叙）产生一些数据。默认是产生0到1之间的数字。

### 基本概念

`AnimationController`实例化的时候至少需要两个东西，`duration`用于指定时长，`vsync`用于提供TickerProvider，TickerProvider就像一个时钟，可以以固定的频率产生事件。往往我们通过组合`SingleTickerProviderStateMixin`类让当前类变成一个TickerProvider。如下初始化过程：

```dart
class MyState extends State<MyStatefulWidget> 
                                    with SingleTickerProviderStateMixin {
    late AnimationController _controller;

    @override
    void initState() {
        super.initState();
        _controller = AnimationController(
            vsync: this,
            duration: const Duration(seconds: 10)
        );
        _controller.forward();
    }
}
```

我们在initState函数中实例化`AnimationController`，并通过forward方法启动`AnimationController`，它将在10秒内以固定的频率不断产生0到1之间的数字。这里需要留意的是要在`initState`函数中初始化。

接下来就可以通过添加listener来查看其产生的值，如下

```dart
@override
void initState() {
     super.initState();
     _controller = AnimationController(
         vsync: this,
         duration: const Duration(seconds: 10)
     );
     _controller.addListener((){
        debugPrint('${_controller.value}, --');
     });
     _controller.forward();
}
```

控制台看到类似如下的输出就对了

```shell
I/flutter (13413): 0.9610909, --
I/flutter (13413): 0.9649901, --
I/flutter (13413): 0.9705375999999999, --
I/flutter (13413): 0.9749848999999999, --
I/flutter (13413): 0.9783161, --
I/flutter (13413): 0.9816525, --
I/flutter (13413): 0.9833181, --
I/flutter (13413): 0.9883176, --
I/flutter (13413): 0.9916456, --
I/flutter (13413): 0.9949795, --
I/flutter (13413): 1.0, --
```

有了上面的这些随时间变化的数字，我们就可以将其应用到widget中，改变widget的颜色，形状，位置等等继而形成动画。

### 应用一: 改变文本的fontSize

先创建一个App类命名为`TestApp1`，继承`StatelessWidget`，重写`build`方法，返回`MaterialApp`，`MaterialApp`的home指定为`TestAppHome`实例。

```dart{codeTitle:TestApp1.dart}
class TestApp1 extends StatelessWidget {
  const TestApp1({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    // TODO: implement build
    return MaterialApp(
      theme: ThemeData(
        primarySwatch: Colors.orange
      ),
      title: 'TestApp1',
      home: const TestAppHome(),
    );
  }
}
```

`TestAppHome`类继承`StatefulWidget`，并重写`createState`方法，返回`TestAppState`实例。

```dart{15,22}{codeTitle:TestAppHome.dart}
class TestAppHome extends StatefulWidget {
  const TestAppHome({Key? key}) : super(key: key);

  @override
  State<StatefulWidget> createState() {
    // TODO: implement createState
    return TestAppState();
  }
}

class TestAppState extends MyAnimationState {
  @override
  Widget build(BuildContext context) {
    // TODO: implement build
    double v = controller.value;
    return Scaffold(
      appBar: AppBar(
        title: const Text('TestApp1'),
      ),
      body: Container(
        alignment: Alignment.center,
        child: Text('hello', style: TextStyle(fontSize: 60+v*30, color: DataUtil.randomColor),),
      ),
    );
  }
}
```

`TestAppState`类继承`MyAnimationController`类，注意15行的变量`v`，将其应用到22行的`TextStyle`的`fontSize`。下面看看`MyAnimationController`的实现

```dart{12-17,20-24}{codeTitle:MyAnimationController.dart}
import 'package:flutter/material.dart';

class MyAnimationState extends State with SingleTickerProviderStateMixin {

  late AnimationController _controller;

  get controller { return _controller; }

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
        vsync: this,
        duration: const Duration(seconds: 1)
    );
    _controller.addListener(onTick);
    _controller.repeat(reverse: true);
  }

  void onTick() {
    setState(() {

    });
  }

  @override
  void dispose() {
    super.dispose();
    _controller.dispose();
  }

  @override
  Widget build(BuildContext context) {
    // TODO: implement build
    throw UnimplementedError();
  }

}
```

这里代码具体的解释就不多说了，一看就懂，最终效果：

<img title="" src="https://cdn.jsdelivr.net/gh/ywxgod/image_source/md-imgs/202206052301540.gif" alt="" data-align="center" width="132">

### 应用二：改变组件的scale属性

这里重用上面的TestApp1，`TestAppHome`和`MyAnimationState`类，创建`TestApp2State`继承`MyAnimationState`，然后重写build方法

```dart
class TestApp2State extends MyAnimationState {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('TestApp2'),
      ),
      body: Center(
        child: AnimatedBuilder(
            animation: controller,
            builder: (context, child) {
                return Transform.scale(
                  scale: controller.value,
                  child: child,
                );
            },
            child: Container(
              width: 150,
              height: 150,
              color: DataUtil.randomColor
            ),
        ),
      ),
    );
  }
}
```

应用一中是在listener的处理函数中调用`setState`方法促使build方法执行，这次是利用`AnimatedBuilder`类来直接处理需要做动画的子组件，将`controller.value`直接赋值给`Transform.scale`方法的`scale`参数。最终的效果如下：

<img title="" src="https://cdn.jsdelivr.net/gh/ywxgod/image_source/md-imgs/202206052332690.gif" alt="" data-align="center" width="142">

### 应用三：设置color属性改变颜色

在应用二中`Container`的颜色也是在不断变化的，那是因为每次build执行时，随机设置了`Container`的`color`属性，所以看上去像是在闪烁的效果。而应用三要实现的颜色变化是一个颜色过渡动画，即从一种颜色过渡到另一种颜色。

要实现颜色过渡动画，我们可以借助`ColorTween`类。在initState中初始化`ColorTween`

```dart
// 声明变量
late Animation<Color?> _colorTween;

@override
void initState() {
  // TODO: implement initState
  super.initState();
  _colorTween = ColorTween(
      begin: Colors.white,
      end: Colors.black
  ).animate(controller);
}
```

ColorTween初始化时只需要传入一个开始颜色和一个结束颜色，然后调用它的animate方法，并将之前的`AnimationController`实例传入即可。至此颜色渐变动画创建完成，下面将其应用于组件的color属性

```dart
@override
Widget build(BuildContext context) {
    // TODO: implement build
    return Scaffold(
      appBar: AppBar(title: const Text('TestApp3'),),
      body: Center(
          child: AnimatedBuilder(
            animation: controller,
            builder: (context, child) {
              return Container(
                  width: 200,
                  height: 200,
                  color: _colorTween.value
              );
            }
          ),
      ),
    );
}
```

跟应用二不同的地方是，`AnimatedBuilder`我们没有设置child，而是直接在builder函数中返回了child。最终的效果如下：

<img title="" src="https://cdn.jsdelivr.net/gh/ywxgod/image_source/md-imgs/202206062118762.gif" alt="" data-align="center" width="141">

不过如果我们想再给child增加旋转，缩放，移动等动画呢？这时就可以像应用二中将`AnimatedBuilder`的child赋值，在build函数中调用对于的函数，如下

```dart
@override
Widget build(BuildContext context) {
    // TODO: implement build
    return Scaffold(
      appBar: AppBar(title: const Text('TestApp3'),),
      body: Center(
          child: AnimatedBuilder(
            animation: controller,
            builder: (context, child) {
              return Transform.rotate(
                angle: controller.value,
                child: child,
              );
            },
            child: Container(
                width: 200,
                height: 200,
                color: _colorTween.value
            ),
          ),
      ),
    );
}
```

最终效果：

<img title="" src="https://cdn.jsdelivr.net/gh/ywxgod/image_source/md-imgs/202206062145003.gif" alt="" data-align="center" width="125">

### 应用四：形变动画的实现

通过`BoxDecoration`我们可以设置容器的边框，圆角，形状等，那我们也可以将其改变应用到动画中。这里用到`DecorationTween`，实例化`DecorationTween`类需要一个开始和结束的`BoxDecoration`，看下面的代码

```dart
@override
void initState() {
    // TODO: implement initState
    super.initState();
    _decorationTween = DecorationTween(
        begin: BoxDecoration(
            color: Colors.red,
            border: Border.all(color: Colors.blue, width: 4),
            borderRadius: BorderRadius.circular(15)
        ),
        end: BoxDecoration(
            color: Colors.lightGreen,
            border: Border.all(color: Colors.amber, width: 4),
            borderRadius: BorderRadius.circular(50)
        ),
    ).animate(controller);
}
```

上面动画设置为边框颜色从blue到amber，圆角半径从15到50，下面需要将其应用到组件上

```dart
@override
Widget build(BuildContext context) {
    // TODO: implement build
    return Scaffold(
        appBar: AppBar(
            title: const Text('TestApp4'),
        ),
        body: Center(
            child: DecoratedBoxTransition(
                decoration: _decorationTween,
                child: Container(
                    width: 200,
                    height: 200,
                ),
            ),
        )
    );
}
```

build中用`DecoratedBoxTransition`类来渲染需要产生动画的容器。`DecoratedBoxTransition`的child为产生动画的容器，decoration为前面的`DecorationTween`实例，具体效果如下：

<img title="" src="https://cdn.jsdelivr.net/gh/ywxgod/image_source/md-imgs/202206072157703.gif" alt="" data-align="center" width="133">

### 应用五：与Flow组件的结合

这个应用的本质跟第一个其实一样，利用controller不同的value值，每次重写调用build函数。Flow组件要求两个必须参数，一个是delegate，一个是children。先创建MyFlowDelegate类

```dart
import 'dart:math';

import 'package:flutter/material.dart';

class MyFlowDelegate extends FlowDelegate {

  double value = 0;

  MyFlowDelegate({this.value = 0}):super();

  @override
  void paintChildren(FlowPaintingContext context) {
    // TODO: implement paintChildren
    double x = context.size.width/2;
    double y = context.size.height/2;

    for(var i=0;i<context.childCount;i++) {
      double r = 45;
      double cx = x - (context.getChildSize(i)?.width??0)/2;
      double cy = y - (context.getChildSize(i)?.height??0)/2;
      double radius = 150;
      double angle = (i/context.childCount)*2*pi+value;
      Matrix4 matrix4 = Matrix4.translationValues(
          cx + cos(angle)*radius,
          cy + sin(angle)*radius,
          0
      );
      // matrix4.rotateX(pi/4);
      context.paintChild(i, transform: matrix4);
    }
  }

  @override
  bool shouldRepaint(FlowDelegate oldDelegate) {
    // TODO: implement shouldRepaint
    return true;
  }
}
```

需要重写paintChildren和shouldRepaint两个方法，第一个是在每次build的时候会执行。我们在paintChildren中精确地设置每一个child的位置。接下来在Home类中创建children

```dart
class TestApp5Home extends StatefulWidget {

  List<Widget> _list = [];

  TestApp5Home({Key? key}) : super(key: key) {
    _list = createFlowChildren(10);
  }

  get list => _list;

  List<Widget> createFlowChildren(int len) {
    return List.generate(len, (index) {
      return Container(
        width: 50,
        height: 50,
        color: DataUtil.randomColor2,
      );
    });
  }

  @override
  State<StatefulWidget> createState() {
    return TestApp5State();
  }
}
```

createFlowChildren函数创建了多个宽高为50的Container。下面将其应用到Flow组件

```dart
class TestApp5State extends MyAnimationState {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('TestApp5'),
      ),
      body: Flow(
        delegate: MyFlowDelegate(value: controller.value),
        children: (widget as TestApp5Home).list,
      ),
    );
  }
}
```

MyAnimationState类的实现跟前面是一样的，这里我们只用到了controller的value值。最终效果：

<img title="" src="https://cdn.jsdelivr.net/gh/ywxgod/image_source/md-imgs/202206082132428.gif" alt="" data-align="center" width="135">

### 总结

- **AnimationController可以有两种应用方式：**
1. 通过listener方法监听value的每次改变，然后在监听函数中调用setState促使build方法的每次调用，而每次在build中读取value值，且将value值设置到各个组件的属性中，如fontSize，scale，color等等。

2. 不用每次调用setState促使多次build调用，而是利用AnimatedBuilder或者DecoratedBoxTransition与其结合自动处理子组件的动画效果。
- **创建动画的必要步骤：**
1. 创建State子类，同时通过with关键字组合SingleTickerProviderStateMixin类。

2. 在initState方法中实例化AnimationController，设置初始的lowerbound、upperBound、duration、vsync，如果要自己促使build多次调用读取value值，就可以调用addListener方法，并在回调中调用setState，否则无需。

3. 调用AnimationController的forward，或者repeat方法启动动画，此时AnimationController已经根据初始的lowerbound、upperBound，duration产生了不同的数据，如果调用addListener就可以在回调函数中看到。

4. 重写dispose方法，调用AnimationController的dispose方法销毁。
- **应用1：**

在listener的回调函数中直接调用setState，意思就是在AnimationController的value每次改变的时候重新调用build，然后在build中读取value值，将其赋值给Text的fontSize，所以我们能看到文本的大小改变的动画了。

- **应用2：**

在State的initState方法中实例化AnimationController时，没有监听value的改变，没有调用setState，而是在build方法中通过AnimatedBuilder来处理子组件的动画。将AnimationController实例赋值给AnimatedBuilder的animation属性，同时builder方法通过Transform的scale,translate,rotate来设置动画。

- **应用3：**

与应用2不同的是，这里并没有调用Transform的方法，只是在AnimatedBuilder的builder中将controller的value值赋值给了子组件的color。Transform的scale,translate,rotate分别用于缩放、平移、旋转子组件，这里只是改变子组件颜色，所以无需Transform类。

- **应用4：**

也是没有直接调用setState促使多次build调用，而是利用DecoratedBoxTransition结合各自Tween类来实现子组件的动画。类似的Transition类还有：

```dart
AlignTransition
DefaultTextStyleTransition
PositionedTransition
RelativePositionedTransition
RotationTransition
ScaleTransition
SizeTransition
SlideTransition
FadeTransition
```

关于Tween相关的类还有：

```dart
AlignmentGeometryTween
AlignmentTween
BorderRadiusTween
BorderTween
BoxConstraintsTween
ColorTween
ConstantTween
DecorationTween
EdgeInsetsGeometryTween
EdgeInsetsTween
FractionalOffsetTween
IntTween
MaterialPointArcTween
Matrix4Tween
RectTween
RelativeRectTween
ReverseTween
ShapeBorderTween
SizeTween
StepTween
TextStyleTween
ThemeDataTween
```

- **应用5：**

原理跟应用1一样，只是将其应用到了Flow组件，通过Flow组件我们可以自定义组件的布局。在FlowDelegate的paintChildren方法中结合controller的value值动态更新每个子组件的位置(缩放/角度等)
