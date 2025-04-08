
---

## 🎯 Goal:
You want documents with:
- `"geo": "US"` to go to **shard1**
- `"geo": "EU"` to go to **shard2**
- `"geo": "other"` to go to **shard3**

And all this should happen **automatically** when inserting documents.

---

## ✅ Yes, this is possible using:
### ➤ **Ranged sharding** with a **custom shard key** on the `geo` field.

---

## 🪜 Step-by-Step Setup

### Step 1: Connect to mongos
```bash
mongosh --host localhost --port 27017
```

---

### Step 2: Create database and collection
```js
use userDB
db.users.insertOne({ userId: 1, name: "John", geo: "US" }) // optional dummy insert
```

---

### Step 3: Enable sharding on the database
```js
sh.enableSharding("userDB")
```

---

### Step 4: Shard the collection using `geo` field as a ranged shard key
```js
sh.shardCollection("userDB.users", { geo: 1 })
```

---

### Step 5: Pre-split chunks and assign to specific shards

This is the magic step that maps **specific geo values to specific shards**.

```js
// Create chunk ranges
sh.splitAt("userDB.users", { geo: "EU" })
sh.splitAt("userDB.users", { geo: "US" })

// Now you have 3 chunks:
// {minKey → "EU"}         → geo: "other"
// {"EU" → "US"}           → geo: "EU"
// {"US" → maxKey}         → geo: "US"

// Move the chunks to specific shards
sh.moveChunk("userDB.users", { geo: "EU" }, "shard2ReplSet")
sh.moveChunk("userDB.users", { geo: "US" }, "shard1ReplSet")
sh.moveChunk("userDB.users", { geo: "ZZZ" }, "shard3ReplSet")  // move final range
```

> ✅ Now:
- `"geo": "US"` → goes to `shard1ReplSet`
- `"geo": "EU"` → goes to `shard2ReplSet`
- `"geo": "other"` (or anything else like "ASIA") → goes to `shard3ReplSet`

---

## 🧪 Test It!

```js
db.users.insertMany([
  { userId: 1, name: "Kanii", geo: "US" },
  { userId: 2, name: "Rita", geo: "EU" },
  { userId: 3, name: "Leo", geo: "ASIA" }
])
```

Then run:
```js
sh.status()
db.users.getShardDistribution()
```

You'll see documents go to the expected shard 🎯

---

## ⚠️ Notes:
- MongoDB compares string values in **lexicographic (alphabetical) order**, so range splitting works based on `"EU" < "US" < others`.
- If `geo` field is missing in a document, it will **not insert** (since shard key is required).
- This works well for a **low cardinality shard key with known values** (like geo, region, category).

---

## ✅ Summary

| `geo` Value | Chunk Range             | Goes To         |
|-------------|-------------------------|-----------------|
| "US"        | `{"US" → MaxKey}`       | `shard1ReplSet` |
| "EU"        | `{"EU" → "US"}`         | `shard2ReplSet` |
| "other"     | `{MinKey → "EU"}`       | `shard3ReplSet` |

---

