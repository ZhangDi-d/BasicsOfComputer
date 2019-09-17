-- 1. 什么是 SQL？SQL 有哪些功能？

答案：SQL 代表结构化查询语言，它是访问关系数据库的通用语言，支持数据的各种增删改查操作。SQL 语句可以分为以下子类：

DQL，数据查询语言。这个就是 SELECT 语句，用于查询数据库中的数据和信息。
DML，数据操作语言。包括 INSERT、UPDATE、DELETE 和 MERGE 语句，主要用于数据的增加、修改和删除。
DDL，数据定义语言。主要包括 CREATE、ALTER 和 DROP 语句，用于定义数据库中的对象，例如表和索引。
TCL，事务控制语言；主要包括 COMMIT、ROLLBACK 和 SAVEPOINT 语句，用于管理数据库的事务。
DCL，数据控制语言。主要包括 GRANT 和 REVOKE 语句，用于控制对象的访问权限。

-- 2. 如何查看员工表中的姓名和性别？
```sql
SELECT emp_name, sex FROM employee;
```

-- 3. 如何查看员工表中的所有字段？
```sql
SELECT * FROM employee;
```

-- 4. 如何知道每个员工一年的总收入？
-- 对于Oracle数据库，一般经常对空值处理的函数为NVL，而mysql中常用到的是ifnull，这两个函数相似，其实都是由一个函数衍生而来，那就是COALESCE()函数。
-- COALESCE()函数 定义：返回列表中第一个非null表达式的值。如果所有表达式求值为null，则返回null
```sql
SELECT emp_name, salary * 12 + COALESCE(bonus, 0) FROM employee;
SELECT emp_name, salary * 12 + IFNULL(bonus, 0) FROM employee;
```

-- 5. 如何为查询结果指定一个容易理解标题
```sql
SELECT emp_name AS "姓名", salary * 12 + COALEASE(bonus, 0) "年薪" FROM employee;
```

-- 6. 怎么查看女性员工的信息？
```sql
SELECT * FROM employee where sex ='女';
```

-- 7. 如何查看月薪范围位于 8000 到 12000 之间的员工？
```sql
SELECT * FROM employee where bonux between 8000 and 12000;
```

-- 8. 确认员工中有没有叫做“张三”、“李四” 或“张飞”的人，有的话查出他们的信息。
```sql
SELECT * FROM employee where emp_name in('张三','李四','张飞');
```

-- 9. 只知道某个员工的姓名里有个“云”字，但不知道具体名字，怎么样查看有哪些这样的员工？
```sql
SELECT * FROM employee where emp_name like '%云%';
```

-- 10. 有些员工有奖金（bonus），另一些没有。怎么查看哪些员工有奖金？
```sql
SELECT * FROM employee where bonux is not null ;
```

-- 11. 在前面我们知道了如何查询女员工，如何查看 2010 年 1 月 1 日之后入职的女员工呢？
```sql
SELECT * FROM employee where sex ='女' and hire_date>'2010-01-01';
SELECT * FROM employee where sex ='女' and hire_date> DATE '2010-01-01';
SELECT * FROM employee where sex ='女' and hire_date> STR_TO_DATE('2010-01-01','%Y-%m-%d') ; -- '%Y-%m-%d'
```
-- 补充:
-- 时间转字符串
```sql
select date_format(now(), '%Y-%m-%d');  
```
-- 字符串转时间
```sql
select str_to_date('2016-01-02', '%Y-%m-%d %H');  -- 2016-01-02 00:00:00
```

-- 12. 以下查询会不会出错，为什么？
```sql
SELECT * FROM employee WHERE 1 = 0 AND 1/0 = 1; -- 不会出错，但是查不到任何数据。
```

-- 13. 如何去除查询结果中的重复记录，比返回如员工性别的不同取值？
```sql
SELECT DISTINCT sex FROM employee;
```

-- 14. 查看员工信息的时候，想要按照薪水从高到低显示，怎么实现？
```sql
SELECT * FROM employee ORDER BY salary desc ;
```

-- 15. 在上面的排序结果中，有些人的薪水一样多；对于这些员工，希望再按照奖金的多少进行排序，又怎么实现？
```sql
SELECT * FROM employee ORDER BY salary desc, bonus desc ;
```

-- 16. 员工的姓名是中文，如何按照姓名的拼音顺序进行排序？
```sql
SELECT * FROM employee ORDER BY CONVERT(emp_name USING gbk);
```
-- 补充:
-- Oracle 实现
```sql
SELECT emp_name
  FROM employee
 ORDER BY NLSSORT(emp_name,'NLS_SORT = SCHINESE_PINYIN_M');
```

-- 17. 由于很多人没有奖金，bonus 字段为空，对于下面的查询： 没有奖金的员工排在最前面还是最后面？
```sql
SELECT * FROM employee ORDER BY bonus;
```
-- 对于 MySQL ，升序时 NULL 值排在最前面，降序时 NULL 值排在最后面。对于 Oracle，默认升序排序时时 NULL 值排在最后面，降序时 NULL 值排在最前面；还可以使用 NULLS FIRST 或 NULLS LAST 指定 NULL 值排在最前或最后。

