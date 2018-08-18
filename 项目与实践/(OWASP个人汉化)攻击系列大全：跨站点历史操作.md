[TOC]

*最新版本 (mm/dd/yy): 10/10/2017*

翻译自[Cross Site History Manipulation (XSHM)](https://www.owasp.org/index.php/Cross_Site_History_Manipulation_(XSHM) "Cross Site History Manipulation (XSHM)")

# 漏洞介绍
跨站点历史操作（XSHM）是一种SOP（同源策略）安全漏洞。SOP是现代浏览器最重要的安全概念。SOP意味着根据设计来自不同来源的网页不能相互通信。跨站点历史操作漏洞是基于客户端浏览器历史记录对象未按每个站点正确分区的事实。操纵浏览器历史记录可能导致SOP失效，允许双向CSRF和其他利用，例如：窃取用户隐私，登录状态检测，资源遍历，敏感信息推断，用户活动跟踪和URL参数窃取。

# 风险因素
通过浏览器的历史记录让SOP失效和盗取用户隐私成为可能。将CSRF与历史操作结合使用，不仅可以实现完整性，还可以保密。可以访问来自不同来源的反馈并实现跨站点条件泄漏。
基于XSHM技术可能产生以下攻击向量：

- 跨站点条件泄漏
- 登陆检测
- 资源遍历
- 状况检测
- 信息推测
- 跨站点用户追踪
- 跨站点URL/参数列举

# 攻击示例
## 什么是条件泄漏
当攻击者可以在攻击的应用程序中推断条件语句的敏感值时，就会发生条件泄漏。例如，如果一个网站包含以下逻辑：

```
Page A: If (CONDITION)
           Redirect(Page B)
```

攻击者可以执行CSRF并获取有关条件值的指示作为反馈。本次攻击由另一个攻击页面执行。然后该站点向受害者站点提交跨站点请求，并通过操纵历史记录对象获得从受害者站点泄漏的所需信息的反馈。重要的是重定向命令可以显示在代码中，或者可以由操作环境完成。

攻击向量:

- 创建一个IFRAME，其src值为Page B
- 记录当前history.length的值
- 将IFRAME的src值改变为Page A
- 如果history.length的值相同，则条件为真

## 登陆检测

下面的IE和Facebook演示可以显示如何识别用户是否正在使用Facebook：“[我正在使用Facebook吗？](http://www.checkmarx.com/Demo/XSHM.aspx "我正在使用Facebook吗？")”

## 跨站点信息推断

如果实施条件重定向，则可以推断来自不同来源的页面的敏感信息。假设在一个非公开访问的HR应用程序中，合法用户可以通过姓名，薪水和其他标准来搜索员工。 如果搜索没有结果，则执行重定向命令到“NotFound”页面。 通过提交以下网址：
`http://Intranet/SearchEmployee.aspx?name=Jon&SalaryFrom=3000&SalaryTo=3500`
然后观察'未找到页面'重定向，攻击者可以推断工人工资的敏感信息

这可以通过如下的攻击向量实现

- 创建一个IFRAME，其中src的值为"NotFound.aspx"
- 记录当前history.length的值
- 将IFRAME的src值改写为"SearchEmployee.aspx?name=Jon&SalaryFrom=3000&SalaryTo=3500"
- 如果history.length的值保持不变，本次搜索不返回结果

通过替换不同的salary参数值反复执行上述的攻击，攻击者可以手机每个员工的敏感信息（薪水）。这是一次十分严重跨站信息泄露。如果应用程序具有带条件重定向的搜索页面的功能，那么此应用程序易受XSHM攻击，本质上它类似于直接暴露于通用XSS——应用程序本身是XSS安全的，但是从不同的站点在IFRAME内部运行它使其易受攻击。

# 相关攻击代理
待定

# 相关攻击类型

- [跨站点脚本（XSS）](https://tinytracer.com/archives/owasp%e4%b8%aa%e4%ba%ba%e6%b1%89%e5%8c%96%e6%94%bb%e5%87%bb%e7%b3%bb%e5%88%97%e5%a4%a7%e5%85%a8%ef%bc%9a%e8%b7%a8%e7%ab%99%e7%82%b9%e8%84%9a%e6%9c%ac%ef%bc%88xss%ef%bc%89/ "跨站点脚本（XSS）")
- [跨站请求伪造（CSRF）](https://tinytracer.com/archives/%ef%bc%88owasp%e4%b8%aa%e4%ba%ba%e6%b1%89%e5%8c%96%ef%bc%89%e6%94%bb%e5%87%bb%e7%b3%bb%e5%88%97%e5%a4%a7%e5%85%a8%ef%bc%9a%e8%b7%a8%e7%ab%99%e8%af%b7%e6%b1%82%e4%bc%aa%e9%80%a0%ef%bc%88csrf%ef%bc%89/ "跨站请求伪造（CSRF）")

# 相关漏洞

- [跨站点脚本缺陷](https://www.owasp.org/index.php/Cross_Site_Scripting_Flaw "跨站点脚本缺陷")

# 相关防御措施

- [输入验证](https://www.owasp.org/index.php/Input_Validation "输入验证")
- [输出验证](https://www.owasp.org/index.php?title=Output_Validation&action=edit&redlink=1 "输出验证")
- [规范化](https://www.owasp.org/index.php/Canonicalization "规范化")

# 参考

- [OWASP以色列当地分会会议介绍（2010年2月）](https://www.owasp.org/index.php/OWASP_Israel_2010_02#19:10_-_19:40.C2.A0:_XSHM_-_Cross_Site_History_Manipulation "OWASP以色列当地分会会议介绍（2010年2月）")

- [跨站点历史操作（XSHM）指南](https://www.checkmarx.com/wp-content/uploads/2012/07/XSHM-Cross-site-history-manipulation.pdf "跨站点历史操作（XSHM）指南")

- [Checkmarx识别新的Web浏览器漏洞](http://www.infosecurity-magazine.com/view/6828/checkmarx-identifies-new-web-browser-vulnerability/ "Checkmarx识别新的Web浏览器漏洞")，InfoSecurity杂志，2010年1月27日

- [适用于Internet Explorer用户的演示——“我使用Facebook吗？”](http://www.checkmarx.com/Demo/XSHM.aspx "适用于Internet Explorer用户的演示——“我使用Facebook吗？”")

- [维基百科：同源政策（SOP）](http://en.wikipedia.org/wiki/Same_origin_policy "维基百科：同源政策（SOP）")