# MongoDB

MongoDB is a popular NoSQL database designed for handling large volumes of unstructured or semi-structured data. Instead of using tables like in traditional SQL databases, MongoDB stores data in flexible, JSON-like documents.

## SQL vs NoSQL MongoDB Comparison

| Feature | SQL Relational DB | NoSQL MongoDB Document DB |
| --- | --- | --- |
| **Data Structure** | Tables with rows and columns | Collections with JSON-like documents BSON |
| **Schema** | Rigid schema must define columns | Dynamic schema schema-less flexible |
| **Query Language** | SQL Structured Query Language | MQL MongoDB Query Language JavaScript style |
| **Best Use Cases** | Banking, ERP, CRM, complex relational systems | Real-time analytics, IoT, content management, apps |
| **Examples** | MySQL, PostgreSQL, Oracle, SQL Server | MongoDB, CouchDB, Amazon DocumentDB |

## Example Document

```json
{
  "_id": "6638fa12345abc",
  "name": "Alice",
  "age": 30,
  "skills": ["Java", "DevOps"],
  "address": {
    "city": "Hyderabad",
    "zip": "500001"
  }
}
```

Developer has chosen the database MongoDB. Hence, we are trying to install it and configure it.

Versions of the DB software will come from the developer, meaning we need to check with developer.

Developer has shared the version information as MongoDB 7.x.

## Setup MongoDB Repo File

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

## Install MongoDB

```shell
dnf install mongodb-org -y
```

## Start and Enable MongoDB Service

```shell
systemctl enable mongod
systemctl start mongod
```

Usually MongoDB opens the port only to `localhost` or `127.0.0.1`, meaning this service can be accessed by the application that is hosted on this server only. However, we need this service to be accessed by another server, so we need to change the config accordingly.

Update the listen address from `127.0.0.1` to `0.0.0.0` in `/etc/mongod.conf`.

```shell
vim /etc/mongod.conf
```

Restart the service to make the changes effective.

```shell
systemctl restart mongod
```
