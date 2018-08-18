[TOC]

*最新版本 (mm/dd/yy): 10/12/2012*

翻译自[CORS RequestPreflighScrutiny](https://www.owasp.org/index.php/CORS_RequestPreflighScrutiny "CORS RequestPreflighScrutiny")

# 漏洞描述
CORS表示跨源资源共享，它作为一种功能提供了以下可能性

- 将资源公开给所有或受限域的Web应用程序
- 一个Web客户端，用于对源域以外的其他域的资源进行AJAX请求

本文将重点介绍CORS W3C规范提出的HTTP请求预检功能，以及（主要）如何针对试图绕过预检流程的CORS HTTP请求在Web应用程序端设置保护。

# 请求预检流程概述
为了不重复说明，并且Mozilla wiki有一篇关于CORS的精彩介绍文章，您可以使用下面的链接阅读关于该流程的描述
[https://developer.mozilla.org/en-US/docs/HTTP_access_control#Preflighted_requests](https://developer.mozilla.org/en-US/docs/HTTP_access_control#Preflighted_requests "https://developer.mozilla.org/en-US/docs/HTTP_access_control#Preflighted_requests")

# 风险
- 请求预检的目的是确保HTTP请求不会数据造成不良影响，因此，使用第一个请求，其中浏览器描述稍后将发送的最终HTTP请求
- 主要的风险是（从web应用端的角度）请求预检过程完全被客户端（浏览器）所掌控，然后Web应用程序始终遵循请求预检过程.....
- 用户可以创建一个没有经过先前预检请求的最终请求（使用类似Curl,OWASO Zap Proxy的工具）然后绕过请求预检以对数据采取不安全的行为

# 对策

我们必须确保请求预检的过程在服务端是合规的，为了实现它，我们将使用JEE Web Filter，它将使用这些步骤检查每个CORS请求：
- 第一步： 确认传入请求的类型
- 第二步： 按照使用临时缓存的类型处理请求，以保持进程的预检步骤的状态。

## 示例实现：过滤器实现

```java
import java.io.IOException;
import java.util.Collections;
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
 * 样本过滤器实施审查CORS“请求预检”过程之后是有关的HTTP请求.<br/>
 * 
 *该实现依赖EHCache API，因为它使用缓存来存储预先发送的请求
 * 
 * 这里假设所有的CORS资源都按上下文路径分组 "/cors/".
 * 
 * 参照 "https://developer.mozilla.org/en-US/docs/HTTP_access_control#Simple_requests"
 */
@SuppressWarnings("static-method")
@WebFilter("/cors/*")
public class CORSRequestPreflightProcessScrutiny implements Filter {

	// 配置过滤器
	private FilterConfig filterConfig = null;

	// 在此期间，我们将请求保留为正确地遵循请求预检过程
	private int requestPreflightCacheDelayInSeconds = 60;

	// 缓存用于缓存预先发送的请求
	private Cache requestPreflightCache = null;

	/**
	 * {@inheritDoc}
	 * 
	 * @see Filter#init(FilterConfig)
	 */
	@Override
	public void init(FilterConfig fConfig) throws ServletException {
		// 获取过滤器配置
		this.filterConfig = fConfig;
		// 初始化预先设置的请求专用缓存，每个项目的缓存设置X分钟过期延迟
		PersistenceConfiguration cachePersistence = new PersistenceConfiguration();
		cachePersistence.strategy(PersistenceConfiguration.Strategy.NONE);
		CacheConfiguration cacheConfig = new CacheConfiguration().memoryStoreEvictionPolicy(MemoryStoreEvictionPolicy.FIFO).eternal(false)
                .timeToLiveSeconds(this.requestPreflightCacheDelayInSeconds)
                .statistics(false).diskExpiryThreadIntervalSeconds(this.requestPreflightCacheDelayInSeconds / 2)
                .persistence(cachePersistence).maxEntriesLocalHeap(10000).logging(false);
		cacheConfig.setName("PreflightedRequestsCacheConfig");
		this.requestPreflightCache = new Cache(cacheConfig);
		this.requestPreflightCache.setName("PreflightedRequestsCache");
		CacheManager.getInstance().addCache(this.requestPreflightCache);
	}

	/**
	 * {@inheritDoc}
	 * 
	 * @see Filter#destroy()
	 */
	@Override
	public void destroy() {
		// 删除缓存
		CacheManager.getInstance().removeCache("PreflightedRequestsCache");
	}

	/**
	 * {@inheritDoc}
	 * 
	 * @see Filter#doFilter(ServletRequest, ServletResponse, FilterChain)
	 */
	@Override
	public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
		HttpServletRequest htReq = ((HttpServletRequest) request);
		HttpServletResponse htResp = ((HttpServletResponse) response);
		int accessDeniedHttpResponse = HttpServletResponse.SC_FORBIDDEN;
		String accessControlAllowMethods = "GET, POST";
		String accessControlAllowOrigin = "http://foo.example";

		/* 步骤 01 : 确认传入请求的类型 */
		CORSRequestPreflightType corsReqType = determineRequestType(htReq);

		/* 步骤 02 : 根据请求类型执行流程 */
		switch (corsReqType) {
		// --客户端发送HTTP请求以预检另一个“复杂”请求
		case REQUEST_FOR_PREFLIGHT: {
			CORSRequestPreflightData corsReq = new CORSRequestPreflightData(htReq);
			// ----步骤 2a: 检查预检的请求是否有效，如果没有，发送拒绝访问（不要提供关于不良请求根本原因的信息）
			if (corsReq.getOrigin().trim().isEmpty() || corsReq.getExpectedMethod().trim().isEmpty()) {
				traceInvalidRequestDetected(htReq);
				htResp.reset();
				htResp.sendError(accessDeniedHttpResponse);
				// 退出过滤器：使用'return'算法中断以避免多个IF语句并增强可读性...
				return;

			}
			// ----步骤 2b: 将预检请求数据存储在缓存中，以将请求保持（标记）为正确地遵循请求预检过程
			Element cachedRequest = new Element(CORSUtils.buildRequestCacheIdentifier(htReq), corsReq);
			this.requestPreflightCache.put(cachedRequest);
			// ----步骤 2c: 返回相应的响应 - 这部分应该根据应用程序的具体限制进行定制
			htResp.reset();
			htResp.setStatus(HttpServletResponse.SC_OK);
			htResp.setHeader("Access-Control-Allow-Origin", accessControlAllowOrigin);
			htResp.setHeader("Access-Control-Allow-Methods", accessControlAllowMethods);
			if (!corsReq.getExpectedCustomHeaders().isEmpty()) {
				htResp.setHeader("Access-Control-Allow-Headers", corsReq.getExpectedCustomHeaders().toString().replaceFirst("\\[", "").replaceFirst("\\]", "").trim());
			}
			htResp.setIntHeader("Access-Control-Max-Age", (this.requestPreflightCacheDelayInSeconds));
			break;
		}
		// --客户端发送的普通HTTP请求需要预检，即预检流程中的“复杂”请求
		case COMPLEX_REQUEST: {
			String rid = CORSUtils.buildRequestCacheIdentifier(htReq);
			// ----步骤 2a: 检查当前请求是否有预先登记的请求缓存条目
			if (this.requestPreflightCache.get(rid) == null) {
				traceInvalidRequestDetected(htReq);
				htResp.reset();
				htResp.sendError(accessDeniedHttpResponse);
				// 退出过滤器：使用'return'算法中断以避免多个IF语句并增强可读性....
				return;
			}
			// ----步骤 2b: 检查在预检请求期间声明的预检信息是否与关键信息上的当前请求匹配
			CORSRequestPreflightData corsPreflightReq = (CORSRequestPreflightData) this.requestPreflightCache.get(rid).getValue();
			String origin = CORSUtils.retrieveHeader("Origin", htReq);
			List<String> customHeaders = CORSUtils.retrieveCustomHeaders(htReq);
			boolean match = false;
			// ------首先比较“Origin”HTTP标头（根据实用方法impl。用于检索标头引用不能为空）...
			if (origin.equals(corsPreflightReq.getOrigin())) {
				// ------继续使用HTTP方法...
				if (accessControlAllowMethods.contains(htReq.getMethod()) && htReq.getMethod().equals(corsPreflightReq.getExpectedMethod())) {
					// ------使用自定义HTTP标头完成（使用一种方法避免手动迭代集合以提高速度）...
					if (customHeaders.size() == corsPreflightReq.getExpectedCustomHeaders().size()) {
						Collections.sort(customHeaders);
						Collections.sort(corsPreflightReq.getExpectedCustomHeaders());
						if (customHeaders.toString().equals(corsPreflightReq.getExpectedCustomHeaders().toString())) {
							match = true;
						}
					}
				}
			}
			if (match) {
				// 继续链接到下一个过滤器
				chain.doFilter(request, response);
			} else {
				traceInvalidRequestDetected(htReq);
				htResp.reset();
				htResp.sendError(accessDeniedHttpResponse);
			}
			break;
		}
		// --客户端发送的普通HTTP请求不需要预检，即预检过程中的“简单”请求s
		case SIMPLE_REQUEST: {
			// 继续链接到下一个过滤器
			chain.doFilter(request, response);
			break;
		}
		// --未知的HTTP请求类型 !
		default: {
			traceInvalidRequestDetected(htReq);
			htResp.reset();
			htResp.sendError(accessDeniedHttpResponse);
			break;
		}
		}
	}

	/**
	 * 确定传入请求类型的内部方法
	 * 
	 * @参数 htReq HTTP 请求
	 * @将该类型作为枚举项返回
	 */
	private CORSRequestPreflightType determineRequestType(HttpServletRequest htReq) {
		CORSRequestPreflightType type = CORSRequestPreflightType.UNKNOWN;

		if ("OPTIONS".equalsIgnoreCase(htReq.getMethod())) {
			type = CORSRequestPreflightType.REQUEST_FOR_PREFLIGHT;
		} else {
			if (!CORSUtils.retrieveCustomHeaders(htReq).isEmpty()) {
				type = CORSRequestPreflightType.COMPLEX_REQUEST;
			} else if ("POST".equalsIgnoreCase(htReq.getMethod()) && !"application/x-www-form-urlencoded".equalsIgnoreCase(htReq.getContentType())
					&& !"multipart/form-data".equalsIgnoreCase(htReq.getContentType()) && !"text/plain".equalsIgnoreCase(htReq.getContentType())) {
				type = CORSRequestPreflightType.COMPLEX_REQUEST;
			} else if ("HEAD".equalsIgnoreCase(htReq.getMethod()) || "DELETE".equalsIgnoreCase(htReq.getMethod()) || "PUT".equalsIgnoreCase(htReq.getMethod())
					|| "TRACE".equalsIgnoreCase(htReq.getMethod()) || "CONNECT".equalsIgnoreCase(htReq.getMethod())) {
				type = CORSRequestPreflightType.COMPLEX_REQUEST;
			} else {
				type = CORSRequestPreflightType.SIMPLE_REQUEST;
			}

		}

		return type;
	}

	/**
	 * 将检测到的无效请求数据添加到跟踪日志中的方法
	 * 
	 * @参数 htReq：检测到无效的请求
	 */
	private void traceInvalidRequestDetected(HttpServletRequest htReq) {
		// 自定义追踪...
		this.filterConfig.getServletContext().log("---[CORS Invalid request detected]---");
		this.filterConfig.getServletContext().log(String.format("Client Address : %s", htReq.getRemoteAddr()));
		this.filterConfig.getServletContext().log(String.format("Target URL     : %s", htReq.getRequestURL()));
		this.filterConfig.getServletContext().log(String.format("Query String   : %s", htReq.getQueryString()));
		this.filterConfig.getServletContext().log(String.format("HTTP Method    : %s", htReq.getMethod()));
		// Print more request useful data.....
		this.filterConfig.getServletContext().log("-------------------------------------");
	}
}
```

## 示例实现：过滤器所使用的实用程序类

```java
import java.util.ArrayList;
import java.util.Enumeration;
import java.util.List;

import javax.servlet.http.HttpServletRequest;

/**
 * 用于CORS数据检索和处理的实用程序类
 */
public class CORSUtils {

	/**
	 * 检索HTTP请求自定义标头列表的方法。
	 * 
	 * @参数httpRequest：源HTTP请求
	 * @返回自定义标头列表（转换为大写以避免大小写问题）
	 */
	public static List<String> retrieveCustomHeaders(HttpServletRequest httpRequest) {
		List<String> xHeaders = new ArrayList<String>();
		String name = null;

		if (httpRequest == null) {
			throw new IllegalArgumentException("HTTP Request cannot be null !");
		}

		Enumeration<String> headers = httpRequest.getHeaderNames();
		while (headers.hasMoreElements()) {
			name = headers.nextElement().toUpperCase().trim();
			if (name.startsWith("X-")) {
				xHeaders.add(name.trim());
			}
		}

		return xHeaders;

	}

	/**
	 * 从源HTTP请求中检索HTTP 标头值的方法 <br/>
	 * 管理标头名称大小写问题并只取第一个值
	 * 
	 * @参数 headerName：HTTP名称
	 * @参数httpRequest：源HTTP请求
	 * @返回HTTP标头值，如果找不到返回“”
	 */
	public static String retrieveHeader(String headerName, HttpServletRequest httpRequest) {
		String value = "";
		String name = null;

		if (httpRequest == null) {
			throw new IllegalArgumentException("HTTP Request cannot be null !");
		}
		if ((headerName == null) || headerName.trim().isEmpty()) {
			throw new IllegalArgumentException("HTTP header name be null !");
		}

		Enumeration<String> headers = httpRequest.getHeaderNames();
		while (headers.hasMoreElements()) {
			name = headers.nextElement();
			if (name.trim().equalsIgnoreCase(headerName)) {
				value = httpRequest.getHeader(name);
				break;
			}
		}

		return value;
	}

	/**
	 * 将请求的标识符构建到预检请求缓存中
	 * 
	 * @参数 httpRequest：源HTTP请求
	 * @将ID作为字符串返回
	 */
	public static String buildRequestCacheIdentifier(HttpServletRequest httpRequest) {
		return (httpRequest.getRemoteAddr() + "_" + httpRequest.getRequestURI()).trim();
	}
}
```

## 示例实现：用于存储预检请求密钥信息的Pojo类

```java
import java.io.Serializable;
import java.util.ArrayList;
import java.util.List;

import javax.servlet.http.HttpServletRequest;

/**
 * 用来存储有关CORS预检请求信息的类
 */
@SuppressWarnings("serial")
public class CORSRequestPreflightData implements Serializable {

	/** 最终的HTTP请求预期方法 */
	private String expectedMethod = null;
	/** F最终的HTTP请求预期自定义标头 */
	private List<String> expectedCustomHeaders = null;

	/** 当前的HTTP请求的URI*/
	private String uri = null;
	/** 当前HTTP请求源标头 */
	private String origin = null;
	/** 当前发送者IP地址 */
	private String sender = null;

	/**
	 * 构造函数
	 * 
	 * @参数httpRequest：源HTTP请求
	 */
	public CORSRequestPreflightData(HttpServletRequest httpRequest) {
		super();
		String tmp = null;
		if (httpRequest == null) {
			throw new IllegalArgumentException("HTTP request cannot be null !");
		}
		this.sender = httpRequest.getRemoteAddr();
		this.uri = httpRequest.getRequestURI();
		this.origin = CORSUtils.retrieveHeader("Origin", httpRequest);
		this.expectedMethod = CORSUtils.retrieveHeader("Access-Control-Request-Method", httpRequest);
		tmp = CORSUtils.retrieveHeader("Access-Control-Request-Headers", httpRequest);
		if (!tmp.trim().isEmpty()) {
			this.expectedCustomHeaders = new ArrayList<String>();
			String[] hs = tmp.split(",");
			for (String h : hs) {
				if ((h != null) && !h.trim().isEmpty()) {
					this.expectedCustomHeaders.add(h.toUpperCase().trim());
				}
			}
		}
	}

	/**
	 * 获取
	 * 
	 * @返回预期的方法
	 */
	public String getExpectedMethod() {
		return this.expectedMethod;
	}

	/**
	 * 获取
	 * 
	 * @返回预期的自定义标头
	 */
	public List<String> getExpectedCustomHeaders() {
		return this.expectedCustomHeaders;
	}

	/**
	 * 获取
	 * 
	 * @返回uri
	 */
	public String getUri() {
		return this.uri;
	}

	/**
	 * 获取
	 * 
	 * @返回源
	 */
	public String getOrigin() {
		return this.origin;
	}

	/**
	 * 获取
	 * 
	 * @返回发送者
	 */
	public String getSender() {
		return this.sender;
	}
}
```

## 示例实现：用于表示不同的CORS请求类型的枚举

```java
/**
 * 枚举不同的CORS“请求预检”HTTP请求类型。 
 */
public enum CORSRequestPreflightType {

	/** 客户端发送HTTP请求以预检另一个“复杂”请求 */
	REQUEST_FOR_PREFLIGHT,

	/** 客户端发送的普通HTTP请求需要预检，即预检流程中的“复杂”请求 */
	COMPLEX_REQUEST,

	/** 客户端发送的普通HTTP请求不需要预检，即预检过程中的“简单”请求 */
	SIMPLE_REQUEST,

	/** 无法确定的请求类型 */
	UNKNOWN;
}
```

注意：W3AF审计工具（[http://w3af.org](http://w3af.org "http://w3af.org"))包含了自动审核Web应用程序的插件，以检查应用端是否实现了这种对策。将这种类型的工具纳入Web应用程序开发流程以便执行定期的自动一级检查（不要取代手动审计，而且必须定期执行手动审计）是非常有用的做法。

# 相关信息

- W3C Specification : [http://www.w3.org/TR/cors/](http://www.w3.org/TR/cors/ "http://www.w3.org/TR/cors/")

- Mozilla Wiki : [https://developer.mozilla.org/en-US/docs/HTTP_access_control](https://developer.mozilla.org/en-US/docs/HTTP_access_control "https://developer.mozilla.org/en-US/docs/HTTP_access_control")

- Wikipedia : [http://en.wikipedia.org/wiki/Cross-origin_resource_sharing](Wikipedia : http://en.wikipedia.org/wiki/Cross-origin_resource_sharing "http://en.wikipedia.org/wiki/Cross-origin_resource_sharing")


