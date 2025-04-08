

Let’s go step-by-step on **how to shard a large existing collection safely and efficiently**, without downtime or data loss.

---

## 🎯 Goal:
You already have a big collection like `userDB.users`, and now you want to shard it to **scale out**.

---

## 🪜 Step-by-Step: Shard an Existing Collection with Data

---

### ✅ 1. **Pre-check: Does the collection already have a good shard key field?**

- Choose a field with:
  - High cardinality
  - Appears in most queries
  - Doesn’t cause hotspotting

📌 **Example:** `geo`, `userId`, or `createdAt`

```js
db.users.createIndex({ geo: 1 })  // Required before sharding
```

> ✅ You **must have an index** on the shard key before sharding.

---

### ✅ 2. Enable sharding on the database
```js
sh.enableSharding("userDB")
```

---

### ✅ 3. Shard the collection
```js
sh.shardCollection("userDB.users", { geo: 1 })
```

> 📌 Note: MongoDB starts with **one chunk** covering the whole range, assigned to one shard.

---

### ✅ 4. Manually Split & Distribute Chunks (Optional but recommended for large collections)

#### a. Pre-split chunks:
```js
sh.splitAt("userDB.users", { geo: "EU" })
sh.splitAt("userDB.users", { geo: "US" })
```

#### b. Move chunks to specific shards:
```js
sh.moveChunk("userDB.users", { geo: "EU" }, "shard2ReplSet")
sh.moveChunk("userDB.users", { geo: "US" }, "shard1ReplSet")
```

---

### ✅ 5. Let MongoDB Auto-Migrate Existing Data (This happens behind the scenes)

- MongoDB uses the **Balancer process** to:
  - Migrate documents to correct shards
  - Split large chunks
  - Balance data over time

💡 Balancer runs **in background**, slowly moving data **without downtime**

---

## 🛠 Optional: Monitor Balancer

Run this to check progress:
```js
sh.isBalancerRunning()
sh.getBalancerState()
db.currentOp({ "desc": /^moveChunk/ })
```

---

## 🧠 Key Points to Know

| Point                        | Explanation |
|------------------------------|-------------|
| Index on shard key required? | ✅ Yes |
| Can I shard without downtime? | ✅ Yes (Balancer runs in background) |
| Data moved instantly?         | ❌ No, moved gradually by balancer |
| Need to reinsert data?        | ❌ No, MongoDB handles it |
| Can I pause balancer?         | ✅ Yes, with `sh.stopBalancer()` |

---

## 🧪 Summary Script

```js
use userDB
db.users.createIndex({ geo: 1 })
sh.enableSharding("userDB")
sh.shardCollection("userDB.users", { geo: 1 })

// Optional split & move
sh.splitAt("userDB.users", { geo: "EU" })
sh.moveChunk("userDB.users", { geo: "EU" }, "shard2ReplSet")

sh.splitAt("userDB.users", { geo: "US" })
sh.moveChunk("userDB.users", { geo: "US" }, "shard1ReplSet")
```

---

