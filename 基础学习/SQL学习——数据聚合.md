[TOC]
- 在计算机术语中，将数据汇总的操作称为聚合，表示“合并到组中”
# 消除重复
-  汇总数据的最基本方法是消除重复，SQL中使用DISTINCT关键字删除输出中重复的行，如
  `select distinct attentio_num from ......`
-  关键字distinct总是跟随在select之后，DISTINCT表示只会返回后面columnlist的列值唯一的那些行

# 聚合函数
-  对单个数字挥着值进行计算的函数为标量函数，如UPPER、LOWER等。聚合函数可用于分组数据，如COUNT、SUM、AVG等，它们的操作对象为一组而非单个数据
-  SUM、AVG、MIN、MAX为返回加和、平均值、最小值和最大值的聚合量。使用示例为
  `select sum(id) from ..... where ......`
## COUNT函数
- COUNT函数可用于返回选中行的数目，而不管任意特定列的值。使用格式为
  `select count(*) from ..... where .....`
  圆括号中的星号表示所有列，SQL检索了那些选中行的所有列，返回行的数目
- COUNT函数可在括号中指定列名，如 `select count(id) from ... where ...` ，当id列为null值时，即使where子句的判断条件成立也不会被计数
- COUNT函数在括号中同时使用列名和DISTINCT作关键字时，可以计算该列唯一值的个数。如 
  `select count(distinct attention_num) from ...`
## 分组数据
- 使用关键字group by 关键字为返回的数据进行分组，对每一组数据进行统计，使用示例:
  `select id ,... from ... group byattention_num`
  此时统计数据的单位不再是单个id，而是attention_num相同的所有id的值
- 注意，使用关键字group by子句时，columnlist中的列要么是group by子句中的列，要么是在聚合函数中使用的列
- group by子句可包含多个列，顺序对数据处理无关
- 使用having语句限制选择条件，having语句在group by 子句之后，order by之前，如
  `select ... from ... group by ... having .... order by ...`
## 内连接
- 当多个表中包含相同的列名时，可以使用内连接将两表的信息组合输出。如表LikeMember和表ForumMembers有共同的列Memberid，通过内连接将两表的内容输出。示例:
  `select * from LikeMembver inner jion FroumMembers on LikeMember.Memberid = ForumMembers.id`
- 内连接返回两个表之间匹配的数据，输出的数据内容与连接顺序无关，输出列顺序与连接顺序一致
- 可使用as指定表别名
- 可以使用from 和 where子句实现内连接，如
```sql
select a.id, b.name from FroumMembers as `a`,  LikeMember as `b` where a.id = b.id;
```
## 外连接
- 内连接的限制为需要在所有连接的表中有相应的匹配，而外连接只需要连接的两表有关联即可（类似于链表）外连接分为左连接、右连接和全连接
- 左连接示例
```sql
select a.id, b.number from LikeMembver as 'a' left join FroumMembers as 'b' on a.id == b.id
```
- 使用left join语句表排列顺序十分重要。left join左边的表为主表，右边的表为从表。主表不为null，从表的值可为null。主表为NULL，从表必为null
- 右连接语法为
  `select ..... from .... right join ..... on .... right join .... on ......`
  与左连接不同在于右连接左表为从表，右表为主表
- 全连接中，两个表都是从表。在这种情况下，将输出各表匹配的行，即使其他表无匹配项
- 自连接允许将一个表和其自身联系起来，处理那些本质上是自引用的表。自连接可以是内连接、外连接，使用表别名区分
## 视图
- 视图是保存在SQL中的select语句，一旦保存则可以像引用数据库中数据一样引用视图。视图不包含数据，但可以像处理真实表一样处理视图
- 使用creat view创建视图，语法: `creat view Viewname as （sql语句） ....`,如
  `creat view search as selsect id from db_1`
- 因为视图不保存物理数据，所以在创建视图时不能使用order by子句
- 创建视图后，可将视图与表等价
- 视图的优势有
  - 减少复杂性，将select语句封装成视图，简化调用
  - 增加可复用性
  - 正确格式化数据
  - 创建计算的列
  - 重新命名列名名称
  - 创建数据子集
  - 加强安全性限制
- 使用alter view关键字修改视图，示例：`alter view Viewname as (修改的sql语句)`
- 删除视图的关键字为drop，如`drop view viewname`

# 集合逻辑
- 合并查询的概念通常称为集合逻辑，可以把每个SELECT查询作为一个集合引用
- 如果数据在表A中或者在表B中，使用UNION操作符聚合多个select语句获取数据
  - 语法：`select .... union select....`
  - 使用`union`合并在一起的select语句，必须有相同数目的列
  - 选取列必须以相同的顺序排列
  - 每个列必须有相同的或者可兼容的类型
  - `union`排除了选取列的重复数据，类似`distinct`
  - 与`union`相比，`union all`语句不排除重复数据，用法与union相同

# 存储过程
-  使用存储过程可以把多条语句保存在一个单独的过程中，还可以把参数和SQL语句结合使用
-  语法： `creat procedure name(参数名 类型） begin select ....... end;`
  示例
```sql
delimiter //
creat procedure test(a int)
begin
select name from db_1 where id = a;
end;
//
delimiter ;
```
- 注意在命令行创建过程时要使用delimiter 临时改变分隔符
- 使用call 调用存储过程 `call test(10)`
- 删除存储过程命令为 drop `drop procedure test`

