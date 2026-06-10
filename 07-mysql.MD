# MySQL

Developer has chosen the database MySQL. Hence, we are trying to install it and configure it.

Versions of the DB software will come from the developer, meaning we need to check with developer.

Developer has shared the version information as MySQL 8.x.

## Install MySQL Server

```shell
dnf install mysql-server -y
```

## Start MySQL Service

```shell
systemctl enable mysqld
systemctl start mysqld
```

Next, we need to change the default root password in order to start using the database service. Use password `RoboShop@1` or any other as per your choice.

```shell
mysql_secure_installation --set-root-pass RoboShop@1
```
