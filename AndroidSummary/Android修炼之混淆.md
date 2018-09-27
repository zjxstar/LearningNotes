# Android修炼之混淆

## 自嘲时刻

作为Java和Android开发者，大家应该都对**混淆**很熟悉了。网上也有各路大神提供的混淆模板，基本上直接拿来用就好。但我还是想捋一捋，因为工作中被**混淆**这家伙“玩弄”了好几次，必须把它记在小本本上。

## 介绍
### 基本概念
**混淆**，字面上来说就是把项目中的包名、类名、方法名和变量名等进行更改，用以迷惑别人。但混淆其实包含了代码压缩、优化、校验等过程，把混淆称作`ProGuard`更合适。

### ProGuard
> PS：[ProGuard官网](https://stuff.mit.edu/afs/sipb/project/android/sdk/android-sdk-linux/tools/proguard/docs/index.html#manual/introduction.html)

ProGuard就是Java对Class文件进行“混淆”的工具。直接贴图吧：

![ProGuard过程](https://user-gold-cdn.xitu.io/2018/9/15/165db322ac5f7091?w=948&h=234&f=png&s=18694)
（看到这个图，大家有没有想到什么？——设计模式中的**责任链模式**。）
1. shrink（压缩）：ProGuard会递归地确定哪些类和类成员被使用，而其他的则被丢弃。
2. optimize（优化）：ProGuard会进一步分析和优化方法。比如一些无用的参数会被丢弃，一些方法会做内联。
3. obfuscate（混淆）：这个过程就是进行重命名了，把原来包含注释意义的类名、方法名等进行无意义重命名。
4. preverify（预校验）：这个步骤是将预校验信息添加到类中。

这四个步骤其实都是可选的。当然，Android中一般情况下我们都会保留前三个步骤，而忽略`preverify`过程，这样可以加快混淆速度。

### Android中的ProGuard
Android中默认集成了ProGuard工具，在sdk目录的/tools/proguard中。混淆开启方法：

```java
android {
    ...
    // AS自动生成
    buildTypes {
        release {
            // 混淆开关
            minifyEnabled true
            // 移除无用的resource文件
            shrinkResources true
            // proguard-android.txt表示默认的混淆规则
            // proguard-rules.pro表示自定义的混淆规则（文件名和后缀可以修改）
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
}
```
默认的`proguard-android.txt`文件在sdk目录/tools/proguard中。该目录下还有个`proguard-android-optimize.txt`文件。而我们自定义的`proguard-rules.pro`文件中有部分基础混淆规则就是来自`proguard-android-optimize.txt`。

### 混淆语法
混淆的语法都可以在上述的`ProGuard官网`中找到，这里只介绍一些常用的语法规则。

1、保留类和类成员<br>

|              保留              |   反之被删除或重命名    |        防止被重命名         |
| :----------------------------: | :---------------------: | :-------------------------: |
|           类和类成员           |          -keep          |         -keepnames          |
|            仅类成员            |    -keepclassmembers    |    -keepclassmembernames    |
| 如果拥有某成员，保留类和类成员 | -keepclasseswithmembers | -keepclasseswithmembernames |

2、类成员中的一些符号<br>

|    符号    |       作用       |
| :--------: | :--------------: |
|  \<init>   |  匹配所有构造器  |
| \<fields>  |    匹配所有域    |
| \<methods> |   匹配所有方法   |
|     *      | 匹配所有域和方法 |

3、一些常用通配符<br>

| 通配符 |                  作用                   |
| :----: | :-------------------------------------: |
|   *    |  匹配任意长度字符，但不含包名分隔符(.)  |
|   **   | 匹配任意长度字符，并且包含包名分隔符(.) |
|  ***   |            匹配任意参数类型             |
|  ...   |       匹配任意长度的任意类型参数        |
|   %    |            匹配任何原始类型             |
|   ?    |        匹配类名中的任何单个字符         |

其他的一些语法基本上属于无变化的那种，在混淆规则中基本都固定使用了，有需要的话可以自行在官方查询。

## 自定义混淆模板
嗯...这模板基本都是大同小异的，这里就直接贴出常用的混淆规则了。
```java
# 优化算法，一般不用修改，来自Google
-optimizations !code/simplification/arithmetic,!code/simplification/cast,!field/*,!class/merging/*
# 代码混淆压缩比，在0~7之间，默认为5，一般不做修改
-optimizationpasses 5
# 混合时不使用大小写混合，混合后的类名为小写
-dontusemixedcaseclassnames
# 不去忽略非公共库的类
-dontskipnonpubliclibraryclasses
# 优化时允许访问并修改有修饰符的类和类的成员 
-allowaccessmodification
# 项目混淆后产生映射文件
-verbose
# 不做预校验
-dontpreverify

# 保留Annotation不混淆
-keepattributes *Annotation*,InnerClasses
# 避免混淆泛型
-keepattributes Signature
# 抛出异常时保留代码行号
-keepattributes SourceFile,LineNumberTable

# 保留四大组件，自定义的Application等
-keep public class * extends android.app.Activity
-keep public class * extends android.app.Appliction
-keep public class * extends android.app.Service
-keep public class * extends android.content.BroadcastReceiver
-keep public class * extends android.content.ContentProvider
-keep public class * extends android.app.backup.BackupAgentHelper
-keep public class * extends android.preference.Preference
-keep public class * extends android.view.View
-keep public class com.google.vending.licensing.ILicensingService
-keep public class com.android.vending.licensing.ILicensingService

# 保留native方法
-keepclasseswithmembernames class * {
    native <methods>;
}

# 保留在Activity中的方法参数是view的方法，
# 这样以来我们在layout中写的onClick就不会被影响
-keepclassmembers class * extends android.app.Activity{
    public void *(android.view.View);
}

# 保留自定义View
-keepclassmembers public class * extends android.view.View {
   void set*(***);
   *** get*();
   public <init>(android.content.Context);
   public <init>(android.content.Context, android.util.AttributeSet);
   public <init>(android.content.Context, android.util.AttributeSet, int);
}

# 保留枚举
-keepclassmembers enum * {
    public static **[] values();
    public static ** valueOf(java.lang.String);
}

# 保留Parcelable序列化对象
-keepclassmembers class * implements android.os.Parcelable {
  public static final android.os.Parcelable$Creator CREATOR;
}

# 保留R文件中的成员
-keepclassmembers class **.R$* {
    public static <fields>;
}

# 保留Serializable序列化的类不被混淆
-keepnames class * implements java.io.Serializable
-keepclassmembers class * implements java.io.Serializable {
    static final long serialVersionUID;
    private static final java.io.ObjectStreamField[] serialPersistentFields;
    !static !transient <fields>;
    !private <fields>;
    !private <methods>;
    private void writeObject(java.io.ObjectOutputStream);
    private void readObject(java.io.ObjectInputStream);
    java.lang.Object writeReplace();
    java.lang.Object readResolve();
}

# -------------------------------------------------------------------------------
# webView处理，项目中没有使用到webView忽略即可
-keepclassmembers class fqcn.of.javascript.interface.for.webview {
    public *;
}
-keepclassmembers class * extends android.webkit.webViewClient {
    public void *(android.webkit.WebView, java.lang.String, android.graphics.Bitmap);
    public boolean *(android.webkit.WebView, java.lang.String);
}
-keepclassmembers class * extends android.webkit.webViewClient {
    public void *(android.webkit.webView, jav.lang.String);
}

# --------------------------保留JS接口-------------------------------------------

# --------------------------保留反射类-------------------------------------------

# --------------------------保留实体类-------------------------------------------

# --------------------------第三方库的混淆规则-----------------------------------

```
以上都是一些常用的混淆规则，其中部分规则其实都在Android默认混淆文件中声明过了。所以具体的自定义混淆规则还是得根据项目进行变化。

### 混淆产物
混淆后一般都有下面几个文件：

1. dump.txt：描述APK文件中所有类的内部结构；
2. mapping.txt：提供混淆前后类、方法、类成员等的对照表
3. seeds.txt：列出没有被混淆的类和成员
4. usage.txt：列出被移除的代码

遇到混淆问题时，我通常是通过查看mapping.txt文件来分析原因的。<br>
如果在进行Crash追踪中遇到了困难，可以使用sdk目录下/tools/proguard/bin中的`proguardgui.bat`可视化工具进行混淆定位。示例如图：

![追踪混淆Crash栈](https://user-gold-cdn.xitu.io/2018/9/15/165dbe5064d9f37e?w=815&h=507&f=png&s=96136)

## 混淆引发的“Xue案”
**场景一** <br>
**问题描述**：业务方集成了我们的一个SDK，SDK的一个页面中会显示Banner图，而Banner图所用的是SDK中自定义的ViewPager组件，结果业务方那边无法显示Banner图。<br>
**原因分析**：通过反编译apk，比对mapping.txt文件，发现Banner图的自定义Adapter中关键方法都被清除了。查看混淆文件，发现只keep了support v4的ViewPager，并没有keep住PagerAdapter。而SDK中又是通过compileOnly（provided）方式依赖v4包的，导致自定义的Adapter在工程编译的时候找不到被引用的关系，然后就被混淆给优化掉了。<br>
**经验**：针对compileOnly（provided）依赖的库，一定要注意相关类的混淆，特别是SDK开发者，因为混淆导致的问题一般较难定位。

**场景二** <br>
**问题描述**：业务方又集成了我们的一个SDK（没错，是“又”），SDK有一个换肤功能，业务方可以通过构造特定格式的资源文件来实现皮肤替换。但在业务通过公司编译工具编译后的apk中无法实现换肤功能。<br>
**原因分析**：机智的我很快想到了混淆问题，于是让业务方加上保留所有R文件的规则，结果还是失败。于是开始了长时间的打log看log过程，还是确定问题出在资源名的混淆上。最后发现业务方虽然在本地编译环境中keep了资源名，但在公司的编译环境中还是对资源进行了混淆，导致换肤失败。<br>
**经验**：对于和资源相关的功能，一定要注意R文件的混淆。不要轻易相信业务，因为他们经常有你预料不到的操作（手动滑稽...）。

工作中遇到过很多次混淆相关的问题，这里就不多说了，只要大家细心点就好了。

## 总结
1. 混淆其实不算是什么新鲜的话题，但对于Java和Android开发者来说是必不可少的，还是要对其有敬畏之心（严肃脸...）。
2. 混淆的好处不言而喻增加了反编译的难度，优化了代码。但仅仅是增加难度而已，对于有心之人还是能从反编译中找到蛛丝马迹。所以，对于敏感代码和数据使用更安全的保护措施是非常必要的，比如加固（硬广一波：[360加固保](http://jiagu.360.cn/#/global/index)）。

## 参考资料
1. [Android安全攻防战，反编译与混淆技术完全解析（下）](https://blog.csdn.net/guolin_blog/article/details/50451259)
2. [Android Studio混淆模板及常用第三方混淆](https://www.jianshu.com/p/f9438603e096)
3. [写给 Android 开发者的混淆使用手册](https://www.diycode.cc/topics/380)
4. [ProGuard](https://stuff.mit.edu/afs/sipb/project/android/sdk/android-sdk-linux/tools/proguard/docs/index.html#manual/introduction.html)
