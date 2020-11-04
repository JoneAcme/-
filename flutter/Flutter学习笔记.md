# Flutter学习笔记

## widget

- 有状态：StatefulWidget 

- 无状态：StatelessWidget

### 有状态与无状态的区别

无状态和有状态的 Widget 本质上是行为一致的。它们每一帧都会重建，不同之处在于 `StatefulWidget` 有一个跨帧存储和恢复状态数据的 `State` 对象。

如果一个 Widget 会变化（例如由于用户交互），它是有状态的。然而，如果一个 Widget 响应变化，它的父 Widget 只要本身不响应变化，就依然是无状态的。

### widget 和 android中View 的区别：

1. widget 有着不一样的生命周期：它们是不可变的，一旦需要变化则生命周期终止。任何时候 widget 或它们的状态变化时， Flutter 框架都会创建一个新的 widget 树的实例
2. Android View 只会绘制一次，除非调用 `invalidate` 才会重绘。





### 简单使用：

#### 基础Text 展示

Text` Widget 是一个普通的 `StatelessWidget

```flutter
Text(
  'I like Flutter!',
  style: TextStyle(fontWeight: FontWeight.bold),
);
```

#### 点击时更新文本

如果想要在点击一个按钮时候，动态改变Text的显示，那么需要将Text 嵌入一个StatefulWidget 中，点击按钮时更新它。

```flutter
import 'package:flutter/material.dart';

void main() {
  runApp(SampleApp());
}

class SampleApp extends StatelessWidget {
  // This widget is the root of your application.
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Sample App',
      theme: ThemeData(
        primarySwatch: Colors.blue,
      ),
      home: SampleAppPage(),
    );
  }
}
// Text 的上一级，StatefulWidget
class SampleAppPage extends StatefulWidget {
  SampleAppPage({Key key}) : super(key: key);

  @override
  _SampleAppPageState createState() => _SampleAppPageState();
}

class _SampleAppPageState extends State<SampleAppPage> {
  // Default placeholder text
  String textToShow = "I Like Flutter";
  
  void _updateText() {
  // 点击事件中，更新State、变量，刷新Test
    setState(() {
      // update the text
      textToShow = "Flutter is Awesome!";
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text("Sample App"),
      ),
      // Text对应的文本为变量
      body: Center(child: Text(textToShow)),
      floatingActionButton: FloatingActionButton(
      // 此处，指定点击事件
        onPressed: _updateText,
        tooltip: 'Update Text',
        child: Icon(Icons.update),
      ),
    );
  }
}
```

#### 添加padding

```flutter
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text("Sample App"),
      ),
      body: Center(
        child: MaterialButton(
          onPressed: () {},
          child: Text('Hello'),
          padding: EdgeInsets.only(left: 10.0, right: 10.0),
        ),
      ),
    );
  }
```

#### 切换Child（add/remove View）

```flutter
import 'package:flutter/material.dart';

void main() {
  runApp(SampleApp());
}

class SampleApp extends StatelessWidget {
  // This widget is the root of your application.
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Sample App',
      theme: ThemeData(
        primarySwatch: Colors.blue,
      ),
      home: SampleAppPage(),
    );
  }
}

class SampleAppPage extends StatefulWidget {
  SampleAppPage({Key key}) : super(key: key);

  @override
  _SampleAppPageState createState() => _SampleAppPageState();
}

class _SampleAppPageState extends State<SampleAppPage> {
  // Default value for toggle
  bool toggle = true;
  void _toggle() {
    setState(() {
      toggle = !toggle;
    });
  }

// 根据标识确定返回哪一个Widget
  _getToggleChild() {
    if (toggle) {
      return Text('Toggle One');
    } else {
      return MaterialButton(onPressed: () {}, child: Text('Toggle Two'));
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text("Sample App"),
      ),
      body: Center(
        child: _getToggleChild(),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: _toggle,
        tooltip: 'Update Text',
        child: Icon(Icons.update),
      ),
    );
  }
}
```

#### Widget 动画

Flutter 通过 `Animation<double>` 的子类 `AnimationController` 来暂停、播放、停止以及逆向播放动画。它需要一个 `Ticker` 在垂直同步 (vsync) 的时候发出信号，并且在运行的时候创建一个介于 0 和 1 之间的线性插值。然后你就可以创建一个或多个 `Animation`，并将它们绑定到控制器上。

