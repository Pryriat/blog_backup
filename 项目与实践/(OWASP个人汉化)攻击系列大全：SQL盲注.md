[TOC]

*最新版本(mm/dd/yy): 08/26/2013*

翻译自[Blind SQL Injection](https://www.owasp.org/index.php/Blind_SQL_Injection "Blind SQL Injection")

# 漏洞描述
- SQL盲注是一种向数据库进行布尔查询，通过返回值来判断答案的SQL注入攻击方式。这种攻击方式通常用于配置了显示通用错误信息，却没有对SQL注入代码进行防护的网络应用。
- 当攻击者利用SQL注入漏洞时，有时候网络应用会显示来自数据库，提示SQL语法错误的信息。SQL盲注与普通SQL注入几乎一致，唯一的区别在于是从数据库中检索数据的方式。在数据库中的数据不在网页显示的情况下，攻击者被迫向数据库提出一系列的布尔查询来窃取数据。这种方式让SQL注入漏洞的利用更加艰难，但并非不可能。

# 攻击模型
- 与SQL注入相同

# 风险因素
- 与SQL注入相同

# 攻击示例
攻击者可以通过如下方式来判断发送请求的返回值
## 基于内容
- 一个简单的页面，通过ID作为参数显示文章，攻击者可以通过一些列的简单测试来判断页面是否有SQL注入漏洞
- 示例url:
  `http://newspaper.com/items.php?id=2`
- 该url向数据库发出如下的查询语句：
  `SELECT title, description, body FROM items WHERE ID = 2`
- 攻击者可能会尝试注入一个返回'false'的查询:
  `http://newspaper.com/items.php?id=2 and 1=2`
- 此时SQL查询语句如下:
  `SELECT title, description, body FROM items WHERE ID = 2 and 1=2`
- 如果该网页存在SQL注入漏洞，则很可能不会返回任何内容。为了确定漏洞，攻击者会注入一个返回'true'的查询
  `http://newspaper.com/items.php?id=2 and 1=1`
- 如果返回'true'页面的内容与返回'false'的不同，攻击者就能区分查询语句执行后返回是'true'还是'false'。确定这一点后，漏洞利用的限制只在于数据库用户权限、不同的SQL语法和攻击者的想象力了。

## 基于时间
- 这种SQL盲注依赖于数据库暂停一定的时间，然后返回SQL语句成功执行的结果。使用这种方式，攻击者使用一下逻辑枚举数据中的每个字母
  - 如果数据库名第一个字母是'A',等待10秒
  - 如果数据库名第一个字母是'B',等待10秒 
  - 以此类推

- 不同数据库的延时语法：
  - Microsoft SQL Server
    `http://www.site.com/vulnerable.php?id=1' waitfor delay '00:00:10'--`
  - MySQL
    `SELECT IF(expression, true, false)`
    - 使用一些耗费时间的操作（如`BENCHMARK()`），在漏洞存在时会延迟服务器的应答。示例
      `BENCHMARK(5000000,ENCODE('MSG','by 5 seconds'))`
      将会执行`ENCODE`函数5000000次
    - 基于数据库服务器的性能和负载，这个操作应在很短的时间内完成。重要的是，从攻击者的角度，指定足够多的`BENCHMARK()`重复执行次数可以显著影响数据库的响应时间。


- 基于时间攻击的语句示例
```sql
UNION SELECT IF(SUBSTRING(user_password,1,1) = CHAR(50),BENCHMARK(5000000,ENCODE('MSG','by 5 seconds')),null) FROM users WHERE user_id = 1;
```
- 如果数据库在长时间后才回应，我们就可以猜测用户id为1的密码的第一个字符是'2'(CHAR(50) == '2')
- 使用这种方法确定剩余的字符很可能将数据库中存储的所有密码枚举出来。这种方法甚至在攻击者进行SQL注入，漏洞页面内容不发生变化的情况下奏效。
- 很明显，在这个示例中，表名和列数都是确定的。不过这两项可以通过猜测或者试错法获得。

- 除了MYSQL，其他数据库也有可以被基于时间攻击利用的函数
  - MS SQL : `'WAIT FOR DELAY '0:0:10`
  - PostgreSQL : `pg_sleep()`
- 手动执行SQL盲注非常耗费时间，但有很多工具可以自动执行盲注。其中之一是OWASP授权计划内开发的SQLMap([http://sqlmap.org/](http://sqlmap.org/ "http://sqlmap.org/"))。另一方面，这一类工具对规则的微小偏差非常敏感。包括
  - 扫描时钟不同步的网站集群
  - 参数获取方法发生改变的www服务器，如从`/index.php?ID=10`变为`/ID,10`

# 远程数据库指纹验证
- 如果一个攻击者能够确定查询语句返回的是'true'还是'false'，那么就有可能获取数据库的指纹。这将让整个攻击变得更加简单。如果基于时间的攻击被成功执行，对于确定正在使用的数据库类型很有帮助。另一个被广泛使用的获取方法是调用返回当前日期的函数。MYSQL、MSSQL和Oracle有不同的返回日期函数分别是`now()`,`getdate()`和`sysdate()`

# 相关的威胁代理
- 与SQL注入相同

# 相关的攻击方式
- [XPath盲注](https://tinytracer.com/archives/%ef%bc%88owasp%e4%b8%aa%e4%ba%ba%e6%b1%89%e5%8c%96%ef%bc%89%e6%94%bb%e5%87%bb%e7%b3%bb%e5%88%97%e5%a4%a7%e5%85%a8%ef%bc%9axpath%e7%9b%b2%e6%b3%a8/ "XPath盲注")
- [SQL注入](https://www.andseclab.cn/2018/04/22/owasp%e6%b1%89%e5%8c%96%e6%94%bb%e5%87%bb%e7%b3%bb%e5%88%97%e5%a4%a7%e5%85%a8%e5%85%ad%e5%8d%81%e4%ba%8c%ef%bc%9asql%e6%b3%a8%e5%85%a5/ "SQL注入")
- [XPath注入](https://www.andseclab.cn/2018/04/20/owasp%e6%b1%89%e5%8c%96%e6%94%bb%e5%87%bb%e7%b3%bb%e5%88%97%e5%a4%a7%e5%85%a8%e5%85%ad%e5%8d%81%e5%85%ab%ef%bc%9axpath%e6%b3%a8%e5%85%a5%e6%94%bb%e5%87%bb/ "XPath注入")
- [LDAP注入](https://tinytracer.com/archives/owasp%e4%b8%aa%e4%ba%ba%e6%b1%89%e5%8c%96%e6%94%bb%e5%87%bb%e7%b3%bb%e5%88%97%e5%a4%a7%e5%85%a8%ef%bc%9aldap%e6%b3%a8%e5%85%a5/ "LDAP注入")
- [服务端包含(SSI)注入](https://www.andseclab.cn/2018/04/20/owasp%e6%b1%89%e5%8c%96%e6%94%bb%e5%87%bb%e7%b3%bb%e5%88%97%e5%a4%a7%e5%85%a8%e4%ba%94%e5%8d%81%e4%ba%94%ef%bc%9a%e6%9c%8d%e5%8a%a1%e5%99%a8%e7%ab%af%e5%8c%85%e5%90%abssi%e6%b3%a8%e5%85%a5/ "服务端包含(SSI)注入")

# 相关漏洞
- [注入漏洞](https://www.owasp.org/index.php/Injection_problem "Injection_problem")

# 相关防护
- [分类：输入验证](https://www.owasp.org/index.php/Category:Input_Validation "Category:Input Validation")

- 查看[OWASP开发指南](https://www.owasp.org/index.php/Category:OWASP_Guide_Project "OWASP开发指南")防御SQL注入部分
- 查看[OWASP SQL注入防御备忘录](https://www.owasp.org/index.php/SQL_Injection_Prevention_Cheat_Sheet "OWASP SQL注入防御备忘录")
- 查看[OWASP代码审计指南](https://www.owasp.org/index.php/Category:OWASP_Code_Review_Project "OWASP代码审计指南") 获悉[审计SQL注入漏洞代码](https://www.owasp.org/index.php/Reviewing_Code_for_SQL_Injection "审计SQL注入漏洞代码")
- 查看[OWASP测试指南](https://www.owasp.org/index.php/Category:OWASP_Testing_Project "OWASP测试指南")关于 [SQL漏洞测试](https://www.owasp.org/index.php/Testing_for_SQL_Injection_(OWASP-DV-005) "SQL漏洞测试")文章

# 参考

- [http://www.cgisecurity.com/questions/blindsql.shtml](http://www.cgisecurity.com/questions/blindsql.shtml "http://www.cgisecurity.com/questions/blindsql.shtml")
- [http://www.imperva.com/application_defense_center/white_papers/blind_sql_server_injection.html](http://www.imperva.com/application_defense_center/white_papers/blind_sql_server_injection.html "http://www.imperva.com/application_defense_center/white_papers/blind_sql_server_injection.html")
- [http://www.securitydocs.com/library/2651](http://www.securitydocs.com/library/2651 "http://www.securitydocs.com/library/2651")
- [http://seclists.org/bugtraq/2005/Feb/0288.html](http://seclists.org/bugtraq/2005/Feb/0288.html "http://seclists.org/bugtraq/2005/Feb/0288.html")
- [http://ferruh.mavituna.com/makale/sql-injection-cheatsheet/](http://ferruh.mavituna.com/makale/sql-injection-cheatsheet/ "http://ferruh.mavituna.com/makale/sql-injection-cheatsheet/")

## 在线参考

- [more Advanced SQL Injection](http://www.nccgroup.com/Libraries/Document_Downloads/more__Advanced_SQL_Injection.sflb.ashx "more Advanced SQL Injection") - by NGS

- [Blind SQL Injection Automation Techniques](http://www.blackhat.com/presentations/bh-usa-04/bh-us-04-hotchkies/bh-us-04-hotchkies.pdf "Blind SQL Injection Automation Techniques") - Black Hat Pdf

- [Blind Sql-Injection in MySQL Databases](http://seclists.org/lists/bugtraq/2005/Feb/0288.html "Blind Sql-Injection in MySQL Databases")

- [Cgisecurity.com: What is Blind SQL Injection?](http://www.cgisecurity.com/questions/blindsql.shtml "Cgisecurity.com: What is Blind SQL Injection?")

- Kevin Spett from SPI Dynamics: [http://www.net-security.org/dl/articles/Blind_SQLInjection.pdf](http://www.net-security.org/dl/articles/Blind_SQLInjection.pdf "http://www.net-security.org/dl/articles/Blind_SQLInjection.pdf")

- [http://www.imperva.com/resources/whitepapers.asp?t=ADC](http://www.imperva.com/resources/whitepapers.asp?t=ADC "http://www.imperva.com/resources/whitepapers.asp?t=ADC")

- [Advanced SQL Injection](https://www.owasp.org/images/7/74/Advanced_SQL_Injection.ppt "Advanced SQL Injection")

## 工具

- [SQL Power Injector](http://www.sqlpowerinjector.com/ "SQL Power Injector")
- [Absinthe :: Automated Blind SQL Injection](http://www.0x90.org/releases/absinthe/ "Absinthe :: Automated Blind SQL Injection") // ver1.3.1
- [SQLBrute - Multi Threaded Blind SQL Injection Bruteforcer](http://www.securiteam.com/tools/5IP0L20I0E.html "SQLBrute - Multi Threaded Blind SQL Injection Bruteforcer") in Python
- [SQLiX - SQL Injection Scanner](https://www.owasp.org/index.php/Category:OWASP_SQLiX_Project "SQLiX - SQL Injection Scanner") in Perl
- [sqlmap, automatic SQL injection tool](http://sqlmap.org/ "sqlmap, automatic SQL injection tool") in Python
- [bsqlbf, a blind SQL injection tool](https://code.google.com/p/bsqlbf-v2/ "bsqlbf, a blind SQL injection tool") in Perl