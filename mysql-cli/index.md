# mysql command


<!--more-->

# 连接数据库

```bash
$ mysql -u root -p
```



# 表结构语句

#### 创建

```sql
CREATE TABLE `forbidden_msg_from_aliyun` (
  `id` bigint(11) NOT NULL AUTO_INCREMENT COMMENT '自增id',
  `words` varchar(128) NOT NULL DEFAULT '' COMMENT '脏字',
  `level` varchar(36) NOT NULL DEFAULT '' COMMENT '级别:block,review',
  `label` varchar(36) NOT NULL DEFAULT '' COMMENT '类型',
  `words_state` int(4) NOT NULL DEFAULT 0 COMMENT '1:use,0:del',
  `vtype` int(4) NOT NULL DEFAULT 0 COMMENT 'TypeAliYunRealLexicon:4',
  `create_time` datetime DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `update_time` datetime DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
  PRIMARY KEY (`id`),
  UNIQUE KEY `vtype` (`words`,`vtype`),
  KEY `forbidden_msg_vtype` (`vtype`),
  KEY `level_index` (`level`),
  KEY `label_index` (`label`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8 COMMENT='阿里云词库';
```

#### 已有表结构加字段

- 多个字段

```sql
ALTER TABLE
  app_update
ADD
  COLUMN `app_version` varchar(16) NOT NULL default '' COMMENT 'app 版本号',
ADD
  COLUMN `update_app_type` tinyint(4) NOT NULL default 0 COMMENT '0:不限制咕咚app版本,1:大于等于某个版本,2:小于等于某个版本';
```

- 单个字段

```sql
ALTER TABLE
  ota_conf
ADD
  COLUMN `expr` text COMMENT 'user_id 尾号或者user_id用逗号隔开';
```

#### 已有表结构加(减)索引

- 多个普通索引

```sql
ALTER table
  modified_nick_log
ADD
  INDEX idx_user_id(user_id),
ADD
  INDEX idx_new_nick(new_nick);
```

- 单个唯一联合索引

```sql
ALTER table
  extend_new_user_profile
ADD
  unique unqi_user_id_name_id_card(`user_id`,`encrypt_id_card`,`encrypt_real_name`);
```

- 删除索引

```sql
ALTER table extend_new_user_profile DROP INDEX uniq_index_user_id;
```

- 单个索引

```sql
ALTER table
  modified_nick_log
ADD
  INDEX idx_user_id(user_id);
```

- 单个唯一索引

```sql
ALTER table
  modified_nick_log
ADD
  unique unqi_user_id(user_id);
```

- 普通联合索引

```sql
ALTER table
  extend_new_user_profile
ADD
  INDEX idx_user_id_name_id_card(`user_id`,`encrypt_id_card`,`encrypt_real_name`);
```

#### 展示数据库中的表

```sql
SHOW TABLES LIKE 'user%';
# or 
SHOW TABLES FROM  winner_chicken_dinner LIKE 'user%';
# or
SHOW TABLES FROM  winner_chicken_dinner；
```

#### 展示表中的详细信息

```sql
mysql> show create table user_profile \G

Create Table: CREATE TABLE `user_profile` (
  `id` int unsigned NOT NULL AUTO_INCREMENT,
  `nick_name` varchar(512) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NOT NULL DEFAULT '',
  `pwd` varchar(256) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NOT NULL DEFAULT '',
  `email` varchar(36) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NOT NULL DEFAULT '',
  `create_time` datetime NOT NULL,
  `update_time` datetime NOT NULL,
  `user_id` varchar(36) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NOT NULL,
  `head_icon` int NOT NULL,
  `level` int NOT NULL,
  PRIMARY KEY (`id`),
  UNIQUE KEY `uniq_email` (`email`),
  UNIQUE KEY `uniq_nick_name` (`nick_name`) USING BTREE,
  UNIQUE KEY `uniq_user_id` (`user_id`)
) ENGINE=InnoDB AUTO_INCREMENT=8 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci
```





# 数据库语句

#### 查看当前数据库

```sql
SELECT DATABASE();
```

####  显示当前时间、用户名、数据库版本

```sql
 SELECT now(), user(), version();
```

####  创建数据库

```sql
CREATE DATABASE IF NOT EXISTS RUNOOB DEFAULT CHARSET utf8 COLLATE utf8_general_ci;
```

#### 查看已有的库

```sql
SHOW DATABASES like 'win%';
# or 
SHOW DATABASES;
```

#### 查看数据库信息

```sql
SHOW CREATE DATABASE winner_chicken_dinner
```

#### 删除库

```sql
drop database RUNOOB;
```

#### 选择数据库

```sql
use winner_chicken_dinner;
```






