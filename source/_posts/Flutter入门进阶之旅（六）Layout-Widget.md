---
title: Flutter入门进阶之旅（六）Layout Widget
date: 2018-12-21 18:11:56
tags:
---
### 往期回顾：
> 前面几期的专栏对大家来说学习起来还算轻松加愉快，我们简单认识了flutter这门新技术，并且尝试着学习了像Text、Image、TextField几个简单的Widget，并且我们用这几个Widget做了一些简单的交互，好像我们并没有注重Widget的显示位置跟排版，我们只是让他显示出来而已，然后要想把这些Widget组合起来放在一个渲染到整个手机屏幕上，我们需要合理的选用一个容器来包裹这些Widget，或者说让这些Widget舒适的排列在一个恰当的容器里，就像我们在做原生android时，如果UI上需要绘制一个水平或者竖直的view排列，我们会选用LinearLayout而不会去选FrameLayout，同理在Flutter上布局的排放我们也要适当的选择正确的容器。

在Flutter中也给我们提供了各种不同应用场景的layout，我们可以根据UI上排版的需要来选用不同的layout去完成我们对UI的绘制，在这些layout中，有些layout的借鉴了前端的盒子布局模型，有些完全跟原生android思想一致，所以对于我们来说学习起来并没有那么抽象，下面我列出几个常用的layout，并且举例跟大家一块看看Flutter到底是怎么完成Widget的布局的。

### Flutter中容器：
- Row、Column
- Stack
- Center
- Container
- ListView
- Align
- Padding
- SizedBox
- AspectRadio
- DecoratedBox
- Opactity
### 下面我列出几种常用的Layout使用场景
#### 1.Row、Column
> 这两种布局方式几乎一样，所以我把它俩放在一块讲解，先看下源码对二者的描述

```
class Column extends Flex {
  /// Creates a vertical array of children.
 
 
class Row extends Flex {
  /// Creates a horizontal array of children.

```
> 从源码注释中我们了解到二者都是一个盛放children widget的array，不同的是一个是在水平方向（horizontal），另一个是竖直方向（vertical）

由于二者类似，我们只拿Row做讲解，来一块分析下Row构造方法，看下提供给我们可定制的属性有哪些

```
Row({
  Key key,
  MainAxisAlignment mainAxisAlignment = MainAxisAlignment.start,
  MainAxisSize mainAxisSize = MainAxisSize.max,
  CrossAxisAlignment crossAxisAlignment = CrossAxisAlignment.center,
  TextDirection textDirection,
  VerticalDirection verticalDirection = VerticalDirection.down,
  TextBaseline textBaseline,
  List<Widget> children = const <Widget>[],
})

```
#### 1.1 属性解析

##### 1.1.1 MainAxisAlignment:

> 主轴方向上的对齐方式，会对child的位置起作用，默认是start。其中MainAxisAlignment枚举值：

- center：将children放置在主轴的中心；
- end：将children放置在主轴的末尾；
- spaceAround：将主轴方向上的空白区域均分，使得children之间的空白区域相等，但是首尾child的空白区域为1/2；
- spaceBetween：将主轴方向上的空白区域均分，使得children之间的空白区域相等，首尾child都靠近首尾，没有间隙；
- spaceEvenly：将主轴方向上的空白区域均分，使得children之间的空白区域相等，包括首尾child；
- start：将children放置在主轴的起点；
其中spaceAround、spaceBetween以及spaceEvenly的区别，就是对待首尾child的方式。其距离首尾的距离分别是空白区域的1/2、0、1。

##### 1.1.2 MainAxisSize：

> 在主轴方向占有空间的值，默认是max。MainAxisSize的取值有两种：

- max：根据传入的布局约束条件，最大化主轴方向的可用空间；
- min：与max相反，是最小化主轴方向的可用空间；
##### 1.1.3 CrossAxisAlignment：

> children在交叉轴方向的对齐方式，与MainAxisAlignment略有不同。CrossAxisAlignment枚举值有如下几种：

- baseline：在交叉轴方向，使得children的baseline对齐；
- center：children在交叉轴上居中展示；
- end：children在交叉轴上末尾展示；
- start：children在交叉轴上起点处展示；
- stretch：让children填满交叉轴方向；
##### 1.1.4TextDirection：

