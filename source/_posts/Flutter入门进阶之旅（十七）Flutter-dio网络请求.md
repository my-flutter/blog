---
title: Flutter入门进阶之旅（十七）Flutter dio网络请求
date: 2019-09-29 11:07:57
tags:
---
### 前言

>前面关于Flutter的讲解部分我把关于flutter的基础入门部分带着大家梳理了一遍，那从本篇博客开始，我们开始进入新的领域，也算是给进阶篇开个头，今天我们来一块学习一下Flutter中的网络请求库--->`Dio`，关于Flutter原生带的Http使用起来不论在功能上还是扩展上都不是那么的强大，鉴于此笔者在这里推荐大家在项目中使用`Dio`封装网络请求库。关于Http的使用读者可自行查阅资料学习。


#### 课程目标
- 使用Dio完成最简单的GET、POST请求
- 基于Dio封装网络请求库，并使用自己封装的网络请求工具类完成GET、POST请求
- 了解`InterceptorsWrapper`拦截器
- 利用拦截器给网络请求添加统一参数（如，token，userId等）
- 统一处理响应返回数据（做json转实体或者格式化操作）
- 操作请求统一拦截

##### 1.使用Dio完成简单的GET、POST请求
1.1使用dio get请求一条json数据
```java 
  getRequest() async {
    Response response = await Dio()
        .get('https://www.wanandroid.com/banner/json');
    this.setState(() {
      result= response.toString();
    });
  }
  ```
 
1.2 利用dio post请求注册一个新用户
```java
postRequest() async {
    var path = "https://www.wanandroid.com/user/register";
    var params = {
      "username": "aa112233",
      "password": "123456",
      "repassword": "123456"
    };
    Response response =
        await Dio().post(path, queryParameters: params);
    this.setState(() {
      result= response.toString();
    });
  }
  ```
>由于简单的GET、跟POST请求操作起来比较简单，我就不单独附效果图跟讲解说明了，相信读者从开始看系列博客到现在，读懂上面的代码已经不在话下了，我把上面的get、跟post请求放到一张效果图上代码也贴到一块供大家读阅，另外感谢`玩安卓`提供的开放API作为本篇博客的请求测试用例。

