# Dispatch

Dispatch is the service which dispatches the product after purchase. It is written in GoLang, so we need to install GoLang.

Developer has chosen GoLang. Check with developer which version of GoLang is needed.

## Install GoLang

```shell
dnf install golang -y
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
curl -L -o /tmp/dispatch.zip https://roboshop-artifacts.s3.amazonaws.com/dispatch-v3.zip
cd /app
unzip /tmp/dispatch.zip
```

Every application developed by development team will have some common softwares that they use as libraries. This application also has dependencies defined in the application configuration.

Download the dependencies and build the software:

```shell
cd /app
go mod init dispatch
go get
go build
```

## Setup SystemD Dispatch Service

We need to setup a new service in systemd so `systemctl` can manage this service.

We already discussed in Linux basics that advantages of `systemctl` managing services. Hence, we are taking that approach, which is also a standard way in the OS.

Create `/etc/systemd/system/dispatch.service`.

```shell
vim /etc/systemd/system/dispatch.service
```

Add the following content:

```ini
[Unit]
Description = Dispatch Service

[Service]
User=roboshop
Environment=AMQP_HOST=<RABBITMQ-IP>
Environment=AMQP_USER=roboshop
Environment=AMQP_PASS=roboshop123
ExecStart=/app/dispatch
SyslogIdentifier=dispatch

[Install]
WantedBy=multi-user.target
```

Ensure you replace `<RABBITMQ-IP>` with the actual RabbitMQ server IP address.

Load the service:

```shell
systemctl daemon-reload
```

The above command is because we added a new service. We are telling systemd to reload so it will detect the new service.

Start the service:

```shell
systemctl enable dispatch
systemctl start dispatch
```
