[TOC]

*最新版本(mm/dd/yy): 01/31/2016*

翻译自[Brute force attack](https://www.owasp.org/index.php/Brute_force_attack "Brute force attack")

# 漏洞描述
- 暴力攻击有多种不同的表现方式，但主要在于攻击者配置预定值，使用这些值向服务器发出请求，然后分析响应。为了效率，攻击者可能会使用字典攻击（有或无突变）或者一种传统的暴力攻击（给定类别的字符，例如：字母数字，特殊字符，大小写）。基于攻击方式，尝试次数，进行攻击系统的效率以及攻击者攻击的系统效率，攻击者能够粗略计算提交所有选择的预定值需要的时间

# 风险因素

# 攻击示例
- 暴力攻击通常被用于认证攻击和在web应用中发现隐藏的内容/页面。这些攻击经常通过GET和POST请求发送到服务器。对于身份验证，暴力攻击经常在账户锁定策略不适用时奏效。

## 示例1
- 一个web应用被暴力攻击，方式为从流行的内容管理系统获取已知页面的列表，对每个已知页面进行简单请求，然后分析HTTP相应代码来确定该页面是否存在于服务器上
- [DirBuster](https://www.owasp.org/index.php/Category:OWASP_DirBuster_Project "DirBuster")是一个执行这种攻击的工具
- 类似的其他工具如下
  - dirb (http://sourceforge.net/projects/dirb/)
  - WebRoot (http://www.cirt.dk/tools/webroot/WebRoot.txt)

- Dirb有如下功能
  - 设置cookies
  - 添加任何http头
  - 使用代理
  - 改变找到的对象
  - 测试http(s)链接
  - 使用定义的词典和模板查找目录和/或文件
  - 更多功能

- 一个简单的示例

```shell
rezos@dojo ~/d/owasp_tools/dirb $ ./dirb http://testsite.test/
-----------------
DIRB v1.9
By The Dark Raver
-----------------
START_TIME: Mon Jul  9 23:13:16 2007
URL_BASE: http://testsite.test/
WORDLIST_FILES: wordlists/common.txt
SERVER_BANNER: lighttpd/1.4.15
NOT_EXISTANT_CODE: 404 [NOT FOUND]
(Location: '' - Size: 345)

-----------------

Generating Wordlist...
Generated Words: 839

---- Scanning URL: http://testsite.test/ ----
FOUND: http://testsite.test/phpmyadmin/
       (***) DIRECTORY (*)
```
从输出中攻击者被提醒存在dmin/ directory目录。现在攻击者已经发现了一个应用中潜在的有价值的目录。在dirb的模板中，还有一个包含有关无效httpd配置信息的字典。这本词典将检测出这种弱点。
- [WebRoot.pl](http://www.cirt.dk/tools/webroot/WebRoot.txt "WebRoot.pl")，由CIRT.DK编写的应用，已经嵌入了解析服务器响应的机制，并基于攻击者指定的短语判断服务器是否返回预期响应。示例

```shell
Np.

./WebRoot.pl -noupdate -host testsite.test -port 80 -verbose -match "test" -url "/private/<BRUTE>" -incremental lowercase -minimum 1 -maximum 1
```

```shell
oo00oo00oo00oo00oo00oo00oo00oo00oo00oo00oo00oo00
o          Webserver Bruteforcing 1.8          o
0  ************* !!! WARNING !!! ************  0
0  ******* FOR PENETRATION USE ONLY *********  0
0  ******************************************  0
o       (c)2007 by Dennis Rand - CIRT.DK       o
oo00oo00oo00oo00oo00oo00oo00oo00oo00oo00oo00oo00
```

```shell
[X] Checking for updates                - NO CHECK
[X] Checking for False Positive Scan    - OK
[X] Using Incremental                   - OK
[X] Starting Scan                       - OK
   GET /private/b HTTP/1.1
   GET /private/z HTTP/1.1
```

```shell
[X] Scan complete                       - OK
[X] Total attempts                      - 26
[X] Sucessfull attempts                 - 1
oo00oo00oo00oo00oo00oo00oo00oo00oo00oo00oo00oo00
```
WebRoot.pl在testsite.test上找到一个文件“/ private / b”，其中包含短语“test”。
- 另一个例子是检查变量值的范围
```shell
./WebRoot.pl -noupdate -host testsite.test -port 80 -verbose -diff "Error" -url "/index.php?id=<BRUTE>" -incremental integer -minimum 1 -maximum 1
```

### 路障
- 像dirb / dirbuster这样的工具的主要问题之一就是分析服务器响应。更先进的服务端认证（如mod_rewrite)自动工具有时无法确认”目录不存在“错误，由于服务器返回200HTTP状态码但页面显示”目录不存在"。这会导致仅依赖于HTTP响应码的暴力攻击工具失效

- 诸如[Burp Suite](http://portswigger.net/ "Burp Suite")等更先进的评定工具可用于解析返回页面的特定部分，寻找指定字串以减少误报

## 示例2
在认证方面，当没有密码策略时，攻击者可以使用普通用户名和密码列表爆破用户名和/或密码字段，直到认证成功。

# 防御工具
## Php-Brute-Force-Attack Detector

[http://yehg.net/lab/pr0js/files.php/php_brute_force_detect.zip](http://yehg.net/lab/pr0js/files.php/php_brute_force_detect.zip "http://yehg.net/lab/pr0js/files.php/php_brute_force_detect.zip") - 防止你的web服务器被诸如WFuzz,OWASP DirBusterd等暴力工具扫描、Nessus, Nikto, Acunetix等漏洞扫描器的扫描。这会帮助你快速识别可能的漏洞挖掘者。
[http://yehg.net/lab/pr0js/tools/php-brute-force-detector-readme.pdf](http://yehg.net/lab/pr0js/tools/php-brute-force-detector-readme.pdf "http://yehg.net/lab/pr0js/tools/php-brute-force-detector-readme.pdf")

# 相关的威胁代理

[分类：认证](https://www.owasp.org/index.php?title=Category:Authentication&action=edit&redlink=1 "Category:Authentication")

# 相关攻击方式

- [SQL盲注](https://tinytracer.com/archives/%ef%bc%88owasp%e4%b8%aa%e4%ba%ba%e6%b1%89%e5%8c%96%ef%bc%89%e6%94%bb%e5%87%bb%e7%b3%bb%e5%88%97%e5%a4%a7%e5%85%a8%ef%bc%9asql%e7%9b%b2%e6%b3%a8/ "SQL盲注")
- [XPath盲注](https://tinytracer.com/archives/%ef%bc%88owasp%e4%b8%aa%e4%ba%ba%e6%b1%89%e5%8c%96%ef%bc%89%e6%94%bb%e5%87%bb%e7%b3%bb%e5%88%97%e5%a4%a7%e5%85%a8%ef%bc%9axpath%e7%9b%b2%e6%b3%a8/ "XPath盲注")

# 相关漏洞

- [会话ID长度不足](https://www.owasp.org/index.php/Insufficient_Session-ID_Length "Insufficient Session-ID Length")

# 相关防御措施

- [认证](https://www.owasp.org/index.php?title=Authentication&action=edit&redlink=1 "Authentication")

# 参考

- [https://www.owasp.org/index.php/Category:OWASP_DirBuster_Project](https://www.owasp.org/index.php/Category:OWASP_DirBuster_Project "https://www.owasp.org/index.php/Category:OWASP_DirBuster_Project") DirBuster - [http://portswigger.net/](http://portswigger.net/ "http://portswigger.net/")