**效果图**
![在这里插入图片描述](https://img-blog.csdnimg.cn/2019092811463753.gif)
**代码**
```java
import 'package:dio/dio.dart';
import 'package:flutter/material.dart';

class NetWorkPage extends StatefulWidget {
  @override
  State<StatefulWidget> createState() => PageState();
}

class PageState extends State<NetWorkPage> {
  var resultJson = "";

  @override
  void initState() {
    super.initState();
  }

  getRequest() async {
    Response response = await Dio()
        .get('https://www.wanandroid.com/banner/json');
    this.setState(() {
      resultJson = response.toString();
    });
  }

  postRequest() async {
    var path = "https://www.wanandroid.com/user/register";
    var params = {
      "username": "aa112233",
      "password": "123456",
      "repassword": "123456"
    };
    Response response =
        await Dio().post(path, queryParameters: params);
    this.setState(() {
      resultJson = response.toString();
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text("Dio网络请求"),
      ),
      body: Column(
        crossAxisAlignment: CrossAxisAlignment.stretch,
        children: <Widget>[
          MaterialButton(
              color: Colors.pinkAccent,
              child: Text("GET 请求"),
              onPressed: () {
                getRequest();
              }),
          MaterialButton(
              color: Colors.blueAccent,
              child: Text("POST 请求"),
              onPressed: () {
                postRequest();
              }),
          Expanded(
              child: Padding(
            padding: EdgeInsets.all(20),
            child: Center(
              child: resultJson.length <= 0
                  ? Text("数据加载中...")
                  : Text(
                      resultJson,
                      style: TextStyle(fontSize: 16),
                    ),
            ),
          ))
        ],
      ),
    );
  }
}
```
##### 2.封装自己的Dio网络请求库
在项目开发过程中随着项目越写越大，代码量也会成倍的增加，`隔离业务抽取共性代码`的思想自然而然的出现在一个严于律己的开发者脑子里，特别是像网络请求这种操作，好的封装不仅会减少冗余代码更能让代码层级变得清晰可读，下面笔者就带着自己的理解对Dio做一个简单封装，笔者自认为才疏学浅，代码中如有写的不好的地方还请各位不吝赐教。

一般我们在处理工具类时都会用到单例的思想，做为一个项目全局的网络请求工具类，我们同样也把`DioUtils`封装成单例模式
```java
  static DioUtils getInstance() {
    if (_instance == null) {
      _instance = new DioUtils();
    }
    return _instance;
  }
  ```
  对于请求参数的初始化跟一些网络配置，Dio为开发者提供了BaseOptions、Options、RequestOptions可选Options配置，三者的优先级关系依次递增
  >试想这样一个场景：一般我们在对网络请求工具类设置参数，理论上是全局不能修改的，如果我们的业务需求必须让我们修改配置参数，比如基于不同的接口服务需要设置不同请求中的`header`内容，这个时候利用options的优先级我们就可以很轻松的处理这个问题，稍后我会在代码里具体讲解
  
  先来看下BaseOptions给我们提供的可供配置的参数：
  ```java
  BaseOptions({
    String method,
    int connectTimeout,
    int receiveTimeout,
    Iterable<Cookie> cookies,
    this.baseUrl,
    this.queryParameters,
    Map<String, dynamic> extra,
    Map<String, dynamic> headers,
    ResponseType responseType = ResponseType.json,
    ContentType contentType,
    ValidateStatus validateStatus,
    bool receiveDataWhenStatusError = true,
    bool followRedirects = true,
    int maxRedirects = 5,
   RequestEncoder requestEncoder,
    ResponseDecoder responseDecoder,
  })
  ```
 上面构造方法中的属性读者基本都能见名知意做到自解释，我就不单独解释了，贴上一段我在代码里的配置：
 ```java
     //请求参数配置
    _baseOptions = new BaseOptions(
      baseUrl: BASE_URL,
      //请求服务地址
      connectTimeout: 5000,
      //响应时间
      receiveTimeout: 5000,
      headers: {
        //需要配置请求的header可在此处配置
      },
      //请求的Content-Type，默认值是[ContentType.json]. 也可以用ContentType.parse("application/x-www-form-urlencoded")
      contentType: ContentType.json,
      //表示期望以那种格式(方式)接受响应数据。接受三种类型 `json`, `stream`, `plain`, `bytes`. 默认值是 `json`,
      responseType: ResponseType.json,
    );
    
   ```
然后把`_baseOptions`通过参数的形式传入Dio实例中完成配置的初始化

```java
     //创建dio实例
    _dio = new Dio(_baseOptions);
```
接下来我把我们开篇用DIO做的简单GET、POST方法用我们新写的工具类封装完成后重新做请求，让我们来一起感受下封装带来的便利。

**封装GET请求**
```java
  /**
   * get请求
   */

  get(url, {data, options, cancleToken}) async {
    print('get request path ------${url}-------请求参数${data}');
    Response response;
    try {
      response = await _dio.get(url,
          queryParameters: data, options: options, cancelToken: cancleToken);
      print('get success ---${response.data}');
    } on DioError catch (e) {
      print('请求失败---错误类型${e.type}');
    }

    return response.data;
  }
```
利用我们封装好的get请求方法，开篇的get请求只需改为：
```java
  getRequest() async {
    String result= await DioUtils().get('/banner/json');
    this.setState(() {
      resultJson = result;
    });
  }

```
或者get的时候需要携带参数，例如如下这样一个请求

> https://www.wanandroid.com/article/list/0/json?cid=60
    方法：GET
   参数：cid 分类的id，上述二级目录的id
  
  对上面的url进行分析
  > `baseUrl`：https://www.wanandroid.com 我们已经在工具类中初始化过了，所以不用设置
  `path`: /article/list/0/json   我们在get请求中需要传入的url
  `data`：cid=60    我们封装好的get请求中对应的data数据

上述请求为：
```java
  getRequest() async {
    var data = {
      "cid": 60
    };
    String result = await DioUtils().get('/article/list/0/json', data: data);
    this.setState(() {
      resultJson = result;
    });
  }

```
在刚刚讲BaseOptions时，我们提到还有Options、RequestOptions可供配置，我们提到可以利用Options的优先级重新覆盖掉原先在工具类里设置好的网络配置，比如修改提前在header中设置好的请求内容。
 还是上述代码，我先修改_baseOptions中的header的配置如下：

 我们利用Options修改BaseUrl后的请求url
 ```java
  _baseOptions = new BaseOptions(
      baseUrl: BASE_URL,
      connectTimeout: 5000,
      receiveTimeout: 5000,
      headers: {
        //预设好的header信息
        "testHeader":"bb"
      },
      contentType: ContentType.json,
      responseType: ResponseType.json,
    );
```

还是上述请求，现在我们重新走一遍上述的get请求，看下log控制台打印的`header信息`
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190929104333187.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly94aWVkb25nLmJsb2cuY3Nkbi5uZXQ=,size_16,color_FFFFFF,t_70)
 然后我修改原先的get请求，给options添加RequestOptions，修改里面的header值如代码所示：
 ```java
   getRequest() async {
    var data = {"cid": 60};
    RequestOptions requestOptions = new RequestOptions(headers: {"testHeader":"aaaa"});
    String result = await DioUtils()
        .get('/article/list/0/json', data: data,options: requestOptions);
    this.setState(() {
      resultJson = result;
    });
  }
```
运行代码，再次执行get请求，控制台的header信息已经被我们修改过了：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190929104637792.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly94aWVkb25nLmJsb2cuY3Nkbi5uZXQ=,size_16,color_FFFFFF,t_70)

 笔者只不过是借用这个例子让读者了解怎么去修改工具类里配置好的参数，通过RequestOptions不仅仅是可以修改`headerl`，基本你能在工具类设置的东西都能做修改，感兴趣的读者可以自行阅读源码测试，限于篇幅问题我这里就不展开讲解了。

