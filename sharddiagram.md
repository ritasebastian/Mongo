

---

## 🎯 Scenario: You Have 3 Shards — What Does a Typical Production MongoDB Sharded Cluster Look Like?

---

## ✅ Objective:
Design a **production-level** MongoDB sharded cluster with:
- **3 shards**
- **High availability (HA)**
- **Replica sets for redundancy**
- **Distributed deployment**

---

## 🧮 Step 1: Total Nodes Needed in Production

### 💼 Assumptions for Production-Ready Setup:
- Each shard is a **replica set with 3 members** (1 primary, 2 secondaries)
- Config Server is a **replica set with 3 members**
- At least **2 mongos routers** for failover/load balancing

### ✅ Total Node Count:

| Component         | Members per group | Total groups | Total nodes |
|------------------|-------------------|--------------|-------------|
| Shards (3x RS)    | 3                 | 3            | 9           |
| Config Server RS  | 3                 | 1            | 3           |
| mongos Routers    | 1–2               | N/A          | 2           |
| **Total**         |                   |              | **14 nodes** |

👉 **You need minimum 14 nodes** to set this up with redundancy.

---

## 🗺️ Step 2: Recommended Node Distribution

### 🖥️ You can have:
- **Servers A, B, C, D, E...N** (total 14 VMs or bare metal)
- Distribute like this:

| Node  | Role(s)                                           |
|-------|---------------------------------------------------|
| Node 1  | Config Server 1 + mongos 1                      |
| Node 2  | Config Server 2 + mongos 2                      |
| Node 3  | Config Server 3                                 |
| Node 4  | Shard1 - Primary                                |
| Node 5  | Shard1 - Secondary                              |
| Node 6  | Shard1 - Secondary                              |
| Node 7  | Shard2 - Primary                                |
| Node 8  | Shard2 - Secondary                              |
| Node 9  | Shard2 - Secondary                              |
| Node 10 | Shard3 - Primary                                |
| Node 11 | Shard3 - Secondary                              |
| Node 12 | Shard3 - Secondary                              |
| Node 13 | Backup/Monitoring server (optional but useful) |
| Node 14 | Application server or load balancer             |

✅ **Best practice**: Avoid placing all replica set members on the same machine or same availability zone (AZ).

---

## 🛠️ Step-by-Step Cluster Setup (Production Style)

---

### 🔹 Step 1: Setup Config Server Replica Set

- On **Node 1, Node 2, Node 3**
- Start `mongod` with `--configsvr` role and same replSet name (`cfgReplSet`)

```bash
mongod --configsvr --replSet cfgReplSet --dbpath /data/configdb --port 27019 --bind_ip_all --fork --logpath /var/log/mongodb/config.log
```

Then initiate:

```bash
mongo --port 27019
rs.initiate({
  _id: "cfgReplSet",
  configsvr: true,
  members: [
    { _id: 0, host: "node1:27019" },
    { _id: 1, host: "node2:27019" },
    { _id: 2, host: "node3:27019" }
  ]
})
```

---

### 🔹 Step 2: Setup Each Shard as Replica Set

Do this 3 times: one for **shard1ReplSet**, one for **shard2ReplSet**, one for **shard3ReplSet**

Example: for shard1

```bash
mongod --shardsvr --replSet shard1ReplSet --dbpath /data/shard --port 27018 --bind_ip_all --fork --logpath /var/log/mongodb/shard1.log
```

Then connect and initiate:

```bash
mongo --port 27018
rs.initiate({
  _id: "shard1ReplSet",
  members: [
    { _id: 0, host: "node4:27018" },
    { _id: 1, host: "node5:27018" },
    { _id: 2, host: "node6:27018" }
  ]
})
```

Repeat for `shard2ReplSet` and `shard3ReplSet`.

---

### 🔹 Step 3: Setup mongos Routers

On **Node 1 and Node 2**:

```bash
mongos --configdb cfgReplSet/node1:27019,node2:27019,node3:27019 --bind_ip_all --port 27017 --fork --logpath /var/log/mongodb/mongos.log
```

Connect to `mongos`:

```bash
mongo --host node1:27017
```

Add shards:

```js
sh.addShard("shard1ReplSet/node4:27018,node5:27018,node6:27018")
sh.addShard("shard2ReplSet/node7:27018,node8:27018,node9:27018")
sh.addShard("shard3ReplSet/node10:27018,node11:27018,node12:27018")
```

Enable sharding on a DB:

```js
sh.enableSharding("salesDB")
sh.shardCollection("salesDB.orders", { orderId: "hashed" })
```

---

## 📈 Monitoring & Maintenance

- Use **MongoDB Ops Manager** or **Prometheus + Grafana**
- Check:
  - Chunk balancing: `sh.status()`
  - Slow queries: `db.currentOp()`
  - Failover and replication: `rs.status()`

---

## 🔐 Bonus Best Practices

- ✅ Use TLS and authentication
- ✅ Enable backups (Atlas or Ops Manager)
- ✅ Place config servers in different AZs
- ✅ Use WiredTiger storage engine
- ✅ Monitor performance and balancing regularly

---
![image](https://github.com/user-attachments/assets/50543b9a-aac1-4b2a-9388-20ad15ae43ac)

