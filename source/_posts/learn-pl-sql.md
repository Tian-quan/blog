---
title: PL/SQL基础
date: 2017-03-07 10:30:16
tags: [SQL,ORACLE]
categories: SQL
---

# 数据准备
```SQL
DROP TABLE students;
create table students
(
  ID int,
  userName varchar(100),
  userPass varchar(100),
  userAge  int
);
-- 插入10条
DECLARE
	i number(2);
BEGIN
	for i in 1..10 loop
		INSERT INTO students VALUES(i,'jack'||i,'jj',20+i);
	end loop;
END;

SELECT * from students;
```

# 语法基础

## if
```SQL
DECLARE
	i number(1) := 5;
	j number(2) := 10;
BEGIN
	if i < 5 then
		DBMS_OUTPUT.PUT_LINE('i<5');
	elsif i=5 then
		DBMS_OUTPUT.PUT_LINE('i=5');
	else
		DBMS_OUTPUT.PUT_LINE('i=5');
	end if;
END;
```
## case when
```SQL
DECLARE
	i number(1) := 5;
	j number(2);
BEGIN
	j:=
	case i
		when 0 then 0
		when 5 then 5
		else 10
	end;
	DBMS_OUTPUT.PUT_LINE(j);

	j :=
	case
		when i < 5 then 0
		when i=5 then 6
		else 10
	end;
	DBMS_OUTPUT.PUT_LINE(j);
END;
```

## loop
```SQL
DECLARE
	i number(1) := 0;
BEGIN
	loop
		i := i + 1;
		DBMS_OUTPUT.PUT_LINE(i);
		exit when i = 5;
	end loop;
END;
```

## while
```SQL
DECLARE
	i number(2) := 0;
BEGIN
	while i < 10 loop
		DBMS_OUTPUT.PUT_LINE(i);
		i := i + 1;
	end loop;
END;
```

## for..in
```SQL
BEGIN
	for i in 0..9 loop
		DBMS_OUTPUT.PUT_LINE(i);
	end loop;
END;
```

## goto
```SQL
DECLARE
	i number(2) := 0;
	j number(2) := 0;
BEGIN
	for j in 0..9 loop
		i := i + 1;
		if i=5 then
			goto EXIT_LOOP;
		end if;
	end loop;
	<<EXIT_LOOP>>
	DBMS_OUTPUT.PUT_LINE(i);
END;
```

# 获取插入语句的返回值. 可以获取增删改返回的数据
```SQL
DECLARE
	row_id ROWID;
	info VARCHAR2(200);
BEGIN
	INSERT INTO students VALUES(1,'jack','jj',23)
	returning rowid,to_char(ID)||':'||userName
	INTO row_id,info;
	DBMS_OUTPUT.PUT_LINE('rowid:'||row_id);
	DBMS_OUTPUT.PUT_LINE('name:'||info);
END;
```

# 自定数据类型
```SQL
DECLARE
	TYPE RECORD_TYPE_PERSON_AGE IS RECORD(
		uName varchar(100),
		age students.userAge%TYPE	--用%TYPE 类型定义与表相配的字段
	);
	userRec RECORD_TYPE_PERSON_AGE;
BEGIN
	SELECT userName,userAge INTO userRec from STUDENTS where ID=1;
	userRec.uName := 'changed';
	DBMS_OUTPUT.PUT_LINE(userRec.uName||' '||userRec.age);
END;
```

# 数组类型
```SQL
DECLARE
	 --定义一个最多保存5个VARCHAR(25)数据类型成员的VARRAY数据类型
   TYPE reg_varray_type IS VARRAY(5) OF VARCHAR(25);
   v_reg_varray REG_VARRAY_TYPE;
BEGIN
	 --用构造函数语法赋予初值
   v_reg_varray := reg_varray_type('1', '2', '3', '4', '5');

   DBMS_OUTPUT.PUT_LINE(v_reg_varray(1)||','
                    ||v_reg_varray(2)||','
                    ||v_reg_varray(3)||','
                    ||v_reg_varray(4));
   DBMS_OUTPUT.PUT_LINE(v_reg_varray(5));
	--用构造函数语法赋予初值后就可以这样对成员赋值
   v_reg_varray(5) := 'xx';
   DBMS_OUTPUT.PUT_LINE('5th is '||v_reg_varray(5));
END;
```

