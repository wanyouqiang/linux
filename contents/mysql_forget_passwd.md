# MariaDB忘记密码解决方法

### 1.在/etc/my.cnf文件的[mysqld]节点中增加skip-grant-tables,然后重启mariadb服务

### 2.使用mysql -uroot -p登录mariadb遇到输入密码敲回车即可

### 3.use mysql;update mysql.user set password=PASSWORD('root') where user='root';

### 4.注释my.cnf文件中的skip-grant-tables

### 5.重启mariadb
