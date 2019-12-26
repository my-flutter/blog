---
title: Flutter入门进阶之旅（二）Hello Flutter
date: 2018-11-15 13:36:21
tags:
---

### 开题 ###
 好像几乎我们学习或者掌握任何一门编程语言都是Hello word开始的，本篇博文做为Flutter入门进阶的第一篇分享，我们也从最简单的Hello world开始，至于Flutter开发环境的配置，跟关于Dart语言的介绍，不是该专栏要讲解的内容，我就不详细做介绍了，读者可自行google或者百度了解一下。

### 准备工作 ###
在开始之前我想先为大家介绍一下Flutter中两个常用的组件MaterialApp跟Scaffold，读者在此不需要完全掌握它们，后续还会有专门的专题对二者讲解，在此大家先简单了解MaterialApp是我们app开发中常用的符合Material Design设计理念的入口Widget，是与我们经常打交道的Widget，也就是我们渲染UI的整体入口，而Scaffold从字面意思大家就能看出是脚手架、骨架的也是，也就是它作为我们app的骨架，快速帮我们打造一个可供二次构建定制的模板，我们可以在上面可以做很多个性化的UI定制。关于Hello Flutter这一章节我们先简单了解这么多即可，读者不用急于马上掌握Flutter的所有内容，今天先简单跟Flutter打个招呼，有个简单的认识即可，后续我们会一起慢慢学习Flutter里面各种Widget。


![image](http://upload-images.jianshu.io/upload_images/7274320-d2831161a2f174c7?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```

import 'package:flutter/material.dart';
 
void main() {
  runApp(new MaterialApp(home: new HelloFlutter()));
}
 
class HelloFlutter extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return new Scaffold(
      appBar: new AppBar(
        title: new Text("Hello Flutter"),
      ),
      body: new Center(child: new Text("Hello Flutter")),
    );
  }
}

```