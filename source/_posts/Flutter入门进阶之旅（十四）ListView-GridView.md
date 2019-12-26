---
title: Flutter入门进阶之旅（十四）ListView&GridView
date: 2019-04-10 12:42:34
tags:
---
>在之前讲`Layout Widget`的文章中，我们掌握了基于不同的场景适当的选择不同的Widget来完成我们的布局要求，但是关于长列表的数据展示我们并没有做展开介绍，而长列表的身影几乎出现在日常生活中的任意一款APP中，鉴于它的重要性，所以我想单独作为一个章节来讲解长列表Widget---`ListView&GirdView`。

### 1.ListView 
##### 1.1 ListView简单列表
看下ListView的构造方法，然后我们用listview来完成一个简单列表
```java
ListView({
    Key key,
    Axis scrollDirection: Axis.vertical,//滚动方向
    bool reverse: false,//是否反向显示数据
    ScrollController controller,
    bool primary,
    ScrollPhysics physics,//物理滚动
    bool shrinkWrap: false,
    EdgeInsetsGeometry padding,
    this.itemExtent,//item有效范围
    bool addAutomaticKeepAlives: true,//自动保存视图缓存
    bool addRepaintBoundaries: true,//添加重绘边界
    List<Widget> children: const <Widget>[],
  })
```
![ListView简单列表](http://upload-images.jianshu.io/upload_images/7274320-43e74b2f3a6fe470.gif?imageMogr2/auto-orient/strip)
**上述效果图样例代码**
```java
import 'package:flutter/material.dart';

void main() {
  runApp(new MaterialApp(
    title: "ListView",
    home: new MyApp(),
  ));
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return new Scaffold(
        appBar: new AppBar(
          title: new Text("ListView"),
        ),
        body: ListView(
          scrollDirection: Axis.vertical, //控制列表方向
          children: <Widget>[
            new Container(
              width: 100.0,
              height: 100.0,
              color: Colors.lightBlue,
            ),
            new Container(
              width: 100.0,
              height: 100.0,
              color: Colors.red,
            ),
            new Container(
              width: 100.0,
              height: 100.0,
              color: Colors.deepPurple,
            ),
            new Container(
              width: 100.0,
              height: 100.0,
              color: Colors.yellow,
            ),
            new Container(
              width: 100.0,
              height: 100.0,
              color: Colors.lightBlue,
            ),
            new Container(
              width: 100.0,
              height: 100.0,
              color: Colors.red,
            ),
            new Container(
              width: 100.0,
              height: 100.0,
              color: Colors.deepPurple,
            ),
            new Container(
              width: 100.0,
              height: 100.0,
              color: Colors.lightBlue,
            ),
            new Container(
              width: 100.0,
              height: 100.0,
              color: Colors.red,
            ),
            new Container(
              width: 100.0,
              height: 100.0,
              color: Colors.deepPurple,
            ),
            new Container(
              width: 100.0,
              height: 100.0,
              color: Colors.lightBlue,
            ),
          ],
        ));
  }
}
```
上述代码中我们利用ListView做了一个简单的长列表布局，通过给`children: <Widget>[]`传入我们要展示的子Widget来完成列表渲染，理论上我们可以传入任意多的子Widget，但是这点使用起来跟`Column`没有太大差别，这显然是不符合我们的预期的，因为我们在使用ListView展示长列表布局的时候，大多数情况下我们是不知道长列表一共有多少个元素，即使是能知道，我们重复性的把每个相似的布局都写一遍，也几乎让人崩溃。

没错，Flutter肯定给我们提供了像原生Android写Adapter一样的方式，我们可以提前写好公共的Item布局，然后通过`ListView.builder（）或者ListView.custom（）`方式自动生成长列表，`ListView.builder（）`和`ListView.custom（）`的用法基本相同，只不过custom可以根据自己的需要控制Item显示方式，如Item显示大小。
##### 1.2 可复用的ListView长列表
下面我带大家一起看下ListView.builder()的构造方法，然后使用ListView.builder写个简单的样式代码，至于ListView.custom的用法我会在下面介绍GridView的时候讲到，读者可参考Listview.bulider自行测试。
```java
ListView.builder({
    Key key,
    Axis scrollDirection: Axis.vertical,
    bool reverse: false,
    ScrollController controller,
    bool primary,
    ScrollPhysics physics,
    bool shrinkWrap: false,
    EdgeInsetsGeometry padding,
    this.itemExtent,
    @required IndexedWidgetBuilder itemBuilder,//item构建者
    int itemCount,//item数量
    bool addAutomaticKeepAlives: true,
    bool addRepaintBoundaries: true,
  })

```
从ListView.bulider的构造方法我们可以看出，它只比上述我们直接使用的ListView的构造方法多了两个参数，也正是这两个参数简化了我们对长列表的操作
>`itemCount：`：被展示的Item的数量
>`itemBuilder`:被展示的Item的构造者（这里读者可以它类比成原生Android的Adapter）

下面我们来看下效果图：
![长列表](http://upload-images.jianshu.io/upload_images/7274320-0c751b27caa9c0d9.gif?imageMogr2/auto-orient/strip)
**样例代码**
```java
import 'package:flutter/material.dart';

void main() {
  runApp(new MaterialApp(
    title: "ListView",
    home: new MyApp(),
  ));
}

class MyApp extends StatefulWidget {
  @override
  State<StatefulWidget> createState() => MyState();
}

class MyState extends State {
  List<ItemEntity> entityList = [];

  @override
  void initState() {
    super.initState();
    for (int i = 0; i < 30; i++) {
      entityList.add(ItemEntity("Item  $i", Icons.accessibility));
    }
  }

  @override
  Widget build(BuildContext context) {
    return new Scaffold(
        appBar: new AppBar(
          title: new Text("ListView"),
        ),
        body: ListView.builder(
          itemBuilder: (BuildContext context, int index) {
            return ItemView(entityList[index]);
          },
          itemCount: entityList.length,
        ));
  }
}

/**
 * 渲染Item的实体类
 */
class ItemEntity {
  String title;
  IconData iconData;

  ItemEntity(this.title, this.iconData);
}

/**
 * ListView builder生成的Item布局，读者可类比成原生Android的Adapter的角色
 */
class ItemView extends StatelessWidget {
  ItemEntity itemEntity;

  ItemView(this.itemEntity);

  @override
  Widget build(BuildContext context) {
    return Flex(
      direction: Axis.vertical,
      children: <Widget>[
        ListTile(
            leading: Icon(itemEntity.iconData), title: Text(itemEntity.title),subtitle: Text('长列表')),
        SizedBox(
          height: 0.2,
          child: Container(
            color: Colors.black,
          ),
        )
      ],
    );
  }
}
```
上述代码中的逻辑读者大部分都可以自解的，我就不多做赘述了，下面我们进入本篇分享的第二部分，关于GridView部分的讲解。

### 2.GridView
跟原生Android一样，`GridView`满足了我们所有使用表格布局的场景，当然如果GridView在每行或者每列都只显示一个Item的时候又相当于ListView的使用场景。但是回到Flutter上`GridView`的用法又跟ListView使用类似，可以直接new对象的方式生成简单的表格布局，也可以像ListView那样通过`builder（）和custom（）`方法来创建可复用的对象

**构造方法**
```java
GridView({
    Key key,
    Axis scrollDirection: Axis.vertical,
    bool reverse: false,
    ScrollController controller,
    bool primary,
    ScrollPhysics physics,
    bool shrinkWrap: false,
    EdgeInsetsGeometry padding,
    @required this.gridDelegate,   //控制GridView显示方式
    bool addAutomaticKeepAlives: true,
    bool addRepaintBoundaries: true,
    List<Widget> children: const <Widget>[],
  })
  ```
  上面提到GridView跟ListView使用方法类似，但是GridView不同于ListView的是它可以在一行或者一列显示多个Item，所以GridView的构造方法对比ListView多了一个`gridDelegate`参数，来配置一行（列）有几个Item和Item的间隔。

**gridDelegate参数说明**
>`gridDelegate`可接收两种参数类型：
>- `SliverGridDelegateWithFixedCrossAxisCount`可以直接指定每行（列）显示多少个Item
>- `SliverGridDelegateWithMaxCrossAxisExtent`会根据GridView的宽度和你设置的每个的宽度来自动计算没行显示多少个Item

先来看下GridView的效果图，关于`gridDelegate`的区别我在代码里做了注释讲解，这里就不在多赘述了读者可结合代码自行理解。

**效果图**
![GridView](http://upload-images.jianshu.io/upload_images/7274320-98f7a6574dc11229?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
**样例代码**
```java
import 'package:flutter/material.dart';

void main() {
  runApp(new MaterialApp(
    title: "ListView",
    home: new MyApp(),
  ));
}

class MyApp extends StatefulWidget {
  @override
  State<StatefulWidget> createState() => MyState();
}
class MyState extends State {
  @override
  Widget build(BuildContext context) {
    return new Scaffold(
        appBar: new AppBar(
          title: new Text("ListView"),
        ),
        body: new GridView(
//          gridDelegate: SliverGridDelegateWithFixedCrossAxisCount(
//              crossAxisCount: 3, //固定显示三个
//              mainAxisSpacing: 10,
//              crossAxisSpacing: 10),
          gridDelegate: SliverGridDelegateWithMaxCrossAxisExtent(
              maxCrossAxisExtent: 100, //根据maxCrossAxisExtent的值对比屏幕的真实宽度，决定一行或者一列显示多少个Item
              mainAxisSpacing: 10,
              crossAxisSpacing: 10),
          children: <Widget>[
            Container(
                child: Icon(Icons.adb, size: 60),
                color: Colors.deepOrangeAccent),
            Container(
                child: Icon(Icons.adb, size: 60),
                color: Colors.deepOrangeAccent),
            Container(
                child: Icon(Icons.adb, size: 60),
                color: Colors.deepOrangeAccent),
            Container(
                child: Icon(Icons.adb, size: 60),
                color: Colors.deepOrangeAccent),
            Container(
                child: Icon(Icons.adb, size: 60),
                color: Colors.deepOrangeAccent),
            Container(
                child: Icon(Icons.adb, size: 60),
                color: Colors.deepOrangeAccent),
            Container(
                child: Icon(Icons.adb, size: 60),
                color: Colors.deepOrangeAccent),
            Container(
                child: Icon(Icons.adb, size: 60),
                color: Colors.deepOrangeAccent),
          ],
        ));
  }
}

```
在本篇分享的第一部分我们介绍ListView的时候提到，为了复用Item布局，我们可以通过builder()或者custom来生成Item布局，在ListView中我们使用的是`builder()`的方式生成的可复用的布局，在gridView中我带大家使用下`custom()`，其实listView跟GridView使用类似，无非就是布局排列不同而已。

**效果图**

![GridView](http://upload-images.jianshu.io/upload_images/7274320-5b745d224260b370.gif?imageMogr2/auto-orient/strip)

**样例代码**
```java
import 'package:flutter/material.dart';

void main() {
  runApp(new MaterialApp(
    title: "ListView",
    home: new MyApp(),
  ));
}

class MyApp extends StatefulWidget {
  @override
  State<StatefulWidget> createState() => MyState();
}

class MyState extends State {
  List<ItemEntity> entityList = [];

  @override
  void initState() {
    super.initState();
    for (int i = 0; i < 30; i++) {
      entityList.add(ItemEntity("Item  $i", Icons.accessibility));
    }
  }

  @override
  Widget build(BuildContext context) {
    return new Scaffold(
        appBar: new AppBar(
          title: new Text("GridView"),
        ),
        body: GridView.custom(
            gridDelegate: SliverGridDelegateWithFixedCrossAxisCount(
              crossAxisCount: 3,
              crossAxisSpacing: 10,
              mainAxisSpacing: 10,
            ),
            childrenDelegate: SliverChildBuilderDelegate(
              (BuildContext context, int index) {
                return ItemView(entityList[index]);
              },
              childCount: entityList.length,
            )));
  }
}

/**
 * 渲染Item的实体类
 */
class ItemEntity {
  String title;
  IconData iconData;

  ItemEntity(this.title, this.iconData);
}

/**
 * GridView builder生成的Item布局，读者可类比成原生Android的Adapter的角色
 */
class ItemView extends StatelessWidget {
  ItemEntity itemEntity;

  ItemView(this.itemEntity);

  @override
  Widget build(BuildContext context) {
    return GestureDetector(
      child: Flex(
        direction: Axis.vertical,
        children: <Widget>[
          Icon(
            itemEntity.iconData,
            size: 60,
          ),
          Text(itemEntity.title),
        ],
      ),
      onTap: () {
        Scaffold.of(context).showSnackBar(
            new SnackBar(content: Text("点击了${itemEntity.title}")));
      },
    );
  }
}
```

### 总结
通过本篇博文我们了解到ListView跟GridView都可以直接用new对象的方式生成列表布局，一个用于表格布局，另一个用于长列表布局，也可以使用new 或者builder（）和custom（）方法来创建可复用对象，从而扩展了ListView跟GridView的灵活性，就像我们原生Android一样，把可复用的布局抽象成Adapter的形式，但是或许你总觉得关于列表还少点什么，下拉刷新？加载更多？

