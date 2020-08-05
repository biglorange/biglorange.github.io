---
title: PSQL语法
date: 2019-10-11 22:16:07
tags:
categories:
description:
---

内置函数：

CAST

​	CAST("XXXXXX",VARCHAR(8));

转换类型，将指定字符串转换为指定类型的数据

TO_HEX()



LPAD

LPAD("XXXXXXXX", 8, "0");

将字符串从左边开始以指定内容填充到指定宽度

eg:LPAD("25", 8, "0");	->00000025

TO_BIGINT();

存储过程



创建存储过程

CREATE OR REPLACE PROCEDURE PROCNAME

()

AS

--声明变量

BEGIN

--存储过程

END;

/

CALL PROCNAME()	--调用存储过程

/