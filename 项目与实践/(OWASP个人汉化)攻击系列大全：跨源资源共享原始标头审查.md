[TOC]

*最新版本 (mm/dd/yy): 08/16/2013*

翻译自[CORS OriginHeaderScrutiny](https://www.owasp.org/index.php/CORS_OriginHeaderScrutiny "CORS OriginHeaderScrutiny")

# 漏洞描述
CORS表示跨源资源共享，它作为一种功能提供了以下可能性

- 将资源公开给所有或受限域的Web应用程序
- 一个Web客户端，用于对源域以外的其他域的资源进行AJAX请求

本文将重点讨论原始标头在Web客户端和Web应用程序之间交换的作用。
基本过程由以下步骤组成（从Mozilla Wiki发出的简单HTTP请求/响应）

## 第一步：web客户端发送请求来从其他域获取资源

```
GET /resources/public-data/ HTTP/1.1
Host: bar.other
User-Agent: Mozilla/5.0 (Macintosh; U; Intel Mac OS X 10.5; en-US; rv:1.9.1b3pre) Gecko/20081130 Minefield/3.1b3pre
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-us,en;q=0.5
Accept-Encoding: gzip,deflate
Accept-Charset: ISO-8859-1,utf-8;q=0.7,*;q=0.7
Connection: keep-alive
Referer: http://foo.example/examples/access-control/simpleXSInvocation.html
Origin: http://foo.example

[Request Body]
```

Web客户端通知使用HTTP请求头“Origin”的源域

## 第二步：web应用程序响应请求

```
HTTP/1.1 200 OK
Date: Mon, 01 Dec 2008 00:23:53 GMT
Server: Apache/2.0.61 
Keep-Alive: timeout=2, max=100
Connection: Keep-Alive
Transfer-Encoding: chunked
Content-Type: application/xml
Access-Control-Allow-Origin: *

[Response Body]
```

Web应用程序使用HTTP响应标头`Access-Control-Allow-Origin`通知Web客户端允许的域。该标头可以通过包含一个'*'来表示所有的域都被允许或者一个指定的域来指示指定的允许域。

## 第三步：web客户端执行web应用程序的响应
根据CORS W3C规范，如果允许Web客户端访问响应数据，则Web客户端（通常是浏览器）使用Web应用程序响应的HTTP标头`Access-Control-Allow-Origin`来确定。

# 风险
提醒：在本文中我们将重点放在Web应用程序端，因为它是我们控制最多的唯一部分。
风险在于web应用端可以在源HTTP请求头中放入任何值来强制web应用端提供目标资源的内容。在浏览器为Web客户端的情况下，标头的值不仅可以由浏览器提供，更可以被其他"web客户端"（类似Curl\Wget\Burp suite..)来修改、覆写"Origin"字段的值...

# 对策
## 对策A:使用CORS请求认证
在此选项中，我们对所访问的资源启用身份验证，并要求将用户/应用程序凭证与CORS请求一起传递。
如果暴露的CORS资源被分类为敏感（CORS的暴露是强制性的），这是一个很好的选择，但如果目的只是为了确保请求发起者确实是允许的（避免恶意请求）之一，那么该选项有一些缺陷，其中包括：
- 目标应用程序必须管理用户（或者应用）的凭证储存库，包括密码有效期，密码重置，暴力破解防护，账户锁定/解锁等功能
- 客户端必须存储（以一种安全的方式）凭证来使用
- 客户端应用程序必须在HTTP请求中管理/配置证书传输，以便只有在CORS请求发送到目标应用程序的情况下才会发送证书。

## 对策B:我们可以在服务端审查源标头的值
在这个选项中，目标是在CORS HTTP请求/响应交换过程的第1步和第2步之间进行操作（见上文）。
为了达成目的，我们将使用JEE web Filter来确保接下来传入的HTTP CROS请求具有如下特点
- 只有一个非空的原始标头实例
- 只有一个非空的host标头实例
- 原始标头的值存在于允许的内部域列表（白名单）中。 当我们在CORS HTTP请求/响应交换过程的第2步之前采取行动时，允许的域列表尚未提供给客户端
- 缓存发送者IP一个小时、如果发送者发送了一次不在白名单中的原始域，则对所有请求将返回一个HTTP 403响应（拖延将导致域名猜测）

