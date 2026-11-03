# Topic 15: MongoDB Aggregation Pipelines

---

### 1. Plain-Language Explanation (Fast Pace)
Think of an **Aggregation Pipeline** as an assembly line in a factory.
Documents enter at the top and flow through a series of **Stages**. Each stage performs an operation (filtering, grouping, reshaping, sorting) and outputs transformed documents to the next stage:
* `$match`: Filter stage. Discards unwanted documents immediately (like a conveyor belt filter).
* `$group`: Grouping stage. Groups documents by an `_id` expression and calculates totals (`$sum`, `$avg`, `$max`, `$min`).
* `$project`: Reshaping stage. Renames fields, creates calculated values, or hides sensitive data.
* `$sort`: Orders the aggregated summary documents.
* `$limit`: Keeps only top results.

---

### 2. The Root Idea Behind It
Standard `.find()` queries can only retrieve raw documents. They cannot calculate sums, group sales by category, compute monthly revenue, or join collections. The aggregation pipeline moves data analysis into the high-speed database engine so your Node server doesn't have to load millions of records into RAM to calculate an average.

---

### 3. Exact Code From Your Files
**Source File (Your Exam Practice Set):** [`D:/_3rd Year/FS/Lab Exam Practice React, DOM & MongoDB.pdf`](file:///D:/_3rd%20Year/FS/Lab%20Exam%20Practice%20React,%20DOM%20&%20MongoDB.pdf#Page27)  
**Part C Level 4 (Exam Problem):**
```javascript
// Exam Task: Group products by category, find max price and count of each category
db.products.aggregate([
  // Stage 1: Filter only products that are currently in stock
  { 
    $match: { inStock: true } 
  },
  // Stage 2: Group by category, compute maximum price and total items
  { 
    $group: {
      _id: "$category",
      maxPrice: { $max: "$price" },
      itemCount: { $sum: 1 },
      avgPrice: { $avg: "$price" }
    } 
  },
  // Stage 3: Reshape output fields
  {
    $project: {
      _id: 0,
      category: "$_id",
      maxPrice: 1,
      itemCount: 1,
      averagePrice: { $round: ["$avgPrice", 2] }
    }
  },
  // Stage 4: Sort by highest maxPrice first
  { 
    $sort: { maxPrice: -1 } 
  }
])
```

#### Line-by-Line Walkthrough:
* **Stage 1 (`$match`):** Discards out-of-stock items immediately. Always put `$match` first so subsequent stages process far fewer documents.
* **Stage 2 (`$group`):**
  * `_id: "$category"`: Groups documents by the `category` field. Notice the `$` sign! `"$category"` tells MongoDB to look at the field's *value*.
  * `maxPrice: { $max: "$price" }`: Tracks the highest price in that category.
  * `itemCount: { $sum: 1 }`: Increments the counter by 1 for every document in the group.
* **Stage 3 (`$project`):**
  * `_id: 0`: Suppresses the default `_id` field.
  * `category: "$_id"`: Renames the grouped key back to a user-friendly name `category`.
* **Stage 4 (`$sort`):** Orders grouped results by `maxPrice` descending.

---

### 4. How It Connects to Other Topics
1. Client requests dashboard metrics: `fetch('/api/analytics/category-summary')`.
2. Express controller runs `await Product.aggregate([...])`.
3. Database executes multi-stage pipeline across disk/memory.
4. Aggregated JSON array returns to Express.
5. React receives clean summary objects and renders chart/table components.

---

### 5. Common Mistakes & Exact Error Messages
* **Mistake 1: Forgetting the `$` prefix in field references:**
  ```javascript
  $group: { _id: "category" } // WRONG! Evaluates to the literal string "category"!
  ```
  * **Result:** All documents get grouped into a single group named `"category"`.
  * **Fix:** Must use `"$category"`.
* **Mistake 2: Missing `_id` in `$group` stage:**
  ```javascript
  { $group: { total: { $sum: "$price" } } } // missing _id!
  ```
  * **Error:** `MongoServerError: The field '_id' must be specified in the $group stage`.
  * If you want a total across the ENTIRE collection, use `_id: null`.
* **Mistake 3: Putting `$match` after `$group` when filtering raw fields:**
  * Filtering raw fields *after* grouping forces MongoDB to group millions of unnecessary documents first, destroying performance.

---

### 6. Optimized & Best-Practice Version
Always put `$match` and `$sort` stages at the very beginning of the pipeline where they can utilize B-Tree indexes:

```javascript
// Optimized pipeline utilizing compound index { category: 1, inStock: 1 }
const categoryStats = await Product.aggregate([
  { $match: { inStock: true, price: { $gt: 0 } } },
  { $group: { _id: "$category", avgPrice: { $avg: "$price" }, count: { $sum: 1 } } },
  { $sort: { count: -1 } },
  { $limit: 10 }
]);
```
