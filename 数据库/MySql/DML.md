1. 插入

   ~~~mysql
   #多行插入
   insert into departments(department_id,department_name,manager_id,location_id) 
   values (10,'Adm',200,1700),(20,'Mar',201,1800),(30,'Pur',114,1700),(40,'Hum',203,2400);
   
   #子查询插入
   insert into departments(department_id,department_name,manager_id,location_id) 
   select department_id,department_name,manager_id,location_id
   from my_departments
   ~~~

   

2. 删除

   * delete可以加where条件，truncate不能加条件
   * truncate删除效率高一点
   * 假如要删除的表中有自增长列，如果用delete删除后，再插入数据，自增长的值从断点开始，而用truncate删除后，再插入数据，自增长列的值从1开始
   * truncate删除没有返回值，delete删除有返回值
   * truncate删除不能回滚，delete删除可以回滚

   ~~~mysql
   #删除所有数据
   truncate table departments;
   delete from departments;
   
   #多表删除
   delete b from beauty b
   inner join boys bo on b.boyfriend_id = bo.id
   where bo.boyName = '张无忌'
   
   delete beauty,boys from beauty
   inner join boys on beauty.boyfriend_id = boys.id
   where boys.boyName = '张无忌'
   ~~~

3. 多表修改

   ~~~mysql
   update boys bo
   inner join beauty b on b.boyfriend_id = bo.id
   set b.boyfriend_id = 2
   
   update boys bo
   inner join beauty b on b.boyfriend_id = bo.id
   set b.phone = '114'
   where bo.boyName = '张无忌'
   
   ~~~

   