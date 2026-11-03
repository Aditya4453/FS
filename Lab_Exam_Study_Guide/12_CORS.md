# Topic 12: CORS (Cross-Origin Resource Sharing)

---

### 1. Plain-Language Explanation (Fast Pace)
**CORS** is a browser security mechanism enforced by the **Same-Origin Policy**.
* An **Origin** is defined as: `protocol + domain + port` (e.g. `http://localhost:5173`).
* If your React frontend runs on `http://localhost:5173` (Vite) and your Express server runs on `http://localhost:5000`, they are **different origins**!
* By default, browsers BLOCK frontend JavaScript from reading responses from a different origin to prevent malicious websites from stealing your bank cookies or private session data.
* **The Solution:** The server must send special HTTP response headers (like `Access-Control-Allow-Origin: *`) giving the browser explicit permission to share the data.

---

### 2. The Root Idea Behind It
The Same-Origin Policy is the cornerstone of web security. Without CORS restrictions, any random tab open in your browser could run `fetch('http://localhost:8080/my-passwords')` and read your local server's private data. CORS allows server operators to whitelist trusted domains.

---

### 3. Exact Code From Your Files
**Source File:** [`D:/_3rd Year/Portfolio_Assignment/server/server.js`](file:///D:/_3rd%20Year/Portfolio_Assignment/server/server.js#L28-L34)  
**Lines 28–34:**
```javascript
28: app.use(cors({
29:   origin: IS_PROD
30:     ? ['https://portfolio-apk.onrender.com', 'https://client-ebon-theta-89.vercel.app']
31:     : '*',
32:   methods: ['GET', 'POST', 'PUT', 'DELETE', 'OPTIONS'],
33:   allowedHeaders: ['Content-Type', 'Authorization'],
34: }));
```

#### Line-by-Line Walkthrough:
* **Line 28:** Mounts the `cors` middleware at the very top of your server before any routes.
* **Line 29–31:** `origin`:
  * In development (`IS_PROD` is false): uses `'*'`, allowing any local port (e.g. Vite on 5173 or 3000) to communicate with the server.
  * In production: restricts access strictly to your deployed frontend domains (`https://client-ebon-theta-89.vercel.app`), preventing unauthorized third-party sites from querying your API!
* **Line 32:** `methods`: Whitelists permitted HTTP verbs (`GET`, `POST`, `PUT`, `DELETE`, `OPTIONS`).
* **Line 33:** `allowedHeaders`: Allows headers like `'Content-Type'` (required for JSON payloads).

---

### 4. How It Connects to Other Topics
1. **Preflight Request:** For `POST` requests with `'Content-Type': 'application/json'`, the browser automatically sends an HTTP `OPTIONS` request first ("Can I send this?").
2. **CORS Check:** Express `cors()` middleware intercepts `OPTIONS` and responds with `Access-Control-Allow-Origin` and `Access-Control-Allow-Headers`.
3. **Actual Request:** Browser inspects the response. If allowed, it sends the actual `POST` request.
4. **If Blocked:** Browser drops the response and throws an error in the developer console before React can see it!

---

### 5. Common Mistakes & Exact Error Messages
* **Mistake 1: Forgetting CORS middleware on Express:**
  * **Browser Console Error:** `Access to fetch at 'http://localhost:5000/api/products' from origin 'http://localhost:5173' has been blocked by CORS policy: No 'Access-Control-Allow-Origin' header is present on the requested resource.`
* **Mistake 2: Placing CORS middleware AFTER routes:**
  ```javascript
  app.get('/api/products', ...);
  app.use(cors()); // Route executed before CORS headers were added!
  ```
* **Mistake 3: Missing OPTIONS method in whitelist:**
  * Results in Preflight check failure: `Response to preflight request doesn't pass access control check: It does not have HTTP ok status.`

---

### 6. Optimized & Best-Practice Version
In Vite development, you can completely avoid CORS issues by setting up a proxy in `vite.config.js`:

```javascript
// client/vite.config.js
export default defineConfig({
  server: {
    proxy: {
      '/api': 'http://localhost:5000', // Browser talks to 5173; Vite proxies to 5000!
    },
  },
});
```
