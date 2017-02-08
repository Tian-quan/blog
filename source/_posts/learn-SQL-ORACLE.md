---
title: SQL基础-ORACLE
date: 2017-02-08 16:50:35
tags: [SQL,ORACLE]
categories: SQL
---

### 正则表达式
```SQL
--1,REGEXP_LIKE
--默认是大小写敏感的
--REGEXP_LIKE(ACCT_TYPE,'E','i')是大小写不敏感
SELECT ACCT_NBR from CCS_ACCT
where REGEXP_LIKE(ACCT_NBR,'1$');

--2,REGEXP_INSTR
--查找第一次出现匹配模式是index
SELECT REGEXP_INSTR(112233,'233')  from dual;
--3,REGEXP_SUBSTR
--查找匹配的字串
SELECT REGEXP_SUBSTR(112233,'[1-2]*') from dual;
--4,REGEXP_REPLACE
--替换匹配字串的值
SELECT REGEXP_REPLACE(112233,'1{2}','aa') from dual;
```

### 集合操作
```SQL
--1,合集;UNION (去重),UNION ALL(不去重)
--UNION
(SELECT ACCT_NBR from CCS_ACCT c WHERE REGEXP_LIKE(ACCT_NBR,'0$'))
UNION
(SELECT ACCT_NBR from CCS_ACCT c WHERE REGEXP_LIKE(ACCT_NBR,'10$'))
;
--UNION ALL
(SELECT ACCT_NBR from CCS_ACCT c WHERE REGEXP_LIKE(ACCT_NBR,'0$'))
UNION ALL
(SELECT ACCT_NBR from CCS_ACCT c WHERE REGEXP_LIKE(ACCT_NBR,'10$'))
;

--2,交集; INTERSECT
(SELECT ACCT_NBR from CCS_ACCT c WHERE REGEXP_LIKE(ACCT_NBR,'0$'))
INTERSECT
(SELECT ACCT_NBR from CCS_ACCT c WHERE REGEXP_LIKE(ACCT_NBR,'10$'))
;

--3.差集. MINUS
(SELECT ACCT_NBR from CCS_ACCT c WHERE REGEXP_LIKE(ACCT_NBR,'0$'))
MINUS
(SELECT ACCT_NBR from CCS_ACCT c WHERE REGEXP_LIKE(ACCT_NBR,'10$'))
;
```

### 字符串操作
```SQL
--1.字符串拼接;concat,ORACLE只能拼接两个field
SELECT concat('a','b') from dual;

--2.字符串长度,length
SELECT length('aaa') from dual;

--3.字符串比较
select case when 'aa'<'ab' then 'true' else 'false' end as result from dual;

--4.字符串裁剪
SELECT SUBSTR('123456789',1,4) from dual;

--5.长度不足左右填充; lpad,rpad ,长度不足则填充,长度够则截取指定长度.
SELECT lpad('1234',10,'*'),lpad('1234',2,'*') from dual;

--6.左右删除;LTRIM,RTRIM
select RTRIM('00111333111','(0|1)'),LTRIM('00111333111','(0|1)') from dual;

--7.字符串替换;REPLACE
SELECT REPLACE('123','2','a') from dual;

--8.查找第一次出现的位置
select INSTR('abcdc', 'c')from dual;
```

### 时间日期操作
```SQL
--1.系统时间,时间戳
SELECT sysdate,SYSTIMESTAMP,CURRENT_DATE
from dual;
--2.字符串转日期
SELECT TO_DATE('2016-12-06 20:15:30','yyyy-mm-dd hh24:mi:ss'),TO_DATE('20161206','yyyy-mm-dd'),cast('11:15:30'as time)
from dual
;
select sysdate,add_months(sysdate,12) from dual;        --加1年
select sysdate,add_months(sysdate,1) from dual;        --加1月
select sysdate,to_char(sysdate+7,'yyyy-mm-dd HH24:MI:SS') from dual;   --加1星期
select sysdate,to_char(sysdate+1,'yyyy-mm-dd HH24:MI:SS') from dual;   --加1天
select sysdate,to_char(sysdate+1/24,'yyyy-mm-dd HH24:MI:SS') from dual;  --加1小时
select sysdate,to_char(sysdate+1/24/60,'yyyy-mm-dd HH24:MI:SS') from dual;  --加1分钟
select sysdate,to_char(sysdate+1/24/60/60,'yyyy-mm-dd HH24:MI:SS') from dual;  --加1秒
--
select months_between(to_date('2005.04.20','yyyy.mm.dd'),to_date('2005.05.20','yyyy.mm.dd')) mon_betw from dual; --时间间隔(月)
select ceil(((To_date('2008-05-02 00:00' , 'yyyy-mm-dd hh24:mi') - To_date('2008-05-01 23:59' , 'yyyy-mm-dd hh24:mi'))) * 24 * 60) FROM DUAL;

SELECT extract(MONTH from sysdate),sysdate from dual;--取出时间中的Year
```

### CAST转换操作
```SQL
--1.转换为Integer
select cast('111' as int) from dual;
--2.转换为NUMBER
select cast('111.333' as  NUMBER(5,2)) from dual;
SELECT to_number('123') from dual;
```

### 分组操作
```SQL
--按季度分组统计
select to_CHAR(a.SETUP_DATE,'yyyy-q') m,count(*)
from CCS_ACCT a
where 1=1
group by to_CHAR(a.SETUP_DATE,'yyyy-q');
```

### ALL ANY EXISTS 操作符
```SQL
--1.ALL.字段值与ALL结果集中进行比较,与每一个比较都成立,则结果为真.
--与not in 差别. 下面的例子,ALL将返回结果集,not in返回空集.
SELECT * from CCS_ACCT
WHERE ACCT_NBR <> ALL(SELECT ACCT_NBR from CCS_ACCT where ACCT_NBR in (1123000,1123001,NULL))
;

SELECT * from CCS_ACCT
WHERE ACCT_NBR not in (1123000,1123001,NULL);
--2.Any,字段值与any结果集中进行比较,只要有一个成立,则结果为真.
SELECT * from CCS_ACCT
WHERE CURR_BAL > any(SELECT CURR_BAL from CCS_ACCT where ROWNUM<5)

--3.EXISTS
SELECT sysdate from dual where EXISTS(select 1 from dual where 1=1);
SELECT sysdate from dual where EXISTS(select 1 from dual where 1<>1);
```

### 常用函数
```SQL
--1.NVL NVL2 NULLIF
--NVL(e1,e2), 相当于 e1 != NULL ? e1 : e2
SELECT NVL(NULL,'0'), NVL('123','1') from dual;
--NVL(field,e1,e2) 相当于 field != NULL ? e1:e2
SELECT NVL2(NULL,'not-null','null'),NVL2('a','not-null','null') from dual;
--NULLIF(e1,e2), 相当于 e1==e2 ? NULL : e1
SELECT NULLIF('a','a'),NULLIF('a','b'),NULLIF('b','a') from dual;

--2. (+)的使用,哪边有（+）哪边就允许为空
SELECT a.*, b.* from a,b where a.ID(+) = b.ID ; --就是一个右连接，等同于select a.*, b.* from a right join b on a.ID=b.ID
SELECT a.*, b.* from a,b where a.ID = b.ID (+);  --就是一个左连接，等同于select a.*, b.* from a left join b on a.ID=b.ID
```
