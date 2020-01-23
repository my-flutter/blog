---
title: Flutter入门进阶之旅（二十）Flutter插件开发
date: 2020-01-23 20:31:49
tags:
---
#### 前言
>鉴于现阶段Flutter技术栈还不是太成熟，在使用Flutter做移动端开发时我们经常需要借助Native平台的力量来补充Flutter在这方面的缺陷，前面两章我们通过学习**把Flutter项目打包成AAR集成到原生平** 跟 **Flutter与原生平台交互**掌握了Flutter与原生平台交互的两种方式，但是有些场景下，我们希望我们Flutter跟原生交互的代码可以`一次开发，多处使用`，类似于库文件一样，可以给其他项目或者其他开发着使用，这就是我们本篇文章要介绍的主题`Flutter插件开发`以及`插件如何引用到项目中`

#### 课程目标
- 学会如何新建Flutter插件，并了解插件项目结构
- 掌握如何把插件引入到现有项目中

#### 1.新建Flutter插件项目
新建Flutter插件项目跟新建Flutter项目的步骤一样，无非是在新建项目的时候选择的工程类型略有不同。
##### 1.1新建项目
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200119093505889.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly94aWVkb25nLmJsb2cuY3Nkbi5uZXQ=,size_16,color_FFFFFF,t_70)

##### 1.2 选择Flutter Plugin
之后跟正常新建Flutter Applicition的操作一样，正常给项目起名字，选择工程路径等一些列的初始化配置一直next到插件项目初始化完毕。之后的操作读者一看便知，也没有什么需要特别注意的地方，我就不逐个贴图了。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200119092932232.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly94aWVkb25nLmJsb2cuY3Nkbi5uZXQ=,size_16,color_FFFFFF,t_70)

##### 1.3 插件项目结构
从下面插件工程项目结构图中我们可以看出，Flutter插件项目跟普通的Flutter项目结构上几乎一样，但是多出了一个example目录，读者打开example目录后，会发现这个example目录下面其实就是一个完整的Flutter项目，没错这个example就是为了方便我们在开发插件方便我们调试开发的功能是否正常可用，没问题的话就可以发布出去或者给其他项目正常使用了。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200119093734184.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly94aWVkb25nLmJsb2cuY3Nkbi5uZXQ=,size_16,color_FFFFFF,t_70)

>插件开发其实用到的知识点就是通过利用我们上节课中讲的Flutter跟原生平台交互的方式来完成的，让Flutter借助Native的功能来完成某种操作，插件化只不过是把调用平台操作的代码模块化，便于后期其他项目或者别人引入，让代码`一次开发，多处使用`，由于涉及到的知识点在上一篇文章中我们都已经讲过了，所以这里就不在细讲插件里的功能代码实现逻辑了，下面我们来简单分析一下这次课程中用到的用Flutter插件工程。

先看效果图：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200120105208886.gif)

在上图的插件工程中我们实现了，获取系统版本号跟一个简单的计算器的功能。下面看一下在插件工程中具体配置。

在插件工程的android端的业务实现逻辑：
```java

class FlutterCalcPlugin : MethodCallHandler {
    companion object {
        @JvmStatic
        fun registerWith(registrar: Registrar) {
            val channel = MethodChannel(registrar.messenger(), "flutter_calc_plugin")
            channel.setMethodCallHandler(FlutterCalcPlugin())
        }
    }

    override fun onMethodCall(call: MethodCall, result: Result) {
        if (call.method == "getPlatformVersion") {
            result.success("Android ${android.os.Build.VERSION.RELEASE}")
        } else if (call.method == "getResult") {
            var a = call.argument<Int>("a")
            var b = call.argument<Int>("b")
            result.success((a!! + b!!).toString())
        } else {
            result.notImplemented()
        }
    }
}
```

由于插件开发完成之后是要在flutter端使用的，换句话说是要给dart文件引用的，所以下面的dart文件中定义的方法声明才是我们开发的插件对调用者提供的方法。如下我们定义了
`getplatformVersion`：获取系统版本号
`getResult(int a, int b)`：计算两个数的和

而两个方法又通过methodChannel与平台交互，借助native端来完成某些具体逻辑，然后把执行完成的结果返回给调用方。插件定义完成并且成功导入到我们的项目中之后，我们就可以在项目中导入相关类以及方法引用，正常去使用我们自己开发的组件了。

插件工程的flutter端代码：
```java

class FlutterCalcPlugin {
  static const MethodChannel _channel =
      const MethodChannel('flutter_calc_plugin');

  static Future<String> getplatformVersion async {
    final String version = await _channel.invokeMethod('getPlatformVersion');
    return version;
  }

  /**
   *计算两个数的和
   */
  static Future<String>  getResult(int a, int b) async {
    Map<String, dynamic> map = {"a": a, "b": b};
    String result = await _channel.invokeMethod("getResult", map);
    print(result+"----------aa--");
    return result;
  }
}

```

