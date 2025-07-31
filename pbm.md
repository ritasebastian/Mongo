Here is a **complete, production-ready guide** to set up a **MongoDB 4.4 three-node replica set** on **Ubuntu 20.04.6 LTS EC2 instances**, and to install and configure **Percona Backup for MongoDB (PBM)** for scheduled backups. This guide assumes 3 EC2 instances with internal connectivity and proper security groups allowing MongoDB ports (27017) and SSH.

---

## **Part 1: Prepare EC2 Instances**

### **1. Initial Setup (All Nodes)**

SSH into each node and perform the following:

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y gnupg curl wget vim
```

### **2. Set Hostnames (Optional but Recommended)**

```bash
sudo hostnamectl set-hostname mongo-node-1  # change 1 → 2 or 3 accordingly
```

### **3. Add to `/etc/hosts` on all 3 nodes**

Assuming private IPs:

```
10.0.0.11 mongo-node-1
10.0.0.12 mongo-node-2
10.0.0.13 mongo-node-3
```

---

## **Part 2: Install MongoDB 4.4 (All Nodes)**

### **1. Import MongoDB GPG Key**

```bash
wget -qO - https://www.mongodb.org/static/pgp/server-4.4.asc | sudo apt-key add -
```

### **2. Add MongoDB 4.4 Repository**

```bash
echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu focal/mongodb-org/4.4 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-4.4.list
```

### **3. Install MongoDB**

```bash
sudo apt update
sudo apt install -y mongodb-org
```

---

## **Part 3: Configure MongoDB Replica Set**

### **1. Edit Config: `/etc/mongod.conf`**

```yaml
# Relevant sections:
net:
  bindIp: 0.0.0.0

replication:
  replSetName: rs0
```

### **2. Enable and Start MongoDB**

```bash
sudo systemctl enable mongod
sudo systemctl start mongod
```

---

## **Part 4: Initiate Replica Set (on Primary Only)**

### **1. Connect to Mongo Shell**

```bash
mongo
```

### **2. Initiate Replica Set**

```js
rs.initiate({
  _id: "rs0",
  members: [
    { _id: 0, host: "mongo-node-1:27017" },
    { _id: 1, host: "mongo-node-2:27017" },
    { _id: 2, host: "mongo-node-3:27017" }
  ]
})
```

### **3. Check Replica Set Status**

```js
rs.status()
```

---

## **Part 5: Install and Configure PBM (All Nodes)**

### **1. Add Percona Repository**

```bash
wget https://repo.percona.com/apt/percona-release_latest.generic_all.deb
sudo dpkg -i percona-release_latest.generic_all.deb
sudo percona-release setup psmdb44
sudo apt update
```

### **2. Install PBM Agent**

```bash
sudo apt install -y percona-backup-mongodb
```

### **3. Create a PBM User in MongoDB**

On the **primary node**:

```js
use admin
db.createUser({
  user: "pbmuser",
  pwd: "strongpassword",
  roles: [
    { role: "backup", db: "admin" },
    { role: "restore", db: "admin" },
    { role: "clusterMonitor", db: "admin" }
  ]
})
```

---

## **Part 6: Configure PBM**

### **1. Create PBM Config File**

**On all nodes**, edit `/etc/pbm-agent.yaml`:

```yaml
mongodb:
  uri: "mongodb://pbmuser:strongpassword@mongo-node-1:27017,mongo-node-2:27017,mongo-node-3:27017/?replicaSet=rs0"
```

### **2. Enable and Start PBM Agent**

```bash
sudo systemctl enable pbm-agent
sudo systemctl start pbm-agent
```

---

## **Part 7: Configure PBM Backups**

### **1. Setup Storage (e.g., S3)**

You can use S3-compatible storage for backups. Set credentials:

```bash
pbm config --file /etc/pbm-agent.yaml --mongodb-uri "mongodb://pbmuser:strongpassword@mongo-node-1:27017/?replicaSet=rs0" \
--storage.type s3 \
--storage.s3.bucket my-mongodb-backups \
--storage.s3.endpoint s3.amazonaws.com \
--storage.s3.region us-east-1 \
--storage.s3.access-key-id <AWS_ACCESS_KEY> \
--storage.s3.secret-access-key <AWS_SECRET>
```

### **2. Create Backup Schedule (CRON)**

```bash
pbm config --set-schedule '0 2 * * *' --set-backup.name nightly
```

You can verify:

```bash
pbm config
pbm list-schedules
```

---

## **Part 8: Validate Backup and Restore**

### **1. Run a Manual Backup**

```bash
pbm backup
pbm status
```

### **2. List and Restore**

```bash
pbm list
pbm restore <backup-name>
```

---

## **Security & Best Practices**

* Use **IAM roles** or **encrypted credentials** for S3.
* Use **TLS** and **authentication** for MongoDB in production.
* Regularly test restore operations.
* Set up **monitoring** with `mongostat`, `mongotop`, or Prometheus exporters.
* Ensure all timezones are synced (use `ntp` or `chrony`).
* Use dedicated EBS volumes for `/var/lib/mongodb`.

---

## **Bonus: Automation with SystemD & Crontab**

PBM handles scheduled backups internally via its own schedule. For additional custom scripts (e.g., validation):

```bash
sudo crontab -e
# Example: Validate PBM backups daily
0 3 * * * /usr/local/bin/check-pbm-status.sh >> /var/log/pbm-check.log 2>&1
```

---

Would you like a **Terraform** or **Ansible playbook** to automate this setup?
