[TOC]
# SELECT语句
-  用法：SELECT 列名 from 表名，select * from tablename表示检索表中所有的数据
-  当列名含有空格时，需要使用\` \`将列名括起，如 select \`student name\` from .....

# 计算和别名
-  使用计算字段，可以检索特定的数据，如选择特定的单词数值等

## 直接量
- 直接量为用' '括起的字符串或者数字，当select语句包含直接量时，将直接量输出为一个没有名字的列，每行都为该直接量。如 select 5, \`student name` from student，输出为

  | (no columnname | student name |
  | :------------: | :----------: |
  |       5        |     aaa      |

- 可以对一个或多个列进行算数运算，支持加减乘除
- 使用concat()连接字段，如select concat(student_name, 'bbb')，在查询或返回时会将student的值与bbb连接起来
- 使用关键字as为列指定标题，用法：select ..... AS '列名'。亦可指定表的别名，如select 12.studentname from 12 AS student。注意，表别名不需要单引号

# 函数
- SQL中包含了多种函数。函数分为标量函数和聚合函数。标量函数只对单行数据进行处理，如LTRIM函数用于删除一个特定值中的起始空格。相反，聚合函数需要对数据集合进行操作，如SUM函数可以计算一个特定的列中所有值的总和。

## 字符函数
- LEFT(字符串，偏移量)：从指定的字符串中抽取从最左开始偏移量个字符并返回
- RIGHT(字符串，偏移量)：从最右开始抽取
- SUBSTRING(字符串，起始位置，偏移量)：从起始位置抽取偏移量个字符返回
- LTRIM(字符串)：删除字符串左侧所有空格
- RTRIM(字符串)
- CONTACT(字符串1、字符串2......)：将参数所有字符串连接成一个字符串
- UPPER(字符串)：转换成大写
- LOWER(字符串)

## 复合函数
- 复合函数：函数的一个重要特性是可以把两个或以上的函数组合成一个复合函数，如select UPPER(RTRIM(\`student name`)) from student

## 日期\时间函数
- 日期\时间函数可用来操作日期值和时间值。
- NOW()：获取当前时间，获取的格式为20xx-xx-xx xx:xx:xx
- DATEDIFF(EndDate, StartDate)：获取两个日期的天数差

## 数值函数
- ROUND(数值, 保留小数位)：对数值进行四舍五入，空位用0填充
- RAND(seed)：产生随机数。当使用RAND()时，将返回0到1随机数。seed参数是一个整数，当执行带seed参数的rand()将返回一个特定的值
- PI()：返回PI值

## 转换函数
- CAST(数据 AS 类型)，如CAST(153 AS char)
- IFNULL(数据，替代值）：如果该值为NULL，返回替代值

# 排序

## ORDER BY子句
- order by 子句总是在FROM子句之后，而FROM子句总是在SELECT之后，示例： select ..... from ...... order by.......
- order by 子句默认以升序排列，即select ... from .... order by .... ASC，如果需要降序排列则将ASC替换为DESC
  -order by 子句接收多个列作排序依据，优先级按顺序递减，如select ... from ... order by student_num, mark.
- order by子句可以接收计算字段
- 排序时NULL优先于数字，数字优先于字符

## 基于列的逻辑
- ASE表达式
  - CASE表达式用在select子句中，对列进行筛选，如select c1, c2... CASE from.....
  - CASE表达式有两种一般格式，通常称为简单格式和查询格式。
  - 简单格式示例：

    ```CPP
    select
    case columnname 
    when value1 then resault 1
    when value2 then resault 2
    else default resault
    end
    ```

  - CASE表达式除了使用除CASE以外的关键字：WHEN、THEN、ELSE和END，需使用这些额外的关键字来完整定义CASE表达式的逻辑。关键字WHEN和THEN定义了要计算的条件，如果WHEN后面的值是TRUE那么使用THEN后面的结果，如WHEN 'F' THEN FRUIT。如果没有满足的WHEN-THEN，那么将使用default的值
  - 查询格式示例

    ```CPP
    select
    case
    when student_num < 1 then 1
    when student_num >1 and student_num < 10 then value2
    else value 3
    end
    from ......
    ```

## 基于行的逻辑
- 应用查询条件
  - SQL中的查询条件是从where子句开始的，关键字where的逻辑为基于列的逻辑。不同之处在于，列逻辑只允许对一个特定列应用，而行逻辑作用于整个表。
  - where子句总是在from和order by子句之间，条件子句放在关键字where后
    - 简单示例：  `select id, studentname,..... from 12 where id<50`;
  - 限制行
    - 在mysql中使用limit关键字来限制返回的行数，一般格式为`select  columnlist from table limit number`
    - limit关键字与order by关键字合用了实现Top N查询 `select .... from .... order by .... limit.....`

## 布尔逻辑
- 设计复杂逻辑条件的能力称为布尔逻辑
- 用于生成布尔逻辑的关键字有AND、OR和NOT。这3个操作符用来为where子句添加额外的功能。
- 圆括号可以设置布尔逻辑的优先级
- 在SQL中，默认先计算NOT后计算AND最后后计算OR，可以使用圆括号改变顺序
- NOT表示对后面的内容否定 `select ... from ... where not .....`
- 不等于运算符（<>）亦可实现否定效果 `select ... from .... where id <> ......`
- BETWEEN操作符限定数据的范围，需要与AND连用，如`select ... from .... where id between 1 and 20`
- IN操作符基于OR限定数据范围，如`select ... from ... where id = 1 or id = 2 等价于 select ... from .... where id in (1,2)`
## NULL值
- NULL值为缺少数据的值，可表示where子句判断中未知的状态
- 可以用is null、is not null来检索没有被赋予数据的值，如 `select ... from ... where student_name is null`
## 模糊匹配
- 模式匹配使用like关键字。模式匹配为针对查找值某个部分的匹配，使用通配符来确定匹配的范围。示例：`select title from ... where id like '% Jack'`。通配符有两种，百分号(%)表示任意字符，下划线(_)表示匹配一个字符
- 按照读音匹配使用soundex函数，示例: `select soundex('Smith');`函数返回一个表示读音的值。