例如，你可以使用 `CurvedAnimation` 来实现一个曲线插值的动画。在这种情况下，控制器决定了动画进度，`CurvedAnimation` 计算用于替换控制器默认线性动画的曲线值。和 Widget 一样，Flutter 中的动画效果也可以组合使用。

在构建 Widget 树的时候，你需要将 `Animation` 对象赋值给某个 Widget 的动画属性，例如 `FadeTransition` 的不透明度属性，并让控制器开始动画。

```flutter
import 'package:flutter/material.dart';

void main() {
  runApp(FadeAppTest());
}

class FadeAppTest extends StatelessWidget {
  // This widget is the root of your application.
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Fade Demo',
      theme: ThemeData(
        primarySwatch: Colors.blue,
      ),
      home: MyFadeTest(title: 'Fade Demo'),
    );
  }
}

class MyFadeTest extends StatefulWidget {
  MyFadeTest({Key key, this.title}) : super(key: key);
  final String title;

  @override
  _MyFadeTest createState() => _MyFadeTest();
}

class _MyFadeTest extends State<MyFadeTest> with TickerProviderStateMixin {
  AnimationController controller;
  CurvedAnimation curve;

  @override
  void initState() {
    super.initState();
    controller = AnimationController(
      duration: const Duration(milliseconds: 2000),
      vsync: this,
    );
    curve = CurvedAnimation(parent: controller, curve: Curves.easeIn);
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text(widget.title),
      ),
      body: Center(
          child: Container(
              child: FadeTransition(
                  opacity: curve,
                  child: FlutterLogo(
                    size: 100.0,
                  )))),
      floatingActionButton: FloatingActionButton(
        tooltip: 'Fade',
        child: Icon(Icons.brush),
        onPressed: () {
          //添加此处，点击可多次执行，否则只能执行一次
          controller.reset();
          // 执行动画
          controller.forward();
        },
      ),
    );
  }
}
```

#### 使用Canvas画图

CustomPaint 画布

CustomPainter 画笔

下面例子是一个空画布的签名功能：

```flutter
import 'package:flutter/material.dart';

void main() => runApp(MaterialApp(home: DemoApp()));

class DemoApp extends StatelessWidget {
  Widget build(BuildContext context) => Scaffold(body: Signature());
}

class Signature extends StatefulWidget {
  SignatureState createState() => SignatureState();
}

class SignatureState extends State<Signature> {
  List<Offset> _points = <Offset>[];

// 清空画布中的点 即 清空画布
  void _toggle(){
    setState(() {
      _points.clear();
    });
  }
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(// 手势识别外面嵌套一层，用于添加个按钮清空画布
      body: GestureDetector(// 手势识别部分
      onPanUpdate: (DragUpdateDetails details) {
        setState(() {
          // 获取点击点
          RenderBox referenceBox = context.findRenderObject();
          Offset localPosition =
          referenceBox.globalToLocal(details.globalPosition);
          // 将点击点放入list
          _points = List.from(_points)..add(localPosition);
        });
      },
      onPanEnd: (DragEndDetails details) => _points.add(null),
      child: CustomPaint(// 画布
        painter: SignaturePainter(_points),// 画笔
        size: Size.infinite,
      ),
    ),
    // 添加一个按钮，点击清空画布
      floatingActionButton: FloatingActionButton(
        onPressed: _toggle,
        tooltip: 'Update Text',
        child: Icon(Icons.update),
      ),
    );
  }
}

// 画笔
class SignaturePainter extends CustomPainter {
  SignaturePainter(this.points);
  final List<Offset> points;
  void paint(Canvas canvas, Size size) {
    var paint = Paint()
      ..color = Colors.black
      ..strokeCap = StrokeCap.round
      ..strokeWidth = 5.0;
    for (int i = 0; i < points.length - 1; i++) {
      if (points[i] != null && points[i + 1] != null)
        canvas.drawLine(points[i], points[i + 1], paint);
    }
  }

  bool shouldRepaint(SignaturePainter other) => other.points != points;
}
```

#### 自定义Widget

