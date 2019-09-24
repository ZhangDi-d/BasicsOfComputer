真正的oracle 大全，ryze。
## Oracle 函数
### 1. 单行函数
#### 1.1 数字函数
ABS(a)  	a的绝对值

ACOS(a) 	a的反余弦值

ASIN(a)		a的反正弦值

ATAN(a) 	a的反正切值

ATAN(a,b) 	反正切值(a,b)

BITAND(a,b) 按位与

CEIL(a) 	a的天花板值

COS(a)   	a的余弦值

COSH(a)   	a的双曲余弦值

EXP(a)  	e的a次幂

FLOOR(a)	a的地板值

LN(a)		以e为底的a的对数

LOG(a,b)	以a为底的b的对数

MOD(a,b)	a/b的余数

NANVL(a,b)	如果a是数字，返回a; 否则返回b

POWER(a,b)  a的b次幂

REMAINDER(a,b) 		a/b的余数,和MOD类似

ROUND(number) 		四舍五入

SIGN(a) 			取a的符号:a>0,返回1;a=0,返回0;a<0,返回-1

SIN(a)				a的正弦值

SINH(a)				a的双曲正弦值

SQRT(a)				a的平凡根

TAN(a)				a的正切值

TANH(a)				a的双曲正切值

TRUNC(number[,decimal])		截取数字：
	trunc(89.975,2)=89.97
	trunc(89.975)=89
	trunc(89.975,-1)=80 	decimal参数为负数时，将小数点左的decimal位置为0

WIDTH_BUCKET(expr,min value,max value,num buckets) 桶函数
	expr 待处理的值
	minvalue 下限值
	maxvalue 上限值
	num buckets  桶的数量
	width_bucket(300,0,2000,4) 返回1  将0到2000分为4个桶，300 位于第一个桶中
		

#### 返回字符值的字符函数
CHR(a) 		将asscii码返回字符
CONCAT(char1,char2) 	拼接字符串
	mysql中，可以多个拼接  concat(char1,char2,char3...)

INITCAP(char)		将char首字母大写返回

LOWER(char1)		转小写

LPAD(string,padded_legnth [,pad_string])	在string左侧拼接pad_string(默认为空格),拼接到指定长度返回

LTRIM(char1 [.char2])  删除char1左侧的char2，默认为空格

NCHR(number)  		和chr类似，返回的总是varchar2

NLS_INITCAP(char1 [,cahr2])			返回char1的第一个字符大写，其余小写

NLS_LOWER(char1 [,cahr2])			返回char的小写

NLS_UPPER(char1 [,cahr2])			返回char的大写

NLSSORT(a [,b])		排序函数
		拼音： select * from table order by nlssort( 排序字段，'nls_sort=schinese_pinyin_m')
		笔划： select * from table order by nlssort( 排序字段，'nls_sort=schinese_stroke_m')
		部首： select * from table order by nlssort( 排序字段，'nls_sort=schinese_radical_m')

REGEXP_REPLACE		与replace 类似

REGEXP_SUBSTR		与substr 类似

REGEXP_LIKE			与like 类似

REGEXP_INSTR		与instr 类似

REPLACE(string,old,new)			替换

RPAD							类似lpad

RTRIM							同ltrim

SOUNDEX(char)		返回字符串的语音表示形式

SUBSTR(char,position,substring_length)		截取字符串

TRANSLATE 与replace 同是替换函数，但是translate一次替换多个单个字符



#### 返回数字值的字符函数
ASCII(char)		char的ascii码

INSTR		字符查找函数
	格式一：instr( string1, string2 )    /   instr(源字符串, 目标字符串)
	格式二：instr( string1, string2 [, start_position [, nth_appearance ] ] )   /   instr(源字符串, 目标字符串, 起始位置, 匹配序号)

LENGTH(char)	返回char的长度

REGEXP_COUNT ( source_char, pattern [, position [, match_param]])	REGEXP_COUNT 返回
pattern 在source_char 串中出现的次数。如果未找到匹配，则函数返回0。

REGEXP_INSTR	正则字符查找函数



#### 字符集函数
NLS_CHARSET_DECL_LEN(byte_count,'char_set_id')
	返回一个 NCHAR 列的声明长度（也就是字符个数）。byte_count 参数是列的宽度。'char_set_id' 参数是字符集 ID。
	
NLS_CHARSET_ID(string)	
	返回字符集对应的 ID。string 参数是 VARCHAR2 值。'CHAR_CS' 的 string 值返回服务器数据库字符集 ID。
	
NLS_CHARSET_NAME(number)
	返回字符集 ID 对应的字符集名称。字符集名称作为 VARCHAR2 值以数据库字符集返回。


#### 日期函数
ADD_MONTHS(date,integer)	在date上增加或者减少integer个月后的时间值

CURRENT_DATE				返回当前会话时区中的当前日期，没有参数和括号		

CURRENT_TIMESTAMP			以timestamp with time zone数据类型返回当前会话时区中的当前日期

DBTIMEZONE					返回时区，没有参数和括号

