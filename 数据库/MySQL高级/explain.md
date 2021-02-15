使用explain关键字可以模拟优化器执行SQL查询语句，从而知道MySQL是如何处理SQL语句的。分析查询语句或者是表结构的性能瓶颈。

1. 作用

   * 表的读取顺序
   * 数据读取操作的操作类型
   * 哪些索引可以使用
   * 哪些索引被实际使用
   * 表之间的引用
   * 每张表有多少行被优化器查询

2. id

   * id相同：执行顺序由上而下
   * id不同：如果是子查询，id的序号会递增，id值越大优先级越高，越先被执行
   * id有相同有不同：id相同的列可以认为是一组，从上往下顺序执行，在所有组中，id值越大，优先级越高，越先执行

3. select_type

   * simple：简单的select查询，查询中不包含子查询或者union
   * primary：查询中若包含任何复杂的子部分，最外层则被标记为primary
   * subQuery：在select或where列表中包含了子查询
   * derived：在from列表中包含的子查询被标记为derived（衍生），MySQL会递归执行这些子查询，把结果放到临时表里
   * union：若第二个select出现在union之后，则被标记为union，若union包含在from子句的查询中，外层的select将被标记为derived
   * union result：从union表中获取结果的select

4. table

   显示这一行的数据是关于哪张表的

5. type

   显示查询使用了何种类型

   system > const > eq_ref > ref > range > index > all

   system：表中只有一行记录，这是const类型的特例，平时不会出现可以忽略

   ~~~mysql
   EXPLAIN select * from (SELECT * FROM m1 WHERE id = '1') d1;
   ~~~

   const：表示通过索引一次就找到了，用于比较primary key或unique索引

   ~~~mysql
   EXPLAIN SELECT * FROM m1 WHERE id = '1';
   ~~~

   eq_ref：唯一性索引扫描，对于每个索引键，表中只有一条记录与之匹配，常见于主键或唯一索引扫描

   ref：非唯一性索引扫描，返回匹配某个单独值的所有行

   ~~~mysql
   create index idx_username on m1(username(20));
   EXPLAIN SELECT * FROM m1 WHERE username = '张三';
   ~~~

   range：使用索引检索给定范围的行，一般出现在where后面between、>、<等查询

   ~~~mysql
   EXPLAIN SELECT * FROM m1 WHERE id between 20 and 30;
   EXPLAIN SELECT * FROM m1 WHERE id  < 10;
   ~~~

   index：扫描全部索引数，比扫描全表快

   ~~~mysql
   EXPLAIN SELECT id FROM m1
   ~~~

   all：扫描全表

   ~~~mysql
   EXPLAIN SELECT * FROM m1
   ~~~

6. possible_keys

   显示可能应用在这张表的索引，一个或多个

   查询涉及到的字段上若存在索引，则被列出，但不一定被查询实际使用

7. key

   实际使用的索引，如果为null，则没有使用索引

   查询的字段为索引列，会出现possible_keys为null，key不为null，即覆盖索引

   ~~~mysql
   EXPLAIN SELECT username FROM m1;
   ~~~

8. ken_len

   表示索引中使用的字节数，可通过该列计算查询中使用的索引长度，在不损失精确性的情况下，长度越短越好（条件越少越短）

   ken_len显示的值为索引字段的最大可能长度，并非实际使用长度。

9. ref

   显示索引的哪一列被使用了，如果可能的话，是一个常数。哪些列或常量被用于查找索引列上的值

10. rows

    根据表统计信息及索引选用情况，大致估算出找到所需的记录需要读取的行数

11. Extra

    包含不适合在其他列中显示但十分重要的额外信息

    * Using filesort（重要）

      说明mysql会对数据使用一个外部的索引排序，而不是按照表内的索引顺序进行读取。MySQL中无法利用索引完成的排序操作成为“文件排序”

    * Using temporary（重要）

      使用了临时表保存中间结果，MySQL在对查询结果排序时使用临时表，常见于排序order by和分组查询group by

    * Using index（重要）

      表示相应的select操作中使用了覆盖索引（Covering index），避免访问了表的数据行，效率不错

      如果同时出现using where，表名索引被用来执行索引键值的查找

      如果没有同时出现using where，表名索引用来读取数据而非执行查找动作

      > 覆盖索引：就是select的数据列只用从索引中就能够取得，不必读取数据行，
      >
      > MySQL可以利用索引返回select列表中的字段，而不必根据索引再次读取数据文件，即查找列要被所建的索引覆盖。不可select *

    * Using where

      表名使用了where过滤

    * Using join buffer

      使用了连接缓存

    * impossible where

      where子句的值总是false，不能用来回去任何数据
      
    * select tables optimized away
    
      在没有group by子句的情况下，基于索引优化min、max，不必等到执行阶段再进行计算，查询执行计划生成的阶段即完成优化
    
    * distinct
    
      优化distinct操作，在找到第一匹配的数据后即停止找同样值的动作

​      

