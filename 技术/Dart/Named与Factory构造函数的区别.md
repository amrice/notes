1 可访问的成员不同

named construtor可以访问this关键字，即能访问所有实例成员；

factory constructor是静态的，所以无法访问this，无法访问实例成员，只能访问静态成员。

```dart{12}
class Test {
  String name = '';
  Test.named(this.name);

  factory Test() {
    this.name = 'test';
  }
}
```

上面的代码6行会报下面的错误：

```shell
Don't access members with `this` unless avoiding shadowing.
Invalid reference to 'this' expression.
```

```dart
class Test {
  String _name;
  static Test? instance;
  Test.named({String name=''}):_name=name;

  factory Test() {
    instance ??= Test.named(name: '');
    return instance!;
  }
}
```

改成上面的就可以了，定义一个静态成员。

2 return语句

named constructor像普通的构造函数一样，是不需要return语句的。

factory constructor必须显式的添加一个return语句

如果注释掉上面的return语句，你会看到下面的报错：

```shell
The body might complete normally, causing 'null' to be returned, but the return type, 'Test', is a potentially non-nullable type.
```

3 返回的实例类型

named constructor只能返回当前类的实例

factory constructor可以返回当前类的实例，也可以返回其子类

4 返回的实例是否是同一个

named constructor永远返回的是一个新的实例

factory constructor可以返回一个新实例，也可以返回一个缓存好的实例，即跟前面相同的实例
