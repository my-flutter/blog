---
title: Flutter入门进阶之旅（十八）Flutter项目打包成aar集成到原生Android项目
date: 2019-10-31 13:30:22
tags:
---
### 前言
>在前面的章节学习中我们已经掌握了从最基本的`hello flutter`到各种基本Widget、各种布局的使用再到多页面切换路由的使用还有各种炫酷的提示跟dialog，还有关于网络请求库Dio的使用，至此我们完全可以使用flutter去开发一款独立可运行的app了，但是基于现阶段flutter技术栈还不是太成熟，纯flutter项目上线风险还是比较大，所以跨平台的混合开发模式自然还是现阶段尝试flutter的主流方式，今天的分享我就跟大家一块把我们写好的flutter项目打包成aar文件嵌入到现有的Android项目中去。

#### 课程目标
- 掌握flutter打包成aar的整体流程
- 利用fat-aar把flutter项目中的三方依赖打入aar资源包中


### 项目准备：
>flutter端项目我采用的是本专栏的实例代码项目：[https://github.com/xiedong11/flutter_app](https://github.com/xiedong11/flutter_app)，Android端原生项目为新建的一个hello world项目，flutter端的相关配置我会上传到github仓库中，原生Android项目比较简单，我只在本博客中贴出部分关键代码

### 1.flutter项目打包成aar
**flutter端项目工程目录**
![Flutter端项目工程目录](https://img-blog.csdnimg.cn/20191028172542568.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly94aWVkb25nLmJsb2cuY3Nkbi5uZXQ=,size_16,color_FFFFFF,t_70)
上面截图的flutter工程需要经过我们的改造才能作为一个aar的形式存在，要被打包成aar的flutter端的项目是作为一个独立的module运行在宿主app（原生Android）中，所以我们需要修改两个地方，让我们打包出来的产物变成aar而不是独立运行的apk。

##### 1.1修改Android下的 build.gradle

```java

// 1. 生成aar产物，需要把`application`改为`library`
apply plugin: 'com.android.library'
apply from: "$flutterRoot/packages/flutter_tools/gradle/flutter.gradle"

android {
    compileSdkVersion 28

    lintOptions {
        disable 'InvalidPackage'
    }

    defaultConfig {
        // 2. flutter 作为寄存于其他app中的产物，所以不应该存在applicationId，所以注释掉该行.
        //applicationId "com.zhuandian.flutterapp"
        minSdkVersion 16
        targetSdkVersion 28
        versionCode flutterVersionCode.toInteger()
        versionName flutterVersionName
        testInstrumentationRunner "androidx.test.runner.AndroidJUnitRunner"
    }

    buildTypes {
        release {
            // TODO: Add your own signing config for the release build.
            // Signing with the debug keys for now, so `flutter run --release` works.
            signingConfig signingConfigs.debug
        }
    }
}
```

##### 1. 2  修改Androidmanifest.xml文件
```java
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.zhuandian.flutterapp">

    <uses-permission android:name="android.permission.INTERNET" />
   
    <!--1.项目作为子项目寄存于原生app中，不需要icon、label等属性，这里直接省去各种配置即可-->
    <application>
        <!--android:name="io.flutter.app.FlutterApplication"-->
        <!--android:icon="@mipmap/ic_launcher"-->
        <!--android:label="flutter_app">-->
        <activity
            android:name=".MainActivity"
            android:configChanges="orientation|keyboardHidden|keyboard|screenSize|locale|layoutDirection|fontScale|screenLayout|density"
            android:hardwareAccelerated="true"
            android:launchMode="singleTop"
            android:theme="@style/LaunchTheme"
            android:windowSoftInputMode="adjustResize">


            <!--2.项目作为子项目寄存于原生app中，入口acitvity不需要配置LAUNCHER，不然应用集成到宿主app中，启动会在桌面上生成两个应用图标-->
           
            <!--<meta-data-->
                <!--android:name="io.flutter.app.android.SplashScreenUntilFirstFrame"-->
                <!--android:value="true" />-->

            <!--<intent-filter>-->
                <!--<action android:name="android.intent.action.MAIN" />-->

                <!--<category android:name="android.intent.category.LAUNCHER" />-->
            <!--</intent-filter>-->
        </activity>
        <activity android:name=".SecondActivity"></activity>
    </application>

</manifest>
```

需要修改的地方，我都在源码里留注释了，这里就不展开赘述了，下面我们来通过gradle获得编译好的`aar`产物

因为在build.gradle 中，我们把`apply plugin: 'com.android.application'`修改成了，`apply plugin: 'com.android.library'`
所以，现在通过Terminal中我执行`gradlew assembleRelease`编译出的产物会有原来的apk变成aar文件，文件输出目录为项目根目录下的`/bulid/app/outputs/aar`
如下图所示：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191028174931515.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly94aWVkb25nLmJsb2cuY3Nkbi5uZXQ=,size_16,color_FFFFFF,t_70)

##### 1.3 Android端项目配置接入aar依赖
1.3.1 新建原生Android项目，我上述打包产出的aar文件作为依赖放入libs文件夹
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191028175430459.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly94aWVkb25nLmJsb2cuY3Nkbi5uZXQ=,size_16,color_FFFFFF,t_70)

