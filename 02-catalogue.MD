# Catalogue

Catalogue is a microservice that is responsible for serving the list of items that displays in Roboshop application.

Developer has chosen NodeJS. Check with developer which version of NodeJS is needed.

Developer has set a context that it can work with NodeJS greater than 20.

## Install NodeJS

By default NodeJS 16 is available. We would like to enable NodeJS 20 and install it.

You can list modules by using:

```shell
dnf module list nodejs
```

Disable current module:

```shell
dnf module disable nodejs -y
```

Enable required module:

```shell
dnf module enable nodejs:20 -y
```

Install NodeJS:

```shell
dnf install nodejs -y
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
curl -o /tmp/catalogue.zip https://roboshop-artifacts.s3.amazonaws.com/catalogue-v3.zip
cd /app
unzip /tmp/catalogue.zip
```

Every application developed by development team will have some common softwares that they use as libraries. This application also has dependencies defined in the application configuration.

Download the dependencies:

```shell
cd /app
npm install
```

## Setup SystemD Catalogue Service

We need to setup a new service in systemd so `systemctl` can manage this service.

We already discussed in Linux basics that advantages of `systemctl` managing services. Hence, we are taking that approach, which is also a standard way in the OS.

Create `/etc/systemd/system/catalogue.service`.

```shell
vim /etc/systemd/system/catalogue.service
```

Add the following content:

```ini
[Unit]
Description = Catalogue Service

[Service]
User=roboshop
Environment=MONGO=true
Environment=MONGO_URL="mongodb://<MONGODB-SERVER-IPADDRESS>:27017/catalogue"
ExecStart=/bin/node /app/server.js
SyslogIdentifier=catalogue

[Install]
WantedBy=multi-user.target
```

Ensure you replace `<MONGODB-SERVER-IPADDRESS>` with the MongoDB server IP address.

Load the service:

```shell
systemctl daemon-reload
```

The above command is because we added a new service. We are telling systemd to reload so it will detect the new service.

Start the service:

```shell
systemctl enable catalogue
systemctl start catalogue
```

## Load Schema and Master Data

For the application to work fully functional, we need to load schema to the database. Additionally, we are going to have master data to be seeded for the applications to work. Here in this case, it is list of products we want to sell.

Schemas are usually part of application code and developer will provide them as part of development.

Master data is usually provided by business operations team.

To load schema or master data, we need to install MongoDB client and then we can load it.

To have MongoDB client installed, setup MongoDB repo and install `mongodb-mongosh`.

Create `/etc/yum.repos.d/mongo.repo`.

```shell
vim /etc/yum.repos.d/mongo.repo
```

Add the following content:

```ini
[mongodb-org-7.0]
name=MongoDB Repository
baseurl=https://repo.mongodb.org/yum/redhat/9/mongodb-org/7.0/x86_64/
enabled=1
gpgcheck=0
```

Install MongoDB shell:

```shell
dnf install mongodb-mongosh -y
```

Load master data of the list of products we want to sell and their quantity information:

```shell
mongosh --host MONGODB-SERVER-IPADDRESS </app/db/master-data.js
```

## Frontend Configuration Note

You need to update catalogue server IP address in frontend configuration.

Configuration file is `/etc/nginx/nginx.conf`.

## Verify Data in MongoDB

Use below commands to check data is loaded into MongoDB or not.

Connect to MongoDB:

```shell
mongosh --host MONGODB-SERVER-IPADDRESS
```

Show databases:

```shell
show dbs
```

Use database:

```shell
use catalogue
```

Show collections:

```shell
show collections
```

Get the items in collection:

```shell
db.products.find()
```
