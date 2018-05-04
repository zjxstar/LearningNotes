## WebView渲染引擎
> [罗大大的系列博客](http://blog.csdn.net/luoshengyang/article/details/46569161)
1. Android4.4之前基于WebKit，详情：[WebKit](https://webkit.org/)；
2. Android4.4之后基于Chromium；

## WebView关键类
1. [WebView](https://developer.android.com/reference/android/webkit/WebView.html)：加载网页的容器，包含了网页加载的常用方法，如：加载、前进、后退等；
2. [WebSettings](https://developer.android.com/reference/android/webkit/WebSettings.html)：对WebView进行配置和管理，如：配置缓存、控制缩放等；
3. [WebViewClient](https://developer.android.com/reference/android/webkit/WebViewClient.html)：帮助WebView处理各种回调、通知和请求事件；
4. [WebChromeClient](https://developer.android.com/reference/android/webkit/WebChromeClient.html)辅助WebView处理JavaScript的对话框、图标、Title、加载进度等信息；

## JS和Android的交互方法
> [参考资料](http://blog.csdn.net/carson_ho/article/details/64904691)
### 对于Android调用JS代码的方法有2种
1. 通过WebView的loadUrl()；直接加载“javascript:方法”；
2. 通过WebView的evaluateJavascript()；4.4以上才能使用，不会使页面刷新，容易获取JS返回的结果；

![两种方法比较](https://user-gold-cdn.xitu.io/2018/2/3/1615a6f8dd74764f?w=953&h=303&f=png&s=41118)

### JS调用Android代码的方法有3种
1. 通过WebView的addJavascriptInterface()进行对象映射；4.2以下注意安全问题，4.2以上要在添加@JavascriptInterface注解；
2. 通过WebViewClient的shouldOverrideUrlLoading()方法回调拦截url；需要事先约定协议，解析url；缺点是JS获取Android方法的返回值复杂；
3. 通过WeChromeClient的onJsAlert()、onJsConfirm()、onJsPrompt方法回调拦截JS对话框alert()、confirm()、prompt()消息；需要事先约定协议；

![三种方法比较](https://user-gold-cdn.xitu.io/2018/2/3/1615a71994ae5fed?w=1084&h=393&f=png&s=87781)

## 交互的安全漏洞
> [参考资料](http://blog.csdn.net/carson_ho/article/details/64904635)
### WebView任意代码执行漏洞
#### 原因
1. WebView中的addJavascriptInterface()接口（4.2之前）；
2. WebView内置导出的searchBoxJavaBridge_对象；
3. WebView内置导出的accessibility和accessibilityTraversal对象；
 
#### 风险

![JS端执行任意代码](https://user-gold-cdn.xitu.io/2018/2/3/1615a73b329a4c2d?w=781&h=296&f=png&s=19098)

#### 解决方法
1. 4.2以上，在方法上增加@JavascriptInterface注解；
2. 4.2以下，注入一段本地JS代码来约束JS端的方法调用；核心：将本地JSCall对象中的所有方法都以字符串的形式写入JS代码中，JS中方法调用返回JSON字符串，本地解析JSON字符串反射调用相应方法；注意：需要过滤掉Object类的方法；

![需要过滤的方法](https://user-gold-cdn.xitu.io/2018/2/3/1615a75ac38dce73?w=112&h=207&f=png&s=1921)

3. 删除内置导出的对象

![删除内置对象](https://user-gold-cdn.xitu.io/2018/2/3/1615a76499db9380?w=708&h=123&f=png&s=11325)

### 域控制不严格漏洞

![涉及的方法](https://user-gold-cdn.xitu.io/2018/2/3/1615a776430e8c89?w=447&h=100&f=png&s=7489)

![方法说明1](https://user-gold-cdn.xitu.io/2018/2/3/1615a777ee21e7da?w=507&h=88&f=png&s=4226)

![方法说明2](https://user-gold-cdn.xitu.io/2018/2/3/1615a77952dd140d?w=518&h=124&f=png&s=5348)

## WebView的缓存机制
> [参考资料1](http://blog.csdn.net/carson_ho/article/details/71402764)
> [参考资料2](https://segmentfault.com/a/1190000004132566#articleHeader3)
### 浏览器页面缓存
#### 原理
根据 HTTP 协议头里的 Cache-Control（或 Expires）和 Last-Modified（或 Etag）等字段来控制文件缓存的机制；
#### WebView端控制
![WeView浏览器缓存控制](https://user-gold-cdn.xitu.io/2018/2/3/1615a79dd30a1409?w=690&h=128&f=png&s=14403)

#### 缓存目录
1. Android4.3：/data/data/package name/cache/webviewCacheChromium

![Android4.3缓存目录](https://user-gold-cdn.xitu.io/2018/2/3/1615a7b259788574?w=268&h=197&f=png&s=7161)

2. Android5.0：/data/data/package name/app_webview/Cache

![Android5.0缓存目录](https://user-gold-cdn.xitu.io/2018/2/3/1615a7bbcac7b413?w=215&h=235&f=png&s=6686)

3. Android6.0：/data/data/package name/cache

![Android6.0缓存目录](https://user-gold-cdn.xitu.io/2018/2/3/1615a7c82fa24e0f?w=266&h=236&f=png&s=7954)

### Application Cache

![Application Cache](https://user-gold-cdn.xitu.io/2018/2/3/1615a7d5fe163c30?w=494&h=119&f=png&s=9406)
<br>
**注意**：高版本中 setAppCachePath 方法不会生效；<br>
Android4.3中的缓存目录位置：
![Android4.3缓存目录](https://user-gold-cdn.xitu.io/2018/2/3/1615a7ef854b8582?w=268&h=244&f=png&s=7387)

### Dom Storage缓存机制
#### 原理
Dom Storage 是通过存储字符串的 Key/Value 对来提供的，并提供 5MB （不同浏览器可能不同，分 HOST)的存储空间（Cookies 才 4KB)。另外 Dom Storage 存储的数据在本地，不像 Cookies，每次请求一次页面，Cookies 都会发送给服务器。
DOM Storage 分为 sessionStorage 和 localStorage。localStorage 对象和 sessionStorage 对象使用方法基本相同，它们的区别在于作用的范围不同。sessionStorage 用来存储与页面相关的数据，它在页面关闭后无法使用。而 localStorage 则持久存在，在页面关闭后也可以使用。
#### 开启方法

![Dom缓存开启](https://user-gold-cdn.xitu.io/2018/2/3/1615a80d58cf795f?w=433&h=41&f=png&s=2587)

#### 缓存目录
Android6.0：

![Android6.0缓存目录](https://user-gold-cdn.xitu.io/2018/2/3/1615a81a39c8d0ab?w=282&h=316&f=png&s=11335) <br>
Android5.0：

![Android5.0缓存目录](https://user-gold-cdn.xitu.io/2018/2/3/1615a8251cc2870a?w=262&h=330&f=png&s=8825)

### Web SQL Database缓存机制
#### 原理
H5 也提供基于 SQL 的数据库存储机制，用于存储适合数据库的结构化数据。根据官方的标准文档，Web SQL Database 存储机制不再推荐使用，将来也不再维护，而是推荐使用 AppCache 和 IndexedDB。

#### 开启方法

![开启方法](https://user-gold-cdn.xitu.io/2018/2/3/1615a8379f3c879a?w=665&h=76&f=png&s=16728)

### Indexed Database机制
#### 原理
IndexedDB 也是一种数据库的存储机制，但不同于已经不再支持的 Web SQL Database。IndexedDB 不是传统的关系数据库，可归为 NoSQL 数据库。IndexedDB 又类似于 Dom Storage 的 key-value 的存储方式，但功能更强大，且存储空间更大。

#### 开发方法

![开启方法](https://user-gold-cdn.xitu.io/2018/2/3/1615a887fe986f78?w=700&h=185&f=png&s=7886)

### File System API
#### 原理
File System API 是 H5 新加入的存储机制。它为 Web App 提供了一个虚拟的文件系统，就像 Native App 访问本地文件系统一样。由于安全性的考虑，这个虚拟文件系统有一定的限制。Web App 在虚拟的文件系统中，可以进行文件（夹）的创建、读、写、删除、遍历等操作。
File System API 也是一种可选的缓存机制，和前面的 SQLDatabase、IndexedDB 和 AppCache 等一样。File System API 有自己的一些特定的优势：
1. 可以满足大块的二进制数据（ large binary blobs）存储需求。
2. 可以通过预加载资源文件来提高性能。
3. 可以直接编辑文件。

**注意**：到目前，Android 系统的 Webview 还不支持 File System API。

### 部分缓存文件的存储格式
以Android4.3中的文件为例：DB文件

![部分文件1](https://user-gold-cdn.xitu.io/2018/2/3/1615a8bcbb2d9d72?w=254&h=445&f=png&s=15576)

![部分文件2](https://user-gold-cdn.xitu.io/2018/2/3/1615a8beca9b4d6a?w=240&h=346&f=png&s=10514)

### 各种缓存机制的比较

![缓存机制比较](https://user-gold-cdn.xitu.io/2018/2/3/1615a8cee7ad7873?w=655&h=368&f=png&s=37928)

## WebView内存泄露问题
> [参考资料](http://leehong2005.com/2016/08/19/webview-memory-leak/)
1. 尽量不要在xml中定义webview节点，而是动态生成；
2. 在onDestroy方法中执行：
```
public void destroy() {
        if (mWebView != null) {
            // 如果先调用destroy()方法，则会命中if (isDestroyed()) return;这一行代码，需要先onDetachedFromWindow()，再
            // destory()
            ViewParent parent = mWebView.getParent();
            if (parent != null) {
                ((ViewGroup) parent).removeView(mWebView);
            }

            mWebView.stopLoading();
            // 退出时调用此方法，移除绑定的服务，否则某些特定系统会报错
            mWebView.getSettings().setJavaScriptEnabled(false);
            mWebView.clearHistory();
            mWebView.clearView();
            mWebView.removeAllViews();

            try {
                mWebView.destroy();
            } catch (Throwable ex) {

            }
        }
    }
```
3. 在单独进程中加载WebView，使用完后直接kill进程；

## WebView的重定向问题
### 常见错误写法
shouldOverrideUrlLoading 方法中调用view.loadUrl方法；

![错误写法](https://user-gold-cdn.xitu.io/2018/2/3/1615a92c5285cdab?w=986&h=423&f=png&s=32698)

### 解决方法
应该去掉view.loadUrl（url）调用；

![源码说明原因](https://user-gold-cdn.xitu.io/2018/2/3/1615a93b0b8ff0fc?w=870&h=594&f=png&s=56741)

### 注意事项
主动调用webview.load(url)的话不会触发shouldOverrideUrlLoading方法；
> [参考](http://blog.csdn.net/a0407240134/article/details/51482021)

### 其他处理重定向方案
1. WebView有一个getHitTestResult():返回的是一个HitTestResult，一般会根据打开的链接的类型，返回一个extra的信息，如果打开链接不是一个url，或者打开的链接是JavaScript的url，他的类型是UNKNOWN_TYPE，这个url就会通过requestFocusNodeHref(Message)异步重定向。返回的extra为null，或者没有返回extra。根据此方法的返回值，判断是否为null，可以用于解决网页重定向。
2. 自定义回退栈。

## 参考资料
1. https://www.kancloud.cn/digest/android-safe/107906
2. https://my.oschina.net/zhibuji/blog/100580
3. https://juejin.im/entry/5977598d51882548c0045bde
