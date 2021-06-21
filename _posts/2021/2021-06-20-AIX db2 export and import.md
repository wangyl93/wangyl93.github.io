---
layout:     post
title:      AIX上DB2 数据的导出和导入
no-post-nav: true
category: other
tags: [other]
excerpt: AIX上DB2 数据的导出和导入
---

## AIX上DB2 数据的导出和导入

数据库导出：

```shell
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

