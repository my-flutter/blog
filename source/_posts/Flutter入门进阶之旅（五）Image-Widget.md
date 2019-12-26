---
title: Flutter入门进阶之旅（五）Image Widget
date: 2018-12-03 18:26:46
tags:
---
### 往期专题回顾：
>前面我们学习了Flutter中用于文本显示的Widget，比如我们显示一行或者一段基本文字会用到Text Widget，如果需要跟Text设置样式，颜色等属性，我们可以通过给Text指定style来定制TextStyle中的样式来展示我们需要的效果，对于文本输入控件，我们学习了TextField，了解到可以通过TextField完成简单的文本输入需求，可以通过InputDecoration给输入框添加样式，比如舒服辅助提示、边框、两边的icon等等。

今天我们来学习一下另外一个Widget---Image，顾名思义Image是用于展示图片的控件，他跟Text一样，同属于StatelessWidget，关于StatelessWidget跟StatefullWidget我会在稍后的文章中具体讲解，在此读者可暂且忽略这一知识点，在做原生Android开发时，我们可以给ImageView指定不同的图片来源，可以是来源网络、drawable、bitmap、文件等，在Flutter中同样支持加载不同来源的图片，只不过Flutter加载不同资源的图片跟原生Android稍微有点差别，下面我们一起进入本期的主题。

### Flutter加载不同资源图片的方式：
- new Image, 用于从ImageProvider获取图像。
- new Image.asset, 用于从AssetBundle获取图像。
- new Image.network, 用于从URL获取图像。
- new Image.file, 用于从文件中获取图像。
- new Image.memory, 用于从内存中获取图像

> 在flutter中Image支持JPEG, PNG, GIF, Animated GIF, WebP, Animated WebP, BMP, 和 WBMP这几种图片格式。

1. 从网络加载图片：

为了养成良好的代码阅读习惯，我们还是先来看下Image.network的构造方法
```

Image.network(String src, {
    Key key,
    double scale: 1.0,//缩小倍数
    this.width,//宽
    this.height,//高
    this.color,
    this.colorBlendMode,
    this.fit, //填充方式
    this.alignment: Alignment.center,
    this.repeat: ImageRepeat.noRepeat, //图片排列方式
    this.centerSlice,
    this.matchTextDirection: false,
    this.gaplessPlayback: false,
    Map<String, String> headers,
  })
```
通过构造方法，我们可以初步清楚的了解到在Flutter中加载网络图片，只需要在Image.network中指定图片的src，也就是目标图片的url即可，如果需要配置图片宽高、缩放比对应构造方法去写就好，下面实例代码，展示了从目标url上加载图片，并且把图片的宽高设置为500*500；

 效果图：
![image](http://upload-images.jianshu.io/upload_images/7274320-c6716f6f63ec15ee?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

 示例代码：
```

import 'package:flutter/material.dart';
 
void main() {
  runApp(new MaterialApp(home: new ImageDemo()));
}
 
class ImageDemo extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return new Scaffold(
      appBar: new AppBar(
        title: new TextField(
          decoration: new InputDecoration(),
        ),
      ),
      body: new Image.network("https://p1.ssl.qhmsg.com/dr/220__/t01d5ccfbf9d4500c75.jpg",width:500,height: 500,)
    );
  }
}
```
2. 从项目本地目录加载图片：
> Flutter中加载项目目录下的图片，需要我们先从pubspec.yaml中声明导入资源后，才能在dart文件中使用图片资源，不然即使图片存在项目目录下，在dart文件中你指定路径后也不能正常加载，这点读者应注意一下，跟我们写原生Android有点不同。

比如我在项目中把images文件夹下的两张图片aaa.png、a.png通过pubspec.yaml导入到项目中：
![image](http://upload-images.jianshu.io/upload_images/7274320-570f3b186a0bd8eb?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

把上述代码中body部分换成Image.asset方式。
```
body: new Image.asset('images/aaa.png',width: 500,height: 500,)
```
> 关于Image.asset的构造方法跟network大同小异，我就不贴出来了，读者可自行查阅源码，下面看下我们从项目目录下加载的图片结束我们本期关于Image的学习。

 效果图：

![image](http://upload-images.jianshu.io/upload_images/7274320-172fb9727db00d2c?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


