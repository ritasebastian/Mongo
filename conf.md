
---

## 📂 Directory Layout Suggestion (Optional)

```bash
/mongodb
├── configsvr/
├── shard1/
├── shard2/
├── shard3/
├── arbiter/
└── logs/
```

---

## 🧾 1. **Config Server - `configsvr.conf`**

```yaml
# configsvr.conf
systemLog:
  destination: file
  path: /mongodb/logs/configsvr.log
  logAppend: true

storage:
  dbPath: /mongodb/configsvr
  journal:
    enabled: true

net:
  bindIp: 0.0.0.0
  port: 27019

replication:
  replSetName: cfgReplSet

sharding:
  clusterRole: configsvr
```

---

## 🧾 2. **Shard Node - `shard1.conf`, `shard2.conf`, etc.**

```yaml
# shard1.conf
systemLog:
  destination: file
  path: /mongodb/logs/shard1.log
  logAppend: true

storage:
  dbPath: /mongodb/shard1
  journal:
    enabled: true

net:
  bindIp: 0.0.0.0
  port: 27018

replication:
  replSetName: shard1ReplSet

sharding:
  clusterRole: shardsvr
```

👉 For `shard2.conf`, just change:
- `path` to `/mongodb/shard2`
- `replSetName` to `shard2ReplSet`

---

## 🧾 3. **Arbiter - Reuse `shard#.conf`**

✅ Arbiter is **just a mongod node** with no data:
- Start it using the **same config** as a shard node
- Add to replica set as arbiter like this:

```js
rs.addArb("arbiter-host:27018")
```

👉 You don’t need a special config — just empty `dbPath`.

---

## 🧾 4. **mongos Router - `mongos.conf`**

```yaml
# mongos.conf
systemLog:
  destination: file
  path: /mongodb/logs/mongos.log
  logAppend: true

net:
  port: 27017
  bindIp: 0.0.0.0

sharding:
  configDB: cfgReplSet/config1:27019,config2:27019,config3:27019
```

> Replace `config1`, `config2`, `config3` with real hostnames or IPs.

---

## ▶️ How to Run with Config

```bash
mongod -f /path/to/configsvr.conf
mongod -f /path/to/shard1.conf
mongod -f /path/to/shard2.conf
mongod -f /path/to/arbiter.conf
mongos -f /path/to/mongos.conf
```

---

