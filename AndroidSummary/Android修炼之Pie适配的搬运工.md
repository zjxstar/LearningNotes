# Android修炼之Pie 适配的搬运工

## 自嘲时刻

Android P正式版（以下称为Pie）已经正式上线了，各大厂商已经开始了系统升级工作，咱做上层开发的也得跟上节奏。当然了，新版本所有的行为更改内容都可以在官网上找到，对于其中如何绕开**非SDK接口限制**的问题，也有各路大神给出了解决方案。所以，我只能当搬运工了（偷笑ing）。下面我会结合现有的开发经验，聊聊Pie中对我们开发影响较大的一些更新，重头戏还是**适配齐刘海**和**非SDK接口限制**。

## 行为变更
这里只会列出部分行为变更（仅代表个人见解），感兴趣的同学直接去官网查看。
> [Pie官方文档](https://developer.android.com/about/versions/pie/)

### 通知渠道设置更新
**更新内容**：<br>
* 屏蔽渠道组：现在，用户可以针对某个应用在通知设置中屏蔽整个渠道组。 您可以使用 isBlocked() 函数确定何时屏蔽一个渠道组，从而不会向该组中的渠道发送任何通知。
* 全新的广播 Intent 类型：现在，当通知渠道和渠道组的屏蔽状态发生变更时，Android 系统将发送广播 Intent。 拥有已屏蔽的渠道或渠道组的应用可以侦听这些 Intent 并做出相应的回应。

**影响**：APP的推送功能可能需要适配，Pie中可以知道渠道组的状态了。

### ImageDecoder & AnimatedImageDrawable
**更新内容**:<br>
* ImageDecoder 类，可提供现代化的图像解码方法。 使用该类取代 BitmapFactory 和 BitmapFactory.Options API。ImageDecoder 让您可通过字节缓冲区、文件或 URI 来创建 Drawable 或 Bitmap。
* AnimatedImageDrawable 类，用于绘制和显示 GIF 和 WebP 动画图像。 AnimatedImageDrawable 的工作方式与 AnimatedVectorDrawable 的相似之处在于，都是渲染线程驱动 AnimatedImageDrawable 的动画。

**影响**：对APP级别的影响应该较小，毕竟大部分都是使用第三方库来实现图片资源的加载。可能对SDK影响较大，SDK里基本都是自己写Image压缩解码逻辑，所以可以考虑使用。

### 统一生物识别身份验证对话框
**更新内容**:<br>
在 Android 9 中，系统代表您的应用提供生物识别身份验证对话框。 该功能可创建标准化的对话框外观、风格和位置，让用户更加确信，他们在使用可信的生物识别凭据检查程序进行身份验证。

**影响**：接入了生物识别功能的（如：指纹）APP就有统一的提示对话框了，挺好的。就是不知道标准的风格是否适合各种应用。

### 屏幕旋转
**更新内容**:<br>
* 为避免无意的旋转，我们新增了一种模式，哪怕设备位置发生变化，也会固定在当前屏幕方向上。 必要时用户可以通过按系统栏上的一个按钮手动触发旋转。
* 从 Android 9 开始，对纵向旋转模式做出了重大变更。纵向模式已重命名为旋转锁定，它会在自动屏幕旋转关闭时启用。当设备处于旋转锁定模式时，用户可将其屏幕锁定到顶层可见 Activity 所支持的任何旋转。 Activity 不应假定它将始终以纵向呈现。

**影响**：用户可以在关闭自动旋转的情况下，手动旋转屏幕；能否旋转取决于顶层可见Activity所支持的旋转设置。

### 文本
**更新内容**:<br>
* 文本预先计算：PrecomputedText 类使您能提前计算和缓存所需信息，改善了文本渲染性能。
* 放大器：Magnifier 类是一种可提供放大器API的微件，可在所有应用中实现一致的放大器功能体验。
* Smart Linkify：Android 9增强了TextClassifier类，该类可利用机器学习在选定文本中识别一些实体并建议采取相应的操作。
* 文本布局：借助几种便捷函数和属性，可以更轻松地实现界面设计。

**影响**：这个无疑强化了TextView的能力，应该比现在设置TextVie的样式方便。

### 电源管理
**更新内容**:<br>
* 新增了一个`应用待机群组`的概念，系统将根据用户的使用模式限制应用对 CPU 或电池等设备资源的访问。 <br>
  五个群组如下：
1. 活跃<br>
    如果用户当前正在使用应用，应用将被归到“活跃”群组中。<br>
    如果应用处于“活跃”群组，系统不会对应用的作业、报警或 FCM 消息施加任何限制。
2. 工作集<br>
    如果应用经常运行，但当前未处于活跃状态，它将被归到“工作集”群组中。<br>
    如果应用处于“工作集”群组，系统会对它运行作业和触发报警的能力施加轻度限制。
3. 常用<br>
    如果应用会定期使用，但不是每天都必须使用，它将被归到“常用”群组中。<br>
    如果应用处于“常用”群组，系统将对它运行作业和触发报警的能力施加较强的限制，也会对高优先级 FCM 消息的数量设定限制。
4. 极少使用<br>
    如果应用不经常使用，那么它属于“极少使用”群组。<br>
    如果应用处于“极少使用”群组，系统将对它运行作业、触发警报和接收高优先级FCM消息的能力施加严格限制。系统还会限制应用连接到网络的能力。
5. 从未使用<br>
    安装但是从未运行过的应用会被归到“从未使用”群组中。 系统会对这些应用施加极强的限制。

**影响**：无疑，Google在帮Android机省电、省资源，算是一个比较好的更新。但是，我相信厂商会对分组策略稍作修改（或者白名单啥的），以方便自身应用可以随时唤起。总的来说，应该对应用的Push类功能影响较大。

### 权限变更
**更新内容**:<br>
* 构建序列号弃用：在 Android 9 中，Build.SERIAL 始终设置为 "UNKNOWN" 以保护用户的隐私。
  如果您的应用需要访问设备的硬件序列号，您应改为请求 READ_PHONE_STATE 权限，然后调用 getSerial()。
* 前台服务：针对 Android 9 或更高版本并使用前台服务的应用必须请求 FOREGROUND_SERVICE 权限。
* 后台对传感器的访问受限：Android 9限制后台应用访问用户输入和传感器数据的能力。
* 限制访问通话记录：Android 9 引入 CALL_LOG 权限组并将 READ_CALL_LOG、WRITE_CALL_LOG 和 PROCESS_OUTGOING_CALLS 权限移入该组。
* 限制访问电话号码：在未首先获得 READ_CALL_LOG 权限的情况下，除了应用的用例需要的其他权限之外，运行于 Android 9 上的应用无法读取电话号码或手机状态。
* 限制访问 Wi-Fi 位置和连接信息：在 Android 9 中，应用进行 Wi-Fi 扫描的权限要求比之前的版本更严格。具体限制见官网。

**影响**：权限变更对收集手机应用信息和监听应用WiFi状态（主要是位置信息）都有很大影响，特别是对这些信息有依赖的业务。

### Apache Http被弃用
**更新内容**:<br>
从 Android 9 开始，默认情况下Apache网络库已从 bootclasspath 中移除且不可用于应用。

**影响**：终于要彻底抛弃Apache库了，但业务硬要用的话，还是可以自己导入jar包或者在AndroidManifest.xml中使用uses-library标签。

## 适配齐刘海
有Pixel的同学可以把系统升级到Pie查看效果，也可以下载相应镜像用模拟器测试（最好使用AS3.1以上）。Pie提供了三种刘海样式：<br>
1. 边角刘海（我相信绝对没厂商用...）

![边角刘海](https://user-gold-cdn.xitu.io/2018/9/21/165fb4eb548357c7?w=366&h=220&f=png&s=28539)

2. 正常刘海（被iPhone X带出来的风格...）

![正常刘海](https://user-gold-cdn.xitu.io/2018/9/21/165fb4ff26e6fa15?w=382&h=192&f=png&s=28562)

3. 刘海+下巴（为啥要有下巴...）

![刘海](https://user-gold-cdn.xitu.io/2018/9/21/165fb51608528901?w=360&h=182&f=png&s=21921)

![下巴](https://user-gold-cdn.xitu.io/2018/9/21/165fb519cd6751bd)

在Pie中，官方提供了`DisplayCutout`类来获取刘海块相关数据；通过`getDisplayCutout()`函数可以知道刘海是否存在；通过窗口布局属性`layoutInDisplayCutoutMode`可以让应用为设备屏幕缺口周围的内容进行布局。直接上代码吧：
```java
    @TargetApi(28)
    public void showDisplayCutout(View view) {
        // 注意：getRootWindowInsets()只有在视图加载完成后，才会返回，否则返回null
        DisplayCutout cutout = getWindow().getDecorView().getRootWindowInsets().getDisplayCutout();
        if (cutout == null) { // 没有刘海的时候拿到的DisplayCutout为null
            Log.d(TAG, "showDisplayCutout: 普通屏，没有刘海");
            return;
        }
        List<Rect> rects = cutout.getBoundingRects(); // 获取可能的凹凸块
        if (rects.size() == 1) {
            Log.d(TAG, "showDisplayCutout: 有刘海");
            Rect rect = rects.get(0);
            Log.d(TAG, "showDisplayCutout: 刘海区域：" + rect);
        } else {
            Log.d(TAG, "showDisplayCutout: 有刘海和下巴");
            for (Rect rect : rects) {
                Log.d(TAG, "showDisplayCutout: 区域：" + rect);
            }
        }

        // 获取安全区域的数据，单位px
        int safeInsetLeft = cutout.getSafeInsetLeft();
        int safeInsetTop = cutout.getSafeInsetTop();
        int safeInsetRight = cutout.getSafeInsetRight();
        int safeInsetBottom = cutout.getSafeInsetBottom();
        Log.d(TAG, "showDisplayCutout: 安全区域距离屏幕左边：" + safeInsetLeft);
        Log.d(TAG, "showDisplayCutout: 安全区域距离屏幕顶部：" + safeInsetTop);
        Log.d(TAG, "showDisplayCutout: 安全区域距离屏幕右边：" + safeInsetRight);
        Log.d(TAG, "showDisplayCutout: 安全区域距离屏幕底部：" + safeInsetBottom);

        int statusBarHeight = getStatusBarHeight();
        Log.d(TAG, "showDisplayCutout: 状态栏高度：" + statusBarHeight);
    }

    private int getStatusBarHeight() {
        int result = 0;
        int resourceId = getResources().getIdentifier("status_bar_height", "dimen", "android");
        if (resourceId > 0) {
            result = getResources().getDimensionPixelSize(resourceId);
        }
        return result;
    }
```
**注意**：

* getRootWindowInsets()只有在View加载完后才会返回值，否则返回null。所以，如果你想在生命周期（onCreate等）里直接调用的话，可能会出现空指针异常，可以使用post runnable的方式处理。
* 刘海的高度不一定和状态栏的高度一样，应该是小于等于状态栏高度。

设置窗口属性：

```java
@Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_second);

        // 设置全屏
        getWindow().getDecorView().setSystemUiVisibility(View.SYSTEM_UI_FLAG_FULLSCREEN | View.SYSTEM_UI_FLAG_LAYOUT_FULLSCREEN);

        //沉浸式状态栏
//        getWindow().addFlags(WindowManager.LayoutParams.FLAG_TRANSLUCENT_STATUS);

        WindowManager.LayoutParams lp = getWindow().getAttributes();
        // 只有当DisplayCutout完全包含在系统栏中时，才允许窗口延伸到DisplayCutout区域。 否则，窗口布局不与DisplayCutout区域重叠。
        lp.layoutInDisplayCutoutMode = WindowManager.LayoutParams.LAYOUT_IN_DISPLAY_CUTOUT_MODE_DEFAULT;
        // 该窗口决不允许与DisplayCutout区域重叠。
//        lp.layoutInDisplayCutoutMode = WindowManager.LayoutParams.LAYOUT_IN_DISPLAY_CUTOUT_MODE_NEVER;
        // 该窗口始终允许延伸到屏幕短边上的DisplayCutout区域。
//        lp.layoutInDisplayCutoutMode = WindowManager.LayoutParams.LAYOUT_IN_DISPLAY_CUTOUT_MODE_SHORT_EDGES;
        getWindow().setAttributes(lp);
    }
```

部分效果图：<br>
1. 普通页面带状态栏
    ![页面带状态栏](https://user-gold-cdn.xitu.io/2018/9/21/165fb845dad9af31?w=384&h=805&f=png&s=67763)

2. 全屏页面默认情况
    ![全屏页面默认情况](https://user-gold-cdn.xitu.io/2018/9/21/165fb85dbd884444?w=384&h=803&f=png&s=65456)

3. 全屏页面强制使用刘海区域
    ![全屏强制使用刘海区域](https://user-gold-cdn.xitu.io/2018/9/21/165fb86b953ee642?w=383&h=800&f=png&s=62403)

4. 沉浸式
    ![沉浸式页面](https://user-gold-cdn.xitu.io/2018/9/21/165fb87f609ee05f?w=392&h=802&f=png&s=69495)

**适配方案**：

* 对于有状态栏的页面，不会受到刘海的影响，内容会展示在状态栏下方。
* 对于全屏页面，默认情况下系统会把展示内容下移到刘海之下，会产生一个黑条，此时需要注意页面底部的内容，要避免因为下移导致的显示不全问题；如果修改了窗口配置为`LAYOUT_IN_DISPLAY_CUTOUT_MODE_SHORT_EDGES`，则会强制使用刘海区域，这时页面要规避刘海块的那一点区域。
* 对于沉浸式页面，是受到影响最大的，默认情况下就会使用刘海区域，所以页面内容要进行区域规避。

## 非SDK接口限制
Android 9（API 级别 28）引入了针对非 SDK 接口的使用限制，无论是直接使用还是通过反射或 JNI 间接使用。无论应用是引用非 SDK 接口还是尝试使用反射或 JNI 获取其句柄，均适用这些限制。<br>
如果我们使用非SDK接口，会出现几种情况：
1. logcat日志<br>
    类似 Accessing hidden field Landroid/os/Message;->flags:I (light greylist, JNI) 格式；
2. 弹警告Toast
3. 引发系统错误

非SDK接口都记录在了不同的名单下：
1. 白名单：SDK
2. 浅灰名单：仍可以访问的非 SDK 函数/字段。
3. 深灰名单：
    对于目标 SDK 低于 API 级别 28 的应用，允许使用深灰名单接口。
    对于目标 SDK 为 API 28 或更高级别的应用：行为与黑名单相同。
4. 黑名单：受限，无论目标 SDK 如何，平台将表现为似乎接口并不存在。

所有的名单文件都在：https://android.googlesource.com/platform/prebuilts/runtime/+/master/appcompat ，其中还有一个`veridex`检测工具（目前只支持linux和mac）。

### 限制原理分析
直接借用大佬们的文章：
[360奇卓社-Android P 调用隐藏API限制原理](https://mp.weixin.qq.com/s/sktB0x5yBexkn4ORQ1YofA)<br>
大致过程：
1. 编译过程中有一个hiddenapi过程，会根据名单文件，对dex中所有函数和字段进行重新标记；
2. runtime的时候，Art虚拟机会根据函数和字段的标记，并结合调用者的身份返回特定值；这些值就决定了调用者是否能成功调用接口。

![阶段一](https://user-gold-cdn.xitu.io/2018/9/21/165fbaed2e360078?w=483&h=307&f=png&s=63531)

![阶段二](https://user-gold-cdn.xitu.io/2018/9/21/165fbaeff62b34b2?w=591&h=266&f=png&s=62137)

### 如何绕过
还是大佬们的文章：
1. [360奇卓社-突破Android P(Preview 1)对调用隐藏API限制的方法](https://mp.weixin.qq.com/s/4k3DBlxlSO2xNNKqjqUdaQ)
2. [一种绕过Android P对非SDK接口限制的简单方法](http://weishu.me/2018/06/07/free-reflection-above-android-p/)
3. [突破Android P非SDK API限制的几种代码实现](https://juejin.im/post/5ba0f3f7e51d450e6f2e39e0?utm_source=gold_browser_extension)

其中替换`classloader`的方式目前是最可靠的。

## 总结
1. 文中适配齐刘海的演示代码可以在：https://github.com/zjxstar/AndroidSamples/tree/master/AdaptPieSample 中查看。
2. 对于非SDK接口，能不用就不用吧（不用才怪...）。
3. 对于其他行为变更，主要关注隐私权限的变更，毕竟用户对隐私权限越来越敏感了。

## 参考资料：
1. [Google官网](https://developer.android.com/about/versions/pie/)；（强调一下）
2. 其他资料已经在文中列出，不再重复；