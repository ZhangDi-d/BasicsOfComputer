## Oracle树查询及相关函数

Oracle树查询的最重要的就是select...start with... connect by ...prior 语法了。依托于该语法，我们可以将一个表形结构的中以树的顺序列出来。在下面列述了Oracle中树型查询的常用查询方式以及经常使用的 与树查询相关的Oracle特性函数等，在这里只涉及到一张表中的树查询方式而不涉及多表中的关联等。
以我做过的一个项目中的表为例，表结构如下：
```
-- ----------------------------
-- Table structure for FLFL
-- Create by zhangdi
-- ----------------------------
DROP TABLE "SCOTT"."FLFL";
CREATE TABLE "SCOTT"."FLFL" (
"ID" NUMBER NOT NULL ,
"MC" NVARCHAR2(20) NULL ,
"FLJB" NUMBER NULL ,
"SJFLID" NUMBER NULL 
)
LOGGING
NOCOMPRESS
NOCACHE

;
COMMENT ON COLUMN "SCOTT"."FLFL"."MC" IS '名称';
COMMENT ON COLUMN "SCOTT"."FLFL"."FLJB" IS '级别';
COMMENT ON COLUMN "SCOTT"."FLFL"."SJFLID" IS '父ID';

-- ----------------------------
-- Records of FLFL
-- ----------------------------
INSERT INTO "SCOTT"."FLFL" VALUES ('1', 'NODE-1', '1', null);
INSERT INTO "SCOTT"."FLFL" VALUES ('2', 'NODE-2', '2', '1');
INSERT INTO "SCOTT"."FLFL" VALUES ('3', 'NODE-3', '2', '1');
INSERT INTO "SCOTT"."FLFL" VALUES ('4', 'NODE-4', '3', '2');
INSERT INTO "SCOTT"."FLFL" VALUES ('5', 'NODE-5', '3', '2');
INSERT INTO "SCOTT"."FLFL" VALUES ('6', 'NODE-6', '3', '2');
INSERT INTO "SCOTT"."FLFL" VALUES ('7', 'NODE-7', '3', '3');
INSERT INTO "SCOTT"."FLFL" VALUES ('8', 'NODE-8', '3', '3');
INSERT INTO "SCOTT"."FLFL" VALUES ('9', 'NODE-9', '3', '3');
INSERT INTO "SCOTT"."FLFL" VALUES ('10', 'NODE-10', '4', '4');
INSERT INTO "SCOTT"."FLFL" VALUES ('11', 'NODE-11', '4', '4');
INSERT INTO "SCOTT"."FLFL" VALUES ('12', 'NODE-12', '4', '6');
INSERT INTO "SCOTT"."FLFL" VALUES ('13', 'NODE-13', '4', '7');
INSERT INTO "SCOTT"."FLFL" VALUES ('14', 'NODE-14', '4', '8');
INSERT INTO "SCOTT"."FLFL" VALUES ('15', 'NODE-15', '4', '8');
INSERT INTO "SCOTT"."FLFL" VALUES ('16', 'NODE-16', '5', '11');
INSERT INTO "SCOTT"."FLFL" VALUES ('17', 'NODE-17', '5', '11');
INSERT INTO "SCOTT"."FLFL" VALUES ('18', 'NODE-18', '5', '13');
INSERT INTO "SCOTT"."FLFL" VALUES ('19', 'ELEM-19', '1', null);
INSERT INTO "SCOTT"."FLFL" VALUES ('20', 'ELEM-20', '2', '19');
INSERT INTO "SCOTT"."FLFL" VALUES ('21', 'ELEM-21', '2', '19');
INSERT INTO "SCOTT"."FLFL" VALUES ('22', 'ELEM-22', '3', '20');
INSERT INTO "SCOTT"."FLFL" VALUES ('23', 'ELEM-23', '3', '20');
INSERT INTO "SCOTT"."FLFL" VALUES ('24', 'ELEM-24', '3', '21');
INSERT INTO "SCOTT"."FLFL" VALUES ('25', 'ELEM-25', '4', '24');

-- ----------------------------
-- Checks structure for table FLFL
-- ----------------------------
ALTER TABLE "SCOTT"."FLFL" ADD CHECK ("ID" IS NOT NULL);
```
FLJB是作为树的级别，在很多查询中可以加快SQL的查询效率。在下面演示的功能基本上不使用这个关键字。

SJFLID存储的是上级ID，如果是顶级父节点，该SJFLID为null（得补充一句，当初的确是这样设计的，不过现在知道，表中最好别有null记录，这会引起全文扫描，建议改成0代替)。

我们从最基本的操作，逐步列出树查询中常见的操作，所以查询出来的节点以家族中的辈份作比方。


1. 查找树中的所有顶级父节点（辈份最长的人）。 假设这个树是个目录结构，那么第一个操作总是找出所有的顶级节点，再根据该节点找到其下属节点。
```
SELECT * FROM flfl WHERE sjflid IS NULL;
```
这是个引子，没用到树型查询.


2.查找一个节点的直属子节点（所有儿子）,如果查找的是直属子类节点，也是不用用到树型查询的。
```
SELECT * FROM flfl WHERE sjflid = 1;
```
这个可以找到ID为1的直属子类节点。