即使我们使用上面的方法，也不可能100％确定请求来自一个预期的客户端应用程序，原因是
- HTTP请求的所有信息都可被伪造
- 浏览器（或其他工具）发送HTTP请求，然后我们有权访问客户端IP地址。

在企业内部应用程序通信中，可以在客户端IP范围上添加检查。“企业对企业”通信为每个部分提供了向其他部分指示其应用程序将使用的IP范围的可能性。

## 简单示例：过滤器实现
```java
import java.io.IOException;
import java.util.ArrayList;
import java.util.Collections;
import java.util.Enumeration;
import java.util.List;

import javax.servlet.Filter;
import javax.servlet.FilterChain;
import javax.servlet.FilterConfig;
import javax.servlet.ServletException;
import javax.servlet.ServletRequest;
import javax.servlet.ServletResponse;
import javax.servlet.annotation.WebFilter;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import net.sf.ehcache.Cache;
import net.sf.ehcache.CacheManager;
import net.sf.ehcache.Element;
import net.sf.ehcache.config.CacheConfiguration;
import net.sf.ehcache.config.PersistenceConfiguration;
import net.sf.ehcache.store.MemoryStoreEvictionPolicy;

/**
 * 将样例过滤器实现用于审查CORS“Origin”HTTP标头.<br/>
 * 
 * 这个实现依赖于EHCache API，因为<br/>
 * 它将缓存用于列入黑名单的客户端IP，以提高性能.
 * 
 * 这里假设所有的CORS资源都按上下文路径分组 "/cors/".
 * 
 */
@WebFilter("/cors/*")
public class CORSOriginHeaderScrutiny implements Filter {

	/** 过滤器配置 */
	@SuppressWarnings("unused")
	private FilterConfig filterConfig = null;

	/** 用于缓存列入黑名单的客户端（请求发件人）IP地址的缓存 */
	private Cache blackListedClientIPCache = null;

	/** 允许域访问资源（白名单） */
	private List<String> allowedDomains = new ArrayList<String>();

	/**
	 * {@inheritDoc}
	 * 
	 * @see Filter#init(FilterConfig)
	 */
	@Override
	public void init(FilterConfig fConfig) throws ServletException {
		// 获取过滤器配置
		this.filterConfig = fConfig;
		// 初始化客户端IP地址专用高速缓存，每个项目的高速缓存60分钟过期
		PersistenceConfiguration cachePersistence = new PersistenceConfiguration();
		cachePersistence.strategy(PersistenceConfiguration.Strategy.NONE);
		CacheConfiguration cacheConfig = new CacheConfiguration().memoryStoreEvictionPolicy(MemoryStoreEvictionPolicy.FIFO)
		.eternal(false)
		.timeToLiveSeconds(3600)
		.diskExpiryThreadIntervalSeconds(450)
		.persistence(cachePersistence)
		.maxEntriesLocalHeap(10000)
		.logging(false);
		cacheConfig.setName("BlackListedClientsCacheConfig");
		this.blackListedClientIPCache = new Cache(cacheConfig);
		this.blackListedClientIPCache.setName("BlackListedClientsCache");
		CacheManager.getInstance().addCache(this.blackListedClientIPCache);
		// 加载域允许使用白名单（例如，在这里硬编码）
		this.allowedDomains.add("http://www.html5rocks.com");
		this.allowedDomains.add("https://www.mydomains.com");
	}

	/**
	 * {@inheritDoc}
	 * 
	 * @see Filter#destroy()
	 */
	@Override
	public void destroy() {
		// 移除缓存
		CacheManager.getInstance().removeCache("BlackListedClientsCache");
	}

	/**
	 * {@inheritDoc}
	 * 
	 * @see Filter#doFilter(ServletRequest, ServletResponse, FilterChain)
	 */
	@Override
	public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) 
        throws IOException, ServletException {
		HttpServletRequest httpRequest = ((HttpServletRequest) request);
		HttpServletResponse httpResponse = ((HttpServletResponse) response);
		List<String> headers = null;
		boolean isValid = false;
		String origin = null;
		String clientIP = httpRequest.getRemoteAddr();

		/* 步骤 0 : 检查黑名单中是否存在客户端IP */
		if (this.blackListedClientIPCache.isKeyInCache(clientIP)) {
			// 返回HTTP错误，没有任何有关请求拒绝原因的信息！
			httpResponse.sendError(HttpServletResponse.SC_FORBIDDEN);
			// 在这里添加跟踪
			// ....
			// 快速退出
			return;
		}

		/* 步骤 1 : 检查我们是否只有一个“Origin”标头的非空实例 */
		headers = CORSOriginHeaderScrutiny.enumAsList(httpRequest.getHeaders("Origin"));
		if ((headers == null) || (headers.size() != 1)) {
			// 如果我们运行到这，这意味着我们的“Origin”标头有多个实例
			// 将客户端IP地址添加到黑名单
			addClientToBlacklist(clientIP);
			// 返回HTTP错误，没有任何有关请求拒绝原因的信息！
			httpResponse.sendError(HttpServletResponse.SC_FORBIDDEN);
			// 在这里添加跟踪
			// ....
			// 快速退出
			return;
		}
		origin = headers.get(0);

		/* 步骤 2 : 检查我们是否只有一个“Host”标头的非空实例 */
		headers = CORSOriginHeaderScrutiny.enumAsList(httpRequest.getHeaders("Host"));
		if ((headers == null) || (headers.size() != 1)) {
			// 如果我们运行到这，这意味着我们的“Host”标头有多个实例
			// 将客户端IP地址添加到黑名单
			addClientToBlacklist(clientIP);
			// 返回HTTP错误，没有任何有关请求拒绝原因的信息
			httpResponse.sendError(HttpServletResponse.SC_FORBIDDEN);
			// 在这里添加跟踪
			// ....
			// 快速退出
			return;
		}

		/* 步骤 3 : 执行分析 - 需要Origin标头 */
		if ((origin != null) && !"".equals(origin.trim())) {
			if (this.allowedDomains.contains(origin)) {
				// 检查来源是否在允许的域中
				isValid = true;
			} else {
				// 将客户端IP地址添加到黑名单
				addClientToBlacklist(clientIP);
				isValid = false;
				// 在这里添加跟踪
				// ....
			}
		}

		/* 步骤 4 : 完成下一步请求 */
		if (isValid) {
			// 分析完成，然后沿着过滤器链传递请求
			chain.doFilter(request, response);
		} else {
			// 返回HTTP错误，没有任何有关请求拒绝原因的信息！
			httpResponse.sendError(HttpServletResponse.SC_FORBIDDEN);
		}
	}

	/**
	 * 客户端黑名单
	 * 
	 * @param clientIP Client IP address
	 */
	private void addClientToBlacklist(String clientIP) {
		// 将客户端IP地址添加到黑名单
		Element cacheElement = new Element(clientIP, clientIP);
		this.blackListedClientIPCache.put(cacheElement);
	}

	/**
	 * 将枚举转换为列表
	 * 
	 * @param tmpEnum Enumeration to convert
	 * @return list of string or null is input enumeration is null
	 */
	private static List<String> enumAsList(Enumeration<String> tmpEnum) {
		if (tmpEnum != null) {
			return Collections.list(tmpEnum);
		}
		return null;
	}
}
```

注意：W3AF审计工具（[http://w3af.org](http://w3af.org "http://w3af.org"))包含了自动审核Web应用程序的插件，以检查应用端是否实现了这种对策。将这种类型的工具纳入Web应用程序开发流程以便执行定期的自动一级检查（不要取代手动审计，而且必须定期执行手动审计）是非常有用的做法。

# 相关链接
- W3C Specification : [http://www.w3.org/TR/cors/](http://www.w3.org/TR/cors/ "http://www.w3.org/TR/cors/")

- Mozilla Wiki : [https://developer.mozilla.org/en-US/docs/HTTP_access_control](https://developer.mozilla.org/en-US/docs/HTTP_access_control "https://developer.mozilla.org/en-US/docs/HTTP_access_control")

- Wikipedia : [http://en.wikipedia.org/wiki/Cross-origin_resource_sharing](http://en.wikipedia.org/wiki/Cross-origin_resource_sharing "http://en.wikipedia.org/wiki/Cross-origin_resource_sharing")

- CORS Abuse : [http://blog.secureideas.com/2013/02/grab-cors-light.html](http://blog.secureideas.com/2013/02/grab-cors-light.html "http://blog.secureideas.com/2013/02/grab-cors-light.html")