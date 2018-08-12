命令
----------
+ 获取 现有连接信息
~~~slq
show processlist;
~~~

+ 修改密码和远程用户创建
~~~sql
select authentication_string from user where User='root';  //update修改需要加多一句flush privileges; 

ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '新密码';

set password for 用户名@localhost = password('新密码'); 

// mysql 远程登陆用户创建
grant all PRIVILEGES on 库名.表或者* to 用户名@'ip地址或者%' identified by '密码';
~~~

+ 临时更换事务级别
```sql
 set session transaction isolation level  事务级别名字; //(临时跟换事务级别)
```

+ 查询 innodb 事务等待信息
```sql
select * from INNODB_LOCK_WAITS \G ;
```

+ 查询 innodb 状态
```sql
SHOW STATUS LIKE 'innodb_row_lock%';
```

配置
--------------

+ 取消授权登陆

> skip-grant-tables 

+ 增加锁等待时间，即增大下面配置项参数值，单位为秒（s）

> innodb_lock_wait_timeout=500
