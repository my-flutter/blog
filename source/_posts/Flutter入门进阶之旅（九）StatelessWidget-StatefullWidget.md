---
title: Flutter入门进阶之旅（九）StatelessWidget & StatefullWidget
date: 2019-01-05 14:39:21
tags:
---
### 引言
> 在前面的学习中我们接触到了flutter中各种基本组件的使用，也学习了一些常用的布局排版方式，掌握了根据不同的UI widget合理的选用不同的Layout方式进行布局，但是我们好像在前面的学习中所有的UI都是静态的，没有任何交互式的体验，换句话说我们在之前所掌握的flutter知识都是比较死板的静态UI页，缺少了那么一点灵动性，那今天这篇文章就算是一个过渡，今天我会带领大家简单认识下flutter中的动态页。

### 布局对比
细心的读者可能有留意到在之前我们的讲解中大部分页的根widget都是继承StatelessWidget，像我们之前讲Text、Image、各种Layout，包括上一节讲Button的时候
```java 
class TextPage extends StatelessWidget{
....
class ImagePage extends StatelessWidget {
...
class LayoutPage extends StatelessWidget {
...
class LayoutPage extends StatelessWidget {
...
```
但是在讲TextField这一节的时候我们的根widget继承的却是StatefullWidget
```java
class TextFieldPage extends StatefulWidget {
...
```
**思考**
![一脸懵逼](http://upload-images.jianshu.io/upload_images/7274320-06299abb9b0fe5e4.jpeg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
> 1.这会有什么不一样吗？
> 2.这跟今天的课程有什么关系呢？
> 3.我应该如何选择，或者我怎么确定何时选用那种根Widget呢？

### 关于StatelessWidget跟StatefullWidget

关于StatelessWidget跟StatefullWidget网上一搜一大推类似的博文介绍，我不想再走老套路，把官方的翻译贴出来，然后大家看完之后还是云里雾里的不知所以然，我想把今天的分享搞的轻松跟通俗一点，不会这么的官方也不会那么的晦涩，我从我个人的理解出发去分析此二者的区别。

其实上面提到的疑惑我相信大多数刚接触flutter的读者应该都会有类似的想法，我们怎么区别二者，何时如何选用二者中的某一个呢？我结合咱们之前的分享用求同存异的观点看问题吧，细心的读者可能会发现我们之前的分像Text、Image、Button、各种Layout、甚至包括GestureDetector这些章节的分享，我们绘制的UI页都是静态的，换句话说就是一旦这些UI页被成功渲染之后就不需要页不可能去改变他的状态，就是一开始是什么样就是什么样，在UI上没有任何的变化。

### 代码分析
![举个例子](http://upload-images.jianshu.io/upload_images/7274320-7713fcc4f8344bf1.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
> 场景一：我要在UI上显示一串文字，这串文字从始至终都不需要改变，也不可能会改变，这种场景下跟布局就需要选用StatelessWidget

```java
import 'package:flutter/material.dart';

class TextPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return new Scaffold(
        appBar: new AppBar(
          title: new Text("Hello Flutter"),
        ),
        body: new Center(
          child: new Text(
            "我从UI被渲染完成之后就这个状态，不可能发生改变",
            style: new TextStyle(fontSize: 18.0),
            textAlign: TextAlign.center,
          ),
        ));
  }
}
```
**效果图**
![StatelessWidget](http://upload-images.jianshu.io/upload_images/7274320-7ca7a9fb5f032667?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


如上图，这串文字从一开始就是这样，也永远是这样，这种场景下你就可以选择用StatelessWidget来渲染你的根布局，当然用StatefullWidget也能完成，但是需要用StatefullWidget实现的布局用StatelessWidget是不能完成的。

> 场景二：UI页上有一个按钮，我每次点击按钮UI页上的Text显示内容加1

这种情况下，我们很清楚的知道当前的UI页是不固定的，换句话说，UI页上的控件可能会在某一个时刻或者某种逻辑状态下改变自身的状态，那这个时候StatelessWidget显然是不能完成这一要求的，我们来用StatefullWidget模拟上场景二的具体实现。

**效果图**
![StatefWidget](http://upload-images.jianshu.io/upload_images/7274320-35d220653d0791e5.gif?imageMogr2/auto-orient/strip)

**示例代码**
```java
import 'package:flutter/material.dart';

class TextPage extends StatefulWidget {
  @override
  State<StatefulWidget> createState() => MyState();
}

class MyState extends State<TextPage> {
  var _count = 0;

  @override
  Widget build(BuildContext context) {
    return new Scaffold(
        appBar: new AppBar(
          title: new Text("Hello StatefulWidget"),
        ),
        body: new Stack(
          children: <Widget>[
            new Align(
              child: new Text(
                "当前count值:$_count",
                style: new TextStyle(fontSize: 18.0),
                textAlign: TextAlign.center,
              ),
            ),
            new Align(
              alignment: new FractionalOffset(0.5, 0.0),
              child: new MaterialButton(
                color: Colors.blueAccent,
                textColor: Colors.white,
                onPressed: () {
                  //重新渲染当前UI页的状态
                  setState(() {
                    _count++;
                  });
                },
                child: new Text('点我加下方文字自动加1'),
              ),
            ),
          ],
        ));
  }
}
```
通过上述代码我们得知在StatefullWidget中通过setState通知重新渲染当前UI页上的所有Widget来完成改变状态。

### 总结
通过今天的学习我们从原先死板的UI静态页过渡到了状态可改变的UI绘制，了解到了StatelessWidget和StatefullWidget的区别，并且能根据不同的UI绘制场景合理的选用不同的根Widget，比如我们所要绘制的UI页的状态包括被渲染的内容都是始终不变的，那我们会选用StatelessWidget来完成，如果所绘制的UI可能在未来的某个场景下发生变化我们会选用StatefullWidget来实现。