1.3.2 修改`dependencies`节点下的`fileTree`依赖配置，支持引入aar依赖支持
```
implementation fileTree(dir: 'libs', include: ['*.jar','*.aar'])
```
```java
dependencies {
    implementation fileTree(dir: 'libs', include: ['*.jar','*.aar'])
    ...
}
```

1.3.3 在原生Android项目中写一个简单的按钮测试flutter项目

```java
class MainActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        //初始化flutter运行环境
        FlutterMain.startInitialization(this)
        tv_test_aar.setOnClickListener {
            startActivity(Intent(this, com.zhuandian.flutterapp.MainActivity::class.java))
        }
    }
}
```
>至此，把flutter项目打包成aar导入到原生Android项目中的所有工程配置已经结束了，读者可以测试上上述整个过程；但有时候我们的flutter项目并不是单纯的flutter官方代码，开发过程中少不了引入一些三方依赖，像上节课我们讲到的Dio网络请求库，或者是通过`pubspec.yaml`引入的其他开源工具类，这种情况下，通过上边的配置方式，你会发现第三方的依赖代码是打不进aar包里的，下面我们就讲解一下借助`fat-aar`的形式把三方依赖代码打入aar包中去

### 2.flutter项目中存在三方依赖
通过上边的配置，我们只能把纯flutter项目打包成aar文件，换句话说，如果我们的flutter项目存在三方依赖是不能正常打包进flutter产物里的，这个时候我们就需要通过在Android原生项目中引入`fat-aar`配置，确保把flutter项目中的三方依赖正常打包进去flutter产物中去。

2.1修改项目工程目录的build.gradle文件，引入fat-aar支持
```java
buildscript {
    repositories {
        google()
        jcenter()
    }

    dependencies {
        classpath 'com.android.tools.build:gradle:3.4.1'
        //引入fat-aar支持
        classpath 'com.kezong:fat-aar:1.1.7'
    }
}
```


2.2 在上边第一部分配置app文件下build.gradle基础上，增加fat-aar相关配置，这里为了切换aar library运行环境，我引入isAarLibrary标志位作为切换环境开关，方便工程配置，具体代码如下：
```java
//是否作为依赖
boolean isAarLibrary = true


if (isAarLibrary) {
    apply plugin: 'com.android.library'
} else {
    apply plugin: 'com.android.application'
}

apply from: "$flutterRoot/packages/flutter_tools/gradle/flutter.gradle"

if (isAarLibrary) {
    //引入fat-aar
    apply plugin: 'com.kezong.fat-aar'
}

android {
    compileSdkVersion 28

    lintOptions {
        disable 'InvalidPackage'
    }

    defaultConfig {
        // TODO: Specify your own unique Application ID (https://developer.android.com/studio/build/application-id.html).
        if (!isAarLibrary) {
            applicationId "com.zhuandian.flutterapp"
        }

        minSdkVersion 16
        targetSdkVersion 28
        versionCode flutterVersionCode.toInteger()
        versionName flutterVersionName
        testInstrumentationRunner "androidx.test.runner.AndroidJUnitRunner"
    }

    buildTypes {
        release {
            // TODO: Add your own signing config for the release build.
            // Signing with the debug keys for now, so `flutter run --release` works.
            signingConfig signingConfigs.debug
        }
    }
}

flutter {
    source '../..'
}

dependencies {
    implementation 'androidx.constraintlayout:constraintlayout:1.1.3'
    testImplementation 'junit:junit:4.12'
    androidTestImplementation 'androidx.test:runner:1.1.0'
    androidTestImplementation 'androidx.test.espresso:espresso-core:3.1.0'
    implementation 'androidx.appcompat:appcompat:1.0.0'

    if (isAarLibrary) {
        //TODO 添加fat-aar处理flutter打包成aar中三方依赖
        def flutterProjectRoot = rootProject.projectDir.parentFile.toPath()
        def plugins = new Properties()
        def pluginsFile = new File(flutterProjectRoot.toFile(), '.flutter-plugins')
        if (pluginsFile.exists()) {
            pluginsFile.withReader('UTF-8') { reader -> plugins.load(reader) }
        }
        plugins.each { name, _ ->
            println name
            embed project(path: ":$name", configuration: 'default')
        }
    }

}


```

2.3 AndroidManifest.xml清单文件中不需要增加额外配置，原生Android端唤起flutter项目aar包的方式也不需要修改，整个引入fat-aar的过程只是为了确保能把flutter项目中的三方依赖代码打入到flutter产物中去，所以关于flutter打包成aar的操作跟第一步保持一致就行，`第二步的配置，只是为了确保flutter项目中的三方依赖能正常打包进flutter产物中去`

效果图是我把本专栏的相关代码作为一个aar集成到一个新建的原生Android项目中，效果图如下：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191031084741783.gif)

项目的完整代码配置在[https://github.com/xiedong11/flutter_app](https://github.com/xiedong11/flutter_app) 中，读者可以参考具体配置细节，笔者在写本篇博文打包aar时的flutter 环境stable版本为 `flutter v1.9.1`，读者尽量用官方稳定版的代码做测试。