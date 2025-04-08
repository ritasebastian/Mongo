
---

## ✅ **Basic Level (Conceptual Understanding)**

1. **What is MongoDB sharding?**  
   Sharding is MongoDB's method for **horizontal scaling**, where data is distributed across multiple machines (shards).

2. **Why do we use sharding in MongoDB?**  
   To handle **large datasets** and **high throughput** by distributing data and queries across multiple nodes.

3. **What are the key components of a sharded cluster?**  
   - **Shards**
   - **Config servers**
   - **mongos router**

4. **What is a shard?**  
   A shard is a MongoDB **replica set** that holds a subset of the data in the sharded cluster.

5. **What is a config server?**  
   It stores metadata about the cluster, such as which data is on which shard and chunk ranges.

6. **What is mongos?**  
   A **query router** that routes client requests to the appropriate shard.

7. **What is a shard key?**  
   A field or combination of fields used to **partition data across shards**.

8. **Can a sharded collection exist without a shard key?**  
   ❌ No, a shard key is **mandatory** to enable sharding on a collection.

9. **Where is the shard key stored?**  
   As part of the collection documents and in the **metadata** managed by config servers.

10. **Can you change the shard key after sharding a collection?**  
   No, you must **drop and recreate** the collection with a new shard key (or use MongoDB 5.0+ resharding feature).

---

## ✅ **Intermediate Level (Practical Usage)**

11. **How does MongoDB decide which shard to store a document in?**  
   Based on the value of the **shard key**, MongoDB maps it to a **chunk range** on a shard.

12. **What is a chunk in MongoDB?**  
   A contiguous range of shard key values that is stored on a single shard.

13. **What is the default chunk size?**  
   64MB

14. **What happens when a chunk exceeds the size limit?**  
   It is automatically **split** into smaller chunks.

15. **How does MongoDB balance chunks across shards?**  
   Using the **Balancer** process that moves chunks to maintain even distribution.

16. **How do you monitor chunk distribution?**  
   Use `sh.status()` and `db.collection.getShardDistribution()`.

17. **How do you manually move a chunk?**  
   `sh.moveChunk("db.collection", { shardKey: value }, "shardName")`

18. **How do you pre-split chunks?**  
   Use `sh.splitAt()` with known shard key boundaries.

19. **How do you pin a chunk to a specific shard?**  
   Use `sh.addShardToZone()` and `sh.updateZoneKeyRange()`.

20. **What is a hot chunk?**  
   A chunk that receives disproportionately high read/write traffic.

---

## ✅ **Advanced Level (Design & Optimization)**

21. **What is the difference between hashed and ranged shard keys?**  
   - **Hashed:** Even distribution, no control over location.
   - **Ranged:** You control ranges, useful for range queries.

22. **When should you use hashed shard keys?**  
   For **write-intensive workloads** with no range queries.

23. **When should you use ranged shard keys?**  
   For **range queries** and **geo-distribution** (e.g., sharding by region or time).

24. **What is a jumbo chunk?**  
   A chunk that is too large to split or move, blocking balancing.

25. **How to fix jumbo chunks?**  
   - Split manually
   - Use smaller documents
   - Avoid unbounded shard key fields

26. **What happens if mongos goes down?**  
   Applications can failover to another mongos — it’s stateless and can be scaled horizontally.

27. **Can I write directly to a shard?**  
   Not recommended. Always write through `mongos`.

28. **What happens if a config server goes down?**  
   You need **a majority** of config servers to be online for the cluster to work.

29. **What happens during a shard failover?**  
   Replica set elects a new primary, no impact if mongos is used properly.

30. **What is a primary shard?**  
   The default shard where **unsharded collections** in a DB are stored.

---

## ✅ **Real-World Scenarios & Admin Tasks**

31. **How do you shard an existing collection with large data?**  
   - Create index on shard key
   - Enable sharding
   - Pre-split + move chunks
   - Let the balancer run in background

32. **How do you migrate a primary shard of a DB?**  
   Use `db.adminCommand({ movePrimary: "db", to: "shardName" })`

33. **Can a collection be both sharded and replicated?**  
   Yes — each shard is a **replica set**, so you get **both**.

34. **What is zone sharding (tag-aware sharding)?**  
   Mapping data ranges to physical locations using **zones**.

35. **What tools can you use to monitor sharded clusters?**  
   - `mongostat`
   - `mongotop`
   - Ops Manager
   - `sh.status()`

36. **How do you prevent write skew in sharded clusters?**  
   - Use hashed keys
   - Avoid monotonic shard keys (like timestamps)

37. **What is the use of the `moveChunk` command?**  
   Manually redistribute data if balancing is off.

38. **Can I disable the balancer?**  
   Yes, using `sh.stopBalancer()` and re-enable with `sh.startBalancer()`.

