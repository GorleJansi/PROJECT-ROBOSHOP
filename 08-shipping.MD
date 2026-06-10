# Shipping

Shipping service is responsible for finding the distance of the package to be shipped and calculating the price based on that.

Shipping service is written in Java, hence we need to install Java.

Maven is a Java packaging software, hence we are going to install `maven`. This takes care of Java installation.

Developer has chosen Maven. Check with developer which version of Maven is needed.

For our requirement, Java greater than or equal to 1.8 and Maven greater than or equal to 3.5 should work.

## Install Maven

```shell
dnf install maven -y
```

## Configure Application

Our application is developed by the developer of our organization and it is not having any RPM software like other softwares. So we need to configure every step manually.

We already discussed in Linux basics section that applications should run as a non-root user.

Add application user:

```shell
useradd --system --home /app --shell /sbin/nologin --comment "roboshop system user" roboshop
```

User `roboshop` is a function or daemon user to run the application. Apart from that, we do not use this user to login to server.

Also, username `roboshop` has been picked because it suits our project name.

We keep application in one standard location. This is a usual practice that runs in the organization.

Setup an app directory:

```shell
mkdir /app
```

Download the application code to the created app directory:

```shell
curl -L -o /tmp/shipping.zip https://roboshop-artifacts.s3.amazonaws.com/shipping-v3.zip
cd /app
unzip /tmp/shipping.zip
```

Every application developed by development team will have some common softwares that they use as libraries. This application also has dependencies defined in the application configuration.

Download the dependencies and build the application:

```shell
cd /app
mvn clean package
mv target/shipping-1.0.jar shipping.jar
```

## Setup SystemD Shipping Service

We need to setup a new service in systemd so `systemctl` can manage this service.

We already discussed in Linux basics that advantages of `systemctl` managing services. Hence, we are taking that approach, which is also a standard way in the OS.

Create `/etc/systemd/system/shipping.service`.

```shell
vim /etc/systemd/system/shipping.service
```

Add the following content:

```ini
[Unit]
Description=Shipping Service

[Service]
User=roboshop
Environment=CART_ENDPOINT=<CART-SERVER-IPADDRESS>:8080
Environment=DB_HOST=<MYSQL-SERVER-IPADDRESS>
ExecStart=/bin/java -jar /app/shipping.jar
SyslogIdentifier=shipping

[Install]
WantedBy=multi-user.target
```

Ensure you replace `<CART-SERVER-IPADDRESS>` and `<MYSQL-SERVER-IPADDRESS>` with the actual server IP addresses.

Load the service:

```shell
systemctl daemon-reload
```

The above command is because we added a new service. We are telling systemd to reload so it will detect the new service.

Start the service:

```shell
systemctl enable shipping
systemctl start shipping
```

## Load Schema and Master Data

For this application to work fully functional, we need to load schema to the database.

Schemas are usually part of application code and developer will provide them as part of development.

We need to load the schema. To load schema we need to install MySQL client.

Install MySQL client:

```shell
dnf install mysql -y
```

Load schema. Schema in database is the structure like what tables to be created and their necessary application layouts.

```shell
mysql -h <MYSQL-SERVER-IPADDRESS> -uroot -pRoboShop@1 < /app/db/schema.sql
```

Create app user. MySQL expects password authentication, hence we need to create the user in MySQL database for shipping app to connect.

```shell
mysql -h <MYSQL-SERVER-IPADDRESS> -uroot -pRoboShop@1 < /app/db/app-user.sql
```

Load master data. This includes the data of all the countries and their cities with distance to those cities.

```shell
mysql -h <MYSQL-SERVER-IPADDRESS> -uroot -pRoboShop@1 < /app/db/master-data.sql
```

This service needs a restart because it is dependent on schema. After loading schema only it will work as expected.

```shell
systemctl restart shipping
```
