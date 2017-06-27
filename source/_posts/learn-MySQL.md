---
title: SQL基础-MySQL
date: 2017-05-16 14:04:54
tags: [SQL,MySQL]
categories: SQL
---

# 时间,日期操作
```SQL
-- 1.当前时间戳, 日期, 时间
SELECT
NOW(),				-- 2017-05-15 18:03:24
CURDATE(),			-- 2017-05-15
CURRENT_TIMESTAMP,  -- 2017-05-15 18:03:24
CURRENT_DATE, 		-- 2017-05-15
CURRENT_TIME		-- 18:03:24
;

-- 2.时间加减 date_add, date_sub
SELECT NOW(),
date_add(NOW(), INTERVAL 1 YEAR),	-- 年
date_add(NOW(), INTERVAL 1 MONTH),	-- 月
date_add(NOW(), INTERVAL 1 DAY),	-- 天
date_add(NOW(), INTERVAL 1 HOUR),	-- 小时
date_add(NOW(), INTERVAL 1 MINUTE),	-- 分钟
date_add(NOW(), INTERVAL 1 SECOND),	-- 秒
date_add(NOW(), interval '01:30' HOUR_MINUTE),
date_add(NOW(), interval '01:30:30' HOUR_SECOND)
;
select date_add(curdate(),interval -day(curdate())+1 day)   -- 获取本月第一天
select last_day(curdate());   -- 获取当月最后一天
select date_add(last_day(curdate()), interval 1 day);   -- 获取下月第一天


-- 3.日期处于年,月,星期的第几天
SELECT
CURRENT_TIMESTAMP,
DAYOFYEAR(NOW()),
DAYOFMONTH(NOW()),
DAYOFWEEK(NOW()),	-- WEEK从周日开始, 周一返回的值是2
DAYNAME(NOW()),		-- 周一返回, Monday.
MONTHNAME(NOW())	-- 五月返回, May
;

-- 4. 截取时间的某个字段
SELECT
NOW(),
YEAR(NOW()),
QUARTER(NOW()),
WEEK(NOW()),
MONTH(NOW()),
DAY(NOW()),
HOUR(NOW()),
MINUTE(NOW()),
SECOND(NOW())
;

-- 5.时间/日期 格式化
SELECT NOW(), DATE_FORMAT(NOW(),'%Y-%m-%d');	-- 日期格式化 2017-05-16 14:01:49	 2017-05-16
SELECT NOW(), TIME_FORMAT(NOW(),'%H:%i:%s');	-- 时间格式化 2017-05-16 14:01:38  14:01:38

-- 6. 时间转换
-- 6.1 日期转字符串
SELECT NOW(), DATE_FORMAT(NOW(),'%Y-%m-%d %H:%i:%s');	-- %H 24小时制, %p pm,am
-- 6.2 字符串转日期
SELECT timestamp('20160101');						    -- 2016-01-01 00:00:00
SELECT timestamp('2016-01-01 10:30');				    -- 2016-01-01 10:30:00
select str_to_date('2016-01-02', '%Y-%m-%d'); 	 		-- 2016-01-02
select str_to_date('2016-01-02 12:11', '%Y-%m-%d %H');  -- 2016-01-02 00:00:00
select str_to_date('2016-01-02 12:11', '%Y-%m-%d %T');  -- 2016-01-02 12:11:00
-- 6.3 unix 时间戳 互转
SELECT FROM_UNIXTIME( 1483200000, '%Y%m%d' );
SELECT UNIX_TIMESTAMP('2017-01-01');
-- 6.4 maketime
select makedate(2001,31); -- '2001-01-31'
select makedate(2001,32); -- '2001-02-01'
select maketime(12,15,30); -- '12:15:30'

-- 7.时间差计算
SELECT DATEDIFF('2017-05-16','2017-05-10'); -- 6
SELECT TIMEDIFF('2017-05-16 12:00:00','2017-05-16 11:00:00'); -- 01:00:00
SELECT HOUR(TIMEDIFF('2017-05-16 12:00:00','2017-05-16 11:00:00')); -- 1
select timestampdiff(year,'2002-05-01','2001-01-01'); -- -1
select timestampdiff(day ,'2002-05-01','2001-01-01'); -- -485
select timestampdiff(hour,'2008-08-08 12:00:00','2008-08-08 00:00:00'); -- -12

```

# 字符串操作
```SQL
SELECT concat('a','b','c') ;	-- 拼接 abc
SELECT concat_ws(',','a','b','c') ;	-- 拼接加间隔符 a,b,c

SELECT STRCMP('ab','aa'), 'ab' > 'aa'; -- 比较 1 1

SELECT LOWER('ABc'), UPPER('abC'); -- 大小写转换, abc  ABC
SELECT replace('hello world hello', 'hello' ,'hi'); -- 替换 hi world hi

SELECT LTRIM('   abc  '), RTRIM('   abc  '), trim('   abc  ');	-- 去除空格

SELECT LEFT('abc',2), right('abc',2);	-- 从左/右截取
SELECT MID('abcd',2,2);
SELECT SUBSTRING('abcd',2,2);

SELECT LENGTH('abc');	-- 长度, 3

SELECT POSITION('b' IN 'abc'), INSTR('abc','b'); -- 查找第一次出现的位置, 2 2

SELECT FIND_IN_SET('b','a,b,c,d'); -- set以逗号为分隔符

SELECT repeat('ab', 3); -- ababab

SELECT REVERSE('abc'); -- 反转 cba
```

# 数字操作
```SQL
SELECT RAND(); -- 随机数

SELECT round(3.14), round(3.56, 1); -- 四舍五入 3 , 3.6

SELECT floor(3.56);	-- 向下取整 3

SELECT CEIL(3.56);	-- 向上取整 4

SELECT truncate(3.146926, 2); -- 截取 3.14

SELECT greatest(1,2,3,4), least(1,2,3,4); -- 集合最大/小值

SELECT SIGN(-10), SIGN(10), SIGN(0); -- 返回代表数字符号 -1 1 0
```

# 正则表达式
```SQL
SELECT * FROM
(
SELECT 'aa' as col UNION ALL
SELECT 'bb' as col
) as t
where t.col REGEXP '^a'
;
```
# 元数据访问
```SQL
-- 当前库
-- 1.TABLE
SHOW TABLES;
SHOW TABLES FROM $库名;
show create table $表名;	-- 创建表的语句


-- 2.COLUMNS
SHOW COLUMNS FROM $表名;
DESCRIBE $表名;
DESC $表名;

-- 3.索引
SHOW KEYS FROM $表名;
SHOW INDEX FROM $表名;

-- information_schema 库
SELECT * FROM COLUMNS;
```

# 常用函数
```SQL
-- 1.IF
SELECT IF( 1 < 0,'T','F'); -- 如果test是真，返回T；否则返回F

-- 2.IFNULL
SELECT IFNULL(NULL,'str'); -- 如果arg1不是空，返回arg1，否则返回arg2

-- 3.NULLIF
SELECT NULLIF(1	,1) ; -- arg1 == arg2 ? NULL : arg1     NULL
SELECT NULLIF(1	,2) ; -- arg1 == arg2 ? NULL : arg1			1
```