39. **What happens to unsharded collections?**  
   Stored entirely on the **primary shard** of the DB.

40. **Can two collections use the same shard key?**  
   Yes, but they must be **sharded separately**.

---

## ✅ **Expert/Architecture-Level Questions**

41. **How does sharding affect backup strategies?**  
   You need **coordinated backups** across shards and config servers (e.g., using Ops Manager).

42. **Can I run a sharded cluster on one node?**  
   Yes for dev/testing — not in production.

43. **What’s the risk of not using `mongos` in your application?**  
   You can end up with stale routing, inconsistent data access, or targeting the wrong shard.

44. **What is sharding awareness in application logic?**  
   Designing the app to send queries with shard key to avoid scatter-gather.

45. **Can I run analytics on sharded data?**  
   Yes, but use **aggregation pipelines** carefully and always **target by shard key**.

46. **What if a query doesn’t include the shard key?**  
   It becomes a **scatter-gather query**, which hits all shards = slower.

47. **What happens if two documents have the same shard key?**  
   Both go to the same chunk/shard — no issue.

48. **How can you enforce geo-locality in a global sharded cluster?**  
   Use **zones + region-based `mongos` routers** + shard key design (e.g., `geo` field).

49. **How does MongoDB handle re-sharding in v5.0+?**  
   **Online resharding** using background migration while the app is live.

50. **What’s a good shard key for time-series data?**  
   Avoid raw timestamps — use **hashed timestamp** or **composite shard key** like `{ region, timestamp }`.

Of course, Kanii! Here's **30 more MongoDB sharding interview questions and answers** to continue your preparation — focusing on **tricky internals**, **performance tuning**, and **real-world troubleshooting**. This brings your total to **80 questions**!

---


### 51. **What is a scatter-gather query in MongoDB?**  
A query that does **not include the shard key**, so it is **broadcasted to all shards**. It’s slower and resource-intensive.

---

### 52. **How can you avoid scatter-gather queries?**  
- Always **include the shard key** in your queries.  
- Use **compound indexes** starting with the shard key.

---

### 53. **Can I use compound shard keys?**  
✅ Yes. You can shard on `{ userId: 1, orderId: 1 }` to increase cardinality and better control chunk splits.

---

### 54. **What are tag-aware sharding zones used for?**  
To **pin specific data ranges** (like countries or customers) to **specific shards or regions**.

---

### 55. **What happens if I insert a document without a shard key into a sharded collection?**  
❌ It will **fail with an error**. Shard key is **mandatory** for inserts into a sharded collection.

---

### 56. **Can I create a TTL index on a sharded collection?**  
✅ Yes, but the TTL field **must be in the shard key or indexed** properly.

---

### 57. **How does MongoDB choose the primary shard for a new database?**  
It picks the **least loaded shard**, or you can **movePrimary** manually later.

---

### 58. **What if I shard by a low-cardinality field like `gender` or `country`?**  
This leads to **uneven chunk distribution** and **hotspots** — it's not recommended unless used with **zones**.

---

### 59. **How do you reshard a collection in MongoDB 5.0+?**  
Using the `reshardCollection` command which **reshuffles data** to a new shard key **online**.

---

### 60. **What causes a shard to be under-utilized?**  
- Poor shard key selection  
- Uneven chunk distribution  
- Balancer disabled  
- Shard not assigned any chunks

---

### 61. **Can sharded collections span across cloud regions?**  
✅ Yes, with **zone-based sharding** and **multi-region clusters** in MongoDB Atlas or manual setups.

---

### 62. **Can you use `$lookup` in sharded collections?**  
✅ Yes (MongoDB 3.6+), but **both collections must be sharded on the same key** or one must be **unsharded**.

---

### 63. **What happens if config servers are not in sync?**  
Cluster becomes **inaccessible** — MongoDB requires **majority of config servers** to be up for metadata access.

---

### 64. **What is the `BalancerRound`?**  
Each execution cycle of the balancer process — it tries to even out chunk distribution.

---

### 65. **How can I monitor the balancer process?**  
Run:  
```js
sh.isBalancerRunning()
sh.getBalancerState()
```

Check logs with:
```bash
grep "moveChunk" mongod.log
```

---

### 66. **What is a migration threshold in balancing?**  
If a shard has **more than 2 chunks** over the average, the balancer moves them to other shards.

---

### 67. **Can I shard system collections?**  
❌ No. System collections (e.g., `system.users`, `system.js`) cannot be sharded.

---

### 68. **Can capped collections be sharded?**  
❌ No. MongoDB does not support sharding **capped collections**.

---

### 69. **What is the impact of frequent chunk splits?**  
- Metadata overhead  
- Increased balancer activity  
- More inter-shard network I/O

---

### 70. **How do I automate chunk pre-splitting during sharding?**  
Use `sh.splitAt()` and `sh.moveChunk()` before bulk loading to avoid initial hotspotting.

