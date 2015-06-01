---
layout:     post
title:      Unity3D项目接入移动基地（和游戏）SDK
category: blog
description: 有关移动基地（咪咕）SDK接入的流程
---

前段时间公司的游戏需要接入移动基地的SDK，虽然我对于安卓开发与SDK接入基本没什么经验，但是自告奋勇的接下了这个工作任务。没有想到，完成这个看起来简单的工作任务用了两个多星期。不过通过这次实践，我学到了一些安卓开发的基本知识，对于Unity3D与Java也有了更多的了解。
##SDK接入流程##
**适用版本V2.2.3**
###拷贝文件###
+ 将SDK文件夹中的assets，libs，res，三个文件夹拷入Assets/Plugins/Android/。
+ 将SDK文件夹中的CMBillingForUnity.jar拷贝到Assets/Plugins/Android/libs.
+ 将SDK文件夹中的CMBillingAndroid.cs拷贝到/Plugins/Android/.
+ 将SDK文件夹中的CMBillingAndroidDemo.cs拷贝到/Plugins/Android/.

###编辑xml文件###
编辑Plugins\Android\res\values\g_strings.xml，将游戏主入口(通常为com.unity3d,player.UnityPlayerActivity)填入: 

```xml
<string name="g_class_name">om.unity3d,player.UnityPlayerActivity</string>
```

###生成自定义Application所需要的jar文件###
####eclipse####
建一个新的project：New->Other...->Android->Android Application Project->Next。此时来到New Android Application对话框，填入的包名（Package Name）必须与Unity3D项目的Bundle Identifier一致，在这里以com.test.sdk为例。Application Name和Project Name可以随意。然后点击Next->Finish完成project的创建。  
在eclipse的Project Explore中，找到src/com.test.sdk/，点击右键->New->Class新建一个java文件,文件名必须是CmgameApplication。在文件中输入以下代码：

```java
    package com.sweatychair.gotoschool;

    import android.text.sdk;
    public class CmgameApplication extends Application {
        public void onCreate() {
            System.loadLibrary("megjb");
        }
    }
```

保存。  
*这段代码扩展了App中的Application类, 在Application初始化的时候，载入library。*  
删除其他在src/com.test.sdk中的java文件。(通常会有MainActivity.java，删除。)  
在项目名上点击右键->Export->Java->JAR file。在JAR File Specification对话框中，取消所有选项，勾选bin。在对话框下方，勾选三个选项：Export all output folders for checked projects, Export Java source files and resources, Export refactorings for checked projects. 选择输入路径。勾选Compress the contents of the JAR file，点击Finish完成。  
将jar拷贝到Assets/Plugins/Android/bin

####Android Studio
[参考文章](http://www.thegamecontriver.com/2015/04/android-plugin-unity-android-studio.html)

####有可能在build时出现的错误
如果在Unity3D生成APK的时候出现错误：

    Conversion to Dalvik format failed: Unable to execute dex: Multiple dex files define ...
查看console中的信息。通常是因为有了重复的Jar文件。删除重复的Jar。

如果错误为：

    Unable to convert classes into dex format.
console信息例子：

    stderr[ UNEXPECTED TOP-LEVEL EXCEPTION: java.lang.IllegalArgumentException: a*lready added: Landroid/support/v4/hardware/display/DisplayManagerCompat;*
通常是有了重复的.class文件。在上面生成的jar文件中，可以删除掉不需要的.class文件。方法如下：  
解压缩CmgameApplication.jar, 在文件夹CmgameApplication/com/test/sdk/中，除去CmgameApplication之外，应该还有其他的.class文件。删除其他文件。  
在terminal中，将文件夹重新打包为Jar文件：

```bash
    jar cf CmgameApplication.jar ./com/
```

如果错误为：

    Error building Player: CommandInvokationFailure: Unable to convert classes into dex format. See the Console for details.
     C:\Program Files\Java\jdk1.7.0_55\bin\java.exe -Xmx1024M -Dcom.android.sdkmanager.toolsdir="C:/Program Files (x86)/Android/android-sdk\tools" -Dfile.encoding=UTF8 -jar "C:/Program Files (x86)/Unity/Editor/Data/BuildTargetTools/AndroidPlayer\sdktools.jar" -
     
     stderr[
     
     UNEXPECTED TOP-LEVEL EXCEPTION:
     com.android.dx.cf.iface.ParseException: class name (com/companyname/unity/ad/R$attr) does not match path (bin/com/companyname/unity/ad/R$attr.class)
         at com.android.dx.cf.direct.DirectClassFile.parse0(DirectClassFile.java:520)
有可能是jar包中有bin文件夹与com文件夹两个。或者是在不同的jar文件中有重复的.class文件。

###编辑AndroidManifest.xml
􏰔􏰔将SDK\AndroidManifest.xml.activity.txt􏱭􏰛􏰌􏰟􏱮􏱯􏰠与SDK\AndroidManifest.xml.permision.txt内容拷入AndroidManifest.xml􏱭。注意不要有重复内容。  
删除原来的Activity中的入口定义：

```xml
    <intent-filter>
    <action android:name="android.intent.action.MAIN" />
    <category android:name="android.intent.category.LAUNCHER" />
    </intent-filter>
```

###完成
有关SDK功能的实现，参考SDK中的CmBillingAndroidDemo.cs更改代码。

###可能的错误
+ 移动基地的logo显示正常，进入游戏时闪退。可能是游戏的入口没有正确设置。确认游戏主入口在Plugins\Android\res\values\g_strings.xml中正确设置。并且确认AndroidManifest.xml里面设置了游戏入口的Activity.
+ 游戏一打开就闪退。可能是SDK/libs/armeabi/libmegjb.so在打包过程中丢失。利用ApkTool解压生成的apk包，查看文件夹armeabi-v7a中是否含有libmegjb.so。如果没有，在Unity3D中将libmegjb.so拷贝到Assets/Plugins/Andorid/armeabi-v7a。如果没有此文件，新建此文件夹。   
*注意，有关libmegjb.so丢失的解决方法是给予我的尝试，没有理论依据。*