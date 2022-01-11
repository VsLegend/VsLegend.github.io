# Mysql常用语句

### MySQL远程连接
```
# 远程登陆
use mysql;
select host, user, authentication_string, plugin from user; #查看用户
GRANT ALL ON *.* TO 'root'@'%';
FLUSH PRIVILEGES; #载入
```


### MySQL修改密码
```
设置密码，必须包含大小写、数字和字符
ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY 'password';
FLUSH PRIVILEGES;
```

### MySQL启动相关命令
```
systemctl start mysqld
systemctl stop mysqld 
systemctl restart mysqld 
systemctl status mysqld 
```
### MySQL自启动
```
systemctl enable mysqld
systemctl daemon-reload
```

#### mysql 配置文件修改
/etc/my.cnf

