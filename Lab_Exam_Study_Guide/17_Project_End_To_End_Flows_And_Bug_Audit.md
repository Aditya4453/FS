# Topic 17: Project End-to-End Flows & Complete Bug Audit

---

## 1. Project Architecture & End-to-End Request Flows

### Flow A: Portfolio Contact Form Submission
```
[User Types Input]
       │
       ▼
[Contact.jsx (onChange)] ────► e.target.value updates formData state
       │
[User Clicks "Send"]
       │
       ▼
[Contact.jsx (handleSubmit)] ─► e.preventDefault()
       │
       ▼
[fetch('/api/contact', POST)] ─► Serializes formData with JSON.stringify()
       │  HTTP POST /api/contact { name, email, subject, message }
       ▼
[server.js (Middleware)] ──────► cors() -> express.json() parses body into req.body
       │
       ▼
[contactRoutes.js] ────────────► Matches POST '/' -> contactController.submitContact
       │
       ▼
[contactController.js] ────────► Validates fields -> calls Contact.create(req.body)
       │
       ▼
[MongoDB Atlas / MemoryDB] ────► Validates Schema & saves document into 'contacts'
       │
       ▼
[HTTP 201 Response] ───────────► res.status(201).json({ success: true, data: { id } })
       │
       ▼
[Contact.jsx (await res.json)] ► Updates btnState to 'success', resets formData fields
       │
       ▼
[React UI Repaints] ───────────► Checkmark animation appears; inputs clear
```

---

### Flow B: FS_20 Products Catalog Flow
```
[Browser / Postman]
       │
       ▼
[HTTP GET /products]
       │
       ▼
[FS_20/server.js] ─────────────► server.get('/products')
       │
       ▼
[Mongoose products.find()] ────► Queries 'products' collection in MongoDB Atlas
       │
       ▼
[MongoDB Database] ────────────► Returns array of product documents
       │
       ▼
[server.js (res.send(data))] ──► Streams JSON array back to client
```

---

## 2. Complete Bug Audit, Bad Practices & Fixes

### Bug 1: Awaiting `req.body` in `FS_20/server.js`
* **File & Line:** [`D:/_3rd Year/FS_20/server.js` Line 23](file:///D:/_3rd%20Year/FS_20/server.js#L23)
* **Current Code:**
  ```javascript
  server.post('/products', async (req, res) => {
    const data = await req.body; // BAD PRACTICE!
    const postResult = await products.insertMany(data);
    res.send("data posted");
  });
  ```
* **Why it's bad:** `req.body` is already synchronously parsed by the `express.json()` middleware. Awaiting it does nothing and signals a misunderstanding of Promises.
* **The Fix:**
  ```javascript
  server.post('/products', async (req, res) => {
    try {
      const data = req.body;
      const postResult = await products.insertMany(data);
      res.status(201).json({ success: true, count: postResult.length });
    } catch (err) {
      res.status(400).json({ success: false, error: err.message });
    }
  });
  ```

---

### Bug 2: Missing CORS in `FS_20/server.js`
* **File:** [`D:/_3rd Year/FS_20/server.js`](file:///D:/_3rd%20Year/FS_20/server.js)
* **Problem:** There is NO `cors` middleware imported or used.
* **Impact:** If you create a React frontend on `localhost:3000` or `localhost:5173` trying to fetch from `localhost:9000/products`, the browser will immediately block the request with a CORS error!
* **The Fix:**
  ```javascript
  const cors = require('cors');
  server.use(cors());
  ```

---

### Bug 3: Hardcoded Sensitive Database Credentials
* **File & Line:** [`D:/_3rd Year/FS_20/server.js` Line 9](file:///D:/_3rd%20Year/FS_20/server.js#L9)
* **Current Code:**
  ```javascript
  const url = "mongodb+srv://modaniaditya06_db_user:modapk@cluster0.iikrdtr.mongodb.net/crud";
  ```
* **Why it's dangerous:** Committing cleartext passwords to GitHub exposes your database to deletion.
* **The Fix:** Move to `.env`:
  ```javascript
  require('dotenv').config();
  const url = process.env.MONGO_URI;
  ```

---

### Bug 4: Missed `useMemo` Optimization in `Projects.jsx`
* **File & Line:** [`D:/_3rd Year/Portfolio_Assignment/client/src/components/Projects.jsx` Lines 15–22](file:///D:/_3rd%20Year/Portfolio_Assignment/client/src/components/Projects.jsx#L15-L22)
* **Problem:** `.filter()` executes on every single component render (e.g. every time a project row accordion is hovered or expanded).
* **The Fix:** Wrap the filter calculation in `useMemo`:
  ```javascript
  const filteredProjects = useMemo(() => {
    return projects.filter((item) => {
      if (filter === 'All') return true;
      if (filter === 'AI & ML') return item.category.includes('AI') || item.category.includes('VISION');
      if (filter === 'Full-Stack') return item.category.includes('FULL-STACK');
      return true;
    });
  }, [projects, filter]);
  ```

---

### Bug 5: String Schema Types for Price in `FS_20/models/products.js`
* **File & Line:** [`D:/_3rd Year/FS_20/models/products.js` Lines 7–10](file:///D:/_3rd%20Year/FS_20/models/products.js#L7-L10)
* **Current Code:**
  ```javascript
  price: {
    type: String, // BAD! Stored as String!
    required: true
  }
  ```
* **Why it's a major exam trap:** Because `price` is stored as a `String`, MongoDB numerical queries (`$gt`, `$lt`, `$max`, `$avg`, `sort({ price: 1 })`) will fail or sort lexicographically (`"1000"` comes before `"20"`)!
* **The Fix:** Change type to `Number`:
  ```javascript
  price: {
    type: Number,
    required: true,
    min: 0
  }
  ```
