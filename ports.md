
---

## 🎯 MongoDB Components and Their Default Ports

| Component         | Default Port | Purpose |
|-------------------|--------------|---------|
| **mongod (Shard)**      | **27018**     | Data node for sharded cluster (can be part of a replica set) |
| **mongod (Config Server)** | **27019**     | Stores cluster metadata and routing info |
| **mongod (Arbiter)**     | **27018** or same as data nodes | Arbiters use the same process/port as regular replica set nodes |
| **mongos (Router)**      | **27017**     | Query router — application connects here |

---

## ✅ Summary

| Component        | Run Command                | Port Example | Notes |
|------------------|----------------------------|--------------|-------|
| Shard Node       | `mongod --shardsvr`        | `27018`      | Part of a replica set (shard1, shard2…) |
| Config Server    | `mongod --configsvr`       | `27019`      | Must run as replica set (`cfgReplSet`) |
| Arbiter          | `rs.addArb("host:27018")`  | `27018`      | No separate port — uses normal mongod port |
| mongos Router    | `mongos --configdb`        | `27017`      | Routes queries to correct shards |

---

## 🧠 Visual Reference

```
┌───────────────┐
│  mongos       │  ← Application connects here (Port 27017)
└───────────────┘
       │
       ▼
┌─────────────────────┐
│ Config Server RS    │  ← 3 nodes (Port 27019)
└─────────────────────┘
       │
       ▼
┌────────────┬────────────┬────────────┐
│  Shard1 RS │  Shard2 RS │  Shard3 RS │  ← Data nodes (Port 27018)
└────────────┴────────────┴────────────┘
   │ Arbiter if needed   (Port 27018)
```

---
