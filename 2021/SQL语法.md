删除数据库

```SQL
DROP DATABASE [DBName];
```

创建数据库、用户并授予权限；

```SQL
CREATE DATABASE [DBNAME];
CREATE USER [USERNAME] IDENTIFIED BY "PASSWORD";
GRANT ALL ON [DBNAME].* TO [USER];
```



查看用户

```SQL
use mysql;
SELECT * from USER;
```

