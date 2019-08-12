Oracle函数之Decode

## DECODE含义
```sql
decode(条件,值1,返回值1,值2,返回值2,…值n,返回值n,缺省值)
```
这个是decode的表达式，具体的含义解释为：
```
IF 条件=值1 THEN
　　　　RETURN(翻译值1)
ELSIF 条件=值2 THEN
　　　　RETURN(翻译值2)
　　　　......
ELSIF 条件=值n THEN
　　　　RETURN(翻译值n)
ELSE
　　　　RETURN(缺省值)
END IF
```

## DECODE的用法
这里主要说的就是decode的用法，在很多时候这个函数还是很有用的。

### 1.翻译值
数据截图：
![](https://img-blog.csdn.net/20181016233814490?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NkdXQ0MDY=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

需求：查询出的数据，1表示男生，2表示女生
```sql
select t.id,
       t.name,
       t.age,
       decode(t.sex, '1', '男生', '2', '女生', '其他') as sex
  from STUDENT2 t
```  

结果：
![](https://img-blog.csdn.net/20181017000207200?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NkdXQ0MDY=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)


### 2.decode比较大小
说明：sign(value)函数会根据value的值为0，正数，负数，分别返回0，1,-1
数据：
![](https://img-blog.csdn.net/20181017000119772?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NkdXQ0MDY=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

需求：年龄在20以上的显示20以上，20以下的显示20以下，20的显示正好20
```sql
select t.id,
       t.name,
       t.age,
       decode(sign(t.age - 20),
              1,
              '20以上',
              -1,
              '20以下',
              0,
              '正好20',
              '未知') as sex
  from STUDENT2 t
```

结果：
![](https://img-blog.csdn.net/20181017000509304?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NkdXQ0MDY=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

### 3.decode分段
数据暂无
需求：工资大于5000为高薪，工资介于3000到5000为中等，工资小于3000为底薪
```sql
select name,
       sal,
       decode(sign(sal - 5000),
              1,
              '高薪',
              0,
              '高薪',
              -1,
              decode(sign(sal - 3000), 1, '中等', 0, '中等', -1, '低薪')) as salname
  from person;

```

### 4.搜索字符串
数据：
![](https://img-blog.csdn.net/20181017001106351?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NkdXQ0MDY=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

需求：找到含有三的姓名
```sql
select t.id,
       decode(instr(t.name, '三'), 0, '姓名不含有三', '姓名含有三') as name,
       t.age,
       t.sex
  from STUDENT2 t
```

结果：
![](https://img-blog.csdn.net/20181017001035781?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NkdXQ0MDY=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)


### 5.判断是否为空
数据：
![](https://img-blog.csdn.net/201810170012561?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NkdXQ0MDY=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

需求：性别为空显示“暂无数据”，不为空原样输出
```sql
select t.id,
       t.name,
       t.age,
       decode(t.sex,NULL,'暂无数据',t.sex) as sex
  from STUDENT2 t
```

结果：
![](https://img-blog.csdn.net/20181017001503295?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NkdXQ0MDY=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

-----------------
转载自:https://blog.csdn.net/sdut406/article/details/82795585