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

- select中写了多余的列，必须得是聚合键。
  - 不是聚合键，分组后的行可能匹配不上该列（有多个对应）

- 在Group by中写了列的别名
  - select字句在group by后面执行。别名可能看不到就错了

- group by语句结果是无序的。需要排序要在select中指定

- where子句中使用聚合函数
  - 只有select和having中可以使用聚合函数

### 为聚合结果指定条件

- group by +  having
  - select product_type, count(\*) from product group by product_type havind count(\*) =2;

- having   子句构成要素，常数，聚合函数，group by中指定的聚合键
  - 建议聚合键条件写在where里
  - where指定行，having指定组。
  - where更快

### 对查询结果进行排序

- order by  && 排序键
  - select product_id, product_name, sale_price, purchase_price from product order by sal_price
  - select语句的末尾
  - 子句顺序 select ->  from ->   where ->   group by ->   having  ->  order by
  - 执行顺序 from  ->  where  ->  groupby  ->  having ->   select  ->  order by 所以orderby 中可以使用别名
  - 降序desc，默认asc
  - 多个排序键
    - 可以用任何列，聚合函数
    - 列编号不建议使用，不方便读，表也可能改
  - NULL 的顺序，在开头或结尾汇总



## 数据更新

### insert

- insertt into table (col1,...) values(val1...)
  - 多行 ,
  - 省略列list
  - 插入NULL 约束没有not null
  - 插入默认值create table 带default

- 从其他表中赋值数据 insert into (collist..) select(collist...) from table;
  - select里的子句where group by也可用

### delete

- delete from table删除表的所有列，指定列是错误的
- delete from table where

### update

- update table set col =  expr ... where ...
  - update product set regist_date = '2019-01-01';所有列
  -  多列更新，col1=expr1,col2=expr2

### 事务

ACID



### 复杂查询

#### 视图

- 感觉视图就是个缓存，理解成指针也行

- create view veiename (vcol1....) as select ...
  - 用到视图的查询 就是select展开的过程，两条以上select，（在视图上制造视图，不建议多重视图这种行为。影响SQL性能）
- 限制
  - 定义视图不能用order by因为表没有顺序
  - 更新操作的限制
    - select子句中未使用distinct
    - from子句中只有一张表
    - 未使用group by
    - 未使用having

- 删除视图 drop view

#### 子查询

- 感觉视图是子查询的alias，子查询更加复杂。也得as起名原则上，匿名应该也可以？

- 标量子查询，where子句不能聚合，那就用个查询把聚合动作封起来。这种查询可以放在任何位置
  - 单行，多行就退化成普通子查询

#### 关联子查询

- 处理多行场景 as发挥真正的用场

  - 注意结合条件的作用域

    ```sql
    select product_type ,product_name, sale_price 
    
    from product as p1
    
    where sale_price > (select avg(sale_price)
    
    				    from product as p2
    
    				    where p1.product_type = p2.product_type
    
    				    group by product_type) ;
    ```


### 函数，谓词，case表达式

#### 函数

- 算数函数 abs mod% round

  - select m,n round(m,n) as resl from table

- 字符串函数

  - 拼接 || select s1,s2,s1||s2 as s12_cat from table; 
    - 不全支持，mysql concat / SQL server +

  - length 不全支持，SQL server 用len

  - lower upper

  - replace (dst,substr, new_substr )语义比较奇怪,目标字符串的一部分换成新的

  - substring(dst from begin for skip)  目标字符串的自定义开头到一段长度

    - 仅PG MySQL支持

      - SQL Server简化，去掉了from for，换成逗号

      - Orcale/DB2进一步简化，改成substr

- 日期函数 差异更大，大家实现的各不相同
  - select current_date current_time current_timestamp
  - extract ( 日期元素 from 日期)
    -  select current_timestamp,extract(year from current_timestamp) as year,......
- 转换函数
  - select cast('0001' as integer) as int_col
  - coalesce 吞掉null select coalesce(null,null,'?') as col1
- 聚合函数

#### 谓词

- like % _ 简单的模式匹配
- between 简写and条件
- is null is not null
- in or的简便写法 not in 
  - 选不出null
  - 子查询
- exists 语法费解，可以使用（not） in代替

```
     select product_name, sale_price 

     from product as P

     where exists (select *//select 1 也行

                             from shopproduct as SP

                             where SP.shop_id = '000c'

                             and SP.product_id = P.product_id)
```



#### case

```
case when expr then expr

    when expr then expr

         ...

    else expr

end
```

```
SELECT product_name,
       CASE WHEN product_type = ' 衣服 '
            THEN 'A:' | | product_type
            WHEN product_type = ' 办公用品 '
            THEN 'B:' | | product_type
            WHEN product_type = ' 厨房用具 '
            THEN 'C:' | | product_type
            ELSE NULL
       END AS abc_product_type
FROM Product;
```

oracle decode

mysql if

### 集合运算



