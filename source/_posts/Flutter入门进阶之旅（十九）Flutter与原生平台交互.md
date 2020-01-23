---
title: Flutter入门进阶之旅（十九）Flutter与原生平台交互
date: 2019-12-31 20:29:58
tags:
---
#### 引言：
>经过前面章节的学习，相信读者已经对flutter有了一个整体的认识，并且也能利用flutter平台提供的一些基础组件自己写一些简单的页面逻辑，甚至有些读者可能已经在用纯flutter开发属于自己的app了，但是可能好多读者都会感觉到有些场景下或者说有些原生平台的东西从flutter端是无法获取的，比如系统版本、电池电量、动态权限申请等系统级的API，flutter并没有直接给我提供相关的API去操作，这个时候我们可能就需要通过借助Native的能力或者与原生平台交互来获取这些数据。

#### 课程目标
- 了解并掌握flutter与原生通信的方法
- 掌握flutter与原生通过MethodChannel相互回调的实现机制
- 掌握原生平台通过EventChannel主动向Flutter传递数据


##### 1.Flutter与原生通信
flutter在与native端进行通信主要借助MethodChannel跟EventChannel建立连接，MethodChannel跟EventChannel就像管道或者桥梁一样，把flutter端跟native连接到一块。
我们来看下官方对此的解释：
>A named channel for communicating with platform plugins using asynchronous  method calls.
>使用异步方法调用与平台插件通信的指定通道。

- 1.）MethodChannel方法签名
```java
    public MethodChannel(BinaryMessenger messenger, String name) {
        this(messenger, name, StandardMethodCodec.INSTANCE);
    }
```
- 2.）EventChannel方法签名
```java
    public EventChannel(BinaryMessenger messenger, String name) {
        this(messenger, name, StandardMethodCodec.INSTANCE);
    }
   ```

为了保证用户界面在交互过程中的流畅性，无论是从Flutter向Native端发送消息，还是Native向Flutter发送消息都是以`异步的形式进行传递`的。
 >1.在整个交互过程中，无论是Flutter端还是Native端都可以通过MethodChannel向对方平台发送两端`提前定义好的方法名`来调用对方平台相对应的消息处理逻辑并且带回返回值给被调用方。
 >2.而EventChannel的使用场景更侧重于Native平台主动向Flutter平台单向给Flutter平台发送消息，Flutter无法返回任何数据给Native端，笔者更愿意把EventChannel描述成是单通的。


到目前为止，我们已经简单的对flutter于native端的通信交互方式有了一个简单的了解，下面我们先来看贴上本节课程要完成的代码效果图，然后再对效果图中提到的案例逐个分析讲解：