---

### 71. **How does sharding affect write durability and consistency?**  
Each shard follows its **own replica set rules**. MongoDB maintains **consistency per shard**, not cluster-wide.

---

### 72. **Is MongoDB sharding ACID-compliant?**  
Yes, MongoDB (4.2+) supports **multi-document transactions** across shards, but with **performance trade-offs**.

---

### 73. **How do you back up a sharded cluster?**  
- Use **mongodump** with `--oplog` per shard  
- Or **Ops Manager** / **MongoDB Atlas automated backup**

---

### 74. **How to restore a sharded cluster backup?**  
You must restore **each shard separately**, then restore the **config server metadata**.

---

### 75. **What are some shard key anti-patterns?**  
- Using `_id` as shard key (monotonic)  
- Low cardinality fields (`gender`, `true/false`)  
- Timestamp-only keys

---

### 76. **What is `maxChunkSize`?**  
Controls when MongoDB should **split a chunk** (default is 64MB)

---

### 77. **How to list all shards in a cluster?**  
```js
sh.status()
db.runCommand({ listShards: 1 })
```

---

### 78. **How can I disable balancing for one collection?**  
```js
sh.disableBalancing("myDB.myCollection")
```

---

### 79. **Can I combine sharding with replica sets?**  
✅ Yes — each **shard is a replica set**, so MongoDB supports both **scaling** and **redundancy**.

---

### 80. **When should I NOT use sharding?**  
- Small data sets (< few GBs)  
- Low write volume  
- Where added complexity isn’t worth the scaling benefits

---


### 81. **What is the role of `sh.enableSharding()`?**  
It enables sharding at the **database level**, allowing collections inside it to be sharded.

---

### 82. **What is the default behavior when you insert a document without a shard key into a sharded collection?**  
It throws an error. Documents **must include the shard key** field to be inserted.

---

### 83. **How do you verify if a collection is sharded?**  
Use `sh.status()` or run:  
```js
db.collection.getShardDistribution()
```

---

### 84. **Can I shard collections that already have a large amount of data?**  
✅ Yes. You must create an index on the shard key first, then shard the collection. Optionally, pre-split chunks.

---

### 85. **What are the risks of using a monotonically increasing shard key?**  
Creates **hotspots**: All new writes go to the **same shard**, causing performance and replication lag issues.

---

### 86. **Can I shard a collection by `_id`?**  
Yes, but it's risky unless it's **hashed** (`{ _id: "hashed" }`) to avoid write skew.

---

### 87. **What happens if I delete a shard?**  
MongoDB **migrates its chunks** to other shards and then **removes** the shard metadata using `removeShard()`.

---

### 88. **Can I temporarily pause the balancer?**  
✅ Yes. Use:
```js
sh.stopBalancer()
```
Restart with:
```js
sh.startBalancer()
```

---

### 89. **What is `mongos`'s role during chunk balancing?**  
`mongos` helps **route operations**, but chunk balancing is done by **config servers + mongod processes**.

---

### 90. **What happens if one shard becomes unavailable?**  
The rest of the cluster stays **online**. Queries hitting the downed shard may fail or partially return results.

---

### 91. **How can you design sharding for multi-tenant SaaS architecture?**  
Shard by **`tenantId`** or `{ tenantId, userId }` to ensure tenant isolation and balanced distribution.

---

### 92. **Can I shard the `admin` or `local` databases?**  
❌ No. These are **system databases** and cannot be sharded.

---

### 93. **What command lists all chunk splits for a collection?**  
You can query the config DB:
```js
use config
db.chunks.find({ ns: "yourDB.yourCollection" })
```

---

### 94. **How can I simulate a shard key hotspot in testing?**  
Insert documents with **sequential values** for the shard key (e.g., `orderId: 1, 2, 3...`) to simulate hotspotting.

---

### 95. **How do you implement geographic sharding (geo-based routing)?**  
Use `zones`, assign `geo` ranges to shards, and place region-based `mongos` routers in local regions.

---

### 96. **Can you run aggregation pipelines on sharded collections?**  
✅ Yes. Aggregation is **shard-aware**, and stages are executed in parallel when possible.

---

### 97. **How does MongoDB determine chunk boundaries?**  
Initially based on **insert patterns**, then auto-split as chunk size crosses threshold (default: 64MB).

---

### 98. **Can I query across multiple databases in a sharded cluster?**  
✅ Yes, but **cross-database joins are not allowed**. You can issue separate queries.

---

### 99. **Can I use `findAndModify` on a sharded collection?**  
✅ Yes, but you **must include the shard key** in the query.

---

### 100. **What are best practices for designing a shard key?**
✅ Should be:
- **High cardinality**
- **Even distribution**
- **Frequently used in queries**
- **Immutable**
- Avoid monotonic or low cardinality values

---
