最近从书上看到一些微件的用法，然后又自己查了一下文档，这里做个记录。下面要说的这些微件都是相对于Material主题来讲的。

## AppBar

AppBar是Scaffold微件的一个属性，它负责呈现应用的顶部区域，先看AppBar可以有哪些组成部分。

![app_bar](https://cdn.jsdelivr.net/gh/ywxgod/image_source/imgs20210617212948.png)

- leading：会显示在title之前，需要给一个Widget实例，通常我们可以给一个IconButton或BackButton。
- title：也是一个Widget实例，这里不限于Text微件。
- actions：一组右对齐的Widgets，通常情况是一些IconButton。
- flexibleSpace：我们可以把AppBar看成有两层，一层放置前面说的各种leading，title，actions这些，另一层就是flexibleSpace所指定的Widget，且它处于底层，高度跟AppBar一致。

```dart
AppBar(
    title: Text('Cha06'),
    leading: IconButton(
        icon: Icon(Icons.menu),
        onPressed: () {},
    ),
    actions: [
        IconButton(icon: Icon(Icons.search), onPressed: () {}),
        IconButton(icon: Icon(Icons.more_vert), onPressed: () {}),
    ],
    flexibleSpace: Container(
        color: Colors.red,
        alignment: Alignment.bottomLeft,
        child: Image.network(
            'https://flutter.cn/assets/flutter-lockup-1caf6476beed76adec3c477586da54de6b552b2f42108ec5bc68dc63bae2df75.png'
        ),
    ),
),
```

上面的flexibleSpace是一个Container，背景色是红色，还有一张图片，图片下左对其，然后渲染出来的效果

![flexibleSpace](https://cdn.jsdelivr.net/gh/ywxgod/image_source/imgs20210617221835.png)

从图中可以看出，flexibleSpace的高度默认是跟AppBar一样的，其实我们还可以给Container设置一个高度，还有就是flexibleSpace这一层确实是在最下面的。

上面的flexibleSpace会直接跟屏幕的最上方对其的，当手机为刘海屏，水滴屏时显示会出现问题，所以flutter里面有一个SafeArea微件来处理这一类的问题。

### SafeArea

从类名看这是一个安全区域，究竟是哪一块区域，看下面的图

![SafeArea](https://cdn.jsdelivr.net/gh/ywxgod/image_source/imgs20210617223635.png)

>   原图来自：https://clein8.tistory.com/entry/Flutter-Widget-01-SafeArea

所以我们重写flexibleSpace

```dart
flexibleSpace: SafeArea(
    child: Container(
        color: Colors.red,
        height: 300,
        alignment: Alignment.bottomLeft,
        child: Image.network(
            'https://flutter.cn/assets/flutter-lockup-1caf6476beed76adec3c477586da54de6b552b2f42108ec5bc68dc63bae2df75.png'
        ),
    ),
)
```

最终的效果

![flexibleSpaceSafeArea](https://cdn.jsdelivr.net/gh/ywxgod/image_source/imgs20210617224221.png)

AppBar还有一个bottom属性，你需要给bottom赋值一个实现PrefferedSizeWidget接口的微件，PrefferedSizeWidget会根据你给定的宽高设置高度，它不受子微件的布局限制。

## Container

Container有点像html中的div，你可以自定义它的边框，颜色，对齐方式，各种约束等。如果没有子组件，它会占用全部可用区域大小。

```dart
Container(
    height: 100.0,
    decoration: BoxDecoration(
        boxShadow: [
            BoxShadow(
                color: Colors.red,
                blurRadius: 10,
                offset: Offset(0, 10),
            )
        ],
        gradient: LinearGradient(
            begin: Alignment.topCenter,
            end: Alignment.bottomCenter,
            colors: [
                Colors.white,
                Colors.lightGreen.shade500
            ]
        ),
        borderRadius: BorderRadius.only(
            bottomLeft: Radius.circular(100),
            bottomRight: Radius.circular(10),
        )
    ),
    child: Text('hello')
)
```

通过decoration属性来设置各种装饰，具体是通过BoxDecoration来处理的，有boxShadow，gradient，borderRadius等。

BoxDecoration用于决定如何绘制Container容器盒子，盒子可以是圆形的，也可以是矩形。Container在逻辑上分为3层：最底层是背景(背景色设置)，中间层是渐变色(渐变色设置)，上层是图片(背景图片设置)。

## Text

用于渲染文本，可以是多行，也可以是单行。给Text设置style属性，可以设置文本的大小，颜色，字体，下划线，斜体，粗体等。

```dart
child: Center(
    child: Text(
        'Container, what?',
        style: TextStyle(
            fontSize: 24,
            color: Colors.blue,
            decoration: TextDecoration.underline,
            decorationColor: Colors.red,
            decorationStyle: TextDecorationStyle.dashed,
            decorationThickness: 2,
            fontStyle: FontStyle.italic,
            fontWeight: FontWeight.bold
        ),
        maxLines: 1,
        overflow: TextOverflow.ellipsis,
        textAlign: TextAlign.justify,
    )
)
```

maxLines指定显示的行数，特别是当文本换行出现多行了；

overflow指定文本溢出时的效果，跟css的效果类似；

## RichText

用于一起渲染多种不同样式的文本。通过嵌套使用TextSpan，每个TextSpan可以有自己的样式。

```dart
Container(
    child: Center(
        child: RichText(
            text: TextSpan(
                text: 'TextSpan',
                // 处理自身文本样式
                style: TextStyle(
                    color: Colors.black,
                    fontSize: 24,
                    fontWeight: FontWeight.w500
                ),
                children: [
                    // 每一个子TextSpan的样式单独设置
                    TextSpan(
                        text: 'for',
                        style: TextStyle(
                            fontSize: 12
                        )
                    ),
                    TextSpan(
                        text: 'mobile',
                        style: TextStyle(
                            color: Colors.amber,
                            fontSize: 36
                        )
                    )
                ]
            ),
        ),
    ),
)
```

## Column与Row

有一点要记录一下，那就是可以通过将其包装到SingleChildScrollView中来实现滚动条功能。SingleChildScrollView默认是纵向滚动的，如果你想让Row横向滚动，可以设置scrollDirection属性值为：Axis.horizontal

还有关于对其，分别可以设置交叉轴的对其属性：crossAxisAlignment和主轴对其属性：mainAxisAlignment。

Column与Row中的子组件不会自动换行，但可以用SingleChildScrollView包装后使其滚动，如果我们要让其换行呢，这时可以用Wrap组件。

## Wrap

Wrap组件有一个children属性，可以使其子组件按照指定方向换行排列。通过direction指定时按照水平方向还是垂直方向排列。跟Column/Row一样，它也有主轴和交叉轴的概念。主轴就是direction指定的方向。

alignment与runAlignment分别设置主轴和交叉轴上的对其方式；spacing与runSpacing分别对应主轴和交叉轴上的子组件间隔；

```dart
@override
Widget build(BuildContext context) {
    // TODO: implement build
    List items = list.map((e){
        return Container(
            width: 80,
            color: Colors.lightBlue,
            padding: EdgeInsets.all(5),
            child: Text(
                'Item $e',
                style: TextStyle(
                    backgroundColor: Colors.white,
                    color: Colors.black,
                ),
            ),
        );
    }).toList();
    return Wrap(
        alignment: WrapAlignment.start,
        spacing: 5,
        runSpacing: 5,
        crossAxisAlignment: WrapCrossAlignment.center,
        children: items,

    );
}
```

## Divider

用于创建水平线，thickness可以定义线条粗细，color可以定义线条颜色

## BottomAppBar

用于Scoffold组件的bottomNavigationBar属性，可以在应用的底部添加一个导航区域。

```dart
BottomAppBar(
    color: Colors.amber,
    child: Row(
        mainAxisAlignment: MainAxisAlignment.spaceEvenly,
        children: [
            Icon(Icons.pause),
            Icon(Icons.stop),
            Icon(Icons.access_time),
            Padding(padding: EdgeInsets.all(32))
        ],
    ),
)
```

## Button

我们有各种按钮，FloatingActionButton、FlatButton、IconButton、RaisedButton、PopupMenuButton、ButtonBar。

### FloatingActionButton

一般用于Scoffold的floatingActionButton属性，我们可以通过设置FloatActionButton的child来设置按钮的内容，比如：Icon(Icons.add)，还可以通过设置backgroundColor来设置按钮的背景色，当然还有onPress属性设置按钮点击时的动作。

Scoffold中的floatingActionButton默认位置在屏幕的右下角，我们可以通过设置floatingActionButtonLocation属性来改变其位置。

### FlatButton/RaisedButton

扁平按钮，没有边框没有阴影效果，有一个child属性可以放置文本组件或者图标。

```dart
Container(
    height: 30,
    child: Row(
        mainAxisAlignment: MainAxisAlignment.spaceBetween,
        children: [
            FlatButton(
                onPressed: (){},
                child: Text('FlatButton'),
            ),
            FlatButton(
                onPressed: (){},
                child: Icon(Icons.search),
                color: Colors.lightBlue,
                textColor: Colors.white,
            )
        ],
    )
)
```

RaisedButton跟FlatButton不同的地方就是有一个默认的较深的背景色，当用户点击不放开时背景会加深。

### IconButton

```dart
IconButton(icon: Icon(Icons.flight), onPressed: (){}),
IconButton(
    icon: Icon(Icons.map),
    onPressed: (){},
    iconSize: 42,
    color: Colors.lightGreen,
    tooltip: 'Map',
)
```

### PopupMenuButton

用于显示菜单项列表。经常用于放置在AppBar的右上角，给用户选择不同的菜单项，有三点要注意

#### itemBuilder

这是必填项，需要给一个函数，参数是context，我们需要在此函数中返回一个List<PopupMenuItem>，即元素是PopupMenuItem的列表。PopupMenuItem决定了菜单每一项的渲染内容和数据，如

```dart
final menuList = todoItems.map((i){
    return PopupMenuItem(
        value: i,
        child: Row(
            children: [
                Icon(i.icon),
                Padding(padding: EdgeInsets.all(8)),
                Text(i.title)
            ],
        ),
    );
}).toList();
```

上面的代码中，我们给每一项添加了三个组件：图标，内边距，文本。需要注意的是value属性，一般是我们定义的每一菜单项对应的数据模型，下面的onSelected回调函数中传出来的值就是这个value。

#### icon与child

icon与child属性二者取其一，就是说不要两个都赋值。

```dart
icon: Icon(Icons.view_list),
// child: Text('P'),
```

#### onSelected

需要给一个回调函数，函数的参数是用户当前选中的数据，不管此菜单项之前是否选中，只用用户点击都会触发此函数执行。（跟Change事件不一样）

```dart
onSelected: (item) {
    print(item.title);
},
```

#### initialValue

初始化时选中的项数据

#### 创建PopupMenuButton的一般步骤

**1. 准备菜单项对应的数据**

```dart
final menuData = [
  {'title': 'Fast Food', 'icon': Icons.fastfood},
  {'title': 'Remind Me', 'icon': Icons.add_alarm},
  {'title': 'Flight', 'icon': Icons.flight},
  {'title': 'Music', 'icon': Icons.audiotrack},
];
```

**2. 定义数据模型类**

```dart
import 'package:flutter/material.dart';

class TodoMenuItem {
  final String title;
  final IconData icon;

  TodoMenuItem(this.title, this.icon);
}
```

**3.实现itemBuilder函数(代码见上文)**

**4.创建PopupMenuButton实例**

```dart
Widget build(BuildContext context) {
    // TODO: implement build
    final todoItems = menuData.map((i)=>TodoMenuItem(i['title'], i['icon'])).toList();
    final menuList = todoItems.map((i){
        return PopupMenuItem(
            value: i,
            child: Row(
                children: [
                    Icon(i.icon),
                    Padding(padding: EdgeInsets.all(8)),
                    Text(i.title)
                ],
            ),
        );
    }).toList();
    return PopupMenuButton(
        // 默认选中最后一个菜单项
        initialValue: todoItems[3],
        icon: Icon(Icons.view_list),
        // child: Text('P'),
        onSelected: (item) {
            print(item.title);
        },
        itemBuilder: (context) {
            return menuList;
        },
    );
}
```

最后有一个注意的地方，如果你的PopupMenuButton是自己封装到了一个类中，且你想把它用在AppBar中，那么可能你的类要实现PrefferedSizeWidget接口，而此接口要求实现prefferedSize存取属性，一个可能的例子。

```dart
@override
// TODO: implement preferredSize
Size get preferredSize => Size.fromHeight(75);
```

### ButtonBar

一组水平放置的组件，可以通过alignment属性设置其主轴的排列方式，通过children属性放置子组件，当子组件太多一行放不下时，会将布局改为垂直放置。children并没有规定是按钮列表，而是Widget列表，所以虽然叫ButtonBar，其实我们可以在children中不放置按钮，比如Text，只是不能点击。

```dart
ButtonBar(
    alignment: MainAxisAlignment.spaceEvenly,
    buttonPadding: EdgeInsets.all(10),
    children: [
        IconButton(icon: Icon(Icons.map), onPressed: (){}),
        IconButton(icon: Icon(Icons.airport_shuttle), onPressed: (){}),
        IconButton(icon: Icon(Icons.brush), onPressed: (){}, highlightColor: Colors.purple,),
        // IconButton(icon: Icon(Icons.print), onPressed: (){}),
        // IconButton(icon: Icon(Icons.offline_bolt_sharp), onPressed: (){}),
        // IconButton(icon: Icon(Icons.wallet_giftcard), onPressed: (){}),
        Text('文本')
    ],
),
```

所以ButtonBar可以看成是一个动态的Column或Row？当子组件垂直排列时就是一个Column，当子组件水平排列时就是一个Row？
