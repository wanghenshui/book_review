## 基础操作

- 数据定义 create drop alter 增删改 对象或表
- _数据查询_ /变更 select insert update delete 针对数据进行查询插入更改删除 
- 数据控制 commit rollback grant revoke 提交回滚给用户权限取消用户权限
- 约束，类型约束 integer char(n) varchar date 主键约束等

## 查询

#### select

- select as -> project
- distinct 限定去重
- where字句
- 省略from？就是个计算器 select (100+200) *3 as calculation 
  - 临时表。有的RDBMS不支持。

#### 算术运算符和比较运算符

- 就是普通的四则运算,注意和null计算
  - 不能对null使用比较运算符。使用is null/is not null

- <>不等于
- 注意字句中放_运算式子比较_效率低下
- 注意字符比较是字典序。

#### 逻辑运算符

- not 相当于！where price>100 <=> where not price<=100
- and or
- null unknown , just fuck null



## 聚合与排序



#### 对表进行聚合查询 - 聚合函数 count/sum/avg/max/min

- count
  - select count(*) from table *
    - 假如针对某列统计，null行不计入。极端情况0，但是不影响count(*)

- sum /avg
  - select sum(sale_price) from product;
    - null 聚合操作中已经提前处理，并不会计入，结果成null
    - 只能数值类型的列，其他类型，sum avg没有意义

- max/min
  - 可用于任何数据类型的列，时间字符串均可

- 使用聚合函数删除重复值 distinct
  - select count(distinct product_type) from product,必须在括号中



#### 对表进行分组

- group by

  - select col1,col2,col3.... from table group by col1,col2,col3...按照groupby进行聚合统计操作
    - select product_type, count(*) from product group by product_type

  - group by 指定列为聚合键或分组列
  - 按照聚合键重新组织表
  - 如果聚合键是null 空行
  - group by 和where clause where先执行



#### 聚合函数和group by clause有关的常见错误

- select中卸了多余的列

