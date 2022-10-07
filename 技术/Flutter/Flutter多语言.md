### 多语言准备

先将fluter_localizations包添加到pubspec.yaml文件中

```yaml
dependencies:
  flutter:
    sdk: flutter
  flutter_localizations: # Add this line
    sdk: flutter         # Add this line
```

然后，安装flutter_localizations依赖包

```shell
flutter pub get
```

现在可以在MaterialApp实例中使用，如下

```dart
import 'package:flutter_localizations/flutter_localizations.dart';


@override
Widget build(BuildContext context) {
    return MaterialApp(
        title: 'Flutter Demo',
        // 开启组件本地化
        localizationsDelegates: const [
            GlobalMaterialLocalizations.delegate,
            GlobalWidgetsLocalizations.delegate,
            GlobalCupertinoLocalizations.delegate
        ],
        // 支持的语言
        supportedLocales: const [
            Locale('en', 'US'),
            Locale('zh', 'CN')
        ],
        // 指定当前语言 - 语言代码，国家代码
        locale: const Locale('zh', 'CN'),
        theme: ThemeData(
            primarySwatch: Colors.blue,
        ),
        home: const MyHomePage(title: 'Flutter Demo Home Page'),
    );
}
```

开启中文显示后的日期选择组件

<img src="file:///C:/Users/wyin/AppData/Roaming/marktext/images/2022-09-13-08-24-35-image.png" title="" alt="" data-align="center">

多语言切换

先创建如下两个json文件，如增加语言需要再创建对应的json文件

```shell
+-- assets
    +-- locale
        +-- en.json
        +-- cn.json
```

填充cn.json内容，如下

```json
{
  "selectDatePickerButton": "选择日期"
}
```

填充en.json

```json
{
  "selectDatePickerButton": "Select a Date"
}
```

然后修改pubspec.yaml，使其包含assets目录

```yaml
  assets:
    - assets/locale/
```
