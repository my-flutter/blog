---
title: Flutter入门进阶之旅（十三）Flutter 路由
date: 2019-02-20 15:33:58
tags:
---
### Flutter路由介绍
>跟Web页或者原生APP一样，我们在使用Flutter 开发APP时也会涉及到多页面之间的跳转、参数传递、参数回传等业务，`Flutter路由`能满足上述我们提到的所有业务类型，此外我们也可以结合`Flutter动画`给路由跳页时添加个性化的跳页动画操作，我会在后续`Flutter动画`章节中具体讲解。通过本节专题，读者不仅仅可以自己动手做一些简单的UI，还能利用`Fluttter 路由`结合之前的课程分享做一些简单的多页面Flutter App。

##### 本期课程目标
- 了解并掌握Flutter路由的简单使用
- 掌握Flutter动态路由跟静态路由的区别
- 掌握Flutter路由在页面间的参数传递以及回传流程
- 借助路由结合之前的课程做一些简单的多页面Flutter APP

#### 关于静态路由跟动态路由
在Flutter中路由分为`静态路由`跟`动态路由`两种：Flutter中所谓的静态路由指的是需要提前把各个需要跳转的页面路径注册在`routes: <String, WidgetBuilder> {}`中，且静态路由不支持向下一个页面传递参数，但是`可以接收下一个页面的返回值`。
动态路由使用就相对来说比较灵活一点，动态路由同样支持向下一个页面传递参数，而且在使用时不需要我们提前规划好页面路径，只需要在具体跳页逻辑中自己去构造`MaterialPageRoute`对象来完成页面跳转，或者用`PageRouterBuilder`来自定义路由跳转时的动画，关于`路由跳转动画`我会在Flutter动画章节中具体讲解，当然动态路由同样也支持页面参数回传。

### 路由场景分类
#### 1.静态路由跳页
>场景一：点击A页面中的按钮跳转到B页，不涉及到数据传递以及回传

