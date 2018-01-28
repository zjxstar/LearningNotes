## HTTP
### 基础
> 超文本传输协议（HTTP，HyperText Transfer Protocol），默认端口：80
### 特点
1. 无连接：无连接的含义是限制每次连接只处理一个请求。服务器处理完客户的请求，并收到客户的应答后，即断开连接。采用这种方式可以节省传输时间。
2. 无状态：HTTP协议是无状态协议，无状态是指协议对于事务处理没有记忆能力。缺少状态意味着如果后续处理需要前面的信息，则它必须重传，这样可能导致每次连接传送的数据量增大。另一方面，在服务器不需要先前信息时它的应答就较快。

### 请求报文
![Http请求头](https://user-gold-cdn.xitu.io/2018/1/22/1611e38e0ef8f459?w=401&h=253&f=png&s=31106)
&emsp;&emsp;一般由请求行、请求头、空行、请求体四部分组成。

### 响应报文
![Http响应头](https://user-gold-cdn.xitu.io/2018/1/22/1611e3f8c1f449e6?w=400&h=246&f=png&s=31142)
&emsp;&emsp;一般由状态行、消息报头、空行、响应正文四部分组成。

### 常见状态码
* 200 OK：客户端请求成功
* 302 Move temporarily：请求的资源临时从不同的 URI响应请求，重定向
* 304 Not Modified：有效缓存
* 400 Bad Request：客户端请求有语法错误，不能被服务器所理解
* 403 Forbidden：服务器收到请求，但是拒绝提供服务
* 404 Not Found：请求失败，请求所希望得到的资源未被在服务器上发现
* 500 Internal Server Error：服务器发生不可预期的错误
* 503 Server Unavailable：服务器当前不能处理客户端的请求，一段时间后可能恢复正常

### 常见通用报头
* Cache-Control：指定请求和响应遵循的缓存机制

### 常用请求报头
* Host：请求的主机名，允许多个域名同处一个IP地址，即虚拟主机
* User-Agent：发送请求的浏览器类型、操作系统等信息
* Accept：客户端可识别的内容类型列表，用于指定客户端接收那些类型的信息
* Accept-Encoding：客户端可识别的数据编码
* Connection：允许客户端和服务器指定与请求/响应连接有关的选项，例如这是为Keep-Alive则表示保持连接
* Referer：允许客户端指定请求uri的源资源地址，这可以允许服务器生成回退链表，可用来登陆、优化cache等
* Range：可以请求实体的一个或者多个子范围

### 常见响应报头
* Location：用于重定向接受者到一个新的位置，常用在更换域名的时候
* Server：包含可服务器用来处理请求的系统信息，与User-Agent请求报头是相对应的

### 常见实体报头
* Content-Type：发送给接收者的实体正文的媒体类型
* Content-Lenght：实体正文的长度
* Content-Encoding：实体报头被用作媒体类型的修饰符，它的值指示了已经被应用到实体正文的附加内容的编码，因而要获得Content-Type报头域中所引用的媒体类型，必须采用相应的解码机制
* Last-Modified：实体报头用于指示资源的最后修改日期和时间
* Expires：实体报头给出响应过期的日期和时间

### SPDY
> 2012年google提出的SPDY方案，主要解决：
1. **降低延迟**：SPDY采取了多路复用（multiplexing）。多路复用通过多个请求stream共享一个tcp连接的方式；
2. **请求优先级**：SPDY允许给每个request设置优先级，这样重要的请求就会优先得到响应；
3. **header压缩**：选择合适的压缩算法可以减小包的大小和数量；
4. **基于HTTPS的加密协议传输**
5. **服务端推送**

![SPDY](https://user-gold-cdn.xitu.io/2018/1/22/1611e61b1ceb0956?w=206&h=240&f=png&s=8881)

### Http2.0
&emsp;&emsp;可以看成是SPDY的升级版。

#### Http2.0与SPDY的区别
* HTTP2.0 支持明文 HTTP 传输，而 SPDY 强制使用 HTTPS
* HTTP2.0 消息头的压缩算法采用 HPACK，而非 SPDY 采用的 DEFLATE

#### Http2.0新特性
* **新的二进制格式**：HTTP1.x的解析是基于文本的
* **多路复用**：即连接共享，即每一个request都是是用作连接共享机制的
* **header压缩**
* **服务端推送**

## HTTPS
> (Hyper Text Transfer Protocol over Secure Socket Layer)，是以安全为目标的HTTP通道，即HTTP下加入SSL层，端口：443

### HTTPS握手流程

![Https流程](https://user-gold-cdn.xitu.io/2018/1/22/1611e7021b647634?w=639&h=560&f=png&s=65353)
注意：下面流程与图中序列不是一致的。
1. 客户端的浏览器向服务器传送客户端SSL协议的版本号，加密算法的种类，产生的随机数，以及其他服务器和客户端之间通讯所需要的各种信息；
2. 服务器向客户端传送SSL协议的版本号，加密算法的种类，随机数以及其他相关信息，同时服务器还将向客户端传送自己的证书；
3. 客户利用服务器传过来的信息验证服务器的合法性，服务器的合法性包括：证书是否过期，发行服务器证书的CA是否可靠，发行者证书的公钥能否正确解开服务器证书的“发行者的数字签名”，服务器证书上的域名是否和服务器的实际域名相匹配。如果合法性验证没有通过，通讯将断开；如果合法性验证通过，将继续进行第四步；
4. 用户端随机产生一个用于后面通讯的“对称密码”，然后用服务器的公钥（服务器的公钥从步骤②中的服务器的证书中获得）对其加密，然后将加密后的“预主密码”传给服务器；
5. 如果服务器要求客户的身份认证（在握手过程中为可选），用户可以建立一个随机数然后对其进行数据签名，将这个含有签名的随机数和客户自己的证书以及加密过的“预主密码”一起传给服务器；
6. 如果服务器要求客户的身份认证，服务器必须检验客户证书和签名随机数的合法性，具体的合法性验证过程包括：客户的证书使用日期是否有效，为客户提供证书的CA是否可靠，发行CA 的公钥能否正确解开客户证书的发行CA的数字签名，检查客户的证书是否在证书废止列表（CRL）中。检验如果没有通过，通讯立刻中断；如果验证通过，服务器将用自己的私钥解开加密的“预主密码”，然后执行一系列步骤来产生主通讯密码（客户端也将通过同样的方法产生相同的主通讯密码）；
7. 服务器和客户端用相同的主密码即“通话密码”，一个对称密钥用于SSL协议的安全数据通讯的加解密通讯。同时在SSL通讯过程中还要完成数据通讯的完整性，防止数据通讯中的任何变化；
8. 客户端向服务器端发出信息，指明后面的数据通讯将使用的步骤⑦中的主密码为对称密钥，同时通知服务器客户端的握手过程结束；
9. 服务器向客户端发出信息，指明后面的数据通讯将使用的步骤⑦中的主密码为对称密钥，同时通知客户端服务器端的握手过程结束；
10. SSL 的握手部分结束，SSL安全通道的数据通讯开始，客户和服务器开始使用相同的对称密钥进行数据通讯，同时进行通讯完整性的检验。

## Android中处理Https
> 官方推荐：[Android推荐方案](https://developer.android.com/training/articles/security-ssl.html)

### 处理 javax.net.ssl.SSLHandshakeException: 证书验证失败
```
/**
     * HTTPS未知的证书颁发机构处理方法
     * Android客户端存储证书
     *
     * @param input 待信任的CA证书流
     * @return SSLContext
     */
    public static SSLContext getSSLContext(InputStream input) {
        try {
            SSLContext sslContext = SSLContext.getInstance("TLS");
            CertificateFactory cf = CertificateFactory.getInstance("X.509");
            Certificate ca = cf.generateCertificate(input);

            KeyStore keyStore = KeyStore.getInstance(KeyStore.getDefaultType());
            keyStore.load(null, null);
            keyStore.setCertificateEntry("ca", ca);

            TrustManagerFactory tmf = TrustManagerFactory.getInstance(TrustManagerFactory.getDefaultAlgorithm());
            tmf.init(keyStore);

            sslContext.init(null, tmf.getTrustManagers(), null);
            input.close();
            return sslContext;
        } catch (NoSuchAlgorithmException e) {
            e.printStackTrace();
        } catch (CertificateException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        } catch (KeyStoreException e) {
            e.printStackTrace();
        } catch (KeyManagementException e) {
            e.printStackTrace();
        }

        return null;
    }
```

### 单独使用SSL和HTTP
```
/**
     * 单独使用SSL + HTTP时
     * @param trustManagers
     * @return
     */
    public static SSLContext getSSLContext(TrustManager[] trustManagers) {
        try {
            SSLContext sslContext = SSLContext.getInstance("TLS");
            sslContext.init(null, trustManagers, null);
            return sslContext;
        } catch (NoSuchAlgorithmException e) {
            e.printStackTrace();
        } catch (KeyManagementException e) {
            e.printStackTrace();
        }
        return null;
    }

    public static TrustManager[] sDefaultTrustManagers = new TrustManager[] {new X509TrustManager() {

        @Override
        public void checkClientTrusted(X509Certificate[] chain, String authType) throws CertificateException {
            // TODO: 2018/1/23双向校验中，向服务端发客户端证书
        }

        @Override
        public void checkServerTrusted(X509Certificate[] chain, String authType) throws CertificateException {
            // TODO: 2018/1/23 双向校验中，校验服务端证书
        }

        @Override
        public X509Certificate[] getAcceptedIssuers() {
            return new X509Certificate[0];
        }
    }};

    public static HostnameVerifier sHostnameVerifier = new HostnameVerifier() {
        @Override
        public boolean verify(String hostname, SSLSession session) {
            // 不验证主机名
            return true;
        }
    };
```
