# Topic 06: `fetch`, `async` / `await`, and Promises

---

### 1. Plain-Language Explanation (Fast Pace)
Network requests take time (e.g. 50ms to 2000ms). If JavaScript paused and waited synchronously for the network response, your whole browser tab would freeze (no scrolling, clicking, or typing).
* A **Promise** is a JavaScript object representing an ongoing background task with three states: *Pending*, *Fulfilled* (success), or *Rejected* (error).
* `async`: Keyword put before a function to allow it to pause internally using `await`.
* `await`: Keyword that pauses the execution of *only this function* until the Promise resolves, while letting the rest of the browser keep running smoothly.
* `fetch(url, options)`: Built-in browser function that makes an HTTP request and returns a Promise resolving to a Response object.

---

### 2. The Root Idea Behind It
Before `async/await`, developers used callback functions (`callback hell`) and chained `.then().then().catch()`. Chaining `.then()` makes code deeply nested and error handling difficult. `async/await` is syntactic sugar over Promises that makes asynchronous code read sequentially like regular synchronous code.

---

### 3. Exact Code From Your Files
**Source File:** [`D:/_3rd Year/Portfolio_Assignment/client/src/components/Contact.jsx`](file:///D:/_3rd%20Year/Portfolio_Assignment/client/src/components/Contact.jsx#L21-L54)  
**Lines 21–54:**
```javascript
21:   const handleSubmit = async (e) => {
22:     e.preventDefault();
...
31:     try {
32:       setBtnState('loading');
33: 
34:       const response = await fetch('/api/contact', {
35:         method: 'POST',
36:         headers: { 'Content-Type': 'application/json' },
37:         body: JSON.stringify(formData),
38:       });
39: 
40:       const result = await response.json();
41: 
42:       if (response.ok && result.success) {
43:         setBtnState('success');
44:         setFormData({ name: '', email: '', subject: '', message: '' });
45:         // Revert button back to idle after 3.5 s
46:         setTimeout(() => setBtnState('idle'), 3500);
47:       } else {
48:         throw new Error(result.error || 'Failed to submit.');
49:       }
50:     } catch (err) {
51:       setErrorMsg(err.message || 'Network error — please try again.');
52:       setBtnState('error');
53:       setTimeout(() => setBtnState('idle'), 3500);
54:     }
55:   };
```

#### Line-by-Line Walkthrough:
* **Line 21:** `const handleSubmit = async (e) => {`: Marks the handler function `async`.
* **Line 31:** `try {`: Opens a structured error boundary block.
* **Line 34:** `const response = await fetch(...)`: Pauses this function until HTTP headers arrive from Express.
* **Line 40:** `const result = await response.json()`: Second `await`! Reading and parsing the HTTP body stream is also asynchronous.
* **Line 42:** `if (response.ok && result.success)`: Checks HTTP status (`response.ok` is true for 200–299 status codes).
* **Line 48:** `throw new Error(...)`: If backend returned 400 or 500 status, manually throws an error to divert into the `catch` block.
* **Line 50:** `} catch (err) {`: Catches network failures, DNS failures, or thrown errors, and informs the user via `setErrorMsg`.

---

### 4. How It Connects to Other Topics
1. **Frontend Trigger:** User submits form.
2. **Async Initiation (Here):** `fetch()` initiates HTTP request across network to Express.
3. **Backend Asynchronous Handling:** Inside Express [`FS_20/server.js`](file:///D:/_3rd%20Year/FS_20/server.js#L18):
   ```javascript
   server.get('/products', async (req, res) => {
     const data = await products.find(); // Async database query!
     res.send(data);
   });
   ```
4. **Database Execution:** MongoDB driver asynchronously fetches data from disk/RAM.
5. **Return Flow:** Express sends response -> client `await response.json()` unlocks -> state updates.

---

### 5. Common Mistakes & Exact Error Messages
* **Mistake 1: Forgetting `await` on `response.json()`:**
  ```javascript
  const result = response.json(); // missing await!
  console.log(result.data);
  ```
  * **Result:** `result` is a `Promise { <pending> }`. `result.data` is `undefined`.
* **Mistake 2: Assuming `fetch` throws on HTTP 400 or 500:**
  * **Fact:** Native `fetch()` only rejects if there is a network crash (e.g. offline, DNS failure, CORS block). A 404 or 500 response is still considered a resolved promise! You MUST check `response.ok`.
* **Mistake 3: Unnecessary await on non-promises:**
  * Seen in [`FS_20/server.js` Line 23](file:///D:/_3rd%20Year/FS_20/server.js#L23):
    ```javascript
    const data = await req.body; // req.body is already a synchronous parsed object!
    ```

---

### 6. Optimized & Best-Practice Version
Always inspect `response.ok`, parse JSON cleanly, and provide meaningful fallback messages:

```javascript
const postData = async (url, payload) => {
  try {
    const res = await fetch(url, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(payload)
    });

    const data = await res.json();
    if (!res.ok) {
      throw new Error(data.error || `Server returned status ${res.status}`);
    }
    return data;
  } catch (err) {
    console.error("Fetch failed:", err.message);
    throw err;
  }
};
```
