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
    * SIMPLE：简单的select查询，查询中不包括子查询或者UNION
    * PRIMARY：查询中若包含任何复杂的子部分，最外层查询被标记为此
    * SUBQUERY：在SELECT或WHERE列表中包含了子查询
    * DERIVED：在FROM列表中包含的子查询被标记为DERIVED，会递归这些子查询，把结果放在临时表中
    * UNION：若第二个SELECT出现在UNION之后，则被标记为UNION；若UNION包含在FROM子句的子查询中，外层的select责备标记为DERIVED
    * UBION RESULT：从UNION中获取结果的SELECT
3. **table**  
显示这一行的数据是哪张表的
4. **type**  

