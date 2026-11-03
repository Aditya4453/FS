# Topic 10: Express Routes and Middleware

---

### 1. Plain-Language Explanation (Fast Pace)
Express is a lightweight web server framework built around a "Pipeline of Functions."
* **Middleware:** A function that sits in the pipeline between receiving the incoming request and sending the outgoing response:
  ```javascript
  function myMiddleware(req, res, next) {
    // 1. inspect or modify req/res
    // 2. call next() to pass control to the next handler
  }
  ```
* If a middleware does not call `next()` or send a response, the request hangs forever!
* **Routes:** Endpoints defined by HTTP method and URL path: `app.get('/path', handler)` or `app.post('/path', handler)`.
* **Express Router (`express.Router()`):** Splits routes into clean separate files instead of putting 50 routes in one file.

---

### 2. The Root Idea Behind It
Web applications have cross-cutting concerns: every single request needs CORS headers checked, cookies parsed, authentication tokens verified, and bodies parsed into JSON. Middleware allows writing this logic once and applying it globally across all endpoints.

---

### 3. Exact Code From Your Files
**Source File:** [`D:/_3rd Year/Portfolio_Assignment/server/server.js`](file:///D:/_3rd%20Year/Portfolio_Assignment/server/server.js#L28-L55)  
**Lines 28–55 & 105–112:**
```javascript
28: app.use(cors({ ... }));
35: app.use(express.json());
36: app.use(express.urlencoded({ extended: true }));
37: 
38: // Request logger middleware
39: app.use((req, res, next) => {
40:   if (!IS_PROD) {
41:     console.log(`[${new Date().toLocaleTimeString()}] ${req.method} ${req.url}`);
42:   }
43:   next();
44: });
45: 
46: // ── API Routes ──────────────────────────────────────────────────────────────
47: app.use('/api/contact',   contactRoutes);
48: app.use('/api/resume',    resumeRoutes);
49: app.use('/api/portfolio', portfolioRoutes);
...
105: // ── Global error handler middleware ─────────────────────────────────────────
106: app.use((err, req, res, next) => {
107:   console.error('[Global Error]', err.stack);
108:   res.status(500).json({
109:     success: false,
110:     error:   'Internal server error.',
111:   });
112: });
```

#### Line-by-Line Walkthrough:
* **Line 35:** `app.use(express.json())`: Built-in middleware. Reads raw streaming bytes from request body and parses them into a JS object at `req.body`.
* **Line 39–44:** Custom middleware: Logs timestamp, HTTP method, and URL. **Line 43** calls `next()` to hand off control to the next middleware!
* **Line 47:** `app.use('/api/contact', contactRoutes)`: Modular route mounting. All routes defined in `contactRoutes.js` are prefixed with `/api/contact`.
* **Line 106–112:** Global error handling middleware. Identified by having **4 arguments**: `(err, req, res, next)`. Any error thrown in any route jumps straight to this handler.

---

### 4. How It Connects to Other Topics
1. Request arrives from React `fetch('/api/contact')`.
2. Enters Middleware Pipeline: `cors()` -> `express.json()` (populates `req.body`) -> Custom logger (`console.log`) -> `next()`.
3. Router matches `/api/contact` -> Calls controller `submitContact(req, res)`.
4. Controller writes to MongoDB via Mongoose.
5. Controller calls `res.status(201).json()`. Request exits the pipeline!

---

### 5. Common Mistakes & Exact Error Messages
* **Mistake 1: Forgetting to call `next()` in middleware:**
  ```javascript
  app.use((req, res, next) => {
    console.log("Logged!");
    // forgot next()!
  });
  ```
  * **Result:** Client request hangs forever until browser times out with `504 Gateway Timeout`.
* **Mistake 2: Placing `express.json()` AFTER route definitions:**
  ```javascript
  app.post('/api/contact', contactController);
  app.use(express.json()); // TOO LATE!
  ```
  * **Result:** `req.body` is `undefined` inside `contactController`.
* **Mistake 3: Error handler missing the 4th `next` argument:**
  ```javascript
  app.use((err, req, res) => {...}); // ONLY 3 ARGS
  ```
  * Express will treat it as normal middleware, NOT an error handler!

---

### 6. Optimized & Best-Practice Version
Always structure router modules and validate middleware order:

```javascript
// routes/productRoutes.js
const express = require('express');
const router = express.Router();

router.get('/', getAllProducts);
router.get('/:id', getProductById);
router.post('/', validateBodyMiddleware, createProduct);

module.exports = router;
```
