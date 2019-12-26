---
title: Flutter入门进阶之旅（七）GestureDetector
date: 2018-12-24 18:34:39
tags:
---
### 引言：
GestureDetector在Flutter中负责处理跟用户的简单手势交互，GestureDetector控件没有图像展示，只是检测用户输入的手势，并作出相应的处理，包括点击、拖动和缩放。许多控件使用GestureDetector为其他控件提供回调，比如IconButton、RaisedButton和FloatingActionButton控件有onPressed回调，当用户点击控件时触发回调，当用户点击控件时触发回调。

我们来一起看下GestureDetector的构造方法：
```java
 GestureDetector({
    Key key,
    this.child,
    this.onTapDown,
    this.onTapUp,
    this.onTap,
    this.onTapCancel,
    this.onDoubleTap,
    this.onLongPress,
    this.onLongPressUp,
    this.onVerticalDragDown,
    this.onVerticalDragStart,
    this.onVerticalDragUpdate,
    this.onVerticalDragEnd,
    this.onVerticalDragCancel,
    this.onHorizontalDragDown,
    this.onHorizontalDragStart,
    this.onHorizontalDragUpdate,
    this.onHorizontalDragEnd,
    this.onHorizontalDragCancel,
    this.onForcePressStart,
    this.onForcePressPeak,
    this.onForcePressUpdate,
    this.onForcePressEnd,
    this.onPanDown,
    this.onPanStart,
    this.onPanUpdate,
    this.onPanEnd,
    this.onPanCancel,
    this.onScaleStart,
    this.onScaleUpdate,
    this.onScaleEnd,
    this.behavior,
    this.excludeFromSemantics = false
  }) 


```
从构造方法中，我们看出GestureDetector构造方法里定义各种事件回调，还有一个child属性，这就意味着我们可以利用GestureDetector包裹本身不支持点击回调事件的Widget赋予它们点击回调能力，像Text、Image我们就不能像使用RaisedButton一样直接给Text、Image绑定onPress回调，但是我们可以借助GestureDetector完成这一操作。

**如图我给Text赋予了各种事件交互：**
![Flutter入门进阶之旅（七）GestureDetector](http://upload-images.jianshu.io/upload_images/7274320-b373f57bd6c535b4?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```java
import 'package:flutter/material.dart';

void main() {
  runApp(new MaterialApp(home: new MyApp()));
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {

    return new Scaffold(
      appBar: new AppBar(
        title: new Text("Gestures"),
      ),
      body: new Center(
          child: new GestureDetector(
            child: new Text("我被赋予了点击触摸能力...",style: new TextStyle(fontSize: 20.0),),
            onTap: () {
              print("------onTap");
            },
            onDoubleTap: () {
              print("------onDoubleTap");
            },
            onLongPress: () {
              print("-----onLongPress");
            },
            onVerticalDragStart: (details){
              print("在垂直方向开始位置:"+details.globalPosition.toString());
            }, onVerticalDragEnd: (details){
            print("在垂直方向结束位置:"+details.primaryVelocity.toString());
          },
          )),
    );
  }
}
```
我们在log命令行抓取到的各种回调事件的交互：
![在这里插入图片描述](http://upload-images.jianshu.io/upload_images/7274320-72f4993c6840d45c?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