-- 18. 薪水最高的 3 位员工都有谁？
```sql
SELECT * FROM employee order by salary desc limit 0,3 ;
SELECT * FROM employee order by salary desc limit 3 ;
```
-- 补充:
mysql limit
```sql
SELECT * FROM table LIMIT 5,10; // 检索记录行 6-15 
SELECT * FROM table LIMIT 95,-1; // 检索记录行 96-last.   
SELECT * FROM table LIMIT 5; //检索前 5 个记录行,LIMIT n 等价于 LIMIT 0,n。  
```
-- -- Oracle 12c 实现
```sql
SELECT emp_name, salary FROM employee  ORDER BY salary DESC FETCH NEXT 3 ROWS ONLY;
```

-- 19. 在上面的问题中，如果有 2 个人的排名都是第 3 位，怎么才能都返回（一共 4 条数据）？
-- Oracle 12c 实现
```sql
SELECT emp_name, salary FROM employee ORDER BY salary DESC FETCH NEXT 3 ROWS WITH TIES;
```

-- 20. 怎么返回第 11 名到 15 名，也就是实现分页显示的效果？
```sql
SELECT * FROM employee order by salary desc limit 10,5 ;
SELECT * FROM employee order by salary desc limit 5 offset 10;
```

-- 21. 什么是函数？SQL 中的函数有哪些分类？
-- 在 SQL 中，函数主要分为两种类型：标量函数（scalar function）和聚合函数（aggregate function）。标量函数针对每一行输入参数，返回一行输出结果。例如，ABS 函数可以计算绝对值。聚合函数针对一组数据进行操作，并且返回一个汇总结果。例如，AVG 函数可以计算一组数据的平均值。

-- 22. 如何知道每个员工的邮箱长度？
```sql
SELECT emp_name,LENGTH(email)FROM employee;
```

-- 23. 如何确认谁的邮箱是“GUANXING@SHUGUO.COM”？
```sql
SELECT * FROM employee where UPPER(email)='GUANXING@SHUGUO.COM'; 
```

-- 24. 以 CSV（逗号分隔符）格式显示员工的姓名、性别、薪水信息，如何写 SQL 查询语句？
```sql
SELECT CONCAT(emp_name,',',sex,',',salary) FROM employee;
SELECT CONCAT_ws(',',emp_name,sex,salary) FROM employee;
```

-- 25. 如何获取员工邮箱中的用户名部分（ @ 符号之前的字符串）？
```sql
SELECT emp_name,SUBSTR(email,1,INSTR(email,'@')-1) FROM employee ;-- INSTR 函数查找 @ 符号的位置，SUBSTR 函数获取该位置之前的子串。
```

-- 26. 将员工邮箱中的“.com”替换为“.net”，写出 SQL 语句？
```sql
SELECT emp_name,REPLACE(email,'.com','.net') FROM employee ;
```

-- 27. 如何返回随机排序的员工信息？
```sql
SELECT emp_name, RAND() FROM employee ORDER BY RAND();
```

-- 28. 数学函数 CEILING、FLOOR 和 ROUND 有什么区别？
```sql
SELECT CEILING(1.1), FLOOR(1.1), ROUND(1.1) FROM employee WHERE emp_id = 1; -- CEILING 向上取整，FLOOR 向下取整，ROUND 四舍五入。Oracle 中使用 CEIL 函数替代 CEILING。
```

