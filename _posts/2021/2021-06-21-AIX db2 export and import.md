---
layout:     post
title:      AIX上DB2 数据的导出和导入
no-post-nav: true
category: other
tags: [other]
excerpt: AIX上DB2 数据的导出和导入
---

## AIX上DB2 数据的导出和导入

```shell
-- 数据库导出
db2 export to /tmp/rfc/ALPHA.DLQ.QUEUE.del of del "select * from wledge.user where mandt='330'"
--打包
tar -cvf ALPHA.DLQ.QUEUE.txt.tar ALPHA.DLQ.QUEUE.del
--压缩
gzip ALPHA.DLQ.QUEUE.del.tar
-- 复制tar 文件到另外一个db server 上
--解压
gzip -dc ALPHA.DLQ.QUEUE.del.tar.gz
--拆包
tar -xvf ALPHA.DLQ.QUEUE.del.tar
--数据库导入：
db2 import from /tmp.rfc/ALPHA.DLQ.QUEUE.del of del "insert into wledge.user"
```

## 参考资料：

db2备份和导入单个表操作 
db2 connect to 数据库名 user 登陆名 using 登陆密码 

>db2 export to t1.ixf of ixf select * from 表名 
>db2 import from t1.ixf of ixf insert into 目标表名或者新表名


导入表的数据

```
import from [path(例:D:"TABLE1.ixf)] of ixf insert into TABLE1;

load from [path(例:D:"TABLE1.ixf)] of ixf insert into TABLE1;

load from [path(例:D:"TABLE1.ixf)] of ixf replace into TABLE1; // 装入数据前，先删除已存在记录

load from [path(例:D:"TABLE1.ixf)] of ixf restart into TABLE1; // 当装入失败时，重新执行，并记录导出结果和
```

错误信息

```
import from [path(例:D:"TABLE1.ixf)] of ixf savecount 1000 messages [path(例:D:"msg.txt)] insert into TABLE1;// 其中，savecount表示完成每1000条操作，记录一次.
```

del与ixf区别：

del格式是一个文本文件，文件按行来存储，含有回车的文本内容在del文件中会另起一行，del文件可视。

ixf格式保存的是结构和数据，是一个二进制文件，ixf文件不可视。



