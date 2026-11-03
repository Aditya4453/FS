# Topic 16: MongoDB Indexes & Query Execution

---

### 1. Plain-Language Explanation (Fast Pace)
Without an index, finding one document in a collection with 1,000,000 documents requires checking every single document from first to last (called a **Collection Scan** or `COLLSCAN`, $O(N)$ time complexity).
* An **Index** is a sorted look-up table stored in RAM (using a B-Tree data structure).
* Instead of scanning 1,000,000 items, MongoDB jumps directly to the entry using binary search in $O(\log N)$ time (called an **Index Scan** or `IXSCAN`).
* Single field index: `db.products.createIndex({ category: 1 })`.
* Compound index: `db.products.createIndex({ category: 1, price: -1 })`.
* Unique index: `db.users.createIndex({ email: 1 }, { unique: true })`.
* Check query speed: `db.products.find({ ... }).explain("executionStats")`.

---

### 2. The Root Idea Behind It
RAM is 100,000 times faster than disk. Indexes keep sorted pointers to documents in RAM so queries execute in 1 millisecond rather than several seconds, preventing database CPU exhaustion.

---

### 3. Exact Code From Your Files
**Source File 1 (Your Contact Model):** [`D:/_3rd Year/Portfolio_Assignment/server/models/Contact.js`](file:///D:/_3rd%20Year/Portfolio_Assignment/server/models/Contact.js#L35-L42)  
**Lines 35–42:**
```javascript
35:   },
36:   {
37:     timestamps: true, // Automatically indexes and manages createdAt and updatedAt!
38:   }
39: );
40: 
41: module.exports = mongoose.model('Contact', contactSchema);
```

**Source File 2 (Your Course Indexing Guide):** [`D:/_3rd Year/FS/MongoDB Lecture 7 – Indexing.html`](file:///D:/_3rd%20Year/FS/MongoDB%20Lecture%207%20–%20Indexing.html)  
```javascript
// Create compound index for fast catalog filtering
db.products.createIndex({ category: 1, price: -1 })

// Inspect execution stats to verify IXSCAN
db.products.find({ category: "electronics", price: { $lte: 500 } })
  .explain("executionStats")
```

#### Line-by-Line Walkthrough:
* **`createIndex({ category: 1, price: -1 })`:** Creates a compound index. Documents are sorted primarily by `category` ascending; within matching categories, sorted by `price` descending.
* **`explain("executionStats")`:** Returns an execution diagnostics report.
* Look for:
  * `stage: "IXSCAN"` (Good! Index used).
  * `stage: "COLLSCAN"` (Bad! Full collection scan).
  * `totalDocsExamined`: Number of actual documents read from disk.
  * `nReturned`: Number of documents that matched the query. Ideal ratio is `totalDocsExamined === nReturned`.

---

### 4. How It Connects to Other Topics
1. Route called: `server.get('/products/:category', ...)`.
2. Query runs: `products.find({ category: req.params.category })`.
3. If indexed on `category`: Database searches B-Tree in RAM in 0.5ms.
4. If NOT indexed: Database reads every document off disk, causing server lag.

---

### 5. Common Mistakes & Exact Error Messages
* **Mistake 1: Wrong compound index field order (The ESR Rule):**
  * Rule: **Equality** fields first, **Sort** fields second, **Range** fields last!
  * If your query is `{ category: "shoes", price: { $gt: 50 } }`, index `{ price: 1, category: 1 }` is inefficient because the range query on `price` prevents using the second key!
* **Mistake 2: Over-indexing:**
  * Every index consumes RAM and slows down `insertOne()` / `updateOne()` because every write must update all B-Trees.
* **Mistake 3: Duplicate key error on unique index:**
  * **Error:** `E11000 duplicate key error collection: crud.users index: email_1 dup key: { email: "test@domain.com" }`.

---

### 6. Optimized & Best-Practice Version
Always verify query execution using `explain("executionStats")` and declare indexes inside Mongoose schemas:

```javascript
// In models/Product.js
productSchema.index({ category: 1, price: -1 }); // Compound index
productSchema.index({ name: 'text' });           // Full-text search index
```
