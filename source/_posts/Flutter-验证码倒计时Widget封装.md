---
title: Flutter 验证码倒计时Widget封装
date: 2020-02-29 11:55:45
tags:
---
#### 引言
>在日常开发过程中，像倒计时这样的场景使用的还是比较多的，比如延时完成一段逻辑，或者在启动页先加载一个闪屏广告，倒计时间到之后再进入app，更常见的场景就是我们在获取手机验证码时用于友好提示用户的等待试图。本次博文我们就一起来了解下基于flutter封装一个倒计时widget的全过程

#### 课程知识点
- 1.关于Timer的使用
- 2.回调函数的传值
- 3.组件封装思想的建立

在开始今天的博文之前，先来看下今天课程所讲内容的效果图：

**效果图**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200228131233947.gif)

#### 逻辑梳理
 从上图我们可以分析得出
 
 1.整个过程倒计时widget一共分为两种状态
- 倒计时中：按照我们设定好的时间，每次递减一秒，直到剩余时间为0.在此期间按钮字体颜色为灰色，按钮不可再次接收点击事件
- 初始状态或完成倒计时：按钮字体颜色为蓝色，点击按钮进入倒计时状态。

2.按钮上的倒计时逻辑，我们借助dart async包下的Timer来完成
```java
Timer.periodic(Duration duration, void callback(Timer timer)) 
```
从方法签名中，我们可以看出，Timer.periodic接收两个参数，分别为时间间隔，跟回调函数。我们利用传入时间间隔为1秒为周期，然后在回调函数中执行，每次时间减1的操作，如果当前剩余时间小于1，我们结束当前Timer，否则一直执行回调函数
```java
    Timer _timer;
    int _countdownTime = 10;

    _timer = Timer.periodic(
        Duration(seconds: 1),
            (Timer timer) =>
        {
          setState(() {
            if (_countdownTime < 1) {
              _timer.cancel();
            } else {
              _countdownTime = _countdownTime - 1;
            }
          })
        });
            
  ```

其他部分涉及到按钮上状态跟点击事件的处理在入门进阶专栏的系列文章中，我都详细讲解过，这里就不展开细说了，读者自行结合代码阅读理解吧。

##### 封装好的倒计时Widget代码：
```java
import 'dart:async';

import 'package:flutter/material.dart';

/**
 * @desc
 * @author xiedong
 * @date   2020-02-28.
 */

class TimerCountDownWidget extends StatefulWidget {
  Function onTimerFinish;

  TimerCountDownWidget({this.onTimerFinish}) : super();

  @override
  State<StatefulWidget> createState() => TimerCountDownWidgetState();
}

class TimerCountDownWidgetState extends State<TimerCountDownWidget> {
  Timer _timer;
  int _countdownTime = 0;

  @override
  Widget build(BuildContext context) {
    return GestureDetector(
      onTap: () {
        if (_countdownTime == 0) {
          setState(() {
            _countdownTime = 60;
          });
          //开始倒计时
          startCountdownTimer();
        }
      },
      child: RaisedButton(
        color: Colors.black12,
        child: Text(
          _countdownTime > 0 ? '$_countdownTime后重新获取' : '获取验证码',
          style: TextStyle(
            fontSize: 14,
            color: _countdownTime > 0
                ? Colors.white
                : Color.fromARGB(255, 17, 132, 255),
          ),
        ),
      ),
    );
  }

  void startCountdownTimer() {
//    const oneSec = const Duration(seconds: 1);
//    var callback = (timer) => {
//      setState(() {
//        if (_countdownTime < 1) {
//          widget.onTimerFinish();
//          _timer.cancel();
//        } else {
//          _countdownTime = _countdownTime - 1;
//        }
//      })
//    };
//
//    _timer = Timer.periodic(oneSec, callback);


    _timer = Timer.periodic(
        Duration(seconds: 1),
        (Timer timer) => {
              setState(() {
                if (_countdownTime < 1) {
                  widget.onTimerFinish();
                  _timer.cancel();
                } else {
                  _countdownTime = _countdownTime - 1;
                }
              })
            });
  }

  @override
  void dispose() {
    super.dispose();
    if (_timer != null) {
      _timer.cancel();
    }
  }
}

```

#####  在页面中作为Widget使用
```java 
import 'package:flutter/cupertino.dart';
import 'package:flutter/material.dart';
import 'package:flutter_app/pages/custom_widget/widget/timer_count_down_widget.dart';

/**
 * @desc
 * @author xiedong
 * @date   2020-02-28.
 */

class VerficationCodePage extends StatefulWidget {
  @override
  State<StatefulWidget> createState() => VerficationCodePageState();
}

class VerficationCodePageState extends State<VerficationCodePage> {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text("验证码倒计时"),
        centerTitle: true,
      ),
      body: Center(
        child: TimerCountDownWidget(onTimerFinish: (){
          print("倒计时结束--------");
        },),
      ),
    );
  }
}
```

运行代码后，我们点击获取验证码按钮，当倒计时结束后，我们传入的回调函数会被调用，如下图在log控制台打印出我们在程序中输入的内容：




![在这里插入图片描述](https://img-blog.csdnimg.cn/20200228131336844.png)

本次博文相关代码：[博文源代码](https://github.com/xiedong11/flutter_app/blob/master/lib/pages/custom_widget/widget/timer_count_down_widget.dart)