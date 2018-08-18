[TOC]

*最新版本 (mm/dd/yy): 05/26/2009*

翻译自[Unicode Encoding](https://www.owasp.org/index.php/Unicode_Encoding "Unicode Encoding")

# 漏洞描述
该攻击利用了在应用程序上实施的Unicode数据格式解码机制中的缺陷。攻击者可以使用此技术对URL中的某些字符进行编码以绕过应用程序过滤器，从而访问Web服务器上的受限资源或强制浏览受保护的页面。

# 攻击示例
- 设想一个包含限制目录或文件（如包括应用用户名的appusers.txt）web应用。攻击者可以使用Unicode格式对字符序列“../”（[路径遍历攻击](https://tinytracer.com/archives/%ef%bc%88owasp%e4%b8%aa%e4%ba%ba%e6%b1%89%e5%8c%96%ef%bc%89%e6%94%bb%e5%87%bb%e7%b3%bb%e5%88%97%e5%a4%a7%e5%85%a8%ef%bc%9a%e8%b7%af%e5%be%84%e9%81%8d%e5%8e%86/ "路径遍历攻击")）进行编码，并尝试访问受保护的资源，如下所示：

- 原始的目录遍历攻击URL (未使用Unicode编码):
  `http://vulneapplication/../../appusers.txt`

- 使用Unicode编码的目录遍历攻击URL
  `http://vulneapplication/%C0AE%C0AE%C0AF%C0AE%C0AE%C0AFappusers.txt`

如上对URL进行Unicode编码将产生与第一个URL相同的结果（路径遍历攻击）。不过，如果应用配备了输入安全过滤机制，它可以拒绝包含“../”序列的任何请求，从而阻止攻击。不过，如果此机制不考虑字符编码，则攻击者可绕过并访问受保护的资源。

# 相关的威胁代理

- [类别：命令执行](https://www.owasp.org/index.php?title=Category:Command_Execution&action=edit&redlink=1 "类别：命令执行")
- [类别：信息披露](https://www.owasp.org/index.php?title=Category:Information_Disclosure&action=edit&redlink=1 "类别：信息披露")

# 相关攻击

- [路径遍历](https://tinytracer.com/archives/%ef%bc%88owasp%e4%b8%aa%e4%ba%ba%e6%b1%89%e5%8c%96%ef%bc%89%e6%94%bb%e5%87%bb%e7%b3%bb%e5%88%97%e5%a4%a7%e5%85%a8%ef%bc%9a%e8%b7%af%e5%be%84%e9%81%8d%e5%8e%86/ "路径遍历")
- [嵌入空代码](https://tinytracer.com/archives/%ef%bc%88owasp%e4%b8%aa%e4%ba%ba%e6%b1%89%e5%8c%96%ef%bc%89%e6%94%bb%e5%87%bb%e7%b3%bb%e5%88%97%e5%a4%a7%e5%85%a8%ef%bc%9a%e5%b5%8c%e5%85%a5%e7%a9%ba%e4%bb%a3%e7%a0%81/ "嵌入空代码")

# 相关的漏洞

- [类别：输入验证漏洞](https://www.owasp.org/index.php/Category:Input_Validation_Vulnerability "类别：输入验证漏洞")

# 相关防御措施

- [类别：输入验证](https://www.owasp.org/index.php/Category:Input_Validation "类别：输入验证")

# 参考

- [http://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2000-0884](http://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2000-0884 "http://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2000-0884") - CVE-2000-0884
- [http://capec.mitre.org/data/definitions/71.html](http://capec.mitre.org/data/definitions/71.html "http://capec.mitre.org/data/definitions/71.html") - 使用Unicode编码绕过验证逻辑
- [http://www.microsoft.com/technet/security/bulletin/MS00-078.mspx](http://www.microsoft.com/technet/security/bulletin/MS00-078.mspx "http://www.microsoft.com/technet/security/bulletin/MS00-078.mspx") - 可用于“Web服务器文件夹遍历”漏洞的修补程序
- [http://www.kb.cert.org/vuls/id/739224](http://www.kb.cert.org/vuls/id/739224 "http://www.kb.cert.org/vuls/id/739224") - 用全宽/半宽Unicode编码绕过HTTP内容扫描系统
- [http://www.cgisecurity.com/lib/URLEmbeddedAttacks.html](http://www.cgisecurity.com/lib/URLEmbeddedAttacks.html "http://www.cgisecurity.com/lib/URLEmbeddedAttacks.html") - URL编码攻击，由Gunter Ollmann撰写