**效果图**
![静态路由跳页](http://upload-images.jianshu.io/upload_images/7274320-67940bcdd5f06467.gif?imageMogr2/auto-orient/strip)
静态路由使用时要求我们在`MaterialApp`中的routes中提前注册路由
```java
new MaterialApp(
      home: new FlutterDemo(),
      routes: <String, WidgetBuilder>{
        'router/new_page': (_) => new StaticNavigatorPage()
      }));
      
  ```

**A页面代码**：
```java
import 'package:flutter/material.dart';
import 'package:flutter_app/pages/simpleWidget/navigator/StaticNavigatorPage.dart';

void main() {
  runApp(new MaterialApp(
      home: new FlutterDemo(),
      routes: <String, WidgetBuilder>{
        'router/new_page': (_) => new StaticNavigatorPage()
      }));
}

class FlutterDemo extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return new Scaffold(
      appBar: new AppBar(
        title: new Text("Flutter进阶之旅"),
      ),
      body: new Center(
          child: new RaisedButton(
              child: new Text("静态路由跳页"),
              onPressed: () {
                Navigator.of(context)
                    .pushNamed('router/new_page'); //这里一定要保证跳页的路由路径跟上面注册的路径一致
              })),
    );
  }
}
```

**B页面代码**：
```java
import 'package:flutter/material.dart';

class StaticNavigatorPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return new Scaffold(
      appBar: new AppBar(
        title: new Text("静态路由页"),
      ),
      floatingActionButton: new FloatingActionButton(
        onPressed: () {
          Navigator.of(context).pop();
        },
        child: new Text("返回"),
      ),
      body: new Center(
        child: Text("静态路由可以传入一个routes参数来定义路由。但是这里定义的路由是静态的，"
            "它不可以向下一个页面传递参数，利用push到一个新页面,pushNamed方法是有一个Future的返回值的"
            "，所以静态路由也是可以接收下一个页面的返回值的。但是不能向下一个页面传递参数"),
      ),
    );
  }
}
```

**上述借助路由跳页过程中我们注意到，大概分以下几步：**
1.注册路由且保证路由的唯一性
2.跳页时使用` Navigator.of(context).pushNamed('路由地址');`
3.使用`Navigator.of(context).pop();`结束当前页

#### 2.静态路由跳页接收下一页的返回值
>场景二：点击A页面上的按钮跳页到B页面，在B页面销毁后A页面接收到B页面回传回来的值并且显示在`AlertDialog`上

**效果图**
![静态路由回传数据](http://upload-images.jianshu.io/upload_images/7274320-1253db2c72ea9a62.gif?imageMogr2/auto-orient/strip)
**A页面代码：**
使用Navigator  push一个新页面,pushNamed方法是有一个Future的返回值的，在then回调中监听并接收新页面回传回来的数据，并且借助`showDialog`显示在`Dialog`上

```java
import 'package:flutter/material.dart';
import 'package:flutter_app/pages/simpleWidget/navigator/StaticNavigatorPageWithParams.dart';

void main() {
  runApp(
      new MaterialApp(home: new FlutterDemo(), routes: <String, WidgetBuilder>{
    'router/new_page_with_callback': (_) => new StaticNavigatorPageWithResult()
  }));
}

class FlutterDemo extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return new Scaffold(
      appBar: new AppBar(
        title: new Text("Flutter进阶之旅"),
      ),
      body: new Center(
          child: new RaisedButton(
              child: new Text("静态路由接收下一页返回值"),
              onPressed: () {
                Navigator.of(context)
                    .pushNamed('router/new_page_with_callback')
                    .then((value) {
                  showDialog(
                      context: context,
                      child: new AlertDialog(
                        content: new Text(value),  
                      ));
                });
              })),
    );
  }
}
```

**B页面代码：**

从效果图中可以看到，当我点击B页面中间的按钮时会把数据回传给上一页，但是直接点击导航栏左上角的返回按钮回到A页面时并不会把数据传递给上一个页面，这里是因为我在B页面的按钮上pop页面出栈的时候把参数放进里面作为了参数传递,`pop()`可接收一个Object对象作为参数传递

```java
Navigator.of(context).pop(T extends Object);
```
这就告诉我们当我们需要给上一个页面回传数据的时候可直接借助pop传递`Navigator.of(context).pop("页面结束后返回的数据");`，不需要传值的时候直接返回空对象`Navigator.of(context).pop()`即可。
```java
import 'package:flutter/material.dart';

class StaticNavigatorPageWithResult extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return new Scaffold(
      appBar: new AppBar(
        title: new Text("静态路由带返回参数"),
      ),
      body: new Center(
        child: new OutlineButton(
          onPressed: () {
            Navigator.of(context).pop("页面结束后返回的数据");
          },
          child: Text("点我返回上个页面结束后返回的数据"),
        ),
      ),
    );
  }
}

```

#### 3.动态路由跳页
文章的开头我们提到借助动态路由可以向下一个页面传递参数，同样也可以接收新页面回传回来的数据。下面我模拟两个使用动态路由的场景，既然动态路由可以传值那就肯定可以不传值跳页，我就不模拟动态路由不传值跳页的例子了，读者借助静态路由的样例自行测试即可，我们主要讲解一下借助动态路由传值的例子。

>场景三：点击A页面中的按钮，把用户名跟密码传递给下一个页面B页面，B页面处理完接收到的数据后把结果回传给A页面，A页面中把从B页面回传回来的数据显示在Dialog上。

**模拟效果图**
![动态路由传递参数](http://upload-images.jianshu.io/upload_images/7274320-e7296f5c7b9921f9.gif?imageMogr2/auto-orient/strip)
使用动态路由时我们不再需要像静态路由那样在`MaterialApp`中的router中提前注册路由路径，只需要在使用`Navigator.push`传入MaterialPageRoute对象即可
```java
 Navigator.push(
    context,
    new MaterialPageRoute(
        //_代表参数为空
        builder: (_) => new DynamicNaviattionPage(
              username: "xiedong",
              password: "123456",
            )));
```

**A页面代码：**
```java
import 'package:flutter/material.dart';
import 'package:flutter_app/pages/simpleWidget/navigator/DynamicNavigationPage.dart';

void main() {
  runApp(new MaterialApp(home: new FlutterDemo()));
}

class FlutterDemo extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return new Scaffold(
        appBar: new AppBar(
          title: new Text("Flutter进阶之旅"),
        ),
        body: new Center(
            child: new RaisedButton(
          onPressed: () {
            Navigator.push(
                context,
                new MaterialPageRoute(
                    //_代表参数为空
                    builder: (_) => new DynamicNaviattionPage(
                          username: "xiedong",
                          password: "123456",
                        ))).then((value) {
              showDialog(
                  context: context,
                  child: new AlertDialog(
                    content: new Text(value),
                  ));
            });
          },
          child: new Text("动态路由传参"),
        )));
  }
}
```

**B页面代码：**
```java
import 'package:flutter/material.dart';

class DynamicNaviattionPage extends StatelessWidget {
  var username;
  var password;

  DynamicNaviattionPage({Key key, this.username, this.password})
      : super(key: key);

  @override
  Widget build(BuildContext context) {
    return new Scaffold(
      appBar: new AppBar(
        title: new Text("动态路由"),
      ),
      body: new Center(
        child: new Column(
          children: <Widget>[
            new MaterialButton(
              onPressed: () {
                Navigator.pop(context, "未查询到改该用户信息");
              },
              child: new Text("点我返回"),
              color: Colors.lightGreen,
            ),
            new Text("上页传递过来的username   $username"),
            new Text("上页传递过来的password   $password"),
          ],
        ),
      ),
    );
  }
}
```


**关于路由重点总结**
- Flutter路由分为静态路由跟动态路由，静态路由不支持向下一个页面传递参数，动态路由可传递
- 静态路由使用时需要提前注册声明页面路径，
- 动态路由可以直接在使用时构造路由对象，不需要提前注册路径。
- 两种类型都支持接收下一页面回传回来的值

