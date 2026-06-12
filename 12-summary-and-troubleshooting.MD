# Roboshop Deployment Summary and Troubleshooting

This file explains what is actually happening in the Roboshop project, the common steps followed for each component, what we are learning, and useful troubleshooting commands.

## What Is Happening

Roboshop is a microservices-based e-commerce application. Instead of running everything as one big application, each feature runs as a separate service.

The main services are:

| Component | Purpose |
| --- | --- |
| Frontend | Serves the website using Nginx |
| Catalogue | Shows product list and product details |
| User | Handles user registration and login |
| Cart | Handles shopping cart operations |
| Shipping | Calculates shipping cost |
| Payment | Handles payment workflow |
| Dispatch | Processes order dispatch messages |
| MongoDB | Stores catalogue and user data |
| Redis | Used as cache/session storage |
| MySQL | Stores shipping-related data |
| RabbitMQ | Message queue used between services |

The frontend receives user requests from the browser. It forwards API requests to backend services using Nginx reverse proxy.

Backend services talk to databases and message queues:

- Catalogue talks to MongoDB.
- User talks to MongoDB and Redis.
- Cart talks to Redis and Catalogue.
- Shipping talks to Cart and MySQL.
- Payment talks to Cart, User, and RabbitMQ.
- Dispatch consumes messages from RabbitMQ.

## Common Steps Followed

Most Roboshop components follow the same deployment pattern.

## 1. Install Required Software

Each component needs a runtime or database software.

Examples:

```shell
dnf install nodejs -y
dnf install maven -y
dnf install python3 gcc python3-devel -y
dnf install golang -y
dnf install mysql-server -y
dnf install redis -y
dnf install rabbitmq-server -y
```

Some software needs module setup before install.

Example for NodeJS:

```shell
dnf module disable nodejs -y
dnf module enable nodejs:20 -y
dnf install nodejs -y
```

Example for Redis:

```shell
dnf module disable redis -y
dnf module enable redis:7 -y
dnf install redis -y
```

## 2. Add Application User

Applications should not run as root unless there is a specific requirement.

```shell
useradd --system --home /app --shell /sbin/nologin --comment "roboshop system user" roboshop
```

The `roboshop` user is a system user used only to run services.

## 3. Create Application Directory

Application code is kept in a standard location.

```shell
mkdir /app
```

## 4. Download and Extract Application Code

Each app artifact is downloaded from S3 and extracted into `/app`.

Example:

```shell
curl -L -o /tmp/cart.zip https://roboshop-artifacts.s3.amazonaws.com/cart-v3.zip
cd /app
unzip /tmp/cart.zip
```

## 5. Install Dependencies or Build Code

Different technologies use different dependency commands.

NodeJS:

```shell
cd /app
npm install
```

Java Maven:

```shell
cd /app
mvn clean package
mv target/shipping-1.0.jar shipping.jar
```

Python:

```shell
cd /app
pip3 install -r requirements.txt
```

GoLang:

```shell
cd /app
go mod init dispatch
go get
go build
```

## 6. Create SystemD Service

Each backend application is configured as a systemd service.

Service files are created inside:

```shell
/etc/systemd/system/
```

Example:

```shell
vim /etc/systemd/system/cart.service
```

Systemd helps us start, stop, restart, enable, and check service status.

## 7. Reload SystemD

Whenever a new service file is created or changed, reload systemd.

```shell
systemctl daemon-reload
```

## 8. Enable and Start Service

Enable means the service starts automatically after reboot.

Start means the service starts now.

```shell
systemctl enable cart
systemctl start cart
```

## 9. Configure Network Access

Some databases listen only on localhost by default.

MongoDB:

- Change bind IP from `127.0.0.1` to `0.0.0.0` in `/etc/mongod.conf`.

Redis:

- Change bind IP from `127.0.0.1` to `0.0.0.0` in `/etc/redis/redis.conf`.
- Change `protected-mode yes` to `protected-mode no`.

After config change, restart the service.

```shell
systemctl restart mongod
systemctl restart redis
```

## 10. Load Schema and Master Data

Some services need database schema or seed data.

MongoDB example:

```shell
mongosh --host MONGODB-SERVER-IPADDRESS < /app/db/master-data.js
```

MySQL examples:

```shell
mysql -h <MYSQL-SERVER-IPADDRESS> -uroot -pRoboShop@1 < /app/db/schema.sql
mysql -h <MYSQL-SERVER-IPADDRESS> -uroot -pRoboShop@1 < /app/db/app-user.sql
mysql -h <MYSQL-SERVER-IPADDRESS> -uroot -pRoboShop@1 < /app/db/master-data.sql
```

## What We Are Learning

This project teaches real DevOps deployment skills.

Important learning points:

- How a microservices application is structured.
- How frontend and backend services communicate.
- How backend services connect to databases.
- How Nginx works as a web server and reverse proxy.
- How MongoDB, Redis, MySQL, and RabbitMQ support application services.
- How to install software using `dnf`.
- How to enable specific software versions using `dnf module`.
- How to create Linux system users for applications.
- How to place application code in a standard directory.
- How to install application dependencies for NodeJS, Python, Java, and Go.
- How to create and manage systemd service files.
- How environment variables are used to connect services.
- How to load database schema and master data.
- How to troubleshoot services using logs, ports, and process checks.

## Common Troubleshooting Commands

## Service Status

Check whether a service is running:

```shell
systemctl status nginx
systemctl status mongod
systemctl status redis
systemctl status catalogue
systemctl status user
systemctl status cart
systemctl status shipping
systemctl status payment
systemctl status dispatch
systemctl status rabbitmq-server
systemctl status mysqld
```

