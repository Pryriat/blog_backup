[TOC]

# 漏洞描述
嵌入空字节/代码技术利用了程序没有正确处理NULL终止符后缀的漏洞。这项技术被用于发起其他攻击，如文件浏览、目录遍历、SQL注入、任意代码执行和其他攻击。可以发现很多易受攻击的应用程序，和利用这种技术来破坏系统的漏洞。
这种技术包括几种表示NULL终止符后缀的变体：

```
PATH%00
PATH[0x00]
PATH[空字符的替代表示]
<script></script>%00
```

# 攻击示例
## 示例1-PHP脚本
在下面的例子中，显示了使用这种技术修改URL并访问文件系统上的任意文件，原因是PHP脚本中存在一个漏洞。

```php
$whatever = addslashes($_REQUEST['whatever']);
include("/path/to/program/" . $whatever . "/header.htm");
```

通过利用NUUL后缀操纵URL，攻击者可以访问UNIX密码文件
`http://vuln.example.com/phpscript.php?whatever=../../../etc/passwd%00`

## 示例2-Adobe PDF ActiveX 攻击
另一个已知的攻击利用了ActiveX组件（pdf.ocx）中的缓冲区溢出来允许远程执行任意代码。发出以下请求时触发漏洞
`GET /some_dir/file.pdf.pdf%00[long string] HTTP/1.1`
在此情况下，请求必须发送到于空字节（％00）处截断成的请求的Microsoft IIS和Netscape Enterprise Web服务器上。尽管查找文件请求的URI被截断，但仍会将长字符串传递给Adobe ActiveX组件。该组件触发RTLHeapFree（）内的缓冲区溢出，允许攻击者覆盖内存中的任意字。可以通过将恶意内容添加到任何嵌入式链接的末尾并引用任何Microsoft IIS或Netscape Enterprise Web服务器来执行此攻击。没有必要建立恶意网站来执行此攻击。

# 额外的参考

- [http://capec.mitre.org/data/definitions/52.html](http://capec.mitre.org/data/definitions/52.html "http://capec.mitre.org/data/definitions/52.html")

- [http://nvd.nist.gov/nvd.cfm?cvename=CVE-2004-0629]( http://nvd.nist.gov/nvd.cfm?cvename=CVE-2004-0629 " http://nvd.nist.gov/nvd.cfm?cvename=CVE-2004-0629")

# 相关威胁

- [类别：信息披露](https://www.owasp.org/index.php?title=Category:Information_Disclosure&action=edit&redlink=1 "类别：信息披露")


# 相关攻击类型

- [路径遍历](https://tinytracer.com/archives/%ef%bc%88owasp%e4%b8%aa%e4%ba%ba%e6%b1%89%e5%8c%96%ef%bc%89%e6%94%bb%e5%87%bb%e7%b3%bb%e5%88%97%e5%a4%a7%e5%85%a8%ef%bc%9a%e8%b7%af%e5%be%84%e9%81%8d%e5%8e%86/ "路径遍历")
- [Unicode编码](https://tinytracer.com/archives/%ef%bc%88owasp%e4%b8%aa%e4%ba%ba%e6%b1%89%e5%8c%96%ef%bc%89%e6%94%bb%e5%87%bb%e7%b3%bb%e5%88%97%e5%a4%a7%e5%85%a8%ef%bc%9aunicode%e7%bc%96%e7%a0%81/ "Unicode编码")

# 相关的漏洞

- [类别：输入验证漏洞](https://www.owasp.org/index.php/Category:Input_Validation_Vulnerability "类别：输入验证漏洞")

# 相关防御措施

- [类别：输入验证](https://www.owasp.org/index.php/Category:Input_Validation "类别：输入验证")


# 漏洞分类

- [类别：资源操作](https://www.owasp.org/index.php/Category:Resource_Manipulation "类别：资源操作")