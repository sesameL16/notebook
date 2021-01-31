1. 数据库操作

   ~~~mysql
   #创建数据库
   CREATE DATABASE IF NOT EXISTS books 
   DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
   #修改字符集
   ALTER DATABASE books CHARACTER SET gbk;
   #删除数据库
   DROP DATABASE IF EXISTS books;
   ~~~

   

2. 表的创建

   ~~~mysql
   -- 历史情报信息
   DROP TABLE IF EXISTS `t_history_qb`;
   CREATE TABLE `t_history_qb` ( 
   	id 	     INT(19) 	NOT NULL AUTO_INCREMENT	COMMENT '主键ID',
   	docWord  VARCHAR(12)				        COMMENT '类型',
   	title 	 VARCHAR(200)						COMMENT '标题',
   	topic    VARCHAR(500)						COMMENT '主题',
   	PRIMARY KEY (`id`)
   )ENGINE = INNODB DEFAULT CHARSET = UTF8mb4 COMMENT = '历史情报信息';
   ~~~

3. 表的修改

   ~~~mysql
   #修改列名
   ALTER TABLE t_history_qb CHANGE COLUMN title qb_title VARCHAR(200);
   #修改类型和约束
   ALTER TABLE t_history_qb MODIFY COLUMN topic INT;
   #添加新列
   ALTER TABLE t_history_qb ADD COLUMN qb_name VARCHAR(200);
   #删除列
   ALTER TABLE t_history_qb DROP COLUMN qb_name;
   #修改表名
   ALTER TABLE t_history_qb RENAME TO history_qb;
   
   DESC history_qb;
   ~~~

4. 复制表

   ~~~mysql
   #只复制表结构
   CREATE TABLE copy LIKE history_qb;
   #复制表结构和数据
   CREATE TABLE copy1 SELECT * FROM history_qb;
   #只复制部分数据
   CREATE TABLE copy2 SELECT id,docWord FROM history_qb;
   #只复制某些字段
   CREATE TABLE copy3 SELECT id,docWord FROM history_qb where 0;
   ~~~

   