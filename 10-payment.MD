# Payment

This service is responsible for payments in Roboshop e-commerce app.

This service is written on Python 3.x, so Python is needed to run this app.

Developer has chosen Python. Check with developer which version of Python is needed.

## Install Python 3

```shell
dnf install python3 gcc python3-devel -y
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
curl -L -o /tmp/payment.zip https://roboshop-artifacts.s3.amazonaws.com/payment-v3.zip
cd /app
unzip /tmp/payment.zip
```

Every application developed by development team will have some common softwares that they use as libraries. This application also has dependencies defined in the application configuration.

Download the dependencies:

```shell
cd /app
pip3 install -r requirements.txt
```

## Setup SystemD Payment Service

We need to setup a new service in systemd so `systemctl` can manage this service.

We already discussed in Linux basics that advantages of `systemctl` managing services. Hence, we are taking that approach, which is also a standard way in the OS.

Create `/etc/systemd/system/payment.service`.

```shell
vim /etc/systemd/system/payment.service
```

Add the following content:

```ini
[Unit]
Description=Payment Service

[Service]
User=root
WorkingDirectory=/app
Environment=CART_HOST=<CART-SERVER-IPADDRESS>
Environment=CART_PORT=8080
Environment=USER_HOST=<USER-SERVER-IPADDRESS>
Environment=USER_PORT=8080
Environment=AMQP_HOST=<RABBITMQ-SERVER-IPADDRESS>
Environment=AMQP_USER=roboshop
Environment=AMQP_PASS=roboshop123

ExecStart=/usr/local/bin/uwsgi --ini payment.ini
ExecStop=/bin/kill -9 $MAINPID
SyslogIdentifier=payment

[Install]
WantedBy=multi-user.target
```

Ensure you replace `<CART-SERVER-IPADDRESS>`, `<USER-SERVER-IPADDRESS>`, and `<RABBITMQ-SERVER-IPADDRESS>` with the actual server IP addresses.

Load the service:

```shell
systemctl daemon-reload
```

The above command is because we added a new service. We are telling systemd to reload so it will detect the new service.

Start the service:

```shell
systemctl enable payment
systemctl start payment
```
