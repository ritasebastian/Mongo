

---

## ✅ Goal: Create a Sharded Collection in MongoDB

We'll:
1. Connect to a **mongos** router
2. Create a **database and collection**
3. Choose a **shard key**
4. Enable sharding
5. Shard the collection

---

## 🧪 Step-by-Step: Create a Sharded Collection

### 🧭 Step 1: Connect to mongos
```bash
mongosh --host localhost --port 27017
```
(`mongos` should be running and connected to config servers and shards)

---

### 🗂️ Step 2: Create the Database and Collection
```js
use salesDB
db.orders.insertOne({ orderId: 1001, item: "laptop", qty: 2 })
```

---

### 🔑 Step 3: Enable Sharding on the Database
```js
sh.enableSharding("salesDB")
```

---

### 🧩 Step 4: Shard the Collection with a Shard Key

#### 📌 Option A: Hashed Shard Key (Even distribution)
```js
sh.shardCollection("salesDB.orders", { orderId: "hashed" })
```

#### 📌 Option B: Ranged Shard Key (Custom control)
```js
sh.shardCollection("salesDB.orders", { orderId: 1 })
```

> ✅ Your `insert` documents **must now include `orderId`**, or you’ll get an error.

---

### 📥 Step 5: Insert Sample Documents
```js
db.orders.insertMany([
  { orderId: 1001, item: "Laptop", qty: 2 },
  { orderId: 1002, item: "Mouse", qty: 5 },
  { orderId: 1003, item: "Monitor", qty: 1 }
])
```

---

### 🔍 Step 6: Check the Shard Status
```js
sh.status()
```

You should see:
```txt
salesDB.orders
    shard key: { "orderId" : "hashed" }
    chunks:
        shard1ReplSet: 1
        shard2ReplSet: 1
        shard3ReplSet: 1
```

🎉 That means your data is being split across shards!

---

## 💡 Notes:
- **Sharding only applies to that collection (`orders`)** — other collections are untouched.
- Choose a shard key based on **query pattern + cardinality**.
- All future inserts into `salesDB.orders` must include `orderId`.

---

