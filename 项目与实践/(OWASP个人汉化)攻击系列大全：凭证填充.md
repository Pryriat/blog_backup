[TOC]

翻译自[Credential stuffing](https://www.owasp.org/index.php/Credential_stuffing "Credential stuffing")

# 漏洞描述

凭证填充是自动注入用户/密码对以欺骗性地获取用户权限、这是暴力攻击的一个子集：大量溢出的凭据会自动输入网站，直到它们可能与现有帐户相匹配，然后攻击者可以劫持以达到自己的目的。

## 唯一性

凭证填充是通过自动化网页注入完成帐户接管的新型攻击形式。凭据填充与数据库的缺陷相关联；二者都完成了账户接管。凭据填充是一个新兴的威胁。
凭据填充无论是在消费者还是企业的角度都是十分危险的，因为这些违规行为的连锁反应。有关此更多信息，请参考示例部分，通过凭证填充展示从一个泄露事件到另一个事件的连接事件链。

## 严重性
凭据填充是用来接管用户帐户的最常用技术之一
攻击剖析
- 攻击者从密码转储网站或从网站漏洞获取溢出的用户名和密码。
- 攻击者使用一个账户检查器来用多个网站测试窃取的凭据（例如社交媒体、在线商城）。
- 成功的登陆（在所有的尝试中占0.1%~0.2%）让攻击者能匹配窃取的凭据来接管账户
- 攻击者将窃取用户存储的信息，信用卡帐号，和其他个人身份信息的帐户
- 攻击者还可以使用账户信息进行更进一步的恶意目的（例如，发送垃圾邮件或创建更多交易）

## 图示
![](https://www.owasp.org/images/d/de/Credentialstuffing.png)

## 参考
[http://michael-coates.blogspot.be/2013/11/how-third-party-password-breaches-put.html](http://michael-coates.blogspot.be/2013/11/how-third-party-password-breaches-put.html "http://michael-coates.blogspot.be/2013/11/how-third-party-password-breaches-put.html")

# 攻击示例

接下来的是已曝光的大规模漏洞分析的摘录，证据表明这些漏洞是凭据填充导致的
- 索尼，2011年漏洞：“我想强调一下，今年早些时候索尼数据集和Gawker漏洞数据中有三分之二的用户对每个系统都使用相同的密码。”
  - 来源：Agile Bits[[1]](https://blog.agilebits.com/2011/06/07/two-thirds-of-web-users-re-use-the-same-passwords/ "[1]")
  - 来源：Wired[[2]](http://www.wired.com/2011/10/93000-sony-accounts-breached/ "[2]")

- 雅虎，2012年漏洞：“索尼和雅虎是什么？ 有共同点？密码！”
  - 来源：Troy Hunt[[3]](http://www.troyhunt.com/2012/07/what-do-sony-and-yahoo-have-in-common.html "[3]")

- Dropbox，2012漏洞：“这些文章中引用的用户名和密码是从无关的服务中窃取的，而不是Dropbox。 攻击者随后使用这些被盗取的凭证尝试登录互联网上的网站，包括Dropbox“。
  - 来源: Dropbox.[[4]](https://blog.dropbox.com/2014/10/dropbox-wasnt-hacked/ "[4]")

- JPMC，2014漏洞：“[违规数据]包含一些在企业挑战赛网站上注册的比赛参与者使用的密码和电子邮件地址组合，该网站是摩根大通在主要城市赞助的一系列年度慈善赛事的在线平台并由外部供应商运行。这些比赛对银行员工和其他公司的员工开放“。
  - 来源: 纽约时报 [[5]](http://dealbook.nytimes.com/2014/10/31/discovery-of-jpmorgan-cyberattack-aided-by-company-that-runs-race-website-for-bank/ "[5]")

从索尼到雅虎到Dropbox的这一连串事件，除了JPMC。 JPMC漏洞来自一个单独和不相关的来源。 我们知道JPMC漏洞是攻击者针对一个无关的第三方体育竞赛/运营网站提供的凭证来对抗JPMC。

# 相关攻击代理

- [类别：认证](https://www.owasp.org/index.php?title=Category:Authentication&action=edit&redlink=1 "Category:Authentication")

# 相关攻击方式

- [暴力攻击](https://tinytracer.com/archives/%ef%bc%88owasp%e4%b8%aa%e4%ba%ba%e6%b1%89%e5%8c%96%ef%bc%89%e6%94%bb%e5%87%bb%e7%b3%bb%e5%88%97%e5%a4%a7%e5%85%a8%ef%bc%9a%e7%bc%93%e5%86%b2%e5%8c%ba%e6%ba%a2%e5%87%ba%e6%94%bb%e5%87%bb/ "暴力攻击")

# 相关漏洞

- [API滥用](https://www.owasp.org/index.php/API_Abuse "API滥用")

# 参考

- 凭据填充的衍生[[6]](https://prezi.com/kdilcmkhhrfl/ramification-of-credential-stuffing/ "[6]")
- 漏洞执行后后窃取数据会发生什么？[[7]](http://www.securityweek.com/what-happens-stolen-data-after-breach "[7]")
- 第三方密码泄露如何使您的网站处于风险之中[[8]](http://michael-coates.blogspot.com/2013/11/how-third-party-password-breaches-put.html "[8]")