---
title: Flutter入门进阶之旅（三）Text Widgets
date: 2018-11-19 13:37:48
tags:
---
Text Widgets是Flutter中一个十分常用的一个Widget，类似于Android平台下的TextView，几乎在每个App的UI中都会或多或少的出现它的身影，让我们去一睹Text的风采吧！

#### 简单Text使用 ####
```

import 'package:flutter/material.dart';
 
void main() {
  runApp(new MaterialApp(home: new TextDemo()));
}
 
class TextDemo extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return new Scaffold(
      appBar: new AppBar(
        title: new Text("Hello Flutter"),
      ),
      body: new Center(
          child: new Text(
        "This is Flutter Widget ---- Text ，is a StatelessWidget",
        style: new TextStyle(
          fontStyle: FontStyle.italic,
          fontSize: 20.0,
          color: Colors.red,
        ),
        textAlign: TextAlign.center,
      )),
    );
  }
}
```
#### 效果图 ####
![image](http://upload-images.jianshu.io/upload_images/7274320-d52b1c5a69b4d499?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

Flutter中的Text Widget跟Android平台下的TextView十分类似，我们也可以跟在原生Android平台下一样的指定Text显示的样式，文字大小，颜色等，来一起看一下Flutter中关于Text的构造方法，以及Text里面有哪些属性可供开发者自己定制。
```
const Text(this.data, {  //Text显示的内容
Key key,
this.style, //Text显示的样式
this.textAlign,//文本应该如何水平对齐,TextAlign.start,end 或者center
this.textDirection, //文本方向,TextDirection.ltr\TextDirection.rtl
this.locale,
this.softWrap,  //是否自动换行，若为false，文字将不考虑容器大小，单行显示，超出屏幕部分将默认截断处理
this.overflow, //当文字超出屏幕的时候，如何处理,TextOverflow.clip(裁剪)\TextOverflow.fade(渐隐)\TextOverflow.ellipsis(省略号)
this.textScaleFactor, //字体显示倍率，上面的例子使用的字体大小是20.0，将字体设置成10.0，然后倍率为2
this.maxLines, //最大行数设置
this.semanticsLabel,
})
```
上述的属性中，我们使用的最多的就是TextStyle属性了，比如我们想自己设定Text显示的颜色，大小，或者下划线、删除线等等各种各样的奇葩样式都可以通过TextStyle来指定，看下TextStyle的构造方法说明：
```
const TextStyle({
    this.inherit: true,         // 为false的时候不显示
    this.color,                    // 颜色 
    this.fontSize,               // 字号
    this.fontWeight,           // 字重，加粗也用这个字段  FontWeight.w700
    this.fontStyle,                // FontStyle.normal  FontStyle.italic斜体
    this.letterSpacing,        // 字符间距  就是单个字母或者汉字之间的间隔，可以是负数
    this.wordSpacing,        // 字间距 句字之间的间距
    this.textBaseline,        // 基线，两个值，字面意思是一个用来排字母的，一人用来排表意字的（类似中文）
    this.height,                // 当用来Text控件上时，行高（会乘以fontSize,所以不以设置过大）
    this.decoration,        // 添加上划线，下划线，删除线 
    this.decorationColor,    // 划线的颜色
    this.decorationStyle,    // 这个style可能控制画实线，虚线，两条线，点, 波浪线等
    this.debugLabel,
    String fontFamily,    // 字体
    String package,
  })
```
TextStyle里面的样式我就不逐个为大家贴效果图了，读者可自行模拟测试下期基本用户，我贴上我的全部样例代码跟统一的效果图供大家参考。

#### 效果图 ####
![image](http://upload-images.jianshu.io/upload_images/7274320-511d1e9226937316?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
# 上图的完整示例代码 #
```
import 'package:flutter/material.dart';
 
void main() {
  runApp(new MaterialApp(home: new TextDemo()));
}
 
class TextDemo extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return new Scaffold(
        appBar: new AppBar(
          title: new Text("Hello Flutter"),
        ),
        body: new Center(
          child: new Column(
            crossAxisAlignment: CrossAxisAlignment.center,
            children: <Widget>[
              new Text(
                'inherit: 为 false 的时候不显示',
                style: new TextStyle(
                  fontSize: 18.0,
                  color: Colors.redAccent,
                  inherit: true,
                ),
              ),
              new Text(
                'color/fontSize: 字体颜色，字号等',
                style: new TextStyle(
                  color: Color.fromARGB(255, 150, 150, 150),
                  fontSize: 22.0,
                ),
              ),
              new Text(
                'fontWeight: 字重',
                style: new TextStyle(
                    fontSize: 18.0,
                    color: Colors.redAccent,
                    fontWeight: FontWeight.w700),
              ),
              new Text(
                'fontStyle: FontStyle.italic 斜体',
                style: new TextStyle(
                  fontStyle: FontStyle.italic,
                ),
              ),
              new Text(
                'letterSpacing: 字符间距',
                style: new TextStyle(
                  letterSpacing: 10.0,
                  // wordSpacing: 15.0
                ),
              ),
              new Text(
                'wordSpacing: 字或单词间距',
                style: new TextStyle(
                    // letterSpacing: 10.0,
                    wordSpacing: 15.0),
              ),
              new Text(
                'textBaseline:这一行的值为TextBaseline.alphabetic',
                style: new TextStyle(textBaseline: TextBaseline.alphabetic),
              ),
              new Text(
                'textBaseline:这一行的值为TextBaseline.ideographic',
                style: new TextStyle(textBaseline: TextBaseline.ideographic),
              ),
              new Text('height: 用在Text控件上的时候，会乘以fontSize做为行高,所以这个值不能设置过大',
                  style: new TextStyle(
                    height: 1.0,
                  )),
              new Text('decoration: TextDecoration.overline 上划线',
                  style: new TextStyle(
                      fontSize: 18.0,
                      color: Colors.redAccent,
                      decoration: TextDecoration.overline,
                      decorationStyle: TextDecorationStyle.wavy)),
              new Text('decoration: TextDecoration.lineThrough 删除线',
                  style: new TextStyle(
                      decoration: TextDecoration.lineThrough,
                      decorationStyle: TextDecorationStyle.dashed)),
              new Text('decoration: TextDecoration.underline 下划线',
                  style: new TextStyle(
                      fontSize: 18.0,
                      color: Colors.redAccent,
                      decoration: TextDecoration.underline,
                      decorationStyle: TextDecorationStyle.dotted)),
            ],
          ),
        ));
  }
}
```