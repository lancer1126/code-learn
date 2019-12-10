# MySQL

## MySQL进阶

### EXPLAIN命令

#### 简介

可以对 SELECT 语句进行分析, 并输出 SELECT 执行的详细信息, 以供开发人员针对性优化.

#### 关键字

1. **id**  
select查询的序列号，包含一组数字，表示查询中执行select子句或操作表的顺序  
    * id相同，执行顺序由上至下  
    * id不同，如果是子查询，id的序号会递增，id值越大优先级越高，越先被执行  
    * id如果相同，可以认为是一组，从上往下顺序执行；在所有组中，id值越大，优先级越高，越先执行
2. **select_type**  
表示查询的类型，主要是用于区别普通查询，联合查询，子查询等复杂查询
    * ***SIMPLE***：简单的select查询，查询中不包括子查询或者UNION
    * ***PRIMARY***：查询中若包含任何复杂的子部分，最外层查询被标记为此
    * ***SUBQUERY***：在SELECT或WHERE列表中包含了子查询
    * ***DERIVED***：在FROM列表中包含的子查询被标记为DERIVED，会递归这些子查询，把结果放在临时表中
    * ***UNION***：若第二个SELECT出现在UNION之后，则被标记为UNION；若UNION包含在FROM子句的子查询中，外层的select责备标记为DERIVED
    * ***UBION RESULT***：从UNION中获取结果的SELECT  
3. **table**  
显示这一行的数据是哪张表的
4. **type**  
显示访问类型，显示的结果值**自好到坏**依次为：  
***system>const>eq_ref>ref***>fulltext>ref_or_null>idnex_merge>unique_subquery>index_subquery>***range>index>ALL***  
    * ***system***：表只有一行记录（等于系统表），是const类型的特例，可以忽略不计
    * ***const***：通过一次索引就能找到，用于比较primary key或者unique索引，因为只匹配一行数据，所以很快能将主键置于where列表中，mysql就能将该查询转换为一个常量
    * ***eq_ref***：唯一性索引扫描，对于每个索引键，表中只有一条记录与之匹配，常见于主键或唯一索引扫描
    * ***ref***：非唯一性索引扫描，返回匹配某个单独值的所有行。本质上也可以是一种索引访问，它返回所有匹配某个单独值的行，但是可能会找到多个复合条件的行。
    * ***range***：只检索**给定范围**的行，使用一个索引来选择行，key列显示使用了哪个索引  
    一般是在where语句中出现了between，<>，in等的查询
    * ***index***：与ALL一样都是读全表，但区别为index只**遍历索引树**，通常比ALL快，因为索引文件通常比数据文件小。虽然ALL和index都是读全表，但index是从索引中读取的，而ALL是从硬盘中读取的
    * ***ALL***：遍历全表
