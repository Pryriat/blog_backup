[TOC]
# A1 - HTML Injection – Reflected (GET、POST)
- 漏洞成因：网站数据提交用到了form表单，且未对表单数据进行验证
  ![](http://ozhtfx691.bkt.clouddn.com/bwapp/A1LM0SZ%5D%29030~19QB@J3%7B%7DV7Y.png)
## EASY
- 安全防护为弱的情况下，在表单提交没有对用户输入的数据进行处理，并且在echo 的时候没有处理就打印到页面
- 对应php代码为：

  ```php
  <?php 
  if(isset($_GET["firstname"]) && isset($_GET["lastname"])) 
  { 
  	$firstname = $_GET["firstname"]; $lastname = $_GET["lastname"]; 
  	if($firstname == "" or $lastname == "") 
  	{ 
  		echo "<font color=\"red\">请输入必填的字段...</font>"; 
  	} 
  	else 
  	{ 
  		echo "Welcome " . $firstname . " --- " . $lastname; 
  	} 
  }
  	?>
  ```

- 此时在输入框中输入

  ```javascript
  <a href=http://www.baidu.com>Click Me</a>
  ```

  即可生成跳转到baidu.com的链接
  ![](http://ozhtfx691.bkt.clouddn.com/bwapp/A1J$CJR%7BUIAY8Q~HX%29~J0BO6E.png)

## medium
- 在安全防护为中等的情况下对输入参数进行urldecode，并将特殊字符处理

  ```php
  urldecode($firstname)
  urldecode($lastname)
  str_replace("<", "&lt;", $input);
  str_replace(">", "&gt;", $input);
  ```

- 此时输入

  ```javascript
  <a href=http://www.baidu.com>Click Me</a>
  ```

无作用
![](http://ozhtfx691.bkt.clouddn.com/bwapp/A1%28HY2%5BI3%7D@AE7%5DLALPK%7BLY5J.png)
- 将对应代码（<、>、/）转换为ascii码输入
  %3ca+href%3dhttp%3a%2f%2fwww.baidu.com%3eClick+Me%3c%2fa%3e
  ![](http://ozhtfx691.bkt.clouddn.com/bwapp/A1@%284M$J%29%60Y%60Z%7D%294A~%7DGM12UW.png)

## 个人感受
- html注入跟CSRF似乎可以构成一套combo....细思极恐

# A2 - Broken Auth. - Password Attacks
- 漏洞成因：密码过于简单
  ![](http://ozhtfx691.bkt.clouddn.com/bwapp/A2%5D3RDTCAC_MJ9%5BPQ_UMC%28IMI.png)
- 遇事不决先burp一遍，不行换字典再burp一遍


# A3 - XSS - Reflected (GET)
## 参考 http://blog.csdn.net/qq_32400847/article/details/53870729
- 漏洞成因：代码直接引用了name参数，过滤检查不严格

## EASY
- 代码直接引用了name参数，并没有任何的过滤与检查

  ```php
  <?php  
   // Is there any input?  
  if( array_key_exists( "name", $_GET ) && $_GET[ 'name' ] != NULL ) {  
      // Feedback for end user  
      $html .= '<pre>Hello ' . $_GET[ 'name' ] . '</pre>';  
  }  
  ?>  
  ```

- 输入 

  ```javascript
  <script>alert(/xss/)</script>
  ```

即可成功弹框。
![](http://ozhtfx691.bkt.clouddn.com/bwapp/A3E@79%5D%25WG42@6%5BLFKFPGO5OJ.png)
## medium
- 对输入进行了过滤，基于黑名单的思想，使用str_replace函数将输入中的script标签删除

  ```php
  <?php
   
  // Is there any input?  
  if( array_key_exists( "name", $_GET ) && $_GET[ 'name' ] != NULL ) {  
      // Get input  
      $name = str_replace( '<script>', '', $_GET[ 'name' ] );  
  
      // Feedback for end user  
      $html .= "<pre>Hello ${name}</pre>";  
  }
  
  ?>
  ```

- 输入

  ```javascript
  <sc<script>ript>alert(/xss/)</script>
  ```

  或者

  ```javascript
  <ScRipt>alert(/xss/)</script>
  ```

  均可成功弹框



## high
- High级别的代码同样使用黑名单过滤输入，preg_replace() 函数用于正则表达式的搜索和替换，这使得双写绕过、大小写混淆绕过（正则表达式中i表示不区分大小写）不再有效。虽然无法使用script标签注入XSS代码，但是可以通过img、body等标签的事件或者iframe等标签的src注入恶意的js代码

  ```php
  <?php  
   
  // Is there any input?  
  if( array_key_exists( "name", $_GET ) && $_GET[ 'name' ] != NULL ) {  
      // Check Anti-CSRF token  
  checkToken( $_REQUEST[ 'user_token' ], $_SESSION[ 'session_token' ], 'index.php' );  
    
      // Get input  
      $name = htmlspecialchars( $_GET[ 'name' ] );  
    
      // Feedback for end user  
      $html .= "<pre>Hello ${name}</pre>";  
  }  
    
  // Generate Anti-CSRF token  
  generateSessionToken();  
    
  ?>  
  ```

- 输入

  ```javascript
  <img src=1 onerror=alert(/xss/)>
  ```

  可成功弹框

# A4 - Insecure DOR (Order Tickets)
- 教程、讲解网上找不到，说说自己玩的过程吧
  ![](http://ozhtfx691.bkt.clouddn.com/bwapp/A4IWD1GFCL_S@$%28B4%28%29YS~U@O.png)
- 通过抓包发现输入框只有票数，但票价是可以自己定义的
  ![](http://ozhtfx691.bkt.clouddn.com/bwapp/A41510821444%281%29.png)
- 算㊣解不.....

# A5 - Insecure WebDAV Configuration
- 原理：服务器对上传文件的类型、内容没有做任何的检查、过滤，存在明显的文件上传漏洞，生成上传路径后，服务器会检查是否上传成功并返回相应提示信息。
  ![](http://ozhtfx691.bkt.clouddn.com/bwapp/A51510821673%281%29.png)
- 抓包，GET改为PUT直接上传
  ![](http://ozhtfx691.bkt.clouddn.com/bwapp/A51510821720%281%29.png)
- 可以搞个大新闻

# A6 - HTML5 Web Storage (Secret)
![](http://ozhtfx691.bkt.clouddn.com/bwapp/A61510822094%281%29.png)
- 难怪打开的时候弹了个窗

# A7 - Directory Traversal - Directories
- 通过改变URL可窥探服务器上的内容
  ![](http://ozhtfx691.bkt.clouddn.com/bwapp/A70%28A5FYLK2VGA%5DJ@Q6XI7$VV.png)
- 将URL中directory指向的目录改变
  ![](http://ozhtfx691.bkt.clouddn.com/bwapp/A73%60W1NZ%25%5B8F1D_1V%25F%7B~PPSN.png)

# A8 - CSRF (Change Password)
![](http://ozhtfx691.bkt.clouddn.com/bwapp/A81510827928%281%29.png)
- 查看源码的数据修改部分
  ![](http://ozhtfx691.bkt.clouddn.com/bwapp/A81510827935%281%29.png)
- 另存为html，修改URL，在input类型后面添加修改值
  ![](http://ozhtfx691.bkt.clouddn.com/bwapp/A81510827946%281%29.png)
- 使用JS语句实现隐藏表单并单击按钮的功能
  ![](http://ozhtfx691.bkt.clouddn.com/bwapp/A81510827956%281%29.png)
- 打开html，修改成功
  ![](http://ozhtfx691.bkt.clouddn.com/bwapp/A81510827962%281%29.png)

# A9 - PHP CGI Remote Code Execution（？？？？？？？）
![](http://ozhtfx691.bkt.clouddn.com/bwapp/A91510828264%281%29.png)
- 打开目标网站，可以直接执行-s命令
  ![](http://ozhtfx691.bkt.clouddn.com/bwapp/A91510828279%281%29.png)
- 输入一坨不知道什么东西后可以在php信息页访问bwapp下的所有页面
  ![](http://ozhtfx691.bkt.clouddn.com/bwapp/A91510828325%281%29.png)
- 只能访问bwapp上的内容，无法通过修改URL访问外部网站

# A10 - Unvalidated Redirects & Forwards (1)
![](http://ozhtfx691.bkt.clouddn.com/bwapp/A101510828512%281%29.png)
- 抓包，修改跳转URL
  ![](http://ozhtfx691.bkt.clouddn.com/bwapp/A101510828520%281%29.png)
  ![](http://ozhtfx691.bkt.clouddn.com/bwapp/A101510828529%281%29.png)