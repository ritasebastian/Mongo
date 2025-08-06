## Clean **step-by-step guide to build a working MongoDB 4.4.24 + PBM 2.3.1 backup system with PITR and scheduled backups** on **Ubuntu 20.04.6 LTS**.
| Region     | AMI ID                  |
| ---------- | ----------------------- |
| us-east-1  | `ami-0fc5d935ebf8bc3bc` |
| us-west-2  | `ami-06e46074ae430fba6` |
| ap-south-1 | `ami-06b263d6ceff0b3dd` |
| eu-west-1  | `ami-0a0e3e6fd0c06c014` |


---

# 🛠️ Full Build-from-Scratch Guide

## ✅ 1. Install MongoDB 4.4.24 Community Edition

### Add MongoDB GPG key and repo

```bash
wget -qO - https://www.mongodb.org/static/pgp/server-4.4.asc | sudo apt-key add -
echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu focal/mongodb-org/4.4 multiverse" \
  | sudo tee /etc/apt/sources.list.d/mongodb-org-4.4.list
sudo apt update
```

### Fix missing `libssl1.1`

```bash
echo "deb http://security.ubuntu.com/ubuntu focal-security main" | sudo tee /etc/apt/sources.list.d/focal-security.list
sudo apt update
sudo apt install -y libssl1.1
sudo rm /etc/apt/sources.list.d/focal-security.list
sudo apt update
```

### Install MongoDB 4.4.24

```bash
sudo apt install -y mongodb-org=4.4.24 mongodb-org-server=4.4.24 \
  mongodb-org-shell=4.4.24 mongodb-org-mongos=4.4.24 mongodb-org-tools=4.4.24
```

---

## ✅ 2. Convert to Single-Node Replica Set (PBM requirement)

### Edit MongoDB config

```bash
sudo vi /etc/mongod.conf
```

Add:

```yaml
replication:
  replSetName: rs0
```

### Restart and initiate replica set

```bash
sudo systemctl restart mongod
mongo --eval 'rs.initiate()'
```

---

## ✅ 3. Download and Install PBM 2.3.1 (manual method)

### Download PBM binaries

```bash
wget https://downloads.percona.com/downloads/percona-backup-mongodb/percona-backup-mongodb-2.3.1/binary/tarball/percona-backup-mongodb-2.3.1-x86_64.tar.gz
tar -xzf percona-backup-mongodb-2.3.1-x86_64.tar.gz
cd percona-backup-mongodb-2.3.1
sudo mv pbm pbm-agent /usr/local/bin/
sudo chmod +x /usr/local/bin/pbm /usr/local/bin/pbm-agent
```

---

## ✅ 4. Configure PBM

### Create config file

```bash
sudo mkdir -p /etc/pbm
sudo vi /etc/pbm/pbm-config.yaml
```

Paste:

```yaml
storage:
  type: filesystem
  filesystem:
    path: /var/backups/pbm
```

### Apply config

```bash
export PBM_MONGODB_URI="mongodb://127.0.0.1:27017"
pbm config --mongodb-uri="mongodb://127.0.0.1:27017" --file /etc/pbm/pbm-config.yaml
```

---

## ✅ 5. Enable Point-In-Time Recovery (PITR)

```bash
pbm config \
  --set pitr.enabled=true \
  --set pitr.oplogSpanMin=1440 \
  --set pitr.compression=s2
```

---

## ✅ 6. Start PBM Agent

### Background start (manual):

```bash
export PBM_MONGODB_URI="mongodb://127.0.0.1:27017"
nohup pbm-agent > /var/log/pbm-agent.log 2>&1 &
```

> You can make this persistent with `systemd` if needed.

---

## ✅ 7. Test Backup + PITR

### Trigger a backup

```bash
pbm backup
```

### Check:

```bash
pbm list
pbm status
```

---

## ✅ 8. Schedule Daily Full Backup and Auto-Purge

### Edit cron:

```bash
sudo crontab -e
```

Add:

```cron
0 0 * * * /usr/local/bin/pbm backup >> /var/log/pbm-cron.log 2>&1
30 0 * * * /usr/local/bin/pbm purge --older-than 7d --confirm >> /var/log/pbm-purge.log 2>&1
```

---

# 🎯 System Ready!

### What You Now Have:

| Feature                    | Status                  |
| -------------------------- | ----------------------- |
| MongoDB 4.4.24 Replica Set | ✅ Enabled               |
| PBM 2.3.1 Installed        | ✅ Manual binary install |
| Local Filesystem Backup    | ✅ `/var/backups/pbm`    |
| PITR                       | ✅ Enabled               |
| Daily Full Backups         | ✅ Via `cron`            |
| Backup Auto-Rotation       | ✅ Via `pbm purge`       |
| Agent Auto-Start           | 🔄 Optional (`systemd`) |

---


