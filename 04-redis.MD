# Redis

Redis, or REmote DIctionary Server, is an open-source, in-memory data store used as a database and cache. It is known for ultra-fast performance due to its in-memory architecture, which stores data in RAM. It uses a simple key-value model but supports complex data types.

Versions of the DB software will come from the developer, meaning we need to check with developer.

## Install Redis

By default Redis 6 is available. We would like to enable Redis 7 and install it.

```shell
dnf module disable redis -y
dnf module enable redis:7 -y
```

Install Redis:

```shell
dnf install redis -y
```

Usually Redis opens the port only to `localhost` or `127.0.0.1`, meaning this service can be accessed by the application that is hosted on this server only. However, we need this service to be accessed by another server, so we need to change the config accordingly.

Update the listen address from `127.0.0.1` to `0.0.0.0` in `/etc/redis/redis.conf`.

Update `protected-mode` from `yes` to `no` in `/etc/redis/redis.conf`.

You can edit file by using:

```shell
vim /etc/redis/redis.conf
```

## Start and Enable Redis Service

```shell
systemctl enable redis
systemctl start redis
```
