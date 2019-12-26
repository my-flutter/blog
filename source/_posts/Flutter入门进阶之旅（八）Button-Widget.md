---
title: Flutter入门进阶之旅（八）Button Widget
date: 2018-12-26 17:49:42
tags:
---
### 引言
>上一篇博文中我们学习了GestureDetector，了解到借助GestureDetector可以赋予某些本身不具备点击回调的widget点击回调能力，完成跟用户的一些简单手势交互并作出相应的逻辑处理，我们还提到了有些widget像RaisedButton和FloatingActionButton控件本身就有onPressed回调，当用户点击控件时触发回调。本篇博客中我们就把上篇博客中提及到的各种Button归类统一说明一下。

**Button集合效果图**
![Button集合效果图](http://upload-images.jianshu.io/upload_images/7274320-32e637cda1f1619d.gif?imageMogr2/auto-orient/strip)


> 上述RaisedButton、FlatButton、OutlineButton、MaterialButton、还有RawMaterialButton、FloationgActionButton，我分三类给大家讲解，OutlineButton作为一类，RaisedButton、FlatButton、MaterialButton、RawMaterialButton不论是从构造方法还是使用上都很相似，这几个button分为一类，另外FloatingActionButton作为一类。

### 1.OutlineButton：
OutlineButton默认带有一个边框，我们可以通过属性指定正常状态，跟用户点击状态下的边框颜色。
![在这里插入图片描述](http://upload-images.jianshu.io/upload_images/7274320-75050c34d38f528c.gif?imageMogr2/auto-orient/strip)

我们来看下OutlineButton的构造方法，并且对它的相应属性做下使用说明。
```java
const OutlineButton({
    Key key,
    @required VoidCallback onPressed,
    ButtonTextTheme textTheme,  //按钮上字体主题
    Color textColor,  //字体颜色
    Color disabledTextColor, //按钮禁用时候文字的颜色
    Color color,  //按钮背景颜色
    Color highlightColor,//点击或者toch控件高亮的时候显示在控件上面，水波纹下面的颜色
    Color splashColor, //水波纹的颜色
    double highlightElevation,//高亮时候的阴影
    this.borderSide,//按钮边框
    this.disabledBorderColor, //按钮禁用时边框的颜色
    this.highlightedBorderColor,//高亮时边框的颜色
    EdgeInsetsGeometry padding,//边距
    ShapeBorder shape, //设置shape
    Clip clipBehavior = Clip.none,
    Widget child,
  }) 
  ```
上述OutlineButton效果图上两个button的具体代码：

**未指定任何样式：**
```java
 new OutlineButton(onPressed: () {}, child: new Text("OutlineButton")),
 ```
 **效果图**
 ![未指定任何样式](http://upload-images.jianshu.io/upload_images/7274320-f2b5a7c4dd4b75aa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
 **设置边框样式**
 ```java
  new OutlineButton(
            textColor: Colors.blue,
            highlightedBorderColor: Colors.deepOrange,
            shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(20.0)),
            borderSide: new BorderSide(color: Colors.blueAccent),
            onPressed: () {},
            child: new Text("OutlineButton")
          ),
```

**效果图**
![设置边框样式](http://upload-images.jianshu.io/upload_images/7274320-0eaf3474f0ac4e3b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 2.FloatingActionButton
FloatingActionButton做过原生安卓开发的读者应该很熟悉这个名字，在Flutter中不光引用了原生安卓的叫法，使用方法也大同小异。
**构造方法**

```java
const FloatingActionButton({
    Key key,
    this.child,
    this.tooltip,
    this.foregroundColor,
    this.backgroundColor,
    this.heroTag = const _DefaultHeroTag(),
    this.elevation = 6.0,
    this.highlightElevation = 12.0,
    @required this.onPressed,
    this.mini = false,
    this.shape = const CircleBorder(),
    this.clipBehavior = Clip.none,
    this.materialTapTargetSize,
    this.isExtended = false,
  })
  ```
  由于使用方法跟原生安卓中类似，我就不展开详细讲解了，稍后我会把篇首效果图上的所有代码贴上，大家可参考里面对floatingactionbutton的用法。
### 3.RaisedButton、FlatButton、OutlineButton、MaterialButton
这几类button十分类似，所以我把他们归为一类做讲解，但是也会有细微的差别，像FlatButton不支持设置手指点击抬起后的颜色，其他常规用法几乎一致，关于具体的细微差别，读者可自行对比构造方法，自行测试，我拿RaisedButton把代码，贴上代码，为大家演示下这几类Button的常用属性配置说明。

```java
 new RaisedButton(
            color: Colors.blueAccent,
            //按钮的背景颜色
            padding: EdgeInsets.all(15.0),
            //按钮距离里面内容的内边距
            textColor: Colors.white,
            //文字的颜色
            textTheme: ButtonTextTheme.normal,
            //按钮的主题
            onHighlightChanged: (bool b) {
              //水波纹高亮变化回调
            },
            disabledTextColor: Colors.black54,
            //按钮禁用时候文字的颜色
            disabledColor: Colors.black54,
            //按钮被禁用的时候显示的颜色
            highlightColor: Colors.green,
            //点击或者toch控件高亮的时候显示在控件上面，水波纹下面的颜色
            splashColor: Colors.amberAccent,
            //水波纹的颜色
            colorBrightness: Brightness.light,
            //按钮主题高亮
            elevation: 10.0,
            //按钮下面的阴影
            highlightElevation: 10.0,
            //高亮时候的阴影
            disabledElevation: 10.0,
            //按下的时候的阴影
            shape: new RoundedRectangleBorder(
                borderRadius: BorderRadius.circular(20.0)),
            //设置形状
            onPressed: () {},
            child: new Text("RaisedButton"),
          ),
  ```


上述讲解中读者如果有不明白的地方可以参考文章开篇中贴出的各种button的效果图源码做参考，或者留言给我一块探讨，文章开篇中贴出的各种button的效果图源码如下。
```java
import 'package:flutter/material.dart';


void main() {
  runApp(new MaterialApp(home: new ButtonPage()));
}
class ButtonPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return new Scaffold(
      appBar: new AppBar(
        title: new Text("Button Widget"),
      ),
      floatingActionButton: new FloatingActionButton(
          onPressed: () {}, child: new Icon(Icons.adb), tooltip: "点击"),
      body: new ListView(
        children: <Widget>[
          new Text("简单样式",
              textAlign: TextAlign.center,
              style: new TextStyle(color: Colors.brown, fontSize: 20.0)),
          new RaisedButton(onPressed: () {}, child: new Text("RaisedButton")),
          new FlatButton(onPressed: () {}, child: new Text("FlatButton")),
          new MaterialButton(
              onPressed: () {}, child: new Text("MaterialButton")),
          new RawMaterialButton(
              onPressed: () {}, child: new Text("RawMaterialButton")),
          new OutlineButton(onPressed: () {}, child: new Text("OutlineButton")),
          new SizedBox(height: 20),
          new Text("升级样式",
              textAlign: TextAlign.center,
              style: new TextStyle(color: Colors.brown, fontSize: 20.0)),
          new RaisedButton(
            color: Colors.blueAccent,
            //按钮的背景颜色
            padding: EdgeInsets.all(15.0),
            //按钮距离里面内容的内边距
            textColor: Colors.white,
            //文字的颜色
            textTheme: ButtonTextTheme.normal,
            //按钮的主题
            onHighlightChanged: (bool b) {
              //水波纹高亮变化回调
            },
            disabledTextColor: Colors.black54,
            //按钮禁用时候文字的颜色
            disabledColor: Colors.black54,
            //按钮被禁用的时候显示的颜色
            highlightColor: Colors.green,
            //点击或者toch控件高亮的时候显示在控件上面，水波纹下面的颜色
            splashColor: Colors.amberAccent,
            //水波纹的颜色
            colorBrightness: Brightness.light,
            //按钮主题高亮
            elevation: 10.0,
            //按钮下面的阴影
            highlightElevation: 10.0,
            //高亮时候的阴影
            disabledElevation: 10.0,
            //按下的时候的阴影
            shape: new RoundedRectangleBorder(
                borderRadius: BorderRadius.circular(20.0)),
            //设置形状
            onPressed: () {},
            child: new Text("RaisedButton"),
          ),
          new FlatButton(
              color: Colors.lightGreen,
              textColor: Colors.red,
              onPressed: () {}, child: new Text("FlatButton")),
          new OutlineButton(
              textColor: Colors.blue,
              highlightedBorderColor: Colors.deepOrange,
              shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(20.0)),
              borderSide: new BorderSide(color: Colors.blueAccent),
              onPressed: () {},
              child: new Text("OutlineButton")
          ),
          MaterialButton(
              color: Colors.yellow,
              textColor: Colors.red,
              onPressed: () {},
              child: new Text("MaterialButton")
          ),
          RawMaterialButton(
              fillColor: Colors.deepOrange,
              onPressed: () {},
              child: new Text("RawMaterialButton",style: new TextStyle(color: Colors.white),)
          )
        ],
      ),
    );
  }
}
``` 

