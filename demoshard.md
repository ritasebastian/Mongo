

---

## ❓ Do you have to run `moveChunk()` every time a document is inserted?

### 🚫 **No!** You **only do it once** during initial setup — like a routing map.

After that:
✅ **Every document insert automatically goes to the correct shard**  
based on the **shard key value** and existing chunk distribution.

---

## 🧠 Here's How It Works:

### 🔁 One-Time Operations:
- `sh.enableSharding("userDB")`
- `sh.shardCollection("userDB.users", { geo: 1 })`
- `sh.splitAt(...)` → defines chunk boundaries
- `sh.moveChunk(...)` → assigns chunks to desired shards

> ✅ After that, **MongoDB mongos router handles everything automatically.**

---

## 📥 What Happens on Each Insert?

Let’s say you insert this:
```js
db.users.insertOne({ userId: 10, name: "Sharmi", geo: "EU" })
```

Here’s what MongoDB does **automatically**:
1. **Mongos** checks `geo` value
2. Finds which **chunk range** it falls in
3. Knows which **shard** owns that chunk
4. Routes the insert to that **shard only**

---

## 💡 Think of it like this:

🔹 `moveChunk()` is like assigning **geo zones** to **post offices**  
🔹 Once assigned, you just **drop the letter**, and MongoDB delivers it to the correct post office.

> 📦 You don't reassign zones every time. You assign once, and routing continues smoothly.

---

## ✅ Summary

| Operation        | One-Time Setup | Needed Per Insert? |
|------------------|----------------|---------------------|
| `enableSharding()` | ✅ Yes         | ❌ No              |
| `shardCollection()` | ✅ Yes        | ❌ No              |
| `splitAt()`       | ✅ Optional    | ❌ No              |
| `moveChunk()`     | ✅ One-time    | ❌ No              |
| Document insert   | ✅ Ongoing     | ✅ Yes (MongoDB routes it automatically) |

---

