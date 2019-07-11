## mysql快速开始

### 安装与启动

```
docker pull mysql

docker run --name mysql -d -e MYSQL_ROOT_PASSWORD=xxx -p 3306:3306 mysql --default-authentication-plugin=mysql_native_password

```

### 常用命令

* 查看基本信息

```
show databases;
use information_schema;
show tables;

```