3.查找一个节点的所有 直属子节点（所有后代）
```
SELECT * FROM flfl START WITH ID = 1 CONNECT BY sjflid = PRIOR ID;
```
这个查找的是ID为1的节点下的所有直属子类节点，包括子辈的和孙子辈的所有直属节点。


4.查找一个节点的直属父节点（父亲）,如果查找的是节点的直属父节点，也是不用用到树型查询的。
```
SELECT b.* FROM flfl a JOIN flfl b ON a.sjflid = b.ID WHERE a.ID = 16;
```
这个找到的是ID为16的节点的直属父节点，要用到同一张表的关联了。


5.查找一个节点的所有直属父节点（祖宗）
```
SELECT * FROM flfl START WITH ID = 16 CONNECT BY PRIOR sjflid = ID;
```
这里查找的就是ID为16的所有直属父节点，打个比方就是找到一个人的父亲、祖父等。但是值得注意的是这个查询出来的结果的顺序是先列出子类节点再列出父类节点，姑且认为是个倒序吧。


6.查询一个节点的兄弟节点（亲兄弟）
```
SELECT a.* FROM flfl a
 WHERE EXISTS (SELECT * FROM flfl b WHERE a.sjflid = b.sjflid AND b.ID = 5);
```
这里查询的就是与ID为5的节点同属一个父节点的节点了，就好比亲兄弟了。


7.查询与一个节点同级的节点（族兄弟）。 如果在表中设置了级别的字段，上表中的FLJB，那么在做这类查询时会很轻松，同一级别的就是与那个节点同级的，在这里列出不使用该字段时的实现!
```
WITH tmp AS
     (SELECT  a.*, LEVEL lev
            FROM flfl a
      START WITH a.sjflid IS NULL
      CONNECT BY a.sjflid = PRIOR a.ID)
SELECT *
  FROM tmp
 WHERE lev = (SELECT lev
                FROM tmp
               WHERE ID = 5)

```
这里使用两个技巧，一个是使用了LEVEL来标识每个节点在表中的级别，还有就是使用with语法模拟出了一张带有级别的临时表。


8.查询一个节点的父节点的的兄弟节点（伯父与叔父）
```
WITH tmp AS
     (SELECT     flfl.*, LEVEL lev
            FROM flfl
      START WITH sjflid IS NULL
      CONNECT BY sjflid = PRIOR ID)
SELECT b.*
  FROM tmp b,
       (SELECT *
          FROM tmp
         WHERE ID = 12 AND lev = 2) a
 WHERE b.lev = 1
UNION ALL
SELECT *
  FROM tmp
 WHERE sjflid = (SELECT DISTINCT x.ID
                            FROM tmp x,
                                 tmp y,
                                 (SELECT *
                                    FROM tmp
                                   WHERE ID = 12 AND lev > 2) z
                           WHERE y.ID = z.sjflid AND x.ID = y.sjflid);
```

9.查询一个节点的父节点的同级节点（族叔）
```
WITH tmp AS
     (SELECT     a.*, LEVEL lev
            FROM flfl a
      START WITH a.sjflid IS NULL
      CONNECT BY a.sjflid = PRIOR a.ID)
SELECT *
  FROM tmp
 WHERE lev = (SELECT lev
                FROM tmp
               WHERE ID = 12) - 1
```

基本上，常见的查询在里面了，不常见的也有部分了。其中，查询的内容都是节点的基本信息，都是数据表中的基本字段，但是在树查询中还有些特殊需求，是对查询数据进行了处理的，常见的包括列出树路径等。

补充一个概念，对于数据库来说，根节点并不一定是在数据库中设计的顶级节点，对于数据库来说，根节点就是start with开始的地方。

下面列出的是一些与树相关的特殊需求。


10.名称要列出名称全部路径
这里常见的有两种情况，一种是是从顶级列出，直到当前节点的名称（或者其它属性）；一种是从当前节点列出，直到顶级节点的名称（或其它属性）。举地址为例：国内的习惯是从省开始、到市、到县、到居委会的，而国外的习惯正好相反（老师说的，还没接过国外的邮件，谁能寄个瞅瞅 ）。

从顶部开始：
```
SELECT     SYS_CONNECT_BY_PATH (mc, '/')
      FROM flfl
     WHERE ID = 16
START WITH sjflid IS NULL
CONNECT BY sjflid = PRIOR ID;
```

从当前节点开始：

```
SELECT     SYS_CONNECT_BY_PATH (mc, '/')
      FROM flfl
START WITH ID = 16
CONNECT BY PRIOR sjflid = ID;
```


11.列出当前节点的根节点。

在前面说过，根节点就是start with开始的地方。
```
SELECT CONNECT_BY_ROOT mc, flfl.*
      FROM flfl
START WITH ID = 16
CONNECT BY PRIOR sjflid = ID;
```

12.列出当前节点是否为叶子,这个比较常见，尤其在动态目录中，在查出的内容是否还有下级节点时，这个函数是很适用的。
```
SELECT     CONNECT_BY_ISLEAF , flfl.*
      FROM flfl
START WITH sjflid IS NULL
CONNECT BY sjflid = PRIOR ID;
```
connect_by_isleaf函数用来判断当前节点是否包含下级节点，如果包含的话，说明不是叶子节点，这里返回0；反之，如果不包含下级节点，这里返回1。