>阿拉伯语系的兼容设置，一般无需处理。

##### 1.1.5 VerticalDirection

> 定义了children摆放顺序，默认是down。VerticalDirection枚举值有两种：

- down：从top到bottom进行布局；
- up：从bottom到top进行布局。
- top对应Row以及Column的话，就是左边和顶部，bottom的话，则是右边和底部。

##### 1.1.6 TextBaseline：

>使用的TextBaseline的方式，有两种，前面已经介绍过。

干巴巴的说了这么多确实有点枯燥，上个图直观感受一下Row，我在Row里面水平排列了几个RaisedButton

### 效果图：
![image](http://upload-images.jianshu.io/upload_images/7274320-d7639a9453b6f563?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 样例代码：
```
import 'package:flutter/material.dart';
 
void main() {
  runApp(new MaterialApp(home: new LayoutDemo()));
}
 
class LayoutDemo extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return new Scaffold(
      appBar: new AppBar(
        title: new Text("水平方向布局"),
      ),
 
      //布局方向  Row:水平布局 Column：垂直布局
      body: new Row(
        mainAxisAlignment: MainAxisAlignment.spaceAround,
        children: <Widget>[
          new RaisedButton(
            onPressed: () {
              print('点击红色按钮');
            },
            color: const Color(0xffff0000),
            child: new Text('红色按钮'),
          ),
          new RaisedButton(
            onPressed: () {
              print("点击蓝色按钮");
            },
            color: const Color(0xff000099),
            child: new Text('蓝色按钮'),
          ),
          new RaisedButton(
            onPressed: () {
              print("点击粉色按钮");
            },
            color: const Color(0xffee9999),
            child: new Text('粉色按钮'),
          )
        ],
      ),
    );
    ;
  }
}
```
 cloumn与Row大同小异，读者可自行测试，我就不详细贴代码演示了。下面我们一块来看另外一种layout方式。
### 2.Stack
> Stack即层叠布局，跟原生Android里面的FrameLayout如出一辙，能够将子widget层叠排列。如果不指定显示位置，默认布局在左上角，如果希望子空间显示在具体的位置，我们可以通过Positioned控件包裹子widget，然后根据定位的子控件的top、right、bottom、left属性来将它们放置在Stack的合适位置上。

#### Stack布局效果图：
![image](http://upload-images.jianshu.io/upload_images/7274320-6a239f32df4a4dcd?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 上述效果图样例代码：
```
import 'package:flutter/material.dart';
 
void main() {
  runApp(new MaterialApp(home: new StackLayoutDemo()));
}
 
class StackLayoutDemo extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return new Scaffold(
        appBar: new AppBar(
          title: new Text('层叠布局'),
        ),
        body: new Center(
          child: new Stack(
            children: <Widget>[
              new Image.network(
                'https://avatar.csdn.net/6/0/6/1_xieluoxixi.jpg',
                scale: 0.5,
              ),
              new Positioned(
                  left: 35.0,
                  right: 35.0,
                  top: 45.0,
                  child: new Text(
                    '第二层内容区域',
                    style: new TextStyle(
                      fontSize: 20.0,
                      fontFamily: 'serif',
                    ),
                  )),
              new Positioned(
                  left: 55.0,
                  right: 55.0,
                  top: 55.0,
                  child: new Text(
                    '第三层 this is the third child',
                    style: new TextStyle(
                      fontSize: 20.0,
                      color: Colors.blue,
                      fontFamily: 'serif',
                    ),
                  ))
            ],
          ),
        ));
  }
}
```
### 3.Center
> Center布局使用比较简单，场景也比较单一，一般用于协助其他子widget布局，包裹其child widget显示在上层布局的中心位置。上述Stack布局中就利用Center让其显示在屏幕正中心，看下官方对Center的解释：

```
class Center extends Align {
  /// Creates a widget that centers its child.
  const Center({ Key key, double widthFactor, double heightFactor, Widget child })
    : super(key: key, widthFactor: widthFactor, heightFactor: heightFactor, child: child);
}
```
比较简单直接，我就不贴效果图了，读者可自行利用center包裹子widget做测试，从而直观体验一下center布局。
```
//Center既中心定位控件，能够将子控件放在其内部中心。
class CenterLayoutDemo extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return new Scaffold(
      appBar: new AppBar(
        title: new Text('中心布局'),
      ),
      body: new Center(
        child: new Text('我在屏幕中央'),
      ),
    );
  }
}
```
### 4.ListView
> Listview使用场景跟原生Android一样，都是用于展示长列表数据，也是我们开发中使用的比较多的widget之一，本章节主要分析layout，就不详细展开分析，带大家简单认识下ListView的用法，后续我们专门抽出一个章节讲解ListView，跟listView类似的还有GridView。
#### ListView效果图：
![image](http://upload-images.jianshu.io/upload_images/7274320-58903beb93b84a6d.gif?imageMogr2/auto-orient/strip)

#### 上述实现样例代码：
```
import 'package:flutter/material.dart';
 
void main() {
  runApp(new MaterialApp(home: new ListViewLayoutDemo()));
}
 
class ListViewLayoutDemo extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return new Scaffold(
      appBar: new AppBar(
        title: new Text('滚动布局'),
      ),
      body: new ListView(
        children: <Widget>[
          new Center(
            child: new Text(
              '\n大标题',
              style: new TextStyle(fontFamily: 'serif', fontSize: 20.0),
            ),
          ),
          new Center(
            child: new Text(
              '小标题',
              style: new TextStyle(
                fontFamily: 'serif',
                fontSize: 12.0,
              ),
            ),
          ),
          new Center(
            child: new Text(
              '''
            内容sadf手动阀防守打法 发生富士达发生发生飞都是
            内容sadf手动阀防守打法 发生富士达发生发生飞都是
            内容sadf手动阀防守打法 发生富士达发生发生飞都是
            内容sadf手动阀防守打法 发生富士达发生发生飞都是
            内容sadf手动阀防守打法 发生富士达发生发生飞都是
            内容sadf手动阀防守打法 发生富士达发生发生飞都是
             内容sadf手动阀防守打法 发生富士达发生发生飞都是
            内容sadf手动阀防守打法 发生富士达发生发生飞都是
            内容sadf手动阀防守打法 发生富士达发生发生飞都是 
            内容sadf手动阀防守打法 发生富士达发生发生飞都是
            内容sadf手动阀防守打法 发生富士达发生发生飞都是 
            内容sadf手动阀防守打法 发生富士达发生发生飞都是
            内容sadf手动阀防守打法 发生富士达发生发生飞都是
            内容sadf手动阀防守打法 发生富士达发生发生飞都是
            内容sadf手动阀防守打法 发生富士达发生发生飞都是
            内容sadf手动阀防守打法 发生富士达发生发生飞都是
            内容sadf手动阀防守打法 发生富士达发生发生飞都是
            内容sadf手动阀防守打法 发生富士达发生发生飞都是
             内容sadf手动阀防守打法 发生富士达发生发生飞都是
            内容sadf手动阀防守打法 发生富士达发生发生飞都是
            内容sadf手动阀防守打法 发生富士达发生发生飞都是
            内容sadf手动阀防守打法 发生富士达发生发生飞都是
            内容sadf手动阀防守打法 发生富士达发生发生飞都是
            内容sadf手动阀防守打法 发生富士达发生发生飞都是
             内容sadf手动阀防守打法 发生富士达发生发生飞都是
            内容sadf手动阀防守打法 发生富士达发生发生飞都是
            内容sadf手动阀防守打法 发生富士达发生发生飞都是
            内容sadf手动阀防守打法 发生富士达发生发生飞都是
            内容sadf手动阀防守打法 发生富士达发生发生飞都是
            内容sadf手动阀防守打法 发生富士达发生发生飞都是
             内容sadf手动阀防守打法 发生富士达发生发生飞都是
            内容sadf手动阀防守打法 发生富士达发生发生飞都是
            内容sadf手动阀防守打法 发生富士达发生发生飞都是
            内容sadf手动阀防守打法 发生富士达发生发生飞都是
            内容sadf手动阀防守打法 发生富士达发生发生飞都是
            内容sadf手动阀防守打法 发生富士达发生发生飞都是
           
                             ''',
              style: new TextStyle(fontSize: 14.0),
            ),
          )
        ],
      ),
    );
  }
}
```
### 5.Align：
> Align即对齐控件，能将子控件按照所指定的方式对齐，并根据子控件的大小调整自己的大小。Align对齐子控件的方式有如下几种
```
bottomCenter    (0.5, 1.0)    底部中心
bottomLeft    (0.0, 1.0)    左下角
bottomRight    (1.0, 1.0)    右下角
center    (0.5, 0.5)    水平垂直居中
centerLeft    (0.0, 0.5)    左边缘中心
centerRight    (1.0, 0.5)    右边缘中心
topCenter    (0.5, 0.0)    顶部中心
topLeft    (0.0, 0.0)    左上角
topRight    (1.0, 0.0)    右上角
```
来简单使用一下：
![image](http://upload-images.jianshu.io/upload_images/7274320-e39394450800b2ce?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
 #### 代码：
```
import 'package:flutter/material.dart';
 
void main() {
  runApp(new MaterialApp(home: new AlignLayoutDemo()));
}
class AlignLayoutDemo extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return new Scaffold(
      appBar: new AppBar(
        title: new Text('Align布局'),
      ),
      body: new Stack(
        children: <Widget>[
          new Align(
            alignment: new FractionalOffset(0.0, 0.5),
            child: new Text(
              '我在左边缘中心',
              style: new TextStyle(fontSize: 35.0),
            ),
          ),
          new Align(
            alignment: new FractionalOffset(1.0, 1.0),
            child: new Text(
              '我在右下角',
              style: new TextStyle(fontSize: 30.0),
            ),
          )
        ],
      ),
    );
  }
}
```
### 6.SizedBox 
> SizedBox能够强制控制子控件的宽高显示，比如我指定一个宽高都为200的SiezedBox布局，其里面的child widget的宽高也就被限定死了最大宽高为SizedBox的宽高，即使他有更大的宽高。
```
import 'package:flutter/material.dart';
 
void main() {
  runApp(new MaterialApp(home: new SizedBoxLayoutDemo()));
}
class SizedBoxLayoutDemo extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return new Scaffold(
      appBar: new AppBar(
        title: new Text('SizedBox布局'),
      ),
      body: new SizedBox(
        width: 200.0,
        height: 200.0,
        child: new Container(
          decoration: new BoxDecoration(color: Colors.red),
        ),
      ),
    );
  }
}
```
利用SizedBox限定Container的宽高为200：
![image](http://upload-images.jianshu.io/upload_images/7274320-93e88a8d08daa582?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 7.Opacity
> Opacity控件能调整子控件的不透明度，使子控件部分透明，不透明度的量从0.0到1.1之间，0.0表示完全透明，1.1表示完全不透明。

如图：我在StackLayout中放置两个子widget，其中Text在Stack下方，我把上层Opacity布局的透明度设置为0.5，我们可以隐约看见下层Text显示的内容，读者可自行把透明度更改值测试结果。
![image](http://upload-images.jianshu.io/upload_images/7274320-8afa7209a0f0bccc?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```
import 'package:flutter/material.dart';
 
void main() {
  runApp(new MaterialApp(home: new OpacityLayoutDemo()));
}
 
//Opacity控件能调整子控件的不透明度，使子控件部分透明，不透明度的量从0.0到1.1之间，0.0表示完全透明，1.1表示完全不透明。
class OpacityLayoutDemo extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return new Scaffold(
      backgroundColor: Colors.white,
      appBar: new AppBar(
        title: new Text('Opacity'),
      ),
      body: new Center(
        child: new Stack(
          alignment: AlignmentDirectional.center,
          children: <Widget>[
            new Text("我在透明区域下方"),
            new Opacity(
              opacity: 0.5,
              child: new Container(
                width: 200.0,
                height: 220.0,
                decoration: new BoxDecoration(color: Colors.redAccent),
              ),
            ),
          ],
        ),
      ),
    );
  }
}
```
本篇内容有点多，旨在带大家一起总结掌握Flutter中常用的几种布局，另外几种布局在现实开发中使用场景比较少，就没有逐一讲解分析， 读者感兴趣的话，可以自行查阅官方文档尝试下具体用法。