![平台交互](https://img-blog.csdnimg.cn/20191231083326293.gif)

##### 2.效果图代码案例分析
- Flutter端通过MethodChannel调用Native平台弹出Toast
- Flutter利用MechtondChannl向Natvie端传递参数调用Native端相关函数，Native接收参数后，并把处理完成后的结果返回给Flutter端。
- Flutter端利用MethodChannel打开原生新页面
- Native端利用MethodChannel传递参数到Flutter端，并接收从Flutter端传递回来的处理后到数据
- Native端利用EventChannel主动向Flutter端发送数据（消息）


两端交互的方法注册逻辑也比较简单，读者一看便知，我就不过多展开描述，下面贴上关于两端交互的两个代表性的例子:
> 1.Flutter 调用原生函数，计算两个数的和并获取处理完成的结果到Flutter端
 2.Native端利用EventChannel主动向Flutter端发送数据（消息）
 

 ##### 2.1 Flutter 调用原生函数，计算两个数的和并获取处理完成的结果到Flutter端
 **场景分析** ：可类比Flutter端处理不了的业务逻辑，这个时候把必要的参数传递给原生，原生处理接收到参数，处理完成后，再把结果回传到Flutter端，完成整个业务逻辑。

**注册通道**
为了确保Flutter与Native能建立正常的通信，我们首先要保证两端注册的MethocChannel的`通道名`channelName一致，如下分别在两端注册channelName：

Native端：
```java
private static final String METHOD_CHANNEL = "com.zhuandian.flutter/android";
 methodChannel = new MethodChannel(getFlutterView(), METHOD_CHANNEL);
 ```
Flutter端：
```java
 static final String METHOD_CHANNEL = "com.zhuandian.flutter/android";
   static final MethodChannel _MethodChannel =MethodChannel(METHOD_CHANNEL); //平台交互通道
```
**Android端：**
```java
public class MainActivity extends FlutterActivity {
    private static final String METHOD_CHANNEL = "com.zhuandian.flutter/android";
    private static final String METHOD_NUMBER_ADD = "numberAdd"; //简单加法计算，并返回两个数的和
    private MethodChannel methodChannel;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
//        setContentView(R.layout.activity_main);
        GeneratedPluginRegistrant.registerWith(this);


        methodChannel = new MethodChannel(getFlutterView(), METHOD_CHANNEL);
        //接受fltuter端传递过来的方法，并做出响应逻辑处理
        methodChannel.setMethodCallHandler(new MethodChannel.MethodCallHandler() {
            @Override
            public void onMethodCall(MethodCall call, MethodChannel.Result result) {
                System.out.println(call.method);
               if (call.method.equals(METHOD_NUMBER_ADD)) {
                    int number1 = call.argument("number1");
                    int number2 = call.argument("number2");
                    result.success(number1 + number2); //返回两个数相加后的值
                } 
            }
        });

    }
    
}
```

**Flutter 端：**
```java
class AndroidPlatformPage extends StatefulWidget {
  @override
  State<StatefulWidget> createState() => PageState();
}

class PageState extends State<AndroidPlatformPage> {
  static final String METHOD_CHANNEL = "com.zhuandian.flutter/android";
  static final String NATIVE_METHOD_ADD = "numberAdd"; //原生android平台定义的供flutter端唤起的方法名
  static final MethodChannel _MethodChannel = MethodChannel(METHOD_CHANNEL); //平台交互通道

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text("平台交互"),
        centerTitle: true,
      ),
      body: Column(
        mainAxisAlignment: MainAxisAlignment.center,
        crossAxisAlignment: CrossAxisAlignment.stretch,
        children: <Widget>[
          RaisedButton(
            color: Colors.orangeAccent,
            child: Text("计算两个数的和"),
            onPressed: () {
              getNumberResult(25, 36);
            },
          ),
        ],
      ),
    );
  }

 
  /**
   * 调用平台方法计算两个数的和，并调用原生toast打印出结果
   */
  void getNumberResult(int i, int j) async {
    Map<String, dynamic> map = {"number1": 12, "number2": 43};
    int result = await _MethodChannel.invokeMethod(NATIVE_METHOD_ADD, map);
    print("调用原生平台计算后的结果为：${result}");
  }
}
```

>通过MethodChannel传递参数并且获取返回值，两端的执行逻辑完全一样，无论是从Flutter端回调Native端端数据，还是Native回调Flutter端数据都是先由被唤起端通过
`methodChannel.setMethodCallHandler`跟`methodChannel.invokeMethod(String method, [ dynamic arguments ])`,一个负责处理回调，一个负责发送事件并且携带参数。

###### 2.1.1事件接收处理端
接收处理回调时`onMethodCall(MethodCall call, MethodChannel.Result result)`通过**methodCall**接收事件发送者传递回来的信息，通过**Result**把处理完的结果发送给事件发送方。
 - 通过`methodCall.method`：来区分不同函数名（方法）名以执行不同的业务逻辑，
 - 通过`methodCall.hasArgument（"key"）`：判断是否有某个key对应的value
 - 通过`methodCall.argument（"key"）`：获取key对应的value值
 - 通过`result.success(object)`：把处理完的结果返回给事件发送方

###### 2.1.2事件发送端
处理事件发送方通过`methodChannel.invokeMethod("方法名","要传递的参数")`把需要传递的参数传递给事件监听者。
其中
- `方法名`:不能为空
- `要传递的参数`:可以为空，若不为空则必须为可Json序列化的对象。

然后监听者通过在`setMethodCallHandler`做出相关操作，对传递过来的参数做出相对于的逻辑处理后再把结果传递回来，以此完成整个平台交互过程。

>上述例子flutter端作为事件发送方，Native端作为事件接收处理方，其实MethodChannel的设计完全可以让二者身份互换，即从Native端去发送消息到Flutter然后获取返回结果，实现的逻辑就是上述的逆过程，在真实开发中应用中笔者认为从原生端通过EventChannel主动向Flutter发消息的使用场景要远远多于**Native端通过MethodChannel回调去处理Flutter端的返回数据**，所以就不单独在这里展开赘述**Native端通过MethodChannel回调去处理Flutter端返回的数据**了，感兴趣的读者可以查看本篇博客配套代码中去查阅相关代码示例跟注释解读，下面我们来看下通过EventChannel主动向Flutter发消息的具体操作流程。

##### 2.2 Native端利用EventChannel主动向Flutter端发送数据（消息）

场景分析：实现从Native端主动向Flutter端传递数据：页面成功渲染后从原生向Flutter端发送一个字符串显示在Flutter端端Text Widget上，当点击Flutter界面上的按钮后，调用原生的方法，主动向Flutter发送消息，更新Flutter端端UI显示。

**效果图**
![Native主动向Flutter发送消息](https://img-blog.csdnimg.cn/20191231094224696.gif)

     
**android端代码**
```java
public class MainActivity extends FlutterActivity {
    private static final String METHOD_CHANNEL = "com.zhuandian.flutter/android";
    private static final String EVENT_CHANNEL = "com.zhuandian.flutter/android/event"; //事件通道，供原生主动调用flutter端使用
    private static final String METHOD_NATIVE_SEND_MESSAGE_FLUTTER = "nativeSendMessage2Flutter"; //原生主动向flutter发送消息
    private EventChannel.EventSink eventSink;
    private MethodChannel methodChannel;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        GeneratedPluginRegistrant.registerWith(this);

        methodChannel = new MethodChannel(getFlutterView(), METHOD_CHANNEL);
        //接受fltuter端传递过来的方法，并做出响应逻辑处理
        methodChannel.setMethodCallHandler(new MethodChannel.MethodCallHandler() {
            @Override
            public void onMethodCall(MethodCall call, MethodChannel.Result result) {
                System.out.println(call.method);
                 if (call.method.equals(METHOD_NATIVE_SEND_MESSAGE_FLUTTER)) {
                    nativeSendMessage2Flutter();
                }
            }
        });


        new EventChannel(getFlutterView(), EVENT_CHANNEL).setStreamHandler(new EventChannel.StreamHandler() {
            @Override
            public void onListen(Object o, EventChannel.EventSink eventSink) {
                MainActivity.this.eventSink = eventSink;
                eventSink.success("事件通道准备就绪");
                //在此不建议做耗时操作，因为当onListen回调被触发后，在此注册当方法需要执行完毕才算结束回调函数
                //的执行，耗时操作可能会导致界面卡死，这里读者需注意！！
            }

            @Override
            public void onCancel(Object o) {

            }
        });


    }


    /**
     * 原生端向flutter主动发送消息；
     */
    private void nativeSendMessage2Flutter() {
        //主动向flutter发送一次更新后的数据
        eventSink.success("原生端向flutter主动发送消息");
    }
}

```

**Flutter端代码**

```java
class AndroidPlatformPage extends StatefulWidget {
  @override
  State<StatefulWidget> createState() => PageState();
}

class PageState extends State<AndroidPlatformPage> {
  static final String METHOD_CHANNEL = "com.zhuandian.flutter/android";
  static final String EVENT_CHANNEL = "com.zhuandian.flutter/android/event";

  static final String NATIVE_SEND_MESSAGE_TO_FLUTTER =
      "nativeSendMessage2Flutter"; //原生主动向flutter发送消息

  static final MethodChannel _MethodChannel =
      MethodChannel(METHOD_CHANNEL); //平台交互通道
  static final EventChannel _EventChannel =
      EventChannel(EVENT_CHANNEL); //原生平台主动调用flutter端事件通道

  String _fromNativeInfo = "";

  @override
  void initState() {
    super.initState();
    _EventChannel.receiveBroadcastStream().listen(_onEvent, onError: _onErroe);
  }
  

  /**
   * 监听原生传递回来的值（通过eventChannel）
   */
  void _onEvent(Object object) {
    print(object.toString() + "-------------从原生主动传递过来的值");
    setState(() {
      _fromNativeInfo = object.toString();
    });
  }

  void _onErroe(Object object) {
    print(object.toString() + "-------------从原生主动传递过来的值");
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text("平台交互"),
        centerTitle: true,
      ),
      body: Column(
        mainAxisAlignment: MainAxisAlignment.center,
        crossAxisAlignment: CrossAxisAlignment.stretch,
        children: <Widget>[
          Text("从原生平台主动传递回来的值"),
          Text(_fromNativeInfo),
          RaisedButton(
            color: Colors.orangeAccent,
            child: Text("点击调用原生主动向flutter发消息方法"),
            onPressed: () {
              _MethodChannel.invokeMethod(NATIVE_SEND_MESSAGE_TO_FLUTTER);
            },
          ),
        ],
      ),
    );
  }
}
```

通过对比MethodChannle回调的形式传递数据，结合EventChannel从Native端主动向Flutter端发送数据我们可以发现后者更注重的监听，我们在Native端通过注册EventChannel对象后，然后通过**EventChannel.EventSink** 的` void success(Object var1)`方法向Flutter发送数据。

而Flutter端，我们通过在页面初始化端时候绑定监听对象
```java
@override
  void initState() {
    super.initState();
    _EventChannel.receiveBroadcastStream().listen(_onEvent, onError: _onErroe);
  }
  ```

 并且通过`_onEvent`监听从原生主动发送过来值，然后注册相应的处理。
 ```java
   /**
   * 监听原生传递回来的值（通过eventChannel）
   */
  void _onEvent(Object object) {
    print(object.toString() + "-------------从原生主动传递过来的值");
    setState(() {
      _fromNativeInfo = object.toString();
    });
  }
  ```


 限于篇幅问题，我在博文开头效果图里展示端代码就不逐一讲解效果图上的代码示例了完整的代码在本篇文章对应的源代码里都没有注释，读者可自行阅读配套源码结合代码里的注释自行测试。
平台交互部分完整代码：
Native端：[https://github.com/xiedong11/flutter_app/blob/master/android/app/src/main/java/com/zhuandian/flutterapp/MainActivity.java](https://github.com/xiedong11/flutter_app/blob/master/android/app/src/main/java/com/zhuandian/flutterapp/MainActivity.java)
Flutter端：[https://github.com/xiedong11/flutter_app/blob/master/lib/pages/platform/android_platform_page.dart](https://github.com/xiedong11/flutter_app/blob/master/lib/pages/platform/android_platform_page.dart)

