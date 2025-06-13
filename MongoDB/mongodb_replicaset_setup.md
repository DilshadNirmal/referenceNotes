# MongoDB Replica Set Deployment & Testing Notes

## Overview

This document captures the complete setup of a geographically distributed MongoDB Replica Set across multiple cloud VMs, from installation to secure authentication and testing with an Express.js API.

---

## 1. GCP Setup

### VM Provisioning

* Launched 3 VM instances in different GCP regions.
* Chose Fedora/Ubuntu as base OS.
* Opened necessary firewall ports for MongoDB:

  * **27017**, **27018**, **27019** (optional custom ports).
* DNS hostnames were configured for each instance to avoid using public IPs directly.

> \[!note]
> **Use Hostnames Instead of IPs**
> Prefer using internal DNS hostnames (e.g., `mongo1.internal`, `mongo2.internal`, etc.) for better maintainability and security.

### System Prep

On each VM:

```bash
sudo dnf update -y           # Or `sudo apt update && sudo apt upgrade -y` for Ubuntu
sudo hostnamectl set-hostname mongo-node-1  # Change per node
```

> \[!note]
> *Optional*
> MongoDB does not require the system hostname to be set to a specific name. It is optional.
>
> *Recommended*
> Setting a proper hostname makes:
>
> * Logs and monitoring easier to interpret
> * Node identity more clear in `rs.status()`
> * Host-based DNS entries (like `mongo1.internal`) more meaningful

---

## 2. MongoDB Installation

### Install MongoDB

For Fedora:

```bash
sudo dnf install -y mongodb mongodb-server
```

For Ubuntu:

```bash
wget -qO - https://www.mongodb.org/static/pgp/server-7.0.asc | sudo apt-key add -
echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu focal/mongodb-org/7.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-7.0.list
sudo apt update
sudo apt install -y mongodb-org
```

Enable and start the service:

```bash
sudo systemctl enable mongod
sudo systemctl start mongod
```

---

## 3. Time Sync Configuration (NTP)

### Goal

Ensure all MongoDB nodes have synchronized system clocks.

### Steps

```bash
sudo dnf install ntpsec -y        # For Fedora
sudo apt install ntp -y           # For Ubuntu
sudo systemctl start ntpd
sudo systemctl enable ntpd
```

### Verify Sync

```bash
timedatectl status
```

Look for:

* `System clock synchronized: yes`
* `NTP service: active` or clocks in sync even if shows `n/a`

---

## 4. MongoDB Replica Set Initialization

### Configuration

Edit `/etc/mongod.conf` for each node:

```yaml
replication:
  replSetName: "rs0"
net:
  bindIp: 0.0.0.0
  port: 27017  # or 27018, 27019 for other nodes
```

Restart MongoDB:

```bash
sudo systemctl restart mongod
```

---

## 5. Keyfile Authentication Setup

### Generate Keyfile (on one node)

```bash
openssl rand -base64 756 > /etc/mongo-keyfile
sudo chmod 400 /etc/mongo-keyfile
sudo chown mongodb:mongodb /etc/mongo-keyfile
```

Copy this keyfile securely to the same location on all nodes using `scp`:

```bash
scp /etc/mongo-keyfile user@mongo2.internal:/etc/mongo-keyfile
scp /etc/mongo-keyfile user@mongo3.internal:/etc/mongo-keyfile
```

### Enable Keyfile Auth

Update `/etc/mongod.conf`:

```yaml
security:
  authorization: enabled
  keyFile: /etc/mongo-keyfile
```

Restart MongoDB:

```bash
sudo systemctl restart mongod
```

> \[!important]
> All nodes must share the same keyfile content. Permissions must be `chmod 400`, and owner should be `mongodb:mongodb`.

---

## 6. Initialize Replica Set

From one primary node:

```bash
mongosh --host mongo1.internal
```

Run:

```js
rs.initiate({
  _id: "rs0",
  members: [
    { _id: 0, host: "mongo1.internal:27017" },
    { _id: 1, host: "mongo2.internal:27018" },
    { _id: 2, host: "mongo3.internal:27019" }
  ]
})
```

Check:

```js
rs.status()
```

> \[!note]
> Make sure ports are opened and clocks are in sync for replica members to elect a primary.

---

## 7. Create Admin User (after PRIMARY elected)

```bash
mongosh --host mongo1.internal
```

```js
use admin

db.createUser({
  user: "adminUser",
  pwd: "securePass123",
  roles: [ { role: "root", db: "admin" } ]
})
```

---

## 8. Connect to MongoDB With Authentication

```bash
mongosh --host mongo1.internal --port 27017 -u adminUser -p securePass123 --authenticationDatabase admin
```

---

## 9. Express.js API to Test Writes

### Server Code

```js
const express = require('express');
const { MongoClient } = require('mongodb');
const app = express();
app.use(express.json());

const uri = "mongodb://adminUser:securePass123@mongo1.internal:27017,mongo2.internal:27018,mongo3.internal:27019/?replicaSet=rs0&authSource=admin";
const client = new MongoClient(uri);

app.post('/add', async (req, res) => {
  try {
    await client.connect();
    const db = client.db("test");
    const result = await db.collection("data").insertOne(req.body);
    res.json(result);
  } catch (err) {
    res.status(500).json({ error: err.message });
  }
});

app.listen(3001, () => console.log('Server running on port 3001'));
```

### Test Insert

```bash
curl -X POST http://localhost:3001/add \
  -H "Content-Type: application/json" \
  -d "$(jq -nc --arg name 'Dilshad' --arg timestamp \"$(date --iso-8601=seconds)\" '{name: $name, timestamp: $timestamp}')"
```

---

## 10. View Inserted Data

```bash
mongosh --host mongo1.internal --port 27017 -u adminUser -p securePass123 --authenticationDatabase admin

use test
db.data.find().pretty()
```

---

## 11. Notes & Issues Faced

* **No primary node initially:** Caused by time sync issues. Resolved using NTP.
* **Keyfile permission issues:** Ensure correct ownership and 400 permission.
* **Cannot create user:** Must wait until replica set has a PRIMARY.
* **Ports:** Use different ports (27017, 27018, 27019) per instance if needed.
* **Use DNS names:** Prefer internal DNS (e.g., `mongo1.internal`) over IPs.
* **Authentication Failed:** Usually from mismatched keyfile or wrong user credentials.

---

## To-Do (Future Expansion)

* Create separate documentation for [Deploying Geographically Distributed Replica Sets](https://www.mongodb.com/docs/manual/tutorial/deploy-geographically-distributed-replica-set/)
* Setup Replica Set Monitoring (MongoDB Ops Manager / Compass)
* Test Automatic Failover
* Add Express routes to monitor replica state or promote member

---

*Created on 2025-06-12 by Dilshad Nirmal (project: Replica Set PoC)*
