# Seata

![image-20220919161437513](image/image-20220919161437513.png)

```sql
回滚日志表：
 
Field         Type
 
branch_id     bigint       PK
 
xid             varchar(100)
 
context         varchar(128)
 
rollback_info longblob
 
log_status     tinyint
 
log_created     datetime
 
log_modified datetime
 
SQL建表语句：
 
CREATE TABLE `undo_log` (
 
  `id` bigint NOT NULL AUTO_INCREMENT,
 
  `branch_id` bigint NOT NULL,
 
  `xid` varchar(100) NOT NULL,
 
  `context` varchar(128) NOT NULL,
 
  `rollback_info` longblob NOT NULL,
 
  `log_status` int NOT NULL,
 
  `log_created` datetime NOT NULL,
 
  `log_modified` datetime NOT NULL,
 
  PRIMARY KEY (`id`),
 
  UNIQUE KEY `ux_undo_log` (`xid`,`branch_id`)
 
) ENGINE=InnoDB AUTO_INCREMENT=4 DEFAULT CHARSET=utf8;
```