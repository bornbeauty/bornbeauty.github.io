---
layout: post
title: Http协议
category: 网络
tags: NetWork
description: 
date: 2016-10-16
---

学习目标：
1、 了解http协议
2、 了解cookie的作用
3、 使用socket实现http请求
4、 说明OKhttp是如何保存cookie的

#### 一、 http协议

##### 1. 什么是http协议
http协议，即超文本传输协议，是一种基于TCP协议，规定了服务器与浏览器之间相互通信规则的运行于应用层的数据传输协议。之所以被称为超文本传输协议，是因为http协议不仅能够传输文本，还能传输图片，视频等多媒体数据。在某些情况下，http协议还会和TSL/SSL协议一起使用，进行更加安全可靠的数据传输工作，即[https](http://www.ruanyifeng.com/blog/2014/02/ssl_tls.html)。

http默认的端口号是80，https默认端口号是443

![](http://images.cnitblog.com/i/116165/201407/111703047392802.png)


##### 2. http的工作流程

一个完整的http操作可分成下面四步：

1. 首先客户端与服务器之间建立链连接。
2. 建立连接后，客户端向服务器发送一个请求。
3. 服务器收到请求后给予相应的回应。
4. 适时关闭客户端与服务端的连接。

##### 3. http请求格式

http请求由三个部分组成：请求行，消息报头，请求正文
具体格式如下:
请求行</br>
消息报头</br></br>
请求正文

###### 请求行：

以请求方法开头+一个空格+请求的URI+一个空格+协议版本，例如：
GET /login.json HTTP/1.1

http请求方式：

- GET     请求获取Request-URI所标识的资源
- POST    在Request-URI所标识的资源后附加新的数据
- HEAD    请求获取由Request-URI所标识的资源的响应消息报头
- PUT     请求服务器存储一个资源，并用Request-URI作为其标识
- DELETE  请求服务器删除Request-URI所标识的资源
- TRACE   请求服务器回送收到的请求信息，主要用于测试或诊断
- CONNECT 保留将来使用
- OPTIONS 请求查询服务器的性能，或者查询与资源相关的选项和需求

###### 请求报头：

请求报头允许客户端向服务端传递请求的附加信息以及客户端自身信息。

- Accept: 表示客户端可以接受的数据格式
- Accept-Encoding: 表示客户端可以接受的编码格式，一般指定压缩格式
- User-Agent: 告诉服务器，客户端的操作系统和浏览器的名称和版本
- Host: 指定服务器 格式：www.163.com:80

###### 请求正文

客户端可以在请求正文中添加要向服务器提交的数据。当然，这个请求要结合实体报头一起使用，请求与相应都可以添加这个实体头。

- Content-Encoding: 实体报头域被用作媒体类型的修饰符，它的值指示了已经被应用到实体正文的附加内容的编码，因而要获得Content-Type报头域中所引用的媒体类型，必须采用相应的解码机制。Content-Encoding这样用于记录文档的压缩方法，例如Content-Encoding：gzip
- Content-Language: 实体报头域描述了资源所用的自然语言。没有设置该域则认为实体内容将提供给所有的语言阅读者。
- Content-Type: 实体报头域用语指明发送给接收者的实体正文的媒体类型。

![](http://img.blog.csdn.net/20131107150723906)

##### 4. http响应

响应报文主要由状态行，响应报头和响应报文三个部分组成。

###### 状态行

状态行由三个部分组成：协议版本+一个空格+状态码+一个空格+状态码描述
例如： HTTP/1.1 200 OK

状态代码有三位数字组成，第一个数字定义了响应的类别，且有五种可能取值：

- 1xx：指示信息--表示请求已接收，继续处理
- 2xx：成功--表示请求已被成功接收、理解、接受
- 3xx：重定向--要完成请求必须进行更进一步的操作
- 4xx：客户端错误--请求有语法错误或请求无法实现
- 5xx：服务器端错误--服务器未能实现合法的请求

##### 响应报头

- Location：响应报头域用于重定向接受者到一个新的位置。
- Server：响应报头域包含了服务器用来处理请求的软件信息。与User-Agent请求报头域是相对应的。

##### 响应报文

服务器将要返回的资源内容存放于此

![](http://img.blog.csdn.net/20131107163544468)

##### 5. 关于https

简单说，https协议就是一种安全版的http协议。通过TSL/SSL协议，提供客户端与服务器之间安全完整的数据传输能力。

TSL/SSL可以保证一下几点：
1. 认证用户和服务器，确保数据发送到正确的客户机和服务器
2. 加密数据以防止数据中途被窃取
3. 维护数据的完整性，确保数据在传输过程中不被改变

工作过程：

1. 客户端给出协议版本号、一个客户端生成的随机数（Client random），以及客户端支持的加密方法。
2. 服务器确认双方使用的加密方法，并给出数字证书、以及一个服务器生成的随机数（Server random）。
3. 客户端确认数字证书有效，然后生成一个新的随机数（Premaster secret），并使用数字证书中的公钥，加密这个随机数，发给鲍勃。
4. 服务器使用自己的私钥，获取爱丽丝发来的随机数（即Premaster secret）。
5. 客户端和服务器根据约定的加密方法，使用前面的三个随机数，生成"对话密钥"（session key），用来加密接下来的整个对话过程。

通过上面5个过程，可以保证数据发送实在正确的客户端和服务器之间进行的，使用这五个过程得到的加密密钥来加密传输数据，这样能保证传输数据的安全和完整性。

[详解可见](http://www.cnblogs.com/JeffreySun/archive/2010/06/24/1627247.html)
[数字证书验证正确性](http://blog.csdn.net/chroming/article/details/50995378)

#### 二、cookie的作用

一般有两个作用：

1. 识别用户身份
   > 由于http请求是无状态的，服务器无法识别请求是来自于哪一个用户，或者说哪一个客户端，所以就需要一个数据来标识身份。
2. 记录历史数据

#### 三、使用socket来实现http协议

  [代码](https://www.zybuluo.com/jimbo/note/536681)

#### 四、OkHTTP是如何保存cookie的

在okhttp3.0中，构建request的时候可以看到如下代码private Request networkRequest(Request request) throws IOException {
    Request.Builder result = request.newBuilder();

    //例行省略....
    
    List<Cookie> cookies = client.cookieJar().loadForRequest(request.url());
    if (!cookies.isEmpty()) {
      result.header("Cookie", cookieHeader(cookies));
    }

    //例行省略....

    return result.build();
  }
```
private Request networkRequest(Request request) throws IOException {
    Request.Builder result = request.newBuilder();

    //例行省略....
    
    List<Cookie> cookies = client.cookieJar().loadForRequest(request.url());
    if (!cookies.isEmpty()) {
      result.header("Cookie", cookieHeader(cookies));
    }

    //例行省略....

    return result.build();
  }
  
  private String cookieHeader(List<Cookie> cookies) {
    StringBuilder cookieHeader = new StringBuilder();
    for (int i = 0, size = cookies.size(); i < size; i++) {
      if (i > 0) {
        cookieHeader.append("; ");
      }
      Cookie cookie = cookies.get(i);
      cookieHeader.append(cookie.name()).append('=').append(cookie.value());
    }
    return cookieHeader.toString();
  }
```

在收到response之后还会执行如下代码：

```
public void receiveHeaders(Headers headers) throws IOException {
    if (client.cookieJar() == CookieJar.NO_COOKIES) return;

    List<Cookie> cookies = Cookie.parseAll(userRequest.url(), headers);
    if (cookies.isEmpty()) return;

    client.cookieJar().saveFromResponse(userRequest.url(), cookies);
  }
```

而这`cookjar`是一个接口：

```
public interface CookieJar {
  /** A cookie jar that never accepts any cookies. */
  CookieJar NO_COOKIES = new CookieJar() {
    @Override public void saveFromResponse(HttpUrl url, List<Cookie> cookies) {
    }

    @Override public List<Cookie> loadForRequest(HttpUrl url) {
      return Collections.emptyList();
    }
  };

  /**
   * Saves {@code cookies} from an HTTP response to this store according to this jar's policy.
   *
   * <p>Note that this method may be called a second time for a single HTTP response if the response
   * includes a trailer. For this obscure HTTP feature, {@code cookies} contains only the trailer's
   * cookies.
   */
  void saveFromResponse(HttpUrl url, List<Cookie> cookies);

  /**
   * Load cookies from the jar for an HTTP request to {@code url}. This method returns a possibly
   * empty list of cookies for the network request.
   *
   * <p>Simple implementations will return the accepted cookies that have not yet expired and that
   * {@linkplain Cookie#matches match} {@code url}.
   */
  List<Cookie> loadForRequest(HttpUrl url);
}
```

所以，我们在生成okhttpclient的时候添加一个实现该接口的类后就可以很轻松的管理cookie了。

```
OkHttpClient client = new OkHttpClient.Builder()
    .cookieJar(new CookieJar() {
        private final HashMap<HttpUrl, List<Cookie>> cookieStore = new HashMap<>();

        @Override
        public void saveFromResponse(HttpUrl url, List<Cookie> cookies) {
            cookieStore.put(url, cookies);
        }

        @Override
        public List<Cookie> loadForRequest(HttpUrl url) {
            List<Cookie> cookies = cookieStore.get(url);
            return cookies != null ? cookies : new ArrayList<Cookie>();
        }
    })
    .build();
```

序列化cookie设计`SerializableOkHttpCookies`,`PersistentCookieStore`这两个类.

SerializableOkHttpCookies

主要做两件事：

- 将Cookie对象输出为ObjectStream
- 将ObjectStream序列化成Cookie对象

```
public class SerializableOkHttpCookies implements Serializable {

    private transient final Cookie cookies;
    private transient Cookie clientCookies;

    public SerializableOkHttpCookies(Cookie cookies) {
        this.cookies = cookies;
    }

    public Cookie getCookies() {
        Cookie bestCookies = cookies;
        if (clientCookies != null) {
            bestCookies = clientCookies;
        }
        return bestCookies;
    }

    private void writeObject(ObjectOutputStream out) throws IOException {
        out.writeObject(cookies.name());
        out.writeObject(cookies.value());
        out.writeLong(cookies.expiresAt());
        out.writeObject(cookies.domain());
        out.writeObject(cookies.path());
        out.writeBoolean(cookies.secure());
        out.writeBoolean(cookies.httpOnly());
        out.writeBoolean(cookies.hostOnly());
        out.writeBoolean(cookies.persistent());
    }

    private void readObject(ObjectInputStream in) throws IOException, ClassNotFoundException {
        String name = (String) in.readObject();
        String value = (String) in.readObject();
        long expiresAt = in.readLong();
        String domain = (String) in.readObject();
        String path = (String) in.readObject();
        boolean secure = in.readBoolean();
        boolean httpOnly = in.readBoolean();
        boolean hostOnly = in.readBoolean();
        boolean persistent = in.readBoolean();
        Cookie.Builder builder = new Cookie.Builder();
        builder = builder.name(name);
        builder = builder.value(value);
        builder = builder.expiresAt(expiresAt);
        builder = hostOnly ? builder.hostOnlyDomain(domain) : builder.domain(domain);
        builder = builder.path(path);
        builder = secure ? builder.secure() : builder;
        builder = httpOnly ? builder.httpOnly() : builder;
        clientCookies =builder.build();
    }
}
```

PersistentCookieStore

封装管理cookie的方法

```
public class PersistentCookieStore {
    private static final String LOG_TAG = "PersistentCookieStore";
    private static final String COOKIE_PREFS = "Cookies_Prefs";

    private final Map<String, ConcurrentHashMap<String, Cookie>> cookies;
    private final SharedPreferences cookiePrefs;


    public PersistentCookieStore(Context context) {
        cookiePrefs = context.getSharedPreferences(COOKIE_PREFS, 0);
        cookies = new HashMap<>();

        //将持久化的cookies缓存到内存中 即map cookies
        Map<String, ?> prefsMap = cookiePrefs.getAll();
        for (Map.Entry<String, ?> entry : prefsMap.entrySet()) {
            String[] cookieNames = TextUtils.split((String) entry.getValue(), ",");
            for (String name : cookieNames) {
                String encodedCookie = cookiePrefs.getString(name, null);
                if (encodedCookie != null) {
                    Cookie decodedCookie = decodeCookie(encodedCookie);
                    if (decodedCookie != null) {
                        if (!cookies.containsKey(entry.getKey())) {
                            cookies.put(entry.getKey(), new ConcurrentHashMap<String, Cookie>());
                        }
                        cookies.get(entry.getKey()).put(name, decodedCookie);
                    }
                }
            }
        }
    }

    protected String getCookieToken(Cookie cookie) {
        return cookie.name() + "@" + cookie.domain();
    }

    public void add(HttpUrl url, Cookie cookie) {
        String name = getCookieToken(cookie);

        //将cookies缓存到内存中 如果缓存过期 就重置此cookie
        if (!cookie.persistent()) {
            if (!cookies.containsKey(url.host())) {
                cookies.put(url.host(), new ConcurrentHashMap<String, Cookie>());
            }
            cookies.get(url.host()).put(name, cookie);
        } else {
            if (cookies.containsKey(url.host())) {
                cookies.get(url.host()).remove(name);
            }
        }

        //讲cookies持久化到本地
        SharedPreferences.Editor prefsWriter = cookiePrefs.edit();
        prefsWriter.putString(url.host(), TextUtils.join(",", cookies.get(url.host()).keySet()));
        prefsWriter.putString(name, encodeCookie(new SerializableOkHttpCookies(cookie)));
        prefsWriter.apply();
    }

    public List<Cookie> get(HttpUrl url) {
        ArrayList<Cookie> ret = new ArrayList<>();
        if (cookies.containsKey(url.host()))
            ret.addAll(cookies.get(url.host()).values());
        return ret;
    }

    public boolean removeAll() {
        SharedPreferences.Editor prefsWriter = cookiePrefs.edit();
        prefsWriter.clear();
        prefsWriter.apply();
        cookies.clear();
        return true;
    }

    public boolean remove(HttpUrl url, Cookie cookie) {
        String name = getCookieToken(cookie);

        if (cookies.containsKey(url.host()) && cookies.get(url.host()).containsKey(name)) {
            cookies.get(url.host()).remove(name);

            SharedPreferences.Editor prefsWriter = cookiePrefs.edit();
            if (cookiePrefs.contains(name)) {
                prefsWriter.remove(name);
            }
            prefsWriter.putString(url.host(), TextUtils.join(",", cookies.get(url.host()).keySet()));
            prefsWriter.apply();

            return true;
        } else {
            return false;
        }
    }

    public List<Cookie> getCookies() {
        ArrayList<Cookie> ret = new ArrayList<>();
        for (String key : cookies.keySet())
            ret.addAll(cookies.get(key).values());

        return ret;
    }

    /**
     * cookies 序列化成 string
     *
     * @param cookie 要序列化的cookie
     * @return 序列化之后的string
     */
    protected String encodeCookie(SerializableOkHttpCookies cookie) {
        if (cookie == null)
            return null;
        ByteArrayOutputStream os = new ByteArrayOutputStream();
        try {
            ObjectOutputStream outputStream = new ObjectOutputStream(os);
            outputStream.writeObject(cookie);
        } catch (IOException e) {
            Log.d(LOG_TAG, "IOException in encodeCookie", e);
            return null;
        }

        return byteArrayToHexString(os.toByteArray());
    }

    /**
     * 将字符串反序列化成cookies
     *
     * @param cookieString cookies string
     * @return cookie object
     */
    protected Cookie decodeCookie(String cookieString) {
        byte[] bytes = hexStringToByteArray(cookieString);
        ByteArrayInputStream byteArrayInputStream = new ByteArrayInputStream(bytes);
        Cookie cookie = null;
        try {
            ObjectInputStream objectInputStream = new ObjectInputStream(byteArrayInputStream);
            cookie = ((SerializableOkHttpCookies) objectInputStream.readObject()).getCookies();
        } catch (IOException e) {
            Log.d(LOG_TAG, "IOException in decodeCookie", e);
        } catch (ClassNotFoundException e) {
            Log.d(LOG_TAG, "ClassNotFoundException in decodeCookie", e);
        }

        return cookie;
    }

    /**
     * 二进制数组转十六进制字符串
     *
     * @param bytes byte array to be converted
     * @return string containing hex values
     */
    protected String byteArrayToHexString(byte[] bytes) {
        StringBuilder sb = new StringBuilder(bytes.length * 2);
        for (byte element : bytes) {
            int v = element & 0xff;
            if (v < 16) {
                sb.append('0');
            }
            sb.append(Integer.toHexString(v));
        }
        return sb.toString().toUpperCase(Locale.US);
    }

    /**
     * 十六进制字符串转二进制数组
     *
     * @param hexString string of hex-encoded values
     * @return decoded byte array
     */
    protected byte[] hexStringToByteArray(String hexString) {
        int len = hexString.length();
        byte[] data = new byte[len / 2];
        for (int i = 0; i < len; i += 2) {
            data[i / 2] = (byte) ((Character.digit(hexString.charAt(i), 16) << 4) + Character.digit(hexString.charAt(i + 1), 16));
        }
        return data;
    }
}
```