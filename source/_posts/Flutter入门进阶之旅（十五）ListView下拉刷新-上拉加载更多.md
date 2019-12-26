---
title: Flutter入门进阶之旅（十五）ListView下拉刷新&上拉加载更多
date: 2019-04-12 10:46:28
tags:
---

### 上期回顾
>在上一篇博文中我们在介绍ListView跟GridView的时候，限于篇幅问题我们只讲解了此二者的简单的使用方法，关于一些在实际开发中更常用的细节问我们并没有来得及跟大家展开讲解，比如我们在使用长列表的时的下拉刷新或者上拉加载更多的逻辑处理，今天的这篇文章我们就来着重分析一下在flutter中我们是如果实现长列表的下拉刷新跟上拉加载更多操作的。

### 前言
现实开发中长列表布局几乎是所有APP的标配，几乎你所使用的任何一款app都能找到长列表的身影，而长列表中必不可少的操作肯定是下拉刷新、上拉加载更多。在原生Android中我们一般使用`RecyclerView`配合`support.v4`包下面的`SwipeRefreshLayout`来完成下拉刷新动作，通过给`RecyclerView`绑定`RecyclerView.OnScrollListener()`拖动监听事件来判断列表的滑动状态来决定是否进行加载更多的操作，有了原生Android开发的经验，我们完全可以把这个思路同样应用在Flutter中的长列表操作上。下面我们一起来看下Flutter中的下拉刷新跟上拉加载更多吧。

#### 1.下拉刷新
Flutter跟Android作为Google的亲儿子无论是在在风格命名还是设计思路上都有很大的相似跟想通性，上一篇博文中我们提到Flutter中使用ListViiew跟GridView来完成长列表布局，跟原生Android命名都一样，在Flutter中给我们提供的`RefreshIndicator`组件跟原生Android中的`SwipeRefreshLayout`设计思路一样，都是为了简化我们完成下拉刷新的监听动作。而且`RefreshIndicator`跟原生Android的`SwipeRefreshLayout`在外观上几乎也一样，都遵循了google material design设计理念。
```java
 /// Creates a refresh indicator.
  ///
  /// The [onRefresh], [child], and [notificationPredicate] arguments must be
  /// non-null. The default
  /// [displacement] is 40.0 logical pixels.
  ///
  /// The [semanticsLabel] is used to specify an accessibility label for this widget.
  /// If it is null, it will be defaulted to [MaterialLocalizations.refreshIndicatorSemanticLabel].
  /// An empty string may be passed to avoid having anything read by screen reading software.
  /// The [semanticsValue] may be used to specify progress on the widget. The
  const RefreshIndicator({
    Key key,
    @required this.child,
    this.displacement = 40.0, //圆环进度条展示居顶部的位置
    @required this.onRefresh, //刷新回调
    this.color,  //圆环进度条颜色
    this.backgroundColor, //背景颜色
    this.notificationPredicate = defaultScrollNotificationPredicate,
    this.semanticsLabel,
    this.semanticsValue,
  }) 
  ```
  在上面的构造方法中必要参数我都给了详细的注释说明，所以这里我就不多展开讲解了，先来看一张下拉刷新的效果图，稍后结合代码我再做具体讲解。
  