EXTRACT (datetime)			
		可以从日期参数中提取year，month day等
		![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9kb2NzLm9yYWNsZS5jb20vY2QvRTExODgyXzAxL3NlcnZlci4xMTIvZTQxMDg0L2ltZy9leHRyYWN0X2RhdGV0aW1lLmdpZg)
		

FROM_TZ(timaezone_stamp,timezone_value)		将时区值和TIMESTAMP(时间戳)转换为TIMESTAMP WITH TIME ZONE值。

LAST_DAY(date)				返回指定日期对应月份的最后一天。	

LOCALTIMESTAMP				返回会话中的日期和时间

MONTHS_BETWEEN				返回两个日期之间的月份数

NEW_TIME(date,timezone1,timezone2)		给出时间dt1在c1时区对应c2时区的日期和时间

NEXT_DAY(date,char)			指定日期date之后的下一个周char指定天的日期，char可以表示为星期或者天数，星期从星期日开始算。char写为数字的话表示：从星期日开始算，今天是第二天，所以数字1，2表示的下周的第一二天，3之后的数字就是明天之后的时间。							

NUMTODSINTERVAL(<x>,<c>) 	x是一个数字,c是一个字符串,表明x的单位,这个函数把x转为interval day to second数据类型 常用的单位有 ('day','hour','minute','second')			

NUMTOYMINTERVAL				与numtodsinterval函数类似,将x转为interval year to month数据类型常用的单位有'year','month'

ORA_DST_AFFECTED(datetime_expr)		ORA_DST_AFFECTED is useful when you are changing the time zone data file for your database. 

ORA_DST_CONVERT				ORA_DST_CONVERT is useful when you are changing the time zone data file for your database. 

ORA_DST_ERROR				ORA_DST_ERROR is useful when you are changing the time zone data file for your database.

ROUND (date)

SESSIONTIMEZONE

SYS_EXTRACT_UTC

SYSDATE

SYSTIMESTAMP

TO_CHAR (datetime)

TO_DSINTERVAL

TO_TIMESTAMP

TO_TIMESTAMP_TZ

TO_YMINTERVAL

TRUNC (date)

TZ_OFFSET


#### 通用比较函数
GREATEST

LEAST

#### 转换函数
ASCIISTR

BIN_TO_NUM

CAST

CHARTOROWID

COMPOSE

CONVERT

DECOMPOSE

HEXTORAW

NUMTODSINTERVAL

NUMTOYMINTERVAL

RAWTOHEX

RAWTONHEX

ROWIDTOCHAR

ROWIDTONCHAR

SCN_TO_TIMESTAMP

TIMESTAMP_TO_SCN

TO_BINARY_DOUBLE

TO_BINARY_FLOAT

TO_BLOB

TO_CHAR (character)

TO_CHAR (datetime)

TO_CHAR (number)

TO_CLOB

TO_DATE

TO_DSINTERVAL

TO_LOB

TO_MULTI_BYTE

TO_NCHAR (character)

TO_NCHAR (datetime)

TO_NCHAR (number)

TO_NCLOB

TO_NUMBER

TO_SINGLE_BYTE

TO_TIMESTAMP

TO_TIMESTAMP_TZ

TO_YMINTERVAL

TREAT

UNISTR

#### 大对象函数
BFILENAME

EMPTY_BLOB, EMPTY_CLOB


#### 集合函数
CARDINALITY

COLLECT

POWERMULTISET

POWERMULTISET_BY_CARDINALITY

SET


#### 继承函数
SYS_CONNECT_BY_PATH

#### DataMining 函数
CLUSTER_ID

CLUSTER_PROBABILITY

CLUSTER_SET

FEATURE_ID

FEATURE_SET

FEATURE_VALUE

PREDICTION

PREDICTION_BOUNDS

PREDICTION_COST

PREDICTION_DETAILS

PREDICTION_PROBABILITY

PREDICTION_SET


#### xml函数
APPENDCHILDXML

DELETEXML

DEPTH

EXISTSNODE

EXTRACT (XML)

EXTRACTVALUE

INSERTCHILDXML

INSERTCHILDXMLAFTER

INSERTCHILDXMLBEFORE

INSERTXMLAFTER

INSERTXMLBEFORE

PATH


SYS_DBURIGEN

SYS_XMLAGG

SYS_XMLGEN

UPDATEXML

XMLAGG

XMLCAST

XMLCDATA

XMLCOLATTVAL

XMLCOMMENT

XMLCONCAT

XMLDIFF

XMLELEMENT

XMLEXISTS

XMLFOREST

XMLISVALID

XMLPARSE

XMLPATCH

XMLPI

XMLQUERY

XMLROOT

XMLSEQUENCE

XMLSERIALIZE

XMLTABLE

XMLTRANSFORM


#### 编码解码函数
DECODE

DUMP

ORA_HASH

VSIZE


#### null相关函数
COALESCE

LNNVL

NANVL

NULLIF

NVL

NVL2


#### 环境和标志函数

SYS_CONTEXT

SYS_GUID

SYS_TYPEID

UID

USER

USERENV

### 2. 聚合函数
### 3. 分析函数
### 4. 对象相关函数
### 5. 模型函数
### 6. OLAP 函数
### 7. Data Cartridge 函数