Restart a service:

```shell
systemctl restart catalogue
```

Stop and start a service:

```shell
systemctl stop catalogue
systemctl start catalogue
```

Enable service on boot:

```shell
systemctl enable catalogue
```

Reload systemd after changing service file:

```shell
systemctl daemon-reload
```

## Logs

View logs for a service:

```shell
journalctl -u catalogue
```

Follow live logs:

```shell
journalctl -u catalogue -f
```

View latest logs:

```shell
journalctl -u catalogue -n 100
```

Check Nginx logs:

```shell
tail -f /var/log/nginx/access.log
tail -f /var/log/nginx/error.log
```

## Ports and Network

Check listening ports:

```shell
ss -lntp
```

Check whether a port is open:

```shell
telnet <SERVER-IP> 8080
```

Alternative using netcat:

```shell
nc -vz <SERVER-IP> 8080
```

Check service endpoint using curl:

```shell
curl http://localhost:8080/health
curl http://<SERVER-IP>:8080/health
```

Check Nginx frontend:

```shell
curl http://localhost
curl http://<FRONTEND-SERVER-IP>
```

## Process Checks

Check running processes:

```shell
ps -ef
```

Search for a process:

```shell
ps -ef | grep node
ps -ef | grep java
ps -ef | grep nginx
```

## Package Checks

Check installed packages:

```shell
rpm -qa | grep nodejs
rpm -qa | grep nginx
rpm -qa | grep redis
rpm -qa | grep mysql
rpm -qa | grep rabbitmq
```

Check available modules:

```shell
dnf module list nodejs
dnf module list nginx
dnf module list redis
```

## Configuration Checks

Check service file content:

```shell
cat /etc/systemd/system/catalogue.service
```

Check MongoDB config:

```shell
cat /etc/mongod.conf
```

Check Redis config:

```shell
cat /etc/redis/redis.conf
```

Check Nginx config syntax:

```shell
nginx -t
```

Check Nginx config:

```shell
cat /etc/nginx/nginx.conf
```

## Database Checks

Connect to MongoDB:

```shell
mongosh --host <MONGODB-SERVER-IPADDRESS>
```

MongoDB useful commands:

```shell
show dbs
use catalogue
show collections
db.products.find()
```

Connect to MySQL:

```shell
mysql -h <MYSQL-SERVER-IPADDRESS> -uroot -pRoboShop@1
```

MySQL useful commands:

```sql
show databases;
use cities;
show tables;
select * from cities limit 5;
```

Check Redis connection:

```shell
redis-cli -h <REDIS-SERVER-IP> ping
```

Expected output:

```text
PONG
```

Check RabbitMQ users:

```shell
rabbitmqctl list_users
```

Check RabbitMQ permissions:

```shell
rabbitmqctl list_permissions
```

## Application Dependency Checks

NodeJS:

```shell
node -v
npm -v
npm install
```

Java and Maven:

```shell
java -version
mvn -version
mvn clean package
```

Python:

```shell
python3 --version
pip3 --version
pip3 install -r requirements.txt
```

GoLang:

```shell
go version
go mod tidy
go build
```

## Common Problems and Fixes

## Service Is Not Starting

Check status and logs:

```shell
systemctl status <service-name>
journalctl -u <service-name> -n 100
```

Common reasons:

- Wrong IP address in environment variables.
- Missing dependency installation.
- Wrong service file path.
- Forgot to run `systemctl daemon-reload`.
- Application code is not present in `/app`.
- Port already in use.

## Application Cannot Connect to Database

Check:

- Database service is running.
- Database port is listening.
- Security group or firewall allows traffic.
- Application service file has correct database IP.
- Database is listening on `0.0.0.0`, not only `127.0.0.1`.

Useful commands:

```shell
systemctl status mongod
systemctl status redis
systemctl status mysqld
ss -lntp
nc -vz <DB-SERVER-IP> <PORT>
```

## Nginx Reverse Proxy Not Working

Check Nginx config:

```shell
nginx -t
systemctl restart nginx
```

Check backend service directly:

```shell
curl http://<BACKEND-SERVER-IP>:8080/health
```

Check Nginx logs:

```shell
tail -f /var/log/nginx/error.log
```

## Schema or Master Data Not Loaded

Check database manually.

MongoDB:

```shell
mongosh --host <MONGODB-SERVER-IPADDRESS>
show dbs
use catalogue
show collections
db.products.find()
```

MySQL:

```shell
mysql -h <MYSQL-SERVER-IPADDRESS> -uroot -pRoboShop@1
show databases;
show tables;
```

## Important Ports

| Component | Port |
| --- | --- |
| Nginx | 80 |
| Backend services | 8080 |
| MongoDB | 27017 |
| Redis | 6379 |
| MySQL | 3306 |
| RabbitMQ AMQP | 5672 |
| RabbitMQ Management UI | 15672 |

## Deployment Flow Order

A good order to deploy Roboshop is:

1. MongoDB
2. Redis
3. MySQL
4. RabbitMQ
5. Catalogue
6. User
7. Cart
8. Shipping
9. Payment
10. Dispatch
11. Frontend

Databases and message queues should be ready before starting the application services that depend on them.

## Quick Final Checklist

- All required packages are installed.
- Application code is extracted under `/app`.
- Dependencies are installed.
- Service files are created correctly.
- `systemctl daemon-reload` is completed.
- Services are enabled and started.
- Correct server IP addresses are used in environment variables.
- Database ports are open for dependent services.
- Schema and master data are loaded where required.
- Nginx reverse proxy points to correct backend service IPs.

