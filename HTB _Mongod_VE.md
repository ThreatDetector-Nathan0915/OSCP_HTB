# MongoDB Enumeration Guide

## Port Scanning

Scan all ports to find high-numbered or non-standard services:

```bash
nmap -p- -sV <target-ip>
```

## Identify MongoDB Service

Look for the default MongoDB port (27017) or any unexpected high port showing MongoDB.

Example result:
```
PORT      STATE SERVICE VERSION
27017/tcp open  mongodb MongoDB 4.4.0
```

## Install mongosh (MongoDB Shell)

```bash
curl -O https://downloads.mongodb.com/compass/mongosh-2.3.2-linux-x64.tgz
tar xfv mongosh-2.3.2-linux-x64.tgz
cd mongosh-2.3.2-linux-x64
./bin/mongosh
```

## Connect to MongoDB

```bash
./bin/mongosh "mongodb://<target-ip>:<port>"
```

## Basic MongoDB Commands

List databases:

```javascript
show dbs
```

Switch to a database:

```javascript
use <database-name>
```

List collections:

```javascript
show collections
```

Find documents:

```javascript
db.<collection-name>.find().pretty()
```
