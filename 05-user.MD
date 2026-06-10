# User

User is a microservice that is responsible for user logins and registrations service in Roboshop e-commerce portal.

Developer has chosen NodeJS. Check with developer which version of NodeJS is needed.

Developer has set a context that it can work with NodeJS greater than 20.

## Install NodeJS

By default NodeJS 16 is available. We would like to enable NodeJS 20 and install it.

You can list modules by using:

```shell
dnf module list
```

Disable current module and enable required module:

```shell
dnf module disable nodejs -y
dnf module enable nodejs:20 -y
```

Install NodeJS:

```shell
dnf install nodejs -y
```

## Configure Application

Our application is developed by the user and it is not having any RPM software like other softwares. So we need to configure every step manually.

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
curl -L -o /tmp/user.zip https://roboshop-artifacts.s3.amazonaws.com/user-v3.zip
cd /app
unzip /tmp/user.zip
```

Every application developed by development team will have some common softwares that they use as libraries. This application also has dependencies defined in the application configuration.

Download the dependencies:

```shell
cd /app
npm install
```

## Setup SystemD User Service

We need to setup a new service in systemd so `systemctl` can manage this service.

We already discussed in Linux basics that advantages of `systemctl` managing services. Hence, we are taking that approach, which is also a standard way in the OS.

Create `/etc/systemd/system/user.service`.

```shell
vim /etc/systemd/system/user.service
```

Add the following content:

```ini
[Unit]
Description = User Service

[Service]
User=roboshop
Environment=MONGO=true
Environment=REDIS_URL='redis://<REDIS-IP-ADDRESS>:6379'
Environment=MONGO_URL="mongodb://<MONGODB-SERVER-IP-ADDRESS>:27017/users"
ExecStart=/bin/node /app/server.js
SyslogIdentifier=user

[Install]
WantedBy=multi-user.target
```

Ensure you replace `<REDIS-IP-ADDRESS>` and `<MONGODB-SERVER-IP-ADDRESS>` with the actual server IP addresses.

Load the service:

```shell
systemctl daemon-reload
```

The above command is because we added a new service. We are telling systemd to reload so it will detect the new service.

Start the service:

```shell
systemctl enable user
systemctl start user
```