这里涉及到跟平台交互的部分我没有具体展开讲解，因为在上一篇文章中我们已经讲过相关的知识点了，如果读者在这里不太明白平台交互相关的逻辑，建议先去读一下上一篇文章**Flutter入门进阶之旅（十九）Flutter与原生平台交互**
[上述插件完整代码地址：https://github.com/xiedong11/flutter_calc_plugin.git](https://github.com/xiedong11/flutter_calc_plugin.git)

#### 2.插件引入到现有项目中
把我们开发完成的插件项目导入到现有项目中使用，我们可以`通过github仓库引入`，或者`本地引入`，当然也可以把开发完成的插件工程上传到flutter的[dart packages](https://pub.flutter-io.cn)上然后通过版本号用pubspec.ymal文件引入，上传dart packages的配置相对麻烦，限于篇幅，这里我就先只介绍前两种方式，读者如果对上传dart packages感兴趣的话可以私下里找我交流或者我会在后续的博客单独整理出一篇博文来具体讲解。

##### 2.1 本地引入
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200120135205200.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly94aWVkb25nLmJsb2cuY3Nkbi5uZXQ=,size_16,color_FFFFFF,t_70)

如图，我把插件工程放在项目跟目录下的`plugin`文件下，插件项目名我们自己可以自己随便定义，我这里把它定义成`flutter_calc_plugin`，那在我们要引入插件的项目中yaml文件里我们通过插件名，加路径的方式把插件导入之后就可以正常使用插件里的功能了。

```java
#本地插件引入
  flutter_calc_plugin:
    path: plugin/flutter_calc_plugin
```

##### 2.2通过github仓库地址引入
 通过github仓库地址引入相对简单一些，就不用把插件拷贝到本地了，只需要在工程的yaml文件中正确配置插件的地址就可以导入了，两种方式配置完成之后都别忘了执行`flutter packages get`让工程依赖同步一下。

```java
  #从github上引入插件依赖
  flutter_calc_plugin:
    git:
     url:
      https://github.com/xiedong11/flutter_calc_plugin.git

```
>这里顺便说一下小细节吧，由于yaml文件对缩进格式要求特别严格，读者在配置插件引用或者其他第三方的库时，一定要注意缩进。

还有就是具体采用上述两种插件的哪一种方式，这个没有固定的答案完全看你个人喜好跟插件的需求吧，举个例子，如果你的插件开发出来几乎不需要修改，那笔者建议通过github或者上传到dart packages的方式引用，这样不仅让你的工程结构更新清晰而且项目也好管理，但是如果先阶段你开发的插件还不太成熟，或者经常需要改动的话，建议使用本地的方式导入，这样修改后调试代码也方便，而且也省去了频繁上传插件的版本到dart packages或者github上去。

还有一个场景就是，比如使用的插件不是自己开发的，而是从github或者dart packages上找的别人开发好的，但恰恰他开发的插件不能完全满足你的业务需求，或者你需要在此插件的基础上重新定制UI或者补充逻辑，那这个时候你也可以把别人开发好的插件下载到本地，然后通过本地的方式引入到你的项目中再去针对你的业务去修改这个插件，直到修改到你满意为止，然后再通过本地方式导入到项目中。

在文章的最后，贴上一下，上述gif图上示例的具体代码实现供读者参考：
```java
import 'package:flutter/material.dart';
import 'dart:async';

import 'package:flutter/services.dart';
import 'package:flutter_calc_plugin/flutter_calc_plugin.dart';

void main() => runApp(MyApp());

class MyApp extends StatefulWidget {
  @override
  _MyAppState createState() => _MyAppState();
}

class _MyAppState extends State<MyApp> {
  String _platformVersion = 'Unknown';
  String addResult = '';
  TextEditingController _addNumber1Controller,_addNumber2Controller;

  @override
  void initState() {
    super.initState();
    _addNumber1Controller = TextEditingController();
    _addNumber2Controller = TextEditingController();
  }

  Future<void> getAddResult() async {
    int addNumber1= int.parse(_addNumber1Controller.value.text);
    int addNumber2=int.parse(_addNumber2Controller.value.text);

    String result = '';
    try {
      result = await FlutterCalcPlugin.getResult(addNumber2, addNumber1);
    } on PlatformException {
      result = '未知错误';
    }
    setState(() {
      addResult = result;
    });
  }

  Future<void> initPlatformState() async {
    String platformVersion;

    try {
      platformVersion = await FlutterCalcPlugin.platformVersion;
    } on PlatformException {
      platformVersion = 'Failed to get platform version.';
    }

    if (!mounted) return;

    setState(() {
      _platformVersion = platformVersion;
    });
  }

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Scaffold(
        appBar: AppBar(
          title: const Text('插件示例'),
        ),
        body: Center(
            child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: <Widget>[
            MaterialButton(
              color: Colors.amber,
              child: Text("获取系统版本"),
              onPressed: () {
                initPlatformState();
              },
            ),
            Text('当前系统版本 : $_platformVersion\n'),
            SizedBox(height: 30),
            Text("加法计算器"),
            Row(
              mainAxisAlignment: MainAxisAlignment.center,

              children: <Widget>[
                SizedBox(
                  width: 80,
                  child: TextField(
                    controller: _addNumber1Controller,
                    keyboardType: TextInputType.number,
                  ),
                ),
                Text("  +  ",style: TextStyle(fontSize: 26),),
                SizedBox(
                  width: 80,
                  child: TextField(
                    controller: _addNumber2Controller,
                    keyboardType: TextInputType.number,
                  ),
                ),
                Text("  = ",style: TextStyle(fontSize: 26),),
              ],
            ),
            SizedBox(height: 30),
            MaterialButton(
              color: Colors.amber,
              child: Text("结果等于"),
              onPressed: () {
                getAddResult();
              },
            ),
            Text(addResult),
          ],
        )),
      ),
    );
  }
}
```
最后本章节以及专栏的所有完整代码如下，读者如果还不太明白，可以下载代码自己跑一篇项目，慢慢的琢磨下具体的实现细节：
[ 专栏代码仓库：https://github.com/xiedong11/flutter_app.git](https://github.com/xiedong11/flutter_app.git)