**POST请求封装**
POST请求封装跟GET类似，这里我就不过多分析了，我直接贴源码了读者可结合代码自行对比差异
```java
  postRequest() async {
    var params = {
      "username": "aa112233",
      "password": "123456",
      "repassword": "123456"
    };
    String result = await DioUtils().post("/user/register",data: params);
    this.setState(() {
      resultJson = result;
    });
  }
  ```

##### 3.利用拦截器给网络请求添加统一参数
在DIo中我们可以通过`Interceptors`为我们的网络请求添加拦截器

```java
_dio.interceptors.add()
```
我们通过`_dio.interceptors.add()`方法可以根据不同业务为我们的工具类添加不同的拦截器，比如在网络请求开始之前，我们给每个请求都添加统一的token，或者userId，或者我们可以对请求返回的数据做统一json格式化处理，对错误响应统一处理，这些业务场景都可以通过`interceptors`来完成，比如下面我的配置：
```java
    //可根据项目需要选择性的添加请求拦截器
    _dio.interceptors.add(
      InterceptorsWrapper(onRequest: (RequestOptions requestions) async {
        //此处可网络请求之前做相关配置，比如会所有请求添加token，或者userId
        requestions.queryParameters["token"] = "testtoken123443423";
        requestions.queryParameters["userId"] = "123456";
        return requestions;
      }, onResponse: (Response response) {
        //此处拦截工作在数据返回之后，可在此对dio请求的数据做二次封装或者转实体类等相关操作
        return response;
      }, onError: (DioError error) {
        //处理错误请求
        return error;
      }),
    );
  }
```

现在我们通过工具类进行的所有的网络请求的url后面都会被加入token=“testtoken123443423”&userId=123456这样两个参数，还是上面的GET请求，下面我们通过代码跟控制台的输出内容看下通过拦截器添加完通用参数的请求，log控制台打印的请求参数信息
```java
  getRequest() async {
    var data = {"cid": 60};
    String result = await DioUtils()
        .get('/article/list/0/json', data: data);
    this.setState(() {
      resultJson = result;
    });
  }
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190929104925793.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly94aWVkb25nLmJsb2cuY3Nkbi5uZXQ=,size_16,color_FFFFFF,t_70)
上面的GET请求，我们在参数里只添加了`cid = 60`的参数，但是通过控制台我们已经清楚的看到token跟userId已经通过拦截器的被我们添加到`queryParameters`里面了。

限于篇幅问题，封装好的DioUtils我就不贴出来了，附上[源码的github地址](https://github.com/xiedong11/flutter_app/blob/master/lib/pages/network/utils/dio_utils.dart)，供读者参考，最后感谢玩安卓开放API平台提供的测试API用于本篇博客中的测试用例。