通过 [组合](https://flutter.cn/docs/resources/technical-overview#everythings-a-widget) 更小的 Widget 来创建自定义 Widget（而不是继承它们）。

```flutter
import 'package:flutter/material.dart';

void main() {
  runApp(MaterialApp(home: Scaffold(body: CustomButton("hahahahah"),)));
}

class CustomButton extends StatelessWidget {
  final String label;

  CustomButton(this.label);

  @override
  Widget build(BuildContext context) {
    return RaisedButton(onPressed: () {}, child: Text(label));
  }
}
```

#### Intents

在 Flutter 中你需要使用 `Navigator` 和 `Route` 在同一个 `Activity` 内的不同界面间进行跳转。

`Route` 是应用内屏幕和页面的抽象，`Navigator` 是管理路径 route 的工具。一个 route 对象大致对应于一个 `Activity`，但是它的含义是不一样的。Navigator 可以通过对 route 进行压栈和弹栈操作实现页面的跳转。Navigator 的工作原理和栈相似，你可以将想要跳转到的 route 压栈 (`push()`)，想要返回的时候将 route 弹栈 (`pop()`)。

在 Android 中，在应用的 `AndroidManifest.xml` 文件中声明 Activity。

在 Flutter 中，你有多种不同的方式在页面间导航：

- 定义一个 route 名字的 `Map`。(MaterialApp)
- 直接导航到一个 route。(WidgetApp)

```flutter
 void main() {
  runApp(MaterialApp(
    home: MyAppHome(), // becomes the route named '/'
    routes: <String, WidgetBuilder> {
      '/a': (BuildContext context) => MyPage(title: 'page A'),
      '/b': (BuildContext context) => MyPage(title: 'page B'),
      '/c': (BuildContext context) => MyPage(title: 'page C'),
    },
  ));
}
```

```flutter
Navigator.of(context).pushNamed('/b');
```

#### StartActivityForResult

```flutter
Map coordinates = await Navigator.of(context).pushNamed('/location');
```
可以弹栈 (pop) 并返回结果:
```flutter
Navigator.of(context).pop({"lat":43.821757,"long":-79.226392});
```

#### 异步

Dart 有一个单线程执行的模型，同时也支持 `Isolate` （在另一个线程运行 Dart 代码的方法），它是一个事件循环和异步编程方式。除非你创建一个 `Isolate`，否则你的 Dart 代码会运行在主 UI 线程，并被一个事件循环所驱动。Flutter 的事件循环对应于 Android 里的主 `Looper`— 也即绑定到主线程上的 `Looper`。

```flutter
Future<void> loadData() async {
  String dataURL = "https://jsonplaceholder.typicode.com/posts";
  http.Response response = await http.get(dataURL);
  setState(() {
    widgets = json.decode(response.body);
  });
}
```

异步加载数据并将之展示在 `ListView` 内：

```flutter
import 'dart:convert';

import 'package:flutter/material.dart';
import 'package:http/http.dart' as http;

void main() {
  runApp(SampleApp());
}

class SampleApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Sample App',
      theme: ThemeData(
        primarySwatch: Colors.blue,
      ),
      home: SampleAppPage(),
    );
  }
}

class SampleAppPage extends StatefulWidget {
  SampleAppPage({Key key}) : super(key: key);

  @override
  _SampleAppPageState createState() => _SampleAppPageState();
}

class _SampleAppPageState extends State<SampleAppPage> {
  List widgets = [];

  @override
  void initState() {
    super.initState();

    loadData();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text("Sample App"),
      ),
      body: ListView.builder(
        itemCount: widgets.length,
        itemBuilder: (BuildContext context, int position) {
          return getRow(position);
        },
      ),
    );
  }

  Widget getRow(int i) {
    return Padding(
      padding: EdgeInsets.all(10.0),
      child: Text("Row ${widgets[i]["title"]}"),
    );
  }

  Future<void> loadData() async {
    String dataURL = "https://run.mocky.io/v3/b36b221c-8b53-4718-9395-6420631883d0";
    http.Response response = await http.get(dataURL);
    setState(() {
      widgets = json.decode(response.body);
      print(response.body);
    });
  }
}
```

此处需要在项目根目录 pubspec.yaml 文件中 添加http依赖

```flutter
...

dependencies:
  flutter:
    sdk: flutter

  http: ^0.12.0+4
  
...
```

