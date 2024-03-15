* **MYSQL**
    * [1.创建索引](#index)




## 1.创建索引

1. 查看索引字段选择性，取值范围为(0, 1]

       SELECT count(DISTINCT(字段名))/count(*)ASSelectivity FROM 表名;
2. 创建前缀索引 

       ALTER TABLE 表名 ADD INDEX`first_name_last_name4`(字段1,字段2(4));
   前缀加到4

       SELECT count(DISTINCT(concat((字段1,left(字段2,4))))/count(*)ASSelectivity FROM 表名;
    但是其缺点是不能用于ORDER BY和GROUP BY操作

