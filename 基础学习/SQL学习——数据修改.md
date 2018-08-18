[TOC]

# 插入数据
-  使用insert语句将数据插入表中，insert有两种使用方式
  -§ 插入在insert语句指定的数据
  - 插入一条select语句获取的数据
    -insert语句直接插入的语法为 `insert into 表名 (列 表） values (数据）`，如
    `insert into students (id, name) values (3123,'jack'),(4321,'bob'); `
    注意，关键字values后边的值的顺序要与insert into后边的列 表顺序对应。该语句为没有指定的值赋予NULL
  - 第二种使用方法为插入select语句的返回列`insert into ..... values select .....`，如
    `insert into students (id,name) values select id,name from ......`

# 删除数据
- 执行delete语句时，会删除整个行而非单个的列，格式为`delete from table_name where ......`，如 `delete from table_name where id=1`

# 更新数据
- 更新数据的过程包括指定更新哪个列，以及选中行的更新，使用update语句
- update 一般格式为 `update table set column1 = value1, column2 = value2 ..... where ......`，如 `update student set name = 'ww', id = 231 where id = 5;`

# 表的属性
- 列
  - 表可以设计为包含任意数目的列，每个列有一些针对该列的具体属性
  - 列的第一个属性是列的名称，表中的每个列必须要有一个唯一的名称
    -列的第二个属性是数据类型，数据类型是决定每个列可以包含什么样数据的关键
  - 列的第三个属性是是否自增。自增型的列意味着当表中每增加一行，会自动按照升序序列将一个数值赋给该列。自增型的列通常和主键一起使用
  - 列的第四个属性是是否允许NULL值，默认为允许，可使用关键字NOT NULL来显式指定不允许
  - 列的第五个属性是是否赋予默认值。当添加行时，如果没有为这个列提供一个值，就为该列自动赋一个默认值
- 主键和索引
  - 索引是一种物理结构，可以为数据库表中任意的列添加索引，索引可以加速数据的检索，但需要更多的空间，且数据更新操作的速度会降低
  - 主键是表中行的唯一标识，不允许包含NULL值。主键通常为自增型的列。主键可以跨越多个列，包含了多个列的主键称为复合主键。
  - 指定一个列为主键后，该列成为了索引，并且包含唯一的值
- 外键
  - 外键是从一个表中的一个列到另一个不同表中的一个列的直接引用。当设置外键时，会要求指定两个列。配置了外键列的表称为子表，被引用的列位于父表中
  - 当设置外键时，对父表中涉及到更新和删除的行可以定义三种具体的行为
    - No Action ：对父表进行更新时不更新子表。注意，使用No Actions时更新父表后要检查子表任一行是否指向了父表中不存在的值
    - Cascade：对父表进行跟新时如果影响到了子表中的行，则自动更新子表中的所有行以反映父表的新值。
    - Set Null：当更新或者删除父表中的值是，如果影响到了子表中的行，自动把子表中所有受影响的行的外键更新为包含一个NULL值

# 创建表
```sql
 create table tablename//创建表名为tablename
(
columnname1 varchar(100)  (auto_increment) (primarykey) (not null),//第一列列名为columnname1，类型为varchar(100)，自动增长，主键，非空(括号内为非必须项）
columnname2 double (default 10),//列名为column2，类型为double，默认值为10
......
constraint foreign key(columnname2) references 'table2' (firstcolumn) //将columnname2作为table2的外键，相关联的列名是firstcolumn
);
```