-- 29. 下图是一个学生成绩表（score），如何知道每个学生的最高得分？ 
![](https://images.gitbook.cn/806c7cb0-ca62-11e9-b421-bb73d778c438) 
```sql
SELECT student_id, GREATEST(chinese, math, english, history)  FROM score;
```

-- 30. 如何知道每个员工的工作年限？
-- CURRENT_DATE 函数返回当前日期，EXTRACT 函数可以提取日期数据中的各个部分，本例中使用 year 参数获取年份信息
```sql
SELECT emp_name, EXTRACT( year FROM CURRENT_DATE) - EXTRACT( year FROM HIRE_DATE) FROM employee;
```

-- 31. 工资信息比较敏感，不宜直接显示。按照范围显示收入水平，小于 10000 显示为“低收入”，大于等于 10000 并且小于 20000 显示为“中等收入”，大于 20000 显示为“高收入”。如何使用 SQL 实现？
```sql
SELECT emp_name,case when salary>20000 then '高收入' when salary>10000 then '中等收入' else '低收入'  end  '薪水等级' FROM employee ;
```

-- 32. 如何统计员工的数量、平均月薪、最高月薪、最低月薪以及月薪的总和？
```sql
SELECT count(emp_name),sum(salary)/12,max(salary) ,min(salary),sum(salary) from employee;
```

-- 33. 以下两个 COUNT 函数返回的结果是否相同？
```sql
SELECT COUNT(*), COUNT(bonus) FROM employee; -- 除了 COUNT (*) 之外，其他聚合函数都会忽略字段中的 NULL 值。
```

-- 34. 群发邮件时，多个邮件地址使用分号进行分隔。如何获取所有员工的群发邮件地址？
-- MYSQL 8
```sql
SELECT GROUP_CONCAT(email SEPARATOR ';') from employee;
```
-- oracle 12 
```sql
select listagg(email,';') within GROUP (order by null ) from employee;
```

-- 35. 如何获取每个部门的统计信息，比如员工的数量、平均月薪？
```sql
select dept_id,count(dept_id) as '部门员工数量',AVG(salary) from employee group by dept_id ;
```

-- 36. 以下语句能否正常运行，为什么？
```sql
SELECT dept_id, COUNT(*), emp_name FROM employee GROUP BY dept_id; -- 不能运行:使用了 GROUP BY 分组之后，SELECT 列表中只能出现分组字段和聚合函数，不能再出现其他字段。
```

-- 37. 如果只想查看平均月薪大于 10000 的部门，怎么实现？
```sql
select dept_id,count(dept_id) as '部门员工数量',AVG(salary) as '平均月薪' from employee group by dept_id HAVING AVG(salary) >10000;
```

-- 38. 如果想要知道哪些部门月薪超过 5000 的员工数量大于 5，如何写 SQL 查询？
```sql
select dept_id, COUNT(*) from employee where salary>5000 group by dept_id having COUNT(*) >5 ;
```

-- SQL 高级查询
-- 39. 什么是连接查询？SQL 中有哪些连接查询？
连接（join）查询是基于两个表中的关联字段将数据行拼接到一起，可以同时返回两个表中的数据。SQL 支持以下连接：
内连接（INNER JOIN），用于返回两个表中满足连接条件的数据行。
左外连接（LEFT OUTER JOIN），返回左表中所有的数据行；对于右表中的数据，如果没有匹配的值，返回空值。
右外连接（RIGHT OUTER JOIN），返回右表中所有的数据行；对于左表中的数据，如果没有匹配的值，返回空值。
全外连接（FULL OUTER JOIN），等价于左外连接加上右外连接，返回左表和右表中所有的数据行。MySQL 不支持全外连接。
交叉连接（CROSS JOIN），也称为笛卡尔积（Cartesian product），两个表的笛卡尔积相当于一个表的所有行和另一个表的所有行两两组合，结果的数量为两个表的行数相乘。
自连接（Self Join），是指连接操作符的两边都是同一个表，即把一个表和它自己进行连接。自连接主要用于处理那些对自己进行了外键引用的表。

-- 40. 如何通过内连接返回员工所在的部门名称？
```sql
select em.emp_name,dept.dept_name from employee em,department dept where  em.dept_id = dept.dept_id  ;
```

-- 41. 统计每个部门的员工数量，同时显示部门名称信息。如何使用连接查询实现？
-- 由于某些部门可能还没有员工，不能使用内连接，而需要使用左外连接或者右外连接；
```sql
select b.dept_id,b.dept_name,count(a.emp_name) from employee a right join department b 
on  a.dept_id = b.dept_id  group by b.dept_id;
```

-- 42. 如何知道每个员工的经理姓名（manager）？
```sql
select a.emp_name as '员工姓名',b.emp_name as '经理姓名'from employee a left join employee b on a.manager = b.emp_id  
```

-- 43. SQL 支持哪些集合运算？
SQL 中提供了以下三种集合运算：
并集运算（UNION、UNION ALL），将两个查询结果合并成一个结果集，包含了第一个查询结果以及第二个查询结果中的数据。
交集运算（INTERSECT），返回两个查询结果中的共同部分，即同时出现在第一个查询结果和第二个查询结果中的数据。MySQL 不支持 INTERSECT。
差集运算（EXCEPT），返回出现在第一个查询结果中，但不在第二个查询结果中的数据。MySQL 不支持 EXCEPT，Oracle 使用 MINUS 替代 EXCEPT。

-- 44. 假设存在以下两个表：
```sql
CREATE TABLE t1(id int);
INSERT INTO t1 VALUES (1);
INSERT INTO t1 VALUES (2);

CREATE TABLE t2(id int);
INSERT INTO t1 VALUES (1);
INSERT INTO t1 VALUES (3);
```
下列查询的结果分别是什么？

-- Oracle 实现
```sql
SELECT id FROM t1 UNION SELECT id FROM t2;  -- 1,2,3
SELECT id FROM t1 UNION ALL SELECT id FROM t2; -- 1,1,2,3  UNION 的结果集中删除了重复的数据，UNION ALL 保留了所有的数据。
SELECT id FROM t1 INTERSECT SELECT id FROM t2;-- 1
SELECT id FROM t1 MINUS SELECT id FROM t2;-- 2
```

-- 45. 对于 MySQL 而言，如何实现上题中的交集运算和差集运算效果？
-- 使用连接查询实现交集运算
```sql
SELECT t1.id FROM t1 JOIN t2 ON (t1.id = t2.id);
```

-- 使用左连接查询实现差集运算
```sql
SELECT t1.id FROM t1
  LEFT JOIN t2 ON (t1.id = t2.id)
 WHERE t2.id IS NULL;
```

-- 46. 什么是子查询？子查询有哪些类型？
子查询（subquery）是指嵌套在其他语句（SELECT、INSERT、UPDATE、DELETE、MERGE）中的 SELECT 语句。子查询中也可以嵌套另外一个子查询，即多层子查询。
子查询可以根据返回数据的内容分为以下类型：
标量子查询（scalar query）：返回单个值（一行一列）的子查询。上面的示例就是一个标量子查询。
行子查询（row query）：返回包含一个或者多个值的单行结果（一行多列），标量子查询是行子查询的特例。
表子查询（table query）：返回一个虚拟的表（多行多列），行子查询是表子查询的特例。

基于子查询和外部查询的关系，也可以分为以下两类：关联子查询（correlated subqueries）和非关联子查询（non-correlated subqueries）。关联子查询会引用外部查询中的列，因而与外部查询产生关联；非关联子查询与外部查询没有关联。
-- 47. 如何找出月薪大于平均月薪的员工？
```sql
select * from employee where salary > (select avg(salary) from employee)
```

-- 48. 以下查询语句的结果是什么？
```sql
SELECT * FROM employee WHERE dept_id = (SELECT dept_id FROM department); -- 执行出错。
```

-- 49. 哪些员工的月薪高于本部门的平均值？
```sql
SELECT * FROM employee a inner join 
(select dept_id,AVG(salary) as avgSalary from employee GROUP by dept_id) b
on a.dept_id = b.dept_id where a.salary > b.avgSalary;
```

-- 50. 显示员工信息时，增加一列，用于显示该员工所在部门的人数。如何编写 SQL 查询？
```sql
select emp_name,(SELECT COUNT(*) FROM employee WHERE dept_id = e.dept_id) from employee e ;
```

-- 51. 以上问题能否使用下面的查询实现？
```sql
SELECT emp_name, dept_count FROM employee e
  JOIN (SELECT COUNT(*) AS dept_count
           FROM employee
          WHERE dept_id = e.dept_id) d
    ON (1=1);   -- 执行出错;
```

-- 52. 找出哪些部门中有女性员工？
```sql
SELECT * FROM department d where d.dept_id in(select distinct dept_id from employee where sex = '女');
SELECT * FROM department d WHERE EXISTS (SELECT 1 FROM employee e WHERE e.sex ='女' AND e.dept_id = d.dept_id);
```

-- 53. 按照部门和职位统计员工的数量，同时统计部门所有职位的员工数据，再加上整个公司的员工数量。如何用一个查询实现？
-- mysql :GROUP BY 支持扩展的 ROLLUP 选项，可以生成按照层级进行汇总的结果，类似于财务报表中的小计、合计和总计。MySQL 中使用 WITH ROLLUP，与 SQL 标准不太一致。
```sql
SELECT dept_id, job_id, COUNT(*) FROM employee GROUP BY dept_id, job_id WITH ROLLUP;
```
-- oracle 
```sql
SELECT dept_id, job_id, COUNT(*) FROM employee GROUP BY ROLLUP (dept_id, job_id);
```

-- 54. GROUP BY 中的另一个选项 CUBE 的作用是什么？
-- CUBE 用于生成多维立方体式的汇总统计。例如，以下查询统计不同部门和职位员工的数量，同时统计部门所有职位的员工数据，加上所有职位的员工数据，以及整个公司的员工数量。
-- Oracle 实现
```sql
SELECT dept_id, job_id, COUNT(*) FROM employee GROUP BY CUBE (dept_id, job_id);
```

-- 55. 使用扩展分组时，会产生一些 NULL 值，如何确认这些 NULL 值代表的意义？
-- 补充:GROUPING 函数用于判断某个统计结果是否与该字段有关。如果是，函数返回 0；否则返回 1。
比如第 3 行数据是所有职位的统计，与职位无关。然后使用 CASE 表达式进行转换显示。
```sql
SELECT CASE GROUPING(dept_id) WHEN 1 THEN '所有部门' ELSE dept_id END, 
       CASE GROUPING(job_id) WHEN 1 THEN '所有职位' ELSE job_id END,
       COUNT(*)
  FROM employee
 GROUP BY dept_id, job_id WITH ROLLUP;   
```

-- 56.如何使用 SQL 查询生成以下连续的数字序列？
![](https://images.gitbook.cn/a8d9e9d0-ca62-11e9-8b48-2f3e78c99db0)
</br>
```sql
WITH RECURSIVE cte (n) AS
(
  SELECT 1 FROM dual
  UNION ALL
  SELECT n + 1 FROM cte WHERE n < 10
)
SELECT * FROM cte;
```
通用表表达式（WITH 子句）是一个在语句级别的临时结果集。定义之后，相当于有了一个表变量，可以在语句中多次引用该通用表表达式。递归（RECURSIVE）形式的通用表表达式可以用于生成序列，遍历层次数据或树状结构的数据。Oracle 中省略 RECURSIVE 即可。

-- 57. 如何获取员工在公司组织结构中的结构图，也就是从最高领导到员工的管理路径？
-- MySQL 实现
```sql
WITH RECURSIVE employee_paths (emp_id, emp_name, path) AS
(
  SELECT emp_id, emp_name, CAST(emp_name AS CHAR(200))
    FROM employee
    WHERE manager IS NULL
  UNION ALL
  SELECT e.emp_id, e.emp_name, CONCAT(ep.path, '->', e.emp_name)
    FROM employee_paths ep 
    JOIN employee e
      ON ep.emp_id = e.manager
)
SELECT * FROM employee_paths ORDER BY path;
```
同样是利用递归通用表表达式实现数据的遍历。Oracle 中省略 RECURSIVE 即可。通用表表达式是 SQL 中非常强大的功能，可以帮助我们简化复杂的连接查询和子查询，并且可以完成递归处理和层次遍历。

-- 58. 什么是窗口函数？有哪些常见的窗口函数？
窗口函数（Window function）也称为分析函数。与聚合函数类似，窗口函数也是基于一组数据进行分析；但是，窗口函数针对每一行数据都会返回一个结果。窗口函数为 SQL 提供了强大的数据分析功能。
常见的窗口函数包括聚合窗口函数和专用的窗口函数。前者就是将聚合函数作为窗口函数使用，包括：COUNT、MIN、MAX、AVG 以及 SUM 等。
专用窗口函数主要包括 ROW_NUMBER、RANK、DENSE_RANK、PERCENT_RANK、CUME_DIST、NTH_VALUE、NTILE、FIRST_VALUE、LAST_VALUE、LEAD 以及 LAG 等。

-- 59. 查询员工的月薪，同时返回该员工所在部门的平均月薪。如何使用聚合函数实现？
```sql
select emp_name, salary , AVG(salary) over (partition by dept_id) from employee ;
```
窗口函数 AVG 基于部门（dept_id）分组后的数据计算平均月薪，为每个员工返回一条记录。窗口函数中的 PARTITION BY 作用类似于 GROUP BY 子句。虽然也可以使用关联子查询与聚合函数实现相同的功能，显然窗口函数更加简单易懂。

-- 60. 查询员工的月薪，同时计算其月薪在部门内的排名？
```sql
select emp_name, salary , dept_id,rank() over (partition by dept_id order by salary desc) from employee ;
select emp_name, salary , dept_id,DENSE_RANK() OVER (PARTITION BY dept_id ORDER BY salary DESC) from employee ;
select emp_name, salary , dept_id,ROW_NUMBER() OVER (PARTITION BY dept_id ORDER BY salary DESC) from employee ;
select emp_name, salary , dept_id,
	ROW_NUMBER() OVER (PARTITION BY dept_id ORDER BY salary DESC),
	rank() over (partition by dept_id order by salary desc),
	DENSE_RANK() OVER (PARTITION BY dept_id ORDER BY salary DESC) from employee ;
```
ROW_NUMBER、RANK 和 DENSE_RANK 都可以用于计算排名。它们不同之处在于对排名相同的数据处理方式不一样。
比如说 10、9、9、8 这四个数，ROW_NUMBER 一定会排出不同的名次（1、2、3、4）；RANK 对于相同的数据排名相同（1、2、2、4）；DENSE_RANK 对于相同的数据排名相同，并且后面的排名不会跳跃（1、2、2、3）。


-- 61. 查询员工的入职日期，同时计算其部门内在该员工之前一个和之后一个入职的员工？
```sql
SELECT emp_name, dept_id, hire_date, 
       LAG(emp_name, 1) OVER (PARTITION BY dept_id ORDER BY hire_date),
       LEAD(emp_name, 1) OVER (PARTITION BY dept_id ORDER BY hire_date)
  FROM employee;
```
LAG 和 LEAD 用于返回排名中相对于当前行的指定偏移量之前和之后的数据。

-- 62. 查询员工的月薪，同时计算其部门内到该员工为止的累计总月薪？
```sql
select emp_id,emp_name,dept_id, hire_date,salary,salary*TIMESTAMPDIFF(MONTH,hire_date, DATE_FORMAT(now(), '%Y-%m-%d')) from employee ; -- 计算的是该员工到目前为止的总月薪
SELECT emp_name, dept_id, salary, SUM(salary) OVER (PARTITION BY dept_id ORDER BY NULL ROWS UNBOUNDED PRECEDING) FROM employee;
```
窗口函数支持定义窗口范围，UNBOUNDED PRECEDING 表示从分组内的第一行到当前行，可以用于计算累计值。

-- 补充:
1.获取当前日期 
```sql
SELECT NOW(),CURDATE(),CURTIME();
```
2.获取前一天
```sql
SELECT  DATE_SUB(CURDATE(),INTERVAL 1 DAY);
```
3.获取后一天
```sql
SELECT  DATE_SUB(CURDATE(),INTERVAL -1 DAY);
```
4.年份差
```sql
SELECT TIMESTAMPDIFF(YEAR,'2019-05-01', DATE_FORMAT(now(), '%Y-%m-%d'));
```
5.月份差
```sql
SELECT TIMESTAMPDIFF(MONTH,'2019-05-01', DATE_FORMAT(now(), '%Y-%m-%d'));
```
6.天数差
```sql
SELECT datediff(DATE_FORMAT(now(), '%Y-%m-%d'),DATE_FORMAT('2018-09-10','%Y-%m-%d'));
SELECT TIMESTAMPDIFF(DAY,'2018-09-10', DATE_FORMAT(now(), '%Y-%m-%d'));
```

-- 63. 查询员工的月薪，同时计算其部门内按照月薪排序后，前一个员工、当前员工以及后一个员工的平均月薪？
```sql
SELECT emp_name, dept_id, hire_date, salary,
       AVG(salary) OVER (PARTITION BY dept_id order by salary rows BETWEEN 1  preceding and 1 following )
  FROM employee;
```

-- 设计与开发
-- 64. 什么是数据库（Database）？什么是数据库管理系统（DBMS）？
数据库（Database）是各种数据的集合，按照一定的数据结构进行存储和管理；数据库管理系统（Database Management System）是用于管理数据库的软件，负责数据库的创建、查询、修改等管理操作。

-- 65. 什么是关系数据库？
关系数据库是指基于关系模型的数据库。在关系模型中，用于存储数据的逻辑结构就是二维表（Table）。表由行和列组成，行也称为记录，代表了单个实体；列也称为字段，代表了实体的某些属性。关系数据库使用 SQL 作为标准语言，执行数据的增删改查以及各种管理操作。关系数据库还定义了三种约束完整性：实体完整性、参照完整性以及用户定义完整性。

-- 66. 关系数据库有哪些完整性约束？
关系数据库定义了以下约束：
非空约束（NOT NULL），用于限制字段不会出现空值。比如员工姓名不能为空。
唯一约束（UNIQUE），用于确保字段中的值不会重复。例如，每个员工的电子邮箱不能重复。每个表可以有多个唯一约束。
主键约束（Primary Key），主键是唯一标识表中每一行的字段。例如员工编号，部门编号等。主键字段必须唯一且非空，每个表可以有且只能有一个主键。
外键约束（FOREIGN KEY），用于表示两个表之间的引用关系。例如，员工属于部门，因此员工表中的部门编号字段可以定义为外键，它引用了部门信息表中的主键。
检查约束（CHECK），可以定义更多用户自定义的业务规则。例如，薪水必须大于 0 ，性别只能是男和女等。
默认值（DEFAULT），用于向字段中插入默认数据。

-- 67. OLTP 和 OLAP 的区别？
..


-- 68. 什么是数据库规范化，有哪些常见的数据库范式？
数据库规范化是一种数据库设计的方法，用于有效地组织数据，减少数据的冗余和相互之间的依赖，增加数据的一致性。由于非规范化的数据库存在冗余，可能导致数据的插入、删除、修改异常等问题，因此引入了规范化过程。

数据库规范化的程度被称为范式（Normal Form），目前已经存在第一范式到第六范式，每个范式都是基于前面范式的增强。
第一范式（First Normal Form），表中的每个属性都是单值属性，每个记录都唯一，也就是需要主键。举例来说，如果员工存在工作邮箱和个人邮箱，不能都放到一个字段，而需要拆分成两个字段；
第二范式（Second Normal Form），首先需要满足第一范式，且不包含任何部分依赖关系。举例来说，如果将学生信息和选课信息放在一起，学号和课程编号可以作为复合主键；但此时学生的其他信息依赖于学号，即主键的一部分。通常使用单列主键可以解决部分依赖问题；
第三范式（Third Normal Form），首先需要满足第二范式，并且不存在传递依赖关系。举例来说，如果将部门信息存储在每个员工记录的后面，那么部门名称依赖部门编号，部门编号又依赖员工编号，这就是传递依赖。解决的方法就是将部门信息单独存储到一个表中；
更高的范式包括 Boyce-Codd 范式、第四范式、第五范式以及第六范式等，不过很少使用到这些高级范式。对于大多数系统而言，满足第三范式即可。
另外，反规范化（Denormalization）是在完成规范化之后执行的相反过程。反规范化通过增加冗余信息，减少 SQL 连接查询的次数，从而减少磁盘 IO 来提高查询时的性能。但是反规范化会导致数据的重复，需要更多的磁盘空间，并且增加了数据维护的复杂性。


-- 69. 什么是实体关系图（ERD）？
实体关系图是一种用于数据库设计的结构图，它描述了数据库中的实体，以及这些实体之间的相互关系。实体代表了一种对象或者概念。例如，员工、部门和职位可以称为实体。每个实体都有一些属性，例如员工拥有姓名、性别、工资等属性。
关系用于表示两个实体之间的关联。例如，员工属于部门。三种主要的关系是一对一、一对多和多对多关系。例如，一个员工只能属于一个部门，一个部门可以有多个员工，部门和员工是一对多的关系。

ERD 也可以按照抽象层次分为三种：

概念 ERD，即概念数据模型。概念 ERD 描述系统中存在的业务对象以及它们之间的关系。
逻辑 ERD，即逻辑数据模型。逻辑 ERD 是对概念数据模型进一步的分解和细化，明确定义每个实体中的属性并描述操作和事务。
物理 ERD，即物理数据模型。物理 ERD 是针对具体数据库的设计描述，需要为每列指定类型、长度、可否为空等属性，为表增加主键、外键以及索引等约束。

-- 70. 数据库常见对象有哪些？
表（Table）、视图（View）、序列（Sequence）、索引（Index）、存储过程（Stored Procedure）、触发器（Trigger）、用户（User）以及同义词（Synonym）等等。其中，表是关系数据库中存储数据的主要形式。

-- 71. 常见 SQL 数据类型有哪些？
SQL 定义了大量的数据类型，其中最常见的类型包括字符类型、数字类型、日期时间类型和二进制数据类型。

字符数据类型，分为固定长度的 CHAR(n) 、可变长度的 VARCHAR(n) 以及字符大对象 CLOB。
数字数据类型，分为精确数字 INTEGER、BIGINT、NUMERIC 以及近似数字 FLOAT、DOUBLE PRECISION 等。
日期时间类型，分为日期 DATE、时间 TIME 以及时间戳 TIMESTAMP 。
二进制数据类型，主要是 BLOB。用于存储图片、文档等二进制数据 。
主流的数据库都支持这些常见的数据类型，但是在类型名称和细节上存在一些差异。另外，SQL 还提供其他的数据类型，例如 XML、JSON 以及自定义的数据类型。

-- 72. CHAR 和 VARCHAR 类型的区别？
CAHR 是固定长度的字符串，如果输入的内容不够使用空格进行填充，通常用于存储固定长度的编码；VARCHAR 是可变长度的字符串，通常用于存储姓名等长度不一致的数据。Oracle 中使用 VARCHAR2 表示变长字符串。
Oracle自己开发了一个数据类型VARCHAR2，这个类型不是一个标准的VARCHAR，它将在数据库中varchar列可以存储空字符串的 特性改为存储NULL值。如果你想有向后兼容的能力，Oracle建议使用VARCHAR2而不是VARCHAR。
	- 1.varchar2把所有字符都占两字节处理(一般情况下)，varchar只对汉字和全角等字符占两字节，数字，英文字符等都是一个字节；
	- 2.VARCHAR2把空串等同于null处理，而varchar仍按照空串处理；
	- 3.VARCHAR2字符要用几个字节存储，要看数据库使用的字符集，
大部分情况下建议使用varchar2类型，可以保证更好的兼容性。

-- 73. 如何创建一个表？
SQL 中创建表的基本语句如下：
```sql
CREATE TABLE table_name
(
  column_1 data_type column_constraint,
  column_2 data_type,
  ...,
  table_constraint
);
```

其中 table_name 指定了表的名称，括号内是字段的定义，创建表时可以指定字段级别的约束（column_constraint）和表级别的约束（table_constraint）。以下是员工表（employee）的创建语句：
```sql
CREATE TABLE employee
    ( emp_id    INTEGER NOT NULL PRIMARY KEY
    , emp_name  VARCHAR(50) NOT NULL
    , sex       VARCHAR(10) NOT NULL
    , dept_id   INTEGER NOT NULL
    , manager   INTEGER
    , hire_date DATE NOT NULL
    , job_id    INTEGER NOT NULL
    , salary    NUMERIC(8,2) NOT NULL
    , bonus     NUMERIC(8,2)
    , email     VARCHAR(100) NOT NULL
    , CONSTRAINT ck_emp_sex CHECK (sex IN ('男', '女'))
    , CONSTRAINT ck_emp_salary CHECK (salary > 0)
    , CONSTRAINT uk_emp_email UNIQUE (email)
    , CONSTRAINT fk_emp_dept FOREIGN KEY (dept_id) REFERENCES department(dept_id)
    , CONSTRAINT fk_emp_job FOREIGN KEY (job_id) REFERENCES job(job_id)
    , CONSTRAINT fk_emp_manager FOREIGN KEY (manager) REFERENCES employee(emp_id)
    ) ;
```

-- 74. 如何基于已有的表复制一个表？
```sql
CREATE TABLE table_name
    AS
SELECT ...;
```
查询的结果也会复制到新的表中，如果在查询中使用 WHERE 子句指定一个永不为真的条件，可以创建只有结构的空表。例如，以下语句基于 employee 表创建一个空的新表：
```sql
CREATE TABLE emp_new
    AS
SELECT *
  FROM employee
 WHERE 1 = 0;
```
MySQL 还支持以下语句复制一个空表：
```sql
CREATE TABLE emp_copy LIKE employee;
```

75. 什么是自增列？
自增列（auto increment），也称为标识列（identity column），用于生成一个自动增长的数字。它的主要用途就是为主键提供唯一值。Oracle 使用标准 SQL 中的 GENERATED ALWAYS AS IDENTITY 表示自增列，MySQL 使用关键字 AUTO_INCREMENT 表示自增列。以下示例演示了自增列的使用：
```sql
-- Oracle 实现
CREATE TABLE emp_identity(
  emp_id     INT GENERATED ALWAYS AS IDENTITY, 
  emp_name   VARCHAR(50) NOT NULL,
  PRIMARY KEY (emp_id)
);

-- MySQL 实现
CREATE TABLE emp_identity(
  emp_id     INT AUTO_INCREMENT,
  emp_name   VARCHAR(50) NOT NULL,
  PRIMARY KEY (emp_id)
);
```

-- 76. 如何修改表的结构？
```sql
ALTER TABLE table_name action;
```
其中，action 表示要执行的修改操作，常见的操作包括增加列，修改列，删除列；增加约束，修改约束，删除约束等。例如，以下语句可以为 emp_new 表增加一列：
```sql
ALTER TABLE emp_new
  ADD weight NUMERIC(4,2) DEFAULT 60 NOT NULL;
```

-- 77. 如何删除一个表？
SQL 中删除表的命令如下：
```sql
DROP TABLE table_name;
```
如果被删除的表是其他表的外键引用表，比如部门表（department），需要先删除子表。Oracle 支持级联删除选项，同时删除父表和子表：
```sql
-- Oracle 实现
DROP TABLE department CASCADE CONSTRAINTS;
```

-- 78. DROP TABLE 和 TRUNCATE TABLE 的区别？
DROP TABLE 用于从数据库中删除表，包括表中的数据和表结构自身。同时还会删除与表相关的的所有对象，包括索引、约束以及访问该表的授权。TRUNCATE TABLE 只是快速删除表中的所有数据，回收表占用的空间，但是会保留表的结构。

-- 79. 什么是数据库事务？
在数据库中，事务（Transaction）是指一个或一组相关的操作（SQL 语句），它们在业务逻辑上是一个原子单元。

-- 80. 数据库事务支持哪些隔离级别？
更新丢失，当两个事务同时更新某一数据时，后者会覆盖前者的结果；
脏读，当一个事务正在操作某些数据但并未提交时，如果另一个事务读取到了未提交的结果，就出现了脏读；
不可重复读，第一个事务第一次读取某一记录后，该数据被另一个事务修改提交，第一个事务再次读取该记录时结果发生了改变；
幻象读，第一个事务第一次读取数据后，另一个事务增加或者删除了某些数据，第一个事务再次读取时结果的数量发生了变化。
Oracle 默认的隔离级别为 READ COMMITTED，MySQL 中 InnoDB 存储引擎的默认隔离级别为 REPEATABLE READ。 

-- 81. MySQL 中的 InnoDB 和 MyISAM 存储引擎有什么区别？
InnoDB支持行锁,事务,外键;MyISAM 支持表锁,不支持事务和外键.

-- 82. 如何插入数据？
SQL 主要提供了两种数据插入的方式：
```sql
INSERT INTO … VALUES ...
INSERT INTO … SELECT ...
```

-- 83. 如何修改数据？
```sql
UPDATE table_name
   SET column1 = expr1,
       column2 = expr2,
       ...
[WHERE condition];
```

-- 84. 如何删除数据？
```sql
DELETE FROM table_name
[WHERE conditions];
```

-- 85. 删除数据时，DELETE 和 TRUNCATE 语句的区别？
|DELETE|TRUNCATE|
|:---|:---|
|用于从表中删除指定的数据行。|用于删除表中的所有行，并释放包含该表的存储空间。|
|删除数据后，可以提交或者回滚。|	操作无法回滚。|
|属于数据操作语言（DML）。	|属于数据定义语言（DDL）。|
|删除数据较多时比较慢。	|执行速度很快。|


-- 86. 什么是 MERGE 或者 UPSERT 操作？
MERGE 是 SQL:2003 标准中引入的一个新的数据操作命令，它可以同时完成 INSERT 和 UPDATE 的操作，甚至 DELETE 的功能。

基本的 MERGE 语句如下：
```sql
MEGRE INTO target_table [AS t_alias] 
USING source_table [AS s_alias]
   ON (condition)
 WHEN MATCHED THEN
      UPDATE SET column1 = expr_1,
                 column2 = expr_2,
                 ...
 WHEN NOT MATCHED THEN
      INSERT (column1, column2, ...)
      VALUES (expr_1, expr_2, ...);
```
其中，target_table 是合并的目标表；USING 指定了数据的来源，可以是一个表或者查询结果集；ON 指定了合并操作的判断条件，对于数据源中的每一行，如果在目标表中存在满足条件的记录，执行 UPDATE 操作更新目标表中对应的记录；如果不存在匹配的记录，执行 INSERT 在目标表中插入一条新记录。

-- 87. 什么是索引？有哪些类型的索引？
索引（Index）是一种数据结构，主要用于提高查询的性能。索引类似于书籍最后的索引，它指向了数据的实际存储位置；索引需要占用额外的存储空间，在进行数据的操作时需要额外的维护。另外，索引也用于实现约束，例如唯一索引用于实现唯一约束和主键约束。

不同的数据库支持的索引不尽相同，但是存在一些通用的索引类型，主要包括：
- B/B+ 树索引，使用平衡树或者扩展的平衡树结构创建索引。这是最常见的一种索引，几乎所有的数据库都支持。这种索引通常用于优化 =、<、<=、>、BETWEEN、IN 以及字符串的前向匹配查询。
- Hash 索引，使用数据的哈希值进行索引。主要用于等值（=）和 IN 查询。
- 聚集索引，将表中的数据按照索引的结构（通常是主键）进行存储。MySQL 中称为聚集索引，Oracle 中称为索引组织表（IOT）。
- 非聚集索引，也称为辅助索引。索引与数据相互独立，MySQL InnoDB 中的索引存储的是主键值，Oracle 中存储的时物理地址。
- 全文索引，用于支持全文搜索。
- 唯一索引与非唯一索引。唯一索引可以确保被索引的数据不会重复，可以实现数据的唯一性约束。非唯一索引仅仅用于提高查询的性能。
- 单列索引与多列索引。基于多个字段创建的索引称为多列索引，也叫复合索引。
- 函数索引。基于函数或者表达式的值创建的索引。


-- 88. 如何查看 SQL 语句的执行计划？
MySQL 查看执行计划：
```sql
EXPLAIN
SELECT *
  FROM employee e
 WHERE emp_id = 5;
```

Oracle 查看执行计划：
```sql
EXPLAIN PLAN FOR
SELECT *
  FROM employee e
 WHERE emp_id = 5;
```
具体的sql执行计划解析参考之前的文章.

-- 89. 以下查询语句会不会使用索引？
```sql
CREATE INDEX idx ON test (col);
SELECT COUNT(*)
  FROM test
 WHERE col * 12 = 2400；
```
不会.

-- 95. 什么是可更新视图？
可更新视图是指可以通过对视图的 INSERT、UPDATE、DELETE 等操作，实现对视图对应的基础表的数据修改。通常来说，可更新视图必须是简单的查询语句，不能包含以下内容：
- 聚合函数，例如 SUM、AVG 以及 COUNT 等；
- DISTINCT 关键字；
- GROUP BY 或者 HAVING 子句；
- 集合操作符 UNION 等；
- 不同的数据库特定的限制

简单来说，可能导致无法通过视图找到对应基础表中的数据的操作都不允许。以下语句创建了一个简单的视图，只包含了开发部门的员工信息，并且隐藏了工资等敏感信息：
```sql
CREATE OR REPLACE VIEW emp_devp
    AS
SELECT emp_id, emp_name, sex, manager, hire_date, job_id, email
  FROM employee
 WHERE dept_id = 4
  WITH CHECK OPTION;
  ```
其中的 WITH CHECK OPTION 确保无法通过视图修改超出其可见范围之外的数据。

















