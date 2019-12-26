---
title: Flutter入门进阶之旅（十二）Flutter 数据存储
date: 2019-01-26 14:39:41
tags:
---
### 前言
>之前的章节我们基本上把Flutter中基础部分的东西都做了简单的讲解，通过前面章节的循序学习读者也基本能完成一些简单的UI绘制并能利用Flutter处理一些简单的用户交互，读者可能也留意到，我们之前的章节中所学习到的内容并没有涉及到数据存储方面的操作，或者说，我们到现在为止并不知道在Flutter中数据应该怎么存，存在哪。本篇博文中笔者将会为大家解决这一疑惑。

### 关于Flutter中的数据存储
相信做过原生Android开发的读者对数据存储并不陌生，在原生Android中我们会把一些轻量级的数据（如用户信息、APP配置信息等）写入`SharedPreferences`做存储，把需要长期存储的数据写入本地文件或者`Sqlite3`，当然Flutter中也同样用一套完整的本地数据存储体系，下面我们就一直来了解下上述提到的这3中本地存储方式在Flutter中使用。

### 1.SharedPreferences
在Flutter中本身并没有内置SharedPreferences存储，但是官方给我们提供了第三方的组件来实现这一存储方式。我们可以通过`pubspec.yaml`文件引入，关于`pubspec.yaml`的使用我们在[Flutter入门进阶之旅（五）Image Widget](https://www.jianshu.com/p/f20877589072),这一章节提到过，只不过在Image使用中我们引入的是`assets`文件依赖。

如下我们在dependencies节点下引入SharedPreferences的依赖，`读者在pubspec.yaml引入依赖时一定要注意代码缩进格式，否则在在执行flutter packages get时很可能会报错`。
```java

dependencies:
  flutter:
    sdk: flutter
    
  # 添加sharedPreference依赖
  shared_preferences: ^0.5.0

dev_dependencies:
  flutter_test:
    sdk: flutter
    
  # 引入本地资源图片
  assets:
     - images/a.png
     - images/aaa.png
```
然后命令行执行`flutter packages get`把远程依赖同步到本地，在此笔者写文章的时候sharedPreference的最新版本是0.5.0，读者可自行去[https://pub.dartlang.org/flutter](https://pub.dartlang.org/flutter)上获取最新版本，同时也可以在上面找到其他需要引入的资源依赖包。


**笔者的话**
>啰里啰嗦的准备工作总算是讲完了，主要是今天的课程涉及到了包依赖管理，可能对于有些初学者有点懵，所以我就借助sharedPreference把依赖引入废话扯了一大通，如果读者已经掌握了上述操作，可跳过准备工作直接到下面的部分。

继续上面的内容，我们先来体验一下sharedPreference，贴个图大家放松一下。
![sharedPreference](http://upload-images.jianshu.io/upload_images/7274320-6184e41c30b34d65.gif?imageMogr2/auto-orient/strip)


从上图中我们看到我们使用`sharedPreference`做了简单存储跟获取的操作，其实`sharedPreference`好像也就这么点左右，不是存就是取。读者在自行操作时一定不要忘记导入`sharedPreference`的包
```java
import 'package:shared_preferences/shared_preferences.dart';
```

#### 存数据
跟原生Android一样，Flutter中操作sp也是通过key-value的方式存取数据
```java
/**
   * 利用SharedPreferences存储数据
   */
  Future saveString() async {
    SharedPreferences sharedPreferences = await SharedPreferences.getInstance();
    sharedPreferences.setString(
        STORAGE_KEY, _textFieldController.value.text.toString());
  }
```
`SharedPreferences`中为我们提供了String、bool、Double、Int、StringList数据类型的存取。
![SharedPreferences](http://upload-images.jianshu.io/upload_images/7274320-4caafb33446cace4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
#### 取数据
```java
/**
   * 获取存在SharedPreferences中的数据
   */
  Future getString() async {
    SharedPreferences sharedPreferences = await SharedPreferences.getInstance();
    setState(() {
      _storageString = sharedPreferences.get(STORAGE_KEY);
    });
  }
```
上述操作逻辑中我们通过`_textFieldController`获取在`TextField`中的值，在按下存储按钮的同时我们把数据写入sp中，当按下获取值的时候我们通过`setState`把从sp中获取的值同步更新到下面的Text上显示。

完整代码：
```java
import 'package:flutter/material.dart';
import 'package:shared_preferences/shared_preferences.dart';

class StoragePage extends StatefulWidget {
  @override
  State<StatefulWidget> createState() => StorageState();
}

class StorageState extends State {
  var _textFieldController = new TextEditingController();
  var _storageString = '';
  final STORAGE_KEY = 'storage_key';

  /**
   * 利用SharedPreferences存储数据
   */
  Future saveString() async {
    SharedPreferences sharedPreferences = await SharedPreferences.getInstance();
    sharedPreferences.setString(
        STORAGE_KEY, _textFieldController.value.text.toString());
  }

  /**
   * 获取存在SharedPreferences中的数据
   */
  Future getString() async {
    SharedPreferences sharedPreferences = await SharedPreferences.getInstance();
    setState(() {
      _storageString = sharedPreferences.get(STORAGE_KEY);
    });
  }

  @override
  Widget build(BuildContext context) {
    return new Scaffold(
      appBar: new AppBar(
        title: new Text('数据存储'),
      ),
      body: new Column(
        children: <Widget>[
          Text("shared_preferences存储", textAlign: TextAlign.center),
          TextField(
            controller: _textFieldController,
          ),
          MaterialButton(
            onPressed: saveString,
            child: new Text("存储"),
            color: Colors.pink,
          ),
          MaterialButton(
            onPressed: getString,
            child: new Text("获取"),
            color: Colors.lightGreen,
          ),
          Text('shared_preferences存储的值为  $_storageString'),


        ],
      ),
    );
  }
}
```

### 2.文件存储
虽然我们今天内容是Flutter的数据存储，尴尬的是Flutter本身都没有内置提到的这三种存储方式，不过好在官方给我们提供了三方的支持库，不知道后续的Flutter版本中会不会对此做改进。操作文件同样我们也需要像SharedPreferences一样，需要在`pubspec.yaml`引入。`在 Flutter 里实现文件读写，需要使用 path_provider 和 dart 的 io 模块。path_provider 负责查找 iOS/Android 的目录文件，IO 模块负责对文件进行读写`
```java
  # 添加文件依赖
  path_provider: ^0.5.0
  ```
笔者在此引入的最新版本是0.5.0，读者可自行去[https://pub.dartlang.org/flutter](https://pub.dartlang.org/flutter)上获取最新版本。

由于整个操作演示逻辑跟SharedPreferences一样，我就不详细讲解文件存储中关于存取数据的具体操作了，稍微我贴上源代码，读者自行查阅代码对比即可，关于文件存储的三个获取文件路径的方法我这里说明一下，做过原生Android开发的读者可能对此不陌生，但是ios或者初学者可能并不了解这个概念，所以我想提出来说明一下。

**在path_provider中有三个获取文件路径的方法：**
 - `getTemporaryDirectory()//获取应用缓存目录，等同IOS的NSTemporaryDirectory()和Android的getCacheDir() 方法`
- `getApplicationDocumentsDirectory()获取应用文件目录类似于Ios的NSDocumentDirectory和Android上的 AppData目录`
- `getExternalStorageDirectory()//这个是存储卡，仅仅在Android平台可以使用`

**来看下操作文件的效果图**：
![文件存储](http://upload-images.jianshu.io/upload_images/7274320-63e5a3d0d4d806fb.gif?imageMogr2/auto-orient/strip)

借用了SharedPreferences存储的逻辑，只是把存储的代码放在了`file.text`中，代码里有详尽的注释，我就不多做解释说明了，读者可自行尝试对比跟`SharedPreferences`的差别

**样例代码**
```java
import 'package:flutter/material.dart';
import 'package:path_provider/path_provider.dart';
import 'dart:io';

class StoragePage extends StatefulWidget {
  @override
  State<StatefulWidget> createState() => StorageState();
}

class StorageState extends State {
  var _textFieldController = new TextEditingController();
  var _storageString = '';

  /**
   * 利用文件存储数据
   */
  saveString() async {
    final file = await getFile('file.text');
    //写入字符串
    file.writeAsString(_textFieldController.value.text.toString());
  }

  /**
   * 获取存在文件中的数据
   */
  Future getString() async {
    final file = await getFile('file.text');
    var filePath  = file.path;
    setState(() {
      file.readAsString().then((String value) {
        _storageString = value +'\n文件存储路径：'+filePath;
      });
    });
  }

  /**
   * 初始化文件路径
   */
  Future<File> getFile(String fileName) async {
    //获取应用文件目录类似于Ios的NSDocumentDirectory和Android上的 AppData目录
    final fileDirectory = await getApplicationDocumentsDirectory();

    //获取存储路径
    final filePath = fileDirectory.path;

    //或者file对象（操作文件记得导入import 'dart:io'）
    return new File(filePath + "/"+fileName);
  }

  @override
  Widget build(BuildContext context) {
    return new Scaffold(
      appBar: new AppBar(
        title: new Text('数据存储'),
      ),
      body: new Column(
        mainAxisAlignment: MainAxisAlignment.center,
        children: <Widget>[
          Text("文件存储", textAlign: TextAlign.center),
          TextField(
            controller: _textFieldController,
          ),
          MaterialButton(
            onPressed: saveString,
            child: new Text("存储"),
            color: Colors.cyan,
          ),
          MaterialButton(
            onPressed: getString,
            child: new Text("获取"),
            color: Colors.deepOrange,
          ),
          Text('从文件存储中获取的值为  $_storageString'),
        ],
      ),
    );
  }
}
```

### 3.Sqflite
在Flutter中的数据库叫`Sqflite`跟原生安卓的`Sqlite`叫法不一样。我们来看下`Sqflite`官方对它的解释说明：
```java
SQLite plugin for Flutter. Supports both iOS and Android.

 Support transactions and batches
 Automatic version managment during open
 Helpers for insert/query/update/delete queries
 DB operation executed in a background thread on iOS and Android
```

通过上面的描述，我们了解到`Sqflite`是一个同时支持Android跟Ios平台的数据库，并且支持标准的`CURD`操作，下面我们还是用上面操作文件跟sp的代码逻辑是一块体验一下`Sqflite`。

同样需要引入依赖：
```java
  #添加Sqflite依赖
  sqflite: ^1.0.0
```

模拟场景：
>利用Sqflite创建一张user表，其中user表中id设置为主键id，且为自增长，name字段为text类型，用户按下存储按钮后，把TextFile输入框里的内容插入到user表中，当按下获取按钮时，取出数据库中最后一条数据显示在下方Text上，并且显示出当前数据库中一共有多少条数据，以及数据库的存储路径。


**效果图**
![Sqflite](http://upload-images.jianshu.io/upload_images/7274320-6b02e0f43c337296.gif?imageMogr2/auto-orient/strip)
**上述描述样式代码**
```java
import 'package:flutter/material.dart';
import 'package:path_provider/path_provider.dart';
import 'dart:io';
import 'package:sqflite/sqflite.dart';

class StoragePage extends StatefulWidget {
  @override
  State<StatefulWidget> createState() => StorageState();
}

class StorageState extends State {
  var _textFieldController = new TextEditingController();
  var _storageString = '';

  /**
   * 利用Sqflite数据库存储数据
   */
  saveString() async {
    final db = await getDataBase('my_db.db');
    //写入字符串
    db.transaction((trx) {
      trx.rawInsert(
          'INSERT INTO user(name) VALUES("${_textFieldController.value.text.toString()}")');
    });
  }

  /**
   * 获取存在Sqflite数据库中的数据
   */
  Future getString() async {
    final db = await getDataBase('my_db.db');
    var dbPath = db.path;
    setState(() {
      db.rawQuery('SELECT * FROM user').then((List<Map> lists) {
        print('----------------$lists');
        var listSize = lists.length;
        //获取数据库中的最后一条数据
        _storageString = lists[listSize - 1]['name'] +
            "\n现在数据库中一共有${listSize}条数据" +
            "\n数据库的存储路径为${dbPath}";
      });
    });
  }

  /**
   * 初始化数据库存储路径
   */
  Future<Database> getDataBase(String dbName) async {
    //获取应用文件目录类似于Ios的NSDocumentDirectory和Android上的 AppData目录
    final fileDirectory = await getApplicationDocumentsDirectory();

    //获取存储路径
    final dbPath = fileDirectory.path;

    //构建数据库对象
    Database database = await openDatabase(dbPath + "/" + dbName, version: 1,
        onCreate: (Database db, int version) async {
      await db.execute("CREATE TABLE user (id INTEGER PRIMARY KEY, name TEXT)");
    });

    return database;
  }

  @override
  Widget build(BuildContext context) {
    return new Scaffold(
      appBar: new AppBar(
        title: new Text('数据存储'),
      ),
      body: new Column(
        mainAxisAlignment: MainAxisAlignment.center,
        children: <Widget>[
          Text("Sqflite数据库存储", textAlign: TextAlign.center),
          TextField(
            controller: _textFieldController,
          ),
          MaterialButton(
            onPressed: saveString,
            child: new Text("存储"),
            color: Colors.cyan,
          ),
          MaterialButton(
            onPressed: getString,
            child: new Text("获取"),
            color: Colors.deepOrange,
          ),
          Text('从Sqflite数据库中获取的值为  $_storageString'),
        ],
      ),
    );
  }
}

```

至此，关于Flutter的本地存储相关的内容就全部讲解完了，在本文章中，我为了清晰代码结构跟业务逻辑，复用的都是同一个存储业务逻辑跟UI便于大家结合代码做对比，读者可结合代码自行对比三种存储方式的细节差别。

