# Topic 11: `req.params`, `req.query`, and `req.body`

---

### 1. Plain-Language Explanation (Fast Pace)
When clients send data to an Express server, they can send it in three distinct places in the HTTP request:
1. `req.params` (Route Parameters): Embedded in the URL path. Used for identifying a specific resource.  
   * Route: `/products/:id` -> URL: `/products/123` -> `req.params.id === "123"`.
2. `req.query` (Query Parameters): Attached after a `?` in the URL. Used for filtering, sorting, and pagination.  
   * URL: `/products?category=electronics&sort=asc` -> `req.query.category === "electronics"`.
3. `req.body` (Request Payload): Secretly carried inside the HTTP body stream. Used for creating or updating large records (passwords, descriptions, forms).  
   * Payload: `{"name":"Laptop"}` -> `req.body.name === "Laptop"` (requires `express.json()`).

---

### 2. The Root Idea Behind It
Different data has different privacy, size, and caching requirements. Passwords should NEVER go in URL query strings (they get saved in browser history, proxy logs, and referer headers). `req.body` carries large payloads securely. `req.params` enables RESTful resource addressing.

---

### 3. Exact Code From Your Files
**Source File 1 (`req.body`):** [`D:/_3rd Year/Portfolio_Assignment/server/controllers/contactController.js`](file:///D:/_3rd%20Year/Portfolio_Assignment/server/controllers/contactController.js#L23-L25)  
**Lines 23–25:**
```javascript
23: const submitContact = async (req, res) => {
24:   try {
25:     const { name, email, subject, message } = req.body;
```

**Source File 2 (`req.params` & `req.body` in FS_20):** [`D:/_3rd Year/FS_20/server.js`](file:///D:/_3rd%20Year/FS_20/server.js#L22-L26)  
**Lines 22–26 & Lines 43–47:**
```javascript
22: server.post('/products',async (req,res)=>{
23:     const data = await req.body;
24:     console.log(data);
25:     const postResult = await products.insertMany(data);
26:     res.send("data posted");
27: })
...
43: // server.get('/:category', async (req,res)=> {
44: //     let result = await fetch(`${API_URL}/categories`);
45: //     let category = req.params.category;
46: // })
```

#### Line-by-Line Walkthrough:
* **`contactController.js` Line 25:** Destructures `name`, `email`, `subject`, `message` from `req.body`. These fields were originally packed by `JSON.stringify()` in React!
* **`FS_20/server.js` Line 23:** Reads `req.body` and stores it into `data`. (Notice: awaiting `req.body` is redundant; `req.body` is already parsed synchronously by `express.json()`).
* **`FS_20/server.js` Line 43:** Shows route parameter definition `/:category`.

---

### 4. How It Connects to Other Topics
* `req.params.id` connects directly to MongoDB: `products.findById(req.params.id)`.
* `req.query` connects directly to MongoDB filters & pagination:  
  `products.find({ category: req.query.category }).limit(Number(req.query.limit))`.
* `req.body` connects directly to Mongoose schema validation & insertion:  
  `Contact.create(req.body)`.

---

### 5. Common Mistakes & Exact Error Messages
* **Mistake 1: Route param name mismatch:**
  * Route definition: `app.get('/users/:userId', ...)`
  * Code: `const id = req.params.id;` // undefined!
  * **Result:** `req.params.id` is `undefined`. Must match parameter name: `req.params.userId`.
* **Mistake 2: Forgetting query params are strings:**
  ```javascript
  const limit = req.query.limit; // e.g. "5" (string!)
  db.collection.find().limit(limit); // May fail or behave unexpectedly
  ```
  * **Fix:** Convert to integer: `parseInt(req.query.limit, 10)` or `Number(req.query.limit)`.
* **Mistake 3: Awaiting `req.body`:**
  * As seen in `FS_20/server.js`: `const data = await req.body;`. While JS allows awaiting non-promises, it demonstrates a misunderstanding of how middleware works.

---

### 6. Optimized & Best-Practice Version
```javascript
// Express controller handling all three:
app.get('/api/products/:category', async (req, res) => {
  const { category } = req.params;            // Route param
  const page = parseInt(req.query.page) || 1; // Query param
  const limit = parseInt(req.query.limit) || 10;
  
  const items = await Product.find({ category })
    .skip((page - 1) * limit)
    .limit(limit);
    
  res.status(200).json(items);
});
```
