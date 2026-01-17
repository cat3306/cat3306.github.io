# mysql FULLTEXT


<!--more-->


MySQL 使用 FULLTEXT 全文索引
FULLTEXT 索引是 MySQL 提供的一种特殊索引类型，专门用于全文搜索。它比 LIKE 模糊查询更高效，特别是在大文本字段上搜索时。

一、创建 FULLTEXT 索引
1. 创建表时定义 FULLTEXT 索引
``` sql
CREATE TABLE articles (
    id INT UNSIGNED AUTO_INCREMENT NOT NULL PRIMARY KEY,
    title VARCHAR(200),
    body TEXT,
    FULLTEXT (title, body)
ENGINE=InnoDB;
```
2. 为已有表添加 FULLTEXT 索引
```sql
ALTER TABLE articles ADD FULLTEXT(title, body);
```
或者

``` sql
CREATE FULLTEXT INDEX ft_index ON articles(title, body);
```
二、使用 FULLTEXT 搜索
1. 基本全文搜索
``` sql
SELECT * FROM articles 
WHERE MATCH(title, body) AGAINST('database');
```
2. 布尔模式搜索（更灵活）
``` sql
SELECT * FROM articles 
WHERE MATCH(title, body) AGAINST('+MySQL -YourSQL' IN BOOLEAN MODE);
```
布尔模式操作符：

\+ 必须包含

\- 必须不包含

\> 提高相关性

< 降低相关性

~ 相关性负值

\* 通配符

"" 短语搜索

3. 相关性排序
``` sql
SELECT id, MATCH(title, body) AGAINST('database') AS score
FROM articles
ORDER BY score DESC;
```

##### 支持中文 需要带有`WITH PARSER ngram`
``` sql 
CREATE TABLE articles (
    id INT UNSIGNED AUTO_INCREMENT NOT NULL PRIMARY KEY,
    title VARCHAR(200),
    body TEXT,
    FULLTEXT (title, body) WITH PARSER ngram
) ENGINE=InnoDB;
```