**效果图**
![在这里插入图片描述](http://upload-images.jianshu.io/upload_images/7274320-d3deb41690c42b09.gif?imageMogr2/auto-orient/strip)
  
**样例代码**

```java
import 'package:flutter/material.dart';

void main() {
  runApp(new MaterialApp(
    title: "ListView",
    debugShowCheckedModeBanner: false,
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
    for (int i = 0; i < 10; i++) {
      entityList.add(ItemEntity("Item  $i", Icons.accessibility));
    }
  }

  Future<Null> _handleRefresh() async {
    print('-------开始刷新------------');
    await Future.delayed(Duration(seconds: 2), () {
      //模拟延时
      setState(() {
        entityList.clear();
        entityList = List.generate(
            10,
            (index) =>
                new ItemEntity("下拉刷新后--item $index", Icons.accessibility));
        return null;
      });
    });
  }

  @override
  Widget build(BuildContext context) {
    return new Scaffold(
        appBar: new AppBar(
          title: new Text("ListView"),
        ),
        body: RefreshIndicator(
            displacement: 50,
            color: Colors.redAccent,
            backgroundColor: Colors.blue,
            child: ListView.builder(
              itemBuilder: (BuildContext context, int index) {
                return ItemView(entityList[index]);
              },
              itemCount: entityList.length,
            ),
            onRefresh: _handleRefresh));
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
    return Container(
        height: 100,
        child: Stack(
          alignment: Alignment.center,
          children: <Widget>[
            Text(
              itemEntity.title,
              style: TextStyle(color: Colors.black87),
            ),
            Positioned(
                child: Icon(
                  itemEntity.iconData,
                  size: 30,
                  color: Colors.blue,
                ),
                left: 5)
          ],
        ));
  }
}
```

上述代码我还是借助上篇博文中讲解ListView的例子，只不过ListView的外层用`RefreshIndicator`包裹了一下，并且给`RefreshIndicator`的`onRefresh`绑定了处理下拉刷新事件的回调函数。
```java
Future<Null> _handleRefresh() async {
    print('-------开始刷新------------');
    await Future.delayed(Duration(seconds: 2), () {
      //模拟延时
      setState(() {
        entityList.clear();
        entityList = List.generate(
            10,
            (index) =>
                new ItemEntity("下拉刷新后--item $index", Icons.accessibility));
        return null;
      });
    });
  }
```
在_handleRefresh（）中，我们通过Future.delayed模拟延时操作，在延时函数执行完毕之后，首先清空我们在initState中模拟的列表数据，然后重新生成刷新后的数据，通过setState重新渲染ListView上绑定的数据来完成下拉刷下这一操作。

#### 2.上拉加载更多
继续完善下拉刷新的代码，我们借助`ScrollController`给ListView添加滑动监听事件
```java
     ListView.builder(
              itemBuilder: (BuildContext context, int index) {
                if (index == entityList.length) {
                  return LoadMoreView();
                } else {
                  return ItemView(entityList[index]);
                }
              },
              itemCount: entityList.length + 1,
              controller: _scrollController,
            ),
            onRefresh: _handleRefresh));
  ```

  然后通过`_scrollController`监听手指上下拖动时在屏幕上产生的滚动距离来判断是否触发加载更多的操作
```java
_scrollController.addListener(() {
      if (_scrollController.position.pixels ==_scrollController.position.maxScrollExtent) {
        print("------------加载更多-------------");
        _getMoreData();
      }
    });
 ```
 我们借助`ScrollController`来判断当前ListView可拖动的距离是否等于listview的最大可拖动距离，如果等于，那么就会触发加载更多的操作，然后我们去做相应的逻辑从而完成加载更多的操作。
 
 其实现在讲完Flutter中下拉刷新跟加载更多的操作，你会发现在Flutter中处理这些操作跟开篇提到的原生Android的处理方式跟思想几乎是一致的，我们来看下完成下拉刷新跟加载更多后的完整效果图。

**效果图**
![加载更多](http://upload-images.jianshu.io/upload_images/7274320-78237b277e781256.gif?imageMogr2/auto-orient/strip)
先来看下整体代码，稍后我在结合代码讲解一下这里需要注意的一个小细节。

```java
import 'package:flutter/material.dart';

void main() {
  runApp(new MaterialApp(
    title: "ListView",
    debugShowCheckedModeBanner: false,
    home: new MyApp(),
  ));
}

class MyApp extends StatefulWidget {
  @override
  State<StatefulWidget> createState() => MyState();
}

class MyState extends State {
  List<ItemEntity> entityList = [];
  ScrollController _scrollController = new ScrollController();
  bool isLoadData = false;

  @override
  void initState() {
    super.initState();
    _scrollController.addListener(() {
      if (_scrollController.position.pixels ==
          _scrollController.position.maxScrollExtent) {
        print("------------加载更多-------------");
        _getMoreData();
      }
    });
    for (int i = 0; i < 10; i++) {
      entityList.add(ItemEntity("Item  $i", Icons.accessibility));
    }
  }

  Future<Null> _getMoreData() async {
    await Future.delayed(Duration(seconds: 2), () { //模拟延时操作
      if (!isLoadData) {
        isLoadData = true;
        setState(() {
          isLoadData = false;
          List<ItemEntity> newList = List.generate(5, (index) =>
          new ItemEntity(
              "上拉加载--item ${index + entityList.length}", Icons.accessibility));
          entityList.addAll(newList);
        });
      }
    });
  }

  Future<Null> _handleRefresh() async {
    print('-------开始刷新------------');
    await Future.delayed(Duration(seconds: 2), () { //模拟延时
      setState(() {
        entityList.clear();
        entityList = List.generate(10,
                (index) =>
            new ItemEntity("下拉刷新后--item $index", Icons.accessibility));
        return null;
      });
    });
  }

  @override
  Widget build(BuildContext context) {
    return new Scaffold(
        appBar: new AppBar(
          title: new Text("ListView"),
        ),
        body: RefreshIndicator(
            displacement: 50,
            color: Colors.redAccent,
            backgroundColor: Colors.blue,
            child: ListView.builder(
              itemBuilder: (BuildContext context, int index) {
                if (index == entityList.length) {
                  return LoadMoreView();
                } else {
                  return ItemView(entityList[index]);
                }
              },
              itemCount: entityList.length + 1,
              controller: _scrollController,
            ),
            onRefresh: _handleRefresh));
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
    return Container(
        height: 100,
        child: Stack(
          alignment: Alignment.center,
          children: <Widget>[
            Text(
              itemEntity.title,
              style: TextStyle(color: Colors.black87),
            ),
            Positioned(
                child: Icon(
                  itemEntity.iconData,
                  size: 30,
                  color: Colors.blue,
                ),
                left: 5)
          ],
        ));
  }
}

class LoadMoreView extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Container(child: Padding(
      padding: const EdgeInsets.all(18.0),
      child: Center(
        child: Row(children: <Widget>[
          new CircularProgressIndicator(),
          Padding(padding: EdgeInsets.all(10)),
          Text('加载中...')
        ], mainAxisAlignment: MainAxisAlignment.center,),
      ),
    ), color: Colors.white70,);
  }

}
```

上述代码中关于listView的itemBuilder的部分代码我做下简单的解释说明：
```java
            ListView.builder(
              itemBuilder: (BuildContext context, int index) {
                if (index == entityList.length) { //是否滑动到底部
                  return LoadMoreView();
                } else {
                  return ItemView(entityList[index]);
                }
              },
              itemCount: entityList.length + 1,
              controller: _scrollController,
            ),
            
   ```

对比一开始下拉刷新的代码，细心的读者可能注意到，加载更多逻辑处理是在itemBuilder的时候多了一个逻辑判断
```java
 if (当前Item的角标==数据集合的长度) {  //滑动到最底部的时候
        显示加载更多的布局  LoadMoreView();
   }else{
       显示正常的Item布局  ItemView();
   }

```
然后就是itemCount的数量自然也要加1，` itemCount: entityList.length + 1`加的这个1就是最底部的LoadMoreView的布局。关于`_scrollController`所触发的回调函数我就不多做讲解了，跟处理下拉刷新时的逻辑代码一样，读者可结合上述完整代码自行对比理解。

好了，至此关于ListView的上拉加载更多跟下拉刷新操作我就为大家讲解完毕了，至于GridView的上下拉操作跟listView原理一样，我就不在此过多废话了，读者可自行写代码测试GridView的上下拉刷新操作。