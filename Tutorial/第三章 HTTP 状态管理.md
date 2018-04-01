# 第三章　HTTP 状态管理

> 标签: cookies
---

最初，HTTP被设计为无状态，请求/响应的协议。协议本身没有特别规定请求/响应模式下的状态会话。然而，随着HTTP协议的普及，诸如，电子商务系统，越来越多的应用开始采用该技术，因此，对状态管理的支持变得十分必要。
## 3.1. HTTP cookies
HTTP cookies是一个标识，或者状态信息的小包，HTTP代理和目标服务器可以用它来交换，以维持连接会话。Netscape工程师曾经把它成为"Magic cookies"，这样cookies的名称就沿用至今。
HttpClient 使用 cookie 接口代表 Cookie的抽象概念。Cookies最简单的格式就是K/V键值对。通常, HTTP cookie 还包含许多属性, 比如，该域是有效的，该路径指定此 cookie 应用到的源服务器上的 url 子集, 以及 cookie 最大有效时间周期。
SetCookie 接口表示由源服务器发送到 HTTP 代理的Set-Cookie 响应头, 以保持会话状态。
ClientCookie接口继承自Cookie接口，并提供客户端额外的一些功能，例如检索cookies属性的能力与原服务器指定的一样。这对于生成 cookie 头非常重要, 因为某些 cookie 规范要求 cookie 头只有在设置 cookie 标头中指定时才应包含某些属性。
## 3.2. Cookie 规范
CookieSpec接口代表COOKIE管理规范。该规范将在以下方面加强：
* Set-Cookie头解析规则
* 已解析Cookie的验证规则
* 格式化Cookie头，头包含原主机地址，端口，路径。
HttpClient自带如下若干CookieSpec实现：
* 严格标准 状态管理策略符合 RFC 6265 (第 4 节) 定义的表现良好的配置文件的语法和语义。
* 标准     状态管理策略符合 RFC 6265 定义的更为宽松的配置文件(第4节),旨在与非良好表现的配置文件的现有服务器进行互操作。
* Netscape草案(废弃)  此策略符合网景发布的原始规范草案。除非真的需要与遗留代码兼容, 否则应避免此情况。
* RFC 2965 (废弃): 符合 RFC2965 定义废弃状态管理规范的状态管理策略。不推荐在新应用程序中使用。
* RFC 2109 (废弃): 符合 RFC2109 定义废弃状态管理规范的状态管理策略。不推荐在新应用程序中使用。
* 浏览器兼容 (废弃):
* 默认:
* 忽略cookies: 忽略所有cookies
强烈建议在新应用程序中使用标准或严格标准的策略。过时的规范只应用于与遗留系统的兼容性。在 HttpClient 的下一个主要版本中, 将删除对过时规范的支持。


## 3.3. 选择cookie策略

```
RequestConfig globalConfig = RequestConfig.custom()
        .setCookieSpec(CookieSpecs.DEFAULT)
        .build();
CloseableHttpClient httpclient = HttpClients.custom()
        .setDefaultRequestConfig(globalConfig)
        .build();
RequestConfig localConfig = RequestConfig.copy(globalConfig)
        .setCookieSpec(CookieSpecs.STANDARD_STRICT)
        .build();
HttpGet httpGet = new HttpGet("/");
httpGet.setConfig(localConfig);
```
## 3.4. 定制cookie策略
```
PublicSuffixMatcher publicSuffixMatcher = PublicSuffixMatcherLoader.getDefault();

Registry<CookieSpecProvider> r = RegistryBuilder.<CookieSpecProvider>create()
        .register(CookieSpecs.DEFAULT,
                new DefaultCookieSpecProvider(publicSuffixMatcher))
        .register(CookieSpecs.STANDARD,
                new RFC6265CookieSpecProvider(publicSuffixMatcher))
        .register("easy", new EasySpecProvider())
        .build();

RequestConfig requestConfig = RequestConfig.custom()
        .setCookieSpec("easy")
        .build();

CloseableHttpClient httpclient = HttpClients.custom()
        .setDefaultCookieSpecRegistry(r)
        .setDefaultRequestConfig(requestConfig)
        .build();
```
## 3.5. Cookie持久性
```
// Create a local instance of cookie store
CookieStore cookieStore = new BasicCookieStore();
// Populate cookies if needed
BasicClientCookie cookie = new BasicClientCookie("name", "value");
cookie.setDomain(".mycompany.com");
cookie.setPath("/");
cookieStore.addCookie(cookie);
// Set the store
CloseableHttpClient httpclient = HttpClients.custom()
        .setDefaultCookieStore(cookieStore)
        .build();
```
## 3.6. HTTP状态管理和执行上下文

```
CloseableHttpClient httpclient = <...>

Lookup<CookieSpecProvider> cookieSpecReg = <...>
CookieStore cookieStore = <...>

HttpClientContext context = HttpClientContext.create();
context.setCookieSpecRegistry(cookieSpecReg);
context.setCookieStore(cookieStore);
HttpGet httpget = new HttpGet("http://somehost/");
CloseableHttpResponse response1 = httpclient.execute(httpget, context);
<...>
// Cookie origin details
CookieOrigin cookieOrigin = context.getCookieOrigin();
// Cookie spec used
CookieSpec cookieSpec = context.getCookieSpec();
```
