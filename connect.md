
---

## 🎯 Goal: Connect to the **Primary Node** of a Replica Set

When you connect to a **replica set** using `mongosh`, you usually want to connect to the **primary** to:
- Run admin commands
- Insert/update data (only allowed on primary)
- Run `rs.status()`, etc.

---

## ✅ Option 1: **Replica Set-Aware Connection (Automatic Primary Detection)**

MongoDB will **automatically detect the current primary** when you connect via a replica set URI:

```bash
mongosh "mongodb://node4:27018,node5:27018,node6:27018/?replicaSet=shard1ReplSet"
```

✔️ This will connect to the **current primary** of the replica set `shard1ReplSet`.

🧠 Even if the primary changes (failover), this connection will still work.

---

## ✅ Option 2: **Connect to a Specific Host (You Think Is Primary)**

If you **know** which node is the primary (e.g., `node4` is primary):

```bash
mongosh --host node4 --port 27018
```

📌 But — this could fail for writes if the node is **not the primary anymore** (read-only behavior).

---

## ✅ Option 3: **Find Out Which Node is Primary**

Use any member (or mongos), and run:
```js
rs.status()
```

Look for:
```json
"stateStr" : "PRIMARY"
```

OR:
```js
rs.isMaster()
```

Returns:
```json
"ismaster" : true,
"primary" : "node4:27018"
```

---

## 💡 Bonus: Use Shell Alias to Always Connect to Primary

Create a shell alias in your terminal (for quick access):
```bash
alias mongo-shard1='mongosh "mongodb://node4:27018,node5:27018,node6:27018/?replicaSet=shard1ReplSet"'
```

Now just type:
```bash
mongo-shard1
```

---

## ✅ Summary

| Task                     | Command |
|--------------------------|---------|
| Connect to shard1 primary (auto) | `mongosh "mongodb://node4,node5,node6/?replicaSet=shard1ReplSet"` |
| Connect to config primary (auto) | `mongosh "mongodb://node1,node2,node3/?replicaSet=cfgReplSet"` |
| Force connect to specific node   | `mongosh --host node4 --port 27018` |
| Check who's primary              | Run `rs.status()` or `rs.isMaster()` |

---