# 行类型, 使用%ROWTYPE
```SQL
DECLARE
    vId students.ID%TYPE := 1; -- 属性类型引用
    rec students%ROWTYPE;	--行类型
BEGIN
    SELECT * INTO rec FROM students where ID=vId;
    DBMS_OUTPUT.PUT_LINE('姓名:'||rec.userName||' 年龄:'||rec.userAge);
END;
```

# 定义表类型, 使用 TABLE
```SQL
DECLARE
	TYPE TABLE_STUDENTS IS TABLE OF STUDENTS%ROWTYPE INDEX BY BINARY_INTEGER;
	recs_students TABLE_STUDENTS;
	loop_count number(1) := 2;
BEGIN
	FOR i IN 1..loop_count LOOP
		SELECT * into recs_students(i) from STUDENTS where id=i;
	END LOOP;
	FOR j IN recs_students.FIRST..recs_students.LAST LOOP
		DBMS_OUTPUT.PUT_LINE(recs_students(j).userName||' '||recs_students(j).ID);
	END LOOP;
END;
```

# 游标

## 游标基础
```SQL
/*
 Cursor_name%FOUND     布尔型属性，当最近一次提取游标操作FETCH成功则为 TRUE,否则为FALSE；
 Cursor_name%NOTFOUND   布尔型属性，与%FOUND相反；
 Cursor_name%ISOPEN     布尔型属性，当游标已打开时返回 TRUE；
 Cursor_name%ROWCOUNT   数字型属性，返回已从游标中读取的记录数。
*/
DECLARE
	cursor some_stu_cursor is SELECT ID,userName from students;
	userId students.ID%TYPE;
	userName students.userName%TYPE;
BEGIN
	open some_stu_cursor;
		loop
			fetch some_stu_cursor into userId,userName;
			exit when some_stu_cursor%notfound; --取不到数据,则退出循环.
			DBMS_OUTPUT.PUT_LINE('id:'||userId||' name:'||userName);
		end loop;
	close some_stu_cursor;
END;
```

## 游标传递参数
```SQL
DECLARE
	cursor some_stu_cursor(id_no number DEFAULT 5) is SELECT ID,userName from students where id <= id_no;
	userId students.ID%TYPE;
	userName students.userName%TYPE;
	rec students%rowtype;
BEGIN
	--隐含打开游标
	for rec in some_stu_cursor(2) LOOP
		--隐含执行一个FETCH语句
		DBMS_OUTPUT.PUT_LINE('id:'||rec.id ||' name:'||rec.userName );
		--隐含监测c_sal%NOTFOUND
	end loop;
	DBMS_OUTPUT.PUT_LINE('======================');
	open some_stu_cursor(id_no => 6);
		loop
			fetch some_stu_cursor into userId,userName;
			exit when some_stu_cursor%notfound; --取不到数据,则退出循环.
			DBMS_OUTPUT.PUT_LINE('id:'||userId||' name:'||userName);
		end loop;
	close some_stu_cursor;
END;
```

##　游标指定当前行
```SQL
DECLARE
	userId STUDENTS.ID%type;
	i number(2) := 1;
	cursor all_stu is SELECT id,userName from students FOR UPDATE;
BEGIN
	for userId in all_stu loop
		update students set id=i where CURRENT of all_stu;--当前行
		i := i+1;
	end loop;
END;
```

## 游标变量
```SQL
DECLARE
	type ID_NAME_REC is RECORD(id STUDENTS.id%type,name students.USERNAME%type);
	type ID_NAME_CURSOR is REF CURSOR RETURN ID_NAME_REC;
	rec ID_NAME_REC;
	cur ID_NAME_CURSOR;
BEGIN
	OPEN cur for SELECT id,username from students;
	loop	--用for .. loop 不行.
		FETCH cur into rec;
		EXIT when cur%notfound;
		DBMS_OUTPUT.PUT_LINE('id:'||rec.id||' name:'||rec.name);
	end loop;
END;
```

## 游标变量 - 没有return
```SQL
DECLARE
	userId students.ID%TYPE;
	type REF_CURSOR is ref cursor;
	var_cur REF_CURSOR;
BEGIN
	open var_cur for SELECT id from students where id > 5;
	loop	--用for .. loop 不行.
		FETCH var_cur into userid;
		EXIT when var_cur%notfound;
		DBMS_OUTPUT.PUT_LINE('id:'||userid);
	end loop;
END;
```