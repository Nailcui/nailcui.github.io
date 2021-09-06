#### 备份恢复

```
# 备份
mysqldump -hlocalhost -P15382 -uroot -proot --single-transaction -E -R --set-gtid-purged=OFF dbname > dbname.sql

# 恢复
mysql -hlocalhost -P3306 -uroot -proot dbname < dbname.sql
```

