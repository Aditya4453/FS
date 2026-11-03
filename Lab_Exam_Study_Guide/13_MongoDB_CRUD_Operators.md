# Topic 13: MongoDB CRUD Operators

---

### 1. Plain-Language Explanation (Fast Pace)
MongoDB stores data as flexible JSON-like documents inside collections.
* **Create:** `insertOne({ doc })`, `insertMany([ { doc1 }, { doc2 } ])`.
* **Read:** `find({ query })`, `findOne({ query })`. Empty `{}` finds everything.
* **Update:** `updateOne({ filter }, { $set: { field: val } })`, `updateMany(...)`.
* **Delete:** `deleteOne({ filter })`, `deleteMany({ filter })`.
* **Query Operators (Preceded by `$`):**
  * Comparison: `$gt` (>), `$gte` (>=), `$lt` (<), `$lte` (<=), `$ne` (!=), `$in` (matches any in array).
  * Logical: `$and`, `$or`, `$nor`.
  * Update: `$set` (replaces field), `$inc` (increments numbers), `$push` (adds to array).

---

### 2. The Root Idea Behind It
Traditional SQL uses fixed column tables with strict types. MongoDB documents allow nested arrays, embedded objects, and varied schemas per document, matching JavaScript's native object representations. Query operators allow expressive, granular querying directly on nested document fields.

---

### 3. Exact Code From Your Files
**Source File 1:** [`D:/_3rd Year/FS_20/server.js`](file:///D:/_3rd%20Year/FS_20/server.js#L18-L27)  
**Lines 18–27:**
```javascript
18: server.get('/products',async (req,res)=>{
19:     const data = await products.find();
20:     res.send(data);
21: })
22: server.post('/products',async (req,res)=>{
23:     const data = await req.body;
24:     console.log(data);
25:     // const data = {name:"laptop",price:"100000"}
26:     const postResult = await products.insertMany(data);
27:     res.send("data posted");
28: })
```

**Source File 2 (Your Exam Practice Set):** [`D:/_3rd Year/FS/Lab Exam Practice React, DOM & MongoDB.pdf`](file:///D:/_3rd%20Year/FS/Lab%20Exam%20Practice%20React,%20DOM%20&%20MongoDB.pdf)  
```javascript
// Level 1: Find products under 500
db.products.find({ price: { $lt: 500 } })

// Level 2: Find electronics with price between 100 and 1000
db.products.find({
  $and: [
    { category: "electronics" },
    { price: { $gte: 100, $lte: 1000 } }
  ]
})

// Level 3: Update price of laptops
db.products.updateMany(
  { name: /laptop/i },
  { $inc: { price: 50 }, $set: { inStock: true } }
)
```

#### Line-by-Line Walkthrough:
* **`FS_20/server.js` Line 19:** `products.find()`: Queries the MongoDB collection through Mongoose. With no arguments, it returns all product documents.
* **`FS_20/server.js` Line 26:** `products.insertMany(data)`: Accepts an array of objects from `req.body` and inserts them all in a single batch operation.
* **Exam PDF Practice:** Shows `$lt`, `$gte`, `$lte` for numerical bounds filtering.

---

### 4. How It Connects to Other Topics
1. Client sends query: `fetch('/api/products?maxPrice=500')`.
2. Express parses query: `const max = Number(req.query.maxPrice)`.
3. MongoDB Executes: `products.find({ price: { $lte: max } })`.
4. Mongoose returns JavaScript array of documents.
5. Express sends JSON -> React renders product cards.

---

### 5. Common Mistakes & Exact Error Messages
* **Mistake 1: Forgetting `$set` in updates:**
  ```javascript
  db.products.updateOne({ _id: id }, { price: 200 }); // WITHOUT $set!
  ```
  * In modern MongoDB: Throws error `MongoServerError: The update operation document must contain atomic operators.`
* **Mistake 2: Querying numbers as strings:**
  ```javascript
  products.find({ price: { $gt: "100" } }); // Comparing string with number
  ```
  * **Result:** In BSON, strings and numbers compare differently; returns incorrect or empty results.
* **Mistake 3: Invalid ObjectId syntax:**
  ```javascript
  products.findOne({ _id: "64a123..." }); // Passing raw string instead of ObjectId
  ```

---

### 6. Optimized & Best-Practice Version
Always enforce typed schemas, projection to return only needed fields, and sanitize inputs:

```javascript
// Return only name and price, omitting internal _id and __v
const cheapProducts = await Product.find(
  { price: { $lt: 500 } },
  { name: 1, price: 1, _id: 0 }
).lean(); // .lean() skips Mongoose document hydration for 5x faster read performance!
```
