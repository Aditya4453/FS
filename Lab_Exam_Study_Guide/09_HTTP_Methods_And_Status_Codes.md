# Topic 09: HTTP Methods and Status Codes

---

### 1. Plain-Language Explanation (Fast Pace)
HTTP is the standard language of the web. Every request from the client includes a **Method (Verb)** declaring its intent, and every response from the server returns a 3-digit **Status Code** declaring the result.
* **Methods:**
  * `GET`: Retrieve data (no request body, idempotent).
  * `POST`: Create new data (has request body).
  * `PUT`: Completely replace an existing resource.
  * `PATCH`: Partially update specific fields of a resource.
  * `DELETE`: Remove a resource.
* **Status Codes (The 5 Families):**
  * `2xx (Success)`: `200 OK` (standard read/update), `201 Created` (new document saved).
  * `3xx (Redirection)`: `301 Moved Permanently`, `304 Not Modified` (cached).
  * `4xx (Client Error)`: `400 Bad Request` (missing fields/invalid data), `401 Unauthorized` (not logged in), `403 Forbidden` (no permission), `404 Not Found` (endpoint doesn't exist).
  * `5xx (Server Error)`: `500 Internal Server Error` (unhandled crash or database disconnected).

---

### 2. The Root Idea Behind It
Without standard HTTP status codes, every backend would invent custom formats. Frontends, caching proxies, CDNs, and API gateways rely on standard codes to know if a request succeeded, can be cached, or must be retried.

---

### 3. Exact Code From Your Files
**Source File 1:** [`D:/_3rd Year/Portfolio_Assignment/server/controllers/contactController.js`](file:///D:/_3rd%20Year/Portfolio_Assignment/server/controllers/contactController.js#L27-L48)  
**Lines 27–48:**
```javascript
27:     // Field validations
28:     if (!name || !email || !message) {
29:       return res.status(400).json({
30:         success: false,
31:         error: 'Please provide name, email, and message.',
32:       });
33:     }
...
43:     const newContact = await Contact.create({ ... });
44: 
45:     return res.status(201).json({
46:       success: true,
47:       message: 'Message sent successfully!',
48:       data: { id: newContact._id },
49:     });
```

**Source File 2:** [`D:/_3rd Year/Portfolio_Assignment/server/server.js`](file:///D:/_3rd%20Year/Portfolio_Assignment/server/server.js#L57-L64)  
**Lines 57–64:**
```javascript
57: app.get('/api/health', (req, res) => {
58:   res.status(200).json({
59:     status:    'healthy',
60:     timestamp: new Date().toISOString(),
61:   });
62: });
```

#### Line-by-Line Walkthrough:
* **Line 29:** `res.status(400).json(...)`: Returns HTTP 400 Bad Request because the client failed to supply required fields (`name`, `email`, `message`).
* **Line 45:** `res.status(201).json(...)`: Returns HTTP 201 Created because a new record was permanently inserted into the MongoDB database.
* **Line 58:** `res.status(200).json(...)`: Returns HTTP 200 OK for standard health check.

---

### 4. How It Connects to Other Topics
1. **Client Action:** Client executes `fetch('/api/contact', { method: 'POST', ... })`.
2. **Server Routing:** Express matches `app.post('/api/contact')`.
3. **Validation & Storage:** If valid, MongoDB saves record -> Controller returns `res.status(201).json()`.
4. **Client Inspection:** React client receives response:
   ```javascript
   if (response.ok && response.status === 201) {
     // success alert!
   }
   ```

---

### 5. Common Mistakes & Exact Error Messages
* **Mistake 1: Returning status 200 on errors:**
  ```javascript
  res.status(200).json({ error: "Password required" }); // BAD PRACTICE!
  ```
  * Client's `response.ok` evaluates to `true`, misleading the client into treating errors as successes.
* **Mistake 2: Missing `return` after sending response:**
  ```javascript
  if (!name) res.status(400).json({ error: "Missing name" });
  res.status(200).json({ success: true }); // TRIES TO SEND AGAIN!
  ```
  * **Crash Error:** `Error [ERR_HTTP_HEADERS_SENT]: Cannot set headers after they are sent to the client`.

---

### 6. Optimized & Best-Practice Version
Always pair return with status codes and structured response schemas:

```javascript
// Express CRUD Status Standards
app.get('/items', (req, res) => res.status(200).json(items));       // 200 OK
app.post('/items', (req, res) => res.status(201).json(newItem));    // 201 Created
app.delete('/items/:id', (req, res) => res.status(204).send());     // 204 No Content
app.use((err, req, res, next) => res.status(500).json({ error: err.message })); // 500
```
