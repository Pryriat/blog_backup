- [TOC]

  *最新版本(mm/dd/yy): 03/1/2010*

  翻译自[Web Parameter Tampering](https://www.owasp.org/index.php/Web_Parameter_Tampering "Web Parameter Tampering")

  # 漏洞描述
  Web参数篡改攻击基于对客户端和服务器之间交换参数的操纵，以修改应用程序数据，例如用户凭证和权限，产品的价格和数量等。通常这类信息存储在cookies，隐藏表单字段或URL查询字符串中，用于控制和增加应用程序功能。执行此类型攻击的目的是为了获取利益，或者利用[中间人攻击](https://www.andseclab.cn/2018/04/15/owasp%e6%b1%89%e5%8c%96%e6%94%bb%e5%87%bb%e7%b3%bb%e5%88%97%e5%a4%a7%e5%85%a8%e5%9b%9b%e5%8d%81%e4%ba%8c%e4%b8%ad%e9%97%b4%e4%ba%ba%e6%94%bb%e5%87%bb%ef%bc%88mitm/ "中间人攻击")来攻击其他人。在这两种情况下，常使用Webscarab和Paros代理等工具。攻击成功取决于完整性和逻辑验证机制错误，对该错误的利用可能导致其他后果，包括XSS，SQL注入，文件包含和路径泄露攻击。有关此漏洞的简短视频描述，[请单击此处](http://www.youtube.com/watch?v=l5LCDEDn7FY&hd=1 "请单击此处")（由[Checkmarx](http://www.checkmarx.com/ "Checkmarx")提供）

  # 攻击示例
  ## 示例1
  表单字段的参数修改可以被认为是Web参数篡改攻击的典型例子。例如，假设一个用户可以在应用界面选择表单字段值（组合框、复选框或其他方式）。当这些值被用户提交时，它们可能被攻击者获取并任意操纵。

  ## 示例2
  当Web应用程序使用隐藏字段存储状态信息时，恶意用户可以篡改存储在浏览器中的值并更改引用的信息。例如，一个在线购物网闸很难使用隐藏字段来引用其项目，如下所示
  `<input type=”hidden” id=”1008” name=”cost” value=”70.00”>`
  在此示例中，攻击者可以通过修改特定项目“value”的值来降低其价格

  # 示例3
  攻击者可以直接篡改URL参数。例如，假设一个Web应用程序，允许用户从组合框中选择他的个人资料并借记该帐户:
  `http://www.attackbank.com/default.asp?profile=741&debit=1000`
  在这种情况下，攻击者可能会篡改URL，并使用其他值进行配置和借记：
  `http://www.attackbank.com/default.asp?profile=852&debit=2000`
  包括属性参数的其他参数都可以被更改。在以下示例中，可以篡改状态变量并从服务器中删除页面：
  `http://www.attackbank.com/savepage.asp?nr=147&status=read`
  修改状态值来删除页面:
  `http://www.attackbank.com/savepage.asp?nr=147&status=del`

  # 相关的威胁代理

  - [类别：客户端攻击](https://www.owasp.org/index.php?title=Category:Client-side_Attacks&action=edit&redlink=1 "类别：客户端攻击")
  - [类别：逻辑攻击](https://www.owasp.org/index.php?title=Category:Logical_Attacks&action=edit&redlink=1 "类别：逻辑攻击")

  # 相关攻击方式

  - [SQL注入](https://www.andseclab.cn/2018/04/22/owasp%e6%b1%89%e5%8c%96%e6%94%bb%e5%87%bb%e7%b3%bb%e5%88%97%e5%a4%a7%e5%85%a8%e5%85%ad%e5%8d%81%e4%ba%8c%ef%bc%9asql%e6%b3%a8%e5%85%a5/ "SQL注入")
  - [XSS攻击](https://tinytracer.com/archives/owasp%e4%b8%aa%e4%ba%ba%e6%b1%89%e5%8c%96%e6%94%bb%e5%87%bb%e7%b3%bb%e5%88%97%e5%a4%a7%e5%85%a8%ef%bc%9a%e8%b7%a8%e7%ab%99%e7%82%b9%e8%84%9a%e6%9c%ac%ef%bc%88xss%ef%bc%89/ "XSS攻击")
  - [路径遍历](https://tinytracer.com/archives/%ef%bc%88owasp%e4%b8%aa%e4%ba%ba%e6%b1%89%e5%8c%96%ef%bc%89%e6%94%bb%e5%87%bb%e7%b3%bb%e5%88%97%e5%a4%a7%e5%85%a8%ef%bc%9a%e8%b7%af%e5%be%84%e9%81%8d%e5%8e%86/ "路径遍历")

  # 相关漏洞

  - [类别：输入验证漏洞](https://www.owasp.org/index.php/Category:Input_Validation_Vulnerability "类别：输入验证漏洞")

  # 相关防御措施

  - [类别：输入验证](https://www.owasp.org/index.php/Category:Input_Validation "类别：输入验证")
  # 参考

  - [http://cwe.mitre.org/data/definitions/472.html ](http://cwe.mitre.org/data/definitions/472.html  "http://cwe.mitre.org/data/definitions/472.html ")- 网页参数篡改
  - [http://www.imperva.com/application_defense_center/glossary/parameter_tampering.html](http://www.imperva.com/application_defense_center/glossary/parameter_tampering.html "http://www.imperva.com/application_defense_center/glossary/parameter_tampering.html") - Imperva - 参数篡改 - 应用程序防御中心
    -[ http://www.cgisecurity.com/owasp/html/ch11s04.html]( http://www.cgisecurity.com/owasp/html/ch11s04.html " http://www.cgisecurity.com/owasp/html/ch11s04.html") - 参数操作 - 第11章——防止常见问题

