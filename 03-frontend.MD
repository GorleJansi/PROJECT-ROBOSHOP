# Frontend

The frontend is the service in Roboshop to serve the web content over Nginx. This will have the web frame for the web application.

This is static content and to serve static content we need a web server.

Developer has chosen Nginx as a web server and thus we will install Nginx Web Server.

## Install Nginx

You can list modules by using:

```shell
dnf module list nginx
```

Install Nginx:

```shell
dnf module disable nginx -y
dnf module enable nginx:1.24 -y
dnf install nginx -y
```

## Start and Enable Nginx Service

```shell
systemctl enable nginx
systemctl start nginx
```

Try to access the service once over the browser and ensure you get some default content.

## Configure Frontend Content

Remove the default content that web server is serving.

```shell
rm -rf /usr/share/nginx/html/*
```

Download the frontend content:

```shell
curl -o /tmp/frontend.zip https://roboshop-artifacts.s3.amazonaws.com/frontend-v3.zip
```

Extract the frontend content:

```shell
cd /usr/share/nginx/html
unzip /tmp/frontend.zip
```

Try to access the Nginx service once more over the browser and ensure you get Roboshop content.

## Create Nginx Reverse Proxy Configuration

Create Nginx reverse proxy configuration to reach backend services.

```shell
vim /etc/nginx/nginx.conf
```

Add the following content:

```nginx
user nginx;
worker_processes auto;
error_log /var/log/nginx/error.log notice;
pid /run/nginx.pid;

include /usr/share/nginx/modules/*.conf;

events {
    worker_connections 1024;
}

http {
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile            on;
    tcp_nopush          on;
    keepalive_timeout   65;
    types_hash_max_size 4096;

    include             /etc/nginx/mime.types;
    default_type        application/octet-stream;

    include /etc/nginx/conf.d/*.conf;

    server {
        listen       80;
        listen       [::]:80;
        server_name  _;
        root         /usr/share/nginx/html;

        include /etc/nginx/default.d/*.conf;

        error_page 404 /404.html;
        location = /404.html {
        }

        error_page 500 502 503 504 /50x.html;
        location = /50x.html {
        }

        location /images/ {
          expires 5s;
          root   /usr/share/nginx/html;
          try_files $uri /images/placeholder.jpg;
        }
        location /api/catalogue/ { proxy_pass http://localhost:8080/; }
        location /api/user/ { proxy_pass http://localhost:8080/; }
        location /api/cart/ { proxy_pass http://localhost:8080/; }
        location /api/shipping/ { proxy_pass http://localhost:8080/; }
        location /api/payment/ { proxy_pass http://localhost:8080/; }

        location /health {
          stub_status on;
          access_log off;
        }

    }
}
```

Ensure you replace `localhost` with the actual IP address of those component servers. The word `localhost` is just used to avoid failures on the Nginx server.

Restart Nginx service to load the configuration changes.

```shell
systemctl restart nginx
```
