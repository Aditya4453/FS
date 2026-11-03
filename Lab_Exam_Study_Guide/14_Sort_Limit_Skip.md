# Topic 14: `sort()`, `limit()`, and `skip()` (Pagination)

---

### 1. Plain-Language Explanation (Fast Pace)
When a database has 100,000 documents, sending all of them to the browser at once crashes the browser and overloads the network.
* `.sort({ field: 1 })`: Orders documents. `1` is Ascending (low to high, A to Z). `-1` is Descending (high to low, Z to A).
* `.limit(N)`: Caps the result to at most `N` documents.
* `.skip(N)`: Jumps over the first `N` documents before picking results.
* **The Universal Pagination Formula:**
  ```javascript
  const skipCount = (pageNumber - 1) * pageSize;
  db.collection.find().sort({ ... }).skip(skipCount).limit(pageSize);
  ```

---

### 2. The Root Idea Behind It
Pagination breaks massive data sets into bite-sized pages (e.g. 10 items per page). Sorting guarantees consistent order across pagination requests so users do not see duplicate items across different pages.

---

### 3. Exact Code From Your Files
**Source File (Your Exam Practice Set):** [`D:/_3rd Year/FS/Lab Exam Practice React, DOM & MongoDB.pdf`](file:///D:/_3rd%20Year/FS/Lab%20Exam%20Practice%20React,%20DOM%20&%20MongoDB.pdf#Page25)  
**Part C Level 2 (Questions 11–15):**
```javascript
// Q11: Top 3 most expensive products
db.products.find().sort({ price: -1 }).limit(3)

// Q12: 5 cheapest products in ascending order
db.products.find().sort({ price: 1 }).limit(5)

// Q13: Page 2 of cheapest products (5 per page)
// Formula: skip = (page - 1) * limit = (2 - 1) * 5 = 5
db.products.find().sort({ price: 1 }).skip(5).limit(5)

// Q14: Sort by category ascending, then price descending
db.products.find().sort({ category: 1, price: -1 })
```

#### Line-by-Line Walkthrough:
* **`sort({ price: -1 })`:** MongoDB orders matching documents from highest price to lowest price.
* **`limit(3)`:** Halts scan after returning top 3.
* **`skip(5).limit(5)`:** Skips items 1 to 5 (Page 1) and returns items 6 to 10 (Page 2).
* **Compound Sort:** `sort({ category: 1, price: -1 })`: Groups by category alphabetically; within each category, orders by price highest to lowest.

---

### 4. How It Connects to Other Topics
1. Client calls `fetch('/api/products?page=2&limit=5')`.
2. Express controller reads: `const page = parseInt(req.query.page); const limit = parseInt(req.query.limit);`.
3. Server executes: `products.find().sort({ createdAt: -1 }).skip((page - 1) * limit).limit(limit)`.
4. MongoDB executes index-backed scan and returns page slice.
5. React receives exactly 5 items and updates state.

---

### 5. Common Mistakes & Exact Error Messages
* **Mistake 1: Sorting without index on large collections:**
  * **Crash Error:** `Executor error during find command :: caused by :: Sort operation used more than the maximum 33554432 bytes of RAM. Add an index, or specify a smaller limit.`
* **Mistake 2: Missing parentheses or argument in `sort()`:**
  ```javascript
  db.products.find().sort("price") // Invalid syntax!
  ```
  * In Mongosh / Mongoose: Throws syntax or casting errors. Must use object syntax: `{ price: 1 }`.
* **Mistake 3: Skipping without a deterministic sort:**
  * If no sort is specified, documents return in natural disk order. Consecutive pages may randomly return duplicate items!

---

### 6. Optimized & Best-Practice Version
Always sort by an indexed field, and return total document count so the frontend can calculate total pages:

```javascript
app.get('/api/products', async (req, res) => {
  const page = Math.max(1, parseInt(req.query.page) || 1);
  const limit = Math.min(50, parseInt(req.query.limit) || 10);
  const skip = (page - 1) * limit;

  const [items, totalCount] = await Promise.all([
    Product.find().sort({ price: 1 }).skip(skip).limit(limit).lean(),
    Product.countDocuments()
  ]);

  res.status(200).json({
    data: items,
    currentPage: page,
    totalPages: Math.ceil(totalCount / limit),
    totalCount
  });
});
```
